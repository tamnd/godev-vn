---
title: Go trở nên ổn định hơn
date: 2011-03-16
by:
- Andrew Gerrand
tags:
- release
summary: Chuyển từ các bản phát hành hàng tuần không ổn định sang các bản phát hành ít thường xuyên hơn nhưng ổn định hơn.
---


Dự án Go đang tiến triển nhanh chóng. Khi chúng tôi hiểu sâu hơn về Go,
chúng tôi buộc phải thay đổi các công cụ, thư viện,
và đôi khi cả bản thân ngôn ngữ.
Chúng tôi cho phép các thay đổi không tương thích ngược để có thể học hỏi từ
những sai lầm thay vì bất tử hóa chúng.
Chúng tôi tin rằng tính linh hoạt ở giai đoạn phát triển này của Go là điều cần thiết
cho sự phát triển của dự án và, cuối cùng, là tuổi thọ của nó.

Kể từ khi Go ra mắt, chúng tôi đã phát hành các bản cập nhật khoảng một lần mỗi tuần.
Mỗi bản phát hành đều kèm theo [ghi chú mô tả những thay đổi](/doc/devel/release.html),
trong đó các thay đổi không tương thích ngược được đánh dấu rõ ràng.
Câu hỏi tôi thường nghe là "Go có ổn định không? Làm sao tôi chắc rằng tôi không cần
cập nhật mã Go của mình mỗi tuần?" Câu trả lời cho những câu hỏi đó bây giờ là "Có,"
và "Bạn sẽ không cần phải làm vậy."

Với bản phát hành tuần này, chúng tôi đang giới thiệu một sơ đồ gắn thẻ bản phát hành mới.
Chúng tôi dự định tiếp tục với các bản phát hành hàng tuần,
nhưng đã đổi tên các thẻ hiện có từ `release` thành `weekly`.
Thẻ `release` sẽ chỉ được áp dụng cho một bản phát hành ổn định được chọn lọc kỹ càng
mỗi một đến hai tháng một lần.
Lịch phát hành thoải mái hơn này sẽ giúp cuộc sống của các lập trình viên Go phổ thông
dễ dàng hơn.

Người dùng vẫn sẽ cần cập nhật mã của họ định kỳ (đây là cái giá phải trả khi sử dụng
một ngôn ngữ còn non trẻ) nhưng ít thường xuyên hơn.
Một lợi ích bổ sung là bằng cách gắn thẻ các bản phát hành ổn định ít thường xuyên hơn,
chúng tôi có thể dành nhiều công sức hơn vào việc tự động hóa các bản cập nhật.
Vì mục đích này, chúng tôi đã giới thiệu gofix, một công cụ sẽ giúp bạn cập nhật mã của mình.

Phiên bản trước đây được gắn thẻ `release.2011-03-07.1` (nay là `weekly.2011-03-07.1`)
đã được đề cử là bản phát hành ổn định đầu tiên của chúng tôi,
và được đặt thẻ `release.r56`.
Khi chúng tôi gắn thẻ mỗi bản phát hành ổn định, chúng tôi sẽ đăng thông báo lên danh sách gửi thư
[golang-announce](http://groups.google.com/group/golang-announce) mới.
(Tại sao không [đăng ký ngay bây giờ](http://groups.google.com/group/golang-announce/subscribe)?)

Kết quả của tất cả những điều này là gì? Bạn có thể tiếp tục giữ cài đặt Go của mình
luôn được cập nhật bằng `hg update release`,
nhưng bây giờ bạn chỉ cần cập nhật khi chúng tôi gắn thẻ một bản phát hành ổn định mới.
Nếu bạn muốn đứng ở tuyến đầu, hãy chuyển sang thẻ weekly
bằng `hg update weekly`.

Chúc bạn lập trình vui vẻ!
