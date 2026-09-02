---
title: Phát hành các Go Module
date: 2019-09-26
by:
- Tyler Bui-Palsulich
tags:
- tools
- versioning
summary: Cách viết và phát hành module để dùng làm dependency.
---

## Giới thiệu

Bài đăng này là phần 3 trong một loạt bài.

  - Phần 1 — [Sử dụng Go Module](/blog/using-go-modules)
  - Phần 2 — [Chuyển đổi sang Go Module](/blog/migrating-to-go-modules)
  - **Phần 3 — Phát hành các Go Module** (bài đăng này)
  - Phần 4 — [Go Module: v2 và Hơn thế nữa](/blog/v2-go-modules)
  - Phần 5 — [Giữ cho các Module của bạn Tương thích](/blog/module-compatibility)

**Lưu ý:** Để biết tài liệu về phát triển module, hãy xem
[Phát triển và phát hành module](/doc/modules/developing).

Bài đăng này thảo luận về cách viết và phát hành module để các module khác có
thể phụ thuộc vào chúng.

Lưu ý: bài đăng này bao gồm quá trình phát triển đến và bao gồm `v1`. Nếu bạn
quan tâm đến `v2`, hãy xem [Go Module: v2 và Hơn thế nữa](/blog/v2-go-modules).

Bài đăng này sử dụng [Git](https://git-scm.com/) trong các ví dụ.
[Mercurial](https://www.mercurial-scm.org/),
[Bazaar](http://wiki.bazaar.canonical.com/), và các hệ thống khác cũng được
hỗ trợ.

## Thiết lập dự án

Với bài đăng này, bạn sẽ cần một dự án có sẵn để làm ví dụ. Vì vậy, hãy bắt
đầu với các tệp từ cuối bài viết
[Sử dụng Go Module](/blog/using-go-modules):

	$ cat go.mod
	module example.com/hello

	go 1.12

	require rsc.io/quote/v3 v3.1.0

	$ cat go.sum
	golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c h1:qgOY6WgZOaTkIIMiVjBQcw93ERBE4m30iBm00nkL0i8=
	golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c/go.mod h1:NqM8EUOU14njkJ3fqMW+pc6Ldnwhi/IjpwHt7yyuwOQ=
	rsc.io/quote/v3 v3.1.0 h1:9JKUTTIUgS6kzR9mK1YuGKv6Nl+DijDNIc0ghT58FaY=
	rsc.io/quote/v3 v3.1.0/go.mod h1:yEA65RcK8LyAZtP9Kv3t0HmxON59tX3rD+tICJqUlj0=
	rsc.io/sampler v1.3.0 h1:7uVkIFmeBqHfdjD+gZwtXXI+RODJ2Wc4O7MPEh/QiW4=
	rsc.io/sampler v1.3.0/go.mod h1:T1hPZKmBbMNahiBKFy5HrXp6adAjACjK9JXDnKaTXpA=

	$ cat hello.go
	package hello

	import "rsc.io/quote/v3"

	func Hello() string {
		return quote.HelloV3()
	}

	func Proverb() string {
		return quote.Concurrency()
	}

	$ cat hello_test.go
	package hello

	import (
		"testing"
	)

	func TestHello(t *testing.T) {
		want := "Hello, world."
		if got := Hello(); got != want {
			t.Errorf("Hello() = %q, want %q", got, want)
		}
	}

	func TestProverb(t *testing.T) {
		want := "Concurrency is not parallelism."
		if got := Proverb(); got != want {
			t.Errorf("Proverb() = %q, want %q", got, want)
		}
	}

	$

Tiếp theo, tạo một kho lưu trữ `git` mới và thêm một commit ban đầu. Nếu bạn
đang phát hành dự án của riêng mình, hãy chắc chắn bao gồm một tệp `LICENSE`.
Chuyển đến thư mục chứa `go.mod` rồi tạo kho lưu trữ:

	$ git init
	$ git add LICENSE go.mod go.sum hello.go hello_test.go
	$ git commit -m "hello: initial commit"
	$

## Phiên bản ngữ nghĩa và module

Mỗi module bắt buộc trong `go.mod` có một
[phiên bản ngữ nghĩa](https://semver.org), phiên bản tối thiểu của dependency
đó để sử dụng khi build module.

Một phiên bản ngữ nghĩa có dạng `vMAJOR.MINOR.PATCH`.

  - Tăng phiên bản `MAJOR` khi bạn thực hiện một thay đổi
    [không tương thích ngược](/doc/go1compat) đối với API công khai của module.
    Điều này chỉ nên được thực hiện khi thực sự cần thiết.
  - Tăng phiên bản `MINOR` khi bạn thực hiện một thay đổi tương thích ngược
    với API, như thay đổi dependency hoặc thêm một hàm, phương thức, trường
    struct hoặc kiểu mới.
  - Tăng phiên bản `PATCH` sau khi thực hiện các thay đổi nhỏ không ảnh hưởng
    đến API công khai hoặc dependency của module, như sửa một lỗi.

Bạn có thể chỉ định các phiên bản pre-release bằng cách thêm dấu gạch nối và
các mã định danh được phân tách bằng dấu chấm (ví dụ: `v1.0.1-alpha` hoặc
`v2.2.2-beta.2`). Các bản phát hành thông thường được lệnh `go` ưu tiên hơn
các phiên bản pre-release, vì vậy người dùng phải yêu cầu các phiên bản
pre-release một cách rõ ràng (ví dụ: `go get example.com/hello@v1.0.1-alpha`)
nếu module của bạn có bất kỳ bản phát hành thông thường nào.

Các phiên bản major `v0` và phiên bản pre-release không đảm bảo tính tương
thích ngược. Chúng cho phép bạn tinh chỉnh API trước khi cam kết ổn định với
người dùng. Tuy nhiên, các phiên bản major `v1` và cao hơn yêu cầu tương thích
ngược trong phiên bản major đó.

Phiên bản được tham chiếu trong `go.mod` có thể là một bản phát hành tường
minh được đánh dấu trong kho lưu trữ (ví dụ: `v1.5.2`), hoặc có thể là một
[pseudo-version](/ref/mod#pseudo-versions) dựa trên một commit cụ thể (ví dụ:
`v0.0.0-20170915032832-14c0d48ead0c`). Pseudo-version là một loại phiên bản
pre-release đặc biệt. Pseudo-version hữu ích khi người dùng cần phụ thuộc vào
một dự án chưa phát hành bất kỳ tag phiên bản ngữ nghĩa nào, hoặc phát triển
dựa trên một commit chưa được đánh tag, nhưng người dùng không nên giả định
rằng pseudo-version cung cấp một API ổn định hoặc đã được kiểm tra kỹ lưỡng.
Việc đánh tag module của bạn với các phiên bản tường minh báo hiệu cho người
dùng rằng các phiên bản cụ thể đã được kiểm tra đầy đủ và sẵn sàng sử dụng.

Một khi bạn bắt đầu đánh tag kho lưu trữ của mình với các phiên bản, điều quan
trọng là tiếp tục đánh tag các bản phát hành mới khi bạn phát triển module.
Khi người dùng yêu cầu một phiên bản mới của module (với `go get -u` hoặc
`go get example.com/hello`), lệnh `go` sẽ chọn phiên bản phát hành ngữ nghĩa
lớn nhất hiện có, ngay cả khi phiên bản đó là từ vài năm trước và có nhiều
thay đổi đằng sau nhánh chính. Tiếp tục đánh tag các bản phát hành mới sẽ làm
cho các cải tiến liên tục của bạn có sẵn cho người dùng.

Đừng xóa các tag phiên bản khỏi kho lưu trữ của bạn. Nếu bạn tìm thấy một lỗi
hoặc vấn đề bảo mật với một phiên bản, hãy phát hành một phiên bản mới. Nếu
người ta phụ thuộc vào một phiên bản bạn đã xóa, build của họ có thể thất bại.
Tương tự, một khi bạn đã phát hành một phiên bản, đừng thay đổi hoặc ghi đè
lên nó. [Mirror module và cơ sở dữ liệu checksum](/blog/module-mirror-launch)
lưu trữ các module, phiên bản của chúng và các hash mã hóa có chữ ký để đảm
bảo rằng việc build một phiên bản nhất định vẫn có thể tái tạo theo thời gian.

## v0: phiên bản ban đầu, chưa ổn định

Hãy đánh tag module với phiên bản ngữ nghĩa `v0`. Phiên bản `v0` không đảm
bảo ổn định, vì vậy hầu hết các dự án nên bắt đầu với `v0` khi tinh chỉnh
API công khai của mình.

Việc đánh tag một phiên bản mới có một vài bước:

1. Chạy `go mod tidy`, để xóa bất kỳ dependency nào mà module có thể đã tích
   lũy nhưng không còn cần thiết nữa.

2. Chạy `go test ./...` lần cuối để đảm bảo mọi thứ đang hoạt động.

3. Đánh tag dự án với một phiên bản mới sử dụng [`git tag`](https://git-scm.com/docs/git-tag).

4. Đẩy tag mới lên kho lưu trữ gốc.

```
$ go mod tidy
$ go test ./...
ok      example.com/hello       0.015s
$ git add go.mod go.sum hello.go hello_test.go
$ git commit -m "hello: changes for v0.1.0"
$ git tag v0.1.0
$ git push origin v0.1.0
$
```

Bây giờ các dự án khác có thể phụ thuộc vào `v0.1.0` của `example.com/hello`.
Đối với module của riêng bạn, bạn có thể chạy `go list -m example.com/hello@v0.1.0`
để xác nhận phiên bản mới nhất có sẵn (module ví dụ này không tồn tại, vì vậy
không có phiên bản nào). Nếu bạn không thấy phiên bản mới nhất ngay lập tức
và bạn đang sử dụng Go module proxy (mặc định kể từ Go 1.13), hãy thử lại sau
vài phút để cho proxy có thời gian tải phiên bản mới.

Nếu bạn thêm vào API công khai, thực hiện một thay đổi breaking đối với module
`v0`, hoặc nâng cấp phiên bản minor hoặc version của một trong các dependency,
hãy tăng phiên bản `MINOR` cho bản phát hành tiếp theo. Ví dụ: bản phát hành
tiếp theo sau `v0.1.0` sẽ là `v0.2.0`.

Nếu bạn sửa một lỗi trong một phiên bản hiện có, hãy tăng phiên bản `PATCH`.
Ví dụ: bản phát hành tiếp theo sau `v0.1.0` sẽ là `v0.1.1`.

## v1: phiên bản ổn định đầu tiên

Khi bạn hoàn toàn chắc chắn rằng API của module ổn định, bạn có thể phát hành
`v1.0.0`. Phiên bản major `v1` thông báo cho người dùng rằng sẽ không có thay
đổi không tương thích nào đối với API của module. Họ có thể nâng cấp lên các
bản phát hành minor và patch `v1` mới, và code của họ không nên bị hỏng. Các
chữ ký hàm và phương thức sẽ không thay đổi, các kiểu được export sẽ không
bị xóa, và tương tự như vậy. Nếu có thay đổi với API, chúng sẽ tương thích
ngược (ví dụ: thêm một trường mới vào một struct) và sẽ được bao gồm trong một
bản phát hành minor mới. Nếu có sửa lỗi (ví dụ: sửa bảo mật), chúng sẽ được
bao gồm trong một bản phát hành patch (hoặc như một phần của bản phát hành minor).

Đôi khi, duy trì tính tương thích ngược có thể dẫn đến các API bất tiện. Điều
đó không sao. Một API không hoàn hảo tốt hơn là phá vỡ code hiện có của người
dùng.

Gói `strings` của thư viện chuẩn là một ví dụ điển hình về việc duy trì tính
tương thích ngược với chi phí là tính nhất quán của API.

  - [`Split`](https://godoc.org/strings#Split) chia một chuỗi thành tất cả các
    chuỗi con được phân tách bởi một dấu phân cách và trả về một slice của các
    chuỗi con giữa các dấu phân cách đó.
  - [`SplitN`](https://godoc.org/strings#SplitN) có thể được sử dụng để kiểm
    soát số lượng chuỗi con cần trả về.

Tuy nhiên, [`Replace`](https://godoc.org/strings#Replace) lấy một số lần đếm
bao nhiêu thể hiện của chuỗi cần thay thế từ đầu (không giống `Split`).

Với `Split` và `SplitN`, bạn sẽ mong đợi các hàm như `Replace` và `ReplaceN`.
Nhưng, chúng tôi không thể thay đổi `Replace` hiện có mà không phá vỡ các
caller, điều mà chúng tôi đã hứa không làm. Vì vậy, trong Go 1.12, chúng tôi
đã thêm một hàm mới,
[`ReplaceAll`](https://godoc.org/strings#ReplaceAll). API kết quả có hơi khác
một chút, vì `Split` và `Replace` hoạt động khác nhau, nhưng sự không nhất quán
đó tốt hơn là một thay đổi breaking.

Giả sử bạn hài lòng với API của `example.com/hello` và muốn phát hành `v1` là
phiên bản ổn định đầu tiên.

Việc đánh tag `v1` sử dụng quy trình tương tự như đánh tag một phiên bản `v0`:
chạy `go mod tidy` và `go test ./...`, đánh tag phiên bản, và đẩy tag lên kho
lưu trữ gốc:

	$ go mod tidy
	$ go test ./...
	ok      example.com/hello       0.015s
	$ git add go.mod go.sum hello.go hello_test.go
	$ git commit -m "hello: changes for v1.0.0"
	$ git tag v1.0.0
	$ git push origin v1.0.0
	$

Tại thời điểm này, API `v1` của `example.com/hello` đã được cố định. Điều này
thông báo cho mọi người rằng API của chúng tôi ổn định và họ nên cảm thấy thoải
mái khi sử dụng nó.

## Kết luận

Bài đăng này đã hướng dẫn quy trình đánh tag một module với các phiên bản ngữ
nghĩa và khi nào nên phát hành `v1`. Một bài đăng trong tương lai sẽ đề cập
đến cách duy trì và phát hành module ở `v2` và hơn thế nữa.

Để cung cấp phản hồi và giúp định hình tương lai của quản lý dependency trong
Go, hãy gửi cho chúng tôi [báo cáo lỗi](/issue/new) hoặc
[báo cáo trải nghiệm](/wiki/ExperienceReports).

Cảm ơn mọi phản hồi và sự trợ giúp cải thiện Go module của bạn.
