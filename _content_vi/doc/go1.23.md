---
title: Ghi chú phát hành Go 1.23
---

<!--
NOTE: In this document and others in this directory, the convention is to
set fixed-width phrases with non-fixed-width spaces, as in
`hello` `world`.
-->

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu Go 1.23 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.23, ra mắt sáu tháng sau [Go 1.22](/doc/go1.22).
Phần lớn các thay đổi nằm ở phần triển khai toolchain, runtime và thư viện.
Như thường lệ, bản phát hành duy trì [cam kết tương thích](/doc/go1compat) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

## Thay đổi ngôn ngữ {#language}

<!-- go.dev/issue/61405, CL 557835, CL 584596 -->
Mệnh đề "range" trong vòng lặp "for-range" giờ chấp nhận các hàm iterator thuộc các kiểu sau

    func(func() bool)
    func(func(K) bool)
    func(func(K, V) bool)

làm biểu thức range.
Các lệnh gọi hàm đối số iterator tạo ra các giá trị lặp cho vòng lặp "for-range".
Để biết chi tiết, xem tài liệu gói [`iter`](/pkg/iter),
[đặc tả ngôn ngữ](/ref/spec#For_range), và bài viết blog [Range over Function
Types](/blog/range-functions).
Để hiểu động lực đằng sau thay đổi này, xem [thảo luận "range-over-func" năm 2022](/issue/56413).

<!-- go.dev/issue/46477, CL 566856, CL 586955, CL 586956 -->
Go 1.23 bao gồm hỗ trợ xem trước cho [generic type alias](/issue/46477).
Xây dựng toolchain với `GOEXPERIMENT=aliastypeparams` kích hoạt tính năng này trong một gói.
(Sử dụng generic alias type qua ranh giới gói chưa được hỗ trợ.)

## Công cụ {#tools}

### Telemetry

<!-- go.dev/issue/58894, go.dev/issue/67111 -->
Từ Go 1.23, toolchain Go có thể thu thập thống kê sử dụng và lỗi
giúp nhóm Go hiểu cách toolchain Go được sử dụng và hoạt động như thế nào.
Chúng tôi gọi những thống kê này là
[Go telemetry](/doc/telemetry).

Go telemetry là _hệ thống opt-in_, được kiểm soát bởi
[lệnh `go` `telemetry`](/cmd/go/#hdr-Manage_telemetry_data_and_settings).
Theo mặc định, các chương trình toolchain
thu thập thống kê trong các tệp đếm có thể kiểm tra cục bộ
nhưng không được sử dụng theo cách khác (`go` `telemetry` `local`).

Để giúp chúng tôi giữ cho Go hoạt động tốt và hiểu cách sử dụng Go,
hãy cân nhắc tham gia Go telemetry bằng cách chạy
`go` `telemetry` `on`.
Trong chế độ đó,
các báo cáo đếm ẩn danh được tải lên
[telemetry.go.dev](https://telemetry.go.dev) hàng tuần,
nơi chúng được tổng hợp thành các biểu đồ và cũng được
cung cấp để tải xuống bởi bất kỳ người đóng góp hoặc người dùng Go nào
muốn phân tích dữ liệu.
Xem "[Go Telemetry](/doc/telemetry)" để biết thêm chi tiết
về hệ thống Go Telemetry.

### Lệnh Go {#go-command}

Đặt biến môi trường `GOROOT_FINAL` không còn có tác dụng gì
([#62047](/issue/62047)).
Các bản phân phối cài đặt lệnh `go` ở vị trí khác với
`$GOROOT/bin/go` nên cài đặt symlink thay vì di chuyển
hoặc sao chép tệp nhị phân `go`.

<!-- go.dev/issue/34208, CL 563137, CL 586095 -->
Cờ mới `go` `env` `-changed` khiến lệnh chỉ in
những cài đặt có giá trị hiệu lực khác với giá trị mặc định
sẽ có trong môi trường trống không có lần sử dụng cờ `-w` trước đó.

<!-- go.dev/issue/27005, CL 585401 -->
Cờ mới `go` `mod` `tidy` `-diff` khiến lệnh không sửa đổi
các tệp mà thay vào đó in các thay đổi cần thiết dưới dạng unified diff.
Nó thoát với mã khác không nếu cần cập nhật.

<!-- go.dev/issue/52792, CL 562775 -->
Lệnh `go` `list` `-m` `-json` giờ bao gồm các trường `Sum` và `GoModSum` mới.
Điều này tương tự với hành vi hiện có của lệnh `go` `mod` `download` `-json`.

<!-- go.dev/issue/65573 ("cmd/go: separate default GODEBUGs from go language version") -->
Directive mới `godebug` trong `go.mod` và `go.work` khai báo một
[cài đặt GODEBUG](/doc/godebug) áp dụng cho work module hoặc workspace đang dùng.

### Vet {#vet}

<!-- go.dev/issue/46136 -->
Lệnh con `go vet` giờ bao gồm trình phân tích
[stdversion](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/stdversion),
đánh dấu các tham chiếu đến các symbol quá mới đối với phiên bản
Go có hiệu lực trong tệp tham chiếu. (Phiên bản hiệu lực được xác định
bởi directive `go` trong tệp `go.mod` bao quanh của tệp, và
bởi bất kỳ [ràng buộc `//go:build`](/cmd/go#hdr-Build_constraints) nào
trong tệp.)

Ví dụ, nó sẽ báo cáo chẩn đoán cho tham chiếu đến hàm
`reflect.TypeFor` (được giới thiệu trong go1.22) từ một tệp trong
module có tệp go.mod chỉ định `go 1.21`.

### Cgo {#cgo}

<!-- go.dev/issue/66456 -->
[`cmd/cgo`](/pkg/cmd/cgo) hỗ trợ cờ mới `-ldflags` để truyền cờ cho linker C.
Lệnh `go` sử dụng nó tự động, tránh lỗi "argument list too long"
với `CGO_LDFLAGS` rất lớn.

### Trace {#trace}

<!-- go.dev/issue/65316 -->
Công cụ `trace` giờ xử lý tốt hơn các trace bị hỏng một phần bằng cách cố gắng
khôi phục dữ liệu trace có thể. Chức năng này đặc biệt hữu ích khi
xem trace được thu thập trong quá trình chương trình bị crash, vì dữ liệu trace
dẫn đến crash giờ sẽ [có thể khôi phục được](/issue/65319) trong hầu hết
các trường hợp.

## Runtime {#runtime}

Traceback được in bởi runtime sau một panic không được xử lý hoặc lỗi nghiêm trọng khác
giờ thụt lề các dòng thứ hai và tiếp theo của thông báo lỗi
(ví dụ, đối số của panic) bằng một tab duy nhất, để nó
có thể được phân biệt rõ ràng với stack trace của
goroutine đầu tiên. Xem [#64590](/issue/64590) để thảo luận.

## Trình biên dịch {#compiler}

Chi phí thời gian build khi build với [tối ưu hóa dựa trên hồ sơ thực thi](/doc/pgo) đã giảm đáng kể.
Trước đây, các build lớn có thể thấy tăng thời gian build hơn 100% khi bật PGO.
Trong Go 1.23, chi phí chỉ còn ở mức phần trăm đơn.

<!-- https://go.dev/issue/62737 , https://golang.org/cl/576681,  https://golang.org/cl/577615 -->
Trình biên dịch trong Go 1.23 giờ có thể chồng chéo các slot stack frame của các biến cục bộ
được truy cập trong các vùng tách biệt của hàm, điều này giảm sử dụng stack
cho các ứng dụng Go.

<!-- https://go.dev/cl/577935 -->
Với 386 và amd64, trình biên dịch sẽ sử dụng thông tin từ PGO để căn chỉnh một số
block nóng trong vòng lặp. Điều này cải thiện hiệu suất thêm 1-1.5% với
chi phí tăng thêm 0.1% kích thước text và binary. Hiện tại điều này chỉ được triển khai
trên 386 và amd64 vì nó không cho thấy cải thiện trên các nền tảng khác.
Việc căn chỉnh block nóng có thể bị vô hiệu hóa với `-gcflags=[<packages>=]-d=alignhot=0`.

## Linker {#linker}

<!-- go.dev/issue/67401, CL 585556, CL 587220, and many more -->
Linker giờ không cho phép sử dụng directive `//go:linkname` để tham chiếu đến
các symbol nội bộ trong thư viện chuẩn (bao gồm runtime) không được
đánh dấu với `//go:linkname` trên định nghĩa của chúng.
Tương tự, linker không cho phép tham chiếu đến các symbol như vậy từ mã
assembly.
Để tương thích ngược, các cách dùng `//go:linkname` hiện có được tìm thấy trong
kho lưu trữ mã nguồn mở lớn vẫn được hỗ trợ.
Bất kỳ tham chiếu mới nào đến các symbol nội bộ thư viện chuẩn sẽ không được cho phép.

Cờ dòng lệnh linker `-checklinkname=0` có thể được dùng để vô hiệu hóa
kiểm tra này, cho mục đích debugging và thử nghiệm.

<!-- CL 473495 -->
Khi xây dựng tệp nhị phân ELF liên kết động (bao gồm tệp nhị phân PIE), cờ
mới `-bindnow` kích hoạt liên kết hàm ngay lập tức.

## Thư viện chuẩn {#library}

### Thay đổi Timer

Go 1.23 thực hiện hai thay đổi đáng kể đối với triển khai của
[`time.Timer`](/pkg/time#Timer) và [`time.Ticker`](/pkg/time#Ticker).

<!-- go.dev/issue/61542 -->
Thứ nhất, các `Timer` và `Ticker` không còn được chương trình tham chiếu
đủ điều kiện thu gom rác ngay lập tức, ngay cả khi
phương thức `Stop` của chúng chưa được gọi.
Các phiên bản Go trước không thu thập các `Timer` chưa dừng cho đến khi
chúng đã kích hoạt và không bao giờ thu thập các `Ticker` chưa dừng.

<!-- go.dev/issue/37196 -->
Thứ hai, channel timer liên quan đến `Timer` hoặc `Ticker` giờ
không có bộ đệm, với dung lượng 0.
Hiệu ứng chính của thay đổi này là Go giờ đảm bảo
rằng với bất kỳ lệnh gọi nào đến phương thức `Reset` hoặc `Stop`, không có giá trị cũ
được chuẩn bị trước lệnh gọi đó sẽ được gửi hoặc nhận sau lệnh gọi.
Các phiên bản Go trước sử dụng channel với bộ đệm một phần tử,
khiến việc sử dụng `Reset` và `Stop` đúng cách trở nên khó khăn.
Hiệu ứng có thể thấy của thay đổi này là `len` và `cap` của timer channel
giờ trả về 0 thay vì 1, điều này có thể ảnh hưởng đến các chương trình
thăm dò độ dài để quyết định liệu việc nhận trên timer channel
có thành công không.
Mã như vậy nên dùng receive không chặn thay thế.

Các hành vi mới này chỉ được kích hoạt khi chương trình Go chính
nằm trong module có dòng `go` trong `go.mod` sử dụng Go 1.23.0 trở lên.
Khi Go 1.23 build các chương trình cũ, các hành vi cũ vẫn có hiệu lực.
[Cài đặt GODEBUG](/doc/godebug) mới [`asynctimerchan=1`](/pkg/time/#NewTimer)
có thể được dùng để quay lại hành vi channel không đồng bộ
ngay cả khi chương trình khai báo Go 1.23.0 trở lên trong tệp `go.mod`.

### Gói unique mới

Gói mới [`unique`](/pkg/unique) cung cấp các tiện ích để
chuẩn hóa các giá trị (như "interning" hay "hash-consing").

Bất kỳ giá trị nào thuộc kiểu có thể so sánh đều có thể được chuẩn hóa với hàm
mới `Make[T]`, tạo ra tham chiếu đến bản sao chuẩn của
giá trị dưới dạng `Handle[T]`.
Hai `Handle[T]` bằng nhau khi và chỉ khi các giá trị được dùng để tạo ra
các handle bằng nhau, cho phép các chương trình loại bỏ trùng lặp giá trị và giảm
kích thước bộ nhớ.
So sánh hai giá trị `Handle[T]` rất hiệu quả, quy về so sánh con trỏ đơn giản.

### Các iterator

Gói mới [`iter`](/pkg/iter) cung cấp các định nghĩa cơ bản để làm việc với
các iterator do người dùng định nghĩa.

Gói [`slices`](/pkg/slices) thêm một số hàm hoạt động với iterator:
- [All](/pkg/slices#All) trả về iterator qua các chỉ số và giá trị của slice.
- [Values](/pkg/slices#Values) trả về iterator qua các phần tử của slice.
- [Backward](/pkg/slices#Backward) trả về iterator lặp qua
  slice theo chiều ngược.
- [Collect](/pkg/slices#Collect) thu thập giá trị từ iterator vào
  slice mới.
- [AppendSeq](/pkg/slices#AppendSeq) thêm giá trị từ iterator vào
  slice hiện có.
- [Sorted](/pkg/slices#Sorted) thu thập giá trị từ iterator vào
  slice mới, rồi sắp xếp slice.
- [SortedFunc](/pkg/slices#SortedFunc) giống `Sorted` nhưng với
  hàm so sánh.
- [SortedStableFunc](/pkg/slices#SortedStableFunc) giống `SortFunc`
  nhưng sử dụng thuật toán sắp xếp ổn định.
- [Chunk](/pkg/slices#Chunk) trả về iterator qua các
  sub-slice liên tiếp tối đa n phần tử của slice.

Gói [`maps`](/pkg/maps) thêm một số hàm hoạt động với iterator:
- [All](/pkg/maps#All) trả về iterator qua các cặp key-value từ map.
- [Keys](/pkg/maps#Keys) trả về iterator qua các key trong map.
- [Values](/pkg/maps#Values) trả về iterator qua các value trong map.
- [Insert](/pkg/maps#Insert) thêm các cặp key-value từ iterator vào map hiện có.
- [Collect](/pkg/maps#Collect) thu thập các cặp key-value từ iterator vào map mới và trả về nó.

### Gói structs mới

Gói mới [`structs`](/pkg/structs) cung cấp
các kiểu cho các trường struct sửa đổi thuộc tính của
kiểu struct chứa chúng như bố cục bộ nhớ.

Trong bản phát hành này, kiểu duy nhất như vậy là
[`HostLayout`](/pkg/structs#HostLayout)
cho biết rằng một struct có trường thuộc kiểu đó
có bố cục tuân theo
kỳ vọng của nền tảng host. HostLayout nên được dùng trong các kiểu được
truyền vào, trả về từ, hoặc được truy cập
qua con trỏ truyền vào/từ API host.
Nếu không có marker này, thứ tự bố cục struct không được
đảm bảo bởi đặc tả ngôn ngữ, mặc dù kể từ Go 1.23
bố cục host và ngôn ngữ tình cờ khớp nhau.

### Thay đổi nhỏ trong thư viện {#minor_library_changes}

#### [`archive/tar`](/pkg/archive/tar/)

Nếu đối số của [`FileInfoHeader`](/pkg/archive/tar#FileInfoHeader) triển khai interface mới [`FileInfoNames`](/pkg/archive/tar#FileInfoNames),
thì các phương thức interface sẽ được dùng để đặt Uname/Gname
của header tệp. Điều này cho phép ứng dụng ghi đè tra cứu
Uname/Gname phụ thuộc hệ thống.

#### [`crypto/tls`](/pkg/crypto/tls/)

Client TLS giờ hỗ trợ [đặc tả nháp](https://www.ietf.org/archive/id/draft-ietf-tls-esni-18.html) Encrypted Client Hello.
Tính năng này có thể được bật bằng cách đặt trường [`Config.EncryptedClientHelloConfigList`](/pkg/crypto/tls#Config.EncryptedClientHelloConfigList)
thành ECHConfigList được mã hóa cho host đang kết nối.

Kiểu [`QUICConn`](/pkg/crypto/tls#QUICConn) được dùng bởi các triển khai QUIC bao gồm các sự kiện mới
báo cáo về trạng thái tiếp tục session, và cung cấp cách cho
tầng QUIC thêm dữ liệu vào các ticket session và mục cache session.

Các bộ mã hóa 3DES đã bị xóa khỏi danh sách mặc định được dùng khi
[`Config.CipherSuites`](/pkg/crypto/tls#Config.CipherSuites) là nil. Giá trị mặc định có thể được khôi phục bằng cách thêm `tls3des=1` vào
biến môi trường GODEBUG.

Cơ chế trao đổi khóa hậu lượng tử thử nghiệm X25519Kyber768Draft00
giờ được bật theo mặc định khi [`Config.CurvePreferences`](/pkg/crypto/tls#Config.CurvePreferences) là nil.
Giá trị mặc định có thể được khôi phục bằng cách thêm `tlskyber=0` vào biến môi trường GODEBUG.
Điều này có thể hữu ích khi xử lý các TLS server có lỗi không xử lý các bản ghi lớn đúng cách,
gây ra timeout trong quá trình bắt tay (xem [TLS post-quantum TL;DR fail](https://tldr.fail/)).

Go 1.23 đã thay đổi hành vi của [`X509KeyPair`](/pkg/crypto/tls#X509KeyPair) và [`LoadX509KeyPair`](/pkg/crypto/tls#LoadX509KeyPair)
để điền trường [`Certificate.Leaf`](/pkg/crypto/tls#Certificate.Leaf) của [`Certificate`](/pkg/crypto/tls#Certificate) được trả về.
[Cài đặt GODEBUG](/doc/godebug) mới `x509keypairleaf` được thêm cho hành vi này.

#### [`crypto/x509`](/pkg/crypto/x509/)

[`CreateCertificateRequest`](/pkg/crypto/x509#CreateCertificateRequest) giờ hỗ trợ đúng cách các thuật toán chữ ký RSA-PSS.

[`CreateCertificateRequest`](/pkg/crypto/x509#CreateCertificateRequest) và [`CreateRevocationList`](/pkg/crypto/x509#CreateRevocationList) giờ xác minh chữ ký được tạo ra bằng khóa công khai của người ký. Nếu chữ ký không hợp lệ, một lỗi được trả về. Đây là hành vi của [`CreateCertificate`](/pkg/crypto/x509#CreateCertificate) từ Go 1.16.

[Cài đặt GODEBUG `x509sha1`](/pkg/crypto/x509#InsecureAlgorithmError) sẽ
bị xóa trong bản phát hành lớn tiếp theo (Go 1.24). Điều này có nghĩa là `crypto/x509`
sẽ không còn hỗ trợ xác minh chữ ký trên các chứng chỉ sử dụng các thuật toán chữ ký dựa trên SHA-1.

Hàm mới [`ParseOID`](/pkg/crypto/x509#ParseOID) phân tích chuỗi Object Identifier ASN.1 được mã hóa dạng chấm.
Kiểu [`OID`](/pkg/crypto/x509#OID) giờ triển khai các interface [`encoding.BinaryMarshaler`](/pkg/encoding#BinaryMarshaler),
[`encoding.BinaryUnmarshaler`](/pkg/encoding#BinaryUnmarshaler), [`encoding.TextMarshaler`](/pkg/encoding#TextMarshaler), [`encoding.TextUnmarshaler`](/pkg/encoding#TextUnmarshaler).

#### [`database/sql`](/pkg/database/sql/)

Các lỗi được trả về bởi các triển khai [`driver.Valuer`](/pkg/driver#Valuer) giờ được wrap để
cải thiện xử lý lỗi trong các thao tác như [`DB.Query`](/pkg/database/sql#DB.Query), [`DB.Exec`](/pkg/database/sql#DB.Exec),
và [`DB.QueryRow`](/pkg/database/sql#DB.QueryRow).

#### [`debug/elf`](/pkg/debug/elf/)

Gói `debug/elf` giờ định nghĩa [`PT_OPENBSD_NOBTCFI`](/pkg/debug/elf#PT_OPENBSD_NOBTCFI). [`ProgType`](/pkg/debug/elf#ProgType) này được
dùng để vô hiệu hóa Branch Tracking Control Flow Integrity (BTCFI) trên
các tệp nhị phân OpenBSD.

Giờ định nghĩa các hằng số kiểu symbol [`STT_RELC`](/pkg/debug/elf#STT_RELC), [`STT_SRELC`](/pkg/debug/elf#STT_SRELC), và
[`STT_GNU_IFUNC`](/pkg/debug/elf#STT_GNU_IFUNC).

#### [`encoding/binary`](/pkg/encoding/binary/)

Các hàm mới [`Encode`](/pkg/encoding/binary#Encode) và [`Decode`](/pkg/encoding/binary#Decode) là các phiên bản tương đương byte slice
của [`Read`](/pkg/encoding/binary#Read) và [`Write`](/pkg/encoding/binary#Write).
[`Append`](/pkg/encoding/binary#Append) cho phép marshaling nhiều dữ liệu vào cùng một byte slice.

#### [`go/ast`](/pkg/go/ast/)

Hàm mới [`Preorder`](/pkg/go/ast#Preorder) trả về iterator thuận tiện qua tất cả các
node của cây cú pháp.

#### [`go/types`](/pkg/go/types/)

<!-- see ../../../../2-language.md -->

Kiểu [`Func`](/pkg/go/types#Func), đại diện cho symbol hàm hoặc phương thức, giờ
có phương thức [`Func.Signature`](/pkg/go/types#Func.Signature) trả về kiểu của hàm, luôn
là `Signature`.

Kiểu [`Alias`](/pkg/go/types#Alias) giờ có phương thức [`Rhs`](/pkg/go/types#Rhs) trả về kiểu ở
bên phải của khai báo của nó: với `type A = B`, `Rhs` của A
là B. ([#66559](/issue/66559))

Các phương thức [`Alias.Origin`](/pkg/go/types#Alias.Origin), [`Alias.SetTypeParams`](/pkg/go/types#Alias.SetTypeParams), [`Alias.TypeParams`](/pkg/go/types#Alias.TypeParams),
và [`Alias.TypeArgs`](/pkg/go/types#Alias.TypeArgs) đã được thêm. Chúng cần thiết cho generic alias type.

<!-- CL 577715, CL 579076 -->
Theo mặc định, go/types giờ tạo ra các node kiểu [`Alias`](/pkg/go/types#Alias) cho type alias.
Hành vi này có thể được kiểm soát bởi cờ `GODEBUG` `gotypesalias`.
Giá trị mặc định của nó đã thay đổi từ 0 trong Go 1.22 thành 1 trong Go 1.23.

#### [`math/rand/v2`](/pkg/math/rand/v2/)

Hàm [`Uint`](/pkg/math/rand/v2#Uint) và phương thức [`Rand.Uint`](/pkg/math/rand/v2#Rand.Uint) đã được thêm.
Chúng bị thiếu sót ngoài ý muốn trong Go 1.22.

Phương thức mới [`ChaCha8.Read`](/pkg/math/rand/v2#ChaCha8.Read) triển khai interface [`io.Reader`](/pkg/io#Reader).

#### [`net`](/pkg/net/)

Kiểu mới [`KeepAliveConfig`](/pkg/net#KeepAliveConfig) cho phép tinh chỉnh các
tùy chọn keep-alive cho kết nối TCP, thông qua phương thức mới [`TCPConn.SetKeepAliveConfig`](/pkg/net#TCPConn.SetKeepAliveConfig)
và các trường KeepAliveConfig mới cho [`Dialer`](/pkg/net#Dialer) và [`ListenConfig`](/pkg/net#ListenConfig).

Kiểu [`DNSError`](/pkg/net#DNSError) giờ wrap các lỗi do timeout hoặc hủy.
Ví dụ, `errors.Is(someDNSErr, context.DeadlineExceedeed)`
giờ sẽ báo cáo liệu lỗi DNS có do timeout không.

Cài đặt `GODEBUG` mới `netedns0=0` vô hiệu hóa việc gửi các header bổ sung EDNS0
trên các yêu cầu DNS, vì chúng được báo cáo là làm hỏng DNS server
trên một số modem.

#### [`net/http`](/pkg/net/http/)

[`Cookie`](/pkg/net/http#Cookie) giờ bảo toàn dấu ngoặc kép bao quanh giá trị cookie.
Trường mới [`Cookie.Quoted`](/pkg/net/http#Cookie.Quoted) cho biết liệu [`Cookie.Value`](/pkg/net/http#Cookie.Value)
có được đặt trong dấu ngoặc kép ban đầu không.

Phương thức mới [`Request.CookiesNamed`](/pkg/net/http#Request.CookiesNamed) lấy tất cả các cookie khớp với tên đã cho.

Trường mới [`Cookie.Partitioned`](/pkg/net/http#Cookie.Partitioned) xác định các cookie có thuộc tính Partitioned.

Các mẫu được dùng bởi [`ServeMux`](/pkg/net/http#ServeMux) giờ cho phép một hoặc nhiều khoảng trắng hoặc tab sau tên phương thức.
Trước đây, chỉ có một khoảng trắng duy nhất được cho phép.

Hàm mới [`ParseCookie`](/pkg/net/http#ParseCookie) phân tích giá trị header Cookie và
trả về tất cả các cookie được đặt trong đó. Vì cùng tên cookie
có thể xuất hiện nhiều lần, Values trả về có thể chứa
nhiều hơn một giá trị cho một key nhất định.

Hàm mới [`ParseSetCookie`](/pkg/net/http#ParseSetCookie) phân tích giá trị header Set-Cookie và
trả về một cookie. Nó trả về lỗi khi có lỗi cú pháp.

[`ServeContent`](/pkg/net/http#ServeContent), [`ServeFile`](/pkg/net/http#ServeFile), và [`ServeFileFS`](/pkg/net/http#ServeFileFS) giờ xóa
các header `Cache-Control`, `Content-Encoding`, `Etag`, và `Last-Modified`
khi phục vụ lỗi. Các header này thường áp dụng cho
nội dung không có lỗi, nhưng không áp dụng cho văn bản lỗi.

Middleware wrap `ResponseWriter` và áp dụng mã hóa ngay lập tức,
chẳng hạn `Content-Encoding: gzip`, sẽ không hoạt động sau
thay đổi này. Hành vi cũ của [`ServeContent`](/pkg/net/http#ServeContent), [`ServeFile`](/pkg/net/http#ServeFile),
và [`ServeFileFS`](/pkg/net/http#ServeFileFS) có thể được khôi phục bằng cách đặt
`GODEBUG=httpservecontentkeepheaders=1`.

Lưu ý rằng middleware thay đổi kích thước nội dung được phục vụ
(chẳng hạn bằng cách nén) đã không hoạt động đúng khi
[`ServeContent`](/pkg/net/http#ServeContent) xử lý yêu cầu Range. Nén ngay lập tức
nên dùng header `Transfer-Encoding` thay vì `Content-Encoding`.

Đối với các yêu cầu đến, trường mới [`Request.Pattern`](/pkg/net/http#Request.Pattern) chứa mẫu [`ServeMux`](/pkg/net/http#ServeMux)
(nếu có) khớp với yêu cầu. Trường này không được đặt khi
`GODEBUG=httpmuxgo121=1` được đặt.

#### [`net/http/httptest`](/pkg/net/http/httptest/)

Phương thức mới [`NewRequestWithContext`](/pkg/net/http/httptest#NewRequestWithContext) tạo một yêu cầu đến với
[`context.Context`](/pkg/context#Context).

#### [`net/netip`](/pkg/net/netip/)

Trong Go 1.22 và trước đó, việc dùng
[`reflect.DeepEqual`](/pkg/reflect#DeepEqual) để so sánh một
[`Addr`](/pkg/net/netip#Addr) chứa địa chỉ IPv4 với một addr chứa
dạng IPv4-mapped IPv6 của địa chỉ đó trả về đúng không chính xác,
mặc dù các giá trị `Addr` khác nhau khi so sánh với `==` hoặc
[`Addr.Compare`](/pkg/net/netip#Addr.Compare).
Lỗi này giờ đã được sửa và cả ba cách tiếp cận đều báo cáo cùng
kết quả.

#### [`os`](/pkg/os/)

Hàm [`Stat`](/pkg/os#Stat) giờ đặt bit [`ModeSocket`](/pkg/os#ModeSocket) cho
các tệp là Unix socket trên Windows. Các tệp này được xác định
bằng cách có reparse tag được đặt thành `IO_REPARSE_TAG_AF_UNIX`.

Trên Windows, các bit mode được báo cáo bởi [`Lstat`](/pkg/os#Lstat) và [`Stat`](/pkg/os#Stat) cho
các reparse point đã thay đổi. Mount point không còn có [`ModeSymlink`](/pkg/os#ModeSymlink) được đặt,
và các reparse point không phải symlink, Unix socket, hoặc dedup file
giờ luôn có [`ModeIrregular`](/pkg/os#ModeIrregular) được đặt.
Hành vi này được kiểm soát bởi cài đặt `winsymlink`.
Đối với Go 1.23, nó mặc định là `winsymlink=1`.
Các phiên bản trước mặc định là `winsymlink=0`.

Hàm [`CopyFS`](/pkg/os#CopyFS) sao chép một [`io/fs.FS`](/pkg/io/fs#FS) vào hệ thống tệp cục bộ.

Trên Windows, [`Readlink`](/pkg/os#Readlink) không còn cố gắng chuẩn hóa các volume
thành ký tự ổ đĩa, điều này không phải lúc nào cũng có thể.
Hành vi này được kiểm soát bởi cài đặt `winreadlinkvolume`.
Đối với Go 1.23, nó mặc định là `winreadlinkvolume=1`.
Các phiên bản trước mặc định là `winreadlinkvolume=0`.

<!-- go.dev/issue/62654, CL 570036, CL 570681 -->
Trên Linux với hỗ trợ pidfd (thường là Linux v5.4+),
các hàm và phương thức liên quan đến [`Process`](/pkg/os#Process) sử dụng pidfd (thay vì
PID) nội bộ, loại bỏ khả năng nhắm mục tiêu sai khi PID được OS tái sử dụng. Hỗ trợ pidfd hoàn toàn trong suốt với người dùng, ngoại trừ các file descriptor tiến trình bổ sung mà tiến trình có thể có.

#### [`path/filepath`](/pkg/path/filepath/)

Hàm mới [`Localize`](/pkg/path/filepath#Localize) chuyển đổi an toàn một
đường dẫn được phân cách bằng dấu gạch chéo thành đường dẫn hệ điều hành.

Trên Windows, [`EvalSymlinks`](/pkg/path/filepath#EvalSymlinks) không còn đánh giá các mount point,
đây là nguồn gốc của nhiều sự không nhất quán và lỗi.
Hành vi này được kiểm soát bởi cài đặt `winsymlink`.
Đối với Go 1.23, nó mặc định là `winsymlink=1`.
Các phiên bản trước mặc định là `winsymlink=0`.

Trên Windows, [`EvalSymlinks`](/pkg/path/filepath#EvalSymlinks) không còn cố gắng chuẩn hóa
các volume thành ký tự ổ đĩa, điều này không phải lúc nào cũng có thể.
Hành vi này được kiểm soát bởi cài đặt `winreadlinkvolume`.
Đối với Go 1.23, nó mặc định là `winreadlinkvolume=1`.
Các phiên bản trước mặc định là `winreadlinkvolume=0`.

#### [`reflect`](/pkg/reflect/)

Các phương thức mới đồng nghĩa với các phương thức cùng tên
trong [`Value`](/pkg/reflect#Value) được thêm vào [`Type`](/pkg/reflect#Type):
1. [`Type.OverflowComplex`](/pkg/reflect#Type.OverflowComplex)
2. [`Type.OverflowFloat`](/pkg/reflect#Type.OverflowFloat)
3. [`Type.OverflowInt`](/pkg/reflect#Type.OverflowInt)
4. [`Type.OverflowUint`](/pkg/reflect#Type.OverflowUint)

Hàm mới [`SliceAt`](/pkg/reflect#SliceAt) tương tự với [`NewAt`](/pkg/reflect#NewAt), nhưng cho slice.

Các phương thức [`Value.Pointer`](/pkg/reflect#Value.Pointer) và [`Value.UnsafePointer`](/pkg/reflect#Value.UnsafePointer) giờ hỗ trợ các giá trị thuộc loại [`String`](/pkg/reflect#String).

Các phương thức mới [`Value.Seq`](/pkg/reflect#Value.Seq) và [`Value.Seq2`](/pkg/reflect#Value.Seq2) trả về các chuỗi lặp qua giá trị
như thể nó được dùng trong vòng lặp for/range.
Các phương thức mới [`Type.CanSeq`](/pkg/reflect#Type.CanSeq) và [`Type.CanSeq2`](/pkg/reflect#Type.CanSeq2) báo cáo liệu việc gọi
[`Value.Seq`](/pkg/reflect#Value.Seq) và [`Value.Seq2`](/pkg/reflect#Value.Seq2), tương ứng, có thành công mà không panic không.

#### [`runtime/debug`](/pkg/runtime/debug/)

Hàm [`SetCrashOutput`](/pkg/runtime/debug#SetCrashOutput) cho phép người dùng chỉ định một
tệp thay thế mà runtime nên ghi báo cáo crash nghiêm trọng của nó.
Nó có thể được dùng để xây dựng cơ chế báo cáo tự động cho tất cả
các crash không mong đợi, không chỉ những crash trong goroutine dùng
`recover` tường minh.

<!-- pacify TestCheckAPIFragments -->

#### [`runtime/pprof`](/pkg/runtime/pprof/)

Độ sâu stack tối đa cho các profile `alloc`, `mutex`, `block`, `threadcreate` và `goroutine`
đã được tăng từ 32 lên 128 frame.

#### [`runtime/trace`](/pkg/runtime/trace/)

<!-- go.dev/issue/65319 -->
Runtime giờ xả dữ liệu trace một cách tường minh khi chương trình crash do
panic không bị bắt. Điều này có nghĩa là dữ liệu trace đầy đủ hơn sẽ có sẵn trong
trace nếu chương trình crash trong khi đang trace.

#### [`slices`](/pkg/slices/)

<!-- see ../../3-iter.md -->

<!-- see ../../3-iter.md -->

Hàm [`Repeat`](/pkg/slices#Repeat) trả về slice mới lặp lại
slice đã cung cấp số lần đã cho.

#### [`sync`](/pkg/sync/)

Phương thức [`Map.Clear`](/pkg/sync#Map.Clear) xóa tất cả các mục, tạo ra
[`Map`](/pkg/sync#Map) rỗng. Nó tương tự với `clear`.

#### [`sync/atomic`](/pkg/sync/atomic/)

<!-- Issue #61395 -->
Các toán tử mới [`And`](/pkg/sync/atomic#And) và [`Or`](/pkg/sync/atomic#Or) áp dụng `AND` hoặc `OR` theo bit vào
đầu vào đã cho, trả về giá trị cũ.

#### [`syscall`](/pkg/syscall/)

Gói syscall giờ định nghĩa [`WSAENOPROTOOPT`](/pkg/syscall#WSAENOPROTOOPT) trên Windows.

Hàm [`GetsockoptInt`](/pkg/syscall#GetsockoptInt) giờ được hỗ trợ trên Windows.

#### [`testing/fstest`](/pkg/testing/fstest/)

[`TestFS`](/pkg/testing/fstest#TestFS) giờ trả về lỗi có cấu trúc có thể được unwrap
(qua phương thức `Unwrap() []error`). Điều này cho phép kiểm tra lỗi
bằng [`errors.Is`](/pkg/errors#Is) hoặc [`errors.As`](/pkg/errors#As).

#### [`text/template`](/pkg/text/template/)

Các template giờ hỗ trợ hành động mới "else with", giúp giảm độ phức tạp của template trong một số trường hợp sử dụng.

#### [`time`](/pkg/time/)

[`Parse`](/pkg/time#Parse) và [`ParseInLocation`](/pkg/time#ParseInLocation) giờ trả về lỗi nếu offset múi giờ
nằm ngoài phạm vi.

Trên Windows, [`Timer`](/pkg/time#Timer), [`Ticker`](/pkg/time#Ticker), và các hàm đưa goroutine vào trạng thái ngủ,
như [`Sleep`](/pkg/time#Sleep), đã được cải thiện độ phân giải thời gian lên 0.5ms thay vì 15.6ms.

#### [`unicode/utf16`](/pkg/unicode/utf16/)

Hàm [`RuneLen`](/pkg/unicode/utf16#RuneLen) trả về số lượng từ 16-bit trong
mã hóa UTF-16 của rune. Nó trả về -1 nếu rune
không phải là giá trị hợp lệ để mã hóa trong UTF-16.

## Các cổng {#ports}

### Darwin {#darwin}

<!-- go.dev/issue/64207 -->
Như đã [thông báo](go1.22#darwin) trong ghi chú phát hành Go 1.22,
Go 1.23 yêu cầu macOS 11 Big Sur trở lên;
hỗ trợ cho các phiên bản trước đã bị ngừng.

### Linux {#linux}

<!-- go.dev/issue/67001 -->
Go 1.23 là bản phát hành cuối cùng yêu cầu phiên bản kernel Linux 2.6.32 trở lên. Go 1.24 sẽ yêu cầu phiên bản kernel Linux 3.2 trở lên.

### OpenBSD {#openbsd}

<!-- go.dev/issue/55999, CL 518629, CL 518630 -->
Go 1.23 thêm hỗ trợ thử nghiệm cho OpenBSD trên RISC-V 64-bit (`GOOS=openbsd`, `GOARCH=riscv64`).

### ARM64 {#arm64}

<!-- go.dev/issue/60905, CL 559555 -->
Go 1.23 giới thiệu biến môi trường mới `GOARM64`, chỉ định phiên bản mục tiêu tối thiểu của kiến trúc ARM64 tại thời điểm biên dịch. Các giá trị được phép là `v8.{0-9}` và `v9.{0-5}`. Điều này có thể được theo sau bởi một tùy chọn chỉ định các extension được triển khai bởi phần cứng mục tiêu. Các tùy chọn hợp lệ là `,lse` và `,crypto`.

Biến môi trường `GOARM64` mặc định là `v8.0`.

### RISC-V {#riscv}

<!-- go.dev/issue/61476, CL 541135 -->
Go 1.23 giới thiệu biến môi trường mới `GORISCV64`, chọn [hồ sơ ứng dụng người dùng RISC-V](https://github.com/riscv/riscv-profiles/blob/main/src/profiles.adoc) để biên dịch. Các giá trị được phép là `rva20u64` và `rva22u64`.

Biến môi trường `GORISCV64` mặc định là `rva20u64`.

### Wasm {#wasm}

<!-- go.dev/issue/63718 -->
Script `go_wasip1_wasm_exec` trong `GOROOT/misc/wasm` đã ngừng hỗ trợ
các phiên bản `wasmtime` < 14.0.0.

<!-- These items need to be completed and moved to an appropriate location in the release notes. -->

<!-- These items need to be reviewed, and mentioned in the Go 1.23 release notes if applicable.

None right now; more may come up later on.
-->

<!-- Maybe should be documented? Maybe shouldn't? Someone familiar with the change needs to determine.

CL 359594 ("x/website/_content/ref/mod: document dotless module paths") - resolved go.dev/issue/32819 ("cmd/go: document that module names without dots are reserved") and also mentioned accepted proposal go.dev/issue/37641
CL 555075 ("x/tools/go/ssa: support range-over-func") - x/tools CL implements range-over-func support in x/tools/go/ssa for accepted proposal https://go.dev/issue/66601; this particular proposal and change doesn't seem to need a dedicated mention in Go 1.23 release notes but someone more familiar should take another look
-->

<!-- Items that don't need to be mentioned in Go 1.23 release notes but are picked up by relnote todo.

CL 458895 - an x/playground fix that mentioned an accepted cmd/go proposal go.dev/issue/40728 in Go 1.16 milestone...
CL 582097 - an x/build CL working on relnote itself; it doesn't need a release note
CL 561935 - crypto CL that used purego tag and mentioned accepted-but-not-implemented proposal https://go.dev/issue/23172 to document purego tag; doesn't need a release note
CL 568340 - fixed a spurious race in time.Ticker.Reset (added via accepted proposal https://go.dev/issue/33184), doesn't seem to need a release note
CL 562619 - x/website CL documented minimum bootstrap version on go.dev, mentioning accepted proposals go.dev/issue/54265 and go.dev/issue/44505; doesn't need a release note
CL 557055 - x/tools CL implemented accepted proposal https://go.dev/issue/46941 for x/tools/go/ssa
CL 564275 - an x/tools CL that updates test data in preparation for accepted proposal https://go.dev/issue/51473; said proposal isn't implemented for Go 1.23 and so it doesn't need a release note
CL 572535 - used "unix" build tag in more places, mentioned accepted proposal https://go.dev/issue/51572; doesn't need a release note
CL 555255 - an x/tools CL implements accepted proposal https://go.dev/issue/53367 for x/tools/go/cfg
CL 585216 - an x/build CL mentions accepted proposal https://go.dev/issue/56001 because it fixed a bug causing downloads not to be produced for that new-to-Go-1.22 port; this isn't relevant to Go 1.23 release notes
CL 481062 - added examples for accepted proposal https://go.dev/issue/56102; doesn't need a release note
CL 497195 - an x/net CL adds one of 4 fields for accepted proposal https://go.dev/issue/57893 in x/net/http2; seemingly not related to net/http and so doesn't need a Go 1.23 release note
CL 463097, CL 568198 - x/net CLs that implemented accepted proposal https://go.dev/issue/57953 for x/net/websocket; no need for rel note
many x/net CLs - work on accepted proposal https://go.dev/issue/58547 to add a QUIC implementation to x/net/quic
CL 514775 - implements a performance optimization for accepted proposal https://go.dev/issue/59488
CL 484995 - x/sys CL implements accepted proposal https://go.dev/issue/59537 to add x/sys/unix API
CL 555597 - optimizes TypeFor (added in accepted proposal https://go.dev/issue/60088) for non-interface types; doesn't seem to need a release note
a few x/tools CLs deprecated and deleted the experimental golang.org/x/tools/cmd/getgo tool per accepted proposal https://go.dev/issue/60951; an unreleased change and not something that's in scope of Go 1.23 release notes
many x/vuln CLs to implement accepted proposal https://go.dev/issue/61347 ("x/vuln: convert govulncheck output to sarif format") in govulncheck
CL 516355 - x/crypto CL that implemented accepted proposal https://go.dev/issue/61447 for x/crypto/ssh; doesn't need a Go 1.23 release note
CL 559799 - a Go 1.22 release note edit CL mentioned a Go 1.22 accepted proposal https://go.dev/issue/62039, a little after Go 1.23 development began
CL 581555 - an x/tools CL mentioned accepted proposal https://go.dev/issue/62292 for x/tools/go/aalysis; doesn't need a Go 1.23 release note
CL 578355 - mentioned accepted proposal https://go.dev/issue/63131 to add GOARCH=wasm32, but that proposal hasn't been implemented in Go 1.23 so it doesn't need a release note
CL 543335 - x/exp CL that backported a change to behavior in slices package (accepted proposal https://go.dev/issue/63393) to x/exp/slices; doesn't need a Go 1.23 release note
CL 556820 - x/tools CL implemented accepted proposal https://go.dev/issue/64548 for x/tools/go/analysis
CL 557056 - x/tools CL implemented accepted proposal https://go.dev/issue/64608 for x/tools/go/packages
CL 558695 - x/crypto CL worked on accepted proposal https://go.dev/issue/64962 for x/crypto/ssh
CL 572016 - x/tools CL implemented accepted proposal https://go.dev/issue/65754 for x/tools/go/cfg
a few x/tools CLs tagged and deleted the golang.org/x/tools/cmd/guru command per accepted proposal https://go.dev/issue/65880; an unreleased change and not something that's in scope of Go 1.23 release notes
CL 580076 - seemingly internal cmd/go change to propagate module information for accepted proposal https://go.dev/issue/66315; doesn't seem to warrant a release note
CL 529816 - the 'tests' vet check was initially added to the 'go test' suite per accepted proposal https://go.dev/issue/44251, but the change was rolled back in CL 571695, with no roll forward as of 2024-05-23; nothing to document in Go 1.23 release notes for it at this time
CL 564035 - changed encoding/xml, but the change was too disrptive and rolled back in CL 570175, reopening tracking issue go.dev/issue/65691; nothing to document in Go 1.23 release notes
CL 587855 - a demonstration of benefit of accepted proposal https://go.dev/issue/60529; actual change isn't happening in Go 1.23 so doesn't need a release note
CL 526875 - x/crypto CL implemented accepted proposal https://go.dev/issue/62518 for x/crypto/ssh
-->
