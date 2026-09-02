---
title: Giữ cho Modules của bạn tương thích
date: 2020-07-07
by:
- Jean Barkhuysen
- Jonathan Amsterdam
tags:
- tools
- versioning
summary: Cách giữ cho các module của bạn tương thích với các phiên bản minor/patch trước đó.
---

## Giới thiệu

Bài viết này là phần 5 trong một loạt bài.

- Phần 1 — [Sử dụng Go Modules](/blog/using-go-modules)
- Phần 2 — [Chuyển đổi sang Go Modules](/blog/migrating-to-go-modules)
- Phần 3 — [Xuất bản Go Modules](/blog/publishing-go-modules)
- Phần 4 — [Go Modules: v2 và xa hơn](/blog/v2-go-modules)
- **Phần 5 — Giữ cho Modules của bạn tương thích** (bài này)

**Lưu ý:** Để xem tài liệu về phát triển modules, hãy xem
[Phát triển và xuất bản modules](/doc/modules/developing).

Các module của bạn sẽ phát triển theo thời gian khi bạn thêm tính năng mới, thay đổi hành vi, và xem xét lại các phần trong bề mặt công khai của module. Như đã thảo luận trong [Go Modules: v2 và xa hơn](/blog/v2-go-modules), các thay đổi phá vỡ tương thích với module v1+ phải xảy ra như một phần của việc tăng phiên bản major (hoặc bằng cách áp dụng đường dẫn module mới).

Tuy nhiên, việc phát hành phiên bản major mới là khó khăn với người dùng. Họ phải tìm phiên bản mới, học API mới, và thay đổi code của mình. Và một số người dùng có thể không bao giờ cập nhật, nghĩa là bạn phải duy trì hai phiên bản cho code mãi mãi. Vì vậy, thường tốt hơn là thay đổi package hiện có của bạn theo cách tương thích.

Trong bài viết này, chúng ta sẽ khám phá một số kỹ thuật để giới thiệu các thay đổi không phá vỡ tương thích. Chủ đề chung là: thêm vào, đừng thay đổi hay xóa. Chúng ta cũng sẽ nói về cách thiết kế API của bạn để tương thích ngay từ đầu.

## Thêm vào một hàm

Thường thì các thay đổi phá vỡ tương thích đến dưới dạng các tham số mới cho hàm. Chúng ta sẽ mô tả một số cách để xử lý loại thay đổi này, nhưng trước tiên hãy xem xét một kỹ thuật không hoạt động.

Khi thêm các tham số mới với giá trị mặc định hợp lý, thường muốn thêm chúng như tham số variadic. Để mở rộng hàm

```
func Run(name string)
```

với tham số `size` bổ sung mặc định là zero, người ta có thể đề xuất

```
func Run(name string, size ...int)
```

với lý do rằng tất cả các lời gọi hiện có sẽ tiếp tục hoạt động. Mặc dù điều đó đúng, các cách dùng khác của `Run` có thể bị phá vỡ, như cái này:

```
package mypkg
var runner func(string) = yourpkg.Run
```

Hàm `Run` ban đầu hoạt động ở đây vì kiểu của nó là `func(string)`, nhưng kiểu của hàm `Run` mới là `func(string, ...int)`, vì vậy phép gán thất bại tại thời điểm biên dịch.

Ví dụ này minh họa rằng tương thích lời gọi không đủ cho tương thích ngược. Thực ra, không có thay đổi tương thích ngược nào bạn có thể thực hiện đối với chữ ký của hàm.

Thay vì thay đổi chữ ký của hàm, hãy thêm hàm mới. Ví dụ, sau khi package `context` được giới thiệu, thông lệ phổ biến là truyền `context.Context` làm tham số đầu tiên cho hàm. Tuy nhiên, các API ổn định không thể thay đổi hàm được xuất để chấp nhận `context.Context` vì nó sẽ phá vỡ tất cả các cách dùng của hàm đó.

Thay vào đó, các hàm mới đã được thêm. Ví dụ, chữ ký của phương thức `Query` của package `database/sql` là (và vẫn là)

```
func (db *DB) Query(query string, args ...interface{}) (*Rows, error)
```

Khi package `context` được tạo, nhóm Go đã thêm một phương thức mới vào `database/sql`:

```
func (db *DB) QueryContext(ctx context.Context, query string, args ...interface{}) (*Rows, error)
```

Để tránh sao chép code, phương thức cũ gọi phương thức mới:

```
func (db *DB) Query(query string, args ...interface{}) (*Rows, error) {
    return db.QueryContext(context.Background(), query, args...)
}
```

Việc thêm phương thức cho phép người dùng di chuyển sang API mới theo tốc độ của riêng họ. Vì các phương thức đọc tương tự và sắp xếp gần nhau, và `Context` có trong tên của phương thức mới, sự mở rộng của API `database/sql` này không làm giảm khả năng đọc hay hiểu package.

Nếu bạn dự đoán rằng một hàm có thể cần thêm tham số trong tương lai, bạn có thể lên kế hoạch trước bằng cách làm cho các tham số tùy chọn là một phần của chữ ký hàm. Cách đơn giản nhất để làm điều đó là thêm một tham số struct duy nhất, như hàm [crypto/tls.Dial](https://pkg.go.dev/crypto/tls?tab=doc#Dial) làm:

```
func Dial(network, addr string, config *Config) (*Conn, error)
```

TLS handshake được thực hiện bởi `Dial` đòi hỏi network và address, nhưng nó có nhiều tham số khác với giá trị mặc định hợp lý. Truyền `nil` cho `config` sử dụng các giá trị mặc định đó; truyền struct `Config` với một số trường được đặt sẽ ghi đè các giá trị mặc định cho các trường đó. Trong tương lai, việc thêm tham số cấu hình TLS mới chỉ cần một trường mới trên struct `Config`, một thay đổi tương thích ngược (hầu như luôn luôn, xem "Duy trì tương thích struct" bên dưới).

Đôi khi các kỹ thuật thêm hàm mới và thêm tùy chọn có thể được kết hợp bằng cách làm cho struct tùy chọn là method receiver. Hãy xem xét sự phát triển của khả năng lắng nghe tại địa chỉ mạng của package `net`. Trước Go 1.11, package `net` chỉ cung cấp hàm `Listen` với chữ ký

```
func Listen(network, address string) (Listener, error)
```

Đối với Go 1.11, hai tính năng đã được thêm vào việc lắng nghe `net`: truyền context, và cho phép người gọi cung cấp "control function" để điều chỉnh kết nối thô sau khi tạo nhưng trước khi bind. Kết quả có thể là một hàm mới nhận context, network, address và control function. Thay vào đó, các tác giả package đã thêm struct [`ListenConfig`](https://pkg.go.dev/net@go1.11?tab=doc#ListenConfig) dự kiến rằng có thể cần thêm tùy chọn sau này. Và thay vì định nghĩa một hàm cấp cao nhất mới với tên cồng kềnh, họ đã thêm phương thức `Listen` vào `ListenConfig`:

```
type ListenConfig struct {
    Control func(network, address string, c syscall.RawConn) error
}

func (*ListenConfig) Listen(ctx context.Context, network, address string) (Listener, error)
```

Một cách khác để cung cấp tùy chọn mới trong tương lai là mẫu "Option types", trong đó các tùy chọn được truyền như tham số variadic, và mỗi tùy chọn là một hàm thay đổi trạng thái của giá trị đang được xây dựng. Chúng được mô tả chi tiết hơn trong bài viết của Rob Pike [Self-referential functions and the design of options](https://commandcenter.blogspot.com/2014/01/self-referential-functions-and-design.html). Một ví dụ được sử dụng rộng rãi là [DialOption](https://pkg.go.dev/google.golang.org/grpc?tab=doc#DialOption) của [google.golang.org/grpc](https://pkg.go.dev/google.golang.org/grpc?tab=doc).

Option types thực hiện cùng vai trò như các struct option trong tham số hàm: chúng là một cách có thể mở rộng để truyền cấu hình điều chỉnh hành vi. Việc quyết định nên dùng cái nào phần lớn là vấn đề phong cách. Hãy xem xét cách dùng đơn giản này của kiểu option `DialOption` của gRPC:

```
grpc.Dial("some-target",
  grpc.WithAuthority("some-authority"),
  grpc.WithMaxDelay(time.Second),
  grpc.WithBlock())
```

Điều này cũng có thể được cài đặt như một struct option:

```
notgrpc.Dial("some-target", &notgrpc.Options{
  Authority: "some-authority",
  MaxDelay:  time.Second,
  Block:     true,
})
```

Functional options có một số nhược điểm: chúng yêu cầu viết tên package trước option cho mỗi lời gọi; chúng làm tăng kích thước namespace của package; và không rõ hành vi nên như thế nào nếu cùng một option được cung cấp hai lần. Mặt khác, các hàm nhận struct option cần một tham số có thể hầu như luôn luôn là `nil`, điều mà một số người thấy không hấp dẫn. Và khi giá trị zero của kiểu có ý nghĩa hợp lệ, thật khó xử để chỉ định rằng option nên có giá trị mặc định của nó, thường đòi hỏi một con trỏ hoặc trường boolean bổ sung.

Cả hai đều là lựa chọn hợp lý để đảm bảo khả năng mở rộng trong tương lai của API công khai module của bạn.

## Làm việc với interface

Đôi khi, các tính năng mới đòi hỏi thay đổi đối với các interface được expose công khai: ví dụ, một interface cần được mở rộng với các phương thức mới. Tuy nhiên, trực tiếp thêm vào interface là một thay đổi phá vỡ tương thích, vậy làm thế nào chúng ta có thể hỗ trợ các phương thức mới trên một interface được expose công khai?

Ý tưởng cơ bản là định nghĩa một interface mới với phương thức mới, rồi ở bất kỳ nơi nào sử dụng interface cũ, kiểm tra động xem kiểu được cung cấp là kiểu cũ hay kiểu mới.

Hãy minh họa điều này với một ví dụ từ package [`archive/tar`](https://pkg.go.dev/archive/tar?tab=doc). [`tar.NewReader`](https://pkg.go.dev/archive/tar?tab=doc#NewReader) chấp nhận `io.Reader`, nhưng theo thời gian, nhóm Go nhận ra rằng sẽ hiệu quả hơn khi bỏ qua từ header tệp này đến tệp khác nếu bạn có thể gọi [`Seek`](https://pkg.go.dev/io?tab=doc#Seeker). Nhưng họ không thể thêm phương thức `Seek` vào `io.Reader`: điều đó sẽ phá vỡ tất cả các người cài đặt `io.Reader`.

Một tùy chọn khác bị loại trừ là thay đổi `tar.NewReader` để chấp nhận [`io.ReadSeeker`](https://pkg.go.dev/io?tab=doc#ReadSeeker) thay vì `io.Reader`, vì nó hỗ trợ cả phương thức `io.Reader` lẫn `Seek` (thông qua `io.Seeker`). Nhưng như chúng ta đã thấy ở trên, việc thay đổi chữ ký hàm cũng là một thay đổi phá vỡ tương thích.

Vì vậy, họ đã quyết định giữ nguyên chữ ký của `tar.NewReader`, nhưng kiểm tra kiểu (và hỗ trợ) `io.Seeker` trong các phương thức `tar.Reader`:

```
package tar

type Reader struct {
  r io.Reader
}

func NewReader(r io.Reader) *Reader {
  return &Reader{r: r}
}

func (r *Reader) Read(b []byte) (int, error) {
  if rs, ok := r.r.(io.Seeker); ok {
    // Use more efficient rs.Seek.
  }
  // Use less efficient r.r.Read.
}
```

(Xem [reader.go](https://github.com/golang/go/blob/60f78765022a59725121d3b800268adffe78bde3/src/archive/tar/reader.go#L837) để xem code thực tế.)

Khi bạn gặp trường hợp muốn thêm phương thức vào interface hiện có, bạn có thể làm theo chiến lược này. Bắt đầu bằng cách tạo interface mới với phương thức mới của bạn, hoặc xác định một interface hiện có với phương thức mới. Tiếp theo, xác định các hàm liên quan cần hỗ trợ nó, kiểm tra kiểu đối với interface thứ hai, và thêm code sử dụng nó.

Chiến lược này chỉ hoạt động khi interface cũ không có phương thức mới vẫn có thể được hỗ trợ, điều này giới hạn khả năng mở rộng trong tương lai của module bạn.

Khi có thể, tốt hơn là tránh hoàn toàn loại vấn đề này. Ví dụ, khi thiết kế constructor, ưu tiên trả về các kiểu cụ thể. Làm việc với các kiểu cụ thể cho phép bạn thêm phương thức trong tương lai mà không ảnh hưởng đến người dùng, không giống như interface. Đặc tính đó cho phép module của bạn được mở rộng dễ dàng hơn trong tương lai.

Mẹo: nếu bạn cần sử dụng interface nhưng không có ý định để người dùng cài đặt nó, bạn có thể thêm một phương thức không được xuất. Điều này ngăn các kiểu được định nghĩa bên ngoài package của bạn thỏa mãn interface của bạn mà không nhúng, cho phép bạn thêm phương thức sau này mà không phá vỡ cài đặt của người dùng. Ví dụ, xem [hàm `private()` của `testing.TB`](https://github.com/golang/go/blob/83b181c68bf332ac7948f145f33d128377a09c42/src/testing/testing.go#L564-L567).

```
// TB is the interface common to T and B.
type TB interface {
	Error(args ...interface{})
	Errorf(format string, args ...interface{})
	// ...

	// A private method to prevent users implementing the
	// interface and so future additions to it will not
	// violate Go 1 compatibility.
	private()
}
```

Chủ đề này cũng được khám phá chi tiết hơn trong bài nói "Detecting Incompatible API Changes" của Jonathan Amsterdam ([video](https://www.youtube.com/watch?v=JhdL5AkH-AQ), [slides](https://github.com/gophercon/2019-talks/blob/master/JonathanAmsterdam-DetectingIncompatibleAPIChanges/slides.pdf)).

## Thêm phương thức cấu hình

Cho đến nay, chúng ta đã nói về các thay đổi phá vỡ tương thích rõ ràng, trong đó việc thay đổi kiểu hoặc hàm sẽ khiến code của người dùng không thể biên dịch. Tuy nhiên, các thay đổi hành vi cũng có thể ảnh hưởng xấu đến người dùng, ngay cả khi code của người dùng tiếp tục biên dịch. Ví dụ, nhiều người dùng mong đợi [`json.Decoder`](https://pkg.go.dev/encoding/json?tab=doc#Decoder) bỏ qua các trường trong JSON mà không có trong struct tham số. Khi nhóm Go muốn trả về lỗi trong trường hợp đó, họ phải cẩn thận. Làm vậy mà không có cơ chế opt-in có nghĩa là nhiều người dùng dựa vào các phương thức đó có thể bắt đầu nhận lỗi mà trước đây không có.

Vì vậy, thay vì thay đổi hành vi cho tất cả người dùng, họ đã thêm một phương thức cấu hình vào struct `Decoder`: [`Decoder.DisallowUnknownFields`](https://pkg.go.dev/encoding/json?tab=doc#Decoder.DisallowUnknownFields). Việc gọi phương thức này cho phép người dùng opt-in vào hành vi mới, nhưng không gọi nó sẽ giữ nguyên hành vi cũ cho người dùng hiện có.

## Duy trì tương thích struct

Chúng ta đã thấy ở trên rằng bất kỳ thay đổi nào đối với chữ ký hàm đều là thay đổi phá vỡ tương thích. Tình huống tốt hơn nhiều với struct. Nếu bạn có kiểu struct được xuất, bạn hầu như luôn có thể thêm một trường hoặc xóa một trường không được xuất mà không phá vỡ tương thích. Khi thêm trường, hãy đảm bảo rằng giá trị zero của nó có ý nghĩa và bảo tồn hành vi cũ, để code hiện có không đặt trường vẫn hoạt động.

Nhớ lại rằng các tác giả package `net` đã thêm `ListenConfig` trong Go 1.11 vì họ nghĩ có thể cần thêm tùy chọn sau này. Hóa ra họ đã đúng. Trong Go 1.13, [trường `KeepAlive`](https://pkg.go.dev/net@go1.13?tab=doc#ListenConfig) đã được thêm vào để cho phép vô hiệu hóa keep-alive hoặc thay đổi kỳ của nó. Giá trị mặc định zero bảo tồn hành vi ban đầu của việc bật keep-alive với kỳ mặc định.

Có một cách tinh tế mà một trường mới có thể phá vỡ code của người dùng một cách bất ngờ. Nếu tất cả các kiểu trường trong một struct đều có thể so sánh, nghĩa là các giá trị của các kiểu đó có thể được so sánh với `==` và `!=` và sử dụng làm key map, thì kiểu struct tổng thể cũng có thể so sánh. Trong trường hợp này, việc thêm trường mới của kiểu không thể so sánh sẽ làm cho kiểu struct tổng thể không thể so sánh, phá vỡ bất kỳ code nào so sánh các giá trị của kiểu struct đó.

Để giữ struct có thể so sánh, đừng thêm các trường không thể so sánh vào nó. Bạn có thể viết test cho điều đó, hoặc dựa vào công cụ [gorelease](https://pkg.go.dev/golang.org/x/exp/cmd/gorelease?tab=doc) sắp tới để phát hiện nó.

Để ngăn việc so sánh ngay từ đầu, hãy đảm bảo struct có trường không thể so sánh. Nó có thể đã có một cái rồi, không có kiểu slice, map hay hàm nào có thể so sánh, nhưng nếu không, có thể thêm như sau:

```
type Point struct {
        _ [0]func()
        X int
        Y int
}
```

Kiểu `func()` không thể so sánh, và mảng có độ dài zero không chiếm không gian. Chúng ta có thể định nghĩa một kiểu để làm rõ ý định:

```
type doNotCompare [0]func()

type Point struct {
        doNotCompare
        X int
        Y int
}
```

Bạn có nên dùng `doNotCompare` trong các struct của mình không? Nếu bạn đã định nghĩa struct để được dùng như con trỏ, nghĩa là nó có các phương thức con trỏ và có thể là hàm constructor `NewXXX` trả về con trỏ, thì việc thêm trường `doNotCompare` có thể là quá mức cần thiết. Người dùng kiểu con trỏ hiểu rằng mỗi giá trị của kiểu là riêng biệt: nếu họ muốn so sánh hai giá trị, họ nên so sánh các con trỏ.

Nếu bạn đang định nghĩa struct được dùng trực tiếp như giá trị, như ví dụ `Point` của chúng ta, thì thường bạn muốn nó có thể so sánh. Trong trường hợp không phổ biến khi bạn có struct giá trị mà bạn không muốn so sánh, thì việc thêm trường `doNotCompare` sẽ cho phép bạn thay đổi struct sau mà không cần lo lắng về việc phá vỡ các phép so sánh. Nhược điểm là kiểu đó sẽ không thể sử dụng làm key map.

## Kết luận

Khi lên kế hoạch API từ đầu, hãy cân nhắc cẩn thận về khả năng mở rộng của API cho các thay đổi mới trong tương lai. Và khi bạn cần thêm tính năng mới, hãy nhớ quy tắc: thêm vào, đừng thay đổi hay xóa, nhớ đến các ngoại lệ, vì interface, tham số hàm và giá trị trả về không thể được thêm theo cách tương thích ngược.

Nếu bạn cần thay đổi API một cách đáng kể, hoặc nếu API bắt đầu mất trọng tâm khi thêm nhiều tính năng hơn, thì có thể đã đến lúc cần một phiên bản major mới. Nhưng hầu hết thời gian, việc thực hiện thay đổi tương thích ngược thì dễ dàng và tránh gây phiền toái cho người dùng của bạn.
