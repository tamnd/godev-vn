<!--{
  "Title": "Phát triển và xuất bản module"
}-->

Bạn có thể tập hợp các package có liên quan vào một module, sau đó xuất bản module đó để các lập trình viên khác sử dụng. Chủ đề này cung cấp tổng quan về việc phát triển và xuất bản module.

Để hỗ trợ việc phát triển, xuất bản và sử dụng module, bạn dùng:

*   Một **quy trình làm việc** để phát triển và xuất bản module, liên tục cập nhật chúng với các phiên bản mới theo thời gian. Xem [Quy trình phát triển và xuất bản module](#workflow).
*	**Các nguyên tắc thiết kế** giúp người dùng module hiểu rõ module và nâng cấp lên các phiên bản mới một cách ổn định. Xem [Thiết kế và phát triển](#design).
*   Một **hệ thống xuất bản phi tập trung** cho module và truy xuất mã nguồn. Bạn đưa module của mình lên từ kho lưu trữ của bản thân và xuất bản kèm theo số phiên bản để các lập trình viên khác sử dụng. Xem [Xuất bản phi tập trung](#decentralized).
*   Một **công cụ tìm kiếm package** và trình duyệt tài liệu (pkg.go.dev) để lập trình viên có thể tìm thấy module của bạn. Xem [Khám phá package](#discovery).
*   Một **quy ước đánh số phiên bản** cho module nhằm truyền đạt mức độ ổn định và khả năng tương thích ngược tới những lập trình viên dùng module. Xem [Đánh số phiên bản](#versioning).
*   **Công cụ Go** giúp các lập trình viên khác quản lý dependency dễ dàng hơn, bao gồm tải mã nguồn module, nâng cấp và các tác vụ tương tự. Xem [Quản lý dependency](/doc/modules/managing-dependencies).

**Xem thêm**

*   Nếu bạn chỉ muốn sử dụng các package do người khác phát triển, chủ đề này không dành cho bạn. Thay vào đó, hãy xem [Quản lý dependency](managing-dependencies).
*   Để xem hướng dẫn bao gồm một số kiến thức cơ bản về phát triển module, hãy xem [Hướng dẫn: Tạo một module Go](/doc/tutorial/create-module).

## Quy trình phát triển và xuất bản module {#workflow}

Khi bạn muốn xuất bản module cho người khác sử dụng, bạn cần tuân theo một số quy ước để việc sử dụng module đó trở nên thuận tiện hơn.

Các bước cấp cao sau đây được mô tả chi tiết hơn trong [Quy trình phát hành và đánh số phiên bản module](release-workflow).

1. Thiết kế và viết code các package mà module sẽ bao gồm.
1. Commit code vào kho lưu trữ theo các quy ước đảm bảo người khác có thể truy cập thông qua công cụ Go.
1. Xuất bản module để lập trình viên có thể tìm thấy.
1. Theo thời gian, cập nhật module với các phiên bản theo quy ước đánh số phiên bản, phản ánh mức độ ổn định và khả năng tương thích ngược của từng phiên bản.

## Thiết kế và phát triển {#design}

Module của bạn sẽ dễ tìm và dễ sử dụng hơn nếu các hàm và package trong đó tạo thành một thể thống nhất. Khi thiết kế API công khai của module, hãy cố gắng giữ cho chức năng tập trung và rõ ràng.

Ngoài ra, việc thiết kế và phát triển module có tính đến khả năng tương thích ngược sẽ giúp người dùng nâng cấp mà ít phải thay đổi code của họ nhất. Bạn có thể áp dụng một số kỹ thuật trong code để tránh phát hành một phiên bản phá vỡ tính tương thích ngược. Để biết thêm về các kỹ thuật đó, xem [Giữ cho module của bạn tương thích](/blog/module-compatibility) trên blog Go.

Trước khi xuất bản module, bạn có thể tham chiếu đến nó trên hệ thống tệp cục bộ bằng cách dùng chỉ thị `replace`. Điều này giúp dễ dàng viết code client gọi các hàm trong module khi module đó vẫn đang được phát triển. Để biết thêm thông tin, xem "Viết code dựa trên module chưa xuất bản" trong [Quy trình phát hành và đánh số phiên bản module](release-workflow#unpublished).

## Xuất bản phi tập trung {#decentralized}

Trong Go, bạn xuất bản module bằng cách gắn tag cho code trong kho lưu trữ để các lập trình viên khác có thể sử dụng. Bạn không cần đẩy module lên một dịch vụ tập trung vì công cụ Go có thể tải xuống module trực tiếp từ kho lưu trữ của bạn (xác định qua đường dẫn module, là một URL đã bỏ phần scheme) hoặc từ một proxy server.

Sau khi import package của bạn vào code, các lập trình viên dùng công cụ Go (bao gồm lệnh `go get`) để tải mã nguồn module về và biên dịch. Để hỗ trợ mô hình này, bạn cần tuân theo các quy ước và thực hành tốt nhất để công cụ Go (thay mặt lập trình viên khác) có thể truy xuất mã nguồn module từ kho lưu trữ của bạn. Chẳng hạn, công cụ Go dùng đường dẫn module bạn chỉ định, cùng với số phiên bản bạn dùng để gắn tag khi phát hành, để xác định vị trí và tải module về cho người dùng.

Để biết thêm về các quy ước và thực hành tốt nhất cho mã nguồn và xuất bản, xem [Quản lý mã nguồn module](/doc/modules/managing-source).

Để xem hướng dẫn từng bước về xuất bản module, xem [Xuất bản một module](publishing).

## Khám phá package {#discovery}

Sau khi bạn xuất bản module và ai đó tải nó về bằng công cụ Go, module sẽ xuất hiện trên trang khám phá package Go tại [pkg.go.dev](https://pkg.go.dev/). Tại đó, lập trình viên có thể tìm kiếm và đọc tài liệu của module.

Để bắt đầu sử dụng module, lập trình viên import các package từ module, sau đó chạy lệnh `go get` để tải mã nguồn về và biên dịch.

Để biết thêm về cách lập trình viên tìm và sử dụng module, xem [Quản lý dependency](managing-dependencies).

## Đánh số phiên bản {#versioning}

Khi bạn liên tục cải tiến module theo thời gian, bạn gán số phiên bản (dựa trên mô hình semantic versioning) nhằm thể hiện mức độ ổn định và khả năng tương thích ngược của từng phiên bản. Điều này giúp các lập trình viên dùng module xác định thời điểm module ổn định và liệu một lần nâng cấp có mang lại những thay đổi đáng kể hay không. Bạn chỉ định số phiên bản của module bằng cách gắn tag cho mã nguồn trong kho lưu trữ với số đó.

Để biết thêm về phát triển các bản cập nhật phiên bản chính, xem [Phát triển bản cập nhật phiên bản chính](major-version).

Để biết thêm về cách áp dụng mô hình semantic versioning cho module Go, xem [Đánh số phiên bản module](/doc/modules/version-numbers).
