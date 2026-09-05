<!--{
  "Title": "Thực thi transaction"
}-->

Bạn có thể thực thi các transaction cơ sở dữ liệu bằng cách sử dụng
[`sql.Tx`](https://pkg.go.dev/database/sql#Tx), đại diện cho một transaction.
Ngoài các phương thức `Commit` và `Rollback` đặc trưng cho transaction,
`sql.Tx` còn có tất cả các phương thức dùng để thực hiện các thao tác cơ sở
dữ liệu thông thường. Để lấy `sql.Tx`, bạn gọi `DB.Begin` hoặc `DB.BeginTx`.

Một [transaction cơ sở dữ liệu](https://en.wikipedia.org/wiki/Database_transaction)
nhóm nhiều thao tác lại như một phần của mục tiêu lớn hơn. Tất cả các thao tác
phải thành công hoặc không thao tác nào được áp dụng, trong khi tính toàn vẹn
của dữ liệu được đảm bảo trong cả hai trường hợp. Thông thường, quy trình làm
việc với transaction bao gồm:

1. Bắt đầu transaction.
2. Thực hiện một tập hợp các thao tác cơ sở dữ liệu.
3. Nếu không có lỗi, commit transaction để áp dụng các thay đổi vào cơ sở dữ liệu.
4. Nếu có lỗi, rollback transaction để giữ nguyên cơ sở dữ liệu.

Gói `sql` cung cấp các phương thức để bắt đầu và kết thúc transaction, cũng
như các phương thức thực hiện các thao tác cơ sở dữ liệu ở giữa. Các phương
thức này tương ứng với bốn bước trong quy trình trên.

*   Bắt đầu transaction.

    [`DB.Begin`](https://pkg.go.dev/database/sql#DB.Begin) hoặc
    [`DB.BeginTx`](https://pkg.go.dev/database/sql#DB.BeginTx) bắt đầu một
    transaction cơ sở dữ liệu mới, trả về `sql.Tx` đại diện cho transaction đó.

*   Thực hiện các thao tác cơ sở dữ liệu.

    Sử dụng `sql.Tx`, bạn có thể truy vấn hoặc cập nhật cơ sở dữ liệu qua
    một loạt thao tác dùng chung một kết nối. Để hỗ trợ điều này, `Tx` xuất
    các phương thức sau:

    *   [`Exec`](https://pkg.go.dev/database/sql#Tx.Exec) và
        [`ExecContext`](https://pkg.go.dev/database/sql#Tx.ExecContext) dùng để
        thay đổi cơ sở dữ liệu thông qua các câu lệnh SQL như `INSERT`,
        `UPDATE` và `DELETE`.

        Để tìm hiểu thêm, xem
        [Thực thi các câu lệnh SQL không trả về dữ liệu](/doc/database/change-data).

    *   [`Query`](https://pkg.go.dev/database/sql#Tx.Query),
        [`QueryContext`](https://pkg.go.dev/database/sql#Tx.QueryContext),
        [`QueryRow`](https://pkg.go.dev/database/sql#Tx.QueryRow) và
        [`QueryRowContext`](https://pkg.go.dev/database/sql#Tx.QueryRowContext)
        dùng cho các thao tác trả về hàng dữ liệu.

        Để tìm hiểu thêm, xem [Truy vấn dữ liệu](/doc/database/querying).

    *   [`Prepare`](https://pkg.go.dev/database/sql#Tx.Prepare),
        [`PrepareContext`](https://pkg.go.dev/database/sql#Tx.PrepareContext),
        [`Stmt`](https://pkg.go.dev/database/sql#Tx.Stmt) và
        [`StmtContext`](https://pkg.go.dev/database/sql#Tx.StmtContext) dùng
        để định nghĩa trước các prepared statement.

        Để tìm hiểu thêm, xem
        [Sử dụng prepared statement](/doc/database/prepared-statements).

*   Kết thúc transaction bằng _một_ trong các cách sau:
    *   Commit transaction bằng
        [`Tx.Commit`](https://pkg.go.dev/database/sql#Tx.Commit).

        Nếu `Commit` thành công (trả về error `nil`), tất cả kết quả truy vấn
        được xác nhận là hợp lệ và tất cả các cập nhật đã thực hiện được áp
        dụng vào cơ sở dữ liệu như một thay đổi nguyên tử duy nhất. Nếu
        `Commit` thất bại, tất cả kết quả từ `Query` và `Exec` trên `Tx` đó
        nên bị loại bỏ vì không hợp lệ.

    *   Rollback transaction bằng
        [`Tx.Rollback`](https://pkg.go.dev/database/sql#Tx.Rollback).

        Dù `Tx.Rollback` có thất bại, transaction cũng sẽ không còn hợp lệ
        và không được commit vào cơ sở dữ liệu.

### Thực hành tốt nhất {#best_practices}

Tuân theo các thực hành tốt nhất dưới đây để điều hướng tốt hơn các ngữ nghĩa
phức tạp và việc quản lý kết nối mà transaction đôi khi yêu cầu.

*   Sử dụng các API được mô tả trong phần này để quản lý transaction. Đừng
    dùng trực tiếp các câu lệnh SQL liên quan đến transaction như `BEGIN` và
    `COMMIT`, vì điều đó có thể khiến cơ sở dữ liệu ở trạng thái không dự
    đoán được, đặc biệt trong các chương trình chạy đồng thời.
*   Khi sử dụng transaction, hãy cẩn thận không gọi trực tiếp các phương thức
    `sql.DB` không thuộc transaction, vì chúng sẽ thực thi ngoài phạm vi
    transaction, khiến mã của bạn có cái nhìn không nhất quán về trạng thái
    cơ sở dữ liệu hoặc thậm chí gây ra deadlock.

### Ví dụ {#example}

Đoạn mã trong ví dụ dưới đây sử dụng transaction để tạo một đơn đặt hàng mới
cho một album. Trong quá trình đó, mã sẽ:

1. Bắt đầu transaction.
2. Defer rollback transaction. Nếu transaction thành công, nó sẽ được commit
   trước khi hàm thoát ra, khiến lời gọi rollback đã defer trở thành no-op.
   Nếu transaction thất bại, nó sẽ không được commit, nghĩa là rollback sẽ
   được gọi khi hàm thoát.
3. Xác nhận rằng còn đủ hàng tồn kho cho album mà khách hàng đặt.
4. Nếu đủ, cập nhật số lượng tồn kho, giảm đi theo số album đã đặt.
5. Tạo đơn đặt hàng mới và lấy ID được tạo tự động của đơn hàng đó để trả về.
6. Commit transaction và trả về ID.

Ví dụ này sử dụng các phương thức `Tx` nhận tham số `context.Context`. Điều
này cho phép hủy quá trình thực thi hàm, bao gồm cả các thao tác cơ sở dữ
liệu, nếu nó chạy quá lâu hoặc kết nối client đóng lại. Để tìm hiểu thêm, xem
[Hủy các thao tác đang thực hiện](/doc/database/cancel-operations).

```
// CreateOrder creates an order for an album and returns the new order ID.
func CreateOrder(ctx context.Context, albumID, quantity, custID int) (orderID int64, err error) {

	// Create a helper function for preparing failure results.
	fail := func(err error) (int64, error) {
		return 0, fmt.Errorf("CreateOrder: %v", err)
	}

	// Get a Tx for making transaction requests.
	tx, err := db.BeginTx(ctx, nil)
	if err != nil {
		return fail(err)
	}
	// Defer a rollback in case anything fails.
	defer tx.Rollback()

	// Confirm that album inventory is enough for the order.
	var enough bool
	if err = tx.QueryRowContext(ctx, "SELECT (quantity >= ?) from album where id = ?",
		quantity, albumID).Scan(&enough); err != nil {
		if err == sql.ErrNoRows {
			return fail(fmt.Errorf("no such album"))
		}
		return fail(err)
	}
	if !enough {
		return fail(fmt.Errorf("not enough inventory"))
	}

	// Update the album inventory to remove the quantity in the order.
	_, err = tx.ExecContext(ctx, "UPDATE album SET quantity = quantity - ? WHERE id = ?",
		quantity, albumID)
	if err != nil {
		return fail(err)
	}

	// Create a new row in the album_order table.
	result, err := tx.ExecContext(ctx, "INSERT INTO album_order (album_id, cust_id, quantity, date) VALUES (?, ?, ?, ?)",
		albumID, custID, quantity, time.Now())
	if err != nil {
		return fail(err)
	}
	// Get the ID of the order item just created.
	orderID, err = result.LastInsertId()
	if err != nil {
		return fail(err)
	}

	// Commit the transaction.
	if err = tx.Commit(); err != nil {
		return fail(err)
	}

	// Return the order ID.
	return orderID, nil
}
```
