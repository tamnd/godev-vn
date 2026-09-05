<!--{
  "Title": "Hướng dẫn: Truy cập cơ sở dữ liệu quan hệ",
  "Breadcrumb": true
}-->

Hướng dẫn này giới thiệu những kiến thức cơ bản về truy cập cơ sở dữ liệu quan hệ
bằng Go và gói `database/sql` trong thư viện chuẩn.

Bạn sẽ tận dụng tối đa hướng dẫn này nếu đã có hiểu biết cơ bản về
Go và các công cụ đi kèm. Nếu đây là lần đầu tiên bạn tiếp xúc với Go, hãy xem
[Hướng dẫn: Bắt đầu với Go](/doc/tutorial/getting-started)
để có phần giới thiệu nhanh.

Gói [`database/sql`](https://pkg.go.dev/database/sql) bạn sẽ
sử dụng bao gồm các kiểu dữ liệu và hàm để kết nối cơ sở dữ liệu, thực thi
giao dịch, hủy một thao tác đang chạy và nhiều tính năng khác. Để biết thêm chi tiết
về cách sử dụng gói này, xem
[Truy cập cơ sở dữ liệu](/doc/database/index).

Trong hướng dẫn này, bạn sẽ tạo một cơ sở dữ liệu, sau đó viết code để truy cập
cơ sở dữ liệu đó. Dự án ví dụ sẽ là một kho lưu trữ dữ liệu về các đĩa nhạc jazz cổ điển.

Trong hướng dẫn này, bạn sẽ thực hiện lần lượt các phần sau:

1. Tạo thư mục cho code của bạn.
2. Cài đặt cơ sở dữ liệu.
3. Import driver cơ sở dữ liệu.
4. Lấy database handle và kết nối.
5. Truy vấn nhiều hàng.
6. Truy vấn một hàng duy nhất.
7. Thêm dữ liệu.

**Lưu ý:** Để xem các hướng dẫn khác, truy cập [Hướng dẫn](/doc/tutorial/index.html).

## Điều kiện tiên quyết {#prerequisites}

*   **Đã cài đặt hệ quản trị cơ sở dữ liệu quan hệ [MySQL](https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/) (DBMS).**
*   **Đã cài đặt Go.** Để biết hướng dẫn cài đặt, xem
    [Cài đặt Go](/doc/install).
*   **Một công cụ để chỉnh sửa code.** Bất kỳ trình soạn thảo văn bản nào bạn có đều dùng được.
*   **Một cửa sổ dòng lệnh.** Go hoạt động tốt trên bất kỳ terminal nào trên Linux và Mac,
    cũng như trên PowerShell hoặc cmd trong Windows.

## Tạo thư mục cho code của bạn {#create_folder}

Để bắt đầu, hãy tạo một thư mục cho code bạn sẽ viết.

1. Mở dấu nhắc lệnh và chuyển đến thư mục home của bạn.

    Trên Linux hoặc Mac:

    ```
    $ cd
    ```

    Trên Windows:

    ```
    C:\> cd %HOMEPATH%
    ```

    Trong phần còn lại của hướng dẫn, chúng ta sẽ dùng $ làm dấu nhắc lệnh. Các
    lệnh được sử dụng cũng hoạt động trên Windows.

2. Từ dấu nhắc lệnh, tạo một thư mục có tên
    data-access.

    ```
    $ mkdir data-access
    $ cd data-access
    ```


3. Tạo một module để quản lý các dependency bạn sẽ thêm vào trong
    hướng dẫn này.

    Chạy lệnh `go mod init`, cung cấp đường dẫn module cho code mới của bạn.

    ```
    $ go mod init example/data-access
    go: creating new go.mod: module example/data-access
    ```

    Lệnh này tạo ra file go.mod, trong đó các dependency bạn thêm vào sẽ được
    liệt kê để theo dõi. Để biết thêm, hãy xem
    [Quản lý dependency](/doc/modules/managing-dependencies).

    **Lưu ý:** Trong quá trình phát triển thực tế, bạn sẽ chỉ định đường dẫn module
    cụ thể hơn theo nhu cầu của mình. Để biết thêm, xem
    [Quản lý dependency](/doc/modules/managing-dependencies#naming_module).

Tiếp theo, bạn sẽ tạo một cơ sở dữ liệu.

## Cài đặt cơ sở dữ liệu {#set_up_database}

Trong bước này, bạn sẽ tạo cơ sở dữ liệu mà mình sẽ làm việc. Bạn sẽ dùng
CLI của chính DBMS để tạo cơ sở dữ liệu và bảng, cũng như thêm dữ liệu.

Bạn sẽ tạo một cơ sở dữ liệu với dữ liệu về các bản ghi nhạc jazz cổ điển trên đĩa vinyl.

Code ở đây sử dụng [MySQL CLI](https://dev.mysql.com/doc/refman/8.0/en/mysql.html),
nhưng hầu hết các DBMS đều có CLI riêng với các tính năng tương tự.

1. Mở một cửa sổ dòng lệnh mới.
2. Tại dòng lệnh, đăng nhập vào DBMS của bạn, như ví dụ dưới đây cho
    MySQL.

    ```
    $ mysql -u root -p
    Enter password:

    mysql>
    ```

3. Tại dấu nhắc lệnh `mysql`, tạo một cơ sở dữ liệu.

    ```
    mysql> create database recordings;
    ```

4. Chuyển sang cơ sở dữ liệu vừa tạo để thêm bảng.

    ```
    mysql> use recordings;
    Database changed
    ```

5. Trong trình soạn thảo văn bản, trong thư mục data-access, tạo một file
    có tên create-tables.sql để chứa script SQL cho việc thêm bảng.
6. Dán đoạn code SQL sau vào file, rồi lưu lại.

    ```
    DROP TABLE IF EXISTS album;
    CREATE TABLE album (
      id         INT AUTO_INCREMENT NOT NULL,
      title      VARCHAR(128) NOT NULL,
      artist     VARCHAR(255) NOT NULL,
      price      DECIMAL(5,2) NOT NULL,
      PRIMARY KEY (`id`)
    );

    INSERT INTO album
      (title, artist, price)
    VALUES
      ('Blue Train', 'John Coltrane', 56.99),
      ('Giant Steps', 'John Coltrane', 63.99),
      ('Jeru', 'Gerry Mulligan', 17.99),
      ('Sarah Vaughan', 'Sarah Vaughan', 34.98);
    ```

    Trong đoạn SQL này, bạn:

    *   Xóa (drop) bảng có tên `album` nếu tồn tại. Thực thi lệnh này trước
        giúp bạn dễ dàng chạy lại script nếu muốn bắt đầu lại từ đầu.

    *   Tạo bảng `album` với bốn cột: `title`, `artist` và `price`.
        Giá trị `id` của mỗi hàng được DBMS tự động tạo ra.

    *   Thêm bốn hàng với các giá trị.

7. Từ dấu nhắc lệnh `mysql`, chạy script vừa tạo.

    Bạn sẽ dùng lệnh `source` theo dạng sau:

    ```
    mysql> source /path/to/create-tables.sql
    ```

8. Tại dấu nhắc lệnh DBMS, dùng câu lệnh `SELECT` để xác nhận bạn đã
    tạo thành công bảng với dữ liệu.

    ```
    mysql> select * from album;
    +----+---------------+----------------+-------+
    | id | title         | artist         | price |
    +----+---------------+----------------+-------+
    |  1 | Blue Train    | John Coltrane  | 56.99 |
    |  2 | Giant Steps   | John Coltrane  | 63.99 |
    |  3 | Jeru          | Gerry Mulligan | 17.99 |
    |  4 | Sarah Vaughan | Sarah Vaughan  | 34.98 |
    +----+---------------+----------------+-------+
    4 rows in set (0.00 sec)
    ```

Tiếp theo, bạn sẽ viết một ít code Go để kết nối và truy vấn.

## Tìm và import driver cơ sở dữ liệu {#import_driver}

Bây giờ bạn đã có một cơ sở dữ liệu với một số dữ liệu, hãy bắt đầu viết code Go.

Tìm và import một driver cơ sở dữ liệu, driver này sẽ dịch các yêu cầu bạn thực hiện
thông qua các hàm trong gói `database/sql` thành các yêu cầu mà cơ sở dữ liệu hiểu.

1. Trong trình duyệt, truy cập trang wiki [SQLDrivers](/wiki/SQLDrivers)
    để xác định driver bạn có thể sử dụng.

    Dùng danh sách trên trang để xác định driver bạn sẽ dùng. Để truy cập
    MySQL trong hướng dẫn này, bạn sẽ sử dụng
    [Go-MySQL-Driver](https://github.com/go-sql-driver/mysql/).

2. Ghi lại tên gói của driver, ở đây là `github.com/go-sql-driver/mysql`.

3. Dùng trình soạn thảo văn bản, tạo một file để viết code Go của bạn và
    lưu file dưới tên main.go trong thư mục data-access đã tạo trước đó.

4. Dán đoạn code sau vào main.go để import gói driver.

    ```
    package main

    import "github.com/go-sql-driver/mysql"
    ```

    Trong đoạn code này, bạn:

    *   Đặt code vào gói `main` để có thể thực thi độc lập.

    *   Import driver MySQL `github.com/go-sql-driver/mysql`.

Sau khi đã import driver, bạn sẽ bắt đầu viết code để truy cập cơ sở dữ liệu.

## Lấy database handle và kết nối {#get_handle}

Bây giờ hãy viết một ít code Go để truy cập cơ sở dữ liệu thông qua database handle.

Bạn sẽ dùng một con trỏ đến struct `sql.DB`, đại diện cho quyền truy cập vào
một cơ sở dữ liệu cụ thể.

#### Viết code

1. Trong main.go, bên dưới đoạn code `import` vừa thêm, dán đoạn code Go
    sau để tạo database handle.

    ```
    var db *sql.DB

    func main() {
    	// Capture connection properties.
    	cfg := mysql.NewConfig()
    	cfg.User = os.Getenv("DBUSER")
    	cfg.Passwd = os.Getenv("DBPASS")
    	cfg.Net = "tcp"
    	cfg.Addr = "127.0.0.1:3306"
    	cfg.DBName = "recordings"

    	// Get a database handle.
    	var err error
    	db, err = sql.Open("mysql", cfg.FormatDSN())
    	if err != nil {
    		log.Fatal(err)
    	}

    	pingErr := db.Ping()
    	if pingErr != nil {
    		log.Fatal(pingErr)
    	}
    	fmt.Println("Connected!")
    }
    ```

    Trong đoạn code này, bạn:

    *   Khai báo biến `db` kiểu [`*sql.DB`](https://pkg.go.dev/database/sql#DB).
        Đây là database handle của bạn.

        Việc đặt `db` là biến toàn cục giúp đơn giản hóa ví dụ này. Trong môi trường
        production, bạn nên tránh dùng biến toàn cục, chẳng hạn bằng cách truyền
        biến vào các hàm cần nó hoặc bọc nó trong một struct.

    *   Dùng [`Config`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config)
        của driver MySQL và phương thức [`FormatDSN`](https://pkg.go.dev/github.com/go-sql-driver/mysql#Config.FormatDSN)
        của kiểu đó để thu thập các thuộc tính kết nối và định dạng chúng thành DSN cho chuỗi kết nối.

        Struct `Config` giúp code dễ đọc hơn so với chuỗi kết nối thông thường.

    *   Gọi [`sql.Open`](https://pkg.go.dev/database/sql#Open)
        để khởi tạo biến `db`, truyền vào giá trị trả về của
        `FormatDSN`.

    *   Kiểm tra lỗi từ `sql.Open`. Nó có thể thất bại nếu, ví dụ như,
        thông tin kết nối cơ sở dữ liệu của bạn không hợp lệ.

        Để đơn giản hóa code, bạn đang gọi `log.Fatal` để kết thúc
        chương trình và in lỗi ra console. Trong code production, bạn sẽ
        muốn xử lý lỗi theo cách linh hoạt hơn.

    *   Gọi [`DB.Ping`](https://pkg.go.dev/database/sql#DB.Ping) để
        xác nhận rằng việc kết nối đến cơ sở dữ liệu hoạt động. Khi chạy,
        `sql.Open` có thể không kết nối ngay lập tức tùy vào driver. Bạn dùng
        `Ping` ở đây để xác nhận rằng gói `database/sql` có thể kết nối khi cần.

    *   Kiểm tra lỗi từ `Ping`, trong trường hợp kết nối thất bại.

    *   In thông báo nếu `Ping` kết nối thành công.

2. Gần đầu file main.go, ngay bên dưới khai báo package,
    import các gói bạn cần để hỗ trợ code vừa viết.

    Đầu file lúc này trông như sau:

    ```
    package main

    import (
    	"database/sql"
    	"fmt"
    	"log"
    	"os"

    	"github.com/go-sql-driver/mysql"
    )
    ```

3. Lưu main.go.

#### Chạy code

1. Bắt đầu theo dõi module driver MySQL như một dependency.

    Dùng [`go get`](/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them)
    để thêm module github.com/go-sql-driver/mysql là dependency cho
    module của bạn. Dùng đối số dấu chấm để có nghĩa là "lấy dependency cho code trong thư
    mục hiện tại."

    ```
    $ go get .
    go: added filippo.io/edwards25519 v1.1.0
    go: added github.com/go-sql-driver/mysql v1.8.1
    ```

    Go đã tải dependency này vì bạn đã thêm nó vào khai báo `import`
    ở bước trước. Để biết thêm về theo dõi dependency, xem
    [Thêm dependency](/doc/modules/managing-dependencies#adding_dependency).

2. Từ dấu nhắc lệnh, đặt biến môi trường `DBUSER` và `DBPASS`
    để chương trình Go sử dụng.

    Trên Linux hoặc Mac:

    ```
    $ export DBUSER=username
    $ export DBPASS=password
    ```

    Trên Windows:

    ```
    C:\Users\you\data-access> set DBUSER=username
    C:\Users\you\data-access> set DBPASS=password
    ```

3. Từ dòng lệnh trong thư mục chứa main.go, chạy code bằng cách
    gõ `go run` với đối số dấu chấm để có nghĩa là "chạy gói trong thư
    mục hiện tại."

    ```
    $ go run .
    Connected!
    ```

Bạn có thể kết nối! Tiếp theo, bạn sẽ truy vấn một số dữ liệu.

## Truy vấn nhiều hàng {#multiple_rows}

Trong phần này, bạn sẽ dùng Go để thực thi một câu lệnh SQL được thiết kế để trả về
nhiều hàng.

Với các câu lệnh SQL có thể trả về nhiều hàng, bạn dùng phương thức `Query`
từ gói `database/sql`, sau đó lặp qua các hàng mà nó trả về. (Bạn sẽ
tìm hiểu cách truy vấn một hàng duy nhất ở phần sau, trong mục
[Truy vấn một hàng duy nhất](#single_row).)
#### Viết code

1. Trong main.go, ngay phía trên `func main`, dán phần định nghĩa
    struct `Album` sau. Bạn sẽ dùng nó để chứa dữ liệu hàng trả về từ truy vấn.

    ```
    type Album struct {
    	ID     int64
    	Title  string
    	Artist string
    	Price  float32
    }
    ```

2. Bên dưới `func main`, dán hàm `albumsByArtist` sau để truy vấn
    cơ sở dữ liệu.

    ```
    // albumsByArtist queries for albums that have the specified artist name.
    func albumsByArtist(name string) ([]Album, error) {
    	// An albums slice to hold data from returned rows.
    	var albums []Album

    	rows, err := db.Query("SELECT * FROM album WHERE artist = ?", name)
    	if err != nil {
    		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
    	}
    	defer rows.Close()
    	// Loop through rows, using Scan to assign column data to struct fields.
    	for rows.Next() {
    		var alb Album
    		if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
    			return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
    		}
    		albums = append(albums, alb)
    	}
    	if err := rows.Err(); err != nil {
    		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
    	}
    	return albums, nil
    }
    ```

    Trong đoạn code này, bạn:

    *   Khai báo một slice `albums` kiểu `Album` đã định nghĩa. Nó sẽ chứa
        dữ liệu từ các hàng trả về. Tên và kiểu của các trường struct tương ứng với
        tên và kiểu của các cột cơ sở dữ liệu.

    *   Dùng [`DB.Query`](https://pkg.go.dev/database/sql#DB.Query) để
        thực thi câu lệnh `SELECT` truy vấn các album với tên nghệ sĩ đã chỉ định.

        Tham số đầu tiên của `Query` là câu lệnh SQL. Sau tham số đó,
        bạn có thể truyền thêm không hoặc nhiều tham số kiểu bất kỳ. Chúng cung cấp
        chỗ để bạn chỉ định giá trị cho các tham số trong câu lệnh SQL.
        Bằng cách tách câu lệnh SQL khỏi giá trị tham số (thay vì
        nối chuỗi bằng `fmt.Sprintf` chẳng hạn), bạn cho phép gói
        `database/sql` gửi các giá trị tách biệt khỏi văn bản SQL,
        loại bỏ mọi rủi ro SQL injection.

    *   Trì hoãn việc đóng `rows` để mọi tài nguyên nó đang giữ sẽ được giải phóng khi
        hàm kết thúc.

    *   Lặp qua các hàng trả về, dùng
        [`Rows.Scan`](https://pkg.go.dev/database/sql#Rows.Scan) để
        gán giá trị cột của mỗi hàng vào các trường của struct `Album`.

        `Scan` nhận một danh sách các con trỏ tới các giá trị Go, nơi giá trị cột
        sẽ được ghi vào. Ở đây, bạn truyền các con trỏ đến các trường của biến
        `alb`, được tạo bằng toán tử `&`.
        `Scan` ghi thông qua các con trỏ để cập nhật các trường của struct.

    *   Bên trong vòng lặp, kiểm tra lỗi khi scan giá trị cột vào các
        trường struct.

    *   Bên trong vòng lặp, thêm `alb` mới vào slice `albums`.

    *   Sau vòng lặp, kiểm tra lỗi từ toàn bộ truy vấn, dùng
        `rows.Err`. Lưu ý rằng nếu bản thân truy vấn thất bại, kiểm tra lỗi
        ở đây là cách duy nhất để biết kết quả không đầy đủ.

3. Cập nhật hàm `main` để gọi `albumsByArtist`.

    Thêm đoạn code sau vào cuối `func main`.

    ```
    albums, err := albumsByArtist("John Coltrane")
    if err != nil {
    	log.Fatal(err)
    }
    fmt.Printf("Albums found: %v\n", albums)
    ```

    Trong code mới, bạn:

    *   Gọi hàm `albumsByArtist` đã thêm, gán giá trị trả về vào
        biến `albums` mới.

    *   In kết quả.

#### Chạy code

Từ dòng lệnh trong thư mục chứa main.go, chạy code.

```
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
```

Tiếp theo, bạn sẽ truy vấn một hàng duy nhất.

## Truy vấn một hàng duy nhất {#single_row}

Trong phần này, bạn sẽ dùng Go để truy vấn một hàng duy nhất trong cơ sở dữ liệu.

Với các câu lệnh SQL mà bạn biết sẽ trả về nhiều nhất một hàng, bạn có thể dùng
`QueryRow`, đơn giản hơn so với dùng vòng lặp `Query`.

#### Viết code

1. Bên dưới `albumsByArtist`, dán hàm `albumByID` sau.

    ```
    // albumByID queries for the album with the specified ID.
    func albumByID(id int64) (Album, error) {
    	// An album to hold data from the returned row.
    	var alb Album

    	row := db.QueryRow("SELECT * FROM album WHERE id = ?", id)
    	if err := row.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
    		if err == sql.ErrNoRows {
    			return alb, fmt.Errorf("albumsById %d: no such album", id)
    		}
    		return alb, fmt.Errorf("albumsById %d: %v", id, err)
    	}
    	return alb, nil
    }
    ```

    Trong đoạn code này, bạn:

    *   Dùng [`DB.QueryRow`](https://pkg.go.dev/database/sql#DB.QueryRow)
        để thực thi câu lệnh `SELECT` truy vấn album với ID đã chỉ định.

        Nó trả về một `sql.Row`. Để đơn giản hóa code gọi hàm
        (code của bạn!), `QueryRow` không trả về lỗi. Thay vào đó,
        nó sắp xếp để trả về bất kỳ lỗi truy vấn nào (chẳng hạn như `sql.ErrNoRows`)
        từ `Rows.Scan` sau đó.

    *   Dùng [`Row.Scan`](https://pkg.go.dev/database/sql#Row.Scan) để sao chép
        giá trị cột vào các trường struct.

    *   Kiểm tra lỗi từ `Scan`.

        Lỗi đặc biệt `sql.ErrNoRows` cho biết truy vấn không trả về
        hàng nào. Thông thường, lỗi đó nên được thay thế bằng văn bản cụ thể hơn,
        chẳng hạn như "no such album" ở đây.

2. Cập nhật `main` để gọi `albumByID`.

    Thêm đoạn code sau vào cuối `func main`.

    ```
    // Hard-code ID 2 here to test the query.
    alb, err := albumByID(2)
    if err != nil {
    	log.Fatal(err)
    }
    fmt.Printf("Album found: %v\n", alb)
    ```

    Trong code mới, bạn:

    *   Gọi hàm `albumByID` đã thêm.

    *   In album ID được trả về.

#### Chạy code

Từ dòng lệnh trong thư mục chứa main.go, chạy code.


```
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
```

Tiếp theo, bạn sẽ thêm một album vào cơ sở dữ liệu.

## Thêm dữ liệu {#add_data}

Trong phần này, bạn sẽ dùng Go để thực thi câu lệnh SQL `INSERT` để thêm
một hàng mới vào cơ sở dữ liệu.

Bạn đã thấy cách dùng `Query` và `QueryRow` với các câu lệnh SQL trả về dữ liệu.
Để thực thi các câu lệnh SQL _không_ trả về dữ liệu, bạn dùng `Exec`.

#### Viết code

1. Bên dưới `albumByID`, dán hàm `addAlbum` sau để chèn một
    album mới vào cơ sở dữ liệu, rồi lưu main.go.

    ```
    // addAlbum adds the specified album to the database,
    // returning the album ID of the new entry
    func addAlbum(alb Album) (int64, error) {
    	result, err := db.Exec("INSERT INTO album (title, artist, price) VALUES (?, ?, ?)", alb.Title, alb.Artist, alb.Price)
    	if err != nil {
    		return 0, fmt.Errorf("addAlbum: %v", err)
    	}
    	id, err := result.LastInsertId()
    	if err != nil {
    		return 0, fmt.Errorf("addAlbum: %v", err)
    	}
    	return id, nil
    }
    ```

    Trong đoạn code này, bạn:

    *   Dùng [`DB.Exec`](https://pkg.go.dev/database/sql#DB.Exec) để
        thực thi câu lệnh `INSERT`.

        Giống như `Query`, `Exec` nhận một câu lệnh SQL theo sau là
        các giá trị tham số cho câu lệnh SQL đó.

    *   Kiểm tra lỗi từ thao tác `INSERT`.

    *   Lấy ID của hàng cơ sở dữ liệu vừa chèn bằng
        [`Result.LastInsertId`](https://pkg.go.dev/database/sql#Result.LastInsertId).

    *   Kiểm tra lỗi khi lấy ID.

2. Cập nhật `main` để gọi hàm `addAlbum` mới.

    Thêm đoạn code sau vào cuối `func main`.

    ```
    albID, err := addAlbum(Album{
    	Title:  "The Modern Sound of Betty Carter",
    	Artist: "Betty Carter",
    	Price:  49.99,
    })
    if err != nil {
    	log.Fatal(err)
    }
    fmt.Printf("ID of added album: %v\n", albID)
    ```

    Trong code mới, bạn:

    *   Gọi `addAlbum` với một album mới, gán ID của album bạn đang
        thêm vào biến `albID`.

#### Chạy code

Từ dòng lệnh trong thư mục chứa main.go, chạy code.

```
$ go run .
Connected!
Albums found: [{1 Blue Train John Coltrane 56.99} {2 Giant Steps John Coltrane 63.99}]
Album found: {2 Giant Steps John Coltrane 63.99}
ID of added album: 5
```

## Kết luận {#conclusion}

Chúc mừng! Bạn vừa dùng Go để thực hiện các thao tác đơn giản với
cơ sở dữ liệu quan hệ.

Các chủ đề đề xuất tiếp theo:

*   Xem hướng dẫn truy cập dữ liệu, bao gồm nhiều thông tin hơn
    về các chủ đề chỉ được đề cập sơ qua ở đây.

*   Nếu bạn mới học Go, bạn sẽ tìm thấy các thực hành tốt hữu ích được mô tả trong
    [Effective Go](/doc/effective_go) và [Cách viết code Go](/doc/code).

*   [Go Tour](/tour/) là phần giới thiệu từng bước tuyệt vời
    về các khái niệm cơ bản của Go.

## Code hoàn chỉnh {#completed_code}

Phần này chứa code của ứng dụng bạn xây dựng trong hướng dẫn này.

```
package main

import (
	"database/sql"
	"fmt"
	"log"
	"os"

	"github.com/go-sql-driver/mysql"
)

var db *sql.DB

type Album struct {
	ID     int64
	Title  string
	Artist string
	Price  float32
}

func main() {
	// Capture connection properties.
	cfg := mysql.NewConfig()
	cfg.User = os.Getenv("DBUSER")
	cfg.Passwd = os.Getenv("DBPASS")
	cfg.Net = "tcp"
	cfg.Addr = "127.0.0.1:3306"
	cfg.DBName = "recordings"

	// Get a database handle.
	var err error
	db, err = sql.Open("mysql", cfg.FormatDSN())
	if err != nil {
		log.Fatal(err)
	}

	pingErr := db.Ping()
	if pingErr != nil {
		log.Fatal(pingErr)
	}
	fmt.Println("Connected!")

	albums, err := albumsByArtist("John Coltrane")
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Albums found: %v\n", albums)

	// Hard-code ID 2 here to test the query.
	alb, err := albumByID(2)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("Album found: %v\n", alb)

	albID, err := addAlbum(Album{
		Title:  "The Modern Sound of Betty Carter",
		Artist: "Betty Carter",
		Price:  49.99,
	})
	if err != nil {
		log.Fatal(err)
	}
	fmt.Printf("ID of added album: %v\n", albID)
}

// albumsByArtist queries for albums that have the specified artist name.
func albumsByArtist(name string) ([]Album, error) {
	// An albums slice to hold data from returned rows.
	var albums []Album

	rows, err := db.Query("SELECT * FROM album WHERE artist = ?", name)
	if err != nil {
		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
	}
	defer rows.Close()
	// Loop through rows, using Scan to assign column data to struct fields.
	for rows.Next() {
		var alb Album
		if err := rows.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
			return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
		}
		albums = append(albums, alb)
	}
	if err := rows.Err(); err != nil {
		return nil, fmt.Errorf("albumsByArtist %q: %v", name, err)
	}
	return albums, nil
}

// albumByID queries for the album with the specified ID.
func albumByID(id int64) (Album, error) {
	// An album to hold data from the returned row.
	var alb Album

	row := db.QueryRow("SELECT * FROM album WHERE id = ?", id)
	if err := row.Scan(&alb.ID, &alb.Title, &alb.Artist, &alb.Price); err != nil {
		if err == sql.ErrNoRows {
			return alb, fmt.Errorf("albumsById %d: no such album", id)
		}
		return alb, fmt.Errorf("albumsById %d: %v", id, err)
	}
	return alb, nil
}

// addAlbum adds the specified album to the database,
// returning the album ID of the new entry
func addAlbum(alb Album) (int64, error) {
	result, err := db.Exec("INSERT INTO album (title, artist, price) VALUES (?, ?, ?)", alb.Title, alb.Artist, alb.Price)
	if err != nil {
		return 0, fmt.Errorf("addAlbum: %v", err)
	}
	id, err := result.LastInsertId()
	if err != nil {
		return 0, fmt.Errorf("addAlbum: %v", err)
	}
	return id, nil
}
```
