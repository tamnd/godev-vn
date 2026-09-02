---
title: "JSON-RPC: câu chuyện về interface"
date: 2010-04-27
by:
- Andrew Gerrand
tags:
- json
- rpc
- technical
summary: Cách sử dụng interface của package net/rpc để xây dựng hệ thống JSON-RPC.
---


Ở đây chúng ta trình bày một ví dụ mà [interface](/doc/effective_go.html#interfaces_and_types) của Go
đã giúp dễ dàng tái cấu trúc một số mã hiện có để làm cho nó linh hoạt và có thể mở rộng hơn.
Ban đầu, [package RPC](/pkg/net/rpc/) trong thư viện chuẩn
sử dụng định dạng truyền tải tùy chỉnh gọi là [gob](/pkg/encoding/gob/).
Với một ứng dụng cụ thể, chúng tôi muốn sử dụng [JSON](/pkg/encoding/json/)
như một định dạng truyền tải thay thế.

Đầu tiên chúng tôi định nghĩa một cặp interface để mô tả chức năng của
định dạng truyền tải hiện có,
một cho phía client và một cho phía server (được mô tả bên dưới).

	type ServerCodec interface {
	 ReadRequestHeader(*Request) error
	 ReadRequestBody(interface{}) error
	 WriteResponse(*Response, interface{}) error
	 Close() error
	}

Ở phía server, chúng tôi đã thay đổi hai chữ ký hàm nội bộ để
chấp nhận interface `ServerCodec` thay vì `gob.Encoder` hiện có. Đây là một trong số đó:

	func sendResponse(sending *sync.Mutex, req *Request,
	 reply interface{}, enc *gob.Encoder, errmsg string)

trở thành

	func sendResponse(sending *sync.Mutex, req *Request,
	  reply interface{}, enc ServerCodec, errmsg string)

Sau đó chúng tôi viết một wrapper `gobServerCodec` đơn giản để tái tạo chức năng gốc.
Từ đó, việc xây dựng `jsonServerCodec` rất đơn giản.

Sau một số thay đổi tương tự ở phía client,
đây là toàn bộ công việc chúng tôi cần làm trên package RPC.
Toàn bộ quá trình này mất khoảng 20 phút!
Sau khi dọn dẹp và kiểm tra mã mới,
[changeset cuối cùng](https://github.com/golang/go/commit/dcff89057bc0e0d7cb14cf414f2df6f5fb1a41ec) đã được gửi.

Trong một ngôn ngữ hướng kế thừa như Java hay C++,
hướng đi rõ ràng là tổng quát hóa lớp RPC,
và tạo ra các lớp con JsonRPC và GobRPC.
Tuy nhiên, cách tiếp cận này trở nên phức tạp nếu bạn muốn thực hiện tổng quát hóa thêm
vuông góc với hệ thống phân cấp đó.
(Ví dụ, nếu bạn muốn triển khai một tiêu chuẩn RPC thay thế).
Trong package Go của chúng tôi, chúng tôi đã chọn một con đường vừa đơn giản hơn về mặt khái niệm
vừa đòi hỏi ít mã hơn để viết hoặc thay đổi.

Một phẩm chất quan trọng của mọi codebase là khả năng bảo trì.
Khi nhu cầu thay đổi, điều cần thiết là thích nghi mã của bạn một cách dễ dàng và gọn gàng,
nếu không nó sẽ trở nên khó làm việc.
Chúng tôi tin rằng hệ thống kiểu nhẹ, hướng kết hợp của Go cung cấp
một phương tiện cấu trúc mã có thể mở rộng.
