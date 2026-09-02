---
title: Ghi chú phát hành Go 1.16
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

## Giới thiệu về Go 1.16 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.16, ra đời sáu tháng sau [Go 1.15](/doc/go1.15).
Hầu hết các thay đổi nằm ở phần triển khai của bộ công cụ, runtime và thư viện.
Như thường lệ, bản phát hành vẫn duy trì [cam kết tương thích](/doc/go1compat.html) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

## Thay đổi ngôn ngữ {#language}

Không có thay đổi về ngôn ngữ.

## Các nền tảng {#ports}

### Darwin và iOS {#darwin}

<!-- golang.org/issue/38485, golang.org/issue/41385, CL 266373, more CLs -->
Go 1.16 thêm hỗ trợ kiến trúc ARM 64-bit trên macOS (còn gọi là Apple Silicon) với `GOOS=darwin`, `GOARCH=arm64`. Giống như cổng `darwin/amd64`, cổng `darwin/arm64` hỗ trợ cgo, liên kết nội bộ và ngoài, các chế độ build `c-archive`, `c-shared` và `pie`, cũng như race detector.

<!-- CL 254740 -->
Cổng iOS, trước đây là `darwin/arm64`, đã được đổi tên thành `ios/arm64`. `GOOS=ios` bao hàm build tag `darwin`, giống như `GOOS=android` bao hàm build tag `linux`. Thay đổi này sẽ trong suốt với bất kỳ ai dùng gomobile để build ứng dụng iOS.

Việc giới thiệu `GOOS=ios` có nghĩa là các tên tệp như `x_ios.go` giờ sẽ chỉ được build cho `GOOS=ios`; xem [`go` `help` `buildconstraint`](/cmd/go/#hdr-Build_constraints) để biết chi tiết. Các gói hiện tại dùng tên tệp dạng này sẽ phải đổi tên tệp.

<!-- golang.org/issue/42100, CL 263798 -->
Go 1.16 thêm cổng `ios/amd64`, nhắm mục tiêu đến iOS simulator chạy trên macOS dựa trên AMD64. Trước đây điều này được hỗ trợ không chính thức thông qua `darwin/amd64` với build tag `ios`. Xem thêm [`misc/ios/README`](/misc/ios/README) để biết chi tiết về cách build chương trình cho iOS và iOS simulator.

<!-- golang.org/issue/23011 -->
Go 1.16 là bản phát hành cuối cùng chạy trên macOS 10.12 Sierra. Go 1.17 sẽ yêu cầu macOS 10.13 High Sierra hoặc mới hơn.

### NetBSD {#netbsd}

<!-- golang.org/issue/30824 -->
Go giờ hỗ trợ kiến trúc ARM 64-bit trên NetBSD (cổng `netbsd/arm64`).

### OpenBSD {#openbsd}

<!-- golang.org/issue/40995 -->
Go giờ hỗ trợ kiến trúc MIPS64 trên OpenBSD (cổng `openbsd/mips64`). Cổng này chưa hỗ trợ cgo.

<!-- golang.org/issue/36435, many CLs -->
Trên kiến trúc x86 64-bit và ARM 64-bit trên OpenBSD (các cổng `openbsd/amd64` và `openbsd/arm64`), các lời gọi hệ thống giờ được thực hiện qua `libc`, thay vì trực tiếp dùng lệnh `SYSCALL`/`SVC`. Điều này đảm bảo tương thích tiến với các phiên bản OpenBSD tương lai. Cụ thể, OpenBSD 6.9 trở đi sẽ yêu cầu các lời gọi hệ thống được thực hiện qua `libc` cho các tệp nhị phân Go không tĩnh.

### 386 {#386}

<!-- golang.org/issue/40255, golang.org/issue/41848, CL 258957, and CL 260017 -->
Như đã [thông báo](go1.15#386) trong ghi chú phát hành Go 1.15, Go 1.16 bỏ hỗ trợ biên dịch chế độ x87 (`GO386=387`). Hỗ trợ cho bộ xử lý không có SSE2 giờ có sẵn bằng chế độ soft float (`GO386=softfloat`). Người dùng chạy trên bộ xử lý không có SSE2 nên thay `GO386=387` bằng `GO386=softfloat`.

### RISC-V {#riscv}

<!-- golang.org/issue/36641, CL 267317 -->
Cổng `linux/riscv64` giờ hỗ trợ cgo và `-buildmode=pie`. Bản phát hành này cũng bao gồm các tối ưu hóa hiệu năng và cải tiến tạo mã cho RISC-V.

## Công cụ {#tools}

### Lệnh Go {#go-command}

#### Modules {#modules}

<!-- golang.org/issue/41330 -->
Chế độ module-aware được bật mặc định, bất kể có tệp `go.mod` trong thư mục làm việc hiện tại hay thư mục cha không. Chính xác hơn, biến môi trường `GO111MODULE` giờ mặc định là `on`. Để chuyển về hành vi trước, đặt `GO111MODULE` thành `auto`.

<!-- golang.org/issue/40728 -->
Các lệnh build như `go` `build` và `go` `test` không còn sửa đổi `go.mod` và `go.sum` theo mặc định. Thay vào đó, chúng báo lỗi nếu dependency module hoặc checksum cần được thêm hoặc cập nhật (như thể cờ `-mod=readonly` được dùng). Yêu cầu module và checksum có thể được điều chỉnh bằng `go` `mod` `tidy` hoặc `go` `get`.

<!-- golang.org/issue/40276 -->
`go` `install` giờ chấp nhận các tham số có hậu tố phiên bản (ví dụ `go` `install` `example.com/cmd@v1.0.0`). Điều này khiến `go` `install` build và cài đặt các gói trong chế độ module-aware, bỏ qua tệp `go.mod` trong thư mục hiện tại hoặc thư mục cha, nếu có. Điều này hữu ích để cài đặt file thực thi mà không ảnh hưởng đến dependency của module chính.

<!-- golang.org/issue/40276 -->
`go` `install`, có hoặc không có hậu tố phiên bản (như mô tả ở trên), giờ là cách được khuyến nghị để build và cài đặt các gói trong chế độ module. `go` `get` nên được dùng với cờ `-d` để điều chỉnh dependency của module hiện tại mà không build gói, và việc dùng `go` `get` để build và cài đặt gói đã bị deprecated. Trong bản phát hành tương lai, cờ `-d` sẽ luôn được bật.

<!-- golang.org/issue/24031 -->
Directive `retract` giờ có thể dùng trong tệp `go.mod` để chỉ ra rằng một số phiên bản đã công bố của module không nên được dùng bởi các module khác. Tác giả module có thể retract một phiên bản sau khi phát hiện vấn đề nghiêm trọng hoặc nếu phiên bản được công bố ngoài ý muốn.

<!-- golang.org/issue/26603 -->
Các lệnh con `go` `mod` `vendor` và `go` `mod` `tidy` giờ chấp nhận cờ `-e`, chỉ thị chúng tiếp tục dù có lỗi trong việc phân giải các gói bị thiếu.

<!-- golang.org/issue/36465 -->
Lệnh `go` giờ bỏ qua các yêu cầu về phiên bản module bị loại trừ bởi directive `exclude` trong module chính. Trước đây, lệnh `go` dùng phiên bản cao hơn tiếp theo so với phiên bản bị loại trừ, nhưng phiên bản đó có thể thay đổi theo thời gian, dẫn đến các build không tái lập được.

<!-- golang.org/issue/43052, golang.org/issue/43985 -->
Trong chế độ module, lệnh `go` giờ không cho phép các đường dẫn import bao gồm ký tự không ASCII hoặc các phần tử đường dẫn có ký tự dấu chấm dẫn đầu (`.`). Đường dẫn module với các ký tự này đã bị không cho phép (xem [Đường dẫn và phiên bản module](/ref/mod#go-mod-file-ident)), vì vậy thay đổi này chỉ ảnh hưởng đến các đường dẫn trong thư mục con của module.

#### Nhúng tệp {#embed}

Lệnh `go` giờ hỗ trợ bao gồm các tệp tĩnh và cây tệp như một phần của file thực thi cuối cùng, bằng cách dùng directive `//go:embed` mới. Xem tài liệu của gói [`embed`](/pkg/embed/) mới để biết chi tiết.

#### `go` `test` {#go-test}

<!-- golang.org/issue/29062 -->
Khi dùng `go` `test`, test gọi `os.Exit(0)` trong quá trình thực thi hàm test giờ sẽ được coi là thất bại. Điều này giúp bắt các trường hợp test gọi mã gọi `os.Exit(0)` và do đó dừng chạy tất cả test tiếp theo. Nếu hàm `TestMain` gọi `os.Exit(0)` thì vẫn được coi là test pass.

<!-- golang.org/issue/39484 -->
`go` `test` báo lỗi khi cờ `-c` hoặc `-i` được dùng cùng với các cờ không xác định. Thông thường, các cờ không xác định được truyền cho test, nhưng khi `-c` hoặc `-i` được dùng, test không được chạy.

#### `go` `get` {#go-get}

<!-- golang.org/issue/37519 -->
Cờ `go` `get` `-insecure` bị deprecated và sẽ bị xóa trong phiên bản tương lai. Cờ này cho phép lấy từ kho lưu trữ và phân giải domain tùy chỉnh bằng các scheme không an toàn như HTTP, và cũng bỏ qua xác thực checksum module bằng cơ sở dữ liệu checksum. Để cho phép dùng các scheme không an toàn, hãy dùng biến môi trường `GOINSECURE` thay thế. Để bỏ qua xác thực checksum module, dùng `GOPRIVATE` hoặc `GONOSUMDB`. Xem `go` `help` `environment` để biết chi tiết.

<!-- golang.org/cl/263267 -->
`go` `get` `example.com/mod@patch` giờ yêu cầu một phiên bản nào đó của `example.com/mod` đã được yêu cầu bởi module chính. (Tuy nhiên, `go` `get` `-u=patch` vẫn tiếp tục patch cả các dependency mới thêm vào.)

#### Biến môi trường `GOVCS` {#govcs}

<!-- golang.org/issue/266420 -->
`GOVCS` là biến môi trường mới giới hạn các công cụ kiểm soát phiên bản mà lệnh `go` có thể dùng để tải xuống mã nguồn. Điều này giảm thiểu các vấn đề bảo mật với các công cụ thường được dùng trong môi trường đáng tin cậy, đã xác thực. Theo mặc định, `git` và `hg` có thể được dùng để tải mã từ bất kỳ kho lưu trữ nào. `svn`, `bzr` và `fossil` chỉ có thể dùng để tải mã từ các kho lưu trữ với đường dẫn module hoặc gói khớp với các mẫu trong biến môi trường `GOPRIVATE`. Xem [`go` `help` `vcs`](/cmd/go/#hdr-Controlling_version_control_with_GOVCS) để biết chi tiết.

#### Mẫu `all` {#all-pattern}

<!-- golang.org/cl/240623 -->
Khi tệp `go.mod` của module chính khai báo `go` `1.16` hoặc cao hơn, mẫu gói `all` giờ chỉ khớp với các gói được nhập bắc cầu bởi gói hoặc test tìm thấy trong module chính. (Các gói được nhập bởi _test của_ các gói được nhập bởi module chính không còn được bao gồm.) Đây là cùng tập hợp gói được giữ lại bởi `go` `mod` `vendor` từ Go 1.11.

#### Cờ build `-toolexec` {#toolexec}

<!-- golang.org/cl/263357 -->
Khi cờ build `-toolexec` được chỉ định để dùng chương trình khi gọi các chương trình toolchain như compile hoặc asm, biến môi trường `TOOLEXEC_IMPORTPATH` giờ được đặt thành import path của gói đang được build.

#### Cờ build `-i` {#i-flag}

<!-- golang.org/issue/41696 -->
Cờ `-i` được chấp nhận bởi `go` `build`, `go` `install` và `go` `test` giờ bị deprecated. Cờ `-i` chỉ thị lệnh `go` cài đặt các gói được nhập bởi các gói được đặt tên trên dòng lệnh. Kể từ khi build cache được giới thiệu trong Go 1.10, cờ `-i` không còn ảnh hưởng đáng kể đến thời gian build, và nó gây ra lỗi khi thư mục cài đặt không thể ghi.

#### Lệnh `list` {#list-buildid}

<!-- golang.org/cl/263542 -->
Khi cờ `-export` được chỉ định, trường `BuildID` giờ được đặt thành build ID của gói được biên dịch. Điều này tương đương với việc chạy `go` `tool` `buildid` trên `go` `list` `-exported` `-f` `{{.Export}}`, nhưng không cần bước thêm.

#### Cờ `-overlay` {#overlay-flag}

<!-- golang.org/issue/39958 -->
Cờ `-overlay` chỉ định tệp cấu hình JSON chứa tập hợp thay thế đường dẫn tệp. Cờ `-overlay` có thể dùng với tất cả các lệnh build và các lệnh con `go` `mod`. Nó chủ yếu dành cho các công cụ editor như gopls để hiểu tác động của các thay đổi chưa lưu trong tệp nguồn. Tệp cấu hình ánh xạ các đường dẫn tệp thực tế sang đường dẫn tệp thay thế và lệnh `go` cùng các build của nó sẽ chạy như thể các đường dẫn tệp thực tế tồn tại với nội dung được cung cấp bởi các đường dẫn tệp thay thế, hoặc không tồn tại nếu các đường dẫn tệp thay thế rỗng.

### Cgo {#cgo}

<!-- CL 252378 -->
Công cụ [cgo](/cmd/cgo) sẽ không còn cố gắng dịch các bitfield struct C thành các trường struct Go, ngay cả khi kích thước của chúng có thể được biểu diễn trong Go. Thứ tự xuất hiện của các bitfield C trong bộ nhớ phụ thuộc vào implementation, vì vậy trong một số trường hợp công cụ cgo tạo ra kết quả không đúng một cách âm thầm.

### Vet {#vet}

#### Cảnh báo mới cho việc dùng testing.T không hợp lệ trong goroutine {#vet-testing-T}

<!-- CL 235677 -->
Công cụ vet giờ cảnh báo về các lời gọi không hợp lệ đến phương thức `Fatal` của `testing.T` từ trong goroutine được tạo trong test. Điều này cũng cảnh báo về các lời gọi đến các phương thức `Fatalf`, `FailNow` và `Skip{,f,Now}` trên các test `testing.T` hoặc benchmark `testing.B`.

Các lời gọi đến các phương thức này dừng thực thi goroutine được tạo, không phải hàm `Test*` hoặc `Benchmark*`. Vì vậy chúng [bắt buộc](/pkg/testing/#T.FailNow) phải được gọi bởi goroutine chạy hàm test hoặc benchmark. Ví dụ:

	func TestFoo(t *testing.T) {
	    go func() {
	        if condition() {
	            t.Fatal("oops") // Điều này thoát hàm nội bộ thay vì TestFoo.
	        }
	        ...
	    }()
	}

Mã gọi `t.Fatal` (hoặc phương thức tương tự) từ goroutine được tạo nên được viết lại để báo hiệu lỗi test bằng `t.Error` và thoát goroutine sớm bằng phương thức thay thế, như dùng lệnh `return`. Ví dụ trước có thể được viết lại như sau:

	func TestFoo(t *testing.T) {
	    go func() {
	        if condition() {
	            t.Error("oops")
	            return
	        }
	        ...
	    }()
	}

#### Cảnh báo mới cho frame pointer {#vet-frame-pointer}

<!-- CL 248686, CL 276372 -->
Công cụ vet giờ cảnh báo về assembly amd64 ghi đè thanh ghi BP (frame pointer) mà không lưu và khôi phục, trái với quy ước gọi hàm. Mã không giữ nguyên thanh ghi BP phải được sửa đổi để không dùng BP hoặc giữ nguyên BP bằng cách lưu và khôi phục. Cách dễ nhất để giữ nguyên BP là đặt kích thước frame thành giá trị khác không, khiến prologue và epilogue được tạo ra tự động giữ nguyên thanh ghi BP cho bạn. Xem [CL 248260](/cl/248260) để biết ví dụ sửa lỗi.

#### Cảnh báo mới cho asn1.Unmarshal {#vet-asn1-unmarshal}

<!-- CL 243397 -->
Công cụ vet giờ cảnh báo về việc truyền không đúng tham số không phải con trỏ hoặc nil cho [`asn1.Unmarshal`](/pkg/encoding/asn1/#Unmarshal). Điều này giống như các kiểm tra hiện có cho [`encoding/json.Unmarshal`](/pkg/encoding/json/#Unmarshal) và [`encoding/xml.Unmarshal`](/pkg/encoding/xml/#Unmarshal).

## Runtime {#runtime}

Gói mới [`runtime/metrics`](/pkg/runtime/metrics/) giới thiệu interface ổn định để đọc các metric do implementation định nghĩa từ Go runtime. Nó thay thế các hàm hiện có như [`runtime.ReadMemStats`](/pkg/runtime/#ReadMemStats) và [`debug.GCStats`](/pkg/runtime/debug/#GCStats) và tổng quát và hiệu quả hơn đáng kể. Xem tài liệu gói để biết thêm chi tiết.

<!-- CL 254659 -->
Đặt biến môi trường `GODEBUG` thành `inittrace=1` giờ khiến runtime phát ra một dòng vào stderr cho mỗi `init` gói, tóm tắt thời gian thực thi và cấp phát bộ nhớ. Trace này có thể dùng để tìm bottleneck hoặc hồi quy trong hiệu năng khởi động Go. [Tài liệu `GODEBUG`](/pkg/runtime/#hdr-Environment_Variables) mô tả định dạng.

<!-- CL 267100 -->
Trên Linux, runtime giờ mặc định trả bộ nhớ về hệ điều hành ngay lập tức (dùng `MADV_DONTNEED`), thay vì lười biếng khi hệ điều hành chịu áp lực bộ nhớ (dùng `MADV_FREE`). Điều này có nghĩa là các số liệu thống kê bộ nhớ cấp tiến trình như RSS sẽ phản ánh chính xác hơn lượng bộ nhớ vật lý được dùng bởi các tiến trình Go. Các hệ thống hiện đang dùng `GODEBUG=madvdontneed=1` để cải thiện hành vi giám sát bộ nhớ không còn cần đặt biến môi trường này.

<!-- CL 220419, CL 271987 -->
Go 1.16 sửa sự không nhất quán giữa race detector và [mô hình bộ nhớ Go](/ref/mem). Race detector giờ theo dõi chính xác hơn các quy tắc đồng bộ hóa kênh của mô hình bộ nhớ. Kết quả là, detector có thể báo cáo các race mà trước đây bỏ lỡ.

## Trình biên dịch {#compiler}

<!-- CL 256459, CL 264837, CL 266203, CL 256460 -->
Trình biên dịch giờ có thể inline các hàm với vòng lặp `for` không có nhãn, giá trị phương thức và type switch. Inliner cũng có thể phát hiện nhiều lời gọi gián tiếp hơn nơi inlining có thể thực hiện.

## Linker {#linker}

<!-- CL 248197 -->
Bản phát hành này bao gồm các cải tiến bổ sung cho linker Go, giảm mức sử dụng tài nguyên linker (cả thời gian và bộ nhớ) và cải thiện tính mạnh mẽ/khả năng bảo trì của mã. Những thay đổi này tạo thành nửa thứ hai của dự án hai bản phát hành để [hiện đại hóa linker Go](/s/better-linker).

Các thay đổi linker trong 1.16 mở rộng các cải tiến 1.15 sang tất cả các kết hợp kiến trúc/hệ điều hành được hỗ trợ (các cải tiến hiệu năng 1.15 chủ yếu tập trung vào các hệ điều hành dựa trên `ELF` và kiến trúc `amd64`). Với tập hợp đại diện các chương trình Go lớn, liên kết nhanh hơn 20-25% so với 1.15 và yêu cầu ít hơn 5-15% bộ nhớ trung bình cho `linux/amd64`, với cải tiến lớn hơn cho các kiến trúc và hệ điều hành khác. Hầu hết tệp nhị phân cũng nhỏ hơn nhờ cắt bỏ ký hiệu tích cực hơn.

<!-- CL 255259 -->
Trên Windows, `go build -buildmode=c-shared` giờ tạo Windows ASLR DLL theo mặc định. ASLR có thể tắt bằng `--ldflags=-aslr=false`.

## Thư viện chuẩn {#library}

### Tệp nhúng {#library-embed}

Gói mới [`embed`](/pkg/embed/) cung cấp quyền truy cập vào các tệp được nhúng vào chương trình trong quá trình biên dịch bằng directive [`//go:embed` mới](#embed).

### Hệ thống tệp {#fs}

Gói mới [`io/fs`](/pkg/io/fs/) định nghĩa interface [`fs.FS`](/pkg/io/fs/#FS), một abstraction cho cây tệp chỉ đọc. Các gói thư viện chuẩn đã được điều chỉnh để sử dụng interface khi phù hợp.

Về phía producer của interface, kiểu mới [`embed.FS`](/pkg/embed/#FS) triển khai `fs.FS`, cũng như [`zip.Reader`](/pkg/archive/zip/#Reader). Hàm mới [`os.DirFS`](/pkg/os/#DirFS) cung cấp triển khai `fs.FS` được hỗ trợ bởi cây tệp hệ điều hành.

Về phía consumer, hàm mới [`http.FS`](/pkg/net/http/#FS) chuyển đổi `fs.FS` thành [`http.FileSystem`](/pkg/net/http/#FileSystem). Ngoài ra, các hàm và phương thức [`ParseFS`](/pkg/html/template/#ParseFS) của các gói [`html/template`](/pkg/html/template/) và [`text/template`](/pkg/text/template/) đọc các template từ `fs.FS`.

Để kiểm thử mã triển khai `fs.FS`, gói mới [`testing/fstest`](/pkg/testing/fstest/) cung cấp hàm [`TestFS`](/pkg/testing/fstest/#TestFS) kiểm tra và báo cáo các lỗi phổ biến. Nó cũng cung cấp triển khai hệ thống tệp trong bộ nhớ đơn giản, [`MapFS`](/pkg/testing/fstest/#MapFS), hữu ích để kiểm thử mã chấp nhận các triển khai `fs.FS`.

### Deprecated io/ioutil {#ioutil}

Gói [`io/ioutil`](/pkg/io/ioutil/) hóa ra là tập hợp các thứ được định nghĩa kém và khó hiểu. Tất cả chức năng do gói cung cấp đã được chuyển sang các gói khác. Gói `io/ioutil` vẫn còn và sẽ tiếp tục hoạt động như trước, nhưng chúng tôi khuyến khích mã mới dùng các định nghĩa mới trong các gói [`io`](/pkg/io/) và [`os`](/pkg/os/). Dưới đây là danh sách vị trí mới của các tên được xuất bởi `io/ioutil`:

  - [`Discard`](/pkg/io/ioutil/#Discard)
    => [`io.Discard`](/pkg/io/#Discard)
  - [`NopCloser`](/pkg/io/ioutil/#NopCloser)
    => [`io.NopCloser`](/pkg/io/#NopCloser)
  - [`ReadAll`](/pkg/io/ioutil/#ReadAll)
    => [`io.ReadAll`](/pkg/io/#ReadAll)
  - [`ReadDir`](/pkg/io/ioutil/#ReadDir)
    => [`os.ReadDir`](/pkg/os/#ReadDir)
    (lưu ý: trả về slice của [`os.DirEntry`](/pkg/os/#DirEntry) thay vì slice của [`fs.FileInfo`](/pkg/io/fs/#FileInfo))
  - [`ReadFile`](/pkg/io/ioutil/#ReadFile)
    => [`os.ReadFile`](/pkg/os/#ReadFile)
  - [`TempDir`](/pkg/io/ioutil/#TempDir)
    => [`os.MkdirTemp`](/pkg/os/#MkdirTemp)
  - [`TempFile`](/pkg/io/ioutil/#TempFile)
    => [`os.CreateTemp`](/pkg/os/#CreateTemp)
  - [`WriteFile`](/pkg/io/ioutil/#WriteFile)
    => [`os.WriteFile`](/pkg/os/#WriteFile)


### Thay đổi nhỏ trong thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ trong thư viện, được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1 trong tâm trí.

#### [archive/zip](/pkg/archive/zip/)

<!-- CL 243937 -->
Phương thức mới [`Reader.Open`](/pkg/archive/zip/#Reader.Open) triển khai interface [`fs.FS`](/pkg/io/fs/#FS).

#### [crypto/dsa](/pkg/crypto/dsa/)

<!-- CL 257939 -->
Gói [`crypto/dsa`](/pkg/crypto/dsa/) giờ bị deprecated. Xem [issue #40337](/issue/40337).

<!-- crypto/dsa -->

#### [crypto/hmac](/pkg/crypto/hmac/)

<!-- CL 261960 -->
[`New`](/pkg/crypto/hmac/#New) giờ sẽ panic nếu các lời gọi riêng biệt đến hàm tạo hash không trả về giá trị mới. Trước đây, hành vi là không xác định và đôi khi tạo ra đầu ra không hợp lệ.

<!-- crypto/hmac -->

#### [crypto/tls](/pkg/crypto/tls/)

<!-- CL 256897 -->
Các thao tác I/O trên kết nối TLS đang đóng hoặc đã đóng giờ có thể được phát hiện bằng lỗi mới [`net.ErrClosed`](/pkg/net/#ErrClosed). Cách dùng điển hình sẽ là `errors.Is(err, net.ErrClosed)`.

<!-- CL 266037 -->
Deadline ghi mặc định giờ được đặt trong [`Conn.Close`](/pkg/crypto/tls/#Conn.Close) trước khi gửi cảnh báo "close notify", để ngăn chặn chặn vô hạn.

<!-- CL 239748 -->
Client giờ trả về lỗi handshake nếu server chọn [giao thức ALPN](/pkg/crypto/tls/#ConnectionState.NegotiatedProtocol) không nằm trong [danh sách được client quảng bá](/pkg/crypto/tls/#Config.NextProtos).

<!-- CL 262857 -->
Server giờ sẽ ưu tiên các cipher suite AEAD khác có sẵn (như ChaCha20Poly1305) hơn các cipher suite AES-GCM nếu client hoặc server không có hỗ trợ phần cứng AES, trừ khi cả [`Config.PreferServerCipherSuites`](/pkg/crypto/tls/#Config.PreferServerCipherSuites) và [`Config.CipherSuites`](/pkg/crypto/tls/#Config.CipherSuites) đều được đặt. Client được giả định không có hỗ trợ phần cứng AES nếu nó không báo hiệu ưu tiên cho các cipher suite AES-GCM.

<!-- CL 246637 -->
[`Config.Clone`](/pkg/crypto/tls/#Config.Clone) giờ trả về nil nếu receiver là nil, thay vì panic.

<!-- crypto/tls -->

#### [crypto/x509](/pkg/crypto/x509/)

Cờ `GODEBUG=x509ignoreCN=0` sẽ bị xóa trong Go 1.17. Nó bật hành vi kế thừa của việc xử lý trường `CommonName` trên chứng chỉ X.509 như tên host khi không có Subject Alternative Names.

<!-- CL 235078 -->
[`ParseCertificate`](/pkg/crypto/x509/#ParseCertificate) và [`CreateCertificate`](/pkg/crypto/x509/#CreateCertificate) giờ thực thi các hạn chế mã hóa chuỗi cho các trường `DNSNames`, `EmailAddresses` và `URIs`. Các trường này chỉ có thể chứa các chuỗi với ký tự trong phạm vi ASCII.

<!-- CL 259697 -->
[`CreateCertificate`](/pkg/crypto/x509/#CreateCertificate) giờ xác minh chữ ký của chứng chỉ được tạo bằng khóa công khai của người ký. Nếu chữ ký không hợp lệ, một lỗi được trả về, thay vì chứng chỉ bị hỏng.

<!-- CL 257939 -->
Xác minh chữ ký DSA không còn được hỗ trợ. Lưu ý rằng tạo chữ ký DSA chưa bao giờ được hỗ trợ. Xem [issue #40337](/issue/40337).

<!-- CL 257257 -->
Trên Windows, [`Certificate.Verify`](/pkg/crypto/x509/#Certificate.Verify) giờ sẽ trả về tất cả các chuỗi chứng chỉ được xây dựng bởi bộ xác minh chứng chỉ của nền tảng, thay vì chỉ chuỗi được xếp hạng cao nhất.

<!-- CL 262343 -->
Phương thức mới [`SystemRootsError.Unwrap`](/pkg/crypto/x509/#SystemRootsError.Unwrap) cho phép truy cập trường [`Err`](/pkg/crypto/x509/#SystemRootsError.Err) qua các hàm gói [`errors`](/pkg/errors).

<!-- CL 230025 -->
Trên hệ thống Unix, gói `crypto/x509` giờ hiệu quả hơn trong cách lưu trữ bản sao system cert pool của nó. Các chương trình chỉ dùng một số ít roots sẽ dùng ít hơn khoảng nửa megabyte bộ nhớ.

<!-- crypto/x509 -->

#### [debug/elf](/pkg/debug/elf/)

<!-- CL 255138 -->
Nhiều hằng số [`DT`](/pkg/debug/elf/#DT_NULL) và [`PT`](/pkg/debug/elf/#PT_NULL) hơn đã được thêm vào.

<!-- debug/elf -->

#### [encoding/asn1](/pkg/encoding/asn1)

<!-- CL 255881 -->
[`Unmarshal`](/pkg/encoding/asn1/#Unmarshal) và [`UnmarshalWithParams`](/pkg/encoding/asn1/#UnmarshalWithParams) giờ trả về lỗi thay vì panic khi tham số không phải con trỏ hoặc là nil. Thay đổi này khớp với hành vi của các gói encoding khác như [`encoding/json`](/pkg/encoding/json).

#### [encoding/json](/pkg/encoding/json/)

<!-- CL 234818 -->
Các tag field struct `json` được hiểu bởi [`Marshal`](/pkg/encoding/json/#Marshal), [`Unmarshal`](/pkg/encoding/json/#Unmarshal) và chức năng liên quan giờ cho phép ký tự dấu chấm phẩy trong tên đối tượng JSON cho field struct Go.

<!-- encoding/json -->

#### [encoding/xml](/pkg/encoding/xml/)

<!-- CL 264024 -->
Encoder luôn chú ý tránh dùng tiền tố namespace bắt đầu bằng `xml`, vốn bị dành riêng bởi đặc tả XML. Giờ, theo đặc tả chặt hơn, kiểm tra đó không phân biệt chữ hoa chữ thường, vì vậy các tiền tố bắt đầu bằng `XML`, `XmL` v.v. cũng được tránh.

<!-- encoding/xml -->

#### [flag](/pkg/flag/)

<!-- CL 240014 -->
Hàm mới [`Func`](/pkg/flag/#Func) cho phép đăng ký cờ được triển khai bằng cách gọi hàm, như giải pháp nhẹ hơn so với triển khai interface [`Value`](/pkg/flag/#Value).

<!-- flag -->

#### [go/build](/pkg/go/build/)

<!-- CL 243941, CL 283636 -->
Struct [`Package`](/pkg/go/build/#Package) có các trường mới báo cáo thông tin về các directive `//go:embed` trong gói: [`EmbedPatterns`](/pkg/go/build/#Package.EmbedPatterns), [`EmbedPatternPos`](/pkg/go/build/#Package.EmbedPatternPos), [`TestEmbedPatterns`](/pkg/go/build/#Package.TestEmbedPatterns), [`TestEmbedPatternPos`](/pkg/go/build/#Package.TestEmbedPatternPos), [`XTestEmbedPatterns`](/pkg/go/build/#Package.XTestEmbedPatterns), [`XTestEmbedPatternPos`](/pkg/go/build/#Package.XTestEmbedPatternPos).

<!-- CL 240551 -->
Trường [`IgnoredGoFiles`](/pkg/go/build/#Package.IgnoredGoFiles) của [`Package`](/pkg/go/build/#Package) sẽ không còn bao gồm các tệp bắt đầu bằng "\_" hoặc ".", vì các tệp đó luôn bị bỏ qua. `IgnoredGoFiles` dành cho các tệp bị bỏ qua do ràng buộc build.

<!-- CL 240551 -->
Trường mới [`IgnoredOtherFiles`](/pkg/go/build/#Package.IgnoredOtherFiles) của [`Package`](/pkg/go/build/#Package) có danh sách các tệp không phải Go bị bỏ qua do ràng buộc build.

<!-- go/build -->

#### [go/build/constraint](/pkg/go/build/constraint/)

<!-- CL 240604 -->
Gói mới [`go/build/constraint`](/pkg/go/build/constraint/) phân tích các dòng ràng buộc build, cả cú pháp `// +build` gốc và cú pháp `//go:build` sẽ được giới thiệu trong Go 1.17. Gói này tồn tại để các công cụ được build với Go 1.16 có thể xử lý mã nguồn Go 1.17. Xem [https://golang.org/design/draft-gobuild](/design/draft-gobuild) để biết chi tiết về các cú pháp ràng buộc build và kế hoạch chuyển đổi sang cú pháp `//go:build`. Lưu ý rằng các dòng `//go:build` **không** được hỗ trợ trong Go 1.16 và chưa nên được đưa vào chương trình Go.

<!-- go/build/constraint -->

#### [html/template](/pkg/html/template/)

<!-- CL 243938 -->
Hàm mới [`template.ParseFS`](/pkg/html/template/#ParseFS) và phương thức [`template.Template.ParseFS`](/pkg/html/template/#Template.ParseFS) giống như [`template.ParseGlob`](/pkg/html/template/#ParseGlob) và [`template.Template.ParseGlob`](/pkg/html/template/#Template.ParseGlob), nhưng đọc template từ [`fs.FS`](/pkg/io/fs/#FS).

<!-- html/template -->

#### [io](/pkg/io/)

<!-- CL 261577 -->
Gói giờ định nghĩa interface [`ReadSeekCloser`](/pkg/io/#ReadSeekCloser).

<!-- CL 263141 -->
Gói giờ định nghĩa [`Discard`](/pkg/io/#Discard), [`NopCloser`](/pkg/io/#NopCloser) và [`ReadAll`](/pkg/io/#ReadAll), để dùng thay cho các tên tương tự trong gói [`io/ioutil`](/pkg/io/ioutil/).

<!-- io -->

#### [log](/pkg/log/)

<!-- CL 264460 -->
Hàm mới [`Default`](/pkg/log/#Default) cung cấp quyền truy cập vào [`Logger`](/pkg/log/#Logger) mặc định.

<!-- log -->

#### [log/syslog](/pkg/log/syslog/)

<!-- CL 264297 -->
[`Writer`](/pkg/log/syslog/#Writer) giờ dùng định dạng thông báo cục bộ (bỏ tên host và dùng timestamp ngắn hơn) khi ghi log đến Unix domain socket tùy chỉnh, khớp với định dạng đã dùng cho socket log mặc định.

<!-- log/syslog -->

#### [mime/multipart](/pkg/mime/multipart/)

<!-- CL 247477 -->
Phương thức [`ReadForm`](/pkg/mime/multipart/#Reader.ReadForm) của [`Reader`](/pkg/mime/multipart/#Reader) không còn từ chối dữ liệu form khi được truyền giá trị int64 tối đa làm giới hạn.

<!-- mime/multipart -->

#### [net](/pkg/net/)

<!-- CL 250357 -->
Trường hợp I/O trên kết nối mạng đã đóng, hoặc I/O trên kết nối mạng bị đóng trước khi I/O hoàn tất, giờ có thể được phát hiện bằng lỗi mới [`ErrClosed`](/pkg/net/#ErrClosed). Cách dùng điển hình sẽ là `errors.Is(err, net.ErrClosed)`. Trong các bản phát hành trước, cách đáng tin cậy duy nhất để phát hiện trường hợp này là khớp chuỗi được trả về bởi phương thức `Error` với `"use of closed network connection"`.

<!-- CL 255898 -->
Trong các bản phát hành Go trước, kích thước backlog listener TCP mặc định trên hệ thống Linux, được đặt bởi `/proc/sys/net/core/somaxconn`, bị giới hạn tối đa là `65535`. Trên kernel Linux phiên bản 4.1 trở lên, tối đa giờ là `4294967295`.

<!-- CL 238629 -->
Trên Linux, tra cứu tên host không còn dùng DNS trước khi kiểm tra `/etc/hosts` khi `/etc/nsswitch.conf` bị thiếu; điều này phổ biến trên hệ thống dựa trên musl và làm cho chương trình Go khớp với hành vi của chương trình C trên các hệ thống đó.

<!-- net -->

#### [net/http](/pkg/net/http/)

<!-- CL 233637 -->
Trong gói [`net/http`](/pkg/net/http/), hành vi của [`StripPrefix`](/pkg/net/http/#StripPrefix) đã được thay đổi để cắt tiền tố khỏi trường `RawPath` của URL request ngoài trường `Path`. Trong các bản phát hành trước, chỉ trường `Path` được cắt, vì vậy nếu URL request chứa các ký tự được thoát, URL sẽ bị sửa đổi để có các trường `Path` và `RawPath` không khớp. Trong Go 1.16, `StripPrefix` cắt cả hai trường. Nếu có ký tự được thoát trong phần tiền tố của URL request, handler phục vụ 404 thay vì hành vi trước đây là gọi handler bên dưới với cặp `Path`/`RawPath` không khớp.

<!-- CL 252497 -->
Gói [`net/http`](/pkg/net/http/) giờ từ chối các HTTP range request dạng `"Range": "bytes=--N"` trong đó `"-N"` là độ dài hậu tố âm, ví dụ `"Range": "bytes=--2"`. Nó giờ phản hồi với `416 "Range Not Satisfiable"`.

<!-- CL 256498, golang.org/issue/36990 -->
Các cookie được đặt với [`SameSiteDefaultMode`](/pkg/net/http/#SameSiteDefaultMode) giờ hoạt động theo spec hiện tại (không có thuộc tính nào được đặt) thay vì tạo ra key SameSite không có giá trị.

<!-- CL 250039 -->
[`Client`](/pkg/net/http/#Client) giờ gửi header `Content-Length:` `0` tường minh trong các request `PATCH` với body rỗng, khớp với hành vi hiện có của `POST` và `PUT`.

<!-- CL 249440 -->
Hàm [`ProxyFromEnvironment`](/pkg/net/http/#ProxyFromEnvironment) không còn trả về cài đặt của biến môi trường `HTTP_PROXY` cho các URL `https://` khi `HTTPS_PROXY` chưa được đặt.

<!-- 259917 -->
Kiểu [`Transport`](/pkg/net/http/#Transport) có trường mới [`GetProxyConnectHeader`](/pkg/net/http/#Transport.GetProxyConnectHeader) có thể được đặt thành hàm trả về các header để gửi đến proxy trong request `CONNECT`. Trên thực tế, `GetProxyConnectHeader` là phiên bản động của trường hiện có [`ProxyConnectHeader`](/pkg/net/http/#Transport.ProxyConnectHeader); nếu `GetProxyConnectHeader` không phải `nil`, thì `ProxyConnectHeader` bị bỏ qua.

<!-- CL 243939 -->
Hàm mới [`http.FS`](/pkg/net/http/#FS) chuyển đổi [`fs.FS`](/pkg/io/fs/#FS) thành [`http.FileSystem`](/pkg/net/http/#FileSystem).

<!-- net/http -->

#### [net/http/httputil](/pkg/net/http/httputil/)

<!-- CL 260637 -->
[`ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy) giờ xả dữ liệu đệm tích cực hơn khi proxy các phản hồi được stream với độ dài body không xác định.

<!-- net/http/httputil -->

#### [net/smtp](/pkg/net/smtp/)

<!-- CL 247257 -->
Phương thức [`Mail`](/pkg/net/smtp/#Client.Mail) của [`Client`](/pkg/net/smtp/#Client) giờ gửi directive `SMTPUTF8` đến các server hỗ trợ nó, báo hiệu rằng các địa chỉ được mã hóa theo UTF-8.

<!-- net/smtp -->

#### [os](/pkg/os/)

<!-- CL 242998 -->
[`Process.Signal`](/pkg/os/#Process.Signal) giờ trả về [`ErrProcessDone`](/pkg/os/#ErrProcessDone) thay vì `errFinished` không được xuất khi tiến trình đã kết thúc.

<!-- CL 261540 -->
Gói định nghĩa kiểu mới [`DirEntry`](/pkg/os/#DirEntry) như alias cho [`fs.DirEntry`](/pkg/io/fs/#DirEntry). Hàm mới [`ReadDir`](/pkg/os/#ReadDir) và phương thức mới [`File.ReadDir`](/pkg/os/#File.ReadDir) có thể dùng để đọc nội dung thư mục thành slice của [`DirEntry`](/pkg/os/#DirEntry). Phương thức [`File.Readdir`](/pkg/os/#File.Readdir) (lưu ý chữ `d` thường trong `dir`) vẫn tồn tại, trả về slice của [`FileInfo`](/pkg/os/#FileInfo), nhưng với hầu hết chương trình sẽ hiệu quả hơn khi chuyển sang [`File.ReadDir`](/pkg/os/#File.ReadDir).

<!-- CL 263141 -->
Gói giờ định nghĩa [`CreateTemp`](/pkg/os/#CreateTemp), [`MkdirTemp`](/pkg/os/#MkdirTemp), [`ReadFile`](/pkg/os/#ReadFile) và [`WriteFile`](/pkg/os/#WriteFile), để dùng thay cho các hàm được định nghĩa trong gói [`io/ioutil`](/pkg/io/ioutil/).

<!-- CL 243906 -->
Các kiểu [`FileInfo`](/pkg/os/#FileInfo), [`FileMode`](/pkg/os/#FileMode) và [`PathError`](/pkg/os/#PathError) giờ là alias cho các kiểu cùng tên trong gói [`io/fs`](/pkg/io/fs/). Các chữ ký hàm trong gói [`os`](/pkg/os/) đã được cập nhật để tham chiếu đến các tên trong gói [`io/fs`](/pkg/io/fs/). Điều này không ảnh hưởng đến mã hiện có.

<!-- CL 243911 -->
Hàm mới [`DirFS`](/pkg/os/#DirFS) cung cấp triển khai [`fs.FS`](/pkg/io/fs/#FS) được hỗ trợ bởi cây tệp hệ điều hành.

<!-- os -->

#### [os/signal](/pkg/os/signal/)

<!-- CL 219640 -->
Hàm mới [`NotifyContext`](/pkg/os/signal/#NotifyContext) cho phép tạo context bị hủy khi nhận các tín hiệu cụ thể.

<!-- os/signal -->

#### [path](/pkg/path/)

<!-- CL 264397, golang.org/issues/28614 -->
Hàm [`Match`](/pkg/path/#Match) giờ trả về lỗi nếu phần không khớp của pattern có lỗi cú pháp. Trước đây, hàm trả về sớm khi khớp thất bại, do đó không báo cáo lỗi cú pháp sau trong pattern.

<!-- path -->

#### [path/filepath](/pkg/path/filepath/)

<!-- CL 267887 -->
Hàm mới [`WalkDir`](/pkg/path/filepath/#WalkDir) tương tự như [`Walk`](/pkg/path/filepath/#Walk), nhưng thường hiệu quả hơn. Hàm được truyền cho `WalkDir` nhận [`fs.DirEntry`](/pkg/io/fs/#DirEntry) thay vì [`fs.FileInfo`](/pkg/io/fs/#FileInfo). (Để làm rõ cho những ai nhớ hàm `Walk` nhận [`os.FileInfo`](/pkg/os/#FileInfo), `os.FileInfo` giờ là alias cho `fs.FileInfo`.)

<!-- CL 264397, golang.org/issues/28614 -->
Các hàm [`Match`](/pkg/path/filepath#Match) và [`Glob`](/pkg/path/filepath#Glob) giờ trả về lỗi nếu phần không khớp của pattern có lỗi cú pháp. Trước đây, các hàm trả về sớm khi khớp thất bại, do đó không báo cáo lỗi cú pháp sau trong pattern.

<!-- path/filepath -->

#### [reflect](/pkg/reflect/)

<!-- CL 192331 -->
Hàm Zero đã được tối ưu hóa để tránh cấp phát. Mã không đúng so sánh Value được trả về với Value khác bằng `==` hoặc `DeepEqual` có thể nhận được kết quả khác so với Go trước. Tài liệu về [`reflect.Value`](/pkg/reflect#Value) mô tả cách so sánh đúng hai `Value`.

<!-- reflect -->

#### [runtime/debug](/pkg/runtime/debug/)

<!-- CL 249677 -->
Các giá trị [`runtime.Error`](/pkg/runtime#Error) được dùng khi `SetPanicOnFault` được bật giờ có thể có phương thức `Addr`. Nếu phương thức đó tồn tại, nó trả về địa chỉ bộ nhớ kích hoạt lỗi.

<!-- runtime/debug -->

#### [strconv](/pkg/strconv/)

<!-- CL 260858 -->
[`ParseFloat`](/pkg/strconv/#ParseFloat) giờ dùng [thuật toán Eisel-Lemire](https://nigeltao.github.io/blog/2020/eisel-lemire.html), cải thiện hiệu năng lên đến gấp 2 lần. Điều này cũng có thể tăng tốc giải mã các định dạng văn bản như [`encoding/json`](/pkg/encoding/json/).

<!-- strconv -->

#### [syscall](/pkg/syscall/)

<!-- CL 263271 -->
[`NewCallback`](/pkg/syscall/?GOOS=windows#NewCallback) và [`NewCallbackCDecl`](/pkg/syscall/?GOOS=windows#NewCallbackCDecl) giờ hỗ trợ đúng các hàm callback với nhiều tham số kích thước nhỏ hơn `uintptr` liên tiếp. Điều này có thể yêu cầu thay đổi cách dùng các hàm này để loại bỏ đệm thủ công giữa các tham số nhỏ.

<!-- CL 261917 -->
[`SysProcAttr`](/pkg/syscall/?GOOS=windows#SysProcAttr) trên Windows có trường mới `NoInheritHandles` tắt kế thừa handle khi tạo tiến trình mới.

<!-- CL 269761, golang.org/issue/42584 -->
[`DLLError`](/pkg/syscall/?GOOS=windows#DLLError) trên Windows giờ có phương thức `Unwrap` để mở bao bọc lỗi bên dưới của nó.

<!-- CL 210639 -->
Trên Linux, [`Setgid`](/pkg/syscall/#Setgid), [`Setuid`](/pkg/syscall/#Setuid) và các lời gọi liên quan giờ được triển khai. Trước đây, chúng trả về lỗi `syscall.EOPNOTSUPP`.

<!-- CL 210639 -->
Trên Linux, các hàm mới [`AllThreadsSyscall`](/pkg/syscall/#AllThreadsSyscall) và [`AllThreadsSyscall6`](/pkg/syscall/#AllThreadsSyscall6) có thể dùng để thực hiện lời gọi hệ thống trên tất cả các luồng Go trong tiến trình. Các hàm này chỉ có thể dùng bởi các chương trình không dùng cgo; nếu chương trình dùng cgo, chúng sẽ luôn trả về [`syscall.ENOTSUP`](/pkg/syscall/#ENOTSUP).

<!-- syscall -->

#### [testing/iotest](/pkg/testing/iotest/)

<!-- CL 199501 -->
Hàm mới [`ErrReader`](/pkg/testing/iotest/#ErrReader) trả về [`io.Reader`](/pkg/io/#Reader) luôn trả về lỗi.

<!-- CL 243909 -->
Hàm mới [`TestReader`](/pkg/testing/iotest/#TestReader) kiểm thử rằng [`io.Reader`](/pkg/io/#Reader) hoạt động đúng.

<!-- testing/iotest -->

#### [text/template](/pkg/text/template/)

<!-- CL 254257, golang.org/issue/29770 -->
Ký tự dòng mới giờ được cho phép bên trong dấu phân cách action, cho phép action kéo dài nhiều dòng.

<!-- CL 243938 -->
Hàm mới [`template.ParseFS`](/pkg/text/template/#ParseFS) và phương thức [`template.Template.ParseFS`](/pkg/text/template/#Template.ParseFS) giống như [`template.ParseGlob`](/pkg/text/template/#ParseGlob) và [`template.Template.ParseGlob`](/pkg/text/template/#Template.ParseGlob), nhưng đọc template từ [`fs.FS`](/pkg/io/fs/#FS).

<!-- text/template -->

#### [text/template/parse](/pkg/text/template/parse/)

<!-- CL 229398, golang.org/issue/34652 -->
[`CommentNode`](/pkg/text/template/parse/#CommentNode) mới đã được thêm vào parse tree. Trường [`Mode`](/pkg/text/template/parse/#Mode) trong `parse.Tree` cho phép truy cập vào nó.

<!-- text/template/parse -->

#### [time/tzdata](/pkg/time/tzdata/)

<!-- CL 261877 -->
Định dạng dữ liệu múi giờ slim giờ được dùng cho cơ sở dữ liệu múi giờ trong `$GOROOT/lib/time/zoneinfo.zip` và bản sao nhúng trong gói này. Điều này giảm kích thước cơ sở dữ liệu múi giờ khoảng 350 KB.

<!-- time/tzdata -->

#### [unicode](/pkg/unicode/)

<!-- CL 248765 -->
Gói [`unicode`](/pkg/unicode/) và hỗ trợ liên quan trong toàn hệ thống đã được nâng cấp từ Unicode 12.0.0 lên [Unicode 13.0.0](https://www.unicode.org/versions/Unicode13.0.0/), bổ sung 5.930 ký tự mới, bao gồm bốn script mới và 55 emoji mới. Unicode 13.0.0 cũng chỉ định plane 3 (U+30000-U+3FFFF) là tertiary ideographic plane.

<!-- unicode -->
