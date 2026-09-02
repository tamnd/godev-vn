---
title: "Go Toolchains"
layout: article
---

## Giới thiệu {#intro}

Bắt đầu từ Go 1.21, bản phân phối Go bao gồm một lệnh `go` và một Go toolchain đi kèm,
là thư viện chuẩn cùng với trình biên dịch, trình hợp dịch và các công cụ khác.
Lệnh `go` có thể sử dụng Go toolchain đi kèm của nó cũng như các phiên bản khác
mà nó tìm thấy trong `PATH` cục bộ hoặc tải xuống khi cần.

Việc chọn Go toolchain nào được sử dụng phụ thuộc vào cài đặt `GOTOOLCHAIN` môi trường
và các dòng `go` và `toolchain` trong tệp `go.mod` của module chính hoặc tệp `go.work` của workspace hiện tại.
Khi bạn di chuyển giữa các module chính và workspace khác nhau,
phiên bản toolchain đang được sử dụng có thể thay đổi, giống như cách các phiên bản dependency module thay đổi.

Trong cấu hình chuẩn, lệnh `go` sử dụng Go toolchain đi kèm của nó
khi toolchain đó ít nhất mới bằng các dòng `go` hoặc `toolchain` trong module chính hoặc workspace.
Ví dụ, khi sử dụng lệnh `go` đi kèm với Go 1.21.3 trong một module chính có dòng `go 1.21.0`,
lệnh `go` sử dụng Go 1.21.3.
Khi dòng `go` hoặc `toolchain` mới hơn toolchain đi kèm,
lệnh `go` thay vào đó chạy toolchain mới hơn.
Ví dụ, khi sử dụng lệnh `go` đi kèm với Go 1.21.3 trong một module chính có dòng `go 1.21.9`,
lệnh `go` tìm và chạy Go 1.21.9 thay thế.
Nó trước tiên tìm kiếm trong PATH một chương trình có tên `go1.21.9` và nếu không tìm thấy thì tải xuống và cache
một bản sao của Go toolchain 1.21.9.
Việc chuyển đổi toolchain tự động này có thể bị tắt, nhưng trong trường hợp đó,
để tương thích tiến chính xác hơn,
lệnh `go` sẽ từ chối chạy trong module chính hoặc workspace mà dòng `go`
yêu cầu phiên bản Go mới hơn.
Nghĩa là, dòng `go` đặt phiên bản Go tối thiểu cần thiết để sử dụng một module hoặc workspace.

Các module là dependency của các module khác có thể cần đặt yêu cầu phiên bản Go tối thiểu
thấp hơn toolchain ưu tiên để dùng khi làm việc trực tiếp trong module đó.
Trong trường hợp này, dòng `toolchain` trong `go.mod` hoặc `go.work` đặt một toolchain ưu tiên
ưu tiên hơn dòng `go` khi lệnh `go` đang quyết định
toolchain nào sẽ sử dụng.

Các dòng `go` và `toolchain` có thể được hiểu là chỉ định các yêu cầu phiên bản
cho dependency của module trên Go toolchain, giống như các dòng `require` trong `go.mod`
chỉ định các yêu cầu phiên bản cho dependency trên các module khác.
Lệnh `go get` quản lý dependency Go toolchain giống như nó
quản lý dependency trên các module khác.
Ví dụ, `go get go@latest` cập nhật module để yêu cầu Go toolchain phát hành mới nhất.

Cài đặt môi trường `GOTOOLCHAIN` có thể ép buộc một phiên bản Go cụ thể, ghi đè
các dòng `go` và `toolchain`. Ví dụ, để kiểm thử gói với Go 1.21rc3:

	GOTOOLCHAIN=go1.21rc3 go test

Cài đặt `GOTOOLCHAIN` mặc định là `auto`, cho phép chuyển đổi toolchain được mô tả trước đó.
Dạng thay thế `<name>+auto` đặt toolchain mặc định để sử dụng trước khi quyết định có
chuyển đổi thêm hay không. Ví dụ `GOTOOLCHAIN=go1.21.3+auto` hướng dẫn lệnh `go`
bắt đầu quyết định với mặc định là sử dụng Go 1.21.3 nhưng vẫn sử dụng toolchain mới hơn nếu
được chỉ định bởi các dòng `go` và `toolchain`.
Vì cài đặt `GOTOOLCHAIN` mặc định có thể được thay đổi với `go env -w`,
nếu bạn đã cài đặt Go 1.21.0 trở lên, thì

	go env -w GOTOOLCHAIN=go1.21.3+auto

tương đương với việc thay thế cài đặt Go 1.21.0 của bạn bằng Go 1.21.3.

Phần còn lại của tài liệu này giải thích chi tiết hơn cách các Go toolchain được đánh phiên bản, chọn và quản lý.

## Các phiên bản Go {#version}

Các phiên bản phát hành của Go sử dụng cú pháp phiên bản '1.*N*.*P*', biểu thị phiên bản *P*th của Go 1.*N*.
Phiên bản phát hành ban đầu là 1.*N*.0, như trong '1.21.0'. Các phiên bản sau như 1.*N*.9 thường được gọi là phiên bản vá lỗi.

Các release candidate của Go 1.*N*, được phát hành trước 1.*N*.0, sử dụng cú pháp phiên bản '1.*N*rc*R*'.
Release candidate đầu tiên cho Go 1.*N* có phiên bản 1.*N*rc1, như trong `1.23rc1`.

Cú pháp '1.*N*' được gọi là "phiên bản ngôn ngữ". Nó biểu thị họ phiên bản Go tổng thể
triển khai phiên bản đó của ngôn ngữ Go và thư viện chuẩn.

Phiên bản ngôn ngữ cho một phiên bản Go là kết quả của việc cắt bỏ mọi thứ sau *N*:
1.21, 1.21rc2, và 1.21.3 đều triển khai phiên bản ngôn ngữ 1.21.

Các Go toolchain đã phát hành như Go 1.21.0 và Go 1.21rc1 báo cáo phiên bản cụ thể đó
(ví dụ: `go1.21.0` hoặc `go1.21rc1`)
từ `go version` và [`runtime.Version`](/pkg/runtime/#Version).
Các Go toolchain chưa phát hành (vẫn đang phát triển) được xây dựng từ kho phát triển Go
thay vào đó chỉ báo cáo phiên bản ngôn ngữ (ví dụ: `go1.21`).

Bất kỳ hai phiên bản Go nào cũng có thể được so sánh để quyết định phiên bản nào nhỏ hơn, lớn hơn,
hay bằng nhau. Nếu các phiên bản ngôn ngữ khác nhau, điều đó quyết định phép so sánh:
1.21.9 < 1.22. Trong một phiên bản ngôn ngữ, thứ tự từ nhỏ nhất đến lớn nhất là:
phiên bản ngôn ngữ chính nó, sau đó là các release candidate được sắp xếp theo *R*, rồi các bản phát hành được sắp xếp theo *P*.

Ví dụ, 1.21 < 1.21rc1 < 1.21rc2 < 1.21.0 < 1.21.1 < 1.21.2.

Trước Go 1.21, phiên bản phát hành ban đầu của một Go toolchain là phiên bản 1.*N*, không phải 1.*N*.0,
vì vậy đối với *N* < 21, thứ tự được điều chỉnh để đặt 1.*N* sau các release candidate.

Ví dụ, 1.20rc1 < 1.20rc2 < 1.20rc3 < 1.20 < 1.20.1.

Các phiên bản Go cũ hơn có các bản beta, với các phiên bản như 1.18beta2.
Các bản beta được đặt ngay trước các release candidate trong thứ tự phiên bản.

Ví dụ, 1.18beta1 < 1.18beta2 < 1.18rc1 < 1.18 < 1.18.1.

<!-- Unpublished note: the download page also lists Go 1.9.2rc2, which does not respect
this version syntax. That was created as a test of some potential release automation
before Go 1.9.2 but is not considered a "real" toolchain. -->

## Tên Go toolchain {#name}

Các Go toolchain tiêu chuẩn được đặt tên là <code>go<i>V</i></code> trong đó *V* là phiên bản Go
biểu thị bản beta, release candidate hoặc bản phát hành.
Ví dụ, `go1.21rc1` và `go1.21.0` là tên toolchain;
`go1.21` và `go1.22` thì không (các phiên bản phát hành ban đầu là `go1.21.0` và `go1.22.0`),
nhưng `go1.20` và `go1.19` thì có.

Các toolchain không tiêu chuẩn sử dụng tên có dạng <code>go<i>V</i>-<i>suffix</i></code>
cho bất kỳ hậu tố nào.

Các toolchain được so sánh bằng cách so sánh phiên bản <code><i>V</i></code> được nhúng trong tên
(bỏ `go` ban đầu và loại bỏ bất kỳ hậu tố nào bắt đầu bằng `-`).
Ví dụ, `go1.21.0` và `go1.21.0-custom` so sánh bằng nhau cho mục đích sắp xếp.

## Cấu hình module và workspace {#config}

Các Go module và workspace chỉ định cấu hình liên quan đến phiên bản
trong các tệp `go.mod` hoặc `go.work` của chúng.

Dòng `go` khai báo phiên bản Go tối thiểu cần thiết để sử dụng
module hoặc workspace.
Vì lý do tương thích, nếu dòng `go` bị bỏ qua khỏi tệp `go.mod`,
module được coi là có dòng `go 1.16` ngầm định,
và nếu dòng `go` bị bỏ qua khỏi tệp `go.work`,
workspace được coi là có dòng `go 1.18` ngầm định.

Dòng `toolchain` khai báo toolchain đề xuất để sử dụng với
module hoặc workspace.
Như được mô tả trong "[Chọn Go toolchain](#select)" bên dưới,
lệnh `go` có thể chạy toolchain cụ thể này khi hoạt động
trong module hoặc workspace đó
nếu phiên bản toolchain mặc định nhỏ hơn phiên bản toolchain đề xuất.
Nếu dòng `toolchain` bị bỏ qua,
module hoặc workspace được coi là có dòng
<code>toolchain go<i>V</i></code> ngầm định,
trong đó *V* là phiên bản Go từ dòng `go`.

Ví dụ, một `go.mod` có dòng `go 1.21.0` không có dòng `toolchain`
được hiểu như thể nó có dòng `toolchain go1.21.0`.

Go toolchain từ chối tải module hoặc workspace khai báo
phiên bản Go tối thiểu cần thiết lớn hơn phiên bản của toolchain.

Ví dụ, Go 1.21.2 sẽ từ chối tải module hoặc workspace
có dòng `go 1.21.3` hoặc `go 1.22`.

Dòng `go` của module phải khai báo phiên bản lớn hơn hoặc bằng
phiên bản `go` được khai báo bởi mỗi module được liệt kê trong các câu lệnh `require`.
Dòng `go` của workspace phải khai báo phiên bản lớn hơn hoặc bằng
phiên bản `go` được khai báo bởi mỗi module được liệt kê trong các câu lệnh `use`.

Ví dụ, nếu module *M* yêu cầu dependency *D* với `go.mod`
khai báo `go 1.22.0`, thì `go.mod` của *M* không thể có dòng `go 1.21.3`.

Dòng `go` cho mỗi module đặt phiên bản ngôn ngữ mà trình biên dịch
thực thi khi biên dịch các gói trong module đó.
Phiên bản ngôn ngữ có thể được thay đổi trên cơ sở từng tệp bằng cách sử dụng
[ràng buộc xây dựng](/cmd/go#hdr-Build_constraints):
nếu ràng buộc xây dựng hiện diện và ngụ ý phiên bản tối thiểu ít nhất là `go1.21`,
phiên bản ngôn ngữ được sử dụng khi biên dịch tệp đó sẽ là phiên bản tối thiểu đó.

Ví dụ, một module chứa mã sử dụng phiên bản ngôn ngữ Go 1.21
nên có tệp `go.mod` với dòng `go` như `go 1.21` hoặc `go 1.21.3`.
Nếu một tệp nguồn cụ thể chỉ nên được biên dịch khi sử dụng Go toolchain mới hơn,
việc thêm `//go:build go1.22` vào tệp nguồn đó vừa đảm bảo rằng chỉ có Go 1.22 và
các toolchain mới hơn mới biên dịch tệp vừa thay đổi phiên bản ngôn ngữ trong tệp đó thành Go 1.22.

Các dòng `go` và `toolchain` được sửa đổi thuận tiện và an toàn nhất
bằng cách sử dụng `go get`; xem phần [dành riêng cho `go get` bên dưới](#get).

Trước Go 1.21, các Go toolchain coi dòng `go` như một yêu cầu tư vấn:
nếu các bản build thành công, toolchain giả định mọi thứ hoạt động,
và nếu không nó in một ghi chú về sự không khớp phiên bản tiềm năng.
Go 1.21 đã thay đổi dòng `go` thành yêu cầu bắt buộc.
Hành vi này được backport một phần cho các phiên bản ngôn ngữ cũ hơn:
Các bản phát hành Go 1.19 bắt đầu từ Go 1.19.13 và các bản phát hành Go 1.20 bắt đầu từ Go 1.20.8,
từ chối tải workspace hoặc module khai báo phiên bản Go 1.22 trở lên.

Trước Go 1.21, các toolchain không yêu cầu module
hoặc workspace phải có dòng `go` lớn hơn hoặc bằng
phiên bản `go` được yêu cầu bởi mỗi module dependency của nó.

## Cài đặt `GOTOOLCHAIN` {#GOTOOLCHAIN}

Lệnh `go` chọn Go toolchain để sử dụng dựa trên cài đặt `GOTOOLCHAIN`.
Để tìm cài đặt `GOTOOLCHAIN`, lệnh `go` sử dụng các quy tắc tiêu chuẩn cho bất kỳ
cài đặt môi trường Go nào:

 - Nếu `GOTOOLCHAIN` được đặt thành giá trị không rỗng trong môi trường tiến trình
   (như được truy vấn bởi [`os.Getenv`](/pkg/os/#Getenv)), lệnh `go` sử dụng giá trị đó.

 - Ngược lại, nếu `GOTOOLCHAIN` được đặt trong tệp mặc định môi trường người dùng
   (được quản lý bởi
   [`go env -w` và `go env -u`](/cmd/go/#hdr-Print_Go_environment_information)),
   lệnh `go` sử dụng giá trị đó.

 - Ngược lại, nếu `GOTOOLCHAIN` được đặt trong tệp mặc định môi trường của Go toolchain đi kèm
   (`$GOROOT/go.env`), lệnh `go` sử dụng giá trị đó.

Trong các Go toolchain tiêu chuẩn, tệp `$GOROOT/go.env` đặt `GOTOOLCHAIN=auto` mặc định,
nhưng các Go toolchain được đóng gói lại có thể thay đổi giá trị này.

Nếu tệp `$GOROOT/go.env` bị thiếu hoặc không đặt mặc định, lệnh `go`
giả định `GOTOOLCHAIN=local`.

Chạy `go env GOTOOLCHAIN` in cài đặt `GOTOOLCHAIN`.

## Chọn Go toolchain {#select}

Khi khởi động, lệnh `go` chọn Go toolchain nào sẽ sử dụng.
Nó tham khảo cài đặt `GOTOOLCHAIN`,
có dạng `<name>`, `<name>+auto`, hoặc `<name>+path`.
`GOTOOLCHAIN=auto` là viết tắt của `GOTOOLCHAIN=local+auto`;
tương tự, `GOTOOLCHAIN=path` là viết tắt của `GOTOOLCHAIN=local+path`.
`<name>` đặt Go toolchain mặc định:
`local` chỉ Go toolchain đi kèm
(cái đi kèm với lệnh `go` đang được chạy), và ngược lại `<name>` phải
là tên Go toolchain cụ thể, như `go1.21.0`.
Lệnh `go` ưu tiên chạy Go toolchain mặc định.
Như đã lưu ý ở trên, bắt đầu từ Go 1.21, các Go toolchain từ chối chạy trong
workspace hoặc module yêu cầu các phiên bản Go mới hơn.
Thay vào đó, chúng báo cáo lỗi và thoát.

Khi `GOTOOLCHAIN` được đặt thành `local`, lệnh `go` luôn chạy Go toolchain đi kèm.

Khi `GOTOOLCHAIN` được đặt thành `<name>` (ví dụ: `GOTOOLCHAIN=go1.21.0`),
lệnh `go` luôn chạy Go toolchain cụ thể đó.
Nếu một binary có tên đó được tìm thấy trong PATH hệ thống, lệnh `go` sử dụng nó.
Ngược lại, lệnh `go` sử dụng Go toolchain mà nó tải xuống và xác minh.

Khi `GOTOOLCHAIN` được đặt thành `<name>+auto` hoặc `<name>+path` (hoặc viết tắt `auto` hoặc `path`),
lệnh `go` chọn và chạy phiên bản Go mới hơn khi cần.
Cụ thể, nó tham khảo các dòng `toolchain` và `go` trong tệp `go.work` của workspace hiện tại,
hoặc khi không có workspace,
tệp `go.mod` của module chính.
Nếu tệp `go.work` hoặc `go.mod` có dòng `toolchain <tname>`
và `<tname>` mới hơn Go toolchain mặc định,
thì lệnh `go` chạy `<tname>` thay thế.
Nếu tệp có dòng `toolchain default`,
thì lệnh `go` chạy Go toolchain mặc định,
vô hiệu hóa bất kỳ nỗ lực cập nhật nào ngoài `<name>`.
Ngược lại, nếu tệp có dòng `go <version>`
và `<version>` mới hơn Go toolchain mặc định,
thì lệnh `go` chạy `go<version>` thay thế.

Để chạy toolchain khác với Go toolchain đi kèm,
lệnh `go` tìm kiếm trong đường dẫn thực thi của tiến trình
(`$PATH` trên Unix và Plan 9, `%PATH%` trên Windows)
cho một chương trình có tên cho trước (ví dụ: `go1.21.3`) và chạy chương trình đó.
Nếu không tìm thấy chương trình như vậy, lệnh `go`
[tải xuống và chạy Go toolchain được chỉ định](#download).
Sử dụng dạng `GOTOOLCHAIN` `<name>+path` tắt dự phòng tải xuống,
khiến lệnh `go` dừng lại sau khi tìm kiếm trong đường dẫn thực thi.

Chạy `go version` in phiên bản của Go toolchain được chọn
(bằng cách chạy triển khai `go version` của toolchain được chọn).

Chạy `GOTOOLCHAIN=local go version` in phiên bản của Go toolchain đi kèm.

Bắt đầu từ Go 1.24, bạn có thể theo dõi quá trình chọn toolchain của lệnh `go`
bằng cách thêm `toolchaintrace=1` vào biến môi trường `GODEBUG` khi bạn chạy lệnh `go`.

## Chuyển đổi Go toolchain {#switch}

Đối với hầu hết các lệnh, tệp `go.work` của workspace hoặc `go.mod` của module chính
sẽ có dòng `go` ít nhất mới bằng dòng `go` trong bất kỳ dependency module nào,
do các [yêu cầu cấu hình](#config) về thứ tự phiên bản.
Trong trường hợp này, việc chọn toolchain khi khởi động chạy Go toolchain đủ mới
để hoàn thành lệnh.

Một số lệnh kết hợp các phiên bản module mới như một phần của hoạt động của chúng:
`go get` thêm các dependency module mới vào module chính;
`go work use` thêm các module cục bộ mới vào workspace;
`go work sync` đồng bộ lại workspace với các module cục bộ có thể đã được cập nhật
kể từ khi workspace được tạo;
`go install package@version` và `go run package@version`
thực tế chạy trong một module chính rỗng và thêm `package@version` như một dependency mới.
Tất cả các lệnh này có thể gặp module có dòng `go` trong `go.mod` yêu cầu phiên bản Go mới hơn phiên bản Go đang thực thi hiện tại.

Khi một lệnh gặp module yêu cầu phiên bản Go mới hơn
và `GOTOOLCHAIN` cho phép chạy các toolchain khác nhau
(nó là một trong các dạng `auto` hoặc `path`),
lệnh `go` chọn và chuyển đổi sang toolchain mới hơn phù hợp
để tiếp tục thực thi lệnh hiện tại.

Bất kỳ lúc nào lệnh `go` chuyển đổi toolchain sau khi chọn toolchain khi khởi động,
nó in một thông báo giải thích lý do. Ví dụ:

	go: module example.com/widget@v1.2.3 requires go >= 1.24rc1; switching to go 1.27.9

Như được hiển thị trong ví dụ, lệnh `go` có thể chuyển sang toolchain
mới hơn yêu cầu được phát hiện.
Nói chung, lệnh `go` nhằm chuyển sang Go toolchain được hỗ trợ.

Để chọn toolchain, lệnh `go` trước tiên lấy danh sách các toolchain có sẵn.
Đối với dạng `auto`, lệnh `go` tải xuống danh sách các toolchain có sẵn.
Đối với dạng `path`, lệnh `go` quét PATH để tìm bất kỳ tệp thực thi nào
được đặt tên cho các toolchain hợp lệ và sử dụng danh sách tất cả toolchain tìm thấy.
Sử dụng danh sách toolchain đó, lệnh `go` xác định tối đa ba ứng viên:

 - release candidate mới nhất của phiên bản ngôn ngữ Go chưa phát hành (1.*N*₃rc*R*₃),
 - bản vá lỗi mới nhất của phiên bản ngôn ngữ Go được phát hành gần đây nhất (1.*N*₂.*P*₂), và
 - bản vá lỗi mới nhất của phiên bản ngôn ngữ Go trước đó (1.*N*₁.*P*₁).

Đây là các bản phát hành Go được hỗ trợ theo [chính sách phát hành](/doc/devel/release#policy) của Go.
Nhất quán với [chọn phiên bản tối thiểu](https://research.swtch.com/vgo-mvs),
lệnh `go` sau đó sử dụng một cách bảo thủ ứng viên có phiên bản _tối thiểu_ (cũ nhất)
thỏa mãn yêu cầu mới.

Ví dụ, giả sử `example.com/widget@v1.2.3` yêu cầu Go 1.24rc1 trở lên.
Lệnh `go` lấy danh sách các toolchain có sẵn
và phát hiện rằng các bản vá lỗi mới nhất của hai Go toolchain gần đây nhất là
Go 1.28.3 và Go 1.27.9,
và release candidate Go 1.29rc2 cũng có sẵn.
Trong tình huống này, lệnh `go` sẽ chọn Go 1.27.9.
Nếu `widget` yêu cầu Go 1.28 trở lên, lệnh `go` sẽ chọn Go 1.28.3,
vì Go 1.27.9 quá cũ.
Nếu `widget` yêu cầu Go 1.29 trở lên, lệnh `go` sẽ chọn Go 1.29rc2,
vì cả Go 1.27.9 và Go 1.28.3 đều quá cũ.

Các lệnh kết hợp các phiên bản module mới yêu cầu các phiên bản Go mới
ghi yêu cầu phiên bản `go` tối thiểu mới vào tệp `go.work` của workspace hiện tại
hoặc tệp `go.mod` của module chính, cập nhật dòng `go`.
Để [có tính lặp lại](https://research.swtch.com/vgo-principles#repeatability),
bất kỳ lệnh nào cập nhật dòng `go` cũng cập nhật dòng `toolchain`
để ghi lại tên toolchain của nó.
Lần tiếp theo lệnh `go` chạy trong workspace hoặc module đó,
nó sẽ sử dụng dòng `toolchain` đã cập nhật đó trong [chọn toolchain](#select).

Ví dụ, `go get example.com/widget@v1.2.3` có thể in thông báo chuyển đổi
như trên và chuyển sang Go 1.27.9.
Go 1.27.9 sẽ hoàn thành `go get` và cập nhật dòng `toolchain`
để dòng đó là `toolchain go1.27.9`.
Lệnh `go` tiếp theo chạy trong module hoặc workspace đó sẽ chọn `go1.27.9`
khi khởi động và sẽ không in bất kỳ thông báo chuyển đổi nào.

Nói chung, nếu bất kỳ lệnh `go` nào được chạy hai lần, nếu lần đầu in thông báo chuyển đổi,
lần thứ hai sẽ không, vì lần đầu cũng đã cập nhật `go.work` hoặc `go.mod`
để chọn toolchain đúng khi khởi động.
Ngoại lệ là các dạng `go install package@version` và `go run package@version`,
chạy trong không có workspace hoặc module chính và không thể ghi dòng `toolchain`.
Chúng in thông báo chuyển đổi mỗi lần cần chuyển sang toolchain mới hơn.

## Tải xuống toolchain {#download}

Khi sử dụng `GOTOOLCHAIN=auto` hoặc `GOTOOLCHAIN=<name>+auto`, lệnh Go
tải xuống các toolchain mới hơn khi cần.
Các toolchain này được đóng gói như các module đặc biệt
với đường dẫn module `golang.org/toolchain`
và phiên bản <code>v0.0.1-go<i>VERSION</i>.<i>GOOS</i>-<i>GOARCH</i></code>.
Các toolchain được tải xuống như bất kỳ module nào khác,
có nghĩa là việc tải xuống toolchain có thể được proxy bằng cách đặt `GOPROXY`
và có checksum của chúng được kiểm tra bởi Go checksum database.
Vì toolchain cụ thể được sử dụng phụ thuộc vào
toolchain mặc định của hệ thống cũng như hệ điều hành và kiến trúc cục bộ (GOOS và GOARCH),
việc ghi checksum module toolchain vào `go.sum` là không thực tế.
Thay vào đó, việc tải xuống toolchain thất bại do thiếu xác minh nếu `GOSUMDB=off`.
Các mẫu `GOPRIVATE` và `GONOSUMDB` không áp dụng cho việc tải xuống toolchain.

## Quản lý yêu cầu phiên bản module Go với `go get` {#get}

Nói chung, lệnh `go` coi các dòng `go` và `toolchain`
như khai báo các dependency toolchain phiên bản cho module chính.
Lệnh `go get` có thể quản lý các dòng này giống như nó quản lý
các dòng `require` chỉ định các dependency module phiên bản.

Ví dụ, `go get go@1.22.1 toolchain@1.24rc1` thay đổi tệp `go.mod` của module chính
để đọc `go 1.22.1` và `toolchain go1.24rc1`.

Lệnh `go` hiểu rằng dependency `go` yêu cầu dependency `toolchain`
với phiên bản Go lớn hơn hoặc bằng.

Tiếp tục ví dụ, `go get go@1.25.0` sau đó sẽ cập nhật
toolchain thành `go1.25.0` cũng.
Khi toolchain khớp chính xác với dòng `go`, nó có thể bị
bỏ qua và suy ra, vì vậy `go get` này sẽ xóa dòng `toolchain`.

Yêu cầu tương tự cũng áp dụng ngược lại khi hạ cấp:
nếu `go.mod` bắt đầu với `go 1.22.1` và `toolchain go1.24rc1`,
thì `go get toolchain@go1.22.9` sẽ chỉ cập nhật dòng `toolchain`,
nhưng `go get toolchain@go1.21.3` sẽ hạ cấp dòng `go` xuống
`go 1.21.3` cũng.
Kết quả sẽ là chỉ còn `go 1.21.3` không có dòng `toolchain`.

Dạng đặc biệt `toolchain@none` có nghĩa là xóa bất kỳ dòng `toolchain` nào,
như trong `go get toolchain@none` hoặc `go get go@1.25.0 toolchain@none`.

Lệnh `go` hiểu cú pháp phiên bản cho
các dependency `go` và `toolchain` cũng như các truy vấn.

Ví dụ, giống như `go get example.com/widget@v1.2` sử dụng
phiên bản `v1.2` mới nhất của `example.com/widget` (có thể là `v1.2.3`),
`go get go@1.22` sử dụng bản phát hành có sẵn mới nhất của phiên bản ngôn ngữ Go 1.22
(có thể là `1.22rc3`, hoặc có thể là `1.22.3`).
Điều tương tự áp dụng cho `go get toolchain@go1.22`.

Các lệnh `go get` và `go mod tidy` duy trì dòng `go` để
lớn hơn hoặc bằng dòng `go` của bất kỳ module dependency bắt buộc nào.

Ví dụ, nếu module chính có `go 1.22.1` và chúng ta chạy
`go get example.com/widget@v1.2.3` khai báo `go 1.24rc1`,
thì `go get` sẽ cập nhật dòng `go` của module chính thành `go 1.24rc1`.

Tiếp tục ví dụ, `go get go@1.22.1` sau đó sẽ
hạ cấp `example.com/widget` về phiên bản tương thích với Go 1.22.1
hoặc loại bỏ yêu cầu hoàn toàn,
giống như khi hạ cấp bất kỳ dependency khác của `example.com/widget`.

Trước Go 1.21, cách đề xuất để cập nhật module lên phiên bản Go mới (ví dụ: Go 1.22)
là `go mod tidy -go=1.22`, để đảm bảo rằng bất kỳ điều chỉnh nào
cụ thể cho Go 1.22 được thực hiện trong `go.mod` cùng lúc dòng `go` được cập nhật.
Dạng đó vẫn hợp lệ, nhưng `go get go@1.22` đơn giản hơn hiện được ưu tiên.

Khi `go get` được chạy trong module trong thư mục nằm trong workspace root,
`go get` hầu như bỏ qua workspace,
nhưng nó cập nhật tệp `go.work` để nâng cấp dòng `go`
khi workspace sẽ bị để lại với dòng `go` quá cũ.

## Quản lý yêu cầu phiên bản workspace Go với `go work` {#work}

Như đã lưu ý trong phần trước, `go get` được chạy trong thư mục
trong workspace root sẽ lo việc cập nhật dòng `go` của tệp `go.work`
khi cần để lớn hơn hoặc bằng bất kỳ module nào trong root đó.
Tuy nhiên, workspace cũng có thể tham chiếu đến các module bên ngoài thư mục root;
việc chạy `go get` trong những thư mục đó có thể dẫn đến cấu hình workspace không hợp lệ,
trong đó phiên bản `go` được khai báo trong `go.work` nhỏ hơn
một hoặc nhiều module trong các chỉ thị `use`.

Lệnh `go work use`, thêm các chỉ thị `use` mới, cũng kiểm tra
rằng phiên bản `go` trong tệp `go.work` đủ mới cho tất cả các
chỉ thị `use` hiện có.
Để cập nhật workspace đã bị lệch phiên bản `go` so với các module của nó,
hãy chạy `go work use` không có đối số.

Các lệnh `go work init` và `go work sync` cũng cập nhật phiên bản `go`
khi cần.

Để xóa dòng `toolchain` khỏi tệp `go.work`, hãy dùng
`go work edit -toolchain=none`.
