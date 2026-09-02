---
path: /doc/go1.17
title: Ghi chú phát hành Go 1.17
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

## Giới thiệu về Go 1.17 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.17, ra đời sáu tháng sau [Go 1.16](/doc/go1.16).
Hầu hết các thay đổi đều nằm trong phần triển khai của toolchain, runtime và thư viện.
Như thường lệ, bản phát hành này duy trì [cam kết tương thích](/doc/go1compat) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước đây.

## Thay đổi về ngôn ngữ {#language}

Go 1.17 bổ sung ba cải tiến nhỏ cho ngôn ngữ.

  - <!-- CL 216424; issue 395 -->
    [Chuyển đổi
    từ slice sang con trỏ mảng](/ref/spec#Conversions_from_slice_to_array_pointer): Biểu thức `s` có
    kiểu `[]T` hiện có thể được chuyển đổi sang kiểu con trỏ mảng
    `*[N]T`. Nếu `a` là kết quả của phép chuyển đổi như vậy, thì các chỉ mục tương ứng trong phạm vi sẽ tham chiếu
    đến cùng các phần tử nền: `&a[i] == &s[i]`
    với `0 <= i < N`. Phép chuyển đổi gây panic nếu
    `len(s)` nhỏ hơn `N`.
  - <!-- CL 312212; issue 40481 -->
    [`unsafe.Add`](/pkg/unsafe#Add):
    `unsafe.Add(ptr, len)` cộng `len`
    vào `ptr` và trả về con trỏ đã cập nhật
    `unsafe.Pointer(uintptr(ptr) + uintptr(len))`.
  - <!-- CL 312212; issue 19367 -->
    [`unsafe.Slice`](/pkg/unsafe#Slice):
    Với biểu thức `ptr` có kiểu `*T`,
    `unsafe.Slice(ptr, len)` trả về một slice có
    kiểu `[]T` mà mảng nền bắt đầu
    tại `ptr` và có độ dài và dung lượng
    là `len`.

Các cải tiến trong package unsafe được thêm vào để đơn giản hóa việc viết code tuân thủ
[các quy tắc an toàn](/pkg/unsafe/#Pointer) của `unsafe.Pointer`, nhưng bản thân các quy tắc vẫn không thay đổi. Cụ thể, các
chương trình hiện có sử dụng đúng `unsafe.Pointer` vẫn
hợp lệ, và các chương trình mới vẫn phải tuân theo các quy tắc khi
sử dụng `unsafe.Add` hoặc `unsafe.Slice`.

Lưu ý rằng phép chuyển đổi mới từ slice sang con trỏ mảng là
trường hợp đầu tiên trong đó một phép chuyển đổi kiểu có thể gây panic lúc chạy chương trình.
Các công cụ phân tích giả định rằng phép chuyển đổi kiểu không bao giờ gây panic
nên được cập nhật để xem xét khả năng này.

## Các nền tảng {#ports}

### Darwin {#darwin}

<!-- golang.org/issue/23011 -->
Như đã [thông báo](go1.16#darwin) trong ghi chú phát hành Go 1.16,
Go 1.17 yêu cầu macOS 10.13 High Sierra trở lên; hỗ trợ
cho các phiên bản cũ hơn đã bị ngừng.

### Windows {#windows}

<!-- golang.org/issue/36439 -->
Go 1.17 bổ sung hỗ trợ kiến trúc ARM 64-bit trên Windows (cổng
`windows/arm64`). Cổng này hỗ trợ cgo.

### OpenBSD {#openbsd}

<!-- golang.org/issue/43005 -->
Kiến trúc MIPS 64-bit trên OpenBSD (cổng `openbsd/mips64`)
hiện hỗ trợ cgo.

<!-- golang.org/issue/36435 -->
Trong Go 1.16, trên các kiến trúc x86 64-bit và ARM 64-bit trên
OpenBSD (các cổng `openbsd/amd64` và `openbsd/arm64`),
các lời gọi hệ thống được thực hiện thông qua `libc`, thay vì
sử dụng trực tiếp các lệnh máy. Trong Go 1.17, điều này cũng
được áp dụng trên các kiến trúc x86 32-bit và ARM 32-bit trên OpenBSD
(các cổng `openbsd/386` và `openbsd/arm`).
Điều này đảm bảo tính tương thích với OpenBSD 6.9 trở lên, yêu cầu
các lời gọi hệ thống phải được thực hiện thông qua `libc` đối với
các nhị phân Go không static.

### ARM64 {#arm64}

<!-- CL 288814 -->
Các chương trình Go hiện duy trì stack frame pointer trên kiến trúc ARM
64-bit trên tất cả các hệ điều hành. Trước đây, stack frame
pointer chỉ được bật trên Linux, macOS và iOS.

### Giá trị GOARCH loong64 được đặt trước {#loong64}

<!-- CL 333909 -->
Trình biên dịch Go chính chưa hỗ trợ kiến trúc LoongArch,
nhưng chúng tôi đã đặt trước giá trị `GOARCH`
"`loong64`".
Điều này có nghĩa là các file Go có tên `*_loong64.go` sẽ
bị [bỏ qua bởi các công cụ Go](/pkg/go/build/#hdr-Build_Constraints) trừ khi giá trị GOARCH đó đang được sử dụng.

## Công cụ {#tools}

### Lệnh go {#go-command}

<a id="lazy-loading"><!-- for existing links only -->
</a>

#### Đồ thị module được cắt tỉa trong các module `go 1.17` {#graph-pruning}

<!-- golang.org/issue/36460 -->
Nếu một module chỉ định `go` `1.17` hoặc cao hơn, đồ thị module
chỉ bao gồm các dependency _trực tiếp_ của
các module `go` `1.17` khác, không phải toàn bộ các dependency bắc cầu của chúng.
(Xem [Cắt tỉa đồ thị module](/ref/mod#graph-pruning)
để biết thêm chi tiết.)

Để lệnh `go` có thể giải quyết chính xác các import bắc cầu bằng
đồ thị module được cắt tỉa, file `go.mod` cho mỗi module cần
bao gồm nhiều thông tin hơn về các dependency bắc cầu liên quan đến module đó.
Nếu một module chỉ định `go` `1.17` hoặc cao hơn trong file
`go.mod` của nó, file `go.mod` đó sẽ chứa một
chỉ thị [`require`](/ref/mod#go-mod-file-require) tường minh cho mỗi module cung cấp một package được import bắc cầu.
(Trong các phiên bản trước, file `go.mod` thường chỉ bao gồm
các yêu cầu tường minh cho các package được import _trực tiếp_.)


Vì file `go.mod` mở rộng cần thiết cho việc cắt tỉa đồ thị module
bao gồm tất cả các dependency cần thiết để tải các import của bất kỳ package nào trong
module chính, nếu module chính chỉ định
`go` `1.17` hoặc cao hơn, công cụ `go` sẽ không còn
đọc (hoặc thậm chí tải) các file `go.mod` cho các dependency nếu chúng
không cần thiết để hoàn thành lệnh được yêu cầu.
(Xem [Lazy loading](/ref/mod#lazy-loading).)

<!-- golang.org/issue/45965 -->
Vì số lượng yêu cầu tường minh có thể lớn hơn đáng kể trong một
file `go.mod` Go 1.17 mở rộng, các yêu cầu mới được thêm vào
đối với các dependency _gián tiếp_ trong một module `go` `1.17`
được duy trì trong một khối `require` riêng biệt với khối
chứa các dependency trực tiếp.

<!-- golang.org/issue/45094 -->
Để thuận tiện cho việc nâng cấp lên đồ thị module được cắt tỉa Go 1.17,
lệnh con [`go` `mod` `tidy`](/ref/mod#go-mod-tidy)
hiện hỗ trợ cờ `-go` để đặt hoặc thay đổi
phiên bản `go` trong file `go.mod`. Để chuyển đổi
file `go.mod` cho một module hiện có sang Go 1.17 mà không
thay đổi các phiên bản đã chọn của các dependency, hãy chạy:

	  go mod tidy -go=1.17

<!-- golang.org/issue/46141 -->
Theo mặc định, `go` `mod` `tidy` xác minh rằng
các phiên bản đã chọn của các dependency liên quan đến module chính là những phiên bản tương tự
sẽ được sử dụng bởi bản phát hành Go trước (Go 1.16 đối với module chỉ định
`go` `1.17`), và duy trì
các mục `go.sum` cần thiết cho bản phát hành đó ngay cả đối với các dependency
thường không cần thiết bởi các lệnh khác.

Cờ `-compat` cho phép ghi đè phiên bản đó để hỗ trợ
các phiên bản cũ hơn (hoặc chỉ mới hơn), tối đa đến phiên bản được chỉ định bởi
chỉ thị `go` trong file `go.mod`. Để tidy
một module `go` `1.17` chỉ dành cho Go 1.17, mà không lưu
checksum (hoặc kiểm tra tính nhất quán với) Go 1.16:

	  go mod tidy -compat=1.17

Lưu ý rằng ngay cả khi module chính được tidy với `-compat=1.17`,
người dùng `require` module từ một
module `go` `1.16` hoặc cũ hơn vẫn có thể
sử dụng nó, miễn là các package chỉ sử dụng các tính năng ngôn ngữ và thư viện tương thích.

<!-- golang.org/issue/46366 -->
Lệnh con [`go` `mod` `graph`](/ref/mod#go-mod-graph)
cũng hỗ trợ cờ `-go`, khiến nó báo cáo
đồ thị như được nhìn thấy bởi phiên bản Go được chỉ định, hiển thị các dependency có thể
bị cắt tỉa đi.

#### Chú thích deprecation trong module {#module-deprecation-comments}

<!-- golang.org/issue/40357 -->
Tác giả module có thể đánh dấu module là deprecated bằng cách thêm một
[chú thích `// Deprecated:`](/ref/mod#go-mod-file-module-deprecation) vào `go.mod`, sau đó gắn tag phiên bản mới.
`go` `get` hiện in cảnh báo nếu một module cần thiết để
build các package được đặt tên trên dòng lệnh bị deprecated. `go`
`list` `-m` `-u` in thông tin deprecated cho tất cả
các dependency (sử dụng `-f` hoặc `-json` để hiển thị đầy đủ
thông báo). Lệnh `go` coi các phiên bản chính khác nhau là
các module riêng biệt, vì vậy cơ chế này có thể được sử dụng, ví dụ, để cung cấp
cho người dùng hướng dẫn migration cho một phiên bản chính mới.

#### `go` `get` {#go-get}

<!-- golang.org/issue/37519 -->
Cờ `go` `get` `-insecure` đã bị deprecated và đã được xóa. Để cho phép sử dụng các scheme không bảo mật
khi tải các dependency, hãy sử dụng biến môi trường `GOINSECURE`.
Cờ `-insecure` cũng bỏ qua xác nhận tổng kiểm tra module, hãy sử dụng `GOPRIVATE` hoặc `GONOSUMDB` nếu
bạn cần chức năng đó. Xem `go` `help`
`environment` để biết chi tiết.

<!-- golang.org/issue/43684 -->
`go` `get` in cảnh báo deprecated khi cài đặt
các lệnh bên ngoài module chính (không có cờ `-d`).
Thay vào đó nên sử dụng `go` `install` `cmd@version`
để cài đặt một lệnh ở phiên bản cụ thể, sử dụng hậu tố như
`@latest` hoặc `@v1.2.3`. Trong Go 1.18, cờ `-d`
sẽ luôn được bật, và `go` `get` sẽ chỉ
được dùng để thay đổi các dependency trong `go.mod`.

#### Các file `go.mod` thiếu chỉ thị `go` {#missing-go-directive}

<!-- golang.org/issue/44976 -->
Nếu file `go.mod` của module chính không chứa
[chỉ thị `go`](/doc/modules/gomod-ref#go) và
lệnh `go` không thể cập nhật file `go.mod`, lệnh
`go` hiện giả định `go 1.11` thay vì bản phát hành hiện tại.
(`go` `mod` `init` đã thêm
các chỉ thị `go` tự động [kể từ
Go 1.12](/doc/go1.12#modules).)

<!-- golang.org/issue/44976 -->
Nếu một dependency module thiếu file `go.mod` tường minh, hoặc
file `go.mod` của nó không chứa
[chỉ thị `go`](/doc/modules/gomod-ref#go),
lệnh `go` hiện giả định `go 1.16` cho
dependency đó thay vì bản phát hành hiện tại. (Các dependency được phát triển trong chế độ
GOPATH có thể thiếu file `go.mod`, và
`vendor/modules.txt` cho đến nay chưa bao giờ ghi lại
các phiên bản `go` được chỉ định bởi các file `go.mod` của dependency.)

#### Nội dung `vendor` {#vendor}

<!-- golang.org/issue/36876 -->
Nếu module chính chỉ định `go` `1.17` hoặc cao hơn,
[`go` `mod` `vendor`](/ref/mod#go-mod-vendor)
hiện chú thích
`vendor/modules.txt` với phiên bản `go` được chỉ định bởi
mỗi module được vendor trong file `go.mod` riêng của nó. Phiên bản được chú thích được sử dụng
khi build các package của module từ mã nguồn vendored.

<!-- golang.org/issue/42970 -->
Nếu module chính chỉ định `go` `1.17` hoặc cao hơn,
`go` `mod` `vendor` hiện bỏ qua các file `go.mod`
và `go.sum` cho các dependency được vendor, vì chúng có thể gây
cản trở khả năng của lệnh `go` khi xác định đúng
thư mục gốc của module khi được gọi trong cây `vendor`.

#### Nhắc nhập mật khẩu {#password-prompts}

<!-- golang.org/issue/44904 -->
Lệnh `go` theo mặc định hiện ngăn chặn các lời nhắc mật khẩu SSH và
các lời nhắc Git Credential Manager khi tải các kho lưu trữ Git bằng SSH, cũng như
nó đã làm trước đây đối với các lời nhắc mật khẩu Git khác. Người dùng xác thực đến
các kho Git riêng tư với SSH được bảo vệ bằng mật khẩu có thể cấu hình
`ssh-agent` để cho phép lệnh `go` sử dụng
các khóa SSH được bảo vệ bằng mật khẩu.

#### `go` `mod` `download` {#go-mod-download}

<!-- golang.org/issue/45332 -->
Khi `go` `mod` `download` được gọi mà không có
đối số, nó sẽ không còn lưu tổng kiểm tra cho nội dung module đã tải về vào
`go.sum`. Nó vẫn có thể thực hiện các thay đổi đối với `go.mod` và
`go.sum` cần thiết để tải danh sách build. Đây là hành vi tương tự như trong Go 1.15. Để lưu tổng kiểm tra cho tất cả các module, hãy sử dụng `go`
`mod` `download` `all`.

#### Dòng `//go:build` {#build-lines}

Lệnh `go` hiện hiểu các dòng `//go:build`
và ưu tiên chúng hơn các dòng `// +build`. Cú pháp mới sử dụng
các biểu thức boolean, giống như Go, và ít có nguy cơ mắc lỗi hơn.
Kể từ bản phát hành này, cú pháp mới được hỗ trợ đầy đủ, và tất cả các file Go
nên được cập nhật để có cả hai dạng với cùng ý nghĩa. Để hỗ trợ việc
migration, [`gofmt`](#gofmt) hiện tự động
đồng bộ hóa hai dạng. Để biết thêm chi tiết về cú pháp và kế hoạch migration, xem
[https://golang.org/design/draft-gobuild](/design/draft-gobuild).

#### `go` `run` {#go_run}

<!-- golang.org/issue/42088 -->
`go` `run` hiện chấp nhận các đối số có hậu tố phiên bản
(ví dụ: `go` `run`
`example.com/cmd@v1.0.0`). Điều này khiến `go`
`run` build và chạy các package trong chế độ nhận biết module, bỏ qua
file `go.mod` trong thư mục hiện tại hoặc bất kỳ thư mục cha nào, nếu
có. Điều này hữu ích để chạy các tệp thực thi mà không cần cài đặt chúng hoặc
không thay đổi các dependency của module hiện tại.

### Gofmt {#gofmt}

`gofmt` (và `go` `fmt`) hiện đồng bộ hóa
các dòng `//go:build` với các dòng `// +build`. Nếu một file
chỉ có các dòng `// +build`, chúng sẽ được di chuyển đến vị trí phù hợp
trong file, và các dòng `//go:build` tương ứng sẽ được
thêm vào. Nếu không, các dòng `// +build` sẽ bị ghi đè dựa trên
bất kỳ dòng `//go:build` hiện có nào. Để biết thêm thông tin, xem
[https://golang.org/design/draft-gobuild](/design/draft-gobuild).

### Vet {#vet}

#### Cảnh báo mới cho các dòng `//go:build` và `// +build` không khớp {#vet-buildtags}

<!-- CL 240609 -->
Công cụ `vet` hiện xác minh rằng các dòng `//go:build` và
`// +build` nằm ở phần đúng của file và
đồng bộ với nhau. Nếu không,
có thể sử dụng [`gofmt`](#gofmt) để sửa chúng. Để biết thêm
thông tin, xem
[https://golang.org/design/draft-gobuild](/design/draft-gobuild).

#### Cảnh báo mới cho việc gọi `signal.Notify` trên channel không có buffer {#vet-sigchanyzer}

<!-- CL 299532 -->
Công cụ vet hiện cảnh báo về các lời gọi đến [signal.Notify](/pkg/os/signal/#Notify)
với các tín hiệu đến được gửi đến một channel không có buffer. Sử dụng channel không có buffer
có nguy cơ bỏ lỡ các tín hiệu được gửi đến chúng vì `signal.Notify` không chặn khi
gửi đến một channel. Ví dụ:

	c := make(chan os.Signal)
	// signals are sent on c before the channel is read from.
	// This signal may be dropped as c is unbuffered.
	signal.Notify(c, os.Interrupt)

Người dùng `signal.Notify` nên sử dụng các channel có đủ dung lượng buffer để theo kịp
tốc độ tín hiệu dự kiến.

#### Cảnh báo mới cho các phương thức Is, As và Unwrap {#vet-error-stdmethods}

<!-- CL 321389 -->
Công cụ vet hiện cảnh báo về các phương thức có tên `As`, `Is` hoặc `Unwrap`
trên các kiểu triển khai interface `error` mà có chữ ký khác với
chữ ký mà package `errors` mong đợi. Các hàm `errors.{As,Is,Unwrap}` mong đợi các phương thức như vậy triển khai `Is(error)` `bool`,
`As(interface{})` `bool`, hoặc `Unwrap()` `error`
tương ứng. Các hàm `errors.{As,Is,Unwrap}` sẽ bỏ qua các phương thức có cùng
tên nhưng chữ ký khác. Ví dụ:

	type MyError struct { hint string }
	func (m MyError) Error() string { ... } // MyError implements error.
	func (MyError) Is(target interface{}) bool { ... } // target is interface{} instead of error.
	func Foo() bool {
		x, y := MyError{"A"}, MyError{"B"}
		return errors.Is(x, y) // returns false as x != y and MyError does not have an `Is(error) bool` function.
	}

### Cover {#cover}

<!-- CL 249759 -->
Công cụ `cover` hiện sử dụng một trình phân tích tối ưu hóa
từ `golang.org/x/tools/cover`, có thể nhanh hơn đáng kể
khi phân tích các profile coverage lớn.

## Trình biên dịch {#compiler}

<!-- golang.org/issue/40724 -->
Go 1.17 triển khai một cách mới để truyền đối số và kết quả hàm bằng
register thay vì stack.
Các benchmark cho một tập hợp đại diện các package và chương trình Go cho thấy
cải thiện hiệu suất khoảng 5%, và mức giảm thông thường trong
kích thước nhị phân khoảng 2%.
Hiện tại điều này được bật cho Linux, macOS và Windows trên
kiến trúc x86 64-bit (các cổng `linux/amd64`,
`darwin/amd64` và `windows/amd64`).

Thay đổi này không ảnh hưởng đến chức năng của bất kỳ code Go an toàn nào
và được thiết kế để không tác động đến hầu hết code assembly.
Nó có thể ảnh hưởng đến code vi phạm
các [quy tắc `unsafe.Pointer`](/pkg/unsafe#Pointer)
khi truy cập các đối số hàm, hoặc phụ thuộc vào
hành vi không có tài liệu liên quan đến việc so sánh các con trỏ code hàm.
Để duy trì khả năng tương thích với các hàm assembly hiện có, trình
biên dịch tạo ra các hàm adapter chuyển đổi giữa quy ước gọi hàm dựa trên register mới và quy ước gọi hàm dựa trên stack trước đây.
Các adapter này thường không hiển thị với người dùng, ngoại trừ việc lấy
địa chỉ của một hàm Go trong code assembly hoặc lấy địa chỉ
của một hàm assembly trong code Go
bằng cách sử dụng `reflect.ValueOf(fn).Pointer()`
hoặc `unsafe.Pointer` sẽ trả về địa chỉ của
adapter.
Code phụ thuộc vào giá trị của các con trỏ code này có thể không còn
hoạt động như mong đợi.
Các adapter cũng có thể gây ra một chi phí hiệu suất rất nhỏ trong hai
trường hợp: gọi một hàm assembly gián tiếp từ Go qua
giá trị `func`, và gọi các hàm Go từ assembly.

<!-- CL 304470 -->
Định dạng stack trace từ runtime (được in khi một panic không được bắt
xảy ra, hoặc khi `runtime.Stack` được gọi) được cải thiện. Trước đây,
các đối số hàm được in dưới dạng các từ thập lục phân dựa trên bố cục bộ nhớ.
Bây giờ mỗi đối số trong mã nguồn được in riêng lẻ, được phân tách
bằng dấu phẩy. Các đối số kiểu tổng hợp (struct, array, string, slice, interface và complex)
được phân tách bằng dấu ngoặc nhọn. Một lưu ý là giá trị của một
đối số chỉ tồn tại trong một register và không được lưu vào bộ nhớ có thể
không chính xác. Các giá trị trả về của hàm (thường không chính xác) không còn
được in nữa.

<!-- CL 283112, golang.org/issue/28727 -->
Các hàm chứa closure hiện có thể được inline.
Một tác dụng của thay đổi này là một hàm có closure có thể
tạo ra một con trỏ code closure riêng biệt cho mỗi vị trí mà hàm
được inline.
Các giá trị hàm Go không thể so sánh trực tiếp, nhưng thay đổi này
có thể phát hiện lỗi trong code sử dụng `reflect`
hoặc `unsafe.Pointer` để bỏ qua hạn chế ngôn ngữ này
và so sánh các hàm bằng con trỏ code.

### Linker {#link}

<!-- CL 310349 -->
Khi linker sử dụng chế độ liên kết ngoài, đây là mặc định
khi liên kết một chương trình sử dụng cgo, và linker được gọi
với tùy chọn `-I`, tùy chọn đó sẽ được truyền đến
linker ngoài dưới dạng tùy chọn `-Wl,--dynamic-linker`.

## Thư viện chuẩn {#library}

### [Cgo](/pkg/runtime/cgo) {#runtime_cgo}

Package [runtime/cgo](/pkg/runtime/cgo) hiện cung cấp một
cơ chế mới cho phép chuyển đổi bất kỳ giá trị Go nào sang biểu diễn an toàn
có thể được dùng để truyền các giá trị giữa C và Go một cách an toàn. Xem
[runtime/cgo.Handle](/pkg/runtime/cgo#Handle) để biết thêm thông tin.

### Phân tích URL query {#semicolons}

<!-- CL 325697, CL 326309 -->

Các package `net/url` và `net/http` trước đây chấp nhận
`";"` (dấu chấm phẩy) như một dấu phân cách cài đặt trong URL query,
ngoài `"&"` (ký hiệu và). Bây giờ, các cài đặt với dấu chấm phẩy không được mã hóa phần trăm bị từ chối và các server `net/http` sẽ ghi một cảnh báo vào
[`Server.ErrorLog`](/pkg/net/http#Server.ErrorLog)
khi gặp một trong URL yêu cầu.

Ví dụ, trước Go 1.17, phương thức [`Query`](/pkg/net/url#URL.Query)
của URL `example?a=1;b=2&c=3` sẽ trả về
`map[a:[1] b:[2] c:[3]]`, trong khi bây giờ nó trả về `map[c:[3]]`.

Khi gặp một query string như vậy,
[`URL.Query`](/pkg/net/url#URL.Query)
và
[`Request.FormValue`](/pkg/net/http#Request.FormValue)
bỏ qua bất kỳ cài đặt nào chứa dấu chấm phẩy,
[`ParseQuery`](/pkg/net/url#ParseQuery)
trả về các cài đặt còn lại và một lỗi, và
[`Request.ParseForm`](/pkg/net/http#Request.ParseForm)
và
[`Request.ParseMultipartForm`](/pkg/net/http#Request.ParseMultipartForm)
trả về một lỗi nhưng vẫn thiết lập các trường `Request` dựa trên
các cài đặt còn lại.

Người dùng `net/http` có thể khôi phục hành vi ban đầu bằng cách sử dụng bộ bao handler
[`AllowQuerySemicolons`](/pkg/net/http#AllowQuerySemicolons) mới.
Điều này cũng sẽ ngăn cảnh báo `ErrorLog`.
Lưu ý rằng việc chấp nhận dấu chấm phẩy như dấu phân cách query có thể dẫn đến các vấn đề bảo mật
nếu các hệ thống khác nhau diễn giải cache key theo cách khác nhau.
Xem [vấn đề 25192](/issue/25192) để biết thêm thông tin.

### TLS strict ALPN {#ALPN}

<!-- CL 289209, CL 325432 -->

Khi [`Config.NextProtos`](/pkg/crypto/tls#Config.NextProtos)
được đặt, các server hiện thực thi rằng có sự trùng lặp giữa các
giao thức được cấu hình và các giao thức ALPN được quảng cáo bởi client, nếu có.
Nếu không có giao thức nào được hỗ trợ lẫn nhau, kết nối sẽ bị đóng với cảnh báo
`no_application_protocol`, theo yêu cầu của RFC 7301. Điều này
giúp giảm thiểu [cuộc tấn công cross-protocol ALPACA](https://alpaca-attack.com/).

Ngoại lệ, khi giá trị `"h2"` được bao gồm trong
`Config.NextProtos` của server, các client HTTP/1.1 sẽ được phép kết nối như
thể chúng không hỗ trợ ALPN.
Xem [vấn đề 46310](/issue/46310) để biết thêm thông tin.

### Các thay đổi nhỏ trong thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ cho thư viện,
được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1 trong tâm trí.

#### [archive/zip](/pkg/archive/zip/)

<!-- CL 312310 -->
Các phương thức mới [`File.OpenRaw`](/pkg/archive/zip#File.OpenRaw), [`Writer.CreateRaw`](/pkg/archive/zip#Writer.CreateRaw), [`Writer.Copy`](/pkg/archive/zip#Writer.Copy) cung cấp hỗ trợ cho các trường hợp mà hiệu suất là mối quan tâm hàng đầu.

<!-- archive/zip -->

#### [bufio](/pkg/bufio/)

<!-- CL 280492 -->
Phương thức [`Writer.WriteRune`](/pkg/bufio/#Writer.WriteRune)
hiện ghi ký tự thay thế U+FFFD cho các giá trị rune âm,
như nó làm đối với các rune không hợp lệ khác.

<!-- bufio -->

#### [bytes](/pkg/bytes/)

<!-- CL 280492 -->
Phương thức [`Buffer.WriteRune`](/pkg/bytes/#Buffer.WriteRune)
hiện ghi ký tự thay thế U+FFFD cho các giá trị rune âm,
như nó làm đối với các rune không hợp lệ khác.

<!-- bytes -->

#### [compress/lzw](/pkg/compress/lzw/)

<!-- CL 273667 -->
Hàm [`NewReader`](/pkg/compress/lzw/#NewReader)
được đảm bảo trả về một giá trị của kiểu mới
[`Reader`](/pkg/compress/lzw/#Reader),
và tương tự [`NewWriter`](/pkg/compress/lzw/#NewWriter)
được đảm bảo trả về một giá trị của kiểu mới
[`Writer`](/pkg/compress/lzw/#Writer).
Cả hai kiểu mới này đều triển khai phương thức `Reset`
([`Reader.Reset`](/pkg/compress/lzw/#Reader.Reset),
[`Writer.Reset`](/pkg/compress/lzw/#Writer.Reset))
cho phép tái sử dụng `Reader` hoặc `Writer`.

<!-- compress/lzw -->

#### [crypto/ed25519](/pkg/crypto/ed25519/)

<!-- CL 276272 -->
Package `crypto/ed25519` đã được viết lại, và tất cả
các thao tác hiện nhanh gấp khoảng hai lần trên amd64 và arm64.
Hành vi quan sát được không thay đổi khác.

<!-- crypto/ed25519 -->

#### [crypto/elliptic](/pkg/crypto/elliptic/)

<!-- CL 233939 -->
Các phương thức [`CurveParams`](/pkg/crypto/elliptic#CurveParams)
hiện tự động gọi các triển khai chuyên dụng nhanh hơn và an toàn hơn
cho các đường cong đã biết (P-224, P-256 và P-521) khi
có sẵn. Lưu ý đây là cách tiếp cận nỗ lực tốt nhất và các ứng dụng
nên tránh sử dụng các phương thức `CurveParams` chung, không có thời gian cố định và thay vào đó sử dụng các
triển khai [`Curve`](/pkg/crypto/elliptic#Curve) chuyên dụng
như [`P256`](/pkg/crypto/elliptic#P256).

<!-- CL 315271, CL 315274 -->
Triển khai đường cong [`P521`](/pkg/crypto/elliptic#P521)
đã được viết lại sử dụng code được tạo ra bởi
[dự án fiat-crypto](https://github.com/mit-plv/fiat-crypto),
dựa trên một mô hình được xác minh chính thức về các
thao tác số học. Nó hiện có thời gian cố định và nhanh gấp ba lần trên amd64 và
arm64. Hành vi quan sát được không thay đổi khác.

<!-- crypto/elliptic -->

#### [crypto/rand](/pkg/crypto/rand/)

<!-- CL 302489, CL 299134, CL 269999 -->
Package `crypto/rand` hiện sử dụng syscall `getentropy`
trên macOS và syscall `getrandom` trên Solaris,
Illumos và DragonFlyBSD.

<!-- crypto/rand -->

#### [crypto/tls](/pkg/crypto/tls/)

<!-- CL 295370 -->
Phương thức mới [`Conn.HandshakeContext`](/pkg/crypto/tls#Conn.HandshakeContext)
cho phép người dùng kiểm soát việc hủy một quá trình bắt tay TLS đang diễn ra.
Context được cung cấp có thể truy cập từ các callback khác nhau thông qua các phương thức mới
[`ClientHelloInfo.Context`](/pkg/crypto/tls#ClientHelloInfo.Context) và
[`CertificateRequestInfo.Context`](/pkg/crypto/tls#CertificateRequestInfo.Context).
Hủy context sau khi bắt tay hoàn tất sẽ không có hiệu lực.

<!-- CL 314609 -->
Thứ tự cipher suite hiện được xử lý hoàn toàn bởi
package `crypto/tls`. Hiện tại, các cipher suite được sắp xếp dựa trên
bảo mật, hiệu suất và hỗ trợ phần cứng của chúng, tính đến
cả phần cứng cục bộ và của peer. Thứ tự của trường
[`Config.CipherSuites`](/pkg/crypto/tls#Config.CipherSuites)
hiện bị bỏ qua, cũng như trường
[`Config.PreferServerCipherSuites`](/pkg/crypto/tls#Config.PreferServerCipherSuites).
Lưu ý rằng `Config.CipherSuites` vẫn cho phép
các ứng dụng chọn các cipher suite TLS 1.0-1.2 nào để bật.

Các cipher suite 3DES đã được chuyển sang
[`InsecureCipherSuites`](/pkg/crypto/tls#InsecureCipherSuites)
do [điểm yếu cơ bản liên quan đến kích thước khối](https://sweet32.info/).
Chúng vẫn được bật theo mặc định nhưng chỉ như phương án cuối cùng,
nhờ vào thay đổi thứ tự cipher suite ở trên.

<!-- golang.org/issue/45428 -->
Bắt đầu từ bản phát hành tiếp theo, Go 1.18,
[`Config.MinVersion`](/pkg/crypto/tls/#Config.MinVersion)
cho các client `crypto/tls` sẽ mặc định là TLS 1.2, tắt TLS 1.0
và TLS 1.1 theo mặc định. Các ứng dụng sẽ có thể ghi đè thay đổi bằng cách
đặt tường minh `Config.MinVersion`.
Điều này sẽ không ảnh hưởng đến các server `crypto/tls`.

<!-- crypto/tls -->

#### [crypto/x509](/pkg/crypto/x509/)

<!-- CL 224157 -->
[`CreateCertificate`](/pkg/crypto/x509/#CreateCertificate)
hiện trả về lỗi nếu khóa riêng tư được cung cấp không khớp với
khóa công khai của parent, nếu có. Chứng chỉ kết quả sẽ không xác minh được.

<!-- CL 315209 -->
Cờ tạm thời `GODEBUG=x509ignoreCN=0` đã bị xóa.

<!-- CL 274234 -->
[`ParseCertificate`](/pkg/crypto/x509/#ParseCertificate)
đã được viết lại, và hiện tiêu thụ ít tài nguyên hơn ~70%. Hành vi quan sát được
khi xử lý chứng chỉ WebPKI không thay đổi khác,
ngoại trừ các thông báo lỗi.

<!-- CL 321190 -->
Trên các hệ thống BSD, `/etc/ssl/certs` hiện được tìm kiếm để tìm
các root được tin cậy. Điều này bổ sung hỗ trợ cho kho chứng chỉ được tin cậy hệ thống mới trong
FreeBSD 12.2+.

<!-- golang.org/issue/41682 -->
Bắt đầu từ bản phát hành tiếp theo, Go 1.18, `crypto/x509` sẽ
từ chối các chứng chỉ được ký bằng hàm băm SHA-1. Điều này không
áp dụng cho các chứng chỉ root tự ký. Các cuộc tấn công thực tế chống lại SHA-1
[đã được chứng minh vào năm 2017](https://shattered.io/) và
các Cơ quan chứng nhận được tin cậy công khai đã không cấp chứng chỉ SHA-1 kể từ năm 2015.

<!-- crypto/x509 -->

#### [database/sql](/pkg/database/sql/)

<!-- CL 258360 -->
Phương thức [`DB.Close`](/pkg/database/sql/#DB.Close) hiện đóng
trường `connector` nếu kiểu trong trường này triển khai
interface [`io.Closer`](/pkg/io/#Closer).

<!-- CL 311572 -->
Các struct mới
[`NullInt16`](/pkg/database/sql/#NullInt16)
và
[`NullByte`](/pkg/database/sql/#NullByte)
đại diện cho các giá trị int16 và byte có thể là null. Chúng có thể được sử dụng như
đích đến của phương thức [`Scan`](/pkg/database/sql/#Scan),
tương tự như NullString.

<!-- database/sql -->

#### [debug/elf](/pkg/debug/elf/)

<!-- CL 239217 -->
Hằng số [`SHT_MIPS_ABIFLAGS`](/pkg/debug/elf/#SHT_MIPS_ABIFLAGS)
đã được thêm vào.

<!-- debug/elf -->

#### [encoding/binary](/pkg/encoding/binary/)

<!-- CL 299531 -->
`binary.Uvarint` sẽ ngừng đọc sau `10 byte` để tránh
tính toán lãng phí. Nếu cần nhiều hơn `10 byte`, số byte được trả về là `-11`.
\
Các phiên bản Go trước đây có thể trả về các giá trị âm lớn hơn khi đọc các varint được mã hóa không đúng.

<!-- encoding/binary -->

#### [encoding/csv](/pkg/encoding/csv/)

<!-- CL 291290 -->
Phương thức mới
[`Reader.FieldPos`](/pkg/encoding/csv/#Reader.FieldPos)
trả về dòng và cột tương ứng với phần bắt đầu của
một trường cụ thể trong bản ghi được trả về gần đây nhất bởi
[`Read`](/pkg/encoding/csv/#Reader.Read).

<!-- encoding/csv -->

#### [encoding/xml](/pkg/encoding/xml/)

<!-- CL 277893 -->
Khi một chú thích xuất hiện trong
[`Directive`](/pkg/encoding/xml/#Directive), nó hiện được thay thế
bằng một khoảng trắng đơn thay vì bị xóa hoàn toàn.

Các tên phần tử hoặc thuộc tính không hợp lệ với các dấu hai chấm đứng đầu, cuối hoặc nhiều
hiện được lưu trữ không thay đổi vào trường
[`Name.Local`](/pkg/encoding/xml/#Name).

<!-- encoding/xml -->

#### [flag](/pkg/flag/)

<!-- CL 271788 -->
Các khai báo Flag hiện gây panic nếu tên không hợp lệ được chỉ định.

<!-- flag -->

#### [go/build](/pkg/go/build/)

<!-- CL 310732 -->
Trường mới
[`Context.ToolTags`](/pkg/go/build/#Context.ToolTags)
giữ các build tag phù hợp với cấu hình toolchain Go hiện tại.

<!-- go/build -->

#### [go/format](/pkg/go/format/)

Các hàm [`Source`](/pkg/go/format/#Source) và
[`Node`](/pkg/go/format/#Node) hiện
đồng bộ hóa các dòng `//go:build` với các dòng `// +build`.
Nếu một file chỉ có các dòng `// +build`, chúng sẽ được
di chuyển đến vị trí phù hợp trong file, và các dòng
`//go:build` tương ứng sẽ được thêm vào. Nếu không,
các dòng `// +build` sẽ bị ghi đè dựa trên bất kỳ dòng `//go:build` hiện có nào. Để biết thêm thông tin, xem
[https://golang.org/design/draft-gobuild](/design/draft-gobuild).

<!-- go/format -->

#### [go/parser](/pkg/go/parser/)

<!-- CL 306149 -->
Giá trị `Mode` mới [`SkipObjectResolution`](/pkg/go/parser/#SkipObjectResolution)
hướng dẫn parser không giải quyết các định danh đến
khai báo của chúng. Điều này có thể cải thiện tốc độ phân tích cú pháp.

<!-- go/parser -->

#### [image](/pkg/image/)

<!-- CL 311129 -->
Các kiểu image cụ thể (`RGBA`, `Gray16` và các kiểu khác)
hiện triển khai interface [`RGBA64Image`](/pkg/image/#RGBA64Image) mới.
Các kiểu cụ thể trước đây triển khai
[`draw.Image`](/pkg/image/draw/#Image) hiện cũng triển khai
[`draw.RGBA64Image`](/pkg/image/draw/#RGBA64Image), một
interface mới trong package `image/draw`.

<!-- image -->

#### [io/fs](/pkg/io/fs/)

<!-- CL 293649 -->
Hàm mới [`FileInfoToDirEntry`](/pkg/io/fs/#FileInfoToDirEntry) chuyển đổi `FileInfo` thành `DirEntry`.

<!-- io/fs -->

#### [math](/pkg/math/)

<!-- CL 247058 -->
Package math hiện định nghĩa thêm ba hằng số: `MaxUint`, `MaxInt` và `MinInt`.
Đối với các hệ thống 32-bit, các giá trị của chúng lần lượt là `2^32 - 1`, `2^31 - 1` và `-2^31`.
Đối với các hệ thống 64-bit, các giá trị của chúng lần lượt là `2^64 - 1`, `2^63 - 1` và `-2^63`.

<!-- math -->

#### [mime](/pkg/mime/)

<!-- CL 305230 -->
Trên các hệ thống Unix, bảng kiểu MIME hiện được đọc từ
[Cơ sở dữ liệu Shared MIME-info](https://specifications.freedesktop.org/shared-mime-info-spec/shared-mime-info-spec-0.21.html)
của hệ thống cục bộ khi có sẵn.

<!-- mime -->

#### [mime/multipart](/pkg/mime/multipart/)

<!-- CL 313809 -->
[`Part.FileName`](/pkg/mime/multipart/#Part.FileName)
hiện áp dụng
[`filepath.Base`](/pkg/path/filepath/#Base) cho
giá trị trả về. Điều này giảm thiểu các lỗ hổng path traversal tiềm ẩn trong
các ứng dụng chấp nhận các thông điệp multipart, chẳng hạn như các server `net/http` gọi
[`Request.FormFile`](/pkg/net/http/#Request.FormFile).

<!-- mime/multipart -->

#### [net](/pkg/net/)

<!-- CL 272668 -->
Phương thức mới [`IP.IsPrivate`](/pkg/net/#IP.IsPrivate) báo cáo liệu một địa chỉ có phải là
địa chỉ IPv4 riêng theo [RFC 1918](https://datatracker.ietf.org/doc/rfc1918)
hay là địa chỉ IPv6 cục bộ theo [RFC 4193](https://datatracker.ietf.org/doc/rfc4193) hay không.

<!-- CL 301709 -->
DNS resolver Go hiện chỉ gửi một truy vấn DNS khi giải quyết một địa chỉ cho mạng chỉ IPv4 hoặc chỉ IPv6,
thay vì truy vấn cho cả hai họ địa chỉ.

<!-- CL 307030 -->
Lỗi sentinel [`ErrClosed`](/pkg/net/#ErrClosed) và
kiểu lỗi [`ParseError`](/pkg/net/#ParseError) hiện triển khai
interface [`net.Error`](/pkg/net/#Error).

<!-- CL 325829 -->
Các hàm [`ParseIP`](/pkg/net/#ParseIP) và [`ParseCIDR`](/pkg/net/#ParseCIDR)
hiện từ chối các địa chỉ IPv4 chứa các thành phần thập phân có số không đứng đầu.
Các thành phần này luôn được diễn giải là thập phân, nhưng một số hệ điều hành coi chúng là bát phân.
Sự không khớp này về lý thuyết có thể dẫn đến các vấn đề bảo mật nếu một ứng dụng Go được dùng để xác thực địa chỉ IP
sau đó được sử dụng ở dạng ban đầu với các ứng dụng không phải Go diễn giải các thành phần là bát phân. Nhìn chung,
nên luôn mã hóa lại các giá trị sau khi xác thực, giúp tránh được loại vấn đề không khớp của parser này.

<!-- net -->

#### [net/http](/pkg/net/http/)

<!-- CL 295370 -->
Package [`net/http`](/pkg/net/http/) hiện sử dụng
[`(*tls.Conn).HandshakeContext`](/pkg/crypto/tls#Conn.HandshakeContext) mới
với context [`Request`](/pkg/net/http/#Request)
khi thực hiện bắt tay TLS trong client hoặc server.

<!-- CL 235437 -->
Đặt các trường `ReadTimeout` hoặc `WriteTimeout` của
[`Server`](/pkg/net/http/#Server) thành giá trị âm hiện chỉ ra không có timeout
thay vì timeout ngay lập tức.

<!-- CL 308952 -->
Hàm [`ReadRequest`](/pkg/net/http/#ReadRequest)
hiện trả về lỗi khi yêu cầu có nhiều header Host.

<!-- CL 313950 -->
Khi tạo redirect đến phiên bản đã được làm sạch của URL,
[`ServeMux`](/pkg/net/http/#ServeMux) hiện luôn
sử dụng URL tương đối trong header `Location`. Trước đây nó
sẽ lặp lại URL đầy đủ của yêu cầu, có thể dẫn đến các redirect không mong muốn nếu client có thể bị buộc gửi URL yêu cầu tuyệt đối.

<!-- CL 308009, CL 313489 -->
Khi diễn giải một số header HTTP được xử lý bởi `net/http`,
các ký tự không phải ASCII hiện bị bỏ qua hoặc từ chối.

<!-- CL 325697 -->
Nếu
[`Request.ParseForm`](/pkg/net/http/#Request.ParseForm)
trả về lỗi khi được gọi bởi
[`Request.ParseMultipartForm`](/pkg/net/http/#Request.ParseMultipartForm),
phương thức sau hiện tiếp tục điền vào
[`Request.MultipartForm`](/pkg/net/http/#Request.MultipartForm)
trước khi trả về nó.

<!-- net/http -->

#### [net/http/httptest](/pkg/net/http/httptest/)

<!-- CL 308950 -->
[`ResponseRecorder.WriteHeader`](/pkg/net/http/httptest/#ResponseRecorder.WriteHeader)
hiện gây panic khi code được cung cấp không phải là mã trạng thái HTTP ba chữ số hợp lệ.
Điều này khớp với hành vi của các triển khai [`ResponseWriter`](/pkg/net/http/#ResponseWriter)
trong package [`net/http`](/pkg/net/http/).

<!-- net/http/httptest -->

#### [net/url](/pkg/net/url/)

<!-- CL 314850 -->
Phương thức mới [`Values.Has`](/pkg/net/url/#Values.Has)
báo cáo liệu một tham số query có được đặt hay không.

<!-- net/url -->

#### [os](/pkg/os/)

<!-- CL 268020 -->
Phương thức [`File.WriteString`](/pkg/os/#File.WriteString)
đã được tối ưu hóa để không tạo bản sao của chuỗi đầu vào.

<!-- os -->

#### [reflect](/pkg/reflect/)

<!-- CL 334669 -->
Phương thức mới
[`Value.CanConvert`](/pkg/reflect/#Value.CanConvert)
báo cáo liệu một giá trị có thể được chuyển đổi sang một kiểu hay không.
Điều này có thể được dùng để tránh panic khi chuyển đổi một slice sang
kiểu con trỏ mảng nếu slice quá ngắn.
Trước đây, chỉ cần sử dụng
[`Type.ConvertibleTo`](/pkg/reflect/#Type.ConvertibleTo)
là đủ, nhưng phép chuyển đổi mới được cho phép từ slice sang con trỏ mảng
có thể gây panic ngay cả khi các kiểu có thể chuyển đổi.

<!-- CL 266197 -->
Các phương thức mới
[`StructField.IsExported`](/pkg/reflect/#StructField.IsExported)
và
[`Method.IsExported`](/pkg/reflect/#Method.IsExported)
báo cáo liệu một trường struct hoặc phương thức kiểu có được xuất ra hay không.
Chúng cung cấp một thay thế dễ đọc hơn so với việc kiểm tra `PkgPath` có rỗng không.

<!-- CL 281233 -->
Hàm mới [`VisibleFields`](/pkg/reflect/#VisibleFields)
trả về tất cả các trường hiển thị trong một kiểu struct, bao gồm các trường bên trong các thành viên struct ẩn danh.

<!-- CL 284136 -->
Hàm [`ArrayOf`](/pkg/reflect/#ArrayOf) hiện gây panic khi
được gọi với độ dài âm.

<!-- CL 301652 -->
Kiểm tra phương thức [`Type.ConvertibleTo`](/pkg/reflect/#Type.ConvertibleTo)
không còn đủ để đảm bảo rằng một lời gọi đến
[`Value.Convert`](/pkg/reflect/#Value.Convert) sẽ không gây panic.
Nó có thể gây panic khi chuyển đổi `[]T` sang `*[N]T` nếu độ dài của slice nhỏ hơn N.
Xem phần [thay đổi ngôn ngữ](#language) ở trên.

<!-- CL 309729 -->
Các phương thức [`Value.Convert`](/pkg/reflect/#Value.Convert) và
[`Type.ConvertibleTo`](/pkg/reflect/#Type.ConvertibleTo)
đã được sửa để không coi các kiểu trong các package khác nhau có cùng tên
là giống nhau, để khớp với những gì ngôn ngữ cho phép.

<!-- reflect -->

#### [runtime/metrics](/pkg/runtime/metrics)

<!-- CL 308933, CL 312431, CL 312909 -->
Các metric mới đã được thêm vào theo dõi tổng số byte và đối tượng được cấp phát và giải phóng.
Một metric mới theo dõi phân phối độ trễ lập lịch goroutine cũng đã
được thêm vào.

<!-- runtime/metrics -->

#### [runtime/pprof](/pkg/runtime/pprof)

<!-- CL 299991 -->
Các block profile không còn bị thiên vị để ưu tiên các sự kiện dài không thường xuyên hơn
các sự kiện ngắn thường xuyên.

<!-- runtime/pprof -->

#### [strconv](/pkg/strconv/)

<!-- CL 170079, CL 170080 -->
Package `strconv` hiện sử dụng thuật toán Ryū của Ulf Adams để định dạng số dấu phẩy động.
Thuật toán này cải thiện hiệu suất trên hầu hết các đầu vào và nhanh hơn hơn 99% trên các đầu vào trường hợp xấu nhất.

<!-- CL 314775 -->
Hàm mới [`QuotedPrefix`](/pkg/strconv/#QuotedPrefix)
trả về chuỗi có dấu ngoặc kép (được hiểu bởi
[`Unquote`](/pkg/strconv/#Unquote))
ở phần đầu của đầu vào.

<!-- strconv -->

#### [strings](/pkg/strings/)

<!-- CL 280492 -->
Phương thức [`Builder.WriteRune`](/pkg/strings/#Builder.WriteRune)
hiện ghi ký tự thay thế U+FFFD cho các giá trị rune âm,
như nó làm đối với các rune không hợp lệ khác.

<!-- strings -->

#### [sync/atomic](/pkg/sync/atomic/)

<!-- CL 241678 -->
`atomic.Value` hiện có các phương thức [`Swap`](/pkg/sync/atomic/#Value.Swap) và
[`CompareAndSwap`](/pkg/sync/atomic/#Value.CompareAndSwap) cung cấp
các thao tác atomic bổ sung.

<!-- sync/atomic -->

#### [syscall](/pkg/syscall/)

<!-- CL 295371 -->

Các hàm [`GetQueuedCompletionStatus`](/pkg/syscall/#GetQueuedCompletionStatus) và
[`PostQueuedCompletionStatus`](/pkg/syscall/#PostQueuedCompletionStatus)
hiện bị deprecated. Các hàm này có chữ ký không đúng và được thay thế bởi
các tương đương trong package [`golang.org/x/sys/windows`](https://godoc.org/golang.org/x/sys/windows).

<!-- CL 313653 -->
Trên các hệ thống giống Unix, nhóm tiến trình của tiến trình con hiện được đặt với các tín hiệu bị chặn.
Điều này tránh gửi `SIGTTOU` đến con khi cha nằm trong nhóm tiến trình nền.

<!-- CL 288298, CL 288300 -->
Phiên bản Windows của
[`SysProcAttr`](/pkg/syscall/#SysProcAttr)
có hai trường mới. `AdditionalInheritedHandles` là
danh sách các handle bổ sung được kế thừa bởi tiến trình con mới.
`ParentProcess` cho phép chỉ định tiến trình
cha của tiến trình mới.

<!-- CL 311570 -->
Hằng số `MSG_CMSG_CLOEXEC` hiện được định nghĩa trên
DragonFly và tất cả các hệ thống OpenBSD (nó đã được định nghĩa trên
một số hệ thống OpenBSD và tất cả các hệ thống FreeBSD, NetBSD và Linux).

<!-- CL 315281 -->
Các hằng số `SYS_WAIT6` và `WEXITED`
hiện được định nghĩa trên các hệ thống NetBSD (`SYS_WAIT6` đã
được định nghĩa trên DragonFly và các hệ thống FreeBSD;
`WEXITED` đã được định nghĩa trên Darwin, DragonFly,
FreeBSD, Linux và Solaris).

<!-- syscall -->

#### [testing](/pkg/testing/)

<!-- CL 310033 -->
Đã thêm [cờ testing](/cmd/go/#hdr-Testing_flags) mới `-shuffle` kiểm soát thứ tự thực thi của các test và benchmark.

<!-- CL 260577 -->
Các phương thức mới
[`T.Setenv`](/pkg/testing/#T.Setenv)
và [`B.Setenv`](/pkg/testing/#B.Setenv)
hỗ trợ đặt biến môi trường trong thời gian
của test hoặc benchmark.

<!-- testing -->

#### [text/template/parse](/pkg/text/template/parse/)

<!-- CL 301493 -->
Giá trị `Mode` mới [`SkipFuncCheck`](/pkg/text/template/parse/#Mode)
thay đổi template parser để không xác minh rằng các hàm được định nghĩa.

<!-- text/template/parse -->

#### [time](/pkg/time/)

<!-- CL 260858 -->
Kiểu [`Time`](/pkg/time/#Time) hiện có phương thức
[`GoString`](/pkg/time/#Time.GoString) sẽ
trả về giá trị hữu ích hơn cho các thời điểm khi được in với
định dạng `%#v` trong package `fmt`.

<!-- CL 264077 -->
Phương thức mới [`Time.IsDST`](/pkg/time/#Time.IsDST) có thể được dùng để kiểm tra liệu thời gian có ở trong
Giờ Tiết kiệm Ánh sáng ban ngày (Daylight Savings Time) trong vị trí được cấu hình của nó hay không.

<!-- CL 293349 -->
Các phương thức mới [`Time.UnixMilli`](/pkg/time/#Time.UnixMilli) và
[`Time.UnixMicro`](/pkg/time/#Time.UnixMicro)
trả về số mili giây và micro giây đã trôi qua kể từ
ngày 1 tháng 1 năm 1970 UTC tương ứng.
\
Các hàm mới [`UnixMilli`](/pkg/time/#UnixMilli) và
[`UnixMicro`](/pkg/time/#UnixMicro)
trả về `Time` cục bộ tương ứng với thời gian Unix đã cho.

<!-- CL 300996 -->
Package hiện chấp nhận dấu phẩy "," như dấu phân cách cho giây thập phân khi phân tích và định dạng thời gian.
Ví dụ, các layout thời gian sau hiện được chấp nhận:

  - 2006-01-02 15:04:05,999999999 -0700 MST
  - Mon Jan \_2 15:04:05,000000 2006
  - Monday, January 2 15:04:05,000 2006


<!-- CL 320252 -->
Hằng số mới [`Layout`](/pkg/time/#Layout)
định nghĩa thời gian tham chiếu.

<!-- time -->

#### [unicode](/pkg/unicode/)

<!-- CL 280493 -->
Các hàm [`Is`](/pkg/unicode/#Is),
[`IsGraphic`](/pkg/unicode/#IsGraphic),
[`IsLetter`](/pkg/unicode/#IsLetter),
[`IsLower`](/pkg/unicode/#IsLower),
[`IsMark`](/pkg/unicode/#IsMark),
[`IsNumber`](/pkg/unicode/#IsNumber),
[`IsPrint`](/pkg/unicode/#IsPrint),
[`IsPunct`](/pkg/unicode/#IsPunct),
[`IsSpace`](/pkg/unicode/#IsSpace),
[`IsSymbol`](/pkg/unicode/#IsSymbol) và
[`IsUpper`](/pkg/unicode/#IsUpper)
hiện trả về `false` trên các giá trị rune âm, như chúng làm đối với các rune không hợp lệ khác.

<!-- unicode -->
