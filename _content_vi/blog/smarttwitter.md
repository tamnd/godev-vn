---
title: "Dự án Go thực tế: SmartTwitter và web.go"
date: 2010-10-19
by:
- Michael Hoisie
tags:
- guest
summary: Cách Michael Hoisie dùng Go để xây dựng SmartTwitter và web.go.
---


_Bài viết tuần này do_ [_Michael Hoisie_](https://github.com/hoisie) _viết.
Là lập trình viên sống tại San Francisco, anh là một trong những người dùng sớm của Go và là tác giả của nhiều thư viện Go phổ biến. Anh chia sẻ kinh nghiệm của mình khi dùng Go:_

Tôi được giới thiệu về Go qua một bài đăng trên [Hacker News](https://news.ycombinator.com/).
Khoảng một giờ sau tôi đã bị cuốn hút. Lúc đó tôi đang làm việc tại một công ty khởi nghiệp web và phát triển các ứng dụng kiểm thử nội bộ bằng Python.
Go mang lại tốc độ, hỗ trợ concurrency tốt hơn và xử lý Unicode hợp lý, nên tôi rất muốn chuyển các chương trình của mình sang ngôn ngữ này.
Thời điểm đó chưa có cách dễ dàng để viết ứng dụng web bằng Go, vì vậy tôi quyết định xây dựng một framework web đơn giản, [web.go](https://github.com/hoisie/web).
Nó được phỏng theo một framework Python phổ biến, [web.py](https://webpy.org/), mà tôi đã làm việc trước đây.
Trong khi làm việc trên web.go, tôi tham gia vào cộng đồng Go, gửi một loạt báo cáo lỗi và đóng góp cho một số gói thư viện chuẩn (chủ yếu là [http](/pkg/http/) và [json](/pkg/json/)).

Sau vài tuần, tôi nhận thấy web.go đang được chú ý trên GitHub.
Điều này thật bất ngờ vì tôi chưa bao giờ thực sự quảng bá dự án.
Tôi nghĩ có một thị trường ngách cho các ứng dụng web đơn giản, nhanh và Go có thể lấp đầy nó.

Một cuối tuần tôi quyết định viết một ứng dụng Facebook đơn giản: nó sẽ đăng lại các cập nhật trạng thái Twitter của bạn lên hồ sơ Facebook của bạn.
Có một ứng dụng Twitter chính thức để làm điều này, nhưng nó đăng lại mọi thứ, tạo ra nhiễu trong feed Facebook của bạn.
Ứng dụng của tôi cho phép bạn lọc retweet, đề cập (mentions), hashtag, trả lời và nhiều hơn nữa.
Điều này trở thành [Smart Twitter](https://www.facebook.com/apps/application.php?id=135488932982), hiện có gần 90.000 người dùng.

Toàn bộ chương trình được viết bằng Go và sử dụng [Redis](https://redis.io/) làm backend lưu trữ.
Nó rất nhanh và bền vững. Hiện tại nó xử lý khoảng hai chục tweet mỗi giây và sử dụng nhiều channel của Go.
Nó chạy trên một instance Virtual Private Server với 2GB RAM, không gặp vấn đề gì khi xử lý tải.
Smart Twitter sử dụng rất ít thời gian CPU và hầu như hoàn toàn bị giới hạn bởi bộ nhớ vì toàn bộ cơ sở dữ liệu được giữ trong bộ nhớ.
Ở bất kỳ thời điểm nào đó cũng có khoảng 10 goroutine đang chạy đồng thời: một nhận kết nối HTTP, một đọc từ Twitter Streaming API, một vài cái để xử lý lỗi, và phần còn lại hoặc xử lý các yêu cầu web hoặc đăng lại các tweet đến.

Smart Twitter cũng tạo ra các dự án Go mã nguồn mở khác:
[mustache.go](https://github.com/hoisie/mustache),
[redis.go](https://github.com/hoisie/redis)
và [twitterstream](https://github.com/hoisie/twitterstream).

Tôi thấy còn nhiều việc cần làm trên web.go.
Ví dụ, tôi muốn thêm hỗ trợ tốt hơn cho streaming connection, websocket, route filter, hỗ trợ tốt hơn trong shared host và cải thiện tài liệu.
Gần đây tôi đã rời công ty khởi nghiệp để làm freelance phần mềm và dự định dùng Go bất cứ khi nào có thể.
Điều này có nghĩa là tôi sẽ dùng nó như backend cho các ứng dụng cá nhân cũng như cho các khách hàng thích làm việc với công nghệ tiên tiến.

Cuối cùng, tôi muốn cảm ơn nhóm Go vì tất cả nỗ lực của họ.
Go là một nền tảng tuyệt vời và tôi nghĩ nó có tương lai sáng lạn.
Tôi hy vọng thấy ngôn ngữ phát triển theo nhu cầu của cộng đồng.
Có rất nhiều thứ thú vị đang xảy ra trong cộng đồng và tôi mong chờ được thấy mọi người có thể tạo ra gì với ngôn ngữ này.
