---
title: Thực hành bảo mật tốt nhất cho lập trình viên Go
layout: article
---

[Quay lại Bảo mật Go](/security)

Trang này cung cấp cho lập trình viên Go các thực hành tốt nhất để ưu tiên bảo mật cho dự án của họ. Từ việc tự động hóa kiểm thử bằng fuzzing đến kiểm tra điều kiện tranh chấp một cách dễ dàng, các mẹo này có thể giúp codebase của bạn an toàn và đáng tin cậy hơn.

## Quét mã nguồn và tệp nhị phân để tìm lỗ hổng bảo mật

Thường xuyên quét mã nguồn và tệp nhị phân để tìm lỗ hổng bảo mật giúp xác định các rủi ro bảo mật tiềm ẩn sớm.
Bạn có thể sử dụng [govulncheck](https://pkg.go.dev/golang.org/x/vuln/cmd/govulncheck),
được hỗ trợ bởi [cơ sở dữ liệu lỗ hổng bảo mật Go](https://pkg.go.dev),
để quét mã của bạn tìm lỗ hổng bảo mật và phân tích những lỗ hổng nào thực sự ảnh hưởng đến bạn.
Bắt đầu với [hướng dẫn govulncheck](/doc/tutorial/govulncheck).

Govulncheck cũng có thể được tích hợp vào quy trình CI/CD.
Nhóm Go cung cấp
[GitHub Action cho govulncheck](https://github.com/marketplace/actions/golang-govulncheck-action)
trên GitHub Marketplace.
Govulncheck cũng hỗ trợ cờ `-json` để giúp lập trình viên tích hợp quét lỗ hổng bảo mật
với các hệ thống CI/CD khác.

Bạn cũng có thể quét lỗ hổng bảo mật trực tiếp trong trình soạn thảo mã nguồn bằng cách sử dụng
[tiện ích mở rộng Go cho Visual Studio Code](/security/vuln/editor).
Bắt đầu với [hướng dẫn này](/doc/tutorial/govulncheck-ide).

## Giữ phiên bản Go và các dependency luôn cập nhật

Giữ [phiên bản Go luôn cập nhật](/doc/install) giúp
truy cập các tính năng ngôn ngữ mới nhất,
cải thiện hiệu suất và vá các lỗ hổng bảo mật đã biết.
Phiên bản Go được cập nhật cũng đảm bảo khả năng tương thích với các phiên bản mới hơn của dependency,
giúp tránh các vấn đề tích hợp tiềm ẩn.
Xem lại [lịch sử bản phát hành Go](/doc/devel/release) để xem
những thay đổi nào đã được thực hiện giữa các bản phát hành.
Nhóm Go phát hành các bản vá nhỏ trong suốt chu kỳ phát hành để giải quyết các lỗi bảo mật.
Hãy chắc chắn cập nhật lên phiên bản Go nhỏ mới nhất để đảm bảo bạn có
các bản vá bảo mật mới nhất.

Duy trì các dependency bên thứ ba luôn cập nhật cũng rất quan trọng cho bảo mật phần mềm,
hiệu suất và tuân thủ các tiêu chuẩn mới nhất trong hệ sinh thái Go.
Tuy nhiên, việc cập nhật lên các phiên bản mới nhất mà không xem xét kỹ lưỡng
[cũng có thể rủi ro](https://research.swtch.com/npm-colors),
có khả năng đưa vào các lỗi mới, thay đổi không tương thích,
hoặc thậm chí mã độc hại.
Do đó, mặc dù việc cập nhật dependency cho các bản vá bảo mật và cải tiến mới nhất là cần thiết,
mỗi lần cập nhật nên được xem xét và kiểm thử cẩn thận.

## Kiểm thử bằng fuzzing để phát hiện các kịch bản khai thác ở trường hợp biên

[Fuzzing](/security/fuzz) là một loại kiểm thử tự động
sử dụng hướng dẫn độ phủ để thao túng các đầu vào ngẫu nhiên và duyệt qua mã nguồn
nhằm tìm và báo cáo các lỗ hổng bảo mật tiềm ẩn như SQL injection,
tràn bộ đệm, từ chối dịch vụ và các cuộc tấn công cross-site scripting.
Fuzzing thường có thể tiếp cận các trường hợp biên mà lập trình viên bỏ qua,
hoặc cho là quá khó xảy ra để kiểm thử.
Bắt đầu với [hướng dẫn này](/doc/tutorial/fuzz).

## Kiểm tra điều kiện tranh chấp bằng bộ phát hiện race của Go

Điều kiện tranh chấp xảy ra khi hai hoặc nhiều [goroutine](/tour/concurrency/1)
truy cập cùng một tài nguyên đồng thời,
và ít nhất một trong số các lần truy cập đó là ghi.
Điều này có thể dẫn đến các vấn đề không thể đoán trước và khó chẩn đoán trong phần mềm của bạn.
Xác định các điều kiện tranh chấp tiềm ẩn trong mã Go của bạn bằng
[bộ phát hiện race](/doc/articles/race_detector) tích hợp sẵn,
có thể giúp bạn đảm bảo tính an toàn và độ tin cậy của các chương trình đồng thời.
Bộ phát hiện race tìm các race xảy ra tại thời điểm chạy,
tuy nhiên, nó sẽ không tìm các race trong các đường dẫn mã không được thực thi.

Để sử dụng bộ phát hiện race, thêm cờ `-race` khi chạy kiểm thử hoặc
xây dựng ứng dụng của bạn,
ví dụ, `go test -race`.
Điều này sẽ biên dịch mã của bạn với bộ phát hiện race được bật và báo cáo bất kỳ
điều kiện tranh chấp nào được phát hiện lúc chạy.
Khi bộ phát hiện race tìm thấy data race trong chương trình, nó sẽ
[in một báo cáo](/doc/articles/race_detector#report-format)
chứa stack trace cho các lần truy cập xung đột,
và các stack nơi các goroutine liên quan được tạo ra.

## Sử dụng Vet để kiểm tra các cấu trúc đáng ngờ

[Lệnh vet](https://pkg.go.dev/cmd/vet) của Go được thiết kế để phân tích
mã nguồn của bạn và gắn cờ các vấn đề tiềm ẩn có thể không nhất thiết là lỗi cú pháp,
nhưng có thể dẫn đến sự cố trong thời gian chạy.
Bao gồm các cấu trúc đáng ngờ, chẳng hạn như mã không thể tiếp cận,
biến không sử dụng và các lỗi phổ biến xung quanh goroutine.
Bằng cách phát hiện sớm các vấn đề này trong quá trình phát triển,
go vet giúp duy trì chất lượng mã, giảm thời gian debug,
và nâng cao độ tin cậy tổng thể của phần mềm.
Để chạy go vet cho một dự án cụ thể, hãy chạy:

```
go vet ./...
```

## Đăng ký golang-announce để nhận thông báo về các bản phát hành bảo mật

Các bản phát hành Go chứa bản vá bảo mật được thông báo trước đến danh sách gửi thư ít hoạt động
[golang-announce@googlegroups.com](https://groups.google.com/group/golang-announce).
Nếu bạn muốn biết khi nào các bản vá bảo mật cho Go đang trên đường ra, hãy đăng ký.
