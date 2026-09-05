<!--{
  "Title": "Xuất bản một module"
}-->

Khi bạn muốn đưa module lên để các lập trình viên khác sử dụng, bạn xuất bản nó để công cụ Go có thể nhìn thấy. Sau khi xuất bản module, các lập trình viên import các package từ nó sẽ có thể giải quyết dependency vào module bằng cách chạy các lệnh như `go get`.

> **Lưu ý:** Đừng thay đổi một phiên bản module đã được gắn tag sau khi xuất bản. Với các lập trình viên đang dùng module, công cụ Go xác thực module được tải xuống so với bản sao đầu tiên đã tải. Nếu hai bản khác nhau, công cụ Go sẽ trả về lỗi bảo mật. Thay vì thay đổi code của phiên bản đã xuất bản trước đó, hãy xuất bản một phiên bản mới.

**Xem thêm**

* Để xem tổng quan về phát triển module, xem [Phát triển và xuất bản module](developing)
* Để xem quy trình phát triển module cấp cao -- bao gồm việc xuất bản -- xem [Quy trình phát hành và đánh số phiên bản module](release-workflow).

## Các bước xuất bản

Thực hiện theo các bước sau để xuất bản một module.

1. Mở dấu nhắc lệnh và chuyển đến thư mục gốc của module trong kho lưu trữ cục bộ.

1.  Chạy `go mod tidy`, lệnh này sẽ xóa mọi dependency mà module có thể đã tích lũy nhưng không còn cần thiết.

    ```
    $ go mod tidy
    ```

1.  Chạy `go test ./...` lần cuối để đảm bảo mọi thứ hoạt động bình thường.

    Lệnh này chạy các unit test bạn đã viết để sử dụng framework kiểm thử Go.

    ```
    $ go test ./...
    ok      example.com/mymodule       0.015s
    ```

1.  Gắn tag dự án với số phiên bản mới bằng lệnh `git tag`.

    Đối với số phiên bản, hãy dùng một con số cho người dùng biết bản chất của những thay đổi trong bản phát hành này. Để biết thêm, xem [Đánh số phiên bản module](version-numbers).

    ```
    $ git commit -m "mymodule: changes for v0.1.0"
    $ git tag v0.1.0
    ```

1.  Đẩy tag mới lên kho lưu trữ gốc.

    ```
    $ git push origin v0.1.0
    ```

1.  Đưa module lên bằng cách chạy [lệnh `go list`](/cmd/go/#hdr-List_packages_or_modules) để khuyến khích Go cập nhật chỉ mục module của nó với thông tin về module bạn đang xuất bản.

    Trước lệnh này, hãy thêm câu lệnh đặt biến môi trường `GOPROXY` thành một Go proxy. Điều này đảm bảo yêu cầu của bạn đến được proxy.

    ```
    $ GOPROXY=proxy.golang.org go list -m example.com/mymodule@v0.1.0
    ```

Các lập trình viên quan tâm đến module của bạn import một package từ nó và chạy [lệnh `go get`]() giống như với bất kỳ module nào khác. Họ có thể chạy [lệnh `go get`]() để lấy phiên bản mới nhất hoặc chỉ định một phiên bản cụ thể, như trong ví dụ sau:

```
$ go get example.com/mymodule@v0.1.0
```
