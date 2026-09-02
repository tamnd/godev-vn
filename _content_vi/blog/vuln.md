---
title: Quản lý lỗ hổng bảo mật cho Go
date: 2022-09-06
by:
- Julie Qiu, for the Go security team
summary: Thông báo về quản lý lỗ hổng bảo mật cho Go, giúp nhà phát triển tìm hiểu về các lỗ hổng bảo mật đã biết trong dependency của họ.
---

Chúng tôi hào hứng thông báo sự hỗ trợ mới của Go cho quản lý lỗ hổng bảo mật, bước đầu tiên giúp các nhà phát triển Go tìm hiểu về các lỗ hổng bảo mật đã biết có thể ảnh hưởng đến họ.

Bài viết này cung cấp tổng quan về những gì có sẵn ngày hôm nay và các bước tiếp theo cho dự án này.

## Tổng quan

Go cung cấp công cụ để phân tích codebase của bạn và phát hiện các lỗ hổng bảo mật đã biết.
Công cụ này được hỗ trợ bởi cơ sở dữ liệu lỗ hổng bảo mật Go, được quản lý bởi nhóm bảo mật Go.
Công cụ của Go giảm tiếng ồn trong kết quả bằng cách chỉ phát hiện các lỗ hổng bảo mật trong các hàm mà code của bạn thực sự gọi.

<div class="image">
  <center>
    <img src="vuln/architecture.png" alt="Sơ đồ kiến trúc hệ thống quản lý lỗ hổng bảo mật của Go"></img>
  </center>
</div>

## Cơ sở dữ liệu lỗ hổng bảo mật Go

Cơ sở dữ liệu lỗ hổng bảo mật Go (https://vuln.go.dev) là nguồn thông tin toàn diện về các lỗ hổng bảo mật đã biết trong các gói có thể import trong các Go module công khai.

Dữ liệu lỗ hổng bảo mật đến từ các nguồn hiện có (như CVE và GHSA) và các báo cáo trực tiếp từ người duy trì gói Go. Thông tin này sau đó được nhóm bảo mật Go xem xét và thêm vào cơ sở dữ liệu.

Chúng tôi khuyến khích người duy trì gói [đóng góp](/s/vulndb-report-new) thông tin về các lỗ hổng bảo mật công khai trong các dự án của họ và [cập nhật](/s/vulndb-report-feedback) thông tin hiện có về các lỗ hổng bảo mật trong các gói Go của họ. Chúng tôi hướng đến làm cho việc báo cáo trở thành một quy trình ít ma sát, vì vậy hãy [gửi cho chúng tôi đề xuất của bạn](/s/vuln-feedback) để cải thiện.

Cơ sở dữ liệu lỗ hổng bảo mật Go có thể được xem trên trình duyệt tại [pkg.go.dev/vuln](https://pkg.go.dev/vuln). Để biết thêm thông tin về cơ sở dữ liệu, hãy xem [go.dev/security/vuln/database](/security/vuln/database).

## Phát hiện lỗ hổng bảo mật sử dụng govulncheck

[Lệnh govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck) mới là cách ít tiếng ồn, đáng tin cậy để người dùng Go tìm hiểu về các lỗ hổng bảo mật đã biết có thể ảnh hưởng đến dự án của họ.
Govulncheck phân tích codebase của bạn và chỉ phát hiện các lỗ hổng bảo mật thực sự ảnh hưởng đến bạn, dựa trên các hàm nào trong code của bạn đang gọi theo cách bắc cầu đến các hàm có lỗ hổng bảo mật.

Bạn có thể cài đặt phiên bản mới nhất của govulncheck bằng [go install](https://pkg.go.dev/cmd/go#hdr-Compile_and_install_packages_and_dependencies):
```
$ go install golang.org/x/vuln/cmd/govulncheck@latest
```

Sau đó, chạy govulncheck trong thư mục dự án của bạn:
```
$ govulncheck ./...
```

Govulncheck là một công cụ độc lập để cho phép cập nhật thường xuyên và lặp đi lặp lại nhanh chóng trong khi chúng tôi thu thập phản hồi từ người dùng. Về lâu dài, chúng tôi có kế hoạch tích hợp công cụ govulncheck vào bản phát hành Go chính.

### Tích hợp

Tốt hơn là tìm hiểu về các lỗ hổng bảo mật sớm nhất có thể trong quá trình phát triển và triển khai. Để tích hợp kiểm tra lỗ hổng bảo mật vào các công cụ và quy trình của riêng bạn, hãy sử dụng [govulncheck -json](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck#hdr-Flags).

Chúng tôi đã tích hợp phát hiện lỗ hổng bảo mật vào các công cụ và dịch vụ Go hiện có, như [trang khám phá gói Go](https://pkg.go.dev). Ví dụ, [trang này](https://pkg.go.dev/golang.org/x/text?tab=versions) hiển thị các lỗ hổng bảo mật đã biết trong từng phiên bản của golang.org/x/text. Tính năng kiểm tra lỗ hổng bảo mật thông qua extension VS Code Go cũng sắp ra mắt.


## Bước tiếp theo

Chúng tôi hy vọng bạn sẽ thấy hỗ trợ quản lý lỗ hổng bảo mật của Go hữu ích và giúp chúng tôi cải thiện nó!

Hỗ trợ quản lý lỗ hổng bảo mật của Go là một tính năng mới đang được phát triển tích cực. Bạn có thể gặp một số lỗi và [hạn chế](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck#hdr-Limitations).

Chúng tôi muốn bạn đóng góp và giúp chúng tôi cải thiện theo những cách sau:

- [Đóng góp thông tin mới](/s/vulndb-report-new) và [cập nhật thông tin hiện có](/s/vulndb-report-feedback) về các lỗ hổng bảo mật công khai cho các gói Go mà bạn duy trì
- [Tham gia khảo sát này](/s/govulncheck-feedback) để chia sẻ trải nghiệm sử dụng govulncheck của bạn
- [Gửi phản hồi cho chúng tôi](/s/vuln-feedback) về các vấn đề và yêu cầu tính năng

Chúng tôi hào hứng hợp tác với bạn để xây dựng hệ sinh thái Go tốt hơn và an toàn hơn.
