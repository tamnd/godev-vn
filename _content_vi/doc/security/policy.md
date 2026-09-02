---
title: Chính sách bảo mật Go
layout: article
breadcrumb: true
---

## Tổng quan

Tài liệu này giải thích quy trình của nhóm Bảo mật Go trong việc xử lý các vấn đề
được báo cáo và những gì cần mong đợi đổi lại.

## Báo cáo lỗi bảo mật

Tất cả các lỗi bảo mật trong bản phân phối Go nên được báo cáo qua email đến
[security@golang.org](mailto:security@golang.org). Email này được gửi đến
nhóm Bảo mật Go.

Để đảm bảo báo cáo của bạn không bị đánh dấu là spam, **hãy bao gồm từ
"vulnerability"** ở bất kỳ đâu trong email của bạn. Hãy sử dụng dòng tiêu đề mô tả rõ ràng
cho email báo cáo của bạn.

Email của bạn sẽ được xác nhận trong vòng 7 ngày, và bạn sẽ được cập nhật thông tin
về tiến trình cho đến khi giải quyết. Vấn đề của bạn sẽ được khắc phục hoặc công bố công khai
trong vòng 90 ngày.

Nếu bạn chưa nhận được phản hồi cho email của mình trong vòng 7 ngày, hãy tiếp tục theo dõi
với nhóm Bảo mật Go lại tại
[security@golang.org](mailto:security@golang.org). Hãy đảm bảo từ
**vulnerability** có trong email của bạn.

Nếu sau 3 ngày nữa bạn vẫn chưa nhận được xác nhận báo cáo của mình,
có thể email của bạn đã bị đánh dấu là spam. Trong trường hợp đó, hãy [gửi vấn đề tại đây](https://g.co/vulnz). Chọn _"I want to
report a technical security or an abuse risk related bug in a Google product
(SQLi, XSS, etc.)"_, và liệt kê _"Go"_ là sản phẩm bị ảnh hưởng.

## Các luồng xử lý

Tùy thuộc vào tính chất của vấn đề của bạn, nhóm Bảo mật Go sẽ phân loại
nó là vấn đề trong luồng PUBLIC, PRIVATE hoặc URGENT. Tất cả các vấn đề bảo mật
sẽ được cấp số CVE.

Nhóm Bảo mật Go không gán nhãn mức độ nghiêm trọng chi tiết truyền thống
(ví dụ CRITICAL, HIGH, MEDIUM, LOW) cho các vấn đề bảo mật vì mức độ nghiêm trọng phụ thuộc
rất nhiều vào cách người dùng đang sử dụng API hoặc chức năng bị ảnh hưởng. Ngoài ra,
khi phát hành CVE cho các vấn đề bảo mật Go, chúng tôi không gán điểm CVSS, vì chúng tôi
về cơ bản không đồng ý với khả năng áp dụng của hệ thống tính điểm cho Go vì
các lý do tương tự. Các bên thứ ba, chẳng hạn như MITRE hoặc NIST, thông qua NVD, có thể gán
điểm CVSS cho các lỗ hổng bảo mật của chúng tôi, nhưng chúng tôi không xác nhận các điểm này là
phản ánh chính xác tác động của chúng.

Ví dụ, tác động của một vấn đề cạn kiệt tài nguyên trong trình phân tích
`encoding/json` phụ thuộc vào những gì đang được phân tích. Nếu người dùng đang phân tích các tệp JSON
đáng tin cậy từ hệ thống tệp cục bộ của họ, tác động có thể ở mức thấp. Nếu người dùng
đang phân tích JSON tùy ý không đáng tin cậy từ phần thân yêu cầu HTTP, tác động có thể
cao hơn nhiều.

Nói vậy, các luồng vấn đề sau đây cho thấy nhóm Bảo mật tin rằng một vấn đề nghiêm trọng
và/hoặc lan rộng đến mức nào. Ví dụ, một vấn đề có tác động trung bình đến đáng kể
đối với nhiều người dùng là vấn đề thuộc luồng PRIVATE trong chính sách này, và
một vấn đề có tác động không đáng kể đến nhỏ, hoặc chỉ ảnh hưởng đến một tập hợp nhỏ
người dùng, là vấn đề thuộc luồng PUBLIC.

### PUBLIC

Các vấn đề trong luồng PUBLIC ảnh hưởng đến các cấu hình hiếm gặp, có tác động rất hạn chế,
hoặc đã được biết đến rộng rãi.

Các vấn đề trong luồng PUBLIC được gán nhãn
[`Proposal-Security`](https://github.com/golang/go/labels/Proposal-Security),
được thảo luận thông qua
[quy trình xem xét đề xuất Go](https://go.googlesource.com/proposal/+/master/README.md#proposal-review)
**được sửa công khai**, và được backport vào [bản phát hành nhỏ](/wiki/MinorReleases) theo lịch tiếp theo
(xảy ra khoảng hàng tháng). Thông báo phát hành
bao gồm chi tiết về các vấn đề này, nhưng không có thông báo trước.

Các ví dụ về vấn đề PUBLIC trong quá khứ bao gồm:

- [#44916](/issue/44916): archive/zip: can panic when calling Reader.Open
- [#44913](/issue/44913): encoding/xml: infinite loop when using xml.NewTokenDecoder with a custom TokenReader
- [#43786](/issue/43786): crypto/elliptic: incorrect operations on the P-224 curve
- [#40928](/issue/40928): net/http/cgi,net/http/fcgi: Cross-Site Scripting (XSS) when Content-Type is not specified
- [#40618](/issue/40618): encoding/binary: ReadUvarint and ReadVarint can read an unlimited number of bytes from invalid inputs
- [#36834](/issue/36834): crypto/x509: certificate validation bypass on Windows 10

### PRIVATE

Các vấn đề trong luồng PRIVATE là vi phạm các thuộc tính bảo mật đã cam kết.

Các vấn đề trong luồng PRIVATE được **sửa trong [bản phát hành nhỏ](/wiki/MinorReleases) theo lịch tiếp theo**,
và được giữ bí mật cho đến khi đó.

Ba đến bảy ngày trước bản phát hành, một thông báo trước được gửi đến
golang-announce, thông báo sự hiện diện của một hoặc nhiều bản vá bảo mật trong
các bản phát hành sắp tới, và liệu các vấn đề có ảnh hưởng đến thư viện chuẩn, toolchain,
hay cả hai, cũng như các ID CVE đã được đặt trước cho mỗi bản vá.

Đối với các vấn đề hiện diện trong [bản phát hành chính ứng viên](/s/release),
chúng tôi tuân theo quy trình tương tự, bao gồm các bản vá trong bản phát hành ứng viên tiếp theo theo lịch.

Một số ví dụ về vấn đề PRIVATE trong quá khứ bao gồm:

- [#53416](/issue/53416): path/filepath: stack exhaustion in Glob
- [#53616](/issue/53616): go/parser: stack exhaustion in all Parse* functions
- [#54658](/issue/54658): net/http: handle server errors after sending GOAWAY
- [#56284](/issue/56284): syscall, os/exec: unsanitized NUL in environment variables

### URGENT

Các vấn đề trong luồng URGENT là mối đe dọa đối với tính toàn vẹn của hệ sinh thái Go, hoặc đang
bị khai thác tích cực trong thực tế gây ra thiệt hại nghiêm trọng. Không có ví dụ gần đây,
nhưng chúng sẽ bao gồm thực thi mã từ xa trong net/http, hoặc
khôi phục khóa thực tế trong crypto/tls.

Các vấn đề trong luồng URGENT được sửa bí mật, và **kích hoạt một bản phát hành bảo mật
riêng ngay lập tức**, có thể không có thông báo trước.

## Gắn cờ các vấn đề hiện có liên quan đến bảo mật

Nếu bạn tin rằng một [vấn đề hiện có](/issue) liên quan đến bảo mật, chúng tôi yêu cầu
bạn gửi email đến [security@golang.org](mailto:security@golang.org).
Email nên bao gồm ID vấn đề và mô tả ngắn gọn về lý do tại sao nó nên
được xử lý theo chính sách bảo mật này.

## Quy trình công bố

Dự án Go sử dụng quy trình công bố sau:

1. Sau khi báo cáo bảo mật được nhận, nó được giao cho một người xử lý chính. Người này
phối hợp quy trình sửa và phát hành.

2. Vấn đề được xác nhận và danh sách phần mềm bị ảnh hưởng được xác định.

3. Mã nguồn được kiểm tra để tìm bất kỳ vấn đề tương tự tiềm ẩn nào.

4. Nếu được xác định, sau khi tham khảo với người gửi, rằng số CVE là
cần thiết, người xử lý chính sẽ lấy một số.

5. Các bản vá được chuẩn bị cho hai bản phát hành chính mới nhất và
bản sửa đổi head/master. Các bản vá được chuẩn bị cho hai bản phát hành chính mới nhất
và được hợp nhất vào head/master.

6. Vào ngày các bản vá được áp dụng, các thông báo được gửi đến
[golang-announce](https://groups.google.com/group/golang-announce),
[golang-dev](https://groups.google.com/group/golang-dev), và
[golang-nuts](https://groups.google.com/group/golang-nuts).

Quy trình này có thể mất một thời gian, đặc biệt khi cần phối hợp với
các nhà duy trì của các dự án khác. Mọi nỗ lực sẽ được thực hiện để xử lý lỗi
kịp thời nhất có thể, tuy nhiên điều quan trọng là chúng tôi phải tuân theo
quy trình được mô tả ở trên để đảm bảo rằng việc công bố được xử lý nhất quán.

Đối với các vấn đề bảo mật bao gồm việc gán số CVE, vấn đề được
liệt kê công khai trong
[sản phẩm "Golang" trên trang web CVEDetails](https://www.cvedetails.com/vulnerability-list/vendor_id-14185/Golang.html)
cũng như
[Trang công bố lỗ hổng bảo mật quốc gia](https://web.nvd.nist.gov/view/vuln/search).

## Nhận cập nhật bảo mật

Cách tốt nhất để nhận thông báo bảo mật là đăng ký danh sách gửi thư
[golang-announce](https://groups.google.com/forum/#!forum/golang-announce).
Bất kỳ tin nhắn nào liên quan đến vấn đề bảo mật sẽ được đặt tiền tố bằng
`[security]`.

## Nhận xét về chính sách này

Nếu bạn có bất kỳ đề xuất nào để cải thiện chính sách này, hãy
[gửi vấn đề](/issue/new) để thảo luận.
