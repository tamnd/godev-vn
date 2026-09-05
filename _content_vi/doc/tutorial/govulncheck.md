<!--{
  "Title": "Hướng dẫn: Tìm và sửa các dependency có lỗ hổng bảo mật với govulncheck",
  "HideTOC": true,
  "Breadcrumb": true
}-->

Govulncheck là một công cụ ít gây nhiễu giúp bạn tìm và sửa các dependency có lỗ hổng bảo mật
trong các dự án Go. Công cụ này thực hiện bằng cách quét các dependency của dự án
để tìm các lỗ hổng bảo mật đã biết rồi xác định bất kỳ lần gọi trực tiếp hoặc
gián tiếp nào đến những lỗ hổng đó trong code của bạn.

Trong hướng dẫn này, bạn sẽ học cách dùng govulncheck để quét một chương trình đơn giản
để tìm lỗ hổng bảo mật. Bạn cũng sẽ học cách ưu tiên và
đánh giá các lỗ hổng bảo mật để có thể tập trung sửa những lỗi quan trọng nhất trước.

Để tìm hiểu thêm về govulncheck, xem
[tài liệu govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck),
và [bài viết blog về quản lý lỗ hổng bảo mật](/blog/vuln) cho Go.
Chúng tôi cũng rất muốn [nghe phản hồi của bạn](/s/govulncheck-feedback).

## Điều kiện tiên quyết

- **Go.** Chúng tôi khuyến nghị sử dụng phiên bản Go mới nhất để thực hiện hướng dẫn này.
  (Để biết hướng dẫn cài đặt, xem [Cài đặt Go](/doc/install).)
- **Trình soạn thảo code.** Bất kỳ trình soạn thảo nào bạn có đều dùng được.
- **Một cửa sổ dòng lệnh.** Go hoạt động tốt trên bất kỳ terminal nào trên Linux và Mac, cũng như trên PowerShell hoặc cmd trong Windows.

Hướng dẫn sẽ dẫn bạn qua các bước sau:

1. Tạo một module Go mẫu với dependency có lỗ hổng bảo mật
2. Cài đặt và chạy govulncheck
3. Đánh giá các lỗ hổng bảo mật
4. Nâng cấp các dependency có lỗ hổng bảo mật

## Tạo một module Go mẫu với dependency có lỗ hổng bảo mật

**Bước 1.** Để bắt đầu, tạo một thư mục mới có tên `vuln-tutorial` và khởi tạo một module Go.
(Nếu bạn mới làm quen với module Go, xem [go.dev/doc/tutorial/create-module](/doc/tutorial/create-module).

Ví dụ, từ thư mục home của bạn, chạy lệnh sau:

```
$ mkdir vuln-tutorial
$ cd vuln-tutorial
$ go mod init vuln.tutorial
```

**Bước 2.** Tạo một file có tên `main.go` trong thư mục `vuln-tutorial`, và sao chép
đoạn code sau vào đó:

```
package main

import (
        "fmt"
        "os"

        "golang.org/x/text/language"
)

func main() {
        for _, arg := range os.Args[1:] {
                tag, err := language.Parse(arg)
                if err != nil {
                        fmt.Printf("%s: error: %v\n", arg, err)
                } else if tag == language.Und {
                        fmt.Printf("%s: undefined\n", arg)
                } else {
                        fmt.Printf("%s: tag %s\n", arg, tag)
                }
        }
}
```

Chương trình mẫu này nhận một danh sách các language tag như là đối số dòng lệnh
và in một thông báo cho mỗi tag cho biết nó đã được phân tích thành công,
tag không được xác định, hoặc có lỗi xảy ra trong khi phân tích tag.

**Bước 3.** Chạy `go mod tidy`, sẽ điền vào file `go.mod` tất cả các
dependency cần thiết bởi code bạn đã thêm vào `main.go` ở bước trước.

Từ thư mục `vuln-tutorial`, chạy:

```
$ go mod tidy
```

Bạn sẽ thấy kết quả này:

```
go: finding module for package golang.org/x/text/language
go: downloading golang.org/x/text v0.9.0
go: found golang.org/x/text/language in golang.org/x/text v0.9.0
```

**Bước 4.** Mở file `go.mod` của bạn để xác minh rằng nó trông như sau:

```
module vuln.tutorial

go 1.20

require golang.org/x/text v0.9.0
```

**Bước 5.** Hạ cấp phiên bản `golang.org/x/text` xuống v0.3.5, phiên bản có chứa
các lỗ hổng bảo mật đã biết. Chạy:

```
$ go get golang.org/x/text@v0.3.5
```

Bạn sẽ thấy kết quả này:

```
go: downgraded golang.org/x/text v0.9.0 => v0.3.5
```

File `go.mod` bây giờ nên có nội dung:

```
module vuln.tutorial

go 1.20

require golang.org/x/text v0.3.5
```

Bây giờ, hãy xem govulncheck hoạt động như thế nào.


## Cài đặt và chạy govulncheck

**Bước 6.** Cài đặt govulncheck bằng lệnh `go install`:

```
$ go install golang.org/x/vuln/cmd/govulncheck@latest
```

**Bước 7.** Từ thư mục bạn muốn phân tích (trong trường hợp này là `vuln-tutorial`). Chạy:

```
$ govulncheck ./...
```

Bạn sẽ thấy kết quả này:

```
govulncheck is an experimental tool. Share feedback at https://go.dev/s/govulncheck-feedback.

Using go1.20.3 and govulncheck@v0.0.0 with
vulnerability data from https://vuln.go.dev (last modified 2023-04-18 21:32:26 +0000 UTC).

Scanning your code and 46 packages across 1 dependent module for known vulnerabilities...
Your code is affected by 1 vulnerability from 1 module.

Vulnerability #1: GO-2021-0113
  Due to improper index calculation, an incorrectly formatted
  language tag can cause Parse to panic via an out of bounds read.
  If Parse is used to process untrusted user inputs, this may be
  used as a vector for a denial of service attack.

  More info: https://pkg.go.dev/vuln/GO-2021-0113

  Module: golang.org/x/text
    Found in: golang.org/x/text@v0.3.5
    Fixed in: golang.org/x/text@v0.3.7

    Call stacks in your code:
      main.go:12:29: vuln.tutorial.main calls golang.org/x/text/language.Parse

=== Informational ===

Found 1 vulnerability in packages that you import, but there are no call
stacks leading to the use of this vulnerability. You may not need to
take any action. See https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck
for details.

Vulnerability #1: GO-2022-1059
  An attacker may cause a denial of service by crafting an
  Accept-Language header which ParseAcceptLanguage will take
  significant time to parse.
  More info: https://pkg.go.dev/vuln/GO-2022-1059
  Found in: golang.org/x/text@v0.3.5
  Fixed in: golang.org/x/text@v0.3.8

```

### Giải thích kết quả

<font size="2">  *Lưu ý: Nếu bạn không dùng phiên bản Go mới nhất,
bạn có thể thấy thêm các lỗ hổng bảo mật từ thư viện chuẩn. </font>

Code của chúng ta bị ảnh hưởng bởi một lỗ hổng bảo mật,
[GO-2021-0113](https://pkg.go.dev/vuln/GO-2021-0113), vì nó gọi trực tiếp
hàm `Parse` của `golang.org/x/text/language` tại phiên bản có lỗ hổng
(v0.3.5).

Một lỗ hổng bảo mật khác, [GO-2022-1059](https://pkg.go.dev/vuln/GO-2022-1059),
tồn tại trong module `golang.org/x/text` tại v0.3.5. Tuy nhiên, nó được báo cáo là
"Informational" vì code của chúng ta không bao giờ (trực tiếp hoặc gián tiếp) gọi bất kỳ
hàm nào có lỗ hổng của nó.

Bây giờ, hãy đánh giá các lỗ hổng bảo mật và xác định hành động cần thực hiện.

### Đánh giá các lỗ hổng bảo mật

a. Đánh giá các lỗ hổng bảo mật.

Trước tiên, đọc mô tả của lỗ hổng bảo mật và xác định xem nó có thực sự
áp dụng cho code và trường hợp sử dụng của bạn hay không. Nếu bạn cần thêm thông tin, hãy truy cập
đường dẫn "More info".

Dựa vào mô tả, lỗ hổng bảo mật GO-2021-0113 có thể gây ra panic khi
`Parse` được dùng để xử lý các đầu vào từ người dùng không tin cậy. Giả sử rằng chúng ta có ý định
chương trình của mình có thể chịu đựng các đầu vào không tin cậy, và chúng ta lo ngại về tấn công từ chối dịch vụ,
thì lỗ hổng bảo mật này có thể áp dụng.

GO-2022-1059 có thể không ảnh hưởng đến code của chúng ta, vì code của chúng ta không gọi
bất kỳ hàm nào có lỗ hổng từ báo cáo đó.

b. Quyết định hành động.

Để khắc phục GO-2021-0113, chúng ta có một vài lựa chọn:
- **Lựa chọn 1: Nâng cấp lên phiên bản đã sửa.** Nếu có bản sửa lỗi, chúng ta có thể loại bỏ dependency có lỗ hổng bằng cách nâng cấp lên phiên bản đã sửa của module.
- **Lựa chọn 2: Ngừng sử dụng ký hiệu có lỗ hổng.** Chúng ta có thể chọn loại bỏ tất cả các lần gọi đến hàm có lỗ hổng trong code của mình. Chúng ta sẽ cần tìm một giải pháp thay thế hoặc tự triển khai.

Trong trường hợp này, có sẵn bản sửa lỗi và hàm `Parse` là thiết yếu cho
chương trình của chúng ta. Hãy nâng cấp dependency lên phiên bản "fixed in", v0.3.7.

Chúng ta quyết định trì hoãn việc sửa lỗ hổng bảo mật informational,
GO-2022-1059, nhưng vì nó nằm trong cùng module với GO-2021-0113, và vì phiên bản fixed in cho nó là v0.3.8, chúng ta có thể
dễ dàng loại bỏ cả hai cùng lúc bằng cách nâng cấp lên v0.3.8.

## Nâng cấp các dependency có lỗ hổng bảo mật

May mắn thay, việc nâng cấp các dependency có lỗ hổng bảo mật khá đơn giản.

**Bước 8.** Nâng cấp `golang.org/x/text` lên v0.3.8:

```
$ go get golang.org/x/text@v0.3.8
```

Bạn sẽ thấy kết quả này:

```
go: upgraded golang.org/x/text v0.3.5 => v0.3.8
```

(Lưu ý rằng chúng ta cũng có thể chọn nâng cấp lên `latest` hoặc bất kỳ phiên bản nào sau v0.3.8).

**Bước 9.** Bây giờ chạy lại govulncheck:

```
$ govulncheck ./...
```

Bạn sẽ thấy kết quả này:

```
govulncheck is an experimental tool. Share feedback at https://go.dev/s/govulncheck-feedback.

Using go1.20.3 and govulncheck@v0.0.0 with
vulnerability data from https://vuln.go.dev (last modified 2023-04-06 19:19:26 +0000 UTC).

Scanning your code and 46 packages across 1 dependent module for known vulnerabilities...
No vulnerabilities found.
```

Cuối cùng, govulncheck xác nhận rằng không tìm thấy lỗ hổng bảo mật nào.

Bằng cách thường xuyên quét các dependency với lệnh govulncheck, bạn có thể
bảo vệ codebase của mình bằng cách xác định, ưu tiên và giải quyết
các lỗ hổng bảo mật.
