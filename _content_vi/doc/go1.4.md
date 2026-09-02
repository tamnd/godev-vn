---
title: Ghi chú phát hành Go 1.4
---

## Giới thiệu về Go 1.4 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.4, ra đời theo lịch sáu tháng sau 1.3.

Nó chỉ chứa một thay đổi ngôn ngữ nhỏ,
dưới dạng một biến thể đơn giản tương thích ngược của vòng lặp `for`-`range`,
và một thay đổi có thể gây hỏng đối với trình biên dịch liên quan đến các phương thức trên con trỏ-đến-con trỏ.

Bản phát hành tập trung chủ yếu vào công việc triển khai, cải thiện bộ gom rác
và chuẩn bị nền tảng cho một bộ gom rác hoàn toàn đồng thời sẽ ra mắt trong
vài bản phát hành tiếp theo.
Stack bây giờ là liên tục, được phân bổ lại khi cần thay vì liên kết vào các
"segment" mới;
bản phát hành này do đó loại bỏ vấn đề tai tiếng "hot stack split".
Có một số công cụ mới có sẵn bao gồm hỗ trợ trong lệnh `go`
cho việc tạo mã nguồn vào thời gian build.
Bản phát hành cũng thêm hỗ trợ cho bộ xử lý ARM trên Android và Native Client (NaCl)
và cho AMD64 trên Plan 9.

Như thường lệ, Go 1.4 vẫn giữ [cam kết
tương thích](/doc/go1compat.html),
và hầu hết mọi thứ
sẽ tiếp tục biên dịch và chạy mà không thay đổi khi chuyển sang 1.4.

## Thay đổi về ngôn ngữ {#language}

### Vòng lặp for-range {#forrange}

Cho đến Go 1.3, vòng lặp `for`-`range` có hai dạng

	for i, v := range x {
		...
	}

và

	for i := range x {
		...
	}

Nếu người ta không quan tâm đến các giá trị vòng lặp, chỉ quan tâm đến chính việc lặp, vẫn phải
đề cập đến một biến (có thể là [blank identifier](/ref/spec#Blank_identifier), như trong
`for` `_` `=` `range` `x`), vì
dạng

	for range x {
		...
	}

không được phép về mặt cú pháp.

Tình huống này có vẻ vụng về, vì vậy kể từ Go 1.4, dạng không có biến bây giờ là hợp lệ.
Pattern này hiếm khi xuất hiện nhưng mã nguồn có thể gọn hơn khi nó xuất hiện.

_Cập nhật_: Thay đổi hoàn toàn tương thích ngược với các chương trình Go hiện có, nhưng các công cụ phân tích parse tree Go có thể cần được sửa đổi để chấp nhận
dạng mới này vì
field `Key` của [`RangeStmt`](/pkg/go/ast/#RangeStmt)
bây giờ có thể là `nil`.

### Lệnh gọi phương thức trên \*\*T {#methodonpointertopointer}

Với các khai báo này,

	type T int
	func (T) M() {}
	var x **T

cả `gc` lẫn `gccgo` đều chấp nhận lệnh gọi phương thức

	x.M()

đây là một dereference kép của con trỏ-đến-con trỏ `x`.
Đặc tả Go cho phép một dereference đơn được chèn tự động,
nhưng không phải hai, vì vậy lệnh gọi này sai theo định nghĩa ngôn ngữ.
Do đó nó đã bị cấm trong Go 1.4, đây là thay đổi gây hỏng,
mặc dù rất ít chương trình sẽ bị ảnh hưởng.

_Cập nhật_: Mã nguồn phụ thuộc vào hành vi sai cũ sẽ không còn
biên dịch được nhưng dễ sửa bằng cách thêm một dereference tường minh.

## Thay đổi về các hệ điều hành và kiến trúc được hỗ trợ {#os}

### Android {#android}

Go 1.4 có thể build các binary cho bộ xử lý ARM chạy hệ điều hành Android.
Nó cũng có thể build một thư viện `.so` có thể được tải bởi một ứng dụng Android
bằng cách sử dụng các gói hỗ trợ trong kho phụ [mobile](https://golang.org/x/mobile).
Một mô tả ngắn về các kế hoạch cho cổng thử nghiệm này có sẵn
[tại đây](/s/go14android).

### NaCl trên ARM {#naclarm}

Bản phát hành trước đã giới thiệu hỗ trợ Native Client (NaCl) cho x86 32 bit
(`GOARCH=386`)
và x86 64 bit dùng con trỏ 32 bit (GOARCH=amd64p32).
Bản phát hành 1.4 thêm hỗ trợ NaCl cho ARM (GOARCH=arm).

### Plan9 trên AMD64 {#plan9amd64}

Bản phát hành này thêm hỗ trợ cho hệ điều hành Plan 9 trên bộ xử lý AMD64,
với điều kiện nhân hỗ trợ system call `nsec` và dùng 4K page.

## Thay đổi về hướng dẫn tương thích {#compatibility}

Gói [`unsafe`](/pkg/unsafe/) cho phép
đánh bại hệ thống kiểu của Go bằng cách khai thác các chi tiết nội bộ của triển khai
hoặc biểu diễn máy của dữ liệu.
Chưa bao giờ được chỉ định rõ ràng rằng cách dùng `unsafe` có ý nghĩa gì
đối với tương thích như được chỉ định trong
[hướng dẫn tương thích Go](go1compat.html).
Câu trả lời, tất nhiên, là chúng tôi không thể đưa ra cam kết tương thích
cho mã nguồn thực hiện các thao tác không an toàn.

Chúng tôi đã làm rõ tình huống này trong tài liệu được bao gồm trong bản phát hành.
[Hướng dẫn tương thích Go](go1compat.html) và
tài liệu cho gói [`unsafe`](/pkg/unsafe/)
bây giờ nói rõ rằng mã nguồn không an toàn không được đảm bảo vẫn tương thích.

_Cập nhật_: Không có thay đổi kỹ thuật nào; đây chỉ là làm rõ
tài liệu.

## Thay đổi về triển khai và công cụ {#impl}

### Thay đổi đối với thời gian chạy {#runtime}

Trước Go 1.4, thời gian chạy (bộ gom rác, hỗ trợ đồng thời, quản lý interface,
map, slice, chuỗi, ...) chủ yếu được viết bằng C, với một số hỗ trợ assembly.
Trong 1.4, nhiều mã nguồn đã được dịch sang Go để bộ gom rác có thể quét
stack của các chương trình trong thời gian chạy và có thông tin chính xác về những biến nào
đang hoạt động.
Thay đổi này lớn nhưng không nên có tác dụng ngữ nghĩa đối với các chương trình.

Việc viết lại này cho phép bộ gom rác trong 1.4 hoàn toàn chính xác,
có nghĩa là nó biết vị trí của tất cả các con trỏ hoạt động trong chương trình.
Điều này có nghĩa là heap sẽ nhỏ hơn vì sẽ không có dương tính giả giữ các không phải con trỏ còn sống.
Các thay đổi liên quan khác cũng giảm kích thước heap, nhỏ hơn 10%-30% tổng thể
so với bản phát hành trước.

Hệ quả là stack không còn phân đoạn, loại bỏ vấn đề "hot split".
Khi đạt giới hạn stack, một stack mới lớn hơn được cấp phát, tất cả các frame hoạt động cho
goroutine được sao chép đến đó, và bất kỳ con trỏ nào vào stack được cập nhật.
Hiệu năng có thể tốt hơn đáng kể trong một số trường hợp và luôn có thể dự đoán hơn.
Chi tiết có trong [tài liệu thiết kế](/s/contigstacks).

Việc dùng các stack liên tục có nghĩa là stack có thể bắt đầu nhỏ hơn mà không gây ra vấn đề hiệu năng,
vì vậy kích thước bắt đầu mặc định của stack goroutine trong 1.4 đã được giảm từ 8192 byte xuống 2048 byte.

Là chuẩn bị cho bộ gom rác đồng thời được lên kế hoạch cho bản phát hành 1.5,
các lần ghi vào các giá trị con trỏ trong heap bây giờ được thực hiện bởi một lệnh gọi hàm,
gọi là write barrier, thay vì trực tiếp từ hàm cập nhật giá trị.
Trong bản phát hành tiếp theo, điều này sẽ cho phép bộ gom rác làm trung gian các lần ghi vào heap trong khi nó đang chạy.
Thay đổi này không có tác dụng ngữ nghĩa đối với các chương trình trong 1.4, nhưng đã
được bao gồm trong bản phát hành để kiểm tra trình biên dịch và hiệu năng kết quả.

Triển khai các giá trị interface đã được sửa đổi.
Trong các bản phát hành trước, interface chứa một từ là một con trỏ hoặc một giá trị
scalar một từ, tùy thuộc vào kiểu của đối tượng cụ thể được lưu trữ.
Triển khai này gây vấn đề cho bộ gom rác,
vì vậy kể từ 1.4, các giá trị interface luôn giữ một con trỏ.
Trong các chương trình đang chạy, hầu hết các giá trị interface vốn đã là con trỏ,
vì vậy tác dụng là tối thiểu, nhưng các chương trình lưu trữ số nguyên (ví dụ) trong
interface sẽ thấy nhiều cấp phát hơn.

Kể từ Go 1.3, thời gian chạy crash nếu nó tìm thấy một từ bộ nhớ nên chứa
một con trỏ hợp lệ nhưng thay vào đó chứa một con trỏ rõ ràng không hợp lệ (ví dụ: giá trị 3).
Các chương trình lưu trữ số nguyên trong các giá trị con trỏ có thể gặp phải kiểm tra này và crash.
Trong Go 1.4, đặt biến [`GODEBUG`](/pkg/runtime/)
`invalidptr=0` vô hiệu hóa
crash như một cách giải quyết, nhưng chúng tôi không thể đảm bảo rằng các bản phát hành tương lai sẽ có thể
tránh crash; cách sửa đúng là viết lại mã nguồn không đặt bí danh số nguyên và con trỏ.

### Assembly {#asm}

Ngôn ngữ được chấp nhận bởi các trình hợp dịch `cmd/5a`, `cmd/6a`
và `cmd/8a` đã có một số thay đổi,
chủ yếu để dễ dàng cung cấp thông tin kiểu cho thời gian chạy hơn.

Đầu tiên, file `textflag.h` định nghĩa các flag cho các chỉ thị `TEXT`
đã được sao chép từ thư mục nguồn linker sang một vị trí chuẩn để nó có thể được
bao gồm với chỉ thị đơn giản

	#include "textflag.h"

Các thay đổi quan trọng hơn là cách nguồn trình hợp dịch có thể định nghĩa thông tin kiểu cần thiết.
Đối với hầu hết các chương trình, sẽ đủ để di chuyển các định nghĩa dữ liệu
(các chỉ thị `DATA` và `GLOBL`)
ra khỏi assembly vào các file Go
và để viết một khai báo Go cho mỗi hàm assembly.
[Tài liệu assembly](/doc/asm#runtime) mô tả những việc cần làm.

_Cập nhật_:
Các file assembly bao gồm `textflag.h` từ vị trí cũ của nó
vẫn sẽ hoạt động, nhưng nên được cập nhật.
Đối với thông tin kiểu, hầu hết các quy trình assembly sẽ không cần thay đổi,
nhưng tất cả nên được kiểm tra.
Các file nguồn assembly định nghĩa dữ liệu,
các hàm có frame stack không rỗng, hoặc các hàm trả về con trỏ
cần chú ý đặc biệt.
Mô tả về các thay đổi cần thiết (nhưng đơn giản)
có trong [tài liệu assembly](/doc/asm#runtime).

Thêm thông tin về các thay đổi này có trong [tài liệu assembly](/doc/asm).

### Trạng thái của gccgo {#gccgo}

Lịch phát hành cho các dự án GCC và Go không trùng nhau.
Bản phát hành GCC 4.9 chứa phiên bản Go 1.2 của gccgo.
Bản phát hành tiếp theo, GCC 5, có thể sẽ có phiên bản Go 1.4 của gccgo.

### Các gói nội bộ {#internalpackages}

Hệ thống gói của Go giúp dễ dàng cấu trúc các chương trình thành các thành phần với ranh giới rõ ràng,
nhưng chỉ có hai dạng truy cập: cục bộ (không xuất) và toàn cục (xuất).
Đôi khi người ta muốn có các thành phần không được xuất,
chẳng hạn để tránh có được các client của các interface vào mã nguồn là một phần của kho lưu trữ công khai
nhưng không nhằm mục đích sử dụng bên ngoài chương trình mà nó thuộc về.

Ngôn ngữ Go không có khả năng thực thi sự phân biệt này, nhưng kể từ Go 1.4, lệnh
[`go`](/cmd/go/) giới thiệu
một cơ chế để định nghĩa các gói "internal" không thể được import bởi các gói bên ngoài
cây nguồn nơi chúng nằm.

Để tạo gói như vậy, đặt nó trong một thư mục có tên `internal` hoặc trong một thư mục con của một thư mục
có tên internal.
Khi lệnh `go` thấy một import của một gói có `internal` trong đường dẫn của nó,
nó xác minh rằng gói đang thực hiện import
nằm trong cây có gốc tại thư mục cha của thư mục `internal`.
Ví dụ, một gói `.../a/b/c/internal/d/e/f`
chỉ có thể được import bởi mã nguồn trong cây thư mục có gốc tại `.../a/b/c`.
Nó không thể được import bởi mã nguồn trong `.../a/b/g` hoặc trong bất kỳ kho lưu trữ nào khác.

Đối với Go 1.4, cơ chế gói internal được thực thi cho kho lưu trữ Go chính;
từ 1.5 trở đi nó sẽ được thực thi cho bất kỳ kho lưu trữ nào.

Chi tiết đầy đủ về cơ chế có trong
[tài liệu thiết kế](/s/go14internal).

### Đường dẫn import chuẩn {#canonicalimports}

Mã nguồn thường nằm trong các kho lưu trữ được lưu trữ bởi các dịch vụ công khai như `github.com`,
có nghĩa là đường dẫn import cho các gói bắt đầu bằng tên của dịch vụ lưu trữ,
`github.com/rsc/pdf` chẳng hạn.
Người ta có thể dùng
[một cơ chế hiện có](/cmd/go/#hdr-Remote_import_paths)
để cung cấp một đường dẫn import "tùy chỉnh" hoặc "vanity" như
`rsc.io/pdf`, nhưng
điều đó tạo ra hai đường dẫn import hợp lệ cho gói.
Đó là một vấn đề: người ta có thể vô tình import gói thông qua hai
đường dẫn riêng biệt trong một chương trình đơn, điều này lãng phí;
bỏ lỡ bản cập nhật cho một gói vì đường dẫn đang được dùng không được nhận ra là
lỗi thời;
hoặc phá vỡ các client dùng đường dẫn cũ bằng cách chuyển gói sang dịch vụ lưu trữ khác.

Go 1.4 giới thiệu một chú thích cho các mệnh đề gói trong mã nguồn Go xác định đường dẫn import chuẩn
cho gói.
Nếu một import được thử bằng một đường dẫn không chuẩn,
lệnh [`go`](/cmd/go/)
sẽ từ chối biên dịch gói đang import.

Cú pháp đơn giản: đặt một comment xác định trên dòng gói.
Đối với ví dụ của chúng ta, mệnh đề gói sẽ đọc:

	package pdf // import "rsc.io/pdf"

Với điều này, lệnh `go` sẽ
từ chối biên dịch một gói import `github.com/rsc/pdf`,
đảm bảo rằng mã nguồn có thể được chuyển mà không làm hỏng người dùng.

Kiểm tra là vào thời gian build, không phải thời gian tải xuống, vì vậy nếu `go` `get`
thất bại vì kiểm tra này, gói được import sai đã được sao chép đến máy cục bộ
và nên được xóa thủ công.

Để bổ sung cho tính năng mới này, một kiểm tra đã được thêm vào thời gian cập nhật để xác minh
rằng kho lưu trữ từ xa của gói cục bộ khớp với kho lưu trữ của import tùy chỉnh.
Lệnh `go` `get` `-u` sẽ thất bại khi
cập nhật một gói nếu kho lưu trữ từ xa của nó đã thay đổi kể từ khi nó được
tải xuống lần đầu.
Flag mới `-f` ghi đè kiểm tra này.

Thêm thông tin có trong
[tài liệu thiết kế](/s/go14customimport).

### Đường dẫn import cho các kho phụ {#subrepo}

Các kho phụ của dự án Go (`code.google.com/p/go.tools` v.v.)
bây giờ có sẵn dưới các đường dẫn import tùy chỉnh thay thế `code.google.com/p/go.` bằng `golang.org/x/`,
như trong `golang.org/x/tools`.
Chúng tôi sẽ thêm các comment import chuẩn vào mã nguồn khoảng ngày 1 tháng 6 năm 2015,
tại thời điểm đó Go 1.4 trở lên sẽ ngừng chấp nhận các đường dẫn `code.google.com` cũ.

_Cập nhật_: Tất cả mã nguồn import từ kho phụ nên thay đổi
để dùng các đường dẫn `golang.org` mới.
Go 1.0 trở lên có thể phân giải và import các đường dẫn mới, vì vậy cập nhật sẽ không phá vỡ
tương thích với các bản phát hành cũ hơn.
Mã nguồn chưa cập nhật sẽ ngừng biên dịch với Go 1.4 khoảng ngày 1 tháng 6 năm 2015.

### Lệnh con go generate {#gogenerate}

Lệnh [`go`](/cmd/go/) có một lệnh con mới,
[`go generate`](/cmd/go/#hdr-Generate_Go_files_by_processing_source),
để tự động hóa việc chạy các công cụ tạo mã nguồn trước khi biên dịch.
Ví dụ, nó có thể được dùng để chạy trình biên dịch trình biên dịch [`yacc`](/cmd/yacc)
trên một file `.y` để tạo ra file nguồn Go triển khai ngữ pháp,
hoặc để tự động hóa việc tạo các phương thức `String` cho các hằng số kiểu dùng công cụ mới
[stringer](https://godoc.org/golang.org/x/tools/cmd/stringer)
trong kho phụ `golang.org/x/tools`.

Để biết thêm thông tin, xem
[tài liệu thiết kế](/s/go1.4-generate).

### Thay đổi đối với xử lý tên file {#filenames}

Các ràng buộc build, còn được gọi là build tag, kiểm soát việc biên dịch bằng cách bao gồm hoặc loại trừ các file
(xem tài liệu [`/go/build`](/pkg/go/build/)).
Việc biên dịch cũng có thể được kiểm soát bởi tên của chính file bằng cách "gắn thẻ" file với
một hậu tố (trước phần mở rộng `.go` hoặc `.s`) với dấu gạch dưới
và tên của kiến trúc hoặc hệ điều hành.
Ví dụ, file `gopher_arm.go` chỉ được biên dịch nếu bộ xử lý đích
là ARM.

Trước Go 1.4, một file chỉ được gọi là `arm.go` cũng được gắn thẻ tương tự, nhưng hành vi này
có thể làm hỏng các nguồn khi các kiến trúc mới được thêm vào, khiến các file đột nhiên được gắn thẻ.
Do đó trong 1.4, một file sẽ được gắn thẻ theo cách này chỉ khi tag (tên kiến trúc hoặc hệ điều hành)
được đứng trước bởi một dấu gạch dưới.

_Cập nhật_: Các gói phụ thuộc vào hành vi cũ sẽ không còn biên dịch đúng.
Các file có tên như `windows.go` hoặc `amd64.go` nên
có các build tag tường minh được thêm vào nguồn hoặc được đổi tên thành thứ gì đó như
`os_windows.go` hoặc `support_amd64.go`.

### Các thay đổi khác đối với lệnh go {#gocmd}

Có một số thay đổi nhỏ đối với lệnh
[`cmd/go`](/cmd/go/)
đáng được ghi chú.

  - Trừ khi [`cgo`](/cmd/cgo/) đang được dùng để build gói,
    lệnh `go` bây giờ từ chối biên dịch các file nguồn C,
    vì các trình biên dịch C liên quan
    ([`6c`](/cmd/6c/) v.v.)
    được dự định xóa khỏi cài đặt trong một bản phát hành tương lai nào đó.
    (Chúng được dùng ngày nay chỉ để build một phần của thời gian chạy.)
    Rất khó để dùng chúng đúng cách trong mọi trường hợp, vì vậy bất kỳ cách dùng tồn tại nào có thể là không đúng,
    vì vậy chúng tôi đã vô hiệu hóa chúng.
  - Lệnh con [`go` `test`](/cmd/go/#hdr-Test_packages)
    có flag mới, `-o`, để đặt tên của binary kết quả,
    tương ứng với cùng flag trong các lệnh con khác.
    Flag `-file` không hoạt động đã bị xóa.
  - Lệnh con [`go` `test`](/cmd/go/#hdr-Test_packages)
    sẽ biên dịch và liên kết tất cả các file `*_test.go` trong gói,
    ngay cả khi không có hàm `Test` nào trong chúng.
    Trước đây nó bỏ qua các file như vậy.
  - Hành vi của flag `-a`
    của lệnh con [`go` `build`](/cmd/go/#hdr-Test_packages)
    đã thay đổi đối với các cài đặt không phải phát triển.
    Đối với các cài đặt chạy bản phân phối đã phát hành, flag `-a` sẽ không còn
    xây dựng lại thư viện chuẩn và các lệnh, để tránh ghi đè lên các file cài đặt.

### Thay đổi về bố cục mã nguồn gói {#pkg}

Trong kho lưu trữ Go chính, mã nguồn cho các gói được giữ trong
thư mục `src/pkg`, điều này có lý nhưng khác với
các kho lưu trữ khác, bao gồm các kho phụ Go.
Trong Go 1.4, cấp `pkg` của cây nguồn bây giờ đã biến mất, vì vậy ví dụ
mã nguồn của gói [`fmt`](/pkg/fmt/), từng được giữ trong
thư mục `src/pkg/fmt`, bây giờ nằm một cấp cao hơn trong `src/fmt`.

_Cập nhật_: Các công cụ như `godoc` khám phá mã nguồn
cần biết về vị trí mới. Tất cả các công cụ và dịch vụ được duy trì bởi nhóm Go
đã được cập nhật.

### SWIG {#swig}

Do các thay đổi thời gian chạy trong bản phát hành này, Go 1.4 yêu cầu SWIG 3.0.3.

### Linh tinh {#misc}

Thư mục `misc` cấp cao nhất của kho lưu trữ chuẩn từng chứa
hỗ trợ Go cho các trình soạn thảo và IDE: các plugin, script khởi tạo v.v.
Việc bảo trì những thứ này ngày càng tốn thời gian
và cần sự trợ giúp bên ngoài vì nhiều trình soạn thảo được liệt kê không được
các thành viên của nhóm cốt lõi sử dụng.
Điều đó cũng yêu cầu chúng tôi đưa ra quyết định về plugin nào tốt nhất cho một
trình soạn thảo nhất định, ngay cả đối với các trình soạn thảo chúng tôi không dùng.

Cộng đồng Go nói chung phù hợp hơn để quản lý thông tin này.
Do đó trong Go 1.4, hỗ trợ này đã bị xóa khỏi kho lưu trữ.
Thay vào đó, có một danh sách được sắp xếp và nhiều thông tin về những gì có sẵn trên
một [trang wiki](/wiki/IDEsAndTextEditorPlugins).

## Hiệu năng {#performance}

Hầu hết các chương trình sẽ chạy với khoảng tốc độ giống như hoặc nhanh hơn một chút trong 1.4 so với 1.3;
một số sẽ chậm hơn một chút.
Có nhiều thay đổi, làm cho rất khó để chính xác về những gì mong đợi.

Như đã đề cập ở trên, nhiều thời gian chạy đã được dịch sang Go từ C,
điều này dẫn đến một số giảm kích thước heap.
Nó cũng cải thiện hiệu năng một chút vì trình biên dịch Go tốt hơn
trong tối ưu hóa, do những thứ như nội tuyến, so với trình biên dịch C được dùng để build
thời gian chạy.

Bộ gom rác đã được tăng tốc, dẫn đến các cải thiện có thể đo được cho
các chương trình nặng về bộ gom rác.
Mặt khác, các write barrier mới làm chậm mọi thứ lại, thường khoảng
cùng lượng nhưng, tùy thuộc vào hành vi của chúng, một số chương trình
có thể chậm hơn hoặc nhanh hơn một chút.

Các thay đổi thư viện ảnh hưởng đến hiệu năng được ghi lại bên dưới.

## Thay đổi đối với thư viện chuẩn {#library}

### Các gói mới {#new_packages}

Không có gói mới nào trong bản phát hành này.

### Thay đổi lớn đối với thư viện {#major_library_changes}

#### bufio.Scanner {#scanner}

Kiểu [`Scanner`](/pkg/bufio/#Scanner) trong gói
[`bufio`](/pkg/bufio/)
đã có một lỗi được sửa có thể yêu cầu thay đổi đối với các
[hàm split tùy chỉnh](/pkg/bufio/#SplitFunc).
Lỗi làm cho không thể tạo một token rỗng tại EOF; sửa
thay đổi các điều kiện kết thúc được thấy bởi hàm split.
Trước đây, quét dừng lại ở EOF nếu không còn dữ liệu nào.
Kể từ 1.4, hàm split sẽ được gọi một lần tại EOF sau khi đầu vào cạn kiệt,
vì vậy hàm split có thể tạo ra một token rỗng cuối cùng
như tài liệu đã hứa.

_Cập nhật_: Các hàm split tùy chỉnh có thể cần được sửa đổi để
xử lý các token rỗng tại EOF như mong muốn.

#### syscall {#syscall}

Gói [`syscall`](/pkg/syscall/) bây giờ bị đóng băng ngoại trừ
các thay đổi cần thiết để duy trì kho lưu trữ cốt lõi.
Cụ thể, nó sẽ không còn được mở rộng để hỗ trợ các system call mới hoặc khác
không được dùng bởi lõi.
Lý do được mô tả chi tiết trong [một
tài liệu riêng](/s/go1.4-syscall).

Một kho phụ mới, [golang.org/x/sys](https://golang.org/x/sys),
đã được tạo để phục vụ là vị trí cho các phát triển mới để hỗ trợ system call
trên tất cả các nhân.
Nó có cấu trúc đẹp hơn, với ba gói mỗi gói giữ triển khai của
system call cho một trong
[Unix](https://godoc.org/golang.org/x/sys/unix),
[Windows](https://godoc.org/golang.org/x/sys/windows) và
[Plan 9](https://godoc.org/golang.org/x/sys/plan9).
Các gói này sẽ được quản lý tổng quát hơn, chấp nhận tất cả các thay đổi hợp lý
phản ánh các interface nhân trong các hệ điều hành đó.
Xem tài liệu và bài viết được đề cập ở trên để biết thêm thông tin.

_Cập nhật_: Các chương trình hiện có không bị ảnh hưởng vì gói `syscall`
phần lớn không thay đổi so với bản phát hành 1.3.
Phát triển tương lai yêu cầu các system call không có trong gói `syscall`
nên build trên `golang.org/x/sys` thay thế.

### Thay đổi nhỏ đối với thư viện {#minor_library_changes}

Danh sách sau đây tóm tắt một số thay đổi nhỏ đối với thư viện, chủ yếu là bổ sung.
Xem tài liệu gói liên quan để biết thêm thông tin về từng thay đổi.

  - [`Writer`](/pkg/archive/zip/#Writer) của gói
    [`archive/zip`](/pkg/archive/zip/) bây giờ hỗ trợ phương thức
    [`Flush`](/pkg/archive/zip/#Writer.Flush).
  - Các gói [`compress/flate`](/pkg/compress/flate/),
    [`compress/gzip`](/pkg/compress/gzip/),
    và [`compress/zlib`](/pkg/compress/zlib/)
    bây giờ hỗ trợ phương thức `Reset`
    cho các decompressor, cho phép chúng tái sử dụng buffer và cải thiện hiệu năng.
    Gói [`compress/gzip`](/pkg/compress/gzip/) cũng có phương thức
    [`Multistream`](/pkg/compress/gzip/#Reader.Multistream) để kiểm soát hỗ trợ
    cho các file multistream.
  - Gói [`crypto`](/pkg/crypto/) bây giờ có
    interface [`Signer`](/pkg/crypto/#Signer), được triển khai bởi các kiểu
    `PrivateKey` trong
    [`crypto/ecdsa`](/pkg/crypto/ecdsa) và
    [`crypto/rsa`](/pkg/crypto/rsa).
  - Gói [`crypto/tls`](/pkg/crypto/tls/)
    bây giờ hỗ trợ ALPN như được định nghĩa trong [RFC 7301](https://tools.ietf.org/html/rfc7301).
  - Gói [`crypto/tls`](/pkg/crypto/tls/)
    bây giờ hỗ trợ lựa chọn lập trình của chứng chỉ server
    thông qua hàm mới [`CertificateForName`](/pkg/crypto/tls/#Config.CertificateForName)
    của cấu trúc [`Config`](/pkg/crypto/tls/#Config).
  - Cũng trong gói crypto/tls, server bây giờ hỗ trợ
    [TLS\_FALLBACK\_SCSV](https://tools.ietf.org/html/draft-ietf-tls-downgrade-scsv-00)
    để giúp client phát hiện các cuộc tấn công fallback.
    (Client Go hoàn toàn không hỗ trợ fallback, vì vậy nó không dễ bị
    các cuộc tấn công đó.)
  - Gói [`database/sql`](/pkg/database/sql/) bây giờ có thể liệt kê tất cả
    [`Driver`](/pkg/database/sql/#Drivers) đã đăng ký.
  - Gói [`debug/dwarf`](/pkg/debug/dwarf/) bây giờ hỗ trợ
    [`UnspecifiedType`](/pkg/debug/dwarf/#UnspecifiedType).
  - Trong gói [`encoding/asn1`](/pkg/encoding/asn1/),
    các phần tử tùy chọn với giá trị mặc định bây giờ chỉ bị bỏ qua nếu chúng có giá trị đó.
  - Gói [`encoding/csv`](/pkg/encoding/csv/) không còn
    đặt ngoặc kép cho các chuỗi rỗng nhưng đặt ngoặc kép cho marker kết thúc dữ liệu `\.` (backslash dot).
    Điều này được cho phép theo định nghĩa CSV và cho phép nó hoạt động tốt hơn với Postgres.
  - Gói [`encoding/gob`](/pkg/encoding/gob/) đã được viết lại để loại bỏ
    việc dùng các thao tác không an toàn, cho phép nó được dùng trong các môi trường không cho phép dùng gói
    [`unsafe`](/pkg/unsafe/).
    Đối với các cách dùng điển hình, nó sẽ chậm hơn 10-30%, nhưng delta phụ thuộc vào kiểu dữ liệu và
    trong một số trường hợp, đặc biệt liên quan đến mảng, nó có thể nhanh hơn.
    Không có thay đổi chức năng.
  - [`Decoder`](/pkg/encoding/xml/#Decoder) của gói
    [`encoding/xml`](/pkg/encoding/xml/)
    bây giờ có thể báo cáo offset đầu vào của nó.
  - Trong gói [`fmt`](/pkg/fmt/),
    định dạng của con trỏ đến map đã thay đổi để nhất quán với con trỏ
    đến struct, mảng, v.v.
    Ví dụ, `&map[string]int{"one":` `1}` bây giờ in theo mặc định là
    `&map[one:` `1]` thay vì một giá trị con trỏ thập lục phân.
  - Các triển khai [`Image`](/pkg/image/#Image)
    của gói [`image`](/pkg/image/)
    như
    [`RGBA`](/pkg/image/#RGBA) và
    [`Gray`](/pkg/image/#Gray) có các phương thức chuyên biệt
    [`RGBAAt`](/pkg/image/#RGBA.RGBAAt) và
    [`GrayAt`](/pkg/image/#Gray.GrayAt) bên cạnh phương thức chung
    [`At`](/pkg/image/#Image.At).
  - Gói [`image/png`](/pkg/image/png/) bây giờ có kiểu
    [`Encoder`](/pkg/image/png/#Encoder)
    để kiểm soát mức nén được dùng để mã hóa.
  - Gói [`math`](/pkg/math/) bây giờ có hàm
    [`Nextafter32`](/pkg/math/#Nextafter32).
  - Kiểu [`Request`](/pkg/net/http/#Request)
    của gói [`net/http`](/pkg/net/http/)
    có phương thức mới [`BasicAuth`](/pkg/net/http/#Request.BasicAuth)
    trả về tên người dùng và mật khẩu từ các yêu cầu được xác thực bằng
    HTTP Basic Authentication Scheme.
  - Kiểu [`Transport`](/pkg/net/http/#Request)
    của gói [`net/http`](/pkg/net/http/)
    có hook mới [`DialTLS`](/pkg/net/http/#Transport.DialTLS)
    cho phép tùy chỉnh hành vi của các kết nối TLS đi.
  - Kiểu [`ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy)
    của gói [`net/http/httputil`](/pkg/net/http/httputil/)
    có field mới,
    [`ErrorLog`](/pkg/net/http/#ReverseProxy.ErrorLog), cung cấp
    kiểm soát người dùng về logging.
  - Gói [`os`](/pkg/os/)
    bây giờ triển khai các symbolic link trên hệ điều hành Windows
    thông qua hàm [`Symlink`](/pkg/os/#Symlink).
    Các hệ điều hành khác đã có chức năng này.
    Ngoài ra còn có hàm mới [`Unsetenv`](/pkg/os/#Unsetenv).
  - Interface [`Type`](/pkg/reflect/#Type)
    của gói [`reflect`](/pkg/reflect/)
    có phương thức mới, [`Comparable`](/pkg/reflect/#type.Comparable),
    báo cáo liệu kiểu có triển khai các so sánh chung không.
  - Cũng trong gói [`reflect`](/pkg/reflect/), interface
    [`Value`](/pkg/reflect/#Value) bây giờ là ba thay vì bốn từ
    vì các thay đổi đối với triển khai các interface trong thời gian chạy.
    Điều này tiết kiệm bộ nhớ nhưng không có tác dụng ngữ nghĩa.
  - Gói [`runtime`](/pkg/runtime/)
    bây giờ triển khai các đồng hồ monotonic trên Windows,
    như nó đã làm cho các hệ thống khác.
  - Bộ đếm [`Mallocs`](/pkg/runtime/#MemStats.Mallocs)
    của gói [`runtime`](/pkg/runtime/)
    bây giờ đếm các phân bổ rất nhỏ đã bị bỏ lỡ trong Go 1.3.
    Điều này có thể làm hỏng các test dùng [`ReadMemStats`](/pkg/runtime/#ReadMemStats)
    hoặc [`AllocsPerRun`](/pkg/testing/#AllocsPerRun)
    do câu trả lời chính xác hơn.
  - Trong gói [`runtime`](/pkg/runtime/),
    một mảng [`PauseEnd`](/pkg/runtime/#MemStats.PauseEnd)
    đã được thêm vào các cấu trúc
    [`MemStats`](/pkg/runtime/#MemStats)
    và [`GCStats`](/pkg/runtime/#GCStats).
    Mảng này là một circular buffer của các thời điểm khi các lần dừng bộ gom rác kết thúc.
    Các khoảng thời gian dừng tương ứng đã được ghi lại trong
    [`PauseNs`](/pkg/runtime/#MemStats.PauseNs)
  - Gói [`runtime/race`](/pkg/runtime/race/)
    bây giờ hỗ trợ FreeBSD, có nghĩa là flag `-race`
    của lệnh [`go`](/pkg/cmd/go/)
    bây giờ hoạt động trên FreeBSD.
  - Gói [`sync/atomic`](/pkg/sync/atomic/)
    có kiểu mới, [`Value`](/pkg/sync/atomic/#Value).
    `Value` cung cấp một cơ chế hiệu quả để tải và
    lưu trữ nguyên tử các giá trị kiểu tùy ý.
  - Trong triển khai của gói [`syscall`](/pkg/syscall/) trên Linux,
    [`Setuid`](/pkg/syscall/#Setuid)
    và [`Setgid`](/pkg/syscall/#Setgid) đã bị vô hiệu hóa
    vì các system call đó hoạt động trên luồng đang gọi, không phải toàn bộ process, điều này là
    khác với các nền tảng khác và không phải kết quả được mong đợi.
  - Gói [`testing`](/pkg/testing/)
    có tính năng mới để cung cấp nhiều kiểm soát hơn trong việc chạy một tập hợp test.
    Nếu mã test chứa một hàm
    <pre>
    func TestMain(m *<a href="/pkg/testing/#M"><code>testing.M</code></a>)
    </pre>
    hàm đó sẽ được gọi thay vì chạy các test trực tiếp.
    Cấu trúc `M` chứa các phương thức để truy cập và chạy các test.
  - Cũng trong gói [`testing`](/pkg/testing/),
    hàm mới [`Coverage`](/pkg/testing/#Coverage)
    báo cáo tỷ lệ test coverage hiện tại,
    cho phép các test riêng lẻ báo cáo chúng đang đóng góp bao nhiêu cho
    coverage tổng thể.
  - Kiểu [`Scanner`](/pkg/text/scanner/#Scanner)
    của gói [`text/scanner`](/pkg/text/scanner/)
    có hàm mới,
    [`IsIdentRune`](/pkg/text/scanner/#Scanner.IsIdentRune),
    cho phép kiểm soát định nghĩa của một identifier khi quét.
  - Các hàm boolean `eq`, `lt`, v.v. của gói
    [`text/template`](/pkg/text/template/)
    đã được tổng quát hóa để cho phép so sánh
    các số nguyên có dấu và không dấu, đơn giản hóa cách dùng của chúng trong thực tế.
    (Trước đây chỉ có thể so sánh các giá trị có cùng dấu.)
    Tất cả các giá trị âm so sánh nhỏ hơn tất cả các giá trị không dấu.
  - Gói `time` bây giờ dùng ký hiệu chuẩn cho tiền tố micro,
    ký hiệu micro (U+00B5 'µ'), để in các khoảng thời gian microsecond.
    [`ParseDuration`](/pkg/time/#ParseDuration) vẫn chấp nhận `us`
    nhưng gói không còn in microsecond là `us`.
    \
    _Cập nhật_: Mã nguồn phụ thuộc vào định dạng đầu ra của khoảng thời gian
    nhưng không dùng ParseDuration sẽ cần được cập nhật.
