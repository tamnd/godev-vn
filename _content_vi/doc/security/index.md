---
title: Bảo mật
layout: article
---

Trang này cung cấp các tài nguyên cho lập trình viên Go để cải thiện bảo mật cho
dự án của họ.

(Xem thêm: [Thực hành bảo mật tốt nhất cho lập trình viên Go](/security/best-practices).)

## Tìm và khắc phục các lỗ hổng bảo mật đã biết

Tính năng phát hiện lỗ hổng bảo mật của Go nhằm mục đích cung cấp các công cụ ít nhiễu, đáng tin cậy cho
lập trình viên để tìm hiểu về các lỗ hổng bảo mật đã biết có thể ảnh hưởng đến dự án của họ.
Để có tổng quan, hãy bắt đầu tại [trang tóm tắt và FAQ này](/security/vuln)
về kiến trúc quản lý lỗ hổng bảo mật của Go. Để có cách tiếp cận thực tế,
hãy khám phá các công cụ bên dưới.

### Quét mã nguồn tìm lỗ hổng bảo mật với govulncheck

Lập trình viên có thể sử dụng công cụ govulncheck để xác định xem có bất kỳ lỗ hổng bảo mật đã biết nào
ảnh hưởng đến mã của họ hay không và ưu tiên các bước tiếp theo dựa trên những hàm và phương thức dễ bị tấn công
nào thực sự được gọi.

- [Xem tài liệu govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck)
- [Hướng dẫn: Bắt đầu với govulncheck](/doc/tutorial/govulncheck)

### Phát hiện lỗ hổng bảo mật từ trình soạn thảo của bạn

Tiện ích mở rộng VS Code Go kiểm tra các dependency bên thứ ba và hiển thị các lỗ hổng bảo mật liên quan.

- [Tài liệu người dùng](/security/vuln/editor)
- [Tải xuống VS Code Go](https://marketplace.visualstudio.com/items?itemName=golang.go)
- [Hướng dẫn: Bắt đầu với VS Code Go](/doc/tutorial/govulncheck-ide)

### Tìm các module Go để xây dựng

[Pkg.go.dev](https://pkg.go.dev/) là một trang web để khám phá, đánh giá và
tìm hiểu thêm về các gói và module Go. Khi khám phá và đánh giá
các gói trên pkg.go.dev, bạn sẽ
[thấy một thông báo ở đầu trang](https://pkg.go.dev/golang.org/x/text@v0.3.7/language)
nếu có lỗ hổng bảo mật trong phiên bản đó. Ngoài ra, bạn có thể xem
[các lỗ hổng bảo mật ảnh hưởng đến từng phiên bản của gói](https://pkg.go.dev/golang.org/x/text@v0.3.7/language?tab=versions)
trên trang lịch sử phiên bản.

### Duyệt cơ sở dữ liệu lỗ hổng bảo mật

Cơ sở dữ liệu lỗ hổng bảo mật Go thu thập dữ liệu trực tiếp từ các nhà duy trì gói Go
cũng như từ các nguồn bên ngoài như [MITRE](https://www.cve.org/) và [GitHub](https://github.com/). Các báo cáo
được chọn lọc bởi nhóm Bảo mật Go.

- [Duyệt báo cáo trong cơ sở dữ liệu lỗ hổng bảo mật Go](https://pkg.go.dev/vuln/)
- [Xem tài liệu Cơ sở dữ liệu lỗ hổng bảo mật Go](/security/vuln/database)
- [Đóng góp lỗ hổng bảo mật công khai vào cơ sở dữ liệu](/s/vulndb-report-new)


## Báo cáo lỗi bảo mật trong dự án Go

### [Chính sách bảo mật](/security/policy)

Tham khảo Chính sách Bảo mật để biết hướng dẫn về cách
[báo cáo lỗ hổng bảo mật trong dự án Go](/security/policy#reporting-a-security-bug).
Trang này cũng trình bày chi tiết quy trình của nhóm bảo mật Go về việc theo dõi vấn đề và
công bố chúng cho công chúng. Xem
[lịch sử bản phát hành](/doc/devel/release) để biết chi tiết về các bản vá bảo mật trong quá khứ.
Theo [chính sách phát hành](/doc/devel/release#policy),
chúng tôi phát hành bản vá bảo mật cho hai bản phát hành chính mới nhất của Go.

- [Quyết định phân loại về các lỗ hổng bảo mật thường được báo cáo](/doc/security/decisions)

## Kiểm thử đầu vào không mong đợi với fuzzing

Go native fuzzing cung cấp một loại kiểm thử tự động liên tục
thao túng các đầu vào cho một chương trình để tìm lỗi. Go hỗ trợ fuzzing trong
toolchain tiêu chuẩn bắt đầu từ Go 1.18. Các bài kiểm thử fuzz Go gốc
[được hỗ trợ bởi OSS-Fuzz](https://google.github.io/oss-fuzz/getting-started/new-project-guide/go-lang/#native-go-fuzzing-support).

- [Xem lại kiến thức cơ bản về fuzzing](/security/fuzz)
- [Hướng dẫn: Bắt đầu với fuzzing](/doc/tutorial/fuzz)

## Bảo mật dịch vụ với các thư viện mật mã của Go

Các thư viện mật mã của Go nhằm giúp lập trình viên xây dựng các ứng dụng an toàn.
Xem tài liệu cho các [gói crypto](https://pkg.go.dev/golang.org/x/crypto)
và [golang.org/x/crypto/](https://pkg.go.dev/golang.org/x/crypto).

## Mật mã tuân thủ FIPS 140-3

Các thư viện mật mã của Go có thể được sử dụng ở chế độ tuân thủ FIPS 140-3 để sử dụng
trong các môi trường bị quản lý. Xem tài liệu [Tuân thủ FIPS 140-3](/doc/security/fips140)
để biết thêm thông tin.
