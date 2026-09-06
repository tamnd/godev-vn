---
title: "Sử dụng go fix để hiện đại hóa mã Go"
date: 2026-02-17
by:
- Alan Donovan
tags:
- go fix
- go vet
- analysis framework
- modernizers
- static analysis
summary: "Go 1.26 bao gồm một triển khai mới của go fix có thể giúp bạn sử dụng các tính năng hiện đại hơn của Go."
---

<style>
.beforeafter {
  display: grid;
  font-size: 180%;
  grid-template-columns: 1fr 2em 1fr;
  @media screen and (max-width: 57.7rem) {
    grid-template-columns: 1fr;
  }
}
.beforeafter-arrow {
  place-self: center;
}
.beforeafter-arrow::before {
  content: "⟶";
  @media screen and (max-width: 57.7rem) {
    content: "⇓";
  }
}
</style>

Bản phát hành 1.26 của Go trong tháng này bao gồm một tiểu lệnh go fix được viết lại hoàn toàn. Go fix sử dụng một bộ thuật toán để xác định các cơ hội cải thiện mã của bạn, thường bằng cách tận dụng các tính năng hiện đại hơn của ngôn ngữ và thư viện. Trong bài viết này, trước tiên chúng tôi sẽ chỉ cho bạn cách sử dụng `go fix` để hiện đại hóa codebase Go của bạn. Sau đó trong [phần thứ hai](#go/analysis), chúng ta sẽ đi sâu vào cơ sở hạ tầng phía sau nó và cách nó đang phát triển. Cuối cùng, chúng tôi sẽ trình bày chủ đề về các công cụ phân tích [“tự phục vụ”](#self-service) để giúp những người duy trì module và các tổ chức mã hóa các hướng dẫn và thực tiễn tốt nhất của riêng họ.

<!-- xem https://go.dev/blog/survey2025#challenges -->

## Chạy go fix

Lệnh `go fix`, giống như `go build` và `go vet`, chấp nhận một tập hợp mẫu biểu thị các gói. Lệnh này sửa tất cả các gói bên dưới thư mục hiện tại:
```
$ go fix ./...
```
Khi thành công, lệnh sẽ âm thầm cập nhật các tệp nguồn của bạn. Lệnh loại bỏ mọi bản sửa đổi chạm đến [các tệp được tạo](https://pkg.go.dev/cmd/go#hdr-Generate_Go_files_by_processing_source) vì trong trường hợp đó, bản sửa phù hợp là thay đổi logic của chính trình tạo. Chúng tôi khuyên bạn chạy `go fix` trên dự án của mình mỗi khi cập nhật bản build sang một bản phát hành toolchain Go mới hơn. Vì lệnh có thể sửa hàng trăm tệp, hãy bắt đầu từ một trạng thái git sạch để thay đổi chỉ bao gồm các chỉnh sửa từ go fix; những người đánh giá mã của bạn sẽ cảm ơn bạn.

Để xem trước các thay đổi mà lệnh trên sẽ thực hiện, hãy sử dụng cờ `-diff`:
```
$ go fix -diff ./...
--- dir/file.go (old)
+++ dir/file.go (new)
-                       eq := strings.IndexByte(pair, '=')
-                       result[pair[:eq]] = pair[1+eq:]
+                       before, after, _ := strings.Cut(pair, "=")
+                       result[before] = after
…
```

Bạn có thể liệt kê các trình sửa khả dụng bằng cách chạy lệnh này:
```
$ go tool fix help
…
Registered analyzers:
    any          replace interface{} with any
    buildtag     check //go:build and // +build directives
    fmtappendf   replace []byte(fmt.Sprintf) with fmt.Appendf
    forvar       remove redundant re-declaration of loop variables
    hostport     check format of addresses passed to net.Dial
    inline       apply fixes based on 'go:fix inline' comment directives
    mapsloop     replace explicit loops over maps with calls to maps package
    minmax       replace if/else statements with calls to min or max
…
```

Thêm tên của một trình phân tích cụ thể sẽ hiển thị tài liệu đầy đủ của nó:
```
$ go tool fix help forvar

forvar: remove redundant re-declaration of loop variables

The forvar analyzer removes unnecessary shadowing of loop variables.
Before Go 1.22, it was common to write `for _, x := range s { x := x ... }`
to create a fresh variable for each iteration. Go 1.22 changed the semantics
of `for` loops, making this pattern redundant. This analyzer removes the
unnecessary `x := x` statement.

This fix only applies to `range` loops.
```
Theo mặc định, lệnh `go fix` chạy tất cả các trình phân tích. Khi sửa một dự án lớn, bạn có thể giảm gánh nặng cho việc đánh giá mã nếu áp dụng các bản sửa từ những trình phân tích tạo ra nhiều bản sửa nhất như các thay đổi mã riêng biệt. Để chỉ bật các trình phân tích cụ thể, hãy sử dụng các cờ tương ứng với tên của chúng. Ví dụ, để chỉ chạy trình sửa `any`, hãy chỉ định cờ `-any`. Ngược lại, để chạy tất cả trình phân tích *ngoại trừ* những trình được chọn, hãy phủ định các cờ, chẳng hạn như `-any=false`.

Giống như `go build` và `go vet`, mỗi lần chạy lệnh `go fix` chỉ phân tích một cấu hình build cụ thể. Nếu dự án của bạn sử dụng nhiều tệp được gắn thẻ cho các CPU hoặc nền tảng khác nhau, bạn có thể muốn chạy lệnh nhiều lần với các giá trị khác nhau của `GOARCH` và `GOOS` để có độ phủ tốt hơn:
```
$ GOOS=linux   GOARCH=amd64 go fix ./...
$ GOOS=darwin  GOARCH=arm64 go fix ./...
$ GOOS=windows GOARCH=amd64 go fix ./...
```
Chạy lệnh nhiều lần cũng tạo cơ hội cho các bản sửa mang tính hiệp lực, như chúng ta sẽ thấy bên dưới.

### Công cụ hiện đại hóa

Sự ra đời của [generics](intro-generics) trong Go 1.18 đánh dấu sự kết thúc của một kỷ nguyên với rất ít thay đổi đối với đặc tả ngôn ngữ và sự bắt đầu của một giai đoạn thay đổi nhanh hơn—dù vẫn thận trọng—đặc biệt là trong các thư viện. Nhiều vòng lặp đơn giản mà các lập trình viên Go thường xuyên viết, chẳng hạn như thu thập các khóa của một map vào một slice, giờ đây có thể được biểu diễn thuận tiện bằng một lời gọi đến hàm generic như [`maps.Keys`](https://pkg.go.dev/maps#Keys). Do đó, các tính năng mới này tạo ra nhiều cơ hội để đơn giản hóa mã hiện có.

Vào tháng 12 năm 2024, trong thời kỳ áp dụng ồ ạt các trợ lý lập trình LLM, chúng tôi nhận thấy rằng các công cụ như vậy—không có gì đáng ngạc nhiên—có xu hướng tạo ra mã Go theo phong cách tương tự như phần lớn mã Go được sử dụng trong quá trình huấn luyện, ngay cả khi đã có những cách mới hơn và tốt hơn để biểu đạt cùng một ý tưởng. Ít rõ ràng hơn, các công cụ tương tự thường từ chối sử dụng những cách mới hơn ngay cả khi được yêu cầu làm vậy bằng các chỉ dẫn chung như “luôn sử dụng các thành ngữ mới nhất của Go 1.25”. Trong một số trường hợp, ngay cả khi được yêu cầu rõ ràng sử dụng một tính năng, mô hình vẫn phủ nhận rằng tính năng đó tồn tại. (Xem [bài nói chuyện](https://www.youtube.com/watch?v=_VePjjjV9JU&t=3m50s) GopherCon 2025 của tôi để biết thêm các chi tiết gây bực bội.) Để đảm bảo các mô hình trong tương lai được huấn luyện với những thành ngữ mới nhất, chúng ta cần đảm bảo rằng các thành ngữ này được phản ánh trong dữ liệu huấn luyện, tức là toàn bộ kho mã Go mã nguồn mở trên toàn cầu.

Trong năm qua, chúng tôi đã xây dựng [hàng chục bộ phân tích](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/modernize) để xác định các cơ hội hiện đại hóa. Dưới đây là ba ví dụ về các bản sửa đổi mà chúng đề xuất:

**minmax** thay thế một câu lệnh `if` bằng việc sử dụng các hàm `min` hoặc `max` của Go 1.21:

<div class="beforeafter">
<pre>
x := f()
if x &lt; 0 {
	x = 0
}
if x > 100 {
	x = 100
}
</pre>
<div class="beforeafter-arrow"></div>
<pre>
x := min(max(f(), 0), 100)
</pre>
</div>

**rangeint** thay thế một vòng lặp `for` có 3 mệnh đề bằng vòng lặp `range`-trên-int của Go 1.22:

<div class="beforeafter">
<pre>
for i := 0; i &lt; n; i++ {
	f()
}
</pre>
<div class="beforeafter-arrow"></div>
<pre>
for range n {
	f()
}
</pre>
</div>

**stringscut** (có đầu ra `-diff` mà chúng ta đã thấy trước đó) thay thế việc sử dụng `strings.Index` và cắt slice bằng `strings.Cut` của Go 1.18:

<div class="beforeafter">
<pre>
i := strings.Index(s, ":")
if i >= 0 {
	 return s[:i]
}
</pre>
<div class="beforeafter-arrow"></div>
<pre>
before, _, ok := strings.Cut(s, ":")
if ok {
	return before
}
</pre>
</div>

Các công cụ hiện đại hóa này được tích hợp trong [gopls](/gopls) để cung cấp phản hồi tức thời khi bạn gõ, và trong `go fix`, để bạn có thể hiện đại hóa nhiều package hoàn chỉnh cùng lúc bằng một lệnh duy nhất. Ngoài việc làm cho mã rõ ràng hơn, các công cụ hiện đại hóa có thể giúp lập trình viên Go tìm hiểu về các tính năng mới hơn. Trong quá trình phê duyệt mỗi thay đổi mới đối với ngôn ngữ và thư viện chuẩn, nhóm đánh giá [đề xuất](https://go.googlesource.com/proposal/+/master/README.md) hiện xem xét liệu thay đổi đó có nên đi kèm với một công cụ hiện đại hóa hay không. Chúng tôi dự kiến sẽ bổ sung thêm nhiều công cụ hiện đại hóa trong mỗi bản phát hành.

## Ví dụ: một công cụ hiện đại hóa cho new(expr) mới của Go 1.26

Go 1.26 bao gồm một thay đổi nhỏ nhưng hữu ích rộng rãi đối với đặc tả ngôn ngữ. Hàm dựng sẵn `new` tạo một biến mới và trả về địa chỉ của biến đó. Trước đây, đối số duy nhất của nó bắt buộc phải là một kiểu, chẳng hạn như `new(string)`, và biến mới được khởi tạo với giá trị “zero” của nó, chẳng hạn như `""`. Trong Go 1.26, hàm `new` có thể được gọi với bất kỳ giá trị nào, khiến nó tạo một biến được khởi tạo bằng giá trị đó, tránh nhu cầu về một câu lệnh bổ sung. Ví dụ:

<div class="beforeafter">
<pre>
ptr := new(string)
*ptr = "go1.25"
</pre>
<div class="beforeafter-arrow"></div>
<pre>
ptr := new("go1.26")
</pre>
</div>

Tính năng này đã lấp đầy một khoảng trống được thảo luận trong hơn một thập kỷ và giải quyết một trong những [đề xuất](/issue/45624) phổ biến nhất về việc thay đổi ngôn ngữ. Tính năng này đặc biệt tiện lợi trong mã sử dụng kiểu con trỏ `*T` để biểu thị một giá trị tùy chọn có kiểu `T`, như thường gặp khi làm việc với các gói tuần tự hóa như [json.Marshal](https://pkg.go.dev/encoding/json#Marshal) hoặc [protocol buffers](https://protobuf.dev/getting-started/gotutorial/). Đây là một mẫu phổ biến đến mức mọi người thường đóng gói nó trong một hàm trợ giúp, chẳng hạn như hàm `newInt` bên dưới, giúp bên gọi không cần thoát khỏi ngữ cảnh biểu thức để thêm các câu lệnh bổ sung:
```
type RequestJSON struct {
	URL      string
	Attempts *int  // (tùy chọn)
}

data, err := json.Marshal(&RequestJSON{
	URL:      url,
	Attempts: newInt(10),
})

func newInt(x int) *int { return &x }
```

Các hàm trợ giúp như `newInt` thường xuyên cần thiết khi làm việc với protocol buffers đến mức chính API `proto` cũng cung cấp chúng dưới dạng [`proto.Int64`](https://pkg.go.dev/google.golang.org/protobuf/proto#Int64), [`proto.String`](https://pkg.go.dev/google.golang.org/protobuf/proto#String), v.v. Nhưng Go 1.26 khiến tất cả các hàm trợ giúp này không còn cần thiết:
```
data, err := json.Marshal(&RequestJSON{
	URL:      url,
	Attempts: new(10),
})
```
Để giúp bạn tận dụng tính năng này, lệnh `go fix` hiện bao gồm một bộ sửa lỗi, [newexpr](https://tip.golang.org/src/cmd/vendor/golang.org/x/tools/go/analysis/passes/modernize/newexpr.go), nhận dạng các hàm “giống new” như `newInt` và đề xuất các bản sửa để thay thế thân hàm bằng `return new(x)` và thay thế mọi lời gọi, dù ở cùng gói hay ở một gói nhập, bằng cách sử dụng trực tiếp `new(expr)`.

Để tránh đưa vào việc sử dụng các tính năng mới quá sớm, các công cụ hiện đại hóa chỉ cung cấp bản sửa cho những tệp yêu cầu ít nhất phiên bản Go tối thiểu phù hợp (trong trường hợp này là 1.26), thông qua một [chỉ thị `go 1.26`](/ref/mod#versions) trong tệp go.mod bao quanh hoặc một ràng buộc xây dựng `//go:build go1.26` [build constraint](https://pkg.go.dev/cmd/go#hdr-Build_constraints) trong chính tệp đó.

Chạy lệnh này để cập nhật tất cả lời gọi có dạng này trong cây mã nguồn của bạn:
```
$ go fix -newexpr ./...
```
Tại thời điểm này, nếu may mắn, tất cả các hàm trợ giúp dạng `newInt` của bạn sẽ trở thành không được sử dụng và có thể xóa an toàn (với điều kiện chúng không phải là một phần của API đã công bố ổn định). Một số lời gọi có thể vẫn còn nếu việc đề xuất bản sửa là không an toàn, chẳng hạn như khi tên `new` bị che khuất cục bộ bởi một khai báo khác. Bạn cũng có thể sử dụng lệnh [deadcode](deadcode) để giúp xác định các hàm không được sử dụng.

## Các bản sửa có tính cộng hưởng

Việc áp dụng một hiện đại hóa có thể tạo ra cơ hội để áp dụng một hiện đại hóa khác. Ví dụ, đoạn mã này, dùng để giới hạn `x` trong phạm vi 0–100, khiến trình hiện đại hóa minmax đề xuất một bản sửa để sử dụng `max`. Sau khi bản sửa đó được áp dụng, nó đề xuất một bản sửa thứ hai, lần này để sử dụng `min`.

<div class="beforeafter">
<pre>
x := f()
if x &lt; 0 {
	x = 0
}
if x > 100 {
	x = 100
}
</pre>
<div class="beforeafter-arrow"></div>
<pre>
x := min(max(f(), 0), 100)
</pre>
</div>

Các hiện tượng cộng hưởng cũng có thể xảy ra giữa những bộ phân tích khác nhau. Ví dụ, một lỗi phổ biến là liên tục nối chuỗi trong một vòng lặp, dẫn đến độ phức tạp thời gian bậc hai—một lỗi và một vector tiềm năng cho cuộc tấn công từ chối dịch vụ. Trình hiện đại hóa `stringsbuilder` nhận ra vấn đề này và đề xuất sử dụng `strings.Builder` của Go 1.10:

<div class="beforeafter">
<pre>
s := ""
for _, b := range bytes {
	s += fmt.Sprintf("%02x", b)
}
use(s)
</pre>
<div class="beforeafter-arrow"></div>
<pre>
var s strings.Builder
for _, b := range bytes {
	s.WriteString(fmt.Sprintf("%02x", b))
}
use(s.String())
</pre>
</div>

Sau khi bản sửa này được áp dụng, một bộ phân tích thứ hai có thể nhận ra rằng các thao tác `WriteString` và `Sprintf` có thể được kết hợp thành `fmt.Fprintf(&s, "%02x", b)`, vừa gọn gàng hơn vừa hiệu quả hơn, và đưa ra một bản sửa thứ hai. (Bộ phân tích thứ hai này là [QF1012](https://staticcheck.dev/docs/checks#QF1012) từ [staticcheck](https://staticcheck.dev/) của Dominik Honnef, hiện đã được bật trong gopls nhưng chưa có trong `go fix`, mặc dù chúng tôi [dự định](/issue/76918) thêm các bộ phân tích staticcheck vào lệnh go bắt đầu từ Go 1.27.)

Do đó, có thể đáng để chạy `go fix` nhiều hơn một lần cho đến khi đạt đến điểm bất động; hai lần thường là đủ.

<!-- Lưu ý: Lý do công cụ không tự áp dụng việc lặp đến điểm bất động là vì (a) dù đã nỗ lực, vẫn có khả năng khác không rằng phép biến đổi làm hỏng bản build, khiến hầu hết các bộ phân tích (những bộ không được đánh dấu RunDespiteErrors) không thể chạy ở lượt thứ hai, và (b) các phép biến đổi trong vòng sửa lỗi đầu tiên có thể thêm import cho những gói mà thông tin kiểu không có sẵn, yêu cầu “build” phải được khởi động lại, điều này là không thể trong nhiều driver như Blaze, nogo, Tricorder, v.v. Về cơ bản, đây là hệ quả của việc framework phân tích được thiết kế giống một bản build phân tán (xử lý theo lô, hạt thô, hàm thuần phân tán) chứ không giống một IDE (các thay đổi cục bộ tương tác, hạt mịn). -->

### Hợp nhất các bản sửa và xung đột

Một lần chạy duy nhất của `go fix` có thể áp dụng hàng chục bản sửa trong cùng một tệp mã nguồn. Về mặt khái niệm, tất cả các bản sửa đều độc lập, tương tự như một tập hợp các commit git có cùng commit cha. Lệnh `go fix` sử dụng một thuật toán hợp nhất ba chiều đơn giản để xử lý tuần tự các bản sửa, tương tự như nhiệm vụ hợp nhất một tập hợp các commit git chỉnh sửa cùng một tệp. Nếu một bản sửa xung đột với danh sách các chỉnh sửa đã được tích lũy cho đến lúc đó, nó sẽ bị loại bỏ và công cụ đưa ra cảnh báo rằng một số bản sửa đã bị bỏ qua và công cụ nên được chạy lại.

Cách này phát hiện đáng tin cậy các xung đột *cú pháp* phát sinh từ những chỉnh sửa chồng lấn, nhưng có thể xảy ra một lớp xung đột khác: xung đột *ngữ nghĩa* xảy ra khi hai thay đổi độc lập về mặt văn bản nhưng ý nghĩa của chúng không tương thích. Ví dụ, hãy xem xét hai bản sửa mà mỗi bản đều xóa lần sử dụng áp chót của một biến cục bộ: mỗi bản sửa đều đúng khi hoạt động riêng lẻ, nhưng khi áp dụng cả hai cùng lúc, biến cục bộ trở thành biến không được sử dụng, và trong Go đó là một lỗi biên dịch. Không bản sửa nào chịu trách nhiệm xóa khai báo biến, nhưng cần có ai đó thực hiện việc đó, và người đó là người dùng `go fix`.

Một xung đột ngữ nghĩa tương tự xảy ra khi một tập hợp các bản sửa khiến một import trở thành không được sử dụng. Vì trường hợp này rất phổ biến, lệnh `go fix` thực hiện một lượt xử lý cuối để phát hiện các import không được sử dụng và tự động loại bỏ chúng.

Các xung đột ngữ nghĩa tương đối hiếm. May mắn là chúng thường tự bộc lộ dưới dạng lỗi biên dịch, khiến chúng không thể bị bỏ qua. Đáng tiếc là khi xảy ra, chúng vẫn đòi hỏi một số công việc thủ công sau khi chạy `go fix`.

Bây giờ hãy tìm hiểu sâu hơn về hạ tầng bên dưới các công cụ này.

<a name='go/analysis'></a>
## Khung phân tích Go

Từ những ngày đầu của Go, lệnh `go` đã có hai lệnh con để phân tích tĩnh, `go vet` và `go fix`, mỗi lệnh có một bộ thuật toán riêng: “checker” và “fixer”. Một checker báo cáo những lỗi có khả năng xảy ra trong mã của bạn, chẳng hạn như truyền một chuỗi thay vì một số nguyên làm toán hạng của phép chuyển đổi `fmt.Printf("%d")`. Một fixer chỉnh sửa mã của bạn một cách an toàn để sửa lỗi hoặc biểu diễn cùng một điều theo cách tốt hơn, có thể rõ ràng hơn, ngắn gọn hơn hoặc hiệu quả hơn. Đôi khi cùng một thuật toán xuất hiện trong cả hai bộ khi nó vừa có thể báo cáo lỗi vừa có thể sửa lỗi một cách an toàn.

Vào năm 2017, chúng tôi đã thiết kế lại chương trình `go vet` vốn nguyên khối lúc đó để tách các thuật toán checker (nay được gọi là “analyzer”) khỏi “driver”, chương trình chạy chúng; kết quả là [khung phân tích Go](https://pkg.go.dev/golang.org/x/tools/go/analysis). Sự phân tách này cho phép viết một analyzer một lần rồi chạy nó trong nhiều driver khác nhau cho các môi trường khác nhau, chẳng hạn như:

- [unitchecker](https://pkg.go.dev/golang.org/x/tools/go/analysis/unitchecker), biến một bộ analyzer thành một lệnh con có thể được chạy bởi hệ thống build tăng dần có khả năng mở rộng của lệnh go, tương tự như trình biên dịch trong go build. Đây là nền tảng của `go fix` và `go vet`.
- [nogo](https://github.com/bazel-contrib/rules_go/blob/master/go/nogo.rst), driver tương tự cho các hệ thống build thay thế như Bazel và Blaze.
- [singlechecker](https://pkg.go.dev/golang.org/x/tools/go/analysis/singlechecker), biến một analyzer thành một lệnh độc lập tải, phân tích cú pháp và kiểm tra kiểu của một tập hợp các gói (có thể là toàn bộ chương trình), sau đó phân tích chúng. Chúng tôi thường sử dụng nó cho các thử nghiệm và phép đo đặc biệt trên corpus của bản sao module ([proxy.golang.org](https://proxy.golang.org/)) corpus.
- [multichecker](https://pkg.go.dev/golang.org/x/tools/go/analysis/multichecker), thực hiện điều tương tự cho một bộ analyzer với CLI kiểu ‘dao quân đội Thụy Sĩ’.
- [gopls](/gopls), [language server](https://microsoft.github.io/language-server-protocol/) đứng sau VS Code và các trình soạn thảo khác, cung cấp chẩn đoán theo thời gian thực từ các analyzer sau mỗi lần nhấn phím trong trình soạn thảo.
- driver có khả năng cấu hình cao được sử dụng bởi công cụ [staticcheck](https://staticcheck.dev/). (Staticcheck cũng cung cấp một bộ lớn các analyzer có thể chạy trong các driver khác.)
- [Tricorder](https://research.google/pubs/tricorder-building-a-program-analysis-ecosystem/), pipeline phân tích tĩnh theo lô được Google sử dụng cho monorepo của mình và tích hợp với hệ thống xem xét mã.
- [MCP server](/gopls/features/mcp) của gopls, giúp các agent lập trình dựa trên LLM có thể truy cập chẩn đoán, cung cấp “guardrails” mạnh mẽ hơn.
- [analysistest](https://pkg.go.dev/golang.org/x/tools/go/analysis/analysistest), bộ công cụ kiểm thử của khung phân tích.

Một lợi ích của khung này là khả năng biểu diễn các analyzer hỗ trợ không báo cáo chẩn đoán hoặc đề xuất bản sửa của riêng chúng, mà thay vào đó tính toán một cấu trúc dữ liệu trung gian có thể hữu ích cho nhiều analyzer khác, giúp phân bổ chi phí xây dựng cấu trúc đó. Ví dụ bao gồm [đồ thị luồng điều khiển](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/ctrlflow), [biểu diễn SSA](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/buildssa) của thân hàm, và các cấu trúc dữ liệu cho [điều hướng AST được tối ưu hóa](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/inspect).

Một lợi ích khác của framework là khả năng hỗ trợ đưa ra các suy luận trên nhiều gói. Một analyzer có thể gắn một “[fact](https://pkg.go.dev/golang.org/x/tools/go/analysis#hdr-Modular_analysis_with_Facts)” vào một hàm hoặc biểu tượng khác, để thông tin thu được trong quá trình phân tích thân hàm có thể được sử dụng khi phân tích một lời gọi đến hàm đó sau này, ngay cả khi lời gọi xuất hiện trong một gói khác hoặc quá trình phân tích sau đó diễn ra trong một tiến trình khác. Điều này giúp dễ dàng định nghĩa các phân tích liên thủ tục có khả năng mở rộng. Ví dụ, trình kiểm tra printf có thể nhận biết khi một hàm như `log.Printf` thực chất chỉ là một wrapper quanh `fmt.Printf`, vì vậy nó biết rằng các lời gọi đến `log.Printf` nên được kiểm tra theo cách tương tự. Quá trình này hoạt động bằng quy nạp, nên công cụ cũng sẽ kiểm tra các lời gọi đến những wrapper tiếp theo quanh `log.Printf`, v.v. Một ví dụ về analyzer sử dụng nhiều fact là [nilaway của Uber](https://github.com/uber-go/nilaway), công cụ này báo cáo các lỗi tiềm ẩn dẫn đến việc giải tham chiếu con trỏ nil.

<img src="gofix-analysis-facts.svg">

Quá trình “phân tích riêng biệt” trong `go fix` tương tự với quá trình biên dịch riêng biệt trong `go build`. Cũng như trình biên dịch xây dựng các gói bắt đầu từ đáy của đồ thị dependency và truyền thông tin kiểu lên các gói nhập, framework phân tích hoạt động từ đáy của đồ thị dependency lên trên, truyền các fact (và kiểu) lên các gói nhập.

Vào năm 2019, khi chúng tôi bắt đầu phát triển [gopls](/gopls), máy chủ ngôn ngữ cho Go, chúng tôi đã thêm khả năng để một analyzer đề xuất một [fix](https://pkg.go.dev/golang.org/x/tools/go/analysis#SuggestedFix) khi báo cáo một chẩn đoán. Ví dụ, analyzer printf đề xuất thay thế `fmt.Printf(msg)` bằng `fmt.Printf("%s", msg)` để tránh định dạng sai nếu giá trị `msg` động chứa ký hiệu `%`. Cơ chế này đã trở thành nền tảng cho nhiều tính năng sửa nhanh và tái cấu trúc của gopls.

Trong khi tất cả những phát triển này đang diễn ra với `go vet`, `go fix` vẫn bị mắc kẹt ở trạng thái như trước [cam kết tương thích của Go](/doc/go1compat), khi những người dùng Go đầu tiên đã sử dụng nó để duy trì mã nguồn của họ trong giai đoạn ngôn ngữ và các thư viện phát triển nhanh chóng và đôi khi không tương thích.

Bản phát hành Go 1.26 đưa framework phân tích của Go vào `go fix`. Các lệnh `go vet` và `go fix` đã hội tụ và hiện gần như giống hệt nhau về cách triển khai. Những khác biệt duy nhất giữa chúng là tiêu chí cho các bộ thuật toán mà chúng sử dụng, và cách chúng xử lý các chẩn đoán đã tính toán. Các [vet analyzers](https://cs.opensource.google/go/go/+/refs/tags/go1.26rc1:src/cmd/vet/main.go;l=62) của Go phải phát hiện các lỗi có khả năng xảy ra với ít dương tính giả; các chẩn đoán của chúng được báo cáo cho người dùng. Các [fix analyzers](https://cs.opensource.google/go/go/+/refs/tags/go1.26rc1:src/cmd/fix/main.go;l=46) của Go phải tạo ra các bản sửa an toàn để áp dụng mà không làm suy giảm tính đúng đắn, hiệu năng hoặc phong cách; các chẩn đoán của chúng có thể không được báo cáo, nhưng các bản sửa được áp dụng trực tiếp. Ngoài sự khác biệt về trọng tâm này, nhiệm vụ phát triển một fixer không khác gì so với việc phát triển một checker.

### Cải thiện cơ sở hạ tầng phân tích

Khi số lượng analyzer trong `go vet` và `go fix` tiếp tục tăng, chúng tôi đã đầu tư vào cơ sở hạ tầng nhằm vừa cải thiện hiệu năng của từng analyzer vừa giúp việc viết từng analyzer mới trở nên dễ dàng hơn.

Ví dụ, hầu hết analyzer bắt đầu bằng cách duyệt cây cú pháp của từng tệp trong gói để tìm một loại nút cụ thể, chẳng hạn như câu lệnh range hoặc literal hàm. Gói [inspector](https://pkg.go.dev/golang.org/x/tools/go/ast/inspector) hiện có giúp việc quét này hiệu quả bằng cách tính trước một chỉ mục gọn nhẹ của toàn bộ quá trình duyệt, để các lần duyệt sau có thể nhanh chóng bỏ qua các cây con không chứa nút cần quan tâm. Gần đây chúng tôi đã mở rộng nó với kiểu dữ liệu [Cursor](https://pkg.go.dev/golang.org/x/tools/go/ast/inspector#Cursor) để cho phép điều hướng linh hoạt và hiệu quả giữa các nút theo cả bốn hướng chính—lên, xuống, trái và phải, tương tự như điều hướng các phần tử của HTML DOM—giúp việc biểu diễn một truy vấn như “tìm từng câu lệnh go là câu lệnh đầu tiên của thân vòng lặp” trở nên dễ dàng và hiệu quả:
```
	var curFile inspector.Cursor = ...

	// Find each go statement that is the first statement of a loop body.
	for curGo := range curFile.Preorder((*ast.GoStmt)(nil)) {
		kind, index := curGo.ParentEdge()
		if kind == edge.BlockStmt_List && index == 0 {
			switch curGo.Parent().ParentEdgeKind() {
			case edge.ForStmt_Body, edge.RangeStmt_Body:
				...
			}
		}
	}
```

Nhiều analyzer bắt đầu bằng việc tìm kiếm lời gọi đến một hàm cụ thể, chẳng hạn như `fmt.Printf`. Lời gọi hàm là một trong những biểu thức có số lượng nhiều nhất trong mã Go, vì vậy thay vì tìm kiếm mọi biểu thức lời gọi và kiểm tra xem đó có phải là lời gọi đến `fmt.Printf` hay không, hiệu quả hơn nhiều là tính trước một chỉ mục của các tham chiếu ký hiệu, việc này được thực hiện bởi analyzer [typeindex](https://pkg.go.dev/golang.org/x/tools/internal/typesinternal/typeindex) và analyzer [helper](https://pkg.go.dev/golang.org/x/tools@v0.41.0/internal/analysis/typeindex) của nó. Sau đó, các lời gọi đến `fmt.Printf` có thể được liệt kê trực tiếp, khiến chi phí tỉ lệ với số lượng lời gọi thay vì kích thước của gói. Với một analyzer như [hostport](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/hostport) tìm kiếm một ký hiệu ít được sử dụng (`net.Dial`), điều này có thể dễ dàng giúp nó [nhanh hơn 1.000×](/cl/657958).

Một số cải tiến cơ sở hạ tầng khác trong năm qua bao gồm:

- một **đồ thị dependency của thư viện chuẩn** mà các analyzer có thể tham khảo để tránh tạo ra chu kỳ import. Ví dụ, chúng ta không thể thêm lời gọi đến `strings.Cut` trong một gói được chính `strings` import.
- hỗ trợ **truy vấn phiên bản Go hiệu lực** của một tệp được xác định bởi tệp go.mod bao quanh và các build tag, để các analyzer không chèn cách sử dụng những tính năng “quá mới”.
- một **thư viện phong phú hơn về các nguyên thủy tái cấu trúc** (ví dụ “xóa câu lệnh này”) xử lý đúng các chú thích liền kề và những trường hợp biên phức tạp khác.

Chúng tôi đã tiến được một chặng đường dài, nhưng vẫn còn nhiều việc phải làm. Logic của fixer có thể khó để làm cho đúng. Vì chúng tôi mong người dùng áp dụng hàng trăm bản sửa được đề xuất chỉ với việc xem xét sơ lược, điều quan trọng là các fixer phải chính xác ngay cả trong những trường hợp biên ít gặp. Chỉ với một ví dụ (xem [bài nói](https://www.youtube.com/watch?v=_VePjjjV9JU&t=13m17s) của tôi tại GopherCon để xem thêm nhiều ví dụ khác), chúng tôi đã xây dựng một modernizer thay thế các lời gọi như `append([]string{}, slice...)` bằng `slices.Clone(slice)` rõ ràng hơn, chỉ để phát hiện rằng khi `slice` rỗng, kết quả của Clone là nil, một thay đổi hành vi tinh tế có thể gây lỗi trong những trường hợp hiếm; vì vậy chúng tôi phải loại trừ [modernizer đó](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/modernize#hdr-Analyzer_appendclipped) khỏi bộ `go fix`.

Một số khó khăn này đối với các tác giả của analyzer có thể được giảm bớt bằng tài liệu tốt hơn (cho cả con người và LLM), đặc biệt là các danh sách kiểm tra những trường hợp biên bất ngờ cần xem xét và kiểm thử. Một công cụ khớp mẫu cho cây cú pháp, tương tự như các công cụ trong [staticcheck](https://pkg.go.dev/honnef.co/go/tools/pattern) và [Tree Sitter](https://tree-sitter.github.io/tree-sitter/using-parsers/queries/index.html), có thể đơn giản hóa công việc tỉ mỉ là xác định hiệu quả các vị trí cần sửa. Một thư viện phong phú hơn về các toán tử để tính toán các bản sửa chính xác sẽ giúp tránh những lỗi phổ biến. Một bộ kiểm thử tốt hơn sẽ cho phép chúng tôi kiểm tra rằng các bản sửa không làm hỏng quá trình build và bảo toàn các thuộc tính động của mã đích. Tất cả những điều này đều nằm trong lộ trình của chúng tôi.

<a name='self-service'></a>
## Mô hình “self-service”

Về cơ bản hơn, chúng tôi đang chuyển sự chú ý vào năm 2026 sang một mô hình “self-service”.

Analyzer `newexpr` mà chúng ta đã thấy trước đó là một modernizer điển hình: một thuật toán riêng biệt được điều chỉnh cho một tính năng cụ thể. Mô hình riêng biệt hoạt động tốt với các tính năng của ngôn ngữ và thư viện chuẩn, nhưng nó không thực sự giúp cập nhật việc sử dụng các gói bên thứ ba. Mặc dù không có gì ngăn bạn viết một modernizer cho các API công khai của riêng mình và chạy nó trên dự án của bạn, không có cách tự động nào để khiến người dùng API của bạn cũng chạy nó. Modernizer của bạn có lẽ sẽ không thuộc về gopls hoặc bộ `go vet` trừ khi API của bạn được sử dụng đặc biệt rộng rãi trong hệ sinh thái Go. Ngay cả trong trường hợp đó, bạn vẫn phải có được các đánh giá mã và phê duyệt rồi chờ bản phát hành tiếp theo.

Theo mô hình self-service, các lập trình viên Go sẽ có thể định nghĩa các bản hiện đại hóa cho API của riêng họ mà người dùng của họ có thể áp dụng mà không gặp tất cả nút thắt của mô hình tập trung hiện tại. Điều này đặc biệt quan trọng khi cộng đồng Go và kho mã Go toàn cầu đang phát triển nhanh hơn nhiều so với khả năng của đội ngũ chúng tôi trong việc xem xét các đóng góp analyzer.

Lệnh `go fix` trong Go 1.26 bao gồm bản xem trước của những thành quả đầu tiên từ mô hình mới này: **trình nội tuyến ở cấp mã nguồn dựa trên chú thích**, được mô tả trong [bài đăng tiếp theo](inliner). Trong năm tới, chúng tôi dự định nghiên cứu thêm hai cách tiếp cận khác trong mô hình này.

Đầu tiên, chúng tôi sẽ khám phá khả năng [tải động](/issue/59869) các công cụ hiện đại hóa từ cây mã nguồn và thực thi chúng một cách an toàn, trong gopls hoặc `go fix`. Theo cách tiếp cận này, một gói cung cấp API cho, chẳng hạn, cơ sở dữ liệu SQL có thể đồng thời cung cấp một trình kiểm tra các cách sử dụng API không đúng, như lỗ hổng SQL injection hoặc việc không xử lý các lỗi nghiêm trọng. Cùng một cơ chế có thể được các maintainer của dự án sử dụng để mã hóa các quy tắc bảo trì nội bộ, chẳng hạn như tránh gọi một số hàm có vấn đề hoặc áp dụng các kỷ luật lập trình chặt chẽ hơn trong những phần quan trọng của mã.

Thứ hai, nhiều trình kiểm tra hiện có có thể được mô tả không chính thức là “đừng quên làm X sau khi bạn làm Y!”, chẳng hạn như “đóng tệp sau khi mở nó”, “hủy context sau khi tạo nó”, “mở khóa mutex sau khi khóa nó”, “thoát khỏi vòng lặp iterator sau khi yield trả về false”, v.v. Điểm chung của các trình kiểm tra như vậy là chúng áp dụng một số bất biến nhất định trên mọi đường dẫn thực thi. Chúng tôi dự định khám phá các khái quát hóa và hợp nhất của những trình kiểm tra luồng điều khiển này để các lập trình viên Go có thể dễ dàng áp dụng chúng vào các miền mới, mà không cần logic phân tích phức tạp, chỉ bằng cách chú thích chính mã của họ.

Chúng tôi hy vọng rằng những công cụ mới này sẽ giúp bạn tiết kiệm công sức trong quá trình bảo trì các dự án Go của mình và giúp bạn tìm hiểu cũng như tận dụng các tính năng mới hơn sớm hơn. Hãy thử `go fix` trên các dự án của bạn và [báo cáo](/issue/new) mọi vấn đề bạn tìm thấy, đồng thời chia sẻ mọi ý tưởng bạn có về các công cụ hiện đại hóa, trình sửa lỗi, trình kiểm tra mới hoặc các cách tiếp cận tự phục vụ cho phân tích tĩnh.

<!--
Local Variables:
indent-tabs-mode: nil
tab-width: 4
End:
-->
