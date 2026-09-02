---
title: Định dạng code của bạn bằng go fmt
date: 2013-01-23
by:
- Andrew Gerrand
tags:
- gofix
- gofmt
- technical
summary: Cách và lý do để định dạng code Go của bạn bằng gofmt.
---

## Giới thiệu

[Gofmt](/cmd/gofmt/) là công cụ tự động định dạng mã nguồn Go.

Code được định dạng bởi gofmt:

  - dễ **viết** hơn: không bao giờ phải lo lắng về các mối lo nhỏ liên quan đến định dạng khi code,

  - dễ **đọc** hơn: khi tất cả code trông giống nhau, bạn không cần chuyển đổi trong tâm trí
    phong cách định dạng của người khác thành thứ gì đó bạn có thể hiểu.

  - dễ **bảo trì** hơn: các thay đổi cơ học đối với nguồn không gây ra
    các thay đổi không liên quan đến định dạng file;
    diff chỉ hiển thị các thay đổi thực sự.

  - **không tranh cãi**: không bao giờ có cuộc tranh luận về khoảng trắng hay vị trí dấu ngoặc nhọn nữa!

## Định dạng code của bạn

Gần đây chúng tôi đã tiến hành khảo sát các gói Go trong thực tế và nhận thấy rằng
khoảng 70% trong số chúng được định dạng theo quy tắc của gofmt.
Điều này nhiều hơn dự kiến - và cảm ơn tất cả những người dùng gofmt - nhưng
sẽ tuyệt vời nếu có thể thu hẹp khoảng cách.

Để định dạng code của bạn, bạn có thể dùng trực tiếp công cụ gofmt:

	gofmt -w yourcode.go

Hoặc bạn có thể dùng lệnh "[go fmt](/cmd/go/#hdr-Gofmt__reformat__package_sources)":

	go fmt path/to/your/package

Để giúp giữ code của bạn trong phong cách chuẩn,
kho lưu trữ Go chứa các hook cho các trình soạn thảo và hệ thống quản lý phiên bản
giúp dễ dàng chạy gofmt trên code của bạn.

Đối với người dùng Vim, [Vim plugin cho Go](https://github.com/fatih/vim-go)
bao gồm lệnh :Fmt chạy gofmt trên buffer hiện tại.

Đối với người dùng emacs, [go-mode.el](https://github.com/dominikh/go-mode.el)
cung cấp hook gofmt-before-save có thể được cài đặt bằng cách thêm dòng này
vào file .emacs của bạn:

	(add-hook 'before-save-hook #'gofmt-before-save)

Đối với người dùng Eclipse hoặc Sublime Text, các dự án [GoClipse](https://github.com/GoClipse/goclipse)
và [GoSublime](https://github.com/DisposaBoy/GoSublime) thêm
một tính năng gofmt cho các trình soạn thảo đó.

Và đối với những người dùng Git, [script misc/git/pre-commit](https://github.com/golang/go/blob/release-branch.go1.1/misc/git/pre-commit)
là hook pre-commit ngăn code Go được định dạng không đúng bị commit.
Nếu bạn dùng Mercurial, [plugin hgstyle](https://bitbucket.org/fhs/hgstyle/overview)
cung cấp hook pre-commit gofmt.

## Biến đổi nguồn cơ học

Một trong những ưu điểm lớn nhất của code được định dạng bởi máy là nó có thể
được biến đổi cơ học mà không tạo ra nhiễu định dạng không liên quan trong diff.
Biến đổi cơ học là vô giá khi làm việc với các codebase lớn,
vì nó toàn diện hơn và ít lỗi hơn so với việc thực hiện các thay đổi rộng rãi bằng tay.
Thật vậy, khi làm việc ở quy mô lớn (như chúng tôi làm ở Google), thường không
thực tế để thực hiện những loại thay đổi này theo cách thủ công.

Cách dễ nhất để thao túng cơ học code Go là với cờ -r của gofmt.
Cờ này chỉ định một quy tắc viết lại có dạng

	pattern -> replacement

trong đó cả pattern và replacement đều là biểu thức Go hợp lệ.
Trong pattern, các định danh chữ thường ký tự đơn đóng vai trò là ký tự đại diện
khớp với các biểu thức con tùy ý,
và những biểu thức đó được thay thế cho các định danh giống nhau trong replacement.

Ví dụ, [thay đổi gần đây này](/cl/7038051) đối với
lõi Go đã viết lại một số cách dùng [bytes.Compare](/pkg/bytes/#Compare)
để dùng [bytes.Equal](/pkg/bytes/#Equal) hiệu quả hơn.
Người đóng góp đã thực hiện thay đổi chỉ bằng hai lần gọi gofmt:

	gofmt -r 'bytes.Compare(a, b) == 0 -> bytes.Equal(a, b)'
	gofmt -r 'bytes.Compare(a, b) != 0 -> !bytes.Equal(a, b)'

Gofmt cũng hỗ trợ [gofix](/cmd/fix/),
có thể thực hiện các biến đổi nguồn phức tạp tùy ý.
Gofix là công cụ vô giá trong những ngày đầu khi chúng tôi thường xuyên thực hiện
các thay đổi không tương thích với ngôn ngữ và thư viện.
Ví dụ, trước Go 1, interface lỗi tích hợp không tồn tại và
quy ước là dùng kiểu os.Error.
Khi chúng tôi [giới thiệu error](/doc/go1.html#errors),
chúng tôi đã cung cấp một module gofix viết lại tất cả các tham chiếu đến os.Error và các
hàm helper liên quan của nó để dùng error và [gói errors](/pkg/errors/) mới.
Sẽ rất khó khăn nếu cố gắng bằng tay,
nhưng với code ở định dạng chuẩn, việc chuẩn bị,
thực hiện và đánh giá thay đổi này chạm đến hầu hết tất cả code Go hiện có tương đối dễ dàng.

Để biết thêm về gofix, xem [bài viết này](/blog/introducing-gofix).
