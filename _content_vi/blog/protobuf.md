---
title: "Thư viện bên thứ ba: goprotobuf và hơn thế nữa"
date: 2010-04-20
by:
- Andrew Gerrand
tags:
- protobuf
- community
summary: Thông báo hỗ trợ Go cho Protocol Buffers, định dạng trao đổi dữ liệu của Google.
---


Vào ngày 24 tháng 3, Rob Pike đã thông báo về [goprotobuf](http://code.google.com/p/goprotobuf/),
các binding Go của định dạng trao đổi dữ liệu [Protocol Buffers](http://code.google.com/apis/protocolbuffers/docs/overview.html) của Google,
thường được gọi tắt là protobufs.
Với thông báo này, Go gia nhập C++, Java,
và Python với tư cách là các ngôn ngữ cung cấp cài đặt protobuf chính thức.
Đây đánh dấu một cột mốc quan trọng trong việc cho phép khả năng tương tác giữa
các hệ thống hiện có và những hệ thống được xây dựng bằng Go.

Dự án goprotobuf gồm hai phần:
một 'plugin trình biên dịch protocol' tạo ra các tệp nguồn Go mà,
sau khi biên dịch, có thể truy cập và quản lý protocol buffers;
và một gói Go cài đặt hỗ trợ thời gian chạy cho việc mã hóa (marshaling),
giải mã (unmarshaling), và truy cập protocol buffers.

Để sử dụng goprotobuf, trước tiên bạn cần cài đặt cả Go và [protobuf](http://code.google.com/p/protobuf/).
Sau đó bạn có thể cài đặt gói 'proto' với [goinstall](/cmd/goinstall/):

	goinstall goprotobuf.googlecode.com/hg/proto

Và sau đó cài đặt plugin trình biên dịch protobuf:

	cd $GOROOT/src/pkg/goprotobuf.googlecode.com/hg/compiler
	make install

Để biết thêm chi tiết, hãy xem tệp [README](http://code.google.com/p/goprotobuf/source/browse/README) của dự án.

Đây là một trong danh sách ngày càng tăng của các [dự án Go](http://godashboard.appspot.com/package) của bên thứ ba.
Kể từ khi thông báo về goprotobuf, các binding X Go đã được tách ra
từ thư viện chuẩn sang dự án [x-go-binding](http://code.google.com/p/x-go-binding/),
và công việc đã bắt đầu với một port [Freetype](http://www.freetype.org/),
[freetype-go](http://code.google.com/p/freetype-go/).
Các dự án bên thứ ba phổ biến khác bao gồm framework web nhẹ
[web.go](https://github.com/hoisie/web),
và các binding Go GTK [gtk-go](https://github.com/mattn/go-gtk).

Chúng tôi muốn khuyến khích sự phát triển của các gói hữu ích khác bởi cộng đồng mã nguồn mở.
Nếu bạn đang làm gì đó, đừng giữ cho riêng mình - hãy cho chúng tôi biết
qua danh sách gửi thư [golang-nuts](http://groups.google.com/group/golang-nuts) của chúng tôi.
