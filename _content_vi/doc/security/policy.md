---
title: Chính sách bảo mật Go
layout: article
breadcrumb: true
---

## Tổng quan

Tài liệu này giải thích quy trình của nhóm Go Security trong việc xử lý các vấn đề được báo cáo và những gì bạn có thể mong đợi nhận được phản hồi.

## Báo cáo lỗi bảo mật

Tất cả các lỗi bảo mật trong bản phân phối Go nên được báo cáo qua email đến [security@golang.org](mailto:security@golang.org). Thư này được chuyển đến nhóm Go Security.

Vui lòng định dạng tiêu đề email của bạn là "Vulnerability: {package name}: {one-line summary}", và tránh gửi tệp đính kèm trong email trừ khi thực sự cần thiết để tránh việc thư của bạn bị đánh dấu là spam.

Nếu muốn, vui lòng đưa thông tin ghi nhận tác giả mà bạn mong muốn vào báo cáo. Thông tin ghi nhận này sẽ được thêm vào CVE nếu một CVE được công bố.

Vui lòng giữ báo cáo ngắn gọn, bao gồm mô tả về vấn đề bạn tìm thấy, cách bạn tin rằng vấn đề đó có thể bị khai thác, và một trường hợp kiểm thử tái tạo nhỏ hoặc chương trình minh họa vấn đề.

Chúng tôi có [danh sách quyết định](/doc/security/decisions) về các nhóm vấn đề thường được báo cáo.

Email của bạn sẽ được xác nhận trong vòng 7 ngày. Nếu bạn chưa nhận được phản hồi sau thời gian đó, vui lòng liên hệ lại với nhóm Go Security tại [security@golang.org](mailto:security@golang.org). Hãy đảm bảo từ **lỗ hổng bảo mật** xuất hiện trong email của bạn.

Nếu sau thêm 3 ngày bạn vẫn chưa nhận được xác nhận về báo cáo, có khả năng email của bạn đã bị đánh dấu là spam. Trong trường hợp đó, vui lòng [gửi issue tại đây](https://g.co/vulnz). Chọn "I want to report a technical security or an abuse risk related bug in a Google product (SQLi, XSS, etc.)", và liệt kê "Go" là sản phẩm bị ảnh hưởng.

Issue của bạn sẽ được sửa hoặc công khai trong vòng 90 ngày sau lần xác nhận ban đầu của chúng tôi.

Các email báo cáo vấn đề bảo mật sẽ được lưu giữ vô thời hạn để theo dõi trạng thái khắc phục và ghi nhận chính xác các phát hiện.

## Luồng xử lý

Tùy thuộc vào bản chất của vấn đề, nhóm Go Security sẽ phân loại vấn đề đó là issue thuộc luồng PUBLIC, PRIVATE hoặc URGENT. Tất cả các vấn đề bảo mật sẽ được cấp số CVE.

Nhóm Go Security không gán các nhãn mức độ nghiêm trọng truyền thống với độ chi tiết cao (ví dụ CRITICAL, HIGH, MEDIUM, LOW) cho các vấn đề bảo mật vì mức độ nghiêm trọng phụ thuộc rất nhiều vào cách người dùng sử dụng API hoặc chức năng bị ảnh hưởng. Ngoài ra, khi cấp CVE cho các vấn đề bảo mật của Go, chúng tôi không gán điểm CVSS, vì về cơ bản chúng tôi không đồng ý với khả năng áp dụng của hệ thống chấm điểm này cho Go vì cùng những lý do đó. Các bên thứ ba, chẳng hạn như MITRE hoặc NIST, thông qua NVD, có thể gán điểm CVSS cho các lỗ hổng bảo mật của chúng tôi, nhưng chúng tôi không xác nhận rằng các điểm số này phản ánh chính xác tác động của chúng.

Ví dụ, tác động của vấn đề cạn kiệt tài nguyên trong bộ phân tích cú pháp `encoding/json` phụ thuộc vào dữ liệu đang được phân tích. Nếu người dùng phân tích các tệp JSON đáng tin cậy từ hệ thống tệp cục bộ của họ, tác động có khả năng là thấp. Nếu người dùng phân tích JSON tùy ý không đáng tin cậy từ phần thân của một yêu cầu HTTP, tác động có thể cao hơn nhiều.

Tuy vậy, các luồng xử lý issue sau đây thể hiện mức độ nghiêm trọng và/hoặc phạm vi ảnh hưởng mà nhóm Security đánh giá cho một vấn đề. Ví dụ, một vấn đề có tác động từ trung bình đến đáng kể đối với nhiều người dùng là issue thuộc luồng PRIVATE trong chính sách này, còn một vấn đề có tác động không đáng kể đến nhỏ, hoặc chỉ ảnh hưởng đến một tập hợp nhỏ người dùng, là issue thuộc luồng PUBLIC.

### PUBLIC

Các vấn đề trong track PUBLIC ảnh hưởng đến những cấu hình đặc thù, có tác động rất hạn chế, hoặc đã được biết đến rộng rãi.

Các vấn đề trong track PUBLIC được gắn nhãn
[`Proposal-Security`](https://github.com/golang/go/labels/Proposal-Security),
được thảo luận thông qua
[quy trình xem xét đề xuất của Go](https://go.googlesource.com/proposal/+/master/README.md#proposal-review)
và **được sửa công khai**, sau đó được backport vào [các bản phát hành
minor](/wiki/MinorReleases) tiếp theo theo lịch trình (diễn ra khoảng hàng tháng). Thông báo bản phát hành bao gồm chi tiết về các vấn đề này, nhưng không có thông báo trước.

Ví dụ về các vấn đề PUBLIC trước đây:

- [#44916](/issue/44916): archive/zip: có thể panic khi gọi Reader.Open
- [#44913](/issue/44913): encoding/xml: vòng lặp vô hạn khi sử dụng xml.NewTokenDecoder với TokenReader tùy chỉnh
- [#43786](/issue/43786): crypto/elliptic: các thao tác không chính xác trên đường cong P-224
- [#40928](/issue/40928): net/http/cgi,net/http/fcgi: Cross-Site Scripting (XSS) khi Content-Type không được chỉ định
- [#40618](/issue/40618): encoding/binary: ReadUvarint và ReadVarint có thể đọc số byte không giới hạn từ các đầu vào không hợp lệ
- [#36834](/issue/36834): crypto/x509: bỏ qua xác thực chứng chỉ trên Windows 10

### PRIVATE

Các vấn đề trong track PRIVATE là những vi phạm đối với các thuộc tính bảo mật đã cam kết.

Các vấn đề trong track PRIVATE được **sửa trong [các bản phát hành
minor](/wiki/MinorReleases) tiếp theo theo lịch trình**, và được giữ kín cho đến thời điểm đó.

Ba đến bảy ngày trước bản phát hành, một thông báo trước được gửi đến
golang-announce, thông báo về sự xuất hiện của một hoặc nhiều bản sửa lỗi bảo mật trong các bản phát hành sắp tới, cũng như liệu các vấn đề này ảnh hưởng đến thư viện chuẩn, toolchain, hay cả hai, cùng với các ID CVE được dành riêng cho từng bản sửa lỗi.

Đối với các vấn đề xuất hiện trong một [release candidate của bản phát hành lớn](/s/release),
chúng tôi tuân theo cùng quy trình, bao gồm các bản sửa lỗi trong release candidate tiếp theo theo lịch trình.

Một số ví dụ về các vấn đề PRIVATE trước đây:

- [#53416](/issue/53416): path/filepath: cạn kiệt stack trong Glob
- [#53616](/issue/53616): go/parser: cạn kiệt stack trong tất cả các hàm Parse*
- [#54658](/issue/54658): net/http: xử lý lỗi máy chủ sau khi gửi GOAWAY
- [#56284](/issue/56284): syscall, os/exec: NUL chưa được làm sạch trong các biến môi trường

### KHẨN CẤP

Các vấn đề thuộc luồng KHẨN CẤP là mối đe dọa đến tính toàn vẹn của hệ sinh thái Go, hoặc đang bị khai thác tích cực trong thực tế dẫn đến thiệt hại nghiêm trọng. Hiện không có ví dụ gần đây, nhưng chúng sẽ bao gồm việc thực thi mã từ xa trong net/http, hoặc khả năng khôi phục khóa trong thực tế ở crypto/tls.

Các vấn đề thuộc luồng KHẨN CẤP được xử lý riêng tư và **kích hoạt một bản phát hành bảo mật chuyên biệt ngay lập tức**, có thể không có thông báo trước.

## Đánh dấu các vấn đề hiện có là liên quan đến bảo mật

Nếu bạn cho rằng một [vấn đề hiện có](/issue) có liên quan đến bảo mật, chúng tôi đề nghị bạn gửi email đến [security@golang.org](mailto:security@golang.org). Email nên bao gồm ID của vấn đề và mô tả ngắn gọn lý do vì sao vấn đề đó nên được xử lý theo chính sách bảo mật này.

## Quy trình công khai thông tin

Dự án Go sử dụng quy trình công khai thông tin sau:

1. Khi báo cáo bảo mật được nhận, báo cáo đó sẽ được giao cho một người xử lý chính. Người này điều phối quá trình sửa lỗi và bản phát hành.

2. Vấn đề được xác nhận và danh sách phần mềm bị ảnh hưởng được xác định.

3. Mã nguồn được kiểm tra để tìm các vấn đề tương tự tiềm ẩn.

4. Nếu sau khi trao đổi với người gửi báo cáo xác định rằng cần có số CVE, người xử lý chính sẽ lấy một số.

5. Các bản sửa lỗi được chuẩn bị cho hai bản phát hành chính gần đây nhất và bản sửa đổi head/master. Các bản sửa lỗi được chuẩn bị cho hai bản phát hành chính gần đây nhất và được hợp nhất vào head/master.

6. Vào ngày các bản sửa lỗi được áp dụng, thông báo được gửi đến [golang-announce](https://groups.google.com/group/golang-announce), [golang-dev](https://groups.google.com/group/golang-dev), và [golang-nuts](https://groups.google.com/group/golang-nuts).

Quy trình này có thể mất một khoảng thời gian, đặc biệt khi cần phối hợp với những người bảo trì của các dự án khác. Mọi nỗ lực sẽ được thực hiện để xử lý lỗi trong thời gian nhanh nhất có thể, tuy nhiên điều quan trọng là chúng ta tuân theo quy trình được mô tả ở trên để đảm bảo việc công khai thông tin được xử lý nhất quán.

Đối với các vấn đề bảo mật bao gồm việc gán số CVE, vấn đề đó được liệt kê công khai dưới
[sản phẩm "Golang" trên trang web CVEDetails](https://www.cvedetails.com/vulnerability-list/vendor_id-14185/Golang.html)
cũng như trên
[trang web National Vulnerability Disclosure](https://web.nvd.nist.gov/view/vuln/search).

## Nhận các bản cập nhật bảo mật

Cách tốt nhất để nhận các thông báo bảo mật là đăng ký vào
danh sách gửi thư [golang-announce](https://groups.google.com/forum/#!forum/golang-announce).
Mọi thư liên quan đến một vấn đề bảo mật sẽ có tiền tố
`[security]`.

## Nhận xét về chính sách này

Nếu bạn có bất kỳ đề xuất nào để cải thiện chính sách này, vui lòng
[gửi một issue](/issue/new) để thảo luận.
