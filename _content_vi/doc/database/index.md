<!--{
  "Title": "Truy cập cơ sở dữ liệu quan hệ",
  "Breadcrumb": true
}-->

Sử dụng Go, bạn có thể tích hợp nhiều loại cơ sở dữ liệu và phương thức truy
cập dữ liệu khác nhau vào ứng dụng của mình. Các chủ đề trong phần này mô tả
cách sử dụng gói
[`database/sql`](https://pkg.go.dev/database/sql) trong thư viện chuẩn để truy
cập cơ sở dữ liệu quan hệ.

Để có hướng dẫn nhập môn về truy cập dữ liệu với Go, xem
[Hướng dẫn: Truy cập cơ sở dữ liệu quan hệ](/doc/tutorial/database-access).

Go cũng hỗ trợ các công nghệ truy cập dữ liệu khác, bao gồm các thư viện ORM
để truy cập cơ sở dữ liệu quan hệ ở mức trừu tượng cao hơn, và cả các kho lưu
trữ dữ liệu NoSQL phi quan hệ.

*   **Thư viện object-relational mapping (ORM).** Trong khi gói `database/sql`
    bao gồm các hàm cho logic truy cập dữ liệu ở mức thấp, bạn cũng có thể
    dùng Go để truy cập kho dữ liệu ở mức trừu tượng cao hơn. Để tìm hiểu
    thêm về hai thư viện object-relational mapping (ORM) phổ biến cho Go, xem
    [GORM](https://gorm.io/index.html)
    ([tài liệu gói](https://pkg.go.dev/gorm.io/gorm)) và
    [ent](https://entgo.io/) ([tài liệu gói](https://pkg.go.dev/entgo.io/ent)).
*   **Kho lưu trữ dữ liệu NoSQL.** Cộng đồng Go đã phát triển driver cho phần
    lớn các kho lưu trữ dữ liệu NoSQL, bao gồm
    [MongoDB](https://docs.mongodb.com/drivers/go/) và
    [Couchbase](https://docs.couchbase.com/go-sdk/current/hello-world/overview.html).
    Bạn có thể tìm kiếm thêm tại [pkg.go.dev](https://pkg.go.dev/).

### Các hệ quản trị cơ sở dữ liệu được hỗ trợ {#supported_dbms}

Go hỗ trợ tất cả các hệ quản trị cơ sở dữ liệu quan hệ phổ biến nhất, bao
gồm MySQL, Oracle, Postgres, SQL Server, SQLite và nhiều hệ khác.

Bạn sẽ tìm thấy danh sách driver đầy đủ tại trang
[SQLDrivers](/wiki/SQLDrivers).

### Các hàm thực thi truy vấn hoặc thay đổi cơ sở dữ liệu {#functions}

Gói `database/sql` bao gồm các hàm được thiết kế đặc biệt cho từng loại thao
tác cơ sở dữ liệu. Ví dụ, trong khi bạn có thể dùng `Query` hoặc `QueryRow`
để thực thi truy vấn, `QueryRow` được thiết kế cho trường hợp bạn chỉ mong
đợi một hàng duy nhất, bỏ qua chi phí trả về `sql.Rows` chỉ chứa một hàng.
Bạn có thể dùng hàm `Exec` để thay đổi cơ sở dữ liệu với các câu lệnh SQL
như `INSERT`, `UPDATE` hoặc `DELETE`.

Để tìm hiểu thêm, xem:

*   [Thực thi các câu lệnh SQL không trả về dữ liệu](/doc/database/change-data)
*   [Truy vấn dữ liệu](/doc/database/querying)

### Transaction {#transactions}

Thông qua `sql.Tx`, bạn có thể viết mã để thực thi các thao tác cơ sở dữ liệu
trong một transaction. Trong một transaction, nhiều thao tác có thể được thực
hiện cùng nhau và kết thúc bằng một lần commit cuối cùng, áp dụng tất cả các
thay đổi trong một bước nguyên tử, hoặc rollback để hủy bỏ chúng.

Để tìm hiểu thêm về transaction, xem
[Thực thi transaction](/doc/database/execute-transactions).

### Hủy truy vấn {#query_cancellation}

Bạn có thể dùng `context.Context` khi muốn có khả năng hủy một thao tác cơ
sở dữ liệu, chẳng hạn khi kết nối của client đóng lại hoặc thao tác chạy lâu
hơn mong muốn.

Với bất kỳ thao tác cơ sở dữ liệu nào, bạn có thể sử dụng hàm trong gói
`database/sql` nhận `Context` làm tham số. Thông qua `Context`, bạn có thể chỉ
định thời gian chờ hoặc thời hạn cho thao tác. Bạn cũng có thể dùng `Context`
để lan truyền yêu cầu hủy qua ứng dụng đến hàm đang thực thi câu lệnh SQL,
đảm bảo tài nguyên được giải phóng khi không còn cần thiết nữa.

Để tìm hiểu thêm, xem
[Hủy các thao tác đang thực hiện](/doc/database/cancel-operations).

### Pool kết nối được quản lý {#connection_pool}

Khi bạn sử dụng database handle `sql.DB`, bạn đang kết nối với một pool kết
nối tích hợp sẵn, pool này tạo và giải phóng các kết nối theo nhu cầu của mã.
Sử dụng handle qua `sql.DB` là cách phổ biến nhất để truy cập cơ sở dữ liệu
với Go. Để tìm hiểu thêm, xem
[Mở database handle](/doc/database/open-handle).

Gói `database/sql` quản lý pool kết nối cho bạn. Tuy nhiên, với các nhu cầu
nâng cao hơn, bạn có thể thiết lập các thuộc tính pool kết nối như mô tả trong
[Thiết lập thuộc tính pool kết nối](/doc/database/manage-connections#connection_pool_properties).

Với những thao tác cần một kết nối dành riêng duy nhất, gói `database/sql`
cung cấp [`sql.Conn`](https://pkg.go.dev/database/sql#Conn). `Conn` đặc biệt
hữu ích khi sử dụng transaction với `sql.Tx` là lựa chọn không phù hợp.

Ví dụ, mã của bạn có thể cần:

*   Thực hiện thay đổi schema thông qua DDL, bao gồm logic chứa ngữ nghĩa
    transaction riêng của nó. Việc kết hợp các hàm transaction của gói `sql`
    với các câu lệnh SQL transaction là thực hành không tốt, như mô tả trong
    [Thực thi transaction](/doc/database/execute-transactions).
*   Thực hiện các thao tác khóa truy vấn tạo bảng tạm thời.

Để tìm hiểu thêm, xem
[Sử dụng kết nối dành riêng](/doc/database/manage-connections#dedicated_connections).
