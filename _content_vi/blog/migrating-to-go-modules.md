---
title: Chuyển đổi sang Go Modules
date: 2019-08-21
by:
- Jean Barkhuysen
tags:
- tools
- versioning
- modules
summary: Cách sử dụng Go modules để quản lý các dependency của chương trình.
---

## Giới thiệu

Bài viết này là phần 2 trong một loạt bài.

  - Phần 1 — [Sử dụng Go Modules](/blog/using-go-modules)
  - **Phần 2 — Chuyển đổi sang Go Modules** (bài này)
  - Phần 3 — [Xuất bản Go Modules](/blog/publishing-go-modules)
  - Phần 4 — [Go Modules: v2 và xa hơn](/blog/v2-go-modules)
  - Phần 5 — [Giữ cho Modules của bạn tương thích](/blog/module-compatibility)

**Lưu ý:** Để xem tài liệu, hãy xem
[Quản lý dependency](/doc/modules/managing-dependencies)
và [Phát triển và xuất bản modules](/doc/modules/developing).

Các dự án Go sử dụng nhiều chiến lược quản lý dependency khác nhau.
Các công cụ [Vendoring](/cmd/go/#hdr-Vendor_Directories) như
[dep](https://github.com/golang/dep) và [glide](https://github.com/Masterminds/glide) rất phổ biến,
nhưng chúng có nhiều khác biệt về hành vi và không phải lúc nào cũng hoạt động tốt cùng nhau.
Một số dự án lưu toàn bộ thư mục GOPATH của họ trong một kho lưu trữ Git duy nhất.
Các dự án khác đơn giản dựa vào `go get` và kỳ vọng các phiên bản khá gần đây của dependency
được cài đặt trong GOPATH.

Hệ thống module của Go, được giới thiệu trong Go 1.11,
cung cấp giải pháp quản lý dependency chính thức được tích hợp vào lệnh `go`.
Bài viết này mô tả các công cụ và kỹ thuật để chuyển đổi một dự án sang modules.

Xin lưu ý: nếu dự án của bạn đã được gắn thẻ ở v2.0.0 trở lên,
bạn sẽ cần cập nhật đường dẫn module khi thêm tệp `go.mod`.
Chúng tôi sẽ giải thích cách thực hiện điều đó mà không ảnh hưởng đến người dùng trong một bài viết tương lai
tập trung vào v2 và xa hơn.

## Chuyển đổi sang Go modules trong dự án của bạn

Một dự án có thể ở một trong ba trạng thái khi bắt đầu chuyển đổi sang Go modules:

  - Một dự án Go hoàn toàn mới.
  - Một dự án Go đã có với công cụ quản lý dependency không phải module.
  - Một dự án Go đã có mà không có bất kỳ công cụ quản lý dependency nào.

Trường hợp đầu tiên được đề cập trong [Sử dụng Go Modules](/blog/using-go-modules);
chúng ta sẽ giải quyết hai trường hợp sau trong bài này.

## Với công cụ quản lý dependency

Để chuyển đổi một dự án đã sử dụng công cụ quản lý dependency, hãy chạy các lệnh sau:

	$ git clone https://github.com/my/project
	[...]
	$ cd project
	$ cat Godeps/Godeps.json
	{
		"ImportPath": "github.com/my/project",
		"GoVersion": "go1.12",
		"GodepVersion": "v80",
		"Deps": [
			{
				"ImportPath": "rsc.io/binaryregexp",
				"Comment": "v0.2.0-1-g545cabd",
				"Rev": "545cabda89ca36b48b8e681a30d9d769a30b3074"
			},
			{
				"ImportPath": "rsc.io/binaryregexp/syntax",
				"Comment": "v0.2.0-1-g545cabd",
				"Rev": "545cabda89ca36b48b8e681a30d9d769a30b3074"
			}
		]
	}
	$ go mod init github.com/my/project
	go: creating new go.mod: module github.com/my/project
	go: copying requirements from Godeps/Godeps.json
	$ cat go.mod
	module github.com/my/project

	go 1.12

	require rsc.io/binaryregexp v0.2.1-0.20190524193500-545cabda89ca
	$

`go mod init` tạo tệp go.mod mới và tự động nhập các dependency từ `Godeps.json`,
`Gopkg.lock`, hoặc một số [định dạng được hỗ trợ khác](https://go.googlesource.com/go/+/362625209b6cd2bc059b6b0a67712ddebab312d9/src/cmd/go/internal/modconv/modconv.go#9).
Tham số truyền vào `go mod init` là đường dẫn module,
vị trí nơi module có thể được tìm thấy.

Đây là thời điểm tốt để dừng lại và chạy `go build ./...` và `go test ./...` trước khi tiếp tục.
Các bước tiếp theo có thể sửa đổi tệp `go.mod` của bạn,
vì vậy nếu bạn muốn thực hiện theo cách tiếp cận lặp dần,
đây là thời điểm gần nhất tệp `go.mod` của bạn sẽ giống với thông số dependency trước module.

	$ go mod tidy
	go: downloading rsc.io/binaryregexp v0.2.1-0.20190524193500-545cabda89ca
	go: extracting rsc.io/binaryregexp v0.2.1-0.20190524193500-545cabda89ca
	$ cat go.sum
	rsc.io/binaryregexp v0.2.1-0.20190524193500-545cabda89ca h1:FKXXXJ6G2bFoVe7hX3kEX6Izxw5ZKRH57DFBJmHCbkU=
	rsc.io/binaryregexp v0.2.1-0.20190524193500-545cabda89ca/go.mod h1:qTv7/COck+e2FymRvadv62gMdZztPaShugOCi3I+8D8=
	$

`go mod tidy` tìm tất cả các package được import theo dây chuyền bởi các package trong module của bạn.
Nó thêm các yêu cầu module mới cho các package không được cung cấp bởi bất kỳ module đã biết nào,
và nó xóa các yêu cầu đối với các module không cung cấp bất kỳ package được import nào.
Nếu một module cung cấp các package chỉ được import bởi các dự án chưa
chuyển đổi sang module,
yêu cầu module sẽ được đánh dấu bằng comment `// indirect`.
Luôn là thực hành tốt khi chạy `go mod tidy` trước khi commit tệp `go.mod`
vào hệ thống quản lý phiên bản.

Hãy kết thúc bằng cách đảm bảo code được build và các test pass:

	$ go build ./...
	$ go test ./...
	[...]
	$

Lưu ý rằng các công cụ quản lý dependency khác có thể chỉ định dependency ở mức
các package riêng lẻ hoặc toàn bộ kho lưu trữ (không phải module),
và thường không nhận ra các yêu cầu được chỉ định trong các tệp `go.mod`
của các dependency.
Do đó, bạn có thể không nhận được chính xác cùng phiên bản của mỗi package như trước,
và có một số rủi ro nâng cấp qua các thay đổi phá vỡ tương thích.
Vì vậy, điều quan trọng là tuân theo các lệnh trên với việc kiểm tra
các dependency kết quả. Để thực hiện điều đó, hãy chạy

	$ go list -m all
	go: finding rsc.io/binaryregexp v0.2.1-0.20190524193500-545cabda89ca
	github.com/my/project
	rsc.io/binaryregexp v0.2.1-0.20190524193500-545cabda89ca
	$

và so sánh các phiên bản kết quả với tệp quản lý dependency cũ của bạn
để đảm bảo rằng các phiên bản được chọn là phù hợp.
Nếu bạn tìm thấy một phiên bản không như bạn muốn,
bạn có thể tìm hiểu lý do bằng cách dùng `go mod why -m` và/hoặc `go mod graph`,
và nâng cấp hoặc hạ cấp xuống phiên bản đúng bằng cách dùng `go get`.
(Nếu phiên bản bạn yêu cầu cũ hơn phiên bản đã được chọn trước đó,
`go get` sẽ hạ cấp các dependency khác khi cần để duy trì tương thích.) Ví dụ,

	$ go mod why -m rsc.io/binaryregexp
	[...]
	$ go mod graph | grep rsc.io/binaryregexp
	[...]
	$ go get rsc.io/binaryregexp@v0.2.0
	$

## Không có công cụ quản lý dependency

Đối với một dự án Go không có hệ thống quản lý dependency, hãy bắt đầu bằng cách tạo tệp `go.mod`:

	$ git clone https://go.googlesource.com/blog
	[...]
	$ cd blog
	$ go mod init golang.org/x/blog
	go: creating new go.mod: module golang.org/x/blog
	$ cat go.mod
	module golang.org/x/blog

	go 1.12
	$

Không có tệp cấu hình từ công cụ quản lý dependency trước đó,
`go mod init` sẽ tạo tệp `go.mod` chỉ với các chỉ thị `module` và `go`.
Trong ví dụ này, chúng ta đặt đường dẫn module là `golang.org/x/blog` vì đó
là [đường dẫn import tùy chỉnh](/cmd/go/#hdr-Remote_import_paths) của nó.
Người dùng có thể import các package với đường dẫn này,
và chúng ta phải cẩn thận không thay đổi nó.

Chỉ thị `module` khai báo đường dẫn module,
và chỉ thị `go` khai báo phiên bản ngôn ngữ Go dự kiến
được dùng để biên dịch code trong module.

Tiếp theo, hãy chạy `go mod tidy` để thêm các dependency của module:

	$ go mod tidy
	go: finding golang.org/x/website latest
	go: finding gopkg.in/tomb.v2 latest
	go: finding golang.org/x/net latest
	go: finding golang.org/x/tools latest
	go: downloading github.com/gorilla/context v1.1.1
	go: downloading golang.org/x/tools v0.0.0-20190813214729-9dba7caff850
	go: downloading golang.org/x/net v0.0.0-20190813141303-74dc4d7220e7
	go: extracting github.com/gorilla/context v1.1.1
	go: extracting golang.org/x/net v0.0.0-20190813141303-74dc4d7220e7
	go: downloading gopkg.in/tomb.v2 v2.0.0-20161208151619-d5d1b5820637
	go: extracting gopkg.in/tomb.v2 v2.0.0-20161208151619-d5d1b5820637
	go: extracting golang.org/x/tools v0.0.0-20190813214729-9dba7caff850
	go: downloading golang.org/x/website v0.0.0-20190809153340-86a7442ada7c
	go: extracting golang.org/x/website v0.0.0-20190809153340-86a7442ada7c
	$ cat go.mod
	module golang.org/x/blog

	go 1.12

	require (
		github.com/gorilla/context v1.1.1
		golang.org/x/net v0.0.0-20190813141303-74dc4d7220e7
		golang.org/x/text v0.3.2
		golang.org/x/tools v0.0.0-20190813214729-9dba7caff850
		golang.org/x/website v0.0.0-20190809153340-86a7442ada7c
		gopkg.in/tomb.v2 v2.0.0-20161208151619-d5d1b5820637
	)
	$ cat go.sum
	cloud.google.com/go v0.26.0/go.mod h1:aQUYkXzVsufM+DwF1aE+0xfcU+56JwCaLick0ClmMTw=
	cloud.google.com/go v0.34.0/go.mod h1:aQUYkXzVsufM+DwF1aE+0xfcU+56JwCaLick0ClmMTw=
	git.apache.org/thrift.git v0.0.0-20180902110319-2566ecd5d999/go.mod h1:fPE2ZNJGynbRyZ4dJvy6G277gSllfV2HJqblrnkyeyg=
	git.apache.org/thrift.git v0.0.0-20181218151757-9b75e4fe745a/go.mod h1:fPE2ZNJGynbRyZ4dJvy6G277gSllfV2HJqblrnkyeyg=
	github.com/beorn7/perks v0.0.0-20180321164747-3a771d992973/go.mod h1:Dwedo/Wpr24TaqPxmxbtue+5NUziq4I4S80YR8gNf3Q=
	[...]
	$

`go mod tidy` đã thêm các yêu cầu module cho tất cả các package được import theo dây chuyền
bởi các package trong module của bạn và đã tạo `go.sum` với các checksum
cho mỗi thư viện tại một phiên bản cụ thể.
Hãy kết thúc bằng cách đảm bảo code vẫn được build và các test vẫn pass:

	$ go build ./...
	$ go test ./...
	ok  	golang.org/x/blog	0.335s
	?   	golang.org/x/blog/content/appengine	[no test files]
	ok  	golang.org/x/blog/content/cover	0.040s
	?   	golang.org/x/blog/content/h2push/server	[no test files]
	?   	golang.org/x/blog/content/survey2016	[no test files]
	?   	golang.org/x/blog/content/survey2017	[no test files]
	?   	golang.org/x/blog/support/racy	[no test files]
	$

Lưu ý rằng khi `go mod tidy` thêm một yêu cầu,
nó thêm phiên bản mới nhất của module.
Nếu `GOPATH` của bạn bao gồm một phiên bản cũ hơn của dependency mà sau đó
đã xuất bản một thay đổi phá vỡ tương thích,
bạn có thể thấy lỗi trong `go mod tidy`, `go build`, hoặc `go test`.
Nếu điều này xảy ra, hãy thử hạ cấp xuống phiên bản cũ hơn bằng `go get` (ví dụ,
`go get github.com/broken/module@v1.1.0`),
hoặc dành thời gian để làm cho module của bạn tương thích với phiên bản mới nhất của mỗi dependency.

### Các test trong chế độ module

Một số test có thể cần điều chỉnh sau khi chuyển đổi sang Go modules.

Nếu một test cần ghi tệp vào thư mục package,
nó có thể fail khi thư mục package nằm trong module cache, chỉ đọc.
Đặc biệt, điều này có thể gây ra `go test all` fail.
Test nên sao chép các tệp cần ghi vào thư mục tạm thời.

Nếu một test dựa vào đường dẫn tương đối (`../package-in-another-module`) để xác định vị trí
và đọc tệp trong package khác,
nó sẽ fail nếu package nằm trong module khác,
sẽ nằm trong thư mục con theo phiên bản của module cache hoặc
một đường dẫn được chỉ định trong chỉ thị `replace`.
Trong trường hợp này, bạn có thể cần sao chép đầu vào test vào module của bạn,
hoặc chuyển đổi đầu vào test từ tệp thô sang dữ liệu nhúng vào tệp nguồn `.go`.

Nếu một test kỳ vọng các lệnh `go` trong test chạy trong chế độ GOPATH, nó có thể fail.
Trong trường hợp này, bạn có thể cần thêm tệp `go.mod` vào cây nguồn cần test,
hoặc đặt `GO111MODULE=off` một cách rõ ràng.

## Xuất bản bản phát hành

Cuối cùng, bạn nên gắn thẻ và xuất bản phiên bản bản phát hành cho module mới của bạn.
Điều này là tùy chọn nếu bạn chưa phát hành bất kỳ phiên bản nào,
nhưng nếu không có bản phát hành chính thức, người dùng downstream sẽ phụ thuộc vào các
commit cụ thể sử dụng [pseudo-versions](/cmd/go/#hdr-Pseudo_versions),
có thể khó hỗ trợ hơn.

	$ git tag v1.2.0
	$ git push origin v1.2.0

Tệp `go.mod` mới của bạn định nghĩa một đường dẫn import chính thức cho module của bạn và thêm
các yêu cầu phiên bản tối thiểu mới. Nếu người dùng của bạn đã sử dụng đường dẫn import đúng,
và các dependency của bạn chưa có thay đổi phá vỡ tương thích, thì việc thêm
tệp `go.mod` là tương thích ngược, nhưng đây là một thay đổi đáng kể, và
có thể làm lộ ra các vấn đề hiện có. Nếu bạn có các thẻ phiên bản hiện có, bạn nên
tăng [phiên bản minor](https://semver.org/#spec-item-7). Hãy xem
[Xuất bản Go Modules](/blog/publishing-go-modules) để tìm hiểu cách tăng và
xuất bản phiên bản.

## Import và đường dẫn module chính thức

Mỗi module khai báo đường dẫn module của nó trong tệp `go.mod`.
Mỗi câu lệnh `import` tham chiếu đến một package trong module phải
có đường dẫn module như tiền tố của đường dẫn package.
Tuy nhiên, lệnh `go` có thể gặp một kho lưu trữ chứa module
thông qua nhiều [đường dẫn import từ xa](/cmd/go/#hdr-Remote_import_paths) khác nhau.
Ví dụ, cả `golang.org/x/lint` và `github.com/golang/lint` đều giải quyết
đến các kho lưu trữ chứa code được lưu trữ tại [go.googlesource.com/lint](https://go.googlesource.com/lint).
[Tệp `go.mod`](https://go.googlesource.com/lint/+/refs/heads/master/go.mod)
có trong kho lưu trữ đó khai báo đường dẫn của nó là `golang.org/x/lint`,
vì vậy chỉ có đường dẫn đó tương ứng với một module hợp lệ.

Go 1.4 cung cấp cơ chế để khai báo đường dẫn import chính thức bằng cách dùng [`// import` comments](/cmd/go/#hdr-Import_path_checking),
nhưng các tác giả package không phải lúc nào cũng cung cấp chúng.
Kết quả là, code viết trước khi có modules có thể đã sử dụng đường dẫn import không chính thức
cho một module mà không xuất hiện lỗi về sự không khớp.
Khi sử dụng modules, đường dẫn import phải khớp với đường dẫn module chính thức,
vì vậy bạn có thể cần cập nhật các câu lệnh `import`:
ví dụ, bạn có thể cần thay đổi `import "github.com/golang/lint"` thành
`import "golang.org/x/lint"`.

Một tình huống khác trong đó đường dẫn chính thức của module có thể khác với đường dẫn
kho lưu trữ xảy ra với các module Go ở phiên bản major 2 hoặc cao hơn.
Một module Go với phiên bản major trên 1 phải bao gồm hậu tố phiên bản major trong đường dẫn module:
ví dụ, phiên bản `v2.0.0` phải có hậu tố `/v2`.
Tuy nhiên, các câu lệnh `import` có thể đã tham chiếu đến các package trong
module _mà không có_ hậu tố đó.
Ví dụ, người dùng không phải module của `github.com/russross/blackfriday/v2` tại
`v2.0.1` có thể đã import nó là `github.com/russross/blackfriday`,
và sẽ cần cập nhật đường dẫn import để bao gồm hậu tố `/v2`.

## Kết luận

Chuyển đổi sang Go modules nên là một quá trình đơn giản đối với hầu hết người dùng.
Các vấn đề không thường xuyên có thể phát sinh do đường dẫn import không chính thức hoặc các thay đổi
phá vỡ tương thích trong một dependency.
Các bài viết tương lai sẽ khám phá [xuất bản các phiên bản mới](/blog/publishing-go-modules),
v2 và xa hơn, và các cách để gỡ lỗi các tình huống kỳ lạ.

Để cung cấp phản hồi và giúp định hình tương lai của quản lý dependency trong Go,
vui lòng gửi cho chúng tôi [báo cáo lỗi](/issue/new) hoặc [báo cáo kinh nghiệm](/wiki/ExperienceReports).

Cảm ơn tất cả phản hồi và sự giúp đỡ của bạn trong việc cải thiện modules.
