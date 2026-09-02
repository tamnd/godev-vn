---
title: Go, Cộng đồng Go, và Đại dịch
date: 2020-03-25
by:
- Carmen Andoh
- Russ Cox
- Steve Francia
summary: Nhóm Go đang tiếp cận đại dịch như thế nào, những gì bạn có thể mong đợi từ chúng tôi, và những gì bạn có thể làm.
---


Go luôn đứng sau những mối quan tâm cơ bản hơn như sức khỏe và an toàn cá nhân và gia đình.
Trên khắp thế giới, vài tháng vừa qua đã rất khủng khiếp,
và chúng ta vẫn đang ở giai đoạn đầu của đại dịch khủng khiếp này.
Có những ngày mà việc làm bất cứ điều gì liên quan đến Go dường như là một sự đảo ngược ưu tiên nghiêm trọng.

Nhưng sau khi chúng ta đã làm tất cả những gì có thể
để chuẩn bị cho bản thân và gia đình cho bất cứ điều gì đang đến,
việc quay trở lại một phép gần đúng của thói quen quen thuộc
và công việc bình thường là một cơ chế đối phó hữu ích.
Theo tinh thần đó, chúng tôi dự định tiếp tục làm việc trên Go
và cố gắng giúp đỡ cộng đồng Go nhiều nhất có thể.

Trong bài đăng này, chúng tôi muốn chia sẻ một vài ghi chú quan trọng về
cách đại dịch ảnh hưởng đến cộng đồng Go,
một vài điều chúng tôi đang làm để giúp đỡ, những gì bạn có thể làm để giúp đỡ,
và kế hoạch của chúng tôi cho Go.

## Hội nghị và buổi gặp mặt

Cộng đồng Go phát triển mạnh dựa trên các hội nghị và buổi gặp mặt trực tiếp.
Chúng tôi đã dự đoán 35 hội nghị trong năm nay
và hàng nghìn buổi gặp mặt, gần như tất cả đã
được thay đổi, hoãn lại, hoặc hủy bỏ.
Chúng tôi sẽ cập nhật
[trang wiki hội nghị](/wiki/Conferences)
khi kế hoạch thay đổi.

Chúng tôi muốn làm tất cả những gì có thể để hỗ trợ các hội nghị Go bị ảnh hưởng.
Chúng tôi cũng muốn hỗ trợ các nỗ lực khám phá
các cách mới để các gopher kết nối trong thời gian giãn cách xã hội.
Ngoài việc tôn trọng các tài trợ hiện có của Google,
chúng tôi quan tâm đến việc cung cấp hỗ trợ cho những người lên kế hoạch
các hội nghị ảo thay thế trong phần còn lại của năm.
Nếu bạn đang tổ chức hội nghị Go và đã bị ảnh hưởng,
hoặc nếu bạn đang cân nhắc tổ chức hội nghị ảo thay thế,
vui lòng liên hệ với Carmen Andoh tại _candoh@google.com_.

Đối với các nhà tổ chức hội nghị,
kênh [#conf-organizers](https://app.slack.com/client/T029RQSE6/C97B0NCVD) trên
[Gophers Slack](https://gophers.slack.com)
là nơi để thảo luận về kế hoạch dự phòng,
thực hành tốt nhất, hủy bỏ và hỗ trợ hoãn lại.
Đây cũng là nơi để chia sẻ ý tưởng cho các sự kiện ảo,
để tiếp tục kết nối và hỗ trợ cộng đồng Go.

Đối với các nhà tổ chức buổi gặp mặt,
[Go Developer Network](https://www.meetup.com/pro/go)
có thể cung cấp giấy phép Zoom for Education cho các buổi gặp mặt
muốn bắt đầu tổ chức các cuộc họp ảo.
Nếu bạn tổ chức một buổi gặp mặt, hoặc bạn muốn tổ chức, chúng tôi khuyến khích bạn
sử dụng cơ hội này để mời các diễn giả từ bên ngoài
khu vực của bạn thuyết trình cho nhóm của bạn.
Để biết thêm thông tin và tham gia,
vui lòng tham gia kênh
[#remotemeetup](https://app.slack.com/client/T029RQSE6/C152YB9UZ) trên
[Gophers Slack](https://gophers.slack.com).

## Đào tạo trực tuyến

Các huấn luyện viên Go mà bạn gặp tại các hội nghị cũng đi khắp thế giới để thực hiện
[đào tạo trực tiếp](/learn/)
cho các công ty muốn được hỗ trợ áp dụng Go.
Việc dạy học trực tiếp đó rất quan trọng để mang
các gopher mới vào cộng đồng;
chúng tôi vô cùng biết ơn các huấn luyện viên vì công việc họ làm.
Thật không may, tất cả các hợp đồng đào tạo tại chỗ đã bị hủy
trong vài tháng tới, và các huấn luyện viên trong cộng đồng của chúng tôi
đã mất nguồn thu nhập chính (hoặc duy nhất) của họ.
Chúng tôi khuyến khích các công ty xem xét đào tạo ảo
và hội thảo trong thời gian khó khăn này.
Hầu hết các huấn luyện viên đang linh hoạt về giá cả,
lịch trình và cấu trúc lớp học.

## Đăng tin tuyển dụng

Chúng tôi biết rằng sự suy thoái hiện tại có nghĩa là một số
gopher đang tìm kiếm công việc mới.
Cộng đồng Go đã xây dựng một số trang đăng tin tuyển dụng đặc thù Go, bao gồm
[Golang Cafe](https://golang.cafe/),
[Golang Projects](https://www.golangprojects.com/),
và
[We Love Go](https://www.welovegolang.com).
[Gophers Slack](https://gophers.slack.com)
cũng có nhiều kênh săn việc: tìm kiếm "job" trong danh sách kênh.
Chúng tôi khuyến khích các nhà tuyển dụng có vị trí mở nào đăng ở
nhiều nơi phù hợp nhất có thể.

## FOSS Responders

Chúng tôi tự hào rằng Go là một phần của hệ sinh thái mã nguồn mở rộng lớn hơn.
[FOSS Responders](https://fossresponders.com)
là một nỗ lực để giúp hệ sinh thái mã nguồn mở
đối phó với tác động của đại dịch.
Nếu bạn muốn làm điều gì đó để giúp đỡ các cộng đồng mã nguồn mở bị ảnh hưởng,
họ đang phối hợp các nỗ lực và cũng có các liên kết đến các nỗ lực khác.
Và nếu bạn biết về các cộng đồng mã nguồn mở khác cần giúp đỡ,
hãy cho họ biết về FOSS Responders.

## COVID-19 Open-Source Help Desk

[COVID-19 Open-Source Help Desk](https://covid-oss-help.org/)
nhằm mục đích giúp đỡ các nhà virus học, dịch tễ học và các chuyên gia lĩnh vực khác
tìm câu trả lời nhanh cho bất kỳ vấn đề nào họ đang gặp phải với
phần mềm tính toán khoa học mã nguồn mở,
từ các chuyên gia trong phần mềm đó,
để họ có thể tập trung thời gian vào những gì họ biết tốt nhất.
Nếu bạn là nhà phát triển hoặc chuyên gia tính toán khoa học
sẵn sàng giúp đỡ bằng cách trả lời các bài đăng của các chuyên gia lĩnh vực,
hãy truy cập trang web để tìm hiểu cách giúp đỡ.

## U.S. Digital Response

Đối với các gopher ở Hoa Kỳ,
[U.S. Digital Response](https://www.usdigitalresponse.org/)
đang làm việc để kết nối các tình nguyện viên đủ điều kiện với
các chính quyền tiểu bang và địa phương cần giúp đỡ kỹ thuật số
trong cuộc khủng hoảng này.
Trích dẫn từ trang web,
"Nếu bạn có kinh nghiệm liên quan
(chăm sóc sức khỏe, dữ liệu, kỹ thuật và phát triển sản phẩm,
quản lý chung, vận hành, chuỗi cung ứng/mua sắm và nhiều hơn nữa),
có thể làm việc tự chủ trong sự mơ hồ,
và sẵn sàng nhảy vào một môi trường cường độ cao,"
xem trang để biết cách tình nguyện.

## Kế hoạch cho Go

Ở đây, tại nhóm Go tại Google, chúng tôi nhận ra rằng
thế giới xung quanh chúng ta đang thay đổi nhanh chóng
và các kế hoạch vượt xa vài tuần tới
không hơn gì những đoán mò đầy hy vọng.
Nói vậy, ngay bây giờ chúng tôi đang làm việc
trên những gì chúng tôi nghĩ là các dự án quan trọng nhất cho năm 2020.
Giống như tất cả các bạn, chúng tôi đang ở công suất thấp hơn, vì vậy công việc
tiếp tục chậm hơn dự kiến.

Phân tích của chúng tôi về khảo sát người dùng Go 2019 gần hoàn thành,
và chúng tôi hy vọng sẽ đăng nó sớm.

Ít nhất là bây giờ, chúng tôi dự định duy trì lịch trình của mình cho Go 1.15,
với sự hiểu biết rằng nó có thể sẽ có ít tính năng mới hơn
và cải tiến so với những gì chúng tôi đã lên kế hoạch ban đầu.
Chúng tôi tiếp tục thực hiện đánh giá code, phân loại issue,
và [đánh giá đề xuất](/s/proposal-minutes).

[Gopls](https://go.googlesource.com/tools/+/refs/heads/master/gopls/README.md)
là backend nhận thức ngôn ngữ hỗ trợ hầu hết các trình soạn thảo Go ngày nay,
và chúng tôi tiếp tục làm việc hướng tới bản phát hành 1.0 của nó.

Trang gói và module Go mới [pkg.go.dev](https://pkg.go.dev)
ngày càng trở nên tốt hơn.
Chúng tôi đang làm việc trên các cải tiến khả năng sử dụng
và các tính năng mới để giúp người dùng tìm kiếm và đánh giá các gói Go tốt hơn.
Chúng tôi cũng đã mở rộng tập hợp các giấy phép được công nhận và cải thiện
bộ phát hiện giấy phép, với nhiều cải tiến hơn sắp tới.

[Các giá trị Gopher](/conduct#values) của chúng tôi
là những gì giữ chúng tôi vững chắc, bây giờ hơn bao giờ hết.
Chúng tôi đang nỗ lực thêm để thân thiện, chào đón,
kiên nhẫn, chu đáo, tôn trọng và bao dung.
Chúng tôi hy vọng mọi người trong cộng đồng Go sẽ cố gắng làm tương tự.

Chúng tôi sẽ tiếp tục sử dụng blog này để cho bạn biết về
tin tức quan trọng cho hệ sinh thái Go.
Trong những khoảnh khắc khi bạn đã chăm sóc những điều quan trọng hơn nhiều
đang xảy ra trong cuộc sống của bạn,
chúng tôi hy vọng bạn sẽ ghé thăm và xem chúng tôi đã làm gì.

Cảm ơn, như thường lệ, vì đã sử dụng Go và là một phần của cộng đồng Go.
Chúng tôi chúc tất cả các bạn những điều tốt nhất trong những thời điểm khó khăn này.
