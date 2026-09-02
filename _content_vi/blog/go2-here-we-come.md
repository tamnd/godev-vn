---
title: Go 2, đây chúng tôi đến!
date: 2018-11-29
by:
- Robert Griesemer
tags:
- go2
- proposals
- community
summary: Cách các đề xuất Go 2 sẽ được đánh giá, lựa chọn và phát hành.
---

## Bối cảnh

Tại GopherCon 2017, Russ Cox chính thức bắt đầu quá trình suy nghĩ về phiên bản lớn
tiếp theo của Go với bài nói chuyện [Tương lai của Go](https://www.youtube.com/watch?v=0Zbh_vmAKvk)
([bài đăng blog](/blog/toward-go2)). Chúng tôi đã gọi không chính thức ngôn ngữ
tương lai này là Go 2, mặc dù giờ chúng tôi hiểu rằng nó sẽ đến theo từng bước
tăng dần chứ không phải với một cú đánh lớn và một bản phát hành chính duy nhất.
Dù vậy, Go 2 là một cái tên hữu ích, dù chỉ để có cách nói về ngôn ngữ tương lai
đó, vì vậy hãy tiếp tục dùng nó cho đến nay.

Một sự khác biệt lớn giữa Go 1 và Go 2 là ai sẽ ảnh hưởng đến thiết kế và cách
đưa ra quyết định. Go 1 là nỗ lực của một nhóm nhỏ với ảnh hưởng bên ngoài khiêm
tốn; Go 2 sẽ được cộng đồng định hướng hơn nhiều. Sau gần 10 năm tiếp cận, chúng
tôi đã học được nhiều về ngôn ngữ và thư viện mà chúng tôi không biết ban đầu, và
điều đó chỉ có thể thực hiện được thông qua phản hồi từ cộng đồng Go.

Năm 2015 chúng tôi giới thiệu [quy trình đề xuất](/s/proposal)
để thu thập một loại phản hồi cụ thể: đề xuất cho các thay đổi ngôn ngữ và thư
viện. Một ủy ban gồm các thành viên cấp cao của nhóm Go đã xem xét, phân loại và
quyết định các đề xuất đến một cách thường xuyên. Điều đó hoạt động khá tốt, nhưng
như một phần của quy trình đó chúng tôi đã bỏ qua tất cả các đề xuất không tương
thích ngược, đơn giản là gán nhãn chúng là Go 2 thay thế. Năm 2017 chúng tôi cũng
ngừng thực hiện bất kỳ thay đổi ngôn ngữ tương thích ngược tăng dần nào, dù nhỏ
đến đâu, để ưu tiên kế hoạch toàn diện hơn tính đến bức tranh lớn hơn của Go 2.

Giờ là lúc hành động theo các đề xuất Go 2, nhưng để làm điều này trước tiên chúng
tôi cần một kế hoạch.

## Tình trạng

Tại thời điểm viết bài, có khoảng 120
[issue mở được gán nhãn đề xuất Go 2](https://github.com/golang/go/issues?page=1&q=is%3Aissue+is%3Aopen+label%3Aproposal+label%3AGo2&utf8=%E2%9C%93).
Mỗi đề xuất trong số đó đề xuất thay đổi thư viện hoặc ngôn ngữ đáng kể, thường
là thay đổi không đáp ứng [đảm bảo tương thích Go 1](/doc/go1compat) hiện có.
Ian Lance Taylor và tôi đã làm việc qua các đề xuất này và phân loại chúng
([Go2Cleanup](https://github.com/golang/go/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3Aproposal+label%3AGo2+label%3AGo2Cleanup),
[NeedsDecision](https://github.com/golang/go/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3Aproposal+label%3AGo2+label%3ANeedsDecision),
v.v.) để có ý tưởng về những gì có ở đó và giúp tiến hành với chúng dễ dàng hơn.
Chúng tôi cũng đã gộp các đề xuất liên quan và đóng những cái có vẻ rõ ràng nằm
ngoài phạm vi của Go, hoặc không thể thực hiện được.

Ý tưởng từ các đề xuất còn lại có thể sẽ ảnh hưởng đến thư viện và ngôn ngữ của
Go 2. Hai chủ đề lớn đã nổi lên sớm: hỗ trợ xử lý lỗi tốt hơn và generics.
[Các bản thảo thiết kế](/blog/go2draft) cho hai lĩnh vực này đã được xuất bản tại
GopherCon năm nay, và cần có thêm khám phá.

Nhưng còn những cái khác thì sao? Chúng tôi bị [ràng buộc](/blog/toward-go2) bởi
thực tế rằng giờ chúng tôi có hàng triệu lập trình viên Go và một lượng lớn code
Go, và chúng tôi cần mang tất cả chúng theo, kẻo chúng tôi có nguy cơ chia rẽ hệ
sinh thái. Điều đó có nghĩa là chúng tôi không thể thực hiện nhiều thay đổi, và
các thay đổi chúng tôi sẽ thực hiện cần được lựa chọn cẩn thận. Để tiến lên, chúng
tôi đang triển khai quy trình đánh giá đề xuất mới cho những thay đổi tiềm năng
đáng kể này.

## Quy trình đánh giá đề xuất

Mục đích của quy trình đánh giá đề xuất là thu thập phản hồi về một số ít đề xuất
được chọn để có thể đưa ra quyết định cuối cùng. Quy trình chạy ít nhiều song song
với một chu kỳ phát hành và bao gồm các bước sau:

1. _Lựa chọn đề xuất_. Nhóm Go chọn một số ít
[đề xuất Go 2](https://github.com/golang/go/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3AGo2+label%3AProposal)
có vẻ đáng xem xét để chấp nhận, mà không đưa ra quyết định cuối cùng.
Xem bên dưới để biết thêm về tiêu chí lựa chọn.

2. _Phản hồi đề xuất_. Nhóm Go gửi thông báo liệt kê các đề xuất được chọn. Thông
báo giải thích với cộng đồng ý định tạm thời để tiến hành với các đề xuất được chọn
và thu thập phản hồi cho từng đề xuất. Điều này cho cộng đồng cơ hội đưa ra gợi ý
và bày tỏ lo ngại.

3. _Triển khai_. Dựa trên phản hồi đó, các đề xuất được triển khai. Mục tiêu cho
những thay đổi ngôn ngữ và thư viện đáng kể này là chuẩn bị sẵn sàng để nộp vào
ngày 1 của chu kỳ phát hành sắp tới.

4. _Phản hồi triển khai_. Trong chu kỳ phát triển, nhóm Go và cộng đồng có cơ hội
thử nghiệm các tính năng mới và thu thập thêm phản hồi.

5. _Quyết định ra mắt_. Vào cuối [chu kỳ phát triển](/wiki/Go-Release-Cycle) ba
tháng (ngay khi bắt đầu đóng băng repo ba tháng trước một bản phát hành), và dựa
trên kinh nghiệm và phản hồi thu thập được trong chu kỳ phát hành, nhóm Go đưa ra
quyết định cuối cùng về việc có nên phát hành từng thay đổi không. Điều này cung
cấp cơ hội để xem xét liệu thay đổi có mang lại lợi ích mong đợi hay tạo ra bất
kỳ chi phí bất ngờ nào. Khi được phát hành, các thay đổi trở thành một phần của
ngôn ngữ và thư viện. Các đề xuất bị loại trừ có thể quay lại bàn vẽ hoặc bị từ
chối vĩnh viễn.

Với hai vòng phản hồi, quy trình này nghiêng về phía từ chối đề xuất, điều này hy
vọng ngăn chặn việc thêm tính năng tràn lan và giúp giữ ngôn ngữ nhỏ gọn và sạch sẽ.

Chúng tôi không thể thực hiện quy trình này cho từng đề xuất Go 2 mở, đơn giản là
có quá nhiều. Đó là nơi tiêu chí lựa chọn phát huy tác dụng.

## Tiêu chí lựa chọn đề xuất

Một đề xuất ít nhất phải:

1. _giải quyết vấn đề quan trọng cho nhiều người_,

2. _có tác động tối thiểu đến tất cả những người khác_, và

3. _đi kèm với giải pháp rõ ràng và được hiểu rõ_.

Yêu cầu 1 đảm bảo rằng bất kỳ thay đổi nào chúng tôi thực hiện giúp được nhiều
nhà phát triển Go nhất có thể (làm cho code của họ mạnh mẽ hơn, dễ viết hơn, có
nhiều khả năng đúng hơn, v.v.), trong khi yêu cầu 2 đảm bảo chúng tôi cẩn thận
để không làm hỏng nhiều nhà phát triển nhất có thể, dù bằng cách phá vỡ chương
trình của họ hay gây ra các thay đổi khác. Theo nguyên tắc chung, chúng tôi nên
nhắm đến việc giúp ít nhất mười lần nhiều nhà phát triển so với những người chúng
tôi làm tổn thương với một thay đổi nhất định. Các thay đổi không ảnh hưởng đến
việc sử dụng Go thực tế không có lợi ích ròng và có chi phí triển khai đáng kể
nên tránh.

Không có yêu cầu 3 chúng tôi không có triển khai đề xuất. Ví dụ, chúng tôi tin
rằng một dạng genericity nào đó có thể giải quyết vấn đề quan trọng cho nhiều
người, nhưng chúng tôi chưa có giải pháp rõ ràng và được hiểu rõ. Điều đó ổn,
nó chỉ có nghĩa là đề xuất cần quay lại bàn vẽ trước khi có thể được xem xét.

## Đề xuất

Chúng tôi cảm thấy đây là một kế hoạch tốt sẽ phục vụ chúng tôi tốt nhưng điều
quan trọng là phải hiểu rằng đây chỉ là điểm khởi đầu. Khi quy trình được sử dụng
chúng tôi sẽ khám phá những cách nó không hoạt động tốt và chúng tôi sẽ tinh chỉnh
khi cần. Phần quan trọng là cho đến khi chúng tôi sử dụng nó trong thực tế, chúng
tôi sẽ không biết cách cải thiện nó.

Nơi an toàn để bắt đầu là với một số lượng nhỏ đề xuất ngôn ngữ tương thích ngược.
Chúng tôi đã không thay đổi ngôn ngữ trong một thời gian dài, vì vậy điều này đưa
chúng tôi trở lại chế độ đó. Ngoài ra, các thay đổi sẽ không yêu cầu chúng tôi
lo lắng về việc phá vỡ code hiện có, và do đó chúng đóng vai trò là thử nghiệm
hoàn hảo.

Với tất cả những điều đó, chúng tôi đề xuất lựa chọn các đề xuất Go 2 sau cho bản
phát hành Go 1.13 (bước 1 trong quy trình đánh giá đề xuất):

1. [_\#20706_](/issue/20706) _Định danh Unicode chung dựa trên_ [_Unicode TR31_](http://unicode.org/reports/tr31/):
Điều này giải quyết vấn đề quan trọng cho các lập trình viên Go dùng bảng chữ cái
không phải phương Tây và nên có ít hoặc không có tác động đến bất kỳ ai khác. Có
các câu hỏi chuẩn hóa mà chúng tôi cần trả lời và nơi phản hồi cộng đồng sẽ quan
trọng, nhưng sau đó con đường triển khai được hiểu rõ. Lưu ý rằng các quy tắc
xuất định danh sẽ không bị ảnh hưởng bởi điều này.

2. [_\#19308_](/issue/19308), [_\#28493_](/issue/28493) _Ký tự nguyên số nguyên nhị phân và hỗ trợ \_ trong ký tự số_:
Đây là những thay đổi tương đối nhỏ có vẻ rất phổ biến trong nhiều lập trình viên.
Chúng có thể không đạt đến ngưỡng giải quyết "vấn đề quan trọng" (số thập lục phân
đã hoạt động tốt cho đến nay) nhưng chúng đưa Go lên ngang tầm với hầu hết các
ngôn ngữ khác về khía cạnh này và giảm bớt điểm đau cho một số lập trình viên.
Chúng có tác động tối thiểu đến những người khác không quan tâm đến ký tự nguyên
số nhị phân hoặc định dạng số, và việc triển khai được hiểu rõ.

3. [_\#19113_](/issue/19113) _Cho phép số nguyên có dấu làm đếm dịch chuyển_:
Ước tính 38% tất cả các phép dịch không hằng số yêu cầu chuyển đổi uint (nhân tạo)
(xem issue để biết thêm chi tiết). Đề xuất này sẽ dọn dẹp nhiều code, đồng bộ hóa
biểu thức dịch tốt hơn với biểu thức chỉ số và các hàm tích hợp cap và len. Nó
hầu hết sẽ có tác động tích cực đến code. Việc triển khai được hiểu rõ.

## Các bước tiếp theo

Với bài đăng này chúng tôi đã thực hiện bước đầu tiên và bắt đầu bước thứ hai của
quy trình đánh giá đề xuất. Giờ là tùy thuộc vào bạn, cộng đồng Go, để cung cấp
phản hồi về các issue được liệt kê ở trên.

Đối với mỗi đề xuất mà chúng tôi có phản hồi rõ ràng và chấp thuận, chúng tôi sẽ
tiến hành triển khai (bước 3 trong quy trình). Vì chúng tôi muốn các thay đổi được
triển khai vào ngày đầu tiên của chu kỳ phát hành tiếp theo (dự kiến ngày 1 tháng 2
năm 2019), chúng tôi có thể bắt đầu triển khai sớm hơn một chút lần này để có thời
gian cho hai tháng phản hồi đầy đủ (tháng 12 năm 2018, tháng 1 năm 2019).

Trong chu kỳ phát triển 3 tháng (tháng 2 đến tháng 5 năm 2019), các tính năng được
chọn được triển khai và có sẵn tại tip và tất cả mọi người sẽ có cơ hội tích lũy
kinh nghiệm với chúng. Điều này cung cấp cơ hội phản hồi khác (bước 4 trong quy
trình).

Cuối cùng, ngay sau khi đóng băng repo (ngày 1 tháng 5 năm 2019), nhóm Go đưa ra
quyết định cuối cùng về việc có giữ các tính năng mới mãi mãi (và bao gồm chúng
trong đảm bảo tương thích Go 1), hoặc có từ bỏ chúng hay không (bước cuối cùng
trong quy trình).

(Vì có khả năng thực sự là một tính năng có thể cần phải xóa đúng khi chúng tôi
đóng băng repo, việc triển khai sẽ cần phải như vậy để tính năng có thể được vô
hiệu hóa mà không làm mất ổn định phần còn lại của hệ thống. Đối với các thay đổi
ngôn ngữ, điều đó có thể có nghĩa là tất cả code liên quan đến tính năng được bảo
vệ bởi một cờ nội bộ.)

Đây sẽ là lần đầu tiên chúng tôi thực hiện quy trình này, do đó, việc đóng băng
repo cũng sẽ là thời điểm tốt để phản ánh về quy trình và điều chỉnh nếu cần.
Hãy xem nó diễn ra như thế nào.

Chúc đánh giá vui vẻ!
