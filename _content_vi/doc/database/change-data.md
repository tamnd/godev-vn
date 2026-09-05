<!--{
  "Title": "Thực thi các câu lệnh SQL không trả về dữ liệu"
}-->

Khi thực hiện các thao tác cơ sở dữ liệu không trả về dữ liệu, hãy sử dụng
phương thức `Exec` hoặc `ExecContext` từ gói `database/sql`. Các câu lệnh SQL
bạn thực thi theo cách này bao gồm `INSERT`, `DELETE` và `UPDATE`.

Khi truy vấn có thể trả về các hàng dữ liệu, hãy dùng phương thức `Query`
hoặc `QueryContext` thay thế. Để tìm hiểu thêm, xem
[Truy vấn cơ sở dữ liệu](/doc/database/querying).

Phương thức `ExecContext` hoạt động giống `Exec` nhưng nhận thêm tham số
`context.Context`, như mô tả trong
[Hủy các thao tác đang thực hiện](/doc/database/cancel-operations).

Đoạn mã trong ví dụ dưới đây sử dụng
[`DB.Exec`](https://pkg.go.dev/database/sql#DB.Exec) để thực thi một câu lệnh
thêm bản ghi album mới vào bảng `album`.

```
func AddAlbum(alb Album) (int64, error) {
	result, err := db.Exec("INSERT INTO album (title, artist) VALUES (?, ?)", alb.Title, alb.Artist)
	if err != nil {
		return 0, fmt.Errorf("AddAlbum: %v", err)
	}

	// Get the new album's generated ID for the client.
	id, err := result.LastInsertId()
	if err != nil {
		return 0, fmt.Errorf("AddAlbum: %v", err)
	}
	// Return the new album's ID.
	return id, nil
}
```

`DB.Exec` trả về hai giá trị: một
[`sql.Result`](https://pkg.go.dev/database/sql#Result) và một error. Khi error
là `nil`, bạn có thể dùng `Result` để lấy ID của bản ghi vừa chèn (như trong
ví dụ) hoặc để lấy số hàng bị ảnh hưởng bởi thao tác.

**Lưu ý:** Các placeholder tham số trong prepared statement thay đổi tùy theo
DBMS và driver bạn đang dùng. Ví dụ,
[pq driver](https://pkg.go.dev/github.com/lib/pq) cho Postgres yêu cầu
placeholder dạng `$1` thay vì `?`.

Nếu mã của bạn sẽ thực thi cùng một câu lệnh SQL nhiều lần, hãy cân nhắc
dùng `sql.Stmt` để tạo một prepared statement có thể tái sử dụng từ câu lệnh
SQL đó. Để tìm hiểu thêm, xem
[Sử dụng prepared statement](/doc/database/prepared-statements).

**Chú ý:** Đừng dùng các hàm định dạng chuỗi như `fmt.Sprintf` để tạo câu
lệnh SQL! Bạn có thể tạo ra nguy cơ SQL injection. Để tìm hiểu thêm, xem
[Tránh nguy cơ SQL injection](/doc/database/sql-injection).

#### Các hàm thực thi câu lệnh SQL không trả về hàng dữ liệu {#no_rows_functions}

<table id="no-rows-functions-list" class="DocTable">
  <thead>
    <tr class="DocTable-head">
      <th class="DocTable-cell" width="20%">Hàm</th>
      <th class="DocTable-cell">Mô tả</th>
    </tr>
  </thead>
  <tbody>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#DB.Exec">DB.Exec</a></code><br/>
        <code><a href="https://pkg.go.dev/database/sql#DB.ExecContext">DB.ExecContext</a></code>
      </td>
      <td class="DocTable-cell">Thực thi một câu lệnh SQL độc lập.</td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#Tx.Exec">Tx.Exec</a></code><br/>
        <code><a href="https://pkg.go.dev/database/sql#Tx.ExecContext">Tx.ExecContext</a></code>
      </td>
      <td class="DocTable-cell">Thực thi một câu lệnh SQL trong phạm vi một transaction lớn hơn. Để tìm hiểu thêm, xem
          <a href="/doc/database/execute-transactions">Thực thi transaction</a>.
      </td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#Stmt.Exec">Stmt.Exec</a></code><br/>
        <code><a href="https://pkg.go.dev/database/sql#Stmt.ExecContext">Stmt.ExecContext</a></code>
      </td>
      <td class="DocTable-cell">Thực thi một prepared statement đã được chuẩn bị trước. Để tìm hiểu thêm, xem
          <a href="/doc/database/prepared-statements">Sử dụng prepared statement</a>.
      </td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#Conn.ExecContext">Conn.ExecContext</a></code>
      </td>
      <td class="DocTable-cell">Dùng với các kết nối dành riêng. Để tìm hiểu thêm, xem
          <a href="/doc/database/manage-connections">Quản lý kết nối</a>.
      </td>
    </tr>
  </tbody>
</table>
