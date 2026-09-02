---
title: Giới thiệu Gofix
date: 2011-04-15
by:
- Russ Cox
tags:
- gofix
- technical
summary: Cách dùng go fix để cập nhật mã nguồn của bạn với mỗi bản phát hành Go mới.
---


Bản phát hành Go tiếp theo sẽ bao gồm những thay đổi API đáng kể trong một số gói Go cơ bản.
Mã [triển khai HTTP server handler](http://codereview.appspot.com/4239076),
[gọi `net.Dial`](http://codereview.appspot.com/4244055),
[gọi `os.Open`](http://codereview.appspot.com/4357052),
hoặc [sử dụng gói reflect](http://codereview.appspot.com/4281055) sẽ
không thể build được trừ khi được cập nhật để dùng các API mới.
Giờ đây khi các bản phát hành của chúng tôi đã [ổn định hơn và ít thường xuyên hơn](/blog/go-becomes-more-stable),
đây sẽ là tình huống phổ biến.
Mỗi thay đổi API này xảy ra trong một snapshot hàng tuần khác nhau và có thể quản lý được khi đứng riêng lẻ; nhưng kết hợp lại, chúng đại diện cho một lượng công sức thủ công đáng kể để cập nhật mã hiện có.

[Gofix](/cmd/fix/) là một công cụ mới giúp giảm công sức cần thiết để cập nhật mã hiện có.
Nó đọc một chương trình từ tệp mã nguồn, tìm kiếm các cách dùng API cũ, viết lại chúng để dùng API hiện tại, và ghi chương trình trở lại tệp.
Không phải tất cả các thay đổi API đều giữ nguyên toàn bộ chức năng của API cũ, vì vậy gofix không thể luôn luôn làm việc hoàn hảo.
Khi gofix không thể viết lại cách dùng của một API cũ, nó in ra cảnh báo cho biết tên tệp và số dòng của cách dùng đó, để nhà phát triển có thể kiểm tra và viết lại mã.
Gofix xử lý các thay đổi dễ dàng, lặp đi lặp lại và tẻ nhạt, để nhà phát triển có thể tập trung vào những thay đổi thực sự cần được chú ý.

Mỗi khi chúng tôi thực hiện một thay đổi API đáng kể, chúng tôi sẽ thêm mã vào gofix để xử lý việc chuyển đổi, ở mức độ có thể thực hiện tự động.
Khi bạn cập nhật lên bản phát hành Go mới và mã của bạn không còn build được nữa, chỉ cần chạy gofix trên thư mục mã nguồn của bạn.

Bạn có thể mở rộng gofix để hỗ trợ các thay đổi với API của riêng bạn.
Chương trình gofix là một driver đơn giản xung quanh các plugin gọi là fix, mỗi cái xử lý một thay đổi API cụ thể.
Hiện tại, việc viết một fix mới đòi hỏi phải thực hiện một số quét và viết lại cây cú pháp go/ast, thường tỷ lệ thuận với độ phức tạp của các thay đổi API.
Nếu bạn muốn khám phá, các ví dụ minh họa là [`netdialFix`](https://go.googlesource.com/go/+/go1/src/cmd/fix/netdial.go),
[`osopenFix`](https://go.googlesource.com/go/+/go1/src/cmd/fix/osopen.go),
[`httpserverFix`](https://go.googlesource.com/go/+/go1/src/cmd/fix/httpserver.go),
và [`reflectFix`](https://go.googlesource.com/go/+/go1/src/cmd/fix/reflect.go),
theo thứ tự tăng dần về độ phức tạp.

Chúng tôi cũng viết mã Go, và mã của chúng tôi cũng bị ảnh hưởng bởi các thay đổi API này như mã của bạn.
Thông thường, chúng tôi viết hỗ trợ gofix cùng lúc với thay đổi API rồi dùng gofix để viết lại các cách dùng trong cây mã nguồn chính.
Chúng tôi dùng gofix để cập nhật các codebase Go khác và các dự án cá nhân.
Chúng tôi thậm chí dùng gofix để cập nhật cây mã nguồn nội bộ của Google khi đến lúc build với bản phát hành Go mới.

Ví dụ, gofix có thể viết lại mã như [đoạn code này từ `fmt/print.go`](http://codereview.appspot.com/4353043/diff/10001/src/pkg/fmt/print.go#newcode657):

	switch f := value.(type) {
	case *reflect.BoolValue:
	    p.fmtBool(f.Get(), verb, field)
	case *reflect.IntValue:
	    p.fmtInt64(f.Get(), verb, field)
	// ...
	case reflect.ArrayOrSliceValue:
	    // Byte slices are special.
	    if f.Type().(reflect.ArrayOrSliceType).Elem().Kind() == reflect.Uint8 {
	        // ...
	    }
	// ...
	}

để thích nghi với reflect API mới:

	switch f := value; f.Kind() {
	case reflect.Bool:
	    p.fmtBool(f.Bool(), verb, field)
	case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
	    p.fmtInt64(f.Int(), verb, field)
	// ...
	case reflect.Array, reflect.Slice:
	    // Byte slices are special.
	    if f.Type().Elem().Kind() == reflect.Uint8 {
	        // ...
	    }
	// ...
	}

Gần như mọi dòng ở trên đã thay đổi theo một cách nào đó.
Các thay đổi trong quá trình viết lại rất nhiều nhưng gần như hoàn toàn mang tính cơ học, đúng kiểu việc mà máy tính rất giỏi thực hiện.

Gofix khả thi vì Go có sự hỗ trợ trong các thư viện chuẩn để [phân tích tệp mã nguồn Go thành cây cú pháp](/pkg/go/parser) và cũng để [in các cây cú pháp đó trở lại mã nguồn Go](/pkg/go/printer).
Điều quan trọng là thư viện in Go in chương trình theo định dạng chính thức (thường được thực thi thông qua công cụ gofmt), cho phép gofix thực hiện các thay đổi cơ học với các chương trình Go mà không gây ra các thay đổi định dạng không cần thiết.
Thực tế, một trong những động lực chính để tạo ra gofmt, có lẽ chỉ đứng sau việc tránh tranh luận về vị trí dấu ngoặc nhọn, là để đơn giản hóa việc tạo ra các công cụ viết lại chương trình Go, như gofix đang làm.

Gofix đã tự chứng minh là không thể thiếu.
Đặc biệt, các thay đổi reflect gần đây sẽ không thể chấp nhận được nếu không có chuyển đổi tự động, và reflect API thực sự cần được làm lại.
Gofix cho chúng tôi khả năng sửa các sai lầm hoặc suy nghĩ lại hoàn toàn về các API gói mà không lo lắng về chi phí chuyển đổi mã hiện có.
Chúng tôi hy vọng bạn thấy gofix hữu ích và tiện lợi như chúng tôi đã thấy.
