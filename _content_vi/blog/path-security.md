---
title: Bảo mật PATH lệnh trong Go
date: 2021-01-19
by:
- Russ Cox
summary: Cách quyết định xem chương trình của bạn có dễ bị tổn thương bởi các vấn đề PATH không, và phải làm gì về điều đó.
---


[Bản phát hành bảo mật Go hôm nay](/s/go-security-release-jan-2021)
sửa một vấn đề liên quan đến tra cứu PATH trong các thư mục không đáng tin
có thể dẫn đến thực thi từ xa trong lệnh `go` `get`.
Chúng tôi mong đợi mọi người sẽ có câu hỏi về ý nghĩa chính xác của điều này
và liệu họ có thể gặp sự cố trong chương trình của chính họ không.
Bài đăng này trình bày chi tiết về lỗi, các bản sửa lỗi chúng tôi đã áp dụng,
cách quyết định xem các chương trình của bạn có dễ bị tổn thương bởi các vấn đề tương tự không,
và những gì bạn có thể làm nếu có.

## Lệnh Go và thực thi từ xa

Một trong các mục tiêu thiết kế cho lệnh `go` là hầu hết các lệnh - bao gồm
`go` `build`, `go` `doc`, `go` `get`, `go` `install` và `go` `list` - không chạy
code tùy ý được tải xuống từ internet.
Có một vài ngoại lệ rõ ràng:
rõ ràng là `go` `run`, `go` `test` và `go` `generate` _có_ chạy code tùy ý - đó là công việc của họ.
Nhưng những lệnh còn lại không được, vì nhiều lý do bao gồm các build có thể tái tạo và bảo mật.
Vì vậy, khi `go` `get` có thể bị lừa để thực thi code tùy ý, chúng tôi coi đó là lỗi bảo mật.

Nếu `go` `get` không được chạy code tùy ý, thì thật không may điều đó có nghĩa là
tất cả các chương trình mà nó gọi, chẳng hạn như trình biên dịch và hệ thống kiểm soát phiên bản, cũng nằm trong vành đai bảo mật.
Ví dụ, chúng tôi đã có các sự cố trong quá khứ trong đó việc sử dụng khéo léo các tính năng trình biên dịch mờ ám
hoặc lỗi thực thi từ xa trong hệ thống kiểm soát phiên bản đã trở thành lỗi thực thi từ xa trong Go.
(Về điều đó, Go 1.16 nhằm cải thiện tình trạng bằng cách giới thiệu cài đặt GOVCS
cho phép cấu hình chính xác hệ thống kiểm soát phiên bản nào được phép và khi nào.)

Tuy nhiên, lỗi hôm nay hoàn toàn là lỗi của chúng tôi, không phải là lỗi hay tính năng mờ ám của `gcc` hoặc `git`.
Lỗi liên quan đến cách Go và các chương trình khác tìm kiếm các file thực thi khác,
vì vậy chúng ta cần dành một chút thời gian để xem xét điều đó trước khi có thể đến các chi tiết.

## Lệnh và PATH và Go

Tất cả các hệ điều hành đều có khái niệm về đường dẫn thực thi
(`$PATH` trên Unix, `%PATH%` trên Windows; để đơn giản, chúng tôi sẽ chỉ sử dụng thuật ngữ PATH),
đó là danh sách các thư mục.
Khi bạn gõ một lệnh vào dấu nhắc shell,
shell tìm kiếm trong mỗi thư mục được liệt kê,
theo thứ tự, một file thực thi với tên bạn đã gõ.
Nó chạy cái đầu tiên nó tìm thấy, hoặc nó in một thông báo như "command not found".

Trên Unix, ý tưởng này lần đầu tiên xuất hiện trong Bourne shell của Seventh Edition Unix (1979). Hướng dẫn giải thích:

> Tham số shell `$PATH` xác định đường dẫn tìm kiếm cho thư mục chứa lệnh.
> Mỗi tên thư mục thay thế được phân tách bằng dấu hai chấm (`:`).
> Đường dẫn mặc định là `:/bin:/usr/bin`.
> Nếu tên lệnh chứa dấu / thì đường dẫn tìm kiếm không được sử dụng.
> Nếu không, mỗi thư mục trong đường dẫn được tìm kiếm một tệp thực thi.

Lưu ý về giá trị mặc định: thư mục hiện tại (được biểu thị ở đây bằng chuỗi rỗng,
nhưng hãy gọi nó là "dot")
được liệt kê trước `/bin` và `/usr/bin`.
MS-DOS rồi Windows đã chọn hard-code hành vi đó:
trên các hệ thống đó, dot luôn được tìm kiếm đầu tiên,
tự động, trước khi xem xét bất kỳ thư mục nào được liệt kê trong `%PATH%`.

Như Grampp và Morris đã chỉ ra trong bài viết cổ điển của họ
"[UNIX Operating System Security](https://people.engr.ncsu.edu/gjin2/Classes/246/Spring2019/Security.pdf)" (1984),
đặt dot trước các thư mục hệ thống trong PATH
có nghĩa là nếu bạn `cd` vào một thư mục và chạy `ls`,
bạn có thể nhận được bản sao độc hại từ thư mục đó
thay vì tiện ích hệ thống.
Và nếu bạn có thể lừa quản trị viên hệ thống chạy `ls` trong thư mục home của bạn
khi đăng nhập với tư cách `root`, thì bạn có thể chạy bất kỳ code nào bạn muốn.
Vì vấn đề này và những vấn đề tương tự,
hầu hết tất cả các bản phân phối Unix hiện đại đặt PATH mặc định của người dùng mới
để loại trừ dot.
Nhưng hệ thống Windows tiếp tục tìm kiếm dot trước, bất kể PATH nói gì.

Ví dụ, khi bạn gõ lệnh

	go version

trên Unix được cấu hình thông thường,
shell chạy một file thực thi `go` từ một thư mục hệ thống trong PATH của bạn.
Nhưng khi bạn gõ lệnh đó trên Windows,
`cmd.exe` kiểm tra dot trước.
Nếu `.\go.exe` (hoặc `.\go.bat` hoặc nhiều lựa chọn khác) tồn tại,
`cmd.exe` chạy file thực thi đó, không phải cái từ PATH của bạn.

Đối với Go, các tìm kiếm PATH được xử lý bởi [`exec.LookPath`](https://pkg.go.dev/os/exec#LookPath),
được gọi tự động bởi
[`exec.Command`](https://pkg.go.dev/os/exec#Command).
Và để phù hợp tốt với hệ thống host, `exec.LookPath` của Go
triển khai quy tắc Unix trên Unix và quy tắc Windows trên Windows.
Ví dụ, lệnh này

	out, err := exec.Command("go", "version").CombinedOutput()

hoạt động giống như gõ `go` `version` vào shell hệ điều hành.
Trên Windows, nó chạy `.\go.exe` khi tồn tại.

(Đáng chú ý là Windows PowerShell đã thay đổi hành vi này,
bỏ việc tìm kiếm ngầm định của dot, nhưng `cmd.exe` và
[hàm `SearchPath` thư viện C Windows](https://docs.microsoft.com/en-us/windows/win32/api/processenv/nf-processenv-searchpatha)
tiếp tục hoạt động như chúng luôn làm.
Go tiếp tục khớp với `cmd.exe`.)

## Lỗi

Khi `go` `get` tải xuống và xây dựng một gói chứa
`import` `"C"`, nó chạy một chương trình gọi là `cgo` để chuẩn bị Go
tương đương với code C liên quan.
Lệnh `go` chạy `cgo` trong thư mục chứa các nguồn gói.
Sau khi `cgo` đã tạo ra các tệp Go đầu ra của nó,
lệnh `go` chính tự gọi trình biên dịch Go
trên các tệp Go được tạo ra
và trình biên dịch C host (`gcc` hoặc `clang`)
để xây dựng bất kỳ nguồn C nào đi kèm với gói.
Tất cả điều này hoạt động tốt.
Nhưng lệnh `go` tìm trình biên dịch C host ở đâu?
Nó tìm kiếm trong PATH, tất nhiên. May mắn thay, trong khi nó chạy trình biên dịch C
trong thư mục nguồn gói, nó thực hiện tra cứu PATH
từ thư mục gốc nơi lệnh `go` được gọi:

	cmd := exec.Command("gcc", "file.c")
	cmd.Dir = "badpkg"
	cmd.Run()

Vì vậy, ngay cả khi `badpkg\gcc.exe` tồn tại trên hệ thống Windows,
đoạn code này sẽ không tìm thấy nó.
Tra cứu xảy ra trong `exec.Command` không biết
về thư mục `badpkg`.

Lệnh `go` sử dụng code tương tự để gọi `cgo`,
và trong trường hợp đó thậm chí không có tra cứu đường dẫn,
vì `cgo` luôn đến từ GOROOT:

	cmd := exec.Command(GOROOT+"/pkg/tool/"+GOOS_GOARCH+"/cgo", "file.go")
	cmd.Dir = "badpkg"
	cmd.Run()

Điều này thậm chí còn an toàn hơn đoạn code trước:
không có cơ hội chạy bất kỳ `cgo.exe` xấu nào có thể tồn tại.

Nhưng hóa ra bản thân cgo cũng gọi trình biên dịch C host,
trên một số tệp tạm thời mà nó tạo ra, có nghĩa là nó tự thực thi code này:

	// chạy trong cgo trong thư mục badpkg
	cmd := exec.Command("gcc", "tmpfile.c")
	cmd.Run()

Bây giờ, vì bản thân cgo đang chạy trong `badpkg`,
không phải trong thư mục nơi lệnh `go` được chạy,
nó sẽ chạy `badpkg\gcc.exe` nếu tệp đó tồn tại,
thay vì tìm `gcc` hệ thống.

Vì vậy, kẻ tấn công có thể tạo một gói độc hại sử dụng cgo và
bao gồm một `gcc.exe`, và sau đó bất kỳ người dùng Windows nào
chạy `go` `get` để tải xuống và xây dựng gói của kẻ tấn công
sẽ chạy `gcc.exe` do kẻ tấn công cung cấp thay vì bất kỳ
`gcc` nào trong đường dẫn hệ thống.

Hệ thống Unix tránh vấn đề đầu tiên vì dot thường không
trong PATH và thứ hai vì giải nén module không
đặt bit thực thi trên các tệp nó ghi.
Nhưng người dùng Unix có dot trước các thư mục hệ thống
trong PATH của họ và đang sử dụng chế độ GOPATH sẽ dễ bị tổn thương như
người dùng Windows.
(Nếu đó là mô tả bạn, hôm nay là ngày tốt để xóa dot khỏi đường dẫn
và bắt đầu sử dụng module Go.)

(Cảm ơn [RyotaK](https://twitter.com/ryotkak) đã [báo cáo vấn đề này](/security) cho chúng tôi.)

## Các bản sửa lỗi

Rõ ràng là không chấp nhận được khi lệnh `go` `get` tải xuống
và chạy `gcc.exe` độc hại.
Nhưng sai lầm thực sự cho phép điều đó là gì?
Và sau đó bản sửa lỗi là gì?

Một câu trả lời có thể là sai lầm là `cgo` thực hiện tìm kiếm trình biên dịch C host
trong thư mục nguồn không đáng tin thay vì trong thư mục nơi lệnh `go`
được gọi.
Nếu đó là sai lầm,
thì bản sửa lỗi là thay đổi lệnh `go` để truyền cho `cgo` đường dẫn đầy đủ đến
trình biên dịch C host, để `cgo` không cần thực hiện tra cứu PATH vào
thư mục không đáng tin.

Một câu trả lời có thể khác là sai lầm là tìm kiếm trong dot
trong quá trình tra cứu PATH, dù xảy ra tự động trên Windows
hay vì một mục PATH rõ ràng trên hệ thống Unix.
Người dùng có thể muốn tìm kiếm trong dot để tìm một lệnh họ đã gõ
trong cửa sổ console hoặc shell,
nhưng khó có thể họ cũng muốn tìm ở đó một tiến trình con của tiến trình con
của một lệnh đã gõ.
Nếu đó là sai lầm,
thì bản sửa lỗi là thay đổi lệnh `cgo` để không tìm kiếm trong dot trong quá trình tra cứu PATH.

Chúng tôi quyết định cả hai đều là sai lầm, vì vậy chúng tôi đã áp dụng cả hai bản sửa lỗi.
Lệnh `go` bây giờ truyền đường dẫn đầy đủ của trình biên dịch C host cho `cgo`.
Ngoài ra, `cgo`, `go` và mọi lệnh khác trong bản phân phối Go
bây giờ sử dụng một biến thể của gói `os/exec` báo cáo lỗi nếu nó sẽ
đã sử dụng một file thực thi từ dot trước đây.
Các gói `go/build` và `go/import` sử dụng chính sách tương tự cho
việc gọi lệnh `go` và các công cụ khác của chúng.
Điều này nên đóng cửa bất kỳ vấn đề bảo mật tương tự nào có thể đang ẩn nấp.

Vì thận trọng, chúng tôi cũng thực hiện sửa chữa tương tự trong
các lệnh như `goimports` và `gopls`,
cũng như các thư viện
`golang.org/x/tools/go/analysis`
và
`golang.org/x/tools/go/packages`,
gọi lệnh `go` như một tiến trình con.
Nếu bạn chạy các chương trình này trong các thư mục không đáng tin -
ví dụ, nếu bạn `git` `checkout` các kho lưu trữ không đáng tin
và `cd` vào chúng rồi chạy các chương trình như thế này,
và bạn sử dụng Windows hoặc sử dụng Unix với dot trong PATH -
thì bạn cũng nên cập nhật bản sao của các lệnh này.
Nếu các thư mục không đáng tin duy nhất trên máy tính của bạn
là những thư mục trong bộ nhớ cache module được quản lý bởi `go` `get`,
thì bạn chỉ cần bản phát hành Go mới.

Sau khi cập nhật lên bản phát hành Go mới, bạn có thể cập nhật lên `gopls` mới nhất bằng cách sử dụng:

	GO111MODULE=on \
	go get golang.org/x/tools/gopls@v0.6.4

và bạn có thể cập nhật lên `goimports` mới nhất hoặc các công cụ khác bằng cách sử dụng:

	GO111MODULE=on \
	go get golang.org/x/tools/cmd/goimports@v0.1.0

Bạn có thể cập nhật các chương trình phụ thuộc vào `golang.org/x/tools/go/packages`,
ngay cả trước khi các tác giả của họ làm vậy,
bằng cách thêm một nâng cấp rõ ràng của dependency trong `go` `get`:

	GO111MODULE=on \
	go get example.com/cmd/thecmd golang.org/x/tools@v0.1.0

Đối với các chương trình sử dụng `go/build`, đủ để bạn biên dịch lại chúng
bằng bản phát hành Go đã cập nhật.

Một lần nữa, bạn chỉ cần cập nhật các chương trình khác này nếu bạn
là người dùng Windows hoặc người dùng Unix có dot trong PATH
_và_ bạn chạy các chương trình này trong các thư mục nguồn mà bạn không tin tưởng
có thể chứa các chương trình độc hại.

## Các chương trình của bạn có bị ảnh hưởng không?

Nếu bạn sử dụng `exec.LookPath` hoặc `exec.Command` trong các chương trình của riêng mình,
bạn chỉ cần lo lắng nếu bạn (hoặc người dùng của bạn) chạy chương trình của bạn
trong một thư mục có nội dung không đáng tin.
Nếu vậy, thì một tiến trình con có thể được khởi động bằng một file thực thi
từ dot thay vì từ thư mục hệ thống.
(Một lần nữa, sử dụng file thực thi từ dot xảy ra luôn trên Windows
và chỉ với các cài đặt PATH không phổ biến trên Unix.)

Nếu bạn lo lắng, thì chúng tôi đã xuất bản biến thể hạn chế hơn
của `os/exec` là [`golang.org/x/sys/execabs`](https://pkg.go.dev/golang.org/x/sys/execabs).
Bạn có thể sử dụng nó trong chương trình của mình bằng cách đơn giản thay thế

	import "os/exec"

bằng

	import exec "golang.org/x/sys/execabs"

và biên dịch lại.

## Bảo mật os/exec theo mặc định

Chúng tôi đang thảo luận trên
[golang.org/issue/38736](/issue/38736)
liệu hành vi Windows luôn ưu tiên thư mục hiện tại
trong các tra cứu PATH (trong `exec.Command` và `exec.LookPath`)
có nên được thay đổi không.
Lý lẽ ủng hộ thay đổi là nó đóng cửa các loại
vấn đề bảo mật được thảo luận trong bài đăng blog này.
Một lý lẽ hỗ trợ là mặc dù API `SearchPath` Windows
và `cmd.exe` vẫn luôn tìm kiếm thư mục hiện tại,
PowerShell, người kế thừa của `cmd.exe`, thì không,
một sự thừa nhận rõ ràng rằng hành vi ban đầu là một sai lầm.
Lý lẽ chống lại thay đổi là nó có thể phá vỡ các chương trình Windows hiện có
có ý định tìm kiếm chương trình trong thư mục hiện tại.
Chúng tôi không biết có bao nhiêu chương trình như vậy tồn tại,
nhưng chúng sẽ gặp lỗi không được giải thích nếu các tra cứu PATH
bắt đầu bỏ qua hoàn toàn thư mục hiện tại.

Cách tiếp cận chúng tôi đã thực hiện trong `golang.org/x/sys/execabs` có thể
là một vùng đất trung gian hợp lý.
Nó tìm thấy kết quả của tra cứu PATH cũ và sau đó trả về một
lỗi rõ ràng thay vì sử dụng kết quả từ thư mục hiện tại.
Lỗi được trả về từ `exec.Command("prog")` khi `prog.exe` tồn tại trông như:

	prog resolves to executable in current directory (.\prog.exe)

Đối với các chương trình thay đổi hành vi, lỗi này sẽ làm rõ những gì đã xảy ra.
Các chương trình có ý định chạy một chương trình từ thư mục hiện tại có thể sử dụng
`exec.Command("./prog")` thay thế (cú pháp đó hoạt động trên tất cả các hệ thống, kể cả Windows).

Chúng tôi đã đề xuất ý tưởng này như một đề xuất mới, [golang.org/issue/43724](/issue/43724).
