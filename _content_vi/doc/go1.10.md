---
title: Ghi chú phát hành Go 1.10
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

## Giới thiệu về Go 1.10 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.10, ra đời sáu tháng sau [Go 1.9](go1.9). Phần lớn các thay đổi nằm ở việc triển khai trình biên dịch, runtime và thư viện. Như thường lệ, bản phát hành duy trì [cam kết tương thích Go 1](/doc/go1compat.html). Chúng tôi mong rằng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

Bản phát hành này cải thiện [bộ nhớ đệm cho các gói đã build](#build), thêm [bộ nhớ đệm cho kết quả kiểm thử thành công](#test), tự động [chạy vet trong quá trình kiểm thử](#test-vet), và cho phép [truyền giá trị string trực tiếp giữa Go và C khi dùng cgo](#cgo). Một [tập cố định các tùy chọn compiler an toàn](#cgo) mới có thể gây ra các lỗi [`invalid flag`](/s/invalidflag) bất ngờ trong mã đã biên dịch thành công với các bản phát hành trước.

## Thay đổi về ngôn ngữ {#language}

Không có thay đổi đáng kể nào về đặc tả ngôn ngữ.

<!-- CL 60230 -->
Một trường hợp đặc biệt liên quan đến dịch chuyển bit của các hằng số không có kiểu đã được làm rõ, và kết quả là các trình biên dịch đã được cập nhật để cho phép biểu thức chỉ số `x[1.0`&nbsp;`<<`&nbsp;`s]` trong đó `s` là số nguyên không dấu; gói [go/types](/pkg/go/types/) đã làm điều này rồi.

<!-- CL 73233 -->
Cú pháp cho biểu thức phương thức đã được cập nhật để cho phép bất kỳ biểu thức kiểu nào làm receiver; điều này khớp với những gì các trình biên dịch đã triển khai. Ví dụ, `struct{io.Reader}.Read` là một biểu thức phương thức hợp lệ, dù không phổ biến, mà các trình biên dịch đã chấp nhận và giờ được phép bởi cú pháp ngôn ngữ.

## Các nền tảng {#ports}

Không có hệ điều hành hay kiến trúc bộ xử lý mới được hỗ trợ trong bản phát hành này. Phần lớn công việc tập trung vào việc củng cố hỗ trợ cho các nền tảng hiện có, đặc biệt là [các lệnh mới trong trình hợp ngữ](#asm) và các cải tiến về mã được tạo ra bởi các trình biên dịch.

Như [đã thông báo trong ghi chú phát hành Go 1.9](go1.9#freebsd), Go 1.10 giờ yêu cầu FreeBSD 10.3 hoặc mới hơn; hỗ trợ cho FreeBSD 9.3 đã bị xóa.

Go giờ chạy trên NetBSD trở lại nhưng yêu cầu NetBSD 8 chưa được phát hành. Chỉ `GOARCH` `amd64` và `386` được sửa. Cổng `arm` vẫn còn bị hỏng.

Trên các hệ thống MIPS 32-bit, các cài đặt biến môi trường mới `GOMIPS=hardfloat` (mặc định) và `GOMIPS=softfloat` chọn việc sử dụng lệnh phần cứng hay mô phỏng phần mềm cho các phép tính dấu phẩy động.

Go 1.10 là bản phát hành cuối cùng chạy trên OpenBSD 6.0. Go 1.11 sẽ yêu cầu OpenBSD 6.2.

Go 1.10 là bản phát hành cuối cùng chạy trên OS X 10.8 Mountain Lion hoặc OS X 10.9 Mavericks. Go 1.11 sẽ yêu cầu OS X 10.10 Yosemite hoặc mới hơn.

Go 1.10 là bản phát hành cuối cùng chạy trên Windows XP hoặc Windows Vista. Go 1.11 sẽ yêu cầu Windows 7 hoặc mới hơn.

## Công cụ {#tools}

### GOROOT & GOTMPDIR mặc định {#goroot}

Nếu biến môi trường `$GOROOT` không được đặt, công cụ go trước đây sử dụng `GOROOT` mặc định được đặt trong quá trình biên dịch toolchain. Giờ, trước khi dùng giá trị mặc định đó, công cụ go cố suy luận `GOROOT` từ đường dẫn thực thi của chính nó. Điều này cho phép các bản phân phối nhị phân được giải nén ở bất kỳ đâu trong hệ thống tệp và sau đó sử dụng mà không cần đặt `GOROOT` một cách tường minh.

Theo mặc định, công cụ go tạo các tệp và thư mục tạm thời trong thư mục tạm thời của hệ thống (ví dụ: `$TMPDIR` trên Unix). Nếu biến môi trường mới `$GOTMPDIR` được đặt, công cụ go sẽ tạo các tệp và thư mục tạm thời trong thư mục đó thay thế.

### Build & Install {#build}

Lệnh `go`&nbsp;`build` giờ phát hiện các gói lỗi thời hoàn toàn dựa trên nội dung của các tệp nguồn, các cờ build được chỉ định và siêu dữ liệu được lưu trữ trong các gói đã biên dịch. Thời gian sửa đổi không còn được tham khảo hay liên quan. Lời khuyên cũ về việc thêm `-a` để buộc build lại trong các trường hợp thời gian sửa đổi gây hiểu lầm vì lý do này hay lý do khác (ví dụ: thay đổi cờ build) không còn cần thiết nữa: các build giờ luôn phát hiện khi nào gói cần được build lại. (Nếu bạn thấy khác đi, vui lòng báo lỗi.)

Các tùy chọn `go`&nbsp;`build` `-asmflags`, `-gcflags`, `-gccgoflags` và `-ldflags` giờ chỉ áp dụng theo mặc định cho các gói được liệt kê trực tiếp trên dòng lệnh. Ví dụ: `go` `build` `-gcflags=-m` `mypkg` truyền cờ `-m` cho trình biên dịch khi build `mypkg` nhưng không truyền cho các dependency của nó. Dạng mới, tổng quát hơn `-asmflags=pattern=flags` (và tương tự cho các tùy chọn khác) áp dụng `flags` chỉ cho các gói khớp với pattern. Ví dụ: `go` `install` `-ldflags=cmd/gofmt=-X=main.version=1.2.3` `cmd/...` cài đặt tất cả các lệnh khớp với `cmd/...` nhưng chỉ áp dụng tùy chọn `-X` cho cờ linker của `cmd/gofmt`. Để biết thêm chi tiết, xem [`go` `help` `build`](/cmd/go/#hdr-Compile_packages_and_dependencies).

Lệnh `go`&nbsp;`build` giờ duy trì một bộ nhớ đệm của các gói đã build gần đây, tách biệt với các gói đã cài đặt trong `$GOROOT/pkg` hoặc `$GOPATH/pkg`. Hiệu quả của bộ nhớ đệm là tăng tốc build khi không cài đặt tường minh các gói hay khi chuyển đổi giữa các bản sao khác nhau của mã nguồn (ví dụ: khi chuyển đổi qua lại giữa các nhánh khác nhau trong hệ thống kiểm soát phiên bản). Lời khuyên cũ về việc thêm cờ `-i` để tăng tốc, như trong `go` `build` `-i` hoặc `go` `test` `-i`, không còn cần thiết nữa: các build chạy nhanh như vậy ngay cả không có `-i`. Để biết thêm chi tiết, xem [`go` `help` `cache`](/cmd/go/#hdr-Build_and_test_caching).

Lệnh `go`&nbsp;`install` giờ chỉ cài đặt các gói và lệnh được liệt kê trực tiếp trên dòng lệnh. Ví dụ: `go` `install` `cmd/gofmt` cài đặt chương trình gofmt nhưng không cài đặt các gói mà nó phụ thuộc vào. Bộ nhớ đệm build mới giúp các lệnh trong tương lai vẫn chạy nhanh như thể các dependency đã được cài đặt. Để buộc cài đặt các dependency, sử dụng cờ `go` `install` `-i` mới. Nhìn chung, việc cài đặt các gói dependency không cần thiết, và khái niệm về các gói đã cài đặt có thể biến mất trong một bản phát hành tương lai.

Nhiều chi tiết của việc triển khai `go`&nbsp;`build` đã thay đổi để hỗ trợ những cải tiến này. Một yêu cầu mới ngụ ý bởi những thay đổi này là các gói chỉ có nhị phân giờ phải khai báo các khối import chính xác trong mã nguồn stub của chúng, để các import đó có thể được cung cấp khi liên kết một chương trình sử dụng gói chỉ có nhị phân. Để biết thêm chi tiết, xem [`go` `help` `filetype`](/cmd/go/#hdr-File_types).

### Test {#test}

Lệnh `go`&nbsp;`test` giờ lưu bộ nhớ đệm kết quả kiểm thử: nếu tệp thực thi kiểm thử và dòng lệnh khớp với một lần chạy trước và các tệp cũng như biến môi trường được tham khảo bởi lần chạy đó không thay đổi, `go` `test` sẽ in kết quả kiểm thử trước đó, thay thế thời gian đã trôi qua bằng chuỗi "(cached)." Bộ nhớ đệm kiểm thử chỉ áp dụng cho kết quả kiểm thử thành công; chỉ cho các lệnh `go` `test` với danh sách gói tường minh; và chỉ cho các dòng lệnh sử dụng một tập con của các cờ kiểm thử `-cpu`, `-list`, `-parallel`, `-run`, `-short` và `-v`. Cách thường dùng để bỏ qua bộ nhớ đệm kiểm thử là dùng `-count=1`.

Lệnh `go`&nbsp;`test` giờ tự động chạy `go` `vet` trên gói đang được kiểm thử, để xác định các vấn đề quan trọng trước khi chạy kiểm thử. Bất kỳ vấn đề nào như vậy đều được xử lý như lỗi build và ngăn thực thi kiểm thử. Chỉ một tập con có độ tin cậy cao của các kiểm tra `go` `vet` hiện có được bật cho lần kiểm tra tự động này. Để tắt việc chạy `go` `vet`, sử dụng `go` `test` `-vet=off`.

Cờ `go` `test` `-coverpkg` giờ diễn giải đối số của nó như một danh sách các pattern phân tách bằng dấu phẩy để khớp với các dependency của mỗi kiểm thử, thay vì một danh sách các gói để nạp mới. Ví dụ: `go` `test` `-coverpkg=all` giờ là một cách có ý nghĩa để chạy kiểm thử với coverage được bật cho gói kiểm thử và tất cả các dependency của nó. Ngoài ra, tùy chọn `go` `test` `-coverprofile` giờ được hỗ trợ khi chạy nhiều kiểm thử.

Trong trường hợp thất bại do timeout, các kiểm thử giờ có nhiều khả năng ghi các profile của chúng trước khi thoát.

Lệnh `go`&nbsp;`test` giờ luôn hợp nhất đầu ra chuẩn và lỗi chuẩn từ một lần thực thi tệp nhị phân kiểm thử nhất định và ghi cả hai vào đầu ra chuẩn của `go` `test`. Trong các bản phát hành trước, `go` `test` chỉ áp dụng việc hợp nhất này hầu hết thời gian.

Đầu ra `go`&nbsp;`test` `-v` giờ bao gồm các dòng cập nhật trạng thái `PAUSE` và `CONT` để đánh dấu khi nào [các kiểm thử song song](/pkg/testing/#T.Parallel) tạm dừng và tiếp tục.

Cờ `go` `test` `-failfast` mới tắt việc chạy các kiểm thử bổ sung sau khi bất kỳ kiểm thử nào thất bại. Lưu ý rằng các kiểm thử đang chạy song song với kiểm thử thất bại được phép hoàn thành.

Cuối cùng, cờ `go` `test` `-json` mới lọc đầu ra kiểm thử qua lệnh mới `go` `tool` `test2json` để tạo ra mô tả thực thi kiểm thử định dạng JSON có thể đọc được bởi máy. Điều này cho phép tạo các bản trình bày phong phú về thực thi kiểm thử trong các IDE và công cụ khác.

Để biết thêm chi tiết về tất cả những thay đổi này, xem [`go` `help` `test`](/cmd/go/#hdr-Test_packages) và [tài liệu test2json](/cmd/test2json/).

### Cgo {#cgo}

Các tùy chọn được chỉ định bởi cgo bằng `#cgo CFLAGS` và tương tự giờ được kiểm tra dựa trên một danh sách các tùy chọn được phép. Điều này đóng một lỗ hổng bảo mật trong đó một gói được tải xuống sử dụng các tùy chọn compiler như <span style="white-space: nowrap">`-fplugin`</span> để chạy mã tùy ý trên máy nơi nó đang được build. Điều này có thể gây ra lỗi build như `invalid flag in #cgo CFLAGS`. Để biết thêm nền tảng và cách xử lý lỗi này, xem [https://golang.org/s/invalidflag](/s/invalidflag).

Cgo giờ triển khai một typedef C như "`typedef` `X` `Y`" bằng cách sử dụng bí danh kiểu Go, để mã Go có thể sử dụng các kiểu `C.X` và `C.Y` thay thế nhau. Nó cũng hỗ trợ việc sử dụng các macro dạng hàm không tham số. Ngoài ra, tài liệu đã được cập nhật để làm rõ rằng các struct Go và mảng Go không được hỗ trợ trong các chữ ký kiểu của các hàm được xuất qua cgo.

Cgo giờ hỗ trợ truy cập trực tiếp vào các giá trị string Go từ C. Các hàm trong preamble C có thể sử dụng kiểu `_GoString_` để nhận một string Go làm đối số. Mã C có thể gọi `_GoStringLen` và `_GoStringPtr` để truy cập trực tiếp nội dung của string. Một giá trị kiểu `_GoString_` có thể được truyền trong một lời gọi đến một hàm Go được xuất nhận đối số kiểu string Go.

Trong quá trình bootstrap toolchain, các biến môi trường `CC` và `CC_FOR_TARGET` chỉ định trình biên dịch C mặc định mà toolchain kết quả sẽ sử dụng cho các build host và target, tương ứng. Tuy nhiên, nếu toolchain sẽ được sử dụng với nhiều target, có thể cần chỉ định một trình biên dịch C khác cho mỗi target (ví dụ: một trình biên dịch khác cho `darwin/arm64` so với `linux/ppc64le`). Tập biến môi trường mới <code>CC\_FOR\__goos_\__goarch_</code> cho phép chỉ định trình biên dịch C mặc định khác cho mỗi target. Lưu ý rằng các biến này chỉ áp dụng trong quá trình bootstrap toolchain, để đặt các giá trị mặc định được sử dụng bởi toolchain kết quả. Các lệnh `go` `build` sau đó sử dụng biến môi trường `CC` hoặc giá trị mặc định được tích hợp sẵn.

Cgo giờ dịch một số kiểu C thường ánh xạ sang kiểu con trỏ trong Go sang `uintptr` thay thế. Các kiểu này bao gồm hệ thống phân cấp `CFTypeRef` trong framework CoreFoundation của Darwin và hệ thống phân cấp `jobject` trong giao diện JNI của Java.

Các kiểu này phải là `uintptr` ở phía Go vì nếu không chúng sẽ gây nhầm lẫn cho bộ gom rác Go; đôi khi chúng không thực sự là con trỏ mà là các cấu trúc dữ liệu được mã hóa trong một số nguyên có kích thước con trỏ. Con trỏ đến bộ nhớ Go không được lưu trữ trong các giá trị `uintptr` này.

Do thay đổi này, các giá trị của các kiểu bị ảnh hưởng cần được khởi tạo bằng hằng số `0` thay vì hằng số `nil`. Go 1.10 cung cấp các module `gofix` để giúp viết lại đó:

	go tool fix -r cftype <pkg>
	go tool fix -r jni <pkg>

Để biết thêm chi tiết, xem [tài liệu cgo](/cmd/cgo/).

### Doc {#doc}

Công cụ `go`&nbsp;`doc` giờ thêm các hàm trả về slice của `T` hoặc `*T` vào phần hiển thị của kiểu `T`, tương tự như hành vi hiện có đối với các hàm trả về kết quả `T` hoặc `*T` đơn. Ví dụ:

	$ go doc mail.Address
	package mail // import "net/mail"

	type Address struct {
		Name    string
		Address string
	}
	    Address represents a single mail address.

	func ParseAddress(address string) (*Address, error)
	func ParseAddressList(list string) ([]*Address, error)
	func (a *Address) String() string
	$

Trước đây, `ParseAddressList` chỉ được hiển thị trong tổng quan gói (`go` `doc` `mail`).

### Fix {#fix}

Công cụ `go`&nbsp;`fix` giờ thay thế các import của `"golang.org/x/net/context"` bằng `"context"`. (Các alias chuyển tiếp trong gói trước làm cho nó hoàn toàn tương đương với gói sau khi sử dụng Go 1.9 trở lên.)

### Get {#get}

Lệnh `go`&nbsp;`get` giờ hỗ trợ các kho lưu trữ mã nguồn Fossil.

### Pprof {#pprof}

Các profile blocking và mutex được tạo bởi gói `runtime/pprof` giờ bao gồm thông tin symbol, do đó chúng có thể được xem bằng `go` `tool` `pprof` mà không cần tệp nhị phân đã tạo ra profile. (Tất cả các loại profile khác đã được thay đổi để bao gồm thông tin symbol trong Go 1.9.)

Trình hiển thị profile [`go`&nbsp;`tool`&nbsp;`pprof`](/cmd/pprof/) đã được cập nhật lên phiên bản git 9e20b5b (2017-11-08) từ [github.com/google/pprof](https://github.com/google/pprof), bao gồm giao diện web được cập nhật.

### Vet {#vet}

Lệnh [`go`&nbsp;`vet`](/cmd/vet/) giờ luôn có quyền truy cập vào thông tin kiểu đầy đủ, cập nhật khi kiểm tra các gói, ngay cả đối với các gói sử dụng cgo hoặc các import qua vendor. Kết quả là các báo cáo sẽ chính xác hơn. Lưu ý rằng chỉ `go`&nbsp;`vet` có quyền truy cập vào thông tin này; `go`&nbsp;`tool`&nbsp;`vet` ở cấp thấp hơn thì không và nên tránh sử dụng trừ khi làm việc trên chính `vet`. (Kể từ Go 1.9, `go`&nbsp;`vet` cung cấp quyền truy cập vào tất cả các cờ giống như `go`&nbsp;`tool`&nbsp;`vet`.)

### Chẩn đoán {#diag}

Bản phát hành này bao gồm một [tổng quan mới về các công cụ chẩn đoán chương trình Go có sẵn](/doc/diagnostics.html).

### Gofmt {#gofmt}

Hai chi tiết nhỏ của định dạng mặc định mã nguồn Go đã thay đổi. Thứ nhất, một số biểu thức slice ba chỉ số phức tạp trước đây được định dạng như `x[i+1`&nbsp;`:`&nbsp;`j:k]` và giờ được định dạng với khoảng cách nhất quán hơn: `x[i+1`&nbsp;`:`&nbsp;`j`&nbsp;`:`&nbsp;`k]`. Thứ hai, các literal interface một phương thức được viết trên một dòng, đôi khi được sử dụng trong type assertion, không còn bị tách sang nhiều dòng.

Lưu ý rằng những loại cập nhật nhỏ cho gofmt như thế này được mong đợi sẽ xảy ra theo thời gian. Nói chung, chúng tôi khuyến cáo không nên xây dựng các hệ thống kiểm tra xem mã nguồn có khớp với đầu ra của một phiên bản gofmt cụ thể hay không. Ví dụ, một kiểm thử tích hợp liên tục thất bại nếu bất kỳ mã nào đã được commit vào kho lưu trữ không "được định dạng đúng" về cơ bản là mong manh và không được khuyến nghị.

Nếu nhiều chương trình phải đồng ý về phiên bản gofmt nào được sử dụng để định dạng tệp nguồn, chúng tôi khuyến nghị chúng thực hiện điều này bằng cách sắp xếp để gọi cùng một binary gofmt. Ví dụ: trong kho lưu trữ mã nguồn mở Go, hook pre-commit Git của chúng tôi được viết bằng Go và có thể import `go/format` trực tiếp, nhưng thay vào đó nó gọi binary `gofmt` được tìm thấy trong đường dẫn hiện tại, để hook pre-commit không cần được biên dịch lại mỗi khi `gofmt` thay đổi.

### Trình biên dịch {#compiler}

Trình biên dịch bao gồm nhiều cải tiến về hiệu suất của mã được tạo, phân bố khá đồng đều trên các kiến trúc được hỗ trợ.

Thông tin gỡ lỗi DWARF được ghi trong các binary đã được cải thiện theo một số cách: giá trị hằng số giờ được ghi lại; thông tin số dòng chính xác hơn, giúp việc duyệt qua chương trình ở cấp nguồn hoạt động tốt hơn; và mỗi gói giờ được trình bày như một đơn vị biên dịch DWARF riêng của nó.

Các [chế độ build](https://docs.google.com/document/d/1nr-TQHw_er6GOQRsF6T43GGhFDelrAP0NqSS_00RgZQ/edit) khác nhau đã được chuyển sang nhiều hệ thống hơn. Cụ thể, `c-shared` giờ hoạt động trên `linux/ppc64le`, `windows/386` và `windows/amd64`; `pie` giờ hoạt động trên `darwin/amd64` và cũng buộc sử dụng liên kết ngoài trên tất cả các hệ thống; và `plugin` giờ hoạt động trên `linux/ppc64le` và `darwin/amd64`.

Cổng `linux/ppc64le` giờ yêu cầu sử dụng liên kết ngoài với bất kỳ chương trình nào sử dụng cgo, ngay cả các sử dụng bởi thư viện chuẩn.

### Trình hợp ngữ {#asm}

Đối với cổng ARM 32-bit, trình hợp ngữ giờ hỗ trợ các lệnh
<code><small>BFC</small></code>,
<code><small>BFI</small></code>,
<code><small>BFX</small></code>,
<code><small>BFXU</small></code>,
<code><small>FMULAD</small></code>,
<code><small>FMULAF</small></code>,
<code><small>FMULSD</small></code>,
<code><small>FMULSF</small></code>,
<code><small>FNMULAD</small></code>,
<code><small>FNMULAF</small></code>,
<code><small>FNMULSD</small></code>,
<code><small>FNMULSF</small></code>,
<code><small>MULAD</small></code>,
<code><small>MULAF</small></code>,
<code><small>MULSD</small></code>,
<code><small>MULSF</small></code>,
<code><small>NMULAD</small></code>,
<code><small>NMULAF</small></code>,
<code><small>NMULD</small></code>,
<code><small>NMULF</small></code>,
<code><small>NMULSD</small></code>,
<code><small>NMULSF</small></code>,
<code><small>XTAB</small></code>,
<code><small>XTABU</small></code>,
<code><small>XTAH</small></code>
và
<code><small>XTAHU</small></code>.

Đối với cổng ARM 64-bit, trình hợp ngữ giờ hỗ trợ các lệnh
<code><small>VADD</small></code>,
<code><small>VADDP</small></code>,
<code><small>VADDV</small></code>,
<code><small>VAND</small></code>,
<code><small>VCMEQ</small></code>,
<code><small>VDUP</small></code>,
<code><small>VEOR</small></code>,
<code><small>VLD1</small></code>,
<code><small>VMOV</small></code>,
<code><small>VMOVI</small></code>,
<code><small>VMOVS</small></code>,
<code><small>VORR</small></code>,
<code><small>VREV32</small></code>
và
<code><small>VST1</small></code>.

Đối với cổng PowerPC 64-bit, trình hợp ngữ giờ hỗ trợ các lệnh POWER9
<code><small>ADDEX</small></code>,
<code><small>CMPEQB</small></code>,
<code><small>COPY</small></code>,
<code><small>DARN</small></code>,
<code><small>LDMX</small></code>,
<code><small>MADDHD</small></code>,
<code><small>MADDHDU</small></code>,
<code><small>MADDLD</small></code>,
<code><small>MFVSRLD</small></code>,
<code><small>MTVSRDD</small></code>,
<code><small>MTVSRWS</small></code>,
<code><small>PASTECC</small></code>,
<code><small>VCMPNEZB</small></code>,
<code><small>VCMPNEZBCC</small></code>
và
<code><small>VMSUMUDM</small></code>.

Đối với cổng S390X, trình hợp ngữ giờ hỗ trợ các lệnh
<code><small>TMHH</small></code>,
<code><small>TMHL</small></code>,
<code><small>TMLH</small></code>
và
<code><small>TMLL</small></code>.

Đối với cổng X86 64-bit, trình hợp ngữ giờ hỗ trợ 359 lệnh mới, bao gồm các tập mở rộng đầy đủ AVX, AVX2, BMI, BMI2, F16C, FMA3, SSE2, SSE3, SSSE3, SSE4.1 và SSE4.2. Trình hợp ngữ cũng không còn triển khai <code><small>MOVL</small></code>&nbsp;<code><small>$0,</small></code>&nbsp;<code><small>AX</small></code> như một lệnh <code><small>XORL</small></code> nữa, để tránh xóa các cờ điều kiện một cách bất ngờ.

### Gccgo {#gccgo}

Do sự đồng bộ lịch phát hành nửa năm một lần của Go với lịch phát hành hàng năm của GCC, GCC phiên bản 7 chứa phiên bản Go 1.8.3 của gccgo. Chúng tôi dự kiến rằng bản phát hành tiếp theo, GCC 8, sẽ chứa phiên bản Go 1.10 của gccgo.

## Runtime {#runtime}

Hành vi của các lời gọi lồng nhau đến [`LockOSThread`](/pkg/runtime/#LockOSThread) và [`UnlockOSThread`](/pkg/runtime/#UnlockOSThread) đã thay đổi. Các hàm này kiểm soát xem một goroutine có bị khóa vào một thread hệ điều hành cụ thể hay không, để goroutine chỉ chạy trên thread đó, và thread đó chỉ chạy goroutine đó. Trước đây, gọi `LockOSThread` nhiều lần liên tiếp tương đương với gọi một lần, và một lần `UnlockOSThread` duy nhất luôn mở khóa thread. Giờ, các lời gọi được lồng nhau: nếu `LockOSThread` được gọi nhiều lần, `UnlockOSThread` phải được gọi cùng số lần để mở khóa thread. Mã hiện có cẩn thận không lồng các lời gọi này sẽ vẫn đúng. Mã hiện có giả định không đúng rằng các lời gọi được lồng nhau sẽ trở nên đúng. Hầu hết các sử dụng các hàm này trong mã nguồn Go công khai thuộc danh mục thứ hai.

Vì một cách sử dụng phổ biến của `LockOSThread` và `UnlockOSThread` là cho phép mã Go sửa đổi đáng tin cậy trạng thái thread-local (ví dụ: không gian tên Linux hoặc Plan 9), runtime giờ coi các thread bị khóa là không phù hợp để tái sử dụng hoặc tạo các thread mới.

Stack trace không còn bao gồm các hàm wrapper ngầm (trước đây được đánh dấu `<autogenerated>`), trừ khi lỗi hoặc panic xảy ra trong chính wrapper. Kết quả là, các số đếm skip được truyền cho các hàm như [`Caller`](/pkg/runtime/#Caller) giờ phải luôn khớp với cấu trúc của mã như được viết, thay vì phụ thuộc vào các quyết định tối ưu hóa và chi tiết triển khai.

Bộ gom rác đã được sửa đổi để giảm tác động của nó lên độ trễ phân bổ. Giờ nó sử dụng ít hơn một phần nhỏ của tổng CPU khi chạy, nhưng nó có thể chạy nhiều thời gian hơn. Tổng CPU tiêu thụ bởi bộ gom rác không thay đổi đáng kể.

Hàm [`GOROOT`](/pkg/runtime/#GOROOT) giờ mặc định (khi biến môi trường `$GOROOT` không được đặt) thành `GOROOT` hoặc `GOROOT_FINAL` có hiệu lực tại thời điểm chương trình đang gọi được biên dịch. Trước đây nó sử dụng `GOROOT` hoặc `GOROOT_FINAL` có hiệu lực tại thời điểm toolchain biên dịch chương trình đang gọi được biên dịch.

Không còn giới hạn về cài đặt [`GOMAXPROCS`](/pkg/runtime/#GOMAXPROCS) nữa. (Trong Go 1.9, giới hạn là 1024.)

## Hiệu suất {#performance}

Như thường lệ, các thay đổi rất đa dạng đến mức khó có thể đưa ra các nhận xét chính xác về hiệu suất. Hầu hết các chương trình nên chạy nhanh hơn một chút, nhờ các cải tiến trong bộ gom rác, mã được tạo tốt hơn và các tối ưu hóa trong thư viện cốt lõi.

## Bộ gom rác {#gc}

Nhiều ứng dụng nên trải nghiệm độ trễ phân bổ thấp hơn đáng kể và overhead hiệu suất tổng thể khi bộ gom rác đang hoạt động.

## Thư viện chuẩn {#library}

Tất cả các thay đổi đối với thư viện chuẩn đều nhỏ. Các thay đổi trong [bytes](#bytes) và [net/url](#net/url) có khả năng yêu cầu cập nhật các chương trình hiện có nhất.

### Các thay đổi nhỏ với thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ khác nhau đối với thư viện, được thực hiện với [cam kết tương thích Go 1](/doc/go1compat) trong tâm trí.

#### [archive/tar](/pkg/archive/tar/)

Nhìn chung, việc xử lý các định dạng header đặc biệt được cải thiện và mở rộng đáng kể.

[`FileInfoHeader`](/pkg/archive/tar/#FileInfoHeader) luôn ghi lại số UID và GID Unix từ đối số [`os.FileInfo`](/pkg/os/#FileInfo) của nó (cụ thể là từ thông tin phụ thuộc hệ thống được trả về bởi phương thức `Sys` của `FileInfo`) trong [`Header`](/pkg/archive/tar/#Header) được trả về. Giờ nó cũng ghi lại tên người dùng và tên nhóm tương ứng với các ID đó, cũng như số thiết bị major và minor cho các tệp thiết bị.

Trường [`Header.Format`](/pkg/archive/tar/#Header) mới có kiểu [`Format`](/pkg/archive/tar/#Format) kiểm soát định dạng header tar nào [`Writer`](/pkg/archive/tar/#Writer) sử dụng. Mặc định, như trước đây, là chọn kiểu header được hỗ trợ rộng rãi nhất có thể mã hóa các trường cần thiết bởi header (USTAR nếu có thể, hoặc PAX nếu có thể, hoặc GNU). [`Reader`](/pkg/archive/tar/#Reader) đặt `Header.Format` cho mỗi header nó đọc.

`Reader` và `Writer` giờ hỗ trợ các bản ghi PAX tùy ý, sử dụng trường [`Header.PAXRecords`](/pkg/archive/tar/#Header) mới, một sự tổng quát hóa của trường `Xattrs` hiện có.

`Reader` không còn khăng khăng rằng tên tệp hoặc tên liên kết trong các header GNU phải là UTF-8 hợp lệ.

Khi viết các header định dạng PAX hoặc GNU, `Writer` giờ bao gồm các trường `Header.AccessTime` và `Header.ChangeTime` (nếu được đặt). Khi viết các header định dạng PAX, thời gian bao gồm độ chính xác dưới giây.

#### [archive/zip](/pkg/archive/zip/)

Go 1.10 thêm hỗ trợ đầy đủ hơn cho thời gian và mã hóa bộ ký tự trong các tệp ZIP.

Định dạng ZIP gốc sử dụng mã hóa MS-DOS tiêu chuẩn của năm, tháng, ngày, giờ, phút và giây vào các trường trong hai giá trị 16-bit. Mã hóa đó không thể đại diện cho múi giờ hoặc giây lẻ, vì vậy nhiều phần mở rộng đã được giới thiệu để cho phép mã hóa phong phú hơn. Trong Go 1.10, [`Reader`](/pkg/archive/zip/#Reader) và [`Writer`](/pkg/archive/zip/#Writer) giờ hỗ trợ phần mở rộng Info-Zip được hiểu rộng rãi mã hóa thời gian riêng biệt dưới dạng Unix 32-bit "giây kể từ epoch". Trường `Modified` mới của [`FileHeader`](/pkg/archive/zip/#FileHeader) có kiểu [`time.Time`](/pkg/time/#Time) làm lỗi thời các trường `ModifiedTime` và `ModifiedDate`, tiếp tục giữ mã hóa MS-DOS. `Reader` và `Writer` giờ áp dụng quy ước thông thường rằng một kho lưu trữ ZIP lưu trữ thời gian Unix độc lập với múi giờ cũng lưu trữ thời gian địa phương trong trường MS-DOS, để múi giờ offset có thể được suy luận. Để tương thích, các phương thức [`ModTime`](/pkg/archive/zip/#FileHeader.ModTime) và [`SetModTime`](/pkg/archive/zip/#FileHeader.SetModTime) hoạt động giống như trong các bản phát hành trước; mã mới nên sử dụng `Modified` trực tiếp.

Header cho mỗi tệp trong kho lưu trữ ZIP có một bit cờ chỉ ra xem các trường tên và comment được mã hóa dưới dạng UTF-8, trái với mã hóa mặc định dành riêng cho hệ thống. Trong Go 1.8 và trước đó, `Writer` không bao giờ đặt bit UTF-8. Trong Go 1.9, `Writer` thay đổi để đặt bit UTF-8 hầu như luôn luôn. Điều này gây ra lỗi khi tạo các kho lưu trữ ZIP chứa tên tệp Shift-JIS. Trong Go 1.10, `Writer` giờ chỉ đặt bit UTF-8 khi cả tên và trường comment đều là UTF-8 hợp lệ và ít nhất một trong số đó không phải ASCII. Vì các mã hóa không phải ASCII rất hiếm khi trông giống UTF-8 hợp lệ, heuristic mới nên đúng gần như luôn luôn. Đặt trường `NonUTF8` mới của `FileHeader` thành true sẽ tắt hoàn toàn heuristic cho tệp đó.

`Writer` cũng giờ hỗ trợ đặt trường comment của bản ghi end-of-central-directory, bằng cách gọi phương thức [`SetComment`](/pkg/archive/zip/#Writer.SetComment) mới của `Writer`.

#### [bufio](/pkg/bufio/)

Các phương thức [`Reader.Size`](/pkg/bufio/#Reader.Size) và [`Writer.Size`](/pkg/bufio/#Writer.Size) mới báo cáo kích thước buffer bên dưới của `Reader` hoặc `Writer`.

#### [bytes](/pkg/bytes/)

Các hàm [`Fields`](/pkg/bytes/#Fields), [`FieldsFunc`](/pkg/bytes/#FieldsFunc), [`Split`](/pkg/bytes/#Split) và [`SplitAfter`](/pkg/bytes/#SplitAfter) luôn trả về các subslice của đầu vào của chúng. Go 1.10 thay đổi mỗi subslice được trả về để có dung lượng bằng độ dài của nó, do đó việc append vào một subslice không thể ghi đè lên dữ liệu liền kề trong đầu vào gốc.

#### [crypto/cipher](/pkg/crypto/cipher/)

[`NewOFB`](/pkg/crypto/cipher/#NewOFB) giờ panic nếu được đưa ra một vector khởi tạo có độ dài không đúng, giống như các constructor khác trong gói luôn làm. (Trước đây nó trả về một triển khai `Stream` nil.)

#### [crypto/tls](/pkg/crypto/tls/)

Server TLS giờ quảng cáo hỗ trợ cho chữ ký SHA-512 khi sử dụng TLS 1.2. Server đã hỗ trợ các chữ ký, nhưng một số client sẽ không chọn chúng trừ khi được quảng cáo tường minh.

#### [crypto/x509](/pkg/crypto/x509/)

[`Certificate.Verify`](/pkg/crypto/x509/#Certificate.Verify) giờ thực thi các ràng buộc tên cho tất cả các tên có trong chứng chỉ, không chỉ tên mà client đã hỏi. Các hạn chế sử dụng khóa mở rộng tương tự giờ cũng được kiểm tra cùng một lúc. Kết quả là, sau khi một chứng chỉ đã được xác thực, giờ nó có thể được tin tưởng hoàn toàn. Không còn cần thiết phải xác thực lại chứng chỉ cho mỗi tên bổ sung hoặc sử dụng khóa.

Các chứng chỉ đã phân tích cũng giờ báo cáo tên URI và các ràng buộc IP, email và URI, sử dụng các trường [`Certificate`](/pkg/crypto/x509/#Certificate) mới `URIs`, `PermittedIPRanges`, `ExcludedIPRanges`, `PermittedEmailAddresses`, `ExcludedEmailAddresses`, `PermittedURIDomains` và `ExcludedURIDomains`. Các chứng chỉ có giá trị không hợp lệ cho những trường đó giờ bị từ chối.

Các hàm [`MarshalPKCS1PublicKey`](/pkg/crypto/x509/#MarshalPKCS1PublicKey) và [`ParsePKCS1PublicKey`](/pkg/crypto/x509/#ParsePKCS1PublicKey) mới chuyển đổi khóa công khai RSA sang và từ dạng được mã hóa PKCS#1.

Hàm [`MarshalPKCS8PrivateKey`](/pkg/crypto/x509/#MarshalPKCS8PrivateKey) mới chuyển đổi khóa riêng tư sang dạng được mã hóa PKCS#8. ([`ParsePKCS8PrivateKey`](/pkg/crypto/x509/#ParsePKCS8PrivateKey) đã tồn tại từ Go 1.)

#### [crypto/x509/pkix](/pkg/crypto/x509/pkix/)

[`Name`](/pkg/crypto/x509/pkix/#Name) giờ triển khai phương thức [`String`](/pkg/crypto/x509/pkix/#Name.String) định dạng distinguished name X.509 theo định dạng RFC 2253 tiêu chuẩn.

#### [database/sql/driver](/pkg/database/sql/driver/)

Các driver hiện đang giữ buffer đích được cung cấp bởi [`driver.Rows.Next`](/pkg/database/sql/driver/#Rows.Next) nên đảm bảo chúng không còn ghi vào buffer được gán cho mảng đích bên ngoài lời gọi đó. Driver phải cẩn thận rằng các buffer bên dưới không bị sửa đổi khi đóng [`driver.Rows`](/pkg/database/sql/driver/#Rows).

Driver muốn xây dựng một [`sql.DB`](/pkg/database/sql/#DB) cho client của họ giờ có thể triển khai interface [`Connector`](/pkg/database/sql/driver/#Connector) và gọi hàm [`sql.OpenDB`](/pkg/database/sql/#OpenDB) mới, thay vì cần mã hóa tất cả cấu hình vào một chuỗi được truyền cho [`sql.Open`](/pkg/database/sql/#Open).

Driver muốn phân tích chuỗi cấu hình chỉ một lần mỗi `sql.DB` thay vì một lần mỗi [`sql.Conn`](/pkg/database/sql/#Conn), hoặc muốn truy cập context bên dưới của mỗi `sql.Conn`, có thể làm cho các triển khai [`Driver`](/pkg/database/sql/driver/#Driver) của họ cũng triển khai phương thức `OpenConnector` mới của [`DriverContext`](/pkg/database/sql/driver/#DriverContext).

Driver triển khai [`ExecerContext`](/pkg/database/sql/driver/#ExecerContext) không còn cần triển khai [`Execer`](/pkg/database/sql/driver/#Execer); tương tự, driver triển khai [`QueryerContext`](/pkg/database/sql/driver/#QueryerContext) không còn cần triển khai [`Queryer`](/pkg/database/sql/driver/#Queryer). Trước đây, ngay cả khi các interface dựa trên context được triển khai, chúng bị bỏ qua trừ khi các interface không dựa trên context cũng được triển khai.

Để cho phép driver cô lập tốt hơn các client khác nhau sử dụng kết nối driver đã lưu bộ nhớ đệm liên tiếp, nếu một [`Conn`](/pkg/database/sql/driver/#Conn) triển khai interface [`SessionResetter`](/pkg/database/sql/driver/#SessionResetter) mới, `database/sql` giờ sẽ gọi `ResetSession` trước khi tái sử dụng `Conn` cho một client mới.

#### [debug/elf](/pkg/debug/elf/)

Bản phát hành này thêm 348 hằng số relocation mới được chia giữa các kiểu relocation [`R_386`](/pkg/debug/elf/#R_386), [`R_AARCH64`](/pkg/debug/elf/#R_AARCH64), [`R_ARM`](/pkg/debug/elf/#R_ARM), [`R_PPC64`](/pkg/debug/elf/#R_PPC64) và [`R_X86_64`](/pkg/debug/elf/#R_X86_64).

#### [debug/macho](/pkg/debug/macho/)

Go 1.10 bổ sung hỗ trợ đọc các relocation từ các section Mach-O, sử dụng trường `Relocs` mới của struct [`Section`](/pkg/debug/macho#Section) và các kiểu [`Reloc`](/pkg/debug/macho/#Reloc), [`RelocTypeARM`](/pkg/debug/macho/#RelocTypeARM), [`RelocTypeARM64`](/pkg/debug/macho/#RelocTypeARM64), [`RelocTypeGeneric`](/pkg/debug/macho/#RelocTypeGeneric) và [`RelocTypeX86_64`](/pkg/debug/macho/#RelocTypeX86_64) cùng các hằng số liên quan mới.

Go 1.10 cũng thêm hỗ trợ cho lệnh load `LC_RPATH`, được đại diện bởi các kiểu [`RpathCmd`](/pkg/debug/macho/#RpathCmd) và [`Rpath`](/pkg/debug/macho/#Rpath), và các [hằng số được đặt tên mới](/pkg/debug/macho/#pkg-constants) cho các bit cờ khác nhau có trong các header.

#### [encoding/asn1](/pkg/encoding/asn1/)

[`Marshal`](/pkg/encoding/asn1/#Marshal) giờ mã hóa đúng các chuỗi chứa dấu hoa thị là kiểu UTF8String thay vì PrintableString, trừ khi chuỗi nằm trong một trường struct với tag buộc sử dụng PrintableString. `Marshal` cũng giờ tôn trọng các tag struct chứa chỉ thị `application`.

Hàm [`MarshalWithParams`](/pkg/encoding/asn1/#MarshalWithParams) mới marshal đối số của nó như thể các params bổ sung là tag trường struct liên kết của nó.

[`Unmarshal`](/pkg/encoding/asn1/#Unmarshal) giờ tôn trọng các tag trường struct sử dụng chỉ thị `explicit` và `tag`.

Cả `Marshal` và `Unmarshal` giờ hỗ trợ tag trường struct mới `numeric`, chỉ ra một ASN.1 NumericString.

#### [encoding/csv](/pkg/encoding/csv/)

[`Reader`](/pkg/encoding/csv/#Reader) giờ không cho phép sử dụng các cài đặt `Comma` và `Comment` vô nghĩa, chẳng hạn như NUL, carriage return, newline, rune không hợp lệ và ký tự thay thế Unicode, hoặc đặt `Comma` và `Comment` bằng nhau.

Trong trường hợp lỗi cú pháp trong bản ghi CSV trải dài nhiều dòng đầu vào, `Reader` giờ báo cáo dòng mà bản ghi bắt đầu trong trường `StartLine` mới của [`ParseError`](/pkg/encoding/csv/#ParseError).

#### [encoding/hex](/pkg/encoding/hex/)

Các hàm [`NewEncoder`](/pkg/encoding/hex/#NewEncoder) và [`NewDecoder`](/pkg/encoding/hex/#NewDecoder) mới cung cấp các chuyển đổi streaming sang và từ hexadecimal, tương tự như các hàm tương đương đã có trong [encoding/base32](/pkg/encoding/base32/) và [encoding/base64](/pkg/encoding/base64/).

Khi các hàm [`Decode`](/pkg/encoding/hex/#Decode) và [`DecodeString`](/pkg/encoding/hex/#DecodeString) gặp đầu vào không hợp lệ, giờ chúng trả về số byte đã chuyển đổi cùng với lỗi. Trước đây chúng luôn trả về số đếm 0 với bất kỳ lỗi nào.

#### [encoding/json](/pkg/encoding/json/)

[`Decoder`](/pkg/encoding/json/#Decoder) thêm phương thức mới [`DisallowUnknownFields`](/pkg/encoding/json/#Decoder.DisallowUnknownFields) khiến nó báo cáo các đầu vào có các trường JSON không xác định là lỗi giải mã. (Hành vi mặc định luôn là bỏ qua các trường không xác định.)

Kết quả từ [sửa lỗi reflect](#reflect), [`Unmarshal`](/pkg/encoding/json/#Unmarshal) không còn có thể giải mã vào các trường bên trong các con trỏ nhúng đến các kiểu struct không được xuất, vì nó không thể khởi tạo con trỏ nhúng không được xuất để trỏ đến bộ nhớ mới. `Unmarshal` giờ trả về lỗi trong trường hợp này.

#### [encoding/pem](/pkg/encoding/pem/)

[`Encode`](/pkg/encoding/pem/#Encode) và [`EncodeToMemory`](/pkg/encoding/pem/#EncodeToMemory) không còn tạo ra đầu ra một phần khi được trình bày với một block không thể mã hóa dưới dạng dữ liệu PEM.

#### [encoding/xml](/pkg/encoding/xml/)

Hàm [`NewTokenDecoder`](/pkg/encoding/xml/#NewTokenDecoder) mới giống như [`NewDecoder`](/pkg/encoding/xml/#NewDecoder) nhưng tạo ra một decoder đọc từ một [`TokenReader`](/pkg/encoding/xml/#TokenReader) thay vì một byte stream định dạng XML. Điều này nhằm cho phép xây dựng các bộ chuyển đổi stream XML trong các thư viện client.

#### [flag](/pkg/flag/)

Hàm [`Usage`](/pkg/flag/#Usage) mặc định giờ in dòng đầu tiên đầu ra của nó ra `CommandLine.Output()` thay vì giả định `os.Stderr`, để thông báo sử dụng được chuyển hướng đúng cách cho các client sử dụng `CommandLine.SetOutput`.

[`PrintDefaults`](/pkg/flag/#PrintDefaults) giờ thêm thụt lề phù hợp sau các ký tự xuống dòng trong các chuỗi sử dụng cờ, để các chuỗi sử dụng nhiều dòng hiển thị đẹp.

[`FlagSet`](/pkg/flag/#FlagSet) thêm các phương thức mới [`ErrorHandling`](/pkg/flag/#FlagSet.ErrorHandling), [`Name`](/pkg/flag/#FlagSet.Name) và [`Output`](/pkg/flag/#FlagSet.Output), để lấy các cài đặt được truyền cho [`NewFlagSet`](/pkg/flag/#NewFlagSet) và [`FlagSet.SetOutput`](/pkg/flag/#FlagSet.SetOutput).

#### [go/doc](/pkg/go/doc/)

Để hỗ trợ [thay đổi doc](#doc) được mô tả ở trên, các hàm trả về slice của `T`, `*T`, `**T`, v.v. giờ được báo cáo trong danh sách `Funcs` của [`Type`](/pkg/go/doc/#Type) của `T`, thay vì trong danh sách `Funcs` của [`Package`](/pkg/go/doc/#Package).

#### [go/importer](/pkg/go/importer/)

Hàm [`For`](/pkg/go/importer/#For) giờ chấp nhận đối số lookup không nil.

#### [go/printer](/pkg/go/printer/)

Các thay đổi về định dạng mặc định của mã nguồn Go được thảo luận trong [phần gofmt](#gofmt) ở trên được triển khai trong gói [go/printer](/pkg/go/printer/) và cũng ảnh hưởng đến đầu ra của gói cấp cao hơn [go/format](/pkg/go/format/).

#### [hash](/pkg/hash/)

Các triển khai của interface [`Hash`](/pkg/hash/#Hash) giờ được khuyến khích triển khai [`encoding.BinaryMarshaler`](/pkg/encoding/#BinaryMarshaler) và [`encoding.BinaryUnmarshaler`](/pkg/encoding/#BinaryUnmarshaler) để cho phép lưu và tạo lại trạng thái nội bộ của chúng, và tất cả các triển khai trong thư viện chuẩn ([hash/crc32](/pkg/hash/crc32/), [crypto/sha256](/pkg/crypto/sha256/), v.v.) giờ triển khai các interface đó.

#### [html/template](/pkg/html/template/)

Kiểu nội dung [`Srcset`](/pkg/html/template#Srcset) mới cho phép xử lý đúng các giá trị trong thuộc tính [`srcset`](https://w3c.github.io/html/semantics-embedded-content.html#element-attrdef-img-srcset) của thẻ `img`.

#### [math/big](/pkg/math/big/)

[`Int`](/pkg/math/big/#Int) giờ hỗ trợ chuyển đổi sang và từ các cơ số 2 đến 62 trong các phương thức [`SetString`](/pkg/math/big/#Int.SetString) và [`Text`](/pkg/math/big/#Text) của nó. (Trước đây nó chỉ cho phép các cơ số 2 đến 36.) Giá trị của hằng số `MaxBase` đã được cập nhật.

[`Int`](/pkg/math/big/#Int) thêm phương thức [`CmpAbs`](/pkg/math/big/#CmpAbs) mới giống như [`Cmp`](/pkg/math/big/#Cmp) nhưng chỉ so sánh các giá trị tuyệt đối (không phải dấu) của các đối số của nó.

[`Float`](/pkg/math/big/#Float) thêm phương thức [`Sqrt`](/pkg/math/big/#Float.Sqrt) mới để tính căn bậc hai.

#### [math/cmplx](/pkg/math/cmplx/)

Các đường phân nhánh và các trường hợp biên khác trong [`Asin`](/pkg/math/cmplx/#Asin), [`Asinh`](/pkg/math/cmplx/#Asinh), [`Atan`](/pkg/math/cmplx/#Atan) và [`Sqrt`](/pkg/math/cmplx/#Sqrt) đã được sửa để khớp với các định nghĩa được sử dụng trong tiêu chuẩn C99.

#### [math/rand](/pkg/math/rand/)

Hàm [`Shuffle`](/pkg/math/rand/#Shuffle) mới và phương thức [`Rand.Shuffle`](/pkg/math/rand/#Rand.Shuffle) tương ứng xáo trộn một chuỗi đầu vào.

#### [math](/pkg/math/)

Các hàm [`Round`](/pkg/math/#Round) và [`RoundToEven`](/pkg/math/#RoundToEven) mới làm tròn các đối số của chúng đến số nguyên dấu phẩy động gần nhất; `Round` làm tròn một nửa số nguyên đến số nguyên lớn hơn của nó (xa số không) trong khi `RoundToEven` làm tròn một nửa số nguyên đến số nguyên chẵn gần nhất của nó.

Các hàm [`Erfinv`](/pkg/math/#Erfinv) và [`Erfcinv`](/pkg/math/#Erfcinv) mới tính hàm lỗi nghịch đảo và hàm lỗi bổ sung nghịch đảo.

#### [mime/multipart](/pkg/mime/multipart/)

[`Reader`](/pkg/mime/multipart/#Reader) giờ chấp nhận các phần có thuộc tính tên tệp rỗng.

#### [mime](/pkg/mime/)

[`ParseMediaType`](/pkg/mime/#ParseMediaType) giờ loại bỏ các giá trị thuộc tính không hợp lệ; trước đây nó trả về các giá trị đó dưới dạng chuỗi rỗng.

#### [net](/pkg/net/)

Các triển khai [`Conn`](/pkg/net/#Conn) và [`Listener`](/pkg/net/#Conn) trong gói này giờ đảm bảo rằng khi `Close` trả về, file descriptor bên dưới đã được đóng. (Trong các bản phát hành trước, nếu `Close` dừng I/O đang chờ trong các goroutine khác, việc đóng file descriptor có thể xảy ra trong một trong các goroutine đó ngay sau khi `Close` trả về.)

[`TCPListener`](/pkg/net/#TCPListener) và [`UnixListener`](/pkg/net/#UnixListener) giờ triển khai [`syscall.Conn`](/pkg/syscall/#Conn), để cho phép đặt các tùy chọn trên file descriptor bên dưới bằng [`syscall.RawConn.Control`](/pkg/syscall/#RawConn).

Các triển khai `Conn` được trả về bởi [`Pipe`](/pkg/net/#Pipe) giờ hỗ trợ đặt deadline đọc và ghi.

Các phương thức [`IPConn.ReadMsgIP`](/pkg/net/#IPConn.ReadMsgIP), [`IPConn.WriteMsgIP`](/pkg/net/#IPConn.WriteMsgIP), [`UDPConn.ReadMsgUDP`](/pkg/net/#UDPConn.ReadMsgUDP) và [`UDPConn.WriteMsgUDP`](/pkg/net/#UDPConn.WriteMsgUDP) giờ được triển khai trên Windows.

#### [net/http](/pkg/net/http/)

Ở phía client, một proxy HTTP (thường được cấu hình nhất bởi [`ProxyFromEnvironment`](/pkg/net/http/#ProxyFromEnvironment)) giờ có thể được chỉ định dưới dạng URL `https://`, nghĩa là client kết nối với proxy qua HTTPS trước khi phát ra yêu cầu HTTP chuẩn, được proxy. (Trước đây, các URL proxy HTTP phải bắt đầu bằng `http://` hoặc `socks5://`.)

Ở phía server, [`FileServer`](/pkg/net/http/#FileServer) và tương đương một tệp của nó [`ServeFile`](/pkg/net/http/#ServeFile) giờ áp dụng các kiểm tra `If-Range` cho các yêu cầu `HEAD`. `FileServer` cũng giờ báo cáo các lỗi đọc thư mục cho `ErrorLog` của [`Server`](/pkg/net/http/#Server). Các handler phục vụ nội dung cũng giờ bỏ qua header `Content-Type` khi phục vụ nội dung có độ dài không.

Phương thức `WriteHeader` của [`ResponseWriter`](/pkg/net/http/#ResponseWriter) giờ panic nếu được truyền mã trạng thái không hợp lệ (không phải 3 chữ số).

<!-- CL 46631 -->
`Server` sẽ không còn thêm Content-Type ngầm khi `Handler` không ghi bất kỳ đầu ra nào.

[`Redirect`](/pkg/net/http/#Redirect) giờ đặt header `Content-Type` trước khi ghi phản hồi HTTP của nó.

#### [net/mail](/pkg/net/mail/)

[`ParseAddress`](/pkg/net/mail/#ParseAddress) và [`ParseAddressList`](/pkg/net/mail/#ParseAddressList) giờ hỗ trợ nhiều định dạng địa chỉ lỗi thời.

#### [net/smtp](/pkg/net/smtp/)

[`Client`](/pkg/net/smtp/#Client) thêm phương thức [`Noop`](/pkg/net/smtp/#Client.Noop) mới, để kiểm tra xem server có vẫn đang phản hồi không. Nó cũng giờ bảo vệ chống lại SMTP injection có thể xảy ra trong các đầu vào cho các phương thức [`Hello`](/pkg/net/smtp/#Client.Hello) và [`Verify`](/pkg/net/smtp/#Client.Verify).

#### [net/textproto](/pkg/net/textproto/)

[`ReadMIMEHeader`](/pkg/net/textproto/#ReadMIMEHeader) giờ từ chối bất kỳ header nào bắt đầu bằng dòng header tiếp tục (thụt lề). Trước đây, một header với dòng đầu tiên thụt lề được xử lý như thể dòng đầu tiên không bị thụt lề.

#### [net/url](/pkg/net/url/)

[`ResolveReference`](/pkg/net/url/#ResolveReference) giờ bảo tồn nhiều dấu gạch chéo đầu trong URL đích. Trước đây nó viết lại nhiều dấu gạch chéo đầu thành một dấu gạch chéo duy nhất, điều này khiến [`http.Client`](/pkg/net/http/#Client) theo một số chuyển hướng không đúng cách.

Ví dụ, đầu ra của mã này đã thay đổi:

	base, _ := url.Parse("http://host//path//to/page1")
	target, _ := url.Parse("page2")
	fmt.Println(base.ResolveReference(target))

Lưu ý các dấu gạch chéo kép xung quanh `path`. Trong Go 1.9 và trước đó, URL đã được giải quyết là `http://host/path//to/page2`: dấu gạch chéo kép trước `path` đã bị viết lại không đúng thành một dấu gạch chéo duy nhất, trong khi dấu gạch chéo kép sau `path` được bảo tồn đúng. Go 1.10 bảo tồn cả hai dấu gạch chéo kép, giải quyết thành `http://host//path//to/page2` theo yêu cầu của [RFC 3986](https://tools.ietf.org/html/rfc3986#section-5.2).

Thay đổi này có thể phá vỡ các chương trình lỗi hiện có vô tình xây dựng URL cơ sở với dấu gạch chéo kép đầu trong đường dẫn và vô tình phụ thuộc vào `ResolveReference` để sửa lỗi đó.

Các phương thức của [`UserInfo`](/pkg/net/url/#UserInfo) giờ xử lý receiver nil tương đương với con trỏ đến `UserInfo` không. Trước đây, chúng panic.

#### [os](/pkg/os/)

[`File`](/pkg/os/#File) thêm các phương thức mới [`SetDeadline`](/pkg/os/#File.SetDeadline), [`SetReadDeadline`](/pkg/os/#File.SetReadDeadline) và [`SetWriteDeadline`](/pkg/os/#File.SetWriteDeadline) cho phép đặt deadline I/O khi file descriptor bên dưới hỗ trợ các thao tác I/O không chặn. Định nghĩa của các phương thức này khớp với các phương thức trong [`net.Conn`](/pkg/net/#Conn). Nếu một phương thức I/O thất bại do bỏ lỡ deadline, nó sẽ trả về lỗi timeout; hàm [`IsTimeout`](/pkg/os/#IsTimeout) mới báo cáo xem một lỗi có đại diện cho timeout hay không.

Cũng khớp với `net.Conn`, phương thức [`Close`](/pkg/os/#File.Close) của `File` giờ đảm bảo rằng khi `Close` trả về, file descriptor bên dưới đã được đóng.

Trên các hệ thống BSD, macOS và Solaris, [`Chtimes`](/pkg/os/#Chtimes) giờ hỗ trợ đặt thời gian tệp với độ chính xác nanosecond (giả sử hệ thống tệp bên dưới có thể đại diện chúng).

#### [reflect](/pkg/reflect/)

Hàm [`Copy`](/pkg/reflect/#Copy) giờ cho phép sao chép từ một chuỗi vào một mảng byte hoặc slice byte, để khớp với [hàm copy tích hợp](/pkg/builtin/#copy).

Trong các struct, các con trỏ nhúng đến các kiểu struct không được xuất trước đây bị báo cáo không đúng với `PkgPath` rỗng trong [`StructField`](/pkg/reflect/#StructField) tương ứng, với kết quả là đối với những trường đó, [`Value.CanSet`](/pkg/reflect/#Value.CanSet) trả về true không đúng và [`Value.Set`](/pkg/reflect/#Value.Set) thành công không đúng. Siêu dữ liệu bên dưới đã được sửa; đối với những trường đó, `CanSet` giờ trả về false đúng và `Set` giờ panic đúng. Điều này có thể ảnh hưởng đến các bộ unmarshal dựa trên reflection trước đây có thể unmarshal vào các trường như vậy nhưng giờ không thể. Ví dụ, xem [ghi chú `encoding/json`](#encoding/json).

#### [runtime/pprof](/pkg/runtime/pprof/)

Như [đã lưu ý ở trên](#pprof), các profile blocking và mutex giờ bao gồm thông tin symbol để chúng có thể được xem mà không cần binary đã tạo ra chúng.

#### [strconv](/pkg/strconv/)

[`ParseUint`](/pkg/strconv/#ParseUint) giờ trả về số nguyên có độ lớn tối đa của kích thước phù hợp với bất kỳ lỗi `ErrRange` nào, như đã được ghi trong tài liệu. Trước đây nó trả về 0 với các lỗi `ErrRange`.

#### [strings](/pkg/strings/)

Kiểu [`Builder`](/pkg/strings/#Builder) mới là sự thay thế cho [`bytes.Buffer`](/pkg/bytes/#Buffer) cho trường hợp sử dụng tích lũy văn bản vào kết quả `string`. API của `Builder` là một tập con bị hạn chế của `bytes.Buffer` cho phép nó tránh một cách an toàn việc tạo bản sao trùng lặp của dữ liệu trong phương thức [`String`](/pkg/strings/#Builder.String).

#### [syscall](/pkg/syscall/)

Trên Windows, trường [`SysProcAttr`](/pkg/syscall/#SysProcAttr) mới `Token` có kiểu [`Token`](/pkg/syscall/#Token) cho phép tạo một tiến trình chạy như người dùng khác trong [`StartProcess`](/pkg/syscall/#StartProcess) (và do đó cũng trong [`os.StartProcess`](/pkg/os/#StartProcess) và [`exec.Cmd.Start`](/pkg/os/exec/#Cmd.Start)). Hàm [`CreateProcessAsUser`](/pkg/syscall/#CreateProcessAsUser) mới cung cấp quyền truy cập vào system call bên dưới.

Trên các hệ thống BSD, macOS và Solaris, [`UtimesNano`](/pkg/syscall/#UtimesNano) giờ được triển khai.

#### [time](/pkg/time/)

[`LoadLocation`](/pkg/time/#LoadLocation) giờ sử dụng thư mục hoặc tệp zip không nén được đặt tên bởi biến môi trường `$ZONEINFO` trước khi tìm kiếm trong danh sách vị trí cài đặt đã biết mặc định dành riêng cho hệ thống hoặc trong `$GOROOT/lib/time/zoneinfo.zip`.

Hàm [`LoadLocationFromTZData`](/pkg/time/#LoadLocationFromTZData) mới cho phép chuyển đổi dữ liệu tệp múi giờ IANA sang [`Location`](/pkg/time/#Location).

#### [unicode](/pkg/unicode/)

Gói [`unicode`](/pkg/unicode/) và hỗ trợ liên quan trong toàn hệ thống đã được nâng cấp từ Unicode 9.0 lên [Unicode 10.0](https://www.unicode.org/versions/Unicode10.0.0/), thêm 8.518 ký tự mới, bao gồm bốn script mới, một thuộc tính mới, ký hiệu tiền tệ Bitcoin và 56 emoji mới.
