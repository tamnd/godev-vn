<!--{
  "Title": "Sử dụng prepared statement"
}-->

Bạn có thể định nghĩa một prepared statement để sử dụng lặp lại. Điều này giúp
mã chạy nhanh hơn một chút bằng cách tránh chi phí tạo lại câu lệnh mỗi lần
mã thực hiện thao tác cơ sở dữ liệu.

**Lưu ý:** Các placeholder tham số trong prepared statement thay đổi tùy theo
DBMS và driver bạn đang dùng. Ví dụ,
[pq driver](https://pkg.go.dev/github.com/lib/pq) cho Postgres yêu cầu
placeholder dạng `$1` thay vì `?`.

### Prepared statement là gì? {#what_prepared_statement}

Prepared statement là SQL được phân tích cú pháp và lưu lại bởi DBMS, thường
chứa các placeholder nhưng không có giá trị tham số thực tế. Sau đó, câu lệnh
có thể được thực thi với một tập hợp các giá trị tham số.

### Cách sử dụng prepared statement {#use_prepared_statement}

Khi bạn muốn thực thi cùng một câu lệnh SQL nhiều lần, bạn có thể dùng
`sql.Stmt` để chuẩn bị câu lệnh SQL trước, sau đó thực thi nó khi cần.

Ví dụ dưới đây tạo một prepared statement chọn một album cụ thể từ cơ sở dữ
liệu. [`DB.Prepare`](https://pkg.go.dev/database/sql#DB.Prepare) trả về một
[`sql.Stmt`](https://pkg.go.dev/database/sql#Stmt) đại diện cho prepared
statement cho một đoạn SQL nhất định. Bạn có thể truyền các tham số cho câu
lệnh SQL vào `Stmt.Exec`, `Stmt.QueryRow` hoặc `Stmt.Query` để chạy câu lệnh.

```
// AlbumByID retrieves the specified album.
func AlbumByID(id int) (Album, error) {
	// Define a prepared statement. You'd typically define the statement
	// elsewhere and save it for use in functions such as this one.
	stmt, err := db.Prepare("SELECT * FROM album WHERE id = ?")
	if err != nil {
		log.Fatal(err)
	}
	defer stmt.Close()

	var album Album

	// Execute the prepared statement, passing in an id value for the
	// parameter whose placeholder is ?
	err := stmt.QueryRow(id).Scan(&album.ID, &album.Title, &album.Artist, &album.Price, &album.Quantity)
	if err != nil {
		if err == sql.ErrNoRows {
			// Handle the case of no rows returned.
		}
		return album, err
	}
	return album, nil
}
```

### Hành vi của prepared statement {#behavior}

Một [`sql.Stmt`](https://pkg.go.dev/database/sql#Stmt) đã được chuẩn bị cung
cấp các phương thức `Exec`, `QueryRow` và `Query` thông thường để gọi câu lệnh.
Để tìm hiểu thêm về cách sử dụng các phương thức này, xem
[Truy vấn dữ liệu](/doc/database/querying) và
[Thực thi các câu lệnh SQL không trả về dữ liệu](/doc/database/change-data).

Tuy nhiên, vì `sql.Stmt` đã đại diện cho một câu lệnh SQL cố định sẵn, các
phương thức `Exec`, `QueryRow` và `Query` của nó chỉ nhận các giá trị tham số
SQL tương ứng với placeholder, bỏ qua phần text SQL.

Bạn có thể định nghĩa `sql.Stmt` mới theo nhiều cách khác nhau, tùy thuộc vào
cách bạn sẽ sử dụng nó.

*   `DB.Prepare` và `DB.PrepareContext` tạo prepared statement có thể thực thi
    độc lập, bên ngoài transaction, giống như `DB.Exec` và `DB.Query`.
*   `Tx.Prepare`, `Tx.PrepareContext`, `Tx.Stmt` và `Tx.StmtContext` tạo
    prepared statement để dùng trong một transaction cụ thể. `Prepare` và
    `PrepareContext` dùng text SQL để định nghĩa câu lệnh. `Stmt` và
    `StmtContext` dùng kết quả của `DB.Prepare` hoặc `DB.PrepareContext`, nghĩa
    là chúng chuyển đổi một `sql.Stmt` không dành cho transaction thành
    `sql.Stmt` dành cho transaction này.
*   `Conn.PrepareContext` tạo prepared statement từ `sql.Conn`, đại diện cho
    một kết nối dành riêng.

Hãy đảm bảo `stmt.Close` được gọi khi mã của bạn dùng xong câu lệnh. Điều
này sẽ giải phóng mọi tài nguyên cơ sở dữ liệu (chẳng hạn các kết nối bên
dưới) có thể gắn với nó. Với các câu lệnh chỉ là biến cục bộ trong một hàm,
chỉ cần `defer stmt.Close()` là đủ.

#### Các hàm tạo prepared statement {#prepared_statement_functions}

<table id="prepared-statement-functions-list" class="DocTable">
    <thead>
        <tr class="DocTable-head">
            <th class="DocTable-cell" width="20%">Hàm</th>
            <th class="DocTable-cell">Mô tả</th>
        </tr>
    </thead>
    <tbody>
        <tr class="DocTable-row">
            <td class="DocTable-cell">
                <code><a href="https://pkg.go.dev/database/sql#DB.Prepare">DB.Prepare</a></code><br />
                <code><a href="https://pkg.go.dev/database/sql#DB.PrepareContext">DB.PrepareContext</a></code>
            </td>
            <td class="DocTable-cell">Chuẩn bị câu lệnh để thực thi độc lập hoặc để chuyển đổi thành
                prepared statement trong transaction bằng Tx.Stmt.</td>
        </tr>
        <tr class="DocTable-row">
            <td class="DocTable-cell">
                <code><a href="https://pkg.go.dev/database/sql#Tx.Prepare">Tx.Prepare</a></code><br />
                <code><a href="https://pkg.go.dev/database/sql#Tx.PrepareContext">Tx.PrepareContext</a></code><br />
                <code><a href="https://pkg.go.dev/database/sql#Tx.Stmt">Tx.Stmt</a></code><br />
                <code><a href="https://pkg.go.dev/database/sql#Tx.StmtContext">Tx.StmtContext</a></code>
            </td>
            <td class="DocTable-cell">Chuẩn bị câu lệnh để dùng trong một transaction cụ thể. Để tìm
                hiểu thêm, xem
                <a href="/doc/database/execute-transactions">Thực thi transaction</a>.
            </td>
        </tr>
        <tr class="DocTable-row">
            <td class="DocTable-cell">
                <code><a href="https://pkg.go.dev/database/sql#Conn.PrepareContext">Conn.PrepareContext</a></code>
            </td>
            <td class="DocTable-cell">Dùng với các kết nối dành riêng. Để tìm hiểu thêm, xem
                <a href="/doc/database/manage-connections">Quản lý kết nối</a>.
            </td>
        </tr>
    </tbody>
</table>
