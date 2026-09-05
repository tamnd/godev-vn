<!--{
  "Title": "Hướng dẫn: Bắt đầu với workspace nhiều module",
  "Breadcrumb": true
}-->

Hướng dẫn này giới thiệu những kiến thức cơ bản về workspace nhiều module trong Go.
Với workspace nhiều module, bạn có thể cho lệnh Go biết rằng bạn đang
viết code trong nhiều module cùng lúc và dễ dàng build và
chạy code trong các module đó.

Trong hướng dẫn này, bạn sẽ tạo hai module trong một workspace nhiều module chung,
thực hiện các thay đổi trên các module đó và xem kết quả
của những thay đổi đó trong một lần build.

<!-- TODO TOC -->

**Lưu ý:** Để xem các hướng dẫn khác, truy cập [Hướng dẫn](/doc/tutorial/index.html).

## Điều kiện tiên quyết

*   **Đã cài đặt Go 1.18 hoặc mới hơn.**
*   **Một công cụ để chỉnh sửa code.** Bất kỳ trình soạn thảo văn bản nào bạn có đều dùng được.
*   **Một cửa sổ dòng lệnh.** Go hoạt động tốt trên bất kỳ terminal nào trên Linux và Mac,
    cũng như trên PowerShell hoặc cmd trong Windows.

Hướng dẫn này yêu cầu go1.18 hoặc mới hơn. Đảm bảo bạn đã cài đặt Go phiên bản 1.18 hoặc mới hơn bằng cách dùng
các đường dẫn tại [go.dev/dl](/dl).

## Tạo một module cho code của bạn {#create_folder}

Để bắt đầu, hãy tạo một module cho code bạn sẽ viết.

1. Mở dấu nhắc lệnh và chuyển đến thư mục home của bạn.

   Trên Linux hoặc Mac:

    ```
    $ cd
    ```

   Trên Windows:

    ```
    C:\> cd %HOMEPATH%
    ```

   Phần còn lại của hướng dẫn sẽ dùng $ làm dấu nhắc lệnh. Các lệnh bạn sử dụng
   cũng hoạt động trên Windows.

2. Từ dấu nhắc lệnh, tạo một thư mục có tên workspace.

    ```
    $ mkdir workspace
    $ cd workspace
    ```

3. Khởi tạo module

   Ví dụ của chúng ta sẽ tạo một module mới `hello` phụ thuộc vào module golang.org/x/example.

   Tạo module hello:

   ```
   $ mkdir hello
   $ cd hello
   $ go mod init example.com/hello
   go: creating new go.mod: module example.com/hello
   ```

   Thêm dependency vào gói golang.org/x/example/hello/reverse bằng `go get`.

   ```
   $ go get golang.org/x/example/hello/reverse
   ```

   Tạo hello.go trong thư mục hello với nội dung sau:

   ```
   package main

   import (
       "fmt"

       "golang.org/x/example/hello/reverse"
   )

   func main() {
       fmt.Println(reverse.String("Hello"))
   }
   ```

   Bây giờ, chạy chương trình hello:

   ```
   $ go run .
   olleH
   ```

## Tạo workspace

Trong bước này, chúng ta sẽ tạo file `go.work` để chỉ định một workspace với module.

#### Khởi tạo workspace

Trong thư mục `workspace`, chạy:

   ```
   $ go work init ./hello
   ```

Lệnh `go work init` yêu cầu `go` tạo một file `go.work`
cho một workspace chứa các module trong thư mục `./hello`.

Lệnh `go` tạo ra file `go.work` trông như sau:

   ```
   go 1.18

   use ./hello
   ```

File `go.work` có cú pháp tương tự như `go.mod`.

Chỉ thị `go` cho Go biết phiên bản Go nào mà file này nên được
diễn giải theo. Nó tương tự như chỉ thị `go` trong file `go.mod`.

Chỉ thị `use` cho Go biết rằng module trong thư mục `hello`
nên là các module chính khi build.

Vì vậy, trong bất kỳ thư mục con nào của `workspace`, module sẽ được kích hoạt.

#### Chạy chương trình trong thư mục workspace

Trong thư mục `workspace`, chạy:

   ```
   $ go run ./hello
   olleH
   ```

Lệnh Go bao gồm tất cả các module trong workspace như các module chính. Điều này cho phép chúng ta
tham chiếu đến một gói trong module, ngay cả khi ở ngoài module. Chạy lệnh `go run`
ngoài module hoặc workspace sẽ dẫn đến lỗi vì lệnh `go` không
biết module nào sẽ sử dụng.

Tiếp theo, chúng ta sẽ thêm một bản sao cục bộ của module `golang.org/x/example/hello` vào workspace.
Module đó được lưu trữ trong một thư mục con của kho lưu trữ Git `go.googlesource.com/example`.
Sau đó chúng ta sẽ thêm một hàm mới vào gói `reverse` mà chúng ta có thể dùng thay vì `String`.

## Tải xuống và sửa đổi module `golang.org/x/example/hello`

   Trong bước này, chúng ta sẽ tải xuống một bản sao của kho lưu trữ Git chứa module `golang.org/x/example/hello`,
   thêm nó vào workspace, sau đó thêm một hàm mới vào module đó mà chúng ta sẽ dùng từ chương trình hello.

1. Clone kho lưu trữ

   Từ thư mục workspace, chạy lệnh `git` để clone kho lưu trữ:

   ```
   $ git clone https://go.googlesource.com/example
   Cloning into 'example'...
   remote: Total 165 (delta 27), reused 165 (delta 27)
   Receiving objects: 100% (165/165), 434.18 KiB | 1022.00 KiB/s, done.
   Resolving deltas: 100% (27/27), done.
   ```

2. Thêm module vào workspace

   Kho lưu trữ Git vừa được checkout vào `./example`.
   Source code cho module `golang.org/x/example/hello` nằm trong `./example/hello`.
   Thêm nó vào workspace:

   ```
   $ go work use ./example/hello
   ```

   Lệnh `go work use` thêm một module mới vào file go.work. Lúc này nó sẽ trông như sau:

   ```
   go 1.18

   use (
       ./hello
       ./example/hello
   )
   ```

   Workspace bây giờ bao gồm cả module `example.com/hello` và module `golang.org/x/example/hello`,
   module này cung cấp gói `golang.org/x/example/hello/reverse`.

   Điều này sẽ cho phép chúng ta dùng code mới chúng ta sẽ viết trong bản sao gói `reverse` của mình
   thay vì phiên bản của gói trong module cache
   mà chúng ta đã tải xuống bằng lệnh `go get`.

3. Thêm hàm mới.

   Chúng ta sẽ thêm một hàm mới để đảo ngược một số vào gói `golang.org/x/example/hello/reverse`.

   Tạo một file mới có tên `int.go` trong thư mục `workspace/example/hello/reverse` với nội dung sau:

   ```
   package reverse

   import "strconv"

   // Int returns the decimal reversal of the integer i.
   func Int(i int) int {
       i, _ = strconv.Atoi(String(strconv.Itoa(i)))
       return i
   }
   ```

4. Sửa đổi chương trình hello để dùng hàm này.

   Sửa đổi nội dung của `workspace/hello/hello.go` như sau:

   ```
   package main

   import (
       "fmt"

       "golang.org/x/example/hello/reverse"
   )

   func main() {
       fmt.Println(reverse.String("Hello"), reverse.Int(24601))
   }
   ```

#### Chạy code trong workspace

   Từ thư mục workspace, chạy

   ```
   $ go run ./hello
   olleH 10642
   ```

   Lệnh Go tìm module `example.com/hello` được chỉ định trong
   dòng lệnh trong thư mục `hello` được chỉ định bởi file `go.work`,
   và tương tự phân giải import `golang.org/x/example/hello/reverse` bằng cách dùng
   file `go.work`.

   `go.work` có thể được dùng thay vì thêm các chỉ thị [`replace`](/ref/mod#go-mod-file-replace)
   để làm việc trên nhiều module.

   Vì hai module nằm trong cùng workspace, việc
   thực hiện thay đổi trong một module và dùng nó trong module khác rất dễ dàng.

#### Bước tiếp theo

   Bây giờ, để phát hành đúng cách các module này, chúng ta sẽ cần tạo một bản phát hành của module `golang.org/x/example/hello`,
   ví dụ tại `v0.1.0`. Điều này thường được thực hiện bằng cách gắn tag một commit trên kho lưu trữ quản lý phiên bản của module.
   Xem
   [tài liệu về quy trình phát hành module](/doc/modules/release-workflow)
   để biết thêm chi tiết. Sau khi phát hành xong, chúng ta có thể tăng yêu cầu đối với
   module `golang.org/x/example/hello` trong `hello/go.mod`:

   ```
   cd hello
   go get golang.org/x/example/hello@v0.1.0
   ```

   Bằng cách đó, lệnh `go` có thể phân giải đúng các module ngoài workspace.

## Tìm hiểu thêm về workspace

   Lệnh `go` có một số lệnh con để làm việc với workspace ngoài `go work init` mà
   chúng ta đã thấy trước đó trong hướng dẫn:

   - `go work use [-r] [dir]` thêm chỉ thị `use` vào file `go.work` cho `dir`,
   nếu nó tồn tại, và xóa thư mục `use` nếu thư mục đối số không tồn tại. Cờ `-r`
   kiểm tra các thư mục con của `dir` đệ quy.
   - `go work edit` chỉnh sửa file `go.work` tương tự như `go mod edit`
   - `go work sync` đồng bộ hóa dependency từ danh sách build của workspace vào mỗi module trong workspace.

   Xem [Workspace](/ref/mod#workspaces) trong Go Modules Reference để biết thêm chi tiết về
   workspace và các file `go.work`.
