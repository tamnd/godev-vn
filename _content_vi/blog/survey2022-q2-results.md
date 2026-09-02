---
title: Kết quả Khảo sát Developer Go 2022 Quý 2
date: 2022-09-08
by:
- Todd Kulesza
tags:
- survey
- community
summary: Phân tích kết quả từ Khảo sát Developer Go 2022 Quý 2.
---

<style type="text/css" scoped>
  .chart {
    margin-left: 1.5rem;
    margin-right: 1.5rem;
    width: 800px;
  }
  .quote {
    margin-left: 2rem;
    margin-right: 2rem;
    color: #999;
    font-style: italic;
    font-size: 120%;
  }
  @media (prefers-color-scheme: dark) {
    .chart {
      border-radius: 8px;
    }
  }
</style>

## Tổng quan

Bài viết này chia sẻ kết quả của ấn bản tháng 6 năm 2022 của Khảo sát Developer Go.
Thay mặt nhóm Go, xin cảm ơn 5.752 người đã chia sẻ
trải nghiệm làm việc với các tính năng mới được giới thiệu trong Go 1.18,
bao gồm generics, hệ thống công cụ bảo mật và workspace. Bạn đã giúp chúng tôi hiểu rõ hơn
về cách developer khám phá và sử dụng chức năng này, và như
bài viết này sẽ thảo luận, cung cấp những thông tin hữu ích cho các
cải tiến bổ sung. Cảm ơn bạn! 💙

### Những phát hiện chính

- __Generics đã được áp dụng nhanh chóng__. Đa số lớn người trả lời đã
  biết rằng generics đã được đưa vào bản phát hành Go 1.18, và khoảng 1 trong 4
  người trả lời cho biết họ đã bắt đầu sử dụng generics trong mã Go của họ.
  Phản hồi đơn lẻ phổ biến nhất liên quan đến generics là "cảm ơn!",
  nhưng rõ ràng là các developer đã gặp phải một số hạn chế của
  việc triển khai generics ban đầu.
- __Fuzzing còn mới với hầu hết developer Go__. Nhận thức về công cụ fuzz
  testing tích hợp sẵn của Go thấp hơn nhiều so với generics, và người trả lời có nhiều
  sự không chắc chắn hơn về lý do tại sao hoặc khi nào họ có thể cân nhắc sử dụng fuzz testing.
- __Dependency bên thứ ba là mối lo ngại bảo mật hàng đầu__. Tránh
  các dependency có lỗ hổng bảo mật đã biết là thách thức bảo mật hàng đầu
  đối với người trả lời. Nói rộng hơn, công việc bảo mật thường có thể
  không có kế hoạch và không được ghi nhận, ngụ ý rằng hệ thống công cụ cần tôn trọng
  thời gian và sự chú ý của developer.
- __Chúng tôi có thể làm tốt hơn khi thông báo chức năng mới__. Những người tham gia
  được lấy mẫu ngẫu nhiên ít có khả năng biết về các bản phát hành hệ thống công cụ Go gần đây hơn
  những người tìm thấy khảo sát qua blog Go. Điều này gợi ý rằng chúng tôi nên
  tìm kiếm ngoài các bài viết blog để truyền đạt các thay đổi trong hệ sinh thái Go, hoặc
  mở rộng nỗ lực chia sẻ các bài viết này rộng rãi hơn.
- __Xử lý lỗi vẫn là thách thức__. Sau khi phát hành generics,
  thách thức hàng đầu của người trả lời khi làm việc với Go đã chuyển sang xử lý lỗi.
  Nhìn chung, sự hài lòng với Go vẫn rất cao, và chúng tôi không tìm thấy
  sự thay đổi đáng kể nào trong cách người trả lời cho biết họ đang sử dụng Go.


### Cách đọc các kết quả này

Trong suốt bài viết này, chúng tôi sử dụng các biểu đồ về phản hồi khảo sát để cung cấp bằng chứng hỗ trợ
cho các phát hiện của chúng tôi. Tất cả các biểu đồ này sử dụng định dạng tương tự. Tiêu đề
là câu hỏi chính xác mà người trả lời khảo sát đã thấy. Trừ khi có ghi chú khác,
các câu hỏi là nhiều lựa chọn và người tham gia chỉ có thể chọn một
câu trả lời duy nhất; phụ đề của mỗi biểu đồ sẽ cho bạn biết nếu câu hỏi cho phép
nhiều câu trả lời hoặc là hộp văn bản mở thay vì câu hỏi nhiều
lựa chọn. Đối với các biểu đồ về câu trả lời văn bản mở, một thành viên nhóm Go
đã đọc và phân loại thủ công tất cả các câu trả lời. Nhiều câu hỏi mở
gợi ra nhiều loại câu trả lời đa dạng; để giữ kích thước biểu đồ hợp lý, chúng tôi
đã rút gọn chúng xuống tối đa 10 chủ đề hàng đầu, với các chủ đề bổ sung được
nhóm lại dưới "Khác".

Để giúp độc giả hiểu trọng lượng bằng chứng đằng sau mỗi phát hiện, chúng tôi
thêm thanh sai số hiển thị khoảng tin cậy 95% cho các câu trả lời; các thanh hẹp hơn
chỉ ra sự tự tin tăng cao. Đôi khi hai hoặc nhiều câu trả lời có
các thanh sai số chồng lấp, có nghĩa là thứ tự tương đối của những câu trả lời đó
không có ý nghĩa thống kê (tức là các câu trả lời thực chất bằng nhau). Góc
dưới bên phải của mỗi biểu đồ hiển thị số người có câu trả lời được
đưa vào biểu đồ, dưới dạng "_n = [số người trả lời]_".

### Ghi chú về phương pháp

Hầu hết người trả lời khảo sát "tự chọn" để tham gia khảo sát, nghĩa là họ tìm thấy
nó trên [blog Go](/blog), [@golang trên
Twitter](https://twitter.com/golang), hoặc các kênh mạng xã hội Go khác. Một vấn đề tiềm năng
với cách tiếp cận này là những người không theo dõi các kênh này ít có khả năng
biết về khảo sát hơn, và có thể trả lời khác với
những người _theo dõi chặt chẽ_ chúng. Khoảng một phần ba người trả lời được
lấy mẫu ngẫu nhiên, nghĩa là họ đã trả lời khảo sát sau khi thấy lời nhắc
trong VS Code (mọi người dùng plugin VS Code Go từ ngày 1 đến ngày
21 tháng 6 năm 2022 có 10% cơ hội nhận được lời nhắc ngẫu nhiên này). Nhóm được lấy mẫu ngẫu nhiên này
giúp chúng tôi tổng quát hóa các phát hiện này cho cộng đồng developer Go rộng lớn hơn. Hầu hết các câu hỏi khảo sát không cho thấy sự khác biệt có ý nghĩa giữa
các nhóm này, nhưng trong một số trường hợp có sự khác biệt quan trọng, độc giả sẽ
thấy các biểu đồ phân tách câu trả lời thành nhóm "Mẫu ngẫu nhiên" và "Tự chọn".

## Generics

<div class="quote">"[Generics] có vẻ như là tính năng rõ ràng duy nhất còn thiếu kể từ lần đầu tiên tôi sử dụng ngôn ngữ. Đã giúp giảm thiểu trùng lặp mã rất nhiều." &mdash; Một người trả lời khảo sát bàn về generics</div>

Sau khi Go 1.18 được phát hành với hỗ trợ cho các tham số kiểu (thường được gọi là _generics_), chúng tôi muốn hiểu nhận thức và việc áp dụng ban đầu của generics trông như thế nào, cũng như xác định các thách thức hoặc rào cản phổ biến khi sử dụng generics.

Đại đa số người trả lời khảo sát (86%) đã biết generics
được đưa vào bản phát hành Go 1.18. Chúng tôi đã hy vọng thấy đa số đơn giản
ở đây, vì vậy đây là nhiều nhận thức hơn chúng tôi kỳ vọng. Chúng tôi cũng thấy
rằng khoảng một phần tư người trả lời đã bắt đầu sử dụng generics trong mã Go (26%),
bao gồm 14% cho biết họ đã sử dụng generics trong production hoặc
mã đã phát hành. Đa số người trả lời (54%) không phản đối sử dụng
generics, nhưng không có nhu cầu sử dụng chúng ngay hôm nay. Chúng tôi cũng thấy rằng 8% người trả lời _muốn_ sử dụng generics trong Go, nhưng hiện đang bị chặn bởi điều gì đó.

<img src="survey2022q2/generics_awareness.svg" alt="Biểu đồ cho thấy hầu hết
người trả lời biết Go 1.18 bao gồm generics" class="chart" /> <img
src="survey2022q2/generics_use.svg" alt="Biểu đồ cho thấy 26% người trả lời đang
sử dụng Go generics" class="chart" />

Điều gì đang ngăn một số developer sử dụng generics? Đa số
người trả lời thuộc một trong hai loại. Thứ nhất, 30% người trả lời cho biết họ gặp phải giới hạn của việc triển khai generics hiện tại, chẳng hạn như muốn có các method tham số hóa, cải thiện type inference hoặc switch trên các kiểu.
Người trả lời cho biết những vấn đề này hạn chế các trường hợp sử dụng tiềm năng cho generics hoặc
cảm thấy chúng làm cho mã generic không cần thiết trở nên dài dòng. Loại thứ hai
liên quan đến việc phụ thuộc vào điều gì đó chưa (chưa) hỗ trợ generics, các linter
là công cụ phổ biến nhất ngăn việc áp dụng, nhưng danh sách này cũng bao gồm
những thứ như tổ chức vẫn ở bản phát hành Go cũ hơn hoặc phụ thuộc vào
bản phân phối Linux chưa cung cấp các package Go 1.18 (26%). Đường cong học tập dốc hoặc thiếu tài liệu hữu ích được trích dẫn bởi 12%
người trả lời. Ngoài các vấn đề hàng đầu này, người trả lời cho chúng tôi biết về nhiều
thách thức ít phổ biến hơn (mặc dù vẫn có ý nghĩa), như được hiển thị trong biểu đồ
dưới đây. Để tránh tập trung vào các giả thuyết, phân tích này chỉ bao gồm những người
đã nói họ đang sử dụng generics, hoặc đã cố gắng sử dụng generics nhưng
bị chặn bởi điều gì đó.

<img src="survey2022q2/text_gen_challenge.svg" alt="Biểu đồ hiển thị các thách thức generics hàng đầu" class="chart" />

Chúng tôi cũng đã yêu cầu người trả lời khảo sát đã thử sử dụng generics chia sẻ thêm
phản hồi. Thật khích lệ, cứ một trong mười người trả lời cho biết generics đã
đơn giản hóa mã của họ, hoặc dẫn đến ít trùng lặp mã hơn. Câu trả lời
phổ biến nhất là một số biến thể của "cảm ơn!" hoặc cảm xúc tích cực
chung (43%); để so sánh, chỉ 6% người trả lời thể hiện phản ứng hoặc cảm xúc tiêu cực. Phản ánh các phát hiện từ câu hỏi "thách thức lớn nhất"
ở trên, gần một phần ba người trả lời thảo luận về việc gặp phải hạn chế
của việc triển khai generics trong Go. Nhóm Go đang sử dụng tập kết quả này
để giúp quyết định liệu hoặc cách nào một số hạn chế này có thể được nới lỏng.

<img src="survey2022q2/text_gen_feedback.svg" alt="Biểu đồ hiển thị hầu hết phản hồi về generics
là tích cực hoặc đề cập đến một hạn chế của việc triển khai hiện tại" class="chart" />

## Bảo mật

<div class="quote">"[Thách thức lớn nhất là] tìm thời gian khi có các ưu tiên cạnh tranh; khách hàng doanh nghiệp muốn các tính năng của họ hơn là bảo mật." &mdash; Một người trả lời khảo sát bàn về các thách thức bảo mật</div>

Sau [vụ vi phạm SolarWinds năm 2020](https://en.wikipedia.org/wiki/2020_United_States_federal_government_data_breach#SolarWinds_exploit),
thực hành phát triển phần mềm an toàn đã nhận được sự chú ý đổi mới.
Nhóm Go đã ưu tiên công việc trong lĩnh vực này, bao gồm các công cụ để tạo [hóa đơn vật liệu phần mềm (SBOM)](https://pkg.go.dev/debug/buildinfo), [fuzz
testing](/doc/fuzz/), và gần đây nhất là [quét lỗ hổng bảo mật](/blog/vuln/). Để hỗ trợ những nỗ lực này, khảo sát này
đã đặt ra một số câu hỏi về các thực hành và thách thức bảo mật phát triển phần mềm. Chúng tôi đặc biệt muốn hiểu:

- Loại công cụ bảo mật nào mà developer Go đang sử dụng hiện nay?
- Developer Go tìm và giải quyết các lỗ hổng bảo mật như thế nào?
- Những thách thức lớn nhất khi viết phần mềm Go an toàn là gì?

Kết quả của chúng tôi cho thấy trong khi các công cụ phân tích tĩnh đang được sử dụng rộng rãi
(65% người trả lời), một thiểu số người trả lời hiện đang sử dụng nó để tìm
lỗ hổng bảo mật (35%) hoặc cải thiện bảo mật mã khác (33%). Người trả lời
cho biết rằng hệ thống công cụ bảo mật thường chạy nhất trong thời gian CI/CD (84%), với
một thiểu số cho biết developer chạy các công cụ này cục bộ trong quá trình phát triển (22%).
Điều này phù hợp với nghiên cứu bảo mật bổ sung mà nhóm chúng tôi đã tiến hành, trong đó
phát hiện rằng quét bảo mật trong thời gian CI/CD là rào chắn mong muốn, nhưng
developer thường coi đây là quá muộn cho thông báo đầu tiên: họ muốn
biết một dependency có thể bị lỗ hổng _trước_ khi xây dựng dựa trên nó, hoặc
xác minh rằng cập nhật phiên bản đã giải quyết một lỗ hổng mà không cần chờ CI
chạy đầy đủ các bài kiểm tra bổ sung cho PR của họ.

<img src="survey2022q2/dev_techniques.svg" alt="Biểu đồ hiển thị sự phổ biến của 9
kỹ thuật phát triển khác nhau" class="chart" /> <img
src="survey2022q2/security_sa_when.svg" alt="Biểu đồ cho thấy hầu hết người trả lời
chạy công cụ bảo mật trong CI" class="chart" />

Chúng tôi cũng đã hỏi người trả lời về những thách thức lớn nhất của họ xung quanh việc phát triển
phần mềm an toàn. Khó khăn phổ biến nhất là đánh giá bảo mật của
các thư viện bên thứ ba (57% người trả lời), một chủ đề mà các scanner lỗ hổng bảo mật
(chẳng hạn như [dependabot của GitHub](https://github.com/dependabot) hoặc
[govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck) của nhóm Go) có thể giúp
giải quyết. Các thách thức hàng đầu khác gợi ý cơ hội cho hệ thống công cụ bảo mật bổ sung: người trả lời cho biết rất khó để áp dụng nhất quán các thực hành tốt nhất khi viết mã, và xác thực rằng mã kết quả không có lỗ hổng bảo mật.

<img src="survey2022q2/security_challenges.svg" alt="Biểu đồ hiển thị thách thức bảo mật phổ biến nhất
là đánh giá bảo mật của các thư viện bên thứ ba"
class="chart" />

Fuzz testing, một cách tiếp cận khác để tăng bảo mật ứng dụng, vẫn còn
khá mới với hầu hết người trả lời. Chỉ 12% cho biết họ sử dụng nó tại nơi làm việc, và 5% cho biết
họ đã áp dụng công cụ fuzzing tích hợp sẵn của Go. Câu hỏi tiếp theo mở hỏi
điều gì làm cho fuzzing khó sử dụng cho thấy rằng lý do chính không phải là
vấn đề kỹ thuật: ba câu trả lời hàng đầu thảo luận về việc không hiểu cách
sử dụng fuzz testing (23%), thiếu thời gian để dành cho fuzzing hoặc bảo mật nói chung
(22%), và hiểu tại sao và khi nào developer có thể muốn sử dụng
fuzz testing (14%). Những phát hiện này chỉ ra rằng chúng tôi vẫn còn công việc cần làm về
mặt truyền đạt giá trị của fuzz testing, những gì nên được fuzz test,
và cách áp dụng nó cho các codebase khác nhau.

<img src="survey2022q2/fuzz_use.svg" alt="Biểu đồ cho thấy hầu hết người trả lời chưa
thử fuzz testing" class="chart" /> <img
src="survey2022q2/text_fuzz_challenge.svg" alt="Biểu đồ cho thấy những thách thức fuzz testing
lớn nhất liên quan đến sự hiểu biết, không phải vấn đề kỹ thuật"
class="chart" />

Để hiểu rõ hơn các tác vụ phổ biến xung quanh việc phát hiện và
giải quyết lỗ hổng bảo mật, chúng tôi đã hỏi người trả lời liệu họ có phát hiện ra bất kỳ lỗ hổng bảo mật nào
trong mã Go của họ hoặc các dependency của nó trong năm vừa qua không. Đối với những người có, chúng tôi đã hỏi tiếp
về cách lỗ hổng bảo mật gần đây nhất được phát hiện, cách họ điều tra và/hoặc giải quyết nó, và điều gì
gặp khó khăn nhất trong toàn bộ quá trình.

Trước tiên, chúng tôi tìm thấy bằng chứng rằng quét lỗ hổng bảo mật là hiệu quả. Một phần tư
người trả lời cho biết họ đã biết về lỗ hổng bảo mật trong một trong
các dependency bên thứ ba của họ. Hãy nhớ rằng, chỉ có khoảng 1/3 người trả lời
đang sử dụng quét lỗ hổng bảo mật nào đó, khi chúng tôi xem xét câu trả lời từ
những người cho biết họ đã chạy một số loại scanner lỗ hổng bảo mật, tỷ lệ này
gần như tăng gấp đôi, từ 25% lên 46%. Ngoài các lỗ hổng bảo mật trong các dependency hoặc trong
Go, 12% người trả lời cho biết họ biết về các lỗ hổng bảo mật trong
chính mã của họ.

Đa số người trả lời cho biết họ biết về lỗ hổng bảo mật qua các
scanner bảo mật (65%). Công cụ đơn lẻ phổ biến nhất được người trả lời trích dẫn là [dependabot
của GitHub](https://github.com/dependabot) (38%), làm cho nó được đề cập thường xuyên
hơn tất cả các scanner lỗ hổng bảo mật khác cộng lại (27%). Sau
các công cụ quét, phương pháp phổ biến nhất để biết về lỗ hổng bảo mật là
các báo cáo công khai, chẳng hạn như ghi chú bản phát hành và CVE (22%).

<img src="survey2022q2/security_found_vuln.svg" alt="Biểu đồ cho thấy rằng hầu hết
người trả lời không tìm thấy lỗ hổng bảo mật trong năm vừa qua"
class="chart" /> <img src="survey2022q2/text_vuln_find.svg" alt="Biểu đồ cho thấy
rằng các scanner lỗ hổng bảo mật là cách phổ biến nhất để biết về lỗ hổng bảo mật"
class="chart" />

Sau khi người trả lời biết về lỗ hổng bảo mật, giải pháp phổ biến nhất là
nâng cấp dependency bị lỗ hổng (67%). Trong số người trả lời cũng
thảo luận về việc sử dụng scanner lỗ hổng bảo mật (đại diện cho những người tham gia đang
thảo luận về lỗ hổng bảo mật trong một dependency bên thứ ba), con số này tăng lên
85%. Chưa đến một phần ba người trả lời thảo luận về việc đọc CVE hoặc
báo cáo lỗ hổng bảo mật (31%), và chỉ 12% đề cập đến việc điều tra sâu hơn để
hiểu liệu (và cách) phần mềm của họ bị ảnh hưởng bởi lỗ hổng bảo mật.

Việc chỉ 12% người trả lời cho biết họ đã thực hiện điều tra về việc liệu
một lỗ hổng bảo mật có thể tiếp cận được trong mã của họ hay không, hoặc tác động tiềm năng của nó đối với
dịch vụ của họ, thật đáng ngạc nhiên. Để hiểu điều này rõ hơn, chúng tôi cũng
nhìn vào những gì người trả lời cho biết là khó khăn nhất khi phản hồi
các lỗ hổng bảo mật. Họ mô tả một số chủ đề khác nhau với tỷ lệ gần bằng nhau,
từ đảm bảo rằng các cập nhật dependency không phá vỡ bất cứ điều gì,
đến hiểu cách cập nhật các dependency gián tiếp qua các file go.mod.
Cũng trong danh sách này là loại điều tra cần thiết để hiểu
tác động hoặc nguyên nhân gốc rễ của lỗ hổng bảo mật. Khi chúng tôi chỉ tập trung vào người trả lời
đã nói rằng họ đã thực hiện những điều tra này, tuy nhiên, chúng tôi thấy một mối tương quan rõ ràng:
70% người trả lời đã nói rằng họ đã thực hiện điều tra về
tác động tiềm năng của lỗ hổng bảo mật đã trích dẫn nó là phần khó khăn nhất
của quá trình này. Lý do bao gồm không chỉ là độ khó của nhiệm vụ, mà còn
thực tế là nó thường là công việc không có kế hoạch và không được ghi nhận.

Nhóm Go tin rằng những điều tra sâu hơn này, đòi hỏi sự hiểu biết về _cách_
một ứng dụng sử dụng một dependency bị lỗ hổng, là rất quan trọng để hiểu rủi ro
mà lỗ hổng bảo mật có thể đặt ra cho một tổ chức, cũng như hiểu liệu việc vi phạm dữ liệu hoặc
xâm phạm bảo mật khác có xảy ra hay không. Vì vậy, [chúng tôi đã thiết kế
`govulncheck`](/blog/vuln) để chỉ cảnh báo developer khi
một lỗ hổng bảo mật được gọi đến, và chỉ cho developer đến các vị trí chính xác trong
mã của họ sử dụng các hàm bị lỗ hổng. Hy vọng của chúng tôi là điều này sẽ giúp
developer dễ dàng hơn để nhanh chóng điều tra các lỗ hổng bảo mật thực sự quan trọng với
ứng dụng của họ, do đó giảm tổng lượng công việc không có kế hoạch trong
không gian này.

<img src="survey2022q2/text_vuln_resolve.svg" alt="Biểu đồ cho thấy hầu hết
người trả lời giải quyết lỗ hổng bảo mật bằng cách nâng cấp dependency" class="chart" />
<img src="survey2022q2/text_vuln_challenge.svg" alt="Biểu đồ cho thấy 6 nhiệm vụ
hòa nhau là khó khăn nhất khi điều tra và giải quyết
lỗ hổng bảo mật" class="chart" />

## Hệ thống công cụ

Tiếp theo, chúng tôi điều tra ba câu hỏi tập trung vào hệ thống công cụ:

- Bức tranh trình soạn thảo có thay đổi kể từ khảo sát lần trước không?
- Các developer có sử dụng workspace không? Nếu có, họ đã gặp phải những thách thức nào khi bắt đầu?
- Developer xử lý tài liệu package nội bộ như thế nào?

VS Code có vẻ đang tiếp tục tăng trưởng về mức độ phổ biến trong số người trả lời khảo sát,
với tỷ lệ người trả lời cho biết đây là trình soạn thảo ưa thích của họ cho mã Go
tăng từ 42% lên 45% kể từ năm 2021. VS Code và GoLand,
hai trình soạn thảo phổ biến nhất, không cho thấy sự khác biệt về mức độ phổ biến giữa
các tổ chức nhỏ và lớn, mặc dù developer sở thích cá nhân có nhiều khả năng
thích VS Code hơn GoLand. Phân tích này loại trừ những người trả lời VS Code được lấy mẫu ngẫu nhiên, chúng tôi kỳ vọng những người chúng tôi mời tham gia khảo sát sẽ thể hiện sở thích
với công cụ được sử dụng để phân phối lời mời, đó chính xác là những gì chúng tôi thấy
(91% người trả lời được lấy mẫu ngẫu nhiên thích VS Code).

Sau khi chuyển đổi năm 2021 sang [cung cấp hỗ trợ Go của VS Code qua gopls
language server](/blog/gopls-vscode-go), nhóm Go đã
quan tâm đến việc hiểu các điểm đau của developer liên quan đến gopls. Trong khi chúng tôi
nhận được một lượng phản hồi lành mạnh từ các developer hiện đang sử dụng gopls, chúng tôi
tự hỏi liệu có một tỷ lệ lớn developer đã tắt nó ngay sau khi
phát hành, điều đó có thể có nghĩa là chúng tôi không nghe phản hồi về các trường hợp sử dụng
đặc biệt có vấn đề. Để trả lời câu hỏi này, chúng tôi đã hỏi người trả lời cho biết
họ thích một trình soạn thảo hỗ trợ gopls liệu họ có _sử dụng_ gopls hay không,
thấy rằng chỉ có 2% cho biết họ đã tắt nó; đối với VS Code
cụ thể, con số này giảm xuống 1%. Điều này tăng thêm sự tự tin của chúng tôi rằng chúng tôi đang
nghe phản hồi từ một nhóm đại diện của developer. Đối với những độc giả
vẫn còn vấn đề chưa giải quyết với gopls, vui lòng cho chúng tôi biết bằng cách <a
href="https://github.com/golang/go/issues">gửi vấn đề trên GitHub</a>.

<img src="survey2022q2/editor_self_select.svg" alt="Biểu đồ hiển thị các
trình soạn thảo ưa thích hàng đầu cho Go là VS Code, GoLand và Vim / Neovim" class="chart" />
<img src="survey2022q2/use_gopls.svg" alt="Biểu đồ cho thấy chỉ 2%
người trả lời đã tắt gopls" class="chart"/>

Liên quan đến workspace, có vẻ nhiều người lần đầu tiên biết về hỗ trợ của Go
cho workspace đa module thông qua khảo sát này. Người trả lời biết đến khảo sát thông qua lời nhắc ngẫu nhiên của VS Code đặc biệt có nhiều khả năng nói rằng họ
chưa nghe nói về workspace trước đây (53% người trả lời được lấy mẫu ngẫu nhiên so với
33% người tự chọn), một xu hướng chúng tôi cũng quan sát thấy với nhận thức về
generics (mặc dù con số này cao hơn cho cả hai nhóm, với 93% người tự chọn
biết rằng generics đã ra mắt trong Go 1.18 so với 68% người được lấy mẫu ngẫu nhiên). Một diễn giải là có một đối tượng lớn các developer Go mà chúng tôi hiện không tiếp cận được qua blog Go hoặc các kênh mạng xã hội hiện có, vốn đã là cơ chế chính của chúng tôi để chia sẻ chức năng mới.

Chúng tôi thấy rằng 9% người trả lời cho biết họ đã thử workspace, và
thêm 5% muốn thử nhưng bị chặn bởi điều gì đó. Người trả lời
thảo luận về nhiều thách thức khác nhau khi cố gắng sử dụng workspace Go. Thiếu
tài liệu và thông báo lỗi hữu ích từ lệnh `go work` đứng đầu
danh sách (21%), tiếp theo là các thách thức kỹ thuật như tái cấu trúc các
kho lưu trữ hiện có (13%). Tương tự như các thách thức được thảo luận trong phần bảo mật,
chúng tôi một lần nữa thấy "thiếu thời gian / không phải ưu tiên" trong danh sách này, chúng tôi diễn giải điều này
có nghĩa là ngưỡng để hiểu và thiết lập workspace vẫn còn hơi quá cao
so với lợi ích chúng cung cấp, có thể vì các developer đã có
các giải pháp thay thế sẵn có.

<img src="survey2022q2/workspaces_use_s.svg" alt="Biểu đồ cho thấy đa số
người trả lời được lấy mẫu ngẫu nhiên chưa biết về workspace trước khảo sát này" class="chart" /> <img src="survey2022q2/text_workspace_challenge.svg"
alt="Biểu đồ cho thấy rằng tài liệu và thông báo lỗi là thách thức hàng đầu
khi cố gắng sử dụng workspace Go" class="chart" />

Trước khi phát hành Go module, các tổ chức có thể chạy các máy chủ tài liệu nội bộ
(chẳng hạn như [máy chủ cung cấp năng lực cho godoc.org](https://github.com/golang/gddo)) để cung cấp cho nhân viên
tài liệu cho các package Go riêng tư, nội bộ. Điều này vẫn đúng với
[pkg.go.dev](https://pkg.go.dev), nhưng việc thiết lập máy chủ như vậy phức tạp hơn
so với trước đây. Để hiểu liệu chúng tôi có nên đầu tư để làm cho quá trình này
dễ dàng hơn không, chúng tôi đã hỏi người trả lời cách họ xem tài liệu cho các module Go nội bộ ngày nay, và liệu đó có phải là cách làm việc ưa thích của họ không.

Kết quả cho thấy cách phổ biến nhất để xem tài liệu Go nội bộ ngày nay
là đọc mã (81%), và trong khi khoảng một nửa người trả lời
hài lòng với điều này, một tỷ lệ lớn muốn có máy chủ tài liệu nội bộ (39%). Chúng tôi cũng hỏi ai có thể là người có nhiều khả năng nhất để
cấu hình và duy trì máy chủ như vậy: theo tỷ lệ 2 trên 1, người trả lời cho rằng
đó sẽ là một kỹ sư phần mềm thay vì ai đó từ nhóm hỗ trợ IT
hoặc vận hành chuyên dụng. Điều này gợi ý mạnh mẽ rằng một máy chủ tài liệu nên là giải pháp turnkey, hoặc ít nhất là dễ dàng để một developer đơn lẻ có thể khởi động nhanh chóng (ví dụ, trong khoảng một giờ ăn trưa), dựa trên lý thuyết rằng loại công việc này là một trách nhiệm nữa trên tấm lịch đã đầy của developer.

<img src="survey2022q2/doc_viewing_today.svg" alt="Biểu đồ cho thấy hầu hết
người trả lời sử dụng trực tiếp mã nguồn để xem tài liệu package nội bộ"
class="chart" /> <img src="survey2022q2/doc_viewing_ideal.svg" alt="Biểu đồ
cho thấy 39% người trả lời muốn sử dụng máy chủ tài liệu thay vì xem mã nguồn để có tài liệu" class="chart" /> <img
src="survey2022q2/doc_server_owner.svg" alt="Biểu đồ cho thấy hầu hết người trả lời
mong đợi một kỹ sư phần mềm chịu trách nhiệm về máy chủ tài liệu như vậy"
class="chart" />

## Chúng tôi đã nghe từ ai

Nhìn chung, nhân khẩu học và firmographic của người trả lời không
thay đổi đáng kể kể từ [khảo sát năm 2021](/blog/survey2021-results) của chúng tôi. Đa số nhỏ của người trả lời (53%) có ít nhất hai năm kinh nghiệm sử dụng Go, trong khi
phần còn lại là mới hơn với cộng đồng Go. Khoảng 1/3 người trả lời làm việc tại các doanh nghiệp nhỏ (< 100 nhân viên), 1/4 làm việc tại các doanh nghiệp cỡ vừa (100 đến 1.000
nhân viên), và 1/4 làm việc tại các doanh nghiệp lớn (> 1.000 nhân viên). Tương tự như năm ngoái, chúng tôi thấy rằng lời nhắc VS Code của chúng tôi đã giúp khuyến khích sự tham gia khảo sát ngoài Bắc Mỹ và Châu Âu.

<img src="survey2022q2/go_exp.svg" alt="Biểu đồ hiển thị phân bố
kinh nghiệm Go của người trả lời" class="chart" /> <img src="survey2022q2/where.svg"
alt="Biểu đồ hiển thị phân bố nơi người trả lời sử dụng Go" class="chart" />
<img src="survey2022q2/org_size.svg" alt="Biểu đồ hiển thị phân bố
quy mô tổ chức cho người trả lời khảo sát" class="chart" /> <img
src="survey2022q2/industry.svg" alt="Biểu đồ hiển thị phân bố phân loại ngành công nghiệp
cho người trả lời khảo sát" class="chart" /> <img
src="survey2022q2/location_s.svg" alt="Biểu đồ hiển thị nơi người trả lời khảo sát sống trên thế giới" class="chart" />

## Người trả lời sử dụng Go như thế nào

Tương tự như phần trước, chúng tôi không tìm thấy bất kỳ thay đổi đáng kể nào theo năm
trong cách người trả lời đang sử dụng Go. Hai trường hợp sử dụng phổ biến nhất vẫn là xây dựng dịch vụ API/RPC (73%) và viết CLI (60%). Chúng tôi
đã sử dụng các mô hình tuyến tính để điều tra liệu có mối quan hệ nào giữa thời gian
một người trả lời đã sử dụng Go và các loại thứ họ đang xây dựng với nó hay không. Chúng tôi thấy rằng người trả lời có < 1 năm kinh nghiệm Go có nhiều khả năng
xây dựng thứ gì đó ở nửa dưới của biểu đồ này (GUI, IoT,
trò chơi, ML/AI hoặc ứng dụng di động), gợi ý rằng có sự quan tâm đến việc sử dụng Go
trong các lĩnh vực này, nhưng sự sụt giảm sau một năm kinh nghiệm cũng ngụ ý
rằng developer gặp phải các rào cản đáng kể khi làm việc với Go trong những lĩnh vực này.

Đa số người trả lời sử dụng Linux (59%) hoặc macOS (52%) khi
phát triển với Go, và đại đa số triển khai lên hệ thống Linux (93%). Trong
chu kỳ này chúng tôi đã thêm một lựa chọn câu trả lời cho việc phát triển trên Windows Subsystem for Linux
(WSL), thấy rằng 13% người trả lời sử dụng điều này khi làm việc với Go.

<img src="survey2022q2/go_app.svg" alt="Biểu đồ hiển thị phân bố những gì
người trả lời xây dựng với Go" class="chart" /> <img src="survey2022q2/os_dev.svg"
alt="Biểu đồ cho thấy Linux và macOS là các hệ thống phát triển phổ biến nhất"
class="chart" /> <img src="survey2022q2/os_deploy.svg" alt="Biểu đồ cho thấy
Linux là nền tảng triển khai phổ biến nhất" class="chart" />

## Cảm nhận và thách thức

Cuối cùng, chúng tôi đã hỏi người trả lời về mức độ hài lòng hoặc không hài lòng tổng thể của họ
với Go trong năm vừa qua, cũng như thách thức lớn nhất họ đối mặt khi sử dụng Go. Chúng tôi thấy rằng 93% người trả lời cho biết họ "hơi" (30%) hoặc "rất" (63%) hài lòng, điều này không khác biệt đáng kể về mặt thống kê
so với 92% người trả lời cho biết họ hài lòng trong
Khảo sát Developer Go 2021.

Sau nhiều năm generics liên tục là thách thức được thảo luận phổ biến nhất
khi sử dụng Go, hỗ trợ cho các tham số kiểu trong Go 1.18 cuối cùng
dẫn đến một thách thức hàng đầu mới: người bạn cũ của chúng ta là xử lý lỗi. Để chắc chắn,
xử lý lỗi về mặt thống kê bằng nhau với một số thách thức khác, bao gồm
các thư viện còn thiếu hoặc chưa trưởng thành cho một số lĩnh vực nhất định, giúp developer học
và thực hiện các thực hành tốt nhất, và các sửa đổi khác cho hệ thống kiểu, chẳng hạn như
hỗ trợ enum hoặc cú pháp lập trình hàm hơn. Sau generics, có vẻ
như có một đuôi rất dài các thách thức đối mặt với developer Go.

<img src="survey2022q2/csat.svg" alt="Biểu đồ cho thấy 93% người trả lời khảo sát
hài lòng khi sử dụng Go, với 4% không hài lòng" class="chart" /> <img
src="survey2022q2/text_biggest_challenge.svg" alt="Biểu đồ cho thấy đuôi dài
các thách thức được báo cáo bởi người trả lời khảo sát" class="chart" />

## Phương pháp khảo sát

Chúng tôi đã công bố công khai khảo sát này vào ngày 1 tháng 6 năm 2022 qua
[go.dev/blog](/blog) và [@golang](https://twitter.com/golang)
trên Twitter. Chúng tôi cũng đã nhắc ngẫu nhiên 10% người dùng [VS
Code](https://code.visualstudio.com/) qua plugin Go từ ngày 1 tháng 6
đến ngày 21 tháng 6. Khảo sát đóng vào ngày 22 tháng 6, và các câu trả lời không đầy đủ (tức là những người đã bắt đầu nhưng không hoàn thành khảo sát) cũng được ghi lại. Chúng tôi đã lọc bỏ
dữ liệu từ những người trả lời đã hoàn thành khảo sát đặc biệt nhanh (< 30
giây) hoặc có xu hướng chọn tất cả các câu trả lời cho các câu hỏi đa lựa chọn. Điều này để lại 5.752 phản hồi.

Khoảng 1/3 người trả lời đến từ lời nhắc VS Code ngẫu nhiên, và nhóm này
có xu hướng có ít kinh nghiệm hơn với Go so với những người tìm thấy khảo sát qua
blog Go hoặc các kênh mạng xã hội của Go. Chúng tôi đã sử dụng các mô hình tuyến tính và logistic để điều tra liệu các sự khác biệt rõ ràng giữa các nhóm này có được giải thích tốt hơn bởi sự khác biệt về kinh nghiệm này hay không, điều này thường là trường hợp. Các ngoại lệ được ghi chú trong văn bản.

Năm nay chúng tôi rất muốn chia sẻ tập dữ liệu thô với cộng đồng,
tương tự như các khảo sát developer từ [Stack
Overflow](https://insights.stackoverflow.com/survey),
[JetBrains](https://www.jetbrains.com/lp/devecosystem-2021/), và những tổ chức khác.
Hướng dẫn pháp lý gần đây tiếc là ngăn chúng tôi làm điều đó ngay bây giờ, nhưng
chúng tôi đang nghiên cứu điều này và mong đợi có thể chia sẻ tập dữ liệu thô cho Khảo sát Developer Go tiếp theo của chúng tôi.

## Kết luận

Lần lặp này của Khảo sát Developer Go tập trung vào chức năng mới từ
bản phát hành Go 1.18. Chúng tôi thấy rằng việc áp dụng generics đang tiến triển tốt, với
developer đã gặp phải một số hạn chế của việc triển khai hiện tại.
Fuzz testing và workspace đã có tốc độ áp dụng chậm hơn, mặc dù phần lớn không phải vì
lý do kỹ thuật: thách thức chính với cả hai là hiểu khi nào và
cách sử dụng chúng. Việc thiếu thời gian của developer để tập trung vào những chủ đề này là một
thách thức khác, và chủ đề này tiếp tục vào hệ thống công cụ bảo mật cũng. Những
phát hiện này đang giúp nhóm Go ưu tiên các nỗ lực tiếp theo của chúng tôi và sẽ
ảnh hưởng đến cách chúng tôi tiếp cận thiết kế hệ thống công cụ trong tương lai.

Cảm ơn bạn đã tham gia cùng chúng tôi trong chuyến tham quan nghiên cứu developer Go,
chúng tôi hy vọng nó mang lại thông tin hữu ích và thú vị. Quan trọng nhất, cảm ơn tất cả những người đã
phản hồi các khảo sát của chúng tôi qua các năm. Phản hồi của bạn giúp chúng tôi hiểu
các ràng buộc mà developer Go làm việc trong đó và xác định những thách thức họ đối mặt. Bằng cách
chia sẻ những kinh nghiệm này, bạn đang giúp cải thiện hệ sinh thái Go cho
tất cả mọi người. Thay mặt cho các Gopher khắp nơi, chúng tôi trân trọng bạn!
