---
title: Mô hình mối đe dọa của Go
layout: article
breadcrumb: true
---

## Tổng quan

Tài liệu này định nghĩa mô hình mối đe dọa chung cho chuỗi công cụ Go và thư viện chuẩn. Trong trường hợp không có tài liệu của gói định nghĩa mô hình mối đe dọa, mô hình được mô tả trong tài liệu này nên được xem là áp dụng cho tất cả các gói trong thư viện chuẩn.

### Mô hình mối đe dọa

#### An toàn khi xây dựng

Việc xây dựng mã Go được giả định là an toàn và không có tác dụng phụ, chẳng hạn như thực thi ngoài dự kiến.

#### Thực thi trong thời gian chạy

Nhìn chung, chúng tôi không xem việc thực thi mã độc là một vấn đề bảo mật liên quan. Người dùng đã quen thuộc với Go được giả định là hiểu những gì họ đang thực thi.

#### An toàn bộ nhớ

Giả định rằng khi không sử dụng gói `unsafe`, tính an toàn bộ nhớ được đảm bảo bởi runtime.

#### Ranh giới tin cậy

Các API được kỳ vọng hợp lý là sẽ nhận đầu vào tùy ý do người dùng cung cấp được giả định là đã được tăng cường để chống lại panic và việc tiêu thụ tài nguyên tùy ý.

Việc truyền dữ liệu rác vào một API dẫn đến đầu ra không mong muốn không được xem là vấn đề bảo mật.

#### Kiểm soát môi trường

Giả định rằng hệ thống cục bộ là an toàn. Các cuộc tấn công dựa trên việc hệ điều hành đã bị xâm nhập trước đó không được xem là có liên quan. Ví dụ, chúng tôi không xem việc kẻ tấn công kiểm soát hệ thống tệp, các biến môi trường như PATH, hoặc quyền truy cập hay kiểm soát bộ nhớ là một phần trong mô hình của chúng tôi.

### Các gói có mô hình riêng

* [encoding/json](/pkg/encoding/json/#hdr-Security_Considerations)
* [encoding/json/v2](/pkg/encoding/json/v2/#hdr-Security_Considerations)
* [encoding/json/jsontext](/pkg/encoding/json/jsontext/#hdr-Security_Considerations)
* [encoding/gob](/pkg/encoding/gob/#hdr-Security)
* [html/template](/pkg/html/template/#hdr-Security_Model)
* [image](/pkg/image/#hdr-Security_Considerations)
* [debug/pe](/pkg/debug/pe/#hdr-Security)
* [debug/macho](/pkg/debug/macho/#hdr-Security)
* [debug/dwarf](/pkg/debug/dwarf/#hdr-Security)
* [debug/plan9obj](/pkg/debug/plan9obj/#hdr-Security)
* [debug/elf](/pkg/debug/elf/#hdr-Security)