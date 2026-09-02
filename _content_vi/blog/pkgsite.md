---
title: Pkg.go.dev đã là mã nguồn mở!
date: 2020-06-15
by:
- Julie Qiu
---


Chúng tôi vui mừng thông báo rằng codebase của [pkg.go.dev](https://pkg.go.dev) hiện đã là mã nguồn mở.

Kho lưu trữ nằm tại [go.googlesource.com/pkgsite](https://go.googlesource.com/pkgsite) và được sao chép sang [github.com/golang/pkgsite](https://github.com/golang/pkgsite).
Chúng tôi sẽ tiếp tục sử dụng Go issue tracker để theo dõi [phản hồi](https://github.com/golang/go/labels/go.dev) liên quan đến pkg.go.dev.

## Đóng góp

Nếu bạn quan tâm đến việc đóng góp cho bất kỳ [issue nào liên quan đến pkg.go.dev](https://github.com/golang/go/labels/go.dev), hãy xem [hướng dẫn đóng góp](https://go.googlesource.com/pkgsite/+/refs/heads/master/CONTRIBUTING.md) của chúng tôi.
Chúng tôi cũng khuyến khích bạn tiếp tục [gửi issue](/s/discovery-feedback) nếu bạn gặp sự cố hoặc có phản hồi.

## Tiếp theo là gì

Chúng tôi thực sự đánh giá cao tất cả phản hồi mà chúng tôi đã nhận được cho đến nay. Đó là sự giúp đỡ to lớn trong việc định hình [lộ trình](https://go.googlesource.com/pkgsite#roadmap) của chúng tôi cho năm tới.
Bây giờ khi pkg.go.dev đã là mã nguồn mở, đây là những gì chúng tôi sẽ làm tiếp theo:

- Chúng tôi có một số thay đổi thiết kế được lên kế hoạch cho pkg.go.dev, để giải quyết [phản hồi về UX](https://github.com/golang/go/issues?q=is%3Aissue+is%3Aopen+label%3Ago.dev+label%3AUX) mà chúng tôi đã nhận được. Bạn có thể mong đợi trải nghiệm tìm kiếm và điều hướng gắn kết hơn. Chúng tôi có kế hoạch chia sẻ các thiết kế này để lấy phản hồi khi chúng đã sẵn sàng.

- Chúng tôi biết rằng có những tính năng có sẵn trên godoc.org mà người dùng muốn thấy trên pkg.go.dev. Chúng tôi đã theo dõi chúng trên [Go issue #39144](/issue/39144), và sẽ ưu tiên thêm chúng trong vài tháng tới. Chúng tôi cũng có kế hoạch tiếp tục cải thiện thuật toán phát hiện giấy phép dựa trên phản hồi.

- Chúng tôi sẽ cải thiện trải nghiệm tìm kiếm dựa trên phản hồi trong [Go issue #37810](/issue/37810), để giúp người dùng dễ dàng tìm thấy các dependency họ đang tìm kiếm và đưa ra quyết định tốt hơn về những dependency nào cần import.

Cảm ơn bạn đã kiên nhẫn với chúng tôi trong quá trình mở mã nguồn pkg.go.dev.
Chúng tôi mong đợi nhận được các đóng góp của bạn và làm việc cùng bạn về tương lai của dự án.
