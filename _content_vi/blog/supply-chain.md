---
title: "Go Giảm thiểu Tấn công Chuỗi Cung ứng như thế nào"
date: 2022-03-31
by:
- Filippo Valsorda
summary: Công cụ và thiết kế của Go giúp giảm thiểu các tấn công chuỗi cung ứng ở nhiều giai đoạn.
---

Kỹ thuật phần mềm hiện đại mang tính cộng tác và dựa trên việc tái sử dụng phần mềm
mã nguồn mở.
Điều đó khiến các mục tiêu dễ bị tấn công chuỗi cung ứng, nơi các dự án phần mềm bị
tấn công bằng cách xâm phạm các dependency của chúng.

Bất kể quy trình hay biện pháp kỹ thuật nào, mỗi dependency không thể tránh khỏi là một
mối quan hệ tin tưởng.
Tuy nhiên, công cụ và thiết kế của Go giúp giảm thiểu rủi ro ở nhiều giai đoạn.


## Tất cả các bản build đều bị "khóa"

Không có cách nào để các thay đổi trong thế giới bên ngoài, chẳng hạn như một phiên bản mới của
dependency được phát hành, tự động ảnh hưởng đến một bản build Go.

Không giống như hầu hết các file quản lý gói khác, các module Go không có danh sách
ràng buộc riêng biệt và một lock file ghim các phiên bản cụ thể.
Phiên bản của mọi dependency đóng góp vào bất kỳ bản build Go nào được xác định hoàn toàn
bởi [file `go.mod`](/ref/mod#go-mod-file) của module chính.

Kể từ Go 1.16, tính xác định này được thực thi theo mặc định, và các lệnh build (`go
build`, `go test`, `go install`, `go run`, ...) [sẽ thất bại nếu go.mod không
đầy đủ](/ref/mod#go-mod-file-updates).
Các lệnh duy nhất sẽ thay đổi `go.mod` (và do đó thay đổi bản build) là
`go get` và `go mod tidy`.
Các lệnh này không được mong đợi chạy tự động hoặc trong CI, vì vậy các thay đổi đối với
cây dependency phải được thực hiện có chủ đích và có cơ hội đi qua
quy trình code review.

Điều này rất quan trọng về mặt bảo mật, bởi vì khi một hệ thống CI hoặc máy mới
chạy `go build`, mã nguồn đã được kiểm tra vào kho là nguồn thông tin
chính xác nhất và đầy đủ nhất về những gì sẽ được build.
Không có cách nào để bên thứ ba ảnh hưởng đến điều đó.

Hơn nữa, khi một dependency được thêm vào bằng `go get`, các dependency bắc cầu của nó
được thêm vào ở phiên bản được chỉ định trong file `go.mod` của dependency đó, không phải ở
phiên bản mới nhất của chúng, nhờ vào
[Minimal version selection](/ref/mod#minimal-version-selection).
Điều tương tự xảy ra với các lệnh gọi
`go install example.com/cmd/devtoolx@latest`, [các lệnh tương đương trong một số
hệ sinh thái bỏ qua việc ghim phiên bản](https://research.swtch.com/npm-colors).
Trong Go, phiên bản mới nhất của `example.com/cmd/devtoolx` sẽ được tải về, nhưng
sau đó tất cả các dependency sẽ được thiết lập bởi file `go.mod` của nó.

Nếu một module bị xâm phạm và một phiên bản độc hại mới được phát hành, không ai
sẽ bị ảnh hưởng cho đến khi họ cập nhật dependency đó một cách rõ ràng, cung cấp
cơ hội để xem xét các thay đổi và thời gian để hệ sinh thái phát hiện
sự kiện.


## Nội dung phiên bản không bao giờ thay đổi

Một thuộc tính quan trọng khác cần thiết để đảm bảo các bên thứ ba không thể ảnh hưởng đến các bản build là
nội dung của một phiên bản module là bất biến.
Nếu một kẻ tấn công xâm phạm một dependency có thể tải lại một
phiên bản hiện có, họ có thể tự động xâm phạm tất cả các dự án phụ thuộc vào nó.

Đó là lý do tại sao có [file `go.sum`](/ref/mod#go-sum-files).
Nó chứa danh sách các hash mã hóa của mỗi dependency đóng góp
vào bản build.
Một lần nữa, một `go.sum` không đầy đủ gây ra lỗi, và chỉ `go
get` và `go mod tidy` mới sẽ sửa đổi nó, vì vậy bất kỳ thay đổi nào đối với nó
sẽ đi kèm với một thay đổi dependency có chủ đích.
Các bản build khác được đảm bảo có đầy đủ bộ checksum.

Đây là tính năng phổ biến của hầu hết các lock files.
Go vượt xa điều đó với
[Checksum Database](/ref/mod#checksum-database) (viết tắt là sumdb),
một danh sách chỉ thêm, có thể xác minh bằng mã hóa toàn cầu của các mục go.sum.
Khi `go get` cần thêm một mục vào file `go.sum`, nó tải mục đó từ
sumdb cùng với bằng chứng mã hóa về tính toàn vẹn của sumdb.
Điều này đảm bảo rằng không chỉ mọi bản build của một module nhất định sử dụng cùng
nội dung dependency, mà mọi module đều sử dụng cùng nội dung dependency!

Sumdb làm cho việc các dependency bị xâm phạm hoặc thậm chí cơ sở hạ tầng Go do
Google vận hành nhắm mục tiêu vào các dependent cụ thể bằng mã nguồn đã được sửa đổi
(ví dụ: backdoor) trở nên không thể.
Bạn được đảm bảo đang sử dụng chính xác cùng một mã mà tất cả những người khác đang sử dụng
v1.9.2 của `example.com/modulex` đang sử dụng và đã xem xét.

Cuối cùng, tính năng yêu thích của tôi trong sumdb: nó không yêu cầu bất kỳ quản lý khóa nào
từ phía tác giả module, và nó hoạt động liền mạch với
bản chất phi tập trung của các module Go.


## VCS là nguồn thông tin chính xác nhất

Hầu hết các dự án được phát triển thông qua một số hệ thống kiểm soát phiên bản (VCS) và sau đó,
trong các hệ sinh thái khác, được tải lên kho gói.
Điều này có nghĩa là có hai tài khoản có thể bị xâm phạm, máy chủ VCS và
kho gói, cái sau được sử dụng ít thường xuyên hơn và dễ bị bỏ qua hơn.
Nó cũng có nghĩa là việc ẩn mã độc trong phiên bản được tải lên
kho dễ dàng hơn, đặc biệt nếu mã nguồn thường xuyên được sửa đổi như một phần của
việc tải lên, ví dụ như để thu nhỏ nó.

Trong Go, không có khái niệm tài khoản kho gói.
Import path của một package nhúng thông tin mà `go mod download`
[cần để tải về module của nó](https://pkg.go.dev/cmd/go#hdr-Remote_import_paths) trực tiếp từ
VCS, nơi các tag xác định các phiên bản.

Chúng tôi có [Go Module Mirror](/blog/module-mirror-launch), nhưng
đó chỉ là một proxy.
Tác giả module không đăng ký tài khoản và không tải phiên bản lên proxy.
Proxy sử dụng logic tương tự mà công cụ `go` sử dụng (trên thực tế, proxy chạy
`go mod download`) để tải về và lưu vào cache một phiên bản.
Vì Checksum Database đảm bảo rằng chỉ có thể có một cây mã nguồn
cho một phiên bản module nhất định, mọi người sử dụng proxy sẽ thấy kết quả giống nhau như
mọi người bỏ qua proxy và tải trực tiếp từ VCS.
(Nếu phiên bản không còn có sẵn trong VCS hoặc nếu nội dung của nó thay đổi,
tải trực tiếp sẽ dẫn đến lỗi, trong khi tải từ proxy có thể
vẫn hoạt động, cải thiện tính khả dụng và bảo vệ hệ sinh thái khỏi các vấn đề
["left-pad"](https://blog.npmjs.org/post/141577284765/kik-left-pad-and-npm).)

Chạy các công cụ VCS trên client tạo ra một bề mặt tấn công khá lớn.
Đó là một nơi khác mà Go Module Mirror hỗ trợ: công cụ `go` trên proxy chạy
bên trong một sandbox mạnh mẽ và được cấu hình để hỗ trợ mọi công cụ VCS, trong khi
[mặc định chỉ hỗ trợ hai hệ thống VCS chính](/ref/mod#vcs-govcs) (git và Mercurial).
Bất kỳ ai sử dụng proxy vẫn có thể tải mã được phát hành bằng các hệ thống VCS không mặc định,
nhưng những kẻ tấn công không thể tiếp cận mã đó trong hầu hết các cài đặt.


## Build code không thực thi nó

Đó là một mục tiêu thiết kế bảo mật rõ ràng của Go toolchain rằng việc tải về
cũng như build code sẽ không cho phép code đó thực thi, ngay cả khi nó không đáng tin cậy và
độc hại.
Điều này khác với hầu hết các hệ sinh thái khác, nhiều trong số đó có hỗ trợ
hạng nhất để chạy code tại thời điểm tải gói.
Các hook "post-install" này đã được sử dụng trong quá khứ như cách thuận tiện nhất để
biến một dependency bị xâm phạm thành máy nhà phát triển bị xâm phạm, và để
[worm](https://en.wikipedia.org/wiki/Computer_worm) qua các tác giả module.

Thành thật mà nói, nếu bạn đang tải về một số code thì thường là để thực thi nó ngay sau đó,
dù là trong quá trình kiểm tra trên máy nhà phát triển hay như một phần của
binary trong production, vì vậy việc thiếu các hook post-install chỉ làm chậm lại
những kẻ tấn công.
(Không có ranh giới bảo mật trong một bản build: bất kỳ package nào đóng góp vào một
bản build đều có thể định nghĩa một hàm `init`.)
Tuy nhiên, nó có thể là một biện pháp giảm thiểu rủi ro có ý nghĩa, vì bạn có thể đang thực thi một
binary hoặc kiểm tra một package chỉ sử dụng một tập con các dependency của module.
Ví dụ, nếu bạn build và thực thi `example.com/cmd/devtoolx` trên macOS thì
không có cách nào để một dependency chỉ dành cho Windows hoặc một dependency của
`example.com/cmd/othertool` xâm phạm máy của bạn.

Trong Go, các module không đóng góp code vào một bản build cụ thể không có tác động bảo mật đến nó.


## "Một chút sao chép tốt hơn một chút dependency"

Biện pháp giảm thiểu rủi ro chuỗi cung ứng phần mềm cuối cùng và có lẽ quan trọng nhất trong
hệ sinh thái Go là biện pháp ít kỹ thuật nhất: Go có văn hóa từ chối các cây
dependency lớn, và ưu tiên sao chép một chút hơn là thêm một dependency mới.
Điều này trở lại một trong những châm ngôn Go: ["a little copying is better
than a little dependency"](https://youtube.com/clip/UgkxWCEmMJFW0-TvSMzcMEAHZcpt2FsVXP65).
Nhãn "zero dependencies" được đeo tự hào bởi các module Go tái sử dụng chất lượng cao.
Nếu bạn thấy mình cần một thư viện, bạn có khả năng sẽ thấy rằng nó sẽ không
khiến bạn phụ thuộc vào hàng chục module khác của các tác giả và chủ sở hữu khác.

Điều đó cũng được hỗ trợ bởi thư viện chuẩn phong phú và các module bổ sung (các module
`golang.org/x/...`), cung cấp các building blocks cấp cao thường được sử dụng
như một HTTP stack, một thư viện TLS, mã hóa JSON, v.v.

Tất cả những điều này có nghĩa là có thể xây dựng các ứng dụng phong phú, phức tạp với
chỉ một vài dependency.
Dù công cụ có tốt đến đâu, nó không thể loại bỏ rủi ro liên quan đến
việc tái sử dụng code, vì vậy biện pháp giảm thiểu mạnh nhất sẽ luôn là một cây dependency nhỏ.
