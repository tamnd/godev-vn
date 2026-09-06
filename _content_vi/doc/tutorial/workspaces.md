---
Title: Hướng dẫn: Bắt đầu với workspace đa-mô-đun
Breadcrumb: true
---

Hướng dẫn này giới thiệu các kiến thức cơ bản về workspace đa module trong Go.
Với workspace đa module, bạn có thể cho lệnh Go biết rằng bạn đang
viết mã trong nhiều module cùng lúc và dễ dàng xây dựng cũng như
chạy mã trong các module đó.

Trong hướng dẫn này, bạn sẽ tạo hai module trong một workspace đa module
dùng chung, thực hiện thay đổi trên các module đó và xem kết quả
của những thay đổi đó trong một bản build.

<!-- TODO TOC -->

**Lưu ý:** Để xem các hướng dẫn khác, hãy xem [Hướng dẫn](/doc/tutorial/index.html).

## Điều kiện tiên quyết

*   **Go.** Chúng tôi khuyên bạn nên sử dụng phiên bản Go mới nhất để thực hiện hướng dẫn này.
    Để biết hướng dẫn cài đặt, hãy xem [Cài đặt Go](/doc/install).
*   **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt.
*   **Một terminal lệnh.** Go hoạt động tốt khi sử dụng bất kỳ terminal nào trên Linux và Mac,
    cũng như PowerShell hoặc cmd trên Windows.

## Tạo một module cho mã của bạn {#create_folder}

Để bắt đầu, hãy tạo một module cho mã mà bạn sẽ viết.

1. Mở dấu nhắc lệnh và chuyển đến thư mục chính của bạn.

   Trên Linux hoặc Mac:

    ```
    $ cd
    ```

   Trên Windows:

    ```
    C:\> cd %HOMEPATH%
    ```

   Phần còn lại của hướng dẫn sẽ hiển thị $ làm dấu nhắc. Các lệnh bạn sử dụng
   cũng sẽ hoạt động trên Windows.

2. Từ dấu nhắc lệnh, tạo một thư mục cho mã của bạn có tên là workspace.

    ```
    $ mkdir workspace
    $ cd workspace
    ```

3. Khởi tạo module

   Ví dụ của chúng ta sẽ tạo một module mới `hello` có dependency vào module golang.org/x/example.

   Tạo module hello:

   ```
   $ mkdir hello
   $ cd hello
   $ go mod init example.com/hello
   go: creating new go.mod: module example.com/hello
   ```

   Thêm dependency vào gói golang.org/x/example/hello/reverse bằng cách sử dụng `go get`.

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

   Bây giờ, hãy chạy chương trình hello:

   ```
   $ go run .
   olleH
   ```

## Tạo workspace

Trong bước này, bạn sẽ tạo tệp `go.work` để chỉ định một workspace với module.

#### Khởi tạo workspace

Trong thư mục `workspace`, chạy:

   ```
   $ go work init ./hello
   ```

Lệnh `go work init` yêu cầu `go` tạo một tệp `go.work`
cho một workspace chứa các module trong thư mục
`./hello`.

Lệnh `go` tạo ra tệp `go.work` có dạng như sau:

   ```
   go 1.18

   use ./hello
   ```

Tệp `go.work` có cú pháp tương tự như `go.mod`.

Chỉ thị `go` cho Go biết phiên bản Go nào mà tệp này nên được
diễn giải. Nó tương tự như chỉ thị `go` trong tệp `go.mod`.

Chỉ thị `use` cho Go biết rằng module trong thư mục `hello`
nên là các module chính khi thực hiện build.

Vì vậy, trong bất kỳ thư mục con nào của `workspace`, module sẽ được kích hoạt.

#### Chạy chương trình trong thư mục workspace

Trong thư mục `workspace`, chạy:

   ```
   $ go run ./hello
   olleH
   ```

Lệnh Go bao gồm tất cả module trong workspace dưới dạng các module chính. Điều này cho phép
bạn tham chiếu đến một package trong module, ngay cả khi ở bên ngoài module. Chạy lệnh `go run`
bên ngoài module hoặc workspace sẽ dẫn đến lỗi vì lệnh `go`
không biết nên sử dụng module nào.

Tiếp theo, bạn sẽ thêm một bản sao cục bộ của module `golang.org/x/example/hello` vào workspace.
Module đó được lưu trong một thư mục con của Git repository `go.googlesource.com/example`.
Sau đó, bạn sẽ thêm một hàm mới vào package `reverse` mà bạn có thể sử dụng thay cho `String`.

## Tải xuống và sửa đổi module `golang.org/x/example/hello`

Trong bước này, bạn sẽ tải xuống một bản sao của Git repo chứa module `golang.org/x/example/hello`,
thêm nó vào workspace, sau đó thêm một hàm mới vào đó để sử dụng từ chương trình hello.

1. Sao chép repository

   Từ thư mục workspace, chạy lệnh `git` để sao chép repository:

   ```
   $ git clone https://go.googlesource.com/example
   Cloning into 'example'...
   remote: Total 165 (delta 27), reused 165 (delta 27)
   Receiving objects: 100% (165/165), 434.18 KiB | 1022.00 KiB/s, done.
   Resolving deltas: 100% (27/27), done.
   ```

2. Thêm module vào workspace

   Git repo vừa được checkout vào `./example`.
   Mã nguồn cho module `golang.org/x/example/hello` nằm trong `./example/hello`.
   Thêm nó vào workspace:

   ```
   $ go work use ./example/hello
   ```

   Lệnh `go work use` thêm một module mới vào tệp go.work. Bây giờ tệp này sẽ có dạng như sau:

   ```
   go 1.18

   use (
       ./hello
       ./example/hello
   )
   ```

   Workspace hiện bao gồm cả module `example.com/hello` và module `golang.org/x/example/hello`,
   module này cung cấp package `golang.org/x/example/hello/reverse`.

   Điều này cho phép bạn sử dụng mã mới mà bạn sẽ viết trong bản sao của package `reverse`
   thay vì phiên bản của package trong module cache
   mà bạn đã tải xuống bằng lệnh `go get`.

3. Thêm hàm mới.

   Bạn sẽ thêm một hàm mới để đảo ngược một số vào package `golang.org/x/example/hello/reverse`.

   Tạo một tệp mới có tên `int.go` trong thư mục `workspace/example/hello/reverse` với nội dung sau:

   ```
   package reverse

   import "strconv"

// Int trả về dạng đảo ngược thập phân của số nguyên i.
   func Int(i int) int {
       i, _ = strconv.Atoi(String(strconv.Itoa(i)))
       return i
   }
   ```

4. Sửa đổi chương trình hello để sử dụng hàm.

   Sửa đổi nội dung của `workspace/hello/hello.go` để có nội dung sau:

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

#### Chạy mã trong workspace

Từ thư mục workspace, chạy

   ```
   $ go run ./hello
   olleH 10642
   ```

Lệnh Go tìm module `example.com/hello` được chỉ định trong dòng lệnh trong thư mục `hello` được chỉ định bởi tệp `go.work`, và tương tự giải quyết import `golang.org/x/example/hello/reverse` bằng cách sử dụng tệp `go.work`.

`go.work` có thể được sử dụng thay cho việc thêm các chỉ thị [`replace`](/ref/mod#go-mod-file-replace) để làm việc trên nhiều module.

Vì hai module nằm trong cùng một workspace nên việc thay đổi trong một module và sử dụng thay đổi đó trong module khác rất dễ dàng.

#### Bước tiếp theo

Bây giờ, để phát hành đúng cách các module này, chúng ta cần tạo một bản phát hành cho module `golang.org/x/example/hello`, ví dụ tại `v0.1.0`. Việc này thường được thực hiện bằng cách gắn thẻ một commit trong kho lưu trữ kiểm soát phiên bản của module. Xem [tài liệu về quy trình phát hành module](/doc/modules/release-workflow) để biết thêm chi tiết. Sau khi bản phát hành hoàn tất, chúng ta có thể tăng yêu cầu đối với module `golang.org/x/example/hello` trong `hello/go.mod`:

   ```
   cd hello
   go get golang.org/x/example/hello@v0.1.0
   ```

Bằng cách đó, lệnh `go` có thể giải quyết đúng các module bên ngoài workspace.

## Tìm hiểu thêm về workspace

Lệnh `go` có một số lệnh con để làm việc với workspace ngoài `go work init` mà chúng ta đã thấy trước đó trong hướng dẫn:

- `go work use [-r] [dir]` thêm chỉ thị `use` vào tệp `go.work` cho `dir` nếu nó tồn tại, và xóa thư mục `use` nếu thư mục được truyền vào không tồn tại. Cờ `-r` kiểm tra đệ quy các thư mục con của `dir`.
- `go work edit` chỉnh sửa tệp `go.work` tương tự như `go mod edit`
- `go work sync` đồng bộ các dependency từ danh sách build của workspace vào từng module trong workspace.

Xem [Workspaces](/ref/mod#workspaces) trong Tham chiếu Go Modules để biết thêm chi tiết về workspace và các tệp `go.work`.
