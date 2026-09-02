---
title: Ghi chú phát hành Go 1.2
---

## Giới thiệu về Go 1.2 {#introduction}

Kể từ khi phát hành [Go phiên bản 1.1](/doc/go1.1.html) vào tháng 4 năm 2013,
lịch phát hành đã được rút ngắn để làm cho quá trình phát hành hiệu quả hơn.
Bản phát hành này, Go phiên bản 1.2 hay Go 1.2, ra đời sau 1.1 khoảng sáu tháng,
trong khi 1.1 mất hơn một năm để xuất hiện sau 1.0.
Vì thang thời gian ngắn hơn, 1.2 là một delta nhỏ hơn so với bước từ 1.0 lên 1.1,
nhưng nó vẫn có một số phát triển đáng kể, bao gồm
bộ lập lịch tốt hơn và một tính năng ngôn ngữ mới.
Tất nhiên, Go 1.2 vẫn giữ [cam kết
tương thích](/doc/go1compat.html).
Đại đa số các chương trình được build với Go 1.1 (hoặc 1.0)
sẽ chạy mà không có bất kỳ thay đổi nào khi chuyển sang 1.2,
mặc dù việc đưa ra một hạn chế
với một góc khuất của ngôn ngữ có thể phơi bày mã nguồn đã sai
(xem thảo luận về [sử dụng nil](#use_of_nil)).

## Thay đổi về ngôn ngữ {#language}

Để củng cố đặc tả, một trường hợp góc khuất đã được làm rõ,
với hệ quả cho các chương trình.
Cũng có một tính năng ngôn ngữ mới.

### Sử dụng nil {#use_of_nil}

Ngôn ngữ bây giờ chỉ định rằng, vì lý do an toàn,
một số cách dùng nhất định của con trỏ nil được đảm bảo kích hoạt một panic thời gian chạy.
Ví dụ, trong Go 1.0, với mã nguồn như

	type T struct {
	    X [1<<24]byte
	    Field int32
	}

	func main() {
	    var x *T
	    ...
	}

con trỏ `nil` `x` có thể được dùng để truy cập bộ nhớ không chính xác:
biểu thức `x.Field` có thể truy cập bộ nhớ tại địa chỉ `1<<24`.
Để ngăn hành vi không an toàn như vậy, trong Go 1.2 các trình biên dịch bây giờ đảm bảo rằng bất kỳ phép gián tiếp nào qua
con trỏ nil, như minh họa ở đây nhưng cũng trong các con trỏ nil đến mảng, giá trị interface nil,
slice nil, v.v., sẽ panic hoặc trả về một giá trị không phải nil an toàn và chính xác.
Tóm lại, bất kỳ biểu thức nào yêu cầu tường minh hoặc ngầm định việc đánh giá một địa chỉ nil đều là lỗi.
Việc triển khai có thể tiêm các bài kiểm tra thêm vào chương trình được biên dịch để thực thi hành vi này.

Chi tiết thêm có trong
[tài liệu thiết kế](/s/go12nil).

_Cập nhật_:
Hầu hết mã nguồn phụ thuộc vào hành vi cũ là sai và sẽ thất bại khi chạy.
Các chương trình như vậy sẽ cần được cập nhật thủ công.

### Slice ba chỉ số {#three_index}

Go 1.2 thêm khả năng chỉ định dung lượng cũng như độ dài khi thực hiện thao tác slicing
trên một mảng hoặc slice hiện có.
Một thao tác slicing tạo ra một slice mới bằng cách mô tả một phần liên tục của một mảng hoặc slice đã tạo:

	var array [10]int
	slice := array[2:4]

Dung lượng của slice là số phần tử tối đa mà slice có thể chứa, ngay cả sau khi reslicing;
nó phản ánh kích thước của mảng cơ bản.
Trong ví dụ này, dung lượng của biến `slice` là 8.

Go 1.2 thêm cú pháp mới để cho phép một thao tác slicing chỉ định dung lượng cũng như độ dài.
Một dấu hai chấm thứ hai đưa vào giá trị dung lượng, giá trị này phải nhỏ hơn hoặc bằng dung lượng của
slice hoặc mảng nguồn, được điều chỉnh cho origin. Ví dụ,

	slice = array[2:4:7]

đặt slice có cùng độ dài như trong ví dụ trước nhưng dung lượng của nó bây giờ chỉ là 5 phần tử (7-2).
Không thể dùng giá trị slice mới này để truy cập ba phần tử cuối của mảng gốc.

Trong ký hiệu ba chỉ số này, chỉ số đầu tiên bị thiếu (`[:i:j]`) mặc định là không nhưng hai
chỉ số kia phải luôn được chỉ định tường minh.
Có thể các bản phát hành tương lai của Go có thể đưa ra các giá trị mặc định cho các chỉ số này.

Chi tiết thêm có trong
[tài liệu thiết kế](/s/go12slice).

_Cập nhật_:
Đây là thay đổi tương thích ngược không ảnh hưởng đến các chương trình hiện có.

## Thay đổi về triển khai và công cụ {#impl}

### Ngắt lịch trong bộ lập lịch {#preemption}

Trong các bản phát hành trước, một goroutine lặp mãi mãi có thể làm đói các goroutine khác
trên cùng một luồng, một vấn đề nghiêm trọng khi GOMAXPROCS
chỉ cung cấp một luồng người dùng.
Trong Go 1.2, điều này được giải quyết một phần: Bộ lập lịch đôi khi được gọi
khi nhập vào một hàm.
Điều này có nghĩa là bất kỳ vòng lặp nào bao gồm một lệnh gọi hàm (không được nội tuyến) có thể
bị ngắt lịch, cho phép các goroutine khác chạy trên cùng một luồng.

### Giới hạn số luồng {#thread_limit}

Go 1.2 giới thiệu một giới hạn có thể cấu hình (mặc định là 10.000) về tổng số luồng
mà một chương trình đơn có thể có trong không gian địa chỉ của nó, để tránh các vấn đề cạn kiệt tài nguyên
trong một số môi trường.
Lưu ý rằng goroutine được ghép lên các luồng vì vậy giới hạn này không trực tiếp
giới hạn số goroutine, chỉ là số có thể đồng thời bị block
trong một system call.
Trên thực tế, giới hạn rất khó đạt tới.

Hàm mới [`SetMaxThreads`](/pkg/runtime/debug/#SetMaxThreads) trong gói
[`runtime/debug`](/pkg/runtime/debug/) kiểm soát giới hạn số luồng.

_Cập nhật_:
Ít hàm sẽ bị ảnh hưởng bởi giới hạn, nhưng nếu một chương trình chết vì nó đạt
giới hạn, nó có thể được sửa đổi để gọi `SetMaxThreads` để đặt số cao hơn.
Thậm chí tốt hơn là tái cấu trúc chương trình để cần ít luồng hơn, giảm tiêu thụ
tài nguyên nhân.

### Kích thước stack {#stack_size}

Trong Go 1.2, kích thước tối thiểu của stack khi một goroutine được tạo đã được nâng từ 4KB lên 8KB.
Nhiều chương trình đang gặp vấn đề hiệu năng với kích thước cũ, có xu hướng
đưa ra các chuyển đổi stack-segment tốn kém trong các phần quan trọng về hiệu năng.
Con số mới được xác định bằng thử nghiệm thực nghiệm.

Ở đầu kia, hàm mới [`SetMaxStack`](/pkg/runtime/debug/#SetMaxStack)
trong gói [`runtime/debug`](/pkg/runtime/debug) kiểm soát
kích thước _tối đa_ của stack của một goroutine đơn.
Mặc định là 1GB trên các hệ thống 64 bit và 250MB trên các hệ thống 32 bit.
Trước Go 1.2, một đệ quy mất kiểm soát có thể dễ dàng tiêu thụ tất cả bộ nhớ trên một máy.

_Cập nhật_:
Kích thước stack tối thiểu tăng có thể làm cho các chương trình có nhiều goroutine dùng
nhiều bộ nhớ hơn. Không có cách giải quyết, nhưng các kế hoạch cho các bản phát hành tương lai
bao gồm công nghệ quản lý stack mới sẽ giải quyết vấn đề tốt hơn.

### Cgo và C++ {#cgo_and_cpp}

Lệnh [`cgo`](/cmd/cgo/) bây giờ sẽ gọi trình biên dịch C++
để build bất kỳ phần nào của thư viện được liên kết đến được viết bằng C++;
[tài liệu](/cmd/cgo/) có thêm chi tiết.

### Godoc và vet được chuyển đến kho phụ go.tools {#go_tools_godoc}

Cả hai binary vẫn được bao gồm trong bản phân phối, nhưng mã nguồn cho các lệnh
godoc và vet đã được chuyển đến
kho phụ [go.tools](https://code.google.com/p/go.tools).

Ngoài ra, lõi của chương trình godoc đã được tách thành một
[thư viện](https://code.google.com/p/go/source/browse/?repo=tools#hg%2Fgodoc),
trong khi bản thân lệnh nằm trong một
[thư mục](https://code.google.com/p/go/source/browse/?repo=tools#hg%2Fcmd%2Fgodoc) riêng biệt.
Việc di chuyển cho phép mã nguồn được cập nhật dễ dàng và việc tách thành thư viện và lệnh
giúp dễ dàng xây dựng các binary tùy chỉnh cho các trang nội bộ và các phương thức triển khai khác nhau.

_Cập nhật_:
Vì godoc và vet không phải là một phần của thư viện,
không có mã nguồn Go nào phụ thuộc vào nguồn của chúng và không cần cập nhật nào.

Các bản phân phối nhị phân có sẵn từ [golang.org](/)
bao gồm các binary này, vì vậy người dùng các bản phân phối này không bị ảnh hưởng.

Khi build từ nguồn, người dùng phải dùng "go get" để cài đặt godoc và vet.
(Các binary sẽ tiếp tục được cài đặt ở các vị trí thông thường của chúng, không phải
`$GOPATH/bin`.)

	$ go get code.google.com/p/go.tools/cmd/godoc
	$ go get code.google.com/p/go.tools/cmd/vet

### Trạng thái của gccgo {#gccgo}

Chúng tôi dự kiến bản phát hành GCC 4.9 trong tương lai sẽ bao gồm gccgo với hỗ trợ đầy đủ
cho Go 1.2.
Trong bản phát hành hiện tại (4.8.2) của GCC, gccgo triển khai Go 1.1.2.

### Thay đổi đối với trình biên dịch và linker gc {#gc_changes}

Go 1.2 có một số thay đổi ngữ nghĩa đối với hoạt động của bộ trình biên dịch gc.
Hầu hết người dùng sẽ không bị ảnh hưởng.

Lệnh [`cgo`](/cmd/cgo/) bây giờ
hoạt động khi C++ được bao gồm trong thư viện được liên kết.
Xem tài liệu [`cgo`](/cmd/cgo/)
để biết chi tiết.

Trình biên dịch gc hiển thị một chi tiết thừa từ nguồn gốc của nó khi
một chương trình không có mệnh đề `package`: nó giả định
file nằm trong gói `main`.
Quá khứ đã được xóa bỏ, và một mệnh đề `package` bị thiếu
bây giờ là lỗi.

Trên ARM, toolchain hỗ trợ "external linking", đây là
một bước để có thể build các thư viện dùng chung với toolchain gc
và cung cấp hỗ trợ liên kết động cho các môi trường
trong đó điều đó là cần thiết.

Trong thời gian chạy cho ARM, với `5a`, trước đây có thể tham chiếu
đến các biến `m` (machine) và `g`
(goroutine) nội bộ của thời gian chạy bằng cách dùng `R9` và `R10` trực tiếp.
Bây giờ cần tham chiếu đến chúng bằng tên đúng của chúng.

Cũng trên ARM, linker `5l` (sic) bây giờ định nghĩa
các lệnh `MOVBS` và `MOVHS`
như các từ đồng nghĩa của `MOVB` và `MOVH`,
để làm rõ hơn sự tách biệt giữa các di chuyển từ phụ có dấu và không dấu;
các phiên bản không dấu đã tồn tại với hậu tố `U`.

### Test coverage {#cover}

Một tính năng mới quan trọng của [`go test`](/pkg/go/) là
bây giờ nó có thể tính toán và, với sự trợ giúp của một chương trình "go tool cover" mới được cài đặt riêng,
hiển thị kết quả test coverage.

Công cụ cover là một phần của kho phụ
[`go.tools`](https://code.google.com/p/go/source/checkout?repo=tools).
Nó có thể được cài đặt bằng cách chạy

	$ go get code.google.com/p/go.tools/cmd/cover

Công cụ cover làm hai việc.
Đầu tiên, khi "go test" được cho flag `-cover`, nó được chạy tự động
để viết lại nguồn cho gói và chèn các câu lệnh instrumentation.
Sau đó test được biên dịch và chạy như thường lệ, và thống kê coverage cơ bản được báo cáo:

	$ go test -cover fmt
	ok  	fmt	0.060s	coverage: 91.4% of statements
	$

Thứ hai, để có báo cáo chi tiết hơn, các flag khác nhau cho "go test" có thể tạo file profile coverage,
mà chương trình cover, được gọi với "go tool cover", sau đó có thể phân tích.

Chi tiết về cách tạo và phân tích thống kê coverage có thể được tìm thấy bằng cách chạy các lệnh

	$ go help testflag
	$ go tool cover -help

### Lệnh go doc bị xóa {#go_doc}

Lệnh "go doc" bị xóa.
Lưu ý rằng bản thân công cụ [`godoc`](/cmd/godoc/) không bị xóa,
chỉ là việc bọc nó bởi lệnh [`go`](/cmd/go/).
Tất cả những gì nó làm là hiển thị tài liệu cho một gói theo đường dẫn gói,
điều mà godoc tự nó đã làm với tính linh hoạt hơn.
Do đó nó đã bị xóa để giảm số lượng công cụ tài liệu và,
như một phần của cấu trúc lại godoc, khuyến khích các tùy chọn tốt hơn trong tương lai.

_Cập nhật_: Đối với những người vẫn cần chức năng chính xác của việc chạy

	$ go doc

trong một thư mục, hành vi giống hệt như khi chạy

	$ godoc .

### Thay đổi đối với lệnh go {#gocmd}

Lệnh [`go get`](/cmd/go/)
bây giờ có flag `-t` khiến nó tải xuống các dependency
của các test được chạy bởi gói, không chỉ của bản thân gói.
Theo mặc định, như trước đây, các dependency của các test không được tải xuống.

## Hiệu năng {#performance}

Có một số cải thiện hiệu năng đáng kể trong thư viện chuẩn; đây là một vài ví dụ.

  - [`compress/bzip2`](/pkg/compress/bzip2/)
    giải nén nhanh hơn khoảng 30%.
  - Gói [`crypto/des`](/pkg/crypto/des/)
    nhanh hơn khoảng năm lần.
  - Gói [`encoding/json`](/pkg/encoding/json/)
    mã hóa nhanh hơn khoảng 30%.
  - Hiệu năng mạng trên các hệ thống Windows và BSD nhanh hơn khoảng 30% thông qua việc sử dụng
    một network poller tích hợp trong thời gian chạy, tương tự như những gì đã được thực hiện cho Linux và OS X
    trong Go 1.1.

## Thay đổi đối với thư viện chuẩn {#library}

### Các gói archive/tar và archive/zip {#archive_tar_zip}

Các gói
[`archive/tar`](/pkg/archive/tar/)
và
[`archive/zip`](/pkg/archive/zip/)
đã có sự thay đổi về ngữ nghĩa có thể làm hỏng các chương trình hiện có.
Vấn đề là cả hai đều cung cấp một triển khai của interface
[`os.FileInfo`](/pkg/os/#FileInfo)
không tuân thủ đặc tả của interface đó.
Cụ thể, phương thức `Name` của chúng trả về tên đường dẫn đầy đủ
của mục, nhưng đặc tả interface yêu cầu
phương thức chỉ trả về tên cơ sở (phần tử đường dẫn cuối cùng).

_Cập nhật_: Vì hành vi này mới được triển khai và
hơi khó hiểu, có thể không có mã nguồn nào phụ thuộc vào hành vi bị hỏng.
Nếu có các chương trình phụ thuộc vào nó, chúng sẽ cần được xác định
và sửa thủ công.

### Gói encoding mới {#encoding}

Có một gói mới, [`encoding`](/pkg/encoding/),
định nghĩa một bộ các interface mã hóa chuẩn có thể được dùng để
xây dựng các bộ marshaler và unmarshaler tùy chỉnh cho các gói như
[`encoding/xml`](/pkg/encoding/xml/),
[`encoding/json`](/pkg/encoding/json/),
và
[`encoding/binary`](/pkg/encoding/binary/).
Các interface mới này đã được dùng để gọn gàng hóa một số triển khai trong
thư viện chuẩn.

Các interface mới được gọi là
[`BinaryMarshaler`](/pkg/encoding/#BinaryMarshaler),
[`BinaryUnmarshaler`](/pkg/encoding/#BinaryUnmarshaler),
[`TextMarshaler`](/pkg/encoding/#TextMarshaler),
và
[`TextUnmarshaler`](/pkg/encoding/#TextUnmarshaler).
Chi tiết đầy đủ có trong [tài liệu](/pkg/encoding/) cho gói
và một [tài liệu thiết kế](/s/go12encoding) riêng biệt.

### Gói fmt {#fmt_indexed_arguments}

Các quy trình in có định dạng của gói [`fmt`](/pkg/fmt/) như [`Printf`](/pkg/fmt/#Printf)
bây giờ cho phép các mục dữ liệu cần in được truy cập theo thứ tự tùy ý
bằng cách dùng một thao tác đánh chỉ số trong các đặc tả định dạng.
Bất cứ khi nào một đối số cần được tìm nạp từ danh sách đối số để định dạng,
dù là giá trị cần định dạng hay số nguyên width hoặc precision,
một ký hiệu đánh chỉ số tùy chọn mới `[`_n_`]`
tìm nạp đối số _n_ thay thế.
Giá trị của _n_ được đánh chỉ số từ 1.
Sau thao tác đánh chỉ số như vậy, đối số tiếp theo cần tìm nạp bởi quá trình xử lý bình thường
sẽ là _n_+1.

Ví dụ, lệnh gọi `Printf` bình thường

	fmt.Sprintf("%c %c %c\n", 'a', 'b', 'c')

sẽ tạo ra chuỗi `"a b c"`, nhưng với các thao tác đánh chỉ số như thế này,

	fmt.Sprintf("%[3]c %[1]c %c\n", 'a', 'b', 'c')

kết quả là `"c a b"`. Chỉ số `[3]` truy cập đối số định dạng thứ ba,
là `'c'`, `[1]` truy cập đối số đầu tiên, `'a'`,
và sau đó lần tìm nạp tiếp theo truy cập đối số sau đó, `'b'`.

Động lực cho tính năng này là các câu lệnh định dạng có thể lập trình để truy cập
các đối số theo thứ tự khác nhau cho việc bản địa hóa, nhưng nó có các cách dùng khác:

	log.Printf("trace: value %v of type %[1]T\n", expensiveFunction(a.b[c]))

_Cập nhật_: Thay đổi về cú pháp của các đặc tả định dạng
hoàn toàn tương thích ngược, vì vậy nó không ảnh hưởng đến bất kỳ chương trình nào đang hoạt động.

### Các gói text/template và html/template {#text_template}

Gói
[`text/template`](/pkg/text/template/)
có một số thay đổi trong Go 1.2, cả hai cũng được phản ánh trong gói
[`html/template`](/pkg/html/template/).

Đầu tiên, có các hàm mặc định mới để so sánh các kiểu cơ bản.
Các hàm được liệt kê trong bảng này, hiển thị tên của chúng và
toán tử so sánh liên quan.

<table cellpadding="0" summary="Template comparison functions">
<tbody><tr>
<th width="50"></th><th width="100">Tên</th> <th width="50">Toán tử</th>
</tr>
<tr>
<td></td><td><code>eq</code></td> <td><code>==</code></td>
</tr>
<tr>
<td></td><td><code>ne</code></td> <td><code>!=</code></td>
</tr>
<tr>
<td></td><td><code>lt</code></td> <td><code>&lt;</code></td>
</tr>
<tr>
<td></td><td><code>le</code></td> <td><code>&lt;=</code></td>
</tr>
<tr>
<td></td><td><code>gt</code></td> <td><code>&gt;</code></td>
</tr>
<tr>
<td></td><td><code>ge</code></td> <td><code>&gt;=</code></td>
</tr>
</tbody></table>

Các hàm này hoạt động hơi khác so với các toán tử Go tương ứng.
Đầu tiên, chúng chỉ hoạt động trên các kiểu cơ bản (`bool`, `int`,
`float64`, `string`, v.v.).
(Go cho phép so sánh mảng và struct cũng vậy, trong một số trường hợp nhất định.)
Thứ hai, các giá trị có thể được so sánh miễn là chúng là cùng loại giá trị:
bất kỳ giá trị số nguyên có dấu nào đều có thể được so sánh với bất kỳ giá trị số nguyên có dấu nào khác chẳng hạn. (Go
không cho phép so sánh một `int8` và một `int16`).
Cuối cùng, hàm `eq` (duy nhất) cho phép so sánh đối số đầu tiên
với một hoặc nhiều đối số tiếp theo. Template trong ví dụ này,

	{{if eq .A 1 2 3}} equal {{else}} not equal {{end}}

báo cáo "equal" nếu `.A` bằng _bất kỳ_ trong số 1, 2 hoặc 3.

Thay đổi thứ hai là một bổ sung nhỏ vào ngữ pháp làm cho các chuỗi "if else if" dễ viết hơn.
Thay vì viết,

	{{if eq .A 1}} X {{else}} {{if eq .A 2}} Y {{end}} {{end}}

người ta có thể gấp "if" thứ hai vào "else" và chỉ có một "end", như thế này:

	{{if eq .A 1}} X {{else if eq .A 2}} Y {{end}}

Hai hình thức có tác dụng giống hệt nhau; sự khác biệt chỉ là về cú pháp.

_Cập nhật_: Cả thay đổi "else if" lẫn các hàm so sánh
không ảnh hưởng đến các chương trình hiện có. Những chương trình
đã định nghĩa các hàm gọi là `eq` v.v. thông qua một function
map không bị ảnh hưởng vì function map liên quan sẽ ghi đè các
định nghĩa hàm mặc định mới.

### Các gói mới {#new_packages}

Có hai gói mới.

  - Gói [`encoding`](/pkg/encoding/) được
    [mô tả ở trên](#encoding).
  - Gói [`image/color/palette`](/pkg/image/color/palette/)
    cung cấp các bảng màu chuẩn.

### Thay đổi nhỏ đối với thư viện {#minor_library_changes}

Danh sách sau đây tóm tắt một số thay đổi nhỏ đối với thư viện, chủ yếu là bổ sung.
Xem tài liệu gói liên quan để biết thêm thông tin về từng thay đổi.

  - Gói [`archive/zip`](/pkg/archive/zip/)
    thêm accessor
    [`DataOffset`](/pkg/archive/zip/#File.DataOffset)
    để trả về offset của dữ liệu (có thể đã nén) của file trong archive.
  - Gói [`bufio`](/pkg/bufio/)
    thêm các phương thức [`Reset`](/pkg/bufio/#Reader.Reset)
    cho [`Reader`](/pkg/bufio/#Reader) và
    [`Writer`](/pkg/bufio/#Writer).
    Các phương thức này cho phép [`Reader`](/pkg/io/#Reader)
    và [`Writer`](/pkg/io/#Writer)
    được tái sử dụng trên đầu vào và đầu ra mới, tiết kiệm
    overhead cấp phát.
  - [`compress/bzip2`](/pkg/compress/bzip2/)
    bây giờ có thể giải nén các archive được nối.
  - Gói [`compress/flate`](/pkg/compress/flate/)
    thêm phương thức [`Reset`](/pkg/compress/flate/#Writer.Reset)
    trên [`Writer`](/pkg/compress/flate/#Writer),
    để có thể giảm cấp phát khi, ví dụ, xây dựng một
    archive để chứa nhiều file nén.
  - Kiểu [`Writer`](/pkg/compress/gzip/#Writer) của gói
    [`compress/gzip`](/pkg/compress/gzip/)
    thêm phương thức [`Reset`](/pkg/compress/gzip/#Writer.Reset)
    để nó có thể được tái sử dụng.
  - Kiểu [`Writer`](/pkg/compress/zlib/#Writer) của gói
    [`compress/zlib`](/pkg/compress/zlib/)
    thêm phương thức [`Reset`](/pkg/compress/zlib/#Writer.Reset)
    để nó có thể được tái sử dụng.
  - Gói [`container/heap`](/pkg/container/heap/)
    thêm phương thức [`Fix`](/pkg/container/heap/#Fix)
    để cung cấp cách hiệu quả hơn để cập nhật vị trí của một mục trong heap.
  - Gói [`container/list`](/pkg/container/list/)
    thêm các phương thức [`MoveBefore`](/pkg/container/list/#List.MoveBefore)
    và
    [`MoveAfter`](/pkg/container/list/#List.MoveAfter),
    thực hiện sắp xếp lại rõ ràng.
  - Gói [`crypto/cipher`](/pkg/crypto/cipher/)
    thêm chế độ GCM mới (Galois Counter Mode), gần như luôn
    được dùng với mã hóa AES.
  - Gói
    [`crypto/md5`](/pkg/crypto/md5/)
    thêm hàm mới [`Sum`](/pkg/crypto/md5/#Sum)
    để đơn giản hóa hashing mà không ảnh hưởng đến hiệu năng.
  - Tương tự, gói
    [`crypto/sha1`](/pkg/crypto/md5/)
    thêm hàm mới [`Sum`](/pkg/crypto/sha1/#Sum).
  - Ngoài ra, gói
    [`crypto/sha256`](/pkg/crypto/sha256/)
    thêm các hàm [`Sum256`](/pkg/crypto/sha256/#Sum256)
    và [`Sum224`](/pkg/crypto/sha256/#Sum224).
  - Cuối cùng, gói [`crypto/sha512`](/pkg/crypto/sha512/)
    thêm các hàm [`Sum512`](/pkg/crypto/sha512/#Sum512) và
    [`Sum384`](/pkg/crypto/sha512/#Sum384).
  - Gói [`crypto/x509`](/pkg/crypto/x509/)
    thêm hỗ trợ để đọc và ghi các extension tùy ý.
  - Gói [`crypto/tls`](/pkg/crypto/tls/) thêm
    hỗ trợ cho TLS 1.1, 1.2 và AES-GCM.
  - Gói [`database/sql`](/pkg/database/sql/) thêm phương thức
    [`SetMaxOpenConns`](/pkg/database/sql/#DB.SetMaxOpenConns)
    trên [`DB`](/pkg/database/sql/#DB) để giới hạn
    số kết nối mở đến cơ sở dữ liệu.
  - Gói [`encoding/csv`](/pkg/encoding/csv/)
    bây giờ luôn cho phép dấu phẩy theo sau trên các field.
  - Gói [`encoding/gob`](/pkg/encoding/gob/)
    bây giờ xử lý các field channel và function của struct như thể chúng không được xuất,
    ngay cả khi chúng là. Tức là, nó bỏ qua chúng hoàn toàn. Trước đây chúng sẽ
    kích hoạt một lỗi, điều này có thể gây ra các vấn đề tương thích không mong muốn nếu một
    cấu trúc được nhúng thêm field như vậy.
    Gói cũng bây giờ hỗ trợ các interface `BinaryMarshaler` và
    `BinaryUnmarshaler` chung của gói
    [`encoding`](/pkg/encoding/)
    được mô tả ở trên.
  - Gói [`encoding/json`](/pkg/encoding/json/)
    bây giờ sẽ luôn escape các ký hiệu ampersand thành "\u0026" khi in chuỗi.
    Bây giờ nó sẽ chấp nhận nhưng sửa UTF-8 không hợp lệ trong
    [`Marshal`](/pkg/encoding/json/#Marshal)
    (đầu vào như vậy trước đây bị từ chối).
    Cuối cùng, bây giờ nó hỗ trợ các interface mã hóa chung của gói
    [`encoding`](/pkg/encoding/)
    được mô tả ở trên.
  - Gói [`encoding/xml`](/pkg/encoding/xml/)
    bây giờ cho phép các thuộc tính được lưu trong con trỏ để được marshal.
    Nó cũng hỗ trợ các interface mã hóa chung của gói
    [`encoding`](/pkg/encoding/)
    được mô tả ở trên thông qua các interface mới
    [`Marshaler`](/pkg/encoding/xml/#Marshaler),
    [`Unmarshaler`](/pkg/encoding/xml/#Unmarshaler),
    và các interface liên quan
    [`MarshalerAttr`](/pkg/encoding/xml/#MarshalerAttr) và
    [`UnmarshalerAttr`](/pkg/encoding/xml/#UnmarshalerAttr).
    Gói cũng thêm phương thức
    [`Flush`](/pkg/encoding/xml/#Encoder.Flush)
    vào kiểu
    [`Encoder`](/pkg/encoding/xml/#Encoder)
    để dùng bởi các encoder tùy chỉnh. Xem tài liệu cho
    [`EncodeToken`](/pkg/encoding/xml/#Encoder.EncodeToken)
    để biết cách dùng nó.
  - Gói [`flag`](/pkg/flag/) bây giờ
    có interface [`Getter`](/pkg/flag/#Getter)
    để cho phép giá trị của flag được lấy. Do
    các hướng dẫn tương thích Go 1, phương thức này không thể được thêm vào interface
    [`Value`](/pkg/flag/#Value) hiện có,
    nhưng tất cả các kiểu flag chuẩn hiện có đều triển khai nó.
    Gói cũng xuất tập flag [`CommandLine`](/pkg/flag/#CommandLine),
    chứa các flag từ dòng lệnh.
  - Cấu trúc [`SliceExpr`](/pkg/go/ast/#SliceExpr)
    của gói [`go/ast`](/pkg/go/ast/)
    có field boolean mới, `Slice3`, được đặt thành true
    khi biểu diễn một biểu thức slice với ba chỉ số (hai dấu hai chấm).
    Mặc định là false, biểu diễn dạng hai chỉ số thông thường.
  - Gói [`go/build`](/pkg/go/build/) thêm
    field `AllTags`
    vào kiểu [`Package`](/pkg/go/build/#Package),
    để dễ dàng xử lý các build tag hơn.
  - Gói [`image/draw`](/pkg/image/draw/) bây giờ
    xuất interface [`Drawer`](/pkg/image/draw/#Drawer),
    bao gồm phương thức [`Draw`](/pkg/image/draw/#Draw) chuẩn.
    Các toán tử Porter-Duff bây giờ triển khai interface này, thực tế là gắn kết một thao tác với
    toán tử draw thay vì cung cấp nó tường minh.
    Cho một ảnh paletted làm đích, triển khai mới
    [`FloydSteinberg`](/pkg/image/draw/#FloydSteinberg)
    của interface
    [`Drawer`](/pkg/image/draw/#Drawer)
    sẽ dùng thuật toán khuếch tán lỗi Floyd-Steinberg để vẽ ảnh.
    Để tạo các palette phù hợp cho quá trình xử lý như vậy, interface mới
    [`Quantizer`](/pkg/image/draw/#Quantizer)
    biểu diễn các triển khai của các thuật toán lượng tử hóa chọn palette
    cho một ảnh màu đầy đủ.
    Không có triển khai nào của interface này trong thư viện.
  - Gói [`image/gif`](/pkg/image/gif/)
    bây giờ có thể tạo các file GIF bằng các hàm mới
    [`Encode`](/pkg/image/gif/#Encode)
    và [`EncodeAll`](/pkg/image/gif/#EncodeAll).
    Đối số options của chúng cho phép chỉ định một
    [`Quantizer`](/pkg/image/draw/#Quantizer) ảnh để dùng;
    nếu là `nil`, GIF được tạo sẽ dùng bảng màu
    [`Plan9`](/pkg/image/color/palette/#Plan9)
    được định nghĩa trong gói mới
    [`image/color/palette`](/pkg/image/color/palette/).
    Options cũng chỉ định một
    [`Drawer`](/pkg/image/draw/#Drawer)
    để dùng để tạo ảnh đầu ra;
    nếu là `nil`, khuếch tán lỗi Floyd-Steinberg được dùng.
  - Phương thức [`Copy`](/pkg/io/#Copy) của gói
    [`io`](/pkg/io/) bây giờ ưu tiên các
    đối số của nó theo cách khác.
    Nếu một đối số triển khai [`WriterTo`](/pkg/io/#WriterTo)
    và đối số kia triển khai [`ReaderFrom`](/pkg/io/#ReaderFrom),
    [`Copy`](/pkg/io/#Copy) bây giờ sẽ gọi
    [`WriterTo`](/pkg/io/#WriterTo) để thực hiện công việc,
    vì vậy cần ít buffer trung gian hơn nói chung.
  - Gói [`net`](/pkg/net/) yêu cầu cgo theo mặc định
    vì hệ điều hành máy chủ nói chung phải làm trung gian cho việc thiết lập lệnh gọi mạng.
    Tuy nhiên trên một số hệ thống, có thể dùng mạng mà không cần cgo, và hữu ích
    khi làm như vậy, chẳng hạn để tránh liên kết động.
    Build tag mới `netgo` (tắt theo mặc định) cho phép xây dựng một
    gói `net` thuần Go trên các hệ thống có thể thực hiện được.
  - Gói [`net`](/pkg/net/) thêm field mới
    `DualStack` vào cấu trúc [`Dialer`](/pkg/net/#Dialer)
    cho việc thiết lập kết nối TCP bằng dual IP stack như được mô tả trong
    [RFC 6555](https://tools.ietf.org/html/rfc6555).
  - Gói [`net/http`](/pkg/net/http/) sẽ không còn
    truyền các cookie không chính xác theo
    [RFC 6265](https://tools.ietf.org/html/rfc6265).
    Nó chỉ ghi nhật ký một lỗi và không gửi gì.
    Ngoài ra,
    hàm [`ReadResponse`](/pkg/net/http/#ReadResponse)
    của gói [`net/http`](/pkg/net/http/)
    bây giờ cho phép tham số `*Request` là `nil`,
    trong trường hợp đó nó giả định là GET request.
    Cuối cùng, một HTTP server bây giờ sẽ phục vụ các yêu cầu HEAD
    trong suốt, không cần trường hợp đặc biệt trong mã handler.
    Khi phục vụ một yêu cầu HEAD, các ghi vào
    [`ResponseWriter`](/pkg/net/http/#ResponseWriter)
    của [`Handler`](/pkg/net/http/#Handler)
    được hấp thụ bởi
    [`Server`](/pkg/net/http/#Server)
    và client nhận một body rỗng theo yêu cầu của đặc tả HTTP.
  - Phương thức [`Cmd.StdinPipe`](/pkg/os/exec/#Cmd.StdinPipe)
    của gói [`os/exec`](/pkg/os/exec/)
    trả về một `io.WriteCloser`, nhưng đã thay đổi triển khai cụ thể
    từ `*os.File` thành một kiểu không xuất nhúng
    `*os.File`, và bây giờ an toàn để đóng giá trị được trả về.
    Trước Go 1.2, có một race không thể tránh khỏi mà thay đổi này sửa.
    Mã nguồn cần truy cập các phương thức của `*os.File` có thể dùng
    một type assertion interface, chẳng hạn như `wc.(interface{ Sync() error })`.
  - Gói [`runtime`](/pkg/runtime/) nới lỏng
    các ràng buộc đối với các hàm finalizer trong
    [`SetFinalizer`](/pkg/runtime/#SetFinalizer): đối số
    thực tế bây giờ có thể là bất kỳ kiểu nào có thể gán cho kiểu hình thức của
    hàm, như trong bất kỳ lệnh gọi hàm bình thường nào trong Go.
  - Gói [`sort`](/pkg/sort/) có hàm mới
    [`Stable`](/pkg/sort/#Stable) triển khai
    sắp xếp ổn định. Tuy nhiên nó kém hiệu quả hơn so với thuật toán sắp xếp thông thường.
  - Gói [`strings`](/pkg/strings/) thêm
    hàm [`IndexByte`](/pkg/strings/#IndexByte)
    để nhất quán với gói [`bytes`](/pkg/bytes/).
  - Gói [`sync/atomic`](/pkg/sync/atomic/)
    thêm một tập hợp mới các hàm swap trao đổi nguyên tử đối số với
    giá trị được lưu trong con trỏ, trả về giá trị cũ.
    Các hàm là
    [`SwapInt32`](/pkg/sync/atomic/#SwapInt32),
    [`SwapInt64`](/pkg/sync/atomic/#SwapInt64),
    [`SwapUint32`](/pkg/sync/atomic/#SwapUint32),
    [`SwapUint64`](/pkg/sync/atomic/#SwapUint64),
    [`SwapUintptr`](/pkg/sync/atomic/#SwapUintptr),
    và
    [`SwapPointer`](/pkg/sync/atomic/#SwapPointer),
    trao đổi một `unsafe.Pointer`.
  - Gói [`syscall`](/pkg/syscall/) bây giờ triển khai
    [`Sendfile`](/pkg/syscall/#Sendfile) cho Darwin.
  - Gói [`testing`](/pkg/testing/)
    bây giờ xuất interface [`TB`](/pkg/testing/#TB).
    Nó ghi lại các phương thức chung với các kiểu
    [`T`](/pkg/testing/#T)
    và
    [`B`](/pkg/testing/#B),
    để dễ dàng chia sẻ mã giữa các test và benchmark hơn.
    Ngoài ra, hàm
    [`AllocsPerRun`](/pkg/testing/#AllocsPerRun)
    bây giờ lượng tử hóa giá trị trả về thành một số nguyên (mặc dù nó
    vẫn có kiểu `float64`), để làm tròn bất kỳ lỗi nào gây ra bởi
    khởi tạo và làm cho kết quả lặp lại được hơn.
  - Gói [`text/template`](/pkg/text/template/)
    bây giờ tự động dereference các giá trị con trỏ khi đánh giá các đối số
    cho các hàm "escape" như "html", để đưa hành vi của các hàm đó
    vào sự đồng thuận với các hàm in khác như "printf".
  - Trong gói [`time`](/pkg/time/), hàm
    [`Parse`](/pkg/time/#Parse)
    và phương thức
    [`Format`](/pkg/time/#Time.Format)
    bây giờ xử lý các offset múi giờ với giây, chẳng hạn trong ngày lịch sử
    "1871-01-01T05:33:02+00:34:08".
    Ngoài ra, khớp pattern trong các định dạng cho các quy trình đó nghiêm ngặt hơn: một chữ cái không viết thường
    bây giờ phải theo sau các từ chuẩn như "Jan" và "Mon".
  - Gói [`unicode`](/pkg/unicode/)
    thêm [`In`](/pkg/unicode/#In),
    một phiên bản đẹp hơn nhưng tương đương với bản gốc
    [`IsOneOf`](/pkg/unicode/#IsOneOf),
    để xem liệu một ký tự có phải là thành viên của một danh mục Unicode không.
