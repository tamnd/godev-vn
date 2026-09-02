---
title: "Go Modules: v2 và Xa hơn"
date: 2019-11-07
by:
- Jean Barkhuysen
- Tyler Bui-Palsulich
tags:
- tools
- versioning
summary: Cách phát hành major version 2 của module.
---

## Giới thiệu

Bài viết này là phần 4 trong một loạt bài.

  - Phần 1 — [Sử dụng Go Modules](/blog/using-go-modules)
  - Phần 2 — [Chuyển đổi sang Go Modules](/blog/migrating-to-go-modules)
  - Phần 3 — [Xuất bản Go Modules](/blog/publishing-go-modules)
  - **Phần 4 — Go Modules: v2 và Xa hơn** (bài này)
  - Phần 5 — [Giữ Modules tương thích](/blog/module-compatibility)

**Lưu ý:** Để xem tài liệu về phát triển modules, hãy xem
[Phát triển và xuất bản modules](/doc/modules/developing).

Khi một dự án thành công trưởng thành và các yêu cầu mới được thêm vào, các tính năng và quyết định thiết kế trong quá khứ có thể không còn hợp lý nữa. Các nhà phát triển có thể muốn tích hợp những bài học họ đã học bằng cách xóa các hàm bị lỗi thời, đổi tên kiểu, hoặc tách các gói phức tạp thành các phần dễ quản lý hơn. Những thay đổi như vậy đòi hỏi nỗ lực từ người dùng hạ nguồn để chuyển đổi code của họ sang API mới, vì vậy chúng không nên được thực hiện nếu không cân nhắc kỹ rằng lợi ích vượt trội chi phí.

Đối với các dự án vẫn đang thử nghiệm, ở major version `v0`, người dùng dự kiến sẽ có những thay đổi không tương thích thỉnh thoảng. Đối với các dự án đã được tuyên bố ổn định, ở major version `v1` trở lên, các thay đổi không tương thích phải được thực hiện trong một major version mới. Bài viết này khám phá ngữ nghĩa major version, cách tạo và xuất bản một major version mới, và cách duy trì nhiều major version của một module.

## Major version và đường dẫn module

Modules đã chính thức hóa một nguyên tắc quan trọng trong Go, [**quy tắc tương thích import**](https://research.swtch.com/vgo-import):

	Nếu một gói cũ và một gói mới có cùng đường dẫn import, gói mới phải tương thích ngược với gói cũ.

Theo định nghĩa, một major version mới của gói không tương thích ngược với phiên bản trước. Điều này có nghĩa là một major version mới của module phải có đường dẫn module khác với phiên bản trước. Bắt đầu từ `v2`, major version phải xuất hiện ở cuối đường dẫn module (được khai báo trong câu lệnh `module` trong tệp `go.mod`). Ví dụ, khi các tác giả của module `github.com/googleapis/gax-go` phát triển `v2`, họ đã sử dụng đường dẫn module mới `github.com/googleapis/gax-go/v2`. Người dùng muốn dùng `v2` phải thay đổi các import gói và yêu cầu module của họ thành `github.com/googleapis/gax-go/v2`.

Sự cần thiết của hậu tố major version là một trong những điểm Go modules khác với hầu hết các hệ thống quản lý dependency khác. Hậu tố cần thiết để giải quyết [vấn đề dependency dạng kim cương](https://research.swtch.com/vgo-import#dependency_story). Trước Go modules, [gopkg.in](http://gopkg.in) cho phép các nhà duy trì gói tuân theo điều mà chúng ta hiện gọi là quy tắc tương thích import. Với gopkg.in, nếu bạn phụ thuộc vào một gói import `gopkg.in/yaml.v1` và một gói khác import `gopkg.in/yaml.v2`, không có xung đột vì hai gói `yaml` có đường dẫn import khác nhau, chúng sử dụng hậu tố phiên bản, giống như Go modules. Vì gopkg.in chia sẻ cùng phương pháp hậu tố phiên bản như Go modules, lệnh Go chấp nhận `.v2` trong `gopkg.in/yaml.v2` như một hậu tố major version hợp lệ. Đây là trường hợp đặc biệt để tương thích với gopkg.in: các module được host tại các tên miền khác cần hậu tố dạng dấu gạch chéo như `/v2`.

## Các chiến lược major version

Chiến lược được khuyến nghị là phát triển các module `v2+` trong một thư mục được đặt tên theo hậu tố major version.

	github.com/googleapis/gax-go @ master branch
	/go.mod    → module github.com/googleapis/gax-go
	/v2/go.mod → module github.com/googleapis/gax-go/v2

Cách tiếp cận này tương thích với các công cụ không nhận thức được modules: các đường dẫn tệp trong kho lưu trữ khớp với các đường dẫn được `go get` mong đợi trong chế độ `GOPATH`. Chiến lược này cũng cho phép tất cả các major version được phát triển cùng nhau trong các thư mục khác nhau.

Các chiến lược khác có thể giữ major version trên các nhánh riêng. Tuy nhiên, nếu code nguồn `v2+` nằm trên nhánh mặc định của kho lưu trữ (thường là `master`), các công cụ không nhận thức phiên bản, bao gồm lệnh `go` ở chế độ `GOPATH`, có thể không phân biệt được giữa các major version.

Các ví dụ trong bài viết này sẽ tuân theo chiến lược thư mục con major version, vì nó cung cấp khả năng tương thích cao nhất. Chúng tôi khuyến nghị các tác giả module theo chiến lược này miễn là họ có người dùng phát triển trong chế độ `GOPATH`.

## Xuất bản v2 và xa hơn

Bài viết này sử dụng `github.com/googleapis/gax-go` làm ví dụ:

	$ pwd
	/tmp/gax-go
	$ ls
	CODE_OF_CONDUCT.md  call_option.go  internal
	CONTRIBUTING.md     gax.go          invoke.go
	LICENSE             go.mod          tools.go
	README.md           go.sum          RELEASING.md
	header.go
	$ cat go.mod
	module github.com/googleapis/gax-go

	go 1.9

	require (
		github.com/golang/protobuf v1.3.1
		golang.org/x/exp v0.0.0-20190221220918-438050ddec5e
		golang.org/x/lint v0.0.0-20181026193005-c67002cb31c3
		golang.org/x/tools v0.0.0-20190114222345-bf090417da8b
		google.golang.org/grpc v1.19.0
		honnef.co/go/tools v0.0.0-20190102054323-c2f93a96b099
	)
	$

Để bắt đầu phát triển `v2` của `github.com/googleapis/gax-go`, chúng ta sẽ tạo một thư mục `v2/` mới và sao chép gói của chúng ta vào đó.

	$ mkdir v2
	$ cp -v *.go v2
	'call_option.go' -> 'v2/call_option.go'
	'gax.go' -> 'v2/gax.go'
	'header.go' -> 'v2/header.go'
	'invoke.go' -> 'v2/invoke.go'
	$

Bây giờ, hãy tạo tệp `go.mod` cho v2 bằng cách sao chép tệp `go.mod` hiện tại và thêm hậu tố `/v2` vào đường dẫn module:

	$ cp go.mod v2/go.mod
	$ go mod edit -module github.com/googleapis/gax-go/v2 v2/go.mod
	$

Lưu ý rằng phiên bản `v2` được coi là một module riêng biệt với các phiên bản `v0 / v1`: cả hai có thể cùng tồn tại trong cùng một build. Vì vậy, nếu module `v2+` của bạn có nhiều gói, bạn nên cập nhật chúng để sử dụng đường dẫn import `/v2` mới: nếu không, module `v2+` của bạn sẽ phụ thuộc vào module `v0 / v1`. Ví dụ, để cập nhật tất cả các tham chiếu `github.com/my/project` thành `github.com/my/project/v2`, bạn có thể dùng `find` và `sed`:

	$ find . -type f \
		-name '*.go' \
		-exec sed -i -e 's,github.com/my/project,github.com/my/project/v2,g' {} \;
	$

Bây giờ chúng ta có module `v2`, nhưng chúng ta muốn thử nghiệm và thực hiện thay đổi trước khi xuất bản một bản phát hành. Cho đến khi chúng ta phát hành `v2.0.0` (hoặc bất kỳ phiên bản nào không có hậu tố tiền phát hành), chúng ta có thể phát triển và thực hiện các thay đổi không tương thích khi quyết định API mới. Nếu chúng ta muốn người dùng có thể thử nghiệm API mới trước khi chúng ta chính thức ổn định, chúng ta có thể xuất bản phiên bản tiền phát hành `v2`:

	$ git tag v2.0.0-alpha.1
	$ git push origin v2.0.0-alpha.1
	$

Khi chúng ta hài lòng với API `v2` và chắc chắn không cần thêm thay đổi không tương thích nào nữa, chúng ta có thể gắn thẻ `v2.0.0`:

	$ git tag v2.0.0
	$ git push origin v2.0.0
	$

Tại thời điểm đó, hiện có hai major version cần duy trì. Các thay đổi tương thích ngược và sửa lỗi sẽ dẫn đến các bản phát hành minor và patch mới (ví dụ: `v1.1.0`, `v2.0.1`, v.v.).

## Kết luận

Các thay đổi major version tạo ra chi phí phát triển và bảo trì, đòi hỏi đầu tư từ người dùng hạ nguồn để chuyển đổi. Dự án càng lớn, các chi phí này càng lớn. Thay đổi major version chỉ nên diễn ra sau khi xác định được lý do thuyết phục. Khi đã xác định được lý do thuyết phục cho một thay đổi không tương thích, chúng tôi khuyến nghị phát triển nhiều major version trong nhánh master vì nó tương thích với nhiều công cụ hiện có hơn.

Các thay đổi không tương thích với module `v1+` luôn nên xảy ra trong module `vN+1` mới. Khi một module mới được phát hành, điều đó có nghĩa là thêm công việc cho cả người duy trì và người dùng cần chuyển đổi sang gói mới. Vì vậy, người duy trì nên xác nhận các API của mình trước khi thực hiện bản phát hành ổn định và cân nhắc kỹ liệu các thay đổi không tương thích có thực sự cần thiết sau `v1` hay không.
