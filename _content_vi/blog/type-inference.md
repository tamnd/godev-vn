---
title: Tất Cả Những Gì Bạn Muốn Biết Về Suy Luận Kiểu - Và Thêm Một Chút Nữa
date: 2023-10-09
by:
- Robert Griesemer
summary: Mô tả cách hoạt động của suy luận kiểu trong Go. Dựa trên bài nói tại GopherCon 2023 cùng tiêu đề.
template: true
---

Đây là phiên bản blog của bài nói về suy luận kiểu tại GopherCon 2023 ở San Diego,
được mở rộng và chỉnh sửa một chút để rõ ràng hơn.


## Suy luận kiểu là gì?

Wikipedia định nghĩa suy luận kiểu như sau:

> Suy luận kiểu là khả năng tự động suy ra, một phần hoặc toàn bộ,
> kiểu của một biểu thức tại thời gian biên dịch.
> Trình biên dịch thường có thể suy ra kiểu của một biến hoặc chữ ký kiểu của
> một hàm, mà không cần chú thích kiểu tường minh.

Cụm từ quan trọng ở đây là "tự động suy ra ... kiểu của một biểu thức".
Go đã hỗ trợ một dạng suy luận kiểu cơ bản từ đầu:

```Go
const x = expr  // kiểu của x là kiểu của expr
var x = expr
x := expr
```

Không có kiểu tường minh nào được đưa ra trong các khai báo này,
do đó kiểu của hằng số và biến `x` ở bên trái `=` và `:=`
là kiểu của các biểu thức khởi tạo tương ứng ở bên phải.
Chúng ta nói rằng các kiểu được _suy luận_ từ (các kiểu của) các biểu thức khởi tạo của chúng.
Với sự ra đời của generics trong Go 1.18, khả năng suy luận kiểu của Go
đã được mở rộng đáng kể.


### Tại sao cần suy luận kiểu?

Trong mã Go không generic, hiệu quả của việc bỏ qua kiểu rõ ràng nhất trong một khai báo
biến ngắn. Một khai báo như vậy kết hợp suy luận kiểu và một chút đường cú pháp,
tức là khả năng bỏ qua từ khóa `var`, thành một câu lệnh rất nhỏ gọn.
Xem xét khai báo biến map sau:

```Go
var m map[string]int = map[string]int{}
```

so với

```Go
m := map[string]int{}
```

Bỏ qua kiểu ở bên trái `:=` giúp loại bỏ sự lặp lại và đồng thời tăng khả năng đọc.

Mã Go generic có tiềm năng tăng đáng kể số lượng kiểu xuất hiện trong mã:
nếu không có suy luận kiểu, mỗi lần khởi tạo hàm và kiểu generic đều cần đối số kiểu.
Khả năng bỏ qua chúng trở nên quan trọng hơn.
Hãy xem xét việc sử dụng hai hàm sau từ
[package slices](https://pkg.go.dev/slices) mới:

```Go
package slices
func BinarySearch[S ~[]E, E cmp.Ordered](x S, target E) (int, bool)
func Sort[S ~[]E, E cmp.Ordered](x S)
```

Nếu không có suy luận kiểu, việc gọi `BinarySearch` và `Sort` cần đối số kiểu tường minh:

```Go
type List []int
var list List
slices.Sort[List, int](list)
index, found := slices.BinarySearch[List, int](list, 42)
```

Chúng ta không muốn lặp lại `[List, int]` với mỗi lần gọi hàm generic như vậy.
Với suy luận kiểu, mã đơn giản hóa thành:

```Go
type List []int
var list List
slices.Sort(list)
index, found := slices.BinarySearch(list, 42)
```

Cách này vừa gọn gàng hơn vừa nhỏ gọn hơn.
Thực tế trông giống hệt như mã không generic,
và suy luận kiểu làm điều đó trở nên khả thi.

Quan trọng là, suy luận kiểu là một cơ chế tùy chọn:
nếu đối số kiểu giúp mã rõ ràng hơn, hãy cứ viết chúng ra.


## Suy luận kiểu là một dạng khớp mẫu kiểu

Suy luận so sánh các mẫu kiểu,
trong đó mẫu kiểu là một kiểu chứa các tham số kiểu.
Vì những lý do sẽ trở nên rõ ràng trong một lúc, các tham số kiểu
đôi khi còn được gọi là _biến kiểu_.
Khớp mẫu kiểu cho phép chúng ta suy ra các kiểu cần
đưa vào các biến kiểu này.
Hãy xem xét một ví dụ ngắn:

```Go
// Từ package slices
// func Sort[S ~[]E, E cmp.Ordered](x S)

type List []int
var list List
slices.Sort(list)
```

Lời gọi hàm `Sort` truyền biến `list` làm đối số hàm cho tham số `x`
của [`slices.Sort`](https://pkg.go.dev/slices#Sort).
Do đó kiểu của `list`, là `List`, phải khớp với kiểu của `x`, là tham số kiểu `S`.
Nếu `S` có kiểu `List`, phép gán này trở thành hợp lệ.
Trên thực tế, [các quy tắc về tính gán được](/ref/spec#Assignability) rất phức tạp,
nhưng hiện tại đủ để giả định rằng các kiểu phải giống nhau.

Khi chúng ta đã suy luận được kiểu cho `S`, chúng ta có thể xem xét
[ràng buộc kiểu](/ref/spec#Type_constraints) cho `S`.
Nó nói, vì ký hiệu tilde `~`, rằng
[_kiểu cơ sở_](/ref/spec#Underlying_types) của `S`
phải là slice `[]E`.
Kiểu cơ sở của `S` là `[]int`, do đó `[]int` phải khớp với `[]E`,
và từ đó chúng ta có thể kết luận rằng `E` phải là `int`.
Chúng ta đã có thể tìm ra các kiểu cho `S` và `E` sao cho các kiểu tương ứng khớp nhau.
Suy luận đã thành công!

Đây là một kịch bản phức tạp hơn với nhiều tham số kiểu:
`S1`, `S2`, `E1` và `E2` từ `slices.EqualFunc`, và `E1` và `E2` từ hàm generic `equal`.
Hàm cục bộ `foo` gọi `slices.EqualFunc` với hàm `equal` làm đối số:

```Go
// Từ package slices
// func EqualFunc[S1 ~[]E1, S2 ~[]E2, E1, E2 any](s1 S1, s2 S2, eq func(E1, E2) bool) bool

// Mã cục bộ
func equal[E1, E2 comparable](E1, E2) bool { … }

func foo(list1 []int, list2 []float64) {
	…
	if slices.EqualFunc(list1, list2, equal) {
		…
	}
	…
}
```

Đây là ví dụ mà suy luận kiểu thực sự tỏa sáng khi chúng ta có thể bỏ qua tới sáu đối số kiểu,
một cho mỗi tham số kiểu.
Cách tiếp cận khớp mẫu kiểu vẫn hoạt động, nhưng chúng ta thấy nó có thể trở nên phức tạp nhanh chóng
vì số lượng mối quan hệ kiểu đang tăng lên.
Chúng ta cần một cách tiếp cận có hệ thống để xác định các tham số kiểu nào và kiểu nào liên quan đến các mẫu nào.

Sẽ hữu ích khi nhìn suy luận kiểu theo một cách hơi khác.


## Phương trình kiểu

Chúng ta có thể tái cấu trúc suy luận kiểu như một bài toán giải phương trình kiểu.
Giải phương trình là điều mà tất cả chúng ta đều quen thuộc từ đại số trung học.
May mắn thay, giải phương trình kiểu là một bài toán đơn giản hơn như chúng ta sẽ thấy sớm thôi.

Hãy nhìn lại ví dụ trước của chúng ta:

```Go
// Từ package slices
// func Sort[S ~[]E, E cmp.Ordered](x S)

type List []int
var list List
slices.Sort(list)
```

Suy luận thành công nếu các phương trình kiểu bên dưới có thể được giải.
Ở đây `≡` đại diện cho [_giống hệt_](/ref/spec#Type_identity),
và `under(S)` biểu diễn [kiểu cơ sở](/ref/spec#Underlying_types) của `S`:

	S ≡ List        // tìm S sao cho S ≡ List là đúng
	under(S) ≡ []E  // tìm E sao cho under(S) ≡ []E là đúng

Các tham số kiểu là _biến_ trong các phương trình.
Giải các phương trình có nghĩa là tìm giá trị (đối số kiểu) cho các biến này
(tham số kiểu), sao cho các phương trình trở thành đúng.
Cách nhìn này làm bài toán suy luận kiểu dễ xử lý hơn vì
nó cung cấp cho chúng ta một khuôn khổ chính thức cho phép viết ra thông tin
đưa vào suy luận.


### Chính xác hơn với các quan hệ kiểu

Cho đến nay chúng ta chỉ nói về các kiểu phải
[giống hệt nhau](/ref/spec#Type_identity).
Nhưng đối với mã Go thực tế, yêu cầu đó quá chặt chẽ.
Trong ví dụ trước, `S` không cần phải giống hệt `List`,
mà `List` phải [gán được](/ref/spec#Assignability) cho `S`.
Tương tự, `S` phải [thỏa mãn](/ref/spec#Satisfying_a_type_constraint)
ràng buộc kiểu tương ứng của nó.
Chúng ta có thể đặt ra các phương trình kiểu chính xác hơn bằng cách sử dụng các toán tử cụ thể
mà chúng ta viết là `:≡` và `∈`:

	S :≡ List         // List có thể gán cho S
	S ∈ ~[]E          // S thỏa mãn ràng buộc ~[]E
	E ∈ cmp.Ordered   // E thỏa mãn ràng buộc cmp.Ordered

Nói chung, chúng ta có thể nói rằng phương trình kiểu có ba dạng:
hai kiểu phải giống hệt nhau, một kiểu phải gán được cho kiểu kia,
hoặc một kiểu phải thỏa mãn ràng buộc kiểu:

	X ≡ Y             // X và Y phải giống hệt nhau
	X :≡ Y            // Y có thể gán cho X
	X ∈ Y             // X thỏa mãn ràng buộc Y

(Lưu ý: Trong bài nói GopherCon chúng tôi đã sử dụng ký hiệu `≡`<sub>A</sub> cho `:≡` và
`≡`<sub>C</sub> cho `∈`.
Chúng tôi tin rằng `:≡` gợi lên quan hệ gán được rõ ràng hơn;
và `∈` biểu đạt trực tiếp rằng kiểu được biểu diễn bởi tham số kiểu phải
là một phần tử của [tập hợp kiểu](/ref/spec#Interface_types) của ràng buộc.)


### Nguồn gốc của các phương trình kiểu

Trong một lời gọi hàm generic, chúng ta có thể có đối số kiểu tường minh,
mặc dù hầu hết thời gian chúng ta hy vọng chúng có thể được suy luận.
Thông thường chúng ta cũng có đối số hàm thông thường.
Mỗi đối số kiểu tường minh đóng góp một phương trình kiểu (tầm thường):
tham số kiểu phải giống hệt với đối số kiểu vì mã nói vậy.
Mỗi đối số hàm thông thường đóng góp một phương trình kiểu khác:
đối số hàm phải gán được cho tham số hàm tương ứng.
Và cuối cùng, mỗi ràng buộc kiểu cũng cung cấp một phương trình kiểu
bằng cách ràng buộc các kiểu nào thỏa mãn ràng buộc.

Tất cả lại, điều này tạo ra `n` tham số kiểu và `m` phương trình kiểu.
Trái với đại số trung học cơ bản, `n` và `m` không nhất thiết phải bằng nhau để
các phương trình kiểu có thể giải được.
Ví dụ, phương trình đơn dưới đây cho phép chúng ta suy ra đối số kiểu cho
hai tham số kiểu:

```Go
map[K]V ≡ map[int]string  // K ➞ int, V ➞ string (n = 2, m = 1)
```

Hãy xem xét từng nguồn phương trình kiểu này lần lượt:


#### 1. Phương trình kiểu từ đối số kiểu

Đối với mỗi khai báo tham số kiểu

```Go
func f[…, P constraint, …]…
```

và đối số kiểu được cung cấp tường minh

```Go
f[…, A, …]…
```

chúng ta nhận được phương trình kiểu

	P ≡ A

Chúng ta có thể giải tầm thường điều này cho `P`: `P` phải là `A` và chúng ta viết `P ➞ A`.
Nói cách khác, không có gì cần làm ở đây.
Chúng ta vẫn có thể viết ra phương trình kiểu tương ứng để đầy đủ,
nhưng trong trường hợp này, trình biên dịch Go đơn giản thay thế các đối số kiểu
cho các tham số kiểu của chúng khắp nơi và sau đó các tham số kiểu đó
biến mất và chúng ta có thể quên chúng.


#### 2. Phương trình kiểu từ phép gán

Đối với mỗi đối số hàm `x` được truyền vào tham số hàm `p`

```Go
f(…, x, …)
```

trong đó `p` hoặc `x` chứa tham số kiểu,
kiểu của `x` phải gán được cho kiểu của tham số `p`.
Chúng ta có thể biểu đạt điều này bằng phương trình

	𝑻(p) :≡ 𝑻(x)

trong đó `𝑻(x)` có nghĩa là "kiểu của `x`".
Nếu cả `p` lẫn `x` không chứa tham số kiểu, không có biến kiểu nào để giải:
phương trình đúng vì phép gán là mã Go hợp lệ,
hoặc sai nếu mã không hợp lệ.
Vì lý do này, suy luận kiểu chỉ xem xét các kiểu chứa tham số kiểu của
hàm (hoặc các hàm) liên quan.

Bắt đầu từ Go 1.21, một hàm chưa được khởi tạo hoặc được khởi tạo một phần
(nhưng không phải lời gọi hàm) cũng có thể được gán cho một biến kiểu hàm, như trong:

```Go
// Từ package slices
// func Sort[S ~[]E, E cmp.Ordered](x S)

var intSort func([]int) = slices.Sort
```

Tương tự như việc truyền tham số, các phép gán như vậy dẫn đến một
phương trình kiểu tương ứng. Đối với ví dụ này:

	𝑻(intSort) :≡ 𝑻(slices.Sort)

hoặc được đơn giản hóa:

	func([]int) :≡ func(S)

cùng với các phương trình cho các ràng buộc cho `S` và `E` từ `slices.Sort`
(xem bên dưới).

#### 3. Phương trình kiểu từ ràng buộc

Cuối cùng, đối với mỗi tham số kiểu `P` mà chúng ta muốn suy luận một đối số kiểu,
chúng ta có thể trích xuất một phương trình kiểu từ ràng buộc của nó vì tham số kiểu
phải thỏa mãn ràng buộc. Cho khai báo

```Go
func f[…, P constraint, …]…
```

chúng ta có thể viết ra phương trình

	P ∈ constraint

Ở đây, `∈` có nghĩa là "phải thỏa mãn ràng buộc" tương đương (gần như) với
việc là một phần tử của tập hợp kiểu của ràng buộc.
Chúng ta sẽ thấy sau rằng một số ràng buộc (như `any`) không hữu ích hoặc
hiện tại không thể sử dụng do giới hạn của việc triển khai.
Suy luận đơn giản bỏ qua các phương trình tương ứng trong những trường hợp đó.


### Tham số kiểu và phương trình có thể từ nhiều hàm

Trong Go 1.18, các tham số kiểu được suy luận phải đều từ cùng một hàm.
Cụ thể, không thể truyền một hàm generic chưa được khởi tạo hoặc được khởi tạo một phần
làm đối số hàm,
hoặc gán nó cho một biến (có kiểu hàm).

Như đã đề cập trước đó, trong Go 1.21 suy luận kiểu cũng hoạt động trong các trường hợp này.
Ví dụ, hàm generic

```Go
func myEq[P comparable](x, y P) bool { return x == y }
```

có thể được gán cho một biến kiểu hàm

```Go
var strEq func(x, y string) bool = myEq  // tương đương với việc dùng myEq[string]
```

mà không cần `myEq` được khởi tạo đầy đủ,
và suy luận kiểu sẽ suy ra rằng đối số kiểu cho `P` phải là `string`.

Hơn nữa, một hàm generic có thể được dùng chưa được khởi tạo hoặc được khởi tạo một phần như
một đối số cho một hàm khác, có thể là generic:

```Go
// Từ package slices
// func CompactFunc[S ~[]E, E any](s S, eq func(E, E) bool) S

type List []int
var list List
result := slices.CompactFunc(list, myEq)  // tương đương với slices.CompactFunc[List, int](list, myEq[int])
```

Trong ví dụ cuối này, suy luận kiểu xác định các đối số kiểu cho cả `CompactFunc`
và `myEq`.
Nói chung hơn, các tham số kiểu từ nhiều hàm tùy ý có thể cần được suy luận.
Với nhiều hàm liên quan, các phương trình kiểu cũng có thể từ hoặc liên quan đến nhiều hàm.
Trong ví dụ `CompactFunc` chúng ta kết thúc với ba tham số kiểu và năm phương trình kiểu:

	Các tham số kiểu và ràng buộc:
		S ~[]E
		E any
		P comparable

	Đối số kiểu tường minh:
		không có

	Phương trình kiểu:
		S :≡ List
		func(E, E) bool :≡ func(P, P) bool
		S ∈ ~[]E
		E ∈ any
		P ∈ comparable

	Kết quả:
		S ➞ List
		E ➞ int
		P ➞ int


### Tham số kiểu bị ràng buộc và tự do

Tại điểm này chúng ta đã hiểu rõ hơn về các nguồn khác nhau của phương trình kiểu,
nhưng chúng ta chưa chính xác về tham số kiểu nào cần giải phương trình cho.
Hãy xem xét một ví dụ khác.
Trong mã bên dưới, thân hàm của `sortedPrint` gọi `slices.Sort` để sắp xếp.
`sortedPrint` và `slices.Sort` đều là hàm generic vì cả hai đều khai báo tham số kiểu.

```Go
// Từ package slices
// func Sort[S ~[]E, E cmp.Ordered](x S)

// sortedPrint in các phần tử của danh sách đã cung cấp theo thứ tự đã sắp xếp.
func sortedPrint[F any](list []F) {
	slices.Sort(list)  // 𝑻(list) là []F
	…                  // in list
}
```

Chúng ta muốn suy luận đối số kiểu cho lời gọi `slices.Sort`.
Việc truyền `list` vào tham số `x` của `slices.Sort` dẫn đến phương trình

	𝑻(x) :≡ 𝑻(list)

tương đương với

	S :≡ []F

Trong phương trình này chúng ta có hai tham số kiểu, `S` và `F`.
Chúng ta cần giải phương trình kiểu cho cái nào?
Vì hàm được gọi là `Sort`, chúng ta quan tâm đến tham số kiểu `S` của nó,
không phải tham số kiểu `F`.
Chúng ta nói rằng `S` _bị ràng buộc_ với `Sort` vì nó được khai báo bởi `Sort`.
`S` là biến kiểu có liên quan trong phương trình này.
Ngược lại, `F` bị ràng buộc với (được khai báo bởi) `sortedPrint`.
Chúng ta nói rằng `F` _tự do_ đối với `Sort`.
Nó có kiểu đã cho của riêng nó.
Kiểu đó là `F`, bất kể `F` là gì (được xác định tại thời gian khởi tạo).
Trong phương trình này, `F` đã được cho, nó là một _hằng số kiểu_.

Khi giải các phương trình kiểu, chúng ta luôn giải cho các tham số kiểu
bị ràng buộc với hàm chúng ta đang gọi
(hoặc gán trong trường hợp gán hàm generic).


## Giải phương trình kiểu

Phần còn thiếu, bây giờ chúng ta đã thiết lập cách thu thập các
tham số kiểu và phương trình kiểu có liên quan, tất nhiên là thuật toán cho phép
chúng ta giải các phương trình.
Sau các ví dụ khác nhau, có lẽ đã trở nên rõ ràng rằng việc giải
`X ≡ Y` đơn giản có nghĩa là so sánh các kiểu `X` và `Y` đệ quy với
nhau, và trong quá trình đó xác định các đối số kiểu phù hợp cho
các tham số kiểu có thể xuất hiện trong `X` và `Y`.
Mục tiêu là làm cho các kiểu `X` và `Y` _giống hệt nhau_.
Quá trình khớp này được gọi là [_unification_](https://en.wikipedia.org/wiki/Unification_(computer_science)).

Các quy tắc về [đồng nhất kiểu](/ref/spec#Type_identity) cho
chúng ta biết cách so sánh các kiểu.
Vì các tham số kiểu _bị ràng buộc_ đóng vai trò của biến kiểu, chúng ta cần
chỉ định cách chúng được khớp với các kiểu khác.
Các quy tắc như sau:

- Nếu tham số kiểu `P` đã có kiểu được suy luận, `P` đại diện cho kiểu đó.
- Nếu tham số kiểu `P` chưa có kiểu được suy luận và được khớp với kiểu khác
`T`, `P` được đặt thành kiểu đó: `P ➞ T`.
Chúng ta nói rằng kiểu `T` đã được suy luận cho `P`.
- Nếu `P` được khớp với tham số kiểu khác `Q`, và cả `P` lẫn `Q`
chưa có kiểu được suy luận, `P` và `Q` được _unify_.

Unification của hai tham số kiểu có nghĩa là chúng được kết hợp lại sao cho
từ đó cả hai đều biểu thị cùng một giá trị tham số kiểu:
nếu một trong `P` hoặc `Q` được khớp với kiểu `T`, cả `P` và `Q` đều
được đặt thành `T` đồng thời
(nói chung, bất kỳ số lượng tham số kiểu nào cũng có thể được unify theo cách này).

Cuối cùng, nếu hai kiểu `X` và `Y` khác nhau, phương trình không thể được thỏa mãn
và việc giải nó thất bại.


### Unify các kiểu cho đồng nhất kiểu

Một vài ví dụ cụ thể sẽ làm rõ thuật toán này.
Xem xét hai kiểu `X` và `Y` chứa ba tham số kiểu bị ràng buộc `A`, `B` và `C`,
tất cả đều xuất hiện trong phương trình kiểu `X ≡ Y`.
Mục tiêu là giải phương trình này cho các tham số kiểu; tức là tìm
các đối số kiểu phù hợp cho chúng sao cho `X` và `Y` trở thành giống hệt nhau
và phương trình trở thành đúng.

```Go
X: map[A]struct{i int; s []B}
Y: map[string]struct{i C; s []byte}
```

Unification tiến hành bằng cách so sánh cấu trúc của `X` và `Y` đệ quy, bắt đầu từ đỉnh.
Chỉ nhìn vào cấu trúc của hai kiểu chúng ta có

```Go
map[…]… ≡ map[…]…
```

với `…` đại diện cho các kiểu khóa và giá trị của map tương ứng mà chúng ta đang
bỏ qua ở bước này.
Vì chúng ta có một map ở cả hai phía, các kiểu giống hệt nhau cho đến nay.
Unification tiến hành đệ quy, trước tiên với các kiểu khóa là `A` cho map `X`,
và `string` cho map `Y`.
Các kiểu khóa tương ứng phải giống hệt nhau, và từ đó chúng ta có thể ngay lập tức suy ra rằng
đối số kiểu cho `A` phải là `string`:

```Go
A ≡ string => A ➞ string
```

Tiếp tục với các kiểu phần tử của map, chúng ta đến

```Go
struct{i int; s []B} ≡ struct{i C; s []byte}
```

Cả hai phía đều là struct nên unification tiến hành với các trường struct.
Chúng giống hệt nhau nếu chúng theo cùng thứ tự, với cùng tên, và kiểu giống hệt.
Cặp trường đầu tiên là `i int` và `i C`.
Tên khớp và vì `int` phải được unify với `C`:

```Go
int ≡ C => C ➞ int
```

Việc khớp kiểu đệ quy này tiếp tục cho đến khi cấu trúc cây của hai kiểu được duyệt đầy đủ,
hoặc cho đến khi xuất hiện xung đột.
Trong ví dụ này, cuối cùng chúng ta kết thúc với

```Go
[]B ≡ []byte => B ≡ byte => B ➞ byte
```

Mọi thứ đều ổn và unification suy ra các đối số kiểu

	A ➞ string
	B ➞ byte
	C ➞ int


### Unify các kiểu có cấu trúc khác nhau

Bây giờ hãy xem xét một biến thể nhỏ của ví dụ trước:
ở đây `X` và `Y` không có cùng cấu trúc kiểu.
Khi các cây kiểu được so sánh đệ quy, unification vẫn thành công suy ra đối số kiểu cho `A`.
Nhưng các kiểu giá trị của các map khác nhau và unification thất bại.

```Go
X: map[A]struct{i int; s []B}
Y: map[string]bool
```

Cả `X` và `Y` đều là kiểu map, vì vậy unification tiến hành đệ quy như trước, bắt đầu với các kiểu khóa.
Chúng ta đến

```Go
A ≡ string => A ➞ string
```

cũng như trước.
Nhưng khi chúng ta tiến hành với các kiểu giá trị của map chúng ta có

```Go
struct{…} ≡ bool
```

Kiểu `struct` không khớp với `bool`; chúng ta có các kiểu khác nhau và unification (và do đó suy luận kiểu) thất bại.


### Unify các kiểu với đối số kiểu xung đột

Một loại xung đột khác xuất hiện khi các kiểu khác nhau khớp với cùng tham số kiểu.
Ở đây chúng ta lại có một phiên bản của ví dụ ban đầu của mình nhưng bây giờ tham số kiểu `A` xuất hiện hai lần trong `X`,
và `C` xuất hiện hai lần trong `Y`.

```Go
X: map[A]struct{i int; s []A}
Y: map[string]struct{i C; s []C}
```

Việc unify kiểu đệ quy hoạt động tốt lúc đầu và chúng ta có các cặp sau của
tham số kiểu và kiểu:

```Go
A   ≡ string => A ➞ string  // kiểu khóa map
int ≡ C      => C ➞ int     // kiểu trường struct đầu tiên
```

Khi chúng ta đến kiểu trường struct thứ hai chúng ta có

```Go
[]A ≡ []C => A ≡ C
```

Vì cả `A` và `C` đều có đối số kiểu được suy luận cho chúng, chúng đại diện cho các đối số kiểu đó,
là `string` và `int` tương ứng.
Đây là các kiểu khác nhau, vì vậy `A` và `C` không thể khớp được.
Unification và do đó suy luận kiểu thất bại.


### Các quan hệ kiểu khác

Unification giải các phương trình kiểu có dạng `X ≡ Y` trong đó mục tiêu là _đồng nhất kiểu_.
Nhưng còn `X :≡ Y` hoặc `X ∈ Y` thì sao?

Một vài quan sát giúp chúng ta ở đây:
Công việc của suy luận kiểu chỉ là tìm các kiểu của các đối số kiểu bị bỏ qua.
Suy luận kiểu luôn được theo sau bởi kiểu hoặc hàm
[khởi tạo](/ref/spec#Instantiations) kiểm tra rằng mỗi đối số kiểu
thực sự thỏa mãn ràng buộc kiểu tương ứng của nó.
Cuối cùng, trong trường hợp lời gọi hàm generic, trình biên dịch cũng kiểm tra rằng
các đối số hàm có thể gán được cho các tham số hàm tương ứng của chúng.
Tất cả các bước này phải thành công để mã hợp lệ.

Nếu suy luận kiểu không đủ chính xác, nó có thể suy ra một đối số kiểu (không chính xác)
trong khi không có kiểu nào có thể tồn tại.
Nếu đó là trường hợp, việc khởi tạo hoặc truyền đối số sẽ thất bại.
Dù sao đi nữa, trình biên dịch sẽ tạo ra một thông báo lỗi.
Chỉ là thông báo lỗi có thể hơi khác một chút.

Quan sát này cho phép chúng ta linh hoạt hơn một chút với các quan hệ kiểu `:≡` và `∈`.
Cụ thể, nó cho phép chúng ta đơn giản hóa chúng sao cho chúng có thể được xử lý
gần như tương tự như `≡`.
Mục tiêu của các đơn giản hóa là trích xuất nhiều thông tin kiểu nhất có thể
từ một phương trình kiểu, và do đó suy ra các đối số kiểu nơi mà một triển khai
chính xác có thể thất bại, vì chúng ta có thể.


### Đơn giản hóa X :≡ Y

Các quy tắc về tính gán được của Go khá phức tạp, nhưng hầu hết thời gian chúng ta thực sự
có thể làm được với đồng nhất kiểu, hoặc một biến thể nhỏ của nó.
Miễn là chúng ta tìm được đối số kiểu tiềm năng, chúng ta hài lòng, chính xác vì suy luận kiểu
vẫn được theo sau bởi khởi tạo kiểu và gọi hàm.
Nếu suy luận tìm thấy một đối số kiểu khi không nên, nó sẽ bị bắt sau.
Do đó, khi khớp cho tính gán được, chúng ta thực hiện các điều chỉnh sau đây cho
thuật toán unification:

- Khi một kiểu có tên (đã định nghĩa) được khớp với một kiểu literal,
  các kiểu cơ sở của chúng được so sánh thay thế.
- Khi so sánh các kiểu channel, hướng channel bị bỏ qua.

Hơn nữa, hướng gán bị bỏ qua: `X :≡ Y` được xử lý như `Y :≡ X`.

Các điều chỉnh này chỉ áp dụng ở cấp độ trên cùng của cấu trúc kiểu:
ví dụ, theo [các quy tắc về tính gán được](/ref/spec#Assignability) của Go,
một kiểu map có tên có thể được gán cho một kiểu map không có tên, nhưng các kiểu khóa và phần tử
vẫn phải giống hệt nhau.
Với những thay đổi này, unification cho tính gán được trở thành một biến thể (nhỏ) của
unification cho đồng nhất kiểu.
Ví dụ sau minh họa điều này.

Giả sử chúng ta đang truyền một giá trị của kiểu `List` trước đó (được định nghĩa là `type List []int`)
vào một tham số hàm kiểu `[]E` trong đó `E` là tham số kiểu bị ràng buộc (tức là `E` được khai báo
bởi hàm generic đang được gọi).
Điều này dẫn đến phương trình kiểu `[]E :≡ List`.
Cố gắng unify hai kiểu này đòi hỏi so sánh `[]E` với `List`.
Hai kiểu này không giống hệt nhau, và nếu không có bất kỳ thay đổi nào về cách unification hoạt động,
nó sẽ thất bại.
Nhưng vì chúng ta đang unify cho tính gán được, khớp ban đầu này không cần phải chính xác.
Không có hại gì khi tiếp tục với kiểu cơ sở của kiểu có tên `List`:
trong trường hợp xấu nhất chúng ta có thể suy ra một đối số kiểu không chính xác, nhưng điều đó sẽ dẫn đến lỗi
sau, khi các phép gán được kiểm tra.
Trong trường hợp tốt nhất, chúng ta tìm thấy một đối số kiểu hữu ích và chính xác.
Trong ví dụ của chúng ta, unification không chính xác thành công và chúng ta suy ra đúng `int` cho `E`.


### Đơn giản hóa X ∈ Y

Khả năng đơn giản hóa quan hệ thỏa mãn ràng buộc thậm chí còn quan trọng hơn vì
các ràng buộc có thể rất phức tạp.

Một lần nữa, thỏa mãn ràng buộc được kiểm tra tại thời gian khởi tạo, vì vậy mục tiêu ở đây là
giúp suy luận kiểu nơi chúng ta có thể.
Đây thường là các tình huống mà chúng ta biết cấu trúc của một tham số kiểu;
ví dụ chúng ta biết rằng nó phải là một
kiểu slice và chúng ta quan tâm đến kiểu phần tử của slice.
Ví dụ, một danh sách tham số kiểu có dạng `[P ~[]E]` cho chúng ta biết rằng dù `P` là gì,
kiểu cơ sở của nó phải có dạng `[]E`.
Đây chính xác là các tình huống mà ràng buộc có một
[kiểu lõi](/ref/spec#Core_types).

Do đó, nếu chúng ta có một phương trình dạng

	P ∈ constraint               // hoặc
	P ∈ ~constraint

và nếu `core(constraint)` (hoặc `core(~constraint)`, tương ứng) tồn tại, phương trình
có thể được đơn giản hóa thành

	P        ≡ core(constraint)
	under(P) ≡ core(~constraint)  // tương ứng

Trong tất cả các trường hợp khác, các phương trình kiểu liên quan đến ràng buộc bị bỏ qua.


### Mở rộng các kiểu được suy luận

Nếu unification thành công, nó tạo ra một ánh xạ từ các tham số kiểu
đến các đối số kiểu được suy luận.
Nhưng unification một mình không đảm bảo rằng các kiểu được suy luận không chứa
các tham số kiểu bị ràng buộc.
Để xem tại sao lại như vậy, hãy xem xét hàm generic `g` bên dưới
được gọi với một đối số duy nhất `x` kiểu `int`:

```Go
func g[A any, B []C, C *A](x A) { … }

var x int
g(x)
```

Ràng buộc kiểu cho `A` là `any` không có kiểu lõi, vì vậy chúng ta
bỏ qua nó. Các ràng buộc kiểu còn lại có kiểu lõi là `[]C`
và `*A` tương ứng. Cùng với đối số được truyền vào `g`, sau các đơn giản hóa nhỏ,
các phương trình kiểu là:

		A :≡ int
		B ≡ []C
		C ≡ *A

Vì mỗi phương trình đặt một tham số kiểu so với kiểu không phải tham số kiểu,
unification có ít việc phải làm và ngay lập tức suy ra

		A ➞ int
		B ➞ []C
		C ➞ *A

Nhưng điều đó để lại các tham số kiểu `A` và `C` trong các kiểu được suy luận, điều này
không hữu ích.
Như trong đại số trung học, khi một phương trình đã được giải cho biến `x`,
chúng ta cần thay thế `x` bằng giá trị của nó trong các phương trình còn lại.
Trong ví dụ của chúng ta, trong bước đầu tiên, `C` trong `[]C` được thay thế bằng
kiểu được suy luận ("giá trị") cho `C`, là `*A`, và chúng ta đến

		A ➞ int
		B ➞ []*A    // đã thay thế *A cho C
		C ➞ *A

Trong hai bước nữa chúng ta thay thế `A` trong các kiểu được suy luận `[]*A` và `*A`
bằng kiểu được suy luận cho `A`, là `int`:

		A ➞ int
		B ➞ []*int  // đã thay thế int cho A
		C ➞ *int    // đã thay thế int cho A

Chỉ đến bây giờ suy luận mới hoàn thành.
Và như trong đại số trung học, đôi khi điều này không hoạt động.
Có thể đạt đến một tình huống như

		X ➞ Y
		Y ➞ *X

Sau một vòng thay thế chúng ta có

		X ➞ *X

Nếu chúng ta tiếp tục, kiểu được suy luận cho `X` cứ tăng lên:

		X ➞ **X	    // đã thay thế *X cho X
		X ➞ ***X    // đã thay thế *X cho X
		v.v.

Suy luận kiểu phát hiện các chu trình như vậy trong quá trình mở rộng và báo cáo
một lỗi (và do đó thất bại).


## Hằng số không có kiểu

Cho đến bây giờ chúng ta đã thấy cách suy luận kiểu hoạt động bằng cách giải các phương trình kiểu
với unification, theo sau bởi mở rộng kết quả.
Nhưng nếu không có kiểu thì sao?
Nếu các đối số hàm là các hằng số không có kiểu thì sao?

Một ví dụ khác giúp chúng ta làm sáng tỏ tình huống này.
Hãy xem xét một hàm `foo` nhận một số lượng đối số tùy ý,
tất cả phải có cùng kiểu.
`foo` được gọi với nhiều đối số hằng số không có kiểu khác nhau, bao gồm biến
`x` kiểu `int`:

```Go
func foo[P any](...P) {}

var x int
foo(x)         // P ➞ int, tương đương foo[int](x)
foo(x, 2.0)    // P ➞ int, 2.0 chuyển đổi sang int mà không mất độ chính xác
foo(x, 2.1)    // P ➞ int, nhưng việc truyền tham số thất bại: 2.1 không thể gán cho int
```

Đối với suy luận kiểu, các đối số có kiểu được ưu tiên hơn các đối số không có kiểu.
Một hằng số không có kiểu chỉ được xem xét để suy luận nếu tham số kiểu mà nó được gán
chưa có kiểu được suy luận.
Trong ba lời gọi đầu tiên tới `foo`, biến `x` xác định kiểu được suy luận cho `P`:
đó là kiểu của `x`, là `int`.
Các hằng số không có kiểu bị bỏ qua để suy luận kiểu trong trường hợp này và các lời gọi hoạt động chính xác
như thể `foo` được khởi tạo tường minh với `int`.

Thú vị hơn nếu `foo` được gọi chỉ với các đối số hằng số không có kiểu.
Trong trường hợp này, suy luận kiểu xem xét [các kiểu mặc định](/ref/spec#Constants)
của các hằng số không có kiểu.
Để nhắc lại nhanh, đây là các kiểu mặc định có thể có trong Go:

```
Example     Constant kind              Default type    Order

true        boolean constant           bool
42          integer constant           int             earlier in list
'x'         rune constant              rune               |
3.1416      floating-point constant    float64            v
-1i         complex constant           complex128      later in list
"gopher"    string constant            string
```

Với thông tin này, hãy xem xét lời gọi hàm

```Go
foo(1, 2)    // P ➞ int (kiểu mặc định cho 1 và 2)
```

Các đối số hằng số không có kiểu `1` và `2` đều là hằng số nguyên, kiểu mặc định của chúng là
`int` và do đó là `int` được suy luận cho tham số kiểu `P` của `foo`.

Nếu các hằng số khác nhau, giả sử hằng số nguyên và dấu phẩy động không có kiểu, cùng cạnh tranh
cho cùng biến kiểu, chúng ta có các kiểu mặc định khác nhau.
Trước Go 1.21, điều này được coi là xung đột và dẫn đến lỗi:

```Go
foo(1, 2.0)    // Go 1.20: lỗi suy luận: kiểu mặc định int, float64 không khớp
```

Hành vi này không rất tiện dụng và cũng khác với hành vi của các hằng số không có kiểu
trong các biểu thức. Ví dụ, Go cho phép biểu thức hằng `1 + 2.0`;
kết quả là hằng số dấu phẩy động `3.0` với kiểu mặc định `float64`.

Trong Go 1.21 hành vi đã được thay đổi phù hợp.
Bây giờ, nếu nhiều hằng số số học không có kiểu được khớp với cùng tham số kiểu,
kiểu mặc định xuất hiện sau trong danh sách `int`, `rune`, `float64`, `complex` được
chọn, khớp với các quy tắc cho [các biểu thức hằng](/ref/spec#Constant_expressions):

```Go
foo(1, 2.0)    // Go 1.21: P ➞ float64 (kiểu mặc định lớn hơn của 1 và 2.0; hành vi như trong 1 + 2.0)
```


## Các tình huống đặc biệt

Đến đây chúng ta đã có bức tranh tổng quan về suy luận kiểu.
Nhưng có một vài tình huống đặc biệt quan trọng đáng được chú ý.


### Phụ thuộc vào thứ tự tham số

Tình huống đầu tiên liên quan đến phụ thuộc thứ tự tham số.
Một thuộc tính quan trọng chúng ta muốn từ suy luận kiểu là các kiểu giống nhau được suy luận
bất kể thứ tự của các tham số hàm (và thứ tự đối số tương ứng trong mỗi lời gọi của hàm đó).

Hãy xem xét lại hàm variadic `foo` của chúng ta:
kiểu được suy luận cho `P` phải giống nhau bất kể thứ tự chúng ta
truyền các đối số `s` và `t` ([playground](/play/p/sOlWutKnDFc)).

```Go
func foo[P any](...P) (x P) {}

type T struct{}

func main() {
	var s struct{}
	var t T
	fmt.Printf("%T\n", foo(s, t))
	fmt.Printf("%T\n", foo(t, s)) // kỳ vọng cùng kết quả bất kể thứ tự tham số
}
```

Từ các lời gọi tới `foo` chúng ta có thể trích xuất các phương trình kiểu có liên quan:

	𝑻(x) :≡ 𝑻(s) => P :≡ struct{}    // phương trình 1
	𝑻(x) :≡ 𝑻(t) => P :≡ T           // phương trình 2

Thật đáng buồn, việc triển khai đơn giản cho `:≡` tạo ra phụ thuộc thứ tự:

Nếu unification bắt đầu với phương trình 1, nó khớp `P` với `struct`; `P` chưa có kiểu được suy luận cho nó
và do đó unification suy ra `P ➞ struct{}`.
Khi unification thấy kiểu `T` sau đó trong phương trình 2, nó tiến hành với kiểu cơ sở của `T` là `struct{}`,
`P` và `under(T)` được unify, và unification và do đó suy luận thành công.

Ngược lại, nếu unification bắt đầu với phương trình 2, nó khớp `P` với `T`; `P` chưa có kiểu được suy luận cho nó
và do đó unification suy ra `P ➞ T`.
Khi unification thấy `struct{}` sau đó trong phương trình 1, nó tiến hành với kiểu cơ sở của kiểu `T` được suy luận cho `P`.
Kiểu cơ sở đó là `struct{}`, khớp với `struct` trong phương trình 1, và unification và do đó suy luận thành công.

Kết quả là, tùy thuộc vào thứ tự mà unification giải hai phương trình kiểu,
kiểu được suy luận là `struct{}` hoặc `T`.
Điều này tất nhiên không thỏa mãn: một chương trình có thể đột ngột ngừng biên dịch chỉ vì các đối số
có thể đã được hoán đổi trong quá trình tái cấu trúc hoặc làm sạch mã.


### Khôi phục tính độc lập thứ tự

May mắn thay, biện pháp khắc phục khá đơn giản.
Tất cả những gì chúng ta cần là một sửa chữa nhỏ trong một số tình huống.

Cụ thể, nếu unification đang giải `P :≡ T` và

- `P` là một tham số kiểu đã suy luận được kiểu `A`: `P ➞ A`
- `A :≡ T` là đúng
- `T` là một kiểu có tên

thì đặt kiểu được suy luận cho `P` thành `T`: `P ➞ T`

Điều này đảm bảo rằng `P` là kiểu có tên nếu có sự lựa chọn, bất kể ở điểm nào kiểu có tên
xuất hiện trong khớp với `P` (tức là bất kể thứ tự nào các phương trình kiểu được giải).
Lưu ý rằng nếu các kiểu có tên khác nhau khớp với cùng tham số kiểu, chúng ta luôn có
thất bại unification vì các kiểu có tên khác nhau thì không giống hệt nhau theo định nghĩa.

Vì chúng ta đã thực hiện các đơn giản hóa tương tự cho channel và interface, chúng cũng cần xử lý đặc biệt tương tự.
Ví dụ, chúng ta bỏ qua hướng channel khi unify cho tính gán được và do đó
có thể suy ra một channel có hướng hoặc hai chiều tùy thuộc vào thứ tự đối số. Các vấn đề tương tự xảy ra
với interface. Chúng ta sẽ không thảo luận về những điều này ở đây.

Quay lại ví dụ của chúng ta, nếu unification bắt đầu với phương trình 1, nó suy ra `P ➞ struct{}` như trước.
Khi nó tiến hành với phương trình 2, như trước, unification thành công, nhưng bây giờ chúng ta có chính xác
điều kiện đòi hỏi một sửa chữa: `P` là tham số kiểu đã có kiểu (`struct{}`),
`struct{}`, `struct{} :≡ T` là đúng (vì `struct{} ≡ under(T)` là đúng), và `T` là một kiểu có tên.
Do đó, unification thực hiện sửa chữa và đặt `P ➞ T`.
Kết quả là, bất kể thứ tự unification, kết quả là giống nhau (`T`) trong cả hai trường hợp.


### Hàm đệ quy với chính nó

Một kịch bản khác gây ra vấn đề trong một triển khai suy luận ngây thơ là các hàm đệ quy.
Hãy xem xét một hàm factorial generic `fact`, được định nghĩa sao cho nó cũng hoạt động với đối số dấu phẩy động
([playground](/play/p/s3wXpgHX6HQ)).
Lưu ý rằng đây không phải là triển khai đúng về mặt toán học của
[hàm gamma](https://en.wikipedia.org/wiki/Gamma_function),
đây chỉ là một ví dụ tiện lợi.

```Go
func fact[P ~int | ~float64](n P) P {
	if n {{raw "<"}}= 1 {
		return 1
	}
	return fact(n-1) * n
}
```

Điểm ở đây không phải là hàm factorial mà là `fact` tự gọi với
đối số `n-1` cùng kiểu `P` với tham số `n` đến.
Trong lời gọi này, tham số kiểu `P` đồng thời là tham số kiểu bị ràng buộc và tự do:
nó bị ràng buộc vì được khai báo bởi `fact`, hàm mà chúng ta đang gọi đệ quy.
Nhưng nó cũng tự do vì được khai báo bởi hàm bao quanh lời gọi, tình cờ
cũng là `fact`.

Phương trình từ việc truyền đối số `n-1` vào tham số `n` đặt `P` so với chính nó:

	𝑻(n) :≡ 𝑻(n-1) => P :≡ P

Unification thấy cùng `P` ở cả hai phía của phương trình.
Unification thành công vì cả hai kiểu đều giống hệt nhau nhưng không có thông tin thu được và `P`
vẫn không có kiểu được suy luận. Kết quả là, suy luận kiểu thất bại.

May mắn thay, mẹo để giải quyết điều này đơn giản:
Trước khi suy luận kiểu được gọi, và chỉ để suy luận kiểu sử dụng tạm thời,
trình biên dịch đổi tên các tham số kiểu trong chữ ký (nhưng không phải thân)
của tất cả các hàm liên quan đến lời gọi tương ứng.
Điều này không thay đổi ý nghĩa của các chữ ký hàm:
chúng biểu thị cùng các hàm generic bất kể tên của các tham số kiểu là gì.

Để minh họa, hãy giả sử `P` trong chữ ký của `fact` được đổi tên thành `Q`.
Hiệu quả là như thể lời gọi đệ quy được thực hiện gián tiếp qua một hàm `helper`
([playground](/play/p/TLpo-0auWwC)):

```Go
func fact[P ~int | ~float64](n P) P {
	if n {{raw "<"}}= 1 {
		return 1
	}
	return helper(n-1) * n
}

func helper[Q ~int | ~float64](n Q) Q {
	return fact(n)
}
```

Với việc đổi tên, hoặc với hàm `helper`, phương trình từ việc truyền
`n-1` vào lời gọi đệ quy của `fact` (hoặc hàm `helper`, tương ứng) thay đổi thành

	𝑻(n) :≡ 𝑻(n-1) => Q :≡ P

Phương trình này có hai tham số kiểu: tham số kiểu bị ràng buộc `Q`, được khai báo bởi
hàm đang được gọi, và tham số kiểu tự do `P`, được khai báo bởi hàm
bao quanh. Phương trình kiểu này được giải tầm thường cho `Q` và dẫn đến suy luận
`Q ➞ P`
đây tất nhiên là điều chúng ta kỳ vọng, và có thể xác minh bằng cách khởi tạo tường minh
lời gọi đệ quy ([playground](/play/p/zkUFvwJ54lC)):

```Go
func fact[P ~int | ~float64](n P) P {
	if n {{raw "<"}}= 1 {
		return 1
	}
	return fact[P](n-1) * n
}
```

## Còn thiếu gì?

Đáng chú ý là vắng mặt trong mô tả của chúng ta là suy luận kiểu cho các kiểu generic:
hiện tại các kiểu generic phải luôn được khởi tạo tường minh.

Có một vài lý do cho điều này. Trước tiên, đối với khởi tạo kiểu, suy luận kiểu
chỉ có đối số kiểu để làm việc; không có đối số khác như trong trường hợp lời gọi hàm.
Kết quả là, ít nhất một đối số kiểu phải luôn được cung cấp
(ngoại trừ các trường hợp đặc biệt khi ràng buộc kiểu quy định chính xác một đối số kiểu có thể
cho tất cả các tham số kiểu).
Do đó, suy luận kiểu cho các kiểu chỉ hữu ích để hoàn chỉnh một
kiểu được khởi tạo một phần trong đó tất cả các đối số kiểu bị bỏ qua có thể được suy luận từ
các phương trình từ ràng buộc kiểu; tức là nơi có ít nhất hai tham số kiểu.
Chúng tôi tin rằng đây không phải là kịch bản rất phổ biến.

Thứ hai, và quan trọng hơn, các tham số kiểu cho phép một loại kiểu đệ quy hoàn toàn mới.
Hãy xem xét kiểu giả thuyết

```Go
type T[P T[P]] interface{ … }
```

trong đó ràng buộc cho `P` là kiểu đang được khai báo.
Kết hợp với khả năng có nhiều tham số kiểu có thể tham chiếu đến nhau
theo cách đệ quy phức tạp, suy luận kiểu trở nên phức tạp hơn nhiều và chúng tôi không
hiểu đầy đủ tất cả các hệ quả của điều đó vào lúc này.
Nói như vậy, chúng tôi tin rằng không quá khó để phát hiện các chu trình và tiến hành với
suy luận kiểu khi không có chu trình như vậy.

Cuối cùng, có những tình huống mà suy luận kiểu đơn giản không đủ mạnh để
thực hiện một suy luận, thường là vì unification hoạt động với một số giả định đơn giản hóa
như đã mô tả trước đó trong bài viết này.
Ví dụ chính ở đây là các ràng buộc không có kiểu lõi,
nhưng một cách tiếp cận tinh tế hơn có thể có thể suy ra thông tin kiểu dù sao.

Đây là tất cả các lĩnh vực mà chúng ta có thể thấy các cải tiến từng bước trong các bản phát hành Go trong tương lai.
Quan trọng là, chúng tôi tin rằng các trường hợp mà suy luận hiện tại thất bại là hiếm hoặc
không quan trọng trong mã production, và triển khai hiện tại của chúng tôi bao gồm một số lượng lớn
đa số tất cả các kịch bản mã hữu ích.

Nói như vậy, nếu bạn gặp tình huống mà bạn tin rằng suy luận kiểu nên hoạt động hoặc
đã đi sai đường, hãy [gửi báo cáo lỗi](/issue/new)!
Như thường lệ, nhóm Go rất muốn nghe từ bạn, đặc biệt là khi nó giúp chúng tôi làm cho Go
thậm chí tốt hơn.
