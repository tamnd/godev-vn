---
title: "Godoc: tài liệu hóa code Go"
date: 2011-03-31
by:
- Andrew Gerrand
tags:
- godoc
- technical
summary: Cách và lý do để tài liệu hóa các gói Go của bạn.
---

[_**Lưu ý, tháng 6 năm 2022**: Để biết hướng dẫn cập nhật về tài liệu hóa code Go,
xem "[Go Doc Comments](/doc/comment)."_]

Dự án Go coi tài liệu là vấn đề nghiêm túc.
Tài liệu là một phần quan trọng để làm cho phần mềm dễ tiếp cận và có thể bảo trì.
Tất nhiên nó phải được viết tốt và chính xác,
nhưng nó cũng phải dễ viết và dễ bảo trì.
Lý tưởng nhất, nó nên được kết hợp với bản thân code để tài liệu
phát triển cùng với code.
Càng dễ dàng để lập trình viên tạo ra tài liệu tốt,
càng tốt cho mọi người.

Để đạt mục tiêu đó, chúng tôi đã phát triển công cụ tài liệu [godoc](/cmd/godoc/).
Bài viết này mô tả cách tiếp cận của godoc đối với tài liệu,
và giải thích cách bạn có thể dùng các quy ước và công cụ của chúng tôi để viết tài liệu tốt
cho các dự án của riêng bạn.

Godoc phân tích mã nguồn Go - bao gồm cả comment - và tạo ra tài liệu
dưới dạng HTML hoặc văn bản thuần túy.
Kết quả cuối cùng là tài liệu được kết hợp chặt chẽ với code mà nó tài liệu hóa.
Ví dụ, thông qua giao diện web của godoc bạn có thể điều hướng từ
[tài liệu](/pkg/strings/#HasPrefix) của một hàm đến [triển khai](/src/strings/strings.go?s=11163:11200#L434) của nó chỉ một click.

Godoc có liên quan về mặt khái niệm với [Docstring](https://www.python.org/dev/peps/pep-0257/) của Python
và [Javadoc](https://www.oracle.com/java/technologies/javase/javadoc-tool.html) của Java
nhưng thiết kế của nó đơn giản hơn.
Các comment được godoc đọc không phải là cấu trúc ngôn ngữ (như với Docstring)
cũng không phải có cú pháp có thể đọc được bởi máy của riêng chúng (như với Javadoc).
Comment Godoc chỉ là các comment tốt, loại bạn muốn đọc ngay cả khi
godoc không tồn tại.

Quy ước đơn giản: để tài liệu hóa một kiểu,
biến, hằng số, hàm, hoặc thậm chí một gói,
viết một comment thông thường ngay trước khai báo của nó,
không có dòng trống chen vào.
Godoc sau đó sẽ trình bày comment đó như văn bản cùng với mục nó tài liệu hóa.
Ví dụ, đây là tài liệu cho hàm [`Fprint`](/pkg/fmt/#Fprint) của gói `fmt`:

	// Fprint formats using the default formats for its operands and writes to w.
	// Spaces are added between operands when neither is a string.
	// It returns the number of bytes written and any write error encountered.
	func Fprint(w io.Writer, a ...interface{}) (n int, err error) {

Lưu ý comment này là một câu hoàn chỉnh bắt đầu bằng tên của
phần tử nó mô tả.
Quy ước quan trọng này cho phép chúng tôi tạo tài liệu ở nhiều định dạng,
từ văn bản thuần túy đến HTML đến trang man UNIX,
và làm cho nó đọc tốt hơn khi các công cụ cắt ngắn nó để ngắn gọn,
chẳng hạn như khi chúng trích xuất dòng hoặc câu đầu tiên.

Các comment trên khai báo gói nên cung cấp tài liệu gói chung.
Các comment này có thể ngắn, như mô tả ngắn gọn của gói [`sort`](/pkg/sort/):

	// Package sort provides primitives for sorting slices and user-defined
	// collections.
	package sort

Chúng cũng có thể chi tiết như phần tổng quan của [gói gob](/pkg/encoding/gob/).
Gói đó dùng quy ước khác cho các gói cần lượng lớn
tài liệu giới thiệu:
comment gói được đặt trong file riêng của nó,
[doc.go](/src/pkg/encoding/gob/doc.go),
chỉ chứa những comment đó và một mệnh đề gói.

Khi viết comment gói ở bất kỳ kích thước nào,
hãy nhớ rằng [câu đầu tiên](/pkg/go/doc/#Package.Synopsis) của nó
sẽ xuất hiện trong [danh sách gói](/pkg/) của godoc.

Các comment không liền kề với khai báo cấp cao nhất bị bỏ qua khỏi đầu ra của godoc,
với một ngoại lệ đáng chú ý.
Các comment cấp cao nhất bắt đầu bằng từ `"BUG(who)"` được nhận ra là các bug đã biết,
và được đưa vào phần "Bugs" của tài liệu gói.
Phần "who" phải là tên người dùng của ai đó có thể cung cấp thêm thông tin.
Ví dụ, đây là vấn đề đã biết từ [gói bytes](/pkg/bytes/#pkg-note-BUG):

	// BUG(r): The rule Title uses for word boundaries does not handle Unicode punctuation properly.

Đôi khi một trường struct, hàm, kiểu, hoặc thậm chí toàn bộ gói trở nên
dư thừa hoặc không cần thiết, nhưng phải được giữ lại để tương thích với các chương trình hiện có.
Để báo hiệu rằng một định danh không nên được dùng, thêm một đoạn vào comment tài liệu của nó
bắt đầu bằng "Deprecated:" theo sau là một số thông tin về việc deprecation.

Có một vài quy tắc định dạng mà Godoc dùng khi chuyển đổi comment thành HTML:

  - Các dòng văn bản tiếp theo được coi là một phần của cùng một đoạn;
    bạn phải để một dòng trống để phân cách các đoạn.

  - Văn bản được định dạng sẵn phải được thụt lề so với văn bản comment xung quanh
    (xem [doc.go](/src/pkg/encoding/gob/doc.go) của gob để có ví dụ).

  - URL sẽ được chuyển đổi thành liên kết HTML; không cần đánh dấu đặc biệt.

Lưu ý rằng không có quy tắc nào trong số này yêu cầu bạn làm bất cứ điều gì khác thường.

Trên thực tế, điều tốt nhất về cách tiếp cận tối giản của godoc là cách dễ dàng để sử dụng.
Kết quả là, rất nhiều code Go, bao gồm toàn bộ thư viện chuẩn,
đã tuân theo các quy ước.

Code của riêng bạn có thể trình bày tài liệu tốt chỉ bằng cách có comment như mô tả ở trên.
Bất kỳ gói Go nào được cài đặt bên trong `$GOROOT/src/pkg` và bất kỳ không gian làm việc `GOPATH` nào
sẽ đã có thể truy cập qua giao diện dòng lệnh và HTTP của godoc,
và bạn có thể chỉ định các đường dẫn bổ sung để lập chỉ mục thông qua cờ `-path` hoặc
chỉ bằng cách chạy `"godoc ."` trong thư mục nguồn.
Xem [tài liệu godoc](/cmd/godoc/) để biết thêm chi tiết.
