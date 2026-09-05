<!--{
  "Title": "Tránh nguy cơ SQL injection"
}-->

Bạn có thể tránh nguy cơ SQL injection bằng cách cung cấp các giá trị tham số
SQL dưới dạng tham số hàm trong gói `sql`. Nhiều hàm trong gói `sql` cung cấp
các tham số cho câu lệnh SQL và cho các giá trị sẽ được dùng trong các tham
số của câu lệnh đó (các hàm khác cung cấp tham số cho prepared statement và
các tham số đi kèm).

Đoạn mã trong ví dụ dưới đây dùng ký hiệu `?` làm placeholder cho tham số
`id`, được cung cấp dưới dạng tham số hàm:

```
// Correct format for executing an SQL statement with parameters.
rows, err := db.Query("SELECT * FROM user WHERE id = ?", id)
```

Các hàm gói `sql` thực hiện thao tác cơ sở dữ liệu tạo prepared statement từ
các tham số bạn cung cấp. Tại thời điểm chạy, gói `sql` chuyển câu lệnh SQL
thành prepared statement và gửi nó cùng với tham số, được tách biệt riêng.

**Lưu ý:** Các placeholder tham số thay đổi tùy theo DBMS và driver bạn đang
dùng. Ví dụ, [pq driver](https://pkg.go.dev/github.com/lib/pq) cho Postgres
chấp nhận dạng placeholder như `$1` thay vì `?`.

Bạn có thể bị cám dỗ dùng một hàm từ gói `fmt` để tạo câu lệnh SQL dưới dạng
chuỗi với các tham số được nhúng trực tiếp, như sau:

```
// SECURITY RISK!
rows, err := db.Query(fmt.Sprintf("SELECT * FROM user WHERE id = %s", id))
```

Cách này không an toàn! Khi bạn làm như vậy, Go tạo toàn bộ câu lệnh SQL,
thay thế format verb `%s` bằng giá trị tham số, trước khi gửi câu lệnh đầy
đủ đến DBMS. Điều này tạo ra nguy cơ
[SQL injection](https://en.wikipedia.org/wiki/SQL_injection) vì người gọi mã
có thể gửi một đoạn SQL bất ngờ dưới dạng tham số `id`. Đoạn SQL đó có thể
hoàn thành câu lệnh SQL theo những cách không thể đoán trước và gây nguy hiểm
cho ứng dụng của bạn.

Ví dụ, bằng cách truyền vào một giá trị `%s` nhất định, bạn có thể nhận được
kết quả như sau, có thể trả về tất cả bản ghi người dùng trong cơ sở dữ liệu:

```
SELECT * FROM user WHERE id = 1 OR 1=1;
```
