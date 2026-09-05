<!--{
  "Title": "Quản lý kết nối"
}-->

Với đại đa số chương trình, bạn không cần điều chỉnh các giá trị mặc định của
pool kết nối `sql.DB`. Nhưng với một số chương trình nâng cao, bạn có thể cần
tinh chỉnh các tham số pool kết nối hoặc làm việc trực tiếp với các kết nối.
Chủ đề này giải thích cách thực hiện điều đó.

Database handle [`sql.DB`](https://pkg.go.dev/database/sql#DB) an toàn khi
được nhiều goroutine sử dụng đồng thời (nghĩa là handle này tương đương với
khái niệm "thread-safe" trong các ngôn ngữ khác). Một số thư viện truy cập cơ
sở dữ liệu khác dựa trên các kết nối chỉ có thể dùng cho một thao tác tại một
thời điểm. Để lấp đầy khoảng cách đó, mỗi `sql.DB` quản lý một pool các kết
nối đang hoạt động đến cơ sở dữ liệu bên dưới, tạo kết nối mới khi cần để
đáp ứng tính song song trong chương trình Go của bạn.

Pool kết nối phù hợp với hầu hết nhu cầu truy cập dữ liệu. Khi bạn gọi một
phương thức `Query` hoặc `Exec` của `sql.DB`, cài đặt của `sql.DB` lấy một
kết nối còn trống từ pool hoặc tạo mới nếu cần. Gói sẽ trả kết nối về pool
khi nó không còn cần dùng nữa. Điều này hỗ trợ mức độ song song cao cho việc
truy cập cơ sở dữ liệu.

### Thiết lập thuộc tính pool kết nối {#connection_pool_properties}

Bạn có thể thiết lập các thuộc tính hướng dẫn cách gói `sql` quản lý pool kết
nối. Để lấy thống kê về hiệu ứng của các thuộc tính này, sử dụng
[`DB.Stats`](https://pkg.go.dev/database/sql#DB.Stats).

#### Thiết lập số lượng kết nối mở tối đa {#max_open_connections}

[`DB.SetMaxOpenConns`](https://pkg.go.dev/database/sql#DB.SetMaxOpenConns)
đặt giới hạn số lượng kết nối mở. Khi vượt quá giới hạn này, các thao tác cơ
sở dữ liệu mới sẽ chờ cho đến khi một thao tác hiện có hoàn thành, lúc đó
`sql.DB` sẽ tạo thêm một kết nối. Theo mặc định, `sql.DB` tạo kết nối mới bất
cứ khi nào tất cả các kết nối hiện có đang được sử dụng mà lại cần thêm kết
nối.

Lưu ý rằng việc đặt giới hạn khiến việc sử dụng cơ sở dữ liệu tương tự như
việc lấy lock hoặc semaphore, với hệ quả là ứng dụng của bạn có thể bị deadlock
khi chờ kết nối cơ sở dữ liệu mới.

#### Thiết lập số lượng kết nối nhàn rỗi tối đa {#max_idle_connections}

[`DB.SetMaxIdleConns`](https://pkg.go.dev/database/sql#DB.SetMaxIdleConns)
thay đổi giới hạn số lượng kết nối nhàn rỗi tối đa mà `sql.DB` duy trì.

Khi một thao tác SQL hoàn thành trên một kết nối cơ sở dữ liệu nhất định,
kết nối đó thường không bị đóng ngay lập tức: ứng dụng có thể cần dùng lại
sớm, và giữ kết nối mở tránh phải kết nối lại cho thao tác tiếp theo. Theo
mặc định, `sql.DB` giữ hai kết nối nhàn rỗi tại bất kỳ thời điểm nào. Tăng
giới hạn này có thể tránh việc kết nối lại thường xuyên trong các chương trình
có mức độ song song cao.

#### Thiết lập thời gian tối đa một kết nối có thể nhàn rỗi {#max_idle_time}

[`DB.SetConnMaxIdleTime`](https://pkg.go.dev/database/sql#DB.SetConnMaxIdleTime)
đặt thời gian tối đa một kết nối có thể nhàn rỗi trước khi bị đóng. Điều này
khiến `sql.DB` đóng các kết nối đã nhàn rỗi lâu hơn khoảng thời gian nhất định.

Theo mặc định, khi một kết nối nhàn rỗi được thêm vào pool kết nối, nó sẽ ở
lại đó cho đến khi cần dùng lại. Khi sử dụng `DB.SetMaxIdleConns` để tăng số
lượng kết nối nhàn rỗi được phép trong các đợt hoạt động song song, việc kết
hợp với `DB.SetConnMaxIdleTime` có thể sắp xếp để giải phóng các kết nối đó
sau đó khi hệ thống nhàn rỗi.

#### Thiết lập thời gian sống tối đa của kết nối {#max_connection_lifetime}

Sử dụng
[`DB.SetConnMaxLifetime`](https://pkg.go.dev/database/sql#DB.SetConnMaxLifetime)
để đặt thời gian tối đa một kết nối có thể mở trước khi bị đóng.

Theo mặc định, một kết nối có thể được sử dụng và tái sử dụng trong thời gian
tùy ý, tùy thuộc vào các giới hạn được mô tả ở trên. Trong một số hệ thống,
chẳng hạn những hệ thống sử dụng máy chủ cơ sở dữ liệu cân bằng tải, việc
đảm bảo ứng dụng không dùng một kết nối cụ thể quá lâu mà không kết nối lại
có thể hữu ích.

### Sử dụng kết nối dành riêng {#dedicated_connections}

Gói `database/sql` bao gồm các hàm bạn có thể dùng khi cơ sở dữ liệu có thể
gán ý nghĩa ẩn cho một chuỗi thao tác được thực thi trên một kết nối cụ thể.

Ví dụ phổ biến nhất là transaction, thường bắt đầu bằng lệnh `BEGIN`, kết
thúc bằng lệnh `COMMIT` hoặc `ROLLBACK`, và bao gồm tất cả các lệnh được phát
ra trên kết nối giữa các lệnh đó trong transaction tổng thể. Với trường hợp
sử dụng này, hãy dùng hỗ trợ transaction của gói `sql`. Xem
[Thực thi transaction](/doc/database/execute-transactions).

Với các trường hợp sử dụng khác khi một chuỗi thao tác riêng lẻ phải được
thực thi trên cùng một kết nối, gói `sql` cung cấp các kết nối dành riêng.
[`DB.Conn`](https://pkg.go.dev/database/sql#DB.Conn) lấy một kết nối dành
riêng, là một [`sql.Conn`](https://pkg.go.dev/database/sql#Conn). `sql.Conn`
có các phương thức `BeginTx`, `ExecContext`, `PingContext`, `PrepareContext`,
`QueryContext` và `QueryRowContext` hoạt động giống các phương thức tương ứng
trên DB nhưng chỉ sử dụng kết nối dành riêng đó. Khi dùng xong kết nối dành
riêng, mã của bạn phải giải phóng nó bằng `Conn.Close`.
