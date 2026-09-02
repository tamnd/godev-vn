---
title: Ghi chú phát hành Go 1.5
---

## Giới thiệu về Go 1.5 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.5,
là một bản phát hành quan trọng, bao gồm các thay đổi kiến trúc lớn đối với phần hiện thực.
Mặc dù vậy, chúng tôi kỳ vọng hầu hết tất cả các chương trình Go vẫn sẽ tiếp tục biên dịch và chạy như trước,
vì bản phát hành vẫn duy trì [cam kết tương thích](/doc/go1compat.html) Go 1.

Những phát triển lớn nhất trong phần hiện thực là:

  - Trình biên dịch và runtime hiện được viết hoàn toàn bằng Go (với một chút assembler).
    C không còn liên quan đến phần hiện thực nữa, vì vậy trình biên dịch C
    từng cần thiết để xây dựng bản phân phối đã không còn nữa.
  - Bộ gom rác hiện là [đồng thời](/s/go14gc) và cung cấp thời gian dừng
    thấp hơn đáng kể bằng cách chạy, khi có thể, song song với các goroutine khác.
  - Theo mặc định, các chương trình Go chạy với `GOMAXPROCS` được đặt thành
    số lõi khả dụng; trong các bản phát hành trước, mặc định là 1.
  - Hỗ trợ [gói nội bộ](/s/go14internal)
    hiện được cung cấp cho tất cả các kho lưu trữ, không chỉ lõi Go.
  - Lệnh `go` hiện cung cấp [hỗ trợ thử nghiệm](/s/go15vendor)
    cho "vendoring" các dependency bên ngoài.
  - Một lệnh `go tool trace` mới hỗ trợ tracing chi tiết
    quá trình thực thi chương trình.
  - Một lệnh `go doc` mới (khác với `godoc`)
    được tùy chỉnh cho việc sử dụng dòng lệnh.

Những thay đổi này và một số thay đổi khác đối với phần hiện thực và công cụ
được thảo luận bên dưới.

Bản phát hành cũng chứa một thay đổi ngôn ngữ nhỏ liên quan đến map literal.

Cuối cùng, thời điểm của [bản phát hành](/s/releasesched)
lệch khỏi khoảng thời gian sáu tháng thông thường,
cả để cung cấp thêm thời gian chuẩn bị cho bản phát hành lớn này và để điều chỉnh lịch trình sau đó
cho phù hợp hơn với thời điểm phát hành.

## Thay đổi về ngôn ngữ {#language}

### Map literal {#map_literals}

Do một sơ suất, quy tắc cho phép bỏ qua kiểu phần tử từ slice literal đã không
được áp dụng cho các khóa map.
Điều này đã được [sửa](/cl/2591) trong Go 1.5.
Một ví dụ sẽ làm rõ điều này.
Kể từ Go 1.5, map literal này,

	m := map[Point]string{
	    Point{29.935523, 52.891566}:   "Persepolis",
	    Point{-25.352594, 131.034361}: "Uluru",
	    Point{37.422455, -122.084306}: "Googleplex",
	}

có thể được viết như sau, mà không cần liệt kê kiểu `Point` một cách tường minh:

	m := map[Point]string{
	    {29.935523, 52.891566}:   "Persepolis",
	    {-25.352594, 131.034361}: "Uluru",
	    {37.422455, -122.084306}: "Googleplex",
	}

## Phần hiện thực {#implementation}

### Không còn C {#c}

Trình biên dịch và runtime hiện được hiện thực bằng Go và assembler, không dùng C.
Mã nguồn C còn lại trong cây chỉ liên quan đến việc kiểm thử hoặc `cgo`.
Có một trình biên dịch C trong cây ở phiên bản 1.4 và trước đó.
Nó được dùng để xây dựng runtime; một trình biên dịch tùy chỉnh là cần thiết một phần để
đảm bảo mã C hoạt động với việc quản lý stack của goroutine.
Vì runtime hiện đã ở dạng Go, không cần trình biên dịch C này nữa và nó đã được loại bỏ.
Chi tiết về quá trình loại bỏ C được thảo luận [ở nơi khác](/s/go13compiler).

Việc chuyển đổi từ C được thực hiện với sự trợ giúp của các công cụ tùy chỉnh được tạo ra cho công việc này.
Quan trọng nhất, trình biên dịch thực sự được chuyển đổi bằng cách dịch tự động
mã C sang Go.
Về thực chất, đây là cùng một chương trình trong một ngôn ngữ khác.
Đây không phải là một hiện thực mới của trình biên dịch vì vậy chúng tôi kỳ vọng quá trình này
sẽ không đưa ra các lỗi trình biên dịch mới.
Tổng quan về quá trình này có sẵn trong các slide của
[bài trình bày này](/talks/2015/gogo.slide).

### Trình biên dịch và công cụ {#compiler_and_tools}

Độc lập nhưng được khuyến khích bởi việc chuyển sang Go, tên của các công cụ đã thay đổi.
Các tên cũ `6g`, `8g` và các tên khác đã biến mất; thay vào đó
chỉ có một binary, có thể truy cập dưới dạng `go` `tool` `compile`,
biên dịch mã nguồn Go thành các binary phù hợp với kiến trúc và hệ điều hành
được chỉ định bởi `$GOARCH` và `$GOOS`.
Tương tự, bây giờ có một linker (`go` `tool` `link`)
và một assembler (`go` `tool` `asm`).
Linker được dịch tự động từ hiện thực C cũ,
nhưng assembler là một hiện thực Go gốc mới được thảo luận
chi tiết hơn bên dưới.

Tương tự như việc bỏ các tên `6g`, `8g`, v.v.,
đầu ra của trình biên dịch và assembler bây giờ được cấp hậu tố `.o` đơn giản
thay vì `.8`, `.6`, v.v.

### Bộ gom rác {#gc}

Bộ gom rác đã được thiết kế lại cho phiên bản 1.5 như một phần của quá trình phát triển
được phác thảo trong [tài liệu thiết kế](/s/go14gc).
Độ trễ dự kiến thấp hơn nhiều so với bộ gom rác trong các bản phát hành trước,
thông qua sự kết hợp của các thuật toán tiên tiến,
[lập lịch](/s/go15gcpacing) tốt hơn cho bộ gom rác,
và chạy nhiều quá trình thu gom song song hơn với chương trình của người dùng.
Giai đoạn "dừng toàn thế giới" (stop the world) của bộ gom rác
sẽ hầu như luôn dưới 10 mili giây và thường ít hơn nhiều.

Đối với các hệ thống được hưởng lợi từ độ trễ thấp, chẳng hạn như các trang web phản hồi người dùng,
sự giảm độ trễ dự kiến với bộ gom rác mới có thể quan trọng.

Chi tiết về bộ gom rác mới được trình bày trong một
[bài nói chuyện](/talks/2015/go-gc.pdf) tại GopherCon 2015.

### Runtime {#runtime}

Trong Go 1.5, thứ tự lập lịch goroutine đã được thay đổi.
Các thuộc tính của bộ lập lịch chưa bao giờ được xác định bởi ngôn ngữ,
nhưng các chương trình phụ thuộc vào thứ tự lập lịch có thể bị hỏng
bởi sự thay đổi này.
Chúng tôi đã thấy một vài chương trình (có lỗi) bị ảnh hưởng bởi sự thay đổi này.
Nếu bạn có các chương trình phụ thuộc ngầm vào thứ tự lập lịch,
bạn sẽ cần cập nhật chúng.

Một thay đổi có thể gây ra lỗi khác là runtime hiện
đặt số lượng luồng mặc định để chạy đồng thời,
được xác định bởi `GOMAXPROCS`, thành số
lõi khả dụng trên CPU.
Trong các bản phát hành trước, mặc định là 1.
Các chương trình không mong đợi chạy với nhiều lõi có thể
bị hỏng ngoài ý muốn.
Chúng có thể được cập nhật bằng cách loại bỏ hạn chế hoặc bằng cách đặt
`GOMAXPROCS` một cách tường minh.
Để thảo luận chi tiết hơn về sự thay đổi này, xem
[tài liệu thiết kế](/s/go15gomaxprocs).

### Xây dựng {#build}

Bây giờ khi trình biên dịch Go và runtime được hiện thực bằng Go, một trình biên dịch Go
phải có sẵn để biên dịch bản phân phối từ mã nguồn.
Do đó, để xây dựng lõi Go, một bản phân phối Go đang hoạt động phải đã có sẵn.
(Những lập trình viên Go không làm việc trên lõi không bị ảnh hưởng bởi sự thay đổi này.)
Bất kỳ bản phân phối Go 1.4 hoặc mới hơn (bao gồm `gccgo`) đều phù hợp.
Để biết chi tiết, xem [tài liệu thiết kế](/s/go15bootstrap).

## Các nền tảng {#ports}

Chủ yếu là do sự chuyển dịch của ngành khỏi kiến trúc x86 32-bit,
bộ bản tải nhị phân được cung cấp đã giảm trong phiên bản 1.5.
Bản phân phối cho hệ điều hành OS X chỉ được cung cấp cho kiến trúc
`amd64`, không phải `386`.
Tương tự, các cổng cho Snow Leopard (Apple OS X 10.6) vẫn hoạt động nhưng không còn
được phát hành dưới dạng bản tải hoặc duy trì vì Apple không còn duy trì phiên bản
đó của hệ điều hành nữa.
Ngoài ra, cổng `dragonfly/386` không còn được hỗ trợ nữa
vì chính DragonflyBSD không còn hỗ trợ kiến trúc 386 32-bit.

Tuy nhiên, có một số cổng mới có thể được xây dựng từ mã nguồn.
Chúng bao gồm `darwin/arm` và `darwin/arm64`.
Cổng mới `linux/arm64` hầu hết đã hoàn chỉnh, nhưng `cgo`
chỉ được hỗ trợ sử dụng liên kết bên ngoài (external linking).

Cũng có thể dùng thử nghiệm là `ppc64`
và `ppc64le` (PowerPC 64-bit, big-endian và little-endian).
Cả hai cổng này đều hỗ trợ `cgo` nhưng
chỉ với liên kết nội bộ (internal linking).

Trên FreeBSD, Go 1.5 yêu cầu FreeBSD 8-STABLE+ vì sử dụng lệnh `SYSCALL` mới.

Trên NaCl, Go 1.5 yêu cầu phiên bản SDK pepper-41. Các phiên bản pepper mới hơn không
tương thích do việc xóa subsystem sRPC khỏi NaCl runtime.

Trên Darwin, việc sử dụng giao diện chứng chỉ X.509 của hệ thống có thể bị tắt
với build tag `ios`.

Cổng Solaris hiện có hỗ trợ đầy đủ cho cgo và các gói
[`net`](/pkg/net/) và
[`crypto/x509`](/pkg/crypto/x509/),
cũng như một số sửa lỗi và cải tiến khác.

## Công cụ {#tools}

### Dịch {#translate}

Như một phần của quá trình loại bỏ C khỏi cây, trình biên dịch và
linker đã được dịch từ C sang Go.
Đó là một quá trình dịch thực sự (có hỗ trợ máy móc), vì vậy các chương trình mới về cơ bản
là các chương trình cũ được dịch chứ không phải các chương trình mới với các lỗi mới.
Chúng tôi tự tin rằng quá trình dịch đã đưa ra ít nếu có bất kỳ lỗi mới nào,
và thực tế đã phát hiện ra một số lỗi trước đây chưa biết, hiện đã được sửa.

Tuy nhiên, assembler là một chương trình mới; nó được mô tả bên dưới.

### Đổi tên {#rename}

Các bộ chương trình là các trình biên dịch (`6g`, `8g`, v.v.),
các assembler (`6a`, `8a`, v.v.),
và các linker (`6l`, `8l`, v.v.)
mỗi loại đã được hợp nhất thành một công cụ duy nhất được cấu hình
bởi các biến môi trường `GOOS` và `GOARCH`.
Các tên cũ đã biến mất; các công cụ mới có thể truy cập thông qua cơ chế `go` `tool`
dưới dạng `go tool compile`,
`go tool asm`,
và `go tool link`.
Ngoài ra, các hậu tố tệp `.6`, `.8`, v.v. cho các
tệp đối tượng trung gian cũng đã biến mất; bây giờ chúng chỉ là các tệp `.o` thông thường.

Ví dụ, để xây dựng và liên kết một chương trình trên amd64 cho Darwin
sử dụng các công cụ trực tiếp, thay vì thông qua `go build`,
người ta sẽ chạy:

	$ export GOOS=darwin GOARCH=amd64
	$ go tool compile program.go
	$ go tool link program.o

### Di chuyển {#moving}

Vì gói [`go/types`](/pkg/go/types/)
đã được chuyển vào kho lưu trữ chính (xem bên dưới),
các công cụ [`vet`](/cmd/vet) và
[`cover`](/cmd/cover)
cũng đã được chuyển.
Chúng không còn được duy trì trong kho lưu trữ `golang.org/x/tools` bên ngoài nữa,
mặc dù mã nguồn (đã deprecated) vẫn còn đó để tương thích với các bản phát hành cũ.

### Trình biên dịch {#compiler}

Như đã mô tả ở trên, trình biên dịch trong Go 1.5 là một chương trình Go duy nhất,
được dịch từ mã nguồn C cũ, thay thế `6g`, `8g`,
v.v.
Mục tiêu của nó được cấu hình bởi các biến môi trường `GOOS` và `GOARCH`.

Trình biên dịch 1.5 hầu như tương đương với trình biên dịch cũ,
nhưng một số chi tiết nội bộ đã thay đổi.
Một thay đổi quan trọng là việc đánh giá hằng số hiện sử dụng
gói [`math/big`](/pkg/math/big/)
thay vì một hiện thực tùy chỉnh (và ít được kiểm thử hơn) về số học độ chính xác cao.
Chúng tôi không mong đợi điều này ảnh hưởng đến kết quả.

Chỉ đối với kiến trúc amd64, trình biên dịch có một tùy chọn mới, `-dynlink`,
hỗ trợ liên kết động bằng cách hỗ trợ tham chiếu đến các ký hiệu Go
được xác định trong các thư viện dùng chung bên ngoài.

### Assembler {#assembler}

Giống như trình biên dịch và linker, assembler trong Go 1.5 là một chương trình duy nhất
thay thế bộ assembler (`6a`,
`8a`, v.v.) và các biến môi trường
`GOARCH` và `GOOS`
cấu hình kiến trúc và hệ điều hành.
Không giống các chương trình khác, assembler là một chương trình Go gốc hoàn toàn mới
được viết bằng Go.

Assembler mới gần như tương thích với các assembler trước đó,
nhưng có một vài thay đổi có thể ảnh hưởng đến một số
tệp mã nguồn assembler.
Xem [hướng dẫn assembler](/doc/asm) đã được cập nhật
để biết thêm thông tin cụ thể về những thay đổi này. Tóm lại:

Đầu tiên, việc đánh giá biểu thức được sử dụng cho hằng số có chút khác biệt.
Bây giờ nó sử dụng số học 64-bit không dấu và thứ tự ưu tiên
của các toán tử (`+`, `-`, `<<`, v.v.)
đến từ Go, không phải C.
Chúng tôi kỳ vọng những thay đổi này ảnh hưởng đến rất ít chương trình nhưng
có thể cần xác minh thủ công.

Có lẽ quan trọng hơn là trên các máy mà
`SP` hoặc `PC` chỉ là bí danh
cho một thanh ghi được đánh số,
chẳng hạn như `R13` cho con trỏ stack và
`R15` cho bộ đếm chương trình phần cứng
trên ARM,
một tham chiếu đến thanh ghi như vậy không bao gồm một ký hiệu
hiện là bất hợp lệ.
Ví dụ, `SP` và `4(SP)` là
bất hợp lệ nhưng `sym+4(SP)` thì ổn.
Trên các máy như vậy, để tham chiếu đến thanh ghi phần cứng sử dụng
tên `R` thực sự của nó.

Một thay đổi nhỏ là một số assembler cũ
cho phép ký hiệu

	constant=value

để xác định một hằng số có tên.
Vì điều này luôn có thể thực hiện với ký hiệu
`#define` kiểu C truyền thống, vẫn được
hỗ trợ (assembler bao gồm một hiện thực của bộ tiền xử lý C đơn giản hóa),
tính năng này đã bị loại bỏ.

### Linker {#link}

Linker trong Go 1.5 hiện là một chương trình Go,
thay thế `6l`, `8l`, v.v.
Hệ điều hành và tập lệnh của nó được chỉ định
bởi các biến môi trường `GOOS` và `GOARCH`.

Có một số thay đổi khác.
Quan trọng nhất là việc bổ sung tùy chọn `-buildmode`
mở rộng kiểu liên kết; bây giờ nó hỗ trợ
các tình huống như xây dựng thư viện dùng chung và cho phép các ngôn ngữ khác
gọi vào các thư viện Go.
Một số trong số này được phác thảo trong [tài liệu thiết kế](/s/execmodes).
Để biết danh sách các build mode có sẵn và cách sử dụng của chúng, chạy

	$ go help buildmode

Một thay đổi nhỏ khác là linker không còn ghi nhớ dấu thời gian xây dựng trong
tiêu đề của các file thực thi Windows.
Ngoài ra, mặc dù điều này có thể được sửa, các file thực thi Windows cgo thiếu một số
thông tin DWARF.

Cuối cùng, cờ `-X`, nhận hai đối số,
như trong

	-X importpath.name value

hiện cũng chấp nhận kiểu cờ Go phổ biến hơn với một đối số duy nhất
là một cặp `name=value`:

	-X importpath.name=value

Mặc dù cú pháp cũ vẫn hoạt động, nên cập nhật các cách dùng cờ này
trong các script và các nơi khác sang dạng mới.

### Lệnh go {#go_command}

Thao tác cơ bản của [lệnh `go`](/cmd/go) là
không thay đổi, nhưng có một số thay đổi đáng chú ý.

Bản phát hành trước đã giới thiệu ý tưởng về một thư mục nội bộ của gói
không thể nhập qua lệnh `go`.
Trong phiên bản 1.4, nó được kiểm thử với việc giới thiệu một số phần tử nội bộ
trong kho lưu trữ lõi.
Như được đề xuất trong [tài liệu thiết kế](/s/go14internal),
thay đổi đó hiện đang được cung cấp cho tất cả các kho lưu trữ.
Các quy tắc được giải thích trong tài liệu thiết kế, nhưng tóm lại bất kỳ
gói nào trong hoặc dưới một thư mục có tên `internal` có thể
được nhập bởi các gói có gốc trong cùng một cây con.
Các gói hiện có với các phần tử thư mục có tên `internal` có thể bị
vô tình hỏng bởi sự thay đổi này, đó là lý do tại sao nó đã được thông báo
trong bản phát hành trước.

Một thay đổi khác trong cách xử lý gói là việc bổ sung thử nghiệm
hỗ trợ "vendoring".
Để biết chi tiết, xem tài liệu cho [lệnh `go`](/cmd/go/#hdr-Vendor_Directories)
và [tài liệu thiết kế](/s/go15vendor).

Cũng có một số thay đổi nhỏ khác.
Đọc [tài liệu](/cmd/go) để biết chi tiết đầy đủ.

  - Hỗ trợ SWIG đã được cập nhật sao cho
    `.swig` và `.swigcxx`
    hiện yêu cầu SWIG 3.0.6 hoặc mới hơn.
  - Lệnh con `install` hiện loại bỏ
    binary được tạo bởi lệnh con `build`
    trong thư mục nguồn, nếu có,
    để tránh vấn đề khi có hai binary trong cây.
  - Tên gói wildcard `std` (thư viện chuẩn)
    hiện loại trừ các lệnh.
    Một wildcard `cmd` mới bao gồm các lệnh.
  - Tùy chọn build `-asmflags` mới
    đặt các cờ để chuyển cho assembler.
    Tuy nhiên,
    tùy chọn build `-ccflags` đã bị loại bỏ;
    nó dành riêng cho trình biên dịch C cũ, hiện đã bị xóa.
  - Tùy chọn build `-buildmode` mới
    đặt build mode, được mô tả ở trên.
  - Tùy chọn build `-pkgdir` mới
    đặt vị trí của các archive gói đã cài đặt,
    để giúp cô lập các bản xây dựng tùy chỉnh.
  - Tùy chọn build `-toolexec` mới
    cho phép thay thế một lệnh khác để gọi
    trình biên dịch v.v.
    Điều này hoạt động như một sự thay thế tùy chỉnh cho `go tool`.
  - Lệnh con `test` hiện có cờ `-count`
    để chỉ định bao nhiêu lần chạy mỗi bài kiểm thử và benchmark.
    Gói [`testing`](/pkg/testing/)
    thực hiện công việc này, thông qua cờ `-test.count`.
  - Lệnh con `generate` có một vài tính năng mới.
    Tùy chọn `-run` chỉ định một biểu thức chính quy để chọn các chỉ thị nào
    cần thực thi; điều này đã được đề xuất nhưng chưa được hiện thực trong 1.4.
    Pattern thực thi bây giờ có quyền truy cập vào hai biến môi trường mới:
    `$GOLINE` trả về số dòng nguồn của chỉ thị
    và `$DOLLAR` mở rộng thành ký hiệu đô la.
  - Lệnh con `get` hiện có cờ `-insecure`
    phải được bật nếu tải từ một kho lưu trữ không an toàn, tức là
    không mã hóa kết nối.

### Lệnh go vet {#vet_command}

Lệnh [`go tool vet`](/cmd/vet) hiện thực hiện
xác thực kỹ lưỡng hơn đối với các struct tag.

### Lệnh trace {#trace_command}

Có một công cụ mới để tracing động quá trình thực thi của các chương trình Go.
Cách sử dụng tương tự như cách công cụ test coverage hoạt động.
Việc tạo trace được tích hợp vào `go test`,
và sau đó một lần thực thi riêng biệt của chính công cụ tracing phân tích kết quả:

	$ go test -trace=trace.out path/to/package
	$ go tool trace [flags] pkg.test trace.out

Các cờ cho phép đầu ra được hiển thị trong cửa sổ trình duyệt.
Để biết chi tiết, chạy `go tool trace -help`.
Cũng có mô tả về cơ sở tracing trong
[bài nói chuyện này](/talks/2015/dynamic-tools.slide)
từ GopherCon 2015.

### Lệnh go doc {#doc_command}

Một vài bản phát hành trước, lệnh `go doc`
đã bị xóa vì không cần thiết.
Người ta luôn có thể chạy "`godoc .`" thay thế.
Bản phát hành 1.5 giới thiệu lệnh [`go doc`](/cmd/doc) mới
với giao diện dòng lệnh thuận tiện hơn
so với `godoc`.
Nó được thiết kế đặc biệt cho việc sử dụng dòng lệnh, và cung cấp bản trình bày
gọn gàng và tập trung hơn về tài liệu của một gói
hoặc các phần tử của nó, tùy theo lệnh gọi.
Nó cũng cung cấp khớp không phân biệt chữ hoa chữ thường và
hỗ trợ hiển thị tài liệu cho các ký hiệu không xuất.
Để biết chi tiết chạy "`go help doc`".

### Cgo {#cgo}

Khi phân tích các dòng `#cgo`,
lệnh gọi `${SRCDIR}` hiện được
mở rộng thành đường dẫn đến thư mục nguồn.
Điều này cho phép các tùy chọn được chuyển đến
trình biên dịch và linker liên quan đến các đường dẫn tệp tương đối với
thư mục mã nguồn. Nếu không có sự mở rộng này, các đường dẫn sẽ
không hợp lệ khi thư mục làm việc hiện tại thay đổi.

Solaris hiện có hỗ trợ cgo đầy đủ.

Trên Windows, cgo hiện sử dụng liên kết bên ngoài theo mặc định.

Khi một struct C kết thúc với một trường có kích thước bằng không, nhưng bản thân struct
không có kích thước bằng không, mã Go không còn có thể tham chiếu đến trường có kích thước bằng không đó.
Bất kỳ tham chiếu nào như vậy sẽ phải được viết lại.

## Hiệu suất {#performance}

Như thường lệ, các thay đổi quá đa dạng và phổ biến khiến việc đưa ra các nhận định chính xác
về hiệu suất là khó khăn.
Các thay đổi thậm chí còn rộng hơn so với thông thường trong bản phát hành này,
bao gồm bộ gom rác mới và việc chuyển đổi runtime sang Go.
Một số chương trình có thể chạy nhanh hơn, một số chậm hơn.
Trung bình các chương trình trong bộ benchmark Go 1 chạy nhanh hơn vài phần trăm trong Go 1.5
so với Go 1.4,
trong khi như đã đề cập ở trên các lần dừng của bộ gom rác
ngắn hơn đáng kể, và hầu như luôn dưới 10 mili giây.

Việc xây dựng trong Go 1.5 sẽ chậm hơn khoảng hai lần.
Việc dịch tự động trình biên dịch và linker từ C sang Go dẫn đến
mã Go không thành ngữ thực hiện kém hơn so với mã Go được viết tốt.
Các công cụ phân tích và tái cấu trúc đã giúp cải thiện mã, nhưng vẫn còn nhiều việc phải làm.
Việc lập profile và tối ưu hóa tiếp theo sẽ tiếp tục trong Go 1.6 và các bản phát hành tương lai.
Để biết thêm chi tiết, xem các [slide này](/talks/2015/gogo.slide)
và [video](/talks/2015/gogo.slide) liên quan tại
[đây](https://www.youtube.com/watch?v=cF1zJYkBW4A).

## Thư viện chuẩn {#library}

### Flag {#flag}

Hàm [`PrintDefaults`](/pkg/flag/#PrintDefaults)
của gói flag, và phương thức trên [`FlagSet`](/pkg/flag/#FlagSet),
đã được sửa đổi để tạo ra các thông báo sử dụng đẹp hơn.
Định dạng đã được thay đổi để thân thiện hơn với con người và trong các
thông báo sử dụng, một từ được trích dẫn bằng \`backtick\` được coi là tên của
toán hạng của cờ để hiển thị trong thông báo sử dụng.
Ví dụ, một cờ được tạo bằng lệnh gọi,

	cpuFlag = flag.Int("cpu", 1, "run `N` processes in parallel")

sẽ hiển thị thông báo trợ giúp,

	-cpu N
	    	run N processes in parallel (default 1)

Ngoài ra, giá trị mặc định hiện chỉ được liệt kê khi nó không phải là giá trị zero của kiểu.

### Số thực trong math/big {#math_big}

Gói [`math/big`](/pkg/math/big/)
có một kiểu dữ liệu cơ bản mới,
[`Float`](/pkg/math/big/#Float),
hiện thực các số dấu phẩy động có độ chính xác tùy ý.
Một giá trị `Float` được biểu diễn bởi một dấu boolean,
một mantissa có độ dài thay đổi, và một số mũ có dấu kích thước cố định 32-bit.
Độ chính xác của một `Float` (kích thước mantissa tính bằng bit)
có thể được chỉ định rõ ràng hoặc được xác định bởi thao tác đầu tiên tạo ra giá trị.
Sau khi được tạo, kích thước mantissa của `Float` có thể được sửa đổi bằng
phương thức [`SetPrec`](/pkg/math/big/#Float.SetPrec).
`Float` hỗ trợ khái niệm về vô cực, chẳng hạn như được tạo bởi
tràn số, nhưng các giá trị dẫn đến tương đương với IEEE 754 NaN
kích hoạt panic.
Các thao tác `Float` hỗ trợ tất cả các chế độ làm tròn IEEE-754.
Khi độ chính xác được đặt thành 24 (53) bit,
các thao tác nằm trong phạm vi giá trị `float32` (`float64`) chuẩn hóa
tạo ra kết quả giống với phép tính IEEE-754 tương ứng
trên các giá trị đó.

### Kiểu Go {#go_types}

Gói [`go/types`](/pkg/go/types/)
cho đến nay đã được duy trì trong kho lưu trữ `golang.org/x`;
kể từ Go 1.5, nó đã được chuyển đến kho lưu trữ chính.
Mã tại vị trí cũ hiện đã bị deprecated.
Cũng có một thay đổi API khiêm tốn trong gói, được thảo luận bên dưới.

Liên quan đến sự di chuyển này, gói
[`go/constant`](/pkg/go/constant/)
cũng đã chuyển đến kho lưu trữ chính;
trước đó là `golang.org/x/tools/exact`.
Gói [`go/importer`](/pkg/go/importer/)
cũng đã chuyển đến kho lưu trữ chính,
cũng như một số công cụ được mô tả ở trên.

### Net {#net}

Trình giải quyết DNS trong gói net hầu như luôn sử dụng `cgo` để truy cập
giao diện hệ thống.
Một thay đổi trong Go 1.5 có nghĩa là trên hầu hết các hệ thống Unix, việc giải quyết DNS
sẽ không còn yêu cầu `cgo`, điều này đơn giản hóa việc thực thi
trên các nền tảng đó.
Bây giờ, nếu cấu hình mạng của hệ thống cho phép, trình giải quyết Go gốc
sẽ đủ.
Hiệu quả quan trọng của thay đổi này là mỗi lần giải quyết DNS chiếm một goroutine
thay vì một luồng,
vì vậy một chương trình với nhiều yêu cầu DNS chưa hoàn thành sẽ tiêu tốn ít tài nguyên hệ điều hành hơn.

Quyết định cách chạy trình giải quyết được áp dụng tại thời điểm chạy, không phải thời điểm xây dựng.
Build tag `netgo` đã được sử dụng để thực thi việc sử dụng
trình giải quyết Go không còn cần thiết nữa, mặc dù nó vẫn hoạt động.
Build tag `netcgo` mới buộc sử dụng trình giải quyết `cgo` tại
thời điểm xây dựng.
Để buộc giải quyết `cgo` tại thời điểm chạy, đặt
`GODEBUG=netdns=cgo` trong môi trường.
Các tùy chọn debug khác được ghi lại [ở đây](/cl/11584).

Thay đổi này chỉ áp dụng cho các hệ thống Unix.
Windows, Mac OS X và các hệ thống Plan 9 hoạt động như trước.

### Reflect {#reflect}

Gói [`reflect`](/pkg/reflect/)
có hai hàm mới: [`ArrayOf`](/pkg/reflect/#ArrayOf)
và [`FuncOf`](/pkg/reflect/#FuncOf).
Các hàm này, tương tự như hàm
[`SliceOf`](/pkg/reflect/#SliceOf) hiện có,
tạo ra các kiểu mới tại thời điểm chạy để mô tả mảng và hàm.

### Tăng cường độ bền {#hardening}

Hàng chục lỗi đã được tìm thấy trong thư viện chuẩn
thông qua kiểm thử ngẫu nhiên với công cụ
[`go-fuzz`](https://github.com/dvyukov/go-fuzz).
Các lỗi đã được sửa trong các gói
[`archive/tar`](/pkg/archive/tar/),
[`archive/zip`](/pkg/archive/zip/),
[`compress/flate`](/pkg/compress/flate/),
[`encoding/gob`](/pkg/encoding/gob/),
[`fmt`](/pkg/fmt/),
[`html/template`](/pkg/html/template/),
[`image/gif`](/pkg/image/gif/),
[`image/jpeg`](/pkg/image/jpeg/),
[`image/png`](/pkg/image/png/), và
[`text/template`](/pkg/text/template/).
Các sửa lỗi này tăng cường hiện thực chống lại các đầu vào không chính xác và độc hại.

### Thay đổi nhỏ đối với thư viện {#minor_library_changes}

  - Kiểu [`Writer`](/pkg/archive/zip/#Writer) của gói
    [`archive/zip`](/pkg/archive/zip/) hiện có một
    phương thức [`SetOffset`](/pkg/archive/zip/#Writer.SetOffset)
    để chỉ định vị trí trong luồng đầu ra để ghi archive.
  - [`Reader`](/pkg/bufio/#Reader) trong gói
    [`bufio`](/pkg/bufio/) hiện có
    phương thức [`Discard`](/pkg/bufio/#Reader.Discard)
    để bỏ qua dữ liệu từ đầu vào.
  - Trong gói [`bytes`](/pkg/bytes/),
    kiểu [`Buffer`](/pkg/bytes/#Buffer)
    hiện có phương thức [`Cap`](/pkg/bytes/#Buffer.Cap)
    báo cáo số byte được phân bổ trong bộ đệm.
    Tương tự, trong cả gói [`bytes`](/pkg/bytes/)
    và [`strings`](/pkg/strings/),
    kiểu [`Reader`](/pkg/bytes/#Reader)
    hiện có phương thức [`Size`](/pkg/bytes/#Reader.Size)
    báo cáo độ dài ban đầu của slice hoặc chuỗi cơ bản.
  - Cả gói [`bytes`](/pkg/bytes/) và
    [`strings`](/pkg/strings/)
    cũng hiện có hàm [`LastIndexByte`](/pkg/bytes/#LastIndexByte)
    xác định vị trí byte ngoài cùng bên phải có giá trị đó trong đối số.
  - Gói [`crypto`](/pkg/crypto/)
    có một giao diện mới, [`Decrypter`](/pkg/crypto/#Decrypter),
    trừu tượng hóa hành vi của khóa riêng được sử dụng trong giải mã không đối xứng.
  - Trong gói [`crypto/cipher`](/pkg/crypto/cipher/),
    tài liệu cho giao diện [`Stream`](/pkg/crypto/cipher/#Stream)
    đã được làm rõ về hành vi khi nguồn và đích có
    độ dài khác nhau.
    Nếu đích ngắn hơn nguồn, phương thức sẽ panic.
    Đây không phải là thay đổi trong hiện thực, chỉ là tài liệu.
  - Cũng trong gói [`crypto/cipher`](/pkg/crypto/cipher/),
    bây giờ có hỗ trợ độ dài nonce khác ngoài 96 byte trong chế độ Galois/Counter (GCM) của AES,
    mà một số giao thức yêu cầu.
  - Trong gói [`crypto/elliptic`](/pkg/crypto/elliptic/),
    hiện có trường `Name` trong struct
    [`CurveParams`](/pkg/crypto/elliptic/#CurveParams),
    và các đường cong được hiện thực trong gói đã được đặt tên.
    Những tên này cung cấp một cách an toàn hơn để chọn một đường cong, trái với
    việc chọn kích thước bit của nó, cho các hệ thống mật mã phụ thuộc vào đường cong.
  - Cũng trong gói [`crypto/elliptic`](/pkg/crypto/elliptic/),
    hàm [`Unmarshal`](/pkg/crypto/elliptic/#Unmarshal)
    hiện xác minh rằng điểm thực sự nằm trên đường cong.
    (Nếu không, hàm trả về nil).
    Thay đổi này bảo vệ chống lại một số cuộc tấn công.
  - Gói [`crypto/sha512`](/pkg/crypto/sha512/)
    hiện có hỗ trợ cho hai phiên bản rút gọn của
    thuật toán băm SHA-512, SHA-512/224 và SHA-512/256.
  - Phiên bản giao thức tối thiểu của gói [`crypto/tls`](/pkg/crypto/tls/)
    hiện mặc định là TLS 1.0.
    Mặc định cũ, SSLv3, vẫn có sẵn thông qua [`Config`](/pkg/crypto/tls/#Config) nếu cần.
  - Gói [`crypto/tls`](/pkg/crypto/tls/)
    hiện hỗ trợ Signed Certificate Timestamps (SCT) như được chỉ định trong RFC 6962.
    Máy chủ phục vụ chúng nếu chúng được liệt kê trong struct
    [`Certificate`](/pkg/crypto/tls/#Certificate),
    và máy khách yêu cầu chúng và hiển thị chúng, nếu có,
    trong struct [`ConnectionState`](/pkg/crypto/tls/#ConnectionState) của nó.
  - Phản hồi OCSP đính kèm với kết nối máy khách [`crypto/tls`](/pkg/crypto/tls/),
    trước đây chỉ có sẵn qua
    phương thức [`OCSPResponse`](/pkg/crypto/tls/#Conn.OCSPResponse),
    hiện được hiển thị trong struct [`ConnectionState`](/pkg/crypto/tls/#ConnectionState).
  - Hiện thực máy chủ [`crypto/tls`](/pkg/crypto/tls/)
    hiện sẽ luôn gọi
    hàm `GetCertificate` trong
    struct [`Config`](/pkg/crypto/tls/#Config)
    để chọn chứng chỉ cho kết nối khi không có chứng chỉ nào được cung cấp.
  - Cuối cùng, các khóa session ticket trong gói
    [`crypto/tls`](/pkg/crypto/tls/)
    hiện có thể được thay đổi trong khi máy chủ đang chạy.
    Điều này được thực hiện thông qua phương thức
    [`SetSessionTicketKeys`](/pkg/crypto/tls/#Config.SetSessionTicketKeys) mới
    của kiểu [`Config`](/pkg/crypto/tls/#Config).
  - Trong gói [`crypto/x509`](/pkg/crypto/x509/),
    các ký tự đại diện hiện chỉ được chấp nhận trong nhãn ngoài cùng bên trái như được xác định trong
    [đặc tả](https://tools.ietf.org/html/rfc6125#section-6.4.3).
  - Cũng trong gói [`crypto/x509`](/pkg/crypto/x509/),
    việc xử lý các phần mở rộng quan trọng chưa biết đã thay đổi.
    Trước đây, chúng gây ra lỗi phân tích cú pháp nhưng bây giờ chúng được phân tích cú pháp và chỉ gây ra lỗi
    trong [`Verify`](/pkg/crypto/x509/#Certificate.Verify).
    Trường `UnhandledCriticalExtensions` mới của
    [`Certificate`](/pkg/crypto/x509/#Certificate) ghi lại các phần mở rộng này.
  - Kiểu [`DB`](/pkg/database/sql/#DB) của gói
    [`database/sql`](/pkg/database/sql/)
    hiện có phương thức [`Stats`](/pkg/database/sql/#DB.Stats)
    để truy xuất thống kê cơ sở dữ liệu.
  - Gói [`debug/dwarf`](/pkg/debug/dwarf/)
    có nhiều bổ sung để hỗ trợ tốt hơn DWARF phiên bản 4.
    Xem ví dụ định nghĩa của kiểu mới
    [`Class`](/pkg/debug/dwarf/#Class).
  - Gói [`debug/dwarf`](/pkg/debug/dwarf/)
    cũng hiện hỗ trợ giải mã bảng dòng DWARF.
  - Gói [`debug/elf`](/pkg/debug/elf/)
    hiện có hỗ trợ cho kiến trúc PowerPC 64-bit.
  - Gói [`encoding/base64`](/pkg/encoding/base64/)
    hiện hỗ trợ mã hóa không đệm thông qua hai biến mã hóa mới,
    [`RawStdEncoding`](/pkg/encoding/base64/#RawStdEncoding) và
    [`RawURLEncoding`](/pkg/encoding/base64/#RawURLEncoding).
  - Gói [`encoding/json`](/pkg/encoding/json/)
    hiện trả về [`UnmarshalTypeError`](/pkg/encoding/json/#UnmarshalTypeError)
    nếu một giá trị JSON không phù hợp với biến hoặc thành phần mục tiêu
    mà nó đang được unmarshal vào.
  - [`Decoder`](/pkg/encoding/json/#Decoder) của `encoding/json`
    có một phương thức mới cung cấp giao diện streaming để giải mã
    một tài liệu JSON:
    [`Token`](/pkg/encoding/json/#Decoder.Token).
    Nó cũng tương tác với chức năng hiện có của `Decode`,
    sẽ tiếp tục thao tác giải mã đã bắt đầu bằng `Decoder.Token`.
  - Gói [`flag`](/pkg/flag/)
    có một hàm mới, [`UnquoteUsage`](/pkg/flag/#UnquoteUsage),
    để hỗ trợ tạo thông báo sử dụng sử dụng quy ước mới
    được mô tả ở trên.
  - Trong gói [`fmt`](/pkg/fmt/),
    một giá trị kiểu [`Value`](/pkg/reflect/#Value) hiện
    in ra những gì nó giữ, thay vì sử dụng phương thức `Stringer`
    của `reflect.Value`, tạo ra những thứ như `<int Value>`.
  - Kiểu [`EmptyStmt`](/pkg/ast/#EmptyStmt)
    trong gói [`go/ast`](/pkg/go/ast/) hiện
    có trường boolean `Implicit` ghi lại liệu dấu chấm phẩy
    có được thêm ngầm hay có trong mã nguồn.
  - Để tương thích tiến, gói [`go/build`](/pkg/go/build/)
    dự trữ các giá trị `GOARCH` cho một số kiến trúc mà Go có thể hỗ trợ một ngày nào đó.
    Đây không phải là lời hứa rằng nó sẽ.
    Ngoài ra, struct [`Package`](/pkg/go/build/#Package)
    hiện có trường `PkgTargetRoot` lưu trữ
    thư mục gốc phụ thuộc kiến trúc để cài đặt, nếu đã biết.
  - Gói [`go/types`](/pkg/go/types/) (mới được di chuyển)
    cho phép kiểm soát tiền tố được đính kèm với các tên cấp gói sử dụng
    kiểu hàm [`Qualifier`](/pkg/go/types/#Qualifier) mới
    làm đối số cho một số hàm. Đây là thay đổi API cho
    gói, nhưng vì nó mới đối với lõi, nó không vi phạm các quy tắc tương thích Go 1
    vì mã sử dụng gói phải yêu cầu rõ ràng tại vị trí mới của nó.
    Để cập nhật, chạy
    [`go fix`](/cmd/go/#hdr-Run_go_tool_fix_on_packages) trên gói của bạn.
  - Trong gói [`image`](/pkg/image/),
    kiểu [`Rectangle`](/pkg/image/#Rectangle)
    hiện hiện thực giao diện [`Image`](/pkg/image/#Image),
    vì vậy `Rectangle` có thể phục vụ như một mask khi vẽ.
  - Cũng trong gói [`image`](/pkg/image/),
    để hỗ trợ xử lý một số ảnh JPEG,
    bây giờ có hỗ trợ cho lấy mẫu phụ YCbCr 4:1:1 và 4:1:0 và hỗ trợ CMYK cơ bản,
    được biểu diễn bởi struct `image.CMYK` mới.
  - Gói [`image/color`](/pkg/image/color/)
    thêm hỗ trợ CMYK cơ bản, thông qua struct mới
    [`CMYK`](/pkg/image/color/#CMYK),
    mô hình màu [`CMYKModel`](/pkg/image/color/#CMYKModel), và hàm
    [`CMYKToRGB`](/pkg/image/color/#CMYKToRGB), theo yêu cầu của một số ảnh JPEG.
  - Cũng trong gói [`image/color`](/pkg/image/color/),
    việc chuyển đổi giá trị [`YCbCr`](/pkg/image/color/#YCbCr)
    sang `RGBA` đã trở nên chính xác hơn.
    Trước đây, 8 bit thấp chỉ là tiếng vang của 8 bit cao;
    bây giờ chúng chứa thông tin chính xác hơn.
    Do thuộc tính tiếng vang của mã cũ, thao tác
    `uint8(r)` để trích xuất giá trị đỏ 8-bit đã hoạt động, nhưng không chính xác.
    Trong Go 1.5, thao tác đó có thể tạo ra giá trị khác.
    Mã đúng là, và luôn là, chọn 8 bit cao:
    `uint8(r>>8)`.
    Nhân tiện, gói `image/draw`
    cung cấp hỗ trợ tốt hơn cho các chuyển đổi như vậy; xem
    [bài đăng blog này](/blog/go-imagedraw-package)
    để biết thêm thông tin.
  - Cuối cùng, kể từ Go 1.5, kiểm tra khớp gần nhất trong
    [`Index`](/pkg/image/color/#Palette.Index)
    hiện tôn trọng kênh alpha.
  - Gói [`image/gif`](/pkg/image/gif/)
    bao gồm một vài tổng quát hóa.
    Một tệp GIF nhiều khung hình hiện có thể có giới hạn tổng thể khác
    với giới hạn của tất cả các khung hình đơn lẻ chứa trong đó.
    Ngoài ra, struct [`GIF`](/pkg/image/gif/#GIF)
    hiện có trường `Disposal`
    chỉ định phương thức xử lý cho mỗi khung hình.
  - Gói [`io`](/pkg/io/)
    thêm hàm [`CopyBuffer`](/pkg/io/#CopyBuffer)
    giống như [`Copy`](/pkg/io/#Copy) nhưng
    sử dụng bộ đệm do người gọi cung cấp, cho phép kiểm soát phân bổ và kích thước bộ đệm.
  - Gói [`log`](/pkg/log/)
    có cờ [`LUTC`](/pkg/log/#LUTC) mới
    khiến dấu thời gian được in trong múi giờ UTC.
    Nó cũng thêm phương thức [`SetOutput`](/pkg/log/#Logger.SetOutput)
    cho các logger do người dùng tạo.
  - Trong Go 1.4, [`Max`](/pkg/math/#Max) không phát hiện tất cả các mẫu bit NaN có thể.
    Điều này đã được sửa trong Go 1.5, vì vậy các chương trình sử dụng `math.Max` trên dữ liệu bao gồm NaN có thể hoạt động khác đi,
    nhưng hiện nay chính xác theo định nghĩa IEEE754 của NaN.
  - Gói [`math/big`](/pkg/math/big/)
    thêm hàm [`Jacobi`](/pkg/math/big/#Jacobi) mới
    cho số nguyên và phương thức
    [`ModSqrt`](/pkg/math/big/#Int.ModSqrt) mới
    cho kiểu [`Int`](/pkg/math/big/#Int).
  - Gói mime
    thêm kiểu [`WordDecoder`](/pkg/mime/#WordDecoder) mới
    để giải mã các tiêu đề MIME chứa các từ được mã hóa RFC 204.
    Nó cũng cung cấp [`BEncoding`](/pkg/mime/#BEncoding) và
    [`QEncoding`](/pkg/mime/#QEncoding)
    như là các hiện thực của các lược đồ mã hóa của RFC 2045 và RFC 2047.
  - Gói [`mime`](/pkg/mime/) cũng thêm một hàm
    [`ExtensionsByType`](/pkg/mime/#ExtensionsByType)
    trả về các phần mở rộng MIME được biết là liên kết với một kiểu MIME nhất định.
  - Có gói [`mime/quotedprintable`](/pkg/mime/quotedprintable/) mới
    hiện thực mã hóa quoted-printable được xác định bởi RFC 2045.
  - Gói [`net`](/pkg/net/) hiện sẽ
    [`Dial`](/pkg/net/#Dial) các hostname bằng cách thử từng
    địa chỉ IP theo thứ tự cho đến khi một địa chỉ thành công.
    Chế độ <code>[Dialer](/pkg/net/#Dialer).DualStack</code>
    hiện hiện thực Happy Eyeballs
    ([RFC 6555](https://tools.ietf.org/html/rfc6555)) bằng cách cho
    họ địa chỉ đầu tiên một lợi thế 300ms; giá trị này có thể bị ghi đè bởi
    `Dialer.FallbackDelay` mới.
  - Một số điểm không nhất quán trong các kiểu được trả về bởi các lỗi trong
    gói [`net`](/pkg/net/) đã được dọn dẹp.
    Hầu hết bây giờ trả về giá trị
    [`OpError`](/pkg/net/#OpError)
    với nhiều thông tin hơn trước.
    Ngoài ra, kiểu [`OpError`](/pkg/net/#OpError)
    hiện bao gồm trường `Source` giữ
    địa chỉ mạng cục bộ.
  - Gói [`net/http`](/pkg/net/http/) hiện
    có hỗ trợ đặt trailer từ máy chủ [`Handler`](/pkg/net/http/#Handler).
    Để biết chi tiết, xem tài liệu cho
    [`ResponseWriter`](/pkg/net/http/#ResponseWriter).
  - Có một phương thức mới để hủy [`net/http`](/pkg/net/http/)
    `Request` bằng cách đặt trường mới
    [`Request.Cancel`](/pkg/net/http/#Request).
    Nó được hỗ trợ bởi `http.Transport`.
    Kiểu của trường `Cancel` tương thích với
    giá trị trả về của [`context.Context.Done`](https://godoc.org/golang.org/x/net/context).
  - Cũng trong gói [`net/http`](/pkg/net/http/),
    có mã để bỏ qua giá trị [`Time`](/pkg/time/#Time) bằng không
    trong hàm [`ServeContent`](/pkg/net/#ServeContent).
    Kể từ Go 1.5, nó cũng bỏ qua giá trị thời gian bằng với epoch Unix.
  - Gói [`net/http/fcgi`](/pkg/net/http/fcgi/)
    xuất hai lỗi mới,
    [`ErrConnClosed`](/pkg/net/http/fcgi/#ErrConnClosed) và
    [`ErrRequestAborted`](/pkg/net/http/fcgi/#ErrRequestAborted),
    để báo cáo các điều kiện lỗi tương ứng.
  - Gói [`net/http/cgi`](/pkg/net/http/cgi/)
    có lỗi xử lý sai giá trị của các biến môi trường
    `REMOTE_ADDR` và `REMOTE_HOST`.
    Điều này đã được sửa.
    Ngoài ra, bắt đầu từ Go 1.5, gói đặt biến `REMOTE_PORT`.
  - Gói [`net/mail`](/pkg/net/mail/)
    thêm kiểu [`AddressParser`](/pkg/net/mail/#AddressParser)
    có thể phân tích cú pháp địa chỉ thư.
  - Gói [`net/smtp`](/pkg/net/smtp/)
    hiện có accessor [`TLSConnectionState`](/pkg/net/smtp/#Client.TLSConnectionState)
    cho kiểu [`Client`](/pkg/net/smtp/#Client)
    trả về trạng thái TLS của máy khách.
  - Gói [`os`](/pkg/os/)
    có hàm [`LookupEnv`](/pkg/os/#LookupEnv) mới
    tương tự như [`Getenv`](/pkg/os/#Getenv)
    nhưng có thể phân biệt giữa biến môi trường trống và biến môi trường không tồn tại.
  - Gói [`os/signal`](/pkg/os/signal/)
    thêm các hàm [`Ignore`](/pkg/os/signal/#Ignore) và
    [`Reset`](/pkg/os/signal/#Reset) mới.
  - Các gói [`runtime`](/pkg/runtime/),
    [`runtime/trace`](/pkg/runtime/trace/),
    và [`net/http/pprof`](/pkg/net/http/pprof/)
    mỗi gói có các hàm mới để hỗ trợ các cơ sở tracing được mô tả ở trên:
    [`ReadTrace`](/pkg/runtime/#ReadTrace),
    [`StartTrace`](/pkg/runtime/#StartTrace),
    [`StopTrace`](/pkg/runtime/#StopTrace),
    [`Start`](/pkg/runtime/trace/#Start),
    [`Stop`](/pkg/runtime/trace/#Stop), và
    [`Trace`](/pkg/net/http/pprof/#Trace).
    Xem tài liệu tương ứng để biết chi tiết.
  - Gói [`runtime/pprof`](/pkg/runtime/pprof/)
    theo mặc định hiện bao gồm thống kê bộ nhớ tổng thể trong tất cả các profile bộ nhớ.
  - Gói [`strings`](/pkg/strings/)
    có hàm [`Compare`](/pkg/strings/#Compare) mới.
    Điều này hiện diện để cung cấp sự đối xứng với gói [`bytes`](/pkg/bytes/)
    nhưng không cần thiết vì các chuỗi hỗ trợ so sánh tự nhiên.
  - Hiện thực [`WaitGroup`](/pkg/sync/#WaitGroup) trong
    gói [`sync`](/pkg/sync/)
    hiện chẩn đoán mã chạy đua lệnh gọi [`Add`](/pkg/sync/#WaitGroup.Add)
    với việc trả về từ [`Wait`](/pkg/sync/#WaitGroup.Wait).
    Nếu phát hiện điều kiện này, hiện thực sẽ panic.
  - Trong gói [`syscall`](/pkg/syscall/),
    struct `SysProcAttr` Linux hiện có trường
    `GidMappingsEnableSetgroups`, được tạo ra do các thay đổi bảo mật trong Linux 3.19.
    Trên tất cả các hệ thống Unix, struct cũng có trường `Foreground` và `Pgid` mới
    để cung cấp kiểm soát nhiều hơn khi exec.
    Trên Darwin, hiện có hàm `Syscall9`
    để hỗ trợ các lệnh gọi có quá nhiều đối số.
  - [`testing/quick`](/pkg/testing/quick/) hiện sẽ
    tạo ra các giá trị `nil` cho các kiểu con trỏ,
    giúp có thể sử dụng với các cấu trúc dữ liệu đệ quy.
    Ngoài ra, gói hiện hỗ trợ tạo các kiểu mảng.
  - Trong các gói [`text/template`](/pkg/text/template/) và
    [`html/template`](/pkg/html/template/),
    các hằng số nguyên quá lớn để được biểu diễn như một số nguyên Go hiện kích hoạt
    lỗi phân tích cú pháp. Trước đây, chúng được chuyển đổi âm thầm sang dấu phẩy động, làm mất
    độ chính xác.
  - Cũng trong các gói [`text/template`](/pkg/text/template/) và
    [`html/template`](/pkg/html/template/),
    phương thức [`Option`](/pkg/text/template/#Template.Option) mới
    cho phép tùy chỉnh hành vi của template trong quá trình thực thi.
    Tùy chọn duy nhất được hiện thực cho phép kiểm soát cách xử lý khóa bị thiếu
    khi lập chỉ mục map.
    Mặc định, hiện có thể ghi đè, là như trước: tiếp tục với giá trị không hợp lệ.
  - Kiểu `Time` của gói [`time`](/pkg/time/)
    có phương thức mới
    [`AppendFormat`](/pkg/time/#Time.AppendFormat),
    có thể được sử dụng để tránh phân bổ khi in giá trị thời gian.
  - Gói [`unicode`](/pkg/unicode/) và hỗ trợ liên quan
    trong toàn bộ hệ thống đã được nâng cấp từ phiên bản 7.0 lên
    [Unicode 8.0](https://www.unicode.org/versions/Unicode8.0.0/).
