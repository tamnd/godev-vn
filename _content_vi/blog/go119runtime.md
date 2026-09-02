---
title: "Go runtime: 4 năm nhìn lại"
date: 2022-09-26
by:
- Michael Knyszek
summary: Cập nhật về tình trạng phát triển của Go runtime
---

Kể từ [bài đăng blog về Go GC năm 2018](/blog/ismmkeynote),
Go GC và Go runtime nói chung đã liên tục cải thiện.
Chúng tôi đã giải quyết một số dự án lớn, được thúc đẩy bởi các chương trình Go
thực tế và những thách thức thực sự mà người dùng Go phải đối mặt.
Hãy để chúng tôi cập nhật cho bạn những điểm nổi bật!

### Có gì mới?

- `sync.Pool`, một công cụ nhận biết GC để tái sử dụng bộ nhớ, có [tác động độ trễ
  thấp hơn](/cl/166960) và [tái chế bộ nhớ hiệu quả hơn nhiều](/cl/166961) so với
  trước.
  (Go 1.13)

- Go runtime trả lại bộ nhớ không cần thiết về hệ điều hành [chủ động hơn nhiều](/issue/30333),
  giảm mức tiêu thụ bộ nhớ dư thừa và nguy cơ lỗi hết bộ nhớ.
  Điều này giảm mức tiêu thụ bộ nhớ khi không tải lên đến 20%.
  (Go 1.13 và 1.14)

- Go runtime có thể preempt goroutine dễ dàng hơn trong nhiều trường hợp,
  giảm độ trễ stop-the-world lên đến 90%.
  [Xem bài nói chuyện từ Gophercon 2020 tại đây.](https://www.youtube.com/watch?v=1I1WmeSjRSw)
  (Go 1.14)

- Go runtime [quản lý timer hiệu quả hơn trước](/cl/171883), đặc biệt trên các
  máy có nhiều lõi CPU.
  (Go 1.14)

- Các lời gọi hàm đã được defer với câu lệnh `defer` giờ tốn kém ít như một lời
  gọi hàm thông thường trong hầu hết các trường hợp.
  [Xem bài nói chuyện từ Gophercon 2020 tại đây.](https://www.youtube.com/watch?v=DHVeUsrKcbM)
  (Go 1.14)

- Đường chậm của bộ cấp phát bộ nhớ [mở rộng](/issue/35112) [tốt hơn](/issue/37487)
  với các lõi CPU, tăng thông lượng lên đến 10% và giảm độ trễ đuôi lên đến 30%,
  đặc biệt trong các chương trình song song cao.
  (Go 1.14 và 1.15)

- Thống kê bộ nhớ Go giờ có thể truy cập trong API chi tiết hơn, linh hoạt hơn và
  hiệu quả hơn, gói [runtime/metrics](https://pkg.go.dev/runtime/metrics).
  Điều này giảm độ trễ khi lấy thống kê runtime hai bậc độ lớn (mili giây xuống
  micro giây).
  (Go 1.16)

- Bộ lập lịch Go dành ít hơn [30% thời gian CPU để quay tìm công việc mới](/issue/43997).
  (Go 1.17)

- Code Go giờ tuân theo [quy ước gọi dựa trên thanh ghi](/issues/40724) trên amd64,
  arm64 và ppc64, cải thiện hiệu quả CPU lên đến 15%.
  (Go 1.17 và Go 1.18)

- Kế toán nội bộ và lập lịch của Go GC đã được [thiết kế lại](/issue/44167), giải
  quyết nhiều vấn đề lâu dài liên quan đến hiệu quả và độ bền.
  Điều này dẫn đến giảm đáng kể độ trễ đuôi của ứng dụng (lên đến 66%) cho các
  ứng dụng mà stack goroutine chiếm phần đáng kể của bộ nhớ sử dụng.
  (Go 1.18)

- Go GC giờ giới hạn [việc sử dụng CPU của chính nó khi ứng dụng nhàn rỗi](/issue/44163).
  Điều này giảm 75% mức sử dụng CPU trong một chu kỳ GC trong các ứng dụng rất
  nhàn rỗi, giảm các đột biến CPU có thể gây nhầm lẫn cho các job shaper.
  (Go 1.19)

Những thay đổi này hầu như vô hình với người dùng: code Go họ đã biết và yêu thích
chạy tốt hơn, chỉ bằng cách nâng cấp Go.

### Một tham số mới

Với Go 1.19 có một tính năng đã được yêu cầu từ lâu đòi hỏi thêm một chút công sức
để sử dụng, nhưng có nhiều tiềm năng: [giới hạn bộ nhớ mềm của Go runtime](https://pkg.go.dev/runtime/debug#SetMemoryLimit).

Trong nhiều năm, Go GC chỉ có một tham số điều chỉnh: `GOGC`.
`GOGC` cho phép người dùng điều chỉnh [sự đánh đổi giữa chi phí CPU và chi phí bộ
nhớ do Go GC thực hiện](https://pkg.go.dev/runtime/debug#SetGCPercent).
Trong nhiều năm, "tham số" này đã phục vụ cộng đồng Go tốt, bao trùm nhiều trường
hợp sử dụng đa dạng.

Nhóm Go runtime đã do dự thêm các tham số mới vào Go runtime, với lý do chính đáng:
mỗi tham số mới đại diện cho một _chiều_ mới trong không gian cấu hình mà chúng tôi
cần kiểm thử và duy trì, có thể là mãi mãi. Sự phổ biến của các tham số cũng đặt
gánh nặng lên các nhà phát triển Go để hiểu và sử dụng chúng hiệu quả, điều này
trở nên khó khăn hơn với nhiều tham số hơn. Do đó, Go runtime luôn hướng đến hoạt
động hợp lý với cấu hình tối thiểu.

Vậy tại sao lại thêm tham số giới hạn bộ nhớ?

Bộ nhớ không thể hoán đổi như thời gian CPU.
Với thời gian CPU, luôn có nhiều hơn trong tương lai, nếu bạn chờ một chút.
Nhưng với bộ nhớ, có giới hạn về những gì bạn có.

Giới hạn bộ nhớ giải quyết hai vấn đề.

Vấn đề đầu tiên là khi mức sử dụng bộ nhớ đỉnh của ứng dụng không thể dự đoán,
`GOGC` một mình hầu như không cung cấp bảo vệ khỏi việc hết bộ nhớ.
Chỉ với `GOGC`, Go runtime đơn giản là không biết có bao nhiêu bộ nhớ có sẵn cho nó.
Đặt giới hạn bộ nhớ cho phép runtime mạnh mẽ hơn trước các đột biến tải tạm thời
và có thể khôi phục bằng cách làm cho nó biết khi nào cần làm việc chăm chỉ hơn để
giảm chi phí bộ nhớ.

Vấn đề thứ hai là để tránh lỗi hết bộ nhớ mà không dùng giới hạn bộ nhớ, `GOGC`
phải được điều chỉnh theo bộ nhớ đỉnh, dẫn đến chi phí CPU GC cao hơn để duy trì
chi phí bộ nhớ thấp, ngay cả khi ứng dụng không ở mức sử dụng bộ nhớ đỉnh và có
đủ bộ nhớ. Điều này đặc biệt liên quan trong thế giới container hóa của chúng ta,
nơi các chương trình được đặt trong các hộp với bộ nhớ dự trữ cụ thể và cô lập; chúng
ta cũng nên tận dụng chúng! Bằng cách cung cấp bảo vệ khỏi các đột biến tải, đặt
giới hạn bộ nhớ cho phép `GOGC` được điều chỉnh tích cực hơn nhiều liên quan đến
chi phí CPU.

Giới hạn bộ nhớ được thiết kế để dễ áp dụng và mạnh mẽ.
Ví dụ, đó là giới hạn trên toàn bộ dấu ấn bộ nhớ của các phần Go của ứng dụng,
không chỉ heap Go, vì vậy người dùng không phải lo lắng về việc tính toán chi phí
Go runtime.
Runtime cũng điều chỉnh chính sách scavenging bộ nhớ của nó để phản hồi với giới
hạn bộ nhớ, vì vậy nó trả lại bộ nhớ về OS chủ động hơn để phản hồi với áp lực
bộ nhớ.

Nhưng trong khi giới hạn bộ nhớ là một công cụ mạnh mẽ, nó vẫn phải được sử dụng
với một số cẩn thận.
Một cảnh báo lớn là nó mở chương trình của bạn cho GC thrashing: một trạng thái trong
đó chương trình dành quá nhiều thời gian chạy GC, dẫn đến không đủ thời gian để
tiến triển có ý nghĩa.
Ví dụ, chương trình Go có thể thrash nếu giới hạn bộ nhớ được đặt quá thấp so với
lượng bộ nhớ mà chương trình thực sự cần.
GC thrashing trước đây không có khả năng xảy ra, trừ khi `GOGC` được điều chỉnh
nặng nề rõ ràng để ưu tiên sử dụng bộ nhớ.
Chúng tôi chọn ưu tiên hết bộ nhớ thay vì thrashing, vì vậy như một biện pháp giảm
thiểu, runtime sẽ giới hạn GC ở 50% tổng thời gian CPU, ngay cả khi điều này có
nghĩa là vượt quá giới hạn bộ nhớ.

Tất cả điều này cần nhiều suy nghĩ, vì vậy như một phần của công việc này, chúng tôi
đã phát hành [hướng dẫn GC mới bóng bẩy](/doc/gc-guide), hoàn chỉnh với các hình
ảnh trực quan tương tác để giúp bạn hiểu chi phí GC và cách điều chỉnh chúng.

### Kết luận

Hãy thử giới hạn bộ nhớ!
Dùng nó trong production!
Đọc [hướng dẫn GC](/doc/gc-guide)!

Chúng tôi luôn tìm kiếm phản hồi về cách cải thiện Go, nhưng cũng rất hữu ích khi
nghe về khi nó hoạt động tốt với bạn.
[Gửi cho chúng tôi phản hồi](https://groups.google.com/g/golang-dev)!
