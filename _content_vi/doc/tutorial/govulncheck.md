<!--{
  "Title": "Hướng dẫn: Tìm và khắc phục các phần phụ thuộc có lỗ hổng bằng govulncheck",
  "HideTOC": true,
  "Breadcrumb": true
}-->

Govulncheck là một công cụ có ít nhiễu giúp bạn tìm và sửa các dependency dễ bị lỗ hổng bảo mật trong các dự án Go của mình. Công cụ này thực hiện bằng cách quét các dependency của dự án để tìm các lỗ hổng bảo mật đã biết, sau đó xác định mọi lệnh gọi trực tiếp hoặc gián tiếp đến các lỗ hổng đó trong mã của bạn.

Trong hướng dẫn này, bạn sẽ học cách sử dụng govulncheck để quét một chương trình đơn giản nhằm tìm lỗ hổng bảo mật. Bạn cũng sẽ học cách ưu tiên và đánh giá các lỗ hổng bảo mật để có thể tập trung sửa những lỗ hổng quan trọng nhất trước.

Để tìm hiểu thêm về govulncheck, hãy xem
[tài liệu govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck),
và [bài đăng blog về quản lý lỗ hổng bảo mật](/blog/vuln) này dành cho Go.
Chúng tôi cũng rất mong nhận được [phản hồi của bạn](/s/govulncheck-feedback).

## Điều kiện tiên quyết

- **Go.** Chúng tôi khuyến nghị sử dụng phiên bản Go mới nhất để thực hiện theo hướng dẫn này.
  Để biết hướng dẫn cài đặt, hãy xem [Cài đặt Go](/doc/install).
- **Trình soạn thảo mã.** Bất kỳ trình soạn thảo nào bạn có đều có thể hoạt động tốt.
- **Thiết bị đầu cuối lệnh.** Go hoạt động tốt khi sử dụng bất kỳ terminal nào trên Linux và Mac, cũng như PowerShell hoặc cmd trên Windows.

Hướng dẫn này sẽ đưa bạn qua các bước sau:

1. Tạo một Go module mẫu với một dependency dễ bị lỗ hổng bảo mật
2. Cài đặt và chạy govulncheck
3. Đánh giá các lỗ hổng bảo mật
4. Nâng cấp các dependency dễ bị lỗ hổng bảo mật

## Tạo một Go module mẫu với một dependency dễ bị lỗ hổng bảo mật

**Bước 1.** Để bắt đầu, hãy tạo một thư mục mới có tên `vuln-tutorial` và khởi tạo một Go module.
(Nếu bạn mới làm quen với Go module, hãy xem [go.dev/doc/tutorial/create-module](/doc/tutorial/create-module).

Ví dụ, từ thư mục chính của bạn, hãy chạy lệnh sau:

```
$ mkdir vuln-tutorial
$ cd vuln-tutorial
$ go mod init vuln.tutorial
```

**Bước 2.** Tạo một tệp có tên `main.go` trong thư mục `vuln-tutorial`, và sao chép
đoạn mã sau vào đó:

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

Chương trình mẫu này nhận một danh sách thẻ ngôn ngữ làm đối số dòng lệnh
và in một thông báo cho từng thẻ, cho biết thẻ đó có được phân tích cú pháp thành công,
không được định nghĩa, hoặc có xảy ra lỗi trong quá trình phân tích cú pháp thẻ hay không.

**Bước 3.** Chạy `go mod tidy`, thao tác này sẽ điền vào tệp `go.mod` tất cả các
dependency cần thiết cho mã bạn đã thêm vào `main.go` trong bước trước.

Từ thư mục `vuln-tutorial`, chạy:

```
$ go mod tidy
```

Bạn sẽ thấy kết quả sau:

```
go: finding module for package golang.org/x/text/language
go: downloading golang.org/x/text v0.9.0
go: found golang.org/x/text/language in golang.org/x/text v0.9.0
```

**Bước 4.** Mở tệp `go.mod` của bạn để xác minh rằng nó có dạng như sau:

```
module vuln.tutorial

go 1.20

require golang.org/x/text v0.9.0
```

**Bước 5.** Hạ cấp phiên bản của `golang.org/x/text` xuống v0.3.5, phiên bản này chứa các
lỗ hổng bảo mật đã biết. Chạy:

```
$ go get golang.org/x/text@v0.3.5
```

Bạn sẽ thấy kết quả sau:

```
go: downgraded golang.org/x/text v0.9.0 => v0.3.5
```

Tệp `go.mod` bây giờ sẽ có nội dung:

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

Bạn sẽ thấy đầu ra sau:

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

### Diễn giải đầu ra

<font size="2">  *Lưu ý: Nếu bạn không sử dụng phiên bản Go mới nhất,
bạn có thể thấy thêm các lỗ hổng bảo mật từ thư viện chuẩn. </font>

Mã của chúng ta bị ảnh hưởng bởi một lỗ hổng bảo mật,
[GO-2021-0113](https://pkg.go.dev/vuln/GO-2021-0113), vì nó gọi trực tiếp
hàm `Parse` của `golang.org/x/text/language` ở một phiên bản có lỗ hổng
(v0.3.5).

Một lỗ hổng bảo mật khác, [GO-2022-1059](https://pkg.go.dev/vuln/GO-2022-1059),
tồn tại trong module `golang.org/x/text` ở v0.3.5. Tuy nhiên, nó được báo cáo là
"Informational" vì mã của chúng ta không bao giờ gọi (trực tiếp hoặc gián tiếp)
bất kỳ hàm nào có lỗ hổng bảo mật của module này.

Bây giờ, hãy đánh giá các lỗ hổng bảo mật và xác định hành động cần thực hiện.

### Đánh giá các lỗ hổng bảo mật

a. Đánh giá các lỗ hổng bảo mật.

Trước tiên, hãy đọc mô tả của lỗ hổng bảo mật và xác định liệu lỗ hổng đó có thực sự áp dụng cho mã của bạn và trường hợp sử dụng của bạn hay không. Nếu cần thêm thông tin, hãy truy cập liên kết "Thông tin thêm".

Dựa trên mô tả, lỗ hổng bảo mật GO-2021-0113 có thể gây ra panic khi `Parse` được sử dụng để xử lý các đầu vào không đáng tin cậy từ người dùng. Giả sử rằng chúng ta muốn chương trình của mình có thể chịu được các đầu vào không đáng tin cậy và chúng ta quan tâm đến việc bị từ chối dịch vụ, do đó lỗ hổng này có khả năng áp dụng.

GO-2022-1059 có khả năng không ảnh hưởng đến mã của chúng ta, vì mã của chúng ta không gọi bất kỳ hàm dễ bị tổn thương nào từ báo cáo đó.

b. Quyết định hành động.

Để giảm thiểu GO-2021-0113, chúng ta có một số tùy chọn:
- **Tùy chọn 1: Nâng cấp lên phiên bản đã được sửa lỗi.** Nếu có bản sửa lỗi, chúng ta có thể loại bỏ một dependency dễ bị tổn thương bằng cách nâng cấp lên phiên bản đã được sửa lỗi của module.
- **Tùy chọn 2: Ngừng sử dụng các ký hiệu dễ bị tổn thương.** Chúng ta có thể chọn loại bỏ tất cả các lệnh gọi đến hàm dễ bị tổn thương trong mã của mình.
  Chúng ta sẽ cần tìm một giải pháp thay thế hoặc tự triển khai nó.

Trong trường hợp này, đã có bản sửa lỗi và hàm `Parse` là phần không thể thiếu trong chương trình của chúng ta. Hãy nâng cấp dependency của chúng ta lên phiên bản "fixed in", v0.3.7.

Chúng ta đã quyết định ưu tiên thấp hơn việc sửa lỗ hổng bảo mật chỉ mang tính thông tin, GO-2022-1059, nhưng vì nó nằm trong cùng module với GO-2021-0113, và vì phiên bản fixed in của nó là v0.3.8, chúng ta có thể dễ dàng loại bỏ cả hai cùng lúc bằng cách nâng cấp lên v0.3.8.

## Nâng cấp các dependency dễ bị tổn thương

May mắn là việc nâng cấp các dependency dễ bị tổn thương khá đơn giản.

**Bước 8.** Nâng cấp `golang.org/x/text` lên v0.3.8:

```
$ go get golang.org/x/text@v0.3.8
```

Bạn sẽ thấy kết quả sau:

```
go: upgraded golang.org/x/text v0.3.5 => v0.3.8
```

(Lưu ý rằng chúng ta cũng có thể chọn nâng cấp lên `latest` hoặc bất kỳ phiên bản nào khác sau v0.3.8).

**Bước 9.** Bây giờ hãy chạy govulncheck lần nữa:

```
$ govulncheck ./...
```

Bây giờ bạn sẽ thấy kết quả sau:

```
govulncheck is an experimental tool. Share feedback at https://go.dev/s/govulncheck-feedback.

Using go1.20.3 and govulncheck@v0.0.0 with
vulnerability data from https://vuln.go.dev (last modified 2023-04-06 19:19:26 +0000 UTC).

Scanning your code and 46 packages across 1 dependent module for known vulnerabilities...
No vulnerabilities found.
```

Cuối cùng, govulncheck xác nhận rằng không tìm thấy lỗ hổng bảo mật nào.

Bằng cách thường xuyên quét các dependency của bạn bằng lệnh govulncheck, bạn có thể bảo vệ codebase của mình bằng cách xác định, ưu tiên và xử lý các lỗ hổng bảo mật.
