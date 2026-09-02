---
title: Go 1.23 đã được phát hành
date: 2024-08-13
by:
- Dmitri Shuralyov, on behalf of the Go team
summary: Go 1.23 bổ sung iterator, tiếp tục cải tiến vòng lặp, cải thiện tương thích, và nhiều hơn nữa.
---

Hôm nay nhóm Go vui mừng phát hành Go 1.23,
bạn có thể tải về từ [trang download](/dl/).

Nếu bạn đã cài Go 1.22 hoặc Go 1.21 trên máy,
bạn cũng có thể thử `go get toolchain@go1.23.0` trong một module hiện có.
Lệnh này sẽ tải toolchain mới và cho phép bạn bắt đầu dùng nó
trong module của mình ngay lập tức. Sau đó, bạn có thể chạy thêm
`go get go@1.23.0` khi sẵn sàng chuyển hoàn toàn sang Go 1.23
và đặt đó là phiên bản Go tối thiểu yêu cầu của module.
Xem [Quản lý yêu cầu phiên bản module Go với go get](/doc/toolchain#get)
để biết thêm về chức năng này.

Go 1.23 có nhiều cải tiến so với Go 1.22. Một số điểm nổi bật bao gồm:

## Thay đổi ngôn ngữ

-	<!-- go.dev/issue/61405, go.dev/issue/61897, go.dev/issue/61899, go.dev/issue/61900 -->
	Biểu thức range trong vòng lặp "for-range" giờ có thể là các hàm iterator,
	chẳng hạn như `func(func(K) bool)`.
	Điều này hỗ trợ các iterator do người dùng định nghĩa trên các chuỗi tùy ý.
	Có một số bổ sung cho các gói chuẩn `slices` và `maps`
	hoạt động với iterator, cũng như gói `iter` mới.
	Ví dụ, nếu bạn muốn thu thập các khóa của map `m` vào một slice
	rồi sắp xếp các giá trị của nó, bạn có thể làm điều đó trong Go 1.23 với `slices.Sorted(maps.Keys(m))`.

	Go 1.23 cũng bao gồm hỗ trợ xem trước cho type alias generic.

	Đọc thêm về [thay đổi ngôn ngữ](/doc/go1.23#language) và [iterator](/doc/go1.23#iterators)
	trong ghi chú phát hành.

## Cải tiến công cụ

-	<!-- go.dev/issue/58894 -->
	Bắt đầu từ Go 1.23, Go toolchain có thể thu thập thống kê về cách dùng và sự cố
	để hiểu cách Go toolchain được sử dụng và hoạt động tốt như thế nào.
	Đây là Go telemetry, một hệ thống _tùy chọn bật_. Hãy cân nhắc bật để giúp chúng tôi giữ Go
	hoạt động tốt và hiểu rõ hơn cách dùng Go.
	Đọc thêm về [Go telemetry](/doc/go1.23#telemetry) trong ghi chú phát hành.
-	Lệnh `go` có các tiện ích mới. Ví dụ, chạy `go env -changed` giúp dễ dàng
	xem chỉ những cài đặt có giá trị hiệu dụng khác với giá trị mặc định, và
	`go mod tidy -diff` giúp xác định các thay đổi cần thiết cho file go.mod và go.sum
	mà không sửa đổi chúng.
	Đọc thêm về [lệnh Go](/doc/go1.23#go-command) trong ghi chú phát hành.
-	Lệnh con `go vet` nay báo cáo các ký hiệu quá mới cho phiên bản Go dự định.
	Đọc thêm về [công cụ](/doc/go1.23#tools) trong ghi chú phát hành.

## Cải tiến thư viện chuẩn

-	Go 1.23 cải tiến cài đặt của `time.Timer` và `time.Ticker`.
	Đọc thêm về [thay đổi timer](/doc/go1.23#timer-changes) trong ghi chú phát hành.
- 	Tổng cộng có 3 gói mới trong thư viện chuẩn Go 1.23: `iter`, `structs` và `unique`.
	Gói `iter` đã được đề cập ở trên.
	Gói `structs` định nghĩa các kiểu đánh dấu để sửa đổi thuộc tính của struct.
	Gói `unique` cung cấp các tiện ích để chuẩn hóa ("interning") các
	giá trị có thể so sánh.
	Đọc thêm về [các gói thư viện chuẩn mới](/doc/go1.23#new-unique-package)
	trong ghi chú phát hành.
-	Có nhiều cải tiến và bổ sung cho thư viện chuẩn được liệt kê trong phần
	[thay đổi nhỏ cho thư viện](/doc/go1.23#minor_library_changes)
	của ghi chú phát hành.
	Tài liệu "Go, Backwards Compatibility, and GODEBUG"
	liệt kê [các cài đặt GODEBUG mới trong Go 1.23](/doc/godebug#go-123).
-	<!-- go.dev/issue/65573 -->
	Go 1.23 hỗ trợ chỉ thị `godebug` mới trong các file `go.mod` và `go.work` để
	cho phép kiểm soát riêng biệt GODEBUG mặc định và chỉ thị "go" của `go.mod`,
	ngoài comment chỉ thị `//go:debug` có từ hai bản phát hành trước (Go 1.21).
	Xem tài liệu cập nhật về [Giá trị GODEBUG mặc định](/doc/godebug#default).

## Thêm cải tiến và thay đổi

-	Go 1.23 thêm hỗ trợ thử nghiệm cho OpenBSD trên RISC-V 64-bit (`openbsd/riscv64`).
	Có một số thay đổi nhỏ liên quan đến Linux, macOS, ARM64, RISC-V và WASI.
	Đọc thêm về [cổng](/doc/go1.23#ports) trong ghi chú phát hành.
-	Thời gian build khi dùng tối ưu hóa dựa trên hồ sơ thực thi (PGO) được giảm, và hiệu năng
	với PGO trên kiến trúc 386 và amd64 được cải thiện.
	Đọc thêm về [runtime, trình biên dịch và trình liên kết](/doc/go1.23#runtime) trong ghi chú phát hành.

Chúng tôi khuyến khích mọi người đọc [ghi chú phát hành Go 1.23](/doc/go1.23) để biết
thông tin đầy đủ và chi tiết về các thay đổi này, và mọi thứ mới trong Go 1.23.

Trong vài tuần tới, hãy chú ý các bài đăng blog tiếp theo sẽ đi sâu hơn
về một số chủ đề được đề cập ở đây, bao gồm "range-over-func", gói `unique` mới,
thay đổi cài đặt timer trong Go 1.23, và nhiều hơn nữa.

---

Cảm ơn tất cả những người đã đóng góp cho bản phát hành này bằng cách viết code và
tài liệu, báo cáo lỗi, chia sẻ phản hồi, và kiểm thử các release
candidate. Nỗ lực của bạn giúp đảm bảo Go 1.23 ổn định nhất có thể.
Như thường lệ, nếu bạn phát hiện bất kỳ vấn đề nào, hãy [tạo issue](/issue/new).

Tận hưởng Go 1.23!
