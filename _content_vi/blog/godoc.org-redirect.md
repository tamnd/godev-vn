---
title: Chuyển hướng yêu cầu godoc.org sang pkg.go.dev
date: 2020-12-15
by:
- Julie Qiu
summary: Kế hoạch chuyển từ godoc.org sang pkg.go.dev.
---


Với sự ra đời của module Go và sự phát triển của hệ sinh thái Go,
[pkg.go.dev](https://pkg.go.dev) đã được
[ra mắt vào năm 2019](/blog/go.dev) để cung cấp một nơi tập trung
mà các nhà phát triển có thể khám phá và đánh giá các gói và module Go. Giống như
godoc.org, pkg.go.dev phục vụ tài liệu Go, nhưng nó cũng hỗ trợ module,
chức năng tìm kiếm tốt hơn, và các tín hiệu giúp người dùng Go tìm đúng
gói.

Như [chúng tôi đã chia sẻ vào tháng 1 năm 2020](/blog/pkg.go.dev-2020), mục tiêu của chúng tôi
là cuối cùng chuyển hướng lưu lượng truy cập từ godoc.org sang trang tương ứng
trên pkg.go.dev. Chúng tôi cũng đã cho phép người dùng tự chọn chuyển hướng
các yêu cầu của họ từ godoc.org sang pkg.go.dev.

Chúng tôi đã nhận được nhiều phản hồi tuyệt vời trong năm nay, đã được theo dõi và
giải quyết thông qua các mốc
[pkgsite/godoc.org-redirect](https://github.com/golang/go/milestone/157?closed=1)
và [pkgsite/design-2020](https://github.com/golang/go/milestone/159?closed=1)
trên trình theo dõi issue Go. Phản hồi của bạn đã dẫn đến hỗ trợ cho
các yêu cầu tính năng phổ biến trên pkg.go.dev,
[mã nguồn mở pkgsite](/blog/pkgsite), và gần đây nhất là
[thiết kế lại pkg.go.dev](/blog/pkgsite-redesign).

## Các bước tiếp theo

Bước tiếp theo trong quá trình di chuyển này là chuyển hướng tất cả yêu cầu từ godoc.org sang
trang tương ứng trên pkg.go.dev.

Điều này sẽ xảy ra vào đầu năm 2021, khi công việc được theo dõi tại
[mốc pkgsite/godoc.org-redirect](https://github.com/golang/go/milestone/157)
hoàn thành.

Trong quá trình di chuyển này, các cập nhật sẽ được đăng lên
[Go issue 43178](/issue/43178).

Chúng tôi khuyến khích mọi người bắt đầu dùng pkg.go.dev ngay hôm nay. Bạn có thể làm vậy bằng cách
truy cập [godoc.org?redirect=on](https://godoc.org?redirect=on), hoặc click
"Always use pkg.go.dev" ở góc trên bên phải của bất kỳ trang godoc.org nào.

## FAQ

**Các URL godoc.org có tiếp tục hoạt động không?**

Có! Chúng tôi sẽ chuyển hướng tất cả yêu cầu đến godoc.org sang trang tương đương
trên pkg.go.dev, vì vậy tất cả bookmarks và link của bạn sẽ tiếp tục đưa bạn đến
tài liệu bạn cần.

**Điều gì sẽ xảy ra với kho lưu trữ golang/gddo?**

[Kho lưu trữ gddo](http://go.googlesource.com/gddo) sẽ vẫn có sẵn
cho bất kỳ ai muốn tiếp tục tự chạy nó, hoặc thậm chí fork và cải thiện nó.
Chúng tôi sẽ đánh dấu nó là archived để làm rõ rằng chúng tôi sẽ không còn chấp nhận
đóng góp nữa. Tuy nhiên, bạn sẽ có thể tiếp tục fork kho lưu trữ.

**api.godoc.org có tiếp tục hoạt động không?**

Quá trình chuyển đổi này sẽ không có tác động đến api.godoc.org. Cho đến khi có API sẵn có
cho pkg.go.dev, api.godoc.org sẽ tiếp tục phục vụ lưu lượng truy cập. Xem
[Go issue 36785](/issue/36785) để biết cập nhật về API cho
pkg.go.dev.

**Huy hiệu godoc.org của tôi có tiếp tục hoạt động không?**

Có! URL huy hiệu cũng sẽ chuyển hướng sang URL tương đương trên pkg.go.dev.
Trang của bạn sẽ tự động nhận huy hiệu pkg.go.dev mới. Bạn cũng có thể tạo huy hiệu mới
tại [pkg.go.dev/badge](https://pkg.go.dev/badge) nếu bạn muốn cập nhật
link huy hiệu của mình.

## Phản hồi

Như thường lệ, hãy thoải mái [tạo issue](/s/pkgsite-feedback)
trên trình theo dõi issue Go cho bất kỳ phản hồi nào.

## Đóng góp

Pkg.go.dev là một [dự án mã nguồn mở](https://go.googlesource.com/pkgsite). Nếu
bạn quan tâm đến việc đóng góp cho dự án pkgsite, hãy xem
[hướng dẫn đóng góp](https://go.googlesource.com/pkgsite/+/refs/heads/master/CONTRIBUTING.md)
hoặc tham gia
[kênh #pkgsite](https://gophers.slack.com/messages/pkgsite) trên Gophers Slack
để tìm hiểu thêm.
