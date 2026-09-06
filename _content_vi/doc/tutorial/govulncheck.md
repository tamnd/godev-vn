---
Title: Hướng dẫn: Tìm và khắc phục các phần phụ thuộc có lỗ hổng bằng govulncheck
HideTOC: true
Breadcrumb: true
---

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


## Install and run govulncheck

**Step 6.** Install govulncheck with the `go install` command:

```
$ go install golang.org/x/vuln/cmd/govulncheck@latest
```

**Step 7.** From the folder you want to analyze (in this case, `vuln-tutorial`). Run:

```
$ govulncheck ./...
```

You should see this output:

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

### Interpreting the output

<font size="2">  *Note: If you are not using the latest version of Go,
you may see additional vulnerabilities from the standard library. </font>

Our code is affected by one vulnerability,
[GO-2021-0113](https://pkg.go.dev/vuln/GO-2021-0113), because it directly calls
the `Parse` function of `golang.org/x/text/language` at a vulnerable version
(v0.3.5).

Another vulnerability, [GO-2022-1059](https://pkg.go.dev/vuln/GO-2022-1059),
exists in the `golang.org/x/text` module at v0.3.5.  However, it is reported as
"Informational" because our code never (directly or indirectly) calls any of
its vulnerable functions.

Now, let's evaluate the vulnerabilities and determine an action to take.

### Evaluate vulnerabilities

a. Evaluate vulnerabilities.

First, read the description of the vulnerability and determine if it actually
applies to your code and your use case. If you need more information, visit
the "More info" link.

Based on the description, vulnerability GO-2021-0113 can cause a panic when
`Parse` is used to process untrusted user inputs. Let's suppose that we intend
our program to withstand untrusted inputs, and we are concerned about denial of
service, so the vulnerability likely applies.

GO-2022-1059 likely does not affect our code, because our code does not call
any vulnerable functions from that report.

b. Decide on an action.

To mitigate GO-2021-0113, we have a few options:
- **Option 1: Upgrade to a fixed version.** If there is a fix available,
  we can remove a vulnerable dependency by upgrading to a fixed version of the module.
- **Option 2: Stop using the vulnerable symbol(s).** We could choose to
  remove all calls to the vulnerable function in our code.
  We would need to find an alternative or implement it ourselves.

In this case, a fix is available, and the `Parse` function is integral to our
program. Let's upgrade our dependency to the "fixed in" version, v0.3.7.

We decided to deprioritize fixing the informational vulnerability,
GO-2022-1059, but because it is in the same module as GO-2021-0113, and because the fixed in version for it is v0.3.8, we can
easily remove both at the same time by upgrading to v0.3.8.

## Upgrade vulnerable dependencies

Luckily, upgrading vulnerable dependencies is quite simple.

**Step 8.** Upgrade `golang.org/x/text` to v0.3.8:

```
$ go get golang.org/x/text@v0.3.8
```

You should see this output:

```
go: upgraded golang.org/x/text v0.3.5 => v0.3.8
```

(Note that we could have also chosen to upgrade to `latest`, or any other version after v0.3.8).

**Step 9.** Now run govulncheck again:

```
$ govulncheck ./...
```

You will now see this output:

```
govulncheck is an experimental tool. Share feedback at https://go.dev/s/govulncheck-feedback.

Using go1.20.3 and govulncheck@v0.0.0 with
vulnerability data from https://vuln.go.dev (last modified 2023-04-06 19:19:26 +0000 UTC).

Scanning your code and 46 packages across 1 dependent module for known vulnerabilities...
No vulnerabilities found.
```

Finally, govulncheck confirms that there are no vulnerabilities found.

By regularly scanning your dependencies with command govulncheck, you can
safeguard your codebase by identifying, prioritizing, and addressing
vulnerabilities.
