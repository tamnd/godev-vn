---
title: Hai bài viết Go gần đây
date: 2013-03-06
by:
- Andrew Gerrand
tags:
- google
- talk
- ethos
summary: "Hai bài viết về Go: \"Go at Google: Language Design in the Service of Software Engineering\" và \"Getting Started with Go, App Engine and Google+ API\""
---

## Giới thiệu

Trong bài đăng blog hôm nay, tôi muốn giới thiệu một vài bài viết gần đây về Go.

## Go tại Google

Vào tháng 10 năm ngoái, Rob Pike đã trình bày một bài keynote tại hội nghị ACM [SPLASH](http://splashcon.org/2012/) ở Tucson.
Bài nói chuyện có tiêu đề [Go at Google](/talks/2012/splash.slide),
là một cuộc thảo luận toàn diện về các động lực đằng sau Go.
Rob sau đó đã mở rộng bài nói của mình thành một bài luận có tên
[Go at Google: Language Design in the Service of Software Engineering](/talks/2012/splash.article).
Đây là phần tóm tắt:

	Ngôn ngữ lập trình Go được hình thành vào cuối năm 2007 như một
	câu trả lời cho một số vấn đề chúng tôi gặp phải khi phát triển
	cơ sở hạ tầng phần mềm tại Google. Bối cảnh điện toán ngày nay
	hầu như không còn liên quan đến môi trường mà các ngôn ngữ đang được
	sử dụng, chủ yếu là C++, Java và Python, đã được tạo ra. Các vấn đề
	do bộ vi xử lý đa lõi, hệ thống mạng, cụm tính toán khổng lồ và mô
	hình lập trình web gây ra đã được xử lý vòng vèo thay vì giải quyết
	trực tiếp. Hơn nữa, quy mô đã thay đổi: các chương trình máy chủ
	ngày nay bao gồm hàng chục triệu dòng mã, được hàng trăm hoặc hàng
	nghìn lập trình viên làm việc cùng nhau, và được cập nhật hàng ngày.
	Tệ hơn nữa, thời gian build, ngay cả trên các cụm biên dịch lớn,
	đã kéo dài đến nhiều phút, thậm chí nhiều giờ.

	Go được thiết kế và phát triển để làm cho việc làm việc trong môi
	trường này trở nên hiệu quả hơn. Ngoài các khía cạnh được biết đến
	như tính đồng thời tích hợp sẵn và bộ gom rác, các cân nhắc thiết kế
	của Go bao gồm quản lý dependency nghiêm ngặt, khả năng thích ứng
	của kiến trúc phần mềm khi hệ thống phát triển, và tính mạnh mẽ trên
	các ranh giới giữa các thành phần.

Bài viết này giải thích cách những vấn đề này được giải quyết trong khi xây dựng một ngôn ngữ
lập trình biên dịch hiệu quả nhưng lại cảm giác nhẹ nhàng và dễ chịu.
Các ví dụ và giải thích sẽ được lấy từ các vấn đề thực tế gặp phải tại Google.

Nếu bạn từng thắc mắc về các quyết định thiết kế đằng sau Go,
bạn có thể tìm thấy câu trả lời trong [bài luận đó](/talks/2012/splash.article).
Đây là tài liệu đọc được khuyến nghị cho cả lập trình viên Go mới lẫn có kinh nghiệm.

## Go tại Google Developers Academy

Tại Google I/O 2012, nhóm Google Developers đã [ra mắt](http://googledevelopers.blogspot.com.au/2012/06/google-launches-new-developer-education.html)
[Google Developers Academy](https://developers.google.com/academy/),
một chương trình cung cấp tài liệu đào tạo về các công nghệ của Google.
Go là một trong những công nghệ đó và chúng tôi vui mừng thông báo bài viết GDA đầu tiên
có Go ở vị trí trung tâm:

[Getting Started with Go, App Engine and Google+ API](https://developers.google.com/appengine/training/go-plus-appengine/) là
phần giới thiệu về cách viết ứng dụng web bằng Go.
Bài viết trình bày cách xây dựng và triển khai ứng dụng App Engine cũng như thực hiện
các lời gọi tới Google+ API bằng Google APIs Go Client.
Đây là điểm khởi đầu tuyệt vời cho các lập trình viên Go muốn bắt đầu với hệ sinh thái phát triển của Google.
