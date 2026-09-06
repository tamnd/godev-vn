<!--{
  "Title": "Hướng dẫn: Phát triển API RESTful với Go và Gin",
  "Breadcrumb": true
}-->

This tutorial giới thiệu những kiến thức cơ bản về cách viết API dịch vụ web RESTful bằng Go và [Gin Web Framework](https://gin-gonic.com/en/docs/) (Gin).

Bạn sẽ khai thác tối đa hướng dẫn này nếu có kiến thức cơ bản về Go và hệ thống công cụ của Go. Nếu đây là lần đầu tiên bạn làm quen với Go, hãy xem [Hướng dẫn: Bắt đầu với Go](/doc/tutorial/getting-started) để có phần giới thiệu nhanh.

Gin đơn giản hóa nhiều tác vụ lập trình liên quan đến việc xây dựng ứng dụng web, bao gồm cả dịch vụ web. Trong hướng dẫn này, bạn sẽ sử dụng Gin để định tuyến yêu cầu, lấy thông tin yêu cầu và marshal JSON cho các phản hồi.

Trong hướng dẫn này, bạn sẽ xây dựng một máy chủ API RESTful với hai endpoint. Dự án ví dụ của bạn sẽ là một kho lưu trữ dữ liệu về các bản thu âm jazz cổ điển.

Hướng dẫn này bao gồm các phần sau:

1. Thiết kế các endpoint API.
2. Tạo một thư mục cho mã của bạn.
3. Tạo dữ liệu.
4. Viết một handler để trả về tất cả mục.
5. Viết một handler để thêm một mục mới.
6. Viết một handler để trả về một mục cụ thể.

**Lưu ý:** Để xem các hướng dẫn khác, hãy xem [Hướng dẫn](/doc/tutorial/index.html).

Để thử hướng dẫn tương tác này mà bạn hoàn thành trong Google Cloud Shell, hãy nhấp vào nút bên dưới.

[![Mở trong Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.png)](https://ide.cloud.google.com/?cloudshell_workspace=~&walkthrough_tutorial_url=https://raw.githubusercontent.com/golang/tour/master/tutorial/web-service-gin.md)

## Điều kiện tiên quyết

*   **Go.** Chúng tôi khuyên bạn nên sử dụng phiên bản Go mới nhất để làm theo hướng dẫn này.
    Để biết hướng dẫn cài đặt, hãy xem [Cài đặt Go](/doc/install).
*   **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt.
*   **Một terminal lệnh.** Go hoạt động tốt khi sử dụng bất kỳ terminal nào trên Linux và Mac,
    cũng như PowerShell hoặc cmd trên Windows.
*   **Công cụ curl.** Trên Linux và Mac, công cụ này thường đã được cài đặt. Trên
    Windows, công cụ này được tích hợp trong Windows 10 Insider build 17063 trở lên. Với các phiên bản
    Windows cũ hơn, bạn có thể cần cài đặt nó. Để biết thêm, hãy xem
    [Tar và Curl đến với Windows](https://docs.microsoft.com/en-us/virtualization/community/team-blog/2017/20171219-tar-and-curl-come-to-windows).

## Thiết kế các endpoint API {#design_endpoints}

Bạn sẽ xây dựng một API cung cấp quyền truy cập vào một cửa hàng bán các bản thu âm cổ điển trên đĩa vinyl. Vì vậy, bạn sẽ cần cung cấp các endpoint để client có thể lấy và thêm album cho người dùng.

Khi phát triển một API, thông thường bạn bắt đầu bằng việc thiết kế các endpoint. Người dùng API của bạn sẽ thành công hơn nếu các endpoint dễ hiểu.

Dưới đây là các endpoint bạn sẽ tạo trong hướng dẫn này.

/albums

- `GET` – Lấy danh sách tất cả album, được trả về dưới dạng JSON.
- `POST` – Thêm một album mới từ dữ liệu yêu cầu được gửi dưới dạng JSON.

/albums/:id

- `GET` – Lấy một album theo ID của nó, trả về dữ liệu album dưới dạng JSON.

Tiếp theo, bạn sẽ tạo một thư mục cho mã của mình.

## Tạo thư mục cho mã của bạn {#create_folder}

Để bắt đầu, hãy tạo một dự án cho mã bạn sẽ viết.

1. Mở dấu nhắc lệnh và chuyển đến thư mục chính của bạn.

Trên Linux hoặc Mac:

    ```
    $ cd
    ```

Trên Windows:

    ```
    C:\> cd %HOMEPATH%
    ```

2. Sử dụng dấu nhắc lệnh, tạo một thư mục cho mã của bạn có tên

web-service-gin.

    ```
    $ mkdir web-service-gin
    $ cd web-service-gin
    ```

3. Tạo một module để bạn có thể quản lý các dependency.

Chạy lệnh `go mod init`, cung cấp cho lệnh này đường dẫn của module mà mã

của bạn sẽ nằm trong đó.

    ```
    $ go mod init example/web-service-gin
    go: creating new go.mod: module example/web-service-gin
    ```

Lệnh này tạo một tệp go.mod trong đó các dependency bạn thêm vào sẽ được

liệt kê để theo dõi. Để biết thêm về cách đặt tên một module bằng đường dẫn

module, hãy xem [Quản lý dependency](/doc/modules/managing-dependencies#naming_module).

Tiếp theo, bạn sẽ thiết kế các cấu trúc dữ liệu để xử lý dữ liệu.

## Tạo dữ liệu {#create_data}

Để giữ mọi thứ đơn giản cho hướng dẫn này, bạn sẽ lưu trữ dữ liệu trong bộ nhớ. Một

API thông thường hơn sẽ tương tác với cơ sở dữ liệu.

Lưu ý rằng việc lưu trữ dữ liệu trong bộ nhớ có nghĩa là tập hợp album sẽ bị mất mỗi

khi bạn dừng máy chủ, sau đó được tạo lại khi bạn khởi động nó.

#### Viết mã

1. Sử dụng trình soạn thảo văn bản, tạo một tệp có tên main.go trong thư mục

web-service. Bạn sẽ viết mã Go của mình trong tệp này.
2. Trong main.go, ở đầu tệp, dán khai báo package sau.

    ```
    package main
    ```

Một chương trình độc lập (trái với một thư viện) luôn nằm trong package `main`.
3. Bên dưới khai báo package, dán khai báo sau của struct `album`. Bạn sẽ sử dụng

struct này để lưu dữ liệu album trong bộ nhớ.

Các thẻ struct như `json:"artist"` chỉ định tên của một trường nên là gì

khi nội dung của struct được tuần tự hóa thành JSON. Nếu không có chúng, JSON

sẽ sử dụng tên trường được viết hoa của struct – một kiểu không phổ biến bằng

trong JSON.

    ```
 // album đại diện cho dữ liệu về một album thu âm.
    type album struct {
    	ID     string  `json:"id"`
    	Title  string  `json:"title"`
    	Artist string  `json:"artist"`
    	Price  float64 `json:"price"`
    }
    ```

4. Bên dưới khai báo struct bạn vừa thêm, dán slice các struct `album` sau chứa

dữ liệu bạn sẽ dùng để khởi tạo.

    ```
 // slice albums để khởi tạo dữ liệu album thu âm.
    var albums = []album{
    	{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
    	{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
    	{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
    }
    ```

Tiếp theo, bạn sẽ viết mã để triển khai endpoint đầu tiên của mình.

## Viết một trình xử lý để trả về tất cả mục {#all_items}

Khi client gửi một yêu cầu tới `GET /albums`, bạn muốn trả về tất cả album dưới dạng JSON.

Để thực hiện việc này, bạn sẽ viết như sau:

*   Logic để chuẩn bị phản hồi
*   Mã để ánh xạ đường dẫn yêu cầu tới logic của bạn

Lưu ý rằng đây là thứ tự ngược lại với cách chúng sẽ được thực thi khi chạy, nhưng bạn đang thêm các dependency trước, sau đó là mã phụ thuộc vào chúng.

#### Viết mã

1. Bên dưới mã struct bạn đã thêm trong phần trước, dán đoạn mã sau để lấy danh sách album.

    Hàm `getAlbums` này tạo JSON từ slice các struct `album`, ghi JSON vào phản hồi.

    ```
    // getAlbums phản hồi bằng danh sách tất cả album dưới dạng JSON.
    func getAlbums(c *gin.Context) {
    	c.IndentedJSON(http.StatusOK, albums)
    }
    ```

    Trong đoạn mã này, bạn:

    *   Viết một hàm `getAlbums` nhận tham số
        [`gin.Context`](https://pkg.go.dev/github.com/gin-gonic/gin#Context).
        Lưu ý rằng bạn có thể đặt cho hàm này bất kỳ tên nào – cả
        Gin lẫn Go đều không yêu cầu một định dạng tên hàm cụ thể.

        `gin.Context` là phần quan trọng nhất của Gin. Nó chứa thông tin
        yêu cầu, xác thực và tuần tự hóa JSON, cùng nhiều chức năng khác.
        (Mặc dù có tên tương tự, nó khác với gói
        [`context`](/pkg/context/) tích hợp sẵn của Go.)

    *   Gọi [`Context.IndentedJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.IndentedJSON)
        để tuần tự hóa struct thành JSON và thêm nó vào phản hồi.

        Đối số đầu tiên của hàm là mã trạng thái HTTP bạn muốn gửi tới
        client. Ở đây, bạn truyền hằng số [`StatusOK`](https://pkg.go.dev/net/http#StatusOK)
        từ gói `net/http` để biểu thị `200 OK`.

        Lưu ý rằng bạn có thể thay thế `Context.IndentedJSON` bằng lời gọi tới
        [`Context.JSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.JSON)
        để gửi JSON nhỏ gọn hơn. Trong thực tế, dạng có thụt lề dễ làm việc
        hơn nhiều khi gỡ lỗi và chênh lệch kích thước thường nhỏ.

2. Gần đầu `main.go`, ngay bên dưới khai báo slice `albums`, dán
    đoạn mã dưới đây để gán hàm xử lý vào một đường dẫn endpoint.

    Việc này thiết lập một liên kết trong đó `getAlbums` xử lý các yêu cầu tới
    đường dẫn endpoint `/albums`.

    ```
    func main() {
    	router := gin.Default()
    	router.GET("/albums", getAlbums)

    	router.Run("localhost:8080")
    }
    ```

    Trong đoạn mã này, bạn:

    *   Khởi tạo một router Gin bằng cách sử dụng
        [`Default`](https://pkg.go.dev/github.com/gin-gonic/gin#Default).
    *   Sử dụng hàm [`GET`](https://pkg.go.dev/github.com/gin-gonic/gin#RouterGroup.GET)
        để liên kết phương thức HTTP `GET` và đường dẫn `/albums` với một hàm
        xử lý.

        Lưu ý rằng bạn đang truyền _tên_ của hàm `getAlbums`. Điều này
        khác với việc truyền _kết quả_ của hàm, việc bạn sẽ làm bằng cách
        truyền `getAlbums()` (lưu ý dấu ngoặc đơn).

    *   Sử dụng hàm [`Run`](https://pkg.go.dev/github.com/gin-gonic/gin#Engine.Run)
        để gắn router vào một `http.Server` và khởi động server.

3. Gần đầu `main.go`, ngay bên dưới khai báo package, import các
    package bạn cần để hỗ trợ đoạn mã vừa viết.

    Các dòng mã đầu tiên sẽ trông như sau:

    ```
    package main

    import (
    	"net/http"

    	"github.com/gin-gonic/gin"
    )
    ```

4. Lưu `main.go`.

#### Chạy mã

1. Bắt đầu theo dõi mô-đun Gin dưới dạng một dependency.

    Tại dòng lệnh, sử dụng [`go get`](/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them)
    để thêm mô-đun github.com/gin-gonic/gin làm dependency cho mô-đun của bạn.
    Sử dụng đối số dấu chấm để biểu thị "lấy dependency cho mã trong
    thư mục hiện tại."

    ```
    $ go get .
    go get: added github.com/gin-gonic/gin v1.7.2
    ```

    Go đã phân giải và tải xuống dependency này để đáp ứng khai báo `import`
    mà bạn đã thêm ở bước trước.

2. Từ dòng lệnh trong thư mục chứa main.go, chạy mã.
    Sử dụng đối số dấu chấm để biểu thị "chạy mã trong thư mục hiện tại."

    ```
    $ go run .
    ```

    Khi mã đang chạy, bạn có một máy chủ HTTP đang hoạt động để có thể
    gửi yêu cầu đến.

3. Từ một cửa sổ dòng lệnh mới, sử dụng `curl` để tạo một yêu cầu đến
    dịch vụ web đang chạy của bạn.

    ```
    $ curl http://localhost:8080/albums
    ```

    Lệnh này sẽ hiển thị dữ liệu mà bạn đã nạp sẵn vào dịch vụ.

    ```
    [
            {
                    "id": "1",
                    "title": "Blue Train",
                    "artist": "John Coltrane",
                    "price": 56.99
            },
            {
                    "id": "2",
                    "title": "Jeru",
                    "artist": "Gerry Mulligan",
                    "price": 17.99
            },
            {
                    "id": "3",
                    "title": "Sarah Vaughan and Clifford Brown",
                    "artist": "Sarah Vaughan",
                    "price": 39.99
            }
    ]
    ```

Bạn đã tạo một API! Trong phần tiếp theo, bạn sẽ tạo một endpoint khác với
mã để xử lý yêu cầu `POST` nhằm thêm một mục.

## Viết handler để thêm một mục mới {#add_item}

Khi client thực hiện yêu cầu `POST` tại `/albums`, bạn muốn thêm album
được mô tả trong phần thân yêu cầu vào dữ liệu album hiện có.

Để thực hiện việc này, bạn sẽ viết:

*   Logic để thêm album mới vào danh sách hiện có.
*   Một đoạn mã để định tuyến yêu cầu `POST` đến logic của bạn.

#### Viết mã

1. Thêm mã để thêm dữ liệu album vào danh sách album.

    Ở đâu đó sau các câu lệnh `import`, hãy dán đoạn mã sau. (Cuối tệp là
    một vị trí phù hợp cho đoạn mã này, nhưng Go không bắt buộc thứ tự mà bạn
    khai báo các hàm.)

    ```
    // postAlbums thêm một album từ JSON nhận được trong phần thân yêu cầu.
    func postAlbums(c *gin.Context) {
    	var newAlbum album

    	// Gọi BindJSON để liên kết JSON đã nhận với
    	// newAlbum.
    	if err := c.BindJSON(&newAlbum); err != nil {
    		return
    	}

    	// Thêm album mới vào slice.
    	albums = append(albums, newAlbum)
    	c.IndentedJSON(http.StatusCreated, newAlbum)
    }
    ```

    Trong đoạn mã này, bạn:

    *   Sử dụng [`Context.BindJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.BindJSON)
        để liên kết phần thân yêu cầu với `newAlbum`.
    *   Nối cấu trúc `album` được khởi tạo từ JSON vào slice `albums`.
    *   Thêm mã trạng thái `201` vào phản hồi, cùng với JSON biểu diễn
        album bạn đã thêm.

2. Thay đổi hàm `main` của bạn để bao gồm hàm `router.POST`, như sau.

    ```
    func main() {
    	router := gin.Default()
    	router.GET("/albums", getAlbums)
    	router.POST("/albums", postAlbums)

    	router.Run("localhost:8080")
    }
    ```

    Trong đoạn mã này, bạn:

    *   Liên kết phương thức `POST` tại đường dẫn `/albums` với hàm
        `postAlbums`.

        Với Gin, bạn có thể liên kết một trình xử lý với tổ hợp phương thức HTTP
        và đường dẫn. Bằng cách này, bạn có thể định tuyến riêng các yêu cầu gửi
        đến một đường dẫn duy nhất dựa trên phương thức mà máy khách đang sử dụng.

#### Chạy mã

1. Nếu máy chủ vẫn đang chạy từ phần trước, hãy dừng nó.
2. Từ dòng lệnh trong thư mục chứa main.go, chạy mã.

    ```
    $ go run .
    ```

3. Từ một cửa sổ dòng lệnh khác, sử dụng `curl` để tạo yêu cầu đến dịch vụ web
    đang chạy của bạn.

    ```
    $ curl http://localhost:8080/albums \
        --include \
        --header "Content-Type: application/json" \
        --request "POST" \
        --data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
    ```

    Lệnh này sẽ hiển thị các header và JSON cho album đã thêm.

    ```
    HTTP/1.1 201 Created
    Content-Type: application/json; charset=utf-8
    Date: Wed, 02 Jun 2021 00:34:12 GMT
    Content-Length: 116

    {
        "id": "4",
        "title": "The Modern Sound of Betty Carter",
        "artist": "Betty Carter",
        "price": 49.99
    }
    ```

4. Giống như phần trước, hãy sử dụng `curl` để lấy toàn bộ danh sách album,
    bạn có thể dùng danh sách này để xác nhận rằng album mới đã được thêm.

    ```
    $ curl http://localhost:8080/albums \
        --header "Content-Type: application/json" \
        --request "GET"
    ```

    Lệnh này sẽ hiển thị danh sách album.

    ```
    [
            {
                    "id": "1",
                    "title": "Blue Train",
                    "artist": "John Coltrane",
                    "price": 56.99
            },
            {
                    "id": "2",
                    "title": "Jeru",
                    "artist": "Gerry Mulligan",
                    "price": 17.99
            },
            {
                    "id": "3",
                    "title": "Sarah Vaughan and Clifford Brown",
                    "artist": "Sarah Vaughan",
                    "price": 39.99
            },
            {
                    "id": "4",
                    "title": "The Modern Sound of Betty Carter",
                    "artist": "Betty Carter",
                    "price": 49.99
            }
    ]
    ```

Trong phần tiếp theo, bạn sẽ thêm mã để xử lý `GET` cho một mục cụ thể.

## Viết một handler để trả về một mục cụ thể {#specific_item}

Khi client thực hiện yêu cầu đến `GET /albums/[id]`, bạn muốn trả về album có ID khớp với tham số đường dẫn `id`.

Để thực hiện việc này, bạn sẽ:

*   Thêm logic để truy xuất album được yêu cầu.
*   Ánh xạ đường dẫn tới logic.

#### Viết mã

1. Bên dưới hàm `postAlbums` mà bạn đã thêm trong phần trước, dán đoạn mã sau để truy xuất một album cụ thể.

    Hàm `getAlbumByID` này sẽ trích xuất ID trong đường dẫn yêu cầu, sau đó tìm một album phù hợp.

    ```
    // getAlbumByID tìm album có giá trị ID khớp với tham số id
    // được client gửi, sau đó trả về album đó dưới dạng phản hồi.
    func getAlbumByID(c *gin.Context) {
    	id := c.Param("id")

    	// Duyệt qua danh sách các album để tìm
    	// album có giá trị ID khớp với tham số.
    	for _, a := range albums {
    		if a.ID == id {
    			c.IndentedJSON(http.StatusOK, a)
    			return
    		}
    	}
    	c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
    }
    ```

    Trong đoạn mã này, bạn:

    *   Sử dụng [`Context.Param`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.Param)
        để truy xuất tham số đường dẫn `id` từ URL. Khi bạn ánh xạ handler này
        tới một đường dẫn, bạn sẽ bao gồm một phần giữ chỗ cho tham số trong
        đường dẫn.
    *   Duyệt qua các struct `album` trong slice, tìm struct có giá trị trường `ID`
        khớp với giá trị tham số `id`. Nếu tìm thấy, bạn tuần tự hóa struct
        `album` đó thành JSON và trả về dưới dạng phản hồi với mã HTTP `200 OK`.

        Như đã đề cập ở trên, một service thực tế có khả năng sẽ sử dụng truy vấn
        cơ sở dữ liệu để thực hiện việc tra cứu này.

    *   Trả về lỗi HTTP `404` với [`http.StatusNotFound`](https://pkg.go.dev/net/http#StatusNotFound)
        nếu không tìm thấy album.

2. Cuối cùng, thay đổi `main` để bao gồm một lệnh gọi mới tới `router.GET`,
    trong đó đường dẫn hiện là `/albums/:id`, như trong ví dụ sau.

    ```
    func main() {
    	router := gin.Default()
    	router.GET("/albums", getAlbums)
    	router.GET("/albums/:id", getAlbumByID)
    	router.POST("/albums", postAlbums)

    	router.Run("localhost:8080")
    }
    ```

    Trong đoạn mã này, bạn:

    *   Liên kết đường dẫn `/albums/:id` với hàm `getAlbumByID`. Trong
        Gin, dấu hai chấm đứng trước một mục trong đường dẫn biểu thị rằng mục đó
        là một tham số đường dẫn.

#### Chạy mã

1. Nếu máy chủ vẫn đang chạy từ phần trước, hãy dừng nó.
2. Từ dòng lệnh trong thư mục chứa main.go, chạy mã để khởi động máy chủ.

    ```
    $ go run .
    ```

3. Từ một cửa sổ dòng lệnh khác, sử dụng `curl` để gửi yêu cầu đến dịch vụ web đang chạy của bạn.

    ```
    $ curl http://localhost:8080/albums/2
    ```

Lệnh này sẽ hiển thị JSON cho album có ID mà bạn đã sử dụng. Nếu không tìm thấy album, bạn sẽ nhận được JSON có thông báo lỗi.

    ```
    {
            "id": "2",
            "title": "Jeru",
            "artist": "Gerry Mulligan",
            "price": 17.99
    }
    ```

## Kết luận {#conclusion}

Chúc mừng! Bạn vừa sử dụng Go và Gin để viết một dịch vụ web RESTful đơn giản.

Các chủ đề tiếp theo được đề xuất:

*   Nếu bạn mới làm quen với Go, bạn sẽ tìm thấy các phương pháp hay hữu ích được mô tả trong [Effective Go](/doc/effective_go) và [How to write Go code](/doc/code).
*   [Go Tour](/tour/) là một bước giới thiệu tuyệt vời theo từng bước về các kiến thức cơ bản của Go.
*   Để biết thêm về Gin, hãy xem [tài liệu gói Gin Web Framework](https://pkg.go.dev/github.com/gin-gonic/gin) hoặc [tài liệu Gin Web Framework](https://gin-gonic.com/en/docs/).

## Mã hoàn chỉnh {#completed_code}

Phần này chứa mã của ứng dụng mà bạn xây dựng bằng hướng dẫn này.

```
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
)

// album represents data about a record album.
type album struct {
	ID     string  `json:"id"`
	Title  string  `json:"title"`
	Artist string  `json:"artist"`
	Price  float64 `json:"price"`
}

// albums slice to seed record album data.
var albums = []album{
	{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
	{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
	{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}

func main() {
	router := gin.Default()
	router.GET("/albums", getAlbums)
	router.GET("/albums/:id", getAlbumByID)
	router.POST("/albums", postAlbums)

	router.Run("localhost:8080")
}

// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
	c.IndentedJSON(http.StatusOK, albums)
}

// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
	var newAlbum album

	// Call BindJSON to bind the received JSON to
	// newAlbum.
	if err := c.BindJSON(&newAlbum); err != nil {
		return
	}

	// Add the new album to the slice.
	albums = append(albums, newAlbum)
	c.IndentedJSON(http.StatusCreated, newAlbum)
}

// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
	id := c.Param("id")

	// Loop through the list of albums, looking for
	// an album whose ID value matches the parameter.
	for _, a := range albums {
		if a.ID == id {
			c.IndentedJSON(http.StatusOK, a)
			return
		}
	}
	c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```
