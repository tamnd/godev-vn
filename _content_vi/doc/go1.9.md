---
title: Ghi chú phát hành Go 1.9
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

## Giới thiệu về Go 1.9 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.9, ra đời sáu tháng sau [Go 1.8](go1.8) và là bản phát hành thứ mười trong [dòng Go 1.x](/doc/devel/release.html). Có hai [thay đổi về ngôn ngữ](#language): bổ sung hỗ trợ cho bí danh kiểu và xác định khi nào các phép toán dấu phẩy động có thể được gộp lại. Phần lớn các thay đổi nằm ở việc triển khai trình biên dịch, runtime và thư viện. Như thường lệ, bản phát hành duy trì [cam kết tương thích Go 1](/doc/go1compat.html). Chúng tôi mong rằng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

Bản phát hành bổ sung [hỗ trợ thời gian đơn điệu trong suốt](#monotonic-time), [song song hóa việc biên dịch các hàm](#parallel-compile) trong một gói, hỗ trợ tốt hơn cho [các hàm trợ giúp kiểm thử](#test-helper), bao gồm một [gói thao tác bit mới](#math-bits), và thêm [kiểu map đồng thời](#sync-map) mới.

## Thay đổi về ngôn ngữ {#language}

Có hai thay đổi về ngôn ngữ.

Go giờ hỗ trợ bí danh kiểu để hỗ trợ việc sửa mã dần dần khi chuyển một kiểu giữa các gói. [Tài liệu thiết kế bí danh kiểu](/design/18130-type-alias) và [một bài viết về tái cấu trúc](/talks/2016/refactor.article) trình bày vấn đề này chi tiết. Tóm lại, khai báo bí danh kiểu có dạng:

	type T1 = T2

Khai báo này giới thiệu tên bí danh `T1` — một cách viết thay thế — cho kiểu được biểu thị bởi `T2`; nghĩa là cả `T1` và `T2` đều biểu thị cùng một kiểu.

<!-- CL 40391 -->
Một thay đổi ngôn ngữ nhỏ hơn là [đặc tả ngôn ngữ giờ chỉ rõ](/ref/spec#Floating_point_operators) khi nào các phép toán dấu phẩy động được phép gộp lại, chẳng hạn bằng cách sử dụng lệnh "fused multiply and add" (FMA) của kiến trúc để tính `x*y` `+` `z` mà không làm tròn kết quả trung gian `x*y`. Để ép làm tròn trung gian, viết `float64(x*y)` `+` `z`.

## Các nền tảng {#ports}

Không có hệ điều hành hay kiến trúc bộ xử lý mới được hỗ trợ trong bản phát hành này.

### ppc64x yêu cầu POWER8 {#power8}

<!-- CL 36725, CL 36832 -->
Cả `GOARCH=ppc64` và `GOARCH=ppc64le` giờ đều yêu cầu hỗ trợ ít nhất POWER8. Trong các bản phát hành trước, chỉ `GOARCH=ppc64le` yêu cầu POWER8, còn kiến trúc `ppc64` big-endian hỗ trợ phần cứng cũ hơn.

### FreeBSD {#freebsd}

Go 1.9 là bản phát hành cuối cùng chạy trên FreeBSD 9.3, vốn đã [không được FreeBSD hỗ trợ](https://www.freebsd.org/security/unsupported.html). Go 1.10 sẽ yêu cầu FreeBSD 10.3 trở lên.

### OpenBSD 6.0 {#openbsd}

<!-- CL 40331 -->
Go 1.9 giờ bật tạo PT\_TLS cho các tệp nhị phân cgo và do đó yêu cầu OpenBSD 6.0 trở lên. Go 1.9 không còn hỗ trợ OpenBSD 5.9.

### Các vấn đề đã biết {#known_issues}

Có một số sự không ổn định trên FreeBSD đã được biết nhưng chưa được hiểu rõ. Điều này có thể dẫn đến sự cố chương trình trong các trường hợp hiếm gặp. Xem [issue 15658](/issue/15658). Mọi sự trợ giúp để giải quyết vấn đề dành riêng cho FreeBSD này đều được trân trọng.

Go đã ngừng chạy các builder NetBSD trong chu kỳ phát triển Go 1.9 do các crash kernel của NetBSD, bao gồm cả NetBSD 7.1. Khi Go 1.9 được phát hành, NetBSD 7.1.1 cũng được phát hành với một bản sửa lỗi. Tuy nhiên, tại thời điểm này chúng tôi không có builder NetBSD nào vượt qua bộ kiểm thử. Mọi sự trợ giúp điều tra các [vấn đề NetBSD khác nhau](https://github.com/golang/go/labels/OS-NetBSD) đều được trân trọng.

## Công cụ {#tools}

### Biên dịch song song {#parallel-compile}

Trình biên dịch Go giờ hỗ trợ biên dịch các hàm của một gói song song, tận dụng nhiều nhân xử lý. Điều này bổ sung thêm vào hỗ trợ biên dịch song song các gói riêng biệt đã có sẵn của lệnh `go`. Biên dịch song song được bật theo mặc định, nhưng có thể tắt bằng cách đặt biến môi trường `GO19CONCURRENTCOMPILATION` thành `0`.

### Khớp vendor với ./... {#vendor-dotdotdot}

<!-- CL 38745 -->
Theo yêu cầu phổ biến, `./...` giờ không còn khớp với các gói trong thư mục `vendor` trong các công cụ chấp nhận tên gói, chẳng hạn như `go` `test`. Để khớp với thư mục vendor, viết `./vendor/...`.

### Di chuyển GOROOT {#goroot}

<!-- CL 42533 -->
[Công cụ go](/cmd/go/) giờ sẽ sử dụng đường dẫn nơi nó được gọi để cố xác định vị trí gốc của cây cài đặt Go. Điều này có nghĩa là nếu toàn bộ cài đặt Go được chuyển đến vị trí mới, công cụ go vẫn tiếp tục hoạt động bình thường. Điều này có thể bị ghi đè bằng cách đặt `GOROOT` trong môi trường, điều chỉ nên làm trong các trường hợp đặc biệt. Lưu ý rằng điều này không ảnh hưởng đến kết quả của hàm [runtime.GOROOT](/pkg/runtime/#GOROOT), hàm này sẽ tiếp tục báo cáo vị trí cài đặt ban đầu; điều này có thể được sửa trong các bản phát hành sau.

### Trình biên dịch {#compiler}

<!-- CL 37441 -->
Phép chia số phức giờ tương thích với C99. Điều này luôn đúng trong gccgo và giờ đã được sửa trong trình biên dịch gc.

<!-- CL 36983 -->
Trình liên kết giờ sẽ tạo thông tin DWARF cho các tệp thực thi cgo trên Windows.

<!-- CL 44210, CL 40095 -->
Trình biên dịch giờ bao gồm phạm vi từ vựng trong DWARF được tạo ra nếu cờ `-N -l` được cung cấp, cho phép các debugger ẩn các biến không trong phạm vi. Phần `.debug_info` giờ là DWARF phiên bản 4.

<!-- CL 43855 -->
Các giá trị của `GOARM` và `GO386` giờ ảnh hưởng đến build ID của gói đã biên dịch, như được sử dụng bởi bộ nhớ đệm dependency của công cụ `go`.

### Trình hợp ngữ {#asm}

<!-- CL 42028 -->
Lệnh ARM bốn toán hạng `MULA` giờ được hợp ngữ đúng, với thanh ghi cộng là đối số thứ ba và thanh ghi kết quả là đối số thứ tư và cuối cùng. Trong các bản phát hành trước, hai ý nghĩa bị đổi ngược. Dạng ba toán hạng, trong đó đối số thứ tư ngầm định giống như đối số thứ ba, không bị ảnh hưởng. Mã sử dụng lệnh `MULA` bốn toán hạng sẽ cần được cập nhật, nhưng chúng tôi tin rằng dạng này rất hiếm khi được sử dụng. `MULAWT` và `MULAWB` đã sử dụng thứ tự đúng trong mọi dạng và không thay đổi.

<!-- CL 42990 -->
Trình hợp ngữ giờ hỗ trợ `ADDSUBPS/PD`, hoàn thiện hai lệnh SSE3 x86 còn thiếu.

### Doc {#go-doc}

<!-- CL 36031 -->
Danh sách đối số dài giờ được cắt bớt. Điều này cải thiện khả năng đọc của `go` `doc` trên một số mã được tạo tự động.

<!-- CL 38438 -->
Xem tài liệu về các trường struct giờ được hỗ trợ. Ví dụ: `go` `doc` `http.Client.Jar`.

### Env {#go-env-json}

<!-- CL 38757 -->
Cờ `-json` mới của `go` `env` cho phép xuất JSON, thay vì định dạng đầu ra mặc định dành riêng cho hệ điều hành.

### Test {#go-test-list}

<!-- CL 41195 -->
Lệnh [`go` `test`](/cmd/go/#hdr-Description_of_testing_flags) chấp nhận cờ `-list` mới, nhận một biểu thức chính quy làm đối số và in ra stdout tên của bất kỳ test, benchmark hay example nào khớp với nó, mà không chạy chúng.

### Pprof {#go-tool-pprof}

<!-- CL 34192 -->
Các profile được tạo bởi gói `runtime/pprof` giờ bao gồm thông tin symbol, do đó chúng có thể được xem bằng `go` `tool` `pprof` mà không cần tệp nhị phân đã tạo ra profile.

<!-- CL 38343 -->
Lệnh `go` `tool` `pprof` giờ sử dụng thông tin proxy HTTP được xác định trong môi trường, sử dụng [`http.ProxyFromEnvironment`](/pkg/net/http/#ProxyFromEnvironment).

### Vet {#vet}

<!-- CL 40112 -->

[Lệnh `vet`](/cmd/vet/) đã được tích hợp tốt hơn vào [công cụ `go`](/cmd/go/), do đó `go` `vet` giờ hỗ trợ tất cả các cờ build tiêu chuẩn trong khi các cờ riêng của `vet` giờ cũng có thể truy cập được từ `go` `vet` cũng như từ `go` `tool` `vet`.

### Gccgo {#gccgo}

Do sự đồng bộ lịch phát hành nửa năm một lần của Go với lịch phát hành hàng năm của GCC, GCC phiên bản 7 chứa phiên bản Go 1.8.3 của gccgo. Chúng tôi dự kiến rằng bản phát hành tiếp theo, GCC 8, sẽ chứa phiên bản Go 1.10 của gccgo.

## Runtime {#runtime}

### Call stack với các frame được inline {#callersframes}

Người dùng của [`runtime.Callers`](/pkg/runtime#Callers) nên tránh kiểm tra trực tiếp slice PC kết quả và thay vào đó sử dụng [`runtime.CallersFrames`](/pkg/runtime#CallersFrames) để có được cái nhìn đầy đủ về call stack, hoặc [`runtime.Caller`](/pkg/runtime#Caller) để lấy thông tin về một caller duy nhất. Điều này là vì một phần tử riêng lẻ của slice PC không thể tính toán cho các frame được inline hay các sắc thái khác của call stack.

Cụ thể, mã lặp trực tiếp qua slice PC và sử dụng các hàm như [`runtime.FuncForPC`](/pkg/runtime#FuncForPC) để giải quyết từng PC riêng lẻ sẽ bỏ lỡ các frame được inline. Để có cái nhìn đầy đủ về stack, mã như vậy nên sử dụng `CallersFrames`. Tương tự, mã không nên giả định rằng độ dài được trả về bởi `Callers` là bất kỳ chỉ số nào về độ sâu call. Thay vào đó, nên đếm số frame được trả về bởi `CallersFrames`.

Mã truy vấn một caller duy nhất ở độ sâu cụ thể nên sử dụng `Caller` thay vì truyền một slice có độ dài 1 cho `Callers`.

[`runtime.CallersFrames`](/pkg/runtime#CallersFrames) đã có sẵn từ Go 1.7, do đó mã có thể được cập nhật trước khi nâng cấp lên Go 1.9.

## Hiệu suất {#performance}

Như thường lệ, các thay đổi rất đa dạng đến mức khó có thể đưa ra các nhận xét chính xác về hiệu suất. Hầu hết các chương trình nên chạy nhanh hơn một chút, nhờ các cải tiến trong bộ gom rác, mã được tạo tốt hơn và các tối ưu hóa trong thư viện cốt lõi.

### Bộ gom rác {#gc}

<!-- CL 37520 -->
Các hàm thư viện từng kích hoạt thu gom rác stop-the-world giờ kích hoạt thu gom rác đồng thời. Cụ thể, [`runtime.GC`](/pkg/runtime/#GC), [`debug.SetGCPercent`](/pkg/runtime/debug/#SetGCPercent) và [`debug.FreeOSMemory`](/pkg/runtime/debug/#FreeOSMemory) giờ kích hoạt thu gom rác đồng thời, chỉ chặn goroutine đang gọi cho đến khi thu gom rác hoàn tất.

<!-- CL 34103, CL 39835 -->
Hàm [`debug.SetGCPercent`](/pkg/runtime/debug/#SetGCPercent) chỉ kích hoạt thu gom rác nếu cần ngay lập tức do giá trị GOGC mới. Điều này cho phép điều chỉnh GOGC theo thời gian thực.

<!-- CL 38732 -->
Hiệu suất phân bổ đối tượng lớn được cải thiện đáng kể trong các ứng dụng sử dụng heap lớn (>50GB) chứa nhiều đối tượng lớn.

<!-- CL 34937 -->
Hàm [`runtime.ReadMemStats`](/pkg/runtime/#ReadMemStats) giờ mất ít hơn 100µs ngay cả với các heap rất lớn.

## Thư viện chuẩn {#library}

### Hỗ trợ thời gian đơn điệu trong suốt {#monotonic-time}

<!-- CL 36255 -->
Gói [`time`](/pkg/time/) giờ theo dõi thời gian đơn điệu một cách trong suốt trong mỗi giá trị [`Time`](/pkg/time/#Time), làm cho việc tính toán thời gian giữa hai giá trị `Time` trở thành một thao tác an toàn khi có điều chỉnh đồng hồ thực. Xem [tài liệu gói](/pkg/time/#hdr-Monotonic_Clocks) và [tài liệu thiết kế](/design/12914-monotonic) để biết chi tiết.

### Gói thao tác bit mới {#math-bits}

<!-- CL 36315 -->
Go 1.9 bao gồm một gói mới, [`math/bits`](/pkg/math/bits/), với các triển khai được tối ưu hóa để thao tác bit. Trên hầu hết các kiến trúc, các hàm trong gói này được trình biên dịch nhận ra thêm và xử lý như các intrinsic để có hiệu suất tốt hơn.

### Các hàm trợ giúp kiểm thử {#test-helper}

<!-- CL 38796 -->
Các phương thức [`(*T).Helper`](/pkg/testing/#T.Helper) và [`(*B).Helper`](/pkg/testing/#B.Helper) mới đánh dấu hàm đang gọi là hàm trợ giúp kiểm thử. Khi in thông tin tệp và dòng, hàm đó sẽ bị bỏ qua. Điều này cho phép viết các hàm trợ giúp kiểm thử trong khi vẫn có số dòng hữu ích cho người dùng.

### Map đồng thời {#sync-map}

<!-- CL 36617 -->
Kiểu [`Map`](/pkg/sync/#Map) mới trong gói [`sync`](/pkg/sync/) là một map đồng thời với thời gian load, store và delete khấu hao không đổi. Nhiều goroutine có thể gọi các phương thức của `Map` đồng thời một cách an toàn.

### Nhãn profiler {#pprof-labels}

<!-- CL 34198 -->
[Gói `runtime/pprof`](/pkg/runtime/pprof) giờ hỗ trợ thêm nhãn vào các bản ghi profiler `pprof`. Nhãn tạo thành một map key-value được sử dụng để phân biệt các lời gọi cùng một hàm trong các ngữ cảnh khác nhau khi xem các profile bằng [lệnh `pprof`](/cmd/pprof/). Hàm [`Do`](/pkg/runtime/pprof/#Do) mới của gói `pprof` chạy mã liên kết với một số nhãn được cung cấp. Các hàm mới khác trong gói giúp làm việc với nhãn.

<!-- runtime/pprof -->

### Các thay đổi nhỏ với thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ khác nhau đối với thư viện, được thực hiện với [cam kết tương thích Go 1](/doc/go1compat) trong tâm trí.

#### [archive/zip](/pkg/archive/zip/)

<!-- CL 39570 -->
[`Writer`](/pkg/archive/zip/#Writer) ZIP giờ đặt bit UTF-8 trong [`FileHeader.Flags`](/pkg/archive/zip/#FileHeader.Flags) khi thích hợp.

<!-- archive/zip -->

#### [crypto/rand](/pkg/crypto/rand/)

<!-- CL 43852 -->
Trên Linux, Go giờ gọi system call `getrandom` mà không có cờ `GRND_NONBLOCK`; nó giờ sẽ chặn cho đến khi kernel có đủ randomness. Trên các kernel không có system call `getrandom`, Go tiếp tục đọc từ `/dev/urandom`.

<!-- crypto/rand -->

#### [crypto/x509](/pkg/crypto/x509/)

<!-- CL 36093 -->
Trên các hệ thống Unix, các biến môi trường `SSL_CERT_FILE` và `SSL_CERT_DIR` giờ có thể được sử dụng để ghi đè vị trí mặc định của hệ thống cho tệp chứng chỉ SSL và thư mục tệp chứng chỉ SSL, tương ứng.

Tệp FreeBSD `/usr/local/etc/ssl/cert.pem` giờ được bao gồm trong đường dẫn tìm kiếm chứng chỉ.

<!-- CL 36900 -->
Gói giờ hỗ trợ các domain bị loại trừ trong các ràng buộc tên. Ngoài việc thực thi các ràng buộc đó, [`CreateCertificate`](/pkg/crypto/x509/#CreateCertificate) sẽ tạo chứng chỉ với các ràng buộc tên bị loại trừ nếu chứng chỉ mẫu được cung cấp có trường [`ExcludedDNSDomains`](/pkg/crypto/x509/#Certificate.ExcludedDNSDomains) mới được điền.

<!-- CL 36696 -->
Nếu bất kỳ phần mở rộng SAN nào, kể cả không có tên DNS, có mặt trong chứng chỉ, thì Common Name từ [`Subject`](/pkg/crypto/x509/#Certificate.Subject) bị bỏ qua. Trong các bản phát hành trước, mã chỉ kiểm tra xem có SAN tên DNS trong chứng chỉ hay không.

<!-- crypto/x509 -->

#### [database/sql](/pkg/database/sql/)

<!-- CL 35476 -->
Gói giờ sẽ sử dụng [`Stmt`](/pkg/database/sql/#Stmt) đã lưu trong bộ nhớ đệm nếu có trong [`Tx.Stmt`](/pkg/database/sql/#Tx.Stmt). Điều này ngăn các câu lệnh bị chuẩn bị lại mỗi khi [`Tx.Stmt`](/pkg/database/sql/#Tx.Stmt) được gọi.

<!-- CL 38533 -->
Gói giờ cho phép driver triển khai trình kiểm tra đối số của riêng họ bằng cách triển khai [`driver.NamedValueChecker`](/pkg/database/sql/driver/#NamedValueChecker). Điều này cũng cho phép driver hỗ trợ các kiểu tham số `OUTPUT` và `INOUT`. [`Out`](/pkg/database/sql/#Out) nên được sử dụng để trả về các tham số đầu ra khi driver hỗ trợ.

<!-- CL 39031 -->
[`Rows.Scan`](/pkg/database/sql/#Rows.Scan) giờ có thể quét các kiểu string do người dùng định nghĩa. Trước đây gói hỗ trợ quét vào các kiểu số như `type` `Int` `int64`. Giờ nó cũng hỗ trợ quét vào các kiểu string như `type` `String` `string`.

<!-- CL 40694 -->
Phương thức [`DB.Conn`](/pkg/database/sql/#DB.Conn) mới trả về kiểu [`Conn`](/pkg/database/sql/#Conn) mới đại diện cho một kết nối độc quyền đến cơ sở dữ liệu từ connection pool. Tất cả các truy vấn chạy trên [`Conn`](/pkg/database/sql/#Conn) sẽ sử dụng cùng một kết nối bên dưới cho đến khi [`Conn.Close`](/pkg/database/sql/#Conn.Close) được gọi để trả kết nối về connection pool.

<!-- database/sql -->

#### [encoding/asn1](/pkg/encoding/asn1/)

<!-- CL 38660 -->
[`NullBytes`](/pkg/encoding/asn1/#NullBytes) và [`NullRawValue`](/pkg/encoding/asn1/#NullRawValue) mới đại diện cho kiểu ASN.1 NULL.

<!-- encoding/asn1 -->

#### [encoding/base32](/pkg/encoding/base32/)

<!-- CL 38634 -->
Phương thức [Encoding.WithPadding](/pkg/encoding/base32/#Encoding.WithPadding) mới bổ sung hỗ trợ cho các ký tự đệm tùy chỉnh và tắt đệm.

<!-- encoding/base32 -->

#### [encoding/csv](/pkg/encoding/csv/)

<!-- CL 41730 -->
Trường [`Reader.ReuseRecord`](/pkg/encoding/csv/#Reader.ReuseRecord) mới kiểm soát xem các lời gọi đến [`Read`](/pkg/encoding/csv/#Reader.Read) có thể trả về một slice chia sẻ backing array của slice được trả về từ lần gọi trước để cải thiện hiệu suất hay không.

<!-- encoding/csv -->

#### [fmt](/pkg/fmt/)

<!-- CL 37051 -->
Cờ sharp ('`#`') giờ được hỗ trợ khi in số dấu phẩy động và số phức. Nó sẽ luôn in dấu thập phân cho `%e`, `%E`, `%f`, `%F`, `%g` và `%G`; nó sẽ không xóa các số 0 ở cuối cho `%g` và `%G`.

<!-- fmt -->

#### [hash/fnv](/pkg/hash/fnv/)

<!-- CL 38356 -->
Gói giờ bao gồm hỗ trợ hash FNV-1 và FNV-1a 128-bit với [`New128`](/pkg/hash/fnv/#New128) và [`New128a`](/pkg/hash/fnv/#New128a), tương ứng.

<!-- hash/fnv -->

#### [html/template](/pkg/html/template/)

<!-- CL 37880, CL 40936 -->
Gói giờ báo lỗi nếu một escaper được định nghĩa trước (một trong "html", "urlquery" và "js") được tìm thấy trong pipeline và không khớp với quyết định của auto-escaper. Điều này tránh một số vấn đề bảo mật hoặc tính đúng đắn. Giờ việc sử dụng một trong những escaper này luôn là no-op hoặc lỗi. (Trường hợp no-op giúp việc chuyển đổi từ [text/template](/pkg/text/template/) dễ dàng hơn.)

<!-- html/template -->

#### [image](/pkg/image/)

<!-- CL 36734 -->
Phương thức [`Rectangle.Intersect`](/pkg/image/#Rectangle.Intersect) giờ trả về một `Rectangle` bằng không khi được gọi trên các hình chữ nhật liền kề nhưng không chồng lên nhau, như đã ghi trong tài liệu. Trong các bản phát hành trước, nó sẽ không chính xác trả về một `Rectangle` rỗng nhưng khác không.

<!-- image -->

#### [image/color](/pkg/image/color/)

<!-- CL 36732 -->
Công thức chuyển đổi YCbCr sang RGBA đã được điều chỉnh để đảm bảo rằng các điều chỉnh làm tròn bao phủ toàn bộ phạm vi RGBA [0, 0xffff].

<!-- image/color -->

#### [image/png](/pkg/image/png/)

<!-- CL 34150 -->
Trường [`Encoder.BufferPool`](/pkg/image/png/#Encoder.BufferPool) mới cho phép chỉ định một [`EncoderBufferPool`](/pkg/image/png/#EncoderBufferPool), sẽ được encoder sử dụng để lấy các buffer `EncoderBuffer` tạm thời khi mã hóa ảnh PNG. Việc sử dụng `BufferPool` giảm số lượng phân bổ bộ nhớ được thực hiện khi mã hóa nhiều ảnh.

<!-- CL 38271 -->
Gói giờ hỗ trợ giải mã ảnh grayscale 8-bit trong suốt ("Gray8").

<!-- image/png -->

#### [math/big](/pkg/math/big/)

<!-- CL 36487 -->
Các phương thức [`IsInt64`](/pkg/math/big/#Int.IsInt64) và [`IsUint64`](/pkg/math/big/#Int.IsUint64) mới báo cáo xem một `Int` có thể được biểu diễn như một giá trị `int64` hay `uint64` hay không.

<!-- math/big -->

#### [mime/multipart](/pkg/mime/multipart/)

<!-- CL 39223 -->
Trường [`FileHeader.Size`](/pkg/mime/multipart/#FileHeader.Size) mới mô tả kích thước của một tệp trong một tin nhắn multipart.

<!-- mime/multipart -->

#### [net](/pkg/net/)

<!-- CL 32572 -->
[`Resolver.StrictErrors`](/pkg/net/#Resolver.StrictErrors) mới cung cấp khả năng kiểm soát cách trình phân giải DNS tích hợp của Go xử lý các lỗi tạm thời trong các truy vấn bao gồm nhiều sub-query, chẳng hạn như tra cứu địa chỉ A+AAAA.

<!-- CL 37260 -->
[`Resolver.Dial`](/pkg/net/#Resolver.Dial) mới cho phép một `Resolver` sử dụng hàm dial tùy chỉnh.

<!-- CL 40510 -->
[`JoinHostPort`](/pkg/net/#JoinHostPort) giờ chỉ đặt địa chỉ trong ngoặc vuông nếu host chứa dấu hai chấm. Trong các bản phát hành trước, nó cũng bao địa chỉ trong ngoặc vuông nếu chứa dấu phần trăm ('`%`').

<!-- CL 37913 -->
Các phương thức mới [`TCPConn.SyscallConn`](/pkg/net/#TCPConn.SyscallConn), [`IPConn.SyscallConn`](/pkg/net/#IPConn.SyscallConn), [`UDPConn.SyscallConn`](/pkg/net/#UDPConn.SyscallConn) và [`UnixConn.SyscallConn`](/pkg/net/#UnixConn.SyscallConn) cung cấp quyền truy cập vào các file descriptor bên dưới của kết nối.

<!-- 45088 -->
Giờ có thể an toàn khi gọi [`Dial`](/pkg/net/#Dial) với địa chỉ thu được từ `(*TCPListener).String()` sau khi tạo listener bằng <code>[Listen](/pkg/net/#Listen)("tcp", ":0")</code>. Trước đây nó thất bại trên một số máy với stack IPv6 được cấu hình không đầy đủ.

<!-- net -->

#### [net/http](/pkg/net/http/)

<!-- CL 37328 -->
Phương thức [`Cookie.String`](/pkg/net/http/#Cookie.String), được sử dụng cho các header `Cookie` và `Set-Cookie`, giờ đặt các giá trị trong ngoặc kép nếu giá trị chứa khoảng trắng hoặc dấu phẩy.

Các thay đổi ở Server:

  - <!-- CL 38194 -->
    [`ServeMux`](/pkg/net/http/#ServeMux) giờ bỏ qua các port trong header host khi khớp handler. Host được khớp không thay đổi cho các yêu cầu `CONNECT`.
  - <!-- CL 44074 -->
    Phương thức [`Server.ServeTLS`](/pkg/net/http/#Server.ServeTLS) mới bọc [`Server.Serve`](/pkg/net/http/#Server.Serve) với hỗ trợ TLS được thêm vào.
  - <!-- CL 34727 -->
    [`Server.WriteTimeout`](/pkg/net/http/#Server.WriteTimeout) giờ áp dụng cho các kết nối HTTP/2 và được thực thi theo từng stream.
  - <!-- CL 43231 -->
    HTTP/2 giờ sử dụng bộ lập lịch ghi ưu tiên theo mặc định. Các frame được lập lịch theo các ưu tiên HTTP/2 như mô tả trong [RFC 7540 Section 5.3](https://tools.ietf.org/html/rfc7540#section-5.3).
  - <!-- CL 36483 -->
    Handler HTTP được trả về bởi [`StripPrefix`](/pkg/net/http/#StripPrefix) giờ gọi handler được cung cấp với một bản sao đã sửa đổi của `*http.Request` gốc. Bất kỳ mã nào lưu trữ trạng thái theo yêu cầu trong các map được khóa bởi `*http.Request` nên sử dụng [`Request.Context`](/pkg/net/http/#Request.Context), [`Request.WithContext`](/pkg/net/http/#Request.WithContext) và [`context.WithValue`](/pkg/context/#WithValue) thay thế.
  - <!-- CL 35490 -->
    [`LocalAddrContextKey`](/pkg/net/http/#LocalAddrContextKey) giờ chứa địa chỉ mạng thực tế của kết nối thay vì địa chỉ giao diện được sử dụng bởi listener.

Các thay đổi ở Client & Transport:

  - <!-- CL 35488 -->
    [`Transport`](/pkg/net/http/#Transport) giờ hỗ trợ thực hiện các yêu cầu qua proxy SOCKS5 khi URL được trả về bởi [`Transport.Proxy`](/pkg/net/http/#Transport.Proxy) có scheme `socks5`.

<!-- net/http -->

#### [net/http/fcgi](/pkg/net/http/fcgi/)

<!-- CL 40012 -->
Hàm [`ProcessEnv`](/pkg/net/http/fcgi/#ProcessEnv) mới trả về các biến môi trường FastCGI liên kết với một yêu cầu HTTP mà không có trường [`http.Request`](/pkg/net/http/#Request) phù hợp, chẳng hạn như `REMOTE_USER`.

<!-- net/http/fcgi -->

#### [net/http/httptest](/pkg/net/http/httptest/)

<!-- CL 34639 -->
Phương thức [`Server.Client`](/pkg/net/http/httptest/#Server.Client) mới trả về một HTTP client được cấu hình để thực hiện yêu cầu đến server kiểm thử.

Phương thức [`Server.Certificate`](/pkg/net/http/httptest/#Server.Certificate) mới trả về chứng chỉ TLS của server kiểm thử, nếu có.

<!-- net/http/httptest -->

#### [net/http/httputil](/pkg/net/http/httputil/)

<!-- CL 43712 -->
[`ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy) giờ proxy tất cả các trailer phản hồi HTTP/2, ngay cả những trailer không được khai báo trong header phản hồi ban đầu. Các trailer không được khai báo như vậy được sử dụng bởi giao thức gRPC.

<!-- net/http/httputil -->

#### [os](/pkg/os/)

<!-- CL 36800 -->
Gói `os` giờ sử dụng runtime poller nội bộ cho I/O tệp. Điều này giảm số lượng thread cần thiết cho các thao tác đọc/ghi trên pipe, và loại bỏ các race condition khi một goroutine đóng tệp trong khi goroutine khác đang sử dụng tệp đó cho I/O.

<!-- CL 37915 -->
Trên Windows, [`Args`](/pkg/os/#Args) giờ được điền mà không cần `shell32.dll`, cải thiện thời gian khởi động tiến trình thêm 1-7 ms.

<!-- os -->

#### [os/exec](/pkg/os/exec/)

<!-- CL 37586 -->
Gói `os/exec` giờ ngăn việc tạo các tiến trình con với bất kỳ biến môi trường trùng lặp nào. Nếu [`Cmd.Env`](/pkg/os/exec/#Cmd.Env) chứa các khóa môi trường trùng lặp, chỉ giá trị cuối cùng trong slice cho mỗi khóa trùng lặp được sử dụng.

<!-- os/exec -->

#### [os/user](/pkg/os/user/)

<!-- CL 37664 -->
[`Lookup`](/pkg/os/user/#Lookup) và [`LookupId`](/pkg/os/user/#LookupId) giờ hoạt động trên các hệ thống Unix khi `CGO_ENABLED=0` bằng cách đọc tệp `/etc/passwd`.

<!-- CL 33713 -->
[`LookupGroup`](/pkg/os/user/#LookupGroup) và [`LookupGroupId`](/pkg/os/user/#LookupGroupId) giờ hoạt động trên các hệ thống Unix khi `CGO_ENABLED=0` bằng cách đọc tệp `/etc/group`.

<!-- os/user -->

#### [reflect](/pkg/reflect/)

<!-- CL 38335 -->
Hàm [`MakeMapWithSize`](/pkg/reflect/#MakeMapWithSize) mới tạo một map với gợi ý dung lượng.

<!-- reflect -->

#### [runtime](/pkg/runtime/)

<!-- CL 37233, CL 37726 -->
Các traceback được tạo bởi runtime và được ghi trong các profile giờ chính xác khi có inlining. Để lấy traceback theo chương trình, các ứng dụng nên sử dụng [`runtime.CallersFrames`](/pkg/runtime/#CallersFrames) thay vì lặp trực tiếp qua kết quả của [`runtime.Callers`](/pkg/runtime/#Callers).

<!-- CL 38403 -->
Trên Windows, Go không còn ép buộc đồng hồ hệ thống chạy ở độ phân giải cao khi chương trình nhàn rỗi. Điều này sẽ giảm tác động của các chương trình Go lên tuổi thọ pin.

<!-- CL 29341 -->
Trên FreeBSD, `GOMAXPROCS` và [`runtime.NumCPU`](/pkg/runtime/#NumCPU) giờ dựa trên CPU mask của tiến trình, thay vì tổng số CPU.

<!-- CL 43641 -->
Runtime có hỗ trợ sơ bộ cho Android O.

<!-- runtime -->

#### [runtime/debug](/pkg/runtime/debug/)

<!-- CL 34013 -->
Gọi [`SetGCPercent`](/pkg/runtime/debug/#SetGCPercent) với giá trị âm không còn chạy thu gom rác ngay lập tức nữa.

<!-- runtime/debug -->

#### [runtime/trace](/pkg/runtime/trace/)

<!-- CL 36015 -->
Trace thực thi giờ hiển thị các sự kiện mark assist, chỉ ra khi nào một goroutine ứng dụng bị buộc phải hỗ trợ thu gom rác vì nó đang phân bổ quá nhanh.

<!-- CL 40810 -->
Các sự kiện "Sweep" giờ bao gồm toàn bộ quá trình tìm kiếm không gian trống cho một phân bổ, thay vì ghi lại từng span riêng lẻ được sweep. Điều này giảm độ trễ phân bổ khi trace các chương trình phân bổ nhiều. Sự kiện sweep hiển thị bao nhiêu byte đã được sweep và bao nhiêu đã được thu hồi.

<!-- runtime/trace -->

#### [sync](/pkg/sync/)

<!-- CL 34310 -->
[`Mutex`](/pkg/sync/#Mutex) giờ công bằng hơn.

<!-- sync -->

#### [syscall](/pkg/syscall/)

<!-- CL 36697 -->
Trường [`Credential.NoSetGroups`](/pkg/syscall/#Credential.NoSetGroups) mới kiểm soát xem các hệ thống Unix có thực hiện system call `setgroups` để thiết lập các nhóm bổ sung khi khởi động tiến trình mới hay không.

<!-- CL 43512 -->
Trường [`SysProcAttr.AmbientCaps`](/pkg/syscall/#SysProcAttr.AmbientCaps) mới cho phép thiết lập ambient capabilities trên Linux 4.3 trở lên khi tạo tiến trình mới.

<!-- CL 37439 -->
Trên Linux x86 64-bit, độ trễ tạo tiến trình đã được tối ưu hóa bằng cách sử dụng `CLONE_VFORK` và `CLONE_VM`.

<!-- CL 37913 -->
Interface [`Conn`](/pkg/syscall/#Conn) mới mô tả một số kiểu trong gói [`net`](/pkg/net/) có thể cung cấp quyền truy cập vào file descriptor bên dưới của chúng bằng cách sử dụng interface [`RawConn`](/pkg/syscall/#RawConn) mới.

<!-- syscall -->

#### [testing/quick](/pkg/testing/quick/)

<!-- CL 39152 -->
Gói giờ chọn các giá trị trong phạm vi đầy đủ khi tạo các số ngẫu nhiên `int64` và `uint64`; trong các bản phát hành trước, các giá trị được tạo luôn bị giới hạn trong phạm vi [-2<sup>62</sup>, 2<sup>62</sup>).

Trong các bản phát hành trước, sử dụng giá trị [`Config.Rand`](/pkg/testing/quick/#Config.Rand) nil khiến một bộ tạo số ngẫu nhiên xác định cố định được sử dụng. Giờ nó sử dụng một bộ tạo số ngẫu nhiên được khởi tạo bằng thời gian hiện tại. Để có hành vi cũ, đặt `Config.Rand` thành `rand.New(rand.NewSource(0))`.

<!-- testing/quick -->

#### [text/template](/pkg/text/template/)

<!-- CL 38420 -->
Việc xử lý các block rỗng, vốn bị hỏng bởi một thay đổi Go 1.8 làm cho kết quả phụ thuộc vào thứ tự của các template, đã được sửa, khôi phục lại hành vi cũ của Go 1.7.

<!-- text/template -->

#### [time](/pkg/time/)

<!-- CL 36615 -->
Các phương thức [`Duration.Round`](/pkg/time/#Duration.Round) và [`Duration.Truncate`](/pkg/time/#Duration.Truncate) mới xử lý việc làm tròn và cắt bớt duration thành bội số của một duration đã cho.

<!-- CL 35710 -->
Việc lấy thời gian và ngủ giờ hoạt động đúng dưới Wine.

Nếu một giá trị `Time` có số đọc đồng hồ đơn điệu, biểu diễn chuỗi của nó (như được trả về bởi `String`) giờ bao gồm một trường cuối cùng `"m=±value"`, trong đó `value` là số đọc đồng hồ đơn điệu được định dạng là số giây thập phân.

<!-- CL 44832 -->
Cơ sở dữ liệu múi giờ `tzdata` được bao gồm đã được cập nhật lên phiên bản 2017b. Như thường lệ, nó chỉ được sử dụng nếu hệ thống chưa có sẵn cơ sở dữ liệu.

<!-- time -->
