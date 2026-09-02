---
title: Go 1.23 được phát hành
date: 2024-08-13
by:
- Dmitri Shuralyov, on behalf of the Go team
summary: Go 1.23 bổ sung iterator, tiếp tục các cải tiến về vòng lặp, cải thiện khả năng tương thích và nhiều hơn nữa.
---

Hôm nay đội ngũ Go vui mừng phát hành Go 1.23,
mà bạn có thể nhận được bằng cách truy cập [trang tải xuống](/dl/).

Nếu bạn đã cài đặt Go 1.22 hoặc Go 1.21 trên máy của mình,
bạn cũng có thể thử `go get toolchain@go1.23.0` trong một module hiện có.
Lệnh này sẽ tải xuống toolchain mới và cho phép bạn bắt đầu sử dụng nó
trong module của mình ngay lập tức. Vào một thời điểm nào đó sau này, bạn có thể tiếp tục
với `go get go@1.23.0` khi sẵn sàng chuyển hoàn toàn sang Go 1.23
và để phiên bản đó trở thành phiên bản Go tối thiểu mà module của bạn yêu cầu.
Xem [Quản lý yêu cầu phiên bản Go của module bằng go get](/doc/toolchain#get)
để biết thêm thông tin về chức năng này.

Go 1.23 đi kèm với nhiều cải tiến so với Go 1.22. Một số điểm nổi bật bao gồm:

## Các thay đổi về ngôn ngữ

-	<!-- go.dev/issue/61405, go.dev/issue/61897, go.dev/issue/61899, go.dev/issue/61900 -->
	Biểu thức phạm vi trong vòng lặp "for-range" giờ đây có thể là các hàm iterator,
	chẳng hạn như `func(func(K) bool)`.
	Điều này hỗ trợ các iterator do người dùng định nghĩa trên các chuỗi tùy ý.
	Có một số bổ sung cho các gói chuẩn `slices` và `maps`
	hoạt động với iterator, cũng như một gói `iter` mới.
	Ví dụ, nếu bạn muốn thu thập các khóa của một map `m` vào một slice
	và sau đó sắp xếp các giá trị của nó, bạn có thể thực hiện điều đó trong Go 1.23 với `slices.Sorted(maps.Keys(m))`.

	Go 1.23 cũng bao gồm hỗ trợ xem trước cho bí danh kiểu generic.

	Đọc thêm về [các thay đổi về ngôn ngữ](/doc/go1.23#language) và [iterator](/doc/go1.23#iterators)
	trong ghi chú bản phát hành.

## Các cải tiến về công cụ

-	<!-- go.dev/issue/58894 -->
	Bắt đầu từ Go 1.23, toolchain Go có thể thu thập thống kê về việc sử dụng và lỗi phá vỡ
	để giúp hiểu cách toolchain Go được sử dụng và mức độ hoạt động hiệu quả của nó.
	Đây là telemetry của Go, một _hệ thống opt-in_. Vui lòng cân nhắc tham gia để giúp chúng tôi duy trì Go
	hoạt động tốt và hiểu rõ hơn về cách sử dụng Go.
	Đọc thêm về [telemetry của Go](/doc/go1.23#telemetry) trong ghi chú bản phát hành.
-	Lệnh `go` có các tiện ích mới. Ví dụ, chạy `go env -changed` giúp dễ dàng hơn khi
	chỉ xem những cài đặt có giá trị hiệu dụng khác với giá trị mặc định, và
	`go mod tidy -diff` giúp xác định các thay đổi cần thiết đối với các tệp go.mod và go.sum
	mà không sửa đổi chúng.
	Đọc thêm về [lệnh Go](/doc/go1.23#go-command) trong ghi chú bản phát hành.
-	Tiểu lệnh `go vet` giờ đây báo cáo các ký hiệu quá mới so với phiên bản Go dự kiến.
	Đọc thêm về [các công cụ](/doc/go1.23#tools) trong ghi chú bản phát hành.

## Cải tiến thư viện chuẩn

-	Go 1.23 cải thiện việc triển khai `time.Timer` và `time.Ticker`.
	Đọc thêm về [các thay đổi về bộ hẹn giờ](/doc/go1.23#timer-changes) trong ghi chú bản phát hành.
-	Có tổng cộng 3 gói mới trong thư viện chuẩn Go 1.23: `iter`, `structs`, và `unique`.
	Gói `iter` được đề cập ở trên.
	Gói `structs` định nghĩa các kiểu đánh dấu để thay đổi các thuộc tính của một struct.
	Gói `unique` cung cấp các tiện ích để chuẩn hóa ("interning") các giá trị có thể so sánh.
	Đọc thêm về [các gói thư viện chuẩn mới](/doc/go1.23#new-unique-package)
	trong ghi chú bản phát hành.
-	Có nhiều cải tiến và bổ sung cho thư viện chuẩn được liệt kê trong phần
	[các thay đổi nhỏ đối với thư viện](/doc/go1.23#minor_library_changes)
	của ghi chú bản phát hành.
	Tài liệu “Go, Tương thích ngược, và GODEBUG”
	liệt kê [các thiết lập GODEBUG mới trong Go 1.23](/doc/godebug#go-123).
-	<!-- go.dev/issue/65573 -->
	Go 1.23 hỗ trợ chỉ thị `godebug` mới trong các tệp `go.mod` và `go.work` để
	cho phép kiểm soát riêng các GODEBUG mặc định và chỉ thị “go” của `go.mod`,
	bổ sung cho các chú thích chỉ thị `//go:debug` đã được cung cấp từ hai bản phát hành trước (Go 1.21).
	Xem tài liệu đã cập nhật về [Giá trị GODEBUG mặc định](/doc/godebug#default).

## Nhiều cải tiến và thay đổi hơn

-	Go 1.23 bổ sung hỗ trợ thử nghiệm cho OpenBSD trên RISC-V 64-bit (`openbsd/riscv64`).
	Có một số thay đổi nhỏ liên quan đến Linux, macOS, ARM64, RISC-V và WASI.
	Đọc thêm về [các cổng](/doc/go1.23#ports) trong ghi chú bản phát hành.
-	Thời gian xây dựng khi sử dụng tối ưu hóa dựa trên hồ sơ thực thi (PGO) được giảm xuống, và hiệu năng
	với PGO trên các kiến trúc 386 và amd64 được cải thiện.
	Đọc thêm về [runtime, trình biên dịch và trình liên kết](/doc/go1.23#runtime) trong ghi chú bản phát hành.

Chúng tôi khuyến khích mọi người đọc [ghi chú bản phát hành Go 1.23](/doc/go1.23) để biết
thông tin đầy đủ và chi tiết về những thay đổi này, cũng như mọi thứ khác đã
được bổ sung trong Go 1.23.

Trong vài tuần tới, hãy đón đọc các bài đăng blog tiếp theo sẽ đi sâu hơn
vào một số chủ đề được đề cập ở đây, bao gồm “range-over-func”, gói `unique` mới,
các thay đổi trong triển khai bộ hẹn giờ của Go 1.23, và nhiều nội dung khác.

---

Cảm ơn tất cả mọi người đã đóng góp cho bản phát hành này bằng cách viết mã và
tài liệu, báo cáo lỗi, chia sẻ phản hồi và thử nghiệm các bản ứng viên phát hành.
Những nỗ lực của các bạn đã giúp đảm bảo rằng Go 1.23 ổn định nhất có thể.
Như thường lệ, nếu bạn nhận thấy bất kỳ vấn đề nào, vui lòng [tạo một issue](/issue/new).

Chúc bạn tận hưởng Go 1.23!
