<!--{
  "Title": "Mở database handle",
  "Breadcrumb": true
}-->

Gói [`database/sql`](https://pkg.go.dev/database/sql) đơn giản hóa việc truy
cập cơ sở dữ liệu bằng cách giảm nhu cầu quản lý kết nối thủ công. Không như
nhiều API truy cập dữ liệu khác, với `database/sql` bạn không cần mở kết nối
tường minh, thực hiện công việc, rồi đóng kết nối. Thay vào đó, mã của bạn
mở một database handle đại diện cho một pool kết nối, sau đó thực hiện các
thao tác truy cập dữ liệu qua handle đó, chỉ gọi phương thức `Close` khi cần
để giải phóng tài nguyên, chẳng hạn tài nguyên được giữ bởi các hàng dữ liệu
đã lấy về hoặc một prepared statement.

Nói cách khác, chính database handle được đại diện bởi
[`sql.DB`](https://pkg.go.dev/database/sql#DB) xử lý các kết nối, mở và đóng
chúng thay mặt mã của bạn. Khi mã của bạn dùng handle để thực hiện các thao
tác cơ sở dữ liệu, các thao tác đó có thể truy cập đồng thời vào cơ sở dữ
liệu. Để tìm hiểu thêm, xem
[Quản lý kết nối](/doc/database/manage-connections).

**Lưu ý:** Bạn cũng có thể đặt trước một kết nối cơ sở dữ liệu. Để biết thêm
thông tin, xem
[Sử dụng kết nối dành riêng](/doc/database/manage-connections#dedicated_connections).

Ngoài các API có sẵn trong gói `database/sql`, cộng đồng Go đã phát triển
driver cho tất cả các hệ quản trị cơ sở dữ liệu phổ biến nhất (và nhiều hệ
ít phổ biến hơn).

Khi mở một database handle, bạn thực hiện các bước tổng quát sau:

1. Tìm driver.

    Driver dịch các yêu cầu và phản hồi giữa mã Go của bạn và cơ sở dữ liệu.
    Để tìm hiểu thêm, xem
    [Tìm và nhập driver cơ sở dữ liệu](#database_driver).

2. Mở database handle.

    Sau khi nhập driver, bạn có thể mở một handle cho cơ sở dữ liệu cụ thể.
    Để tìm hiểu thêm, xem [Mở database handle](#opening_handle).

3. Xác nhận kết nối.

    Sau khi mở database handle, mã của bạn có thể kiểm tra xem kết nối có
    khả dụng không. Để tìm hiểu thêm, xem
    [Xác nhận kết nối](#confirm_connection).

Mã của bạn thường không mở hoặc đóng kết nối cơ sở dữ liệu một cách tường
minh, điều đó được thực hiện bởi database handle. Tuy nhiên, mã của bạn nên
giải phóng các tài nguyên đã lấy về trong quá trình, chẳng hạn một `sql.Rows`
chứa kết quả truy vấn. Để tìm hiểu thêm, xem
[Giải phóng tài nguyên](#free_resources).

### Tìm và nhập driver cơ sở dữ liệu {#database_driver}

Bạn sẽ cần một driver cơ sở dữ liệu hỗ trợ DBMS bạn đang dùng. Để tìm driver
cho cơ sở dữ liệu của mình, xem [SQLDrivers](/wiki/SQLDrivers).

Để driver có thể dùng được trong mã, bạn nhập nó như bất kỳ gói Go nào khác.
Đây là một ví dụ:

```
import "github.com/go-sql-driver/mysql"
```

Lưu ý rằng nếu bạn không gọi trực tiếp bất kỳ hàm nào từ gói driver, chẳng
hạn khi nó được gói `sql` sử dụng ngầm, bạn cần dùng blank import, với tiền
tố dấu gạch dưới trước đường dẫn import:

```
import _ "github.com/go-sql-driver/mysql"
```

**Lưu ý:** Theo thực hành tốt nhất, hãy tránh dùng API riêng của driver cơ sở
dữ liệu cho các thao tác cơ sở dữ liệu. Thay vào đó, hãy dùng các hàm trong
gói `database/sql`. Điều này giúp mã của bạn liên kết lỏng lẻo với DBMS, dễ
dàng chuyển sang DBMS khác khi cần.

### Mở database handle {#opening_handle}

Database handle `sql.DB` cung cấp khả năng đọc và ghi vào cơ sở dữ liệu, dù
là riêng lẻ hay trong một transaction.

Bạn có thể lấy database handle bằng cách gọi `sql.Open` (nhận một connection
string) hoặc `sql.OpenDB` (nhận một `driver.Connector`). Cả hai đều trả về con
trỏ đến [`sql.DB`](https://pkg.go.dev/database/sql#DB).

**Lưu ý:** Hãy đảm bảo giữ thông tin xác thực cơ sở dữ liệu của bạn khỏi mã
nguồn Go. Để tìm hiểu thêm, xem
[Lưu trữ thông tin xác thực cơ sở dữ liệu](#store_credentials).

#### Mở với connection string {#open_connection_string}

Sử dụng [hàm `sql.Open`](https://pkg.go.dev/database/sql#Open) khi bạn muốn
kết nối bằng connection string. Định dạng của chuỗi này thay đổi tùy theo
driver bạn đang dùng.

Đây là ví dụ cho MySQL:

```
db, err = sql.Open("mysql", "username:password@tcp(127.0.0.1:3306)/jazzrecords")
if err != nil {
	log.Fatal(err)
}
```

Tuy nhiên, bạn có thể thấy rằng việc lưu trữ các thuộc tính kết nối theo
cách có cấu trúc hơn sẽ cho ra mã dễ đọc hơn. Chi tiết sẽ thay đổi tùy theo
driver.

Ví dụ, bạn có thể thay thế ví dụ trên bằng đoạn mã sau, sử dụng
[`Config`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config) của MySQL
driver để chỉ định các thuộc tính và phương thức
[`FormatDSN`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config.FormatDSN)
để tạo connection string.

```
// Specify connection properties.
cfg := mysql.NewConfig()
cfg.User = username
cfg.Passwd = password
cfg.Net = "tcp"
cfg.Addr = "127.0.0.1:3306"
cfg.DBName = "jazzrecords"

// Get a database handle.
db, err = sql.Open("mysql", cfg.FormatDSN())
if err != nil {
	log.Fatal(err)
}
```

#### Mở với Connector {#open_connector}

Sử dụng [hàm `sql.OpenDB`](https://pkg.go.dev/database/sql#OpenDB) khi bạn
muốn tận dụng các tính năng kết nối đặc thù của driver không có sẵn trong
connection string. Mỗi driver hỗ trợ tập hợp thuộc tính kết nối riêng của nó,
thường cung cấp các cách tùy chỉnh yêu cầu kết nối đặc trưng cho DBMS.

Điều chỉnh ví dụ `sql.Open` trước đó để dùng `sql.OpenDB`, bạn có thể tạo
handle với mã như sau:

```
// Specify connection properties.
cfg := mysql.NewConfig()
cfg.User = username
cfg.Passwd = password
cfg.Net = "tcp"
cfg.Addr = "127.0.0.1:3306"
cfg.DBName = "jazzrecords"

// Get a driver-specific connector.
connector, err := mysql.NewConnector(&cfg)
if err != nil {
	log.Fatal(err)
}

// Get a database handle.
db = sql.OpenDB(connector)
```

#### Xử lý lỗi {#handle_errors}

Mã của bạn nên kiểm tra lỗi từ việc tạo handle, chẳng hạn với `sql.Open`.
Đây sẽ không phải là lỗi kết nối. Thay vào đó, bạn sẽ nhận lỗi nếu `sql.Open`
không thể khởi tạo handle. Điều này có thể xảy ra, ví dụ, nếu nó không thể
phân tích cú pháp DSN bạn đã chỉ định.

### Xác nhận kết nối {#confirm_connection}

Khi bạn mở một database handle, gói `sql` có thể không tạo kết nối cơ sở dữ
liệu mới ngay lập tức. Thay vào đó, nó có thể tạo kết nối khi mã của bạn cần.
Nếu bạn không dùng cơ sở dữ liệu ngay và muốn xác nhận rằng có thể thiết lập
kết nối, hãy gọi
[`Ping`](https://pkg.go.dev/database/sql#DB.Ping) hoặc
[`PingContext`](https://pkg.go.dev/database/sql#DB.PingContext).

Đoạn mã trong ví dụ dưới đây ping cơ sở dữ liệu để xác nhận kết nối.

```
db, err = sql.Open("mysql", connString)

// Confirm a successful connection.
if err := db.Ping(); err != nil {
	log.Fatal(err)
}
```

### Lưu trữ thông tin xác thực cơ sở dữ liệu {#store_credentials}

Hãy tránh lưu trữ thông tin xác thực cơ sở dữ liệu trong mã Go của bạn, điều
đó có thể làm lộ nội dung cơ sở dữ liệu cho người khác. Thay vào đó, hãy tìm
cách lưu trữ chúng ở một vị trí nằm ngoài mã nhưng mã vẫn có thể truy cập.
Ví dụ, hãy cân nhắc một ứng dụng quản lý bí mật lưu trữ thông tin xác thực
và cung cấp API mà mã của bạn có thể dùng để lấy thông tin xác thực khi xác
thực với DBMS.

Một cách tiếp cận phổ biến là lưu trữ các bí mật trong biến môi trường trước
khi chương trình khởi động, có thể được tải từ một secret manager, sau đó
chương trình Go của bạn có thể đọc chúng bằng
[`os.Getenv`](https://pkg.go.dev/os#Getenv):

```
username := os.Getenv("DB_USER")
password := os.Getenv("DB_PASS")
```

Cách tiếp cận này cũng cho phép bạn tự đặt các biến môi trường khi kiểm tra
cục bộ.

### Giải phóng tài nguyên {#free_resources}

Mặc dù bạn không quản lý hay đóng kết nối một cách tường minh với gói
`database/sql`, mã của bạn nên giải phóng các tài nguyên đã lấy khi chúng
không còn cần thiết. Các tài nguyên đó có thể bao gồm tài nguyên được giữ bởi
`sql.Rows` đại diện cho dữ liệu trả về từ truy vấn hoặc `sql.Stmt` đại diện
cho một prepared statement.

Thông thường, bạn đóng tài nguyên bằng cách defer một lời gọi đến hàm `Close`
để tài nguyên được giải phóng trước khi hàm bao quanh thoát ra.

Đoạn mã trong ví dụ dưới đây defer `Close` để giải phóng tài nguyên được giữ
bởi [`sql.Rows`](https://pkg.go.dev/database/sql#Rows).

```
rows, err := db.Query("SELECT * FROM album WHERE artist = ?", artist)
if err != nil {
	log.Fatal(err)
}
defer rows.Close()

// Loop through returned rows.
```
