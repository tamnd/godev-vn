---
title: Ghi chú phát hành Go 1.11
---

<!--
NOTE: In this document and others in this directory, the convention is to
set fixed-width phrases with non-fixed-width spaces, as in
`hello` `world`.
Do not send CLs removing the interior tags from such phrases.
-->

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu về Go 1.11 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.11, ra đời sáu tháng sau [Go 1.10](go1.10). Phần lớn các thay đổi nằm ở việc triển khai trình biên dịch, runtime và thư viện. Như thường lệ, bản phát hành duy trì [cam kết tương thích Go 1](/doc/go1compat.html). Chúng tôi mong rằng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

## Thay đổi về ngôn ngữ {#language}

Không có thay đổi nào về đặc tả ngôn ngữ.

## Các nền tảng {#ports}

<!-- CL 94255, CL 115038, etc -->
Như [đã thông báo trong ghi chú phát hành Go 1.10](go1.10#ports), Go 1.11 giờ yêu cầu OpenBSD 6.2 trở lên, macOS 10.10 Yosemite trở lên, hoặc Windows 7 trở lên; hỗ trợ cho các phiên bản trước của các hệ điều hành này đã bị xóa.

<!-- CL 121657 -->
Go 1.11 hỗ trợ bản phát hành OpenBSD 6.4 sắp tới. Do các thay đổi trong kernel OpenBSD, các phiên bản Go cũ hơn sẽ không hoạt động trên OpenBSD 6.4.

Có [các vấn đề đã biết](/issue/25206) với NetBSD trên phần cứng i386.

<!-- CL 107935 -->
Trình phát hiện race giờ được hỗ trợ trên `linux/ppc64le` và ở mức độ thấp hơn trên `netbsd/amd64`. Hỗ trợ trình phát hiện race trên NetBSD có [các vấn đề đã biết](/issue/26403).

<!-- CL 109255 -->
Memory sanitizer (`-msan`) giờ được hỗ trợ trên `linux/arm64`.

<!-- CL 93875 -->
Các chế độ build `c-shared` và `c-archive` giờ được hỗ trợ trên `freebsd/amd64`.

<!-- CL 108475 -->
Trên các hệ thống MIPS 64-bit, các cài đặt biến môi trường mới `GOMIPS64=hardfloat` (mặc định) và `GOMIPS64=softfloat` chọn việc sử dụng lệnh phần cứng hay mô phỏng phần mềm cho các phép tính dấu phẩy động. Đối với các hệ thống 32-bit, biến môi trường vẫn là `GOMIPS`, như [đã thêm trong Go 1.10](go1.10#mips).

<!-- CL 107475 -->
Trên các hệ thống ARM soft-float (`GOARM=5`), Go giờ sử dụng một interface dấu phẩy động phần mềm hiệu quả hơn. Điều này trong suốt với mã Go, nhưng code hợp ngữ ARM sử dụng các lệnh dấu phẩy động không được bảo vệ trên GOARM sẽ bị hỏng và phải được chuyển sang [interface mới](/cl/107475).

<!-- CL 94076 -->
Go 1.11 trên ARMv7 không còn yêu cầu Linux kernel được cấu hình với `KUSER_HELPERS`. Cài đặt này được bật trong các cấu hình kernel mặc định, nhưng đôi khi bị tắt trong các cấu hình thu gọn.

### WebAssembly {#wasm}

Go 1.11 thêm một cổng thử nghiệm cho [WebAssembly](https://webassembly.org) (`js/wasm`).

Các chương trình Go hiện tại biên dịch sang một module WebAssembly bao gồm Go runtime để lập lịch goroutine, thu gom rác, map, v.v. Kết quả là, kích thước tối thiểu là khoảng 2 MB, hoặc 500 KB khi nén. Các chương trình Go có thể gọi vào JavaScript bằng gói thử nghiệm mới [`syscall/js`](/pkg/syscall/js/). Kích thước binary và khả năng tương tác với các ngôn ngữ khác chưa được ưu tiên nhưng có thể được giải quyết trong các bản phát hành tương lai.

Do việc thêm giá trị `GOOS` mới "`js`" và giá trị `GOARCH` "`wasm`", các tệp Go được đặt tên `*_js.go` hoặc `*_wasm.go` giờ sẽ bị [bỏ qua bởi các công cụ Go](/pkg/go/build/#hdr-Build_Constraints) trừ khi các giá trị GOOS/GOARCH đó đang được sử dụng. Nếu bạn có các tên tệp hiện tại khớp với các pattern đó, bạn sẽ cần đổi tên chúng.

Thông tin thêm có thể được tìm thấy trên [trang wiki WebAssembly](/wiki/WebAssembly).

### Các giá trị GOARCH cho RISC-V được đặt trước {#riscv}

<!-- CL 106256 -->
Trình biên dịch Go chính chưa hỗ trợ kiến trúc RISC-V nhưng chúng tôi đã đặt trước các giá trị `GOARCH` "`riscv`" và "`riscv64`", như được sử dụng bởi Gccgo, hỗ trợ RISC-V. Điều này có nghĩa là các tệp Go được đặt tên `*_riscv.go` giờ cũng sẽ bị [bỏ qua bởi các công cụ Go](/pkg/go/build/#hdr-Build_Constraints) trừ khi các giá trị GOOS/GOARCH đó đang được sử dụng.

## Công cụ {#tools}

### Module, phiên bản gói và quản lý dependency {#modules}

Go 1.11 thêm hỗ trợ sơ bộ cho [một khái niệm mới được gọi là "module,"](/cmd/go/#hdr-Modules__module_versions__and_more) một giải pháp thay thế cho GOPATH với hỗ trợ tích hợp cho versioning và phân phối gói. Khi sử dụng module, các nhà phát triển không còn bị giới hạn làm việc bên trong GOPATH, thông tin phụ thuộc phiên bản rõ ràng nhưng nhẹ, và các build đáng tin cậy và có thể tái tạo hơn.

Hỗ trợ module được coi là thử nghiệm. Các chi tiết có khả năng thay đổi theo phản hồi từ người dùng Go 1.11, và chúng tôi có nhiều công cụ được lên kế hoạch. Mặc dù chi tiết hỗ trợ module có thể thay đổi, các dự án chuyển đổi sang module bằng Go 1.11 sẽ tiếp tục hoạt động với Go 1.12 và các phiên bản sau. Nếu bạn gặp lỗi khi sử dụng module, vui lòng [báo cáo vấn đề](/issue/new) để chúng tôi có thể sửa chúng. Để biết thêm thông tin, xem [tài liệu lệnh `go`](/cmd/go#hdr-Modules__module_versions__and_more).

### Hạn chế đường dẫn import {#importpath}

Vì hỗ trợ module Go gán ý nghĩa đặc biệt cho ký hiệu `@` trong các thao tác dòng lệnh, lệnh `go` giờ không cho phép sử dụng các đường dẫn import chứa ký hiệu `@`. Các đường dẫn import như vậy chưa bao giờ được `go` `get` cho phép, vì vậy hạn chế này chỉ có thể ảnh hưởng đến người dùng xây dựng cây GOPATH tùy chỉnh bằng các phương tiện khác.

### Nạp gói {#gopackages}

Gói mới [`golang.org/x/tools/go/packages`](https://godoc.org/golang.org/x/tools/go/packages) cung cấp một API đơn giản để xác định và nạp các gói mã nguồn Go. Mặc dù chưa phải là một phần của thư viện chuẩn, đối với nhiều tác vụ nó thay thế hiệu quả gói [`go/build`](/pkg/go/build), API của gói này không thể hỗ trợ đầy đủ module. Vì nó chạy một lệnh truy vấn ngoài như [`go list`](/cmd/go/#hdr-List_packages) để lấy thông tin về các gói Go, nó cho phép xây dựng các công cụ phân tích hoạt động tốt như nhau với các hệ thống build thay thế như [Bazel](https://bazel.build) và [Buck](https://buckbuild.com).

### Yêu cầu bộ nhớ đệm build {#gocache}

Go 1.11 sẽ là bản phát hành cuối cùng hỗ trợ đặt biến môi trường `GOCACHE=off` để tắt [bộ nhớ đệm build](/cmd/go/#hdr-Build_and_test_caching), được giới thiệu trong Go 1.10. Bắt đầu từ Go 1.12, bộ nhớ đệm build sẽ là bắt buộc, như một bước hướng tới việc loại bỏ `$GOPATH/pkg`. Hỗ trợ nạp module và gói được mô tả ở trên đã yêu cầu bộ nhớ đệm build được bật. Nếu bạn đã tắt bộ nhớ đệm build để tránh các vấn đề bạn gặp phải, vui lòng [báo cáo vấn đề](/issue/new) để cho chúng tôi biết về chúng.

### Trình biên dịch {#compiler}

<!-- CL 109918 -->
Nhiều hàm hơn giờ đủ điều kiện để được inline theo mặc định, bao gồm các hàm gọi `panic`.

<!-- CL 97375 -->
Trình biên dịch giờ hỗ trợ thông tin cột trong [các chỉ thị dòng](/cmd/compile/#hdr-Compiler_Directives).

<!-- CL 106797 -->
Một định dạng dữ liệu xuất gói mới đã được giới thiệu. Điều này nên trong suốt với người dùng cuối, ngoại trừ việc tăng tốc thời gian build cho các dự án Go lớn. Nếu nó gây ra vấn đề, có thể tắt lại bằng cách truyền `-gcflags=all=-iexport=false` cho công cụ `go` khi build một binary.

<!-- CL 100459 -->
Trình biên dịch giờ từ chối các biến chưa sử dụng được khai báo trong guard type switch, chẳng hạn như `x` trong ví dụ sau:

	func f(v interface{}) {
		switch x := v.(type) {
		}
	}

Điều này đã bị từ chối bởi cả `gccgo` và [go/types](/pkg/go/types/).

### Trình hợp ngữ {#assembler}

<!-- CL 113315 -->
Trình hợp ngữ cho `amd64` giờ chấp nhận các lệnh AVX512.

### Gỡ lỗi {#debugging}

<!-- CL 100738, CL 93664 -->
Trình biên dịch giờ tạo ra thông tin gỡ lỗi chính xác hơn đáng kể cho các binary được tối ưu hóa, bao gồm thông tin vị trí biến, số dòng và vị trí breakpoint. Điều này nên giúp gỡ lỗi các binary được biên dịch _không có_ `-N`&nbsp;`-l` là khả thi. Vẫn còn các hạn chế về chất lượng của thông tin gỡ lỗi, một số là cơ bản, và một số sẽ tiếp tục cải thiện với các bản phát hành tương lai.

<!-- CL 118276 -->
Các section DWARF giờ được nén theo mặc định vì thông tin gỡ lỗi mở rộng và chính xác hơn được tạo bởi trình biên dịch. Điều này trong suốt với hầu hết các công cụ ELF (chẳng hạn như debugger trên Linux và \*BSD) và được hỗ trợ bởi debugger Delve trên tất cả các nền tảng, nhưng có hỗ trợ hạn chế trong các công cụ gốc trên macOS và Windows. Để tắt nén DWARF, truyền `-ldflags=-compressdwarf=false` cho công cụ `go` khi build một binary.

<!-- CL 109699 -->
Go 1.11 thêm hỗ trợ thử nghiệm cho việc gọi các hàm Go từ bên trong một debugger. Điều này hữu ích, ví dụ, khi gọi các phương thức `String` khi dừng tại một breakpoint. Điều này hiện chỉ được hỗ trợ bởi Delve (phiên bản 1.1.0 trở lên).

### Test {#test}

Kể từ Go 1.10, lệnh `go`&nbsp;`test` chạy `go`&nbsp;`vet` trên gói đang được kiểm thử, để xác định các vấn đề trước khi chạy kiểm thử. Vì `vet` kiểm tra kiểu mã với [go/types](/pkg/go/types/) trước khi chạy, các kiểm thử không kiểm tra kiểu giờ sẽ thất bại. Cụ thể, các kiểm thử chứa biến chưa sử dụng bên trong closure được biên dịch với Go 1.10, vì trình biên dịch Go chấp nhận sai ([Issue #3059](/issues/3059)), nhưng giờ sẽ thất bại, vì `go/types` báo cáo đúng lỗi "unused variable" trong trường hợp này.

<!-- CL 102696 -->
Cờ `-memprofile` cho `go`&nbsp;`test` giờ mặc định thành profile "allocs", ghi lại tổng số byte được phân bổ kể từ khi kiểm thử bắt đầu (bao gồm các byte đã được thu gom rác).

### Vet {#vet}

<!-- CL 108555 -->
Lệnh [`go`&nbsp;`vet`](/cmd/vet/) giờ báo cáo lỗi nghiêm trọng khi gói đang được phân tích không kiểm tra kiểu thành công. Trước đây, một lỗi kiểm tra kiểu chỉ khiến một cảnh báo được in, và `vet` thoát với trạng thái 1.

<!-- CL 108559 -->
Ngoài ra, [`go`&nbsp;`vet`](/cmd/vet) đã trở nên mạnh mẽ hơn khi kiểm tra định dạng cho các wrapper `printf`. Vet giờ phát hiện lỗi trong ví dụ này:

	func wrapper(s string, args ...interface{}) {
		fmt.Printf(s, args...)
	}

	func main() {
		wrapper("%s", 42)
	}

### Trace {#trace}

<!-- CL 63274 -->
Với [API chú thích người dùng](/pkg/runtime/trace/#hdr-User_annotation) của gói `runtime/trace` mới, người dùng có thể ghi lại thông tin cấp ứng dụng trong các trace thực thi và tạo các nhóm goroutine liên quan. Lệnh `go`&nbsp;`tool`&nbsp;`trace` trực quan hóa thông tin này trong chế độ xem trace và trang phân tích task/region người dùng mới.

### Cgo {#cgo}

Kể từ Go 1.10, cgo đã dịch một số kiểu con trỏ C sang kiểu Go `uintptr`. Các kiểu này bao gồm hệ thống phân cấp `CFTypeRef` trong framework CoreFoundation của Darwin và hệ thống phân cấp `jobject` trong giao diện JNI của Java. Trong Go 1.11, một số cải tiến đã được thực hiện cho mã phát hiện các kiểu này. Mã sử dụng các kiểu này có thể cần một số cập nhật. Xem [ghi chú phát hành Go 1.10](go1.10.html#cgo) để biết chi tiết. <!-- CL 126275, CL 127156, CL 122217, CL 122575, CL 123177 -->

### Lệnh go {#go_command}

<!-- CL 126656 -->
Biến môi trường `GOFLAGS` giờ có thể được sử dụng để đặt các cờ mặc định cho lệnh `go`. Điều này hữu ích trong một số tình huống nhất định. Liên kết có thể chậm đáng kể trên các hệ thống kém mạnh do DWARF, và người dùng có thể muốn đặt `-ldflags=-w` theo mặc định. Đối với module, một số người dùng và hệ thống CI sẽ muốn sử dụng vendor luôn, vì vậy họ nên đặt `-mod=vendor` theo mặc định. Để biết thêm thông tin, xem [tài liệu lệnh `go`](/cmd/go/#hdr-Environment_variables).

### Godoc {#godoc}

Go 1.11 sẽ là bản phát hành cuối cùng hỗ trợ giao diện dòng lệnh của `godoc`. Trong các bản phát hành tương lai, `godoc` sẽ chỉ là một web server. Người dùng nên sử dụng `go` `doc` để có đầu ra trợ giúp dòng lệnh thay thế.

<!-- CL 85396, CL 124495 -->
Web server `godoc` giờ hiển thị phiên bản Go nào giới thiệu các tính năng API mới. Phiên bản Go ban đầu của các kiểu, hàm và phương thức được hiển thị căn phải. Ví dụ, xem [`UserCacheDir`](/pkg/os/#UserCacheDir), với "1.11" ở bên phải. Đối với các trường struct, các comment inline được thêm khi trường struct được thêm vào trong một phiên bản Go khác với khi bản thân kiểu được giới thiệu. Để xem ví dụ về trường struct, xem [`ClientTrace.Got1xxResponse`](/pkg/net/http/httptrace/#ClientTrace.Got1xxResponse).

### Gofmt {#gofmt}

Một chi tiết nhỏ của định dạng mặc định mã nguồn Go đã thay đổi. Khi định dạng các danh sách biểu thức với comment inline, các comment được căn chỉnh theo một heuristic. Tuy nhiên, trong một số trường hợp việc căn chỉnh sẽ bị tách ra quá dễ dàng, hoặc giới thiệu quá nhiều khoảng trắng. Heuristic đã được thay đổi để hoạt động tốt hơn cho mã do người viết.

Lưu ý rằng những loại cập nhật nhỏ cho gofmt như thế này được mong đợi sẽ xảy ra theo thời gian. Nói chung, các hệ thống cần định dạng nhất quán của mã nguồn Go nên sử dụng một phiên bản cụ thể của binary `gofmt`. Xem tài liệu gói [go/format](/pkg/go/format/) để biết thêm thông tin.

### Run {#run}

<!-- CL 109341 -->
Lệnh [`go`&nbsp;`run`](/cmd/go/) giờ cho phép một đường dẫn import đơn, tên thư mục hoặc pattern khớp với một gói đơn. Điều này cho phép `go`&nbsp;`run`&nbsp;`pkg` hoặc `go`&nbsp;`run`&nbsp;`dir`, quan trọng nhất là `go`&nbsp;`run`&nbsp;`.`

## Runtime {#runtime}

<!-- CL 85887 -->
Runtime giờ sử dụng layout heap thưa, vì vậy không còn giới hạn kích thước heap Go nữa (trước đây, giới hạn là 512GiB). Điều này cũng sửa các lỗi "address space conflict" hiếm gặp trong các binary Go/C hỗn hợp hoặc các binary được biên dịch với `-race`.

<!-- CL 108679, CL 106156 -->
Trên macOS và iOS, runtime giờ sử dụng `libSystem.dylib` thay vì gọi trực tiếp kernel. Điều này nên làm cho các binary Go tương thích hơn với các phiên bản tương lai của macOS và iOS. Gói [syscall](/pkg/syscall) vẫn thực hiện các system call trực tiếp; việc sửa điều này được lên kế hoạch cho một bản phát hành tương lai.

## Hiệu suất {#performance}

Như thường lệ, các thay đổi rất đa dạng đến mức khó có thể đưa ra các nhận xét chính xác về hiệu suất. Hầu hết các chương trình nên chạy nhanh hơn một chút, nhờ mã được tạo tốt hơn và các tối ưu hóa trong thư viện cốt lõi.

<!-- CL 74851 -->
Có nhiều thay đổi hiệu suất đối với gói `math/big` cũng như nhiều thay đổi trên toàn cây dành riêng cho `GOARCH=arm64`.

### Trình biên dịch {#performance-compiler}

<!-- CL 110055 -->
Trình biên dịch giờ tối ưu hóa các thao tác xóa map có dạng:

	for k := range m {
		delete(m, k)
	}

<!-- CL 109517 -->
Trình biên dịch giờ tối ưu hóa mở rộng slice có dạng `append(s,`&nbsp;`make([]T,`&nbsp;`n)...)`.

<!-- CL 100277, CL 105635, CL 109776 -->
Trình biên dịch giờ thực hiện kiểm tra biên giới và loại bỏ nhánh tích cực hơn đáng kể. Đáng chú ý, giờ nó nhận ra các quan hệ bắc cầu, vì vậy nếu `i<j` và `j<len(s)`, nó có thể sử dụng các thực tế này để loại bỏ kiểm tra biên giới cho `s[i]`. Nó cũng hiểu số học đơn giản như `s[i-10]` và có thể nhận ra nhiều trường hợp quy nạp hơn trong các vòng lặp. Hơn nữa, trình biên dịch giờ sử dụng thông tin biên giới để tối ưu hóa các thao tác dịch chuyển tích cực hơn.

## Thư viện chuẩn {#library}

Tất cả các thay đổi đối với thư viện chuẩn đều nhỏ.

### Các thay đổi nhỏ với thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ khác nhau đối với thư viện, được thực hiện với [cam kết tương thích Go 1](/doc/go1compat) trong tâm trí.

<!-- CL 115095: https://golang.org/cl/115095: yes (`go test pkg` now always builds pkg even if there are no test files): cmd/go: output coverage report even if there are no test files -->
<!-- CL 110395: https://golang.org/cl/110395: cmd/go, cmd/compile: use Windows response files to avoid arg length limits -->
<!-- CL 112436: https://golang.org/cl/112436: cmd/pprof: add readline support similar to upstream -->

#### [crypto](/pkg/crypto/)

<!-- CL 64451 -->
Một số thao tác crypto, bao gồm [`ecdsa.Sign`](/pkg/crypto/ecdsa/#Sign), [`rsa.EncryptPKCS1v15`](/pkg/crypto/rsa/#EncryptPKCS1v15) và [`rsa.GenerateKey`](/pkg/crypto/rsa/#GenerateKey), giờ đọc ngẫu nhiên thêm một byte ngẫu nhiên để đảm bảo các kiểm thử không phụ thuộc vào hành vi nội bộ.

<!-- crypto -->

#### [crypto/cipher](/pkg/crypto/cipher/)

<!-- CL 48510, CL 116435 -->
Hàm [`NewGCMWithTagSize`](/pkg/crypto/cipher/#NewGCMWithTagSize) mới triển khai Galois Counter Mode với độ dài tag không chuẩn để tương thích với các hệ thống mật mã hiện có.

<!-- crypto/cipher -->

#### [crypto/rsa](/pkg/crypto/rsa/)

<!-- CL 103876 -->
[`PublicKey`](/pkg/crypto/rsa/#PublicKey) giờ triển khai phương thức [`Size`](/pkg/crypto/rsa/#PublicKey.Size) trả về kích thước modulus tính bằng byte.

<!-- crypto/rsa -->

#### [crypto/tls](/pkg/crypto/tls/)

<!-- CL 85115 -->
Phương thức [`ExportKeyingMaterial`](/pkg/crypto/tls/#ConnectionState.ExportKeyingMaterial) mới của [`ConnectionState`](/pkg/crypto/tls/#ConnectionState) cho phép xuất keying material gắn với kết nối theo RFC 5705.

<!-- crypto/tls -->

#### [crypto/x509](/pkg/crypto/x509/)

<!-- CL 123355, CL 123695 -->
Hành vi cũ, không dùng nữa, coi trường `CommonName` như một hostname khi không có Subject Alternative Names, giờ bị tắt khi CN không phải là hostname hợp lệ. `CommonName` có thể bị bỏ qua hoàn toàn bằng cách thêm giá trị thử nghiệm `x509ignoreCN=1` vào biến môi trường `GODEBUG`. Khi CN bị bỏ qua, các chứng chỉ không có SAN xác thực theo các chuỗi với các ràng buộc tên thay vì trả về `NameConstraintsWithoutSANs`.

<!-- CL 113475 -->
Các hạn chế sử dụng khóa mở rộng lại chỉ được kiểm tra nếu chúng xuất hiện trong trường `KeyUsages` của [`VerifyOptions`](/pkg/crypto/x509/#VerifyOptions), thay vì luôn được kiểm tra. Điều này khớp với hành vi của Go 1.9 và trước đó.

<!-- CL 102699 -->
Giá trị được trả về bởi [`SystemCertPool`](/pkg/crypto/x509/#SystemCertPool) giờ được lưu trong bộ nhớ đệm và có thể không phản ánh các thay đổi hệ thống giữa các lần gọi.

<!-- crypto/x509 -->

#### [debug/elf](/pkg/debug/elf/)

<!-- CL 112115 -->
Nhiều hằng số [`ELFOSABI`](/pkg/debug/elf/#ELFOSABI_NONE) và [`EM`](/pkg/debug/elf/#EM_NONE) hơn đã được thêm vào.

<!-- debug/elf -->

#### [encoding/asn1](/pkg/encoding/asn1/)

<!-- CL 110561 -->
`Marshal` và [`Unmarshal`](/pkg/encoding/asn1/#Unmarshal) giờ hỗ trợ các chú thích lớp "private" cho các trường.

<!-- encoding/asn1 -->

#### [encoding/base32](/pkg/encoding/base32/)

<!-- CL 112516 -->
Decoder giờ nhất quán trả về `io.ErrUnexpectedEOF` cho một chunk không hoàn chỉnh. Trước đây nó trả về `io.EOF` trong một số trường hợp.

<!-- encoding/base32 -->

#### [encoding/csv](/pkg/encoding/csv/)

<!-- CL 99696 -->
`Reader` giờ từ chối các nỗ lực đặt trường [`Comma`](/pkg/encoding/csv/#Reader.Comma) thành ký tự double-quote, vì ký tự double-quote đã có ý nghĩa đặc biệt trong CSV.

<!-- encoding/csv -->

<!-- CL 100235 was reverted -->

#### [html/template](/pkg/html/template/)

<!-- CL 121815 -->
Gói đã thay đổi hành vi khi một giá trị interface có kiểu được truyền cho một hàm escaper ngầm. Trước đây giá trị như vậy được ghi ra là (một dạng được escape của) `<nil>`. Giờ các giá trị như vậy bị bỏ qua, giống như một giá trị `nil` không có kiểu là (và luôn đã) bị bỏ qua.

<!-- html/template -->

#### [image/gif](/pkg/image/gif/)

<!-- CL 93076 -->
GIF động không lặp giờ được hỗ trợ. Chúng được biểu thị bằng cách có [`LoopCount`](/pkg/image/gif/#GIF.LoopCount) là -1.

<!-- image/gif -->

#### [io/ioutil](/pkg/io/ioutil/)

<!-- CL 105675 -->
Hàm [`TempFile`](/pkg/io/ioutil/#TempFile) giờ hỗ trợ chỉ định vị trí đặt các ký tự ngẫu nhiên trong tên tệp. Nếu đối số `prefix` bao gồm một "`*`", chuỗi ngẫu nhiên thay thế "`*`". Ví dụ, đối số `prefix` là "`myname.*.bat`" sẽ tạo ra tên tệp ngẫu nhiên như "`myname.123456.bat`". Nếu không có "`*`" nào được bao gồm, hành vi cũ được giữ lại, và các chữ số ngẫu nhiên được thêm vào cuối.

<!-- io/ioutil -->

#### [math/big](/pkg/math/big/)

<!-- CL 108996 -->
[`ModInverse`](/pkg/math/big/#Int.ModInverse) giờ trả về nil khi g và n không nguyên tố cùng nhau. Kết quả trước đây không xác định.

<!-- math/big -->

#### [mime/multipart](/pkg/mime/multipart/)

<!-- CL 121055 -->
Việc xử lý form-data với tên tệp bị thiếu/rỗng đã được khôi phục về hành vi trong Go 1.9: trong [`Form`](/pkg/mime/multipart/#Form) cho phần form-data, giá trị có sẵn trong trường `Value` thay vì trường `File`. Trong các bản phát hành Go 1.10 đến 1.10.3, một phần form-data với tên tệp bị thiếu/rỗng và trường "Content-Type" không rỗng được lưu trữ trong trường `File`. Thay đổi này là lỗi trong 1.10 và đã được hoàn nguyên về hành vi 1.9.

<!-- mime/multipart -->

#### [mime/quotedprintable](/pkg/mime/quotedprintable/)

<!-- CL 121095 -->
Để hỗ trợ đầu vào không hợp lệ được tìm thấy trong thực tế, gói giờ cho phép các byte không phải ASCII nhưng không xác thực mã hóa của chúng.

<!-- mime/quotedprintable -->

#### [net](/pkg/net/)

<!-- CL 72810 -->
Kiểu [`ListenConfig`](/pkg/net/#ListenConfig) mới và trường [`Dialer.Control`](/pkg/net/#Dialer.Control) mới cho phép đặt các tùy chọn socket trước khi chấp nhận và tạo kết nối, tương ứng.

<!-- CL 76391 -->
Các phương thức `Read` và `Write` của [`syscall.RawConn`](/pkg/syscall/#RawConn) giờ hoạt động đúng trên Windows.

<!-- CL 107715 -->
Gói `net` giờ tự động sử dụng [system call `splice`](https://man7.org/linux/man-pages/man2/splice.2.html) trên Linux khi sao chép dữ liệu giữa các kết nối TCP trong [`TCPConn.ReadFrom`](/pkg/net/#TCPConn.ReadFrom), được gọi bởi [`io.Copy`](/pkg/io/#Copy). Kết quả là proxy TCP nhanh hơn, hiệu quả hơn.

<!-- CL 108297 -->
Các phương thức [`TCPConn.File`](/pkg/net/#TCPConn.File), [`UDPConn.File`](/pkg/net/#UDPConn.File), [`UnixConn.File`](/pkg/net/#UnixCOnn.File) và [`IPConn.File`](/pkg/net/#IPConn.File) không còn đặt `*os.File` được trả về vào chế độ blocking.

<!-- net -->

#### [net/http](/pkg/net/http/)

<!-- CL 71272 -->
Kiểu [`Transport`](/pkg/net/http/#Transport) có tùy chọn [`MaxConnsPerHost`](/pkg/net/http/#Transport.MaxConnsPerHost) mới cho phép giới hạn số kết nối tối đa mỗi host.

<!-- CL 79919 -->
Kiểu [`Cookie`](/pkg/net/http/#Cookie) có trường [`SameSite`](/pkg/net/http/#Cookie.SameSite) mới (của kiểu mới cũng được đặt tên [`SameSite`](/pkg/net/http/#SameSite)) để đại diện cho thuộc tính cookie mới được hầu hết các trình duyệt hỗ trợ gần đây. `Transport` của `net/http` không sử dụng thuộc tính `SameSite` cho chính nó, nhưng gói hỗ trợ phân tích và tuần tự hóa thuộc tính cho các trình duyệt sử dụng.

<!-- CL 81778 -->
Không còn được phép tái sử dụng một [`Server`](/pkg/net/http/#Server) sau khi gọi [`Shutdown`](/pkg/net/http/#Server.Shutdown) hoặc [`Close`](/pkg/net/http/#Server.Close). Điều này chưa bao giờ được hỗ trợ chính thức trong quá khứ và có hành vi thường gây ngạc nhiên. Giờ, tất cả các lời gọi tương lai đến các phương thức `Serve` của server sẽ trả về lỗi sau khi shutdown hoặc close.

<!-- CL 89275 was reverted before Go 1.11 -->

<!-- CL 93296 -->
Hằng số `StatusMisdirectedRequest` giờ được định nghĩa cho mã trạng thái HTTP 421.

<!-- CL 123875 -->
HTTP server sẽ không còn hủy context hoặc gửi trên các kênh [`CloseNotifier`](/pkg/net/http/#CloseNotifier) khi nhận các yêu cầu HTTP/1.1 được pipeline. Các trình duyệt không sử dụng HTTP pipelining, nhưng một số client (chẳng hạn như `apt` của Debian) có thể được cấu hình để làm vậy.

<!-- CL 115255 -->
[`ProxyFromEnvironment`](/pkg/net/http/#ProxyFromEnvironment), được sử dụng bởi [`DefaultTransport`](/pkg/net/http/#DefaultTransport), giờ hỗ trợ ký hiệu CIDR và các port trong biến môi trường `NO_PROXY`.

<!-- net/http -->

#### [net/http/httputil](/pkg/net/http/httputil/)

<!-- CL 77410 -->
[`ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy) có tùy chọn [`ErrorHandler`](/pkg/net/http/httputil/#ReverseProxy.ErrorHandler) mới để cho phép thay đổi cách xử lý lỗi.

<!-- CL 115135 -->
`ReverseProxy` giờ cũng truyền các header yêu cầu "`TE:`&nbsp;`trailers`" đến backend, theo yêu cầu của giao thức gRPC.

<!-- net/http/httputil -->

#### [os](/pkg/os/)

<!-- CL 78835 -->
Hàm [`UserCacheDir`](/pkg/os/#UserCacheDir) mới trả về thư mục gốc mặc định để sử dụng cho dữ liệu đã lưu trong bộ nhớ đệm dành riêng cho người dùng.

<!-- CL 94856 -->
[`ModeIrregular`](/pkg/os/#ModeIrregular) mới là một bit [`FileMode`](/pkg/os/#FileMode) để biểu thị rằng một tệp không phải là tệp thông thường, nhưng không có gì khác được biết về nó, hoặc nó không phải là socket, thiết bị, named pipe, symlink hoặc các kiểu tệp khác mà Go có bit mode được định nghĩa.

<!-- CL 99337 -->
[`Symlink`](/pkg/os/#Symlink) giờ hoạt động đối với người dùng không có đặc quyền trên Windows 10 trên các máy có Developer Mode được bật.

<!-- CL 100077 -->
Khi một descriptor không chặn được truyền cho [`NewFile`](/pkg/os#NewFile), `*File` kết quả sẽ được giữ ở chế độ không chặn. Điều này có nghĩa là I/O cho `*File` đó sẽ sử dụng runtime poller thay vì một thread riêng, và các phương thức [`SetDeadline`](/pkg/os/#File.SetDeadline) sẽ hoạt động.

<!-- os -->

#### [os/signal](/pkg/os/signal/)

<!-- CL 108376 -->
Hàm [`Ignored`](/pkg/os/signal/#Ignored) mới báo cáo xem một tín hiệu hiện đang bị bỏ qua hay không.

<!-- os/signal -->

#### [os/user](/pkg/os/user/)

<!-- CL 92456 -->
Gói `os/user` giờ có thể được build ở chế độ Go thuần túy bằng cách sử dụng build tag "`osusergo`", độc lập với việc sử dụng biến môi trường `CGO_ENABLED=0`. Trước đây, cách duy nhất để sử dụng triển khai Go thuần túy của gói là tắt hỗ trợ `cgo` trên toàn bộ chương trình.

<!-- os/user -->

<!-- CL 101715 was reverted -->

#### [runtime](/pkg/runtime/)

<!-- CL 70993 -->
Đặt biến môi trường <code>GODEBUG=tracebackancestors=_N_</code> giờ mở rộng các traceback với các stack mà tại đó các goroutine được tạo, trong đó _N_ giới hạn số lượng goroutine tổ tiên cần báo cáo.

<!-- runtime -->

#### [runtime/pprof](/pkg/runtime/pprof/)

<!-- CL 102696 -->
Bản phát hành này thêm kiểu profile "allocs" mới profile tổng số byte được phân bổ kể từ khi chương trình bắt đầu (bao gồm các byte đã được thu gom rác). Điều này giống hệt với profile "heap" hiện có được xem ở chế độ `-alloc_space`. Giờ `go test -memprofile=...` báo cáo profile "allocs" thay vì profile "heap".

<!-- runtime/pprof -->

#### [sync](/pkg/sync/)

<!-- CL 87095 -->
Profile mutex giờ bao gồm contention reader/writer cho [`RWMutex`](/pkg/sync/#RWMutex). Contention writer/writer đã được bao gồm trong profile mutex.

<!-- sync -->

#### [syscall](/pkg/syscall/)

<!-- CL 106275 -->
Trên Windows, một số trường đã được thay đổi từ `uintptr` sang kiểu [`Pointer`](/pkg/syscall/?GOOS=windows&GOARCH=amd64#Pointer) mới để tránh các vấn đề với bộ gom rác Go. Thay đổi tương tự đã được thực hiện với gói [`golang.org/x/sys/windows`](https://godoc.org/golang.org/x/sys/windows). Đối với bất kỳ mã nào bị ảnh hưởng, người dùng trước tiên nên di chuyển khỏi gói `syscall` sang gói `golang.org/x/sys/windows`, và sau đó thay đổi để sử dụng `Pointer`, trong khi tuân thủ các [quy tắc chuyển đổi `unsafe.Pointer`](/pkg/unsafe/#Pointer).

<!-- CL 118658 -->
Trên Linux, tham số `flags` cho [`Faccessat`](/pkg/syscall/?GOOS=linux&GOARCH=amd64#Faccessat) giờ được triển khai giống như trong glibc. Trong các bản phát hành Go trước, tham số flags bị bỏ qua.

<!-- CL 118658 -->
Trên Linux, tham số `flags` cho [`Fchmodat`](/pkg/syscall/?GOOS=linux&GOARCH=amd64#Fchmodat) giờ được xác thực. `fchmodat` của Linux không hỗ trợ tham số `flags` nên giờ chúng tôi mô phỏng hành vi của glibc và trả về lỗi nếu nó khác không.

<!-- syscall -->

#### [text/scanner](/pkg/text/scanner/)

<!-- CL 112037 -->
Phương thức [`Scanner.Scan`](/pkg/text/scanner/#Scanner.Scan) giờ trả về token [`RawString`](/pkg/text/scanner/#RawString) thay vì [`String`](/pkg/text/scanner/#String) cho các raw string literal.

<!-- text/scanner -->

#### [text/template](/pkg/text/template/)

<!-- CL 84480 -->
Sửa đổi các biến template qua các phép gán giờ được phép thông qua token `=`:

	  {{ $v := "init" }}
	  {{ if true }}
	    {{ $v = "changed" }}
	  {{ end }}
	  v: {{ $v }} {{/* "changed" */}}

<!-- CL 95215 -->
Trong các phiên bản trước, các giá trị `nil` không có kiểu được truyền cho các hàm template bị bỏ qua. Giờ chúng được truyền như các đối số bình thường.

<!-- text/template -->

#### [time](/pkg/time/)

<!-- CL 98157 -->
Phân tích các múi giờ được biểu thị bằng dấu và offset giờ được hỗ trợ. Trong các phiên bản trước, tên múi giờ số (chẳng hạn như `+03`) không được coi là hợp lệ, và chỉ các chữ viết tắt ba chữ cái (chẳng hạn như `MST`) được chấp nhận khi mong đợi tên múi giờ.

<!-- time -->
