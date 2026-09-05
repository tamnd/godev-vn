<!--{
  "Title": "Tổ chức một module Go"
}-->

Một câu hỏi phổ biến của các lập trình viên mới làm quen với Go là "Tôi nên tổ chức dự án Go của mình như thế nào?" về mặt bố cục tệp và thư mục. Mục tiêu của tài liệu này là cung cấp một số hướng dẫn giúp trả lời câu hỏi đó. Để tận dụng tốt nhất tài liệu này, hãy đảm bảo bạn đã quen với những kiến thức cơ bản về module Go bằng cách đọc [hướng dẫn](/doc/tutorial/create-module) và [quản lý mã nguồn module](/doc/modules/managing-source).

Dự án Go có thể bao gồm các package, chương trình dòng lệnh hoặc kết hợp cả hai. Hướng dẫn này được tổ chức theo loại dự án.

### Package cơ bản

Một package Go cơ bản có toàn bộ code trong thư mục gốc của dự án. Dự án bao gồm một module duy nhất, module đó bao gồm một package duy nhất. Tên package khớp với thành phần cuối cùng của đường dẫn module. Với một package rất đơn giản chỉ cần một tệp Go, cấu trúc dự án là:

```
project-root-directory/
  go.mod
  modname.go
  modname_test.go
```

_[xuyên suốt tài liệu này, tên tệp/package hoàn toàn là tùy ý]_

Giả sử thư mục này được tải lên một kho lưu trữ GitHub tại `github.com/someuser/modname`, dòng `module` trong tệp `go.mod` phải là `module github.com/someuser/modname`.

Code trong `modname.go` khai báo package với:

```
package modname

// ... package code here
```

Sau đó người dùng có thể phụ thuộc vào package này bằng cách `import` nó trong code Go của họ:

```
import "github.com/someuser/modname"
```

Một package Go có thể được chia thành nhiều tệp, tất cả đều nằm trong cùng một thư mục, ví dụ:

```
project-root-directory/
  go.mod
  modname.go
  modname_test.go
  auth.go
  auth_test.go
  hash.go
  hash_test.go
```

Tất cả các tệp trong thư mục đều khai báo `package modname`.

### Lệnh cơ bản

Một chương trình thực thi cơ bản (hay công cụ dòng lệnh) được cấu trúc theo độ phức tạp và kích thước code. Chương trình đơn giản nhất có thể bao gồm một tệp Go duy nhất nơi `func main` được định nghĩa. Các chương trình lớn hơn có thể chia code thành nhiều tệp, tất cả đều khai báo `package main`:

```
project-root-directory/
  go.mod
  auth.go
  auth_test.go
  client.go
  main.go
```

Ở đây tệp `main.go` chứa `func main`, nhưng đây chỉ là quy ước. Tệp "main" cũng có thể được đặt tên là `modname.go` (với giá trị `modname` phù hợp) hoặc bất kỳ tên nào khác.

Giả sử thư mục này được tải lên một kho lưu trữ GitHub tại `github.com/someuser/modname`, dòng `module` trong tệp `go.mod` phải là:

```
module github.com/someuser/modname
```

Và người dùng có thể cài đặt nó trên máy của họ bằng:

```
$ go install github.com/someuser/modname@latest
```

### Package hoặc lệnh với các package hỗ trợ

Các package hoặc lệnh lớn hơn có thể hưởng lợi từ việc tách một số chức năng ra các package hỗ trợ. Ban đầu, nên đặt các package như vậy vào thư mục `internal`; [điều này ngăn](https://pkg.go.dev/cmd/go#hdr-Internal_Directories) các module khác phụ thuộc vào các package mà chúng ta không nhất thiết muốn công khai và hỗ trợ cho các mục đích bên ngoài. Vì các dự án khác không thể import code từ thư mục `internal` của chúng ta, chúng ta có thể tự do tái cấu trúc API của nó và di chuyển mọi thứ mà không ảnh hưởng đến người dùng bên ngoài. Cấu trúc dự án cho một package vì vậy sẽ là:

```
project-root-directory/
  internal/
    auth/
      auth.go
      auth_test.go
    hash/
      hash.go
      hash_test.go
  go.mod
  modname.go
  modname_test.go
```

Tệp `modname.go` khai báo `package modname`, `auth.go` khai báo `package auth` và tương tự. `modname.go` có thể import package `auth` như sau:

```
import "github.com/someuser/modname/internal/auth"
```

Bố cục cho một lệnh có các package hỗ trợ trong thư mục `internal` rất tương tự, ngoại trừ tệp (hoặc các tệp) trong thư mục gốc khai báo `package main`.

### Nhiều package

Một module có thể bao gồm nhiều package có thể import; mỗi package có thư mục riêng và có thể được cấu trúc phân cấp. Đây là cấu trúc dự án mẫu:

```
project-root-directory/
  go.mod
  modname.go
  modname_test.go
  auth/
    auth.go
    auth_test.go
    token/
      token.go
      token_test.go
  hash/
    hash.go
  internal/
    trace/
      trace.go
```

Nhắc lại, chúng ta giả sử dòng `module` trong `go.mod` là:

```
module github.com/someuser/modname
```

Package `modname` nằm trong thư mục gốc, khai báo `package modname` và có thể được import bởi người dùng với:

```
import "github.com/someuser/modname"
```

Các package con có thể được import bởi người dùng như sau:

```
import "github.com/someuser/modname/auth"
import "github.com/someuser/modname/auth/token"
import "github.com/someuser/modname/hash"
```

Package `trace` nằm trong `internal/trace` không thể được import từ bên ngoài module này. Nên giữ các package trong `internal` càng nhiều càng tốt.

### Nhiều lệnh

Nhiều chương trình trong cùng một kho lưu trữ thường có các thư mục riêng biệt:

```
project-root-directory/
  go.mod
  internal/
    ... shared internal packages
  prog1/
    main.go
  prog2/
    main.go
```

Trong mỗi thư mục, các tệp Go của chương trình khai báo `package main`. Một thư mục `internal` ở cấp cao nhất có thể chứa các package dùng chung được sử dụng bởi tất cả các lệnh trong kho lưu trữ.

Người dùng có thể cài đặt các chương trình này như sau:

```
$ go install github.com/someuser/modname/prog1@latest
$ go install github.com/someuser/modname/prog2@latest
```

Một quy ước phổ biến là đặt tất cả các lệnh trong một kho lưu trữ vào thư mục `cmd`; mặc dù điều này không thực sự cần thiết trong một kho lưu trữ chỉ gồm các lệnh, nhưng nó rất hữu ích trong một kho lưu trữ hỗn hợp có cả lệnh và các package có thể import, như chúng ta sẽ thảo luận tiếp theo.

### Package và lệnh trong cùng một kho lưu trữ

Đôi khi một kho lưu trữ sẽ cung cấp cả các package có thể import và các lệnh có thể cài đặt với chức năng liên quan. Đây là cấu trúc dự án mẫu cho một kho lưu trữ như vậy:

```
project-root-directory/
  go.mod
  modname.go
  modname_test.go
  auth/
    auth.go
    auth_test.go
  internal/
    ... internal packages
  cmd/
    prog1/
      main.go
    prog2/
      main.go
```

Giả sử module này có tên `github.com/someuser/modname`, người dùng bây giờ có thể import các package từ nó:

```
import "github.com/someuser/modname"
import "github.com/someuser/modname/auth"
```

Và cài đặt các chương trình từ nó:

```
$ go install github.com/someuser/modname/cmd/prog1@latest
$ go install github.com/someuser/modname/cmd/prog2@latest
```

### Dự án server

Go là lựa chọn ngôn ngữ phổ biến để triển khai *server*. Cấu trúc của các dự án như vậy rất đa dạng, do có nhiều khía cạnh của việc phát triển server: giao thức (REST? gRPC?), triển khai, tệp front-end, container hóa, script và nhiều hơn nữa. Chúng ta sẽ tập trung hướng dẫn vào các phần dự án được viết bằng Go.

Các dự án server thường không có package để xuất, vì server thường là một binary khép kín (hoặc một nhóm binary). Do đó, nên giữ các package Go cài đặt logic của server trong thư mục `internal`. Hơn nữa, vì dự án có thể có nhiều thư mục khác với các tệp không phải Go, nên giữ tất cả các lệnh Go cùng nhau trong thư mục `cmd`:

```
project-root-directory/
  go.mod
  internal/
    auth/
      ...
    metrics/
      ...
    model/
      ...
  cmd/
    api-server/
      main.go
    metrics-analyzer/
      main.go
    ...
  ... the project's other directories with non-Go code
```

Trong trường hợp kho lưu trữ server phát triển thêm các package trở nên hữu ích để chia sẻ với các dự án khác, tốt nhất là tách các package đó ra thành các module riêng biệt.
