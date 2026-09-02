---
title: Quét lỗ hổng bảo mật trong IDE
layout: article
---

[Quay lại Bảo mật Go](/security)

Các trình soạn thảo tích hợp với [Go language server](https://pkg.go.dev/golang.org/x/tools/cmd/gopls), chẳng hạn như [VS Code với tiện ích mở rộng Go](https://marketplace.visualstudio.com/items?itemName=golang.go), có thể phát hiện lỗ hổng bảo mật trong các dependency của bạn.

Có hai chế độ để phát hiện lỗ hổng bảo mật trong các dependency. Cả hai đều được hỗ trợ bởi [cơ sở dữ liệu lỗ hổng bảo mật Go](https://vuln.go.dev) và bổ sung cho nhau.

* Phân tích dựa trên import: ở chế độ này, các trình soạn thảo báo cáo lỗ hổng bảo mật bằng cách quét tập hợp các gói được import trong workspace, và hiển thị kết quả dưới dạng chẩn đoán trong các tệp `go.mod`. Điều này nhanh, nhưng có thể báo cáo dương tính giả trong trường hợp mã của bạn import các gói chứa các ký hiệu dễ bị tấn công nhưng các hàm có lỗ hổng bảo mật không thể tiếp cận. Chế độ này có thể được bật bởi cài đặt gopls [`"vulncheck": "Imports"`](https://github.com/golang/tools/blob/master/gopls/doc/settings.md#vulncheck-enum).
* Phân tích `Govulncheck`: dựa trên công cụ dòng lệnh [`govulncheck`](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck), được nhúng trong `gopls`. Điều này cung cấp một cách ít nhiễu, đáng tin cậy để xác nhận xem mã của bạn có thực sự gọi các hàm dễ bị tấn công hay không. Vì phân tích này có thể tốn kém để tính toán, nó phải được kích hoạt thủ công bằng cách sử dụng hành động mã "Run govulncheck to verify" liên kết với các báo cáo chẩn đoán từ phân tích dựa trên Import, hoặc sử dụng code lens [`"codelenses.run_govulncheck"`](https://github.com/golang/tools/blob/master/gopls/doc/settings.md#run-govulncheck) trên các tệp `go.mod`.

<div style="text-align: center;"><img src="vscode.gif" alt="Vulncheck">

<em>Go: Toggle Vulncheck</em> <a
href="https://user-images.githubusercontent.com/4999471/206977512-a821107d-9ffb-4456-9b27-6a6a4f900ba6.mp4">(vulncheck.mp4)</a>
</div>

Các tính năng này có sẵn trong `gopls` v0.11.0 trở lên. Hãy chia sẻ phản hồi của bạn tại [go.dev/s/vsc-vulncheck-feedback](/s/vsc-vulncheck-feedback).

## Hướng dẫn cho từng trình soạn thảo

### VS Code

[Tiện ích mở rộng Go](https://marketplace.visualstudio.com/items?itemName=golang.go) cung cấp tích hợp với gopls. Các cài đặt sau cần thiết để bật các tính năng quét lỗ hổng bảo mật:

```
"go.diagnostic.vulncheck": "Imports", // bật phân tích dựa trên import theo mặc định.
"gopls": {
  "ui.codelenses": {
    "run_govulncheck": true  // code lens "Run govulncheck" trên tệp go.mod.
  }
}
```

Lệnh ["Go Toggle Vulncheck"](https://github.com/golang/vscode-go/wiki/Commands#go-toggle-vulncheck) có thể được sử dụng để bật và tắt phân tích dựa trên import cho workspace hiện tại.

### Vim/NeoVim

Khi sử dụng [coc.nvim](https://www.vim.org/scripts/script.php?script_id=5779), cài đặt sau sẽ bật phân tích dựa trên import.

```
{
    "codeLens.enable": true,
    "languageserver": {
        "go": {
            "command": "gopls",
            ...
            "initializationOptions": {
                "vulncheck": "Imports",
            }
        }
    }
}
```

## Lưu ý và Cảnh báo

- Tiện ích mở rộng không quét các gói riêng tư cũng không gửi bất kỳ thông tin nào về các module riêng tư. Tất cả phân tích được thực hiện bằng cách lấy danh sách các module dễ bị tấn công đã biết từ cơ sở dữ liệu lỗ hổng bảo mật Go và sau đó tính toán giao cắt cục bộ.
- Phân tích dựa trên import sử dụng danh sách các gói trong các module workspace, có thể khác với những gì bạn thấy từ các tệp `go.mod` nếu `go.work` hoặc module `replace`/`exclude` được sử dụng.
- Kết quả phân tích govulncheck có thể trở nên lỗi thời khi bạn sửa đổi mã hoặc cơ sở dữ liệu lỗ hổng bảo mật Go được cập nhật. Để vô hiệu hóa thủ công các kết quả phân tích, hãy sử dụng codelens `"Reset go.mod diagnostics"` được hiển thị ở đầu tệp `go.mod`. Nếu không, kết quả sẽ tự động bị vô hiệu hóa sau một giờ.
- Các tính năng này hiện chưa báo cáo lỗ hổng bảo mật trong thư viện chuẩn hoặc toolchain. Chúng tôi vẫn đang điều tra UX về nơi hiển thị kết quả và cách giúp người dùng xử lý các vấn đề.
