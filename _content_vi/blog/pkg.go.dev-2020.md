---
title: Các bước tiếp theo cho pkg.go.dev
date: 2020-01-31
by:
- Julie Qiu
summary: Những gì nhóm Go đang lên kế hoạch cho pkg.go.dev trong năm 2020.
---

## Giới thiệu

Năm 2019, chúng tôi đã ra mắt [go.dev](/), một trung tâm mới dành cho các lập trình viên Go.

Là một phần của trang web, chúng tôi cũng ra mắt [pkg.go.dev](https://pkg.go.dev), một nguồn thông tin tập trung về các package và module Go. Giống như [godoc.org](https://godoc.org), pkg.go.dev phục vụ tài liệu Go. Tuy nhiên, nó còn hiểu về các module và có thông tin về các phiên bản trước của một package!

Trong suốt năm nay, chúng tôi sẽ bổ sung các tính năng cho [pkg.go.dev](https://pkg.go.dev) để giúp người dùng hiểu rõ hơn về các dependency của họ và đưa ra quyết định tốt hơn về những thư viện cần import.

## Chuyển hướng các yêu cầu từ godoc.org sang pkg.go.dev

Để giảm thiểu sự nhầm lẫn về việc sử dụng trang web nào, cuối năm nay chúng tôi có kế hoạch chuyển hướng lưu lượng truy cập từ [godoc.org](https://godoc.org) sang trang tương ứng trên [pkg.go.dev](https://pkg.go.dev). Chúng tôi cần sự giúp đỡ của bạn để đảm bảo rằng pkg.go.dev đáp ứng được tất cả nhu cầu của người dùng. Chúng tôi khuyến khích mọi người bắt đầu sử dụng pkg.go.dev ngay hôm nay cho tất cả các nhu cầu và cung cấp phản hồi.

Phản hồi của bạn sẽ định hướng kế hoạch chuyển đổi của chúng tôi, với mục tiêu đưa [pkg.go.dev](https://pkg.go.dev) trở thành nguồn thông tin và tài liệu chính về các package và module. Chúng tôi chắc chắn rằng có những điều bạn muốn thấy trên pkg.go.dev, và chúng tôi muốn nghe từ bạn về những tính năng đó!

Bạn có thể chia sẻ phản hồi với chúng tôi qua các kênh sau:

  - Đăng lên [Go issue tracker](/s/discovery-feedback).
  - Gửi email tới [go-discovery-feedback@google.com](mailto:go-discovery-feedback@google.com).
  - Nhấp vào "Share Feedback" hoặc "Report an Issue" ở chân trang go.dev.

Như một phần của quá trình chuyển đổi này, chúng tôi cũng sẽ thảo luận về các kế hoạch truy cập API cho [pkg.go.dev](https://pkg.go.dev). Chúng tôi sẽ đăng cập nhật trên [Go issue 33654](/s/discovery-updates).

## Các câu hỏi thường gặp

Kể từ khi ra mắt vào tháng 11, chúng tôi đã nhận được rất nhiều phản hồi hữu ích về [pkg.go.dev](https://pkg.go.dev) từ người dùng Go. Trong phần còn lại của bài đăng này, chúng tôi muốn giải đáp một số câu hỏi thường gặp.

### Package của tôi không hiển thị trên pkg.go.dev! Làm thế nào để thêm nó?

Chúng tôi thường xuyên theo dõi [Go Module Index](https://index.golang.org/index) để tìm các package mới cần thêm vào [pkg.go.dev](https://pkg.go.dev). Nếu bạn không thấy một package trên pkg.go.dev, bạn có thể thêm nó bằng cách tải phiên bản module từ [proxy.golang.org](https://proxy.golang.org). Xem [go.dev/about](/about) để biết hướng dẫn.

### Package của tôi có hạn chế về giấy phép. Vấn đề là gì?

Chúng tôi hiểu rằng việc không thể xem toàn bộ package mà bạn muốn trên [pkg.go.dev](https://pkg.go.dev) là một trải nghiệm khó chịu. Chúng tôi đánh giá cao sự kiên nhẫn của bạn khi chúng tôi cải thiện thuật toán phát hiện giấy phép.

Kể từ khi ra mắt vào tháng 11, chúng tôi đã thực hiện các cải tiến sau:

  - Cập nhật [chính sách giấy phép](https://pkg.go.dev/license-policy) để bao gồm danh sách các giấy phép mà chúng tôi phát hiện và công nhận
  - Hợp tác với nhóm [licensecheck](https://github.com/google/licensecheck) để cải thiện việc phát hiện thông báo bản quyền
  - Thiết lập quy trình xem xét thủ công cho các trường hợp đặc biệt

Như thường lệ, chính sách giấy phép của chúng tôi có tại [pkg.go.dev/license-policy](https://pkg.go.dev/license-policy). Nếu bạn gặp sự cố, hãy [gửi một issue lên Go issue tracker](/s/discovery-feedback) hoặc gửi email tới [go-discovery-feedback@google.com](mailto:go-discovery-feedback@google.com) để chúng tôi có thể làm việc trực tiếp với bạn!

### pkg.go.dev có được mã nguồn mở để tôi có thể chạy nó tại nơi làm việc cho code riêng tư không?

Chúng tôi hiểu rằng các công ty có code riêng tư muốn chạy một máy chủ tài liệu hỗ trợ module. Chúng tôi muốn giúp đáp ứng nhu cầu đó, nhưng chúng tôi cảm thấy chưa hiểu rõ nhu cầu đó một cách đầy đủ.

Chúng tôi đã nghe từ người dùng rằng việc chạy máy chủ [godoc.org](https://godoc.org) phức tạp hơn mức cần thiết, vì nó được thiết kế để phục vụ ở quy mô internet công cộng thay vì chỉ trong nội bộ một công ty. Chúng tôi tin rằng máy chủ [pkg.go.dev](https://pkg.go.dev) hiện tại cũng sẽ gặp vấn đề tương tự.

Chúng tôi nghĩ rằng một máy chủ mới có nhiều khả năng là câu trả lời đúng cho việc sử dụng với code riêng tư, thay vì đặt mọi công ty vào tình huống phải đối mặt với sự phức tạp khi chạy codebase [pkg.go.dev](https://pkg.go.dev) ở quy mô internet. Ngoài việc phục vụ tài liệu, một máy chủ mới cũng có thể cung cấp thông tin cho [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports?tab=doc) và [gopls](https://pkg.go.dev/golang.org/x/tools/gopls).

Nếu bạn muốn chạy một máy chủ như vậy, hãy điền vào [**khảo sát 3-5 phút**](https://google.qualtrics.com/jfe/form/SV_6FHmaLveae6d8Bn) này để giúp chúng tôi hiểu rõ hơn nhu cầu của bạn. Khảo sát này sẽ mở đến ngày 1 tháng 3 năm 2020.

Chúng tôi rất hứng khởi về tương lai của [pkg.go.dev](https://pkg.go.dev) trong năm 2020, và hy vọng bạn cũng vậy! Chúng tôi mong được nghe phản hồi của bạn và làm việc cùng cộng đồng Go trong quá trình chuyển đổi này.
