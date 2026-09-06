---
title: Ghi chú bản phát hành Go 1.25
---

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu về Go 1.25 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.25, ra mắt vào [Tháng 8 năm 2025](/doc/devel/release#go1.25.0), sáu tháng sau [Go 1.24](/doc/go1.24).
Phần lớn thay đổi của bản phát hành này nằm trong việc triển khai toolchain, runtime và các thư viện.
Như thường lệ, bản phát hành vẫn duy trì cam kết tương thích của Go 1.
Chúng tôi kỳ vọng hầu hết chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

## Các thay đổi đối với ngôn ngữ {#language}

<!-- go.dev/issue/70128 -->

Không có thay đổi nào về ngôn ngữ ảnh hưởng đến các chương trình Go trong Go 1.25.
Tuy nhiên, trong [đặc tả ngôn ngữ](/ref/spec), khái niệm kiểu cốt lõi đã được loại bỏ để thay thế bằng phần mô tả riêng.
Xem [bài đăng trên blog](/blog/coretypes) tương ứng để biết thêm thông tin.

## Công cụ {#tools}

### Lệnh Go {#go-command}

Tùy chọn `-asan` của `go build` hiện mặc định thực hiện phát hiện rò rỉ khi chương trình kết thúc.
Tùy chọn này sẽ báo lỗi nếu bộ nhớ được C cấp phát không được giải phóng và không được tham chiếu bởi bất kỳ bộ nhớ nào khác được C hoặc Go cấp phát.
Các báo cáo lỗi mới này có thể được vô hiệu hóa bằng cách đặt `ASAN_OPTIONS=detect_leaks=0` trong môi trường khi chạy chương trình.

<!-- go.dev/issue/71867 -->
Bản phân phối Go sẽ bao gồm ít binary công cụ được xây dựng sẵn hơn. Các binary cốt lõi của toolchain như trình biên dịch và trình liên kết vẫn sẽ được đưa vào, nhưng các công cụ không được gọi bởi các thao tác build hoặc test sẽ được xây dựng và chạy bởi `go tool` khi cần.

<!-- go.dev/issue/42965 -->
[Chỉ thị](/ref/mod#go-mod-file-ignore) `ignore` mới của `go.mod` có thể được dùng để chỉ định các thư mục mà lệnh `go` nên bỏ qua. Các tệp trong những thư mục này và các thư mục con của chúng sẽ bị lệnh `go` bỏ qua khi khớp các mẫu gói, chẳng hạn như `all` hoặc `./...`, nhưng vẫn sẽ được đưa vào các tệp zip của module.

<!-- go.dev/issue/68106 -->
Tùy chọn `-http` mới của `go doc` sẽ khởi động một máy chủ tài liệu hiển thị tài liệu cho đối tượng được yêu cầu và mở tài liệu trong một cửa sổ trình duyệt.

<!-- go.dev/issue/69712 -->

Tùy chọn `go version -m -json` mới sẽ in các biểu diễn JSON của các cấu trúc `runtime/debug.BuildInfo` được nhúng trong các tệp binary Go được chỉ định.

<!-- go.dev/issue/34055 -->
Lệnh `go` hiện hỗ trợ sử dụng một thư mục con của một repository làm đường dẫn cho gốc module khi [giải quyết đường dẫn module](/ref/mod#vcs-find) bằng cú pháp
`<meta name="go-import" content="root-path vcs repo-url subdir">` để chỉ ra rằng `root-path` tương ứng với `subdir` của `repo-url` sử dụng hệ thống kiểm soát phiên bản `vcs`.

<!-- go.dev/issue/71294 -->

Mẫu gói `work` mới khớp với tất cả gói trong các module work (trước đây được gọi là main): hoặc module work duy nhất trong chế độ module hoặc tập hợp các module workspace trong chế độ workspace.

<!-- go.dev/issue/65847 -->

Khi lệnh go cập nhật dòng `go` trong tệp `go.mod` hoặc `go.work`, nó [không còn](/ref/mod#go-mod-file-toolchain) thêm một dòng toolchain chỉ định phiên bản hiện tại của lệnh.

### Vet {#vet}

Lệnh `go vet` bao gồm các bộ phân tích mới:

<!-- go.dev/issue/18022 -->

- [waitgroup](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/waitgroup),
  báo cáo các lệnh gọi sai vị trí tới [`sync.WaitGroup.Add`](/pkg/sync#WaitGroup.Add); và

<!-- go.dev/issue/28308 -->

- [hostport](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/hostport),
  báo cáo việc sử dụng `fmt.Sprintf("%s:%d", host, port)` để
  tạo địa chỉ cho [`net.Dial`](/pkg/net#Dial), vì các địa chỉ này sẽ không hoạt động với
  IPv6; thay vào đó đề xuất sử dụng [`net.JoinHostPort`](/pkg/net#JoinHostPort).

## Runtime {#runtime}

### `GOMAXPROCS` nhận biết container

<!-- go.dev/issue/73193 -->

Hành vi mặc định của `GOMAXPROCS` đã thay đổi. Trong các phiên bản Go trước,
`GOMAXPROCS` mặc định là số CPU logic có sẵn khi khởi động
([`runtime.NumCPU`](/pkg/runtime#NumCPU)). Go 1.25 giới thiệu hai thay đổi:

1. Trên Linux, runtime xem xét giới hạn băng thông CPU của cgroup
   chứa tiến trình, nếu có. Nếu giới hạn băng thông CPU thấp hơn số
   CPU logic có sẵn, `GOMAXPROCS` sẽ mặc định theo giới hạn thấp hơn
   đó. Trong các hệ thống runtime container như Kubernetes, giới hạn băng
   thông CPU của cgroup thường tương ứng với tùy chọn "CPU limit". Runtime Go
   không xem xét tùy chọn "CPU requests".

2. Trên tất cả hệ điều hành, runtime định kỳ cập nhật `GOMAXPROCS` nếu số
   CPU logic có sẵn hoặc giới hạn băng thông CPU của cgroup thay đổi.

Cả hai hành vi này được tự động vô hiệu hóa nếu `GOMAXPROCS` được đặt
thủ công thông qua biến môi trường `GOMAXPROCS` hoặc một lệnh gọi tới
[`runtime.GOMAXPROCS`](/pkg/runtime#GOMAXPROCS). Chúng cũng có thể được vô hiệu hóa
một cách rõ ràng bằng các thiết lập [GODEBUG](/doc/godebug)
`containermaxprocs=0` và `updatemaxprocs=0`,
tương ứng.

Để hỗ trợ việc đọc các giới hạn cgroup đã được cập nhật, runtime sẽ giữ các
bộ mô tả tệp đã lưu trong bộ nhớ đệm cho các tệp cgroup trong suốt vòng đời
của tiến trình.

### Bộ gom rác thử nghiệm mới

<!-- go.dev/issue/73581 -->

Một bộ gom rác mới hiện có sẵn dưới dạng thử nghiệm. Thiết kế của bộ gom
rác này cải thiện hiệu năng đánh dấu và quét các đối tượng nhỏ thông qua
tính cục bộ tốt hơn và khả năng mở rộng theo CPU. Kết quả benchmark thay đổi,
nhưng chúng tôi kỳ vọng mức giảm chi phí thu gom rác từ 10—40% trong các
chương trình thực tế sử dụng nhiều bộ gom rác.

Bộ gom rác mới có thể được bật bằng cách đặt `GOEXPERIMENT=greenteagc`
khi build. Chúng tôi kỳ vọng thiết kế này sẽ tiếp tục phát triển và cải thiện.
Vì mục đích đó, chúng tôi khuyến khích các nhà phát triển Go dùng thử và
phản hồi lại trải nghiệm của họ. Xem [vấn đề trên GitHub](/issue/73581) để biết
thêm chi tiết về thiết kế và hướng dẫn chia sẻ phản hồi.

### Trace flight recorder

<!-- go.dev/issue/63185 -->

[Runtime execution traces](/pkg/runtime/trace) từ lâu đã cung cấp một cách mạnh mẽ nhưng tốn kém để hiểu và gỡ lỗi hành vi cấp thấp của một ứng dụng. Tuy nhiên, do kích thước của chúng và chi phí liên tục ghi một execution trace, chúng thường không thực tế khi dùng để gỡ lỗi các sự kiện hiếm gặp.

API mới [`runtime/trace.FlightRecorder`](/pkg/runtime/trace#FlightRecorder) cung cấp một cách nhẹ để thu thập một runtime execution trace bằng cách liên tục ghi trace vào một ring buffer trong bộ nhớ. Khi một sự kiện quan trọng xảy ra, chương trình có thể gọi [`FlightRecorder.WriteTo`](/pkg/runtime/trace#FlightRecorder.WriteTo) để chụp ảnh nhanh vài giây cuối cùng của trace vào một tệp. Cách tiếp cận này tạo ra một trace nhỏ hơn nhiều bằng cách cho phép các ứng dụng chỉ thu thập những trace quan trọng.

Khoảng thời gian và lượng dữ liệu được thu thập bởi [`FlightRecorder`](/pkg/runtime/trace#FlightRecorder) có thể được cấu hình trong [`FlightRecorderConfig`](/pkg/runtime/trace#FlightRecorderConfig).

### Thay đổi đối với đầu ra panic không được xử lý

<!-- go.dev/issue/71517 -->

Thông báo được in khi một chương trình thoát do một panic không được xử lý, vốn đã được recover và panic lại, không còn lặp lại nội dung của giá trị panic.

Trước đây, một chương trình panic với `panic("PANIC")`, recover panic đó, sau đó panic lại với giá trị ban đầu sẽ in:

    panic: PANIC [recovered]
      panic: PANIC

Chương trình này giờ sẽ in:

    panic: PANIC [recovered, repanicked]

### Tên VMA trên Linux

<!-- go.dev/issue/71546 -->

Trên các hệ thống Linux có kernel hỗ trợ tên vùng bộ nhớ ảo ẩn danh (VMA)
(`CONFIG_ANON_VMA_NAME`), runtime Go sẽ chú thích các ánh xạ bộ nhớ ẩn danh bằng ngữ cảnh về mục đích của chúng. Ví dụ: `[anon: Go: heap]` cho bộ nhớ heap. Có thể tắt tính năng này bằng [thiết lập GODEBUG](/doc/godebug) `decoratemappings=0`.

## Trình biên dịch {#compiler}

### Lỗi con trỏ `nil`

<!-- https://go.dev/issue/72860, CL 657715 -->

Bản phát hành này sửa một [lỗi trình biên dịch](/issue/72860), được giới thiệu trong Go 1.21, có thể trì hoãn không chính xác các kiểm tra con trỏ nil. Các chương trình như sau, vốn trước đây chạy thành công (không đúng), giờ sẽ (đúng) panic với ngoại lệ con trỏ nil:

```
package main

import "os"

func main() {
	f, err := os.Open("nonExistentFile")
	name := f.Name()
	if err != nil {
		return
	}
	println(name)
}
```

Chương trình này không đúng vì nó sử dụng kết quả của `os.Open` trước khi kiểm tra lỗi. Nếu `err` khác nil, thì kết quả `f` có thể là nil, trong trường hợp đó `f.Name()` sẽ panic. Tuy nhiên, trong các phiên bản Go từ 1.21 đến 1.24, trình biên dịch đã trì hoãn không chính xác việc kiểm tra nil cho đến *sau* khi kiểm tra lỗi, khiến chương trình chạy thành công, vi phạm đặc tả Go. Trong Go 1.25, chương trình sẽ không còn chạy thành công nữa. Nếu thay đổi này ảnh hưởng đến mã của bạn, giải pháp là đặt kiểm tra lỗi khác nil sớm hơn trong mã, tốt nhất là ngay sau câu lệnh tạo ra lỗi.

### Hỗ trợ DWARF5

<!-- https://go.dev/issue/26379 -->

Trình biên dịch và trình liên kết trong Go 1.25 hiện tạo thông tin gỡ lỗi
bằng [DWARF phiên bản 5](https://dwarfstd.org/dwarf5std.html). Phiên bản DWARF
mới hơn làm giảm không gian cần thiết cho thông tin gỡ lỗi trong các
binary Go, đồng thời giảm thời gian liên kết, đặc biệt đối với các
binary Go lớn.
Có thể tắt việc tạo DWARF 5 bằng cách đặt biến môi trường
`GOEXPERIMENT=nodwarf5` tại thời điểm build
(fallback này có thể bị loại bỏ trong một bản phát hành Go trong tương lai).

### Slice nhanh hơn

<!-- CLs 653856, 657937, 663795, 664299 -->

Trình biên dịch hiện có thể cấp phát vùng lưu trữ phía sau cho slice trên
stack trong nhiều trường hợp hơn, giúp cải thiện hiệu năng. Thay đổi này có
khả năng làm tăng ảnh hưởng của việc sử dụng
[unsafe.Pointer](/pkg/unsafe#Pointer) không đúng, xem ví dụ [issue
73199](/issue/73199). Để tìm ra các vấn đề này, có thể sử dụng
[công cụ bisect](https://pkg.go.dev/golang.org/x/tools/cmd/bisect) để
tìm vùng cấp phát gây ra sự cố bằng cách sử dụng cờ
`-compile=variablemake`. Tất cả các vùng cấp phát stack mới như vậy cũng có
thể được tắt bằng cách sử dụng `-gcflags=all=-d=variablemakehash=n`.

## Trình liên kết {#linker}

<!-- CL 660996 -->

Trình liên kết hiện chấp nhận tùy chọn dòng lệnh `-funcalign=N`, tùy chọn này
chỉ định sự căn chỉnh của các điểm vào hàm.
Giá trị mặc định phụ thuộc vào nền tảng và không thay đổi trong
bản phát hành này.

## Thư viện chuẩn {#library}

### Gói testing/synctest mới

<!-- go.dev/issue/67434, go.dev/issue/73567 -->
Gói [`testing/synctest`](/pkg/testing/synctest) mới
cung cấp hỗ trợ để kiểm thử mã đồng thời.

Hàm [`Test`](/pkg/testing/synctest#Test) chạy một hàm kiểm thử trong một
"bong bóng" được cô lập. Bên trong bong bóng, thời gian được ảo hóa: các hàm
của gói [`time`](/pkg/time) hoạt động trên một đồng hồ giả và đồng hồ tiến
lên tức thời nếu tất cả goroutine trong bong bóng đều bị chặn.

Hàm [`Wait`](/pkg/testing/synctest#Wait) chờ tất cả goroutine trong
bong bóng hiện tại bị chặn.

Gói này lần đầu có trong Go 1.24 dưới dạng `GOEXPERIMENT=synctest`, với
API hơi khác một chút. Thử nghiệm này hiện đã được đưa vào trạng thái
khả dụng chung. API cũ vẫn còn nếu đặt `GOEXPERIMENT=synctest`,
nhưng sẽ bị loại bỏ trong Go 1.26.

### Gói thử nghiệm mới encoding/json/v2 {#json_v2}

Go 1.25 bao gồm một triển khai JSON thử nghiệm mới,
có thể được bật bằng cách đặt biến môi trường
`GOEXPERIMENT=jsonv2` tại thời điểm build.

Khi được bật, có hai gói mới khả dụng:
- Gói [`encoding/json/v2`](/pkg/encoding/json/v2) là
  một bản sửa đổi lớn của gói `encoding/json`.
- Gói [`encoding/json/jsontext`](/pkg/encoding/json/jsontext)
  cung cấp xử lý cấp thấp hơn cho cú pháp JSON.

Ngoài ra, khi GOEXPERIMENT "jsonv2" được bật:
- Gói [`encoding/json`](/pkg/encoding/json)
  sử dụng triển khai JSON mới.
  Hành vi marshaling và unmarshaling không bị ảnh hưởng,
  nhưng nội dung văn bản của các lỗi được trả về bởi hàm trong gói có thể thay đổi.
- Gói [`encoding/json`](/pkg/encoding/json) chứa
  một số tùy chọn mới có thể được sử dụng
  để cấu hình marshaler và unmarshaler.

Triển khai mới hoạt động tốt hơn đáng kể so với
triển khai hiện có trong nhiều trường hợp. Nhìn chung,
hiệu năng encoding là tương đương giữa hai triển khai
và decoding nhanh hơn đáng kể trong triển khai mới.
Xem kho lưu trữ [github.com/go-json-experiment/jsonbench](https://github.com/go-json-experiment/jsonbench)
để có phân tích chi tiết hơn.

Xem [vấn đề đề xuất](/issue/71497) để biết thêm chi tiết.

Chúng tôi khuyến khích người dùng của [`encoding/json`](/pkg/encoding/json) kiểm thử
chương trình của họ với `GOEXPERIMENT=jsonv2` được bật để giúp phát hiện
bất kỳ vấn đề tương thích nào với triển khai mới.

Chúng tôi kỳ vọng thiết kế của [`encoding/json/v2`](/pkg/encoding/json/v2)
sẽ tiếp tục phát triển. Chúng tôi khuyến khích các nhà phát triển dùng thử
API mới và cung cấp phản hồi về [vấn đề đề xuất](/issue/71497).

### Các thay đổi nhỏ đối với thư viện {#minor_library_changes}

#### [`archive/tar`](/pkg/archive/tar/)

Triển khai [`Writer.AddFS`](/pkg/archive/tar#Writer.AddFS) hiện hỗ trợ các liên kết tượng trưng
cho những hệ thống tệp triển khai [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS).

#### [`encoding/asn1`](/pkg/encoding/asn1/)

[`Unmarshal`](/pkg/encoding/asn1#Unmarshal) và [`UnmarshalWithParams`](/pkg/encoding/asn1#UnmarshalWithParams)
hiện phân tích cú pháp các kiểu ASN.1 T61String và BMPString nhất quán hơn. Điều này có thể
khiến một số mã hóa không đúng định dạng trước đây vẫn được chấp nhận giờ đây bị từ chối.

#### [`crypto`](/pkg/crypto/)

[`MessageSigner`](/pkg/crypto#MessageSigner) là một interface ký mới có thể được các bộ ký triển khai nếu muốn tự băm thông điệp cần ký. Một hàm mới cũng được giới thiệu, [`SignMessage`](/pkg/crypto#SignMessage), hàm này cố gắng nâng cấp một interface [`Signer`](/pkg/crypto#Signer) thành [`MessageSigner`](/pkg/crypto#MessageSigner), sử dụng phương thức [`MessageSigner.SignMessage`](/pkg/crypto#MessageSigner.SignMessage) nếu thành công, và [`Signer.Sign`](/pkg/crypto#Signer.Sign) nếu không. Điều này có thể được sử dụng khi mã muốn hỗ trợ cả [`Signer`](/pkg/crypto#Signer) và [`MessageSigner`](/pkg/crypto#MessageSigner).

Việc thay đổi thiết lập `fips140` trong [GODEBUG setting](/doc/godebug) sau khi chương trình đã khởi động giờ đây không còn tác dụng.
Trước đây, điều này được ghi nhận là không được phép và có thể gây panic nếu bị thay đổi.

SHA-1, SHA-256 và SHA-512 giờ đây chậm hơn trên amd64 khi không có các lệnh AVX2.
Tất cả bộ xử lý máy chủ (và hầu hết các bộ xử lý khác) được sản xuất từ năm 2015 đều hỗ trợ AVX2.

#### [`crypto/ecdsa`](/pkg/crypto/ecdsa/)

Các hàm và phương thức mới [`ParseRawPrivateKey`](/pkg/crypto/ecdsa#ParseRawPrivateKey),
[`ParseUncompressedPublicKey`](/pkg/crypto/ecdsa#ParseUncompressedPublicKey),
[`PrivateKey.Bytes`](/pkg/crypto/ecdsa#PrivateKey.Bytes) và
[`PublicKey.Bytes`](/pkg/crypto/ecdsa#PublicKey.Bytes) triển khai các mã hóa cấp thấp, thay thế nhu cầu sử dụng các hàm và phương thức của [`crypto/elliptic`](/pkg/crypto/elliptic) hoặc [`math/big`](/pkg/math/big).

Khi chế độ FIPS 140-3 được bật, việc ký giờ đây nhanh hơn gấp bốn lần, tương đương với hiệu năng của chế độ không phải FIPS.

#### [`crypto/ed25519`](/pkg/crypto/ed25519/)

Khi chế độ FIPS 140-3 được bật, việc ký giờ đây nhanh hơn gấp bốn lần, tương đương với hiệu năng của chế độ không phải FIPS.

#### [`crypto/elliptic`](/pkg/crypto/elliptic/)

Các phương thức ẩn và không được ghi nhận `Inverse` và `CombinedMult` trên một số triển khai [`Curve`](/pkg/crypto/elliptic#Curve) đã bị xóa.

#### [`crypto/rsa`](/pkg/crypto/rsa/)

[`PublicKey`](/pkg/crypto/rsa#PublicKey) không còn tuyên bố rằng giá trị modulus được xử lý như bí mật. [`VerifyPKCS1v15`](/pkg/crypto/rsa#VerifyPKCS1v15) và [`VerifyPSS`](/pkg/crypto/rsa#VerifyPSS) trước đây đã cảnh báo rằng mọi đầu vào đều là công khai và có thể bị lộ, đồng thời có các tấn công toán học có thể khôi phục modulus từ các giá trị công khai khác.

Việc tạo khóa giờ đây nhanh hơn gấp ba lần.

#### [`crypto/sha1`](/pkg/crypto/sha1/)

Việc băm hiện nhanh hơn gấp hai lần trên amd64 khi có các chỉ thị SHA-NI.

#### [`crypto/sha3`](/pkg/crypto/sha3/)

Phương thức mới [`SHA3.Clone`](/pkg/crypto/sha3#SHA3.Clone) triển khai [`hash.Cloner`](/pkg/hash#Cloner).

Việc băm hiện nhanh hơn gấp hai lần trên các bộ xử lý Apple M.

#### [`crypto/tls`](/pkg/crypto/tls/)

Trường mới [`ConnectionState.CurveID`](/pkg/crypto/tls#ConnectionState.CurveID) cung cấp cơ chế trao đổi khóa được dùng để thiết lập kết nối.

Callback mới [`Config.GetEncryptedClientHelloKeys`](/pkg/crypto/tls#Config.GetEncryptedClientHelloKeys) có thể được dùng để thiết lập các [`EncryptedClientHelloKey`](/pkg/crypto/tls#EncryptedClientHelloKey) mà máy chủ sử dụng khi máy khách gửi phần mở rộng Encrypted Client Hello.

Các thuật toán chữ ký SHA-1 hiện không còn được cho phép trong các quá trình bắt tay TLS 1.2, theo [RFC 9155](https://www.rfc-editor.org/rfc/rfc9155.html).
Chúng có thể được bật lại bằng [thiết lập GODEBUG](/doc/godebug) `tlssha1=1`.

Khi [chế độ FIPS 140-3](/doc/security/fips140) được bật, Extended Master Secret hiện là bắt buộc trong TLS 1.2, và Ed25519 cùng X25519MLKEM768 hiện được cho phép.

Các máy chủ TLS hiện ưu tiên phiên bản giao thức được hỗ trợ cao nhất, ngay cả khi đó không phải là phiên bản giao thức được máy khách ưu tiên nhất.

<!-- CL 687855 -->
Cả máy khách và máy chủ TLS hiện nghiêm ngặt hơn trong việc tuân theo đặc tả và từ chối hành vi không đúng đặc tả. Các kết nối với các đối tác tuân thủ sẽ không bị ảnh hưởng.

#### [`crypto/x509`](/pkg/crypto/x509/)

[`CreateCertificate`](/pkg/crypto/x509#CreateCertificate), [`CreateCertificateRequest`](/pkg/crypto/x509#CreateCertificateRequest), và [`CreateRevocationList`](/pkg/crypto/x509#CreateRevocationList) hiện có thể chấp nhận interface ký [`crypto.MessageSigner`](/pkg/crypto#MessageSigner) cũng như [`crypto.Signer`](/pkg/crypto#Signer). Điều này cho phép các hàm này sử dụng các signer triển khai interface ký "one-shot", trong đó việc băm được thực hiện như một phần của thao tác ký thay vì do bên gọi thực hiện.

[`CreateCertificate`](/pkg/crypto/x509#CreateCertificate) hiện sử dụng SHA-256 đã cắt ngắn để điền `SubjectKeyId` nếu trường này bị thiếu.
[Thiết lập GODEBUG](/doc/godebug) `x509sha256skid=0` sẽ khôi phục về SHA-1.

[`ParseCertificate`](/pkg/crypto/x509#ParseCertificate) hiện từ chối các chứng chỉ chứa phần mở rộng BasicConstraints có pathLenConstraint âm.

[`ParseCertificate`](/pkg/crypto/x509#ParseCertificate) hiện xử lý các chuỗi được mã hóa bằng các kiểu ASN.1 T61String và BMPString nhất quán hơn. Điều này có thể khiến một số mã hóa không đúng định dạng trước đây vẫn được chấp nhận nay bị từ chối.

#### [`debug/elf`](/pkg/debug/elf/)

Gói [`debug/elf`](/pkg/debug/elf) bổ sung hai hằng số mới:
- [`PT_RISCV_ATTRIBUTES`](/pkg/debug/elf#PT_RISCV_ATTRIBUTES)
- [`SHT_RISCV_ATTRIBUTES`](/pkg/debug/elf#SHT_RISCV_ATTRIBUTES)
  để phân tích cú pháp ELF của RISC-V.

#### [`go/ast`](/pkg/go/ast/)

Các hàm [`FilterPackage`](/pkg/ast#FilterPackage), [`PackageExports`](/pkg/ast#PackageExports) và
[`MergePackageFiles`](/pkg/ast#MergePackageFiles), cùng với kiểu [`MergeMode`](/pkg/go/ast#MergeMode) và các
hằng số của nó, đều đã bị loại bỏ sử dụng, vì chúng chỉ được dùng với
cơ chế [`Object`](/pkg/ast#Object) và [`Package`](/pkg/ast#Package) đã lỗi thời từ lâu.

Hàm mới [`PreorderStack`](/pkg/go/ast#PreorderStack), giống như [`Inspect`](/pkg/go/ast#Inspect), duyệt qua cây cú pháp
và cung cấp quyền kiểm soát việc đi xuống các cây con, nhưng để tiện lợi
nó cũng cung cấp ngăn xếp của các nút bao quanh tại mỗi
điểm.

#### [`go/parser`](/pkg/go/parser/)

Hàm [`ParseDir`](/pkg/go/parser#ParseDir) đã bị loại bỏ sử dụng.

#### [`go/token`](/pkg/go/token/)

Phương thức mới [`FileSet.AddExistingFiles`](/pkg/go/token#FileSet.AddExistingFiles) cho phép các
[`File`](/pkg/go/token#File) hiện có được thêm vào một [`FileSet`](/pkg/go/token#FileSet),
hoặc một [`FileSet`](/pkg/go/token#FileSet) được tạo cho một tập hợp
[`File`](/pkg/go/token#File) tùy ý, giúp giảm bớt các vấn đề liên quan đến một
[`FileSet`](/pkg/go/token#FileSet) toàn cục duy nhất trong các ứng dụng chạy lâu dài.

#### [`go/types`](/pkg/go/types/)

[`Var`](/pkg/go/types#Var) hiện có phương thức [`Var.Kind`](/pkg/go/types#Var.Kind) phân loại biến thành một trong
các loại: cấp gói, receiver, tham số, kết quả, biến cục bộ hoặc
trường của struct.

Hàm mới [`LookupSelection`](/pkg/go/types#LookupSelection) tra cứu trường hoặc phương thức của một
tên và kiểu receiver đã cho, giống như hàm hiện có [`LookupFieldOrMethod`](/pkg/go/types#LookupFieldOrMethod)
nhưng trả về kết quả dưới dạng một [`Selection`](/pkg/go/types#Selection).

#### [`hash`](/pkg/hash/)

Interface mới [`XOF`](/pkg/hash#XOF) có thể được triển khai bởi các "hàm đầu ra có thể mở rộng", là các hàm băm có độ dài đầu ra tùy ý hoặc không giới hạn như [`SHAKE`](/pkg/crypto/sha3#SHAKE).

Các hàm băm triển khai interface [`Cloner`](/pkg/hash#Cloner) mới có thể trả về bản sao trạng thái của chúng. Tất cả các triển khai [`Hash`](/pkg/hash#Hash) trong thư viện chuẩn hiện đều triển khai [`Cloner`](/pkg/hash#Cloner).

#### [`hash/maphash`](/pkg/hash/maphash/)

Phương thức [`Hash.Clone`](/pkg/hash/maphash#Hash.Clone) mới triển khai [`hash.Cloner`](/pkg/hash#Cloner).

#### [`io/fs`](/pkg/io/fs/)

Một interface [`ReadLinkFS`](/pkg/io/fs#ReadLinkFS) mới cung cấp khả năng đọc các liên kết tượng trưng trong một hệ thống tệp.

#### [`log/slog`](/pkg/log/slog/)

[`GroupAttrs`](/pkg/log/slog#GroupAttrs) tạo một nhóm [`Attr`](/pkg/log/slog#Attr) từ một lát cắt các giá trị [`Attr`](/pkg/log/slog#Attr).

[`Record`](/pkg/log/slog#Record) hiện có phương thức [`Source`](/pkg/log/slog#Record.Source), trả về vị trí nguồn của nó hoặc nil nếu không khả dụng.

#### [`mime/multipart`](/pkg/mime/multipart/)

Hàm trợ giúp mới [`FileContentDisposition`](/pkg/mime/multipart#FileContentDisposition) xây dựng các trường tiêu đề Content-Disposition dạng multipart.

#### [`net`](/pkg/net/)

[`LookupMX`](/pkg/net#LookupMX) và [`Resolver.LookupMX`](/pkg/net#Resolver.LookupMX) hiện trả về các tên DNS trông giống địa chỉ IP hợp lệ, cũng như các tên miền hợp lệ.  
Trước đây, nếu một máy chủ tên trả về một địa chỉ IP dưới dạng tên DNS, [`LookupMX`](/pkg/net#LookupMX) sẽ loại bỏ nó, theo yêu cầu của các RFC.  
Tuy nhiên, trên thực tế các máy chủ tên đôi khi vẫn trả về địa chỉ IP.

Trên Windows, [`ListenMulticastUDP`](/pkg/net#ListenMulticastUDP) hiện hỗ trợ các địa chỉ IPv6.

Trên Windows, hiện có thể chuyển đổi giữa một [`os.File`](/pkg/os#File) và một kết nối mạng. Cụ thể, các hàm [`FileConn`](/pkg/net#FileConn), [`FilePacketConn`](/pkg/net#FilePacketConn), và [`FileListener`](/pkg/net#FileListener) hiện đã được triển khai, và trả về một kết nối mạng hoặc listener tương ứng với một tệp đang mở.  
Tương tự, các phương thức `File` của [`TCPConn`](/pkg/net#TCPConn.File), [`UDPConn`](/pkg/net#UDPConn.File), [`UnixConn`](/pkg/net#UnixConn.File), [`IPConn`](/pkg/net#IPConn.File), [`TCPListener`](/pkg/net#TCPListener.File), và [`UnixListener`](/pkg/net#UnixListener.File) hiện đã được triển khai, và trả về [`os.File`](/pkg/os#File) bên dưới của một kết nối mạng.

#### [`net/http`](/pkg/net/http/)

[`CrossOriginProtection`](/pkg/net/http#CrossOriginProtection) mới triển khai các cơ chế bảo vệ chống lại [Cross-Site Request Forgery (CSRF)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/CSRF) bằng cách từ chối các yêu cầu trình duyệt cross-origin không an toàn.
Nó sử dụng [siêu dữ liệu Fetch của trình duyệt hiện đại](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Site), không yêu cầu token hoặc cookie, đồng thời hỗ trợ các cách bỏ qua dựa trên origin và mẫu.

#### [`os`](/pkg/os/)

Trên Windows, [`NewFile`](/pkg/os#NewFile) hiện hỗ trợ các handle được mở cho I/O bất đồng bộ (nghĩa là,
[`syscall.FILE_FLAG_OVERLAPPED`](/pkg/syscall#FILE_FLAG_OVERLAPPED) được chỉ định trong lời gọi [`syscall.CreateFile`](/pkg/syscall#CreateFile)).
Các handle này được liên kết với cổng hoàn tất I/O của runtime Go,
cung cấp các lợi ích sau cho [`File`](/pkg/os#File) tương ứng:

* Các phương thức I/O ([`File.Read`](/pkg/os#File.Read), [`File.Write`](/pkg/os#File.Write), [`File.ReadAt`](/pkg/os#File.ReadAt), và [`File.WriteAt`](/pkg/os#File.WriteAt)) không chặn một luồng OS.
* Các phương thức deadline ([`File.SetDeadline`](/pkg/os#File.SetDeadline), [`File.SetReadDeadline`](/pkg/os#File.SetReadDeadline), và [`File.SetWriteDeadline`](/pkg/os#File.SetWriteDeadline)) được hỗ trợ.

Cải tiến này đặc biệt hữu ích cho các ứng dụng giao tiếp qua named pipe trên Windows.

Lưu ý rằng một handle chỉ có thể được liên kết với một cổng hoàn tất tại một thời điểm.
Nếu handle được cung cấp cho [`NewFile`](/pkg/os#NewFile) đã được liên kết với một cổng hoàn tất,
[`File`](/pkg/os#File) được trả về sẽ bị hạ cấp xuống chế độ I/O đồng bộ.
Trong trường hợp này, các phương thức I/O sẽ chặn một luồng OS, và các phương thức deadline không có tác dụng.

Các filesystem được trả về bởi [`DirFS`](/pkg/os#DirFS) và [`Root.FS`](/pkg/os#Root.FS) triển khai interface mới [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS).
[`CopyFS`](/pkg/os#CopyFS) hỗ trợ symlink khi sao chép các filesystem triển khai [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS).

Kiểu [`Root`](/pkg/os#Root) hỗ trợ các phương thức bổ sung sau:

* [`Root.Chmod`](/pkg/os#Root.Chmod)
* [`Root.Chown`](/pkg/os#Root.Chown)
* [`Root.Chtimes`](/pkg/os#Root.Chtimes)
* [`Root.Lchown`](/pkg/os#Root.Lchown)
* [`Root.Link`](/pkg/os#Root.Link)
* [`Root.MkdirAll`](/pkg/os#Root.MkdirAll)
* [`Root.ReadFile`](/pkg/os#Root.ReadFile)
* [`Root.Readlink`](/pkg/os#Root.Readlink)
* [`Root.RemoveAll`](/pkg/os#Root.RemoveAll)
* [`Root.Rename`](/pkg/os#Root.Rename)
* [`Root.Symlink`](/pkg/os#Root.Symlink)
* [`Root.WriteFile`](/pkg/os#Root.WriteFile)

<!-- go.dev/issue/73126 is documented as part of 67002 -->

#### [`reflect`](/pkg/reflect/)

Hàm [`TypeAssert`](/pkg/reflect#TypeAssert) mới cho phép chuyển đổi một [`Value`](/pkg/reflect#Value) trực tiếp thành một giá trị Go
có kiểu đã cho. Điều này giống như sử dụng phép khẳng định kiểu trên kết quả của [`Value.Interface`](/pkg/reflect#Value.Interface),
nhưng tránh việc cấp phát bộ nhớ không cần thiết.

#### [`regexp/syntax`](/pkg/regexp/syntax/)

Cú pháp lớp ký tự `\p{name}` và `\P{name}` hiện chấp nhận các tên
Any, ASCII, Assigned, Cn và LC, cũng như các bí danh danh mục Unicode như `\p{Letter}` cho `\pL`.
Theo [Unicode TR18](https://unicode.org/reports/tr18/), chúng cũng hiện sử dụng
tra cứu tên không phân biệt chữ hoa chữ thường, bỏ qua khoảng trắng, dấu gạch dưới và dấu gạch nối.

#### [`runtime`](/pkg/runtime/)

Các hàm dọn dẹp được lập lịch bởi [`AddCleanup`](/pkg/runtime#AddCleanup) hiện được thực thi
đồng thời và song song, giúp các thao tác dọn dẹp phù hợp hơn cho việc
sử dụng nhiều như gói [`unique`](/pkg/unique). Lưu ý rằng từng thao tác dọn dẹp riêng lẻ vẫn nên
chuyển công việc của mình sang một goroutine mới nếu chúng phải thực thi hoặc
chặn trong thời gian dài để tránh chặn hàng đợi dọn dẹp.

Một thiết lập mới `GODEBUG=checkfinalizers=1` giúp tìm các vấn đề phổ biến với
finalizer và thao tác dọn dẹp, chẳng hạn như các vấn đề được mô tả [trong hướng dẫn
GC](/doc/gc-guide#Finalizers_cleanups_and_weak_pointers).
Ở chế độ này, runtime chạy chẩn đoán trong mỗi chu kỳ thu gom rác,
và cũng sẽ thường xuyên báo cáo độ dài của hàng đợi finalizer và
dọn dẹp tới stderr để giúp xác định các vấn đề với
finalizer và/hoặc thao tác dọn dẹp chạy trong thời gian dài.
Xem [tài liệu GODEBUG](https://pkg.go.dev/runtime#hdr-Environment_Variables)
để biết thêm chi tiết.

Hàm [`SetDefaultGOMAXPROCS`](/pkg/runtime#SetDefaultGOMAXPROCS) mới đặt `GOMAXPROCS` thành giá trị
mặc định của runtime, như thể biến môi trường `GOMAXPROCS` chưa được đặt. Điều này
hữu ích để bật [mặc định `GOMAXPROCS` mới](#container-aware-gomaxprocs) nếu nó đã bị
vô hiệu hóa bởi biến môi trường `GOMAXPROCS` hoặc một lần gọi trước đó tới
[`GOMAXPROCS`](/pkg/runtime#GOMAXPROCS).

#### [`runtime/pprof`](/pkg/runtime/pprof/)

Hồ sơ mutex cho tranh chấp trên các khóa nội bộ của runtime hiện trỏ chính xác đến phần kết thúc của vùng tới hạn gây ra độ trễ. Điều này khớp với hành vi của hồ sơ đối với tranh chấp trên các giá trị `sync.Mutex`. Thiết lập `runtimecontentionstacks` cho `GODEBUG`, vốn cho phép chọn hành vi bất thường của Go 1.22 đến 1.24 đối với phần này của hồ sơ, hiện đã bị loại bỏ.

#### [`sync`](/pkg/sync/)

Phương thức mới [`WaitGroup.Go`](/pkg/sync#WaitGroup.Go) giúp mẫu sử dụng phổ biến để tạo và đếm goroutine trở nên thuận tiện hơn.

#### [`testing`](/pkg/testing/)

Các phương thức mới [`T.Attr`](/pkg/testing#T.Attr), [`B.Attr`](/pkg/testing#B.Attr), và [`F.Attr`](/pkg/testing#F.Attr) ghi một thuộc tính vào nhật ký kiểm thử. Một thuộc tính là một khóa và giá trị tùy ý được liên kết với một kiểm thử.

Ví dụ, trong một kiểm thử có tên `TestF`,  
`t.Attr("key", "value")` ghi:

```
=== ATTR  TestF key value
```

Với cờ `-json`, các thuộc tính xuất hiện dưới dạng một hành động mới "attr".

<!-- go.dev/issue/59928 -->

Phương thức mới [`Output`](/pkg/testing#T.Output) của [`T`](/pkg/testing#T), [`B`](/pkg/testing#B) và [`F`](/pkg/testing#F) cung cấp một [`io.Writer`](/pkg/io#Writer) ghi vào cùng luồng đầu ra kiểm thử như [`TB.Log`](/pkg/testing#TB.Log). Giống như `TB.Log`, đầu ra được thụt lề, nhưng không bao gồm số dòng và tệp.

<!-- https://go.dev/issue/70464, CL 630137 -->
Hàm [`AllocsPerRun`](/pkg/testing#AllocsPerRun) hiện panic nếu các kiểm thử song song đang chạy.  
Kết quả của [`AllocsPerRun`](/pkg/testing#AllocsPerRun) vốn không ổn định nếu các kiểm thử khác đang chạy.  
Hành vi panic mới giúp phát hiện các lỗi như vậy.

#### [`testing/fstest`](/pkg/testing/fstest/)

[`MapFS`](/pkg/testing/fstest#MapFS) triển khai interface [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS) mới.  
[`TestFS`](/pkg/testing/fstest#TestFS) sẽ kiểm tra chức năng của interface [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS) nếu được triển khai.  
[`TestFS`](/pkg/testing/fstest#TestFS) sẽ không còn lần theo các liên kết tượng trưng để tránh đệ quy không giới hạn.

#### [`unicode`](/pkg/unicode/)

Map [danh sách bí danh `CategoryAliases`](/pkg/unicode#CategoryAliases) mới cung cấp quyền truy cập vào các tên bí danh của danh mục, chẳng hạn như “Letter” cho “L”.

Các danh mục mới [`Cn`](/pkg/unicode#Cn) và [`LC`](/pkg/unicode#LC) lần lượt định nghĩa các codepoint chưa được gán và các chữ cái có phân biệt chữ hoa chữ thường.
Các danh mục này luôn được Unicode định nghĩa nhưng đã vô tình bị bỏ sót trong các phiên bản Go trước đó.
Danh mục [`C`](/pkg/unicode#C) hiện bao gồm [`Cn`](/pkg/unicode#Cn), nghĩa là đã thêm tất cả các code point chưa được gán.

#### [`unique`](/pkg/unique/)

Gói [`unique`](/pkg/unique) hiện thu hồi các giá trị interned nhanh hơn, hiệu quả hơn và song song hơn. Do đó, các ứng dụng sử dụng [`Make`](/pkg/unique#Make) hiện ít có khả năng gặp tình trạng bộ nhớ tăng đột biến khi có nhiều giá trị thực sự duy nhất được intern.

Các giá trị được truyền vào [`Make`](/pkg/unique#Make) có chứa [`Handle`](/pkg/unique#Handle) trước đây yêu cầu nhiều chu kỳ thu gom rác để được thu thập, tỷ lệ thuận với độ sâu của chuỗi giá trị [`Handle`](/pkg/unique#Handle). Hiện nay, khi không còn được sử dụng, chúng được thu thập kịp thời trong một chu kỳ duy nhất.

## Các cổng {#ports}

### Darwin

<!-- go.dev/issue/69839 -->
Như đã [thông báo](/doc/go1.24#darwin) trong ghi chú bản phát hành Go 1.24, Go 1.25 yêu cầu macOS 12 Monterey hoặc mới hơn.
Hỗ trợ cho các phiên bản trước đã bị ngừng.

### Windows

<!-- go.dev/issue/71671 -->
Go 1.25 là bản phát hành cuối cùng chứa cổng windows/arm 32-bit [bị lỗi](/doc/go1.24#windows) (`GOOS=windows` `GOARCH=arm`). Cổng này sẽ bị xóa trong Go 1.26.

### AMD64

<!-- go.dev/issue/71204 -->
Ở chế độ `GOAMD64=v3` hoặc cao hơn, trình biên dịch hiện sẽ sử dụng các lệnh fused multiply-add để làm cho phép tính số thực nhanh hơn và chính xác hơn. Điều này có thể thay đổi các giá trị số thực chính xác mà một chương trình tạo ra.

Để tránh việc hợp nhất, hãy sử dụng phép ép kiểu `float64` rõ ràng, như `float64(a*b)+c`.

### Loong64

<!-- CLs 533717, 533716, 543316, 604176 -->
Cổng linux/loong64 hiện hỗ trợ trình phát hiện race, thu thập thông tin traceback từ mã C bằng [`runtime.SetCgoTraceback`](/pkg/runtime#SetCgoTraceback), và liên kết các chương trình cgo bằng chế độ liên kết nội bộ.

### RISC-V

<!-- CL 420114 -->
Cổng linux/riscv64 hiện hỗ trợ chế độ xây dựng `plugin`.

<!-- https://go.dev/issue/61476, CL 633417 -->
Biến môi trường `GORISCV64` hiện chấp nhận một giá trị mới `rva23u64`, giá trị này chọn hồ sơ ứng dụng chế độ người dùng RVA23U64.

<!--
Đầu ra từ relnote todo được tạo và xem xét vào ngày 2025-05-23, cùng thông tin tóm tắt từ bug/CL: -->

<!-- Các mục không cần được đề cập trong ghi chú bản phát hành Go 1.25 nhưng được relnote todo thu thập
Chỉ cập nhật các đề xuất cũ
accepted proposal https://go.dev/issue/30999 (from https://go.dev/cl/671795)
accepted proposal https://go.dev/issue/36532 (from https://go.dev/cl/647555)
accepted proposal https://go.dev/issue/48429 (from https://go.dev/cl/648577)
accepted proposal https://go.dev/issue/51572 (from https://go.dev/cl/651996)
accepted proposal https://go.dev/issue/51430 (from https://go.dev/cl/644997, https://go.dev/cl/646355)
accepted proposal https://go.dev/issue/60905 (from https://go.dev/cl/645795)
accepted proposal https://go.dev/issue/61716 (from https://go.dev/cl/644475)
accepted proposal https://go.dev/issue/64876 (from https://go.dev/cl/649435)
accepted proposal https://go.dev/issue/70123 (from https://go.dev/cl/657116)
accepted proposal https://go.dev/issue/61901 (from https://go.dev/cl/647875)
accepted proposal https://go.dev/issue/64207 (from https://go.dev/cl/647015, https://go.dev/cl/652235)
accepted proposal https://go.dev/issue/70200 (from https://go.dev/cl/674916)

Đối với các subrepo:
accepted proposal https://go.dev/issue/53757 (from https://go.dev/cl/644575)
accepted proposal https://go.dev/issue/54743 (from https://go.dev/cl/532415)
accepted proposal https://go.dev/issue/57792 (from https://go.dev/cl/649716, https://go.dev/cl/651737)
accepted proposal https://go.dev/issue/58523 (from https://go.dev/cl/538235)
accepted proposal https://go.dev/issue/61537 (from https://go.dev/cl/531935)
accepted proposal https://go.dev/issue/61940 (from https://go.dev/cl/650235)
accepted proposal https://go.dev/issue/67839 (from https://go.dev/cl/646535)
accepted proposal https://go.dev/issue/68780 (from https://go.dev/cl/659835)
accepted proposal https://go.dev/issue/69095 (from https://go.dev/cl/649320, https://go.dev/cl/649321, https://go.dev/cl/649337, https://go.dev/cl/649376, https://go.dev/cl/649377, https://go.dev/cl/649378, https://go.dev/cl/649379, https://go.dev/cl/649380, https://go.dev/cl/649397, https://go.dev/cl/649398, https://go.dev/cl/649419, https://go.dev/cl/649497, https://go.dev/cl/649498, https://go.dev/cl/649618, https://go.dev/cl/649675, https://go.dev/cl/649676, https://go.dev/cl/649677, https://go.dev/cl/649695, https://go.dev/cl/649696, https://go.dev/cl/649697, https://go.dev/cl/649698, https://go.dev/cl/649715, https://go.dev/cl/649717, https://go.dev/cl/649718, https://go.dev/cl/649755, https://go.dev/cl/649775, https://go.dev/cl/649795, https://go.dev/cl/649815, https://go.dev/cl/649835, https://go.dev/cl/651336, https://go.dev/cl/651736, https://go.dev/cl/651737, https://go.dev/cl/658018)
-->
[cross-site request forgery (csrf)]: https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/CSRF
[sec-fetch-site]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Site
