---
title: Ghi chú bản phát hành Go 1.27
template: false
---

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu về Go 1.27 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.27, ra mắt vào [tháng 8 năm 2026](/doc/devel/release#go1.27.0), sáu tháng sau [Go 1.26](/doc/go1.26).
Phần lớn thay đổi của bản phát hành này nằm trong việc triển khai toolchain, runtime và các thư viện.
Như thường lệ, bản phát hành này duy trì [cam kết về tính tương thích](/doc/go1compat) của Go 1.
Chúng tôi kỳ vọng hầu hết chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

## Thay đổi đối với ngôn ngữ {#language}

<!-- go.dev/issue/77273 -->

Go 1.27 hiện hỗ trợ [phương thức generic](/issue/77273):
một [khai báo phương thức](/ref/spec#Method_declarations) có thể khai báo riêng
[các tham số kiểu](/ref/spec#Type_parameter_declarations) của nó.
Thay đổi được mong đợi rộng rãi này cho phép thêm các hàm generic trong
không gian tên của một kiểu dữ liệu cụ thể, trong khi trước đây phải khai báo
các hàm như vậy với phạm vi của toàn bộ gói.
Ví dụ, `math/rand/v2` hiện cũng khai báo một phương thức generic với chữ ký
`(*Rand) N[Int intType](Int) Int`, trong khi trước đây nó chỉ có một
hàm generic `N[Int intType](Int) Int` (xem [bên dưới](#rand) để biết chi tiết).
Lưu ý rằng các phương thức của [interface](/ref/spec#Interface_types) không được khai báo
tham số kiểu và các phương thức interface cũng không thể được triển khai bằng các phương thức generic.

<!-- go.dev/issue/9859 -->

Một khóa trong [struct literal](/ref/spec#Composite_literals) giờ đây có thể là bất kỳ
[trình chọn trường](/issue/9859) hợp lệ nào cho kiểu struct, không chỉ là
tên trường (cấp cao nhất) của struct.

<!-- go.dev/issue/77245 -->

Suy luận kiểu hàm đã được [tổng quát hóa](/issue/77245) để áp dụng trong mọi
ngữ cảnh mà một hàm generic được [gán](/ref/spec#Assignability) cho một
biến có kiểu hàm tương ứng (hoặc được chuyển đổi thành kiểu hàm tương ứng).

## Công cụ {#tools}

<!-- go.dev/issue/77177 -->

Việc phân tích cú pháp tệp phản hồi (`@file`) hiện được hỗ trợ cho các công cụ `compile`, `link`, `asm`, `cgo`, `cover` và `pack`.
Tệp phản hồi chứa các đối số được phân tách bằng khoảng trắng, hỗ trợ chuỗi được đặt trong dấu nháy đơn và dấu nháy kép, chuỗi thoát và tiếp nối dòng bằng dấu gạch chéo ngược-dòng mới.
Định dạng này tương thích với triển khai tệp phản hồi của GCC để đảm bảo khả năng tương tác với các hệ thống build hiện có.

### Lệnh Go {#go-command}

<!-- go.dev/issue/78090 -->

Lệnh `go` không còn hỗ trợ hệ thống quản lý phiên bản `bzr`.
Lệnh này sẽ không còn có thể trực tiếp tải các module được lưu trữ trên các máy chủ `bzr`.

#### `GODEBUG`

<!-- go.dev/issue/79422 -->

Bắt đầu từ Go 1.27, lệnh `go` nhận dạng một thiết lập `GODEBUG`
mà phần hỗ trợ đã bị gỡ bỏ (chẳng hạn như `asynctimerchan`, xem [bên dưới](#runtime)) nếu thiết lập này xuất hiện trong các tệp
`go.mod` (các mục `godebug`) và các tệp mã nguồn `.go` (các chú thích `//go:debug`).
Lệnh này chấp nhận các thiết lập đó nếu chúng được đặt thành giá trị mặc định cuối cùng được xác lập trước khi
thiết lập bị gỡ bỏ.
Nếu chúng được đặt thành một giá trị cũ, lệnh `go` sẽ thất bại.
Thay đổi này phù hợp với tinh thần của [cam kết tương thích Go 1](/doc/go1compat)
và cho phép các chương trình hiện có đặt các thiết lập `GODEBUG` được hỗ trợ tiếp tục
xây dựng và chạy mà không cần thay đổi ngay cả khi phần hỗ trợ cho thiết lập tương ứng đã bị gỡ bỏ.

#### `go test`

`go test` giờ đây gọi kiểm tra vet `stdversion` theo mặc định.
Kiểm tra này báo cáo việc sử dụng các ký hiệu trong thư viện chuẩn quá mới
so với phiên bản Go đang được áp dụng trong tệp tham chiếu,
được xác định bởi chỉ thị `go` trong `go.mod` và các thẻ xây dựng trên tệp đó.

<!-- go.dev/issue/62728 -->

`go test -json` giờ đây chú thích các dòng `"Action":"output"` bằng một trường mới tùy chọn
`"OutputType"`, chỉ định kiểu đầu ra.
Hiện tại, các giá trị có thể có bao gồm "error", "error-continue" và "frame".
Xem [trợ giúp cmd/test2json](/cmd/test2json#hdr-Output_Format) để biết chi tiết.

#### `go doc`

<!-- go.dev/issue/63696 -->

Lệnh `go doc` giờ đây hỗ trợ cú pháp `package@version`, chẳng hạn như
`go doc example.com/pkg@v1.2.3`.

<!-- go.dev/issue/26715 -->

Lệnh `go doc` giờ đây chấp nhận tùy chọn dòng lệnh `-ex` để
liệt kê các ví dụ có thể thực thi của gói hoặc ký hiệu đã cho.
Khi tên ví dụ được truyền trên dòng lệnh (chẳng hạn như
`go doc bytes.ExampleBuffer`), `go doc` giờ đây in mã nguồn của ví dụ
cùng với các chú thích.

#### `go fix`

Lệnh `go fix` chứa một số công cụ hiện đại hóa mới (`atomictypes`, `embedlit`, `slicesbackward` và `unsafefuncs`).

<!-- go.dev/issue/77581 -->

Trình phân tích `fmtappendf` hiện có đã bị gỡ bỏ do các mối quan ngại về phong cách.

Trình phân tích `waitgroup` hiện có đã được đổi tên thành `waitgroupgo` để tránh nhầm lẫn.

#### `go mod tidy`

<!-- go.dev/issue/56471 -->

Đối với các module chỉ định `go 1.27` trở lên trong tệp `go.mod`, `go mod tidy` giờ đây tự động hợp nhất các khối require trùng lặp. Điều này đảm bảo tệp duy trì cấu trúc chuẩn, gọn gàng, chứa nhiều nhất hai khối require: một khối cho dependency trực tiếp và một khối cho dependency gián tiếp.

Các khối chú thích hiện có được gắn với dependency được bảo toàn trong quá trình hợp nhất này. Nếu một khối chú thích được liên kết với một tập chỉ thị hỗn hợp (chứa cả dependency trực tiếp và gián tiếp), khối chú thích sẽ được hợp nhất và gắn vào khối dependency trực tiếp mới.

Trước đây, nếu tệp `go.mod` tích lũy nhiều khối require rời rạc (thường do chỉnh sửa thủ công, xung đột hợp nhất Git chưa được giải quyết hoặc nâng cấp cũ), `go mod tidy` sẽ giữ nguyên các khối thừa hoặc vô tình tạo thêm các khối mới. Công cụ này giờ đây thực thi nghiêm ngặt bố cục hai khối, hợp nhất các yêu cầu khác nhau vào các khối tương ứng và tự động dọn dẹp cấu trúc của tệp module.

### Trace

<!-- go.dev/issue/78921 -->

Tùy chọn dòng lệnh `-http` của `go tool trace` giờ đây giới hạn địa chỉ lắng nghe ở localhost khi chỉ được truyền một cổng (ví dụ: `-http=:6060`). Thay đổi này giúp `go tool trace` nhất quán với hành vi của cờ `-http` của `go tool pprof`.  
Để lắng nghe trên tất cả địa chỉ, hãy chỉ định rõ địa chỉ được yêu cầu (ví dụ: `-http=0.0.0.0:6060`).

## Runtime {#runtime}

<!-- CL 742580 -->

Dấu vết ngăn xếp cho các module có chỉ thị `go` cấu hình Go 1.27 trở lên giờ đây sẽ bao gồm nhãn goroutine của [`runtime/pprof`](https://pkg.go.dev/runtime/pprof) trong dòng tiêu đề. Hành vi này có thể được vô hiệu hóa bằng thiết lập `tracebacklabels=0` `GODEBUG` (được thêm trong [Go 1.26](/doc/godebug#go-126)). Tùy chọn loại bỏ này dự kiến sẽ được giữ vô thời hạn trong trường hợp nhãn goroutine thu thập thông tin nhạy cảm không nên được cung cấp trong dấu vết ngăn xếp.

<!-- CL 781580 -->

Thiết lập `asynctimerchan` `GODEBUG` (được thêm trong [Go 1.23](/doc/godebug#go-123)) đã bị xóa vĩnh viễn. Các channel được tạo bởi package [`time`](https://pkg.go.dev/time) giờ đây luôn không có bộ đệm (đồng bộ), bất kể các thiết lập `GODEBUG`.

### Cấp phát bộ nhớ nhanh hơn

<!-- go.dev.issue/79286 -->

Trình biên dịch hiện tạo ra các lệnh gọi đến các thường trình cấp phát bộ nhớ
chuyên biệt theo kích thước, giúp giảm chi phí của một số lần cấp phát bộ nhớ
nhỏ (<80 byte) lên đến 30%.
Các cải tiến khác nhau tùy thuộc vào khối lượng công việc, nhưng mức cải thiện
tổng thể được kỳ vọng là ~1% trong các chương trình thực tế có nhiều hoạt động
cấp phát.
Điều này khiến kích thước binary tăng khoảng 60 KB (không phụ thuộc vào
khối lượng công việc).
Vui lòng [tạo issue](/issue/new) nếu bạn nhận thấy bất kỳ hồi quy nào.
Bạn có thể đặt `GOEXPERIMENT=nosizespecializedmalloc` tại thời điểm build để
vô hiệu hóa tính năng này.
Thiết lập opt-out này dự kiến sẽ bị xóa trong Go 1.28.

### Hồ sơ rò rỉ goroutine {#goroutineleak-profiles}

<!-- go.dev/issue/74609 -->

Một kiểu hồ sơ mới báo cáo các goroutine bị rò rỉ, trước đây có sẵn dưới dạng
một thử nghiệm trong [Go 1.26](/doc/go1.26#goroutineleak-profiles), hiện đã
được cung cấp rộng rãi.
Kiểu hồ sơ mới, có tên là `goroutineleak`, được hỗ trợ trong gói
[`runtime/pprof`](/pkg/runtime/pprof).
Nó cũng có sẵn dưới dạng endpoint [`net/http/pprof`](/pkg/net/http/pprof)
`/debug/pprof/goroutineleak`.

Một goroutine *bị rò rỉ* là một goroutine bị chặn trên một nguyên thủy đồng thời
nào đó (channel, [`sync.Mutex`](/pkg/sync#Mutex), [`sync.Cond`](/pkg/sync#Cond), v.v.)
mà không thể nào được bỏ chặn.
Runtime phát hiện các goroutine bị rò rỉ bằng cách sử dụng bộ gom rác: nếu một
goroutine G bị chặn trên nguyên thủy đồng thời P, và P không thể truy cập được
từ bất kỳ goroutine nào có thể chạy hoặc bất kỳ goroutine nào mà *các
goroutine đó* có thể bỏ chặn, thì P không thể được bỏ chặn, vì vậy goroutine G
sẽ không bao giờ thức dậy.
Mặc dù không thể phát hiện các goroutine bị chặn vĩnh viễn trong mọi trường
hợp, cách tiếp cận này phát hiện được một lớp lớn các kiểu rò rỉ như vậy.

Vì kỹ thuật này dựa trên khả năng truy cập, runtime có thể không xác định được
các rò rỉ gây ra bởi việc chặn trên các nguyên thủy đồng thời có thể truy cập
thông qua các biến toàn cục hoặc các biến cục bộ của các goroutine có thể chạy.

Xem [ghi chú bản phát hành Go 1.26](/doc/go1.26#goroutineleak-profiles) để biết một ví dụ.

Đặc biệt cảm ơn Vlad Saioc tại Uber vì đã đóng góp công việc này.

Thiết lập `goroutineleakprofile` `GOEXPERIMENT` hiện đã bị xóa.

## Compiler {#compiler}

Trình biên dịch hiện giải quyết tên tệp tương đối trong chỉ thị `//line` hoặc `/*line*/`
dựa trên thư mục của tệp chứa chỉ thị đó,
phù hợp với hành vi của [`go/scanner`](/pkg/go/scanner).
Các tên tệp tuyệt đối không bị ảnh hưởng.
Xem [#70478](/issue/70478).

<!-- go.dev/issue/60324, CL 770200 -->

Trình biên dịch hiện tạo các tên đơn giản hơn cho các hàm literal
(closure).
Trước đây, khi hàm chứa nó được inline, tên của hàm literal
có thể trở nên rất dài.
Giờ đây trình biên dịch chọn cùng một tên cho hàm literal
bất kể có inline hay không.
Nó cũng có thể kết hợp nhiều thể hiện của cùng một hàm literal
(khi hàm chứa nó được inline) để chia sẻ cùng một mã trong
binary đã biên dịch.
Thay đổi này không ảnh hưởng đến chức năng của mã Go.
Các bài kiểm tra kiểm tra tên symbol có thể cần được cập nhật, mặc dù
khuyến nghị không phụ thuộc vào tên của các hàm literal.
Đối với các chương trình [so sánh không chính xác](/pkg/reflect#Value.Pointer)
con trỏ mã hàm để kiểm tra tính bằng nhau,
vấn đề có thể dễ lộ rõ hơn với Go 1.27, vì các hàm literal với dữ liệu
closure được capture khác nhau có thể có con trỏ mã bằng nhau trong nhiều trường hợp hơn.

## Linker {#linker}

<!-- CL 751260, go.dev/issue/58722 -->

Khi nhắm mục tiêu macOS, linker hiện chấp nhận các tùy chọn dòng lệnh `-macos` và `-macsdk`,
dùng để chỉ định phiên bản OS và SDK trong
lệnh tải `LC_BUILD_VERSION`.
Theo mặc định, nó chọn phiên bản macOS cũ nhất được hỗ trợ (hiện là
[13.0.0](#darwin)) và phiên bản SDK gần đây (hiện là 26.2.0).

## Thư viện chuẩn {#library}

### Các gói encoding/json/v2 và encoding/json/jsontext mới {#jsonv2}

<!-- go.dev/issue/71497 -->

Hiện có hai gói mới:

  - Gói [`encoding/json/v2`](/pkg/encoding/json/v2) là một bản sửa đổi lớn
    của [`encoding/json`](/pkg/encoding/json). Gói này cung cấp
    [`Marshal`](/pkg/encoding/json/v2#Marshal),
    [`MarshalWrite`](/pkg/encoding/json/v2#MarshalWrite),
    [`MarshalEncode`](/pkg/encoding/json/v2#MarshalEncode),
    [`Unmarshal`](/pkg/encoding/json/v2#Unmarshal),
    [`UnmarshalRead`](/pkg/encoding/json/v2#UnmarshalRead), và
    [`UnmarshalDecode`](/pkg/encoding/json/v2#UnmarshalDecode),
    tất cả đều chấp nhận các đối số [`Options`](/pkg/encoding/json/v2#Options)
    dạng biến số để cấu hình hành vi marshal và unmarshal.

  - Gói [`encoding/json/jsontext`](/pkg/encoding/json/jsontext) cung cấp
    xử lý cú pháp JSON ở mức thấp hơn.
    Các kiểu [`Encoder`](/pkg/encoding/json/jsontext#Encoder) và
    [`Decoder`](/pkg/encoding/json/jsontext#Decoder) hoạt động trên
    JSON dưới dạng một chuỗi
    [`Token`](/pkg/encoding/json/jsontext#Token) và
    [`Value`](/pkg/encoding/json/jsontext#Value),
    duy trì một máy trạng thái để đảm bảo chuỗi được tạo ra hoặc được tiêu thụ
    là văn bản JSON hợp lệ.

Gói v2 chọn các giá trị mặc định nghiêm ngặt hơn và có khả năng tương tác tốt hơn so với v1:
nó từ chối UTF-8 không hợp lệ trong chuỗi JSON và từ chối các tên trùng lặp trong
một đối tượng JSON. Xem tài liệu gói [`encoding/json`](/pkg/encoding/json#hdr-Migrating_to_v2) v1
để biết toàn bộ tập hợp khác biệt về hành vi và
các tùy chọn có sẵn để điều chỉnh chúng.

Gói [`encoding/json`](/pkg/encoding/json) hiện được hỗ trợ bởi
cài đặt v2. Hành vi marshal và unmarshal được giữ nguyên, nhưng
văn bản chính xác của thông báo lỗi có thể khác. Gói này cũng có thêm một số
[`Options`](/pkg/encoding/json#Options) mới có thể cấu hình v2 hoạt động
với ngữ nghĩa v1 để tránh yêu cầu di chuyển hoàn toàn sang API mới.
API v1 sẽ tiếp tục được hỗ trợ và người dùng không bắt buộc phải di chuyển.

Hiệu năng marshal nhìn chung tương đương với cài đặt trước đó,
trong khi hiệu năng unmarshal nhanh hơn đáng kể.

Người dùng gặp vấn đề tương thích với cài đặt mới
có thể vô hiệu hóa nó bằng cách đặt `GOEXPERIMENT=nojsonv2` tại thời điểm build,
khôi phục cài đặt v1 ban đầu.
Tùy chọn vô hiệu hóa này dự kiến sẽ bị loại bỏ trong một bản phát hành tương lai.

Một số thay đổi đáng chú ý đối với `encoding/json/v2` trong quá trình `GOEXPERIMENT`
bao gồm các loại bỏ sau:
tùy chọn tag `format` ([#79071](/issue/79071)),
tùy chọn tag `unknown` ([#77271](/issue/77271)),
tùy chọn marshal `DiscardUnknownMembers` ([#77271](/issue/77271)),
và lỗi sentinel `SkipFunc` ([#74324](/issue/74324)).
Ngoài ra, tùy chọn tag `inline` đã được đổi tên thành `embed` ([#79985](/issue/79985)), và
hành vi đã được cập nhật cho tùy chọn tag
[`string`](/pkg/encoding/json/v2#hdr-JSON_Representation_of_Go_structs)
([#79065](/issue/79065))
và tùy chọn [`MatchCaseInsensitiveNames`](/pkg/encoding/json/v2#MatchCaseInsensitiveNames)
([CL 792780](/cl/792780)).
Trong `encoding/json/jsontext`, các bộ truy cập số của [`Token`](/pkg/encoding/json/jsontext#Token)
đã được thay đổi để cũng trả về lỗi ([#77666](/issue/77666)).

Xem [vấn đề đề xuất](/issue/71497) để biết thông tin nền và chi tiết bổ sung.
Nếu bạn cần vô hiệu hóa cài đặt mới, [vui lòng tạo một issue](/issue/new).

### New crypto/mldsa package {#crypto_mldsa}

<!-- https://go.dev/issue/77626, https://go.dev/issue/78888 -->

The new [`crypto/mldsa`](/pkg/crypto/mldsa) package implements the post-quantum ML-DSA signature
scheme specified in FIPS 204.

[`crypto/x509`](/pkg/crypto/x509) now supports ML-DSA private keys, public keys, and signatures.

[`crypto/tls`](/pkg/crypto/tls) now supports ML-DSA signatures in TLS 1.3, with the new
[`MLDSA44`](/pkg/crypto/tls#MLDSA44), [`MLDSA65`](/pkg/crypto/tls#MLDSA65),
and [`MLDSA87`](/pkg/crypto/tls#MLDSA87) [`SignatureScheme`](/pkg/crypto/tls#SignatureScheme) values.

### New uuid package {#uuid}

<!-- https://go.dev/issue/62026 -->

The new [`uuid`](/pkg/uuid) package generates and parses UUIDs.


### New experimental simd package {#simd}

Go 1.27 introduces a new experimental [`simd`](/pkg/simd)
package that provides portable and vector-size-agnostic SIMD
support. It will make use of the hardware instructions
if they are available.
This package is enabled by setting the environment variable
`GOEXPERIMENT=simd` at build time.

The `simd` package is available on all architectures, and provides
vector types of unspecified size such as [`Int8s`](/pkg/simd#Int8s) and [`Float32s`](/pkg/simd#Float32s).
It supports a "scalable" subset of the operations present in the
[`simd/archsimd`](/pkg/simd/archsimd) package
that are hardware-supported or easily emulated across architectures
and vector widths.

See the [proposal issue](/issue/78902) for more details.

### Experimental simd/archsimd package {#archsimd}

Go 1.27 continues the experimental support for SIMD operations in
the [`simd/archsimd`](/pkg/simd/archsimd/) package that began in [Go 1.26](/doc/go1.26#simd).
This release revises the amd64 API and adds support for arm64 "Neon" 128-bit SIMD and WebAssembly 128-bit SIMD.
The `simd/archsimd` package is enabled by setting the environment variable
`GOEXPERIMENT=simd` at build time.

This package provides access to architecture-specific SIMD operations.
It supports 128-bit vector types on wasm, arm64, and amd64,
and 256-bit and 512-bit vector types on some amd64 processors.
The API is not yet considered stable.

See the [package documentation](/pkg/simd/archsimd) and the [proposal issue](/issue/73787) for more details.

We intend to provide support for additional architectures in future versions, but the
API is intentionally architecture-specific and thus non-portable.

### Thay đổi nhỏ đối với thư viện {#minor_library_changes}

#### [`bytes`](/pkg/bytes/)

<!-- 6-stdlib/99-minor/bytes/71151.md -->

Hàm mới [`CutLast`](/pkg/bytes#CutLast) cắt một `[]byte`
xung quanh lần xuất hiện cuối cùng của dấu phân cách.
Hàm này có thể thay thế và đơn giản hóa một số cách sử dụng phổ biến của [`LastIndex`](/pkg/bytes#LastIndex).

#### [`compress/flate`](/pkg/compress/flate/)

<!-- go.dev/issue/75532, CL 707355 -->

Tốc độ nén được cải thiện trong Go 1.27.
Kết quả đầu ra đã mã hóa chính xác từ [`Writer`](/pkg/compress/flate#Writer) có thể khác với Go 1.26
do thay đổi trong cách triển khai bộ mã hóa.
Vì DEFLATE là phương thức nén nền tảng được sử dụng trong [`archive/zip`](/pkg/archive/zip), [`compress/gzip`](/pkg/compress/gzip), [`compress/zlib`](/pkg/compress/zlib), và [`image/png`](/pkg/image/png),
kết quả đầu ra từ các gói đó cũng có thể đã thay đổi.

#### [`crypto`](/pkg/crypto/)

<!-- 6-stdlib/99-minor/crypto/77626.md -->

Giá trị [`MLDSAMu`](/pkg/crypto#MLDSAMu) [`Hash`](/pkg/crypto#Hash) mới
được thêm vào để sử dụng làm cơ chế báo hiệu cho việc ký External μ ML-DSA.

#### [`crypto/ecdsa`](/pkg/crypto/ecdsa/)

<!-- 6-stdlib/99-minor/crypto/ecdsa/hashlen.md -->

[`PrivateKey.Sign`](/pkg/crypto/ecdsa#PrivateKey.Sign) hiện kiểm tra rằng độ dài của hash là chính xác,
nếu một [`SignerOpts`](/pkg/crypto#SignerOpts) khác nil được cung cấp.

#### [`crypto/tls`](/pkg/crypto/tls/)

<!-- 6-stdlib/99-minor/crypto/tls/77363.md -->

Trường mới [`QUICConfig.ClientHelloInfoConn`](/pkg/crypto/tls#QUICConfig.ClientHelloInfoConn) chỉ định [`net.Conn`](/pkg/net#Conn) được sử dụng
cho trường [`ClientHelloInfo.Conn`](/pkg/crypto/tls#ClientHelloInfo.Conn) trong quá trình bắt tay của máy chủ QUIC.

<!-- 6-stdlib/99-minor/crypto/tls/78543.md -->

Việc trao đổi khóa [`MLKEM1024`](/pkg/crypto/tls#MLKEM1024) hiện được hỗ trợ. Có thể bật tính năng này bằng cách thêm nó vào
[`Config.CurvePreferences`](/pkg/crypto/tls#Config.CurvePreferences).

<!-- 6-stdlib/99-minor/crypto/tls/78888.md -->
<!-- crypto/tls ML-DSA support is documented in doc/next/6-stdlib/70-mldsa.md. -->

<!-- 6-stdlib/99-minor/crypto/tls/79367.md -->

[`Config.Rand`](/pkg/crypto/tls#Config.Rand) hiện đã bị loại bỏ.
Để kiểm thử xác định, hãy sử dụng [`testing/cryptotest.SetGlobalRandom`](/pkg/testing/cryptotest#SetGlobalRandom).

<!-- 6-stdlib/99-minor/crypto/tls/tlsmlkem.md -->

Các trao đổi khóa lai hậu lượng tử hiện có thể được bật rõ ràng trong
[`Config.CurvePreferences`](/pkg/crypto/tls#Config.CurvePreferences) ngay cả khi các tùy chọn `tlsmlkem=0` hoặc `tlssecpmlkem=0` `GODEBUG`
được sử dụng. Những tùy chọn đó luôn chỉ được dùng để áp dụng cho
tập mặc định được sử dụng khi [`Config.CurvePreferences`](/pkg/crypto/tls#Config.CurvePreferences) là nil.

<!-- go.dev/issue/24673 -->

Trường mới [`ConnectionState.LocalCertificate`](/pkg/crypto/tls#ConnectionState.LocalCertificate)
chứa chuỗi chứng chỉ được trình bày cho phía đối tác kết nối trong quá trình bắt tay.

<!-- go.dev/issue/75316 -->

Các thiết lập `GODEBUG` `tlsunsafeekm` (được thêm trong [Go 1.22](/doc/godebug#go-122)),
`tlsrsakex` (được thêm trong [Go 1.22](/doc/godebug#go-122)),
`tls3des` (được thêm trong [Go 1.23](/doc/godebug#go-123)),
`tls10server` (được thêm trong [Go 1.22](/doc/godebug#go-122)),
và `x509keypairleaf` (được thêm trong [Go 1.23](/doc/godebug#go-123))
đã bị xóa vĩnh viễn.

<!-- go.dev/issue/72111 -->

Thiết lập `GODEBUG` `tlskyber` (được thêm trong [Go 1.23](/doc/godebug#go-123))
đã bị xóa trong [Go 1.24](/doc/godebug#go-124) nhưng việc xóa này chưa được
ghi nhận trong [ghi chú phát hành cho Go 1.24](/doc/go1.24.md) vào thời điểm đó;
hiện đã được ghi nhận.

#### [`crypto/x509`](/pkg/crypto/x509/)

<!-- 6-stdlib/99-minor/crypto/x509/75260.md -->

Khi phân tích cú pháp vào các trường [`pkix.Name`](/pkg/crypto/x509/pkix#Name),
hiện hỗ trợ phạm vi rộng hơn các kiểu [`pkix.AttributeTypeAndValue.Value`](/pkg/crypto/x509/pkix#AttributeTypeAndValue.Value),
và các kiểu không xác định được sẽ được phân tích thành
[`asn1.RawValue`](/pkg/encoding/asn1#RawValue).

<!-- 6-stdlib/99-minor/crypto/x509/76133.md -->

Các trường mới [`Certificate.RawSignatureAlgorithm`](/pkg/crypto/x509#Certificate.RawSignatureAlgorithm), [`CertificateRequest.RawSignatureAlgorithm`](/pkg/crypto/x509#CertificateRequest.RawSignatureAlgorithm),
và [`RevocationList.RawSignatureAlgorithm`](/pkg/crypto/x509#RevocationList.RawSignatureAlgorithm) hiển thị
AlgorithmIdentifier được mã hóa DER của thuật toán chữ ký, bao gồm cả khi
trường SignatureAlgorithm là [`UnknownSignatureAlgorithm`](/pkg/crypto/x509#UnknownSignatureAlgorithm).

<!-- 6-stdlib/99-minor/crypto/x509/77865.md -->

[`SystemCertPool`](/pkg/crypto/x509#SystemCertPool) hiện tôn trọng SSL_CERT_FILE và SSL_CERT_DIR trên Windows và
Darwin. Khi các biến môi trường này được đặt, các root được tải từ đĩa và
thay vì sử dụng các API xác minh chứng chỉ của nền tảng, trình xác minh Go
gốc được sử dụng. Hành vi này có thể được vô hiệu hóa bằng
`GODEBUG=x509sslcertoverrideplatform=0`.

<!-- 6-stdlib/99-minor/crypto/x509/78888.md -->
<!-- crypto/x509 ML-DSA support is documented in doc/next/6-stdlib/70-mldsa.md. -->

#### [`crypto/x509/pkix`](/pkg/crypto/x509/pkix/)

<!-- 6-stdlib/99-minor/crypto/x509/pkix/33093.md -->

[`RDNSequence.String`](/pkg/crypto/x509/pkix#RDNSequence.String) (và do đó [`Name.String`](/pkg/crypto/x509/pkix#Name.String)) hiện hiển thị các
giá trị thuộc tính có kiểu chuỗi dưới dạng chuỗi ngay cả khi OID của thuộc tính không được nhận dạng.
Trước đây, các giá trị như vậy luôn được mã hóa thập lục phân ở dạng DER.
Xem [#33093](/issue/33093).

#### [`database/sql`](/pkg/database/sql/)

<!-- 6-stdlib/99-minor/database/sql/67546.md -->

Hàm mới [`ConvertAssign`](/pkg/database/sql#ConvertAssign) cung cấp cho trình điều khiển cơ sở dữ liệu quyền truy cập
vào các chuyển đổi kiểu được thực hiện bởi [`Rows.Scan`](/pkg/database/sql#Rows.Scan).

#### [`database/sql/driver`](/pkg/database/sql/driver/)

<!-- 6-stdlib/99-minor/database/sql/driver/67546.md -->

Các driver có thể triển khai interface mới [`RowsColumnScanner`](/pkg/database/sql/driver#RowsColumnScanner)
để quét trực tiếp vào các đích do người dùng cung cấp.

#### [`go/constant`](/pkg/go/constant/)

<!-- 6-stdlib/99-minor/go/constant/79042.md -->

Hàm [`StringLen`](/pkg/go/constant#StringLen) mới trả về
độ dài của một chuỗi [`Value`](/pkg/go/constant#Value) mà không
cần xây dựng đầy đủ `Value`.

#### [`go/scanner`](/pkg/go/scanner/)

<!-- 6-stdlib/99-minor/go/scanner/74958.md -->

Scanner hiện cho phép lấy vị trí kết thúc của một token thông qua phương thức [`Scanner.End`](/pkg/go/scanner#Scanner.End) mới.

#### [`go/token`](/pkg/go/token/)

<!-- 6-stdlib/99-minor/go/token/76285.md -->

[`File`](/pkg/go/token#File) hiện có phương thức `String`.

#### [`go/types`](/pkg/go/types/)

<!-- 6-stdlib/99-minor/go/types/69420.md -->

Kiểu [`Hasher`](/pkg/go/types#Hasher) là một triển khai của
[`maphash.Hasher`](/pkg/maphash#Hasher) cho các [`Type`](/pkg/go/types#Type)s
tôn trọng quan hệ tương đương [`Identical`](/pkg/go/types#Identical), cho phép `Types`
được sử dụng trong bảng băm và các cấu trúc dữ liệu tương tự.
[`HasherIgnoreTags`](/pkg/go/types#HasherIgnoreTags) là hasher tương tự cho [`IdenticalIgnoreTags`](/pkg/go/types#IdenticalIgnoreTags).

<!-- 6-stdlib/99-minor/go/types/76472.md -->
<!-- CL 736441 -->

Thiết lập `gotypesalias` `GODEBUG` (được thêm vào trong [Go 1.22](/doc/godebug#go-122))
đã bị xóa vĩnh viễn và gói [`go/types`](/pkg/go/types)
hiện luôn tạo một nút kiểu [`Alias`](/pkg/go/types#Alias) cho
[khai báo alias](/ref/spec#Alias_declarations) bất kể các thiết lập `GODEBUG`.

<!-- 6-stdlib/99-minor/go/types/79287.md -->
<!-- nothing to see here but some String methods -->

#### [`hash/maphash`](/pkg/hash/maphash/)

<!-- 6-stdlib/99-minor/hash/maphash/70471.md -->

Kiểu interface [`Hasher`](/pkg/hash/maphash#Hasher) định nghĩa hợp đồng giữa các giá trị của một
kiểu cụ thể và các cấu trúc dữ liệu dựa trên hash trong tương lai như bảng
băm và bộ lọc Bloom; xem [#70471](/issue/70471).

Kiểu [`ComparableHasher`](/pkg/hash/maphash#ComparableHasher) cung cấp một
triển khai thuận tiện của [`Hasher`](/pkg/hash/maphash#Hasher) cho các
kiểu có thể so sánh trong đó phương thức `Equal` được định nghĩa là `==`.

#### [`math/big`](/pkg/math/big/)

<!-- 6-stdlib/99-minor/math/big/76821.md -->
<!-- go.dev/issue/76821 -->

[`Int`](/pkg/math/big#Int) hiện có phương thức [`Divide`](/pkg/math/big#Int.Divide)
để tính thương và phần dư của hai giá trị [`Int`](/pkg/math/big#Int).
Phương thức này hỗ trợ các chế độ làm tròn [`Trunc`](/pkg/math/big#Trunc), [`Floor`](/pkg/math/big#Floor),
[`Round`](/pkg/math/big#Round) và [`Ceil`](/pkg/math/big#Ceil).

#### [`math/rand/v2`](/pkg/math/rand/v2/) {#rand}

<!-- 6-stdlib/99-minor/math/rand/v2/77853.md -->

[`Rand`](/pkg/math/rand/v2#Rand) hiện hỗ trợ phương thức generic
[`N`](/pkg/math/rand/v2#Rand.N),
phù hợp với hành vi của hàm [`N`](/pkg/math/rand/v2#N) cấp cao nhất.

#### [`net`](/pkg/net/)

<!-- 6-stdlib/99-minor/net/78137.md -->

Các phương thức đọc của [`UnixConn`](/pkg/net#UnixConn) hiện trả về trực tiếp [`io.EOF`](/pkg/io#EOF) thay vì bọc nó trong [`net.OpError`](/pkg/net#OpError) khi thao tác đọc bên dưới trả về EOF.

#### [`net/http`](/pkg/net/http/)

<!-- 6-stdlib/99-minor/net/http/21753.md -->

[`Transport`](/pkg/net/http#Transport) và [`Server`](/pkg/net/http#Server) hỗ trợ thương lượng giao thức TLS ALPN trên các kết nối [`net.Conn`](/pkg/net#Conn) do người dùng cung cấp, triển khai phương thức
`ConnectionState() tls.ConnectionState`.

<!-- 6-stdlib/99-minor/net/http/75500.md -->

Máy chủ HTTP/2 hiện chấp nhận các tín hiệu ưu tiên từ client, như được định nghĩa trong RFC 9218,
cho phép nó ưu tiên phục vụ các stream HTTP/2 có mức ưu tiên cao hơn. Nếu
muốn dùng hành vi cũ, trong đó các stream được phục vụ theo cách xoay vòng
bất kể mức ưu tiên, có thể đặt [`Server.DisableClientPriority`](/pkg/net/http#Server.DisableClientPriority) thành `true`.

<!-- 6-stdlib/99-minor/net/http/77370.md -->

[`Response.Body`](/pkg/net/http#Response.Body) HTTP/1 hiện tự động xả mọi nội dung chưa đọc khi được
đóng, lên đến một giới hạn thận trọng, để cho phép tái sử dụng kết nối tốt hơn. Với hầu hết
chương trình, thay đổi này không có tác động hoặc sẽ cải thiện hiệu năng.
Trong một số trường hợp hiếm, các chương trình không hưởng lợi từ việc tái sử dụng kết nối có thể
gặp suy giảm hiệu năng nếu trước đây chúng cho phép một lượng lớn kết nối nhàn rỗi
tồn tại không đúng cách; thường là do đặt
[`Transport.MaxIdleConns`](/pkg/net/http#Transport.MaxIdleConns) thành `0` hoặc sử dụng các [`Client`](/pkg/net/http#Client) khác nhau cho các
yêu cầu khác nhau, qua đó bỏ qua giới hạn của [`Transport.MaxIdleConns`](/pkg/net/http#Transport.MaxIdleConns). Trong các trường hợp này,
đặt [`Transport.DisableKeepAlives`](/pkg/net/http#Transport.DisableKeepAlives) thành `true` sẽ tắt việc tái sử dụng kết nối.
Tuy nhiên, suy giảm hiệu năng như vậy thường cho thấy cấu hình
hoặc cách sử dụng [`Transport`](/pkg/net/http#Transport) hoặc [`Client`](/pkg/net/http#Client) không phù hợp ngay từ đầu, và việc xem xét kỹ hơn
có thể sẽ hữu ích.

<!-- go.dev/issue/79936 -->

Trường mới [`Server.MaxHeaderValueCount`](/pkg/net/http#Server.MaxHeaderValueCount)
cho phép các máy chủ HTTP kiểm soát số lượng giá trị header mà chúng
sẵn sàng chấp nhận. Nếu không được đặt,
[`DefaultMaxHeaderValueCount`](/pkg/net/http#DefaultMaxHeaderValueCount) sẽ được
sử dụng.

#### [`net/http/httptest`](/pkg/net/http/httptest/)

<!-- 6-stdlib/99-minor/net/http/httptest/76608.md -->

The new [`NewTestServer`](/pkg/net/http/httptest#NewTestServer) function
creates a [`Server`](/pkg/net/http/httptest#Server) configured to use an in-memory
fake network suitable for use with the [`testing/synctest`](/pkg/testing/synctest) package.

#### [`net/url`](/pkg/net/url/)

<!-- 6-stdlib/99-minor/net/url/73450.md -->

The new [`URL.Clone`](/pkg/net/url#URL.Clone) method creates a deep copy of a URL.
The new [`Values.Clone`](/pkg/net/url#Values.Clone) method creates a deep copy of Values.

#### [`runtime/secret`](/pkg/runtime/secret/)

Goroutines that are created while in [secret mode](/pkg/runtime/secret#Do)
will now themselves execute in secret mode.

#### [`strings`](/pkg/strings/)

<!-- 6-stdlib/99-minor/strings/71151.md -->

The new [`CutLast`](/pkg/strings#CutLast) function slices a string
around the last occurrence of a separator.
It can replace and simplify some common uses of [`LastIndex`](/pkg/strings#LastIndex).

#### [`syscall`](/pkg/syscall/)

<!-- CL 750680 -->

On Plan 9, the [`Errno`](/pkg/syscall#Errno) type is now defined and
implements the `error` interface, as on other platforms. Plan 9 system
calls return `ErrorString` values, so `Errno` is never returned by this
package on Plan 9. It is defined so that portable code referring to
`syscall.Errno` builds on Plan 9 without build constraints.

#### [`testing/synctest`](/pkg/testing/synctest/)

<!-- 6-stdlib/99-minor/testing/synctest/77169.md -->

The new [`Sleep`](/pkg/testing/synctest#Sleep) helper function combines [`time.Sleep`](/pkg/time#Sleep) and [`synctest.Wait`](/pkg/testing/synctest#Wait).

#### [`unicode`](/pkg/unicode/)

<!-- 6-stdlib/99-minor/unicode/77266.md -->

The unicode package and associated support throughout the system have been upgraded from Unicode 15 to Unicode 17.
See the [Unicode 16.0.0](https://www.unicode.org/versions/Unicode16.0.0/) and
[Unicode 17.0.0](https://www.unicode.org/versions/Unicode17.0.0/)
release notes for information about the changes.

## Cổng {#ports}

### Darwin {#darwin}

<!-- go.dev/issue/75836 -->

Như đã [thông báo](go1.26#darwin) trong ghi chú bản phát hành Go 1.26,
Go 1.27 yêu cầu macOS 13 Ventura hoặc mới hơn;
hỗ trợ cho các phiên bản trước đã bị ngừng.

### PowerPC {#ppc64}

<!-- go.dev/issue/76244 -->

Trên cổng PowerPC 64-bit big-endian trên Linux (`GOOS=linux` `GOARCH=ppc64`),
chuỗi công cụ Go hiện tạo các binary sử dụng
ABI hệ thống ELFv2.
Hỗ trợ ELFv2 yêu cầu nhân Linux 3.13 hoặc mới hơn.
RHEL7 đã đưa ngược hỗ trợ này vào nhân 3.10 của nó.

Cgo, các tệp thực thi độc lập vị trí (PIE), và liên kết bên ngoài
hiện đã được hỗ trợ.
Việc sử dụng các tính năng này yêu cầu runtime tương thích với ELFv2
(libc và tất cả thư viện được liên kết và tải).

Đối với các chương trình không sử dụng cgo, chuỗi công cụ Go vẫn
tạo các binary tĩnh với liên kết nội bộ theo mặc định.
Đối với các chương trình có tùy chọn cgo, nếu cần một binary tĩnh thuần Go,
bạn có thể đặt biến môi trường `CGO_ENABLED=0`
khi chạy `go build`.
