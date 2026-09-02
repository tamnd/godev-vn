---
title: Quyết định bảo mật của Go
layout: article
breadcrumb: true
---

## Tổng quan {#overview}

Tài liệu này bao gồm các quyết định mà nhóm Go Security đã đưa ra về
nhiều vấn đề thường được báo cáo. Phần lớn tài liệu này đóng vai trò
tham chiếu cho những điều chúng tôi không xem là một lỗ hổng bảo mật.

Danh sách này không đầy đủ.

## Nguyên tắc chung {#principles}

Không phải mọi lỗi đều là một lỗ hổng bảo mật.

Đối với bất kỳ lỗi nào, hầu như luôn có thể xây dựng một kịch bản
giả định trong đó lỗi đó gây ra một lỗ hổng bảo mật.

Ví dụ, hãy tưởng tượng một lỗi trong bộ giải mã ảnh khiến nó
giải mã không chính xác một số đầu vào. Bây giờ hãy tưởng tượng một hệ thống
giả định chấp nhận hình ảnh của các tờ séc viết tay và thực hiện chuyển khoản
ngân hàng để đáp ứng các tờ séc đó. Nếu hệ thống này chứa một cổng
xác thực yêu cầu và một backend thực hiện việc chuyển khoản, một
sự khác biệt trong bộ phân tích cú pháp giữa các bộ giải mã ảnh được
cổng và backend sử dụng có thể khiến cổng chuyển một thao tác độc hại
sang backend.

Việc phân loại lỗi phân tích sai ảnh là một lỗ hổng bảo mật dựa trên
kịch bản này không mang lại mục đích hữu ích nào. Cảnh báo người dùng rằng họ
bị ảnh hưởng bởi một "lỗ hổng bảo mật" thuộc loại này chỉ tạo ra nhiễu,
vì rất ít hoặc không có người dùng nào thực sự bị ảnh hưởng. Gọi đây là một
lỗ hổng bảo mật gây bất lợi cho bảo mật, vì nó khiến người dùng
bỏ qua các báo cáo do xem chúng phần lớn là vô nghĩa.

Kịch bản này cũng là một ví dụ về lý do chúng tôi không gán nhãn mức độ
nghiêm trọng cho các vấn đề bảo mật: Đối với bộ xử lý séc giả định,
đây có thể là một vấn đề nghiêm trọng, nhưng đối với hầu hết người dùng thì
nó hoàn toàn không phải là vấn đề.

Theo nguyên tắc chung, chỉ việc có thể xây dựng một
kịch bản giả định trong đó một lỗi gây ra vấn đề bảo mật
không đủ để chúng tôi xử lý chính lỗi đó như một vấn đề bảo mật.

## Lỗ hổng bảo mật {#vulns}

### Thực thi mã từ xa {#rce}

Một kịch bản cho phép kẻ tấn công thực thi mã trong tình huống
mà việc thực thi mã không được mong đợi là một lỗ hổng bảo mật thuộc
PRIVATE-track. Quyết định này được ưu tiên hơn tất cả các quyết định khác.

Quyết định này không bao gồm các hàm được mong đợi là sẽ thực thi mã.

Được bao gồm:

- Một hàm phân tích cú pháp thực thi mã với đầu vào độc hại.
- Một yêu cầu độc hại khiến máy chủ HTTP thực thi mã.

Không được bao gồm:

- Lệnh `go test` chạy các bài kiểm thử. Việc chạy `go test` với các
  bài kiểm thử do kẻ tấn công kiểm soát không nằm trong mô hình mối đe dọa của chúng tôi.

### Hoảng loạn {#panics}

Một panic khi xử lý đầu vào do kẻ tấn công kiểm soát có thể là một lỗ hổng bảo mật.

Một panic trong một máy chủ được kỳ vọng xử lý các yêu cầu do kẻ tấn công kiểm soát, chẳng hạn như máy chủ `net/http`, thường là lỗ hổng bảo mật theo luồng PRIVATE.

Một panic trong một client, chẳng hạn như client `net/http`, thường là lỗ hổng bảo mật theo luồng PUBLIC.

Một panic trong một hàm phân tích cú pháp của một gói xử lý đầu vào có khả năng độc hại, chẳng hạn như `archive/zip` hoặc `image/png`, thường là lỗ hổng bảo mật theo luồng PUBLIC.

Một đầu vào gây ra panic do kích thước tự nhiên của nó, chẳng hạn như một hình ảnh rất lớn, thường không được phân loại là lỗ hổng bảo mật.

Các đầu vào không hợp lệ cho những hàm không được thiết kế rõ ràng để phân tích dữ liệu có khả năng độc hại không nằm trong mô hình mối đe dọa của chúng tôi và nhìn chung nằm ngoài phạm vi của các lỗi bảo mật. Ví dụ, các *trình phân tích* hình ảnh được kỳ vọng bảo vệ trước các đầu vào không hợp lệ, nhưng một panic trong một *bộ mã hóa* hình ảnh có thể là lỗi nhưng sẽ không được xử lý như một lỗ hổng bảo mật.

Một panic trong một hàm đã lỗi thời không phải là một lỗ hổng bảo mật; xem quyết định về [API đã lỗi thời](#deprecated) bên dưới.

### Tiêu thụ tài nguyên quá mức {#quadratic}

Nhìn chung, chúng tôi xem việc tiêu thụ CPU hoặc bộ nhớ quá mức, chẳng hạn như một hàm có thời gian chạy là O(n²) theo kích thước đầu vào, tương đương với một panic.

### Xây dựng mã độc {#malicious-build}

Việc xây dựng một chương trình do kẻ tấn công kiểm soát phải an toàn, ngay cả khi việc chạy chương trình đó không an toàn. Ví dụ, chúng tôi dự định việc xây dựng mã không đáng tin cậy rồi chạy trong sandbox phải an toàn. Chúng tôi đặc biệt khuyến nghị bất kỳ ai xây dựng mã không đáng tin cậy xem đây là một biện pháp phòng thủ nhiều lớp, và việc xây dựng mã không đáng tin cậy cũng nên được thực hiện trong môi trường sandbox không có đặc quyền.

Việc trích xuất dữ liệu không nằm trong mô hình mối đe dọa của chúng tôi. Việc xây dựng một chương trình do kẻ tấn công kiểm soát có thể tạo ra đầu ra chứa nội dung của các tệp cục bộ bất kỳ.

Việc *thực thi* mã độc cũng không nằm trong mô hình mối đe dọa của chúng tôi. Chạy một chương trình độc hại rõ ràng cho phép chương trình đó thực hiện các hành động độc hại.

Một lỗ hổng bảo mật, thường là theo luồng PRIVATE:

- Một module độc hại có thể khiến "go build" thực thi mã tùy ý.

Không được xử lý như một vấn đề bảo mật:

- "go build" tạo ra lỗi chứa văn bản từ một tệp cục bộ bất kỳ.
- Một tệp thực thi đã biên dịch chứa nội dung từ một tệp cục bộ bất kỳ.
- Một chương trình độc hại có thể làm hỏng trạng thái của runtime và thực thi mã tùy ý.
- Một lỗi biên dịch cho phép làm rối mã độc.

## Non-Vulnerabilities {#non-vuln}

### Attacker-controlled environment {#attacker-control}

If an attack relies on the attacker having control over the environment
a program runs in, it is not a vulnerability.

This includes, but is not restricted to, an attacker with the ability to
add programs to `$PATH` or set arbitrary environment variables.

### Deprecated APIs {#deprecated}

The bar for a bug in a deprecated API to be considered a security issue
is higher than it is for a supported API.

In particular, a panic or excessive resource consumption in a deprecated API
is never handled as a vulnerability. Users who require security hardening
should use supported APIs.

### image, x/image: Large images {#large-image}

Parsing a large image can allocate a large amount of memory.
For example, a 65536x65536 32-bit color image requires 16MiB
to store uncompressed.

Many image compression formats can reduce a large, simple image
to a very small file size. Decoding the small file may allocate
a large amount of memory.

Users parsing untrusted images should verify the image size prior
to parsing, using a function such as
[image.DecodeConfig](https://pkg.go.dev/image#DecodeConfig).

We do not consider it to be a vulnerability for an image parsing
function to decode a large, well-compressed image.

### net/http: Redirects {#http-redirect}

The `net/http` package's HTTP client handles redirects.
It implements security-relevant behavior in redirect handling.
For example, it strips the "Authorization" header when following
a redirect to a domain that is not a subdomain or exact match
for the initial request's domain.

Header stripping is a defense-in-depth measure,
avoiding the case where a misconfigured or compromised server
inadvertently forwards a client request containing sensitive headers
to an untrusted destination.
Failure to strip headers on redirect does not, by itself, permit
an attacker to acquire credentials passed in header.
However, it may be combined with other vulnerabilities
(for example, a server with an open redirect vulnerability)
to do so.

Changing the HTTP client's behavior runs a high risk of breaking
existing users who depend on the current behavior.
For example, the client's same origin policy currently permits subdomains
(a redirect from example.com to www.example.com will preserve headers),
while the WHATWG Fetch standard does not.
Aligning the client with the standard may be worthwhile,
but doing so in a security release is more likely to cause
pain to existing users than it is to address real vulnerabilities.

Since redirect sanitization is a defense-in-depth measure,
and making changes to it is risky,
we consider all aspects of the HTTP client redirects
to be out of scope for the security bug process.
