<!--{
  "Title": "Hủy các thao tác đang thực hiện"
}-->

Bạn có thể quản lý các thao tác đang thực hiện bằng cách sử dụng
[`context.Context`](https://pkg.go.dev/context#Context) trong Go. `Context` là
một kiểu dữ liệu chuẩn trong Go có khả năng báo hiệu khi thao tác tổng thể mà
nó đại diện đã bị hủy và không còn cần thiết nữa. Bằng cách truyền
`context.Context` qua các lời gọi hàm và dịch vụ trong ứng dụng, các thành
phần đó có thể dừng hoạt động sớm và trả về lỗi khi quá trình xử lý của chúng
không còn cần thiết. Để tìm hiểu thêm về `Context`, xem
[Go Concurrency Patterns: Context](/blog/context).

Ví dụ, bạn có thể muốn:

*   Kết thúc các thao tác chạy lâu, bao gồm các thao tác cơ sở dữ liệu mất
    quá nhiều thời gian để hoàn thành.
*   Lan truyền yêu cầu hủy từ nơi khác, chẳng hạn khi client đóng kết nối.

Nhiều API dành cho lập trình viên Go bao gồm các phương thức nhận tham số
`Context`, giúp bạn dễ dàng sử dụng `Context` trong toàn bộ ứng dụng.

### Hủy thao tác cơ sở dữ liệu sau khi hết thời gian chờ {#timeout_cancel}

Bạn có thể sử dụng `Context` để đặt thời gian chờ hoặc thời hạn, sau đó một
thao tác sẽ bị hủy. Để tạo một `Context` có thời gian chờ hoặc thời hạn, gọi
[`context.WithTimeout`](https://pkg.go.dev/context#WithTimeout) hoặc
[`context.WithDeadline`](https://pkg.go.dev/context#WithDeadline).

Đoạn mã trong ví dụ thời gian chờ dưới đây tạo một `Context` và truyền vào
phương thức [`QueryContext`](https://pkg.go.dev/database/sql#DB.QueryContext)
của `sql.DB`.

```
func QueryWithTimeout(ctx context.Context) {
	// Create a Context with a timeout.
	queryCtx, cancel := context.WithTimeout(ctx, 5*time.Second)
	defer cancel()

	// Pass the timeout Context with a query.
	rows, err := db.QueryContext(queryCtx, "SELECT * FROM album")
	if err != nil {
		log.Fatal(err)
	}
	defer rows.Close()

	// Handle returned rows.
}
```

Khi một context được tạo từ context bên ngoài, như `queryCtx` được tạo từ
`ctx` trong ví dụ này, nếu context bên ngoài bị hủy thì context dẫn xuất cũng
tự động bị hủy theo. Ví dụ, trong các HTTP server, phương thức
`http.Request.Context` trả về một context gắn với request đó. Context đó bị
hủy nếu HTTP client ngắt kết nối hoặc hủy HTTP request (điều này có thể xảy ra
trong HTTP/2). Truyền context của một HTTP request vào `QueryWithTimeout` ở
trên sẽ khiến truy vấn cơ sở dữ liệu dừng sớm nếu HTTP request tổng thể bị
hủy hoặc nếu truy vấn mất hơn năm giây.

**Lưu ý:** Luôn defer lời gọi đến hàm `cancel` được trả về khi bạn tạo một
`Context` mới có thời gian chờ hoặc thời hạn. Điều này giải phóng các tài
nguyên mà `Context` mới nắm giữ khi hàm chứa nó thoát ra. Nó cũng hủy
`queryCtx`, nhưng vào thời điểm hàm trả về, không còn gì đang sử dụng
`queryCtx` nữa.
