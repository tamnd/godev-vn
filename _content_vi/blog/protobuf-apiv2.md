---
title: API mới của Go cho Protocol Buffers
date: 2020-03-02
by:
- Joe Tsai
- Damien Neil
- Herbie Ong
tags:
- protobuf
- technical
summary: Thông báo về bản sửa đổi lớn của API Go cho protocol buffers.
---

## Giới thiệu

Chúng tôi vui mừng thông báo về bản phát hành sửa đổi lớn của API Go cho
[protocol buffers](https://developers.google.com/protocol-buffers),
định dạng trao đổi dữ liệu trung lập với ngôn ngữ của Google.

## Lý do cần API mới

Các binding protocol buffer đầu tiên cho Go được
[Rob Pike thông báo](/blog/third-party-libraries-goprotobuf-and)
vào tháng 3 năm 2010. Go 1 sẽ không được phát hành cho đến hai năm sau đó.

Trong thập kỷ kể từ bản phát hành đầu tiên đó, gói này đã phát triển và
tiến hóa cùng với Go. Yêu cầu của người dùng cũng ngày càng tăng.

Nhiều người muốn viết các chương trình sử dụng reflection để kiểm tra
các thông điệp protocol buffer. Gói
[`reflect`](https://pkg.go.dev/reflect)
cung cấp một góc nhìn về các kiểu và giá trị Go,
nhưng bỏ qua thông tin từ hệ thống kiểu của protocol buffer. Chẳng hạn,
chúng ta có thể muốn viết một hàm duyệt qua một mục nhật ký và xóa bất kỳ
trường nào được chú thích là chứa dữ liệu nhạy cảm. Các chú thích đó không
phải là một phần của hệ thống kiểu Go.

Một nhu cầu phổ biến khác là sử dụng các cấu trúc dữ liệu khác ngoài những
cấu trúc được tạo ra bởi trình biên dịch protocol buffer, chẳng hạn như kiểu
thông điệp động có thể biểu diễn các thông điệp mà kiểu của chúng không được
biết tại thời điểm biên dịch.

Chúng tôi cũng nhận thấy rằng một nguồn gốc thường gặp của các vấn đề là
interface
[`proto.Message`](https://pkg.go.dev/github.com/golang/protobuf/proto?tab=doc#Message),
vốn xác định các giá trị của các kiểu thông điệp được tạo ra, lại mô tả
quá ít về hành vi của những kiểu đó. Khi người dùng tạo ra các kiểu cài đặt
interface đó (thường là vô tình bằng cách nhúng một thông điệp vào một struct
khác) và truyền các giá trị của những kiểu đó vào các hàm mong đợi một giá trị
thông điệp được tạo ra, chương trình sẽ bị crash hoặc hoạt động không thể đoán
trước được.

Cả ba vấn đề này đều có một nguyên nhân chung và một giải pháp chung:
Interface `Message` nên mô tả đầy đủ hành vi của một thông điệp, và các hàm
thao tác trên các giá trị `Message` nên chấp nhận tự do bất kỳ kiểu nào cài
đặt interface đó đúng cách.

Vì không thể thay đổi định nghĩa hiện có của kiểu `Message` trong khi vẫn
duy trì tính tương thích API của gói, chúng tôi quyết định đã đến lúc bắt
đầu làm việc trên một phiên bản mới, không tương thích ngược của module protobuf.

Hôm nay, chúng tôi vui mừng phát hành module mới đó. Hy vọng bạn sẽ thích nó.

## Reflection

Reflection là tính năng hàng đầu của cài đặt mới. Tương tự như cách gói
`reflect` cung cấp góc nhìn về các kiểu và giá trị Go, gói
[`google.golang.org/protobuf/reflect/protoreflect`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc)
cung cấp góc nhìn về các giá trị theo hệ thống kiểu của protocol buffer.

Mô tả đầy đủ về gói `protoreflect` sẽ quá dài cho bài viết này, nhưng hãy
cùng xem cách chúng ta có thể viết hàm xóa thông tin nhạy cảm trong nhật ký
mà chúng tôi đã đề cập trước đó.

Trước tiên, chúng ta sẽ viết một tệp `.proto` định nghĩa phần mở rộng của kiểu
[`google.protobuf.FieldOptions`](https://github.com/protocolbuffers/protobuf/blob/b96241b1b716781f5bc4dc25e1ebb0003dfaba6a/src/google/protobuf/descriptor.proto#L509)
để chúng ta có thể chú thích các trường là có chứa thông tin nhạy cảm hay không.

	syntax = "proto3";
	import "google/protobuf/descriptor.proto";
	package golang.example.policy;
	extend google.protobuf.FieldOptions {
		bool non_sensitive = 50000;
	}

Chúng ta có thể dùng tùy chọn này để đánh dấu một số trường là không nhạy cảm.

	message MyMessage {
		string public_name = 1 [(golang.example.policy.non_sensitive) = true];
	}

Tiếp theo, chúng ta sẽ viết một hàm Go nhận bất kỳ giá trị thông điệp nào
và xóa tất cả các trường nhạy cảm.

	// Redact clears every sensitive field in pb.
	func Redact(pb proto.Message) {
	   // ...
	}

Hàm này nhận một
[`proto.Message`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Message),
một kiểu interface được cài đặt bởi tất cả các kiểu thông điệp được tạo ra.
Kiểu này là một alias cho kiểu được định nghĩa trong gói `protoreflect`:

	type ProtoMessage interface{
		ProtoReflect() Message
	}

Để tránh làm đầy namespace của các thông điệp được tạo ra, interface chỉ
chứa một phương thức duy nhất trả về một
[`protoreflect.Message`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Message),
cung cấp quyền truy cập vào nội dung của thông điệp.

(Tại sao lại dùng alias? Vì `protoreflect.Message` có một phương thức tương
ứng trả về `proto.Message` gốc, và chúng ta cần tránh vòng lặp import giữa
hai gói.)

Phương thức
[`protoreflect.Message.Range`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Message.Range)
gọi một hàm cho mỗi trường được điền trong một thông điệp.

	m := pb.ProtoReflect()
	m.Range(func(fd protoreflect.FieldDescriptor, v protoreflect.Value) bool {
		// ...
		return true
	})

Hàm range được gọi với một
[`protoreflect.FieldDescriptor`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#FieldDescriptor)
mô tả kiểu protocol buffer của trường, và một
[`protoreflect.Value`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Value)
chứa giá trị của trường.

Phương thức
[`protoreflect.FieldDescriptor.Options`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Descriptor.Options)
trả về các tùy chọn của trường dưới dạng thông điệp `google.protobuf.FieldOptions`.

	opts := fd.Options().(*descriptorpb.FieldOptions)

(Tại sao lại có type assertion? Vì gói `descriptorpb` được tạo ra phụ thuộc
vào `protoreflect`, gói `protoreflect` không thể trả về kiểu tùy chọn cụ thể
mà không gây ra vòng lặp import.)

Sau đó chúng ta có thể kiểm tra các tùy chọn để xem giá trị của boolean mở
rộng của mình:

	if proto.GetExtension(opts, policypb.E_NonSensitive).(bool) {
		return true // don't redact non-sensitive fields
	}

Lưu ý rằng chúng ta đang xem _descriptor_ của trường ở đây, không phải
_giá trị_ của trường. Thông tin chúng ta quan tâm nằm trong hệ thống kiểu
của protocol buffer, không phải hệ thống kiểu Go.

Đây cũng là một ví dụ về khu vực mà chúng ta đã đơn giản hóa API của gói
`proto`. Hàm
[`proto.GetExtension`](https://pkg.go.dev/github.com/golang/protobuf/proto?tab=doc#GetExtension)
ban đầu trả về cả một giá trị và một lỗi. Hàm
[`proto.GetExtension`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#GetExtension)
mới chỉ trả về một giá trị, trả về giá trị mặc định cho trường nếu nó không
có mặt. Lỗi giải mã extension được báo cáo tại thời điểm `Unmarshal`.

Khi đã xác định được một trường cần xóa thông tin nhạy cảm, việc xóa nó rất
đơn giản:

	m.Clear(fd)

Tổng hợp tất cả những điều trên, hàm xóa thông tin nhạy cảm hoàn chỉnh của
chúng ta là:

	// Redact clears every sensitive field in pb.
	func Redact(pb proto.Message) {
		m := pb.ProtoReflect()
		m.Range(func(fd protoreflect.FieldDescriptor, v protoreflect.Value) bool {
			opts := fd.Options().(*descriptorpb.FieldOptions)
			if proto.GetExtension(opts, policypb.E_NonSensitive).(bool) {
				return true
			}
			m.Clear(fd)
			return true
		})
	}

Một cài đặt đầy đủ hơn có thể đệ quy đi sâu vào các trường có giá trị
là thông điệp. Hy vọng rằng ví dụ đơn giản này cho thấy một chút về reflection
của protocol buffer và các ứng dụng của nó.

## Các phiên bản

Chúng tôi gọi phiên bản gốc của Go protocol buffers là APIv1, và phiên bản
mới là APIv2. Vì APIv2 không tương thích ngược với APIv1, chúng ta cần sử dụng
các đường dẫn module khác nhau cho mỗi phiên bản.

(Các phiên bản API này không giống với các phiên bản của ngôn ngữ protocol
buffer: `proto1`, `proto2` và `proto3`. APIv1 và APIv2 là các cài đặt cụ thể
trong Go, đều hỗ trợ các phiên bản ngôn ngữ `proto2` và `proto3`.)

Module
[`github.com/golang/protobuf`](https://pkg.go.dev/github.com/golang/protobuf?tab=overview)
là APIv1.

Module
[`google.golang.org/protobuf`](https://pkg.go.dev/google.golang.org/protobuf?tab=overview)
là APIv2. Chúng tôi đã tận dụng sự cần thiết phải thay đổi đường dẫn import
để chuyển sang một đường dẫn không gắn với một nhà cung cấp hosting cụ thể.
(Chúng tôi đã cân nhắc `google.golang.org/protobuf/v2`, để làm rõ rằng đây là
phiên bản lớn thứ hai của API, nhưng cuối cùng chọn đường dẫn ngắn hơn vì nó
là lựa chọn tốt hơn về lâu dài.)

Chúng tôi biết rằng không phải tất cả người dùng sẽ chuyển sang phiên bản
lớn mới của một gói với cùng tốc độ. Một số sẽ chuyển đổi nhanh chóng; những
người khác có thể vẫn ở phiên bản cũ vô thời hạn. Ngay cả trong một chương
trình đơn lẻ, một số phần có thể sử dụng API này trong khi những phần khác
sử dụng API kia. Do đó, điều cần thiết là chúng ta tiếp tục hỗ trợ các chương
trình sử dụng APIv1.

  - `github.com/golang/protobuf@v1.3.4` là phiên bản APIv1 mới nhất trước APIv2.

  - `github.com/golang/protobuf@v1.4.0` là phiên bản APIv1 được cài đặt theo
    thuật ngữ của APIv2. API vẫn giống nhau, nhưng cài đặt bên dưới được hỗ
    trợ bởi cái mới. Phiên bản này chứa các hàm để chuyển đổi giữa interface
    `proto.Message` của APIv1 và APIv2 để dễ dàng chuyển tiếp giữa hai phiên bản.

  - `google.golang.org/protobuf@v1.20.0` là APIv2. Module này phụ thuộc vào
    `github.com/golang/protobuf@v1.4.0`, do đó bất kỳ chương trình nào sử dụng
    APIv2 sẽ tự động chọn một phiên bản APIv1 tích hợp với nó.

(Tại sao bắt đầu ở phiên bản `v1.20.0`? Để rõ ràng hơn. Chúng tôi không
dự đoán APIv1 sẽ đạt đến `v1.20.0`, vì vậy số phiên bản một mình đã đủ để
phân biệt rõ ràng giữa APIv1 và APIv2.)

Chúng tôi có kế hoạch duy trì hỗ trợ cho APIv1 vô thời hạn.

Tổ chức này đảm bảo rằng bất kỳ chương trình nào cũng sẽ chỉ sử dụng một
cài đặt protocol buffer duy nhất, bất kể phiên bản API nào nó sử dụng. Nó cho
phép các chương trình áp dụng API mới một cách dần dần, hoặc không áp dụng
chút nào, trong khi vẫn được hưởng lợi từ cài đặt mới. Nguyên tắc chọn phiên
bản tối thiểu có nghĩa là các chương trình có thể vẫn ở cài đặt cũ cho đến
khi người bảo trì chọn cập nhật lên cái mới (trực tiếp hoặc bằng cách cập
nhật một dependency).

## Các tính năng đáng chú ý khác

Gói
[`google.golang.org/protobuf/encoding/protojson`](https://pkg.go.dev/google.golang.org/protobuf/encoding/protojson)
chuyển đổi các thông điệp protocol buffer sang và từ JSON sử dụng
[ánh xạ JSON chuẩn](https://developers.google.com/protocol-buffers/docs/proto3#json),
và khắc phục một số vấn đề với gói `jsonpb` cũ mà rất khó thay đổi mà không
gây ra vấn đề cho người dùng hiện tại.

Gói
[`google.golang.org/protobuf/types/dynamicpb`](https://pkg.go.dev/google.golang.org/protobuf/types/dynamicpb)
cung cấp một cài đặt `proto.Message` cho các thông điệp mà kiểu protocol buffer
của chúng được suy ra tại thời gian chạy.

Gói
[`google.golang.org/protobuf/testing/protocmp`](https://pkg.go.dev/google.golang.org/protobuf/testing/protocmp)
cung cấp các hàm để so sánh các thông điệp protocol buffer với gói
[`github.com/google/cmp`](https://pkg.go.dev/github.com/google/go-cmp/cmp).

Gói
[`google.golang.org/protobuf/compiler/protogen`](https://pkg.go.dev/google.golang.org/protobuf/compiler/protogen?tab=doc)
cung cấp hỗ trợ để viết các plugin của trình biên dịch protocol.

## Kết luận

Module `google.golang.org/protobuf` là sự cải tổ lớn về hỗ trợ protocol buffers
của Go, cung cấp hỗ trợ hạng nhất cho reflection, các cài đặt thông điệp tùy
chỉnh, và một bề mặt API được làm sạch hơn. Chúng tôi có kế hoạch duy trì API
trước đó vô thời hạn như một lớp bọc của cái mới, cho phép người dùng áp dụng
API mới dần dần theo tốc độ của riêng họ.

Mục tiêu của chúng tôi trong bản cập nhật này là cải thiện những lợi ích của
API cũ trong khi giải quyết những thiếu sót của nó. Khi chúng tôi hoàn thành
từng thành phần của cài đặt mới, chúng tôi đưa nó vào sử dụng trong codebase
của Google. Việc triển khai dần dần này đã cho chúng tôi sự tự tin vào cả
khả năng sử dụng của API mới lẫn hiệu năng và tính đúng đắn của cài đặt mới.
Chúng tôi tin rằng nó đã sẵn sàng cho môi trường production.

Chúng tôi rất hào hứng với bản phát hành này và hy vọng rằng nó sẽ phục vụ
hệ sinh thái Go trong mười năm tới và xa hơn nữa!
