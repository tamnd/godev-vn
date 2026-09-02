---
title: "Hai hướng dẫn mới cho Go 1.18"
date: 2022-01-14
by:
- Katie Hockman, for the Go team
summary: Hai hướng dẫn mới đã được xuất bản để chuẩn bị cho bản phát hành Go 1.18.
---

Go 1.18 sắp được phát hành, và bản phát hành này bao gồm một số khái niệm mới trong Go.
Chúng tôi đã xuất bản hai hướng dẫn mới để giúp bạn làm quen với những tính năng sắp ra mắt này.

Hướng dẫn mới đầu tiên sẽ [giúp bạn bắt đầu với generics](/doc/tutorial/generics).
Hướng dẫn này đưa bạn qua quá trình tạo một hàm generic có thể xử lý nhiều kiểu dữ liệu khác nhau,
và gọi nó từ mã của bạn. Sau khi tạo xong hàm generic, bạn sẽ tìm hiểu về ràng buộc kiểu (type constraints)
và viết một số ràng buộc cho hàm của mình. Bạn cũng có thể xem thêm
[bài nói chuyện tại GopherCon về generics](https://www.youtube.com/watch?v=35eIxI_n5ZM&t=1755s) để tìm hiểu thêm.

Hướng dẫn mới thứ hai sẽ [giúp bạn bắt đầu với fuzzing](/doc/tutorial/fuzz).
Hướng dẫn này trình bày cách fuzzing có thể phát hiện lỗi trong mã của bạn, và hướng dẫn quy trình
chẩn đoán cũng như sửa chữa các vấn đề đó. Trong hướng dẫn này, bạn sẽ viết mã có một vài lỗi
và dùng fuzzing để tìm, sửa và xác minh các lỗi đó bằng lệnh go. Xin cảm ơn đặc biệt Beth Brown
vì công sức của cô trong hướng dẫn về fuzzing!

Go 1.18 Beta 1 đã được phát hành tháng trước, bạn có thể tải về tại
[trang tải xuống](/dl/#go1.18beta1).

Xem đầy đủ [bản nháp ghi chú phát hành của Go 1.18](https://tip.golang.org/doc/go1.18)
để biết thêm chi tiết về những gì sẽ có trong bản phát hành.

Như thường lệ, nếu bạn phát hiện bất kỳ vấn đề nào, hãy [gửi báo cáo lỗi](/issue/new).

Chúng tôi hy vọng bạn thích các hướng dẫn này, và chờ đón mọi điều sắp đến trong năm 2022!
