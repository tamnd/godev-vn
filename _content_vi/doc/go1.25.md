---
title: Ghi chú phát hành Go 1.25
---

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu Go 1.25 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.25, ra mắt vào [tháng 8 năm 2025](/doc/devel/release#go1.25.0), sáu tháng sau [Go 1.24](/doc/go1.24).
Phần lớn các thay đổi nằm ở phần triển khai toolchain, runtime và thư viện.
Như thường lệ, bản phát hành duy trì cam kết tương thích của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

## Thay đổi ngôn ngữ {#language}

<!-- go.dev/issue/70128 -->

Không có thay đổi ngôn ngữ nào ảnh hưởng đến các chương trình Go trong Go 1.25.
Tuy nhiên, trong [đặc tả ngôn ngữ](/ref/spec), khái niệm về core type
đã được xóa bỏ để thay bằng văn xuôi chuyên biệt.
Xem [bài viết blog](/blog/coretypes) tương ứng để biết thêm thông tin.

## Công cụ {#tools}

### Lệnh Go {#go-command}

Tùy chọn `-asan` của `go build` giờ mặc định thực hiện phát hiện rò rỉ khi
chương trình kết thúc.
Điều này sẽ báo lỗi nếu bộ nhớ được cấp phát bởi C không được giải phóng và
không được tham chiếu bởi bất kỳ bộ nhớ nào khác được cấp phát bởi C hoặc Go.
Các báo cáo lỗi mới này có thể bị vô hiệu hóa bằng cách đặt
`ASAN_OPTIONS=detect_leaks=0` trong môi trường khi chạy chương trình.

<!-- go.dev/issue/71867 -->
Bản phân phối Go sẽ bao gồm ít tệp nhị phân công cụ được tạo sẵn hơn. Các tệp nhị phân
toolchain cốt lõi như trình biên dịch và linker vẫn được bao gồm, nhưng các công cụ
không được gọi bởi các thao tác build hoặc test sẽ được build
và chạy bởi `go tool` khi cần.

<!-- go.dev/issue/42965 -->
Directive mới `ignore` trong `go.mod` [directive](/ref/mod#go-mod-file-ignore) có thể được dùng để
chỉ định các thư mục mà lệnh `go` nên bỏ qua. Các tệp trong các thư mục này
và các thư mục con của chúng sẽ bị lệnh `go` bỏ qua khi khớp các mẫu gói,
như `all` hoặc `./...`, nhưng vẫn sẽ được bao gồm trong các tệp zip module.

<!-- go.dev/issue/68106 -->
Tùy chọn mới `go doc` `-http` sẽ khởi động một server tài liệu hiển thị
tài liệu cho đối tượng được yêu cầu, và mở tài liệu trong cửa sổ trình duyệt.

<!-- go.dev/issue/69712 -->

Tùy chọn mới `go version -m -json` sẽ in mã hóa JSON của các cấu trúc
`runtime/debug.BuildInfo` được nhúng trong các tệp nhị phân Go đã cho.

<!-- go.dev/issue/34055 -->
Lệnh `go` giờ hỗ trợ sử dụng thư mục con của kho lưu trữ làm
đường dẫn gốc module, khi [phân giải đường dẫn module](/ref/mod#vcs-find) sử dụng cú pháp
`<meta name="go-import" content="root-path vcs repo-url subdir">` để chỉ ra
rằng `root-path` tương ứng với `subdir` của `repo-url` với
hệ thống quản lý phiên bản `vcs`.

<!-- go.dev/issue/71294 -->

Mẫu gói `work` mới khớp với tất cả các gói trong các work module (trước đây gọi là main)
: hoặc module work đơn trong chế độ module hoặc tập hợp các module workspace
trong chế độ workspace.

<!-- go.dev/issue/65847 -->

Khi lệnh go cập nhật dòng `go` trong tệp `go.mod` hoặc `go.work`,
nó [không còn](/ref/mod#go-mod-file-toolchain) thêm dòng toolchain
chỉ định phiên bản hiện tại của lệnh.

### Vet {#vet}

Lệnh `go vet` bao gồm các trình phân tích mới:

<!-- go.dev/issue/18022 -->

- [waitgroup](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/waitgroup),
  báo cáo các lệnh gọi không đúng chỗ đến [`sync.WaitGroup.Add`](/pkg/sync#WaitGroup.Add); và

<!-- go.dev/issue/28308 -->

- [hostport](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/hostport),
  báo cáo việc sử dụng `fmt.Sprintf("%s:%d", host, port)` để
  xây dựng địa chỉ cho [`net.Dial`](/pkg/net#Dial), vì chúng sẽ không hoạt động với
  IPv6; thay vào đó nó gợi ý dùng [`net.JoinHostPort`](/pkg/net#JoinHostPort).

## Runtime {#runtime}

### `GOMAXPROCS` nhận biết container

<!-- go.dev/issue/73193 -->

Hành vi mặc định của `GOMAXPROCS` đã thay đổi. Trong các phiên bản Go trước,
`GOMAXPROCS` mặc định bằng số CPU logic có sẵn khi khởi động
([`runtime.NumCPU`](/pkg/runtime#NumCPU)). Go 1.25 giới thiệu hai thay đổi:

1. Trên Linux, runtime xem xét giới hạn băng thông CPU của cgroup
   chứa tiến trình, nếu có. Nếu giới hạn băng thông CPU thấp hơn
   số CPU logic có sẵn, `GOMAXPROCS` sẽ mặc định bằng giới hạn thấp hơn.
   Trong các hệ thống runtime container như Kubernetes, giới hạn băng thông CPU của cgroup
   thường tương ứng với tùy chọn "CPU limit". Runtime Go
   không xem xét tùy chọn "CPU requests".

2. Trên tất cả hệ điều hành, runtime định kỳ cập nhật `GOMAXPROCS` nếu số
   CPU logic có sẵn hoặc giới hạn băng thông CPU của cgroup thay đổi.

Cả hai hành vi này đều tự động bị vô hiệu hóa nếu `GOMAXPROCS` được đặt
thủ công thông qua biến môi trường `GOMAXPROCS` hoặc lệnh gọi đến
[`runtime.GOMAXPROCS`](/pkg/runtime#GOMAXPROCS). Chúng cũng có thể bị vô hiệu hóa tường minh với [cài đặt GODEBUG](/doc/godebug)
`containermaxprocs=0` và `updatemaxprocs=0`, tương ứng.

Để hỗ trợ đọc giới hạn cgroup được cập nhật, runtime sẽ giữ các
file descriptor được lưu trong cache cho các tệp cgroup trong suốt thời gian tiến trình.

### Bộ gom rác thử nghiệm mới

<!-- go.dev/issue/73581 -->

Một bộ gom rác mới giờ có sẵn dưới dạng thử nghiệm. Thiết kế của
bộ gom rác này cải thiện hiệu suất đánh dấu và quét các đối tượng nhỏ
thông qua tính cục bộ tốt hơn và khả năng mở rộng CPU. Kết quả benchmark khác nhau, nhưng chúng tôi kỳ vọng
giảm từ 10 đến 40% chi phí thu gom rác trong các chương trình thực tế
sử dụng nhiều bộ gom rác.

Bộ gom rác mới có thể được bật bằng cách đặt `GOEXPERIMENT=greenteagc`
tại thời điểm build. Chúng tôi kỳ vọng thiết kế sẽ tiếp tục phát triển và cải thiện. Vì vậy,
chúng tôi khuyến khích các nhà phát triển Go thử nó và báo cáo kinh nghiệm của họ.
Xem [GitHub issue](/issue/73581) để biết thêm chi tiết về thiết kế và
hướng dẫn chia sẻ phản hồi.

### Trace flight recorder

<!-- go.dev/issue/63185 -->

[Các execution trace của runtime](/pkg/runtime/trace) từ lâu đã cung cấp một cách mạnh mẽ,
nhưng tốn kém để hiểu và debug hành vi cấp thấp của một ứng dụng.
Thật không may, do kích thước và chi phí của việc liên tục ghi execution trace,
chúng thường không thực tế để debug các sự kiện hiếm.

API mới [`runtime/trace.FlightRecorder`](/pkg/runtime/trace#FlightRecorder)
cung cấp cách nhẹ nhàng để chụp execution trace bằng cách liên tục
ghi trace vào bộ đệm vòng trong bộ nhớ. Khi một sự kiện đáng kể
xảy ra, chương trình có thể gọi
[`FlightRecorder.WriteTo`](/pkg/runtime/trace#FlightRecorder.WriteTo) để
chụp nhanh vài giây cuối của trace vào tệp. Cách tiếp cận này tạo ra
trace nhỏ hơn nhiều bằng cách cho phép ứng dụng chỉ chụp các trace quan trọng.

Khoảng thời gian và lượng dữ liệu được chụp bởi
[`FlightRecorder`](/pkg/runtime/trace#FlightRecorder) có thể được cấu hình trong
[`FlightRecorderConfig`](/pkg/runtime/trace#FlightRecorderConfig).

### Thay đổi đầu ra panic không được xử lý

<!-- go.dev/issue/71517 -->

Thông báo được in khi chương trình thoát do panic không được xử lý
đã được khôi phục và repanic không còn lặp lại văn bản của giá trị panic.

Trước đây, một chương trình panic với `panic("PANIC")`,
khôi phục panic, rồi repanic với giá trị gốc sẽ in:

    panic: PANIC [recovered]
      panic: PANIC

Chương trình này giờ sẽ in:

    panic: PANIC [recovered, repanicked]

### Tên VMA trên Linux

<!-- go.dev/issue/71546 -->

Trên các hệ thống Linux có hỗ trợ kernel cho tên vùng bộ nhớ ảo ẩn danh (VMA)
(`CONFIG_ANON_VMA_NAME`), runtime Go sẽ chú thích các ánh xạ bộ nhớ ẩn danh
với thông tin ngữ cảnh về mục đích của chúng. Ví dụ: `[anon: Go: heap]` cho bộ nhớ heap.
Điều này có thể bị vô hiệu hóa với [cài đặt GODEBUG](/doc/godebug)
`decoratemappings=0`.

## Trình biên dịch {#compiler}

### Lỗi con trỏ `nil`

<!-- https://go.dev/issue/72860, CL 657715 -->

Bản phát hành này sửa một [lỗi trình biên dịch](/issue/72860), được giới thiệu trong Go 1.21, có thể
làm trễ việc kiểm tra con trỏ nil không đúng. Các chương trình như sau, trước đây
thực thi thành công (không chính xác), giờ sẽ (đúng) panic với ngoại lệ con trỏ nil:

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

Chương trình này không đúng vì nó sử dụng kết quả của `os.Open` trước khi
kiểm tra lỗi. Nếu `err` khác nil, thì kết quả `f` có thể là nil, trong
trường hợp đó `f.Name()` nên panic. Tuy nhiên, trong các phiên bản Go 1.21 đến 1.24,
trình biên dịch đã làm trễ việc kiểm tra nil không đúng đến *sau* việc kiểm tra lỗi,
khiến chương trình thực thi thành công, vi phạm đặc tả Go. Trong Go
1.25, nó sẽ không còn chạy thành công. Nếu thay đổi này ảnh hưởng đến mã của bạn,
giải pháp là đặt việc kiểm tra lỗi không nil sớm hơn trong mã, tốt nhất là
ngay sau câu lệnh tạo ra lỗi.

### Hỗ trợ DWARF5

<!-- https://go.dev/issue/26379 -->

Trình biên dịch và linker trong Go 1.25 giờ tạo ra thông tin debug
bằng [DWARF phiên bản 5](https://dwarfstd.org/dwarf5std.html). Phiên bản DWARF
mới hơn giảm không gian cần thiết cho thông tin debug trong các tệp nhị phân Go,
và giảm thời gian liên kết, đặc biệt là cho các tệp nhị phân Go lớn.
Tạo DWARF 5 có thể bị vô hiệu hóa bằng cách đặt biến môi trường
`GOEXPERIMENT=nodwarf5` tại thời điểm build (tùy chọn dự phòng này có thể bị xóa trong bản phát hành Go tương lai).

### Slice nhanh hơn

<!-- CLs 653856, 657937, 663795, 664299 -->

Trình biên dịch giờ có thể cấp phát vùng lưu trữ nền cho các slice trên
stack trong nhiều tình huống hơn, điều này cải thiện hiệu suất. Thay đổi này có
tiềm năng khuếch đại các hiệu ứng của việc sử dụng
[unsafe.Pointer](/pkg/unsafe#Pointer) không chính xác, xem ví dụ [issue
73199](/issue/73199). Để theo dõi các vấn đề này, [công cụ
bisect](https://pkg.go.dev/golang.org/x/tools/cmd/bisect) có thể được
dùng để tìm việc cấp phát gây ra sự cố bằng cách sử dụng cờ
`-compile=variablemake`. Tất cả các cấp phát stack mới như vậy cũng có thể
bị tắt bằng cách sử dụng `-gcflags=all=-d=variablemakehash=n`.

## Linker {#linker}

<!-- CL 660996 -->

Linker giờ chấp nhận tùy chọn dòng lệnh `-funcalign=N`, chỉ định
sự căn chỉnh của các đầu vào hàm.
Giá trị mặc định phụ thuộc vào nền tảng, và không thay đổi trong
bản phát hành này.

## Thư viện chuẩn {#library}

### Gói testing/synctest mới

<!-- go.dev/issue/67434, go.dev/issue/73567 -->
Gói mới [`testing/synctest`](/pkg/testing/synctest) cung cấp hỗ trợ
để test mã đồng thời.

Hàm [`Test`](/pkg/testing/synctest#Test) chạy hàm test trong một
"bong bóng" biệt lập. Trong bong bóng, thời gian được ảo hóa: các hàm của gói [`time`](/pkg/time)
hoạt động trên đồng hồ giả và đồng hồ tiến về phía trước ngay lập tức nếu
tất cả goroutine trong bong bóng đều bị chặn.

Hàm [`Wait`](/pkg/testing/synctest#Wait) chờ tất cả goroutine trong
bong bóng hiện tại bị chặn.

Gói này lần đầu có sẵn trong Go 1.24 dưới `GOEXPERIMENT=synctest`, với
API hơi khác. Thử nghiệm giờ đã được nâng cấp lên trạng thái chính thức.
API cũ vẫn còn hiện diện nếu `GOEXPERIMENT=synctest` được đặt,
nhưng sẽ bị xóa trong Go 1.26.

### Gói encoding/json/v2 thử nghiệm mới {#json_v2}

Go 1.25 bao gồm một triển khai JSON mới, thử nghiệm,
có thể được bật bằng cách đặt biến môi trường
`GOEXPERIMENT=jsonv2` tại thời điểm build.

Khi được bật, hai gói mới có sẵn:
- Gói [`encoding/json/v2`](/pkg/encoding/json/v2) là
  bản sửa đổi lớn của gói `encoding/json`.
- Gói [`encoding/json/jsontext`](/pkg/encoding/json/jsontext)
  cung cấp xử lý cú pháp JSON cấp thấp hơn.

Ngoài ra, khi GOEXPERIMENT "jsonv2" được bật:
- Gói [`encoding/json`](/pkg/encoding/json) sử dụng triển khai JSON mới.
  Hành vi marshaling và unmarshaling không bị ảnh hưởng,
  nhưng văn bản của các lỗi được trả về bởi các hàm gói có thể thay đổi.
- Gói [`encoding/json`](/pkg/encoding/json) chứa
  một số tùy chọn mới có thể được dùng
  để cấu hình marshaler và unmarshaler.

Triển khai mới hoạt động tốt hơn đáng kể so với triển khai hiện có trong nhiều tình huống. Nói chung,
hiệu suất encoding tương đương giữa các triển khai
và decoding nhanh hơn đáng kể trong triển khai mới.
Xem kho lưu trữ [github.com/go-json-experiment/jsonbench](https://github.com/go-json-experiment/jsonbench)
để biết phân tích chi tiết hơn.

Xem [issue đề xuất](/issue/71497) để biết thêm chi tiết.

Chúng tôi khuyến khích người dùng [`encoding/json`](/pkg/encoding/json) kiểm thử
chương trình của họ với `GOEXPERIMENT=jsonv2` được bật để giúp phát hiện
bất kỳ vấn đề tương thích nào với triển khai mới.

Chúng tôi kỳ vọng thiết kế của [`encoding/json/v2`](/pkg/encoding/json/v2)
sẽ tiếp tục phát triển. Chúng tôi khuyến khích các nhà phát triển thử API mới
và cung cấp phản hồi trên [issue đề xuất](/issue/71497).

### Thay đổi nhỏ trong thư viện {#minor_library_changes}

#### [`archive/tar`](/pkg/archive/tar/)

Triển khai [`Writer.AddFS`](/pkg/archive/tar#Writer.AddFS) giờ hỗ trợ symbolic link
cho các hệ thống tệp triển khai [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS).

#### [`encoding/asn1`](/pkg/encoding/asn1/)

[`Unmarshal`](/pkg/encoding/asn1#Unmarshal) và [`UnmarshalWithParams`](/pkg/encoding/asn1#UnmarshalWithParams)
giờ phân tích các kiểu ASN.1 T61String và BMPString nhất quán hơn. Điều này có thể
dẫn đến một số mã hóa không đúng định dạng đã được chấp nhận trước đây giờ bị từ chối.

#### [`crypto`](/pkg/crypto/)

[`MessageSigner`](/pkg/crypto#MessageSigner) là interface ký mới có thể
được triển khai bởi các signer muốn tự hash thông điệp cần ký.
Một hàm mới cũng được giới thiệu, [`SignMessage`](/pkg/crypto#SignMessage),
cố gắng nâng cấp interface [`Signer`](/pkg/crypto#Signer) lên
[`MessageSigner`](/pkg/crypto#MessageSigner), sử dụng phương thức
[`MessageSigner.SignMessage`](/pkg/crypto#MessageSigner.SignMessage) nếu
thành công, và [`Signer.Sign`](/pkg/crypto#Signer.Sign) nếu không. Điều này có thể được
dùng khi mã muốn hỗ trợ cả [`Signer`](/pkg/crypto#Signer) và
[`MessageSigner`](/pkg/crypto#MessageSigner).

Thay đổi cài đặt GODEBUG `fips140` [GODEBUG setting](/doc/godebug) sau khi chương trình đã khởi động giờ là no-op.
Trước đây, nó được tài liệu hóa là không được phép, và có thể gây ra panic nếu thay đổi.

SHA-1, SHA-256, và SHA-512 giờ chậm hơn trên amd64 khi các lệnh AVX2 không có sẵn.
Tất cả bộ xử lý server (và hầu hết các bộ xử lý khác) được sản xuất từ năm 2015 đều hỗ trợ AVX2.

#### [`crypto/ecdsa`](/pkg/crypto/ecdsa/)

Các hàm và phương thức mới [`ParseRawPrivateKey`](/pkg/crypto/ecdsa#ParseRawPrivateKey),
[`ParseUncompressedPublicKey`](/pkg/crypto/ecdsa#ParseUncompressedPublicKey),
[`PrivateKey.Bytes`](/pkg/crypto/ecdsa#PrivateKey.Bytes), và
[`PublicKey.Bytes`](/pkg/crypto/ecdsa#PublicKey.Bytes)
triển khai các mã hóa cấp thấp, thay thế nhu cầu sử dụng
các hàm và phương thức [`crypto/elliptic`](/pkg/crypto/elliptic) hoặc [`math/big`](/pkg/math/big).

Khi chế độ FIPS 140-3 được bật, ký hiệu giờ nhanh hơn bốn lần, khớp với
hiệu suất của chế độ không phải FIPS.

#### [`crypto/ed25519`](/pkg/crypto/ed25519/)

Khi chế độ FIPS 140-3 được bật, ký hiệu giờ nhanh hơn bốn lần, khớp với
hiệu suất của chế độ không phải FIPS.

#### [`crypto/elliptic`](/pkg/crypto/elliptic/)

Các phương thức ẩn và không có tài liệu `Inverse` và `CombinedMult` trên một số
triển khai [`Curve`](/pkg/crypto/elliptic#Curve) đã bị xóa.

#### [`crypto/rsa`](/pkg/crypto/rsa/)

[`PublicKey`](/pkg/crypto/rsa#PublicKey) không còn tuyên bố rằng giá trị modulus
được coi là bí mật. [`VerifyPKCS1v15`](/pkg/crypto/rsa#VerifyPKCS1v15) và
[`VerifyPSS`](/pkg/crypto/rsa#VerifyPSS) đã cảnh báo rằng tất cả đầu vào là
công khai và có thể bị rò rỉ, và có các cuộc tấn công toán học có thể khôi phục
modulus từ các giá trị công khai khác.

Tạo khóa giờ nhanh hơn ba lần.

#### [`crypto/sha1`](/pkg/crypto/sha1/)

Hashing giờ nhanh hơn hai lần trên amd64 khi các lệnh SHA-NI có sẵn.

#### [`crypto/sha3`](/pkg/crypto/sha3/)

Phương thức mới [`SHA3.Clone`](/pkg/crypto/sha3#SHA3.Clone) triển khai [`hash.Cloner`](/pkg/hash#Cloner).

Hashing giờ nhanh hơn hai lần trên các bộ xử lý Apple M.

#### [`crypto/tls`](/pkg/crypto/tls/)

Trường mới [`ConnectionState.CurveID`](/pkg/crypto/tls#ConnectionState.CurveID)
hiển thị cơ chế trao đổi khóa được dùng để thiết lập kết nối.

Callback mới [`Config.GetEncryptedClientHelloKeys`](/pkg/crypto/tls#Config.GetEncryptedClientHelloKeys)
có thể được dùng để đặt các [`EncryptedClientHelloKey`](/pkg/crypto/tls#EncryptedClientHelloKey)
cho server sử dụng khi client gửi extension Encrypted Client Hello.

Các thuật toán chữ ký SHA-1 giờ bị cấm trong các bắt tay TLS 1.2, theo
[RFC 9155](https://www.rfc-editor.org/rfc/rfc9155.html).
Chúng có thể được bật lại với [cài đặt GODEBUG](/doc/godebug) `tlssha1=1`.

Khi [chế độ FIPS 140-3](/doc/security/fips140) được bật, Extended Master Secret
giờ được yêu cầu trong TLS 1.2, và Ed25519 và X25519MLKEM768 giờ được cho phép.

TLS server giờ ưu tiên phiên bản giao thức được hỗ trợ cao nhất, ngay cả khi nó không phải là
phiên bản được ưu tiên nhất của client.

<!-- CL 687855 -->
Cả TLS client và server giờ nghiêm ngặt hơn trong việc tuân theo các đặc tả
và từ chối hành vi không theo đặc tả. Kết nối với các peer tuân thủ nên
không bị ảnh hưởng.

#### [`crypto/x509`](/pkg/crypto/x509/)

[`CreateCertificate`](/pkg/crypto/x509#CreateCertificate),
[`CreateCertificateRequest`](/pkg/crypto/x509#CreateCertificateRequest), và
[`CreateRevocationList`](/pkg/crypto/x509#CreateRevocationList) giờ có thể chấp nhận
interface ký [`crypto.MessageSigner`](/pkg/crypto#MessageSigner) cũng như
[`crypto.Signer`](/pkg/crypto#Signer). Điều này cho phép các hàm này sử dụng
các signer triển khai interface ký "one-shot", trong đó hashing được thực hiện như
một phần của thao tác ký, thay vì bởi người gọi.

[`CreateCertificate`](/pkg/crypto/x509#CreateCertificate) giờ sử dụng SHA-256 bị cắt ngắn
để điền `SubjectKeyId` nếu nó bị thiếu.
[Cài đặt GODEBUG](/doc/godebug) `x509sha256skid=0` quay lại SHA-1.

[`ParseCertificate`](/pkg/crypto/x509#ParseCertificate) giờ từ chối các chứng chỉ
chứa extension BasicConstraints có pathLenConstraint âm.

[`ParseCertificate`](/pkg/crypto/x509#ParseCertificate) giờ xử lý các chuỗi được mã hóa
với các kiểu ASN.1 T61String và BMPString nhất quán hơn. Điều này có thể dẫn đến
một số mã hóa không đúng định dạng đã được chấp nhận trước đây giờ bị từ chối.

#### [`debug/elf`](/pkg/debug/elf/)

Gói [`debug/elf`](/pkg/debug/elf) thêm hai hằng số mới:
- [`PT_RISCV_ATTRIBUTES`](/pkg/debug/elf#PT_RISCV_ATTRIBUTES)
- [`SHT_RISCV_ATTRIBUTES`](/pkg/debug/elf#SHT_RISCV_ATTRIBUTES)
  để phân tích ELF RISC-V.

#### [`go/ast`](/pkg/go/ast/)

Các hàm [`FilterPackage`](/pkg/ast#FilterPackage), [`PackageExports`](/pkg/ast#PackageExports), và
[`MergePackageFiles`](/pkg/ast#MergePackageFiles), và kiểu [`MergeMode`](/pkg/go/ast#MergeMode) cùng các
hằng số của nó, đều bị deprecated, vì chúng chỉ dùng với bộ máy
[`Object`](/pkg/ast#Object) và [`Package`](/pkg/ast#Package) bị deprecated từ lâu.

Hàm mới [`PreorderStack`](/pkg/go/ast#PreorderStack), giống [`Inspect`](/pkg/go/ast#Inspect), duyệt qua
cây cú pháp và cung cấp kiểm soát việc đi xuống các cây con, nhưng như một
tiện lợi nó cũng cung cấp stack của các node bao quanh tại mỗi điểm.

#### [`go/parser`](/pkg/go/parser/)

Hàm [`ParseDir`](/pkg/go/parser#ParseDir) bị deprecated.

#### [`go/token`](/pkg/go/token/)

Phương thức mới [`FileSet.AddExistingFiles`](/pkg/go/token#FileSet.AddExistingFiles) cho phép các
[`File`](/pkg/go/token#File) hiện có được thêm vào [`FileSet`](/pkg/go/token#FileSet),
hoặc [`FileSet`](/pkg/go/token#FileSet) được xây dựng cho một tập hợp tùy ý
các [`File`](/pkg/go/token#File), giảm bớt các vấn đề liên quan đến một
[`FileSet`](/pkg/go/token#FileSet) toàn cục duy nhất trong các ứng dụng chạy lâu dài.

#### [`go/types`](/pkg/go/types/)

[`Var`](/pkg/go/types#Var) giờ có phương thức [`Var.Kind`](/pkg/go/types#Var.Kind) phân loại biến là một
trong: biến cấp gói, receiver, tham số, kết quả, biến cục bộ, hoặc
trường struct.

Hàm mới [`LookupSelection`](/pkg/go/types#LookupSelection) tra cứu trường hoặc phương thức của một
tên và kiểu receiver đã cho, giống hàm [`LookupFieldOrMethod`](/pkg/go/types#LookupFieldOrMethod) hiện có,
nhưng trả về kết quả dưới dạng [`Selection`](/pkg/go/types#Selection).

#### [`hash`](/pkg/hash/)

Interface mới [`XOF`](/pkg/hash#XOF) có thể được triển khai bởi "các hàm đầu ra mở rộng",
là các hàm hash có độ dài đầu ra tùy ý hoặc không giới hạn
như [SHAKE](/pkg/crypto/sha3#SHAKE).

Các hash triển khai interface mới [`Cloner`](/pkg/hash#Cloner) có thể trả về một bản sao trạng thái của chúng.
Tất cả các triển khai [`Hash`](/pkg/hash#Hash) của thư viện chuẩn giờ triển khai [`Cloner`](/pkg/hash#Cloner).

#### [`hash/maphash`](/pkg/hash/maphash/)

Phương thức mới [`Hash.Clone`](/pkg/hash/maphash#Hash.Clone) triển khai [`hash.Cloner`](/pkg/hash#Cloner).

#### [`io/fs`](/pkg/io/fs/)

Interface mới [`ReadLinkFS`](/pkg/io/fs#ReadLinkFS) cung cấp khả năng đọc symbolic link trong hệ thống tệp.

#### [`log/slog`](/pkg/log/slog/)

[`GroupAttrs`](/pkg/log/slog#GroupAttrs) tạo một [`Attr`](/pkg/log/slog#Attr) nhóm từ một slice các giá trị [`Attr`](/pkg/log/slog#Attr).

[`Record`](/pkg/log/slog#Record) giờ có phương thức [`Source`](/pkg/log/slog#Record.Source),
trả về vị trí nguồn của nó hoặc nil nếu không có.

#### [`mime/multipart`](/pkg/mime/multipart/)

Hàm helper mới [`FileContentDisposition`](/pkg/mime/multipart#FileContentDisposition) xây dựng các trường header
Content-Disposition multipart.

#### [`net`](/pkg/net/)

[`LookupMX`](/pkg/net#LookupMX) và [`Resolver.LookupMX`](/pkg/net#Resolver.LookupMX) giờ trả về các tên DNS trông
giống như địa chỉ IP hợp lệ, cũng như các tên miền hợp lệ.
Trước đây nếu một name server trả về địa chỉ IP như tên DNS,
[`LookupMX`](/pkg/net#LookupMX) sẽ loại bỏ nó, theo yêu cầu của các RFC.
Tuy nhiên, các name server trong thực tế đôi khi trả về địa chỉ IP.

Trên Windows, [`ListenMulticastUDP`](/pkg/net#ListenMulticastUDP) giờ hỗ trợ địa chỉ IPv6.

Trên Windows, giờ có thể chuyển đổi giữa [`os.File`](/pkg/os#File)
và kết nối mạng. Cụ thể, các hàm [`FileConn`](/pkg/net#FileConn),
[`FilePacketConn`](/pkg/net#FilePacketConn), và
[`FileListener`](/pkg/net#FileListener) giờ được triển khai, và
trả về kết nối mạng hoặc listener tương ứng với tệp đang mở.
Tương tự, các phương thức `File` của [`TCPConn`](/pkg/net#TCPConn.File),
[`UDPConn`](/pkg/net#UDPConn.File), [`UnixConn`](/pkg/net#UnixConn.File),
[`IPConn`](/pkg/net#IPConn.File), [`TCPListener`](/pkg/net#TCPListener.File),
và [`UnixListener`](/pkg/net#UnixListener.File) giờ được triển khai, và trả về
[`os.File`](/pkg/os#File) nền của kết nối mạng.

#### [`net/http`](/pkg/net/http/)

[`CrossOriginProtection`](/pkg/net/http#CrossOriginProtection) mới triển khai các biện pháp bảo vệ chống lại [Cross-Site
Request Forgery (CSRF)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/CSRF) bằng cách từ chối các yêu cầu trình duyệt cross-origin không an toàn.
Nó sử dụng [Fetch metadata của trình duyệt hiện đại](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Site), không yêu cầu token
hoặc cookie, và hỗ trợ bypass dựa trên origin và mẫu.

#### [`os`](/pkg/os/)

Trên Windows, [`NewFile`](/pkg/os#NewFile) giờ hỗ trợ các handle được mở cho I/O không đồng bộ (tức là,
[`syscall.FILE_FLAG_OVERLAPPED`](/pkg/syscall#FILE_FLAG_OVERLAPPED) được chỉ định trong lệnh gọi [`syscall.CreateFile`](/pkg/syscall#CreateFile)).
Các handle này được liên kết với cổng hoàn thành I/O của runtime Go,
cung cấp các lợi ích sau cho [`File`](/pkg/os#File) kết quả:

- Các phương thức I/O ([`File.Read`](/pkg/os#File.Read), [`File.Write`](/pkg/os#File.Write), [`File.ReadAt`](/pkg/os#File.ReadAt), và [`File.WriteAt`](/pkg/os#File.WriteAt)) không chặn luồng OS.
- Các phương thức deadline ([`File.SetDeadline`](/pkg/os#File.SetDeadline), [`File.SetReadDeadline`](/pkg/os#File.SetReadDeadline), và [`File.SetWriteDeadline`](/pkg/os#File.SetWriteDeadline)) được hỗ trợ.

Cải tiến này đặc biệt có lợi cho các ứng dụng giao tiếp qua named pipe trên Windows.

Lưu ý rằng một handle chỉ có thể được liên kết với một cổng hoàn thành tại một thời điểm.
Nếu handle được cung cấp cho [`NewFile`](/pkg/os#NewFile) đã được liên kết với một cổng hoàn thành,
[`File`](/pkg/os#File) trả về bị hạ cấp xuống chế độ I/O đồng bộ.
Trong trường hợp này, các phương thức I/O sẽ chặn luồng OS, và các phương thức deadline không có tác dụng.

Các hệ thống tệp được trả về bởi [`DirFS`](/pkg/os#DirFS) và [`Root.FS`](/pkg/os#Root.FS) triển khai interface mới [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS).
[`CopyFS`](/pkg/os#CopyFS) hỗ trợ symlink khi sao chép các hệ thống tệp triển khai [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS).

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

Hàm mới [`TypeAssert`](/pkg/reflect#TypeAssert) cho phép chuyển đổi [`Value`](/pkg/reflect#Value) trực tiếp thành giá trị Go
của kiểu đã cho. Điều này giống như sử dụng type assertion trên kết quả của [`Value.Interface`](/pkg/reflect#Value.Interface),
nhưng tránh các cấp phát bộ nhớ không cần thiết.

#### [`regexp/syntax`](/pkg/regexp/syntax/)

Cú pháp lớp ký tự `\p{name}` và `\P{name}` giờ chấp nhận các tên
Any, ASCII, Assigned, Cn, và LC, cũng như các bí danh danh mục Unicode như `\p{Letter}` cho `\pL`.
Theo [Unicode TR18](https://unicode.org/reports/tr18/), chúng cũng giờ sử dụng
tra cứu tên không phân biệt hoa thường, bỏ qua dấu cách, dấu gạch dưới và dấu gạch ngang.

#### [`runtime`](/pkg/runtime/)

Các hàm cleanup được lên lịch bởi [`AddCleanup`](/pkg/runtime#AddCleanup) giờ được thực thi
đồng thời và song song, làm cho cleanup khả thi hơn cho việc sử dụng nhiều
như gói [`unique`](/pkg/unique). Lưu ý rằng các cleanup riêng lẻ vẫn nên
chuyển công việc của chúng sang goroutine mới nếu chúng phải thực thi hoặc
chặn trong thời gian dài để tránh chặn hàng đợi cleanup.

Cài đặt mới `GODEBUG=checkfinalizers=1` giúp tìm các vấn đề phổ biến với
finalizer và cleanup, như những vấn đề được mô tả [trong hướng dẫn GC](/doc/gc-guide#Finalizers_cleanups_and_weak_pointers).
Trong chế độ này, runtime chạy chẩn đoán trên mỗi chu kỳ thu gom rác,
và cũng sẽ thường xuyên báo cáo độ dài hàng đợi finalizer và
cleanup cho stderr để giúp xác định các vấn đề với
các finalizer và/hoặc cleanup chạy lâu.
Xem [tài liệu GODEBUG](https://pkg.go.dev/runtime#hdr-Environment_Variables)
để biết thêm chi tiết.

Hàm mới [`SetDefaultGOMAXPROCS`](/pkg/runtime#SetDefaultGOMAXPROCS) đặt `GOMAXPROCS` thành giá trị
mặc định runtime, như thể biến môi trường `GOMAXPROCS` không được đặt. Điều này
hữu ích để bật [giá trị mặc định `GOMAXPROCS` mới](#container-aware-gomaxprocs) nếu nó đã
bị vô hiệu hóa bởi biến môi trường `GOMAXPROCS` hoặc lệnh gọi trước đó đến
[`GOMAXPROCS`](/pkg/runtime#GOMAXPROCS).

#### [`runtime/pprof`](/pkg/runtime/pprof/)

Profile mutex cho sự tranh chấp trên các khóa nội bộ runtime giờ trỏ chính xác
đến cuối phần quan trọng gây ra độ trễ. Điều này khớp với
hành vi của profile cho sự tranh chấp trên các giá trị `sync.Mutex`. Cài đặt
`runtimecontentionstacks` cho `GODEBUG`, cho phép chọn tham gia vào hành vi
bất thường của Go 1.22 đến 1.24 cho phần này của profile, giờ
đã bị xóa.

#### [`sync`](/pkg/sync/)

Phương thức mới [`WaitGroup.Go`](/pkg/sync#WaitGroup.Go)
làm cho mẫu phổ biến về tạo và đếm goroutine thuận tiện hơn.

#### [`testing`](/pkg/testing/)

Các phương thức mới [`T.Attr`](/pkg/testing#T.Attr), [`B.Attr`](/pkg/testing#B.Attr), và [`F.Attr`](/pkg/testing#F.Attr) phát ra
một thuộc tính vào log test. Thuộc tính là
key và value tùy ý liên quan đến test.

Ví dụ, trong test có tên `TestF`,
`t.Attr("key", "value")` phát ra:

```
=== ATTR  TestF key value
```

Với cờ `-json`, các thuộc tính xuất hiện như một "attr" action mới.

<!-- go.dev/issue/59928 -->

Phương thức mới [`Output`](/pkg/testing#T.Output) của [`T`](/pkg/testing#T), [`B`](/pkg/testing#B) và [`F`](/pkg/testing#F) cung cấp [`io.Writer`](/pkg/io#Writer)
ghi vào cùng luồng đầu ra test như [`TB.Log`](/pkg/testing#TB.Log).
Giống `TB.Log`, đầu ra được thụt lề, nhưng không bao gồm số tệp và dòng.

<!-- https://go.dev/issue/70464, CL 630137 -->
Hàm [`AllocsPerRun`](/pkg/testing#AllocsPerRun) giờ panic
nếu các test song song đang chạy.
Kết quả của [`AllocsPerRun`](/pkg/testing#AllocsPerRun) vốn dĩ
không ổn định nếu các test khác đang chạy.
Hành vi panic mới giúp bắt các lỗi như vậy.

#### [`testing/fstest`](/pkg/testing/fstest/)

[`MapFS`](/pkg/testing/fstest#MapFS) triển khai interface mới [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS).
[`TestFS`](/pkg/testing/fstest#TestFS) sẽ xác minh chức năng của interface [`io/fs.ReadLinkFS`](/pkg/io/fs#ReadLinkFS) nếu được triển khai.
[`TestFS`](/pkg/testing/fstest#TestFS) sẽ không còn theo dõi symlink để tránh đệ quy không giới hạn.

<!-- #### [`testing/synctest`](/pkg/testing/synctest/) mentioned above -->

#### [`unicode`](/pkg/unicode/)

Map mới [`CategoryAliases`](/pkg/unicode#CategoryAliases) cung cấp quyền truy cập vào các tên bí danh danh mục, như "Letter" cho "L".

Các danh mục mới [`Cn`](/pkg/unicode#Cn) và [`LC`](/pkg/unicode#LC) định nghĩa các code point chưa được gán và các chữ cái có vỏ, tương ứng.
Chúng luôn được Unicode định nghĩa nhưng vô tình bị bỏ qua trong các phiên bản Go trước.
Danh mục [`C`](/pkg/unicode#C) giờ bao gồm [`Cn`](/pkg/unicode#Cn), nghĩa là nó đã thêm tất cả các code point chưa được gán.

#### [`unique`](/pkg/unique/)

Gói [`unique`](/pkg/unique) giờ thu hồi các giá trị được intern tích cực hơn,
hiệu quả hơn và song song. Kết quả là, các ứng dụng sử dụng
[`Make`](/pkg/unique#Make) giờ ít có khả năng gặp tình trạng bùng nổ bộ nhớ khi nhiều
giá trị thực sự duy nhất được intern.

Các giá trị được truyền cho [`Make`](/pkg/unique#Make) chứa các [`Handle`](/pkg/unique#Handle) trước đây yêu cầu nhiều
chu kỳ thu gom rác để thu thập, tỷ lệ với độ sâu của chuỗi
các giá trị [`Handle`](/pkg/unique#Handle). Giờ, một khi không được sử dụng, chúng được thu thập ngay lập tức trong một chu kỳ duy nhất.

## Các cổng {#ports}

### Darwin

<!-- go.dev/issue/69839 -->
Như đã [thông báo](/doc/go1.24#darwin) trong ghi chú phát hành Go 1.24, Go 1.25 yêu cầu macOS 12 Monterey trở lên.
Hỗ trợ cho các phiên bản trước đã bị ngừng.

### Windows

<!-- go.dev/issue/71671 -->
Go 1.25 là bản phát hành cuối cùng chứa cổng windows/arm 32-bit [bị lỗi](/doc/go1.24#windows) (`GOOS=windows` `GOARCH=arm`). Nó sẽ bị xóa trong Go 1.26.

### AMD64

<!-- go.dev/issue/71204 -->
Trong chế độ `GOAMD64=v3` trở lên, trình biên dịch giờ sẽ sử dụng các lệnh
fused multiply-add để làm cho phép tính số học dấu phẩy động nhanh hơn và
chính xác hơn. Điều này có thể thay đổi các giá trị dấu phẩy động chính xác mà
chương trình tạo ra.

Để tránh fusing, hãy sử dụng cast `float64` tường minh, như `float64(a*b)+c`.

### Loong64

<!-- CLs 533717, 533716, 543316, 604176 -->
Cổng linux/loong64 giờ hỗ trợ trình phát hiện race, thu thập thông tin traceback từ mã C
bằng [`runtime.SetCgoTraceback`](/pkg/runtime#SetCgoTraceback), và liên kết các chương trình cgo với
chế độ liên kết nội bộ.

### RISC-V

<!-- CL 420114 -->
Cổng linux/riscv64 giờ hỗ trợ chế độ build `plugin`.

<!-- https://go.dev/issue/61476, CL 633417 -->
Biến môi trường `GORISCV64` giờ chấp nhận giá trị mới `rva23u64`,
chọn hồ sơ ứng dụng người dùng RVA23U64.

[cross-site request forgery (csrf)]: https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/CSRF
[sec-fetch-site]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Site
