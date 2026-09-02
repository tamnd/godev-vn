---
path: /doc/go1.22
title: Ghi chú phát hành Go 1.22
---

<!--
NOTE: In this document and others in this directory, the convention is to
set fixed-width phrases with non-fixed-width spaces, as in
<code>hello</code> <code>world</code>.
Do not send CLs removing the interior tags from such phrases.
-->

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu Go 1.22 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.22, ra mắt sáu tháng sau [Go 1.21](/doc/go1.21).
Phần lớn các thay đổi nằm ở phần triển khai toolchain, runtime và thư viện.
Như thường lệ, bản phát hành duy trì [cam kết tương thích](/doc/go1compat) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

## Thay đổi ngôn ngữ {#language}

<!-- loop variable scope -->
<!-- range over int -->
Go 1.22 thực hiện hai thay đổi đối với vòng lặp `for`.

  - Trước đây, các biến được khai báo trong vòng lặp `for` được tạo một lần và cập nhật sau mỗi lần lặp. Trong Go 1.22, mỗi lần lặp tạo ra các biến mới, nhằm tránh các lỗi chia sẻ vô tình.
    [Hỗ trợ chuyển đổi](/wiki/LoopvarExperiment#my-test-fails-with-the-change-how-can-i-debug-it)
    được mô tả trong đề xuất vẫn hoạt động theo cách tương tự như trong Go 1.21.
  - Vòng lặp `for` giờ có thể duyệt qua các số nguyên.
    Ví dụ [như sau](/play/p/ky02zZxgk_r?v=gotip):

    	package main

    	import "fmt"

    	func main() {
    	  for i := range 10 {
    	    fmt.Println(10 - i)
    	  }
    	  fmt.Println("go1.22 has lift-off!")
    	}

    Xem đặc tả ngôn ngữ để biết [chi tiết](/ref/spec#For_range).

<!-- range over func GOEXPERIMENT; https://go.dev/issue/61405, https://go.dev/issue/61897, CLs 510541,539277,540263,543319 -->


Go 1.22 bao gồm bản xem trước về một thay đổi ngôn ngữ đang được xem xét
cho phiên bản Go trong tương lai: [trình lặp range-over-function](/wiki/RangefuncExperiment).
Xây dựng với `GOEXPERIMENT=rangefunc` sẽ kích hoạt tính năng này.

## Công cụ {#tools}

### Lệnh Go {#go-command}

<!-- https://go.dev/issue/60056 -->

Các lệnh trong [workspace](/ref/mod#workspaces) giờ có thể
sử dụng thư mục `vendor` chứa các dependency của
workspace. Thư mục này được tạo bởi
[`go` `work` `vendor`](/pkg/cmd/go#hdr-Make_vendored_copy_of_dependencies),
và được dùng bởi các lệnh build khi cờ `-mod` được đặt thành
`vendor`, đây là giá trị mặc định khi thư mục `vendor`
của workspace tồn tại.

Lưu ý rằng nội dung của thư mục `vendor` cho một workspace khác
với nội dung của một module đơn: nếu thư mục gốc của workspace cũng
chứa một trong các module trong workspace, thư mục `vendor` của nó
có thể chứa các dependency của workspace hoặc của module,
nhưng không thể chứa cả hai.

<!-- CL 518775, https://go.dev/issue/60915 -->

`go` `get` không còn được hỗ trợ bên ngoài module trong chế độ
`GOPATH` cũ (tức là với `GO111MODULE=off`).
Các lệnh build khác, như `go` `build` và
`go` `test`, sẽ tiếp tục hoạt động không giới hạn thời gian
cho các chương trình `GOPATH` cũ.

<!-- CL 518776 -->

`go` `mod` `init` không còn cố gắng nhập
các yêu cầu module từ các tệp cấu hình dành cho các công cụ vendoring khác
(như `Gopkg.lock`).

<!-- CL 495447 -->

`go` `test` `-cover` giờ in tóm tắt độ bao phủ
cho các gói đã được bao phủ nhưng không có tệp kiểm thử riêng. Trước Go 1.22, một
lần chạy `go` `test` `-cover` cho gói như vậy sẽ
báo cáo

`?     mymod/mypack    [no test files]`

và giờ với Go 1.22, các hàm trong gói được coi là chưa được bao phủ:

`mymod/mypack    coverage: 0.0% of statements`

Lưu ý rằng nếu một gói không chứa bất kỳ mã thực thi nào, chúng ta không thể báo cáo
tỷ lệ bao phủ có ý nghĩa; với các gói như vậy, công cụ `go`
sẽ tiếp tục báo cáo rằng không có tệp kiểm thử.

<!-- CL 522239, https://go.dev/issue/46330 -->

Các lệnh build `go` gọi đến linker giờ sẽ báo lỗi nếu
linker bên ngoài (C) sẽ được sử dụng nhưng cgo chưa được bật. (Runtime Go
yêu cầu hỗ trợ cgo để đảm bảo tương thích với các thư viện bổ sung
được thêm bởi linker C.)

### Trace {#trace}

<!-- https://go.dev/issue/63960 -->

Giao diện web của công cụ `trace` đã được làm mới nhẹ nhàng như một phần của
công việc hỗ trợ trình tracer mới, giải quyết một số vấn đề và cải thiện khả năng
đọc của các trang phụ.
Giao diện web giờ hỗ trợ khám phá các trace theo dạng xem hướng luồng (thread-oriented).
Trình xem trace giờ cũng hiển thị toàn bộ thời lượng của tất cả các lệnh gọi hệ thống.
\
Những cải tiến này chỉ áp dụng cho việc xem các trace được tạo ra bởi các chương trình xây dựng với
Go 1.22 trở lên.
Một bản phát hành tương lai sẽ mang một số cải tiến này đến các trace được tạo bởi phiên bản Go cũ hơn.

### Vet {#vet}

#### Tham chiếu đến biến vòng lặp {#vet-loopclosure}

<!-- CL 539016, https://go.dev/issue/63888: cmd/vet: do not report variable capture for loop variables with the new lifetime rules -->
Hành vi của công cụ `vet` đã thay đổi để phù hợp với
ngữ nghĩa mới (xem ở trên) của các biến vòng lặp trong Go 1.22.
Khi phân tích một tệp yêu cầu Go 1.22 trở lên
(do tệp go.mod hoặc ràng buộc build theo từng tệp),
`vet` không còn báo cáo các tham chiếu đến
biến vòng lặp từ bên trong function literal có thể
tồn tại lâu hơn lần lặp.
Trong Go 1.22, các biến vòng lặp được tạo mới cho mỗi lần lặp,
vì vậy các tham chiếu như vậy không còn có nguy cơ sử dụng biến
sau khi nó đã được cập nhật bởi vòng lặp.

#### Cảnh báo mới về giá trị thiếu sau append {#vet-appends}

<!-- CL 498416, https://go.dev/issue/60448: add a new analyzer for check missing values after append -->
Công cụ `vet` giờ báo cáo các lệnh gọi đến
[`append`](/pkg/builtin/#append) không truyền
giá trị nào để thêm vào slice, như `slice = append(slice)`.
Câu lệnh như vậy không có tác dụng gì, và kinh nghiệm cho thấy đây hầu như luôn là lỗi.

#### Cảnh báo mới về việc defer `time.Since` {#vet-defers}

<!-- CL 527095, https://go.dev/issue/60048: time.Since should not be used in defer statement -->
Công cụ vet giờ báo cáo lệnh gọi không được defer đến
[`time.Since(t)`](/pkg/time/#Since) trong câu lệnh `defer`.
Điều này tương đương với việc gọi `time.Now().Sub(t)` trước câu lệnh `defer`,
chứ không phải khi hàm được defer được gọi. Trong hầu hết các trường hợp, mã đúng
yêu cầu defer lệnh gọi `time.Since`. Ví dụ:

	t := time.Now()
	defer log.Println(time.Since(t)) // non-deferred call to time.Since
	tmp := time.Since(t); defer log.Println(tmp) // equivalent to the previous defer

	defer func() {
	  log.Println(time.Since(t)) // a correctly deferred call to time.Since
	}()

#### Cảnh báo mới về các cặp key-value không khớp trong lệnh gọi `log/slog` {#vet-slog}

<!-- CL 496156, https://go.dev/issue/59407: log/slog: add vet checks for variadic ...any inputs -->
Công cụ vet giờ báo cáo các đối số không hợp lệ trong các lệnh gọi đến hàm và phương thức
trong gói structured logging, [`log/slog`](/pkg/log/slog),
nhận các cặp key/value xen kẽ.
Nó báo cáo các lệnh gọi mà một đối số ở vị trí key không phải là
`string` hay `slog.Attr`, và nơi key cuối cùng thiếu giá trị của nó.

## Runtime {#runtime}

<!-- CL 543255 -->
Runtime giờ lưu trữ metadata bộ gom rác dựa trên kiểu gần hơn với từng
đối tượng trên heap, cải thiện hiệu suất CPU (độ trễ hoặc thông lượng) của các chương trình Go
từ 1 đến 3%.
Thay đổi này cũng giảm tổng bộ nhớ sử dụng của phần lớn các chương trình Go
khoảng 1% bằng cách loại bỏ trùng lặp metadata dư thừa.
Một số chương trình có thể thấy cải thiện nhỏ hơn vì thay đổi này điều chỉnh ranh giới
lớp kích thước của bộ cấp phát bộ nhớ, khiến một số đối tượng có thể bị chuyển lên lớp kích thước cao hơn.

Hệ quả của thay đổi này là địa chỉ của một số đối tượng trước đây
luôn được căn chỉnh theo ranh giới 16 byte (hoặc cao hơn) giờ chỉ được căn chỉnh theo ranh giới 8
byte.
Một số chương trình sử dụng lệnh assembly yêu cầu địa chỉ bộ nhớ phải
được căn chỉnh hơn 8 byte và dựa vào hành vi căn chỉnh cũ của bộ cấp phát bộ nhớ
có thể bị lỗi, nhưng chúng tôi kỳ vọng những chương trình như vậy là hiếm.
Các chương trình đó có thể được xây dựng với `GOEXPERIMENT=noallocheaders` để quay lại
bố cục metadata cũ và khôi phục hành vi căn chỉnh trước đây, nhưng
chủ gói nên cập nhật mã assembly của họ để tránh giả định căn chỉnh, vì giải pháp tạm thời này
sẽ bị xóa trong bản phát hành tương lai.

<!-- CL 525475 -->
Trên cổng `windows/amd64`, các chương trình liên kết hoặc tải các thư viện Go được xây dựng với
`-buildmode=c-archive` hoặc `-buildmode=c-shared` giờ có thể dùng
hàm Win32 `SetUnhandledExceptionFilter` để bắt các ngoại lệ không được xử lý
bởi runtime Go. Lưu ý rằng điều này đã được hỗ trợ trên cổng `windows/386`.

## Trình biên dịch {#compiler}

<!-- https://go.dev/issue/61577 -->
Các bản build [tối ưu hóa dựa trên hồ sơ thực thi (PGO)](/doc/pgo)
giờ có thể devirtualize một tỷ lệ lệnh gọi cao hơn so với trước đây.
Hầu hết các chương trình từ bộ chương trình Go đại diện giờ thấy cải thiện từ 2 đến
14% lúc runtime khi bật PGO.

<!-- https://go.dev/cl/528321 -->
Trình biên dịch giờ xen kẽ devirtualization và inlining, vì vậy các
lệnh gọi phương thức interface được tối ưu hóa tốt hơn.

<!-- https://go.dev/issue/61502 -->
Go 1.22 cũng bao gồm bản xem trước về một triển khai nâng cao của giai đoạn inlining của trình biên dịch, sử dụng heuristic để thúc đẩy khả năng inline tại các điểm gọi được coi là "quan trọng" (ví dụ, trong vòng lặp) và ngăn cản việc inline tại các điểm gọi được coi là "không quan trọng" (ví dụ, trên đường dẫn panic).
Xây dựng với `GOEXPERIMENT=newinliner` kích hoạt heuristic điểm gọi mới;
xem [issue #61502](/issue/61502) để biết thêm thông tin và cung cấp phản hồi.

## Linker {#linker}

<!-- CL 493136 -->
Các cờ `-s` và `-w` của linker giờ hoạt động nhất quán hơn
trên tất cả các nền tảng.
Cờ `-w` ngăn chặn việc tạo thông tin debug DWARF.
Cờ `-s` ngăn chặn việc tạo bảng symbol.
Cờ `-s` cũng ngụ ý cờ `-w`, có thể bị phủ nhận bằng `-w=0`.
Tức là, `-s` `-w=0` sẽ tạo ra tệp nhị phân với thông tin debug DWARF nhưng không có bảng symbol.

<!-- CL 511475 -->
Trên các nền tảng ELF, cờ linker `-B` giờ chấp nhận một dạng đặc biệt:
với `-B` `gobuildid`, linker sẽ tạo GNU build ID (note ELF `NT_GNU_BUILD_ID`) được dẫn xuất từ Go build ID.

<!-- CL 534555 -->
Trên Windows, khi xây dựng với `-linkmode=internal`, linker giờ
bảo tồn thông tin SEH từ các tệp đối tượng C bằng cách sao chép các section `.pdata`
và `.xdata` vào tệp nhị phân cuối cùng.
Điều này giúp ích cho việc debug và profiling tệp nhị phân bằng các công cụ native, như WinDbg.
Lưu ý rằng cho đến nay, các trình xử lý ngoại lệ SEH của các hàm C không được tôn trọng,
vì vậy thay đổi này có thể khiến một số chương trình hoạt động khác đi.
`-linkmode=external` không bị ảnh hưởng bởi thay đổi này, vì các linker bên ngoài
đã bảo tồn thông tin SEH.

## Bootstrap {#bootstrap}

Như đã đề cập trong [ghi chú phát hành Go 1.20](/doc/go1.20#bootstrap), Go 1.22 giờ yêu cầu
bản phát hành điểm cuối cùng của Go 1.20 trở lên để bootstrap.
Chúng tôi kỳ vọng Go 1.24 sẽ yêu cầu bản phát hành điểm cuối cùng của Go 1.22 trở lên để bootstrap.

## Thư viện chuẩn {#library}

### Gói math/rand/v2 mới {#math_rand_v2}

<!-- CL 502495 -->
<!-- CL 502497 -->
<!-- CL 502498 -->
<!-- CL 502499 -->
<!-- CL 502500 -->
<!-- CL 502505 -->
<!-- CL 502506 -->
<!-- CL 516857 -->
<!-- CL 516859 -->

Go 1.22 bao gồm gói "v2" đầu tiên trong thư viện chuẩn,
[`math/rand/v2`](/pkg/math/rand/v2/).
Các thay đổi so với [`math/rand`](/pkg/math/rand/) được
chi tiết trong [đề xuất #61716](/issue/61716). Các thay đổi quan trọng nhất là:

  - Phương thức `Read`, đã bị deprecated trong `math/rand`,
    không được mang sang `math/rand/v2`.
    (Nó vẫn còn trong `math/rand`.)
    Phần lớn các lệnh gọi đến `Read` nên dùng
    [`Read` của `crypto/rand`](/pkg/crypto/rand/#Read) thay thế.
    Nếu không, có thể xây dựng một `Read` tùy chỉnh bằng phương thức `Uint64`.
  - Bộ tạo ngẫu nhiên toàn cục được truy cập qua các hàm cấp cao nhất được seeded ngẫu nhiên vô điều kiện.
    Vì API đảm bảo không có chuỗi kết quả cố định,
    các tối ưu hóa như trạng thái bộ tạo ngẫu nhiên theo luồng giờ trở nên khả thi.
  - Interface [`Source`](/pkg/math/rand/v2/#Source)
    giờ chỉ có một phương thức `Uint64`;
    không còn interface `Source64`.
  - Nhiều phương thức giờ sử dụng các thuật toán nhanh hơn mà không thể áp dụng trong `math/rand`
    vì chúng thay đổi các luồng đầu ra.
  - Các hàm và phương thức cấp cao nhất
    `Intn`,
    `Int31`,
    `Int31n`,
    `Int63`,
    và
    `Int64n`
    từ `math/rand`
    được đặt tên theo phong cách thành ngữ hơn trong `math/rand/v2`:
    `IntN`,
    `Int32`,
    `Int32N`,
    `Int64`,
    và
    `Int64N`.
    Còn có thêm các hàm và phương thức cấp cao nhất mới
    `Uint32`,
    `Uint32N`,
    `Uint64`,
    `Uint64N`,
    và
    `UintN`.
  - Hàm generic mới [`N`](/pkg/math/rand/v2/#N)
    tương tự như
    [`Int64N`](/pkg/math/rand/v2/#Int64N) hoặc
    [`Uint64N`](/pkg/math/rand/v2/#Uint64N)
    nhưng hoạt động cho bất kỳ kiểu số nguyên nào.
    Ví dụ, một khoảng thời gian ngẫu nhiên từ 0 đến 5 phút là
    `rand.N(5*time.Minute)`.
  - Bộ tạo LFSR của Mitchell & Reeds được cung cấp bởi
    [`Source` của `math/rand`](/pkg/math/rand/#Source)
    đã được thay thế bằng hai nguồn bộ tạo số giả ngẫu nhiên hiện đại hơn:
    [`ChaCha8`](/pkg/math/rand/v2/#ChaCha8) và
    [`PCG`](/pkg/math/rand/v2/#PCG).
    ChaCha8 là bộ tạo số ngẫu nhiên mới, mạnh về mật mã
    có hiệu suất tương tự PCG.
    ChaCha8 là thuật toán được sử dụng cho các hàm cấp cao nhất trong `math/rand/v2`.
    Kể từ Go 1.22, các hàm cấp cao nhất của `math/rand` (khi không được seeded tường minh)
    và runtime Go cũng sử dụng ChaCha8 cho tính ngẫu nhiên.

Chúng tôi có kế hoạch đưa vào một công cụ di chuyển API trong bản phát hành tương lai, có thể là Go 1.23.

### Gói go/version mới {#go-version}

<!-- https://go.dev/issue/62039, https://go.dev/cl/538895 -->
Gói mới [`go/version`](/pkg/go/version/) triển khai các hàm
để xác thực và so sánh các chuỗi phiên bản Go.

### Các mẫu routing nâng cao {#enhanced_routing_patterns}

<!-- https://go.dev/issue/61410 -->
HTTP routing trong thư viện chuẩn giờ linh hoạt hơn.
Các mẫu được dùng bởi [`net/http.ServeMux`](/pkg/net/http#ServeMux) đã được nâng cao để chấp nhận các phương thức và ký tự đại diện.

Đăng ký một handler với một phương thức, như `"POST /items/create"`, giới hạn
các lệnh gọi đến handler cho các yêu cầu với phương thức đã cho. Một mẫu có phương thức được ưu tiên hơn một mẫu khớp không có phương thức.
Một trường hợp đặc biệt là đăng ký một handler với `  "GET" ` cũng đăng ký nó với `"HEAD"`.

Các ký tự đại diện trong mẫu, như `/items/{id}`, khớp với các segment của đường dẫn URL.
Giá trị segment thực tế có thể được truy cập bằng cách gọi phương thức [`Request.PathValue`](/pkg/net/http#Request.PathValue).
Một ký tự đại diện kết thúc bằng "...", như `/files/{path...}`, phải xuất hiện ở cuối mẫu và khớp với tất cả các segment còn lại.

Một mẫu kết thúc bằng "/" khớp với tất cả các đường dẫn có nó làm tiền tố, như trước đây.
Để khớp chính xác với mẫu bao gồm dấu gạch chéo cuối, kết thúc nó bằng `{$}`,
như trong `/exact/match/{$}`.

Nếu hai mẫu chồng chéo nhau về các yêu cầu khớp, thì mẫu cụ thể hơn sẽ được ưu tiên.
Nếu không có mẫu nào cụ thể hơn, các mẫu sẽ xung đột.
Quy tắc này tổng quát hóa các quy tắc ưu tiên gốc và duy trì thuộc tính là thứ tự đăng ký
mẫu không quan trọng.

Thay đổi này phá vỡ tương thích ngược theo những cách nhỏ, một số rõ ràng, các mẫu với "{" và "}" hoạt động khác đi,
và một số ít rõ ràng hơn, xử lý các đường dẫn được escape đã được cải thiện.
Thay đổi này được kiểm soát bởi trường [`GODEBUG`](/doc/godebug) có tên `httpmuxgo121`.
Đặt `httpmuxgo121=1` để khôi phục hành vi cũ.

### Thay đổi nhỏ trong thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ trong thư viện,
được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1 trong tâm trí.
Cũng có nhiều cải tiến hiệu suất không được liệt kê ở đây.

[archive/tar](/pkg/archive/tar/)

:   <!-- https://go.dev/issue/58000, CL 513316 -->
    Phương thức mới [`Writer.AddFS`](/pkg/archive/tar#Writer.AddFS) thêm tất cả các tệp từ một [`fs.FS`](/pkg/io/fs#FS) vào kho lưu trữ.

<!-- archive/tar -->

[archive/zip](/pkg/archive/zip/)

:   <!-- https://go.dev/issue/54898, CL 513438 -->
    Phương thức mới [`Writer.AddFS`](/pkg/archive/zip#Writer.AddFS) thêm tất cả các tệp từ một [`fs.FS`](/pkg/io/fs#FS) vào kho lưu trữ.

<!-- archive/zip -->

[bufio](/pkg/bufio/)

:   <!-- https://go.dev/issue/56381, CL 498117 -->
    Khi một [`SplitFunc`](/pkg/bufio#SplitFunc) trả về [`ErrFinalToken`](/pkg/bufio#ErrFinalToken) với token `nil`, [`Scanner`](/pkg/bufio#Scanner) giờ sẽ dừng ngay lập tức.
    Trước đây, nó sẽ báo cáo một token rỗng cuối cùng trước khi dừng, điều này thường không mong muốn.
    Các caller muốn báo cáo một token rỗng cuối cùng có thể làm điều đó bằng cách trả về `[]byte{}` thay vì `nil`.

<!-- bufio -->

[cmp](/pkg/cmp/)

:   <!-- https://go.dev/issue/60204 -->
    <!-- CL 504883 -->
    Hàm mới `Or` trả về phần tử đầu tiên trong một chuỗi giá trị không phải là giá trị zero.

<!-- cmp -->

[crypto/tls](/pkg/crypto/tls/)

:   <!-- https://go.dev/issue/43922, CL 544155 -->
    [`ConnectionState.ExportKeyingMaterial`](/pkg/crypto/tls#ConnectionState.ExportKeyingMaterial) giờ sẽ
    trả về lỗi trừ khi TLS 1.3 đang được sử dụng, hoặc extension `extended_master_secret` được hỗ trợ bởi cả server và
    client. `crypto/tls` đã hỗ trợ extension này từ Go 1.20. Điều này có thể bị vô hiệu hóa với
    cài đặt GODEBUG `tlsunsafeekm=1`.

    <!-- https://go.dev/issue/62459, CL 541516 -->
    Theo mặc định, phiên bản tối thiểu được cung cấp bởi các server `crypto/tls` giờ là TLS 1.2 nếu không được chỉ định với
    [`config.MinimumVersion`](/pkg/crypto/tls#Config.MinimumVersion), khớp với hành vi của các client `crypto/tls`.
    Thay đổi này có thể được hoàn nguyên với cài đặt GODEBUG `tls10server=1`.

    <!-- https://go.dev/issue/63413, CL 541517 -->
    Theo mặc định, các bộ mã hóa không có hỗ trợ ECDHE không còn được cung cấp bởi client hay server trong quá trình bắt tay trước TLS 1.3.
    Thay đổi này có thể được hoàn nguyên với cài đặt GODEBUG `tlsrsakex=1`.

<!-- crypto/tls -->

[crypto/x509](/pkg/crypto/x509/)

:   <!-- https://go.dev/issue/57178 -->
    Phương thức mới [`CertPool.AddCertWithConstraint`](/pkg/crypto/x509#CertPool.AddCertWithConstraint)
    có thể được dùng để thêm các ràng buộc tùy chỉnh vào chứng chỉ gốc được áp dụng trong quá trình xây dựng chuỗi.

    <!-- https://go.dev/issue/58922, CL 519315-->
    Trên Android, chứng chỉ gốc giờ sẽ được tải từ `/data/misc/keychain/certs-added` cũng như `/system/etc/security/cacerts`.

    <!-- https://go.dev/issue/60665, CL 520535 -->
    Kiểu mới, [`OID`](/pkg/crypto/x509#OID), hỗ trợ Object Identifiers ASN.1 với các thành phần riêng lẻ lớn hơn 31 bit. Một trường mới sử dụng kiểu này, [`Policies`](/pkg/crypto/x509#Certificate.Policies),
    được thêm vào struct `Certificate`, và giờ được điền trong quá trình phân tích. Bất kỳ OID nào không thể được biểu diễn
    bằng [`asn1.ObjectIdentifier`](/pkg/encoding/asn1#ObjectIdentifier) sẽ xuất hiện trong `Policies`,
    nhưng không phải trong trường `PolicyIdentifiers` cũ.
    Khi gọi [`CreateCertificate`](/pkg/crypto/x509#CreateCertificate), trường `Policies` bị bỏ qua, và
    các chính sách được lấy từ trường `PolicyIdentifiers`. Sử dụng cài đặt GODEBUG `x509usepolicies=1` đảo ngược điều này,
    điền chính sách chứng chỉ từ trường `Policies`, và bỏ qua trường `PolicyIdentifiers`. Chúng tôi có thể thay đổi
    giá trị mặc định của `x509usepolicies` trong Go 1.23, làm cho `Policies` trở thành trường mặc định để marshaling.

<!-- crypto/x509 -->

[database/sql](/pkg/database/sql/)

:   <!-- https://go.dev/issue/60370, CL 501700 -->
    Kiểu mới [`Null[T]`](/pkg/database/sql/#Null)
    cung cấp cách scan các cột nullable cho bất kỳ kiểu cột nào.

<!-- database/sql -->

[debug/elf](/pkg/debug/elf/)

:   <!-- https://go.dev/issue/61974, CL 469395 -->
    Hằng số `R_MIPS_PC32` được định nghĩa để sử dụng với các hệ thống MIPS64.

    <!-- https://go.dev/issue/63725, CL 537615 -->
    Các hằng số `R_LARCH_*` bổ sung được định nghĩa để sử dụng với các hệ thống LoongArch.

<!-- debug/elf -->

[encoding](/pkg/encoding/)

:   <!-- https://go.dev/issue/53693, https://go.dev/cl/504884 -->
    Các phương thức mới `AppendEncode` và `AppendDecode` được thêm vào
    mỗi kiểu `Encoding` trong các gói
    [`encoding/base32`](/pkg/encoding/base32),
    [`encoding/base64`](/pkg/encoding/base64), và
    [`encoding/hex`](/pkg/encoding/hex)
    đơn giản hóa việc mã hóa và giải mã từ và đến các byte slice bằng cách xử lý quản lý bộ nhớ đệm byte slice.

    <!-- https://go.dev/cl/505236 -->
    Các phương thức
    [`base32.Encoding.WithPadding`](/pkg/encoding/base32#Encoding.WithPadding) và
    [`base64.Encoding.WithPadding`](/pkg/encoding/base64#Encoding.WithPadding)
    giờ sẽ panic nếu đối số `padding` là một giá trị âm khác với
    `NoPadding`.

<!-- encoding -->

[encoding/json](/pkg/encoding/json/)

:   <!-- https://go.dev/cl/521675 -->
    Chức năng marshaling và encoding giờ escape
    các ký tự `'\b'` và `'\f'` thành
    `\b` và `\f` thay vì
    `\u0008` và `\u000c`.

<!-- encoding/json -->

[go/ast](/pkg/go/ast/)

:   <!-- https://go.dev/issue/52463, https://go/dev/cl/504915 -->
    Các khai báo sau liên quan đến
    [phân giải định danh cú pháp](https://pkg.go.dev/go/ast#Object)
    giờ đã [bị deprecated](/issue/52463):
    `Ident.Obj`,
    `Object`,
    `Scope`,
    `File.Scope`,
    `File.Unresolved`,
    `Importer`,
    `Package`,
    `NewPackage`.
    Nhìn chung, các định danh không thể được phân giải chính xác mà không có thông tin kiểu.
    Hãy xem xét, ví dụ, định danh `K`
    trong `T{K: ""}`: nó có thể là tên của biến cục bộ
    nếu T là kiểu map, hoặc tên của trường nếu T là kiểu struct.
    Các chương trình mới nên dùng gói [go/types](/pkg/go/types)
    để phân giải định danh; xem
    [`Object`](https://pkg.go.dev/go/types#Object),
    [`Info.Uses`](https://pkg.go.dev/go/types#Info.Uses), và
    [`Info.Defs`](https://pkg.go.dev/go/types#Info.Defs) để biết chi tiết.

    <!-- https://go.dev/issue/60061 -->
    Hàm mới [`ast.Unparen`](https://pkg.go.dev/go/ast#Unparen)
    loại bỏ bất kỳ
    [dấu ngoặc đơn](https://pkg.go.dev/go/ast#ParenExpr) bao quanh nào khỏi
    một [biểu thức](https://pkg.go.dev/go/ast#Expr).

<!-- go/ast -->

[go/types](/pkg/go/types/)

:   <!-- https://go.dev/issue/63223, CL 521956, CL 541737 -->
    Kiểu mới [`Alias`](/pkg/go/types#Alias) đại diện cho type alias.
    Trước đây, type alias không được biểu diễn tường minh, vì vậy một tham chiếu đến type alias tương đương với
    việc viết ra kiểu được đặt bí danh, và tên của alias bị mất.
    Biểu diễn mới giữ lại `Alias` trung gian.
    Điều này cho phép báo cáo lỗi được cải thiện (tên của type alias có thể được báo cáo), và cho phép xử lý tốt hơn
    các khai báo kiểu vòng liên quan đến type alias.
    Trong bản phát hành tương lai, các kiểu `Alias` cũng sẽ mang [thông tin tham số kiểu](/issue/46477).
    Hàm mới [`Unalias`](/pkg/go/types#Unalias) trả về kiểu thực tế được ký hiệu bởi một
    kiểu `Alias` (hoặc bất kỳ [`Type`](/pkg/go/types#Type) nào khác).

    Vì các kiểu `Alias` có thể phá vỡ các type switch hiện có không biết cần kiểm tra chúng,
    chức năng này được kiểm soát bởi trường [`GODEBUG`](/doc/godebug) có tên `gotypesalias`.
    Với `gotypesalias=0`, mọi thứ hoạt động như trước, và kiểu `Alias` không bao giờ được tạo.
    Với `gotypesalias=1`, kiểu `Alias` được tạo và các client phải chấp nhận chúng.
    Mặc định là `gotypesalias=0`.
    Trong bản phát hành tương lai, mặc định sẽ được thay đổi thành `gotypesalias=1`.
    _Các client của [`go/types`](/pkg/go/types) được khuyến khích điều chỉnh mã của họ càng sớm càng tốt
    để hoạt động với `gotypesalias=1` nhằm loại bỏ vấn đề sớm._

    <!-- https://go.dev/issue/62605, CL 540056 -->
    Struct [`Info`](/pkg/go/types#Info) giờ xuất bản
    map [`FileVersions`](/pkg/go/types#Info.FileVersions)
    cung cấp thông tin phiên bản Go theo từng tệp.

    <!-- https://go.dev/issue/62037, CL 541575 -->
    Phương thức helper mới [`PkgNameOf`](/pkg/go/types#Info.PkgNameOf) trả về tên gói cục bộ
    cho khai báo import đã cho.

    <!-- https://go.dev/issue/61035, multiple CLs, see issue for details -->
    Triển khai của [`SizesFor`](/pkg/go/types#SizesFor) đã được điều chỉnh để tính toán
    cùng kích thước kiểu như trình biên dịch khi đối số trình biên dịch cho `SizesFor` là `"gc"`.
    Triển khai [`Sizes`](/pkg/go/types#Sizes) mặc định được dùng bởi bộ kiểm tra kiểu giờ là
    `types.SizesFor("gc", "amd64")`.

    <!-- https://go.dev/issue/64295, CL 544035 -->
    Vị trí bắt đầu ([`Pos`](/pkg/go/types#Scope.Pos))
    của khối môi trường từ vựng ([`Scope`](/pkg/go/types#Scope))
    đại diện cho thân hàm đã thay đổi:
    trước đây nó bắt đầu tại dấu ngoặc nhọn mở của thân hàm,
    nhưng giờ bắt đầu tại token `func` của hàm.

[html/template](/pkg/html/template/)

:   <!-- https://go.dev/issue/61619, CL 507995 -->
    Các template literal JavaScript giờ có thể chứa các hành động template Go, và việc phân tích một template chứa chúng sẽ
    không còn trả về `ErrJSTemplate`. Tương tự, cài đặt GODEBUG `jstmpllitinterp` không
    còn có tác dụng gì.

<!-- html/template -->

[io](/pkg/io/)

:   <!-- https://go.dev/issue/61870, CL 526855 -->
    Phương thức mới [`SectionReader.Outer`](/pkg/io#SectionReader.Outer) trả về [`ReaderAt`](/pkg/io#ReaderAt), offset, và kích thước được truyền cho [`NewSectionReader`](/pkg/io#NewSectionReader).

<!-- io -->

[log/slog](/pkg/log/slog/)

:   <!-- https://go.dev/issue/62418 -->
    Hàm mới [`SetLogLoggerLevel`](/pkg/log/slog#SetLogLoggerLevel)
    kiểm soát mức cho cầu nối giữa các gói `slog` và `log`. Nó đặt mức tối thiểu
    cho các lệnh gọi đến các hàm logging cấp cao nhất của `slog`, và đặt mức cho các lệnh gọi đến `log.Logger`
    đi qua `slog`.

[math/big](/pkg/math/big/)

:   <!-- https://go.dev/issue/50489, CL 539299 -->
    Phương thức mới [`Rat.FloatPrec`](/pkg/math/big#Rat.FloatPrec) tính số chữ số thập phân phần lẻ
    cần thiết để biểu diễn một số hữu tỉ chính xác dưới dạng số dấu phẩy động, và liệu biểu diễn thập phân chính xác
    có khả thi ngay từ đầu không.

<!-- math/big -->

[net](/pkg/net/)

:   <!-- https://go.dev/issue/58808 -->
    Khi [`io.Copy`](/pkg/io#Copy) sao chép
    từ `TCPConn` sang `UnixConn`,
    nó giờ sẽ dùng lệnh gọi hệ thống `splice(2)` của Linux nếu có thể,
    sử dụng phương thức mới [`TCPConn.WriteTo`](/pkg/net#TCPConn.WriteTo).

    <!-- CL 467335 -->
    Go DNS Resolver, được dùng khi xây dựng với "-tags=netgo",
    giờ tìm kiếm tên khớp trong tệp hosts của Windows,
    nằm tại `%SystemRoot%\System32\drivers\etc\hosts`,
    trước khi thực hiện truy vấn DNS.

<!-- net -->

[net/http](/pkg/net/http/)

:   <!-- https://go.dev/issue/51971 -->
    Các hàm mới
    [`ServeFileFS`](/pkg/net/http#ServeFileFS),
    [`FileServerFS`](/pkg/net/http#FileServerFS), và
    [`NewFileTransportFS`](/pkg/net/http#NewFileTransportFS)
    là các phiên bản của
    `ServeFile`, `FileServer`, và `NewFileTransport` hiện có,
    hoạt động trên một `fs.FS`.

    <!-- https://go.dev/issue/61679 -->
    HTTP server và client giờ từ chối các yêu cầu và phản hồi chứa
    header `Content-Length` rỗng không hợp lệ.
    Hành vi cũ có thể được khôi phục bằng cách đặt
    trường [`GODEBUG`](/doc/godebug) `httplaxcontentlength=1`.

    <!-- https://go.dev/issue/61410, CL 528355 -->
    Phương thức mới
    [`Request.PathValue`](/pkg/net/http#Request.PathValue)
    trả về các giá trị ký tự đại diện đường dẫn từ một yêu cầu
    và phương thức mới
    [`Request.SetPathValue`](/pkg/net/http#Request.SetPathValue)
    đặt các giá trị ký tự đại diện đường dẫn trên một yêu cầu.

<!-- net/http -->

[net/http/cgi](/pkg/net/http/cgi/)

:   <!-- CL 539615 -->
    Khi thực thi một tiến trình CGI, biến `PATH_INFO` giờ luôn
    được đặt thành chuỗi rỗng hoặc một giá trị bắt đầu bằng ký tự `/`,
    theo yêu cầu của RFC 3875. Trước đây có thể một số tổ hợp của
    [`Handler.Root`](/pkg/net/http/cgi#Handler.Root)
    và URL yêu cầu vi phạm yêu cầu này.

<!-- net/http/cgi -->

[net/netip](/pkg/net/netip/)

:   <!-- https://go.dev/issue/61642 -->
    Phương thức mới [`AddrPort.Compare`](/pkg/net/netip#AddrPort.Compare)
    so sánh hai `AddrPort`.

<!-- net/netip -->

[os](/pkg/os/)

:   <!-- CL 516555 -->
    Trên Windows, hàm [`Stat`](/pkg/os#Stat) giờ theo dõi tất cả các reparse point
    liên kết đến một thực thể được đặt tên khác trong hệ thống.
    Trước đây nó chỉ theo dõi các reparse point `IO_REPARSE_TAG_SYMLINK` và
    `IO_REPARSE_TAG_MOUNT_POINT`.

    <!-- CL 541015 -->
    Trên Windows, truyền [`O_SYNC`](/pkg/os#O_SYNC) cho [`OpenFile`](/pkg/os#OpenFile) giờ khiến các thao tác ghi đi thẳng đến đĩa, tương đương với `O_SYNC` trên các nền tảng Unix.

    <!-- CL 452995 -->
    Trên Windows, các hàm [`ReadDir`](/pkg/os#ReadDir),
    [`File.ReadDir`](/pkg/os#File.ReadDir),
    [`File.Readdir`](/pkg/os#File.Readdir),
    và [`File.Readdirnames`](/pkg/os#File.Readdirnames)
    giờ đọc các mục thư mục theo lô để giảm số lượng lệnh gọi hệ thống,
    cải thiện hiệu suất lên đến 30%.

    <!-- https://go.dev/issue/58808 -->
    Khi [`io.Copy`](/pkg/io#Copy) sao chép
    từ `File` sang `net.UnixConn`,
    nó giờ sẽ dùng lệnh gọi hệ thống `sendfile(2)` của Linux nếu có thể,
    sử dụng phương thức mới [`File.WriteTo`](/pkg/os#File.WriteTo).

<!-- os -->

[os/exec](/pkg/os/exec/)

:   <!-- CL 528037 -->
    Trên Windows, [`LookPath`](/pkg/os/exec#LookPath) giờ
    bỏ qua các mục rỗng trong `%PATH%`, và trả về
    `ErrNotFound` (thay vì `ErrNotExist`) nếu
    không tìm thấy phần mở rộng tệp thực thi để phân giải tên không mơ hồ.

    <!-- CL 528038, CL 527820 -->
    Trên Windows, [`Command`](/pkg/os/exec#Command) và
    [`Cmd.Start`](/pkg/os/exec#Cmd.Start) không còn
    gọi `LookPath` nếu đường dẫn đến tệp thực thi đã là tuyệt đối và có phần mở rộng tệp thực thi. Ngoài ra,
    `Cmd.Start` không còn ghi lại phần mở rộng đã phân giải trở lại vào
    trường [`Path`](/pkg/os/exec#Cmd.Path),
    vì vậy giờ an toàn để gọi phương thức `String` đồng thời
    với lệnh gọi đến `Start`.

<!-- os/exec -->

[reflect](/pkg/reflect/)

:   <!-- https://go.dev/issue/61827, CL 517777 -->
    Phương thức [`Value.IsZero`](/pkg/reflect/#Value.IsZero)
    giờ sẽ trả về true cho số dấu phẩy động hoặc complex
    âm zero, và sẽ trả về true cho giá trị struct nếu một
    trường trống (trường có tên `_`) có giá trị
    khác zero theo cách nào đó.
    Những thay đổi này làm cho `IsZero` nhất quán với việc so sánh
    một giá trị với zero bằng toán tử `==` của ngôn ngữ.

    <!-- https://go.dev/issue/59599, CL 511035 -->
    Hàm [`PtrTo`](/pkg/reflect/#PtrTo) bị deprecated,
    thay bằng [`PointerTo`](/pkg/reflect/#PointerTo).

    <!-- https://go.dev/issue/60088, CL 513478 -->
    Hàm mới [`TypeFor`](/pkg/reflect/#TypeFor)
    trả về [`Type`](/pkg/reflect/#Type) đại diện cho
    đối số kiểu T.
    Trước đây, để lấy giá trị `reflect.Type` cho một kiểu, cần phải dùng
    `reflect.TypeOf((*T)(nil)).Elem()`.
    Giờ có thể viết là `reflect.TypeFor[T]()`.

<!-- reflect -->

[runtime/metrics](/pkg/runtime/metrics/)

:   <!-- https://go.dev/issue/63340 -->
    Bốn metric histogram mới
    `/sched/pauses/stopping/gc:seconds`,
    `/sched/pauses/stopping/other:seconds`,
    `/sched/pauses/total/gc:seconds`, và
    `/sched/pauses/total/other:seconds` cung cấp thêm chi tiết
    về các khoảng dừng stop-the-world.
    Các metric "stopping" báo cáo thời gian từ khi quyết định dừng thế giới
    đến khi tất cả các goroutine dừng lại.
    Các metric "total" báo cáo thời gian từ khi quyết định dừng thế giới
    đến khi nó được khởi động lại.

    <!-- https://go.dev/issue/63340 -->
    Metric `/gc/pauses:seconds` bị deprecated, vì nó tương đương với metric mới `/sched/pauses/total/gc:seconds`.

    <!-- https://go.dev/issue/57071 -->
    `/sync/mutex/wait/total:seconds` giờ bao gồm sự tranh chấp trên các khóa nội bộ runtime ngoài
    [`sync.Mutex`](/pkg/sync#Mutex) và
    [`sync.RWMutex`](/pkg/sync#RWMutex).

<!-- runtime/metrics -->

[runtime/pprof](/pkg/runtime/pprof/)

:   <!-- https://go.dev/issue/61015 -->
    Các profile mutex giờ scale sự tranh chấp theo số goroutine bị chặn trên mutex.
    Điều này cung cấp biểu diễn chính xác hơn về mức độ mà mutex là điểm tắc nghẽn trong
    chương trình Go.
    Ví dụ, nếu 100 goroutine bị chặn trên một mutex trong 10 mili giây, một mutex profile sẽ
    giờ ghi lại 1 giây độ trễ thay vì 10 mili giây độ trễ.

    <!-- https://go.dev/issue/57071 -->
    Các profile mutex giờ cũng bao gồm sự tranh chấp trên các khóa nội bộ runtime ngoài
    [`sync.Mutex`](/pkg/sync#Mutex) và
    [`sync.RWMutex`](/pkg/sync#RWMutex).
    Tranh chấp trên các khóa nội bộ runtime luôn được báo cáo tại `runtime._LostContendedRuntimeLock`.
    Bản phát hành tương lai sẽ thêm stack trace đầy đủ trong các trường hợp này.

    <!-- https://go.dev/issue/50891 -->
    Các CPU profile trên nền tảng Darwin giờ chứa bản đồ bộ nhớ của tiến trình, cho phép xem
    disassembly trong công cụ pprof.

<!-- runtime/pprof -->

[runtime/trace](/pkg/runtime/trace/)

:   <!-- https://go.dev/issue/60773 -->
    Execution tracer đã được cải tiến hoàn toàn trong bản phát hành này, giải quyết một số vấn đề tồn đọng lâu dài
    và mở đường cho các trường hợp sử dụng mới cho execution trace.

    Execution trace giờ sử dụng đồng hồ của hệ điều hành trên hầu hết các nền tảng (ngoại trừ Windows) để
    có thể tương quan chúng với các trace được tạo bởi các thành phần cấp thấp hơn.
    Execution trace không còn phụ thuộc vào độ tin cậy của đồng hồ nền tảng để tạo ra trace chính xác.
    Execution trace giờ được phân vùng đều đặn ngay lập tức và do đó có thể được xử lý theo cách streamable.
    Execution trace giờ chứa thời lượng đầy đủ cho tất cả các lệnh gọi hệ thống.
    Execution trace giờ chứa thông tin về các luồng hệ điều hành mà goroutine thực thi trên đó.
    Tác động độ trễ của việc bắt đầu và dừng execution trace đã giảm đáng kể.
    Execution trace giờ có thể bắt đầu hoặc kết thúc trong giai đoạn đánh dấu thu gom rác.

    Để cho phép các nhà phát triển Go tận dụng những cải tiến này, một
    gói đọc trace thử nghiệm có tại [golang.org/x/exp/trace](/pkg/golang.org/x/exp/trace).
    Lưu ý rằng gói này chỉ hoạt động với các trace được tạo bởi các chương trình xây dựng với Go 1.22 hiện tại.
    Hãy thử gói và cung cấp phản hồi trên
    [issue đề xuất tương ứng](/issue/62627).

    Nếu bạn gặp vấn đề với triển khai execution tracer mới, bạn có thể chuyển lại
    triển khai cũ bằng cách xây dựng chương trình Go của bạn với `GOEXPERIMENT=noexectracer2`.
    Nếu bạn làm vậy, hãy file một issue, nếu không tùy chọn này sẽ bị xóa trong bản phát hành tương lai.

<!-- runtime/trace -->

[slices](/pkg/slices/)

:   <!-- https://go.dev/issue/56353 -->
    <!-- CL 504882 -->
    Hàm mới `Concat` nối nhiều slice lại với nhau.

    <!-- https://go.dev/issue/63393 -->
    <!-- CL 543335 -->
    Các hàm thu nhỏ kích thước của slice (`Delete`, `DeleteFunc`, `Compact`, `CompactFunc`, và `Replace`) giờ zero hóa các phần tử giữa độ dài mới và độ dài cũ.

    <!-- https://go.dev/issue/63913 -->
    <!-- CL 540155 -->
    `Insert` giờ luôn panic nếu đối số `i` nằm ngoài phạm vi. Trước đây nó không panic trong trường hợp này nếu không có phần tử nào để chèn.

<!-- slices -->

[syscall](/pkg/syscall/)

:   <!-- https://go.dev/issue/60797 -->
    Gói `syscall` đã bị [đóng băng](/s/go1.4-syscall) từ Go 1.4 và bị đánh dấu là deprecated trong Go 1.11, khiến nhiều trình soạn thảo cảnh báo về bất kỳ việc sử dụng nào của gói.
    Tuy nhiên, một số chức năng không bị deprecated yêu cầu sử dụng gói `syscall`, như trường [`os/exec.Cmd.SysProcAttr`](/pkg/os/exec#Cmd).
    Để tránh những cảnh báo không cần thiết về mã như vậy, gói `syscall` không còn bị đánh dấu là deprecated.
    Gói vẫn bị đóng băng với hầu hết các chức năng mới, và mã mới vẫn được khuyến khích sử dụng [`golang.org/x/sys/unix`](/pkg/golang.org/x/sys/unix) hoặc [`golang.org/x/sys/windows`](/pkg/golang.org/x/sys/windows) khi có thể.

    <!-- https://go.dev/issue/51246, CL 520266 -->
    Trên Linux, trường mới [`SysProcAttr.PidFD`](/pkg/syscall#SysProcAttr) cho phép lấy PID FD khi khởi động tiến trình con thông qua [`StartProcess`](/pkg/syscall#StartProcess) hoặc [`os/exec`](/pkg/os/exec).

    <!-- CL 541015 -->
    Trên Windows, truyền [`O_SYNC`](/pkg/syscall#O_SYNC) cho [`Open`](/pkg/syscall#Open) giờ khiến các thao tác ghi đi thẳng đến đĩa, tương đương với `O_SYNC` trên các nền tảng Unix.

<!-- syscall -->

[testing/slogtest](/pkg/testing/slogtest/)

:   <!-- https://go.dev/issue/61758 -->
    Hàm mới [`Run`](/pkg/testing/slogtest#Run) dùng sub-test để chạy các test case,
    cung cấp kiểm soát chi tiết hơn.

<!-- testing/slogtest -->

## Các cổng {#ports}

### Darwin {#darwin}

<!-- CL 461697 -->
Trên macOS với kiến trúc x86 64-bit (cổng `darwin/amd64`),
toolchain Go giờ tạo ra các tệp thực thi position-independent (PIE) theo mặc định.
Các tệp nhị phân không phải PIE có thể được tạo bằng cách chỉ định cờ build `-buildmode=exe`.
Trên macOS 64-bit dựa trên ARM (cổng `darwin/arm64`),
toolchain Go đã tạo PIE theo mặc định.

<!-- go.dev/issue/64207 -->
Go 1.22 là bản phát hành cuối cùng sẽ chạy trên macOS 10.15 Catalina. Go 1.23 sẽ yêu cầu macOS 11 Big Sur trở lên.

### ARM {#arm}

<!-- CL 514907 -->
Biến môi trường `GOARM` giờ cho phép chọn sử dụng dấu phẩy động phần mềm hay phần cứng.
Trước đây, các giá trị `GOARM` hợp lệ là `5`, `6`, hoặc `7`. Giờ những giá trị đó có thể
được theo sau tùy chọn bởi `,softfloat` hoặc `,hardfloat` để chọn triển khai dấu phẩy động.

Tùy chọn mới này mặc định là `softfloat` cho phiên bản `5` và `hardfloat` cho phiên bản
`6` và `7`.

### Loong64 {#loong64}

<!-- CL 481315 -->
Cổng `loong64` giờ hỗ trợ truyền đối số và kết quả hàm bằng thanh ghi.

<!-- CL 481315,537615,480878 -->
Cổng `linux/loong64` giờ hỗ trợ address sanitizer, memory sanitizer, linker relocation kiểu mới, và chế độ build `plugin`.

### OpenBSD {#openbsd}

<!-- CL 517935 -->
Go 1.22 thêm cổng thử nghiệm cho OpenBSD trên PowerPC 64-bit big-endian
(`openbsd/ppc64`).
