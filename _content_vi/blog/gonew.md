---
title: Thử nghiệm với các template dự án
date: 2023-07-31
by:
- Cameron Balahan
summary: Thông báo golang.org/x/tools/cmd/gonew, công cụ thử nghiệm để bắt đầu dự án Go mới từ các template được định nghĩa trước
---

Khi bạn bắt đầu một dự án mới trong Go, bạn có thể bắt đầu bằng cách clone một dự án hiện có.
Theo cách đó, bạn có thể bắt đầu với thứ gì đó đã hoạt động,
thực hiện các thay đổi dần dần thay vì bắt đầu từ đầu.

Từ lâu, chúng tôi đã nghe từ các nhà phát triển Go rằng việc bắt đầu
thường là phần khó khăn nhất.
Các nhà phát triển mới đến từ các ngôn ngữ khác mong đợi hướng dẫn về bố cục dự án mặc định,
các nhà phát triển có kinh nghiệm làm việc theo nhóm mong đợi sự nhất quán trong dependency của dự án,
và các nhà phát triển thuộc mọi loại mong đợi một cách dễ dàng để thử các sản phẩm và dịch vụ mới
mà không cần sao chép và dán từ các mẫu trên web.

Để đạt mục tiêu đó, hôm nay chúng tôi đã xuất bản `gonew`, một công cụ thử nghiệm để khởi tạo
các dự án mới trong Go từ các template được định nghĩa trước.
Bất kỳ ai cũng có thể viết template, được đóng gói và phân phối dưới dạng module,
tận dụng module proxy và checksum database của Go để có bảo mật và khả dụng tốt hơn.

Phiên bản nguyên mẫu `gonew` cố tình ở mức tối thiểu:
những gì chúng tôi đã phát hành hôm nay là một nguyên mẫu cực kỳ giới hạn nhằm cung cấp
một cơ sở từ đó chúng tôi có thể thu thập phản hồi và định hướng cộng đồng.
Hãy thử nó, [cho chúng tôi biết bạn nghĩ gì](/s/gonew-feedback),
và giúp chúng tôi xây dựng một công cụ hữu ích hơn cho mọi người.

## Bắt đầu

Bắt đầu bằng cách cài đặt `gonew` bằng [`go install`](https://pkg.go.dev/cmd/go#hdr-Compile_and_install_packages_and_dependencies):

```
$ go install golang.org/x/tools/cmd/gonew@latest
```

Để sao chép một template hiện có, chạy `gonew` trong thư mục cha của dự án mới
với hai đối số:
đầu tiên, đường dẫn đến template bạn muốn sao chép,
và thứ hai, tên module của dự án bạn đang tạo. Ví dụ:

```
$ gonew golang.org/x/example/helloserver example.com/myserver
$ cd ./myserver
```

Sau đó bạn có thể đọc và chỉnh sửa các file trong `./myserver` để tùy chỉnh.

Chúng tôi đã viết hai template để bạn bắt đầu:

- [hello](https://pkg.go.dev/golang.org/x/example/hello):
  Công cụ dòng lệnh in lời chào,
  với các cờ tùy chỉnh.
- [helloserver](https://pkg.go.dev/golang.org/x/example/helloserver): Một HTTP server phục vụ các lời chào.

## Viết template của riêng bạn

Viết template của riêng bạn đơn giản như [tạo bất kỳ module nào khác](/doc/tutorial/create-module) trong Go.
Xem các ví dụ chúng tôi đã liên kết ở trên để bắt đầu.

Cũng có các ví dụ từ nhóm [Google Cloud](https://github.com/GoogleCloudPlatform/go-templates)
và [Service Weaver](https://github.com/ServiceWeaver/template).

## Các bước tiếp theo

Hãy thử `gonew` và cho chúng tôi biết cách chúng tôi có thể làm cho nó tốt hơn và hữu ích hơn.
Nhớ rằng, `gonew` hiện chỉ là một thử nghiệm;
chúng tôi cần [phản hồi của bạn để làm đúng](/s/gonew-feedback).
