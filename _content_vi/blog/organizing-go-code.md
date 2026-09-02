---
title: Tổ chức code Go
date: 2012-08-16
by:
- Andrew Gerrand
tags:
- godoc
- gopath
- interface
- libraries
- tools
- technical
summary: Cách đặt tên và đóng gói các phần của chương trình Go để phục vụ người dùng tốt nhất.
---

## Giới thiệu

Code Go được tổ chức khác với các ngôn ngữ khác.
Bài đăng này thảo luận về cách đặt tên và đóng gói các phần tử của chương trình Go
để phục vụ người dùng tốt nhất.

## Chọn tên tốt

Tên bạn chọn ảnh hưởng đến cách bạn nghĩ về code của mình,
vì vậy hãy cẩn thận khi đặt tên gói và các định danh được xuất của nó.

Tên gói cung cấp ngữ cảnh cho nội dung của nó.
Ví dụ, gói [bytes](/pkg/bytes/) từ
thư viện chuẩn xuất kiểu `Buffer`.
Bản thân tên `Buffer` không mang tính mô tả lắm,
nhưng khi kết hợp với tên gói của nó, ý nghĩa của nó trở nên rõ ràng: `bytes.Buffer`.
Nếu gói có tên ít mô tả hơn,
như `util`, bộ đệm có thể sẽ có tên dài hơn và vụng về hơn là `util.BytesBuffer`.

Đừng ngại đổi tên mọi thứ khi bạn làm việc.
Khi bạn dành thời gian với chương trình của mình, bạn sẽ hiểu rõ hơn cách các phần của nó kết hợp với nhau và,
do đó, tên của chúng nên là gì.
Không cần phải khóa mình vào các quyết định ban đầu.
(Lệnh [gofmt](/cmd/gofmt/) có cờ `-r`
cung cấp tìm kiếm và thay thế có nhận thức về cú pháp,
giúp tái cấu trúc quy mô lớn dễ dàng hơn.)

Một cái tên tốt là phần quan trọng nhất của giao diện phần mềm:
tên là thứ đầu tiên mà mỗi client của code sẽ thấy.
Do đó, một tên được chọn tốt là điểm khởi đầu cho tài liệu tốt.
Nhiều thực hành sau đây xuất hiện một cách tự nhiên từ việc đặt tên tốt.

## Chọn đường dẫn import tốt (làm cho gói của bạn có thể "go get")

Đường dẫn import là chuỗi mà người dùng dùng để import một gói.
Nó chỉ định thư mục (tương đối với `$GOROOT/src/pkg` hoặc `$GOPATH/src`)
nơi mã nguồn của gói nằm.

Đường dẫn import nên là duy nhất trên toàn cầu, vì vậy hãy sử dụng đường dẫn của kho lưu trữ nguồn của bạn làm cơ sở.
Ví dụ, gói `websocket` từ kho con `go.net` có
đường dẫn import là `"golang.org/x/net/websocket"`.
Dự án Go sở hữu đường dẫn `"github.com/golang"`,
vì vậy đường dẫn đó không thể được sử dụng bởi một tác giả khác cho một gói khác.
Vì URL kho lưu trữ và đường dẫn import là một và giống nhau,
lệnh `go get` có thể tự động lấy và cài đặt gói.

Nếu bạn không sử dụng kho lưu trữ nguồn được lưu trữ,
hãy chọn một tiền tố duy nhất nào đó như tên miền,
công ty hoặc tên dự án.
Ví dụ, đường dẫn import của tất cả code Go nội bộ của Google bắt đầu bằng
chuỗi `"google"`.

Phần tử cuối cùng của đường dẫn import thường giống với tên gói.
Ví dụ, đường dẫn import `"net/http"` chứa gói `http`.
Đây không phải là yêu cầu bắt buộc - bạn có thể làm cho chúng khác nhau nếu muốn - nhưng
bạn nên tuân theo quy ước vì tính dự đoán được:
người dùng có thể ngạc nhiên khi import `"foo/bar"` giới thiệu định danh
`quux` vào không gian tên gói.

Đôi khi mọi người đặt `GOPATH` thành thư mục gốc của kho lưu trữ nguồn của họ và
đặt các gói vào các thư mục tương đối với thư mục gốc kho lưu trữ,
chẳng hạn như `"src/my/package"`.
Một mặt, điều này giữ cho đường dẫn import ngắn (`"my/package"` thay vì
`"github.com/me/project/my/package"`),
nhưng mặt khác nó phá vỡ `go get` và buộc người dùng phải đặt lại `GOPATH`
để sử dụng gói. Đừng làm điều này.

## Giảm thiểu giao diện được xuất

Code của bạn có khả năng bao gồm nhiều phần code hữu ích nhỏ,
và do đó thật hấp dẫn khi muốn hiển thị nhiều chức năng đó trong giao diện được xuất của gói.
Hãy cưỡng lại sự thôi thúc đó!

Giao diện bạn cung cấp càng lớn, bạn càng phải hỗ trợ nhiều hơn.
Người dùng sẽ nhanh chóng phụ thuộc vào mọi kiểu,
hàm, biến và hằng số bạn xuất,
tạo ra một hợp đồng ngầm mà bạn phải tôn trọng mãi mãi hoặc có nguy cơ
phá vỡ các chương trình của người dùng.
Khi chuẩn bị Go 1, chúng tôi đã xem xét cẩn thận các giao diện được xuất của thư viện chuẩn
và loại bỏ các phần mà chúng tôi chưa sẵn sàng cam kết.
Bạn nên cẩn thận tương tự khi phân phối các thư viện của riêng mình.

Nếu không chắc, hãy bỏ ra ngoài!

## Đặt gì vào trong một gói

Thật dễ chỉ ném tất cả mọi thứ vào một gói "túi đựng đồ linh tinh",
nhưng điều này làm loãng ý nghĩa của tên gói (vì nó phải bao gồm
nhiều chức năng) và buộc người dùng của các phần nhỏ của gói
phải biên dịch và liên kết rất nhiều code không liên quan.

Mặt khác, cũng dễ đi quá đà khi chia code của bạn
thành các gói nhỏ,
trong trường hợp đó bạn có khả năng sẽ bị sa lầy vào thiết kế giao diện,
thay vì chỉ hoàn thành công việc.

Hãy nhìn vào các thư viện chuẩn Go như một hướng dẫn.
Một số gói của nó lớn và một số nhỏ.
Ví dụ, gói [http](/pkg/net/http/) bao gồm
17 tệp nguồn go (không tính bài kiểm thử) và xuất 109 định danh,
và gói [hash](/pkg/hash/) bao gồm một tệp
xuất chỉ ba khai báo.
Không có quy tắc cứng nhắc; cả hai cách tiếp cận đều phù hợp trong ngữ cảnh của chúng.

Nói như vậy, gói main thường lớn hơn các gói khác.
Các lệnh phức tạp chứa rất nhiều code ít hữu ích bên ngoài
ngữ cảnh của file thực thi,
và thường đơn giản hơn khi chỉ giữ tất cả ở một nơi.
Ví dụ, lệnh go dài hơn 12000 dòng trải rộng trên [34 tệp](/src/cmd/go/).

## Tài liệu hóa code của bạn

Tài liệu tốt là một phẩm chất thiết yếu của code có thể sử dụng và bảo trì được.
Đọc bài viết [Godoc: documenting Go code](/doc/articles/godoc_documenting_go_code.html)
để tìm hiểu cách viết các chú thích tài liệu tốt.
