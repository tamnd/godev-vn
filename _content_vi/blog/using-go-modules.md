---
title: Sử dụng Go Modules
date: 2019-03-19
by:
- Tyler Bui-Palsulich
- Eno Compton
tags:
- tools
- versioning
summary: Giới thiệu các thao tác cơ bản để bắt đầu sử dụng Go modules.
---

## Giới thiệu

Bài viết này là phần 1 trong một loạt bài.

  - **Phần 1 — Sử dụng Go Modules** (bài này)
  - Phần 2 — [Chuyển đổi sang Go Modules](/blog/migrating-to-go-modules)
  - Phần 3 — [Xuất bản Go Modules](/blog/publishing-go-modules)
  - Phần 4 — [Go Modules: v2 và Xa hơn](/blog/v2-go-modules)
  - Phần 5 — [Giữ Modules tương thích](/blog/module-compatibility)

**Lưu ý:** Để xem tài liệu về quản lý dependency với modules, hãy xem
[Quản lý dependency](/doc/modules/managing-dependencies).

Go 1.11 và 1.12 bao gồm [hỗ trợ sơ bộ cho modules](/doc/go1.11#modules),
[hệ thống quản lý dependency mới](/blog/versioning-proposal) của Go
giúp thông tin phiên bản dependency trở nên tường minh hơn và dễ quản lý hơn.
Bài blog này giới thiệu các thao tác cơ bản cần thiết để bắt đầu sử dụng modules.

Module là một tập hợp các [gói Go](/ref/spec#Packages)
được lưu trong một cây tệp với một tệp `go.mod` ở gốc.
Tệp `go.mod` định nghĩa _đường dẫn module_ của module, đây cũng là đường dẫn import được dùng cho thư mục gốc, và _các yêu cầu dependency_, là các module khác cần thiết để build thành công.
Mỗi yêu cầu dependency được viết dưới dạng một đường dẫn module và một
[phiên bản ngữ nghĩa](http://semver.org/) cụ thể.

Từ Go 1.11, lệnh go cho phép sử dụng modules khi thư mục hiện tại hoặc bất kỳ thư mục cha nào có `go.mod`, miễn là thư mục đó nằm _ngoài_ `$GOPATH/src`.
(Trong `$GOPATH/src`, để tương thích, lệnh go vẫn chạy ở chế độ GOPATH cũ, ngay cả khi tìm thấy `go.mod`. Xem [tài liệu lệnh go](/cmd/go/#hdr-Preliminary_module_support) để biết chi tiết.)
Bắt đầu từ Go 1.13, chế độ module sẽ là mặc định cho tất cả quá trình phát triển.

Bài viết này hướng dẫn một loạt các thao tác phổ biến xuất hiện khi phát triển code Go với modules:

  - Tạo một module mới.
  - Thêm một dependency.
  - Nâng cấp dependency.
  - Thêm dependency vào một major version mới.
  - Nâng cấp dependency lên một major version mới.
  - Xóa dependency không dùng.

## Tạo một module mới

Hãy tạo một module mới.

Tạo một thư mục mới, trống ở đâu đó ngoài `$GOPATH/src`,
`cd` vào thư mục đó, rồi tạo một tệp nguồn mới, `hello.go`:

	package hello

	func Hello() string {
		return "Hello, world."
	}

Hãy viết một bài test cũng vậy, trong `hello_test.go`:

	package hello

	import "testing"

	func TestHello(t *testing.T) {
		want := "Hello, world."
		if got := Hello(); got != want {
			t.Errorf("Hello() = %q, want %q", got, want)
		}
	}

Lúc này, thư mục chứa một gói, nhưng không phải một module,
vì không có tệp `go.mod`.
Nếu chúng ta đang làm việc trong `/home/gopher/hello` và chạy `go test` ngay bây giờ,
chúng ta sẽ thấy:

	$ go test
	PASS
	ok  	_/home/gopher/hello	0.020s
	$

Dòng cuối tóm tắt kết quả test tổng thể của gói.
Vì chúng ta đang làm việc ngoài `$GOPATH` và cũng ngoài bất kỳ module nào,
lệnh `go` không biết đường dẫn import cho thư mục hiện tại và tạo ra một đường dẫn giả dựa trên tên thư mục: `_/home/gopher/hello`.

Hãy làm cho thư mục hiện tại trở thành gốc của một module bằng cách dùng `go mod init` rồi thử `go test` lại:

	$ go mod init example.com/hello
	go: creating new go.mod: module example.com/hello
	$ go test
	PASS
	ok  	example.com/hello	0.020s
	$

Chúc mừng! Bạn đã viết và test module đầu tiên của mình.

Lệnh `go mod init` đã viết tệp `go.mod`:

	$ cat go.mod
	module example.com/hello

	go 1.12
	$

Tệp `go.mod` chỉ xuất hiện ở gốc của module.
Các gói trong thư mục con có đường dẫn import gồm đường dẫn module cộng với đường dẫn đến thư mục con.
Ví dụ, nếu chúng ta tạo một thư mục con `world`,
chúng ta không cần (và cũng không nên) chạy `go mod init` ở đó.
Gói sẽ tự động được nhận ra là một phần của module `example.com/hello`, với đường dẫn import `example.com/hello/world`.

## Thêm một dependency

Động lực chính của Go modules là cải thiện trải nghiệm sử dụng (tức là thêm dependency vào) code được viết bởi các nhà phát triển khác.

Hãy cập nhật `hello.go` để import `rsc.io/quote` và dùng nó để cài đặt `Hello`:

	package hello

	import "rsc.io/quote"

	func Hello() string {
		return quote.Hello()
	}

Bây giờ hãy chạy lại test:

	$ go test
	go: finding rsc.io/quote v1.5.2
	go: downloading rsc.io/quote v1.5.2
	go: extracting rsc.io/quote v1.5.2
	go: finding rsc.io/sampler v1.3.0
	go: finding golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
	go: downloading rsc.io/sampler v1.3.0
	go: extracting rsc.io/sampler v1.3.0
	go: downloading golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
	go: extracting golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
	PASS
	ok  	example.com/hello	0.023s
	$

Lệnh `go` phân giải các import bằng cách sử dụng các phiên bản module dependency cụ thể được liệt kê trong `go.mod`.
Khi gặp một `import` của gói không được cung cấp bởi bất kỳ module nào trong `go.mod`, lệnh `go` tự động tìm kiếm module chứa gói đó và thêm vào `go.mod`, sử dụng phiên bản mới nhất.
("Mới nhất" được định nghĩa là phiên bản ổn định được gắn thẻ mới nhất (không phải [tiền phát hành](https://semver.org/#spec-item-9)), hoặc phiên bản tiền phát hành được gắn thẻ mới nhất, hoặc phiên bản chưa được gắn thẻ mới nhất.)
Trong ví dụ của chúng ta, `go test` đã phân giải import mới `rsc.io/quote` thành module `rsc.io/quote v1.5.2`.
Nó cũng tải xuống hai dependency được sử dụng bởi `rsc.io/quote`, cụ thể là `rsc.io/sampler` và `golang.org/x/text`.
Chỉ các dependency trực tiếp được ghi lại trong tệp `go.mod`:

	$ cat go.mod
	module example.com/hello

	go 1.12

	require rsc.io/quote v1.5.2
	$

Một lệnh `go test` thứ hai sẽ không lặp lại công việc này,
vì `go.mod` đã được cập nhật và các module đã tải xuống được cache cục bộ (trong `$GOPATH/pkg/mod`):

	$ go test
	PASS
	ok  	example.com/hello	0.020s
	$

Lưu ý rằng mặc dù lệnh `go` giúp thêm dependency mới nhanh chóng và dễ dàng, nhưng nó không phải là không có chi phí.
Module của bạn giờ đây _phụ thuộc_ trực tiếp vào dependency mới trong các lĩnh vực quan trọng như độ chính xác, bảo mật và cấp phép hợp lệ, chỉ kể vài điều.
Để biết thêm các cân nhắc, hãy xem bài blog của Russ Cox, "[Our Software Dependency Problem](https://research.swtch.com/deps)."

Như chúng ta đã thấy ở trên, việc thêm một dependency trực tiếp thường kéo theo các dependency gián tiếp khác.
Lệnh `go list -m all` liệt kê module hiện tại và tất cả dependency của nó:

	$ go list -m all
	example.com/hello
	golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
	rsc.io/quote v1.5.2
	rsc.io/sampler v1.3.0
	$

Trong đầu ra của `go list`, module hiện tại, còn được gọi là _main module_, luôn là dòng đầu tiên, tiếp theo là các dependency được sắp xếp theo đường dẫn module.

Phiên bản `v0.0.0-20170915032832-14c0d48ead0c` của `golang.org/x/text` là ví dụ về [pseudo-version](/ref/mod#pseudo-versions), là cú pháp phiên bản của lệnh `go` cho một commit chưa được gắn thẻ cụ thể.

Ngoài `go.mod`, lệnh `go` duy trì một tệp tên `go.sum` chứa [các hash mật mã](/cmd/go/#hdr-Module_downloading_and_verification) dự kiến của nội dung các phiên bản module cụ thể:

	$ cat go.sum
	golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c h1:qgOY6WgZO...
	golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c/go.mod h1:Nq...
	rsc.io/quote v1.5.2 h1:w5fcysjrx7yqtD/aO+QwRjYZOKnaM9Uh2b40tElTs3...
	rsc.io/quote v1.5.2/go.mod h1:LzX7hefJvL54yjefDEDHNONDjII0t9xZLPX...
	rsc.io/sampler v1.3.0 h1:7uVkIFmeBqHfdjD+gZwtXXI+RODJ2Wc4O7MPEh/Q...
	rsc.io/sampler v1.3.0/go.mod h1:T1hPZKmBbMNahiBKFy5HrXp6adAjACjK9...
	$

Lệnh `go` sử dụng tệp `go.sum` để đảm bảo rằng các lần tải xuống trong tương lai của các module này sẽ lấy về các bit giống với lần tải xuống đầu tiên, nhằm đảm bảo các module mà dự án của bạn phụ thuộc không thay đổi ngoài mong muốn, dù vì lý do độc hại, vô tình hay bất kỳ lý do nào khác.
Cả `go.mod` và `go.sum` đều nên được đưa vào kiểm soát phiên bản.

## Nâng cấp dependency

Với Go modules, các phiên bản được tham chiếu bằng các thẻ phiên bản ngữ nghĩa.
Một phiên bản ngữ nghĩa có ba phần: major, minor và patch.
Ví dụ, với `v0.1.2`, phiên bản major là 0, phiên bản minor là 1, và phiên bản patch là 2.
Hãy cùng thực hiện một vài nâng cấp phiên bản minor.
Ở phần tiếp theo, chúng ta sẽ xem xét nâng cấp major version.

Từ đầu ra của `go list -m all`, chúng ta có thể thấy mình đang dùng phiên bản chưa được gắn thẻ của `golang.org/x/text`.
Hãy nâng cấp lên phiên bản được gắn thẻ mới nhất và test xem mọi thứ vẫn hoạt động không:

	$ go get golang.org/x/text
	go: finding golang.org/x/text v0.3.0
	go: downloading golang.org/x/text v0.3.0
	go: extracting golang.org/x/text v0.3.0
	$ go test
	PASS
	ok  	example.com/hello	0.013s
	$

Tuyệt vời! Mọi thứ đều pass.
Hãy xem lại `go list -m all` và tệp `go.mod`:

	$ go list -m all
	example.com/hello
	golang.org/x/text v0.3.0
	rsc.io/quote v1.5.2
	rsc.io/sampler v1.3.0
	$ cat go.mod
	module example.com/hello

	go 1.12

	require (
		golang.org/x/text v0.3.0 // indirect
		rsc.io/quote v1.5.2
	)
	$

Gói `golang.org/x/text` đã được nâng cấp lên phiên bản được gắn thẻ mới nhất (`v0.3.0`).
Tệp `go.mod` cũng đã được cập nhật để chỉ định `v0.3.0`.
Chú thích `indirect` cho biết một dependency không được module này sử dụng trực tiếp, mà chỉ gián tiếp qua các dependency module khác.
Xem `go help modules` để biết chi tiết.

Bây giờ hãy thử nâng cấp phiên bản minor của `rsc.io/sampler`.
Bắt đầu tương tự, bằng cách chạy `go get` và chạy test:

	$ go get rsc.io/sampler
	go: finding rsc.io/sampler v1.99.99
	go: downloading rsc.io/sampler v1.99.99
	go: extracting rsc.io/sampler v1.99.99
	$ go test
	--- FAIL: TestHello (0.00s)
	    hello_test.go:8: Hello() = "99 bottles of beer on the wall, 99 bottles of beer, ...", want "Hello, world."
	FAIL
	exit status 1
	FAIL	example.com/hello	0.014s
	$

Ôi! Lỗi test cho thấy phiên bản mới nhất của `rsc.io/sampler` không tương thích với cách sử dụng của chúng ta.
Hãy liệt kê các phiên bản được gắn thẻ của module đó:

	$ go list -m -versions rsc.io/sampler
	rsc.io/sampler v1.0.0 v1.2.0 v1.2.1 v1.3.0 v1.3.1 v1.99.99
	$

Chúng ta đã dùng v1.3.0; v1.99.99 rõ ràng là không phù hợp.
Có lẽ chúng ta có thể thử dùng v1.3.1:

	$ go get rsc.io/sampler@v1.3.1
	go: finding rsc.io/sampler v1.3.1
	go: downloading rsc.io/sampler v1.3.1
	go: extracting rsc.io/sampler v1.3.1
	$ go test
	PASS
	ok  	example.com/hello	0.022s
	$

Lưu ý `@v1.3.1` tường minh trong đối số `go get`.
Nói chung, mỗi đối số truyền cho `go get` có thể có một phiên bản tường minh; mặc định là `@latest`, phân giải thành phiên bản mới nhất như đã định nghĩa trước đó.

## Thêm dependency vào một major version mới

Hãy thêm một hàm mới vào gói của chúng ta:
`func Proverb` trả về một câu châm ngôn về tính đồng thời của Go,
bằng cách gọi `quote.Concurrency`, được cung cấp bởi module `rsc.io/quote/v3`.
Đầu tiên, cập nhật `hello.go` để thêm hàm mới:

	package hello

	import (
		"rsc.io/quote"
		quoteV3 "rsc.io/quote/v3"
	)

	func Hello() string {
		return quote.Hello()
	}

	func Proverb() string {
		return quoteV3.Concurrency()
	}

Sau đó, thêm một test vào `hello_test.go`:

	func TestProverb(t *testing.T) {
		want := "Concurrency is not parallelism."
		if got := Proverb(); got != want {
			t.Errorf("Proverb() = %q, want %q", got, want)
		}
	}

Rồi chạy test:

	$ go test
	go: finding rsc.io/quote/v3 v3.1.0
	go: downloading rsc.io/quote/v3 v3.1.0
	go: extracting rsc.io/quote/v3 v3.1.0
	PASS
	ok  	example.com/hello	0.024s
	$

Lưu ý rằng module của chúng ta hiện phụ thuộc vào cả `rsc.io/quote` lẫn `rsc.io/quote/v3`:

	$ go list -m rsc.io/q...
	rsc.io/quote v1.5.2
	rsc.io/quote/v3 v3.1.0
	$

Mỗi major version khác nhau (`v1`, `v2`, v.v.) của một Go module dùng một đường dẫn module khác nhau: bắt đầu từ `v2`, đường dẫn phải kết thúc bằng major version.
Trong ví dụ, `v3` của `rsc.io/quote` không còn là `rsc.io/quote` nữa: thay vào đó, nó được xác định bởi đường dẫn module `rsc.io/quote/v3`.
Quy ước này được gọi là [đặt tên import theo phiên bản ngữ nghĩa](https://research.swtch.com/vgo-import), và nó đặt tên khác nhau cho các gói không tương thích (những gói có major version khác nhau).
Ngược lại, `v1.6.0` của `rsc.io/quote` nên tương thích ngược với `v1.5.2`, vì vậy nó tái sử dụng tên `rsc.io/quote`.
(Ở phần trước, `rsc.io/sampler` `v1.99.99` _đáng lẽ_ phải tương thích ngược với `rsc.io/sampler` `v1.3.0`, nhưng lỗi hoặc giả định sai về hành vi module của client đều có thể xảy ra.)

Lệnh `go` cho phép một build bao gồm nhiều nhất một phiên bản của bất kỳ đường dẫn module cụ thể nào, nghĩa là nhiều nhất một trong mỗi major version: một `rsc.io/quote`, một `rsc.io/quote/v2`, một `rsc.io/quote/v3`, v.v.
Điều này cung cấp cho các tác giả module một quy tắc rõ ràng về khả năng trùng lặp của một đường dẫn module: không thể xây dựng một chương trình với cả `rsc.io/quote v1.5.2` và `rsc.io/quote v1.6.0` cùng lúc.
Đồng thời, cho phép các major version khác nhau của một module (vì chúng có đường dẫn khác nhau) cung cấp cho người dùng module khả năng nâng cấp lên major version mới từng bước.
Trong ví dụ này, chúng ta muốn dùng `quote.Concurrency` từ `rsc/quote/v3 v3.1.0` nhưng chưa sẵn sàng chuyển đổi các lần sử dụng `rsc.io/quote v1.5.2`.
Khả năng chuyển đổi từng bước đặc biệt quan trọng trong một chương trình hoặc codebase lớn.

## Nâng cấp dependency lên một major version mới

Hãy hoàn thành việc chuyển đổi từ `rsc.io/quote` sang chỉ dùng `rsc.io/quote/v3`.
Do thay đổi major version, chúng ta nên dự kiến rằng một số API có thể đã bị xóa, đổi tên hoặc thay đổi theo cách không tương thích.
Đọc tài liệu, chúng ta có thể thấy `Hello` đã trở thành `HelloV3`:

	$ go doc rsc.io/quote/v3
	package quote // import "rsc.io/quote/v3"

	Package quote collects pithy sayings.

	func Concurrency() string
	func GlassV3() string
	func GoV3() string
	func HelloV3() string
	func OptV3() string
	$

Chúng ta có thể cập nhật cách dùng `quote.Hello()` trong `hello.go` để dùng `quoteV3.HelloV3()`:

	package hello

	import quoteV3 "rsc.io/quote/v3"

	func Hello() string {
		return quoteV3.HelloV3()
	}

	func Proverb() string {
		return quoteV3.Concurrency()
	}

Rồi ở thời điểm này, không cần alias import đổi tên nữa, vì vậy chúng ta có thể hoàn tác:

	package hello

	import "rsc.io/quote/v3"

	func Hello() string {
		return quote.HelloV3()
	}

	func Proverb() string {
		return quote.Concurrency()
	}

Hãy chạy lại test để đảm bảo mọi thứ hoạt động:

	$ go test
	PASS
	ok      example.com/hello       0.014s

## Xóa dependency không dùng

Chúng ta đã xóa tất cả các lần dùng `rsc.io/quote`, nhưng nó vẫn xuất hiện trong `go list -m all` và trong tệp `go.mod`:

	$ go list -m all
	example.com/hello
	golang.org/x/text v0.3.0
	rsc.io/quote v1.5.2
	rsc.io/quote/v3 v3.1.0
	rsc.io/sampler v1.3.1
	$ cat go.mod
	module example.com/hello

	go 1.12

	require (
		golang.org/x/text v0.3.0 // indirect
		rsc.io/quote v1.5.2
		rsc.io/quote/v3 v3.0.0
		rsc.io/sampler v1.3.1 // indirect
	)
	$

Tại sao? Vì việc build một gói đơn lẻ, như với `go build` hay `go test`, có thể dễ dàng biết khi nào thiếu thứ gì đó và cần được thêm vào, nhưng không biết khi nào một thứ gì đó có thể được xóa an toàn.
Việc xóa một dependency chỉ có thể thực hiện sau khi kiểm tra tất cả các gói trong một module và tất cả các kết hợp build tag có thể cho các gói đó.
Một lệnh build thông thường không tải thông tin này, vì vậy nó không thể xóa dependency một cách an toàn.

Lệnh `go mod tidy` dọn sạch các dependency không dùng này:

	$ go mod tidy
	$ go list -m all
	example.com/hello
	golang.org/x/text v0.3.0
	rsc.io/quote/v3 v3.1.0
	rsc.io/sampler v1.3.1
	$ cat go.mod
	module example.com/hello

	go 1.12

	require (
		golang.org/x/text v0.3.0 // indirect
		rsc.io/quote/v3 v3.1.0
		rsc.io/sampler v1.3.1 // indirect
	)

	$ go test
	PASS
	ok  	example.com/hello	0.020s
	$

## Kết luận

Go modules là tương lai của quản lý dependency trong Go.
Chức năng module hiện có sẵn trong tất cả các phiên bản Go được hỗ trợ (tức là Go 1.11 và Go 1.12).

Bài viết này đã giới thiệu các quy trình làm việc sau với Go modules:

  - `go mod init` tạo một module mới, khởi tạo tệp `go.mod` mô tả nó.
  - `go build`, `go test` và các lệnh build gói khác thêm dependency mới vào `go.mod` khi cần.
  - `go list -m all` in ra các dependency của module hiện tại.
  - `go get` thay đổi phiên bản yêu cầu của một dependency (hoặc thêm một dependency mới).
  - `go mod tidy` xóa các dependency không dùng.

Chúng tôi khuyến khích bạn bắt đầu sử dụng modules trong phát triển cục bộ và thêm các tệp `go.mod` và `go.sum` vào dự án của bạn.
Để cung cấp phản hồi và giúp định hình tương lai của quản lý dependency trong Go, hãy gửi cho chúng tôi
[báo cáo lỗi](/issue/new) hoặc [báo cáo trải nghiệm](/wiki/ExperienceReports).

Cảm ơn tất cả các phản hồi và sự hỗ trợ của bạn trong việc cải thiện modules.
