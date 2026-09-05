<!--{
  "Title": "Truy vấn dữ liệu"
}-->

Khi thực thi một câu lệnh SQL trả về dữ liệu, hãy dùng một trong các phương
thức `Query` được cung cấp trong gói `database/sql`. Mỗi phương thức trả về
một `Row` hoặc `Rows` mà bạn có thể sao chép dữ liệu vào các biến bằng phương
thức `Scan`. Bạn sẽ dùng các phương thức này để thực thi các câu lệnh `SELECT`
chẳng hạn.

Khi thực thi một câu lệnh không trả về dữ liệu, bạn có thể dùng phương thức
`Exec` hoặc `ExecContext` thay thế. Để tìm hiểu thêm, xem
[Thực thi các câu lệnh không trả về dữ liệu](/doc/database/change-data).

Gói `database/sql` cung cấp hai cách để thực thi truy vấn lấy kết quả.

*   **Truy vấn một hàng duy nhất** - `QueryRow` trả về tối đa một `Row` từ
    cơ sở dữ liệu. Để tìm hiểu thêm, xem
    [Truy vấn một hàng duy nhất](#single_row).
*   **Truy vấn nhiều hàng** - `Query` trả về tất cả các hàng khớp dưới dạng
    struct `Rows` mà mã của bạn có thể lặp qua. Để tìm hiểu thêm, xem
    [Truy vấn nhiều hàng](#multiple_rows).

Nếu mã của bạn sẽ thực thi cùng một câu lệnh SQL nhiều lần, hãy cân nhắc dùng
prepared statement. Để tìm hiểu thêm, xem
[Sử dụng prepared statement](/doc/database/prepared-statements).

**Chú ý:** Đừng dùng các hàm định dạng chuỗi như `fmt.Sprintf` để tạo câu
lệnh SQL! Bạn có thể tạo ra nguy cơ SQL injection. Để tìm hiểu thêm, xem
[Tránh nguy cơ SQL injection](/doc/database/sql-injection).

### Truy vấn một hàng duy nhất {#single_row}

`QueryRow` lấy tối đa một hàng cơ sở dữ liệu, chẳng hạn khi bạn muốn tra cứu
dữ liệu theo một ID duy nhất. Nếu truy vấn trả về nhiều hàng, phương thức
`Scan` sẽ loại bỏ tất cả trừ hàng đầu tiên.

`QueryRowContext` hoạt động giống `QueryRow` nhưng nhận thêm tham số
`context.Context`. Để tìm hiểu thêm, xem
[Hủy các thao tác đang thực hiện](/doc/database/cancel-operations).

Ví dụ dưới đây dùng truy vấn để kiểm tra xem có đủ hàng tồn kho để hỗ trợ
một giao dịch mua hàng không. Câu lệnh SQL trả về `true` nếu có đủ, `false`
nếu không.
[`Row.Scan`](https://pkg.go.dev/database/sql#Row.Scan) sao chép giá trị boolean
trả về vào biến `enough` thông qua một con trỏ.

```
func canPurchase(id int, quantity int) (bool, error) {
	var enough bool
	// Query for a value based on a single row.
	if err := db.QueryRow("SELECT (quantity >= ?) from album where id = ?",
		quantity, id).Scan(&enough); err != nil {
		if err == sql.ErrNoRows {
			return false, fmt.Errorf("canPurchase %d: unknown album", id)
		}
		return false, fmt.Errorf("canPurchase %d: %v", id, err)
	}
	return enough, nil
}
```

**Lưu ý:** Các placeholder tham số trong prepared statement thay đổi tùy theo
DBMS và driver bạn đang dùng. Ví dụ,
[pq driver](https://pkg.go.dev/github.com/lib/pq) cho Postgres yêu cầu
placeholder dạng `$1` thay vì `?`.

#### Xử lý lỗi {#single_row_errors}

Bản thân `QueryRow` không trả về lỗi. Thay vào đó, `Scan` báo cáo bất kỳ lỗi
nào từ quá trình tra cứu và quét kết hợp. Nó trả về
[`sql.ErrNoRows`](https://pkg.go.dev/database/sql#ErrNoRows) khi truy vấn
không tìm thấy hàng nào.

#### Các hàm trả về một hàng duy nhất {#single_row_functions}

<table id="single-row-functions-list" class="DocTable">
  <thead>
    <tr class="DocTable-head">
      <th class="DocTable-cell" width="20%">Hàm</th>
      <th class="DocTable-cell">Mô tả</th>
    </tr>
  </thead>
  <tbody>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#DB.QueryRow">DB.QueryRow</a></code><br />
        <code><a href="https://pkg.go.dev/database/sql#DB.QueryRowContext">DB.QueryRowContext</a></code>
      </td>
      <td class="DocTable-cell">Thực thi truy vấn một hàng độc lập.</td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#Tx.QueryRow">Tx.QueryRow</a></code><br />
        <code><a href="https://pkg.go.dev/database/sql#Tx.QueryRowContext">Tx.QueryRowContext</a></code>
      </td>
      <td class="DocTable-cell">Thực thi truy vấn một hàng trong phạm vi một transaction lớn hơn. Để
        tìm hiểu thêm, xem
        <a href="/doc/database/execute-transactions">Thực thi transaction</a>.
      </td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#Stmt.QueryRow">Stmt.QueryRow</a></code><br />
        <code><a href="https://pkg.go.dev/database/sql#Stmt.QueryRowContext">Stmt.QueryRowContext</a></code>
      </td>
      <td class="DocTable-cell">Thực thi truy vấn một hàng bằng prepared statement đã có sẵn. Để tìm
        hiểu thêm, xem
        <a href="/doc/database/prepared-statements">Sử dụng prepared statement</a>.
      </td>
    </tr>
    <tr class="DocTable-row">
        <td class="DocTable-cell">
  <code><a href="https://pkg.go.dev/database/sql#Conn.QueryRowContext">Conn.QueryRowContext</a></code>
      </td>
      <td class="DocTable-cell">Dùng với các kết nối dành riêng. Để tìm hiểu thêm, xem
        <a href="/doc/database/manage-connections">Quản lý kết nối</a>.
      </td>
    </tr>
  </tbody>
</table>

### Truy vấn nhiều hàng {#multiple_rows}

Bạn có thể truy vấn nhiều hàng bằng `Query` hoặc `QueryContext`, trả về `Rows`
đại diện cho kết quả truy vấn. Mã của bạn lặp qua các hàng trả về bằng
[`Rows.Next`](https://pkg.go.dev/database/sql#Rows.Next). Mỗi lần lặp gọi
`Scan` để sao chép các giá trị cột vào các biến.

`QueryContext` hoạt động giống `Query` nhưng nhận thêm tham số
`context.Context`. Để tìm hiểu thêm, xem
[Hủy các thao tác đang thực hiện](/doc/database/cancel-operations).

Ví dụ dưới đây thực thi truy vấn để trả về các album của một nghệ sĩ cụ thể.
Các album được trả về trong `sql.Rows`. Mã dùng
[`Rows.Scan`](https://pkg.go.dev/database/sql#Rows.Scan) để sao chép các giá
trị cột vào các biến được đại diện bởi con trỏ.

```
func albumsByArtist(artist string) ([]Album, error) {
	rows, err := db.Query("SELECT * FROM album WHERE artist = ?", artist)
	if err != nil {
		return nil, err
	}
	defer rows.Close()

	// An album slice to hold data from returned rows.
	var albums []Album

	// Loop through rows, using Scan to assign column data to struct fields.
	for rows.Next() {
		var alb Album
		if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist,
			&alb.Price, &alb.Quantity); err != nil {
			return albums, err
		}
		albums = append(albums, alb)
	}
	if err = rows.Err(); err != nil {
		return albums, err
	}
	return albums, nil
}
```

Lưu ý lời gọi defer đến
[`rows.Close`](https://pkg.go.dev/database/sql#Rows.Close). Điều này giải
phóng mọi tài nguyên được giữ bởi các hàng dù hàm trả về theo cách nào. Lặp
qua toàn bộ các hàng cũng đóng nó một cách ngầm định, nhưng tốt hơn là dùng
`defer` để đảm bảo `rows` được đóng bất kể điều gì xảy ra.

**Lưu ý:** Các placeholder tham số trong prepared statement thay đổi tùy theo
DBMS và driver bạn đang dùng. Ví dụ,
[pq driver](https://pkg.go.dev/github.com/lib/pq) cho Postgres yêu cầu
placeholder dạng `$1` thay vì `?`.

#### Xử lý lỗi {#multiple_rows_errors}

Hãy đảm bảo kiểm tra lỗi từ `sql.Rows` sau khi lặp qua kết quả truy vấn. Nếu
truy vấn thất bại, đây là cách mã của bạn phát hiện ra điều đó.

#### Các hàm trả về nhiều hàng {#multiple_rows_functions}

<table id="multiple-row-functions-list" class="DocTable">
  <thead>
    <tr class="DocTable-head">
      <th class="DocTable-cell" width="20%">Hàm</th>
      <th class="DocTable-cell">Mô tả</th>
    </tr>
  </thead>
  <tbody>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#DB.Query">DB.Query</a></code><br />
        <code><a href="https://pkg.go.dev/database/sql#DB.QueryContext">DB.QueryContext</a></code>
      </td>
      <td class="DocTable-cell">Thực thi truy vấn độc lập.</td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#Tx.Query">Tx.Query</a></code><br />
        <code><a href="https://pkg.go.dev/database/sql#Tx.QueryContext">Tx.QueryContext</a></code>
      </td>
      <td class="DocTable-cell">Thực thi truy vấn trong phạm vi một transaction lớn hơn. Để tìm hiểu
        thêm, xem
        <a href="/doc/database/execute-transactions">Thực thi transaction</a>.
      </td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#Stmt.Query">Stmt.Query</a></code><br />
        <code><a href="https://pkg.go.dev/database/sql#Stmt.QueryContext">Stmt.QueryContext</a></code>
      </td>
      <td class="DocTable-cell">Thực thi truy vấn bằng prepared statement đã có sẵn. Để tìm hiểu
        thêm, xem
        <a href="/doc/database/prepared-statements">Sử dụng prepared statement</a>.
    </td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">
        <code><a href="https://pkg.go.dev/database/sql#Conn.QueryContext">Conn.QueryContext</a></code>
      </td>
      <td class="DocTable-cell">Dùng với các kết nối dành riêng. Để tìm hiểu thêm, xem
        <a href="/doc/database/manage-connections">Quản lý kết nối</a>.
      </td>
    </tr>
  </tbody>
</table>

### Xử lý giá trị cột có thể null {#nullable_columns}

Gói `database/sql` cung cấp một số kiểu đặc biệt bạn có thể dùng làm tham số
cho hàm `Scan` khi giá trị của cột có thể là null. Mỗi kiểu bao gồm một
trường `Valid` báo cáo xem giá trị có khác null không, và một trường chứa giá
trị nếu có.

Đoạn mã trong ví dụ dưới đây truy vấn tên khách hàng. Nếu giá trị tên là null,
mã thay thế bằng một giá trị khác để dùng trong ứng dụng.

```
var s sql.NullString
err := db.QueryRow("SELECT name FROM customer WHERE id = ?", id).Scan(&s)
if err != nil {
	log.Fatal(err)
}

// Find customer name, using placeholder if not present.
name := "Valued Customer"
if s.Valid {
	name = s.String
}
```

Xem thêm về từng kiểu trong tài liệu gói `sql`:

*    [`NullBool`](https://pkg.go.dev/database/sql#NullBool)
*    [`NullFloat64`](https://pkg.go.dev/database/sql#NullFloat64)
*    [`NullInt32`](https://pkg.go.dev/database/sql#NullInt32)
*    [`NullInt64`](https://pkg.go.dev/database/sql#NullInt64)
*    [`NullString`](https://pkg.go.dev/database/sql#NullString)
*    [`NullTime`](https://pkg.go.dev/database/sql#NullTime)

### Lấy dữ liệu từ các cột {#column_data}

Khi lặp qua các hàng được trả về bởi truy vấn, bạn dùng `Scan` để sao chép
các giá trị cột của một hàng vào các giá trị Go, như mô tả trong tài liệu
[`Rows.Scan`](https://pkg.go.dev/database/sql#Rows.Scan).

Có một tập hợp chuyển đổi dữ liệu cơ bản được tất cả driver hỗ trợ, chẳng
hạn chuyển đổi SQL `INT` thành Go `int`. Một số driver mở rộng tập hợp chuyển
đổi này; xem tài liệu của từng driver để biết chi tiết.

Như bạn có thể mong đợi, `Scan` sẽ chuyển đổi từ kiểu cột sang kiểu Go tương
tự. Ví dụ, `Scan` sẽ chuyển đổi từ SQL `CHAR`, `VARCHAR` và `TEXT` sang Go
`string`. Tuy nhiên, `Scan` cũng thực hiện chuyển đổi sang kiểu Go khác phù
hợp với giá trị cột. Ví dụ, nếu cột là `VARCHAR` luôn chứa một số, bạn có thể
chỉ định kiểu số Go, chẳng hạn `int`, để nhận giá trị, và `Scan` sẽ chuyển
đổi bằng `strconv.Atoi` cho bạn.

Để biết thêm chi tiết về các chuyển đổi mà hàm `Scan` thực hiện, xem tài liệu
[`Rows.Scan`](https://pkg.go.dev/database/sql#Rows.Scan).

### Xử lý nhiều tập kết quả {#multiple_result_sets}

Khi thao tác cơ sở dữ liệu của bạn có thể trả về nhiều tập kết quả, bạn có
thể lấy chúng bằng cách dùng
[`Rows.NextResultSet`](https://pkg.go.dev/database/sql#Rows.NextResultSet).
Điều này có thể hữu ích, ví dụ, khi bạn gửi SQL truy vấn riêng biệt nhiều
bảng, trả về một tập kết quả cho mỗi bảng.

`Rows.NextResultSet` chuẩn bị tập kết quả tiếp theo để lời gọi `Rows.Next` lấy
hàng đầu tiên từ tập đó. Nó trả về một giá trị boolean cho biết có tập kết quả
tiếp theo hay không.

Đoạn mã trong ví dụ dưới đây dùng `DB.Query` để thực thi hai câu lệnh SQL.
Tập kết quả đầu tiên từ truy vấn đầu tiên trong thủ tục, lấy tất cả các hàng
trong bảng `album`. Tập kết quả tiếp theo từ truy vấn thứ hai, lấy các hàng từ
bảng `song`.

```
rows, err := db.Query("SELECT * from album; SELECT * from song;")
if err != nil {
	log.Fatal(err)
}
defer rows.Close()

// Loop through the first result set.
for rows.Next() {
	// Handle result set.
}

// Advance to next result set.
rows.NextResultSet()

// Loop through the second result set.
for rows.Next() {
	// Handle second set.
}

// Check for any error in either result set.
if err := rows.Err(); err != nil {
	log.Fatal(err)
}
```
