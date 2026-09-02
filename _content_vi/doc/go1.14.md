---
title: Ghi chú phát hành Go 1.14
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

## Giới thiệu về Go 1.14 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.14, ra đời sáu tháng sau [Go 1.13](go1.13).
Hầu hết các thay đổi nằm ở phần triển khai của bộ công cụ, runtime và thư viện.
Như thường lệ, bản phát hành vẫn duy trì [cam kết tương thích](/doc/go1compat.html) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

Hỗ trợ module trong lệnh `go` hiện sẵn sàng cho môi trường production, và chúng tôi khuyến khích tất cả người dùng [di chuyển sang Go modules để quản lý dependency](/blog/migrating-to-go-modules). Nếu bạn không thể di chuyển do sự cố trong bộ công cụ Go, hãy đảm bảo rằng sự cố đó có [issue đang mở](/issue?q=is%3Aissue+is%3Aopen+label%3Amodules) được ghi nhận. (Nếu issue không thuộc milestone `Go1.15`, hãy cho chúng tôi biết lý do nó ngăn bạn di chuyển để chúng tôi có thể ưu tiên phù hợp.)

## Thay đổi ngôn ngữ {#language}

Theo [đề xuất về interface chồng lấp](https://github.com/golang/proposal/blob/master/design/6977-overlapping-interfaces.md),
Go 1.14 giờ cho phép nhúng các interface có tập phương thức chồng lấp: các phương thức từ interface được nhúng có thể có cùng tên và chữ ký giống hệt với các phương thức đã có trong interface (nhúng). Điều này giải quyết các vấn đề thường xảy ra (nhưng không phải độc quyền) với đồ thị nhúng hình kim cương. Các phương thức được khai báo tường minh trong interface phải vẫn [duy nhất](https://tip.golang.org/ref/spec#Uniqueness_of_identifiers), như trước.

## Các nền tảng {#ports}

### Darwin {#darwin}

Go 1.14 là bản phát hành cuối cùng chạy trên macOS 10.11 El Capitan.
Go 1.15 sẽ yêu cầu macOS 10.12 Sierra hoặc mới hơn.

<!-- golang.org/issue/34749 -->
Go 1.14 là bản phát hành Go cuối cùng hỗ trợ tệp nhị phân 32-bit trên macOS (cổng `darwin/386`). Chúng không còn được macOS hỗ trợ, bắt đầu từ macOS 10.15 (Catalina). Go tiếp tục hỗ trợ cổng 64-bit `darwin/amd64`.

<!-- golang.org/issue/34751 -->
Go 1.14 có thể là bản phát hành Go cuối cùng hỗ trợ tệp nhị phân 32-bit trên iOS, iPadOS, watchOS và tvOS (cổng `darwin/arm`). Go tiếp tục hỗ trợ cổng 64-bit `darwin/arm64`.

### Windows {#windows}

<!-- CL 203601 -->
Các tệp nhị phân Go trên Windows giờ có [DEP (Data Execution Prevention)](https://docs.microsoft.com/en-us/windows/win32/memory/data-execution-prevention) được bật.

<!-- CL 202439 -->
Trên Windows, tạo tệp qua [`os.OpenFile`](/pkg/os#CreateFile) với cờ [`os.O_CREATE`](/pkg/os/#O_CREATE), hoặc qua [`syscall.Open`](/pkg/syscall#Open) với cờ [`syscall.O_CREAT`](/pkg/syscall#O_CREAT), giờ sẽ tạo tệp chỉ đọc nếu bit `0o200` (quyền ghi của owner) không được đặt trong tham số quyền. Điều này làm cho hành vi trên Windows giống hơn với Unix.

### WebAssembly {#wasm}

<!-- CL 203600 -->
Các giá trị JavaScript được tham chiếu từ Go qua các đối tượng `js.Value` giờ có thể được bộ gom rác thu hồi.

<!-- CL 203600 -->
Các giá trị `js.Value` không còn có thể so sánh bằng toán tử `==`, và thay vào đó phải so sánh bằng phương thức `Equal` của chúng.

<!-- CL 203600 -->
`js.Value` giờ có các phương thức `IsUndefined`, `IsNull` và `IsNaN`.

### RISC-V {#riscv}

<!-- Issue 27532 -->
Go 1.14 có hỗ trợ thực nghiệm cho 64-bit RISC-V trên Linux (`GOOS=linux`, `GOARCH=riscv64`). Lưu ý rằng hiệu năng, tính ổn định cú pháp assembly và có thể cả tính đúng đắn vẫn đang được phát triển.

### FreeBSD {#freebsd}

<!-- CL 199919 -->
Go giờ hỗ trợ kiến trúc ARM 64-bit trên FreeBSD 12.0 hoặc mới hơn (cổng `freebsd/arm64`).

### Native Client (NaCl) {#nacl}

<!-- golang.org/issue/30439 -->
Như đã [thông báo](go1.13#ports) trong ghi chú phát hành Go 1.13,
Go 1.14 bỏ hỗ trợ nền tảng Native Client (`GOOS=nacl`).

### Illumos {#illumos}

<!-- CL 203758 -->
Runtime giờ tôn trọng giới hạn CPU của zone (`zone.cpu-cap` resource control) cho `runtime.NumCPU` và giá trị mặc định của `GOMAXPROCS`.

## Công cụ {#tools}

### Lệnh Go {#go-command}

#### Vendoring {#vendor}

<!-- golang.org/issue/33848 -->

Khi module chính chứa thư mục `vendor` cấp cao nhất và tệp `go.mod` của nó chỉ định `go` `1.14` hoặc cao hơn, lệnh `go` giờ mặc định là `-mod=vendor` cho các thao tác chấp nhận cờ đó. Giá trị mới cho cờ đó, `-mod=mod`, khiến lệnh `go` tải module từ module cache (như khi không có thư mục `vendor`).

Khi `-mod=vendor` được đặt (tường minh hay mặc định), lệnh `go` giờ xác minh rằng tệp `vendor/modules.txt` của module chính nhất quán với tệp `go.mod` của nó.

`go` `list` `-m` không còn âm thầm bỏ qua các dependency bắc cầu không cung cấp gói trong thư mục `vendor`. Nó giờ thất bại tường minh nếu `-mod=vendor` được đặt và thông tin được yêu cầu cho module không được đề cập trong `vendor/modules.txt`.

#### Các cờ {#go-flags}

<!-- golang.org/issue/32502, golang.org/issue/30345 -->
Lệnh `go` `get` không còn chấp nhận cờ `-mod`. Trước đây, cài đặt của cờ này hoặc [bị bỏ qua](/issue/30345) hoặc [khiến build thất bại](/issue/32502).

<!-- golang.org/issue/33326 -->
`-mod=readonly` giờ được đặt mặc định khi tệp `go.mod` là chỉ đọc và không có thư mục `vendor` cấp cao nhất.

<!-- golang.org/issue/31481 -->
`-modcacherw` là cờ mới chỉ thị lệnh `go` để giữ các thư mục mới tạo trong module cache với quyền mặc định thay vì làm chúng chỉ đọc.
Việc dùng cờ này làm tăng khả năng các test hoặc công cụ khác vô tình thêm tệp không được bao gồm trong checksum đã xác minh của module. Tuy nhiên, nó cho phép dùng `rm` `-rf` (thay vì `go` `clean` `-modcache`) để xóa module cache.

<!-- golang.org/issue/34506 -->
`-modfile=file` là cờ mới chỉ thị lệnh `go` đọc (và có thể ghi) tệp `go.mod` thay thế thay vì tệp trong thư mục gốc của module. Tệp tên `go.mod` vẫn phải có để xác định thư mục gốc module, nhưng không được truy cập. Khi `-modfile` được chỉ định, tệp `go.sum` thay thế cũng được dùng: đường dẫn của nó được dẫn xuất từ cờ `-modfile` bằng cách cắt đuôi `.mod` và thêm `.sum`.

#### Biến môi trường {#go-env-vars}

<!-- golang.org/issue/32966 -->
`GOINSECURE` là biến môi trường mới chỉ thị lệnh `go` không yêu cầu kết nối HTTPS, và bỏ qua xác thực chứng chỉ, khi lấy trực tiếp một số module từ nguồn của chúng. Giống như biến `GOPRIVATE` hiện có, giá trị của `GOINSECURE` là danh sách phân cách bằng dấu phẩy các mẫu glob.

#### Các lệnh ngoài module {#commands-outside-modules}

<!-- golang.org/issue/32027 -->
Khi chế độ module-aware được bật tường minh (bằng cách đặt `GO111MODULE=on`), hầu hết các lệnh module có chức năng hạn chế hơn nếu không có tệp `go.mod`. Ví dụ, `go` `build`, `go` `run` và các lệnh build khác chỉ có thể build các gói trong thư viện chuẩn và các gói được chỉ định dưới dạng tệp `.go` trên dòng lệnh.

Trước đây, lệnh `go` phân giải từng đường dẫn gói thành phiên bản mới nhất của module nhưng không ghi lại đường dẫn module hay phiên bản. Điều này dẫn đến [các build chậm, không tái lập được](/issue/32027).

`go` `get` tiếp tục hoạt động như trước, cũng như `go` `mod` `download` và `go` `list` `-m` với phiên bản tường minh.

#### Phiên bản `+incompatible` {#incompatible-versions}

<!-- golang.org/issue/34165 -->

Nếu phiên bản mới nhất của module chứa tệp `go.mod`, `go` `get` sẽ không còn nâng cấp lên phiên bản major [không tương thích](/cmd/go/#hdr-Module_compatibility_and_semantic_versioning) của module đó trừ khi phiên bản đó được yêu cầu tường minh hoặc đã được yêu cầu. `go` `list` cũng bỏ qua các phiên bản major không tương thích cho module đó khi lấy trực tiếp từ kiểm soát phiên bản, nhưng có thể bao gồm chúng nếu được báo cáo bởi proxy.

#### Bảo trì tệp `go.mod` {#go.mod}

<!-- golang.org/issue/34822 -->

Các lệnh `go` khác ngoài `go` `mod` `tidy` không còn xóa directive `require` chỉ định phiên bản dependency gián tiếp đã được ngụ ý bởi các dependency (bắc cầu) khác của module chính.

Các lệnh `go` khác ngoài `go` `mod` `tidy` không còn chỉnh sửa tệp `go.mod` nếu các thay đổi chỉ là thẩm mỹ.

Khi `-mod=readonly` được đặt, các lệnh `go` sẽ không còn thất bại do directive `go` bị thiếu hoặc comment `// indirect` có lỗi.

#### Tải xuống module {#module-downloading}

<!-- golang.org/issue/26092 -->
Lệnh `go` giờ hỗ trợ kho lưu trữ Subversion trong chế độ module.

<!-- golang.org/issue/30748 -->
Lệnh `go` giờ bao gồm các đoạn thông báo lỗi văn bản thuần từ các module proxy và các máy chủ HTTP khác.
Thông báo lỗi chỉ được hiển thị nếu nó là UTF-8 hợp lệ và chỉ bao gồm các ký tự đồ họa và dấu cách.

#### Kiểm thử {#go-test}

<!-- golang.org/issue/24929 -->
`go test -v` giờ phát trực tiếp đầu ra `t.Log` ngay khi xảy ra, thay vì ở cuối tất cả các test.

## Runtime {#runtime}

<!-- CL 190098 -->
Bản phát hành này cải thiện hiệu năng của hầu hết các lần dùng `defer` để gánh chịu gần như không có overhead so với gọi trực tiếp hàm được defer. Do đó, `defer` giờ có thể được dùng trong mã nhạy cảm về hiệu năng mà không lo overhead.

<!-- CL 201760, CL 201762 and many others -->
Goroutine giờ có thể bị preempt bất đồng bộ. Do đó, các vòng lặp không có lời gọi hàm không còn có thể làm nghẽn scheduler hoặc trì hoãn đáng kể bộ gom rác. Điều này được hỗ trợ trên tất cả nền tảng ngoại trừ `windows/arm`, `darwin/arm`, `js/wasm` và `plan9/*`.

Hậu quả của việc triển khai preemption là trên hệ thống Unix, bao gồm Linux và macOS, các chương trình được build với Go 1.14 sẽ nhận nhiều tín hiệu hơn so với các chương trình được build với các bản phát hành trước. Điều này có nghĩa là các chương trình sử dụng các gói như [`syscall`](/pkg/syscall/) hoặc [`golang.org/x/sys/unix`](https://godoc.org/golang.org/x/sys/unix) sẽ thấy nhiều lời gọi hệ thống chậm thất bại với lỗi `EINTR` hơn. Những chương trình đó sẽ phải xử lý các lỗi đó theo cách nào đó, thường là lặp lại để thử lại lời gọi hệ thống. Để biết thêm thông tin, xem [`man 7 signal`](https://man7.org/linux/man-pages/man7/signal.7.html) cho hệ thống Linux hoặc tài liệu tương tự cho các hệ thống khác.

<!-- CL 201765, CL 195701 and many others -->
Trình cấp phát trang hiệu quả hơn và gánh chịu tranh chấp lock ít hơn đáng kể ở các giá trị cao của `GOMAXPROCS`. Điều này rõ nhất nhất là độ trễ thấp hơn và thông lượng cao hơn cho các cấp phát lớn được thực hiện song song với tốc độ cao.

<!-- CL 171844 and many others -->
Bộ đếm thời gian nội bộ, được dùng bởi [`time.After`](/pkg/time/#After), [`time.Tick`](/pkg/time/#Tick), [`net.Conn.SetDeadline`](/pkg/net/#Conn) và các hàm tương tự, hiệu quả hơn, với ít tranh chấp lock hơn và ít context switch hơn. Đây là cải tiến hiệu năng không gây ra thay đổi nào hiển thị với người dùng.

## Trình biên dịch {#compiler}

<!-- CL 162237 -->
Bản phát hành này thêm `-d=checkptr` như một tùy chọn compile-time để thêm instrumentation kiểm tra rằng mã Go đang tuân theo các quy tắc an toàn `unsafe.Pointer` một cách động. Tùy chọn này được bật theo mặc định (ngoại trừ trên Windows) với các cờ `-race` hoặc `-msan`, và có thể tắt bằng `-gcflags=all=-d=checkptr=0`.
Cụ thể, `-d=checkptr` kiểm tra những điều sau:

 1. Khi chuyển đổi `unsafe.Pointer` thành `*T`, con trỏ kết quả phải được căn chỉnh phù hợp với `T`.
 2. Nếu kết quả của phép tính con trỏ trỏ vào đối tượng heap Go, một trong các toán hạng kiểu `unsafe.Pointer` phải trỏ vào cùng đối tượng.

Hiện không khuyến nghị dùng `-d=checkptr` trên Windows vì nó gây ra cảnh báo sai trong thư viện chuẩn.

<!-- CL 204338 -->
Trình biên dịch giờ có thể phát ra log có thể đọc bằng máy về các tối ưu hóa chính bằng cờ `-json`, bao gồm inlining, phân tích thoát, loại bỏ kiểm tra giới hạn và loại bỏ kiểm tra nil.

<!-- CL 196959 -->
Chẩn đoán phân tích thoát chi tiết (`-m=2`) giờ hoạt động lại. Điều này đã bị bỏ khỏi triển khai phân tích thoát mới trong bản phát hành trước.

<!-- CL 196217 -->
Tất cả ký hiệu Go trong tệp nhị phân macOS giờ bắt đầu bằng dấu gạch dưới, theo quy ước của nền tảng.

<!-- CL 202117 -->
Bản phát hành này bao gồm hỗ trợ thực nghiệm cho instrumentation coverage do trình biên dịch chèn vào cho fuzzing.
Xem [issue 14565](/issue/14565) để biết thêm chi tiết.
API này có thể thay đổi trong các bản phát hành tương lai.

<!-- CL 174704 -->
<!-- CL 196784 -->
Loại bỏ kiểm tra giới hạn giờ dùng thông tin từ việc tạo slice và có thể loại bỏ kiểm tra cho các chỉ số có kiểu nhỏ hơn `int`.

## Thư viện chuẩn {#library}

### Gói hashing chuỗi byte mới {#hash_maphash}

<!-- golang.org/issue/28322, CL 186877 -->
Go 1.14 bao gồm gói mới [`hash/maphash`](/pkg/hash/maphash/), cung cấp các hàm hash trên chuỗi byte. Các hàm hash này dự định dùng để triển khai bảng hash hoặc các cấu trúc dữ liệu khác cần ánh xạ chuỗi tùy ý hoặc chuỗi byte sang phân phối đồng đều trên các số nguyên 64-bit không dấu.

Các hàm hash có khả năng chống va chạm nhưng không bảo mật mật mã.

Giá trị hash của một chuỗi byte nhất định nhất quán trong một tiến trình, nhưng sẽ khác nhau trong các tiến trình khác nhau.

### Thay đổi nhỏ trong thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ trong thư viện, được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1 trong tâm trí.

#### [crypto/tls](/pkg/crypto/tls/)

<!-- CL 191976 -->
Hỗ trợ cho SSL phiên bản 3.0 (SSLv3) đã bị xóa. Lưu ý rằng SSLv3 là giao thức [bị phá vỡ về mặt mật mã](https://tools.ietf.org/html/rfc7568) có trước TLS.

<!-- CL 191999 -->
TLS 1.3 không thể bị tắt qua biến môi trường `GODEBUG` nữa. Dùng trường [`Config.MaxVersion`](/pkg/crypto/tls/#Config.MaxVersion) để cấu hình phiên bản TLS.

<!-- CL 205059 -->
Khi nhiều chuỗi chứng chỉ được cung cấp qua trường [`Config.Certificates`](/pkg/crypto/tls/#Config.Certificates), chuỗi đầu tiên tương thích với peer giờ được tự động chọn. Điều này cho phép ví dụ cung cấp chứng chỉ ECDSA và RSA, và để gói tự động chọn cái tốt nhất. Lưu ý rằng hiệu năng của việc chọn này sẽ kém trừ khi trường [`Certificate.Leaf`](/pkg/crypto/tls/#Certificate.Leaf) được đặt. Trường [`Config.NameToCertificate`](/pkg/crypto/tls/#Config.NameToCertificate), chỉ hỗ trợ liên kết một chứng chỉ với tên đã cho, giờ bị deprecated và nên để là `nil`. Tương tự, phương thức [`Config.BuildNameToCertificate`](/pkg/crypto/tls/#Config.BuildNameToCertificate), xây dựng trường `NameToCertificate` từ các chứng chỉ leaf, giờ bị deprecated và không nên được gọi.

<!-- CL 175517 -->
Các hàm mới [`CipherSuites`](/pkg/crypto/tls/#CipherSuites) và [`InsecureCipherSuites`](/pkg/crypto/tls/#InsecureCipherSuites) trả về danh sách các cipher suite hiện được triển khai. Hàm mới [`CipherSuiteName`](/pkg/crypto/tls/#CipherSuiteName) trả về tên cho ID cipher suite.

<!-- CL 205058, 205057 -->
Các phương thức mới [`(*ClientHelloInfo).SupportsCertificate`](/pkg/crypto/tls/#ClientHelloInfo.SupportsCertificate) và [`(*CertificateRequestInfo).SupportsCertificate`](/pkg/crypto/tls/#CertificateRequestInfo.SupportsCertificate) cho biết liệu peer có hỗ trợ chứng chỉ nhất định không.

<!-- CL 174329 -->
Gói `tls` không còn hỗ trợ tiện ích mở rộng Next Protocol Negotiation (NPN) kế thừa và giờ chỉ hỗ trợ ALPN. Trong các bản phát hành trước, nó hỗ trợ cả hai. Không có thay đổi API và các ứng dụng sẽ hoạt động giống hệt như trước. Hầu hết các client và server khác đã xóa hỗ trợ NPN để ưu tiên ALPN chuẩn hóa.

<!-- CL 205063, 205062 -->
Chữ ký RSA-PSS giờ được dùng khi được hỗ trợ trong TLS 1.2 handshake. Điều này sẽ không ảnh hưởng đến hầu hết ứng dụng, nhưng các triển khai [`Certificate.PrivateKey`](/pkg/crypto/tls/#Certificate.PrivateKey) tùy chỉnh không hỗ trợ chữ ký RSA-PSS sẽ cần dùng trường [`Certificate.SupportedSignatureAlgorithms`](/pkg/crypto/tls/#Certificate.SupportedSignatureAlgorithms) mới để tắt chúng.

<!-- CL 205059, 205059 -->
[`Config.Certificates`](/pkg/crypto/tls/#Config.Certificates) và [`Config.GetCertificate`](/pkg/crypto/tls/#Config.GetCertificate) giờ đều có thể là nil nếu [`Config.GetConfigForClient`](/pkg/crypto/tls/#Config.GetConfigForClient) được đặt. Nếu callback không trả về chứng chỉ cũng không có lỗi, `unrecognized_name` giờ được gửi.

<!-- CL 205058 -->
Trường mới [`CertificateRequestInfo.Version`](/pkg/crypto/tls/#CertificateRequestInfo.Version) cung cấp phiên bản TLS cho callback chứng chỉ client.

<!-- CL 205068 -->
Các hằng số mới `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256` và `TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256` dùng tên cuối cùng cho các cipher suite trước đây gọi là `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305` và `TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305`.

<!-- crypto/tls -->

#### [crypto/x509](/pkg/crypto/x509/)

<!-- CL 204046 -->
[`Certificate.CreateCRL`](/pkg/crypto/x509/#Certificate.CreateCRL)
giờ hỗ trợ issuer Ed25519.

#### [debug/dwarf](/pkg/debug/dwarf/)

<!-- CL 175138 -->
Gói `debug/dwarf` giờ hỗ trợ đọc DWARF phiên bản 5.

Phương thức mới [`(*Data).AddSection`](/pkg/debug/dwarf/#Data.AddSection) hỗ trợ thêm các section DWARF tùy ý mới từ tệp đầu vào vào `Data` DWARF.

<!-- CL 192698 -->
Phương thức mới [`(*Reader).ByteOrder`](/pkg/debug/dwarf/#Reader.ByteOrder) trả về thứ tự byte của đơn vị biên dịch hiện tại. Điều này có thể dùng để diễn giải các thuộc tính được mã hóa theo thứ tự gốc, chẳng hạn như mô tả vị trí.

<!-- CL 192699 -->
Phương thức mới [`(*LineReader).Files`](/pkg/debug/dwarf/#LineReader.Files) trả về bảng tên tệp từ line reader. Điều này có thể dùng để diễn giải giá trị của các thuộc tính DWARF như `AttrDeclFile`.

<!-- debug/dwarf -->

#### [encoding/asn1](/pkg/encoding/asn1/)

<!-- CL 126624 -->
[`Unmarshal`](/pkg/encoding/asn1/#Unmarshal) giờ hỗ trợ kiểu chuỗi ASN.1 BMPString, được biểu diễn bởi hằng số mới [`TagBMPString`](/pkg/encoding/asn1/#TagBMPString).

<!-- encoding/asn1 -->

#### [encoding/json](/pkg/encoding/json/)

<!-- CL 200677 -->
Kiểu [`Decoder`](/pkg/encoding/json/#Decoder) hỗ trợ phương thức mới [`InputOffset`](/pkg/encoding/json/#Decoder.InputOffset) trả về offset byte trong luồng đầu vào của vị trí decoder hiện tại.

<!-- CL 200217 -->
[`Compact`](/pkg/encoding/json/#Compact) không còn thoát các ký tự `U+2028` và `U+2029`, vốn không bao giờ là tính năng được ghi lại. Để thoát đúng cách, xem [`HTMLEscape`](/pkg/encoding/json/#HTMLEscape).

<!-- CL 195045 -->
[`Number`](/pkg/encoding/json/#Number) không còn chấp nhận các số không hợp lệ, để tuân theo hành vi được ghi lại chính xác hơn. Nếu chương trình cần chấp nhận các số không hợp lệ như chuỗi rỗng, hãy xem xét bọc kiểu bằng [`Unmarshaler`](/pkg/encoding/json/#Unmarshaler).

<!-- CL 200237 -->
[`Unmarshal`](/pkg/encoding/json/#Unmarshal) giờ có thể hỗ trợ các key map với kiểu cơ sở là chuỗi triển khai [`encoding.TextUnmarshaler`](/pkg/encoding/#TextUnmarshaler).

<!-- encoding/json -->

#### [go/build](/pkg/go/build/)

<!-- CL 203820, 211657 -->
Kiểu [`Context`](/pkg/go/build/#Context) có trường mới `Dir` có thể dùng để đặt thư mục làm việc cho build. Mặc định là thư mục hiện tại của tiến trình đang chạy. Trong chế độ module, điều này được dùng để xác định module chính.

<!-- go/build -->

#### [go/doc](/pkg/go/doc/)

<!-- CL 204830 -->
Hàm mới [`NewFromFiles`](/pkg/go/doc/#NewFromFiles) tính tài liệu gói từ danh sách `*ast.File` và liên kết các ví dụ với phần tử gói phù hợp. Thông tin mới có trong trường `Examples` mới trong các kiểu [`Package`](/pkg/go/doc/#Package), [`Type`](/pkg/go/doc/#Type) và [`Func`](/pkg/go/doc/#Func), và trường [`Suffix`](/pkg/go/doc/#Example.Suffix) mới trong kiểu [`Example`](/pkg/go/doc/#Example).

<!-- go/doc -->

#### [io/ioutil](/pkg/io/ioutil/)

<!-- CL 198488 -->
[`TempDir`](/pkg/io/ioutil/#TempDir) giờ có thể tạo các thư mục có tên với tiền tố và hậu tố có thể dự đoán. Cũng như [`TempFile`](/pkg/io/ioutil/#TempFile), nếu pattern chứa '\*', chuỗi ngẫu nhiên thay thế '\*' cuối cùng.

#### [log](/pkg/log/)

<!-- CL 186182 -->
Cờ mới [`Lmsgprefix`](https://tip.golang.org/pkg/log/#pkg-constants) có thể dùng để yêu cầu các hàm logging phát tiền tố đầu ra tùy chọn ngay trước thông báo log thay vì ở đầu dòng.

<!-- log -->

#### [math](/pkg/math/)

<!-- CL 127458 -->
Hàm mới [`FMA`](/pkg/math/#FMA) tính `x*y+z` trong số thực mà không có làm tròn trung gian của phép tính `x*y`. Một số kiến trúc triển khai phép tính này bằng các lệnh phần cứng chuyên dụng để có hiệu năng bổ sung.

<!-- math -->

#### [math/big](/pkg/math/big/)

<!-- CL 164972 -->
Phương thức [`GCD`](/pkg/math/big/#Int.GCD) giờ cho phép đầu vào `a` và `b` là không hoặc âm.

<!-- math/big -->

#### [math/bits](/pkg/math/bits/)

<!-- CL 197838 -->
Các hàm mới [`Rem`](/pkg/math/bits/#Rem), [`Rem32`](/pkg/math/bits/#Rem32) và [`Rem64`](/pkg/math/bits/#Rem64) hỗ trợ tính số dư ngay cả khi số nguyên tràn.

<!-- math/bits -->

#### [mime](/pkg/mime/)

<!-- CL 186927 -->
Kiểu mặc định của tệp `.js` và `.mjs` giờ là `text/javascript` thay vì `application/javascript`. Điều này phù hợp với [bản nháp IETF](https://datatracker.ietf.org/doc/draft-ietf-dispatch-javascript-mjs/) coi `application/javascript` là lỗi thời.

<!-- mime -->

#### [mime/multipart](/pkg/mime/multipart/)

Phương thức mới [`NextRawPart`](/pkg/mime/multipart/#Reader.NextRawPart) của [`Reader`](/pkg/mime/multipart/#Reader) hỗ trợ lấy phần MIME tiếp theo mà không giải mã `quoted-printable` trong suốt.

<!-- mime/multipart -->

#### [net/http](/pkg/net/http/)

<!-- CL 200760 -->
Phương thức mới [`Values`](/pkg/net/http/#Header.Values) của [`Header`](/pkg/net/http/#Header) có thể dùng để lấy tất cả giá trị liên kết với key đã chuẩn hóa.

<!-- CL 61291 -->
Trường mới [`DialTLSContext`](/pkg/net/http/#Transport.DialTLSContext) của [`Transport`](/pkg/net/http/#Transport) có thể dùng để chỉ định hàm dial tùy chọn để tạo kết nối TLS cho các request HTTPS không qua proxy. Trường mới này có thể dùng thay cho [`DialTLS`](/pkg/net/http/#Transport.DialTLS), vốn giờ được coi là deprecated; `DialTLS` vẫn tiếp tục hoạt động, nhưng mã mới nên dùng `DialTLSContext`, cho phép transport hủy dial ngay khi chúng không còn cần thiết.

<!-- CL 192518, CL 194218 -->
Trên Windows, [`ServeFile`](/pkg/net/http/#ServeFile) giờ phục vụ đúng các tệp lớn hơn 2GB.

<!-- net/http -->

#### [net/http/httptest](/pkg/net/http/httptest/)

<!-- CL 201557 -->
Trường mới [`EnableHTTP2`](/pkg/net/http/httptest/#Server.EnableHTTP2) của [`Server`](/pkg/net/http/httptest/#Server) hỗ trợ bật HTTP/2 trên máy chủ test.

<!-- net/http/httptest -->

#### [net/textproto](/pkg/net/textproto/)

<!-- CL 200760 -->
Phương thức mới [`Values`](/pkg/net/textproto/#MIMEHeader.Values) của [`MIMEHeader`](/pkg/net/textproto/#MIMEHeader) có thể dùng để lấy tất cả giá trị liên kết với key đã chuẩn hóa.

<!-- net/textproto -->

#### [net/url](/pkg/net/url/)

<!-- CL 185117 -->
Khi phân tích URL thất bại (ví dụ bởi [`Parse`](/pkg/net/url/#Parse) hoặc [`ParseRequestURI`](/pkg/net/url/#ParseRequestURI)), thông báo [`Error`](/pkg/net/url/#Error.Error) kết quả giờ sẽ trích dẫn URL không thể phân tích được. Điều này cung cấp cấu trúc rõ ràng hơn và nhất quán với các lỗi phân tích khác.

<!-- net/url -->

#### [os/signal](/pkg/os/signal/)

<!-- CL 187739 -->
Trên Windows, các sự kiện `CTRL_CLOSE_EVENT`, `CTRL_LOGOFF_EVENT` và `CTRL_SHUTDOWN_EVENT` giờ tạo ra tín hiệu `syscall.SIGTERM`, tương tự như cách Control-C và Control-Break tạo ra tín hiệu `syscall.SIGINT`.

<!-- os/signal -->

#### [plugin](/pkg/plugin/)

<!-- CL 191617 -->
Gói `plugin` giờ hỗ trợ `freebsd/amd64`.

<!-- plugin -->

#### [reflect](/pkg/reflect/)

<!-- CL 85661 -->
[`StructOf`](/pkg/reflect#StructOf) giờ hỗ trợ tạo các kiểu struct với trường không xuất (unexported), bằng cách đặt trường `PkgPath` trong phần tử `StructField`.

<!-- reflect -->

#### [runtime](/pkg/runtime/)

<!-- CL 200081 -->
`runtime.Goexit` không còn có thể bị hủy bởi `panic`/`recover` đệ quy.

<!-- CL 188297, CL 191785 -->
Trên macOS, `SIGPIPE` không còn được chuyển tiếp đến các handler tín hiệu được cài đặt trước khi Go runtime được khởi tạo. Điều này cần thiết vì macOS gửi `SIGPIPE` [đến luồng chính](/issue/33384) thay vì luồng đang ghi vào pipe đã đóng.

<!-- runtime -->

#### [runtime/pprof](/pkg/runtime/pprof/)

<!-- CL 204636, 205097 -->
Profile được tạo ra không còn bao gồm các pseudo-PC dùng cho các dấu inline. Thông tin ký hiệu của các hàm inline được mã hóa theo [định dạng](https://github.com/google/pprof/blob/5e96527/proto/profile.proto#L177-L184) mà công cụ pprof mong đợi. Đây là bản vá lỗi hồi quy được đưa ra trong các bản phát hành gần đây.

<!-- runtime/pprof -->

#### [strconv](/pkg/strconv/)

Kiểu [`NumError`](/pkg/strconv/#NumError) giờ có phương thức [`Unwrap`](/pkg/strconv/#NumError.Unwrap) có thể dùng để lấy lý do chuyển đổi thất bại. Điều này hỗ trợ dùng giá trị `NumError` với [`errors.Is`](/pkg/errors/#Is) để xem liệu lỗi bên dưới có phải là [`strconv.ErrRange`](/pkg/strconv/#pkg-variables) hay [`strconv.ErrSyntax`](/pkg/strconv/#pkg-variables) không.

<!-- strconv -->

#### [sync](/pkg/sync/)

<!-- CL 200577 -->
Mở khóa `Mutex` bị tranh chấp cao giờ nhường CPU trực tiếp cho goroutine tiếp theo đang chờ `Mutex` đó. Điều này cải thiện đáng kể hiệu năng của các mutex bị tranh chấp cao trên máy có nhiều CPU.

<!-- sync -->

#### [testing](/pkg/testing/)

<!-- CL 201359 -->
Gói testing giờ hỗ trợ hàm cleanup, được gọi sau khi test hoặc benchmark kết thúc, bằng cách gọi [`T.Cleanup`](/pkg/testing#T.Cleanup) hoặc [`B.Cleanup`](/pkg/testing#B.Cleanup) tương ứng.

<!-- testing -->

#### [text/template](/pkg/text/template/)

<!-- CL 206124 -->
Gói text/template giờ báo cáo đúng lỗi khi đối số được đặt trong ngoặc đơn được dùng làm hàm. Điều này thường xuất hiện trong các trường hợp sai như `{{if (eq .F "a") or (eq .F "b")}}`. Nên viết là `{{if or (eq .F "a") (eq .F "b")}}`. Trường hợp sai chưa bao giờ hoạt động như mong đợi, và giờ sẽ được báo cáo với lỗi `can't give argument to non-function`.

<!-- CL 207637 -->
[`JSEscape`](/pkg/text/template/#JSEscape) giờ thoát các ký tự `&` và `=` để giảm thiểu tác động của việc đầu ra của nó bị dùng sai trong các ngữ cảnh HTML.

<!-- text/template -->

#### [unicode](/pkg/unicode/)

Gói [`unicode`](/pkg/unicode/) và hỗ trợ liên quan trong toàn hệ thống đã được nâng cấp từ Unicode 11.0 lên [Unicode 12.0](https://www.unicode.org/versions/Unicode12.0.0/), bổ sung 554 ký tự mới, bao gồm bốn script mới và 61 emoji mới.

<!-- unicode -->
