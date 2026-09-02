---
title: Phương thức generic
date: 2026-08-26
by:
- Mark Freeman
summary: Go 1.27 bổ sung phương thức generic—một tính năng ngôn ngữ được mong đợi nhiều.
template: true
---

Generics là một thay đổi sâu sắc đối với Go—nó về cơ bản đã mở rộng các loại chương trình mà các gopher có thể viết bằng cách thêm tham số kiểu vào ngôn ngữ.

Giờ đây, người dùng có thể biểu diễn các kiểu và hàm generics trong Go. Những cấu trúc này làm giảm nhu cầu về các kiểu dữ liệu và hàm chuyên biệt, giúp chương trình bớt dài dòng hơn và cải thiện tính tiện dụng của ngôn ngữ trong những trường hợp đó.

Để minh họa, các loại danh sách liên kết khác nhau giờ đây có thể được rút gọn thành một định nghĩa kiểu duy nhất:

```go
// trước Go 1.18
type ListOfInts struct {
	elem int
	next *ListOfInts
}
type ListOfStrings struct {
	elem string
	next *ListOfStrings
}
// ...

// sau Go 1.18
type List[E any] struct {
	elem E
	next *List[E]
}
```

Tương tự, việc sắp xếp các loại dữ liệu có thứ tự khác nhau có thể được biểu diễn bằng một hàm duy nhất:

```go
// trước Go 1.18
func SortInts(s []int) { /* ... */ }
func SortStrings(s []string) { /* ... */ }
// ...

// sau Go 1.18
func Sort[E cmp.Ordered](s []E) { /* ... */ }
```

Tuy nhiên, khả năng tương tự đã không được đưa vào các method.

Đề xuất [generics](/issue/43651) lập luận rằng vì các method của interface generic khó được triển khai hiệu quả (được thảo luận [sau](#the-trouble-with-generic-interface-methods)), nên việc thêm tham số kiểu vào các method không phải interface (hay method "cụ thể") cũng không hợp lý. Theo quan điểm đó, các method chủ yếu là phương tiện để triển khai một interface.

Go 1.27 có một cách nhìn khác và do đó bổ sung các method generic vào Go. Trong bài viết này, chúng tôi sẽ giải thích sự thay đổi trong cách nhìn này và minh họa một số cách sử dụng tính năng mới này.

# Các method để tổ chức

Các method cho phép tổ chức chức năng xoay quanh các kiểu. Để minh họa, hãy xem xét lại danh sách liên kết ở trên (bổ sung thêm một số tiện ích):

```go
type List[E any] struct {
	elem E
	next *List[E]
}
func NewList[E any](elems ...E) List[E] { /* ... */ }
func (List[E]) String() string { /* ... */ }
```

Giả sử bạn muốn ánh xạ cấu trúc này để chứa một giá trị thuộc một kiểu khác, chẳng hạn như một `string`. Một method như `ToString` sẽ thực hiện việc đó:

```go
func (List[E]) ToString(f func(E) string) List[string] { /* ... */ }
```

Bằng cách cung cấp hàm “biến đổi” `f`, `ToString` có thể được tùy chỉnh bằng các thủ tục “có sẵn”.

Ví dụ, `strconv.Itoa` có thể hoạt động với một `List[int]`:

```go
func main() {
	fmt.Println(NewList(1, 2, 3).ToString(strconv.Itoa)) // [1 2 3]
}
```

Với một `List[[]byte]`, có thêm nhiều lựa chọn khác (tùy thuộc vào trường hợp sử dụng):

```go
func main() {
	l := NewList([]byte("Hallo Welt"), []byte("Helló világ"))
	fmt.Println(l.ToString(hex.EncodeToString))					// [48616c6c6f2057656c74 48656c6cc3b32076696cc3a167]
	fmt.Println(l.ToString(base64.StdEncoding.EncodeToString))	// [SGFsbG8gV2VsdA== SGVsbMOzIHZpbMOhZw==]
}
```

Lưu ý rằng việc tham số hóa `List` cho phép tổng quát hóa kiểu *nguồn*, nhưng không phải kiểu *đích*, vì kiểu đó phụ thuộc vào phép biến đổi được áp dụng. Điều này có thể hợp lý khi chỉ có một kiểu đích duy nhất, nhưng nếu có nhiều kiểu thì sao?

Trong Go 1.18, một hàm generic có thể được sử dụng cho việc này:

```go
// sau Go 1.18
func MapList[E, R any](l List[E], f func(E) R) List[R] { /* ... */ }
```

Nhưng việc sử dụng một hàm có nhược điểm là đưa `MapList` vào phạm vi package—nếu nhiều kiểu dữ liệu hỗ trợ các thao tác ánh xạ, mọi thứ có thể trở nên lộn xộn. Hơn nữa, các lời gọi được xâu chuỗi phải được viết theo kiểu “từ trong ra ngoài”:

```go
func main() {
	fmt.Println(MapList(MapList(NewList(0, 2, 4), add(2)), divideBy(2))) // [1 2 3]
}
```

Vì Go 1.18 không hỗ trợ các method generic, chúng ta cần một hàm generic làm giải pháp thay thế. Điều này giờ đã được khắc phục trong Go 1.27:

```go
// sau Go 1.27
func (List[E]) Map[R any](f func(E) R) List[R] { /* ... */ }
```

Điều này ngắn gọn, biểu đạt tốt, *và* có phạm vi cục bộ. Ngoài ra, các lời gọi được xâu chuỗi dễ đọc hơn, vì chúng có thể được viết tự nhiên hơn từ trái sang phải:

```go
func main() {
	fmt.Println(NewList(0, 2, 4).Map(add(2)).Map(divideBy(2))) // [1 2 3]
}
```

Bạn thích dạng “từ trong ra ngoài” hơn? Một biểu thức method có thể chuyển đổi bất kỳ method nào, bao gồm cả method generic, thành dạng tương đương là hàm của nó. Vì vậy, nếu muốn, có thể khôi phục cấu trúc lời gọi hàm:

```go
func main() {
	f := List[int].Map[int]
	fmt.Println(f(f(NewList(0, 2, 4), add(2)), divideBy(2))) // [1 2 3]
}
```

Các method generic, giống như những generics khác trong Go, phải được khởi tạo (một cách tường minh hoặc ngầm định) trước khi được sử dụng (được gọi hoặc chuyển đổi thành một hàm).

# Tách rời các vấn đề

Nếu xem các phương thức *cũng* như một công cụ tổ chức, thì lập luận của Go 1.18 có vẻ quá hạn chế. Mặc dù một phương thức cụ thể generic không thể giúp triển khai một interface (nếu không có các phương thức interface generic), nó vẫn có thể hữu ích cho việc tổ chức mã. Nói cách khác, các mối quan tâm này có thể được tách rời.

Để giải thích rõ hơn, hãy xem xét một interface đơn giản:

```go
type I interface {
	M()
}
type T struct{}                   // một struct, không phải một interface
func (T) M[P any]() { /* ... */ } // một phương thức *cụ thể* generic
```

Ở đây, `T.M` được tham số hóa bởi `P`. Mọi lần khởi tạo của `T.M` sẽ tạo ra một chữ ký phương thức giống với `I.M`:

```go
func main() {
	T{}.M[int]()
}
```

Cả `T.M[int]` và `I.M` đều có chữ ký `func M()`. Tuy nhiên, điều này không có nghĩa là `T` triển khai `I`. Việc triển khai interface là một thuộc tính của một kiểu (có thể đã được khởi tạo), không phải của bất kỳ phương thức cụ thể nào. Quan trọng là `T` không khai báo `T.M[int]`, mà là đối tác generic (và do đó chưa được khởi tạo) của nó là `T.M`.

Để `T.M` tham gia vào việc triển khai interface, cần phải tồn tại một phương thức generic `I.M` phù hợp:

```go
type I interface {
	M[P any]() // một phương thức *interface* generic
}
type T struct{}
func (T) M[P any]() { /* ... */ }
```

Vì cú pháp của Go 1.27 không cho phép các phương thức interface khai báo tham số kiểu, nên không thể viết đoạn mã này.

Nhưng *tại sao* Go không thể có các phương thức interface generic? Để hiểu điều này, cần đi đường vòng một chút.

# Vấn đề với các phương thức interface generic

Một giá trị interface là một hộp có thể chứa một giá trị khác—giá trị trong hộp có thể thuộc bất kỳ kiểu nào, miễn là kiểu đó triển khai interface. Điều này có nghĩa là các phương thức được khai báo trên kiểu là một tập siêu của các phương thức được khai báo trên interface.

Để minh họa, `T` triển khai `I` bên dưới:

```go
type I interface {
	M()
}
type T struct{}
func (T) M() { /* ... */ }
```

Có thể gọi các phương thức trên các giá trị interface:

```go
func main() {
	F(T{})
}

func F(i I) {
	i.M() // "được hỗ trợ" bởi T.M
}
```

Ở trên, dễ dàng quan sát thấy rằng lời gọi tới `i.M` sẽ được chuyển tới `T.M`. Nhưng trong các chương trình phức tạp hơn, mối quan hệ giữa một lời gọi phương thức trên giá trị interface và các đích *có thể có* của nó ít rõ ràng hơn đáng kể. Mối quan hệ này thường trải rộng qua các package, khiến việc suy luận trong trường hợp tổng quát trở nên khó khăn hoặc không thể.

Để minh họa, hãy đưa vào một ranh giới package:

```go
// -- package main --
import "p"

func main() {
	p.F(T{})
}
type T struct{}
func (T) M() { /* ... */ }

// -- package p --
func F(i I) {
	i.M()
}
type I interface {
	M()
}
```

Vì hai package được biên dịch riêng biệt, package `main` sẽ không biết `p.F` sử dụng `T{}` như thế nào. Để đảm bảo tồn tại bất kỳ phương thức nào mà `p.F` *có thể* gọi, trình biên dịch tạo mã cho *tất cả* các phương thức không generic của một kiểu tại thời điểm khai báo (hoặc khởi tạo) kiểu đó. Bằng cách này, bất kể `p.F` làm gì với `T{}`, mã cần thiết sẽ tồn tại trong thời gian chạy.

Bây giờ, giả sử `I.M` là generic:

```go
// -- package main --
import "p"

func main() {
	p.F(T{})
}
type T struct{}
func (T) M[P any]() { /* ... */ }

// -- package p --
func F(i I) {
	i.M[int]()
}
type I interface {
	M[P any]()
}
```

Một lần nữa, package `main` sẽ không biết `p.F` sử dụng `T{}` như thế nào—bao gồm cả cách nó có thể khởi tạo `T.M`. Để xử lý *mọi* cách sử dụng `T{}` bên trong `p.F`, trình biên dịch sẽ cần khởi tạo `T.M` với *mọi tham số kiểu có thể có*.

Điều này không thực tế với cách tiếp cận khởi tạo của Go, trong đó trình biên dịch tạo mã cụ thể cho từng phương thức dựa trên các tham số kiểu. Nếu các tham số phương thức được “đóng hộp” (tức là được truyền dưới dạng các giá trị của interface ràng buộc), mỗi lần khởi tạo sẽ chia sẻ cùng một mã. Điều này sẽ tránh có quá nhiều lần khởi tạo với cái giá là chi phí gọi gián tiếp—ngay cả đối với các lời gọi trực tiếp tới các phương thức đã được khởi tạo.

# Kết luận

Go 1.27 giới thiệu tham số kiểu trên các phương thức cụ thể—tính năng này đã được cộng đồng Go mong muốn từ lâu, vì nó cho phép viết mã dễ sử dụng và dễ đọc hơn. Mặc dù chúng ta không thể hỗ trợ các phương thức interface generic, chúng tôi quyết định rằng việc cho phép tham số kiểu trên các phương thức cụ thể vẫn đáng để thực hiện.

Chúng tôi hy vọng bạn thích sử dụng các phương thức generic của Go và tìm ra những cách hữu ích để sử dụng chúng trong các dự án của mình!
