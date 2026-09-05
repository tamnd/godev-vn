<!--{
  "Title": "Hướng dẫn: Tìm và sửa các dependency có lỗ hổng bảo mật với VS Code Go",
  "Breadcrumb": true
}-->

[Quay lại Go Security](/security)

Bạn có thể quét code của mình để tìm lỗ hổng bảo mật trực tiếp từ trình soạn thảo với tiện ích mở rộng Go cho Visual Studio Code.

Lưu ý: để xem giải thích về cách sửa lỗ hổng bảo mật được đề cập trong các hình ảnh bên dưới, xem [hướng dẫn govulncheck](/doc/tutorial/govulncheck).

## Điều kiện tiên quyết:

- **Go.** Chúng tôi khuyến nghị sử dụng phiên bản Go mới nhất để thực hiện hướng dẫn này. Để biết hướng dẫn cài đặt, xem [Cài đặt Go](/doc/install).
- **VS Code**, cập nhật lên phiên bản mới nhất. [Tải xuống tại đây](https://code.visualstudio.com/). Bạn cũng có thể dùng Vim (xem [tại đây](/security/vuln/editor#editor-specific-instructions) để biết chi tiết), nhưng hướng dẫn này tập trung vào VS Code Go.
- **Tiện ích mở rộng VS Code Go**, có thể [tải xuống tại đây](https://marketplace.visualstudio.com/items?itemName=golang.go).
- **Thay đổi cài đặt dành riêng cho trình soạn thảo.** Bạn sẽ cần sửa đổi cài đặt IDE của mình theo [các thông số kỹ thuật này](/security/vuln/editor#editor-specific-instructions) trước khi có thể sao chép lại các kết quả dưới đây.


## Cách quét lỗ hổng bảo mật bằng VS Code Go

**Bước 1.** Chạy "Go: Toggle Vulncheck"

Lệnh [Toggle Vulncheck](https://github.com/golang/vscode-go/wiki/Commands#go-toggle-vulncheck) hiển thị phân tích lỗ hổng bảo mật cho tất cả các dependency được liệt kê trong module của bạn. Để dùng lệnh này, mở [command palette](https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette) trong IDE của bạn (Ctrl+Shift+P trên Linux/Windows hoặc Cmd+Shift+P trên Mac OS) và chạy "Go: Toggle Vulncheck." Trong file go.mod của bạn, bạn sẽ thấy các chẩn đoán cho các dependency có lỗ hổng bảo mật được sử dụng cả trực tiếp lẫn gián tiếp trong code của bạn.

<div class="image">
  <center>
    <img style="width: 100%" width="2110" height="952" src="editor_tutorial_1.png" alt="Run Toggle Vulncheck"></img>
  </center>
</div>

Lưu ý: Để tự sao chép lại hướng dẫn này trong trình soạn thảo của bạn, hãy sao chép đoạn code dưới đây vào file main.go của bạn.

```
// This program takes language tags as command-line
// arguments and parses them.

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

Sau đó, đảm bảo file go.mod tương ứng cho chương trình trông như sau:


```
module module1

go 1.18

require golang.org/x/text v0.3.5
```

Bây giờ, chạy `go mod tidy` để đảm bảo file go.sum của bạn được cập nhật.

**Bước 2.** Chạy govulncheck qua một code action.

Chạy govulncheck bằng code action cho phép bạn tập trung vào các dependency thực sự được gọi trong code của bạn. Code action trong VS Code được đánh dấu bằng biểu tượng bóng đèn; di chuột qua dependency liên quan để xem thông tin về lỗ hổng bảo mật, rồi chọn "Quick Fix" để thấy menu các tùy chọn. Trong số đó, chọn "run govulncheck to verify." Điều này sẽ trả về kết quả govulncheck liên quan trong terminal của bạn.

<div class="image">
  <center>
    <img style="width: 100%" width="2110" height="952" src="editor_tutorial_2.png" alt="govulncheck code action"></img>
  </center>
</div>

<div class="image">
  <center>
    <img style="width: 100%" width="2110" height="952" src="editor_tutorial_3.png" alt="VS Code Go govulncheck output"></img>
  </center>
</div>

**Bước 3**. Di chuột qua một dependency được liệt kê trong file go.mod của bạn.

Thông tin govulncheck liên quan về một dependency cụ thể cũng có thể được tìm thấy bằng cách di chuột qua dependency trong file go.mod. Để xem nhanh thông tin dependency, tùy chọn này thậm chí còn hiệu quả hơn so với dùng code action.

<div class="image">
  <center>
    <img style="width: 100%" width="2110" height="952" src="editor_tutorial_4.png" alt="Hover over dependency for vulnerability information"></img>
  </center>
</div>

**Bước 4.** Nâng cấp lên phiên bản "fixed in" của dependency.

Code action cũng có thể được dùng để nhanh chóng nâng cấp lên phiên bản dependency đã khắc phục lỗ hổng bảo mật. Thực hiện điều này bằng cách chọn tùy chọn "Upgrade" trong menu code action.

<div class="image">
  <center>
    <img style="width: 100%" width="2110" height="952" src="editor_tutorial_5.png" alt="Upgrade to Latest via code action menu"></img>
  </center>
</div>


## Tài nguyên bổ sung

- Xem [trang này](/security/vuln/editor) để biết thêm thông tin về quét lỗ hổng bảo mật trong IDE của bạn. [Phần Notes and Caveats](/security/vuln/editor#notes-and-caveats), đặc biệt, thảo luận về các trường hợp đặc biệt mà việc quét lỗ hổng bảo mật có thể phức tạp hơn ví dụ trên.
- [Cơ sở dữ liệu lỗ hổng bảo mật Go](https://pkg.go.dev/vuln/) chứa thông tin từ nhiều nguồn hiện có ngoài các báo cáo trực tiếp từ người bảo trì gói Go đến nhóm bảo mật Go.
- Xem trang [Quản lý lỗ hổng bảo mật Go](/security/vuln/) cung cấp góc nhìn tổng quát về kiến trúc của Go để phát hiện, báo cáo và quản lý lỗ hổng bảo mật.
