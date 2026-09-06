---
title: "Mẫu hình đồng thời trong Go: Hết thời gian chờ, tiếp tục"
date: 2010-09-23
by:
- Andrew Gerrand
tags:
- concurrency
- technical
summary: Cách triển khai thời gian chờ bằng cách sử dụng hỗ trợ đồng thời của Go.
template: true
---


Lập trình đồng thời có những thành ngữ riêng.
Một ví dụ điển hình là timeout. Mặc dù channel của Go không hỗ trợ chúng trực tiếp,
chúng rất dễ triển khai.
Giả sử chúng ta muốn nhận từ channel `ch`,
nhưng muốn chờ tối đa một giây để giá trị đến.
Chúng ta sẽ bắt đầu bằng cách tạo một channel báo hiệu và khởi chạy một goroutine
ngủ trước khi gửi trên channel:

{{raw `
	timeout := make(chan bool, 1)
	go func() {
	    time.Sleep(1 * time.Second)
	    timeout <- true
	}()
`}}

Sau đó chúng ta có thể dùng câu lệnh `select` để nhận từ `ch` hoặc `timeout`.
Nếu không có gì đến trên `ch` sau một giây,
trường hợp timeout sẽ được chọn và nỗ lực đọc từ ch bị bỏ qua.

{{raw `
	select {
	case <-ch:
	    // một lần đọc từ ch đã xảy ra
	case <-timeout:
	    // lần đọc từ ch đã hết thời gian chờ
	}
`}}

Channel `timeout` được đệm với chỗ chứa 1 giá trị,
cho phép goroutine timeout gửi đến channel rồi thoát.
Goroutine không biết (hoặc không quan tâm) liệu giá trị có được nhận hay không.
Điều này có nghĩa là goroutine sẽ không bị treo mãi nếu việc nhận từ `ch` xảy ra
trước khi đạt đến thời điểm timeout.
Channel `timeout` cuối cùng sẽ được giải phóng bởi bộ gom rác.

(Trong ví dụ này chúng ta dùng `time.Sleep` để minh họa cơ chế của goroutine và channel.
Trong các chương trình thực tế bạn nên dùng [`time.After`](/pkg/time/#After),
một hàm trả về một channel và gửi trên channel đó sau khoảng thời gian được chỉ định.)

Hãy xem một biến thể khác của mẫu này.
Trong ví dụ này chúng ta có một chương trình đọc đồng thời từ nhiều cơ sở dữ liệu được sao chép.
Chương trình chỉ cần một trong các câu trả lời,
và nó nên nhận câu trả lời đến đầu tiên.

Hàm `Query` nhận một slice các kết nối cơ sở dữ liệu và một chuỗi `query`.
Nó truy vấn từng cơ sở dữ liệu song song và trả về phản hồi đầu tiên mà nó nhận được:

{{raw `
	func Query(conns []Conn, query string) Result {
	    ch := make(chan Result)
	    for _, conn := range conns {
	        go func(c Conn) {
	            select {
	            case ch <- c.DoQuery(query):
	            default:
	            }
	        }(conn)
	    }
	    return <-ch
	}
`}}

Trong ví dụ này, closure thực hiện một lần gửi không chặn,
bằng cách dùng thao tác gửi trong câu lệnh `select` với trường hợp `default`.
Nếu việc gửi không thể thực hiện ngay lập tức thì trường hợp mặc định sẽ được chọn.
Việc làm cho thao tác gửi không chặn đảm bảo rằng không goroutine nào được khởi chạy
trong vòng lặp sẽ bị treo.
Tuy nhiên, nếu kết quả đến trước khi hàm chính thực hiện đến thao tác nhận,
việc gửi có thể thất bại vì không có ai sẵn sàng nhận.

Vấn đề này là một ví dụ kinh điển trong sách giáo khoa về thứ được gọi là [điều kiện tranh chấp](https://en.wikipedia.org/wiki/Race_condition),
nhưng cách sửa rất đơn giản.
Chúng ta chỉ cần đảm bảo đệm channel `ch` (bằng cách thêm độ dài bộ đệm
làm đối số thứ hai cho [make](/pkg/builtin/#make)),
đảm bảo lần gửi đầu tiên có nơi để đặt giá trị.
Điều này đảm bảo việc gửi sẽ luôn thành công,
và giá trị đầu tiên đến sẽ được lấy bất kể thứ tự thực thi.

Hai ví dụ này minh họa sự đơn giản mà Go có thể biểu đạt các tương tác phức tạp giữa các goroutine.
