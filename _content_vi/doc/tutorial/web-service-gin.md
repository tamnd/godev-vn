<!--{
  "Title": "Hướng dẫn: Phát triển RESTful API với Go và Gin",
  "Breadcrumb": true
}-->

Hướng dẫn này giới thiệu những kiến thức cơ bản về việc viết một RESTful web service API với Go
và [Gin Web Framework](https://gin-gonic.com/en/docs/) (Gin).

Bạn sẽ tận dụng tối đa hướng dẫn này nếu đã có hiểu biết cơ bản về Go
và các công cụ đi kèm. Nếu đây là lần đầu tiên bạn tiếp xúc với Go, hãy xem
[Hướng dẫn: Bắt đầu với Go](/doc/tutorial/getting-started)
để có phần giới thiệu nhanh.

Gin đơn giản hóa nhiều tác vụ coding liên quan đến việc xây dựng ứng dụng web,
bao gồm cả web service. Trong hướng dẫn này, bạn sẽ dùng Gin để định tuyến request,
lấy thông tin chi tiết của request và marshal JSON cho response.

Trong hướng dẫn này, bạn sẽ xây dựng một RESTful API server với hai endpoint. Dự án ví dụ
của bạn sẽ là một kho lưu trữ dữ liệu về các đĩa nhạc jazz cổ điển.

Hướng dẫn bao gồm các phần sau:

1. Thiết kế các API endpoint.
2. Tạo thư mục cho code của bạn.
3. Tạo dữ liệu.
4. Viết handler để trả về tất cả các mục.
5. Viết handler để thêm một mục mới.
6. Viết handler để trả về một mục cụ thể.

**Lưu ý:** Để xem các hướng dẫn khác, truy cập [Hướng dẫn](/doc/tutorial/index.html).

Để thử hướng dẫn này dưới dạng tương tác trong Google Cloud Shell,
nhấn nút bên dưới.

[![Open in Cloud Shell](https://gstatic.com/cloudssh/images/open-btn.png)](https://ide.cloud.google.com/?cloudshell_workspace=~&walkthrough_tutorial_url=https://raw.githubusercontent.com/golang/tour/master/tutorial/web-service-gin.md)


## Điều kiện tiên quyết

*   **Đã cài đặt Go 1.16 hoặc mới hơn.** Để biết hướng dẫn cài đặt, xem
    [Cài đặt Go](/doc/install).
*   **Một công cụ để chỉnh sửa code.** Bất kỳ trình soạn thảo văn bản nào bạn có đều dùng được.
*   **Một cửa sổ dòng lệnh.** Go hoạt động tốt trên bất kỳ terminal nào trên Linux và Mac,
    cũng như trên PowerShell hoặc cmd trong Windows.
*   **Công cụ curl.** Trên Linux và Mac, công cụ này đã được cài đặt sẵn. Trên
    Windows, nó có trong Windows 10 Insider build 17063 và các phiên bản sau. Với các
    phiên bản Windows cũ hơn, bạn có thể cần cài đặt nó. Để biết thêm, xem
    [Tar and Curl Come to Windows](https://docs.microsoft.com/en-us/virtualization/community/team-blog/2017/20171219-tar-and-curl-come-to-windows).

## Thiết kế các API endpoint {#design_endpoints}

Bạn sẽ xây dựng một API cung cấp quyền truy cập vào một cửa hàng bán các bản ghi nhạc cổ điển
trên vinyl. Vì vậy, bạn sẽ cần cung cấp các endpoint thông qua đó client có thể lấy
và thêm album cho người dùng.

Khi phát triển API, bạn thường bắt đầu bằng cách thiết kế các endpoint. Người dùng
API của bạn sẽ thành công hơn nếu các endpoint dễ hiểu.

Dưới đây là các endpoint bạn sẽ tạo trong hướng dẫn này.

/albums
*   `GET` – Lấy danh sách tất cả album, trả về dưới dạng JSON.
*   `POST` – Thêm một album mới từ dữ liệu request được gửi dưới dạng JSON.

/albums/:id
*   `GET` – Lấy một album theo ID, trả về dữ liệu album dưới dạng JSON.

Tiếp theo, bạn sẽ tạo một thư mục cho code của mình.

## Tạo thư mục cho code của bạn {#create_folder}

Để bắt đầu, hãy tạo một dự án cho code bạn sẽ viết.

1. Mở dấu nhắc lệnh và chuyển đến thư mục home của bạn.

    Trên Linux hoặc Mac:

    ```
    $ cd
    ```

    Trên Windows:

    ```
    C:\> cd %HOMEPATH%
    ```

2. Dùng dấu nhắc lệnh, tạo một thư mục có tên web-service-gin.

    ```
    $ mkdir web-service-gin
    $ cd web-service-gin
    ```

3. Tạo một module để quản lý dependency.

    Chạy lệnh `go mod init`, cung cấp đường dẫn module cho code của bạn.

    ```
    $ go mod init example/web-service-gin
    go: creating new go.mod: module example/web-service-gin
    ```

    Lệnh này tạo ra file go.mod, trong đó các dependency bạn thêm vào sẽ được
    liệt kê để theo dõi. Để biết thêm về việc đặt tên module với đường dẫn module, xem
    [Quản lý dependency](/doc/modules/managing-dependencies#naming_module).

Tiếp theo, bạn sẽ thiết kế cấu trúc dữ liệu để xử lý dữ liệu.

## Tạo dữ liệu {#create_data}

Để giữ mọi thứ đơn giản cho hướng dẫn, bạn sẽ lưu trữ dữ liệu trong bộ nhớ. Một
API điển hình hơn sẽ tương tác với cơ sở dữ liệu.

Lưu ý rằng việc lưu trữ dữ liệu trong bộ nhớ có nghĩa là tập hợp album sẽ bị mất mỗi
lần bạn dừng server, sau đó được tạo lại khi bạn khởi động lại.

#### Viết code

1. Dùng trình soạn thảo văn bản của bạn, tạo một file có tên main.go trong thư mục web-service.
    Bạn sẽ viết code Go của mình trong file này.
2. Vào main.go, ở đầu file, dán phần khai báo package sau.

    ```
    package main
    ```

    Một chương trình độc lập (trái với thư viện) luôn ở trong gói `main`.

3. Bên dưới khai báo package, dán phần khai báo struct `album` sau.
    Bạn sẽ dùng nó để lưu trữ dữ liệu album trong bộ nhớ.

    Các struct tag như ``json:"artist"`` chỉ định tên của trường sẽ là gì
    khi nội dung struct được serialize thành JSON. Nếu không có chúng, JSON
    sẽ dùng tên trường viết hoa của struct, một phong cách ít phổ biến hơn trong JSON.

    ```
    // album represents data about a record album.
    type album struct {
    	ID     string  `json:"id"`
    	Title  string  `json:"title"`
    	Artist string  `json:"artist"`
    	Price  float64 `json:"price"`
    }
    ```

4. Bên dưới phần khai báo struct vừa thêm, dán slice của
    các struct `album` sau chứa dữ liệu bạn sẽ dùng để bắt đầu.

    ```
    // albums slice to seed record album data.
    var albums = []album{
    	{ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
    	{ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
    	{ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
    }
    ```

Tiếp theo, bạn sẽ viết code để triển khai endpoint đầu tiên của mình.

## Viết handler để trả về tất cả các mục {#all_items}

Khi client thực hiện request tại `GET /albums`, bạn muốn trả về tất cả
album dưới dạng JSON.

Để làm điều này, bạn sẽ viết:

*   Logic để chuẩn bị response
*   Code để ánh xạ đường dẫn request đến logic của bạn

Lưu ý rằng đây là ngược với cách chúng sẽ được thực thi khi chạy, nhưng bạn đang
thêm các dependency trước, rồi mới đến code phụ thuộc vào chúng.

#### Viết code

1. Bên dưới code struct bạn đã thêm ở phần trước, dán
    đoạn code sau để lấy danh sách album.

    Hàm `getAlbums` này tạo JSON từ slice các struct `album`,
    ghi JSON vào response.

    ```
    // getAlbums responds with the list of all albums as JSON.
    func getAlbums(c *gin.Context) {
    	c.IndentedJSON(http.StatusOK, albums)
    }
    ```

    Trong đoạn code này, bạn:

    *   Viết hàm `getAlbums` nhận tham số
        [`gin.Context`](https://pkg.go.dev/github.com/gin-gonic/gin#Context).
        Lưu ý rằng bạn có thể đặt tên cho hàm này bất kỳ cái tên nào, không có
        Gin lẫn Go yêu cầu định dạng tên hàm cụ thể.

        `gin.Context` là phần quan trọng nhất của Gin. Nó mang thông tin chi tiết request,
        xác thực và serialize JSON, và nhiều hơn nữa. (Mặc dù tên tương tự,
        đây khác với gói
        [`context`](/pkg/context/) có sẵn trong Go.)

    *   Gọi [`Context.IndentedJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.IndentedJSON)
        để serialize struct thành JSON và thêm vào response.

        Đối số đầu tiên của hàm là mã trạng thái HTTP bạn muốn gửi đến
        client. Ở đây, bạn đang truyền hằng số [`StatusOK`](https://pkg.go.dev/net/http#StatusOK)
        từ gói `net/http` để chỉ định `200 OK`.

        Lưu ý rằng bạn có thể thay thế `Context.IndentedJSON` bằng lần gọi
        [`Context.JSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.JSON)
        để gửi JSON nhỏ gọn hơn. Trong thực tế, dạng có thụt lề dễ làm việc hơn nhiều
        khi debug và sự khác biệt về kích thước thường nhỏ.

2. Gần đầu main.go, ngay bên dưới khai báo slice `albums`, dán
    đoạn code bên dưới để gán hàm handler cho đường dẫn endpoint.

    Điều này thiết lập mối liên kết trong đó `getAlbums` xử lý các request đến
    đường dẫn endpoint `/albums`.

    ```
    func main() {
    	router := gin.Default()
    	router.GET("/albums", getAlbums)

    	router.Run("localhost:8080")
    }
    ```

    Trong đoạn code này, bạn:

    *   Khởi tạo router Gin bằng [`Default`](https://pkg.go.dev/github.com/gin-gonic/gin#Default).
    *   Dùng hàm [`GET`](https://pkg.go.dev/github.com/gin-gonic/gin#RouterGroup.GET)
        để liên kết phương thức HTTP `GET` và đường dẫn `/albums` với một hàm handler.

        Lưu ý rằng bạn đang truyền _tên_ của hàm `getAlbums`. Điều này
        khác với việc truyền _kết quả_ của hàm, thứ bạn sẽ làm bằng cách
        truyền `getAlbums()` (chú ý dấu ngoặc đơn).

    *   Dùng hàm [`Run`](https://pkg.go.dev/github.com/gin-gonic/gin#Engine.Run)
        để gắn router vào `http.Server` và khởi động server.

3. Gần đầu main.go, ngay bên dưới khai báo package, import
    các gói bạn cần để hỗ trợ code vừa viết.

    Các dòng đầu tiên của code nên trông như sau:

    ```
    package main

    import (
    	"net/http"

    	"github.com/gin-gonic/gin"
    )
    ```

4. Lưu main.go.

#### Chạy code

1. Bắt đầu theo dõi module Gin như một dependency.

    Tại dòng lệnh, dùng [`go get`](/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them)
    để thêm module github.com/gin-gonic/gin là dependency cho module của bạn.
    Dùng đối số dấu chấm để có nghĩa là "lấy dependency cho code trong thư mục hiện tại."

    ```
    $ go get .
    go get: added github.com/gin-gonic/gin v1.7.2
    ```

    Go đã phân giải và tải dependency này để đáp ứng khai báo `import`
    bạn đã thêm ở bước trước.

2. Từ dòng lệnh trong thư mục chứa main.go, chạy code.
    Dùng đối số dấu chấm để có nghĩa là "chạy code trong thư mục hiện tại."

    ```
    $ go run .
    ```

    Khi code đang chạy, bạn có một HTTP server đang hoạt động mà bạn có thể
    gửi request đến.

3. Từ một cửa sổ dòng lệnh mới, dùng `curl` để thực hiện request đến
    web service đang chạy của bạn.

    ```
    $ curl http://localhost:8080/albums
    ```

    Lệnh này sẽ hiển thị dữ liệu bạn đã tạo cho service.

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

Bạn đã khởi động một API! Trong phần tiếp theo, bạn sẽ tạo một endpoint khác với
code để xử lý request `POST` để thêm một mục.

## Viết handler để thêm một mục mới {#add_item}

Khi client thực hiện request `POST` tại `/albums`, bạn muốn thêm album
được mô tả trong body request vào dữ liệu album hiện có.

Để làm điều này, bạn sẽ viết:

*   Logic để thêm album mới vào danh sách hiện có.
*   Một ít code để định tuyến request `POST` đến logic của bạn.

#### Viết code

1. Thêm code để thêm dữ liệu album vào danh sách album.

    Ở đâu đó sau các câu lệnh `import`, dán đoạn code sau. (Cuối
    file là nơi tốt cho code này, nhưng Go không bắt buộc thứ tự
    bạn khai báo các hàm.)

    ```
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
    ```

    Trong đoạn code này, bạn:

    *   Dùng [`Context.BindJSON`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.BindJSON)
        để bind body request vào `newAlbum`.
    *   Thêm struct `album` được khởi tạo từ JSON vào slice `albums`.
    *   Thêm mã trạng thái `201` vào response, cùng với JSON đại diện cho
        album bạn đã thêm.

2. Thay đổi hàm `main` để bao gồm hàm `router.POST`,
    như ví dụ sau.

    ```
    func main() {
    	router := gin.Default()
    	router.GET("/albums", getAlbums)
    	router.POST("/albums", postAlbums)

    	router.Run("localhost:8080")
    }
    ```

    Trong đoạn code này, bạn:

    *   Liên kết phương thức `POST` tại đường dẫn `/albums` với hàm `postAlbums`.

        Với Gin, bạn có thể liên kết một handler với tổ hợp phương thức HTTP và đường dẫn.
        Bằng cách này, bạn có thể định tuyến riêng các request được gửi đến
        một đường dẫn duy nhất dựa trên phương thức client đang sử dụng.

#### Chạy code

1. Nếu server vẫn đang chạy từ phần trước, hãy dừng nó lại.
2. Từ dòng lệnh trong thư mục chứa main.go, chạy code.

    ```
    $ go run .
    ```

3. Từ một cửa sổ dòng lệnh khác, dùng `curl` để thực hiện request đến
    web service đang chạy của bạn.

    ```
    $ curl http://localhost:8080/albums \
        --include \
        --header "Content-Type: application/json" \
        --request "POST" \
        --data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
    ```

    Lệnh này sẽ hiển thị header và JSON cho album đã thêm.

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

4. Như ở phần trước, dùng `curl` để lấy danh sách đầy đủ album,
    bạn có thể dùng để xác nhận rằng album mới đã được thêm vào.

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

Trong phần tiếp theo, bạn sẽ thêm code để xử lý `GET` cho một mục cụ thể.

## Viết handler để trả về một mục cụ thể {#specific_item}

Khi client thực hiện request `GET /albums/[id]`, bạn muốn trả về
album có ID khớp với tham số đường dẫn `id`.

Để làm điều này, bạn sẽ:

*   Thêm logic để lấy album được yêu cầu.
*   Ánh xạ đường dẫn đến logic.

#### Viết code

1. Bên dưới hàm `postAlbums` bạn đã thêm ở phần trước, dán
    đoạn code sau để lấy một album cụ thể.

    Hàm `getAlbumByID` này sẽ trích xuất ID trong đường dẫn request, sau đó
    tìm một album khớp.

    ```
    // getAlbumByID locates the album whose ID value matches the id
    // parameter sent by the client, then returns that album as a response.
    func getAlbumByID(c *gin.Context) {
    	id := c.Param("id")

    	// Loop over the list of albums, looking for
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

    Trong đoạn code này, bạn:

    *   Dùng [`Context.Param`](https://pkg.go.dev/github.com/gin-gonic/gin#Context.Param)
        để lấy tham số đường dẫn `id` từ URL. Khi bạn ánh xạ
        handler này vào một đường dẫn, bạn sẽ bao gồm một placeholder cho tham số trong
        đường dẫn.
    *   Lặp qua các struct `album` trong slice, tìm một struct có trường `ID`
        khớp với giá trị tham số `id`. Nếu tìm thấy, bạn serialize
        struct `album` đó thành JSON và trả về như một response với mã HTTP `200 OK`.

        Như đã đề cập ở trên, một service thực tế có thể sẽ dùng truy vấn cơ sở dữ liệu
        để thực hiện việc tra cứu này.

    *   Trả về lỗi HTTP `404` với [`http.StatusNotFound`](https://pkg.go.dev/net/http#StatusNotFound)
        nếu không tìm thấy album.

2. Cuối cùng, thay đổi `main` để bao gồm một lần gọi mới đến `router.GET`,
    trong đó đường dẫn bây giờ là `/albums/:id`, như ví dụ sau.

    ```
    func main() {
    	router := gin.Default()
    	router.GET("/albums", getAlbums)
    	router.GET("/albums/:id", getAlbumByID)
    	router.POST("/albums", postAlbums)

    	router.Run("localhost:8080")
    }
    ```

    Trong đoạn code này, bạn:

    *   Liên kết đường dẫn `/albums/:id` với hàm `getAlbumByID`. Trong
        Gin, dấu hai chấm đứng trước một mục trong đường dẫn biểu thị rằng mục đó là
        một tham số đường dẫn.

#### Chạy code

1. Nếu server vẫn đang chạy từ phần trước, hãy dừng nó lại.
2. Từ dòng lệnh trong thư mục chứa main.go, chạy code để
    khởi động server.

    ```
    $ go run .
    ```

3. Từ một cửa sổ dòng lệnh khác, dùng `curl` để thực hiện request đến
    web service đang chạy của bạn.

    ```
    $ curl http://localhost:8080/albums/2
    ```

    Lệnh này sẽ hiển thị JSON cho album có ID bạn đã dùng. Nếu
    album không được tìm thấy, bạn sẽ nhận được JSON với thông báo lỗi.

    ```
    {
            "id": "2",
            "title": "Jeru",
            "artist": "Gerry Mulligan",
            "price": 17.99
    }
    ```

## Kết luận {#conclusion}

Chúc mừng! Bạn vừa dùng Go và Gin để viết một RESTful web service đơn giản.

Các chủ đề đề xuất tiếp theo:

*   Nếu bạn mới học Go, bạn sẽ tìm thấy các thực hành tốt hữu ích được mô tả trong
    [Effective Go](/doc/effective_go) và
    [Cách viết code Go](/doc/code).
*   [Go Tour](/tour/) là phần giới thiệu từng bước tuyệt vời
    về các khái niệm cơ bản của Go.
*   Để biết thêm về Gin, xem [tài liệu gói Gin Web Framework](https://pkg.go.dev/github.com/gin-gonic/gin)
    hoặc [tài liệu Gin Web Framework](https://gin-gonic.com/en/docs/).

## Code hoàn chỉnh {#completed_code}

Phần này chứa code của ứng dụng bạn xây dựng trong hướng dẫn này.

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
