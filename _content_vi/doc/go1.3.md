---
title: Ghi chú phát hành Go 1.3
---

## Giới thiệu về Go 1.3 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.3, ra đời sáu tháng sau 1.2
và không có thay đổi ngôn ngữ nào.
Nó tập trung chủ yếu vào công việc triển khai, cung cấp
bộ gom rác chính xác,
một tái cấu trúc lớn của toolchain trình biên dịch giúp
build nhanh hơn, đặc biệt cho các dự án lớn,
cải thiện hiệu năng đáng kể trên diện rộng,
và hỗ trợ cho DragonFly BSD, Solaris, Plan 9 và kiến trúc Native Client của Google (NaCl).
Nó cũng có một tinh chỉnh quan trọng đối với mô hình bộ nhớ liên quan đến đồng bộ hóa.
Như thường lệ, Go 1.3 vẫn giữ [cam kết
tương thích](/doc/go1compat.html),
và hầu hết mọi thứ
sẽ tiếp tục biên dịch và chạy mà không thay đổi khi chuyển sang 1.3.

## Thay đổi về các hệ điều hành và kiến trúc được hỗ trợ {#os}

### Ngừng hỗ trợ Windows 2000 {#win2000}

Microsoft đã ngừng hỗ trợ Windows 2000 vào năm 2010.
Vì nó có [khó khăn triển khai](https://codereview.appspot.com/74790043)
liên quan đến xử lý ngoại lệ (tín hiệu trong thuật ngữ Unix),
kể từ Go 1.3, Go cũng không hỗ trợ nó.

### Hỗ trợ DragonFly BSD {#dragonfly}

Go 1.3 bây giờ bao gồm hỗ trợ thử nghiệm cho DragonFly BSD trên các kiến trúc `amd64` (x86 64 bit) và `386` (x86 32 bit).
Nó dùng DragonFly BSD 3.6 trở lên.

### Hỗ trợ FreeBSD {#freebsd}

Mặc dù không được thông báo vào thời điểm đó, nhưng kể từ khi phát hành Go 1.2, hỗ trợ cho Go trên FreeBSD
yêu cầu FreeBSD 8 trở lên.

Kể từ Go 1.3, hỗ trợ cho Go trên FreeBSD yêu cầu rằng nhân được biên dịch với
flag `COMPAT_FREEBSD32` được cấu hình.

Cùng với việc chuyển sang EABI syscall cho các nền tảng ARM, Go 1.3 sẽ chỉ chạy trên FreeBSD 10.
Các nền tảng x86, 386 và amd64, không bị ảnh hưởng.

### Hỗ trợ Native Client {#nacl}

Hỗ trợ cho kiến trúc máy ảo Native Client đã trở lại Go với bản phát hành 1.3.
Nó chạy trên các kiến trúc Intel 32 bit (`GOARCH=386`) và cả trên Intel 64 bit, nhưng dùng
con trỏ 32 bit (`GOARCH=amd64p32`).
Chưa có hỗ trợ cho Native Client trên ARM.
Lưu ý rằng đây là Native Client (NaCl), không phải Portable Native Client (PNaCl).
Chi tiết về Native Client có [tại đây](https://developers.google.com/native-client/dev/);
cách thiết lập phiên bản Go được mô tả [tại đây](/wiki/NativeClient).

### Hỗ trợ NetBSD {#netbsd}

Kể từ Go 1.3, hỗ trợ cho Go trên NetBSD yêu cầu NetBSD 6.0 trở lên.

### Hỗ trợ OpenBSD {#openbsd}

Kể từ Go 1.3, hỗ trợ cho Go trên OpenBSD yêu cầu OpenBSD 5.5 trở lên.

### Hỗ trợ Plan 9 {#plan9}

Go 1.3 bây giờ bao gồm hỗ trợ thử nghiệm cho Plan 9 trên kiến trúc `386` (x86 32 bit).
Nó yêu cầu syscall `Tsemacquire`, đã có trong Plan 9 từ tháng 6 năm 2012.

### Hỗ trợ Solaris {#solaris}

Go 1.3 bây giờ bao gồm hỗ trợ thử nghiệm cho Solaris trên kiến trúc `amd64` (x86 64 bit).
Nó yêu cầu illumos, Solaris 11 trở lên.

## Thay đổi về mô hình bộ nhớ {#memory}

Mô hình bộ nhớ Go 1.3 [thêm một quy tắc mới](https://codereview.appspot.com/75130045)
liên quan đến gửi và nhận trên các channel có buffer,
để làm rõ rằng một channel có buffer có thể được dùng như một
semaphore đơn giản, dùng một lần gửi vào
channel để acquire và một lần nhận từ channel để release.
Đây không phải là thay đổi ngôn ngữ, chỉ là làm rõ về một thuộc tính được mong đợi của giao tiếp.

## Thay đổi về triển khai và công cụ {#impl}

### Stack {#stacks}

Go 1.3 đã thay đổi triển khai của goroutine stack từ mô hình
"phân đoạn" cũ sang mô hình liên tục.
Khi một goroutine cần stack nhiều hơn hiện có, stack của nó được chuyển sang một khối bộ nhớ đơn lớn hơn.
Overhead của thao tác chuyển này được khấu hao tốt và loại bỏ vấn đề "hot spot" cũ
khi một phép tính liên tục bước qua ranh giới phân đoạn.
Chi tiết bao gồm số hiệu năng có trong
[tài liệu thiết kế](/s/contigstacks) này.

### Thay đổi đối với bộ gom rác {#garbage_collector}

Trong một thời gian, bộ gom rác đã _chính xác_ khi kiểm tra
các giá trị trong heap; bản phát hành Go 1.3 thêm độ chính xác tương đương cho các giá trị trên stack.
Điều này có nghĩa là một giá trị Go không phải con trỏ như một số nguyên sẽ không bao giờ bị nhầm là
con trỏ và ngăn bộ nhớ không dùng khỏi bị thu hồi.

Bắt đầu từ Go 1.3, thời gian chạy giả định rằng các giá trị có kiểu con trỏ
chứa các con trỏ và các giá trị khác thì không.
Giả định này là cơ bản cho hành vi chính xác của cả mở rộng stack
và bộ gom rác.
Các chương trình dùng [gói unsafe](/pkg/unsafe/)
để lưu số nguyên trong các giá trị kiểu con trỏ là bất hợp pháp và sẽ crash nếu thời gian chạy phát hiện hành vi.
Các chương trình dùng [gói unsafe](/pkg/unsafe/) để lưu các con trỏ
trong các giá trị kiểu số nguyên cũng bất hợp pháp nhưng khó chẩn đoán hơn trong quá trình thực thi.
Vì các con trỏ bị ẩn khỏi thời gian chạy, một mở rộng stack hoặc bộ gom rác
có thể thu hồi bộ nhớ mà chúng trỏ đến, tạo ra
[dangling pointer](https://en.wikipedia.org/wiki/Dangling_pointer).

_Cập nhật_: Mã nguồn dùng `unsafe.Pointer` để chuyển đổi
một giá trị kiểu số nguyên được giữ trong bộ nhớ thành con trỏ là bất hợp pháp và phải được viết lại.
Mã nguồn như vậy có thể được xác định bởi `go vet`.

### Duyệt map {#map}

Các lần duyệt qua các map nhỏ không còn xảy ra theo thứ tự nhất quán.
Go 1 định nghĩa rằng "[Thứ tự duyệt qua map
không được chỉ định và không được đảm bảo là giống nhau từ lần duyệt này sang lần tiếp theo.](/ref/spec#For_statements)"
Để giữ mã nguồn không phụ thuộc vào thứ tự duyệt map,
Go 1.0 bắt đầu mỗi lần duyệt map tại một chỉ số ngẫu nhiên trong map.
Một triển khai map mới được giới thiệu trong Go 1.1 đã bỏ qua việc ngẫu nhiên hóa
lần duyệt cho các map với tám hoặc ít phần tử hơn, mặc dù thứ tự duyệt
vẫn có thể khác nhau từ hệ thống này sang hệ thống khác.
Điều này đã cho phép mọi người viết các chương trình Go 1.1 và Go 1.2
phụ thuộc vào thứ tự duyệt map nhỏ và do đó chỉ hoạt động đáng tin cậy trên một số hệ thống nhất định.
Go 1.3 tái giới thiệu duyệt ngẫu nhiên cho các map nhỏ để xóa bỏ các lỗi này.

_Cập nhật_: Nếu mã nguồn giả định thứ tự duyệt cố định cho các map nhỏ,
nó sẽ bị hỏng và phải được viết lại để không giả định điều đó.
Vì chỉ các map nhỏ bị ảnh hưởng, vấn đề thường phát sinh nhiều nhất trong các test.

### Linker {#liblink}

Là một phần của [cải cách tổng thể](/s/go13linker) đối với
Go linker, các trình biên dịch và linker đã được tái cấu trúc.
Linker vẫn là một chương trình C, nhưng bây giờ giai đoạn lựa chọn lệnh
là một phần của linker đã được chuyển sang trình biên dịch thông qua việc tạo ra một thư viện mới
gọi là `liblink`.
Bằng cách thực hiện lựa chọn lệnh chỉ một lần, khi gói được biên dịch lần đầu,
điều này có thể tăng tốc đáng kể việc biên dịch các dự án lớn.

_Cập nhật_: Mặc dù đây là thay đổi nội bộ lớn, nó không nên có
tác dụng gì đối với các chương trình.

### Trạng thái của gccgo {#gccgo}

Bản phát hành GCC 4.9 sẽ chứa phiên bản Go 1.2 (không phải 1.3) của gccgo.
Lịch phát hành cho các dự án GCC và Go không trùng nhau,
có nghĩa là 1.3 sẽ có sẵn trong nhánh phát triển nhưng
bản phát hành GCC tiếp theo, 4.10, có thể sẽ có phiên bản Go 1.4 của gccgo.

### Thay đổi đối với lệnh go {#gocmd}

Lệnh [`cmd/go`](/cmd/go/) có một số
tính năng mới.
Các lệnh con [`go run`](/cmd/go/) và
[`go test`](/cmd/go/)
hỗ trợ tùy chọn mới `-exec` để chỉ định cách thay thế
để chạy binary kết quả.
Mục đích trước mắt của nó là hỗ trợ NaCl.

Hỗ trợ test coverage của lệnh con [`go test`](/cmd/go/)
bây giờ tự động đặt chế độ coverage thành `-atomic`
khi race detector được bật, để loại bỏ các báo cáo sai về truy cập không an toàn
vào các bộ đếm coverage.

Lệnh con [`go test`](/cmd/go/)
bây giờ luôn build gói, ngay cả khi không có file test nào.
Trước đây, nó sẽ không làm gì nếu không có file test nào.

Lệnh con [`go build`](/cmd/go/)
hỗ trợ tùy chọn mới `-i` để cài đặt các dependency
của target được chỉ định, nhưng không phải bản thân target.

Biên dịch chéo với [`cgo`](/cmd/cgo/) được bật
bây giờ được hỗ trợ.
Các biến môi trường CC\_FOR\_TARGET và CXX\_FOR\_TARGET được dùng
khi chạy all.bash để chỉ định các trình biên dịch chéo
cho mã C và C++ tương ứng.

Cuối cùng, lệnh go bây giờ hỗ trợ các gói import các file Objective-C
(có hậu tố `.m`) thông qua cgo.

### Thay đổi đối với cgo {#cgo}

Lệnh [`cmd/cgo`](/cmd/cgo/),
xử lý các khai báo `import "C"` trong các gói Go,
đã sửa một lỗi nghiêm trọng có thể làm cho một số gói ngừng biên dịch.
Trước đây, tất cả các con trỏ đến các kiểu struct chưa hoàn chỉnh đều được dịch sang kiểu Go `*[0]byte`,
với tác dụng là trình biên dịch Go không thể chẩn đoán việc truyền con trỏ của một loại struct
cho một hàm mong đợi loại khác.
Go 1.3 sửa lỗi này bằng cách dịch mỗi struct chưa hoàn chỉnh khác nhau
sang một kiểu được đặt tên khác nhau.

Với khai báo C `typedef struct S T` cho một `struct S` chưa hoàn chỉnh,
một số mã Go dùng lỗi này để tham chiếu đến các kiểu `C.struct_S` và `C.T` thay thế cho nhau.
Cgo bây giờ tường minh cho phép cách dùng này, ngay cả đối với các kiểu struct đã hoàn chỉnh.
Tuy nhiên, một số mã Go cũng dùng lỗi này để truyền (ví dụ) một `*C.FILE`
từ gói này sang gói khác.
Điều này không hợp lệ và không còn hoạt động nữa: nói chung các gói Go
nên tránh phơi bày các kiểu và tên C trong API của chúng.

_Cập nhật_: Mã nguồn nhầm lẫn các con trỏ đến các kiểu chưa hoàn chỉnh hoặc
truyền chúng qua ranh giới gói sẽ không còn biên dịch được
và phải được viết lại.
Nếu việc chuyển đổi là đúng và phải được giữ lại,
hãy dùng một chuyển đổi tường minh qua [`unsafe.Pointer`](/pkg/unsafe/#Pointer).

### Yêu cầu SWIG 3.0 cho các chương trình dùng SWIG {#swig}

Đối với các chương trình Go dùng SWIG, SWIG phiên bản 3.0 hiện được yêu cầu.
Lệnh [`cmd/go`](/cmd/go) bây giờ sẽ liên kết
các file object được tạo bởi SWIG trực tiếp vào binary, thay vì
build và liên kết với một thư viện dùng chung.

### Phân tích flag dòng lệnh {#gc_flag}

Trong toolchain gc, các trình hợp dịch bây giờ dùng
các quy tắc phân tích flag dòng lệnh giống như gói flag Go, khác biệt
so với cách phân tích flag Unix truyền thống.
Điều này có thể ảnh hưởng đến các script gọi công cụ trực tiếp.
Ví dụ,
`go tool 6a -SDfoo` bây giờ phải được viết
`go tool 6a -S -D foo`.
(Thay đổi tương tự đã được thực hiện cho các trình biên dịch và linker trong [Go 1.1](/doc/go1.1#gc_flag).)

### Thay đổi đối với godoc {#godoc}

Khi được gọi với flag `-analysis`,
[godoc](https://godoc.org/golang.org/x/tools/cmd/godoc)
bây giờ thực hiện phân tích tĩnh tinh vi của mã nguồn nó đánh chỉ số.
Kết quả phân tích được trình bày trong cả chế độ xem nguồn và
chế độ xem tài liệu gói, và bao gồm biểu đồ lệnh gọi của mỗi gói
và các mối quan hệ giữa
các định nghĩa và tham chiếu,
các kiểu và phương thức của chúng,
các interface và triển khai của chúng,
các thao tác gửi và nhận trên channel,
các hàm và các caller của chúng, và
các điểm gọi và các callee của chúng.

### Linh tinh {#misc}

Chương trình `misc/benchcmp` so sánh
hiệu năng qua các lần chạy benchmarking đã được viết lại.
Từng là một script shell và awk trong kho lưu trữ chính, bây giờ nó là một chương trình Go trong repo `go.tools`.
Tài liệu có [tại đây](https://godoc.org/golang.org/x/tools/cmd/benchcmp).

Đối với một số ít người chúng ta build các bản phân phối Go, công cụ `misc/dist` đã được
di chuyển và đổi tên; bây giờ nó nằm trong `misc/makerelease`, vẫn trong kho lưu trữ chính.

## Hiệu năng {#performance}

Hiệu năng của các binary Go trong bản phát hành này đã được cải thiện trong nhiều trường hợp do các thay đổi
trong thời gian chạy và bộ gom rác, cùng với một số thay đổi đối với thư viện.
Các trường hợp đáng chú ý bao gồm:

  - Thời gian chạy xử lý defer hiệu quả hơn, giảm dung lượng bộ nhớ khoảng hai kilobyte
    mỗi goroutine gọi defer.
  - Bộ gom rác đã được tăng tốc, sử dụng thuật toán sweep đồng thời,
    song song hóa tốt hơn và các trang lớn hơn.
    Tác dụng tích lũy có thể là giảm 50-70% thời gian dừng của bộ gom rác.
  - Race detector (xem [hướng dẫn này](/doc/articles/race_detector.html))
    bây giờ nhanh hơn khoảng 40%.
  - Gói biểu thức chính quy [`regexp`](/pkg/regexp/)
    bây giờ nhanh hơn đáng kể cho một số biểu thức đơn giản do triển khai
    một engine thực thi một lượt thứ hai.
    Việc chọn engine nào để dùng là tự động;
    chi tiết bị ẩn khỏi người dùng.

Ngoài ra, thời gian chạy bây giờ bao gồm trong các dump stack thời gian một goroutine đã bị block,
điều này có thể là thông tin hữu ích khi gỡ lỗi các deadlock hoặc vấn đề hiệu năng.

## Thay đổi đối với thư viện chuẩn {#library}

### Các gói mới {#new_packages}

Gói mới [`debug/plan9obj`](/pkg/debug/plan9obj/) đã được thêm vào thư viện chuẩn.
Nó triển khai truy cập vào các file object Plan 9 [a.out](https://9p.io/magic/man2html/6/a.out).

### Thay đổi lớn đối với thư viện {#major_library_changes}

Một lỗi trước đây trong [`crypto/tls`](/pkg/crypto/tls/)
làm cho có thể bỏ qua xác minh trong TLS một cách vô tình.
Trong Go 1.3, lỗi đã được sửa: người ta phải chỉ định ServerName hoặc
InsecureSkipVerify, và nếu ServerName được chỉ định thì nó được thực thi.
Điều này có thể làm hỏng mã nguồn hiện có phụ thuộc không đúng vào
hành vi không an toàn.

Có một kiểu mới quan trọng được thêm vào thư viện chuẩn: [`sync.Pool`](/pkg/sync/#Pool).
Nó cung cấp một cơ chế hiệu quả để triển khai một số loại cache nhất định mà bộ nhớ
có thể được thu hồi tự động bởi hệ thống.

Trình trợ giúp benchmarking [`B`](/pkg/testing/#B)
của gói [`testing`](/pkg/testing/),
bây giờ có phương thức
[`RunParallel`](/pkg/testing/#B.RunParallel)
để dễ dàng chạy các benchmark tập thể dục nhiều CPU hơn.

_Cập nhật_: Sửa lỗi crypto/tls có thể làm hỏng mã nguồn hiện có, nhưng
mã nguồn đó là sai và nên được cập nhật.

### Thay đổi nhỏ đối với thư viện {#minor_library_changes}

Danh sách sau đây tóm tắt một số thay đổi nhỏ đối với thư viện, chủ yếu là bổ sung.
Xem tài liệu gói liên quan để biết thêm thông tin về từng thay đổi.

  - Trong gói [`crypto/tls`](/pkg/crypto/tls/),
    hàm mới [`DialWithDialer`](/pkg/crypto/tls/#DialWithDialer)
    cho phép thiết lập kết nối TLS bằng một dialer hiện có, giúp dễ dàng hơn
    kiểm soát các tùy chọn dial như timeout.
    Gói bây giờ cũng báo cáo phiên bản TLS được dùng bởi kết nối trong cấu trúc
    [`ConnectionState`](/pkg/crypto/tls/#ConnectionState).
  - Hàm [`CreateCertificate`](/pkg/crypto/x509/#CreateCertificate)
    của gói [`crypto/tls`](/pkg/crypto/tls/)
    bây giờ hỗ trợ phân tích cú pháp (và ở nơi khác, tuần tự hóa) các yêu cầu chữ ký chứng chỉ PKCS #10.
  - Các hàm in có định dạng của gói `fmt` bây giờ định nghĩa `%F`
    như là từ đồng nghĩa cho `%f` khi in các giá trị dấu phẩy động.
  - Các kiểu [`Int`](/pkg/math/big/#Int) và
    [`Rat`](/pkg/math/big/#Rat) của gói
    [`math/big`](/pkg/math/big/)
    bây giờ triển khai
    [`encoding.TextMarshaler`](/pkg/encoding/#TextMarshaler) và
    [`encoding.TextUnmarshaler`](/pkg/encoding/#TextUnmarshaler).
  - Hàm lũy thừa số phức, [`Pow`](/pkg/math/cmplx/#Pow),
    bây giờ chỉ định hành vi khi đối số đầu tiên là không.
    Nó chưa được định nghĩa trước đây.
    Chi tiết có trong [tài liệu cho hàm](/pkg/math/cmplx/#Pow).
  - Gói [`net/http`](/pkg/net/http/) bây giờ phơi bày
    các thuộc tính của một kết nối TLS được dùng để thực hiện một yêu cầu client trong field mới
    [`Response.TLS`](/pkg/net/http/#Response).
  - Gói [`net/http`](/pkg/net/http/) bây giờ
    cho phép đặt một trình ghi nhật ký lỗi server tùy chọn
    với [`Server.ErrorLog`](/pkg/net/http/#Server).
    Mặc định vẫn là tất cả lỗi đi đến stderr.
  - Gói [`net/http`](/pkg/net/http/) bây giờ
    hỗ trợ vô hiệu hóa các kết nối HTTP keep-alive trên server
    với [`Server.SetKeepAlivesEnabled`](/pkg/net/http/#Server.SetKeepAlivesEnabled).
    Mặc định tiếp tục là server thực hiện keep-alive (tái sử dụng
    kết nối cho nhiều yêu cầu) theo mặc định.
    Chỉ các server bị hạn chế tài nguyên hoặc những server trong quá trình tắt graceful
    sẽ muốn vô hiệu hóa chúng.
  - Gói [`net/http`](/pkg/net/http/) thêm một cài đặt tùy chọn
    [`Transport.TLSHandshakeTimeout`](/pkg/net/http/#Transport)
    để giới hạn lượng thời gian các yêu cầu client HTTP sẽ chờ
    bắt tay TLS hoàn thành.
    Bây giờ nó cũng được đặt theo mặc định
    trên [`DefaultTransport`](/pkg/net/http#DefaultTransport).
  - [`DefaultTransport`](/pkg/net/http/#DefaultTransport)
    của gói [`net/http`](/pkg/net/http/),
    được dùng bởi mã client HTTP, bây giờ
    bật [TCP
    keep-alives](https://en.wikipedia.org/wiki/Keepalive#TCP_keepalive) theo mặc định.
    Các giá trị [`Transport`](/pkg/net/http/#Transport)
    khác với field `Dial` là nil tiếp tục hoạt động như trước:
    không có TCP keep-alive nào được dùng.
  - Gói [`net/http`](/pkg/net/http/)
    bây giờ bật [TCP
    keep-alives](https://en.wikipedia.org/wiki/Keepalive#TCP_keepalive) cho các yêu cầu server đến khi
    [`ListenAndServe`](/pkg/net/http/#ListenAndServe)
    hoặc
    [`ListenAndServeTLS`](/pkg/net/http/#ListenAndServeTLS)
    được dùng.
    Khi server được khởi động theo cách khác, TCP keep-alive không được bật.
  - Gói [`net/http`](/pkg/net/http/) bây giờ
    cung cấp một callback tùy chọn [`Server.ConnState`](/pkg/net/http/#Server)
    để hook vào các giai đoạn khác nhau của vòng đời kết nối server
    (xem [`ConnState`](/pkg/net/http/#ConnState)).
    Điều này có thể được dùng để triển khai rate limiting hoặc graceful shutdown.
  - Client HTTP của gói [`net/http`](/pkg/net/http/)
    bây giờ có field tùy chọn [`Client.Timeout`](/pkg/net/http/#Client)
    để chỉ định end-to-end timeout cho các yêu cầu được thực hiện bằng
    client.
  - Phương thức [`Request.ParseMultipartForm`](/pkg/net/http/#Request.ParseMultipartForm)
    của gói [`net/http`](/pkg/net/http/)
    bây giờ sẽ trả về lỗi nếu `Content-Type` của body
    không phải là `multipart/form-data`.
    Trước Go 1.3, nó sẽ âm thầm thất bại và trả về `nil`.
    Mã nguồn dựa vào hành vi trước đây nên được cập nhật.
  - Trong gói [`net`](/pkg/net/),
    cấu trúc [`Dialer`](/pkg/net/#Dialer) bây giờ
    có tùy chọn `KeepAlive` để chỉ định một khoảng keep-alive cho kết nối.
  - [`Transport`](/pkg/net/http/#Transport)
    của gói [`net/http`](/pkg/net/http/)
    bây giờ đóng [`Request.Body`](/pkg/net/http/#Request)
    nhất quán, ngay cả khi có lỗi.
  - Gói [`os/exec`](/pkg/os/exec/) bây giờ triển khai
    những gì tài liệu đã luôn nói về các đường dẫn tương đối cho binary.
    Cụ thể, nó chỉ gọi [`LookPath`](/pkg/os/exec/#LookPath)
    khi tên file binary không chứa dấu phân cách đường dẫn.
  - Hàm [`SetMapIndex`](/pkg/reflect/#Value.SetMapIndex)
    trong gói [`reflect`](/pkg/reflect/)
    không còn panic khi xóa từ một map `nil`.
  - Nếu goroutine chính gọi
    [`runtime.Goexit`](/pkg/runtime/#Goexit)
    và tất cả các goroutine khác kết thúc thực thi, chương trình bây giờ luôn crash,
    báo cáo một deadlock được phát hiện.
    Các phiên bản trước của Go xử lý tình huống này không nhất quán: hầu hết các trường hợp
    được báo cáo là deadlock, nhưng một số trường hợp tầm thường thoát sạch.
  - Gói runtime/debug bây giờ có hàm mới
    [`debug.WriteHeapDump`](/pkg/runtime/debug/#WriteHeapDump)
    ghi ra mô tả về heap.
  - Hàm [`CanBackquote`](/pkg/strconv/#CanBackquote)
    trong gói [`strconv`](/pkg/strconv/)
    bây giờ coi ký tự `DEL`, `U+007F`, là
    không in được.
  - Gói [`syscall`](/pkg/syscall/) bây giờ cung cấp
    [`SendmsgN`](/pkg/syscall/#SendmsgN)
    như một phiên bản thay thế của
    [`Sendmsg`](/pkg/syscall/#Sendmsg)
    trả về số byte đã ghi.
  - Trên Windows, gói [`syscall`](/pkg/syscall/) bây giờ
    hỗ trợ quy ước gọi cdecl thông qua việc thêm hàm mới
    [`NewCallbackCDecl`](/pkg/syscall/#NewCallbackCDecl)
    bên cạnh hàm hiện có
    [`NewCallback`](/pkg/syscall/#NewCallback).
  - Gói [`testing`](/pkg/testing/) bây giờ
    chẩn đoán các test gọi `panic(nil)`, gần như luôn là sai.
    Ngoài ra, các test bây giờ ghi profile (nếu được gọi với các flag profiling) ngay cả khi thất bại.
  - Gói [`unicode`](/pkg/unicode/) và hỗ trợ liên quan
    trong toàn bộ hệ thống đã được nâng cấp từ
    Unicode 6.2.0 lên [Unicode 6.3.0](https://www.unicode.org/versions/Unicode6.3.0/).
