---
title: "//go:fix inline và trình nội tuyến cấp mã nguồn"
date: 2026-03-10
by:
- Alan Donovan
tags:
- go fix
- go vet
- analysis framework
- modernizers
- source-level inliner
- static analysis
summary: "Cách hoạt động của trình nội tuyến cấp mã nguồn trong Go 1.26 và cách nó hỗ trợ bạn tự phục vụ việc di chuyển API."
---

<style>
.beforeafter {
  justify-content: center;
  display: grid;
  gap: 1em;
  margin: 1em;
  grid-template-columns: minmax(min-content, 1fr) auto minmax(min-content, 1fr);
  font-size: 180%;
  @media screen and (max-width: 57.7rem) {
    grid-template-columns: 1fr;
  }
}
#content .beforeafter pre {
  margin: 0em; /* Handled by grid gap */
}
.beforeafter-context {
  grid-column: 1 / -1;
}
#content .beforeafter > pre:nth-of-type(1) { background: var(--color-diff-old); }
#content .beforeafter > pre:nth-of-type(2) { background: var(--color-diff-new); }
.beforeafter-arrow {
  place-self: center;
  /* Undo unnecessary grid gap. */
  margin: -0.5em;
}
.beforeafter-arrow::before {
  content: "⟶";
  @media screen and (max-width: 57.7rem) {
    content: "⇓";
  }
}
</style>

Go 1.26 có một triển khai hoàn toàn mới của lệnh con `go fix`, được thiết kế để giúp bạn giữ mã Go luôn hiện đại và cập nhật. Để tìm hiểu giới thiệu, hãy bắt đầu bằng cách đọc [bài viết gần đây](gofix) của chúng tôi về chủ đề này.
Trong bài viết này, chúng ta sẽ xem xét một tính năng cụ thể: trình nội tuyến cấp mã nguồn.

Trong khi `go fix` có một số công cụ hiện đại hóa chuyên biệt cho các tính năng ngôn ngữ và thư viện mới cụ thể, trình nội tuyến cấp mã nguồn là thành quả đầu tiên trong nỗ lực của chúng tôi nhằm cung cấp các công cụ hiện đại hóa và phân tích "[tự phục vụ](gofix#self-service)".
Nó cho phép bất kỳ tác giả gói nào có thể diễn đạt các di chuyển và cập nhật API đơn giản theo cách trực tiếp và an toàn.
Chúng tôi sẽ trình bày trước về trình nội tuyến cấp mã nguồn là gì và cách bạn có thể sử dụng nó, sau đó đi sâu vào một số khía cạnh của vấn đề và công nghệ đằng sau nó.

## Nội tuyến cấp mã nguồn

Năm 2023, chúng tôi đã xây dựng một [thuật toán](https://pkg.go.dev/golang.org/x/tools/internal/refactor/inline) để nội tuyến cấp mã nguồn các lời gọi hàm trong Go. "Nội tuyến" một lời gọi có nghĩa là thay thế lời gọi đó bằng một bản sao thân hàm được gọi, thế các đối số vào vị trí các tham số. Chúng tôi gọi nó là nội tuyến "cấp mã nguồn" vì nó thay đổi mã nguồn một cách vĩnh viễn. Ngược lại, thuật toán nội tuyến trong một trình biên dịch điển hình, bao gồm cả trình biên dịch của Go, áp dụng phép biến đổi tương tự nhưng lên [biểu diễn trung gian](https://en.wikipedia.org/wiki/Intermediate_representation) tạm thời của trình biên dịch, nhằm tạo ra mã hiệu quả hơn.

Nếu bạn đã từng gọi tính năng tái cấu trúc tương tác "[Inline call](/gopls/features/transformation#refactorinlinecall-inline-call-to-function)" của [gopls](/gopls/), bạn đã sử dụng trình nội tuyến cấp mã nguồn. (Trong VS Code, code action này có thể tìm thấy trong menu "Source Action...") Các ảnh chụp màn hình trước và sau dưới đây cho thấy hiệu ứng của việc nội tuyến lời gọi tới `sum` từ hàm tên `six`.

<center>
<img src="/gopls/assets/inline-before.png"/>

<img src="/gopls/assets/inline-after.png"/>
</center>

Trình nội tuyến là một thành phần xây dựng quan trọng cho nhiều công cụ chuyển đổi mã nguồn. Ví dụ, gopls dùng nó cho các tái cấu trúc "Change signature" và "Remove unused parameter" vì, như chúng ta sẽ thấy bên dưới, nó xử lý nhiều vấn đề đúng đắn tinh tế phát sinh khi tái cấu trúc các lời gọi hàm.

Trình nội tuyến này cũng là một trong các bộ phân tích trong lệnh `go fix` hoàn toàn mới.
Trong `go fix`, nó cho phép di chuyển và nâng cấp API tự phục vụ bằng cách dùng chỉ thị comment mới `//go:fix inline`.
Hãy cùng xem một số ví dụ về cách hoạt động và cách sử dụng tính năng này.

### Ví dụ: đổi tên `ioutil.ReadFile`

Trong Go 1.16, hàm `ioutil.ReadFile`, đọc nội dung của một tệp, đã bị deprecated để nhường chỗ cho hàm mới `os.ReadFile`. Về bản chất, hàm đã được đổi tên, mặc dù tất nhiên [cam kết tương thích](/doc/go1compat) của Go ngăn chúng tôi xóa bỏ tên cũ.

```go
package ioutil

import "os"

// ReadFile reads the file named by filename…
// Deprecated: As of Go 1.16, this function simply calls [os.ReadFile].
func ReadFile(filename string) ([]byte, error) {
	return os.ReadFile(filename)
}
```

Lý tưởng là chúng tôi muốn thay đổi mọi chương trình Go trên thế giới để không còn dùng `ioutil.ReadFile` nữa mà gọi `os.ReadFile`. Trình nội tuyến có thể giúp chúng tôi làm điều đó. Trước tiên, chúng tôi chú thích hàm cũ với `//go:fix inline`. Comment này báo cho công cụ biết rằng bất cứ khi nào thấy một lời gọi tới hàm này, nó nên nội tuyến lời gọi đó.

```go
package ioutil

import "os"

// ReadFile reads the file named by filename…
// Deprecated: As of Go 1.16, this function simply calls [os.ReadFile].
//go:fix inline
func ReadFile(filename string) ([]byte, error) {
	return os.ReadFile(filename)
}
```

Khi chúng ta chạy `go fix` trên một tệp chứa lời gọi tới `ioutil.ReadFile`, nó áp dụng phép thay thế:

```
$ go fix -diff ./...
-import "io/ioutil"
+import "os"

-	data, err := ioutil.ReadFile("hello.txt")
+	data, err := os.ReadFile("hello.txt")
```

Lời gọi đã được nội tuyến, thực chất là thay thế lời gọi tới một hàm bằng lời gọi tới hàm khác.

Vì trình nội tuyến thay thế một lời gọi hàm bằng bản sao thân hàm được gọi, chứ không phải bằng một biểu thức tùy ý, về nguyên tắc phép biến đổi không thay đổi hành vi của chương trình (trừ mã kiểm tra call stack, tất nhiên).
Điều này khác với các công cụ cho phép viết lại tùy ý như `gofmt -r`, vốn rất mạnh mẽ nhưng cần được theo dõi chặt chẽ.

Trong nhiều năm qua, các đồng nghiệp Google của chúng tôi trên các nhóm hỗ trợ Java, Kotlin và C++ đã dùng các công cụ nội tuyến cấp mã nguồn như thế này.
Cho đến nay, các công cụ này đã loại bỏ hàng triệu lời gọi tới các hàm deprecated trong codebase của Google.
Người dùng chỉ cần thêm các chỉ thị và chờ đợi.
Qua đêm, các robot lặng lẽ chuẩn bị, kiểm thử và gửi các lô thay đổi mã nguồn trên toàn bộ monorepo hàng tỷ dòng code.
Nếu mọi thứ suôn sẻ, đến sáng hôm sau mã cũ không còn được dùng nữa và có thể xóa an toàn.
Trình nội tuyến của Go còn khá mới mẻ nhưng đã được dùng để chuẩn bị hơn 18.000 changelist cho monorepo của Google.

### Ví dụ: sửa các lỗi thiết kế API

Với một chút sáng tạo, nhiều kiểu di chuyển có thể được diễn đạt dưới dạng nội tuyến.
Hãy xem gói `oldmath` giả định này:

```go
// Package oldmath is the bad old math package.
package oldmath

// Sub returns x - y.
func Sub(y, x int) int

// Inf returns positive infinity.
func Inf() float64

// Neg returns -x.
func Neg(x int) int
```

Nó có một số lỗi thiết kế: hàm `Sub` khai báo các tham số theo thứ tự sai; hàm `Inf` ngầm ưu tiên một trong hai giá trị vô cực; và hàm `Neg` dư thừa so với `Sub`. May mắn thay chúng ta có gói `newmath` tránh được những sai lầm này, và chúng ta muốn người dùng chuyển sang dùng nó. Bước đầu tiên là triển khai API cũ theo gói mới và đánh dấu các hàm cũ là deprecated. Sau đó chúng ta thêm các chỉ thị nội tuyến:

```
// Package oldmath is the bad old math package.
package oldmath

import "newmath"

// Sub returns x - y.
// Deprecated: the parameter order is confusing.
//go:fix inline
func Sub(y, x int) int {
	return newmath.Sub(x, y)
}

// Inf returns positive infinity.
// Deprecated: there are two infinite values; be explicit.
//go:fix inline
func Inf() float64 {
	return newmath.Inf(+1)
}

// Neg returns -x.
// Deprecated: this function is unnecessary.
//go:fix inline
func Neg(x int) int {
	return newmath.Sub(0, x)
}
```

Bây giờ, khi người dùng `oldmath` chạy lệnh `go fix` trên mã của họ, tất cả các lời gọi tới hàm cũ sẽ được thay thế bằng các hàm tương ứng mới. Nhân tiện, gopls đã bao gồm bộ phân tích `inline` trong bộ công cụ của mình một thời gian rồi, vì vậy nếu trình soạn thảo của bạn dùng gopls, ngay sau khi bạn thêm các chỉ thị `//go:fix inline`, bạn sẽ thấy một cảnh báo tại mỗi điểm gọi, chẳng hạn như "call of `oldmath.Sub` should be inlined", cùng với một gợi ý sửa lỗi để nội tuyến lời gọi cụ thể đó.

Ví dụ, đoạn mã cũ này:
```
import "oldmath"

var nine = oldmath.Sub(1, 10) // diagnostic: "call to oldmath.Sub should be inlined"
```
sẽ được chuyển đổi thành:
```
import "newmath"

var nine = newmath.Sub(10, 1)
```
Hãy lưu ý rằng sau khi sửa, các đối số của `Sub` theo đúng thứ tự logic. Đây là sự tiến bộ! Nếu may mắn, trình nội tuyến sẽ loại bỏ thành công mọi lời gọi tới các hàm trong `oldmath`, có thể cho phép bạn xóa nó như một dependency.

Bộ phân tích `inline` cũng hoạt động với các kiểu và hằng số. Nếu gói `oldmath` của chúng ta ban đầu đã khai báo một kiểu dữ liệu cho số hữu tỷ và một hằng số cho π, chúng ta có thể dùng các khai báo chuyển tiếp sau để di chuyển chúng sang gói `newmath` trong khi vẫn bảo toàn hành vi của mã hiện có:
```
package oldmath

//go:fix inline
type Rational = newmath.Rational

//go:fix inline
const Pi = newmath.Pi
```

Mỗi khi bộ phân tích `inline` gặp một tham chiếu đến `oldmath.Rational` hoặc `oldmath.Pi`, nó sẽ cập nhật chúng để tham chiếu đến `newmath`.

## Bên trong trình nội tuyến

Nhìn qua, nội tuyến mã nguồn có vẻ đơn giản: chỉ cần thay thế lời gọi bằng thân của hàm được gọi, giới thiệu các biến cho các tham số hàm, và gắn các đối số gọi vào các biến đó.
Nhưng việc xử lý đúng đắn tất cả các trường hợp phức tạp và ngoại lệ trong khi vẫn tạo ra kết quả chấp nhận được là một thách thức kỹ thuật không nhỏ: trình nội tuyến có khoảng 7.000 dòng logic dày đặc giống như trình biên dịch.
Hãy xem sáu khía cạnh của vấn đề khiến nó trở nên phức tạp như vậy.

### 1. Loại bỏ tham số

Một trong những nhiệm vụ quan trọng nhất của trình nội tuyến là cố gắng thay thế từng lần xuất hiện của một tham số trong hàm được gọi bằng đối số tương ứng từ lời gọi. Trong trường hợp đơn giản nhất, đối số là một literal tầm thường như `0` hoặc `""`, vì vậy việc thay thế rất đơn giản và tham số có thể được loại bỏ.

<div class="beforeafter">
<div class="beforeafter-context"><pre>
//go:fix inline
func show(prefix, item string) {
	fmt.Println(prefix, item)
}
</pre></div>
<pre>
show("", "hello")
</pre>
<div class="beforeafter-arrow"></div>
<pre>
fmt.Println("", "hello")
</pre>
</div>

Với các literal ít tầm thường hơn như `404` hoặc `"go.dev"`, việc thay thế cũng đơn giản, miễn là tham số xuất hiện trong hàm được gọi nhiều nhất một lần. Nhưng nếu nó xuất hiện nhiều lần, sẽ không đẹp về mặt văn phong nếu rải các bản sao của các magic value này trong mã vì điều đó làm mờ mối liên hệ giữa chúng; một thay đổi sau này chỉ vào một trong số chúng có thể tạo ra sự không nhất quán.

Trong những trường hợp như vậy, trình nội tuyến phải thận trọng và tạo ra kết quả thận trọng hơn. Khi một hoặc nhiều tham số không thể hoàn toàn thay thế vì bất kỳ lý do gì, trình nội tuyến chèn một khai báo "ràng buộc tham số" rõ ràng:

<div class="beforeafter">
<div class="beforeafter-context"><pre>
//go:fix inline
func printPair(before, x, y, after string) {
	fmt.Println(before, x, after)
	fmt.Println(before, y, after)
}
</pre></div>
<pre>
printPair("[", "one", "two", "]")
</pre>
<div class="beforeafter-arrow"></div>
<pre>
// khai báo "ràng buộc tham số"
var before, after = "[", "]"
fmt.Println(before, "one", after)
fmt.Println(before, "two", after)
</pre>
</div>

### 2. Hiệu ứng phụ

Trong Go, cũng như trong tất cả các ngôn ngữ lập trình mệnh lệnh, việc gọi một hàm có thể có hiệu ứng phụ là cập nhật các biến, điều này lại có thể ảnh hưởng đến hành vi của các hàm khác. Hãy xét lời gọi tới `add` sau đây:

```go
func add(x, y int) int { return y + x }

z = add(f(), g())
```

Một phép nội tuyến đơn giản của lời gọi sẽ thay `x` bằng `f()` và `y` bằng `g()`, với kết quả này:

```
z = g() + f()
```

Nhưng kết quả này không đúng vì việc đánh giá `g()` bây giờ xảy ra trước `f()`; nếu hai hàm có hiệu ứng phụ, các hiệu ứng đó sẽ được quan sát theo thứ tự khác và có thể ảnh hưởng đến kết quả của biểu thức. Tất nhiên, việc viết mã phụ thuộc vào thứ tự hiệu ứng giữa các đối số gọi là không tốt, nhưng điều đó không có nghĩa là người ta không làm vậy, và các công cụ của chúng tôi phải xử lý đúng.

Vì vậy, trình nội tuyến phải cố gắng chứng minh rằng `f()` và `g()` không có hiệu ứng phụ lên nhau. Nếu thành công, nó có thể tiến hành an toàn với kết quả trên. Nếu không, nó phải dùng ràng buộc tham số rõ ràng:

```
var x = f()
z = g() + x
```

Khi xét đến hiệu ứng phụ, không chỉ các biểu thức đối số mới quan trọng. Thứ tự mà các tham số được đánh giá so với mã khác trong hàm được gọi cũng quan trọng. Hãy xét lời gọi này tới `add2`:

```go
//go:fix inline
func add2(x, y int) int {
	return x + other() + y
}

add2(f(), g())
```

Lần này, tham số `x` và `y` được dùng theo cùng thứ tự chúng được khai báo, vì vậy phép thay thế `f() + other() + g()` sẽ không thay đổi thứ tự hiệu ứng của `f()` và `g()`, nhưng nó sẽ thay đổi thứ tự của các hiệu ứng của `other()` và `g()`. Hơn nữa, nếu thân hàm dùng một tham số trong vòng lặp, phép thay thế có thể thay đổi số lần hiệu ứng xảy ra.

Trình nội tuyến dùng một [phân tích nguy cơ](https://cs.opensource.google/go/x/tools/+/refs/tags/v0.42.0:internal/refactor/inline/inline.go;l=1978;drc=e3a69ffcdbb984f50100e76ebca6ff53cf88de9c) mới để mô hình hóa thứ tự hiệu ứng trong mỗi hàm được gọi. Tuy nhiên, khả năng của nó trong việc xây dựng các bằng chứng an toàn cần thiết còn khá hạn chế. Ví dụ, nếu các lời gọi `f()` và `g()` là các accessor đơn giản, sẽ hoàn toàn an toàn khi gọi chúng theo bất kỳ thứ tự nào. Thật vậy, một trình biên dịch tối ưu có thể dùng kiến thức về nội bộ của `f` và `g` để sắp xếp lại an toàn hai lời gọi. Nhưng không giống như trình biên dịch tạo ra mã đối tượng phản ánh mã nguồn tại một thời điểm cụ thể, mục đích của trình nội tuyến là thực hiện các thay đổi vĩnh viễn cho mã nguồn, vì vậy nó không thể tận dụng các chi tiết tạm thời. Như một ví dụ cực đoan, hãy xét hàm `start` này:

```
func start() { /* TODO: implement */ }
```

Một trình biên dịch tối ưu được tự do xóa mỗi lời gọi tới `start()` vì nó không có hiệu ứng nào hôm nay, nhưng trình nội tuyến thì không, vì nó có thể trở nên quan trọng vào ngày mai.

<!-- There's a bit of a contradiction here since the hazard analysis uses implementation details du jour. -->

Tóm lại, trình nội tuyến có thể tạo ra các kết quả mà, theo con mắt của một người bảo trì dự án có kinh nghiệm, rõ ràng là quá thận trọng. Trong những trường hợp như vậy, mã đã sửa sẽ được hưởng lợi về mặt văn phong từ một chút dọn dẹp thủ công.

### 3. Biểu thức hằng số "có thể thất bại"

Bạn có thể tưởng tượng (như tôi đã từng nghĩ) rằng sẽ luôn an toàn khi thay thế một biến tham số bằng một đối số hằng số cùng kiểu. Đáng ngạc nhiên, điều này không phải lúc nào cũng đúng, bởi vì một số kiểm tra trước đây được thực hiện tại thời điểm chạy bây giờ sẽ xảy ra và thất bại tại thời điểm biên dịch. Hãy xét lời gọi tới hàm `index` này:

```
//go:fix inline
func index(s string, i int) byte {
	return s[i]
}

index("", 0)
```

Một trình nội tuyến ngây thơ có thể thay `s` bằng `""` và `i` bằng `0`, kết quả là `""[0]`, nhưng đây thực ra không phải là biểu thức Go hợp lệ vì chỉ số cụ thể này vượt ngoài giới hạn của chuỗi cụ thể này. Vì biểu thức `""[0]` được tạo từ các hằng số, nó được đánh giá tại thời điểm biên dịch, và một chương trình chứa nó sẽ không biên dịch được. Ngược lại, chương trình gốc chỉ thất bại nếu thực thi đến lời gọi `index` này, điều mà trong một chương trình hoạt động bình thường có lẽ không xảy ra.

Do đó, trình nội tuyến phải theo dõi tất cả các biểu thức và toán hạng của chúng có thể trở thành hằng số trong quá trình thay thế tham số, kích hoạt các kiểm tra bổ sung tại thời điểm biên dịch. Nó xây dựng một [hệ ràng buộc](https://cs.opensource.google/go/x/tools/+/master:internal/refactor/inline/falcon.go;l=43;drc=1aca71e85510ecc45dddbc335b30b64298c2a31e) và cố gắng giải quyết nó. Mỗi ràng buộc không thỏa mãn được giải quyết bằng cách thêm một ràng buộc rõ ràng cho các tham số bị ràng buộc.

<!--
  The fundamental reason for falcon is that we can't type-check the result
  since in a "separate analysis" system we don't have type information
  for all dependencies. See hidden comment within section
  [gofix#synergistic-fixes](gofix#synergistic-fixes).
-->

### 4. Che khuất

Các biểu thức đối số thông thường chứa một hoặc nhiều định danh tham chiếu đến các ký hiệu (biến, hàm, v.v.) trong tệp của caller. Trình nội tuyến phải đảm bảo rằng mỗi tên trong biểu thức đối số vẫn tham chiếu đến cùng ký hiệu sau khi thay thế tham số; nói cách khác, không có tên nào của caller bị *che khuất* trong hàm được gọi. Nếu điều này thất bại, trình nội tuyến phải chèn thêm ràng buộc tham số, như trong ví dụ này:

<div class="beforeafter">
<div class="beforeafter-context"><pre>
//go:fix inline
func f(val string) {
	x := 123
	fmt.Println(val, x)
}
</pre></div>
<pre>
x := "hello"
f(x)
</pre>
<div class="beforeafter-arrow"></div>
<pre>
x := "hello"
{
	// thêm khai báo "ràng buộc tham số"
	// để đọc x của caller trước khi nó bị che khuất
	var val string = x
	x := 123
	fmt.Println(val, x)
}
</pre>
</div>

Ngược lại, trình nội tuyến cũng phải kiểm tra rằng mỗi tên trong thân hàm *được gọi* vẫn tham chiếu đến cùng thứ khi nó được ghép vào điểm gọi. Nói cách khác, không có tên nào của hàm được gọi bị che khuất hoặc thiếu trong caller. Đối với các tên thiếu, trình nội tuyến có thể cần chèn thêm các import.

### 5. Biến không dùng đến

Khi một biểu thức đối số không có hiệu ứng và tham số tương ứng của nó không bao giờ được dùng, biểu thức có thể bị loại bỏ. Tuy nhiên, nếu biểu thức chứa tham chiếu cuối cùng tới một biến cục bộ ở caller, điều này có thể gây ra lỗi biên dịch vì biến đó giờ không được dùng đến.

<div class="beforeafter">
<div class="beforeafter-context"><pre>
//go:fix inline
func f(_ int) { print("hello") }
</pre></div>
<pre>
x := 42
f(x)
</pre>
<div class="beforeafter-arrow"></div>
<pre>
x := 42 // lỗi: biến không được dùng: x
print("hello")
</pre>
</div>

Vì vậy, trình nội tuyến phải tính đến các tham chiếu đến biến cục bộ và tránh xóa tham chiếu cuối cùng. (Tất nhiên vẫn có thể xảy ra trường hợp hai lần sửa nội tuyến khác nhau mỗi lần xóa tham chiếu *áp chót* đến một biến, vì vậy hai lần sửa đó có giá trị khi xét riêng lẻ nhưng không khi kết hợp; xem thảo luận về [xung đột ngữ nghĩa](gofix#merging-fixes-and-conflicts) trong bài viết trước. Tiếc thay, trong trường hợp này không thể tránh khỏi việc dọn dẹp thủ công.)

### 6. Defer

Trong một số trường hợp, đơn giản là không thể nội tuyến lời gọi.
Hãy xét một lời gọi tới hàm dùng câu lệnh `defer`:
nếu chúng ta loại bỏ lời gọi, hàm bị defer sẽ thực thi khi hàm *caller* trả về, điều này là quá muộn.
Tất cả những gì chúng ta có thể làm an toàn khi hàm được gọi dùng `defer` là đặt thân hàm được gọi trong một hàm literal và gọi ngay lập tức.
Hàm literal `func() { ... }()` này phân định vòng đời của câu lệnh `defer`, như trong ví dụ này:

<div class="beforeafter">
<div class="beforeafter-context"><pre>
//go:fix inline
func callee() {
	defer f()
	…
}
</pre></div>
<pre>
callee()
</pre>
<div class="beforeafter-arrow"></div>
<pre>
func() {
	defer f()
	…
}()
</pre>
</div>

Nếu bạn gọi trình nội tuyến trong gopls, bạn sẽ thấy nó thực hiện thay đổi như trên và giới thiệu hàm literal. Kết quả này có thể phù hợp trong môi trường tương tác, vì bạn có thể ngay lập tức chỉnh sửa mã (hoặc hoàn tác sửa đổi) tùy ý, nhưng hiếm khi được mong muốn trong một công cụ batch, vì vậy theo chính sách, bộ phân tích trong `go fix` từ chối nội tuyến các lời gọi "được thêm literal" như vậy.

### Một trình biên dịch tối ưu hóa cho "sự gọn gàng"

Chúng ta đã thấy nửa chục ví dụ về cách trình nội tuyến xử lý đúng các trường hợp biên ngữ nghĩa phức tạp.
(Xin trân trọng cảm ơn Rob Findley, Jonathan Amsterdam, Olena Synenka và Lasse Folger về những hiểu biết sâu sắc, các cuộc thảo luận, đánh giá, tính năng và sửa lỗi.)
Bằng cách tích hợp tất cả sự thông minh vào trình nội tuyến, người dùng có thể đơn giản áp dụng tái cấu trúc "Inline call" trong IDE hoặc thêm chỉ thị `//go:fix inline` vào các hàm của riêng mình và tự tin rằng các phép biến đổi mã kết quả có thể được áp dụng với chỉ cần xem xét sơ bộ nhất.

Mặc dù chúng tôi đã đạt được tiến bộ tốt hướng tới mục tiêu đó, chúng tôi chưa hoàn toàn đạt được nó, và có lẽ chúng tôi sẽ không bao giờ hoàn toàn. Hãy xét một trình biên dịch. Một trình biên dịch đúng tạo ra đầu ra chính xác cho bất kỳ đầu vào nào và không bao giờ biên dịch sai mã của bạn; đây là kỳ vọng cơ bản mà mọi người dùng nên có đối với trình biên dịch của mình. Một trình biên dịch *tối ưu hóa* tạo ra mã được lựa chọn cẩn thận cho tốc độ mà không hy sinh sự an toàn. Tương tự, một trình nội tuyến cũng giống như một trình biên dịch tối ưu hóa có mục tiêu không phải là tốc độ mà là *sự gọn gàng*: nội tuyến một lời gọi không bao giờ được thay đổi hành vi của chương trình, và lý tưởng là nó tạo ra mã gọn gàng tối đa. Thật không may, một trình biên dịch tối ưu hóa [được chứng minh](https://en.wikipedia.org/wiki/Rice%27s_theorem) là không bao giờ hoàn tất: chứng minh hai chương trình khác nhau là tương đương là một bài toán không thể giải quyết, và sẽ luôn có những cải tiến mà một chuyên gia biết là an toàn nhưng trình biên dịch không thể chứng minh. Cũng vậy với trình nội tuyến: sẽ luôn có những trường hợp đầu ra của trình nội tuyến quá phức tạp hoặc kém về mặt văn phong so với của một chuyên gia con người, và sẽ luôn có thêm "tối ưu hóa gọn gàng" để thêm vào.

## Hãy thử xem!

Chúng tôi hy vọng chuyến tham quan trình nội tuyến này giúp bạn hiểu một số thách thức liên quan, cũng như các ưu tiên và định hướng của chúng tôi trong việc cung cấp các công cụ biến đổi mã nguồn đúng đắn, tự phục vụ. Hãy thử trình nội tuyến, bằng cách tương tác trong IDE, hoặc thông qua các chỉ thị `//go:fix inline` và lệnh `go fix`, và chia sẻ với chúng tôi trải nghiệm của bạn và bất kỳ ý tưởng nào bạn có về các cải tiến thêm hoặc công cụ mới.
