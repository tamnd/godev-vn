---
title: Thay đổi module mới trong Go 1.16
date: 2021-02-18
by:
- Jay Conrod
tags:
- modules
- versioning
summary: Go 1.16 bật module theo mặc định, cung cấp cách mới để cài đặt executable, và cho phép tác giả module thu hồi các phiên bản đã xuất bản.
---


Hy vọng bạn đang thích Go 1.16!
Bản phát hành này có nhiều tính năng mới, đặc biệt là về module.
[Ghi chú phát hành](/doc/go1.16) mô tả ngắn gọn các thay đổi này, nhưng hãy khám phá một số trong số chúng sâu hơn.

## Module bật mặc định

Lệnh `go` giờ build các gói trong chế độ module-aware theo mặc định, ngay cả khi không có `go.mod`.
Đây là bước lớn hướng tới việc sử dụng module trong tất cả các dự án.

Vẫn có thể build các gói trong chế độ GOPATH bằng cách đặt biến môi trường `GO111MODULE` thành `off`.
Bạn cũng có thể đặt `GO111MODULE` thành `auto` để bật chế độ module-aware chỉ khi có file go.mod trong thư mục hiện tại hoặc bất kỳ thư mục cha nào.
Đây là mặc định trước đây.
Lưu ý rằng bạn có thể đặt `GO111MODULE` và các biến khác vĩnh viễn bằng `go env -w`:

    go env -w GO111MODULE=auto

Chúng tôi có kế hoạch bỏ hỗ trợ chế độ GOPATH trong Go 1.17.
Nói cách khác, Go 1.17 sẽ bỏ qua `GO111MODULE`.
Nếu bạn có các dự án không build trong chế độ module-aware, đây là lúc để di chuyển.
Nếu có vấn đề ngăn bạn di chuyển, hãy xem xét tạo một [issue](/issue/new) hoặc [báo cáo trải nghiệm](/wiki/ExperienceReports).

## Không tự động thay đổi go.mod và go.sum

Trước đây, khi lệnh `go` tìm thấy vấn đề với `go.mod` hoặc `go.sum` như thiếu chỉ thị `require` hoặc thiếu sum, nó sẽ cố gắng tự động sửa vấn đề.
Chúng tôi nhận được nhiều phản hồi rằng hành vi này gây ngạc nhiên, đặc biệt với các lệnh như `go list` thường không có tác dụng phụ.
Các sửa lỗi tự động không phải lúc nào cũng mong muốn: nếu một gói được import không được cung cấp bởi bất kỳ module nào bắt buộc, lệnh `go` sẽ thêm một dependency mới, có thể kích hoạt nâng cấp các dependency chung.
Thậm chí một đường dẫn import bị viết sai cũng dẫn đến tra cứu mạng (thất bại).

Trong Go 1.16, các lệnh module-aware báo lỗi sau khi phát hiện vấn đề trong `go.mod` hoặc `go.sum` thay vì cố gắng tự động sửa vấn đề.
Trong hầu hết các trường hợp, thông báo lỗi khuyến nghị một lệnh để sửa vấn đề.


    $ go build
    example.go:3:8: no required module provides package golang.org/x/net/html; to add it:
        go get golang.org/x/net/html
    $ go get golang.org/x/net/html
    $ go build

Như trước, lệnh `go` có thể dùng thư mục `vendor` nếu nó tồn tại (xem [Vendoring](/ref/mod#vendoring) để biết chi tiết).
Các lệnh như `go get` và `go mod tidy` vẫn sửa đổi `go.mod` và `go.sum`, vì mục đích chính của chúng là quản lý dependency.

## Cài đặt executable ở phiên bản cụ thể

Lệnh `go install` giờ có thể cài đặt executable ở phiên bản cụ thể bằng cách chỉ định hậu tố `@version`.

    go install golang.org/x/tools/gopls@v0.6.5

Khi dùng cú pháp này, `go install` cài đặt lệnh từ phiên bản module chính xác đó, bỏ qua bất kỳ file `go.mod` nào trong thư mục hiện tại và thư mục cha.
(Không có hậu tố `@version`, `go install` tiếp tục hoạt động như trước, build chương trình bằng các yêu cầu phiên bản và replacement được liệt kê trong `go.mod` của module hiện tại.)

Chúng tôi từng khuyến nghị `go get -u program` để cài đặt executable, nhưng cách dùng này gây quá nhiều nhầm lẫn với ý nghĩa của `go get` để thêm hoặc thay đổi yêu cầu phiên bản module trong `go.mod`.
Và để tránh vô tình sửa đổi `go.mod`, mọi người bắt đầu đề xuất các lệnh phức tạp hơn như:

    cd $HOME; GO111MODULE=on go get program@latest

Giờ tất cả chúng ta có thể dùng `go install program@latest` thay thế.
Xem [`go install`](/ref/mod#go-install) để biết chi tiết.

Để loại bỏ sự mơ hồ về phiên bản nào được sử dụng, có một số hạn chế về những chỉ thị nào có thể có trong file `go.mod` của chương trình khi dùng cú pháp cài đặt này.
Đặc biệt, các chỉ thị `replace` và `exclude` không được phép, ít nhất là hiện tại.
Về lâu dài, khi `go install program@version` mới hoạt động tốt cho đủ các trường hợp sử dụng, chúng tôi có kế hoạch làm cho `go get` ngừng cài đặt binary lệnh.
Xem [issue 43684](/issue/43684) để biết chi tiết.

## Thu hồi module

Bạn đã bao giờ vô tình xuất bản một phiên bản module trước khi nó sẵn sàng chưa?
Hoặc bạn đã phát hiện ra vấn đề ngay sau khi một phiên bản được xuất bản cần được sửa nhanh chóng?
Lỗi trong các phiên bản đã xuất bản rất khó sửa.
Để giữ cho các build module xác định, một phiên bản không thể được sửa đổi sau khi xuất bản.
Ngay cả khi bạn xóa hoặc thay đổi tag phiên bản, [`proxy.golang.org`](https://proxy.golang.org) và các proxy khác có thể đã lưu trữ bản gốc.

Tác giả module giờ có thể *thu hồi* các phiên bản module bằng chỉ thị `retract` trong `go.mod`.
Phiên bản bị thu hồi vẫn tồn tại và có thể được tải xuống (vì vậy các build phụ thuộc vào nó sẽ không bị hỏng), nhưng lệnh `go` sẽ không tự động chọn nó khi giải quyết phiên bản như `@latest`.
`go get` và `go list -m -u` sẽ in cảnh báo về các cách dùng hiện có.

Ví dụ, giả sử tác giả của một thư viện phổ biến `example.com/lib` phát hành `v1.0.5`, rồi phát hiện ra một vấn đề bảo mật mới.
Họ có thể thêm một chỉ thị vào file `go.mod` của họ như sau:

    // Remote-triggered crash in package foo. See CVE-2021-01234.
    retract v1.0.5


Tiếp theo, tác giả có thể tag và push phiên bản `v1.0.6`, phiên bản cao nhất mới.
Sau đó, người dùng đã phụ thuộc vào `v1.0.5` sẽ được thông báo về việc thu hồi khi họ kiểm tra cập nhật hoặc khi họ nâng cấp một gói phụ thuộc.
Thông báo có thể bao gồm văn bản từ comment trên chỉ thị `retract`.

    $ go list -m -u all
    example.com/lib v1.0.0 (retracted)
    $ go get .
    go: warning: example.com/lib@v1.0.5: retracted by module author:
        Remote-triggered crash in package foo. See CVE-2021-01234.
    go: to switch to the latest unretracted version, run:
        go get example.com/lib@latest

Để có hướng dẫn tương tác dựa trên trình duyệt, hãy xem [Retract Module Versions](https://play-with-go.dev/retract-module-versions_go116_en/) trên [play-with-go.dev](https://play-with-go.dev/).
Xem [tài liệu chỉ thị `retract`](/ref/mod#go-mod-file-retract) để biết chi tiết cú pháp.

## Kiểm soát công cụ quản lý phiên bản với GOVCS

Lệnh `go` có thể tải xuống mã nguồn module từ một mirror như [proxy.golang.org](https://proxy.golang.org) hoặc trực tiếp từ kho lưu trữ quản lý phiên bản bằng `git`, `hg`, `svn`, `bzr`, hoặc `fossil`.
Quyền truy cập quản lý phiên bản trực tiếp rất quan trọng, đặc biệt cho các module riêng tư không có sẵn trên proxy, nhưng cũng là vấn đề bảo mật tiềm năng: một lỗi trong công cụ quản lý phiên bản có thể bị khai thác bởi máy chủ độc hại để chạy code không mong muốn.

Go 1.16 giới thiệu một biến cấu hình mới, `GOVCS`, cho phép người dùng chỉ định module nào được phép dùng công cụ quản lý phiên bản cụ thể.
`GOVCS` nhận danh sách các quy tắc `pattern:vcslist` phân tách bằng dấu phẩy.
`pattern` là một pattern [`path.Match`](/pkg/path#Match) khớp với một hoặc nhiều phần tử hàng đầu của đường dẫn module.
Các pattern đặc biệt `public` và `private` khớp với module công khai và riêng tư (`private` được định nghĩa là module khớp với các pattern trong `GOPRIVATE`; `public` là tất cả những thứ còn lại).
`vcslist` là danh sách các lệnh quản lý phiên bản được phép phân tách bằng ký tự pipe, hoặc từ khóa `all` hoặc `off`.

Ví dụ:

    GOVCS=github.com:git,evil.com:off,*:git|hg

Với cài đặt này, module có đường dẫn trên `github.com` có thể được tải xuống bằng `git`; đường dẫn trên `evil.com` không thể được tải xuống bằng bất kỳ lệnh quản lý phiên bản nào, và tất cả các đường dẫn khác (`*` khớp với tất cả) có thể được tải xuống bằng `git` hoặc `hg`.

Nếu `GOVCS` không được đặt, hoặc nếu một module không khớp với bất kỳ pattern nào, lệnh `go` dùng mặc định này: `git` và `hg` được phép cho module công khai, và tất cả công cụ đều được phép cho module riêng tư.
Lý do chỉ cho phép Git và Mercurial là hai hệ thống này đã được chú ý nhiều nhất đến các vấn đề về chạy như client của máy chủ không đáng tin cậy.
Ngược lại, Bazaar, Fossil và Subversion chủ yếu được dùng trong môi trường tin cậy, được xác thực và không được xem xét kỹ lưỡng như các bề mặt tấn công.
Tức là, cài đặt mặc định là:

    GOVCS=public:git|hg,private:all

Xem [Kiểm soát công cụ quản lý phiên bản với `GOVCS`](/ref/mod#vcs-govcs) để biết thêm chi tiết.

## Tiếp theo là gì?

Hy vọng bạn thấy những tính năng này hữu ích. Chúng tôi đã làm việc chăm chỉ cho bộ tính năng module tiếp theo cho Go 1.17, đặc biệt là [lazy module loading](/issue/36460), sẽ làm cho quá trình tải module nhanh hơn và ổn định hơn.
Như thường lệ, nếu bạn gặp lỗi mới, hãy cho chúng tôi biết trên [issue tracker](https://github.com/golang/go/issues). Chúc code vui!
