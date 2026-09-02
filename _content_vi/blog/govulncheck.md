---
title: Govulncheck v1.0.0 đã được phát hành!
date: 2023-07-13
by:
- Julie Qiu, for the Go security team
summary: Phiên bản v1.0.0 của golang.org/x/vuln đã được phát hành, giới thiệu API mới và các cải tiến khác.
---

Chúng tôi vui mừng thông báo rằng govulncheck v1.0.0 đã được phát hành,
cùng với v1.0.0 của API để tích hợp quét lỗ hổng vào các công cụ khác!

Hỗ trợ quản lý lỗ hổng của Go đã được [thông báo lần đầu](/blog/vuln) vào tháng 9 năm ngoái.
Chúng tôi đã thực hiện một số thay đổi kể từ đó, đỉnh điểm là bản phát hành hôm nay.

Bài đăng này mô tả công cụ lỗ hổng Go đã được cập nhật và cách bắt đầu
sử dụng nó. Chúng tôi cũng vừa xuất bản một
[hướng dẫn thực hành bảo mật tốt nhất](/security/best-practices)
để giúp bạn ưu tiên bảo mật trong các dự án Go của mình.

## Govulncheck

[Govulncheck](https://golang.org/x/vuln/cmd/govulncheck)
là một công cụ dòng lệnh giúp người dùng Go tìm các lỗ hổng đã biết trong
các dependency của dự án của họ.
Công cụ này có thể phân tích cả codebase lẫn binary,
và nó giảm nhiễu bằng cách ưu tiên các lỗ hổng trong các hàm mà
code của bạn thực sự đang gọi.

Bạn có thể cài đặt phiên bản mới nhất của govulncheck bằng
[go install](https://pkg.go.dev/cmd/go#hdr-Compile_and_install_packages_and_dependencies):

```
go install golang.org/x/vuln/cmd/govulncheck@latest
```

Sau đó, chạy govulncheck bên trong module của bạn:
```
govulncheck ./...
```

Xem [hướng dẫn govulncheck](/doc/tutorial/govulncheck)
để biết thêm thông tin về cách bắt đầu sử dụng công cụ.

Kể từ bản phát hành này, giờ đây có sẵn một API ổn định,
được mô tả tại [golang.org/x/vuln/scan](https://golang.org/x/vuln/scan).
API này cung cấp chức năng tương tự như lệnh govulncheck,
cho phép nhà phát triển tích hợp trình quét bảo mật và các công cụ khác với govulncheck.
Ví dụ, xem
[tích hợp osv-scanner với govulncheck](https://github.com/google/osv-scanner/blob/d93d6b73e90ae392fe2b1b64a33bda6976b65b2d/internal/sourceanalysis/go.go#L20).

## Cơ sở dữ liệu

Govulncheck được hỗ trợ bởi cơ sở dữ liệu lỗ hổng Go, [https://vuln.go.dev](https://vuln.go.dev),
cung cấp nguồn thông tin toàn diện về các lỗ hổng đã biết
trong các module Go công khai.
Bạn có thể duyệt các mục trong cơ sở dữ liệu tại [pkg.go.dev/vuln](https://pkg.go.dev/vuln).

Kể từ bản phát hành đầu tiên, chúng tôi đã cập nhật [API cơ sở dữ liệu](/security/vuln/database#api)
để cải thiện hiệu năng và đảm bảo khả năng mở rộng lâu dài.
Một công cụ thử nghiệm để tạo chỉ mục cơ sở dữ liệu lỗ hổng của riêng bạn được
cung cấp tại [golang.org/x/vulndb/cmd/indexdb](https://golang.org/x/vulndb/cmd/indexdb).

Nếu bạn là người bảo trì gói Go, chúng tôi khuyến khích bạn
[đóng góp thông tin](/s/vulndb-report-new)
về các lỗ hổng công khai trong dự án của bạn.

Để biết thêm thông tin về cơ sở dữ liệu lỗ hổng Go,
xem [go.dev/security/vuln/database](/security/vuln/database).

## Tích hợp

Phát hiện lỗ hổng giờ đã được tích hợp vào bộ công cụ mà nhiều
quy trình làm việc của nhà phát triển Go đã sử dụng.

Dữ liệu từ cơ sở dữ liệu lỗ hổng Go có thể được duyệt tại
[pkg.go.dev/vuln](https://pkg.go.dev/vuln).
Thông tin lỗ hổng cũng được hiển thị trên các trang tìm kiếm và gói
của pkg.go.dev. Ví dụ,
[trang phiên bản của golang.org/x/text/language](https://pkg.go.dev/golang.org/x/text/language?tab=versions)
hiển thị các lỗ hổng trong các phiên bản cũ hơn của module.

Bạn cũng có thể chạy govulncheck trực tiếp trong trình soạn thảo của mình bằng extension Go
cho Visual Studio Code.
Xem [hướng dẫn](/doc/tutorial/govulncheck-ide) để bắt đầu.

Cuối cùng, chúng tôi biết rằng nhiều nhà phát triển sẽ muốn chạy govulncheck như một phần
của hệ thống CI/CD của họ.
Để bắt đầu, chúng tôi đã cung cấp một
[GitHub Action cho govulncheck](https://github.com/marketplace/actions/golang-govulncheck-action)
để tích hợp với các dự án của bạn.

## Hướng dẫn video

Nếu bạn quan tâm đến một bản demo về các tích hợp được mô tả ở trên,
chúng tôi đã trình bày hướng dẫn về các công cụ này tại Google I/O năm nay, trong bài nói chuyện,
[Build more secure apps with Go and Google](https://www.youtube.com/watch?v=HSt6FhsPT8c&ab_channel=TheGoProgrammingLanguage).

## Phản hồi

Như mọi khi, chúng tôi hoan nghênh phản hồi của bạn! Xem chi tiết về
[cách đóng góp và giúp chúng tôi cải thiện](/security/vuln/#feedback).

Chúng tôi hy vọng bạn sẽ thấy bản phát hành mới nhất về hỗ trợ quản lý lỗ hổng
của Go hữu ích và cùng làm việc với chúng tôi để xây dựng một
hệ sinh thái Go an toàn và đáng tin cậy hơn.
