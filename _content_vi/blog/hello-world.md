---
title: "Go: Những điều mới trong tháng 3 năm 2010"
date: 2010-03-18
by:
- Andrew Gerrand
summary: Bài đăng đầu tiên!
---


Chào mừng đến với Blog Go chính thức. Chúng tôi, nhóm Go,
hy vọng sẽ sử dụng blog này để cập nhật cho thế giới về sự phát triển của
ngôn ngữ lập trình Go và hệ sinh thái thư viện và ứng dụng ngày càng lớn mạnh xung quanh nó.

Đã vài tháng kể từ khi chúng tôi ra mắt (tháng 11 năm ngoái),
vì vậy hãy nói về những gì đã xảy ra trong thế giới Go kể từ đó.

Nhóm cốt lõi tại Google đã tiếp tục phát triển ngôn ngữ,
trình biên dịch, các gói, công cụ và tài liệu.
Các trình biên dịch giờ tạo ra code trong một số trường hợp nhanh hơn từ 2x đến một bậc độ lớn
so với lúc phát hành.
Chúng tôi đã tổng hợp một số biểu đồ về một số [Benchmark](http://godashboard.appspot.com/benchmarks) được chọn lọc,
và trang [Build Status](http://godashboard.appspot.com/) theo dõi
độ tin cậy của mỗi changeset được gửi lên kho lưu trữ.

Chúng tôi đã thực hiện các thay đổi cú pháp để làm cho ngôn ngữ ngắn gọn hơn,
nhất quán và linh hoạt hơn.
Dấu chấm phẩy đã được [hầu như loại bỏ hoàn toàn](http://groups.google.com/group/golang-nuts/t/5ee32b588d10f2e9) khỏi ngôn ngữ.
Cú pháp [...T](/doc/go_spec.html#Function_types)
giúp đơn giản hóa việc xử lý số lượng tham số hàm có kiểu tùy ý.
Cú pháp x[lo:] giờ là viết tắt của x[lo:len(x)].
Go cũng giờ hỗ trợ số phức một cách tự nhiên.
Xem [ghi chú phát hành](/doc/devel/release.html) để biết thêm.

[Godoc](/cmd/godoc/) giờ cung cấp hỗ trợ tốt hơn cho
các thư viện bên thứ ba,
và một công cụ mới - [goinstall](/cmd/goinstall) - đã được
phát hành để giúp dễ dàng cài đặt chúng.
Ngoài ra, chúng tôi đã bắt đầu làm việc trên một hệ thống theo dõi gói để làm
cho việc tìm kiếm những gì bạn cần dễ dàng hơn.
Bạn có thể xem phần đầu của điều này trên [trang Gói](http://godashboard.appspot.com/package).

Hơn 40.000 dòng code đã được thêm vào [thư viện chuẩn](/pkg/),
bao gồm nhiều gói hoàn toàn mới, một phần đáng kể được viết bởi các cộng tác viên bên ngoài.

Nói về bên thứ ba, kể từ khi ra mắt, một cộng đồng sôi động đã phát triển
trên [danh sách thư của chúng tôi](http://groups.google.com/group/golang-nuts/) và
kênh irc (#go-nuts trên freenode).
Chúng tôi đã chính thức thêm hơn 50 người vào dự án.
Những đóng góp của họ trải dài từ sửa lỗi và chỉnh sửa tài liệu đến
các gói cốt lõi và hỗ trợ cho các hệ điều hành bổ sung (Go giờ được hỗ trợ trên FreeBSD,
và một [cổng Windows](http://code.google.com/p/go/wiki/WindowsPort) đang được thực hiện).
Chúng tôi coi những đóng góp cộng đồng này là thành công lớn nhất của chúng tôi cho đến nay.

Chúng tôi cũng đã nhận được một số đánh giá tốt. [Bài báo gần đây trên PC World](http://www.pcworld.idg.com.au/article/337773/google_go_captures_developers_imaginations/)
này đã tóm tắt sự nhiệt tình xung quanh dự án.
Một số blogger đã bắt đầu ghi lại kinh nghiệm của họ với ngôn ngữ
(xem [đây](http://golang.tumblr.com/),
[đây](http://www.infi.nl/blog/view/id/47),
và [đây](http://freecella.blogspot.com/2010/01/gospecify-basic-setup-of-projects.html)
chẳng hạn). Phản ứng chung của người dùng chúng tôi là rất tích cực;
một người dùng lần đầu đã nhận xét ["Tôi rời đi vô cùng ấn tượng. Go đi trên một ranh giới thanh lịch giữa sự đơn giản và sức mạnh."](https://groups.google.com/group/golang-nuts/browse_thread/thread/5fabdd59f8562ed2)

Về tương lai: chúng tôi đã lắng nghe vô số giọng nói cho chúng tôi biết những gì họ cần,
và giờ đang tập trung vào việc chuẩn bị Go sẵn sàng cho thời đại hoàng kim.
Chúng tôi đang cải thiện bộ thu gom rác, bộ lập lịch runtime,
công cụ và thư viện chuẩn, cũng như khám phá các tính năng ngôn ngữ mới.
Năm 2010 sẽ là một năm thú vị cho Go, và chúng tôi mong muốn được hợp tác
với cộng đồng để biến nó thành một năm thành công.
