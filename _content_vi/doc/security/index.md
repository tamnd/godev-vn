---
title: Bảo mật
layout: article
---

Trang này cung cấp tài nguyên cho các nhà phát triển Go để cải thiện bảo mật cho các dự án của họ.

(Xem thêm: [Các phương pháp hay nhất về bảo mật cho nhà phát triển Go](/security/best-practices).)

## Tìm và khắc phục các lỗ hổng bảo mật đã biết

Công cụ phát hiện lỗ hổng bảo mật của Go hướng đến việc cung cấp các công cụ đáng tin cậy, ít gây nhiễu để các nhà phát triển tìm hiểu về các lỗ hổng bảo mật đã biết có thể ảnh hưởng đến dự án của họ. Để có cái nhìn tổng quan, hãy bắt đầu với [trang tóm tắt và câu hỏi thường gặp này](/security/vuln)
về kiến trúc quản lý lỗ hổng bảo mật của Go. Để tiếp cận theo hướng thực hành, hãy khám phá các công cụ bên dưới.

### Quét mã để tìm lỗ hổng bảo mật bằng govulncheck

Các nhà phát triển có thể sử dụng công cụ govulncheck để xác định liệu bất kỳ lỗ hổng bảo mật đã biết nào có ảnh hưởng đến mã của họ hay không, đồng thời ưu tiên các bước tiếp theo dựa trên những hàm và phương thức có lỗ hổng bảo mật thực sự được gọi.

- [Xem tài liệu govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck)
- [Hướng dẫn: Bắt đầu với govulncheck](/doc/tutorial/govulncheck)

### Phát hiện lỗ hổng bảo mật từ trình soạn thảo của bạn

Phần mở rộng Go cho VS Code kiểm tra các dependency bên thứ ba và hiển thị các lỗ hổng bảo mật liên quan.

- [Tài liệu người dùng](/security/vuln/editor)
- [Tải xuống VS Code Go](https://marketplace.visualstudio.com/items?itemName=golang.go)
- [Hướng dẫn: Bắt đầu với VS Code Go](/doc/tutorial/govulncheck-ide)

### Tìm các module Go để xây dựng dựa trên

[Pkg.go.dev](https://pkg.go.dev/) là một website để khám phá, đánh giá và tìm hiểu thêm về các gói và module Go. Khi khám phá và đánh giá
các gói trên pkg.go.dev, bạn sẽ
[thấy một biểu ngữ ở đầu trang](https://pkg.go.dev/golang.org/x/text@v0.3.7/language)
nếu phiên bản đó có lỗ hổng bảo mật. Ngoài ra, bạn có thể xem
[các lỗ hổng bảo mật ảnh hưởng đến từng phiên bản của một gói](https://pkg.go.dev/golang.org/x/text@v0.3.7/language?tab=versions)
trên trang lịch sử phiên bản.

### Duyệt cơ sở dữ liệu lỗ hổng bảo mật

Cơ sở dữ liệu lỗ hổng bảo mật của Go thu thập dữ liệu trực tiếp từ các nhà duy trì gói Go cũng như từ các nguồn bên ngoài như [MITRE](https://www.cve.org/) và [GitHub](https://github.com/). Các báo cáo
được nhóm Go Security tuyển chọn.

- [Duyệt các báo cáo trong cơ sở dữ liệu lỗ hổng bảo mật của Go](https://pkg.go.dev/vuln/)
- [Xem tài liệu Cơ sở dữ liệu lỗ hổng bảo mật của Go](/security/vuln/database)
- [Đóng góp một lỗ hổng bảo mật công khai vào cơ sở dữ liệu](/s/vulndb-report-new)


## Báo cáo lỗi bảo mật trong dự án Go

### [Chính sách bảo mật](/security/policy)

Tham khảo Chính sách bảo mật để biết hướng dẫn về cách
[báo cáo một lỗ hổng bảo mật trong dự án Go](/security/policy#reporting-a-security-bug).
Trang này cũng trình bày quy trình của nhóm bảo mật Go trong việc theo dõi các vấn đề
và công bố chúng cho công chúng. Xem
[lịch sử bản phát hành](/doc/devel/release) để biết chi tiết về các bản sửa lỗi
bảo mật trước đây. Theo [chính sách bản phát hành](/doc/devel/release#policy),
chúng tôi cung cấp các bản sửa lỗi bảo mật cho hai bản phát hành chính gần đây nhất của Go.

- [Các quyết định phân loại về những lỗ hổng bảo mật thường được báo cáo](/doc/security/decisions)

## Kiểm thử đầu vào không mong muốn bằng fuzzing

Fuzzing gốc của Go cung cấp một kiểu kiểm thử tự động liên tục thao tác với
đầu vào của chương trình để tìm lỗi. Go hỗ trợ fuzzing trong bộ công cụ tiêu chuẩn
bắt đầu từ Go 1.18. Các bài kiểm thử fuzz gốc của Go được
[OSS-Fuzz hỗ trợ](https://google.github.io/oss-fuzz/getting-started/new-project-guide/go-lang/#native-go-fuzzing-support).

- [Xem lại những kiến thức cơ bản về fuzzing](/security/fuzz)
- [Hướng dẫn: Bắt đầu với fuzzing](/doc/tutorial/fuzz)

## Bảo mật dịch vụ bằng các thư viện mật mã của Go

Các thư viện mật mã của Go nhằm giúp nhà phát triển xây dựng các ứng dụng an toàn.
Xem tài liệu về [các gói crypto](https://pkg.go.dev/golang.org/x/crypto)
và [golang.org/x/crypto/](https://pkg.go.dev/golang.org/x/crypto).

## Mật mã tuân thủ FIPS 140-3

Các thư viện mật mã của Go có thể được sử dụng ở chế độ tuân thủ FIPS 140-3 để dùng
trong các môi trường được quản lý. Xem tài liệu [Tuân thủ FIPS 140-3](/doc/security/fips140)
để biết thêm thông tin.
