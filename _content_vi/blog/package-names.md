---
title: Tên gói
date: 2015-02-04
by:
- Sameer Ajmani
tags:
- package
- names
- style
summary: Cách đặt tên cho các gói của bạn.
---

## Giới thiệu

Code Go được tổ chức thành các gói.
Trong một gói, code có thể tham chiếu bất kỳ định danh (tên) nào được định nghĩa bên trong, trong khi
các client của gói chỉ có thể tham chiếu các kiểu, hàm, hằng số và biến được xuất của gói.
Các tham chiếu như vậy luôn bao gồm tên gói làm tiền tố: `foo.Bar` tham chiếu đến
tên được xuất `Bar` trong gói được import có tên `foo`.

Tên gói tốt làm cho code tốt hơn.
Tên gói cung cấp ngữ cảnh cho nội dung của nó, giúp client dễ hiểu hơn
gói dùng để làm gì và cách sử dụng nó.
Tên cũng giúp người bảo trì gói xác định cái gì thuộc và không thuộc
về gói khi nó phát triển.
Các gói được đặt tên tốt giúp dễ tìm thấy code bạn cần hơn.

Effective Go cung cấp
[hướng dẫn](/doc/effective_go.html#names) để đặt tên
gói, kiểu, hàm và biến.
Bài viết này mở rộng cuộc thảo luận đó và khảo sát các tên được tìm thấy trong thư viện chuẩn.
Nó cũng thảo luận về tên gói xấu và cách sửa chúng.

## Tên gói

Tên gói tốt ngắn và rõ ràng.
Chúng là chữ thường, không có `under_scores` hoặc `mixedCaps`.
Chúng thường là danh từ đơn giản, chẳng hạn như:

  - `time` (cung cấp chức năng để đo và hiển thị thời gian)
  - `list` (triển khai danh sách liên kết đôi)
  - `http` (cung cấp triển khai HTTP client và server)

Phong cách tên điển hình của ngôn ngữ khác có thể không thành ngữ trong một
chương trình Go.
Đây là hai ví dụ về tên có thể là phong cách tốt trong các ngôn ngữ khác nhưng
không phù hợp trong Go:

  - `computeServiceClient`
  - `priority_queue`

Một gói Go có thể xuất nhiều kiểu và hàm.
Ví dụ, một gói `compute` có thể xuất kiểu `Client` với các phương thức để
sử dụng dịch vụ cũng như các hàm để phân chia một tác vụ tính toán qua
nhiều client.

**Viết tắt một cách khôn ngoan.**
Tên gói có thể được viết tắt khi viết tắt quen thuộc với
lập trình viên.
Các gói được sử dụng rộng rãi thường có tên nén:

  - `strconv` (string conversion)
  - `syscall` (system call)
  - `fmt` (formatted I/O)

Mặt khác, nếu viết tắt tên gói làm nó mơ hồ hoặc không rõ ràng,
đừng làm vậy.

**Đừng lấy tên tốt từ người dùng.**
Tránh đặt cho gói tên thường được sử dụng trong code client.
Ví dụ, gói buffered I/O được gọi là `bufio`, không phải `buf`, vì `buf`
là tên biến tốt cho một bộ đệm.

## Đặt tên nội dung gói

Tên gói và tên nội dung của nó được kết hợp, vì code client sử dụng chúng
cùng nhau.
Khi thiết kế một gói, hãy xem xét quan điểm của client.

**Tránh lặp lại.**
Vì code client sử dụng tên gói làm tiền tố khi tham chiếu đến
nội dung gói, tên cho các nội dung đó không cần lặp lại tên gói.
HTTP server được cung cấp bởi gói `http` được gọi là `Server`, không phải
`HTTPServer`.
Code client tham chiếu kiểu này là `http.Server`, vì vậy không có sự mơ hồ.

**Đơn giản hóa tên hàm.**
Khi một hàm trong gói pkg trả về một giá trị kiểu `pkg.Pkg` (hoặc
`*pkg.Pkg`), tên hàm thường có thể bỏ qua tên kiểu mà không gây nhầm lẫn:

	start := time.Now()                                  // start là time.Time
	t, err := time.Parse(time.Kitchen, "6:06PM")         // t là time.Time
	ctx = context.WithTimeout(ctx, 10*time.Millisecond)  // ctx là context.Context
	ip, ok := userip.FromContext(ctx)                    // ip là net.IP

Một hàm có tên `New` trong gói `pkg` trả về một giá trị kiểu `pkg.Pkg`.
Đây là điểm vào tiêu chuẩn cho code client sử dụng kiểu đó:

	 q := list.New()  // q là *list.List

Khi một hàm trả về một giá trị kiểu `pkg.T`, nơi `T` không phải là `Pkg`, tên
hàm có thể bao gồm `T` để làm cho code client dễ hiểu hơn.
Một tình huống phổ biến là một gói với nhiều hàm giống New:

	d, err := time.ParseDuration("10s")  // d là time.Duration
	elapsed := time.Since(start)         // elapsed là time.Duration
	ticker := time.NewTicker(d)          // ticker là *time.Ticker
	timer := time.NewTimer(d)            // timer là *time.Timer

Các kiểu trong các gói khác nhau có thể có cùng tên, vì từ quan điểm
của client, các tên đó được phân biệt bởi tên gói.
Ví dụ, thư viện chuẩn bao gồm nhiều kiểu có tên `Reader`,
bao gồm `jpeg.Reader`, `bufio.Reader` và `csv.Reader`.
Mỗi tên gói phù hợp với `Reader` để tạo ra tên kiểu tốt.

Nếu bạn không thể đưa ra tên gói là tiền tố có ý nghĩa cho
nội dung của gói, ranh giới trừu tượng của gói có thể không đúng.
Viết code sử dụng gói của bạn như client sẽ làm, và tái cấu trúc các gói của bạn nếu kết quả có vẻ kém.
Cách tiếp cận này sẽ tạo ra các gói dễ hiểu hơn cho client và
dễ bảo trì hơn cho nhà phát triển gói.

## Đường dẫn gói

Một gói Go có cả tên và đường dẫn.
Tên gói được chỉ định trong câu lệnh package của các tệp nguồn của nó;
code client sử dụng nó làm tiền tố cho các tên được xuất của gói.
Code client sử dụng đường dẫn gói khi import gói.
Theo quy ước, phần tử cuối cùng của đường dẫn gói là tên gói:

	import (
		"context"                // gói context
		"fmt"                    // gói fmt
		"golang.org/x/time/rate" // gói rate
		"os/exec"                // gói exec
	)

Các công cụ build ánh xạ các đường dẫn gói vào các thư mục.
Lệnh go sử dụng biến môi trường [GOPATH](/doc/code.html#GOPATH)
để tìm các tệp nguồn cho đường dẫn `"github.com/user/hello"`
trong thư mục `$GOPATH/src/github.com/user/hello`.
(Tình huống này nên quen thuộc, tất nhiên, nhưng quan trọng là phải rõ ràng
về thuật ngữ và cấu trúc của các gói.)

**Thư mục.**
Thư viện chuẩn sử dụng các thư mục như `crypto`, `container`, `encoding`,
và `image` để nhóm các gói cho các giao thức và thuật toán liên quan.
Không có mối quan hệ thực sự nào giữa các gói trong một trong các thư mục này;
một thư mục chỉ cung cấp một cách để sắp xếp các tệp.
Bất kỳ gói nào cũng có thể import bất kỳ gói nào khác miễn là việc import không tạo ra
một chu kỳ.

Cũng như các kiểu trong các gói khác nhau có thể có cùng tên mà không mơ hồ,
các gói trong các thư mục khác nhau có thể có cùng tên.
Ví dụ,
[runtime/pprof](/pkg/runtime/pprof) cung cấp dữ liệu profiling
ở định dạng được mong đợi bởi công cụ profiling [pprof](https://github.com/google/pprof),
trong khi [net/http/pprof](/pkg/net/http/pprof)
cung cấp các endpoint HTTP để trình bày dữ liệu profiling ở định dạng này.
Code client sử dụng đường dẫn gói để import gói, vì vậy không có
nhầm lẫn.
Nếu một tệp nguồn cần import cả hai gói `pprof`, nó có thể
[đổi tên](/ref/spec#Import_declarations) một hoặc cả hai cục bộ.
Khi đổi tên một gói được import, tên cục bộ nên tuân theo cùng
hướng dẫn như tên gói (chữ thường, không có `under_scores` hoặc `mixedCaps`).

## Tên gói xấu

Tên gói xấu làm cho code khó điều hướng và bảo trì hơn.
Đây là một số hướng dẫn để nhận ra và sửa tên xấu.

**Tránh tên gói không có ý nghĩa.**
Các gói có tên `util`, `common`, hoặc `misc` không cung cấp cho client bất kỳ cảm giác nào về
gói chứa gì.
Điều này làm cho client khó sử dụng gói hơn và làm cho người bảo trì khó giữ gói tập trung hơn.
Theo thời gian, chúng tích lũy các dependency có thể làm cho việc biên dịch chậm hơn đáng kể
và không cần thiết, đặc biệt trong các chương trình lớn.
Và vì tên gói như vậy là chung chung, chúng có nhiều khả năng xung đột với
các gói khác được import bởi code client, buộc client phải đặt tên để
phân biệt chúng.

**Chia nhỏ các gói chung.**
Để sửa các gói như vậy, hãy tìm kiếm các kiểu và hàm có các phần tử tên chung và
kéo chúng vào gói riêng của chúng.
Ví dụ, nếu bạn có

	package util
	func NewStringSet(...string) map[string]bool {...}
	func SortStringSet(map[string]bool) []string {...}

thì code client trông như thế này

	set := util.NewStringSet("c", "a", "b")
	fmt.Println(util.SortStringSet(set))

Kéo các hàm này ra khỏi `util` vào một gói mới, chọn tên phù hợp với
nội dung:

	package stringset
	func New(...string) map[string]bool {...}
	func Sort(map[string]bool) []string {...}

thì code client trở thành

	set := stringset.New("c", "a", "b")
	fmt.Println(stringset.Sort(set))

Khi bạn đã thực hiện thay đổi này, dễ hơn để thấy cách cải thiện gói mới:

	package stringset
	type Set map[string]bool
	func New(...string) Set {...}
	func (s Set) Sort() []string {...}

tạo ra code client thậm chí đơn giản hơn:

	set := stringset.New("c", "a", "b")
	fmt.Println(set.Sort())

Tên của gói là một phần quan trọng trong thiết kế của nó.
Hãy làm việc để loại bỏ các tên gói không có ý nghĩa khỏi các dự án của bạn.

**Đừng sử dụng một gói duy nhất cho tất cả API của bạn.**
Nhiều lập trình viên có thiện ý đặt tất cả các interface được hiển thị bởi
chương trình của họ vào một gói duy nhất có tên `api`, `types`, hoặc `interfaces`, nghĩ rằng
điều này giúp tìm các điểm vào cho codebase của họ dễ dàng hơn.
Đây là một sai lầm.
Các gói như vậy gặp phải các vấn đề tương tự như những gói có tên `util` hoặc `common`,
phát triển không có giới hạn, không cung cấp hướng dẫn cho người dùng, tích lũy
các dependency và xung đột với các import khác.
Hãy chia nhỏ chúng, có thể sử dụng các thư mục để phân tách các gói công khai khỏi
triển khai.

**Tránh va chạm tên gói không cần thiết.**
Mặc dù các gói trong các thư mục khác nhau có thể có cùng tên, các gói thường xuyên
được sử dụng cùng nhau nên có tên riêng biệt.
Điều này giảm sự nhầm lẫn và nhu cầu đổi tên cục bộ trong code client.
Vì lý do tương tự, hãy tránh sử dụng cùng tên với các gói chuẩn phổ biến như
`io` hoặc `http`.

## Kết luận

Tên gói là trung tâm của việc đặt tên tốt trong các chương trình Go.
Hãy dành thời gian để chọn tên gói tốt và tổ chức code của bạn tốt.
Điều này giúp client hiểu và sử dụng các gói của bạn và giúp người bảo trì
phát triển chúng một cách duyên dáng.

## Đọc thêm

  - [Effective Go](/doc/effective_go.html)
  - [How to Write Go Code](/doc/code.html)
  - [Organizing Go Code (bài đăng blog 2012)](/blog/organizing-go-code)
  - [Organizing Go Code (bài nói chuyện Google I/O 2014)](/talks/2014/organizeio.slide)
