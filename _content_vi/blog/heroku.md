---
title: Go tại Heroku
date: 2011-04-21
by:
- Keith Rarick
- Blake Mizerany
tags:
- guest
summary: Hai kỹ sư hệ thống của Heroku chia sẻ kinh nghiệm sử dụng Go.
---


_Bài đăng blog tuần này được viết bởi_ [_Keith Rarick_](http://xph.us/)
_và_ [_Blake Mizerany_](http://itsbonus.heroku.com/),
_kỹ sư hệ thống tại_ [Heroku](http://www.heroku.com/).
_Theo lời của họ, họ "ăn, ngủ và mơ về hệ thống phân tán." Ở đây họ chia sẻ kinh nghiệm sử dụng Go._

Một vấn đề lớn đi kèm với việc xây dựng hệ thống phân tán là việc điều phối
các máy chủ vật lý.
Mỗi máy chủ cần biết các thông tin khác nhau về hệ thống như một tổng thể.
Dữ liệu quan trọng này bao gồm khóa, dữ liệu cấu hình,
và nhiều thứ khác, và nó phải nhất quán và có sẵn ngay cả khi kho dữ liệu gặp sự cố,
vì vậy chúng tôi cần một kho dữ liệu với đảm bảo tính nhất quán vững chắc.
Giải pháp của chúng tôi cho vấn đề này là [Doozer](http://xph.us/2011/04/13/introducing-doozer.html),
một kho dữ liệu mới, nhất quán, có tính sẵn sàng cao được viết bằng Go.

Cốt lõi của Doozer là [Paxos](http://en.wikipedia.org/wiki/Paxos_(computer_science)),
một họ các giao thức để giải quyết sự đồng thuận trong một mạng không đáng tin cậy của các nút không đáng tin cậy.
Mặc dù Paxos là thiết yếu để chạy một hệ thống chịu lỗi,
nó nổi tiếng là khó triển khai.
Ngay cả các triển khai ví dụ có thể tìm thấy trực tuyến cũng phức tạp và khó theo dõi,
mặc dù đã được đơn giản hóa cho mục đích giáo dục.
Các hệ thống sản xuất hiện có có tiếng là còn tệ hơn.

May mắn thay, các nguyên thủy tương tranh của Go đã làm cho nhiệm vụ dễ dàng hơn nhiều.
Paxos được định nghĩa theo các tiến trình độc lập,
đồng thời giao tiếp thông qua truyền thông điệp.
Trong Doozer, các tiến trình này được triển khai dưới dạng goroutine,
và các giao tiếp của chúng là các thao tác channel.
Theo cách mà bộ thu gom rác cải thiện trên malloc và free,
chúng tôi nhận thấy rằng [goroutine và channel](/blog/share-memory-by-communicating)
cải thiện trên cách tiếp cận dựa trên khóa để xử lý tương tranh.
Các công cụ này cho phép chúng tôi tránh quản lý sổ sách phức tạp và tập trung vào vấn đề trong tầm tay.
Chúng tôi vẫn ngạc nhiên về việc chỉ cần bao ít dòng code để đạt được điều gì đó
nổi tiếng là khó khăn.

Các gói chuẩn trong Go là một thắng lợi lớn khác cho Doozer.
Nhóm Go rất thực dụng về những gì đưa vào chúng.
Chẳng hạn, một gói mà chúng tôi nhanh chóng thấy hữu ích là [websocket](/pkg/websocket/).
Khi chúng tôi đã có một kho dữ liệu hoạt động, chúng tôi cần một cách dễ dàng để xem xét nó
và hình dung hoạt động.
Sử dụng gói websocket, Keith đã có thể thêm trình xem web trên chuyến tàu về nhà của anh ấy
mà không cần các dependency bên ngoài.
Đây là bằng chứng thực sự cho thấy Go kết hợp tốt như thế nào giữa lập trình hệ thống và ứng dụng.

Một trong những cải tiến năng suất yêu thích của chúng tôi được cung cấp bởi trình định dạng mã nguồn của Go:
[gofmt](/cmd/gofmt/).
Chúng tôi không bao giờ tranh luận về nơi đặt dấu ngoặc nhọn,
tab so với khoảng trắng, hoặc liệu chúng tôi có nên căn chỉnh các phép gán không.
Chúng tôi đơn giản đồng ý rằng mọi thứ dừng lại ở đầu ra mặc định từ gofmt.

Triển khai Doozer thật đơn giản và thỏa mãn.
Go xây dựng các binary liên kết tĩnh có nghĩa là Doozer không có dependency bên ngoài;
đó là một tệp đơn có thể được sao chép lên bất kỳ máy nào và được khởi chạy ngay lập tức
để tham gia vào một cụm các Doozer đang chạy.

Cuối cùng, sự tập trung điên cuồng của Go vào sự đơn giản và tính trực giao phù hợp với
quan điểm của chúng tôi về kỹ thuật phần mềm.
Giống như nhóm Go, chúng tôi thực dụng về những tính năng nào đưa vào Doozer.
Chúng tôi chú ý đến từng chi tiết, thích thay đổi tính năng hiện có thay vì
giới thiệu tính năng mới.
Theo nghĩa này, Go là sự kết hợp hoàn hảo cho Doozer.

Chúng tôi đã có các dự án tương lai trong đầu cho Go. Doozer chỉ là sự khởi đầu của một hệ thống lớn hơn nhiều.
