---
title: "//go:fix inline và trình nội tuyến ở cấp mã nguồn"
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
summary: "Cách trình nội tuyến ở cấp mã nguồn của Go 1.26 hoạt động và cách nó có thể giúp bạn thực hiện các quá trình di chuyển API tự phục vụ."
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
  margin: 0em; /* Được xử lý bởi khoảng cách của lưới */
}
.beforeafter-context {
  grid-column: 1 / -1;
}
#content .beforeafter > pre:nth-of-type(1) { background: var(--color-diff-old); }
#content .beforeafter > pre:nth-of-type(2) { background: var(--color-diff-new); }
.beforeafter-arrow {
  place-self: center;
  /* Hoàn tác khoảng cách lưới không cần thiết. */
  margin: -0.5em;
}
.beforeafter-arrow::before {
  content: "⟶";
  @media screen and (max-width: 57.7rem) {
    content: "⇓";
  }
}
</style>

Go 1.26 chứa một triển khai hoàn toàn mới của lệnh con `go fix`,
được thiết kế để giúp bạn giữ mã Go của mình luôn cập nhật và hiện đại. Để
giới thiệu, trước tiên hãy đọc [bài đăng gần đây](gofix) của chúng tôi về chủ đề này.
Trong bài đăng này, chúng ta sẽ xem xét một tính năng cụ thể, trình
inline ở mức mã nguồn.

Mặc dù `go fix` có một số công cụ hiện đại hóa được xây dựng riêng cho các tính
năng mới cụ thể của ngôn ngữ và thư viện,
trình inline ở mức mã nguồn là thành quả đầu tiên trong nỗ lực của chúng tôi nhằm cung cấp
các công cụ hiện đại hóa và phân tích
“[tự phục vụ](gofix#self-service)”.
Nó cho phép mọi tác giả gói thể hiện các quá trình di chuyển và
cập nhật API đơn giản theo cách trực tiếp và an toàn.
Trước tiên, chúng tôi sẽ giải thích trình inline ở mức mã nguồn là gì và cách bạn có thể sử dụng nó,
sau đó chúng ta sẽ đi sâu vào một số khía cạnh của vấn đề và công nghệ đứng sau nó.

## Inline ở mức mã nguồn

Vào năm 2023, chúng tôi đã xây dựng một [thuật toán](https://pkg.go.dev/golang.org/x/tools/internal/refactor/inline) để inline ở mức mã nguồn các lời gọi hàm trong Go. “Inline” một lời gọi có nghĩa là thay thế lời gọi đó bằng một bản sao của thân hàm được gọi, thay thế các đối số cho các tham số. Chúng tôi gọi đây là inline “ở mức mã nguồn” vì nó sửa đổi mã nguồn một cách lâu dài. Ngược lại, thuật toán inline được tìm thấy trong một trình biên dịch điển hình, bao gồm cả trình biên dịch của Go, áp dụng một phép biến đổi tương tự, nhưng trên [biểu diễn trung gian](https://en.wikipedia.org/wiki/Intermediate_representation), tạm thời của trình biên dịch, để tạo ra mã hiệu quả hơn.

Nếu bạn đã từng gọi chức năng tái cấu trúc tương tác [gopls](/gopls/) "[Inline call](/gopls/features/transformation#refactorinlinecall-inline-call-to-function)", bạn đã sử dụng trình inline ở mức mã nguồn. (Trong VS Code, hành động mã này có thể được tìm thấy trong menu “Source Action…”.) Các ảnh chụp màn hình trước và sau bên dưới cho thấy hiệu quả của việc inline lời gọi đến `sum` từ hàm có tên `six`.

<center>
<img src="/gopls/assets/inline-before.png"/>

<img src="/gopls/assets/inline-after.png"/>
</center>

Trình inline là một khối xây dựng quan trọng cho một số công cụ biến đổi mã nguồn. Ví dụ, gopls sử dụng nó cho các chức năng tái cấu trúc “Change signature” và “Remove unused parameter” vì, như chúng ta sẽ thấy bên dưới, nó xử lý nhiều vấn đề tinh tế về tính đúng đắn phát sinh khi tái cấu trúc các lời gọi hàm.

Cùng một trình inline này cũng là một trong các trình phân tích trong lệnh `go fix` hoàn toàn mới.
Trong `go fix`, nó cho phép di chuyển và nâng cấp API theo cơ chế tự phục vụ bằng cách sử dụng một chú thích chỉ thị `//go:fix inline` mới.
Hãy xem một vài ví dụ về cách thức hoạt động của điều này và những trường hợp có thể sử dụng nó.

### Ví dụ: đổi tên `ioutil.ReadFile`

Trong Go 1.16, hàm `ioutil.ReadFile`, dùng để đọc nội dung của một tệp, đã bị loại bỏ để ưu tiên hàm `os.ReadFile` mới. Về bản chất, hàm này đã được đổi tên, mặc dù tất nhiên [cam kết tương thích](/doc/go1compat) của Go ngăn chúng ta xóa tên cũ.

```go
package ioutil

import "os"

// ReadFile đọc tệp được chỉ định bởi filename…
// Deprecated: Kể từ Go 1.16, hàm này chỉ đơn giản gọi [os.ReadFile].
func ReadFile(filename string) ([]byte, error) {
	return os.ReadFile(filename)
}
```

Lý tưởng nhất là chúng ta muốn thay đổi mọi chương trình Go trên thế giới để ngừng sử dụng `ioutil.ReadFile` và gọi `os.ReadFile` thay thế. Inliner có thể giúp chúng ta làm điều đó. Đầu tiên, chúng ta chú thích hàm cũ bằng `//go:fix inline`. Chú thích này cho công cụ biết rằng bất cứ khi nào nó thấy lời gọi đến hàm này, nó nên inline lời gọi đó.

```go
package ioutil

import "os"

// ReadFile đọc tệp được chỉ định bởi filename…
// Deprecated: Kể từ Go 1.16, hàm này chỉ đơn giản gọi [os.ReadFile].
//go:fix inline
func ReadFile(filename string) ([]byte, error) {
	return os.ReadFile(filename)
}
```

Khi chạy `go fix` trên một tệp chứa lời gọi đến `ioutil.ReadFile`, nó áp dụng thay thế:

```
$ go fix -diff ./...
-import "io/ioutil"
+import "os"

-	data, err := ioutil.ReadFile("hello.txt")
+	data, err := os.ReadFile("hello.txt")
```

Lời gọi đã được inline, về bản chất thay thế lời gọi đến một hàm bằng lời gọi đến một hàm khác.

Vì inliner thay thế một lời gọi hàm bằng bản sao của phần thân
của hàm được gọi, chứ không phải bằng một biểu thức tùy ý nào đó,
về nguyên tắc việc biến đổi này không nên thay đổi hành vi của chương trình
(ngoại trừ mã kiểm tra ngăn xếp lời gọi, tất nhiên).
Điều này khác với các công cụ khác cho phép viết lại tùy ý,
chẳng hạn như `gofmt -r`, vốn rất mạnh nhưng cần được theo dõi cẩn thận.

Trong nhiều năm qua, các đồng nghiệp của chúng ta tại Google thuộc các nhóm hỗ trợ
Java, Kotlin và C++ đã sử dụng những công cụ inliner ở cấp mã nguồn như thế này.
Cho đến nay, các công cụ này đã loại bỏ hàng triệu lời gọi đến các hàm
đã bị loại bỏ trong cơ sở mã của Google.
Người dùng chỉ cần thêm các chỉ thị và chờ đợi.
Trong đêm, các robot âm thầm chuẩn bị, kiểm thử và gửi các lô
thay đổi mã trên một monorepo chứa hàng tỷ dòng mã.
Nếu mọi việc diễn ra thuận lợi, đến sáng mã cũ không còn được sử dụng nữa và có thể
được xóa một cách an toàn.
Inliner của Go là một công cụ tương đối mới, nhưng nó đã được sử dụng để
chuẩn bị hơn 18.000 changelist cho monorepo của Google.

### Ví dụ: khắc phục các lỗi thiết kế API

Với một chút sáng tạo, nhiều kiểu di chuyển có thể được biểu diễn dưới dạng các phép inlining. Hãy xem xét gói `oldmath` giả định sau:

```go
// Package oldmath là gói toán học cũ tệ hại.
package oldmath

// Sub trả về x - y.
func Sub(y, x int) int

// Inf trả về vô cực dương.
func Inf() float64

// Neg trả về -x.
func Neg(x int) int
```

Gói này có một số lỗi thiết kế: hàm `Sub` khai báo các tham số theo thứ tự sai; hàm `Inf` ngầm ưu tiên một trong hai giá trị vô cực; và hàm `Neg` bị trùng lặp với `Sub`. May mắn là chúng ta có gói `newmath` tránh được những sai lầm này, và muốn người dùng chuyển sang sử dụng nó. Bước đầu tiên là triển khai API cũ dựa trên gói mới và đánh dấu các hàm cũ là lỗi thời. Sau đó, chúng ta thêm các chỉ thị inliner:

```
// Package oldmath là gói toán học cũ tệ hại.
package oldmath

import "newmath"

// Sub trả về x - y.
// Deprecated: thứ tự tham số gây nhầm lẫn.
//go:fix inline
func Sub(y, x int) int {
	return newmath.Sub(x, y)
}

// Inf trả về vô cực dương.
// Deprecated: có hai giá trị vô cực; hãy chỉ rõ.
//go:fix inline
func Inf() float64 {
	return newmath.Inf(+1)
}

// Neg trả về -x.
// Deprecated: hàm này không cần thiết.
//go:fix inline
func Neg(x int) int {
	return newmath.Sub(0, x)
}
```

Bây giờ, khi người dùng của `oldmath` chạy lệnh `go fix` trên mã của họ, lệnh này sẽ thay thế tất cả các lời gọi đến các hàm cũ bằng các hàm tương ứng mới. Nhân tiện, gopls đã đưa `inline` vào bộ phân tích của nó từ một thời gian, vì vậy nếu trình soạn thảo của bạn sử dụng gopls, ngay khi bạn thêm các chỉ thị `//go:fix inline`, bạn sẽ bắt đầu thấy chẩn đoán tại mỗi vị trí gọi, chẳng hạn như “lời gọi `oldmath.Sub` nên được inlined”, cùng với bản sửa được đề xuất để inlining lời gọi cụ thể đó.

Ví dụ, mã cũ này:
```
import "oldmath"

var nine = oldmath.Sub(1, 10) // diagnostic: "call to oldmath.Sub should be inlined"
```
sẽ được chuyển đổi thành:
```
import "newmath"

var nine = newmath.Sub(10, 1)
```
Lưu ý rằng sau khi sửa, các đối số của `Sub` đã ở đúng thứ tự logic. Đây là một bước tiến! Nếu may mắn, inliner sẽ loại bỏ thành công mọi lời gọi đến các hàm trong `oldmath`, có thể cho phép bạn xóa nó như một dependency.

Bộ phân tích `inline` cũng hoạt động với các kiểu và hằng số. Nếu gói `oldmath` của chúng ta ban đầu đã khai báo một kiểu dữ liệu cho các số hữu tỉ và một hằng số cho π, chúng ta có thể sử dụng các khai báo chuyển tiếp sau để di chuyển chúng sang gói `newmath` trong khi vẫn giữ nguyên hành vi của mã hiện có:
```
package oldmath

//go:fix inline
type Rational = newmath.Rational

//go:fix inline
const Pi = newmath.Pi
```

Mỗi khi bộ phân tích `inline` gặp một tham chiếu đến `oldmath.Rational` hoặc `oldmath.Pi`, nó sẽ cập nhật chúng để thay vào đó tham chiếu đến `newmath`.

## Bên trong trình inline

Thoạt nhìn, inline mã nguồn có vẻ đơn giản: chỉ cần thay thế
lời gọi bằng thân của hàm được gọi, tạo biến cho các
tham số của hàm, và gán các đối số của lời gọi cho những biến đó.
Nhưng việc xử lý đúng tất cả sự phức tạp và các trường hợp góc
trong khi vẫn tạo ra kết quả chấp nhận được không phải là một thách thức kỹ thuật nhỏ:
trình inline chứa khoảng 7.000 dòng logic dày đặc, giống như logic của trình biên dịch.
Hãy cùng xem xét sáu khía cạnh của vấn đề khiến việc này trở nên khó khăn đến vậy.

### 1. Loại bỏ tham số

Một trong những nhiệm vụ quan trọng nhất của trình inline là cố gắng thay thế mỗi lần xuất hiện của một tham số trong hàm được gọi bằng đối số tương ứng của nó từ lời gọi. Trong trường hợp đơn giản nhất, đối số là một literal đơn giản như `0` hoặc `""`, vì vậy việc thay thế rất trực tiếp và tham số có thể được loại bỏ.

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

Đối với các literal ít đơn giản hơn như `404` hoặc `"go.dev"`, việc thay thế cũng tương tự, miễn là tham số chỉ xuất hiện trong hàm được gọi nhiều nhất một lần. Nhưng nếu nó xuất hiện nhiều lần, việc rải các bản sao của những giá trị đặc biệt này khắp mã sẽ là một phong cách không tốt vì nó làm mờ mối quan hệ giữa chúng; một thay đổi sau này chỉ đối với một trong số chúng có thể tạo ra sự không nhất quán.

Trong những trường hợp như vậy, trình inline phải xử lý cẩn thận và tạo ra kết quả thận trọng hơn. Bất cứ khi nào một hoặc nhiều tham số không thể được thay thế hoàn toàn vì bất kỳ lý do nào, trình inline sẽ chèn một khai báo “gán tham số” rõ ràng:

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
// a “parameter binding” declaration
var before, after = "[", "]"
fmt.Println(before, "one", after)
fmt.Println(before, "two", after)
</pre>
</div>

### 2. Tác dụng phụ

Trong Go, cũng như trong mọi ngôn ngữ lập trình mệnh lệnh, việc gọi một hàm có thể có tác dụng phụ là cập nhật các biến, từ đó có thể ảnh hưởng đến hành vi của các hàm khác. Hãy xem lời gọi đến `add` bên dưới:

```go
func add(x, y int) int { return y + x }

z = add(f(), g())
```

Một phép nội tuyến hóa đơn giản của lời gọi này sẽ thay thế `x` bằng `f()` và `y` bằng `g()`, cho kết quả sau:

```
z = g() + f()
```

Nhưng kết quả này không đúng vì việc đánh giá `g()` giờ đây xảy ra trước `f()`; nếu hai hàm này có tác dụng phụ, các tác dụng đó sẽ được quan sát theo một thứ tự khác và có thể ảnh hưởng đến kết quả của biểu thức. Dĩ nhiên, viết mã phụ thuộc vào thứ tự tác dụng giữa các đối số của lời gọi hàm là cách làm không tốt, nhưng điều đó không có nghĩa là mọi người không làm vậy, và các công cụ của chúng ta phải xử lý đúng trường hợp này.

Vì vậy, trình nội tuyến hóa phải cố gắng chứng minh rằng `f()` và `g()` không có tác dụng phụ lẫn nhau. Nếu thành công, nó có thể tiếp tục một cách an toàn với kết quả ở trên. Nếu không, nó phải quay lại cách liên kết tham số tường minh:

```
var x = f()
z = g() + x
```

Khi xem xét tác dụng phụ, không chỉ các biểu thức đối số là quan trọng. Thứ tự mà các tham số được đánh giá so với mã khác trong hàm được gọi cũng có ý nghĩa. Hãy xem lời gọi đến `add2` sau:

```go
//go:fix inline
func add2(x, y int) int {
	return x + other() + y
}

add2(f(), g())
```

Lần này, các tham số `x` và `y` được sử dụng theo cùng thứ tự mà chúng được khai báo, vì vậy phép thay thế `f() + other() + g()` sẽ không thay đổi thứ tự tác dụng của `f()` và `g()`—nhưng nó sẽ thay đổi thứ tự của bất kỳ tác dụng nào của `other()` và `g()`. Hơn nữa, nếu thân hàm sử dụng một tham số bên trong một vòng lặp, phép thay thế có thể thay đổi số lần xuất hiện của các tác dụng.

Trình nội tuyến hóa sử dụng một [phân tích rủi ro](https://cs.opensource.google/go/x/tools/+/refs/tags/v0.42.0:internal/refactor/inline/inline.go;l=1978;drc=e3a69ffcdbb984f50100e76ebca6ff53cf88de9c) mới để mô hình hóa thứ tự tác dụng trong mỗi hàm được gọi. Tuy nhiên, khả năng xây dựng các chứng minh an toàn cần thiết của nó khá hạn chế. Ví dụ, nếu các lời gọi `f()` và `g()` là các hàm truy cập đơn giản, việc gọi chúng theo thứ tự nào cũng hoàn toàn an toàn. Thực tế, một trình biên dịch tối ưu hóa có thể sử dụng kiến thức về phần triển khai bên trong của `f` và `g` để sắp xếp lại hai lời gọi này một cách an toàn. Nhưng khác với trình biên dịch, vốn tạo ra mã đối tượng phản ánh mã nguồn tại một thời điểm cụ thể, mục đích của trình nội tuyến hóa là thực hiện các thay đổi vĩnh viễn lên mã nguồn, vì vậy nó không thể tận dụng các chi tiết tạm thời. Là một ví dụ cực đoan, hãy xem xét hàm `start` này:

```
func start() { /* TODO: implement */ }
```

Một trình biên dịch tối ưu hóa được phép xóa mỗi lời gọi đến `start()` vì hiện tại nó không có tác dụng gì, nhưng trình nội tuyến hóa thì không, vì nó có thể trở nên quan trọng vào ngày mai.

<!-- Có một chút mâu thuẫn ở đây vì phân tích rủi ro sử dụng các chi tiết triển khai du jour. -->

Tóm lại, trình nội tuyến hóa có thể tạo ra các kết quả mà—dưới con mắt am hiểu của người duy trì dự án—rõ ràng là quá thận trọng. Trong những trường hợp như vậy, mã đã được sửa sẽ được cải thiện về mặt phong cách nếu thực hiện một chút dọn dẹp thủ công.

### 3. Biểu thức hằng “có thể thất bại”

Bạn có thể tưởng tượng (như tôi từng làm) rằng việc luôn thay thế một biến tham số bằng một đối số hằng có cùng kiểu sẽ an toàn. Thật bất ngờ, điều này hóa ra không đúng, vì một số kiểm tra trước đây được thực hiện tại thời điểm chạy giờ sẽ diễn ra—và thất bại—tại thời điểm biên dịch. Hãy xem lời gọi này tới hàm `index`:

```
//go:fix inline
func index(s string, i int) byte {
	return s[i]
}

index("", 0)
```

Một trình inline ngây thơ có thể thay thế `s` bằng `""` và `i` bằng `0`, tạo ra `""[0]`, nhưng đây thực sự không phải là một biểu thức Go hợp lệ vì chỉ mục cụ thể này nằm ngoài phạm vi của chuỗi cụ thể này. Vì biểu thức `""[0]` được tạo thành từ các hằng, nó được đánh giá tại thời điểm biên dịch, và một chương trình chứa biểu thức này thậm chí sẽ không thể xây dựng. Ngược lại, chương trình ban đầu chỉ thất bại nếu quá trình thực thi đi tới lời gọi `index` này, điều mà trong một chương trình hoạt động bình thường có lẽ không xảy ra.

Do đó, trình inline phải theo dõi tất cả biểu thức và toán hạng của chúng có thể trở thành hằng trong quá trình thay thế tham số, từ đó kích hoạt các kiểm tra bổ sung tại thời điểm biên dịch. Nó xây dựng một [hệ thống ràng buộc](https://cs.opensource.google/go/x/tools/+/master:internal/refactor/inline/falcon.go;l=43;drc=1aca71e85510ecc45dddbc335b30b64298c2a31e) và cố gắng giải quyết nó. Mỗi ràng buộc chưa được thỏa mãn được giải quyết bằng cách thêm một liên kết tường minh cho các tham số bị ràng buộc.

<!--
  Lý do cơ bản của falcon là chúng ta không thể kiểm tra kiểu của kết quả
  vì trong một hệ thống “phân tích riêng biệt”, chúng ta không có thông tin kiểu
  cho tất cả dependency. Xem chú thích ẩn trong phần
  [gofix#synergistic-fixes](gofix#synergistic-fixes).
-->

### 4. Che khuất tên

Các biểu thức đối số thông thường chứa một hoặc nhiều định danh tham chiếu đến các ký hiệu (biến, hàm, v.v.) trong tệp của bên gọi. Trình inline phải đảm bảo rằng mỗi tên trong biểu thức đối số sẽ tham chiếu đến cùng một ký hiệu sau khi thay thế tham số; nói cách khác, không tên nào của bên gọi bị *che khuất* trong bên được gọi. Nếu điều này thất bại, trình inline một lần nữa phải chèn các liên kết tham số, như trong ví dụ này:

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
	// một khai báo “liên kết tham số” khác
	// để đọc x của bên gọi trước khi che khuất nó
	var val string = x
	x := 123
	fmt.Println(val, x)
}
</pre>
</div>

Ngược lại, trình inline cũng phải kiểm tra rằng mỗi tên trong thân hàm *bên được gọi* sẽ tham chiếu đến cùng một thứ khi nó được ghép vào vị trí gọi. Nói cách khác, không tên nào của bên được gọi bị che khuất hoặc thiếu trong bên gọi. Đối với các tên bị thiếu, trình inline có thể cần chèn thêm các import.

### 5. Biến không được sử dụng

Khi một biểu thức đối số không có tác động nào và tham số tương ứng của nó không bao giờ được sử dụng, biểu thức đó có thể được loại bỏ. Tuy nhiên, nếu biểu thức chứa tham chiếu cuối cùng đến một biến cục bộ tại phía gọi, điều này có thể gây ra lỗi biên dịch vì biến đó hiện không còn được sử dụng.

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
x := 42 // error: unused variable: x
print("hello")
</pre>
</div>

Vì vậy, trình inline phải tính đến các tham chiếu đến biến cục bộ và tránh loại bỏ tham chiếu cuối cùng. (Tất nhiên vẫn có khả năng hai bản sửa inline khác nhau mỗi bản đều loại bỏ tham chiếu *thứ hai tính từ cuối* đến một biến, vì vậy hai bản sửa này hợp lệ khi tách riêng nhưng không hợp lệ khi cùng áp dụng; xem phần thảo luận về [xung đột ngữ nghĩa](gofix#merging-fixes-and-conflicts) trong bài đăng trước. Đáng tiếc là trong trường hợp này việc dọn dẹp thủ công là không thể tránh khỏi.)

### 6. Defer

Trong một số trường hợp, đơn giản là không thể inline để loại bỏ lời gọi.
Hãy xem xét một lời gọi đến một hàm sử dụng câu lệnh `defer`:
nếu chúng ta loại bỏ lời gọi, hàm được trì hoãn sẽ thực thi
khi hàm *gọi* trả về, tức là quá muộn.
Điều duy nhất chúng ta có thể làm an toàn khi hàm được gọi sử dụng `defer` là
đặt thân của hàm được gọi vào một function literal và gọi nó ngay lập tức.
Function literal này, `func() { … }()`, giới hạn thời gian tồn tại của
câu lệnh `defer`, như trong ví dụ sau:

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

Nếu bạn gọi trình inline trong gopls, bạn sẽ thấy rằng nó thực hiện thay đổi được hiển thị ở trên và đưa vào function literal. Kết quả này có thể phù hợp trong môi trường tương tác, vì bạn có khả năng sẽ ngay lập tức tinh chỉnh mã (hoặc hoàn tác bản sửa) theo ý muốn, nhưng hiếm khi phù hợp trong một công cụ chạy theo lô, vì vậy theo chính sách, bộ phân tích trong `go fix` từ chối inline các lời gọi “được chuyển thành literal” như vậy.

### Một trình biên dịch tối ưu hóa cho “tính gọn gàng”

Đến đây, chúng ta đã thấy nửa tá ví dụ về cách trình inliner xử lý đúng các trường hợp biên ngữ nghĩa phức tạp.
(Xin cảm ơn rất nhiều Rob Findley, Jonathan Amsterdam, Olena Synenka và Lasse Folger vì những hiểu biết, thảo luận, đánh giá, tính năng và bản sửa lỗi.)
Bằng cách đưa toàn bộ phần thông minh vào trình inliner, người dùng có thể chỉ cần áp dụng thao tác tái cấu trúc “Inline call” trong IDE của họ hoặc thêm chỉ thị `//go:fix inline` vào các hàm của chính mình, đồng thời tin tưởng rằng các phép biến đổi mã thu được có thể được áp dụng chỉ với việc xem xét sơ bộ nhất.

Mặc dù chúng ta đã đạt được nhiều tiến bộ hướng tới mục tiêu đó, chúng ta vẫn chưa hoàn toàn đạt được nó, và có khả năng chúng ta sẽ không bao giờ đạt được đầy đủ. Hãy xem xét một trình biên dịch. Một trình biên dịch đúng đắn tạo ra đầu ra chính xác với mọi đầu vào và không bao giờ biên dịch sai mã của bạn; đây là kỳ vọng cơ bản mà mọi người dùng nên có đối với trình biên dịch của mình. Một trình biên dịch *tối ưu hóa* tạo ra mã được lựa chọn cẩn thận để có tốc độ cao mà không ảnh hưởng đến tính an toàn. Tương tự, trình inliner giống như một trình biên dịch tối ưu hóa có mục tiêu không phải là tốc độ mà là *tính gọn gàng*: việc inline một lời gọi không bao giờ được thay đổi hành vi của chương trình, và lý tưởng nhất là nó tạo ra mã gọn gàng, ngăn nắp nhất có thể. Đáng tiếc, một trình biên dịch tối ưu hóa được [chứng minh](https://en.wikipedia.org/wiki/Rice%27s_theorem) là không bao giờ hoàn thiện: việc chứng minh hai chương trình khác nhau là tương đương là một bài toán không quyết định được, và sẽ luôn có những cải tiến mà chuyên gia biết là an toàn nhưng trình biên dịch không thể chứng minh. Điều tương tự cũng đúng với trình inliner: sẽ luôn có những trường hợp đầu ra của trình inliner quá cầu kỳ hoặc kém hơn về mặt phong cách so với đầu ra của một chuyên gia con người, và sẽ luôn có thêm các “tối ưu hóa tính gọn gàng” cần được bổ sung.

## Hãy thử!

Chúng tôi hy vọng chuyến tham quan này về trình inliner giúp bạn cảm nhận được một số thách thức liên quan, cũng như các ưu tiên và hướng đi của chúng tôi trong việc cung cấp các công cụ biến đổi mã tự phục vụ, đúng đắn. Vui lòng thử trình inliner, dù là tương tác trực tiếp trong IDE của bạn, hoặc thông qua các chỉ thị `//go:fix inline` và lệnh `go fix`, rồi chia sẻ với chúng tôi trải nghiệm của bạn cùng bất kỳ ý tưởng nào bạn có về các cải tiến tiếp theo hoặc công cụ mới.
