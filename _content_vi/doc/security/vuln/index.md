---
title: Quản lý lỗ hổng bảo mật Go
layout: article
---

[Quay lại Bảo mật Go](/security)

## Tổng quan

Go giúp lập trình viên phát hiện, đánh giá và giải quyết các lỗi hoặc điểm yếu có
nguy cơ bị khai thác bởi kẻ tấn công. Phía sau, nhóm Go chạy một
quy trình để chọn lọc các báo cáo về lỗ hổng bảo mật, được lưu trữ trong cơ sở dữ liệu
lỗ hổng bảo mật Go. Các thư viện và công cụ khác nhau có thể đọc và phân tích các báo cáo đó
để hiểu cách các dự án người dùng cụ thể có thể bị ảnh hưởng. Chức năng
này được tích hợp vào
[trang khám phá gói Go](https://pkg.go.dev) và một công cụ CLI mới,
govulncheck.

Dự án này đang trong quá trình phát triển và đang được tích cực phát triển.
Chúng tôi hoan nghênh [phản hồi](#feedback) của bạn để giúp chúng tôi cải thiện!

**LƯU Ý**: Để báo cáo lỗ hổng bảo mật trong dự án Go, hãy xem [Chính sách Bảo mật Go](/security/policy).

## Kiến trúc

<div class="image">
  <center>
    <img style="width: 100%" width="2110" height="952" src="architecture.png" alt="Go Vulnerability Management Architecture"></img>
  </center>
</div>

Quản lý lỗ hổng bảo mật trong Go bao gồm các phần cấp cao sau:

1. Một **pipeline dữ liệu** thu thập thông tin lỗ hổng bảo mật từ các nguồn khác nhau,
bao gồm [Cơ sở dữ liệu lỗ hổng bảo mật quốc gia (NVD)](https://nvd.nist.gov/),
[Cơ sở dữ liệu khuyến nghị GitHub](https://github.com/advisories),
và [trực tiếp từ các nhà duy trì gói Go](/s/vulndb-report-new).
2. Một **cơ sở dữ liệu lỗ hổng bảo mật** được điền báo cáo bằng thông tin
từ pipeline dữ liệu.
Tất cả các báo cáo trong cơ sở dữ liệu được xem xét và chọn lọc bởi nhóm Bảo mật Go.
Các báo cáo được định dạng theo [định dạng Open Source Vulnerability (OSV)](https://ossf.github.io/osv-schema/)
và có thể truy cập thông qua [API](/security/vuln/database#api).
3. **Tích hợp** với [pkg.go.dev](https://pkg.go.dev)
và govulncheck để cho phép lập trình viên tìm lỗ hổng bảo mật trong
các dự án của họ. [Lệnh govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck)
phân tích codebase của bạn và chỉ hiển thị các lỗ hổng bảo mật thực sự ảnh hưởng
đến bạn, dựa trên các hàm nào trong mã của bạn đang gián tiếp gọi các hàm dễ bị tấn công.
Govulncheck cung cấp một cách ít nhiễu, đáng tin cậy để tìm các lỗ hổng bảo mật đã biết
trong các dự án của bạn.

## Tài nguyên

### Cơ sở dữ liệu lỗ hổng bảo mật Go

[Cơ sở dữ liệu lỗ hổng bảo mật Go](https://vuln.go.dev) chứa thông tin
từ nhiều nguồn hiện có ngoài các báo cáo trực tiếp từ các nhà duy trì gói Go
đến nhóm bảo mật Go.
Mỗi mục trong cơ sở dữ liệu được xem xét để đảm bảo mô tả lỗ hổng bảo mật,
thông tin gói và ký hiệu, và chi tiết phiên bản đều chính xác.

Xem [go.dev/security/vuln/database](/security/vuln/database) để biết thêm thông tin
về cơ sở dữ liệu lỗ hổng bảo mật Go,
và [pkg.go.dev/vuln](https://pkg.go.dev/vuln) để xem lỗ hổng bảo mật trong
cơ sở dữ liệu trên trình duyệt của bạn.

Chúng tôi khuyến khích các nhà duy trì gói [đóng góp](#feedback)
thông tin về các lỗ hổng bảo mật công khai trong các dự án của riêng họ và
[gửi cho chúng tôi các đề xuất](/s/vuln-feedback) về cách giảm
rào cản.

### Phát hiện lỗ hổng bảo mật cho Go

Tính năng phát hiện lỗ hổng bảo mật của Go nhằm cung cấp một cách ít nhiễu, đáng tin cậy cho người dùng Go
để tìm hiểu về các lỗ hổng bảo mật đã biết có thể ảnh hưởng đến dự án của họ.
Kiểm tra lỗ hổng bảo mật được tích hợp vào các công cụ và dịch vụ của Go, bao gồm
một công cụ dòng lệnh mới, [govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck),
[trang khám phá gói Go](https://pkg.go.dev), [các trình soạn thảo chính](/security/vuln/editor) như VS Code với tiện ích mở rộng Go.

Để bắt đầu sử dụng govulncheck, chạy lệnh sau từ dự án của bạn:

```
$ go install golang.org/x/vuln/cmd/govulncheck@latest
$ govulncheck ./...
```

Để bật phát hiện lỗ hổng bảo mật trong trình soạn thảo của bạn, xem hướng dẫn trong trang [tích hợp trình soạn thảo](/security/vuln/editor).

### Go CNA

Nhóm bảo mật Go là một [Cơ quan Đánh số CVE](https://www.cve.org/ProgramOrganization/CNAs).
Xem [go.dev/security/vuln/cna](/security/vuln/cna) để biết thêm thông tin.

## Phản hồi

Chúng tôi rất mong bạn đóng góp và giúp chúng tôi cải thiện theo
các cách sau:

- [Đóng góp mới](/s/vulndb-report-new) và
  [cập nhật thông tin hiện có](/s/vulndb-report-feedback) về
  các lỗ hổng bảo mật công khai cho các gói Go bạn duy trì
- [Tham gia khảo sát này](/s/govulncheck-feedback) để chia sẻ
  kinh nghiệm sử dụng govulncheck của bạn
- [Gửi phản hồi cho chúng tôi](/s/vuln-feedback) về các vấn đề và
  yêu cầu tính năng

## Câu hỏi thường gặp

**Làm thế nào tôi báo cáo lỗ hổng bảo mật trong dự án Go?**

Báo cáo tất cả các lỗi bảo mật trong dự án Go qua email đến [security@golang.org](mailto:security@golang.org).
Đọc [Chính sách bảo mật Go](/security/policy) để biết thêm thông tin về quy trình của chúng tôi.

**Làm thế nào tôi thêm lỗ hổng bảo mật công khai vào cơ sở dữ liệu lỗ hổng bảo mật Go?**

Để yêu cầu thêm một lỗ hổng bảo mật công khai vào cơ sở dữ liệu lỗ hổng bảo mật Go,
[điền vào biểu mẫu này](/s/vulndb-report-new).

Một lỗ hổng bảo mật được coi là công khai nếu nó đã được công bố công khai,
hoặc nếu nó tồn tại trong một gói bạn duy trì (và bạn đã sẵn sàng công bố nó).
Biểu mẫu chỉ dành cho các lỗ hổng bảo mật công khai trong các gói Go có thể import mà
không được duy trì bởi Nhóm Go (bất cứ thứ gì ngoài thư viện chuẩn Go,
toolchain Go và các module golang.org).

Biểu mẫu cũng có thể được sử dụng để yêu cầu ID CVE mới.
[Đọc thêm tại đây](/security/vuln/cna) về Cơ quan Đánh số CVE của Go.

**Làm thế nào tôi đề xuất chỉnh sửa một lỗ hổng bảo mật?**

Để đề xuất chỉnh sửa một báo cáo hiện có trong cơ sở dữ liệu lỗ hổng bảo mật Go,
[điền vào biểu mẫu tại đây](/s/vulndb-report-feedback).

**Làm thế nào tôi báo cáo vấn đề hoặc đưa ra phản hồi về govulncheck?**

Gửi vấn đề hoặc phản hồi của bạn [trên trình theo dõi vấn đề Go](/s/vuln-feedback).

**Tôi đã tìm thấy lỗ hổng bảo mật này trong một cơ sở dữ liệu khác. Tại sao nó không có trong cơ sở dữ liệu lỗ hổng bảo mật Go?**

Các báo cáo có thể bị loại trừ khỏi cơ sở dữ liệu lỗ hổng bảo mật Go vì nhiều lý do,
bao gồm lỗ hổng bảo mật liên quan không có trong gói Go,
lỗ hổng bảo mật nằm trong lệnh có thể cài đặt thay vì gói có thể import,
hoặc lỗ hổng bảo mật được bao gồm trong một lỗ hổng bảo mật khác đã có
trong cơ sở dữ liệu.
Bạn có thể tìm hiểu thêm về
[lý do của nhóm Bảo mật Go khi loại trừ báo cáo tại đây](/security/vuln/database#excluded-reports).
Nếu bạn nghĩ rằng một báo cáo đã bị loại trừ không chính xác khỏi vuln.go.dev,
[hãy cho chúng tôi biết](/s/vulndb-report-feedback).

**Tại sao cơ sở dữ liệu lỗ hổng bảo mật Go không sử dụng nhãn mức độ nghiêm trọng?**

Hầu hết các định dạng báo cáo lỗ hổng bảo mật sử dụng nhãn mức độ nghiêm trọng như "LOW," "MEDIUM",
và "CRITICAL" để chỉ ra tác động của các lỗ hổng bảo mật khác nhau và
giúp lập trình viên ưu tiên các vấn đề bảo mật.
Tuy nhiên, vì một số lý do, Go tránh sử dụng các nhãn như vậy.

Tác động của một lỗ hổng bảo mật hiếm khi mang tính phổ quát,
có nghĩa là các chỉ số mức độ nghiêm trọng thường có thể gây hiểu lầm.
Ví dụ: một crash trong một trình phân tích có thể là vấn đề có mức độ nghiêm trọng cao nếu
nó được sử dụng để phân tích đầu vào do người dùng cung cấp và có thể được khai thác trong cuộc tấn công DoS,
nhưng nếu trình phân tích được sử dụng để phân tích các tệp cấu hình cục bộ,
ngay cả khi gọi mức độ nghiêm trọng là "thấp" cũng có thể là phóng đại.

Gắn nhãn mức độ nghiêm trọng cũng nhất thiết mang tính chủ quan.
Điều này đúng ngay cả với [chương trình CVE](https://www.cve.org/About/Overview),
đặt ra một công thức để phân tích các khía cạnh liên quan của lỗ hổng bảo mật,
chẳng hạn như vectơ tấn công, độ phức tạp và khả năng khai thác.
Tuy nhiên, tất cả những điều này đòi hỏi đánh giá chủ quan.

Chúng tôi tin rằng mô tả tốt về lỗ hổng bảo mật hữu ích hơn các chỉ số mức độ nghiêm trọng.
Một mô tả tốt có thể phân tích vấn đề là gì,
cách nó có thể được kích hoạt, và những gì người tiêu dùng nên xem xét khi xác định
tác động lên phần mềm của riêng họ.

Hãy thoải mái [gửi vấn đề](/s/vuln-feedback)
nếu bạn muốn chia sẻ suy nghĩ của mình với chúng tôi về chủ đề này.
