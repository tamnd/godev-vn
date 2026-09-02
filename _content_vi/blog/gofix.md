---
title: "Dùng go fix để hiện đại hóa code Go"
date: 2026-02-17
by:
- Alan Donovan
tags:
- go fix
- go vet
- analysis framework
- modernizers
- static analysis
summary: "Go 1.26 có triển khai mới của go fix có thể giúp bạn dùng các tính năng hiện đại hơn của Go."
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


Bản phát hành 1.26 của Go tháng này bao gồm subcommand go fix được viết lại hoàn toàn. Go fix sử dụng một bộ thuật toán để xác định các cơ hội cải thiện code của bạn, thường bằng cách tận dụng các tính năng hiện đại hơn của ngôn ngữ và thư viện. Trong bài đăng này, trước tiên chúng tôi sẽ chỉ cho bạn cách dùng `go fix` để hiện đại hóa codebase Go của bạn. Sau đó trong [phần thứ hai](#go/analysis) chúng tôi sẽ đi sâu vào cơ sở hạ tầng phía sau nó và cách nó đang phát triển. Cuối cùng, chúng tôi sẽ trình bày chủ đề ["tự phục vụ"](#self-service) - các công cụ phân tích để giúp người dùng module và tổ chức mã hóa các hướng dẫn và thực hành tốt nhất của riêng họ.

## Chạy go fix

Lệnh `go fix`, giống như `go build` và `go vet`, nhận một tập hợp các pattern biểu thị các gói. Lệnh này sửa tất cả các gói trong thư mục hiện tại:
```
$ go fix ./...
```
Khi thành công, nó âm thầm cập nhật các file nguồn của bạn. Nó loại bỏ bất kỳ sửa lỗi nào chạm vào [các file được tạo ra](https://pkg.go.dev/cmd/go#hdr-Generate_Go_files_by_processing_source) vì sửa lỗi phù hợp trong trường hợp đó là đối với logic của chính generator. Chúng tôi khuyến nghị chạy `go fix` trên dự án của bạn mỗi khi bạn cập nhật build lên bản phát hành Go toolchain mới hơn. Vì lệnh có thể sửa hàng trăm file, hãy bắt đầu từ trạng thái git sạch sẽ để thay đổi chỉ bao gồm các chỉnh sửa từ go fix; người đánh giá code của bạn sẽ cảm ơn bạn.

Để xem trước những thay đổi mà lệnh trên sẽ thực hiện, dùng cờ `-diff`:
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

Bạn có thể liệt kê các fixer có sẵn bằng cách chạy lệnh này:
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

Thêm tên của một analyzer cụ thể sẽ hiển thị tài liệu đầy đủ của nó:
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
Mặc định, lệnh `go fix` chạy tất cả các analyzer. Khi sửa một dự án lớn, có thể giảm gánh nặng đánh giá code nếu bạn áp dụng các sửa lỗi từ các analyzer năng suất cao nhất như các thay đổi code riêng biệt. Để chỉ bật các analyzer cụ thể, dùng các cờ khớp với tên của chúng. Ví dụ, để chỉ chạy fixer `any`, chỉ định cờ `-any`. Ngược lại, để chạy tất cả các analyzer *ngoại trừ* các analyzer được chọn, hãy phủ định các cờ, ví dụ `-any=false`.

Như với `go build` và `go vet`, mỗi lần chạy lệnh `go fix` chỉ phân tích một cấu hình build cụ thể. Nếu dự án của bạn sử dụng nhiều file được gắn thẻ cho các CPU hoặc nền tảng khác nhau, bạn có thể muốn chạy lệnh nhiều hơn một lần với các giá trị `GOARCH` và `GOOS` khác nhau để có độ bao phủ tốt hơn:
```
$ GOOS=linux   GOARCH=amd64 go fix ./...
$ GOOS=darwin  GOARCH=arm64 go fix ./...
$ GOOS=windows GOARCH=amd64 go fix ./...
```
Chạy lệnh nhiều hơn một lần cũng tạo ra cơ hội cho các sửa lỗi hiệp lực, như chúng ta sẽ thấy bên dưới.

### Các công cụ hiện đại hóa

Việc giới thiệu [generics](intro-generics) trong Go 1.18 đánh dấu sự kết thúc của một kỷ nguyên có rất ít thay đổi đối với đặc tả ngôn ngữ và bắt đầu của một giai đoạn thay đổi nhanh hơn - dù vẫn cẩn thận - đặc biệt là trong các thư viện. Nhiều vòng lặp tầm thường mà lập trình viên Go thường viết, chẳng hạn như để thu thập các khóa của map vào một slice, giờ có thể được biểu diễn thuận tiện như một lời gọi đến hàm generic như [`maps.Keys`](https://pkg.go.dev/maps#Keys). Do đó các tính năng mới này tạo ra nhiều cơ hội để đơn giản hóa code hiện có.

Vào tháng 12 năm 2024, trong làn sóng áp dụng các trợ lý coding LLM ồ ạt, chúng tôi nhận thấy rằng các công cụ như vậy có xu hướng - không ngạc nhiên - tạo ra code Go theo phong cách tương tự với khối lượng lớn code Go được dùng trong quá trình đào tạo, ngay cả khi có những cách mới hơn, tốt hơn để diễn đạt cùng một ý tưởng. Ít rõ ràng hơn, các công cụ tương tự thường từ chối sử dụng các cách mới hơn ngay cả khi được chỉ đạo làm như vậy theo thuật ngữ chung như "luôn dùng các idiom mới nhất của Go 1.25." Trong một số trường hợp, ngay cả khi được yêu cầu rõ ràng để sử dụng một tính năng, mô hình sẽ phủ nhận rằng nó tồn tại. (Xem bài nói chuyện GopherCon 2025 của tôi [tại đây](https://www.youtube.com/watch?v=_VePjjjV9JU&t=3m50s) để biết thêm chi tiết gây bực bội.) Để đảm bảo rằng các mô hình tương lai được đào tạo trên các idiom mới nhất, chúng tôi cần đảm bảo rằng các idiom này được phản ánh trong dữ liệu đào tạo, tức là corpus toàn cầu của code Go mã nguồn mở.

Trong năm qua, chúng tôi đã xây dựng [hàng chục analyzer](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/modernize) để xác định các cơ hội hiện đại hóa. Đây là ba ví dụ về các sửa lỗi mà chúng đề xuất:

**minmax** thay thế câu lệnh `if` bằng cách dùng hàm `min` hoặc `max` của Go 1.21:

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

**rangeint** thay thế vòng lặp `for` 3 mệnh đề bằng vòng lặp `range`-over-int của Go 1.22:

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

**stringscut** (mà đầu ra `-diff` chúng ta đã thấy trước đó) thay thế cách dùng `strings.Index` và slicing bằng `strings.Cut` của Go 1.18:

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

Các công cụ hiện đại hóa này được bao gồm trong [gopls](/gopls), để cung cấp phản hồi tức thì khi bạn gõ, và trong `go fix`, để bạn có thể hiện đại hóa một số gói đầy đủ cùng một lúc trong một lệnh duy nhất. Ngoài việc làm cho code rõ ràng hơn, các công cụ hiện đại hóa có thể giúp lập trình viên Go tìm hiểu về các tính năng mới hơn. Là một phần của quy trình phê duyệt mỗi thay đổi mới đối với ngôn ngữ và thư viện chuẩn, nhóm đánh giá [đề xuất](https://go.googlesource.com/proposal/+/master/README.md) giờ xem xét liệu nó có nên đi kèm với một công cụ hiện đại hóa không. Chúng tôi kỳ vọng thêm nhiều công cụ hiện đại hóa hơn với mỗi bản phát hành.

## Ví dụ: công cụ hiện đại hóa cho new(expr) mới trong Go 1.26

Go 1.26 bao gồm một thay đổi nhỏ nhưng hữu ích rộng rãi cho đặc tả ngôn ngữ. Hàm tích hợp `new` tạo một biến mới và trả về địa chỉ của nó. Trong lịch sử, đối số duy nhất của nó bắt buộc phải là một kiểu, chẳng hạn như `new(string)`, và biến mới được khởi tạo thành giá trị "zero" của nó, chẳng hạn như `""`. Trong Go 1.26, hàm `new` có thể được gọi với bất kỳ giá trị nào, khiến nó tạo ra một biến được khởi tạo thành giá trị đó, tránh cần thêm câu lệnh. Ví dụ:

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

Tính năng này lấp đầy một khoảng trống đã được thảo luận trong hơn một thập kỷ và giải quyết một trong những [đề xuất](/issue/45624) phổ biến nhất cho một thay đổi ngôn ngữ. Nó đặc biệt thuận tiện trong code sử dụng kiểu con trỏ `*T` để biểu thị giá trị tùy chọn của kiểu `T`, như thường gặp khi làm việc với các gói serialization như [json.Marshal](https://pkg.go.dev/encoding/json#Marshal) hoặc [protocol buffers](https://protobuf.dev/getting-started/gotutorial/). Đây là một pattern phổ biến đến nỗi mọi người thường bắt nó trong một helper, chẳng hạn như hàm `newInt` bên dưới, giúp người gọi không cần phải thoát khỏi ngữ cảnh biểu thức để giới thiệu các câu lệnh bổ sung:
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

Các helper như `newInt` rất thường xuyên cần thiết với protocol buffers đến nỗi API `proto` tự cung cấp chúng như [`proto.Int64`](https://pkg.go.dev/google.golang.org/protobuf/proto#Int64), [`proto.String`](https://pkg.go.dev/google.golang.org/protobuf/proto#String), và v.v. Nhưng Go 1.26 làm cho tất cả các helper này không cần thiết:
```
data, err := json.Marshal(&RequestJSON{
	URL:      url,
	Attempts: new(10),
})
```
Để giúp bạn tận dụng tính năng này, lệnh `go fix` giờ bao gồm một fixer, [newexpr](https://tip.golang.org/src/cmd/vendor/golang.org/x/tools/go/analysis/passes/modernize/newexpr.go), nhận ra các hàm "giống new" như `newInt` và đề xuất các sửa lỗi để thay thế thân hàm bằng `return new(x)` và thay thế mọi lời gọi, dù trong cùng gói hay gói nhập, bằng cách dùng trực tiếp `new(expr)`.

Để tránh đưa vào sớm các cách dùng của các tính năng mới, các công cụ hiện đại hóa chỉ đề xuất sửa lỗi trong các file yêu cầu ít nhất phiên bản Go tối thiểu phù hợp (1.26 trong trường hợp này), thông qua chỉ thị [`go 1.26`](/ref/mod#versions) trong file go.mod bao quanh hoặc ràng buộc build `//go:build go1.26` trong file đó.

Chạy lệnh này để cập nhật tất cả các lời gọi có dạng này trong cây nguồn của bạn:
```
$ go fix -newexpr ./...
```
Lúc này, may mắn thay, tất cả các hàm helper kiểu `newInt` của bạn sẽ không còn được dùng và có thể được xóa an toàn (giả sử chúng không phải là một phần của API ổn định đã xuất bản). Một số lời gọi có thể vẫn còn nơi không an toàn để đề xuất sửa lỗi, chẳng hạn như khi tên `new` bị che khuất bởi khai báo khác. Bạn cũng có thể dùng lệnh [deadcode](deadcode) để giúp xác định các hàm không được dùng.

## Các sửa lỗi hiệp lực

Áp dụng một hiện đại hóa có thể tạo ra cơ hội áp dụng một cái khác. Ví dụ, đoạn code này, giới hạn `x` trong khoảng 0-100, khiến công cụ hiện đại hóa minmax đề xuất sửa lỗi để dùng `max`. Khi sửa lỗi đó được áp dụng, nó đề xuất sửa lỗi thứ hai, lần này để dùng `min`.

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

Hiệp lực cũng có thể xảy ra giữa các analyzer khác nhau. Ví dụ, một lỗi phổ biến là liên tục nối chuỗi trong vòng lặp, dẫn đến độ phức tạp thời gian bậc hai - một lỗi và vectơ tiềm năng cho cuộc tấn công từ chối dịch vụ. Công cụ hiện đại hóa `stringsbuilder` nhận ra vấn đề và đề xuất dùng `strings.Builder` của Go 1.10:

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

Khi sửa lỗi này được áp dụng, một analyzer thứ hai có thể nhận ra rằng các thao tác `WriteString` và `Sprintf` có thể được kết hợp thành `fmt.Fprintf(&s, "%02x", b)`, vừa rõ ràng hơn vừa hiệu quả hơn, và đề xuất sửa lỗi thứ hai. (Analyzer thứ hai này là [QF1012](https://staticcheck.dev/docs/checks#QF1012) từ [staticcheck](https://staticcheck.dev/) của Dominik Honnef, đã được bật trong gopls nhưng chưa có trong `go fix`, dù chúng tôi [có kế hoạch](/issue/76918) thêm các analyzer staticcheck vào lệnh go bắt đầu từ Go 1.27.)

Do đó, có thể đáng chạy `go fix` nhiều hơn một lần cho đến khi nó đạt điểm cố định; hai lần thường là đủ.

### Gộp sửa lỗi và xung đột

Một lần chạy `go fix` có thể áp dụng hàng chục sửa lỗi trong cùng file nguồn. Tất cả các sửa lỗi đều độc lập về mặt khái niệm, tương tự như một tập hợp git commit với cùng parent. Lệnh `go fix` sử dụng thuật toán gộp ba chiều đơn giản để hòa giải các sửa lỗi theo trình tự, tương tự như nhiệm vụ gộp một tập hợp các git commit chỉnh sửa cùng file. Nếu một sửa lỗi xung đột với danh sách các chỉnh sửa tích lũy cho đến nay, nó bị loại bỏ, và công cụ phát ra cảnh báo rằng một số sửa lỗi đã bị bỏ qua và công cụ nên được chạy lại.

Điều này đáng tin cậy phát hiện các xung đột *cú pháp* phát sinh từ các chỉnh sửa chồng chéo, nhưng một loại xung đột khác là có thể: một xung đột *ngữ nghĩa* xảy ra khi hai thay đổi độc lập về mặt văn bản nhưng ý nghĩa của chúng không tương thích. Ví dụ xem xét hai sửa lỗi mỗi sửa lỗi xóa cách dùng thứ hai đến cuối của biến cục bộ: mỗi sửa lỗi đều ổn bởi chính nó, nhưng khi cả hai được áp dụng cùng nhau, biến cục bộ trở nên không được dùng, và trong Go đó là lỗi biên dịch. Không sửa lỗi nào chịu trách nhiệm xóa khai báo biến, nhưng ai đó phải làm, và người đó là người dùng `go fix`.

Xung đột ngữ nghĩa tương tự xảy ra khi một tập hợp các sửa lỗi khiến một import trở nên không được dùng. Vì trường hợp này rất phổ biến, lệnh `go fix` áp dụng một lần duyệt cuối cùng để phát hiện các import không được dùng và tự động xóa chúng.

Xung đột ngữ nghĩa tương đối hiếm. May mắn thay, chúng thường bộc lộ như các lỗi biên dịch, khiến chúng không thể bỏ qua. Thật không may, khi chúng xảy ra, chúng đòi hỏi một số công việc thủ công sau khi chạy `go fix`.

Bây giờ hãy đi sâu vào cơ sở hạ tầng bên dưới các công cụ này.

<a name='go/analysis'></a>
## Framework phân tích Go

Từ những ngày đầu của Go, lệnh `go` đã có hai subcommand để phân tích tĩnh, `go vet` và `go fix`, mỗi lệnh có bộ thuật toán riêng: "checker" và "fixer". Một checker báo cáo các lỗi có thể xảy ra trong code của bạn, chẳng hạn như truyền chuỗi thay vì số nguyên làm toán hạng của chuyển đổi `fmt.Printf("%d")`. Một fixer chỉnh sửa code của bạn một cách an toàn để sửa lỗi hoặc diễn đạt cùng một thứ theo cách tốt hơn, có lẽ rõ ràng hơn, ngắn gọn hơn, hoặc hiệu quả hơn. Đôi khi cùng một thuật toán xuất hiện trong cả hai bộ khi nó vừa có thể báo cáo lỗi vừa có thể sửa lỗi an toàn.

Năm 2017 chúng tôi thiết kế lại chương trình `go vet` nguyên khối lúc đó để tách các thuật toán checker (giờ được gọi là "analyzer") khỏi "driver", chương trình chạy chúng; kết quả là [framework phân tích Go](https://pkg.go.dev/golang.org/x/tools/go/analysis). Sự tách biệt này cho phép một analyzer được viết một lần rồi chạy trong nhiều loại driver khác nhau cho các môi trường khác nhau, chẳng hạn như:

- [unitchecker](https://pkg.go.dev/golang.org/x/tools/go/analysis/unitchecker), biến một bộ analyzer thành subcommand có thể được chạy bởi hệ thống build tăng dần có thể mở rộng của lệnh go, tương tự như trình biên dịch trong go build. Đây là cơ sở của `go fix` và `go vet`.
- [nogo](https://github.com/bazel-contrib/rules_go/blob/master/go/nogo.rst), driver tương đương cho các hệ thống build thay thế như Bazel và Blaze.
- [singlechecker](https://pkg.go.dev/golang.org/x/tools/go/analysis/singlechecker), biến một analyzer thành lệnh độc lập tải, phân tích và kiểm tra kiểu một tập hợp gói (có thể là toàn bộ chương trình) rồi phân tích chúng. Chúng tôi thường dùng nó cho các thí nghiệm và đo lường tùy hứng trên corpus của module mirror ([proxy.golang.org](https://proxy.golang.org/)).
- [multichecker](https://pkg.go.dev/golang.org/x/tools/go/analysis/multichecker), làm tương tự cho một bộ analyzer với giao diện CLI kiểu 'dao đa năng'.
- [gopls](/gopls), [language server](https://microsoft.github.io/language-server-protocol/) phía sau VS Code và các trình soạn thảo khác, cung cấp chẩn đoán theo thời gian thực từ các analyzer sau mỗi lần gõ phím trong trình soạn thảo.
- driver cực kỳ có thể cấu hình được sử dụng bởi công cụ [staticcheck](https://staticcheck.dev/). (Staticcheck cũng cung cấp bộ lớn các analyzer có thể chạy trong các driver khác.)
- [Tricorder](https://research.google/pubs/tricorder-building-a-program-analysis-ecosystem/), pipeline phân tích tĩnh hàng loạt được dùng bởi monorepo của Google và tích hợp với hệ thống đánh giá code của nó.
- [MCP server](/gopls/features/mcp) của gopls, cung cấp chẩn đoán cho các agent coding dựa trên LLM, cung cấp "guardrail" mạnh mẽ hơn.
- [analysistest](https://pkg.go.dev/golang.org/x/tools/go/analysis/analysistest), test harness của framework phân tích.

Một lợi ích của framework là khả năng biểu thị các analyzer helper không báo cáo chẩn đoán hoặc đề xuất sửa lỗi của riêng chúng mà thay vào đó tính toán một số cấu trúc dữ liệu trung gian có thể hữu ích cho nhiều analyzer khác, phân bổ chi phí xây dựng của nó. Các ví dụ bao gồm [đồ thị luồng điều khiển](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/ctrlflow), [biểu diễn SSA](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/buildssa) của thân hàm, và các cấu trúc dữ liệu để [điều hướng AST được tối ưu hóa](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/inspect).

Một lợi ích khác của framework là hỗ trợ cho việc suy luận qua các gói. Một analyzer có thể gắn một "[fact](https://pkg.go.dev/golang.org/x/tools/go/analysis#hdr-Modular_analysis_with_Facts)" vào một hàm hoặc ký hiệu khác để thông tin học được khi phân tích thân hàm có thể được dùng khi sau đó phân tích một lời gọi đến hàm, ngay cả khi lời gọi xuất hiện trong một gói khác hoặc phân tích sau này xảy ra trong một tiến trình khác. Điều này làm cho dễ dàng định nghĩa các phân tích liên thủ tục có thể mở rộng. Ví dụ, checker printf có thể biết khi một hàm như `log.Printf` thực sự chỉ là wrapper xung quanh `fmt.Printf`, vì vậy nó biết rằng các lời gọi đến `log.Printf` nên được kiểm tra theo cách tương tự. Quá trình này hoạt động bằng cách quy nạp, vì vậy công cụ cũng sẽ kiểm tra các lời gọi đến các wrapper tiếp theo xung quanh `log.Printf`, và v.v. Một ví dụ về analyzer sử dụng nhiều fact là [nilaway của Uber](https://github.com/uber-go/nilaway), báo cáo các lỗi tiềm năng dẫn đến dereference con trỏ nil.

<img src="gofix-analysis-facts.svg">

Quá trình "phân tích riêng biệt" trong `go fix` tương tự với quá trình biên dịch riêng biệt trong `go build`. Giống như trình biên dịch xây dựng các gói bắt đầu từ đáy của đồ thị dependency và truyền thông tin kiểu lên các gói nhập, framework phân tích làm việc từ đáy của đồ thị dependency lên, truyền các fact (và kiểu) lên các gói nhập.

Năm 2019, khi chúng tôi bắt đầu phát triển [gopls](/gopls), language server cho Go, chúng tôi đã thêm khả năng cho analyzer đề xuất một [sửa lỗi](https://pkg.go.dev/golang.org/x/tools/go/analysis#SuggestedFix) khi báo cáo chẩn đoán. Ví dụ, analyzer printf đề xuất thay thế `fmt.Printf(msg)` bằng `fmt.Printf("%s", msg)` để tránh định dạng sai nếu giá trị động `msg` chứa ký hiệu `%`. Cơ chế này đã trở thành cơ sở cho nhiều tính năng sửa lỗi nhanh và tái cấu trúc của gopls.

Trong khi tất cả những phát triển này đang xảy ra với `go vet`, `go fix` vẫn bị mắc kẹt như trước [lời hứa tương thích Go](/doc/go1compat), khi những người dùng Go đầu tiên dùng nó để duy trì code trong quá trình phát triển nhanh chóng và đôi khi không tương thích của ngôn ngữ và thư viện.

Bản phát hành Go 1.26 mang framework phân tích Go sang `go fix`. Các lệnh `go vet` và `go fix` đã hội tụ và giờ gần như giống hệt nhau về triển khai. Sự khác biệt duy nhất giữa chúng là tiêu chí cho các bộ thuật toán họ dùng, và những gì họ làm với các chẩn đoán đã tính toán. Các [analyzer go vet](https://cs.opensource.google/go/go/+/refs/tags/go1.26rc1:src/cmd/vet/main.go;l=62) phải phát hiện các lỗi có thể xảy ra với ít dương tính giả; chẩn đoán của chúng được báo cáo cho người dùng. Các [analyzer go fix](https://cs.opensource.google/go/go/+/refs/tags/go1.26rc1:src/cmd/fix/main.go;l=46) phải tạo ra các sửa lỗi an toàn để áp dụng mà không gây hồi quy về tính đúng đắn, hiệu năng, hoặc phong cách; chẩn đoán của chúng có thể không được báo cáo, nhưng các sửa lỗi được áp dụng trực tiếp. Ngoài sự khác biệt về trọng tâm này, nhiệm vụ phát triển fixer không khác gì phát triển checker.

### Cải thiện cơ sở hạ tầng phân tích

Khi số lượng analyzer trong `go vet` và `go fix` tiếp tục tăng, chúng tôi đã đầu tư vào cơ sở hạ tầng để cải thiện hiệu năng của mỗi analyzer và giúp viết mỗi analyzer mới dễ dàng hơn.

Ví dụ, hầu hết các analyzer bắt đầu bằng cách duyệt qua cây cú pháp của mỗi file trong gói để tìm một loại node cụ thể như câu lệnh range hoặc hàm literal. Gói [inspector](https://pkg.go.dev/golang.org/x/tools/go/ast/inspector) hiện có làm cho lần quét này hiệu quả bằng cách tính toán trước một chỉ mục gọn của một lần duyệt đầy đủ để các lần duyệt sau có thể nhanh chóng bỏ qua các subtree không chứa node quan tâm. Gần đây chúng tôi đã mở rộng nó với kiểu dữ liệu [Cursor](https://pkg.go.dev/golang.org/x/tools/go/ast/inspector#Cursor) để cho phép điều hướng linh hoạt và hiệu quả giữa các node theo cả bốn hướng - lên, xuống, trái và phải, tương tự như điều hướng các phần tử của HTML DOM - giúp dễ dàng và hiệu quả biểu thị một truy vấn như "tìm mỗi câu lệnh go là câu lệnh đầu tiên của thân vòng lặp":
```
	var curFile inspector.Cursor = ...

	// Tìm mỗi câu lệnh go là câu lệnh đầu tiên của thân vòng lặp.
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
Nhiều analyzer bắt đầu bằng cách tìm kiếm các lời gọi đến một hàm cụ thể, chẳng hạn như `fmt.Printf`. Các lời gọi hàm là một trong những biểu thức phổ biến nhất trong code Go, vì vậy thay vì tìm kiếm mọi biểu thức lời gọi và kiểm tra xem đó có phải là lời gọi `fmt.Printf` không, sẽ hiệu quả hơn nhiều khi tính toán trước một chỉ mục các tham chiếu ký hiệu, được thực hiện bởi [typeindex](https://pkg.go.dev/golang.org/x/tools/internal/typesinternal/typeindex) và analyzer [helper](https://pkg.go.dev/golang.org/x/tools@v0.41.0/internal/analysis/typeindex) của nó. Sau đó các lời gọi `fmt.Printf` có thể được liệt kê trực tiếp, làm cho chi phí tỷ lệ với số lượng lời gọi thay vì kích thước của gói. Đối với một analyzer như [hostport](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/hostport) tìm kiếm một ký hiệu hiếm khi dùng (`net.Dial`), điều này có thể dễ dàng làm cho nó [nhanh hơn 1.000 lần](/cl/657958).

Một số cải tiến cơ sở hạ tầng khác trong năm qua bao gồm:

- **đồ thị dependency của thư viện chuẩn** mà các analyzer có thể tham khảo để tránh đưa vào các chu kỳ nhập. Ví dụ, chúng ta không thể đưa vào lời gọi `strings.Cut` trong một gói tự nó được nhập bởi `strings`.
- hỗ trợ **truy vấn phiên bản Go hiệu quả** của một file được xác định bởi file go.mod bao quanh và các build tag, để các analyzer không chèn cách dùng các tính năng "quá mới".
- thư viện phong phú hơn của **các nguyên tố tái cấu trúc** (ví dụ "xóa câu lệnh này") xử lý đúng các comment liền kề và các trường hợp biên khó khác.

Chúng tôi đã đi một chặng đường dài, nhưng vẫn còn nhiều việc phải làm. Logic fixer có thể khó để làm đúng. Vì chúng tôi kỳ vọng người dùng áp dụng hàng trăm sửa lỗi được đề xuất với chỉ đánh giá qua loa, điều quan trọng là các fixer phải đúng ngay cả trong các trường hợp biên khó hiểu. Chỉ là một ví dụ (xem bài nói chuyện GopherCon của tôi [tại đây](https://www.youtube.com/watch?v=_VePjjjV9JU&t=13m17s) để biết thêm), chúng tôi đã xây dựng một công cụ hiện đại hóa thay thế các lời gọi như `append([]string{}, slice...)` bằng `slices.Clone(slice)` rõ ràng hơn chỉ để phát hiện ra rằng, khi `slice` rỗng, kết quả của Clone là nil, một thay đổi hành vi tinh tế trong các trường hợp hiếm có thể gây lỗi; vì vậy chúng tôi phải loại trừ [công cụ hiện đại hóa đó](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/modernize#hdr-Analyzer_appendclipped) khỏi bộ `go fix`.

Một số khó khăn này cho tác giả analyzer có thể được cải thiện với tài liệu tốt hơn (cho cả con người và LLM), đặc biệt là danh sách kiểm tra các trường hợp biên đáng ngạc nhiên để xem xét và kiểm thử. Một engine khớp pattern cho cây cú pháp, tương tự với những cái trong [staticcheck](https://pkg.go.dev/honnef.co/go/tools/pattern) và [Tree Sitter](https://tree-sitter.github.io/tree-sitter/using-parsers/queries/index.html), có thể đơn giản hóa nhiệm vụ phức tạp của việc xác định hiệu quả các vị trí cần sửa. Thư viện phong phú hơn của các toán tử để tính toán các sửa lỗi chính xác sẽ giúp tránh các lỗi phổ biến. Test harness tốt hơn sẽ cho phép chúng tôi kiểm tra rằng các sửa lỗi không phá vỡ build và bảo tồn các thuộc tính động của code mục tiêu. Tất cả những điều này nằm trong lộ trình của chúng tôi.

<a name='self-service'></a>
## Mô hình "tự phục vụ"

Căn bản hơn, chúng tôi đang hướng sự chú ý vào năm 2026 đến mô hình "tự phục vụ".

Analyzer `newexpr` chúng ta thấy trước đó là một công cụ hiện đại hóa điển hình: một thuật toán bespoke được điều chỉnh cho một tính năng cụ thể. Mô hình bespoke hoạt động tốt cho các tính năng của ngôn ngữ và thư viện chuẩn, nhưng nó thực sự không giúp ích cho việc cập nhật cách dùng các gói bên thứ ba. Mặc dù không có gì ngăn bạn viết công cụ hiện đại hóa cho các API công khai của riêng bạn và chạy nó trên dự án của riêng bạn, không có cách tự động nào để yêu cầu người dùng API của bạn chạy nó cũng vậy. Công cụ hiện đại hóa của bạn có thể không thuộc về gopls hay bộ `go vet` trừ khi API của bạn được sử dụng rộng rãi đặc biệt trên hệ sinh thái Go. Ngay cả trong trường hợp đó, bạn sẽ phải nhận được các đánh giá code và phê duyệt rồi chờ bản phát hành tiếp theo.

Theo mô hình tự phục vụ, các lập trình viên Go sẽ có thể định nghĩa các hiện đại hóa cho các API của riêng họ mà người dùng của họ có thể áp dụng mà không có tất cả các nút cổ chai của mô hình tập trung hiện tại. Điều này đặc biệt quan trọng khi cộng đồng Go và corpus Go toàn cầu đang tăng trưởng nhanh hơn nhiều so với khả năng của nhóm chúng tôi để đánh giá các đóng góp analyzer.

Lệnh `go fix` trong Go 1.26 bao gồm bản xem trước về trái cây đầu tiên của mô hình mới này: **inline người được chú thích ở cấp độ nguồn**, được mô tả trong [bài đăng tiếp theo](inliner). Trong năm tới, chúng tôi có kế hoạch điều tra thêm hai cách tiếp cận trong mô hình này.

Thứ nhất, chúng tôi sẽ khám phá khả năng [tải động](/issue/59869) các công cụ hiện đại hóa từ cây nguồn và thực thi chúng một cách an toàn, hoặc trong gopls hoặc `go fix`. Trong cách tiếp cận này, một gói cung cấp API cho, ví dụ, cơ sở dữ liệu SQL, có thể bổ sung cung cấp checker cho các cách dùng sai API, chẳng hạn như lỗ hổng SQL injection hoặc không xử lý các lỗi quan trọng. Cơ chế tương tự có thể được dùng bởi người duy trì dự án để mã hóa các quy tắc housekeeping nội bộ, chẳng hạn như tránh gọi một số hàm có vấn đề hoặc thực thi các kỷ luật coding mạnh mẽ hơn trong các phần quan trọng của code.

Thứ hai, nhiều checker hiện có có thể được mô tả không chính thức là "đừng quên X sau khi bạn Y!", chẳng hạn như "đóng file sau khi bạn mở nó", "hủy context sau khi bạn tạo nó", "mở khóa mutex sau khi bạn khóa nó", "thoát khỏi vòng lặp iterator sau khi yield trả về false", và v.v. Những gì các checker như vậy có điểm chung là họ thực thi các bất biến nhất định trên tất cả các đường thực thi. Chúng tôi có kế hoạch khám phá các tổng quát hóa và thống nhất của các checker luồng điều khiển này để lập trình viên Go có thể dễ dàng áp dụng chúng cho các lĩnh vực mới, không cần logic phân tích phức tạp, đơn giản bằng cách chú thích code của riêng họ.

Chúng tôi hy vọng rằng các công cụ mới này sẽ giúp bạn tiết kiệm công sức trong quá trình bảo trì các dự án Go của bạn và giúp bạn tìm hiểu về và hưởng lợi từ các tính năng mới hơn sớm hơn. Hãy thử `go fix` trên các dự án của bạn và [báo cáo](/issue/new) bất kỳ vấn đề nào bạn tìm thấy, và hãy chia sẻ bất kỳ ý tưởng nào bạn có về các công cụ hiện đại hóa, fixer, checker mới, hoặc các cách tiếp cận tự phục vụ để phân tích tĩnh.
