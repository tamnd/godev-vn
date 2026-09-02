---
title: "Go tại I/O: Các câu hỏi thường gặp"
date: 2010-05-27
by:
- Andrew Gerrand
tags:
- appengine
summary: Hỏi đáp về Go từ Google I/O 2010.
---


Trong số những sản phẩm nổi bật được ra mắt tại Google I/O tuần trước,
nhóm nhỏ của chúng tôi đã trình bày trước những hội trường chật kín người và gặp gỡ nhiều lập trình viên Go, cả hiện tại lẫn tương lai.
Điều đặc biệt đáng mừng là được gặp gỡ nhiều người, sau khi tìm hiểu sơ qua về Go, đã hào hứng với những lợi ích tiềm năng (cả trước mắt lẫn lâu dài) mà họ có thể nhận được khi sử dụng Go.

Chúng tôi đã nhận được rất nhiều câu hỏi hay trong suốt I/O, và trong bài viết này tôi muốn tóm tắt và mở rộng một số câu hỏi đó.

Go phù hợp đến mức nào cho các hệ thống production?
Go đã sẵn sàng và ổn định. Chúng tôi vui mừng thông báo rằng Google đang sử dụng
Go cho một số hệ thống production,
và chúng hoạt động tốt.
Tất nhiên vẫn còn nhiều điều cần cải thiện, đó là lý do chúng tôi tiếp tục
phát triển ngôn ngữ,
thư viện, công cụ và runtime.

Bạn có kế hoạch triển khai generics không?
Nhiều đề xuất về các tính năng tương tự generics đã được thảo luận công khai và nội bộ,
nhưng cho đến nay chúng tôi vẫn chưa tìm được đề xuất nào nhất quán với phần còn lại của ngôn ngữ.
Chúng tôi cho rằng một trong những điểm mạnh then chốt của Go là sự đơn giản,
vì vậy chúng tôi thận trọng khi đưa vào các tính năng mới có thể khiến ngôn ngữ
trở nên khó hiểu hơn.
Ngoài ra, chúng tôi càng viết nhiều code Go (và do đó càng học được cách viết code Go tốt hơn),
chúng tôi càng ít cảm thấy cần đến tính năng ngôn ngữ như vậy.

Bạn có kế hoạch hỗ trợ lập trình GPU không?
Chúng tôi chưa có kế hoạch ngay lập tức cho việc này,
nhưng vì Go không phụ thuộc vào kiến trúc nên hoàn toàn có thể thực hiện.
Khả năng khởi chạy một goroutine chạy trên kiến trúc bộ xử lý khác,
và sử dụng channel để giao tiếp giữa các goroutine chạy trên các kiến trúc riêng biệt,
có vẻ là những ý tưởng hay.

Có kế hoạch hỗ trợ Go trên App Engine không?
Cả nhóm Go lẫn nhóm App Engine đều muốn điều này xảy ra.
Như thường lệ, đây là vấn đề về nguồn lực và thứ tự ưu tiên để xác định nếu và khi nào
nó trở thành hiện thực.

Có kế hoạch hỗ trợ Go trên Android không?
Cả hai trình biên dịch Go đều hỗ trợ tạo mã ARM, vì vậy điều này hoàn toàn khả thi.
Dù chúng tôi nghĩ Go sẽ là ngôn ngữ tuyệt vời để viết ứng dụng di động,
hỗ trợ Android không phải là thứ đang được tích cực phát triển.

Tôi có thể dùng Go cho việc gì?
Go được thiết kế với lập trình hệ thống trong tâm trí.
Máy chủ, clients, cơ sở dữ liệu, cache, bộ cân bằng tải,
bộ phân phối, đây là những ứng dụng mà Go rõ ràng hữu ích,
và đây là cách chúng tôi đã bắt đầu sử dụng nó trong Google.
Tuy nhiên, từ khi Go được phát hành mã nguồn mở, cộng đồng đã tìm thấy nhiều ứng dụng
đa dạng cho ngôn ngữ này.
Từ ứng dụng web đến game đến các công cụ đồ họa,
Go hứa hẹn sẽ tỏa sáng như một ngôn ngữ lập trình đa năng.
Tiềm năng chỉ bị giới hạn bởi sự hỗ trợ thư viện,
và điều này đang cải thiện với tốc độ đáng kinh ngạc.
Ngoài ra, các nhà giáo dục cũng bày tỏ quan tâm đến việc sử dụng Go để dạy lập trình,
cho rằng cú pháp ngắn gọn và tính nhất quán của nó phù hợp cho nhiệm vụ này.

Cảm ơn tất cả những ai đã tham dự các bài trình bày của chúng tôi,
hoặc đến nói chuyện với chúng tôi tại Office Hours.
Chúng tôi hy vọng sẽ gặp lại bạn tại các sự kiện trong tương lai.

Video bài nói chuyện của Rob và Russ [có thể xem trên YouTube](https://youtu.be/jgVhBThJdXc).
