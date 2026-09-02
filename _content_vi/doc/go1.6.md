---
title: Ghi chú phát hành Go 1.6
---

<!--
Edit .,s;^PKG:([a-z][A-Za-z0-9_/]+);<a href="/pkg/\1/"><code>\1</code></a>;g
Edit .,s;^([a-z][A-Za-z0-9_/]+)\.([A-Z][A-Za-z0-9_]+\.)?([A-Z][A-Za-z0-9_]+)([ .',]|$);<a href="/pkg/\1/#\2\3"><code>\3</code></a>\4;g
-->

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu về Go 1.6 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.6, ra đời sáu tháng sau phiên bản 1.5.
Phần lớn các thay đổi nằm ở phần triển khai ngôn ngữ, runtime và các thư viện.
Không có thay đổi nào đối với đặc tả ngôn ngữ.
Như thường lệ, bản phát hành duy trì [cam kết tương thích](/doc/go1compat.html) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

Bản phát hành bổ sung các port mới cho [Linux trên MIPS 64-bit và Android trên x86 32-bit](#ports);
định nghĩa và thực thi [các quy tắc chia sẻ con trỏ Go với C](#cgo);
hỗ trợ [HTTP/2 tự động, trong suốt](#http2);
và một cơ chế mới để [tái sử dụng template](#template).

## Thay đổi về ngôn ngữ {#language}

Không có thay đổi nào về ngôn ngữ trong bản phát hành này.

## Các port {#ports}

Go 1.6 bổ sung các port thử nghiệm cho
Linux trên MIPS 64-bit (`linux/mips64` và `linux/mips64le`).
Các port này hỗ trợ `cgo` nhưng chỉ với liên kết nội bộ.

Go 1.6 cũng bổ sung một port thử nghiệm cho Android trên x86 32-bit (`android/386`).

Trên FreeBSD, Go 1.6 mặc định sử dụng `clang` thay vì `gcc` làm trình biên dịch C ngoài.

Trên Linux trên PowerPC 64-bit little-endian (`linux/ppc64le`),
Go 1.6 hiện hỗ trợ `cgo` với liên kết ngoài và
đã gần như đầy đủ tính năng.

Trên NaCl, Go 1.5 yêu cầu phiên bản SDK pepper-41.
Go 1.6 bổ sung hỗ trợ cho các phiên bản SDK mới hơn.

Trên các hệ thống x86 32-bit sử dụng chế độ biên dịch `-dynlink` hoặc `-shared`,
thanh ghi CX hiện bị ghi đè bởi một số tham chiếu bộ nhớ và nên
tránh sử dụng trong mã assembly viết tay.
Xem [tài liệu assembly](/doc/asm#x86) để biết thêm chi tiết.

## Công cụ {#tools}

### Cgo {#cgo}

Có một thay đổi lớn và một thay đổi nhỏ đối với [`cgo`](/cmd/cgo/).

Thay đổi lớn là định nghĩa các quy tắc chia sẻ con trỏ Go với mã C,
để đảm bảo rằng mã C đó có thể cùng tồn tại với bộ gom rác của Go.
Tóm lại, Go và C có thể chia sẻ bộ nhớ được cấp phát bởi Go
khi một con trỏ đến bộ nhớ đó được truyền cho C trong một lời gọi `cgo`,
với điều kiện bộ nhớ đó không chứa con trỏ đến bộ nhớ do Go cấp phát,
và C không giữ lại con trỏ sau khi lời gọi trả về.
Các quy tắc này được runtime kiểm tra trong quá trình thực thi chương trình:
nếu runtime phát hiện vi phạm, nó sẽ in thông báo chẩn đoán và làm dừng chương trình.
Việc kiểm tra có thể bị tắt bằng cách đặt biến môi trường
`GODEBUG=cgocheck=0`, nhưng lưu ý rằng phần lớn
mã được các kiểm tra xác định là không tương thích một cách tinh tế với bộ gom rác
theo cách này hay cách khác.
Tắt các kiểm tra thường chỉ dẫn đến các chế độ lỗi khó hiểu hơn.
Nên ưu tiên sửa mã có vấn đề thay vì tắt các kiểm tra.
Xem [tài liệu `cgo`](/cmd/cgo/#hdr-Passing_pointers) để biết thêm chi tiết.

Thay đổi nhỏ là
việc bổ sung các kiểu `C.complexfloat` và `C.complexdouble` tường minh,
tách biệt với `complex64` và `complex128` của Go.
Giống như các kiểu số khác, kiểu phức hợp của C và kiểu phức hợp của Go không
còn có thể thay thế lẫn nhau.

### Bộ công cụ biên dịch {#compiler}

Bộ công cụ biên dịch hầu như không thay đổi.
Bên trong, thay đổi quan trọng nhất là bộ phân tích cú pháp hiện được viết tay
thay vì được tạo ra từ [yacc](/cmd/yacc/).

Trình biên dịch, trình liên kết và lệnh `go` có cờ mới `-msan`,
tương tự như `-race` và chỉ khả dụng trên linux/amd64,
cho phép tương tác với [Clang MemorySanitizer](https://clang.llvm.org/docs/MemorySanitizer.html).
Tương tác như vậy chủ yếu hữu ích để kiểm tra một chương trình chứa mã C hoặc C++ đáng ngờ.

Trình liên kết có tùy chọn mới `-libgcc` để đặt vị trí dự kiến
của thư viện hỗ trợ trình biên dịch C khi liên kết mã [`cgo`](/cmd/cgo/).
Tùy chọn này chỉ được tham khảo khi sử dụng `-linkmode=internal`,
và có thể đặt thành `none` để tắt việc sử dụng thư viện hỗ trợ.

Việc triển khai [các chế độ build bắt đầu từ Go 1.5](/doc/go1.5#link) đã được mở rộng cho nhiều hệ thống hơn.
Bản phát hành này bổ sung hỗ trợ cho chế độ `c-shared` trên `android/386`, `android/amd64`,
`android/arm64`, `linux/386` và `linux/arm64`;
cho chế độ `shared` trên `linux/386`, `linux/arm`, `linux/amd64` và `linux/ppc64le`;
và cho chế độ `pie` mới (tạo tệp thực thi độc lập vị trí) trên
`android/386`, `android/amd64`, `android/arm`, `android/arm64`, `linux/386`,
`linux/amd64`, `linux/arm`, `linux/arm64` và `linux/ppc64le`.
Xem [tài liệu thiết kế](/s/execmodes) để biết chi tiết.

Nhắc lại, cờ `-X` của trình liên kết đã thay đổi trong Go 1.5.
Trong Go 1.4 và trước đó, nó nhận hai đối số, như trong

	-X importpath.name value

Go 1.5 đã thêm cú pháp thay thế sử dụng một đối số duy nhất
là cặp `name=value`:

	-X importpath.name=value

Trong Go 1.5, cú pháp cũ vẫn được chấp nhận, sau khi in cảnh báo
gợi ý sử dụng cú pháp mới.
Go 1.6 tiếp tục chấp nhận cú pháp cũ và in cảnh báo.
Go 1.7 sẽ bỏ hỗ trợ cho cú pháp cũ.

### Gccgo {#gccgo}

Lịch trình phát hành của các dự án GCC và Go không trùng nhau.
GCC phiên bản 5 chứa phiên bản Go 1.4 của gccgo.
Bản phát hành tiếp theo, GCC 6, sẽ có phiên bản Go 1.6.1 của gccgo.

### Lệnh go {#go_command}

Hoạt động cơ bản của lệnh [`go`](/cmd/go) không thay đổi, nhưng có một số thay đổi đáng chú ý.

Go 1.5 đã giới thiệu hỗ trợ thử nghiệm cho vendoring,
được kích hoạt bằng cách đặt biến môi trường `GO15VENDOREXPERIMENT` thành `1`.
Go 1.6 giữ lại hỗ trợ vendoring, không còn được coi là thử nghiệm,
và bật theo mặc định.
Có thể tắt tường minh bằng cách đặt
biến môi trường `GO15VENDOREXPERIMENT` thành `0`.
Go 1.7 sẽ bỏ hỗ trợ cho biến môi trường này.

Vấn đề có khả năng xảy ra nhất do bật vendoring theo mặc định xuất hiện
trong các cây mã nguồn chứa thư mục có tên `vendor` hiện có mà
không mong muốn được diễn giải theo ngữ nghĩa vendoring mới.
Trong trường hợp này, cách sửa đơn giản nhất là đổi tên thư mục thành bất kỳ tên nào khác
ngoài `vendor` và cập nhật bất kỳ đường dẫn import bị ảnh hưởng nào.

Để biết chi tiết về vendoring,
xem tài liệu cho [lệnh `go`](/cmd/go/#hdr-Vendor_Directories)
và [tài liệu thiết kế](/s/go15vendor).

Có một cờ build mới, `-msan`,
biên dịch Go với hỗ trợ cho LLVM memory sanitizer.
Cờ này chủ yếu dành cho việc liên kết với mã C hoặc C++
đang được kiểm tra bằng memory sanitizer.

### Lệnh go doc {#doc_command}

Go 1.5 đã giới thiệu
lệnh [`go doc`](/cmd/go/#hdr-Show_documentation_for_package_or_symbol),
cho phép tham chiếu đến các package chỉ bằng tên package, như trong
`go` `doc` `http`.
Khi có sự mơ hồ, hành vi của Go 1.5 là sử dụng package
có đường dẫn import theo thứ tự từ điển sớm nhất.
Trong Go 1.6, sự mơ hồ được giải quyết bằng cách ưu tiên các đường dẫn import có
ít thành phần hơn, phá vỡ sự ràng buộc bằng so sánh từ điển.
Một tác động quan trọng của thay đổi này là các bản sao gốc của package
hiện được ưu tiên hơn các bản sao được vendor.
Các tìm kiếm thành công cũng có xu hướng chạy nhanh hơn.

### Lệnh go vet {#vet_command}

Lệnh [`go vet`](/cmd/vet) hiện chẩn đoán
việc truyền giá trị hàm hoặc phương thức làm đối số cho `Printf`,
chẳng hạn như khi truyền `f` trong khi `f()` mới là ý định.

## Hiệu suất {#performance}

Như thường lệ, các thay đổi rất chung chung và đa dạng nên rất khó đưa ra tuyên bố chính xác
về hiệu suất.
Một số chương trình có thể chạy nhanh hơn, một số chậm hơn.
Trung bình, các chương trình trong bộ benchmark Go 1 chạy nhanh hơn vài phần trăm trong Go 1.6
so với Go 1.5.
Thời gian tạm dừng của bộ gom rác thậm chí còn thấp hơn so với Go 1.5,
đặc biệt đối với các chương trình sử dụng
lượng bộ nhớ lớn.

Đã có những tối ưu hóa đáng kể mang lại cải thiện hơn 10% cho
các triển khai của các package
[`compress/bzip2`](/pkg/compress/bzip2/),
[`compress/gzip`](/pkg/compress/gzip/),
[`crypto/aes`](/pkg/crypto/aes/),
[`crypto/elliptic`](/pkg/crypto/elliptic/),
[`crypto/ecdsa`](/pkg/crypto/ecdsa/) và
[`sort`](/pkg/sort/).

## Thư viện chuẩn {#library}

### HTTP/2 {#http2}

Go 1.6 bổ sung hỗ trợ trong suốt trong package
[`net/http`](/pkg/net/http/)
cho [giao thức HTTP/2 mới](https://http2.github.io/).
Các client và server Go sẽ tự động sử dụng HTTP/2 khi phù hợp khi dùng HTTPS.
Không có API được xuất khẩu nào dành riêng cho chi tiết xử lý giao thức HTTP/2,
cũng như không có API được xuất khẩu nào dành riêng cho HTTP/1.1.

Các chương trình phải tắt HTTP/2 có thể làm như vậy bằng cách đặt
[`Transport.TLSNextProto`](/pkg/net/http/#Transport) (cho client)
hoặc
[`Server.TLSNextProto`](/pkg/net/http/#Server) (cho server)
thành một map không nil, rỗng.

Các chương trình cần điều chỉnh chi tiết dành riêng cho giao thức HTTP/2 có thể import và sử dụng
[`golang.org/x/net/http2`](https://golang.org/x/net/http2),
đặc biệt là các hàm
[ConfigureServer](https://godoc.org/golang.org/x/net/http2/#ConfigureServer)
và
[ConfigureTransport](https://godoc.org/golang.org/x/net/http2/#ConfigureTransport).

### Runtime {#runtime}

Runtime đã bổ sung tính năng phát hiện nhẹ, theo khả năng tốt nhất việc sử dụng đồng thời sai của map.
Như thường lệ, nếu một goroutine đang ghi vào một map, không goroutine nào khác nên
đọc hoặc ghi map đó đồng thời.
Nếu runtime phát hiện điều kiện này, nó sẽ in chẩn đoán và làm dừng chương trình.
Cách tốt nhất để tìm hiểu thêm về vấn đề là chạy chương trình
dưới
[bộ phát hiện race](/blog/race-detector),
sẽ xác định race đáng tin cậy hơn
và cung cấp thêm chi tiết.

Đối với các panic kết thúc chương trình, runtime hiện mặc định
chỉ in stack của goroutine đang chạy,
không in tất cả goroutine hiện có.
Thường chỉ goroutine hiện tại mới liên quan đến panic,
vì vậy việc bỏ qua các goroutine khác sẽ giảm đáng kể đầu ra không liên quan
trong thông báo crash.
Để xem stack của tất cả goroutine trong thông báo crash, đặt biến môi trường
`GOTRACEBACK` thành `all`
hoặc gọi
[`debug.SetTraceback`](/pkg/runtime/debug/#SetTraceback)
trước khi crash, và chạy lại chương trình.
Xem [tài liệu runtime](/pkg/runtime/#hdr-Environment_Variables) để biết chi tiết.

_Cập nhật_:
Các panic không bắt được nhằm mục đích dump trạng thái của toàn bộ chương trình,
chẳng hạn như khi phát hiện timeout hoặc khi xử lý tường minh một tín hiệu nhận được,
bây giờ nên gọi `debug.SetTraceback("all")` trước khi panic.
Tìm kiếm các lần sử dụng
[`signal.Notify`](/pkg/os/signal/#Notify) có thể giúp xác định mã đó.

Trên Windows, các chương trình Go trong Go 1.5 và trước đó đã buộc
độ phân giải bộ hẹn giờ Windows toàn cục xuống 1ms khi khởi động
bằng cách gọi `timeBeginPeriod(1)`.
Go không còn cần điều này để có hiệu suất bộ lập lịch tốt,
và việc thay đổi độ phân giải bộ hẹn giờ toàn cục đã gây ra vấn đề trên một số hệ thống,
vì vậy lời gọi đã bị loại bỏ.

Khi sử dụng `-buildmode=c-archive` hoặc
`-buildmode=c-shared` để build một archive hoặc một thư viện chia sẻ,
việc xử lý tín hiệu đã thay đổi.
Trong Go 1.5, archive hoặc thư viện chia sẻ sẽ cài đặt một trình xử lý tín hiệu
cho hầu hết các tín hiệu.
Trong Go 1.6, nó sẽ chỉ cài đặt một trình xử lý tín hiệu cho các
tín hiệu đồng bộ cần thiết để xử lý panic thời gian chạy trong mã Go:
SIGBUS, SIGFPE, SIGSEGV.
Xem package [os/signal](/pkg/os/signal) để biết thêm
chi tiết.

### Reflect {#reflect}

Package
[`reflect`](/pkg/reflect/)
đã [giải quyết một sự không tương thích lâu dài](/issue/12367)
giữa các bộ công cụ gc và gccgo
liên quan đến các kiểu struct lồng nhau không được xuất khẩu chứa các trường được xuất khẩu.
Mã duyệt qua các cấu trúc dữ liệu bằng reflection, đặc biệt để triển khai
serialization theo tinh thần của các package
[`encoding/json`](/pkg/encoding/json/) và
[`encoding/xml`](/pkg/encoding/xml/),
có thể cần được cập nhật.

Vấn đề phát sinh khi sử dụng reflection để duyệt qua
một trường kiểu struct lồng nhau không được xuất khẩu
vào một trường được xuất khẩu của struct đó.
Trong trường hợp này, `reflect` đã báo cáo không chính xác
trường lồng nhau là được xuất khẩu, bằng cách trả về `Field.PkgPath` rỗng.
Bây giờ nó báo cáo chính xác rằng trường không được xuất khẩu
nhưng bỏ qua điều đó khi đánh giá quyền truy cập vào các trường được xuất khẩu
chứa trong struct.

_Cập nhật_:
Thông thường, mã trước đây duyệt qua các struct và sử dụng

	f.PkgPath != ""

để loại trừ các trường không thể truy cập

bây giờ nên dùng

	f.PkgPath != "" && !f.Anonymous

Ví dụ, xem các thay đổi đối với việc triển khai của
[`encoding/json`](https://go-review.googlesource.com/#/c/14011/2/src/encoding/json/encode.go) và
[`encoding/xml`](https://go-review.googlesource.com/#/c/14012/2/src/encoding/xml/typeinfo.go).

### Sắp xếp {#sort}

Trong package
[`sort`](/pkg/sort/),
việc triển khai
[`Sort`](/pkg/sort/#Sort)
đã được viết lại để thực hiện ít hơn khoảng 10% lần gọi đến các phương thức
`Less` và `Swap`
của [`Interface`](/pkg/sort/#Interface), với tổng thời gian tiết kiệm tương ứng.
Thuật toán mới chọn một thứ tự khác so với trước đây
đối với các giá trị bằng nhau (những cặp mà `Less(i,` `j)` và `Less(j,` `i)` đều là false).

_Cập nhật_:
Định nghĩa của `Sort` không đảm bảo về thứ tự cuối cùng của các giá trị bằng nhau,
nhưng hành vi mới vẫn có thể làm hỏng các chương trình mong đợi một thứ tự cụ thể.
Các chương trình như vậy nên hoặc tinh chỉnh các triển khai `Less` của họ
để báo cáo thứ tự mong muốn
hoặc chuyển sang
[`Stable`](/pkg/sort/#Stable),
giữ nguyên thứ tự đầu vào ban đầu
của các giá trị bằng nhau.

### Template {#template}

Trong package
[text/template](/pkg/text/template/),
có hai tính năng mới đáng kể giúp viết template dễ dàng hơn.

Thứ nhất, hiện có thể [cắt bỏ khoảng trắng xung quanh các hành động template](/pkg/text/template/#hdr-Text_and_spaces),
điều này giúp định nghĩa template dễ đọc hơn.
Dấu trừ ở đầu một hành động có nghĩa là cắt bỏ khoảng trắng trước hành động,
và dấu trừ ở cuối một hành động có nghĩa là cắt bỏ khoảng trắng sau hành động.
Ví dụ, template

	{{23 -}}
	   <
	{{- 45}}

định dạng thành `23<45`.

Thứ hai, [hành động `{{block}}` mới](/pkg/text/template/#hdr-Actions),
kết hợp với việc cho phép định nghĩa lại các template có tên,
cung cấp một cách đơn giản để định nghĩa các phần của template mà
có thể được thay thế trong các lần khởi tạo khác nhau.
Có [một ví dụ](/pkg/text/template/#example_Template_block)
trong package `text/template` minh họa tính năng mới này.

### Thay đổi nhỏ đối với thư viện {#minor_library_changes}

  - Việc triển khai của package [`archive/tar`](/pkg/archive/tar/)
    sửa nhiều lỗi trong các trường hợp hiếm của định dạng tệp.
    Một thay đổi rõ ràng là phương thức
    [`Read`](/pkg/archive/tar/#Reader.Read) của kiểu
    [`Reader`](/pkg/archive/tar/#Reader)
    giờ đây trình bày nội dung của các loại tệp đặc biệt là rỗng,
    trả về `io.EOF` ngay lập tức.
  - Trong package [`archive/zip`](/pkg/archive/zip/), kiểu
    [`Reader`](/pkg/archive/zip/#Reader) hiện có phương thức
    [`RegisterDecompressor`](/pkg/archive/zip/#Reader.RegisterDecompressor),
    và kiểu
    [`Writer`](/pkg/archive/zip/#Writer) hiện có phương thức
    [`RegisterCompressor`](/pkg/archive/zip/#Writer.RegisterCompressor),
    cho phép kiểm soát tùy chọn nén cho từng tệp zip riêng lẻ.
    Các phương thức này có quyền ưu tiên hơn các hàm toàn cục
    [`RegisterDecompressor`](/pkg/archive/zip/#RegisterDecompressor) và
    [`RegisterCompressor`](/pkg/archive/zip/#RegisterCompressor) đã có sẵn.
  - Kiểu [`Scanner`](/pkg/bufio/#Scanner) của package [`bufio`](/pkg/bufio/)
    giờ có phương thức
    [`Buffer`](/pkg/bufio/#Scanner.Buffer),
    để chỉ định buffer ban đầu và kích thước buffer tối đa sử dụng trong quá trình quét.
    Điều này cho phép, khi cần, quét các token lớn hơn
    `MaxScanTokenSize`.
    Cũng cho `Scanner`, package hiện định nghĩa
    giá trị lỗi [`ErrFinalToken`](/pkg/bufio/#ErrFinalToken), để sử dụng bởi
    [các hàm split](/pkg/bufio/#SplitFunc) để hủy xử lý hoặc trả về một token rỗng cuối cùng.
  - Package [`compress/flate`](/pkg/compress/flate/)
    đã đánh dấu không dùng nữa các triển khai lỗi
    [`ReadError`](/pkg/compress/flate/#ReadError) và
    [`WriteError`](/pkg/compress/flate/#WriteError).
    Trong Go 1.5, chúng chỉ hiếm khi được trả về khi gặp lỗi;
    bây giờ chúng không bao giờ được trả về, mặc dù vẫn được định nghĩa để tương thích.
  - Các package [`compress/flate`](/pkg/compress/flate/),
    [`compress/gzip`](/pkg/compress/gzip/) và [`compress/zlib`](/pkg/compress/zlib/)
    giờ báo cáo
    [`io.ErrUnexpectedEOF`](/pkg/io/#ErrUnexpectedEOF) cho các luồng đầu vào bị cắt ngắn, thay vì
    [`io.EOF`](/pkg/io/#EOF).
  - Package [`crypto/cipher`](/pkg/crypto/cipher/) hiện
    ghi đè buffer đích trong trường hợp giải mã GCM thất bại.
    Điều này cho phép mã AESNI tránh sử dụng buffer tạm thời.
  - Package [`crypto/tls`](/pkg/crypto/tls/)
    có nhiều thay đổi nhỏ.
    Nó hiện cho phép
    [`Listen`](/pkg/crypto/tls/#Listen)
    thành công khi
    [`Config`](/pkg/crypto/tls/#Config)
    có `Certificates` là nil, miễn là callback `GetCertificate` được đặt,
    nó bổ sung hỗ trợ cho các bộ mã hóa RSA với AES-GCM,
    và
    nó bổ sung một
    [`RecordHeaderError`](/pkg/crypto/tls/#RecordHeaderError)
    để cho phép client (đặc biệt là package [`net/http`](/pkg/net/http/))
    báo cáo lỗi tốt hơn khi cố gắng kết nối TLS đến một server không phải TLS.
  - Package [`crypto/x509`](/pkg/crypto/x509/)
    giờ cho phép các chứng chỉ chứa số serial âm
    (về mặt kỹ thuật là lỗi, nhưng tiếc là phổ biến trong thực tế),
    và nó định nghĩa một
    [`InsecureAlgorithmError`](/pkg/crypto/x509/#InsecureAlgorithmError) mới
    để đưa ra thông báo lỗi tốt hơn khi từ chối chứng chỉ
    được ký bằng thuật toán không an toàn như MD5.
  - Các package [`debug/dwarf`](/pkg/debug/dwarf) và
    [`debug/elf`](/pkg/debug/elf/)
    cùng nhau bổ sung hỗ trợ cho các phần DWARF nén.
    Mã người dùng không cần cập nhật: các phần được giải nén tự động khi đọc.
  - Package [`debug/elf`](/pkg/debug/elf/)
    bổ sung hỗ trợ cho các phần ELF nén chung.
    Mã người dùng không cần cập nhật: các phần được giải nén tự động khi đọc.
    Tuy nhiên, các
    [`Section`](/pkg/debug/elf/#Section) nén không hỗ trợ truy cập ngẫu nhiên:
    chúng có trường `ReaderAt` là nil.
  - Package [`encoding/asn1`](/pkg/encoding/asn1/)
    giờ xuất khẩu
    [các hằng số tag và class](/pkg/encoding/asn1/#pkg-constants)
    hữu ích cho việc phân tích nâng cao các cấu trúc ASN.1.
  - Cũng trong package [`encoding/asn1`](/pkg/encoding/asn1/),
    [`Unmarshal`](/pkg/encoding/asn1/#Unmarshal) giờ từ chối nhiều mã hóa số nguyên và độ dài không chuẩn.
  - [`Decoder`](/pkg/encoding/base64/#Decoder) của package [`encoding/base64`](/pkg/encoding/base64)
    đã được sửa để xử lý các byte cuối cùng của đầu vào.
    Trước đây nó xử lý nhiều token bốn byte nhất có thể nhưng bỏ qua phần còn lại, tối đa ba byte.
    Do đó, `Decoder` hiện xử lý đúng các đầu vào trong mã hóa không có đệm (như
    [RawURLEncoding](/pkg/encoding/base64/#RawURLEncoding)),
    nhưng nó cũng từ chối các đầu vào trong mã hóa có đệm bị cắt ngắn hoặc kết thúc bằng các byte không hợp lệ,
    chẳng hạn như khoảng trắng ở cuối.
  - Package [`encoding/json`](/pkg/encoding/json/)
    giờ kiểm tra cú pháp của một
    [`Number`](/pkg/encoding/json/#Number)
    trước khi marshal nó, yêu cầu nó tuân theo đặc tả JSON cho các giá trị số.
    Như trong các bản phát hành trước, `Number` bằng không (chuỗi rỗng) được marshal thành literal 0 (không).
  - Hàm [`Marshal`](/pkg/encoding/xml/#Marshal) của package [`encoding/xml`](/pkg/encoding/xml/)
    giờ hỗ trợ thuộc tính `cdata`, tương tự như `chardata`
    nhưng mã hóa đối số của nó trong một hoặc nhiều tag `<![CDATA[ ... ]]>`.
  - Cũng trong package [`encoding/xml`](/pkg/encoding/xml/),
    phương thức [`Token`](/pkg/encoding/xml/#Decoder.Token) của
    [`Decoder`](/pkg/encoding/xml/#Decoder)
    giờ báo cáo lỗi khi gặp EOF trước khi thấy tất cả các tag mở được đóng,
    nhất quán với yêu cầu chung của nó là các tag trong đầu vào phải được khớp đúng cách.
    Để tránh yêu cầu đó, sử dụng
    [`RawToken`](/pkg/encoding/xml/#Decoder.RawToken).
  - Package [`fmt`](/pkg/fmt/) giờ cho phép
    bất kỳ kiểu số nguyên nào làm đối số cho đặc tả chiều rộng và độ chính xác `*` của
    [`Printf`](/pkg/fmt/#Printf).
    Trong các bản phát hành trước, đối số cho `*` được yêu cầu có kiểu `int`.
  - Cũng trong package [`fmt`](/pkg/fmt/),
    [`Scanf`](/pkg/fmt/#Scanf) giờ có thể quét chuỗi thập lục phân sử dụng %X, như một bí danh cho %x.
    Cả hai định dạng đều chấp nhận mọi kết hợp của thập lục phân chữ hoa và chữ thường.
  - Các package [`image`](/pkg/image/)
    và
    [`image/color`](/pkg/image/color/)
    bổ sung kiểu
    [`NYCbCrA`](/pkg/image/#NYCbCrA)
    và
    [`NYCbCrA`](/pkg/image/color/#NYCbCrA)
    để hỗ trợ hình ảnh Y'CbCr với alpha không nhân trước.
  - Việc triển khai [`MultiWriter`](/pkg/io/#MultiWriter) của package [`io`](/pkg/io/)
    giờ triển khai phương thức `WriteString`,
    để sử dụng bởi
    [`WriteString`](/pkg/io/#WriteString).
  - Trong package [`math/big`](/pkg/math/big/),
    [`Int`](/pkg/math/big/#Int) bổ sung
    [`Append`](/pkg/math/big/#Int.Append)
    và
    [`Text`](/pkg/math/big/#Int.Text)
    để kiểm soát nhiều hơn việc in.
  - Cũng trong package [`math/big`](/pkg/math/big/),
    [`Float`](/pkg/math/big/#Float) giờ triển khai
    [`encoding.TextMarshaler`](/pkg/encoding/#TextMarshaler) và
    [`encoding.TextUnmarshaler`](/pkg/encoding/#TextUnmarshaler),
    cho phép nó được serialize theo dạng tự nhiên bởi các package
    [`encoding/json`](/pkg/encoding/json/) và
    [`encoding/xml`](/pkg/encoding/xml/).
  - Cũng trong package [`math/big`](/pkg/math/big/),
    phương thức [`Append`](/pkg/math/big/#Float.Append) của
    [`Float`](/pkg/math/big/#Float) giờ hỗ trợ đối số độ chính xác đặc biệt -1.
    Như trong
    [`strconv.ParseFloat`](/pkg/strconv/#ParseFloat),
    độ chính xác -1 có nghĩa là sử dụng số chữ số nhỏ nhất cần thiết sao cho
    [`Parse`](/pkg/math/big/#Float.Parse)
    đọc kết quả vào một `Float` có cùng độ chính xác
    sẽ mang lại giá trị ban đầu.
  - Package [`math/rand`](/pkg/math/rand/)
    bổ sung hàm
    [`Read`](/pkg/math/rand/#Read)
    và tương tự
    [`Rand`](/pkg/math/rand/#Rand) bổ sung phương thức
    [`Read`](/pkg/math/rand/#Rand.Read).
    Điều này giúp tạo dữ liệu kiểm thử giả ngẫu nhiên dễ dàng hơn.
    Lưu ý rằng, giống như phần còn lại của package,
    các phương thức này không nên được sử dụng trong các ngữ cảnh mật mã;
    cho những mục đích đó, hãy sử dụng package [`crypto/rand`](/pkg/crypto/rand/).
  - Hàm [`ParseMAC`](/pkg/net/#ParseMAC) của package [`net`](/pkg/net/)
    giờ chấp nhận địa chỉ lớp liên kết IP-over-InfiniBand (IPoIB) 20 byte.
  - Cũng trong package [`net`](/pkg/net/),
    đã có một vài thay đổi đối với tra cứu DNS.
    Đầu tiên, triển khai lỗi [`DNSError`](/pkg/net/#DNSError) giờ triển khai
    [`Error`](/pkg/net/#Error),
    và đặc biệt phương thức mới
    [`IsTemporary`](/pkg/net/#DNSError.IsTemporary)
    của nó trả về true cho các lỗi máy chủ DNS.
    Thứ hai, các hàm tra cứu DNS như
    [`LookupAddr`](/pkg/net/#LookupAddr)
    giờ trả về tên miền có gốc (với dấu chấm ở cuối)
    trên Plan 9 và Windows, để khớp với hành vi của Go trên các hệ thống Unix.
  - Package [`net/http`](/pkg/net/http/)
    có một số bổ sung nhỏ ngoài hỗ trợ HTTP/2 đã được thảo luận.
    Đầu tiên,
    [`FileServer`](/pkg/net/http/#FileServer) giờ sắp xếp các danh sách thư mục được tạo ra theo tên tệp.
    Thứ hai,
    hàm [`ServeFile`](/pkg/net/http/#ServeFile) giờ từ chối phục vụ kết quả
    nếu đường dẫn URL của yêu cầu chứa ".." (dấu chấm-chấm) là một phần tử đường dẫn.
    Các chương trình thường nên sử dụng `FileServer` và
    [`Dir`](/pkg/net/http/#Dir)
    thay vì gọi `ServeFile` trực tiếp.
    Các chương trình cần phục vụ nội dung tệp trong phản hồi cho các URL chứa dấu chấm-chấm vẫn có thể
    gọi [`ServeContent`](/pkg/net/http/#ServeContent).
    Thứ ba,
    [`Client`](/pkg/net/http/#Client) giờ cho phép mã người dùng đặt
    header `Expect:` `100-continue` (xem
    [`Transport.ExpectContinueTimeout`](/pkg/net/http/#Transport)).
    Thứ tư, có
    [năm mã lỗi mới](/pkg/net/http/#pkg-constants):
    `StatusPreconditionRequired` (428),
    `StatusTooManyRequests` (429),
    `StatusRequestHeaderFieldsTooLarge` (431) và
    `StatusNetworkAuthenticationRequired` (511) từ RFC 6585,
    cũng như `StatusUnavailableForLegalReasons` (451) mới được phê duyệt gần đây.
    Thứ năm, việc triển khai và tài liệu hóa
    [`CloseNotifier`](/pkg/net/http/#CloseNotifier)
    đã được thay đổi đáng kể.
    Interface [`Hijacker`](/pkg/net/http/#Hijacker)
    giờ hoạt động đúng trên các kết nối đã được sử dụng trước đây với `CloseNotifier`.
    Tài liệu hiện mô tả khi nào `CloseNotifier`
    được mong đợi hoạt động.
  - Cũng trong package [`net/http`](/pkg/net/http/),
    có một vài thay đổi liên quan đến việc xử lý một cấu trúc
    [`Request`](/pkg/net/http/#Request) với trường `Method` của nó được đặt thành chuỗi rỗng.
    Trường `Method` rỗng luôn được ghi lại là bí danh cho `"GET"`
    và vẫn như vậy.
    Tuy nhiên, Go 1.6 sửa một vài routine không xử lý `Method` rỗng
    giống như `"GET"` tường minh.
    Đặc biệt, trong các bản phát hành trước
    [`Client`](/pkg/net/http/#Client) theo dõi các chuyển hướng chỉ với
    `Method` được đặt tường minh thành `"GET"`;
    trong Go 1.6, `Client` cũng theo dõi các chuyển hướng cho `Method` rỗng.
    Cuối cùng,
    [`NewRequest`](/pkg/net/http/#NewRequest) chấp nhận một đối số `method` chưa được
    ghi lại là được phép rỗng.
    Trong các bản phát hành trước, truyền đối số `method` rỗng dẫn đến
    một `Request` với trường `Method` rỗng.
    Trong Go 1.6, kết quả `Request` luôn có trường
    `Method` được khởi tạo: nếu đối số của nó là chuỗi rỗng, `NewRequest`
    đặt trường `Method` trong `Request` được trả về thành `"GET"`.
  - [`ResponseRecorder`](/pkg/net/http/httptest/#ResponseRecorder) của package
    [`net/http/httptest`](/pkg/net/http/httptest/)
    giờ khởi tạo header Content-Type mặc định
    sử dụng cùng thuật toán phát hiện nội dung như trong
    [`http.Server`](/pkg/net/http/#Server).
  - [`Parse`](/pkg/net/url/#Parse) của package [`net/url`](/pkg/net/url/)
    giờ nghiêm ngặt hơn và tuân thủ đặc tả hơn đối với việc phân tích
    tên máy chủ.
    Ví dụ, khoảng trắng trong tên máy chủ không còn được chấp nhận.
  - Cũng trong package [`net/url`](/pkg/net/url/),
    kiểu [`Error`](/pkg/net/url/#Error) giờ triển khai
    [`net.Error`](/pkg/net/#Error).
  - Các hàm [`IsExist`](/pkg/os/#IsExist),
    [`IsNotExist`](/pkg/os/#IsNotExist),
    và
    [`IsPermission`](/pkg/os/#IsPermission) của package [`os`](/pkg/os/)
    giờ trả về kết quả đúng khi hỏi về một
    [`SyscallError`](/pkg/os/#SyscallError).
  - Trên các hệ thống giống Unix, khi một lần ghi
    vào [`os.Stdout`
    hoặc `os.Stderr`](/pkg/os/#pkg-variables) (chính xác hơn là một `os.File`
    được mở cho file descriptor 1 hoặc 2) thất bại do lỗi broken pipe,
    chương trình sẽ phát tín hiệu `SIGPIPE`.
    Theo mặc định, điều này sẽ khiến chương trình thoát; điều này có thể được thay đổi bằng cách gọi hàm
    [`Notify`](/pkg/os/signal/#Notify) của package
    [`os/signal`](/pkg/os/signal)
    cho `syscall.SIGPIPE`.
    Một lần ghi vào broken pipe trên file descriptor khác 1 hoặc 2 sẽ đơn giản
    trả về `syscall.EPIPE` (có thể được bọc trong
    [`os.PathError`](/pkg/os#PathError)
    và/hoặc [`os.SyscallError`](/pkg/os#SyscallError))
    cho người gọi.
    Hành vi cũ của việc phát tín hiệu `SIGPIPE` không thể bắt
    sau 10 lần ghi liên tiếp vào broken pipe không còn xảy ra nữa.
  - Trong package [`os/exec`](/pkg/os/exec/),
    phương thức [`Output`](/pkg/os/exec/#Cmd.Output) của
    [`Cmd`](/pkg/os/exec/#Cmd) tiếp tục trả về một
    [`ExitError`](/pkg/os/exec/#ExitError) khi lệnh thoát với trạng thái không thành công.
    Nếu lỗi chuẩn bị bị bỏ qua,
    `ExitError` được trả về giờ chứa tiền tố và hậu tố
    (hiện tại là 32 kB) của đầu ra lỗi chuẩn của lệnh thất bại,
    để gỡ lỗi hoặc đưa vào thông báo lỗi.
    Phương thức [`String`](/pkg/os/exec/#ExitError.String)
    của `ExitError` không hiển thị lỗi chuẩn đã chụp;
    các chương trình phải truy xuất nó từ cấu trúc dữ liệu
    một cách riêng biệt.
  - Trên Windows, hàm [`Join`](/pkg/path/filepath/#Join) của package [`path/filepath`](/pkg/path/filepath/)
    giờ xử lý đúng trường hợp khi base là đường dẫn drive tương đối.
    Ví dụ, ``Join(`c:`,`` `` `a`) `` giờ
    trả về `` `c:a` `` thay vì `` `c:\a` `` như trong các bản phát hành trước.
    Điều này có thể ảnh hưởng đến mã mong đợi kết quả không chính xác.
  - Trong package [`regexp`](/pkg/regexp/),
    kiểu
    [`Regexp`](/pkg/regexp/#Regexp) luôn an toàn để sử dụng bởi
    các goroutine đồng thời.
    Nó sử dụng một [`sync.Mutex`](/pkg/sync/#Mutex) để bảo vệ
    bộ nhớ cache của các không gian tạm thời được sử dụng trong quá trình tìm kiếm biểu thức chính quy.
    Một số server có nhiều đồng thời sử dụng cùng `Regexp` từ nhiều goroutine
    đã thấy hiệu suất giảm do tranh chấp trên mutex đó.
    Để giúp các server đó, `Regexp` giờ có phương thức
    [`Copy`](/pkg/regexp/#Regexp.Copy),
    tạo một bản sao của `Regexp` chia sẻ hầu hết cấu trúc
    của bản gốc nhưng có không gian cache tạm thời riêng.
    Hai goroutine có thể sử dụng các bản sao khác nhau của `Regexp`
    mà không có tranh chấp mutex.
    Một bản sao có thêm chi phí không gian, vì vậy `Copy`
    chỉ nên được sử dụng khi đã quan sát thấy tranh chấp.
  - Package [`strconv`](/pkg/strconv/)
    bổ sung
    [`IsGraphic`](/pkg/strconv/#IsGraphic),
    tương tự như [`IsPrint`](/pkg/strconv/#IsPrint).
    Nó cũng bổ sung
    [`QuoteToGraphic`](/pkg/strconv/#QuoteToGraphic),
    [`QuoteRuneToGraphic`](/pkg/strconv/#QuoteRuneToGraphic),
    [`AppendQuoteToGraphic`](/pkg/strconv/#AppendQuoteToGraphic),
    và
    [`AppendQuoteRuneToGraphic`](/pkg/strconv/#AppendQuoteRuneToGraphic),
    tương tự như
    [`QuoteToASCII`](/pkg/strconv/#QuoteToASCII),
    [`QuoteRuneToASCII`](/pkg/strconv/#QuoteRuneToASCII),
    và các hàm tương tự.
    Họ `ASCII` thoát tất cả các ký tự khoảng trắng ngoại trừ khoảng trắng ASCII (U+0020).
    Ngược lại, họ `Graphic` không thoát bất kỳ ký tự khoảng trắng Unicode nào (danh mục Zs).
  - Trong package [`testing`](/pkg/testing/),
    khi một test gọi
    [t.Parallel](/pkg/testing/#T.Parallel),
    test đó bị tạm dừng cho đến khi tất cả các test không song song hoàn thành, và sau đó
    test đó tiếp tục thực thi cùng với tất cả các test song song khác.
    Go 1.6 thay đổi thời gian được báo cáo cho test đó:
    trước đây, thời gian chỉ tính phần thực thi song song,
    nhưng bây giờ nó cũng tính thời gian từ khi bắt đầu kiểm thử
    đến lời gọi `t.Parallel`.
  - Package [`text/template`](/pkg/text/template/)
    chứa hai thay đổi nhỏ, ngoài [các thay đổi lớn](#template)
    được mô tả ở trên.
    Đầu tiên, nó thêm kiểu
    [`ExecError`](/pkg/text/template/#ExecError) mới
    được trả về cho bất kỳ lỗi nào trong quá trình
    [`Execute`](/pkg/text/template/#Template.Execute)
    không xuất phát từ `Write` đến writer bên dưới.
    Người gọi có thể phân biệt lỗi sử dụng template với lỗi I/O bằng cách kiểm tra
    `ExecError`.
    Thứ hai, phương thức
    [`Funcs`](/pkg/text/template/#Template.Funcs)
    giờ kiểm tra rằng các tên được sử dụng làm khóa trong
    [`FuncMap`](/pkg/text/template/#FuncMap)
    là các định danh có thể xuất hiện trong một lời gọi hàm template.
    Nếu không, `Funcs` sẽ panic.
  - Hàm [`Parse`](/pkg/time/#Parse) của package [`time`](/pkg/time/)
    luôn từ chối bất kỳ ngày trong tháng nào lớn hơn 31,
    chẳng hạn như ngày 32 tháng 1.
    Trong Go 1.6, `Parse` giờ cũng từ chối ngày 29 tháng 2 trong các năm không nhuận,
    ngày 30, 31 tháng 2, ngày 31 tháng 4, ngày 31 tháng 6, ngày 31 tháng 9 và ngày 31 tháng 11.
