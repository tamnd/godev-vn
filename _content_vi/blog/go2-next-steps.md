---
title: Các bước tiếp theo hướng tới Go 2
date: 2019-06-26
by:
- Robert Griesemer, for the Go team
tags:
- go2
- proposals
- community
summary: Những thay đổi ngôn ngữ Go 2 nào chúng ta nên đưa vào Go 1.14?
---

## Tình trạng

Chúng tôi đang trên đường hướng tới bản phát hành Go 1.13,
hy vọng vào đầu tháng 8 năm nay.
Đây là bản phát hành đầu tiên sẽ bao gồm những thay đổi cụ thể
cho ngôn ngữ (chứ không chỉ là điều chỉnh nhỏ đối với đặc tả),
sau một thời gian tạm dừng lâu hơn đối với bất kỳ thay đổi nào như vậy.

Để đến được những thay đổi ngôn ngữ này,
chúng tôi bắt đầu với một tập hợp nhỏ các đề xuất khả thi,
được chọn từ danh sách lớn hơn nhiều gồm
[đề xuất Go 2](https://github.com/golang/go/issues?utf8=%E2%9C%93&q=is%3Aissue+is%3Aopen+label%3AGo2+label%3AProposal),
theo quy trình đánh giá đề xuất mới được phác thảo
trong bài đăng blog "[Go 2, đây chúng tôi đến!](/blog/go2-here-we-come)".
Chúng tôi muốn lựa chọn đề xuất ban đầu của mình
tương đối nhỏ và hầu hết không gây tranh cãi,
để có cơ hội hợp lý để chúng
vượt qua quy trình.
Các thay đổi đề xuất phải tương thích ngược
để có tác động gián đoạn tối thiểu vì
[module](/blog/using-go-modules),
cuối cùng sẽ cho phép lựa chọn phiên bản ngôn ngữ dành riêng cho module,
chưa phải là chế độ build mặc định.
Tóm lại, vòng thay đổi ban đầu này là về
việc đưa quả bóng lăn trở lại và tích lũy kinh nghiệm
với quy trình mới, thay vì giải quyết các vấn đề lớn.

[Danh sách đề xuất ban đầu](/blog/go2-here-we-come) của chúng tôi -
[định danh Unicode chung](/issue/20706),
[ký tự nguyên số nhị phân](/issue/19308),
[dấu phân cách cho ký tự số](/issue/28493),
[đếm dịch chuyển số nguyên có dấu](/issue/19113) -
đã được cả thu hẹp và mở rộng.
Các định danh Unicode chung không đạt tiêu chí
vì chúng tôi không có tài liệu thiết kế cụ thể kịp thời.
Đề xuất cho ký tự nguyên số nhị phân được mở rộng đáng kể
và dẫn đến một cuộc cải tổ và hiện đại hóa toàn diện của
[cú pháp ký tự số của Go](/design/19308-number-literals).
Và chúng tôi đã thêm đề xuất bản thảo thiết kế Go 2 về
[kiểm tra lỗi](/design/go2draft-error-inspection),
đã được
[chấp nhận một phần](/issue/29934#issuecomment-489682919).

Với những thay đổi ban đầu này có trong Go 1.13,
giờ là lúc nhìn về phía trước đến Go 1.14
và xác định những gì chúng tôi muốn giải quyết tiếp theo.

## Đề xuất cho Go 1.14

Các mục tiêu chúng tôi có cho Go ngày nay cũng giống như năm 2007: để
[làm cho phát triển phần mềm có khả năng mở rộng](/blog/toward-go2).
Ba rào cản lớn nhất trên con đường đến cải thiện khả năng mở rộng cho Go là
quản lý gói và phiên bản,
hỗ trợ xử lý lỗi tốt hơn,
và generics.

Với hỗ trợ module Go ngày càng mạnh hơn,
hỗ trợ cho quản lý gói và phiên bản đang được giải quyết.
Điều này để lại hỗ trợ xử lý lỗi tốt hơn và generics.
Chúng tôi đã làm việc về cả hai và trình bày
[các bản thảo thiết kế](/design/go2draft)
tại GopherCon năm ngoái ở Denver.
Từ đó chúng tôi đã lặp đi lặp lại các thiết kế đó.
Đối với xử lý lỗi, chúng tôi đã xuất bản một đề xuất cụ thể,
được sửa đổi và đơn giản hóa đáng kể (xem bên dưới).
Đối với generics, chúng tôi đang tiến triển, với một bài nói chuyện
("Generics in Go" của Ian Lance Taylor)
[sắp diễn ra](https://www.gophercon.com/agenda/session/49028)
tại GopherCon năm nay ở San Diego,
nhưng chúng tôi chưa đến giai đoạn đề xuất cụ thể.

Chúng tôi cũng muốn tiếp tục với những cải tiến nhỏ hơn
cho ngôn ngữ.
Đối với Go 1.14, chúng tôi đã chọn các đề xuất sau:

[\#32437](/issue/32437).
Hàm kiểm tra lỗi Go tích hợp, "try"
([tài liệu thiết kế](/design/32437-try-builtin)).

Đây là đề xuất cụ thể của chúng tôi cho việc cải thiện xử lý lỗi.
Mặc dù extension ngôn ngữ được đề xuất, hoàn toàn tương thích ngược
là tối thiểu, chúng tôi kỳ vọng tác động lớn đến code xử lý lỗi.
Đề xuất này đã thu hút số lượng bình luận khổng lồ,
và không dễ để theo dõi.
Chúng tôi khuyến nghị bắt đầu với
[bình luận ban đầu](/issue/32437#issue-452239211)
để có phác thảo nhanh và sau đó đọc tài liệu thiết kế chi tiết.
Bình luận ban đầu chứa một vài liên kết dẫn đến tóm tắt
của phản hồi cho đến nay.
Vui lòng tuân theo các khuyến nghị phản hồi
(xem phần "Các bước tiếp theo" bên dưới) trước khi đăng.

[\#6977](/issue/6977).
Cho phép nhúng các interface chồng chéo
([tài liệu thiết kế](/design/6977-overlapping-interfaces)).

Đây là đề xuất cũ, tương thích ngược để làm cho việc nhúng interface
dễ dung túng hơn.

[\#32479](/issue/32479) Chẩn đoán chuyển đổi `string(int)` trong `go vet`.

Chuyển đổi `string(int)` được giới thiệu sớm trong Go để thuận tiện,
nhưng nó gây nhầm lẫn cho người mới (`string(10)` là `"\n"` không phải `"10"`)
và không còn hợp lý nữa khi chuyển đổi có sẵn
trong gói `unicode/utf8`.
Vì loại bỏ chuyển đổi này không phải là thay đổi tương thích ngược,
chúng tôi đề xuất bắt đầu bằng lỗi `vet` thay thế.

[\#32466](/issue/32466) Áp dụng các nguyên tắc mật mã
([tài liệu thiết kế](/design/cryptography-principles)).

Đây là yêu cầu phản hồi về một tập hợp các nguyên tắc thiết kế cho
các thư viện mật mã mà chúng tôi muốn áp dụng.
Xem thêm
[đề xuất liên quan để xóa hỗ trợ SSLv3](/issue/32716)
khỏi `crypto/tls`.

## Các bước tiếp theo

Chúng tôi đang tích cực thu hút phản hồi về tất cả các đề xuất này.
Chúng tôi đặc biệt quan tâm đến bằng chứng dựa trên thực tế
minh họa lý do tại sao một đề xuất có thể không hoạt động tốt trong thực tế,
hoặc các khía cạnh vấn đề mà chúng tôi có thể đã bỏ sót trong thiết kế.
Các ví dụ thuyết phục ủng hộ một đề xuất cũng rất hữu ích.
Mặt khác, các bình luận chỉ chứa ý kiến cá nhân
ít có thể thực hiện được hơn:
chúng tôi có thể thừa nhận chúng nhưng không thể giải quyết chúng
theo bất kỳ cách xây dựng nào.
Trước khi đăng, hãy dành thời gian đọc các
tài liệu thiết kế chi tiết và phản hồi trước hoặc tóm tắt phản hồi.
Đặc biệt trong các cuộc thảo luận dài, mối lo ngại của bạn có thể đã
được nêu ra và thảo luận trong các bình luận trước.

Trừ khi có lý do mạnh mẽ để thậm chí không tiến vào giai đoạn
thử nghiệm với một đề xuất nhất định,
chúng tôi đang có kế hoạch triển khai tất cả các đề xuất này vào đầu
[chu kỳ Go 1.14](/wiki/Go-Release-Cycle)
(đầu tháng 8 năm 2019)
để chúng có thể được đánh giá trong thực tế.
Theo
[quy trình đánh giá đề xuất](/blog/go2-here-we-come),
quyết định cuối cùng sẽ được
đưa ra vào cuối chu kỳ phát triển (đầu tháng 11 năm 2019).

Cảm ơn bạn đã giúp làm cho Go trở thành ngôn ngữ tốt hơn!
