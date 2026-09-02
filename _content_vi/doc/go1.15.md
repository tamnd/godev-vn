---
title: Ghi chú phát hành Go 1.15
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

## Giới thiệu về Go 1.15 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.15, ra đời sáu tháng sau [Go 1.14](go1.14).
Hầu hết các thay đổi nằm ở phần triển khai của bộ công cụ, runtime và thư viện.
Như thường lệ, bản phát hành vẫn duy trì [cam kết tương thích](/doc/go1compat.html) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

Go 1.15 bao gồm [cải tiến đáng kể cho linker](#linker), cải thiện [cấp phát cho đối tượng nhỏ ở số lõi cao](#runtime), và deprecated [X.509 CommonName](#commonname). `GOPROXY` giờ hỗ trợ bỏ qua các proxy trả về lỗi và một [gói tzdata nhúng mới](#time/tzdata) đã được thêm vào.

## Thay đổi ngôn ngữ {#language}

Không có thay đổi về ngôn ngữ.

## Các nền tảng {#ports}

### Darwin {#darwin}

Như đã [thông báo](go1.14#darwin) trong ghi chú phát hành Go 1.14, Go 1.15 yêu cầu macOS 10.12 Sierra hoặc mới hơn; hỗ trợ cho các phiên bản cũ hơn đã bị ngừng.

<!-- golang.org/issue/37610, golang.org/issue/37611, CL 227582, and CL 227198  -->
Như đã [thông báo](/doc/go1.14#darwin) trong ghi chú phát hành Go 1.14, Go 1.15 bỏ hỗ trợ tệp nhị phân 32-bit trên macOS, iOS, iPadOS, watchOS và tvOS (các cổng `darwin/386` và `darwin/arm`). Go tiếp tục hỗ trợ các cổng 64-bit `darwin/amd64` và `darwin/arm64`.

### Windows {#windows}

<!-- CL 214397 and CL 230217 -->
Go giờ tạo file thực thi Windows ASLR khi cờ `-buildmode=pie` của cmd/link được cung cấp. Lệnh go mặc định dùng `-buildmode=pie` trên Windows.

<!-- CL 227003 -->
Các cờ `-race` và `-msan` giờ luôn bật `-d=checkptr`, kiểm tra việc dùng `unsafe.Pointer`. Trước đây, điều này chỉ áp dụng trên tất cả hệ điều hành trừ Windows.

<!-- CL 211139 -->
Các DLL được Go build không còn khiến tiến trình thoát khi nhận tín hiệu (như Ctrl-C tại terminal).

### Android {#android}

<!-- CL 235017, golang.org/issue/38838 -->
Khi liên kết các tệp nhị phân cho Android, Go 1.15 chọn tường minh linker `lld` có trong các phiên bản NDK gần đây. Linker `lld` tránh được crash trên một số thiết bị, và được lên kế hoạch trở thành linker NDK mặc định trong phiên bản NDK tương lai.

### OpenBSD {#openbsd}

<!-- CL 234381 -->
Go 1.15 thêm hỗ trợ cho OpenBSD 6.7 trên `GOARCH=arm` và `GOARCH=arm64`. Các phiên bản Go trước đã hỗ trợ OpenBSD 6.7 trên `GOARCH=386` và `GOARCH=amd64`.

### RISC-V {#riscv}

<!-- CL 226400, CL 226206, and others -->
Đã có tiến bộ trong việc cải thiện tính ổn định và hiệu năng của cổng 64-bit RISC-V trên Linux (`GOOS=linux`, `GOARCH=riscv64`). Nó giờ cũng hỗ trợ preemption bất đồng bộ.

### 386 {#386}

<!-- golang.org/issue/40255 -->
Go 1.15 là bản phát hành cuối cùng hỗ trợ phần cứng dấu phẩy động x87 (`GO386=387`). Các bản phát hành tương lai sẽ yêu cầu ít nhất hỗ trợ SSE2 trên 386, nâng yêu cầu `GOARCH=386` tối thiểu của Go lên Intel Pentium 4 (phát hành năm 2000) hoặc AMD Opteron/Athlon 64 (phát hành năm 2003).

## Công cụ {#tools}

### Lệnh Go {#go-command}

<!-- golang.org/issue/37367 -->
Biến môi trường `GOPROXY` giờ hỗ trợ bỏ qua các proxy trả về lỗi. Các URL proxy giờ có thể phân cách bằng dấu phẩy (`,`) hoặc dấu pipe (`|`). Nếu URL proxy được theo sau bởi dấu phẩy, lệnh `go` chỉ thử proxy tiếp theo trong danh sách sau phản hồi HTTP 404 hoặc 410. Nếu URL proxy được theo sau bởi dấu pipe, lệnh `go` sẽ thử proxy tiếp theo trong danh sách sau bất kỳ lỗi nào. Lưu ý rằng giá trị mặc định của `GOPROXY` vẫn là `https://proxy.golang.org,direct`, vốn không rơi về `direct` trong trường hợp có lỗi.

#### `go` `test` {#go-test}

<!-- https://golang.org/issue/36134 -->
Thay đổi cờ `-timeout` giờ làm mất hiệu lực kết quả test được cache. Kết quả được cache cho lần chạy test với timeout dài sẽ không còn được tính là pass khi `go` `test` được gọi lại với timeout ngắn.

#### Phân tích cờ {#go-flag-parsing}

<!-- https://golang.org/cl/211358 -->
Nhiều vấn đề phân tích cờ trong `go` `test` và `go` `vet` đã được sửa. Đáng chú ý, các cờ được chỉ định trong `GOFLAGS` được xử lý nhất quán hơn, và cờ `-outputdir` giờ diễn giải các đường dẫn tương đối so với thư mục làm việc của lệnh `go` (thay vì thư mục làm việc của mỗi test riêng lẻ).

#### Module cache {#module-cache}

<!-- https://golang.org/cl/219538 -->
Vị trí của module cache giờ có thể được đặt bằng biến môi trường `GOMODCACHE`. Giá trị mặc định của `GOMODCACHE` là `GOPATH[0]/pkg/mod`, vị trí của module cache trước thay đổi này.

<!-- https://golang.org/cl/221157 -->
Giờ có giải pháp tạm thời cho các lỗi "Access is denied" của Windows trong các lệnh `go` truy cập module cache, do các chương trình bên ngoài đang scan đồng thời hệ thống tệp (xem [issue #36568](/issue/36568)). Giải pháp này không được bật mặc định vì không an toàn khi dùng đồng thời với module cache khi Go phiên bản 1.14.2 và 1.13.10 trở xuống cùng chạy. Có thể bật bằng cách đặt tường minh biến môi trường `GODEBUG=modcacheunzipinplace=1`.

### Vet {#vet}

#### Cảnh báo mới cho string(x) {#vet-string-int}

<!-- CL 212919, 232660 -->
Công cụ vet giờ cảnh báo về các chuyển đổi dạng `string(x)` trong đó `x` có kiểu số nguyên khác với `rune` hoặc `byte`. Kinh nghiệm với Go cho thấy nhiều chuyển đổi dạng này nhầm lẫn rằng `string(x)` cho kết quả là biểu diễn chuỗi của số nguyên `x`. Thực ra nó cho kết quả là chuỗi chứa mã hóa UTF-8 của giá trị `x`. Ví dụ, `string(9786)` không cho kết quả là chuỗi `"9786"`; nó cho kết quả là chuỗi `"\xe2\x98\xba"`, hay `"☺"`.

Mã dùng đúng `string(x)` có thể viết lại thành `string(rune(x))`. Hoặc, trong một số trường hợp, gọi `utf8.EncodeRune(buf, x)` với slice byte phù hợp `buf` có thể là giải pháp đúng. Mã khác rất có thể nên dùng `strconv.Itoa` hoặc `fmt.Sprint`.

Kiểm tra vet mới này được bật mặc định khi dùng `go` `test`.

Chúng tôi đang cân nhắc cấm chuyển đổi này trong bản phát hành Go tương lai. Nghĩa là ngôn ngữ sẽ thay đổi để chỉ cho phép `string(x)` cho số nguyên `x` khi kiểu của `x` là `rune` hoặc `byte`. Thay đổi ngôn ngữ như vậy sẽ không tương thích ngược. Chúng tôi đang dùng kiểm tra vet này như bước thử nghiệm đầu tiên hướng tới thay đổi ngôn ngữ.

#### Cảnh báo mới cho chuyển đổi interface không thể thực hiện {#vet-impossible-interface}

<!-- CL 218779, 232660 -->
Công cụ vet giờ cảnh báo về các type assertion từ một kiểu interface sang kiểu interface khác khi type assertion sẽ luôn thất bại. Điều này sẽ xảy ra nếu cả hai kiểu interface đều triển khai phương thức cùng tên nhưng với chữ ký kiểu khác nhau.

Không có lý do gì để viết type assertion luôn thất bại, vì vậy bất kỳ mã nào kích hoạt kiểm tra vet này nên được viết lại.

Kiểm tra vet mới này được bật mặc định khi dùng `go` `test`.

Chúng tôi đang cân nhắc cấm các type assertion interface không thể thực hiện trong bản phát hành Go tương lai. Thay đổi ngôn ngữ như vậy sẽ không tương thích ngược. Chúng tôi đang dùng kiểm tra vet này như bước thử nghiệm đầu tiên hướng tới thay đổi ngôn ngữ.

## Runtime {#runtime}

<!-- CL 221779 -->
Nếu `panic` được gọi với giá trị có kiểu dẫn xuất từ bất kỳ kiểu nào trong số: `bool`, `complex64`, `complex128`, `float32`, `float64`, `int`, `int8`, `int16`, `int32`, `int64`, `string`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`, thì giá trị sẽ được in ra, thay vì chỉ in địa chỉ của nó. Trước đây, điều này chỉ đúng với các giá trị chính xác những kiểu này.

<!-- CL 228900 -->
Trên hệ thống Unix, nếu lệnh `kill` hoặc lời gọi hệ thống `kill` được dùng để gửi tín hiệu `SIGSEGV`, `SIGBUS` hoặc `SIGFPE` đến chương trình Go, và tín hiệu không được xử lý qua [`os/signal.Notify`](/pkg/os/signal/#Notify), chương trình Go giờ sẽ crash đáng tin cậy với stack trace. Trong các bản phát hành trước, hành vi là không thể đoán trước.

<!-- CL 221182, CL 229998 -->
Cấp phát đối tượng nhỏ giờ hoạt động tốt hơn nhiều ở số lõi cao, và có độ trễ tệ nhất thấp hơn.

<!-- CL 216401 -->
Chuyển đổi giá trị số nguyên nhỏ thành giá trị interface không còn gây ra cấp phát.

<!-- CL 216818 -->
Các nhận không chặn trên kênh đã đóng giờ hoạt động tốt như các nhận không chặn trên kênh mở.

## Trình biên dịch {#compiler}

<!-- CL 229578 -->
[Quy tắc an toàn](/pkg/unsafe/#Pointer) của gói `unsafe` cho phép chuyển đổi `unsafe.Pointer` thành `uintptr` khi gọi một số hàm nhất định. Trước đây, trong một số trường hợp, trình biên dịch cho phép nhiều chuyển đổi nối tiếp (ví dụ: `syscall.Syscall(…,` `uintptr(uintptr(ptr)),` `…)`). Trình biên dịch giờ yêu cầu đúng một chuyển đổi. Mã dùng nhiều chuyển đổi nên được cập nhật để thỏa mãn quy tắc an toàn.

<!-- CL 230544, CL 231397 -->
Go 1.15 giảm kích thước tệp nhị phân điển hình khoảng 5% so với Go 1.14 bằng cách loại bỏ một số kiểu metadata GC và tích cực hơn trong việc loại bỏ metadata kiểu không dùng.

<!-- CL 219357, CL 231600 -->
Bộ công cụ giờ giảm nhẹ [lỗi CPU Intel SKX102](https://www.intel.com/content/www/us/en/support/articles/000055650/processors.html) trên `GOARCH=amd64` bằng cách căn chỉnh các hàm theo ranh giới 32 byte và đệm lệnh jump. Mặc dù đệm này làm tăng kích thước tệp nhị phân, điều này được bù đắp hơn bởi các cải thiện kích thước tệp nhị phân đã đề cập ở trên.

<!-- CL 222661 -->
Go 1.15 thêm cờ `-spectre` vào cả trình biên dịch và assembler, để cho phép bật các biện pháp giảm thiểu Spectre. Chúng hầu như không bao giờ cần thiết và được cung cấp chủ yếu như cơ chế "phòng thủ theo chiều sâu". Xem [trang wiki Spectre](/wiki/Spectre) để biết chi tiết.

<!-- CL 228578 -->
Trình biên dịch giờ từ chối các directive trình biên dịch `//go:` không có ý nghĩa với khai báo được áp dụng với lỗi "misplaced compiler directive". Các directive áp dụng sai như vậy bị hỏng trước đây, nhưng trình biên dịch âm thầm bỏ qua.

<!-- CL 206658, CL 205066 -->
Log tối ưu hóa `-json` của trình biên dịch giờ báo cáo các bản sao lớn (>= 128 byte) và bao gồm giải thích về các quyết định phân tích thoát.

## Linker {#linker}

Bản phát hành này bao gồm các cải tiến đáng kể cho linker Go, giảm mức sử dụng tài nguyên linker (cả thời gian và bộ nhớ) và cải thiện tính mạnh mẽ/khả năng bảo trì của mã.

Với tập hợp đại diện các chương trình Go lớn, liên kết nhanh hơn 20% và yêu cầu ít hơn 30% bộ nhớ trung bình, với các hệ điều hành dựa trên `ELF` (Linux, FreeBSD, NetBSD, OpenBSD, Dragonfly và Solaris) chạy trên kiến trúc `amd64`, với cải tiến khiêm tốn hơn cho các kết hợp kiến trúc/hệ điều hành khác.

Những người đóng góp chính cho hiệu năng linker tốt hơn là định dạng object file được thiết kế lại, và việc cải tổ các giai đoạn nội bộ để tăng tính đồng thời (ví dụ: áp dụng các relocation cho các ký hiệu song song). Các object file trong Go 1.15 lớn hơn một chút so với các file tương đương 1.14.

Những thay đổi này là một phần của dự án nhiều bản phát hành để [hiện đại hóa linker Go](/s/better-linker), nghĩa là sẽ có thêm cải tiến linker trong các bản phát hành tương lai.

<!-- CL 207877 -->
Linker giờ mặc định dùng chế độ liên kết nội bộ cho `-buildmode=pie` trên `linux/amd64` và `linux/arm64`, vì vậy các cấu hình này không còn yêu cầu C linker. Chế độ liên kết ngoài (là mặc định trong Go 1.14 cho `-buildmode=pie`) vẫn có thể được yêu cầu bằng cờ `-ldflags=-linkmode=external`.

## Objdump {#objdump}

<!-- CL 225459 -->
Công cụ [objdump](/cmd/objdump/) giờ hỗ trợ dịch hợp ngữ trong cú pháp GNU assembler với cờ `-gnu`.

## Thư viện chuẩn {#library}

### Gói tzdata nhúng mới {#time_tzdata}

<!-- CL 224588 -->
Go 1.15 bao gồm gói mới [`time/tzdata`](/pkg/time/tzdata/), cho phép nhúng cơ sở dữ liệu múi giờ vào chương trình. Import gói này (dạng `import _ "time/tzdata"`) cho phép chương trình tìm thông tin múi giờ ngay cả khi cơ sở dữ liệu múi giờ không có sẵn trên hệ thống cục bộ. Bạn cũng có thể nhúng cơ sở dữ liệu múi giờ bằng cách build với `-tags timetzdata`. Cả hai cách đều tăng kích thước chương trình khoảng 800 KB.

### Cgo {#cgo}

<!-- CL 235817 -->
Go 1.15 sẽ dịch kiểu C `EGLConfig` thành kiểu Go `uintptr`. Thay đổi này tương tự như cách Go 1.12 và mới hơn xử lý `EGLDisplay`, CoreFoundation của Darwin và các kiểu JNI của Java. Xem [tài liệu cgo](/cmd/cgo/#hdr-Special_cases) để biết thêm thông tin.

<!-- CL 250940 -->
Trong Go 1.15.3 và mới hơn, cgo sẽ không cho phép mã Go cấp phát kiểu struct không xác định (struct C được định nghĩa chỉ là `struct S;` hoặc tương tự) trên stack hoặc heap. Mã Go chỉ được phép dùng con trỏ đến các kiểu đó. Cấp phát instance của struct như vậy và truyền con trỏ, hoặc giá trị struct đầy đủ, cho mã C luôn không an toàn và khó hoạt động đúng; giờ bị cấm. Cách sửa là viết lại mã Go để chỉ dùng con trỏ, hoặc đảm bảo mã Go thấy định nghĩa đầy đủ của struct bằng cách bao gồm header C phù hợp.

### Deprecated X.509 CommonName {#commonname}

<!-- CL 231379 -->
Hành vi kế thừa đã deprecated của việc xử lý trường `CommonName` trên chứng chỉ X.509 như tên host khi không có Subject Alternative Names giờ bị tắt mặc định. Có thể bật lại tạm thời bằng cách thêm giá trị `x509ignoreCN=0` vào biến môi trường `GODEBUG`.

Lưu ý rằng nếu `CommonName` là tên host không hợp lệ, nó luôn bị bỏ qua, bất kể cài đặt `GODEBUG`. Tên không hợp lệ bao gồm những tên có ký tự nào khác ngoài chữ cái, chữ số, dấu gạch ngang và dấu gạch dưới, và những tên có nhãn trống hoặc dấu chấm ở cuối.

### Thay đổi nhỏ trong thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ trong thư viện, được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1 trong tâm trí.

#### [bufio](/pkg/bufio/)

<!-- CL 225357, CL 225557 -->
Khi [`Scanner`](/pkg/bufio/#Scanner) được dùng với [`io.Reader`](/pkg/io/#Reader) không hợp lệ vốn không đúng trả về số âm từ `Read`, `Scanner` sẽ không còn panic, mà thay vào đó trả về lỗi mới [`ErrBadReadCount`](/pkg/bufio/#ErrBadReadCount).

<!-- bufio -->

#### [context](/pkg/context/)

<!-- CL 223777 -->
Tạo `Context` dẫn xuất bằng cách dùng parent nil giờ bị cấm tường minh. Bất kỳ nỗ lực nào như vậy với các hàm [`WithValue`](/pkg/context/#WithValue), [`WithDeadline`](/pkg/context/#WithDeadline) hoặc [`WithCancel`](/pkg/context/#WithCancel) sẽ gây ra panic.

<!-- context -->

#### [crypto](/pkg/crypto/)

<!-- CL 231417, CL 225460 -->
Các kiểu `PrivateKey` và `PublicKey` trong các gói [`crypto/rsa`](/pkg/crypto/rsa/), [`crypto/ecdsa`](/pkg/crypto/ecdsa/) và [`crypto/ed25519`](/pkg/crypto/ed25519/) giờ có phương thức `Equal` để so sánh các khóa về sự tương đương hoặc tạo các interface an toàn kiểu cho khóa công khai. Chữ ký phương thức tương thích với [định nghĩa tương đương của `go-cmp`](https://pkg.go.dev/github.com/google/go-cmp/cmp#Equal).

<!-- CL 224937 -->
[`Hash`](/pkg/crypto/#Hash) giờ triển khai [`fmt.Stringer`](/pkg/fmt/#Stringer).

<!-- crypto -->

#### [crypto/ecdsa](/pkg/crypto/ecdsa/)

<!-- CL 217940 -->
Các hàm mới [`SignASN1`](/pkg/crypto/ecdsa/#SignASN1) và [`VerifyASN1`](/pkg/crypto/ecdsa/#VerifyASN1) cho phép tạo và xác minh chữ ký ECDSA trong mã hóa ASN.1 DER chuẩn.

<!-- crypto/ecdsa -->

#### [crypto/elliptic](/pkg/crypto/elliptic/)

<!-- CL 202819 -->
Các hàm mới [`MarshalCompressed`](/pkg/crypto/elliptic/#MarshalCompressed) và [`UnmarshalCompressed`](/pkg/crypto/elliptic/#UnmarshalCompressed) cho phép mã hóa và giải mã các điểm đường cong elliptic NIST ở định dạng nén.

<!-- crypto/elliptic -->

#### [crypto/rsa](/pkg/crypto/rsa/)

<!-- CL 226203 -->
[`VerifyPKCS1v15`](/pkg/crypto/rsa/#VerifyPKCS1v15) giờ từ chối các chữ ký ngắn không hợp lệ với các số 0 dẫn đầu bị thiếu, theo RFC 8017.

<!-- crypto/rsa -->

#### [crypto/tls](/pkg/crypto/tls/)

<!-- CL 214977 -->
Kiểu mới [`Dialer`](/pkg/crypto/tls/#Dialer) và phương thức [`DialContext`](/pkg/crypto/tls/#Dialer.DialContext) của nó cho phép dùng context để kết nối và bắt tay với máy chủ TLS.

<!-- CL 229122 -->
Callback mới [`VerifyConnection`](/pkg/crypto/tls/#Config.VerifyConnection) trên kiểu [`Config`](/pkg/crypto/tls/#Config) cho phép logic xác minh tùy chỉnh cho mọi kết nối. Nó có quyền truy cập vào [`ConnectionState`](/pkg/crypto/tls/#ConnectionState) bao gồm chứng chỉ peer, SCT và phản hồi OCSP được đính kèm.

<!-- CL 230679 -->
Các khóa session ticket được tự động tạo giờ tự động xoay mỗi 24 giờ, với thời hạn sống 7 ngày, để hạn chế tác động của chúng đến forward secrecy.

<!-- CL 231317 -->
Thời hạn sống session ticket trong TLS 1.2 và trước đó, nơi các khóa session được tái sử dụng cho các kết nối được tiếp tục, giờ bị giới hạn trong 7 ngày, cũng để hạn chế tác động của chúng đến forward secrecy.

<!-- CL 231038 -->
Các kiểm tra bảo vệ downgrade phía client được chỉ định trong RFC 8446 giờ được thực thi. Điều này có khả năng gây ra lỗi kết nối cho client gặp phải middlebox hoạt động như các cuộc tấn công downgrade không được phép.

<!-- CL 208226 -->
[`SignatureScheme`](/pkg/crypto/tls/#SignatureScheme), [`CurveID`](/pkg/crypto/tls/#CurveID) và [`ClientAuthType`](/pkg/crypto/tls/#ClientAuthType) giờ triển khai [`fmt.Stringer`](/pkg/fmt/#Stringer).

<!-- CL 236737 -->
Các trường `OCSPResponse` và `SignedCertificateTimestamps` của [`ConnectionState`](/pkg/crypto/tls/#ConnectionState) giờ được điền lại trên các kết nối được tiếp tục phía client.

<!-- CL 227840 -->
[`tls.Conn`](/pkg/crypto/tls/#Conn) giờ trả về lỗi mờ trên các kết nối bị hỏng vĩnh viễn, bao bọc [`net.Error`](/pkg/net/http/#Error) tạm thời. Để truy cập `net.Error` gốc, dùng [`errors.As`](/pkg/errors/#As) (hoặc [`errors.Unwrap`](/pkg/errors/#Unwrap)) thay vì type assertion.

<!-- crypto/tls -->

#### [crypto/x509](/pkg/crypto/x509/)

<!-- CL 231378, CL 231380, CL 231381 -->
Nếu tên trên chứng chỉ hoặc tên đang được xác minh (với [`VerifyOptions.DNSName`](/pkg/crypto/x509/#VerifyOptions.DNSName) hoặc [`VerifyHostname`](/pkg/crypto/x509/#Certificate.VerifyHostname)) không hợp lệ, chúng giờ sẽ được so sánh không phân biệt chữ hoa chữ thường mà không xử lý thêm (không áp dụng ký tự đại diện hoặc xóa dấu chấm ở cuối). Tên không hợp lệ bao gồm những tên có ký tự nào khác ngoài chữ cái, chữ số, dấu gạch ngang và dấu gạch dưới, những tên có nhãn trống, và tên trên chứng chỉ có dấu chấm ở cuối.

<!-- CL 217298 -->
Hàm mới [`CreateRevocationList`](/pkg/crypto/x509/#CreateRevocationList) và kiểu [`RevocationList`](/pkg/crypto/x509/#RevocationList) cho phép tạo Certificate Revocation Lists X.509 v2 tuân thủ RFC 5280.

<!-- CL 227098 -->
[`CreateCertificate`](/pkg/crypto/x509/#CreateCertificate) giờ tự động tạo `SubjectKeyId` nếu template là CA và không chỉ định tường minh.

<!-- CL 228777 -->
[`CreateCertificate`](/pkg/crypto/x509/#CreateCertificate) giờ trả về lỗi nếu template chỉ định `MaxPathLen` nhưng không phải CA.

<!-- CL 205237 -->
Trên hệ thống Unix khác macOS, biến môi trường `SSL_CERT_DIR` giờ có thể là danh sách phân cách bằng dấu hai chấm.

<!-- CL 227037 -->
Trên macOS, các tệp nhị phân giờ luôn được liên kết với `Security.framework` để trích xuất trusted roots của hệ thống, bất kể cgo có sẵn hay không. Hành vi kết quả sẽ nhất quán hơn với bộ xác minh của hệ điều hành.

<!-- crypto/x509 -->

#### [crypto/x509/pkix](/pkg/crypto/x509/pkix/)

<!-- CL 229864, CL 240543 -->
[`Name.String`](/pkg/crypto/x509/pkix/#Name.String) giờ in các thuộc tính không chuẩn từ [`Names`](/pkg/crypto/x509/pkix/#Name.Names) nếu [`ExtraNames`](/pkg/crypto/x509/pkix/#Name.ExtraNames) là nil.

<!-- crypto/x509/pkix -->

#### [database/sql](/pkg/database/sql/)

<!-- CL 145758 -->
Phương thức mới [`DB.SetConnMaxIdleTime`](/pkg/database/sql/#DB.SetConnMaxIdleTime) cho phép xóa kết nối khỏi connection pool sau khi nó ở trạng thái rảnh trong một khoảng thời gian, bất kể tổng thời gian sống của kết nối. Trường [`DBStats.MaxIdleTimeClosed`](/pkg/database/sql/#DBStats.MaxIdleTimeClosed) hiển thị tổng số kết nối bị đóng do `DB.SetConnMaxIdleTime`.

<!-- CL 214317 -->
Getter mới [`Row.Err`](/pkg/database/sql/#Row.Err) cho phép kiểm tra lỗi truy vấn mà không gọi `Row.Scan`.

<!-- database/sql -->

#### [database/sql/driver](/pkg/database/sql/driver/)

<!-- CL 174122 -->
Interface mới [`Validator`](/pkg/database/sql/driver/#Validator) có thể được triển khai bởi `Conn` để cho phép driver báo hiệu liệu kết nối có hợp lệ hay nên bị loại bỏ.

<!-- database/sql/driver -->

#### [debug/pe](/pkg/debug/pe/)

<!-- CL 222637 -->
Gói giờ định nghĩa các hằng số `IMAGE_FILE`, `IMAGE_SUBSYSTEM` và `IMAGE_DLLCHARACTERISTICS` được dùng bởi định dạng tệp PE.

<!-- debug/pe -->

#### [encoding/asn1](/pkg/encoding/asn1/)

<!-- CL 226984 -->
[`Marshal`](/pkg/encoding/asn1/#Marshal) giờ sắp xếp các thành phần của SET OF theo X.690 DER.

<!-- CL 227320 -->
[`Unmarshal`](/pkg/encoding/asn1/#Unmarshal) giờ từ chối các tag và Object Identifier không được mã hóa tối thiểu theo X.690 DER.

<!-- encoding/asn1 -->

#### [encoding/json](/pkg/encoding/json/)

<!-- CL 199837 -->
Gói giờ có giới hạn nội bộ về độ sâu lồng tối đa khi giải mã. Điều này giảm khả năng đầu vào lồng sâu có thể dùng lượng lớn bộ nhớ stack, hoặc thậm chí gây ra panic "goroutine stack exceeds limit".

<!-- encoding/json -->

#### [flag](/pkg/flag/)

<!-- CL 221427 -->
Khi gói `flag` thấy `-h` hoặc `-help`, và các cờ đó không được định nghĩa, nó giờ in thông báo sử dụng. Nếu [`FlagSet`](/pkg/flag/#FlagSet) được tạo với [`ExitOnError`](/pkg/flag/#ExitOnError), [`FlagSet.Parse`](/pkg/flag/#FlagSet.Parse) sau đó sẽ thoát với trạng thái 2. Trong bản phát hành này, trạng thái thoát cho `-h` hoặc `-help` đã được thay đổi thành 0. Điều này đặc biệt áp dụng cho xử lý mặc định của các cờ dòng lệnh.

#### [fmt](/pkg/fmt/)

<!-- CL 215001 -->
Các verb in `%#g` và `%#G` giờ giữ lại các số 0 ở cuối cho các giá trị số thực.

<!-- fmt -->

#### [go/format](/pkg/go/format/)

<!-- golang.org/issue/37476, CL 231461, CL 240683 -->
Các hàm [`Source`](/pkg/go/format/#Source) và [`Node`](/pkg/go/format/#Node) giờ chuẩn hóa tiền tố literal số và số mũ như một phần của định dạng mã nguồn Go. Điều này khớp với hành vi của lệnh [`gofmt`](/pkg/cmd/gofmt/) như đã triển khai [từ Go 1.13](/doc/go1.13#gofmt).

<!-- go/format -->

#### [html/template](/pkg/html/template/)

<!-- CL 226097 -->
Gói giờ dùng Unicode escapes (`\uNNNN`) trong tất cả ngữ cảnh JavaScript và JSON. Điều này sửa các lỗi thoát trong ngữ cảnh `application/ld+json` và `application/json`.

<!-- html/template -->

#### [io/ioutil](/pkg/io/ioutil/)

<!-- CL 212597 -->
[`TempDir`](/pkg/io/ioutil/#TempDir) và [`TempFile`](/pkg/io/ioutil/#TempFile) giờ từ chối các pattern chứa dấu phân cách đường dẫn. Nghĩa là các lời gọi như `ioutil.TempFile("/tmp",` `"../base*")` sẽ không còn thành công. Điều này ngăn việc duyệt thư mục không mong muốn.

<!-- io/ioutil -->

#### [math/big](/pkg/math/big/)

<!-- CL 230397 -->
Phương thức mới [`Int.FillBytes`](/pkg/math/big/#Int.FillBytes) cho phép serialize vào các slice byte đã cấp phát trước có kích thước cố định.

<!-- math/big -->

#### [math/cmplx](/pkg/math/cmplx/)

<!-- CL 220689 -->
Các hàm trong gói này đã được cập nhật để tuân thủ tiêu chuẩn C99 (Annex G IEC 60559-compatible complex arithmetic) liên quan đến xử lý các đối số đặc biệt như vô cực, NaN và signed zero.

<!-- math/cmplx-->

#### [net](/pkg/net/)

<!-- CL 228645 -->
Nếu thao tác I/O vượt quá deadline đặt bởi phương thức [`Conn.SetDeadline`](/pkg/net/#Conn), `Conn.SetReadDeadline` hoặc `Conn.SetWriteDeadline`, nó giờ sẽ trả về lỗi là hoặc bao bọc [`os.ErrDeadlineExceeded`](/pkg/os/#ErrDeadlineExceeded). Điều này có thể dùng để phát hiện đáng tin cậy liệu lỗi có do vượt quá deadline không. Các bản phát hành trước khuyến nghị gọi phương thức `Timeout` trên lỗi, nhưng các thao tác I/O có thể trả về lỗi mà `Timeout` trả về `true` mặc dù deadline không bị vượt quá.

<!-- CL 228641 -->
Phương thức mới [`Resolver.LookupIP`](/pkg/net/#Resolver.LookupIP) hỗ trợ tra cứu IP cụ thể theo mạng và chấp nhận context.

#### [net/http](/pkg/net/http/)

<!-- CL 231418, CL 231419 -->
Phân tích giờ nghiêm ngặt hơn như biện pháp cứng hóa chống các cuộc tấn công request smuggling: khoảng trắng không ASCII không còn được cắt như SP và HTAB, và hỗ trợ cho `Transfer-Encoding` "`identity`" đã bị bỏ.

<!-- net/http -->

#### [net/http/httputil](/pkg/net/http/httputil/)

<!-- CL 230937 -->
[`ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy) giờ hỗ trợ không sửa đổi header `X-Forwarded-For` khi mục bản đồ `Request.Header` đầu vào cho trường đó là `nil`.

<!-- CL 224897 -->
Khi request Switching Protocol (như WebSocket) được xử lý bởi [`ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy) bị hủy, kết nối backend giờ được đóng đúng cách.

#### [net/http/pprof](/pkg/net/http/pprof/)

<!-- CL 147598, CL 229537 -->
Tất cả endpoint profile giờ hỗ trợ tham số "`seconds`". Khi có, endpoint profile trong số giây được chỉ định và báo cáo sự khác biệt. Ý nghĩa của tham số "`seconds`" trong profile `cpu` và endpoint trace không thay đổi.

#### [net/url](/pkg/net/url/)

<!-- CL 227645 -->
Trường mới `RawFragment` và phương thức mới [`EscapedFragment`](/pkg/net/url/#URL.EscapedFragment) của [`URL`](/pkg/net/url/#URL) cung cấp chi tiết và kiểm soát về mã hóa chính xác của một fragment cụ thể. Chúng tương tự với `RawPath` và [`EscapedPath`](/pkg/net/url/#URL.EscapedPath).

<!-- CL 207082 -->
Phương thức mới [`Redacted`](/pkg/net/url/#URL.Redacted) của [`URL`](/pkg/net/url/#URL) trả về URL ở dạng chuỗi với bất kỳ mật khẩu nào được thay thế bằng `xxxxx`.

#### [os](/pkg/os/)

<!-- CL -->
Nếu thao tác I/O vượt quá deadline đặt bởi phương thức [`File.SetDeadline`](/pkg/os/#File.SetDeadline), [`File.SetReadDeadline`](/pkg/os/#File.SetReadDeadline) hoặc [`File.SetWriteDeadline`](/pkg/os/#File.SetWriteDeadline), nó giờ sẽ trả về lỗi là hoặc bao bọc [`os.ErrDeadlineExceeded`](/pkg/os/#ErrDeadlineExceeded). Điều này có thể dùng để phát hiện đáng tin cậy liệu lỗi có do vượt quá deadline không. Các bản phát hành trước khuyến nghị gọi phương thức `Timeout` trên lỗi, nhưng các thao tác I/O có thể trả về lỗi mà `Timeout` trả về `true` mặc dù deadline không bị vượt quá.

<!-- CL 232862 -->
Các gói `os` và `net` giờ tự động thử lại các lời gọi hệ thống thất bại với `EINTR`. Trước đây điều này dẫn đến lỗi giả, trở nên phổ biến hơn trong Go 1.14 với việc bổ sung preemption bất đồng bộ. Giờ điều này được xử lý trong suốt.

<!-- CL 229101 -->
Kiểu [`os.File`](/pkg/os/#File) giờ hỗ trợ phương thức [`ReadFrom`](/pkg/os/#File.ReadFrom). Điều này cho phép dùng lời gọi hệ thống `copy_file_range` trên một số hệ thống khi dùng [`io.Copy`](/pkg/io/#Copy) để sao chép dữ liệu từ một `os.File` sang cái khác. Hệ quả là [`io.CopyBuffer`](/pkg/io/#CopyBuffer) sẽ không luôn dùng buffer được cung cấp khi sao chép vào `os.File`. Nếu chương trình muốn ép dùng buffer được cung cấp, có thể thực hiện bằng cách viết `io.CopyBuffer(struct{ io.Writer }{dst}, src, buf)`.

#### [plugin](/pkg/plugin/)

<!-- CL 182959 -->
Tạo DWARF giờ được hỗ trợ (và bật mặc định) cho `-buildmode=plugin` trên macOS.

<!-- CL 191617 -->
Build với `-buildmode=plugin` giờ được hỗ trợ trên `freebsd/amd64`.

#### [reflect](/pkg/reflect/)

<!-- CL 228902 -->
Gói `reflect` giờ không cho phép truy cập phương thức của tất cả các trường không xuất, trong khi trước đây cho phép truy cập các trường nhúng không xuất. Mã phụ thuộc vào hành vi trước đây nên được cập nhật để thay vào đó truy cập phương thức được promote tương ứng của biến bao.

#### [regexp](/pkg/regexp/)

<!-- CL 187919 -->
Phương thức mới [`Regexp.SubexpIndex`](/pkg/regexp/#Regexp.SubexpIndex) trả về chỉ số của biểu thức con đầu tiên với tên đã cho trong biểu thức chính quy.

<!-- regexp -->

#### [runtime](/pkg/runtime/)

<!-- CL 216557 -->
Một số hàm, bao gồm [`ReadMemStats`](/pkg/runtime/#ReadMemStats) và [`GoroutineProfile`](/pkg/runtime/#GoroutineProfile), không còn chặn nếu đang thu gom rác.

#### [runtime/pprof](/pkg/runtime/pprof/)

<!-- CL 189318 -->
Profile goroutine giờ bao gồm các nhãn profile liên quan đến mỗi goroutine tại thời điểm profiling. Tính năng này chưa được triển khai cho profile được báo cáo với `debug=2`.

#### [strconv](/pkg/strconv/)

<!-- CL 216617 -->
[`FormatComplex`](/pkg/strconv/#FormatComplex) và [`ParseComplex`](/pkg/strconv/#ParseComplex) được thêm vào để làm việc với số phức.

[`FormatComplex`](/pkg/strconv/#FormatComplex) chuyển đổi số phức thành chuỗi dạng (a+bi), trong đó a và b là phần thực và phần ảo.

[`ParseComplex`](/pkg/strconv/#ParseComplex) chuyển đổi chuỗi thành số phức có độ chính xác được chỉ định. `ParseComplex` chấp nhận số phức ở định dạng `N+Ni`.

<!-- strconv -->

#### [sync](/pkg/sync/)

<!-- CL 205899, golang.org/issue/33762 -->
Phương thức mới [`Map.LoadAndDelete`](/pkg/sync/#Map.LoadAndDelete) xóa key một cách atomic và trả về giá trị trước đó nếu có.

<!-- CL 205899 -->
Phương thức [`Map.Delete`](/pkg/sync/#Map.Delete) hiệu quả hơn.

<!-- sync -->

#### [syscall](/pkg/syscall/)

<!-- CL 231638 -->
Trên hệ thống Unix, các hàm dùng [`SysProcAttr`](/pkg/syscall/#SysProcAttr) giờ sẽ từ chối các nỗ lực đặt cả hai trường `Setctty` và `Foreground`, vì cả hai đều dùng trường `Ctty` nhưng theo cách không tương thích. Chúng tôi kỳ vọng rất ít chương trình hiện tại đặt cả hai trường.

Đặt trường `Setctty` giờ yêu cầu trường `Ctty` được đặt thành số file descriptor trong tiến trình con, như được xác định bởi trường `ProcAttr.Files`. Dùng descriptor con luôn hoạt động, nhưng có một số trường hợp nhất định dùng file descriptor cha cũng có thể hoạt động. Một số chương trình đặt `Setctty` sẽ cần thay đổi giá trị của `Ctty` để dùng số descriptor con.

<!-- CL 220578 -->
[Giờ có thể](/pkg/syscall/#Proc.Call) gọi các lời gọi hệ thống trả về giá trị dấu phẩy động trên `windows/amd64`.

#### [testing](/pkg/testing/)

<!-- golang.org/issue/28135 -->
Kiểu `testing.T` giờ có phương thức [`Deadline`](/pkg/testing/#T.Deadline) báo cáo thời gian tệp nhị phân test sẽ vượt quá timeout.

<!-- golang.org/issue/34129 -->
Hàm `TestMain` không còn bắt buộc gọi `os.Exit`. Nếu hàm `TestMain` trả về, tệp nhị phân test sẽ gọi `os.Exit` với giá trị được trả về bởi `m.Run`.

<!-- CL 226877, golang.org/issue/35998 -->
Các phương thức mới [`T.TempDir`](/pkg/testing/#T.TempDir) và [`B.TempDir`](/pkg/testing/#B.TempDir) trả về các thư mục tạm thời được tự động dọn dẹp vào cuối test.

<!-- CL 229085 -->
`go` `test` `-v` giờ nhóm đầu ra theo tên test, thay vì in tên test trên mỗi dòng.

<!-- testing -->

#### [text/template](/pkg/text/template/)

<!-- CL 226097 -->
[`JSEscape`](/pkg/text/template/#JSEscape) giờ sử dụng nhất quán Unicode escapes (`\u00XX`), tương thích với JSON.

<!-- text/template -->

#### [time](/pkg/time/)

<!-- CL 220424, CL 217362, golang.org/issue/33184 -->
Phương thức mới [`Ticker.Reset`](/pkg/time/#Ticker.Reset) hỗ trợ thay đổi duration của ticker.

<!-- CL 227878 -->
Khi trả về lỗi, [`ParseDuration`](/pkg/time/#ParseDuration) giờ trích dẫn giá trị gốc.

<!-- time -->
