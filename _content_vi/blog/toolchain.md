---
title: Tương thích tiến và Quản lý toolchain trong Go 1.21
date: 2023-08-14T12:00:01Z
by:
- Russ Cox
summary: Go 1.21 quản lý các Go toolchain giống như bất kỳ dependency nào khác; bạn sẽ không bao giờ cần tải về và cài đặt thủ công một Go toolchain nữa.
---

Ngoài [cam kết mở rộng về tương thích ngược](compat) trong Go 1.21,
Go 1.21 còn giới thiệu khả năng tương thích tiến tốt hơn cho mã Go,
nghĩa là Go 1.21 trở đi sẽ cẩn thận hơn để không biên dịch sai
mã yêu cầu một phiên bản Go mới hơn.
Cụ thể, dòng `go` trong `go.mod` giờ xác định phiên bản Go toolchain tối thiểu bắt buộc,
trong khi ở các bản phát hành trước đó nó chỉ là một gợi ý không được thực thi chặt chẽ.

Để việc theo kịp các yêu cầu này trở nên dễ dàng hơn,
Go 1.21 cũng giới thiệu tính năng quản lý toolchain,
cho phép các module khác nhau sử dụng các Go toolchain khác nhau
giống như chúng có thể sử dụng các phiên bản khác nhau của một module bắt buộc.
Sau khi cài đặt Go 1.21, bạn sẽ không bao giờ phải tải về và cài đặt thủ công một Go toolchain nữa.
Lệnh `go` có thể làm điều đó cho bạn.

Phần còn lại của bài viết này mô tả chi tiết hơn cả hai thay đổi trong Go 1.21.

## Tương thích tiến {#forward}

Tương thích tiến đề cập đến điều gì xảy ra khi một Go toolchain
cố gắng biên dịch mã Go được viết cho phiên bản Go mới hơn.
Nếu chương trình của tôi phụ thuộc vào module M và cần một bản sửa lỗi
được thêm vào trong M v1.2.3, tôi có thể thêm `require M v1.2.3` vào `go.mod`,
đảm bảo rằng chương trình của tôi sẽ không được biên dịch với các phiên bản cũ hơn của M.
Nhưng nếu chương trình của tôi yêu cầu một phiên bản Go cụ thể, thì
chưa có cách nào để diễn đạt điều đó: đặc biệt là dòng `go` trong `go.mod`
không truyền đạt được điều này.

Ví dụ, nếu tôi viết mã sử dụng generics mới
được thêm vào trong Go 1.18, tôi có thể viết `go 1.18` trong tệp `go.mod` của mình,
nhưng điều đó sẽ không ngăn các phiên bản Go cũ hơn cố gắng biên dịch mã,
tạo ra các lỗi như:

	$ cat go.mod
	go 1.18
	module example

	$ go version
	go version go1.17

	$ go build
	# example
	./x.go:2:6: missing function body
	./x.go:2:7: syntax error: unexpected [, expecting (
	note: module requires Go 1.18
	$

Hai lỗi biên dịch là nhiễu gây nhầm lẫn.
Vấn đề thực sự được lệnh `go` in ra như một gợi ý:
chương trình biên dịch thất bại, nên lệnh `go` chỉ ra
khả năng không khớp phiên bản.

Trong ví dụ này, chúng ta may mắn khi build thất bại.
Nếu tôi viết mã chỉ chạy đúng trong Go 1.19 trở lên,
vì nó phụ thuộc vào một lỗi đã được sửa trong bản vá đó,
nhưng tôi không sử dụng bất kỳ tính năng ngôn ngữ hoặc gói nào riêng của Go 1.19
trong mã, các phiên bản Go cũ hơn sẽ biên dịch nó
và thành công trong im lặng.

Bắt đầu từ Go 1.21, các Go toolchain sẽ coi dòng `go` trong
`go.mod` không phải là hướng dẫn mà là quy tắc bắt buộc, và dòng này có thể
liệt kê các bản phát hành điểm cụ thể hoặc release candidate.
Tức là, Go 1.21.0 hiểu rằng nó không thể build được mã
mà ghi `go 1.21.1` trong tệp `go.mod`,
chứ chưa nói đến mã ghi các phiên bản muộn hơn nhiều như `go 1.22.0`.

Lý do chính khiến chúng tôi cho phép các phiên bản Go cũ hơn cố gắng
biên dịch mã mới hơn là để tránh các lỗi build không cần thiết.
Thật rất bực bội khi bị thông báo rằng phiên bản Go của bạn quá
cũ để build một chương trình, đặc biệt nếu nó có thể hoạt động được
(có thể yêu cầu đó quá thận trọng không cần thiết),
và đặc biệt khi việc cập nhật lên phiên bản Go mới hơn khá phiền phức.
Để giảm tác động của việc thực thi dòng `go` như một yêu cầu bắt buộc,
Go 1.21 cũng thêm tính năng quản lý toolchain vào bản phân phối cốt lõi.

## Quản lý Toolchain

Khi bạn cần một phiên bản mới của một Go module, lệnh `go`
sẽ tải nó về cho bạn.
Bắt đầu từ Go 1.21, khi bạn cần một Go toolchain mới hơn,
lệnh `go` cũng tải nó về cho bạn.
Chức năng này giống như `nvm` của Node hay `rustup` của Rust, nhưng được tích hợp
vào lệnh `go` cốt lõi thay vì là một công cụ riêng.

Nếu bạn đang chạy Go 1.21.0 và chạy một lệnh `go`, chẳng hạn `go build`,
trong một module với `go.mod` ghi `go 1.21.1`,
lệnh `go` của Go 1.21.0 sẽ nhận thấy rằng bạn cần Go 1.21.1,
tải nó về, và gọi lại lệnh `go` của phiên bản đó để hoàn thành quá trình build.
Khi lệnh `go` tải về và chạy các toolchain khác này,
nó không cài đặt chúng vào PATH của bạn hay ghi đè lên cài đặt hiện tại.
Thay vào đó, nó tải chúng về dưới dạng Go module, kế thừa tất cả
[các lợi ích bảo mật và quyền riêng tư của module](/blog/module-mirror-launch),
và sau đó chạy chúng từ bộ nhớ đệm module.

Cũng có một dòng `toolchain` mới trong `go.mod` xác định
Go toolchain tối thiểu cần dùng khi làm việc trong một module cụ thể.
Trái ngược với dòng `go`, dòng `toolchain` không áp đặt yêu cầu
cho các module khác.
Ví dụ, một `go.mod` có thể ghi:

	module m
	go 1.21.0
	toolchain go1.21.4

Điều này nói rằng các module khác yêu cầu `m` cần cung cấp ít nhất Go 1.21.0,
nhưng khi chúng tôi đang làm việc trong `m` chính nó, chúng tôi muốn một toolchain mới hơn,
ít nhất là Go 1.21.4.

Các yêu cầu `go` và `toolchain` có thể được cập nhật bằng `go get`
giống như các yêu cầu module thông thường. Ví dụ, nếu bạn đang sử dụng một
trong các release candidate của Go 1.21, bạn có thể bắt đầu sử dụng Go 1.21.0
trong một module cụ thể bằng cách chạy:

	go get go@1.21.0

Lệnh đó sẽ tải về và chạy Go 1.21.0 để cập nhật dòng `go`,
và các lần gọi lệnh `go` trong tương lai sẽ thấy dòng
`go 1.21.0` và tự động gọi lại phiên bản đó.

Hoặc nếu bạn muốn bắt đầu sử dụng Go 1.21.0 trong một module nhưng để
dòng `go` ở phiên bản cũ hơn, để giúp duy trì tương thích với
người dùng các phiên bản Go cũ hơn, bạn có thể cập nhật dòng `toolchain`:

	go get toolchain@go1.21.0

Nếu bạn muốn biết phiên bản Go nào đang chạy trong một module cụ thể,
câu trả lời giống như trước: chạy `go version`.

Bạn có thể buộc sử dụng một phiên bản Go toolchain cụ thể bằng cách dùng
biến môi trường GOTOOLCHAIN.
Ví dụ, để kiểm thử mã với Go 1.20.4:

	GOTOOLCHAIN=go1.20.4 go test

Cuối cùng, cài đặt GOTOOLCHAIN có dạng `version+auto` có nghĩa là
sử dụng `version` theo mặc định nhưng cũng cho phép nâng cấp lên các phiên bản mới hơn.
Nếu bạn đã cài đặt Go 1.21.0, thì khi Go 1.21.1 được phát hành,
bạn có thể thay đổi mặc định hệ thống của mình bằng cách thiết lập GOTOOLCHAIN mặc định:

	go env -w GOTOOLCHAIN=go1.21.1+auto

Bạn sẽ không bao giờ phải tải về và cài đặt thủ công một Go toolchain nữa.
Lệnh `go` sẽ lo việc đó cho bạn.

Xem "[Go Toolchains](/doc/toolchain)" để biết thêm chi tiết.
