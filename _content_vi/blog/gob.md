---
title: Dữ liệu theo kiểu Gob
date: 2011-03-24
by:
- Rob Pike
tags:
- gob
- json
- protobuf
- xml
- technical
summary: Giới thiệu gob, định dạng mã hóa truyền dữ liệu Go-to-Go tốc độ cao.
---

## Giới thiệu

Để truyền một cấu trúc dữ liệu qua mạng hoặc lưu trữ nó trong file,
nó phải được mã hóa và sau đó giải mã trở lại.
Có nhiều cách mã hóa sẵn có, tất nhiên:
[JSON](http://www.json.org/), [XML](http://www.w3.org/XML/),
[protocol buffers](http://code.google.com/p/protobuf) của Google, và nhiều hơn nữa.
Và giờ có thêm một cái khác, được cung cấp bởi gói [gob](/pkg/encoding/gob/) của Go.

Tại sao định nghĩa một cách mã hóa mới? Đó là nhiều công việc và dư thừa.
Tại sao không chỉ dùng một trong các định dạng hiện có? Chà,
một điều là chúng tôi có làm vậy!
Go có [các gói](/pkg/) hỗ trợ tất cả các cách mã hóa
vừa đề cập ([gói protocol buffer](https://github.com/golang/protobuf)
nằm trong kho lưu trữ riêng nhưng là một trong những gói được tải xuống thường xuyên nhất).
Và cho nhiều mục đích, bao gồm giao tiếp với các công cụ và hệ thống được viết bằng các ngôn ngữ khác,
chúng là lựa chọn phù hợp.

Nhưng đối với môi trường dành riêng cho Go, chẳng hạn như giao tiếp giữa hai server được viết bằng Go,
có cơ hội xây dựng thứ gì đó dễ sử dụng hơn nhiều và có thể hiệu quả hơn.

Gob hoạt động với ngôn ngữ theo cách mà một cách mã hóa được định nghĩa bên ngoài,
độc lập ngôn ngữ không thể.
Đồng thời, có những bài học cần học từ các hệ thống hiện có.

## Mục tiêu

Gói gob được thiết kế với một số mục tiêu trong đầu.

Đầu tiên, và hiển nhiên nhất, nó phải rất dễ sử dụng.
Thứ nhất, vì Go có reflection, không cần ngôn ngữ định nghĩa interface riêng
hoặc "trình biên dịch giao thức".
Cấu trúc dữ liệu tự nó là tất cả những gì gói cần để tìm ra cách
mã hóa và giải mã nó.
Mặt khác, cách tiếp cận này có nghĩa là gob sẽ không bao giờ hoạt động tốt bằng
với các ngôn ngữ khác, nhưng điều đó ổn:
gob không ngại ngùng là Go-centric.

Hiệu quả cũng quan trọng. Biểu diễn dạng văn bản,
điển hình bởi XML và JSON, quá chậm để đặt ở trung tâm của một
mạng truyền thông hiệu quả.
Mã hóa nhị phân là cần thiết.

Các luồng gob phải tự mô tả. Mỗi luồng gob,
đọc từ đầu, chứa đủ thông tin để toàn bộ
luồng có thể được phân tích bởi một agent không biết gì trước về nội dung của nó.
Thuộc tính này có nghĩa là bạn sẽ luôn có thể giải mã luồng gob được lưu trữ trong file,
ngay cả lâu sau khi bạn quên dữ liệu đó đại diện cho gì.

Cũng có một số điều cần học từ kinh nghiệm của chúng tôi với protocol buffers của Google.

## Điểm yếu của protocol buffer

Protocol buffers có tác động lớn đến thiết kế của gob,
nhưng có ba tính năng đã được cố tình tránh.
(Bỏ qua thuộc tính rằng protocol buffers không tự mô tả:
nếu bạn không biết định nghĩa dữ liệu được dùng để mã hóa protocol buffer,
bạn có thể không thể phân tích nó.)

Thứ nhất, protocol buffers chỉ hoạt động trên kiểu dữ liệu mà chúng tôi gọi là struct trong Go.
Bạn không thể mã hóa một số nguyên hoặc mảng ở cấp độ cao nhất,
chỉ một struct với các trường bên trong nó.
Điều đó có vẻ là hạn chế vô nghĩa, ít nhất là trong Go.
Nếu tất cả những gì bạn muốn gửi là một mảng số nguyên,
tại sao bạn phải đặt nó vào struct trước?

Tiếp theo, định nghĩa protocol buffer có thể chỉ định rằng các trường `T.x` và `T.y`
bắt buộc phải có bất cứ khi nào một giá trị của kiểu `T` được mã hóa hoặc giải mã.
Mặc dù các trường bắt buộc như vậy có vẻ là ý tưởng hay,
chúng tốn kém để triển khai vì codec phải duy trì cấu trúc
dữ liệu riêng trong khi mã hóa và giải mã,
để có thể báo cáo khi thiếu các trường bắt buộc.
Chúng cũng là vấn đề bảo trì. Theo thời gian,
người ta có thể muốn sửa đổi định nghĩa dữ liệu để xóa một trường bắt buộc,
nhưng điều đó có thể khiến các client hiện có của dữ liệu bị crash.
Tốt hơn là không có chúng trong mã hóa.
(Protocol buffers cũng có các trường tùy chọn.
Nhưng nếu chúng ta không có trường bắt buộc, tất cả các trường đều tùy chọn và như vậy thôi.
Sẽ có thêm để nói về các trường tùy chọn một chút sau.)

Điểm yếu thứ ba của protocol buffer là các giá trị mặc định.
Nếu protocol buffer bỏ qua giá trị cho trường "có mặc định",
thì cấu trúc được giải mã hoạt động như thể trường được đặt thành giá trị đó.
Ý tưởng này hoạt động tốt khi bạn có các phương thức getter và setter để kiểm soát
quyền truy cập vào trường,
nhưng khó xử lý sạch sẽ hơn khi container chỉ là một struct thông thường theo phong cách Go.
Các trường bắt buộc cũng khó triển khai:
người ta định nghĩa các giá trị mặc định ở đâu,
chúng có kiểu gì (văn bản có phải UTF-8 không? byte không được giải thích? bao nhiêu bit
trong một float?) và mặc dù có vẻ đơn giản,
có một số phức tạp trong thiết kế và triển khai của chúng
cho protocol buffers.
Chúng tôi quyết định bỏ chúng khỏi gob và quay lại quy tắc mặc định đơn giản nhưng hiệu quả của Go:
trừ khi bạn đặt thứ gì đó khác, nó có "giá trị zero" cho kiểu đó -
và nó không cần phải truyền.

Vậy gob cuối cùng trông giống như một loại protocol buffer được tổng quát hóa, đơn giản hóa. Chúng hoạt động như thế nào?

## Giá trị

Dữ liệu gob được mã hóa không phải về các kiểu như `int8` và `uint16`.
Thay vào đó, phần nào tương tự với các hằng số trong Go,
các giá trị nguyên của nó là các số trừu tượng, không có kích thước,
có dấu hoặc không dấu.
Khi bạn mã hóa một `int8`, giá trị của nó được truyền như một
số nguyên có độ dài biến không có kích thước.
Khi bạn mã hóa một `int64`, giá trị của nó cũng được truyền như một
số nguyên có độ dài biến không có kích thước.
(Có dấu và không dấu được xử lý khác nhau,
nhưng tương tự không có kích thước cũng áp dụng cho giá trị không dấu.) Nếu cả hai có giá trị 7,
các bit được gửi trên mạng sẽ giống hệt nhau.
Khi bộ nhận giải mã giá trị đó, nó đặt nó vào biến của bộ nhận,
có thể có kiểu nguyên tùy ý.
Vì vậy bộ mã hóa có thể gửi 7 đến từ `int8`,
nhưng bộ nhận có thể lưu nó trong `int64`.
Điều này ổn: giá trị là số nguyên và miễn là nó vừa, mọi thứ hoạt động.
(Nếu không vừa, lỗi xảy ra.) Sự tách rời khỏi kích thước của
biến cho mã hóa một số linh hoạt:
chúng ta có thể mở rộng kiểu của biến nguyên khi phần mềm phát triển,
nhưng vẫn có thể giải mã dữ liệu cũ.

Sự linh hoạt này cũng áp dụng cho con trỏ.
Trước khi truyền, tất cả con trỏ được làm phẳng.
Các giá trị của kiểu `int8`, `*int8`, `**int8`,
`****int8`, v.v. đều được truyền như một giá trị nguyên,
sau đó có thể được lưu trong `int` của bất kỳ kích thước nào,
hoặc `*int`, hoặc `******int`, v.v.
Một lần nữa, điều này cho phép sự linh hoạt.

Sự linh hoạt cũng xảy ra vì, khi giải mã một struct,
chỉ những trường được gửi bởi bộ mã hóa mới được lưu trữ trong đích. Cho giá trị

	type T struct{ X, Y, Z int } // Chỉ các trường xuất được mã hóa và giải mã.
	var t = T{X: 7, Y: 0, Z: 8}

việc mã hóa `t` chỉ gửi 7 và 8.
Vì nó là zero, giá trị của `Y` thậm chí không được gửi;
không cần gửi giá trị zero.

Thay vào đó bộ nhận có thể giải mã giá trị vào cấu trúc này:

	type U struct{ X, Y *int8 } // Lưu ý: con trỏ đến int8
	var u U

và có giá trị của `u` chỉ với `X` được đặt (đến địa chỉ của biến `int8` được đặt thành 7);
trường `Z` bị bỏ qua - bạn sẽ đặt nó ở đâu? Khi giải mã struct,
các trường được khớp theo tên và kiểu tương thích,
và chỉ các trường tồn tại trong cả hai mới bị ảnh hưởng.
Cách tiếp cận đơn giản này xử lý khéo léo vấn đề "trường tùy chọn":
khi kiểu `T` phát triển bằng cách thêm các trường,
các bộ nhận lỗi thời vẫn sẽ hoạt động với phần kiểu mà chúng nhận ra.
Vì vậy gob cung cấp kết quả quan trọng của các trường tùy chọn - khả năng mở rộng -
mà không có bất kỳ cơ chế hay ký hiệu bổ sung nào.

Từ số nguyên chúng ta có thể xây dựng tất cả các kiểu khác:
byte, chuỗi, mảng, slice, map, thậm chí float.
Các giá trị floating-point được biểu diễn bởi mẫu bit dấu phẩy động IEEE 754 của chúng,
được lưu trữ như một số nguyên, hoạt động tốt miễn là bạn biết kiểu của chúng, điều mà chúng ta luôn biết.
Nhân tiện, số nguyên đó được gửi theo thứ tự byte đảo ngược vì các giá trị phổ biến
của số dấu phẩy động,
chẳng hạn như số nguyên nhỏ, có nhiều số không ở cuối thấp mà chúng ta có thể tránh truyền.

Một tính năng hay của gob mà Go làm có thể là chúng cho phép bạn
định nghĩa cách mã hóa của riêng bạn bằng cách có kiểu của bạn thỏa mãn các interface [GobEncoder](/pkg/encoding/gob/#GobEncoder)
và [GobDecoder](/pkg/encoding/gob/#GobDecoder),
theo cách tương tự với interface [Marshaler](/pkg/encoding/json/#Marshaler)
và [Unmarshaler](/pkg/encoding/json/#Unmarshaler) của gói
[JSON](/pkg/encoding/json/)
và cũng với interface [Stringer](/pkg/fmt/#Stringer)
từ [gói fmt](/pkg/fmt/).
Cơ sở này làm cho có thể biểu diễn các tính năng đặc biệt,
thực thi các ràng buộc, hoặc ẩn bí mật khi bạn truyền dữ liệu.
Xem [tài liệu](/pkg/encoding/gob/) để biết chi tiết.

## Kiểu trên đường truyền

Lần đầu tiên bạn gửi một kiểu nhất định, gói gob bao gồm trong luồng dữ liệu
một mô tả về kiểu đó.
Trên thực tế, điều xảy ra là bộ mã hóa được dùng để mã hóa,
theo định dạng mã hóa gob tiêu chuẩn, một struct nội bộ mô tả
kiểu và cung cấp cho nó một số duy nhất.
(Các kiểu cơ bản, cộng với bố cục của cấu trúc mô tả kiểu,
được định nghĩa trước bởi phần mềm để khởi động.) Sau khi kiểu được mô tả,
nó có thể được tham chiếu bởi số kiểu của nó.

Vì vậy khi chúng ta gửi kiểu đầu tiên `T`, bộ mã hóa gob gửi mô tả
của `T` và gắn thẻ cho nó với một số kiểu, ví dụ 127.
Tất cả các giá trị, bao gồm cả giá trị đầu tiên, sau đó được đặt tiền tố bởi số đó,
vì vậy một luồng giá trị `T` trông như sau:

	("define type id" 127, definition of type T)(127, T value)(127, T value), ...

Các số kiểu này làm cho có thể mô tả các kiểu đệ quy và gửi
các giá trị của các kiểu đó.
Vì vậy gob có thể mã hóa các kiểu như cây:

	type Node struct {
	    Value       int
	    Left, Right *Node
	}

(Để độc giả tự khám phá cách quy tắc zero-defaulting làm điều này hoạt động,
ngay cả khi gob không biểu diễn con trỏ.)

Với thông tin kiểu, luồng gob là hoàn toàn tự mô tả ngoại trừ
tập hợp các kiểu bootstrap,
là điểm bắt đầu được định nghĩa rõ ràng.

## Biên dịch một máy

Lần đầu tiên bạn mã hóa một giá trị của một kiểu nhất định,
gói gob xây dựng một máy được thông dịch nhỏ dành riêng cho kiểu dữ liệu đó.
Nó dùng reflection trên kiểu để xây dựng máy đó,
nhưng một khi máy được xây dựng nó không phụ thuộc vào reflection.
Máy dùng gói unsafe và một số mẹo để chuyển đổi dữ liệu thành
byte được mã hóa với tốc độ cao.
Nó có thể dùng reflection và tránh unsafe,
nhưng sẽ chậm hơn đáng kể.
(Cách tiếp cận tốc độ cao tương tự được áp dụng bởi hỗ trợ protocol buffer cho Go,
có thiết kế bị ảnh hưởng bởi việc triển khai gob.) Các giá trị tiếp theo
của cùng kiểu dùng máy đã được biên dịch,
vì vậy chúng có thể được mã hóa ngay lập tức.

[Cập nhật: Kể từ Go 1.4, gói unsafe không còn được dùng bởi gói gob, với mức giảm hiệu năng nhỏ.]

Giải mã tương tự nhưng khó hơn. Khi bạn giải mã một giá trị,
gói gob giữ một byte slice đại diện cho một giá trị của một kiểu được định nghĩa bởi bộ mã hóa để giải mã,
cộng với giá trị Go để giải mã vào đó.
Gói gob xây dựng máy cho cặp đó:
kiểu gob được gửi trên đường truyền giao với kiểu Go được cung cấp để giải mã.
Một khi máy giải mã đó được xây dựng,
nó lại là engine không dùng reflection sử dụng các phương thức unsafe để đạt tốc độ tối đa.

## Sử dụng

Có rất nhiều thứ xảy ra dưới nắp capo, nhưng kết quả là một
hệ thống mã hóa hiệu quả, dễ sử dụng để truyền dữ liệu.
Đây là một ví dụ hoàn chỉnh cho thấy các kiểu mã hóa và giải mã khác nhau.
Lưu ý cách dễ dàng để gửi và nhận giá trị;
tất cả những gì bạn cần làm là trình bày các giá trị và biến cho [gói gob](/pkg/encoding/gob/)
và nó thực hiện tất cả công việc.

	package main

	import (
	    "bytes"
	    "encoding/gob"
	    "fmt"
	    "log"
	)

	type P struct {
	    X, Y, Z int
	    Name    string
	}

	type Q struct {
	    X, Y *int32
	    Name string
	}

	func main() {
	    // Khởi tạo bộ mã hóa và giải mã. Thông thường enc và dec sẽ được
	    // ràng buộc với các kết nối mạng và bộ mã hóa và giải mã sẽ
	    // chạy trong các tiến trình khác nhau.
	    var network bytes.Buffer        // Đại diện cho kết nối mạng
	    enc := gob.NewEncoder(&network) // Sẽ ghi vào network.
	    dec := gob.NewDecoder(&network) // Sẽ đọc từ network.
	    // Mã hóa (gửi) giá trị.
	    err := enc.Encode(P{3, 4, 5, "Pythagoras"})
	    if err != nil {
	        log.Fatal("encode error:", err)
	    }
	    // Giải mã (nhận) giá trị.
	    var q Q
	    err = dec.Decode(&q)
	    if err != nil {
	        log.Fatal("decode error:", err)
	    }
	    fmt.Printf("%q: {%d,%d}\n", q.Name, *q.X, *q.Y)
	}

Bạn có thể biên dịch và chạy code ví dụ này trên [Go Playground](/play/p/_-OJV-rwMq).

[Gói rpc](/pkg/net/rpc/) xây dựng trên gob để biến
tự động hóa mã hóa/giải mã này thành phương tiện truyền tải cho các lời gọi phương thức qua mạng.
Đó là chủ đề của một bài viết khác.

## Chi tiết

[Tài liệu gói gob](/pkg/encoding/gob/),
đặc biệt là file [doc.go](/src/pkg/encoding/gob/doc.go),
mở rộng nhiều chi tiết được mô tả ở đây và bao gồm một ví dụ làm việc đầy đủ
cho thấy cách mã hóa biểu diễn dữ liệu.
Nếu bạn quan tâm đến phần bên trong của việc triển khai gob,
đó là nơi tốt để bắt đầu.
