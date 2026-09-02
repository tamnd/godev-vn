---
title: Ghi chú phát hành Go 1.1
---

## Giới thiệu về Go 1.1 {#introduction}

VIỆC PHÁT HÀNH [Go phiên bản 1](/doc/go1.html) (Go 1 hay Go 1.0) vào
tháng 3 năm 2012 đã mở ra một thời kỳ ổn định mới trong ngôn ngữ và thư viện Go.
Sự ổn định đó đã giúp nuôi dưỡng một cộng đồng người dùng Go và
các hệ thống ngày càng phát triển trên toàn thế giới.
Một số bản phát hành "điểm" từ
đó đến nay, 1.0.1, 1.0.2 và 1.0.3, đã được phát hành.
Các bản phát hành điểm này sửa các lỗi đã biết nhưng không thực hiện
các thay đổi không nghiêm trọng đối với việc triển khai.

Bản phát hành mới này, Go 1.1, vẫn giữ [cam kết
tương thích](/doc/go1compat.html) nhưng thêm một số thay đổi ngôn ngữ đáng kể
(tất nhiên là tương thích ngược), có một danh sách dài các thay đổi thư viện (cũng tương thích),
và bao gồm công việc lớn về triển khai trình biên dịch,
thư viện và thời gian chạy.
Trọng tâm là hiệu năng.
Benchmarking là một khoa học không chính xác nhất, nhưng chúng tôi thấy sự cải thiện đáng kể,
đôi khi đáng kể, cho nhiều chương trình test.
Chúng tôi tin rằng nhiều chương trình của người dùng cũng sẽ thấy cải thiện
chỉ bằng cách cập nhật cài đặt Go và biên dịch lại.

Tài liệu này tóm tắt các thay đổi giữa Go 1 và Go 1.1.
Rất ít mã nguồn, nếu có, cần sửa đổi để chạy với Go 1.1,
mặc dù một số trường hợp lỗi hiếm gặp xuất hiện với bản phát hành này
và cần được giải quyết nếu chúng phát sinh.
Chi tiết xuất hiện bên dưới; xem đặc biệt phần thảo luận về
[int 64 bit](#int) và [ký tự literal Unicode](#unicode_literals).

## Thay đổi về ngôn ngữ {#language}

[Tài liệu tương thích Go](/doc/go1compat.html) hứa hẹn
rằng các chương trình viết theo đặc tả ngôn ngữ Go 1 sẽ tiếp tục hoạt động,
và những cam kết đó được duy trì.
Tuy nhiên, để củng cố đặc tả, có
các chi tiết về một số trường hợp lỗi đã được làm rõ.
Ngoài ra còn có một số tính năng ngôn ngữ mới.

### Chia số nguyên cho không {#divzero}

Trong Go 1, phép chia số nguyên cho không là hằng số tạo ra một panic thời gian chạy:

	func f(x int) int {
		return x/0
	}

Trong Go 1.1, phép chia số nguyên cho không là hằng số không phải là chương trình hợp lệ, vì vậy đó là lỗi lúc biên dịch.

### Surrogate trong Unicode literal {#unicode_literals}

Định nghĩa về string và rune literal đã được tinh chỉnh để loại trừ các surrogate half khỏi
tập hợp các điểm mã Unicode hợp lệ.
Xem phần [Unicode](#unicode) để biết thêm thông tin.

### Giá trị phương thức {#method_values}

Go 1.1 bây giờ triển khai
[giá trị phương thức](/ref/spec#Method_values),
là các hàm đã được gắn kết với một giá trị receiver cụ thể.
Ví dụ, cho một giá trị
[`Writer`](/pkg/bufio/#Writer)
`w`,
biểu thức
`w.Write`,
một giá trị phương thức, là một hàm sẽ luôn ghi vào `w`; nó tương đương với
một hàm literal đóng gói trên `w`:

	func (p []byte) (n int, err error) {
		return w.Write(p)
	}

Giá trị phương thức khác với biểu thức phương thức, tạo ra các hàm
từ các phương thức của một kiểu nhất định; biểu thức phương thức `(*bufio.Writer).Write`
tương đương với một hàm có thêm đối số đầu tiên, một receiver kiểu
`(*bufio.Writer)`:

	func (w *bufio.Writer, p []byte) (n int, err error) {
		return w.Write(p)
	}

_Cập nhật_: Không có mã nguồn hiện có nào bị ảnh hưởng; thay đổi hoàn toàn tương thích ngược.

### Yêu cầu return {#return}

Trước Go 1.1, một hàm trả về một giá trị cần có "return" tường minh
hoặc lệnh gọi `panic` ở
cuối hàm; đây là một cách đơn giản để buộc lập trình viên
phải rõ ràng về ý nghĩa của hàm. Nhưng có nhiều trường hợp
mà "return" cuối cùng rõ ràng là không cần thiết, chẳng hạn như một hàm chỉ có
vòng lặp "for" vô hạn.

Trong Go 1.1, quy tắc về các câu lệnh "return" cuối cùng được nới lỏng hơn.
Nó giới thiệu khái niệm
[_terminating statement_](/ref/spec#Terminating_statements),
một câu lệnh được đảm bảo là câu lệnh cuối cùng mà một hàm thực thi.
Các ví dụ bao gồm
các vòng lặp "for" không có điều kiện và các câu lệnh "if-else"
mà mỗi nửa kết thúc bằng "return".
Nếu câu lệnh cuối cùng của một hàm có thể được chứng minh _về mặt cú pháp_ là
một terminating statement, không cần câu lệnh "return" cuối cùng nào.

Lưu ý rằng quy tắc hoàn toàn có tính cú pháp: nó không chú ý đến các giá trị trong
mã nguồn và do đó không yêu cầu phân tích phức tạp.

_Cập nhật_: Thay đổi tương thích ngược, nhưng mã nguồn hiện có
với các câu lệnh "return" và lệnh gọi `panic` thừa có thể
được đơn giản hóa thủ công.
Mã nguồn như vậy có thể được xác định bởi `go vet`.

## Thay đổi về triển khai và công cụ {#impl}

### Trạng thái của gccgo {#gccgo}

Lịch phát hành GCC không trùng với lịch phát hành Go, vì vậy một số sai lệch là không thể tránh khỏi trong
các bản phát hành của `gccgo`.
Phiên bản 4.8.0 của GCC phát hành vào tháng 3 năm 2013 bao gồm phiên bản gần như Go 1.1 của `gccgo`.
Thư viện của nó hơi tụt hậu so với bản phát hành, nhưng sự khác biệt lớn nhất là các giá trị phương thức không được triển khai.
Vào khoảng tháng 7 năm 2013, chúng tôi dự kiến GCC 4.8.2 sẽ phát hành với một `gccgo`
cung cấp triển khai Go 1.1 hoàn chỉnh.

### Phân tích flag dòng lệnh {#gc_flag}

Trong toolchain gc, các trình biên dịch và linker bây giờ dùng
các quy tắc phân tích flag dòng lệnh giống như gói flag Go, khác biệt
so với cách phân tích flag Unix truyền thống. Điều này có thể ảnh hưởng đến các script gọi
công cụ trực tiếp.
Ví dụ,
`go tool 6c -Fw -Dfoo` bây giờ phải được viết
`go tool 6c -F -w -D foo`.

### Kích thước int trên các nền tảng 64 bit {#int}

Ngôn ngữ cho phép triển khai chọn liệu kiểu `int` và
`uint` là 32 hay 64 bit. Các triển khai Go trước đây làm cho `int`
và `uint` là 32 bit trên tất cả các hệ thống. Cả hai triển khai gc và gccgo
bây giờ làm cho
`int` và `uint` là 64 bit trên các nền tảng 64 bit như AMD64/x86-64.
Trong số các thứ khác, điều này cho phép cấp phát slice với
hơn 2 tỷ phần tử trên các nền tảng 64 bit.

_Cập nhật_:
Hầu hết chương trình sẽ không bị ảnh hưởng bởi thay đổi này.
Vì Go không cho phép chuyển đổi ngầm giữa các
[kiểu số](/ref/spec#Numeric_types) khác nhau,
không có chương trình nào sẽ ngừng biên dịch do thay đổi này.
Tuy nhiên, các chương trình chứa các giả định ngầm
rằng `int` chỉ là 32 bit có thể thay đổi hành vi.
Ví dụ, mã nguồn này in một số dương trên các hệ thống 64 bit và
một số âm trên các hệ thống 32 bit:

	x := ^uint32(0) // x là 0xffffffff
	i := int(x)     // i là -1 trên hệ thống 32 bit, 0xffffffff trên hệ thống 64 bit
	fmt.Println(i)

Mã nguồn di động nhằm mở rộng dấu 32 bit (tạo ra `-1` trên tất cả các hệ thống)
thay vào đó sẽ nói:

	i := int(int32(x))

### Kích thước heap trên các kiến trúc 64 bit {#heap}

Trên các kiến trúc 64 bit, kích thước heap tối đa đã được mở rộng đáng kể,
từ vài gigabyte lên đến vài chục gigabyte.
(Chi tiết chính xác phụ thuộc vào hệ thống và có thể thay đổi.)

Trên các kiến trúc 32 bit, kích thước heap không thay đổi.

_Cập nhật_:
Thay đổi này không có tác dụng đối với các chương trình hiện có ngoài việc cho phép chúng
chạy với heap lớn hơn.

### Unicode {#unicode}

Để có thể biểu diễn các điểm mã lớn hơn 65535 trong UTF-16,
Unicode định nghĩa _surrogate half_,
một dải điểm mã chỉ được dùng trong việc lắp ráp các giá trị lớn, và chỉ trong UTF-16.
Các điểm mã trong dải surrogate đó là bất hợp pháp cho bất kỳ mục đích nào khác.
Trong Go 1.1, ràng buộc này được tuân thủ bởi trình biên dịch, thư viện và thời gian chạy:
một surrogate half là bất hợp pháp như một giá trị rune, khi được mã hóa dưới dạng UTF-8, hoặc khi
được mã hóa riêng lẻ dưới dạng UTF-16.
Khi gặp phải, ví dụ khi chuyển đổi từ rune sang UTF-8, nó được
xử lý như một lỗi mã hóa và sẽ trả về rune thay thế,
[`utf8.RuneError`](/pkg/unicode/utf8/#RuneError),
U+FFFD.

Chương trình này,

	import "fmt"

	func main() {
	    fmt.Printf("%+q\n", string(0xD800))
	}

in `"\ud800"` trong Go 1.0, nhưng in `"\ufffd"` trong Go 1.1.

Các giá trị Unicode surrogate-half bây giờ là bất hợp pháp trong các hằng số rune và string, vì vậy các hằng số như
`'\ud800'` và `"\ud800"` bây giờ bị các trình biên dịch từ chối.
Khi được viết tường minh dưới dạng các byte được mã hóa UTF-8,
các chuỗi như vậy vẫn có thể được tạo ra, như trong `"\xed\xa0\x80"`.
Tuy nhiên, khi một chuỗi như vậy được giải mã dưới dạng một chuỗi rune, như trong vòng lặp range, nó sẽ chỉ tạo ra các giá trị `utf8.RuneError`.

Dấu thứ tự byte Unicode U+FEFF, được mã hóa trong UTF-8, bây giờ được phép là ký tự đầu tiên
của một file nguồn Go.
Mặc dù sự xuất hiện của nó trong mã hóa UTF-8 không có thứ tự byte là rõ ràng không cần thiết,
một số trình soạn thảo thêm dấu này như một loại "số ma thuật" xác định một file được mã hóa UTF-8.

_Cập nhật_:
Hầu hết chương trình sẽ không bị ảnh hưởng bởi thay đổi surrogate.
Các chương trình phụ thuộc vào hành vi cũ nên được sửa đổi để tránh vấn đề.
Thay đổi dấu thứ tự byte hoàn toàn tương thích ngược.

### Race detector {#race}

Một bổ sung lớn cho các công cụ là một _race detector_, một cách
tìm lỗi trong các chương trình gây ra bởi truy cập đồng thời của cùng một
biến, trong đó ít nhất một trong các truy cập là ghi.
Tính năng mới này được tích hợp vào công cụ `go`.
Hiện tại, nó chỉ có sẵn trên các hệ thống Linux, Mac OS X và Windows với
bộ xử lý x86 64 bit.
Để bật nó, đặt flag `-race` khi build hoặc test chương trình
(ví dụ: `go test -race`).
Race detector được ghi lại trong [một bài viết riêng](/doc/articles/race_detector.html).

### Trình hợp dịch gc {#gc_asm}

Do thay đổi của [`int`](#int) lên 64 bit và
một [biểu diễn nội bộ mới của các hàm](/s/go11func),
sắp xếp các đối số hàm trên stack đã thay đổi trong toolchain gc.
Các hàm viết bằng assembly sẽ cần được sửa đổi ít nhất
để điều chỉnh các offset của frame pointer.

_Cập nhật_:
Lệnh `go vet` bây giờ kiểm tra rằng các hàm được triển khai trong assembly
khớp với các prototype hàm Go mà chúng triển khai.

### Thay đổi đối với lệnh go {#gocmd}

Lệnh [`go`](/cmd/go/) đã có một số
thay đổi nhằm cải thiện trải nghiệm cho người dùng Go mới.

Đầu tiên, khi biên dịch, test hoặc chạy mã Go, lệnh `go` bây giờ sẽ đưa ra thông báo lỗi chi tiết hơn,
bao gồm danh sách các đường dẫn đã tìm kiếm, khi không thể tìm thấy một gói.

	$ go build foo/quxx
	can't load package: package foo/quxx: cannot find package "foo/quxx" in any of:
	        /home/you/go/src/pkg/foo/quxx (from $GOROOT)
	        /home/you/src/foo/quxx (from $GOPATH)

Thứ hai, lệnh `go get` không còn cho phép `$GOROOT`
là đích mặc định khi tải xuống mã nguồn gói.
Để dùng lệnh `go get`,
một [`$GOPATH` hợp lệ](/doc/code.html#GOPATH) bây giờ là bắt buộc.

	$ GOPATH= go get code.google.com/p/foo/quxx
	package code.google.com/p/foo/quxx: cannot download, $GOPATH not set. For more details see: go help gopath

Cuối cùng, do thay đổi trước đó, lệnh `go get` cũng sẽ thất bại
khi `$GOPATH` và `$GOROOT` được đặt thành cùng giá trị.

	$ GOPATH=$GOROOT go get code.google.com/p/foo/quxx
	warning: GOPATH set to GOROOT (/home/you/go) has no effect
	package code.google.com/p/foo/quxx: cannot download, $GOPATH must not be set to $GOROOT. For more details see: go help gopath

### Thay đổi đối với lệnh go test {#gotest}

Lệnh [`go test`](/cmd/go/#hdr-Test_packages)
không còn xóa binary khi chạy với profiling được bật,
để dễ dàng phân tích profile hơn.
Triển khai tự động đặt flag `-c`, vì vậy sau khi chạy,

	$ go test -cpuprofile cpuprof.out mypackage

file `mypackage.test` sẽ được để lại trong thư mục nơi `go test` được chạy.

Lệnh [`go test`](/cmd/go/#hdr-Test_packages)
bây giờ có thể tạo thông tin profiling
báo cáo nơi goroutine bị block, tức là,
nơi chúng có xu hướng bị đình trệ khi chờ một sự kiện như giao tiếp channel.
Thông tin được trình bày như một
_blocking profile_
được bật bằng tùy chọn
`-blockprofile`
của
`go test`.
Chạy `go help test` để biết thêm thông tin.

### Thay đổi đối với lệnh go fix {#gofix}

Lệnh [`fix`](/cmd/fix/), thường được chạy như
`go fix`, không còn áp dụng các sửa lỗi để cập nhật mã nguồn từ
trước Go 1 để dùng các API Go 1.
Để cập nhật mã nguồn trước Go 1 lên Go 1.1, dùng một toolchain Go 1.0
để chuyển đổi mã nguồn sang Go 1.0 trước.

### Ràng buộc build {#tags}

Tag "`go1.1`" đã được thêm vào danh sách mặc định
[ràng buộc build](/pkg/go/build/#hdr-Build_Constraints).
Điều này cho phép các gói tận dụng các tính năng mới trong Go 1.1 trong khi
vẫn tương thích với các phiên bản Go trước đó.

Để build một file chỉ với Go 1.1 trở lên, thêm ràng buộc build này:

	// +build go1.1

Để build một file chỉ với Go 1.0.x, dùng ràng buộc ngược lại:

	// +build !go1.1

### Các nền tảng bổ sung {#platforms}

Toolchain Go 1.1 thêm hỗ trợ thử nghiệm cho `freebsd/arm`,
`netbsd/386`, `netbsd/amd64`, `netbsd/arm`,
`openbsd/386` và `openbsd/amd64`.

Cần bộ xử lý ARMv6 trở lên cho `freebsd/arm` hoặc
`netbsd/arm`.

Go 1.1 thêm hỗ trợ thử nghiệm cho `cgo` trên `linux/arm`.

### Biên dịch chéo {#crosscompile}

Khi biên dịch chéo, công cụ `go` sẽ tắt hỗ trợ `cgo`
theo mặc định.

Để bật tường minh `cgo`, đặt `CGO_ENABLED=1`.

## Hiệu năng {#performance}

Hiệu năng của mã nguồn được biên dịch với bộ công cụ gc Go 1.1 nên tốt hơn đáng kể
cho hầu hết các chương trình Go.
Cải thiện điển hình so với Go 1.0 dường như khoảng 30%-40%, đôi khi
nhiều hơn, nhưng đôi khi ít hơn hoặc thậm chí không tồn tại.
Có quá nhiều tinh chỉnh nhỏ về hiệu năng qua các công cụ và thư viện
để liệt kê tất cả ở đây, nhưng các thay đổi lớn sau đây đáng được ghi nhớ:

  - Các trình biên dịch gc tạo ra mã nguồn tốt hơn trong nhiều trường hợp, đáng chú ý nhất là
    đối với dấu phẩy động trên kiến trúc Intel 32 bit.
  - Các trình biên dịch gc thực hiện nội tuyến (inlining) nhiều hơn, bao gồm một số thao tác
    trong thời gian chạy như [`append`](/pkg/builtin/#append)
    và chuyển đổi interface.
  - Có một triển khai mới của map Go với giảm đáng kể về
    dung lượng bộ nhớ và thời gian CPU.
  - Bộ gom rác đã được song song hóa nhiều hơn, điều này có thể giảm
    độ trễ cho các chương trình chạy trên nhiều CPU.
  - Bộ gom rác cũng chính xác hơn, điều này tốn một lượng nhỏ
    thời gian CPU nhưng có thể giảm đáng kể kích thước heap, đặc biệt
    trên các kiến trúc 32 bit.
  - Do sự kết hợp chặt chẽ hơn của thời gian chạy và thư viện mạng, cần ít
    chuyển đổi ngữ cảnh hơn trong các thao tác mạng.

## Thay đổi đối với thư viện chuẩn {#library}

### bufio.Scanner {#bufio_scanner}

Các quy trình khác nhau để quét đầu vào văn bản trong gói
[`bufio`](/pkg/bufio/),
[`ReadBytes`](/pkg/bufio/#Reader.ReadBytes),
[`ReadString`](/pkg/bufio/#Reader.ReadString)
và đặc biệt là
[`ReadLine`](/pkg/bufio/#Reader.ReadLine),
phức tạp không cần thiết để dùng cho các mục đích đơn giản.
Trong Go 1.1, một kiểu mới,
[`Scanner`](/pkg/bufio/#Scanner),
đã được thêm vào để dễ dàng thực hiện các nhiệm vụ đơn giản như
đọc đầu vào dưới dạng chuỗi dòng hoặc các từ phân cách bởi khoảng trắng.
Nó đơn giản hóa vấn đề bằng cách kết thúc quét trên
đầu vào có vấn đề như các dòng quá dài, và có một mặc định đơn giản:
đầu vào hướng dòng, với mỗi dòng được loại bỏ dấu kết thúc.
Đây là mã nguồn để tái tạo đầu vào một dòng tại một thời điểm:

	scanner := bufio.NewScanner(os.Stdin)
	for scanner.Scan() {
	    fmt.Println(scanner.Text()) // Println sẽ thêm lại '\n' cuối
	}
	if err := scanner.Err(); err != nil {
	    fmt.Fprintln(os.Stderr, "reading standard input:", err)
	}

Hành vi quét có thể được điều chỉnh thông qua một hàm để kiểm soát việc chia nhỏ đầu vào
(xem tài liệu cho [`SplitFunc`](/pkg/bufio/#SplitFunc)),
nhưng đối với các vấn đề khó hoặc cần tiếp tục sau lỗi, interface cũ
vẫn có thể được yêu cầu.

### net {#net}

Các resolver theo giao thức trong gói [`net`](/pkg/net/) trước đây
không nghiêm ngặt về tên mạng được truyền vào.
Mặc dù tài liệu đã rõ ràng
rằng các mạng hợp lệ duy nhất cho
[`ResolveTCPAddr`](/pkg/net/#ResolveTCPAddr)
là `"tcp"`,
`"tcp4"` và `"tcp6"`, triển khai Go 1.0 âm thầm chấp nhận bất kỳ chuỗi nào.
Triển khai Go 1.1 trả về lỗi nếu mạng không phải là một trong các chuỗi đó.
Điều tương tự cũng đúng với các resolver theo giao thức khác [`ResolveIPAddr`](/pkg/net/#ResolveIPAddr),
[`ResolveUDPAddr`](/pkg/net/#ResolveUDPAddr) và
[`ResolveUnixAddr`](/pkg/net/#ResolveUnixAddr).

Triển khai trước đó của
[`ListenUnixgram`](/pkg/net/#ListenUnixgram)
trả về một
[`UDPConn`](/pkg/net/#UDPConn) như là
biểu diễn của endpoint kết nối.
Triển khai Go 1.1 thay vào đó trả về một
[`UnixConn`](/pkg/net/#UnixConn)
để cho phép đọc và ghi
với các phương thức
[`ReadFrom`](/pkg/net/#UnixConn.ReadFrom)
và
[`WriteTo`](/pkg/net/#UnixConn.WriteTo)
của nó.

Các cấu trúc dữ liệu
[`IPAddr`](/pkg/net/#IPAddr),
[`TCPAddr`](/pkg/net/#TCPAddr) và
[`UDPAddr`](/pkg/net/#UDPAddr)
thêm một field chuỗi mới gọi là `Zone`.
Mã nguồn dùng composite literal không có tag (ví dụ: `net.TCPAddr{ip, port}`)
thay vì literal có tag (`net.TCPAddr{IP: ip, Port: port}`)
sẽ bị hỏng do field mới.
Các quy tắc tương thích Go 1 cho phép thay đổi này: mã nguồn client phải dùng literal có tag để tránh hỏng như vậy.

_Cập nhật_:
Để sửa hỏng gây ra bởi field struct mới,
`go fix` sẽ viết lại mã nguồn để thêm tag cho các kiểu này.
Tổng quát hơn, `go vet` sẽ xác định các composite literal
nên được sửa đổi để dùng field tag.

### reflect {#reflect}

Gói [`reflect`](/pkg/reflect/) có một số bổ sung quan trọng.

Bây giờ có thể chạy câu lệnh "select" bằng
gói `reflect`; xem mô tả của
[`Select`](/pkg/reflect/#Select)
và
[`SelectCase`](/pkg/reflect/#SelectCase)
để biết chi tiết.

Phương thức mới
[`Value.Convert`](/pkg/reflect/#Value.Convert)
(hoặc
[`Type.ConvertibleTo`](/pkg/reflect/#Type))
cung cấp chức năng để thực thi một thao tác chuyển đổi Go hoặc type assertion
trên một
[`Value`](/pkg/reflect/#Value)
(hoặc kiểm tra khả năng của nó).

Hàm mới
[`MakeFunc`](/pkg/reflect/#MakeFunc)
tạo một hàm wrapper để dễ dàng gọi một hàm với
[`Values`](/pkg/reflect/#Value) hiện có,
thực hiện các chuyển đổi Go chuẩn giữa các đối số, ví dụ
để truyền một `int` thực tế cho một `interface{}` hình thức.

Cuối cùng, các hàm mới
[`ChanOf`](/pkg/reflect/#ChanOf),
[`MapOf`](/pkg/reflect/#MapOf)
và
[`SliceOf`](/pkg/reflect/#SliceOf)
xây dựng các
[`Types`](/pkg/reflect/#Type) mới
từ các kiểu hiện có, ví dụ để xây dựng kiểu `[]T` chỉ cho
`T`.

### time {#time}

Trên FreeBSD, Linux, NetBSD, OS X và OpenBSD, các phiên bản trước của gói
[`time`](/pkg/time/)
trả về thời gian với độ chính xác microsecond.
Triển khai Go 1.1 trên các
hệ thống này bây giờ trả về thời gian với độ chính xác nanosecond.
Các chương trình ghi ra một định dạng bên ngoài với độ chính xác microsecond
và đọc lại, kỳ vọng phục hồi giá trị ban đầu, sẽ bị ảnh hưởng
bởi mất độ chính xác.
Có hai phương thức mới của [`Time`](/pkg/time/#Time),
[`Round`](/pkg/time/#Time.Round)
và
[`Truncate`](/pkg/time/#Time.Truncate),
có thể được dùng để loại bỏ độ chính xác khỏi một thời gian trước khi truyền nó đến
lưu trữ bên ngoài.

Phương thức mới
[`YearDay`](/pkg/time/#Time.YearDay)
trả về số ngày trong năm được chỉ số nguyên 1-indexed của năm được chỉ định bởi giá trị thời gian.

Kiểu
[`Timer`](/pkg/time/#Timer)
có một phương thức mới
[`Reset`](/pkg/time/#Timer.Reset)
sửa đổi timer để hết hạn sau một khoảng thời gian được chỉ định.

Cuối cùng, hàm mới
[`ParseInLocation`](/pkg/time/#ParseInLocation)
giống như
[`Parse`](/pkg/time/#Parse) hiện có
nhưng phân tích cú pháp thời gian trong ngữ cảnh của một location (múi giờ), bỏ qua
thông tin múi giờ trong chuỗi được phân tích cú pháp.
Hàm này giải quyết một nguồn nhầm lẫn phổ biến trong API time.

_Cập nhật_:
Mã nguồn cần đọc và ghi thời gian bằng định dạng bên ngoài với
độ chính xác thấp hơn nên được sửa đổi để dùng các phương thức mới.

### Cây con Exp và old được chuyển đến kho phụ go.exp và go.text {#exp_old}

Để các bản phân phối nhị phân dễ dàng truy cập chúng nếu muốn, các cây nguồn `exp`
và `old`, không được bao gồm trong các bản phân phối nhị phân,
đã được chuyển đến kho phụ `go.exp` mới tại
`code.google.com/p/go.exp`. Để truy cập gói `ssa`,
ví dụ, chạy

	$ go get code.google.com/p/go.exp/ssa

và sau đó trong mã nguồn Go,

	import "code.google.com/p/go.exp/ssa"

Gói cũ `exp/norm` cũng đã được chuyển, nhưng đến một kho lưu trữ mới
`go.text`, nơi các API Unicode và các gói liên quan đến văn bản khác sẽ
được phát triển.

### Các gói mới {#new_packages}

Có ba gói mới.

  - Gói [`go/format`](/pkg/go/format/) cung cấp
    một cách thuận tiện cho một chương trình để truy cập các khả năng định dạng của lệnh
    [`go fmt`](/cmd/go/#hdr-Run_gofmt_on_package_sources).
    Nó có hai hàm,
    [`Node`](/pkg/go/format/#Node) để định dạng một
    [`Node`](/pkg/go/ast/#Node) của Go parser,
    và
    [`Source`](/pkg/go/format/#Source)
    để định dạng lại mã nguồn Go tùy ý thành định dạng chuẩn theo cung cấp bởi lệnh
    [`go fmt`](/cmd/go/#hdr-Run_gofmt_on_package_sources).
  - Gói [`net/http/cookiejar`](/pkg/net/http/cookiejar/) cung cấp những cơ bản để quản lý cookie HTTP.
  - Gói [`runtime/race`](/pkg/runtime/race/) cung cấp các tiện ích cấp thấp để phát hiện data race.
    Nó là nội bộ cho race detector và không xuất bất kỳ chức năng hiển thị với người dùng nào khác.

### Thay đổi nhỏ đối với thư viện {#minor_library_changes}

Danh sách sau đây tóm tắt một số thay đổi nhỏ đối với thư viện, chủ yếu là bổ sung.
Xem tài liệu gói liên quan để biết thêm thông tin về từng thay đổi.

  - Gói [`bytes`](/pkg/bytes/) có hai hàm mới,
    [`TrimPrefix`](/pkg/bytes/#TrimPrefix)
    và
    [`TrimSuffix`](/pkg/bytes/#TrimSuffix),
    với các thuộc tính tự giải thích.
    Ngoài ra, kiểu [`Buffer`](/pkg/bytes/#Buffer)
    có phương thức mới
    [`Grow`](/pkg/bytes/#Buffer.Grow) cung cấp một số kiểm soát về cấp phát bộ nhớ bên trong buffer.
    Cuối cùng, kiểu
    [`Reader`](/pkg/bytes/#Reader) bây giờ có phương thức
    [`WriteTo`](/pkg/strings/#Reader.WriteTo)
    để nó triển khai interface
    [`io.WriterTo`](/pkg/io/#WriterTo).
  - Gói [`compress/gzip`](/pkg/compress/gzip/) có
    phương thức [`Flush`](/pkg/compress/gzip/#Writer.Flush) mới cho kiểu
    [`Writer`](/pkg/compress/gzip/#Writer)
    để flush `flate.Writer` bên dưới của nó.
  - Gói [`crypto/hmac`](/pkg/crypto/hmac/) có hàm mới,
    [`Equal`](/pkg/crypto/hmac/#Equal), để so sánh hai MAC.
  - Gói [`crypto/x509`](/pkg/crypto/x509/)
    bây giờ hỗ trợ các block PEM (xem
    [`DecryptPEMBlock`](/pkg/crypto/x509/#DecryptPEMBlock) chẳng hạn),
    và một hàm mới
    [`ParseECPrivateKey`](/pkg/crypto/x509/#ParseECPrivateKey) để phân tích các khóa riêng tư elliptic curve.
  - Gói [`database/sql`](/pkg/database/sql/)
    có phương thức mới
    [`Ping`](/pkg/database/sql/#DB.Ping)
    cho kiểu [`DB`](/pkg/database/sql/#DB)
    kiểm tra tình trạng kết nối.
  - Gói [`database/sql/driver`](/pkg/database/sql/driver/)
    có interface mới
    [`Queryer`](/pkg/database/sql/driver/#Queryer)
    mà một
    [`Conn`](/pkg/database/sql/driver/#Conn)
    có thể triển khai để cải thiện hiệu năng.
  - [`Decoder`](/pkg/encoding/json/#Decoder)
    của gói [`encoding/json`](/pkg/encoding/json/)
    có phương thức mới
    [`Buffered`](/pkg/encoding/json/#Decoder.Buffered)
    để cung cấp truy cập vào dữ liệu còn lại trong buffer của nó,
    cũng như phương thức mới
    [`UseNumber`](/pkg/encoding/json/#Decoder.UseNumber)
    để unmarshal một giá trị vào kiểu mới
    [`Number`](/pkg/encoding/json/#Number),
    một chuỗi, thay vì float64.
  - Gói [`encoding/xml`](/pkg/encoding/xml/)
    có hàm mới,
    [`EscapeText`](/pkg/encoding/xml/#EscapeText),
    ghi đầu ra XML đã được escape,
    và một phương thức trên
    [`Encoder`](/pkg/encoding/xml/#Encoder),
    [`Indent`](/pkg/encoding/xml/#Encoder.Indent),
    để chỉ định đầu ra thụt lề.
  - Trong gói [`go/ast`](/pkg/go/ast/), kiểu mới
    [`CommentMap`](/pkg/go/ast/#CommentMap)
    và các phương thức liên quan giúp dễ dàng trích xuất và xử lý các comment trong các chương trình Go.
  - Trong gói [`go/doc`](/pkg/go/doc/),
    parser bây giờ theo dõi tốt hơn các chú thích có kiểu như `TODO(joe)`
    trong toàn bộ mã nguồn,
    thông tin mà lệnh [`godoc`](/cmd/godoc/)
    có thể lọc hoặc trình bày theo giá trị của flag `-notes`.
  - Tính năng "noescape" không được ghi lại và chỉ được triển khai một phần của gói
    [`html/template`](/pkg/html/template/)
    đã bị xóa; các chương trình phụ thuộc vào nó sẽ bị hỏng.
  - Gói [`image/jpeg`](/pkg/image/jpeg/) bây giờ
    đọc các file JPEG progressive và xử lý thêm một số cấu hình lấy mẫu phụ.
  - Gói [`io`](/pkg/io/) bây giờ xuất
    interface [`io.ByteWriter`](/pkg/io/#ByteWriter) để nắm bắt chức năng phổ biến
    của việc ghi một byte tại một thời điểm.
    Nó cũng xuất một lỗi mới, [`ErrNoProgress`](/pkg/io/#ErrNoProgress),
    dùng để biểu thị rằng một triển khai `Read` đang lặp mà không cung cấp dữ liệu.
  - Gói [`log/syslog`](/pkg/log/syslog/) bây giờ cung cấp hỗ trợ tốt hơn
    cho các tính năng logging đặc thù của hệ điều hành.
  - Kiểu [`Int`](/pkg/math/big/#Int)
    của gói [`math/big`](/pkg/math/big/)
    bây giờ có các phương thức
    [`MarshalJSON`](/pkg/math/big/#Int.MarshalJSON)
    và
    [`UnmarshalJSON`](/pkg/math/big/#Int.UnmarshalJSON)
    để chuyển đổi sang và từ biểu diễn JSON.
    Ngoài ra,
    [`Int`](/pkg/math/big/#Int)
    bây giờ có thể chuyển đổi trực tiếp sang và từ `uint64` bằng
    [`Uint64`](/pkg/math/big/#Int.Uint64)
    và
    [`SetUint64`](/pkg/math/big/#Int.SetUint64),
    trong khi
    [`Rat`](/pkg/math/big/#Rat)
    có thể làm tương tự với `float64` bằng
    [`Float64`](/pkg/math/big/#Rat.Float64)
    và
    [`SetFloat64`](/pkg/math/big/#Rat.SetFloat64).
  - Gói [`mime/multipart`](/pkg/mime/multipart/)
    có phương thức mới cho
    [`Writer`](/pkg/mime/multipart/#Writer),
    [`SetBoundary`](/pkg/mime/multipart/#Writer.SetBoundary),
    để định nghĩa chuỗi phân cách boundary dùng để đóng gói đầu ra.
    [`Reader`](/pkg/mime/multipart/#Reader) bây giờ cũng
    giải mã trong suốt bất kỳ phần nào được mã hóa `quoted-printable` và xóa
    header `Content-Transfer-Encoding` khi làm vậy.
  - Hàm [`ListenUnixgram`](/pkg/net/#ListenUnixgram)
    của gói [`net`](/pkg/net/)
    đã thay đổi kiểu trả về: bây giờ nó trả về một
    [`UnixConn`](/pkg/net/#UnixConn)
    thay vì một
    [`UDPConn`](/pkg/net/#UDPConn), đây rõ ràng là một sai lầm trong Go 1.0.
    Vì thay đổi API này sửa một lỗi, nó được cho phép bởi các quy tắc tương thích Go 1.
  - Gói [`net`](/pkg/net/) bao gồm kiểu mới,
    [`Dialer`](/pkg/net/#Dialer), để cung cấp các tùy chọn cho
    [`Dial`](/pkg/net/#Dialer.Dial).
  - Gói [`net`](/pkg/net/) thêm hỗ trợ cho
    các địa chỉ IPv6 link-local với zone qualifier, chẳng hạn như `fe80::1%lo0`.
    Các cấu trúc địa chỉ [`IPAddr`](/pkg/net/#IPAddr),
    [`UDPAddr`](/pkg/net/#UDPAddr) và
    [`TCPAddr`](/pkg/net/#TCPAddr)
    ghi zone trong một field mới, và các hàm kỳ vọng dạng chuỗi của các địa chỉ này, chẳng hạn như
    [`Dial`](/pkg/net/#Dial),
    [`ResolveIPAddr`](/pkg/net/#ResolveIPAddr),
    [`ResolveUDPAddr`](/pkg/net/#ResolveUDPAddr) và
    [`ResolveTCPAddr`](/pkg/net/#ResolveTCPAddr),
    bây giờ chấp nhận dạng zone-qualified.
  - Gói [`net`](/pkg/net/) thêm
    [`LookupNS`](/pkg/net/#LookupNS) vào bộ các hàm phân giải.
    `LookupNS` trả về [các bản ghi NS](/pkg/net/#NS) cho một tên host.
  - Gói [`net`](/pkg/net/) thêm các phương thức đọc và ghi packet theo giao thức cho
    [`IPConn`](/pkg/net/#IPConn)
    ([`ReadMsgIP`](/pkg/net/#IPConn.ReadMsgIP)
    và [`WriteMsgIP`](/pkg/net/#IPConn.WriteMsgIP)) và
    [`UDPConn`](/pkg/net/#UDPConn)
    ([`ReadMsgUDP`](/pkg/net/#UDPConn.ReadMsgUDP) và
    [`WriteMsgUDP`](/pkg/net/#UDPConn.WriteMsgUDP)).
    Đây là các phiên bản chuyên biệt của các phương thức `ReadFrom` và `WriteTo` của [`PacketConn`](/pkg/net/#PacketConn)
    cung cấp truy cập vào dữ liệu out-of-band liên quan đến các packet.
  - Gói [`net`](/pkg/net/) thêm các phương thức cho
    [`UnixConn`](/pkg/net/#UnixConn) để cho phép đóng một nửa kết nối
    ([`CloseRead`](/pkg/net/#UnixConn.CloseRead) và
    [`CloseWrite`](/pkg/net/#UnixConn.CloseWrite)),
    khớp với các phương thức hiện có của [`TCPConn`](/pkg/net/#TCPConn).
  - Gói [`net/http`](/pkg/net/http/) bao gồm một số bổ sung mới.
    [`ParseTime`](/pkg/net/http/#ParseTime) phân tích cú pháp chuỗi thời gian, thử
    một số định dạng thời gian HTTP phổ biến.
    Phương thức [`PostFormValue`](/pkg/net/http/#Request.PostFormValue) của
    [`Request`](/pkg/net/http/#Request) giống như
    [`FormValue`](/pkg/net/http/#Request.FormValue) nhưng bỏ qua các tham số URL.
    Interface [`CloseNotifier`](/pkg/net/http/#CloseNotifier) cung cấp cơ chế
    cho một server handler để phát hiện khi nào client đã ngắt kết nối.
    Kiểu `ServeMux` bây giờ có phương thức
    [`Handler`](/pkg/net/http/#ServeMux.Handler) để truy cập `Handler` của một đường dẫn
    mà không thực thi nó.
    `Transport` bây giờ có thể hủy một yêu cầu đang xử lý bằng
    [`CancelRequest`](/pkg/net/http/#Transport.CancelRequest).
    Cuối cùng, Transport bây giờ tích cực hơn trong việc đóng các kết nối TCP khi
    một [`Response.Body`](/pkg/net/http/#Response) được đóng trước khi
    được tiêu thụ hoàn toàn.
  - Gói [`net/mail`](/pkg/net/mail/) có hai hàm mới,
    [`ParseAddress`](/pkg/net/mail/#ParseAddress) và
    [`ParseAddressList`](/pkg/net/mail/#ParseAddressList),
    để phân tích các địa chỉ mail được định dạng theo RFC 5322 thành
    các cấu trúc [`Address`](/pkg/net/mail/#Address).
  - Kiểu [`Client`](/pkg/net/smtp/#Client)
    của gói [`net/smtp`](/pkg/net/smtp/)
    có phương thức mới,
    [`Hello`](/pkg/net/smtp/#Client.Hello),
    truyền thông điệp `HELO` hoặc `EHLO` đến server.
  - Gói [`net/textproto`](/pkg/net/textproto/)
    có hai hàm mới,
    [`TrimBytes`](/pkg/net/textproto/#TrimBytes) và
    [`TrimString`](/pkg/net/textproto/#TrimString),
    thực hiện cắt chỉ ASCII của khoảng trắng đầu và cuối.
  - Phương thức mới [`os.FileMode.IsRegular`](/pkg/os/#FileMode.IsRegular) giúp dễ dàng hỏi liệu một file có phải là file thông thường không.
  - Gói [`os/signal`](/pkg/os/signal/) có hàm mới,
    [`Stop`](/pkg/os/signal/#Stop), dừng gói gửi
    bất kỳ tín hiệu nào thêm đến channel.
  - Gói [`regexp`](/pkg/regexp/)
    bây giờ hỗ trợ khớp từ trái sang phải dài nhất theo gốc Unix thông qua phương thức
    [`Regexp.Longest`](/pkg/regexp/#Regexp.Longest),
    trong khi
    [`Regexp.Split`](/pkg/regexp/#Regexp.Split) chia
    chuỗi thành các phần dựa trên các phân cách được định nghĩa bởi biểu thức chính quy.
  - Gói [`runtime/debug`](/pkg/runtime/debug/)
    có ba hàm mới liên quan đến sử dụng bộ nhớ.
    Hàm [`FreeOSMemory`](/pkg/runtime/debug/#FreeOSMemory)
    kích hoạt một lần chạy của bộ gom rác và sau đó cố gắng trả lại bộ nhớ không dùng
    cho hệ điều hành;
    hàm [`ReadGCStats`](/pkg/runtime/debug/#ReadGCStats)
    truy xuất thống kê về bộ gom rác; và
    [`SetGCPercent`](/pkg/runtime/debug/#SetGCPercent)
    cung cấp một cách lập trình để kiểm soát tần suất bộ gom rác chạy,
    bao gồm tắt hoàn toàn.
  - Gói [`sort`](/pkg/sort/) có hàm mới,
    [`Reverse`](/pkg/sort/#Reverse).
    Bọc đối số của một lệnh gọi đến
    [`sort.Sort`](/pkg/sort/#Sort)
    bằng một lệnh gọi đến `Reverse` làm cho thứ tự sắp xếp bị đảo ngược.
  - Gói [`strings`](/pkg/strings/) có hai hàm mới,
    [`TrimPrefix`](/pkg/strings/#TrimPrefix)
    và
    [`TrimSuffix`](/pkg/strings/#TrimSuffix)
    với các thuộc tính tự giải thích, và phương thức mới
    [`Reader.WriteTo`](/pkg/strings/#Reader.WriteTo) để kiểu
    [`Reader`](/pkg/strings/#Reader)
    bây giờ triển khai interface
    [`io.WriterTo`](/pkg/io/#WriterTo).
  - Hàm [`Fchflags`](/pkg/syscall/#Fchflags)
    của gói [`syscall`](/pkg/syscall/)
    trên nhiều BSD (bao gồm Darwin) đã thay đổi signature.
    Bây giờ nó nhận một int làm tham số đầu tiên thay vì một chuỗi.
    Vì thay đổi API này sửa một lỗi, nó được cho phép bởi các quy tắc tương thích Go 1.
  - Gói [`syscall`](/pkg/syscall/) cũng đã nhận được nhiều cập nhật
    để làm cho nó toàn diện hơn về hằng số và system call cho mỗi hệ điều hành được hỗ trợ.
  - Gói [`testing`](/pkg/testing/) bây giờ tự động hóa việc tạo
    thống kê cấp phát trong các test và benchmark bằng hàm mới
    [`AllocsPerRun`](/pkg/testing/#AllocsPerRun). Và phương thức
    [`ReportAllocs`](/pkg/testing/#B.ReportAllocs)
    trên [`testing.B`](/pkg/testing/#B) sẽ bật in
    thống kê cấp phát bộ nhớ cho benchmark đang gọi. Nó cũng giới thiệu
    phương thức [`AllocsPerOp`](/pkg/testing/#BenchmarkResult.AllocsPerOp) của
    [`BenchmarkResult`](/pkg/testing/#BenchmarkResult).
    Ngoài ra còn có hàm
    [`Verbose`](/pkg/testing/#Verbose) mới để kiểm tra trạng thái của flag
    dòng lệnh `-v`,
    và phương thức mới
    [`Skip`](/pkg/testing/#B.Skip) của
    [`testing.B`](/pkg/testing/#B) và
    [`testing.T`](/pkg/testing/#T)
    để đơn giản hóa việc bỏ qua một test không phù hợp.
  - Trong các gói [`text/template`](/pkg/text/template/)
    và
    [`html/template`](/pkg/html/template/),
    các template bây giờ có thể dùng dấu ngoặc đơn để nhóm các phần tử của pipeline, đơn giản hóa việc xây dựng các pipeline phức tạp.
    Ngoài ra, như một phần của parser mới, interface
    [`Node`](/pkg/text/template/parse/#Node) có thêm hai phương thức để cung cấp
    báo cáo lỗi tốt hơn.
    Mặc dù điều này vi phạm các quy tắc tương thích Go 1,
    không có mã nguồn hiện có nào nên bị ảnh hưởng vì interface này được dự định rõ ràng chỉ được dùng
    bởi các gói
    [`text/template`](/pkg/text/template/)
    và
    [`html/template`](/pkg/html/template/)
    và có các biện pháp bảo vệ để đảm bảo điều đó.
  - Triển khai gói [`unicode`](/pkg/unicode/) đã được cập nhật lên Unicode phiên bản 6.2.0.
  - Trong gói [`unicode/utf8`](/pkg/unicode/utf8/),
    hàm mới [`ValidRune`](/pkg/unicode/utf8/#ValidRune) báo cáo liệu rune có phải là điểm mã Unicode hợp lệ không.
