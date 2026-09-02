---
path: /doc/go1.21
title: Ghi chú phát hành Go 1.21
---

<!--
NOTE: In this document and others in this directory, the convention is to
set fixed-width phrases with non-fixed-width spaces, as in
`hello` `world`.
Do not send CLs removing the interior tags from such phrases.
-->

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu Go 1.21 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.21, ra mắt sáu tháng sau [Go 1.20](/doc/go1.20).
Phần lớn thay đổi nằm ở phần triển khai của toolchain, runtime và thư viện.
Như mọi khi, bản phát hành duy trì [cam kết tương thích](/doc/go1compat) của Go 1;
thực tế, Go 1.21 [cải thiện cam kết đó](#tools).
Chúng tôi kỳ vọng hầu như tất cả chương trình Go vẫn tiếp tục biên dịch và chạy như trước.

<!-- https://go.dev/issue/57631 -->
Go 1.21 giới thiệu một thay đổi nhỏ về cách đánh số bản phát hành.
Trước đây, chúng tôi dùng Go 1._N_ để chỉ cả phiên bản ngôn ngữ Go tổng thể và họ phát hành
cũng như bản phát hành đầu tiên trong họ đó.
Bắt đầu từ Go 1.21, bản phát hành đầu tiên giờ là Go 1._N_.0.
Hôm nay chúng tôi phát hành cả ngôn ngữ Go 1.21 và triển khai ban đầu của nó, bản phát hành Go 1.21.0.
Các ghi chú này đề cập đến "Go 1.21"; các công cụ như `go` `version` sẽ báo cáo "`go1.21.0`"
(cho đến khi bạn nâng cấp lên Go 1.21.1).
Xem "[Phiên bản Go](/doc/toolchain#version)" trong tài liệu "Go Toolchains" để biết chi tiết
về cách đánh số phiên bản mới.

## Thay đổi ngôn ngữ {#language}

Go 1.21 thêm ba hàm built-in mới vào ngôn ngữ.

  - <!-- https://go.dev/issue/59488 -->
    Các hàm mới `min` và `max` tính
    giá trị nhỏ nhất (hoặc lớn nhất, với `max`) trong một số lượng cố định
    các đối số đã cho.
    Xem đặc tả ngôn ngữ để biết
    [chi tiết](/ref/spec#Min_and_max).
  - <!-- https://go.dev/issue/56351 -->
    Hàm mới `clear` xóa tất cả phần tử khỏi một
    map hoặc đặt tất cả phần tử của một slice về zero.
    Xem đặc tả ngôn ngữ để biết
    [chi tiết](/ref/spec#Clear).


<!-- https://go.dev/issue/57411 -->
Thứ tự khởi tạo gói giờ được định nghĩa chính xác hơn. Thuật toán
mới là:

- Sắp xếp tất cả gói theo đường dẫn import.
- Lặp lại cho đến khi danh sách gói trống:
  - Tìm gói đầu tiên trong danh sách mà tất cả import của nó đã được
    khởi tạo.
  - Khởi tạo gói đó và xóa nó khỏi danh sách.

Điều này có thể thay đổi hành vi của một số chương trình dựa trên
thứ tự khởi tạo cụ thể không được thể hiện bằng các import tường minh.
Hành vi của các chương trình như vậy không được định nghĩa rõ ràng bởi
đặc tả trong các bản phát hành trước. Quy tắc mới cung cấp định nghĩa rõ ràng.


Nhiều cải tiến tăng sức mạnh và độ chính xác của suy diễn kiểu đã được thực hiện.

  - <!-- https://go.dev/issue/59338 -->
    Một hàm generic (có thể được khởi tạo một phần) giờ có thể được gọi với các đối số
    bản thân là các hàm generic (có thể được khởi tạo một phần).
    Compiler sẽ cố gắng suy diễn các đối số kiểu còn thiếu của callee (như trước) và,
    cho mỗi đối số là hàm generic không được khởi tạo đầy đủ,
    các đối số kiểu còn thiếu của nó (mới).
    Các trường hợp sử dụng điển hình là các lời gọi đến các hàm generic hoạt động trên container
    (như [slices.IndexFunc](/pkg/slices#IndexFunc)) nơi một đối số hàm
    cũng có thể là generic, và nơi đối số kiểu của hàm được gọi và các đối số của nó
    được suy diễn từ kiểu container.
    Tổng quát hơn, một hàm generic giờ có thể được dùng mà không cần khởi tạo tường minh khi
    nó được gán cho một biến hoặc được trả về như một giá trị kết quả nếu các đối số kiểu có thể
    được suy diễn từ phép gán.
  - <!-- https://go.dev/issue/60353, https://go.dev/issue/57192, https://go.dev/issue/52397, https://go.dev/issue/41176 -->
    Suy diễn kiểu giờ cũng xem xét các phương thức khi một giá trị được gán cho interface:
    các đối số kiểu cho các tham số kiểu được dùng trong chữ ký phương thức có thể được suy diễn từ
    các kiểu tham số tương ứng của các phương thức khớp.
  - <!-- https://go.dev/issue/51593 https://go.dev/issue/39661 -->
    Tương tự, vì một đối số kiểu phải triển khai tất cả phương thức của ràng buộc tương ứng của nó,
    các phương thức của đối số kiểu và ràng buộc được khớp, có thể dẫn đến việc suy diễn
    các đối số kiểu bổ sung.
  - <!-- https://go.dev/issue/58671 -->
    Nếu nhiều đối số hằng số không được gõ thuộc các loại khác nhau (chẳng hạn một int không được gõ và
    một hằng số dấu phẩy động không được gõ) được truyền cho các tham số có cùng kiểu
    tham số kiểu (không được chỉ định khác), thay vì lỗi, suy diễn kiểu giờ xác định
    kiểu bằng cách tiếp cận giống như một toán tử với các toán hạng hằng số không được gõ.
    Thay đổi này điều chỉnh các kiểu được suy diễn từ các đối số hằng số không được gõ cho phù hợp với
    các kiểu của biểu thức hằng số.
  - <!-- https://go.dev/issue/59750 -->
    Suy diễn kiểu giờ chính xác khi khớp các kiểu tương ứng trong phép gán:
    các kiểu thành phần (chẳng hạn các phần tử của slice, hoặc các kiểu tham số trong chữ ký hàm)
    phải giống hệt nhau (cho các đối số kiểu phù hợp) để khớp, nếu không suy diễn thất bại.
    Thay đổi này tạo ra thông báo lỗi chính xác hơn:
    khi trước đây suy diễn kiểu có thể thành công không đúng và dẫn đến phép gán không hợp lệ,
    compiler giờ báo cáo lỗi suy diễn nếu hai kiểu không thể khớp được.

<!-- https://go.dev/issue/58650 -->
Tổng quát hơn, mô tả về
[suy diễn kiểu](/ref/spec#Type_inference)
trong đặc tả ngôn ngữ đã được làm rõ.
Tổng hợp lại, tất cả những thay đổi này làm cho suy diễn kiểu mạnh mẽ hơn và các lỗi suy diễn ít bất ngờ hơn.

<!-- https://go.dev/issue/57969 -->

Go 1.21 bao gồm bản xem trước về một thay đổi ngôn ngữ mà chúng tôi đang xem xét cho phiên bản Go tương lai:
biến vòng lặp for thành theo từng lần lặp thay vì theo toàn bộ vòng lặp, để tránh các lỗi chia sẻ vô tình.
Để biết chi tiết về cách thử thay đổi ngôn ngữ đó, xem [trang wiki LoopvarExperiment](/wiki/LoopvarExperiment).

<!-- https://go.dev/issue/25448 -->

Go 1.21 giờ định nghĩa rằng nếu một goroutine đang panic và recover được gọi trực tiếp bởi một hàm deferred,
giá trị trả về của recover được đảm bảo không phải nil. Để đảm bảo điều này, gọi panic
với giá trị interface nil (hoặc nil không được gõ) gây ra runtime panic kiểu
[`*runtime.PanicNilError`](/pkg/runtime/#PanicNilError).

Để hỗ trợ các chương trình được viết cho các phiên bản Go cũ hơn, nil panic có thể được bật lại bằng cách đặt
`GODEBUG=panicnil=1`.
Cài đặt này được bật tự động khi biên dịch chương trình mà main package của nó
nằm trong module khai báo `go` `1.20` hoặc cũ hơn.

## Công cụ {#tools}

Go 1.21 thêm hỗ trợ cải thiện cho tương thích ngược và tương thích tiến
trong Go toolchain.

<!-- https://go.dev/issue/56986 -->
Để cải thiện tương thích ngược, Go 1.21 chính thức hóa
việc Go sử dụng biến môi trường GODEBUG để kiểm soát
hành vi mặc định cho các thay đổi không vi phạm theo
[chính sách tương thích](/doc/go1compat)
nhưng dù sao có thể khiến các chương trình hiện tại bị hỏng.
(Ví dụ: các chương trình phụ thuộc vào hành vi có lỗi có thể bị hỏng
khi lỗi được sửa, nhưng các sửa lỗi không được coi là thay đổi vi phạm.)
Khi Go phải thực hiện loại thay đổi hành vi này,
nó giờ chọn giữa hành vi cũ và mới dựa trên
dòng `go` trong tệp `go.work` của workspace
hoặc tệp `go.mod` của main module.
Nâng cấp lên Go toolchain mới nhưng để dòng `go`
ở phiên bản Go cũ (cũ hơn) sẽ giữ nguyên hành vi của toolchain cũ.
Với hỗ trợ tương thích này, Go toolchain mới nhất luôn nên là
triển khai tốt nhất, an toàn nhất của phiên bản Go cũ.
Xem "[Go, Tương thích ngược, và GODEBUG](/doc/godebug)" để biết chi tiết.

<!-- https://go.dev/issue/57001 -->
Để cải thiện tương thích tiến, Go 1.21 giờ đọc dòng `go`
trong tệp `go.work` hoặc `go.mod` như một yêu cầu tối thiểu nghiêm ngặt: `go` `1.21.0` có nghĩa là
workspace hoặc module không thể dùng với Go 1.20 hoặc Go 1.21rc1.
Điều này cho phép các dự án phụ thuộc vào các bản sửa trong các phiên bản Go mới hơn
để đảm bảo họ không được dùng với các phiên bản cũ hơn.
Nó cũng cho phép báo cáo lỗi tốt hơn cho các dự án sử dụng tính năng Go mới:
khi vấn đề là cần phiên bản Go mới hơn,
vấn đề đó được báo cáo rõ ràng, thay vì cố build code
và in lỗi về import chưa giải quyết hoặc lỗi cú pháp.

Để làm cho các yêu cầu phiên bản nghiêm ngặt hơn mới này dễ quản lý hơn,
lệnh `go` giờ có thể gọi không chỉ toolchain
đi kèm trong bản phát hành của nó mà còn các phiên bản Go toolchain khác tìm thấy trong PATH
hoặc được tải xuống theo yêu cầu.
Nếu dòng `go` trong `go.mod` hoặc `go.work`
khai báo yêu cầu tối thiểu về phiên bản Go mới hơn, lệnh `go`
sẽ tìm và chạy phiên bản đó tự động.
Directive `toolchain` mới đặt toolchain tối thiểu được đề xuất để sử dụng,
có thể mới hơn tối thiểu `go` nghiêm ngặt.
Xem "[Go Toolchains](/doc/toolchain)" để biết chi tiết.

### Lệnh go {#go-command}

<!-- https://go.dev/issue/58099, CL 474236 -->
Cờ build `-pgo` giờ mặc định là `-pgo=auto`,
và hạn chế chỉ định một main package duy nhất trên dòng
lệnh đã được bỏ. Nếu tệp có tên `default.pgo` có mặt
trong thư mục của main package, lệnh `go` sẽ dùng
nó để bật tối ưu hóa dựa trên hồ sơ thực thi khi build chương trình tương ứng.

Cờ `-C` `dir` giờ phải là cờ đầu tiên
trên dòng lệnh khi được dùng.

<!-- https://go.dev/issue/37708, CL 463837 -->
Tùy chọn `go` `test` mới
`-fullpath` in tên đường dẫn đầy đủ trong các thông báo log test,
thay vì chỉ tên cơ sở.

<!-- https://go.dev/issue/15513, CL 466397 -->
Cờ `go` `test` `-c` giờ
hỗ trợ ghi test binary cho nhiều gói, mỗi gói vào
`pkg.test` nơi `pkg` là tên gói.
Sẽ là lỗi nếu nhiều hơn một gói test được biên dịch có tên gói đã cho.

<!-- https://go.dev/issue/15513, CL 466397 -->
Cờ `go` `test` `-o` giờ
chấp nhận đối số thư mục, trong đó các test binary được ghi vào thư mục đó
thay vì thư mục hiện tại.

<!-- https://go.dev/issue/31544, CL 477839 -->
Khi dùng external (C) linker với cgo được bật, gói `runtime/cgo` giờ được
cung cấp cho Go linker như một dependency bổ sung để đảm bảo rằng Go
runtime tương thích với bất kỳ thư viện bổ sung nào được thêm bởi C linker.

### Cgo {#cgo}

<!-- CL 490819 -->
Trong các tệp `import "C"`, Go toolchain giờ
báo cáo đúng lỗi khi cố khai báo phương thức Go trên kiểu C.

## Runtime {#runtime-changes}

<!-- https://go.dev/issue/7181 -->
Khi in các stack rất sâu, runtime giờ in 50 frame đầu tiên
(trong cùng) theo sau bởi 50 frame dưới cùng (ngoài cùng),
thay vì chỉ in 100 frame đầu tiên. Điều này giúp dễ dàng hơn để
xem cách các stack đệ quy sâu bắt đầu, và đặc biệt
có giá trị để debug stack overflow.

<!-- https://go.dev/issue/59960 -->
Trên các nền tảng Linux hỗ trợ transparent huge pages, Go runtime
giờ quản lý rõ ràng hơn những phần nào của heap có thể được hỗ trợ bởi huge pages.
Điều này dẫn đến sử dụng bộ nhớ tốt hơn: các heap nhỏ
nên thấy ít bộ nhớ được sử dụng hơn (lên đến 50% trong các trường hợp cực đoan) trong khi
các heap lớn nên thấy ít huge pages bị phá vỡ hơn cho các phần dày đặc của
heap, cải thiện việc sử dụng CPU và độ trễ lên đến 1%. Hệ quả của điều này
là runtime không còn cố gắng khắc phục một cài đặt cấu hình Linux cụ thể
có vấn đề, điều này có thể dẫn đến chi phí bộ nhớ cao hơn. Cách sửa được khuyến nghị là
điều chỉnh cài đặt huge page của OS theo [hướng dẫn GC](/doc/gc-guide#Linux_transparent_huge_pages).
Tuy nhiên, cũng có các giải pháp thay thế khác. Xem [phần về
`max_ptes_none`](/doc/gc-guide#Linux_THP_max_ptes_none_workaround).

<!-- https://go.dev/issue/57069, https://go.dev/issue/56966 -->
Do điều chỉnh bộ gom rác bên trong runtime,
các ứng dụng có thể thấy giảm đến 40% trong độ trễ đuôi ứng dụng
và giảm nhỏ trong việc sử dụng bộ nhớ. Một số ứng dụng cũng có thể thấy
giảm nhỏ trong throughput.
Giảm sử dụng bộ nhớ nên tỷ lệ với sự giảm throughput,
sao cho đánh đổi throughput/bộ nhớ của bản phát hành trước có thể được khôi phục (với ít thay đổi về độ trễ) bằng cách
tăng `GOGC` và/hoặc `GOMEMLIMIT` một chút.

<!-- https://go.dev/issue/51676 -->
Các lời gọi từ C sang Go trên các thread được tạo trong C yêu cầu một số thiết lập để chuẩn bị cho
thực thi Go. Trên các nền tảng Unix, thiết lập này giờ được giữ nguyên qua nhiều
lời gọi từ cùng một thread. Điều này giảm đáng kể chi phí của
các lời gọi C sang Go tiếp theo từ khoảng 1-3 microsecond mỗi lời gọi xuống còn khoảng 100-200
nanosecond mỗi lời gọi.

## Compiler {#compiler}

Tối ưu hóa dựa trên hồ sơ thực thi (PGO), được thêm như bản xem trước trong Go 1.20, giờ sẵn sàng
để sử dụng chung. PGO bật các tối ưu hóa bổ sung trên code được xác định là
nóng bởi hồ sơ của các workload sản xuất. Như đã đề cập trong
[phần lệnh Go](#go-command), PGO được bật theo mặc định cho
các binary chứa hồ sơ `default.pgo` trong thư mục main
package. Cải thiện hiệu năng thay đổi tùy theo hành vi ứng dụng, với hầu hết các chương trình từ một tập đại diện các chương trình Go thấy
từ 2 đến 7% cải thiện khi bật PGO. Xem
[hướng dẫn sử dụng PGO](/doc/pgo) để biết tài liệu chi tiết.

<!-- https://go.dev/issue/59959 -->

Các build PGO giờ có thể devirtualize một số lời gọi phương thức interface, thêm
lời gọi cụ thể đến callee phổ biến nhất. Điều này bật tối ưu hóa thêm,
chẳng hạn như inline callee.

<!-- CL 497455 -->

Go 1.21 cải thiện tốc độ build lên đến 6%, chủ yếu nhờ vào việc build
compiler với PGO.

## Assembler {#assembler}

<!-- https://go.dev/issue/58378 -->

Trên amd64, các hàm assembly frameless nosplit không còn tự động được đánh dấu là `NOFRAME`.
Thay vào đó, thuộc tính `NOFRAME` phải được chỉ định tường minh nếu muốn,
vốn đã là hành vi trên các kiến trúc khác hỗ trợ frame pointer.
Với điều này, runtime giờ duy trì các frame pointer cho quá trình chuyển đổi stack.

<!-- CL 476295 -->

Trình xác minh kiểm tra việc sử dụng sai `R15` khi dynamic linking trên amd64 đã được cải thiện.

## Linker {#linker}

<!-- https://go.dev/issue/57302, CL 461749, CL 457455 -->
Trên windows/amd64, linker (với sự trợ giúp của compiler) giờ phát ra
dữ liệu SEH unwinding theo mặc định, cải thiện việc tích hợp
của các ứng dụng Go với Windows debugger và các công cụ khác.

<!-- CL 463395, CL 461315 -->

Trong Go 1.21, linker (với sự trợ giúp của compiler) giờ có khả năng
xóa các biến map toàn cục chết (không được tham chiếu), nếu số
mục trong bộ khởi tạo biến đủ lớn, và nếu các biểu thức khởi tạo không có tác dụng phụ.

## Thư viện chuẩn {#library}

### Gói log/slog mới {#slog}

<!--
 https://go.dev/issue/59060, https://go.dev/issue/59141, https://go.dev/issue/59204, https://go.dev/issue/59280,
        https://go.dev/issue/59282, https://go.dev/issue/59339, https://go.dev/issue/59345, https://go.dev/issue/61200,
        CL 477295, CL 484096, CL 486376, CL 486415, CL 487855, CL 508195
 -->
Gói [log/slog](/pkg/log/slog) mới cung cấp structured logging với các cấp độ.
Structured logging phát ra các cặp khóa-giá trị
để cho phép xử lý nhanh, chính xác các lượng lớn dữ liệu log.
Gói hỗ trợ tích hợp với các công cụ và dịch vụ phân tích log phổ biến.

### Gói testing/slogtest mới {#slogtest}

<!-- CL 487895 -->
Gói [testing/slogtest](/pkg/testing/slogtest) mới có thể giúp
xác nhận các triển khai [slog.Handler](/pkg/log/slog#Handler).

### Gói slices mới {#slices}

<!-- https://go.dev/issue/45955, https://go.dev/issue/54768 -->
<!-- https://go.dev/issue/57348, https://go.dev/issue/57433 -->
<!-- https://go.dev/issue/58565, https://go.dev/issue/60091 -->
<!-- https://go.dev/issue/60546 -->
<!-- CL 467417, CL 468855, CL 483175, CL 496078, CL 498175, CL 502955 -->
Gói [slices](/pkg/slices) mới cung cấp nhiều thao tác thông thường
trên các slice, sử dụng các hàm generic hoạt động với slice
của bất kỳ kiểu phần tử nào.

### Gói maps mới {#maps}

<!-- https://go.dev/issue/57436, CL 464343 -->
Gói [maps](/pkg/maps/) mới cung cấp một số
thao tác thông thường trên các map, sử dụng các hàm generic hoạt động với
các map của bất kỳ kiểu khóa hoặc phần tử nào.

### Gói cmp mới {#cmp}

<!-- https://go.dev/issue/59488, CL 496356 -->
Gói [cmp](/pkg/cmp/) mới định nghĩa ràng buộc kiểu
[`Ordered`](/pkg/cmp/#Ordered) và
hai hàm generic mới
[`Less`](/pkg/cmp/#Less)
và [`Compare`](/pkg/cmp/#Compare) hữu ích với các
[kiểu có thứ tự](/ref/spec/#Comparison_operators).

### Các thay đổi nhỏ trong thư viện {#minor_library_changes}

Như mọi khi, có nhiều thay đổi và cập nhật nhỏ trong thư viện,
được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1
được ghi nhớ.
Ngoài ra cũng có nhiều cải thiện hiệu năng, không được liệt kê ở đây.

#### [archive/tar](/pkg/archive/tar/)

<!-- https://go.dev/issue/54451, CL 491175 -->
Triển khai interface
[`io/fs.FileInfo`](/pkg/io/fs/#FileInfo)
được trả bởi
[`Header.FileInfo`](/pkg/archive/tar/#Header.FileInfo)
giờ triển khai phương thức `String` gọi
[`io/fs.FormatFileInfo`](/pkg/io/fs/#FormatFileInfo).

<!-- archive/tar -->

#### [archive/zip](/pkg/archive/zip/)

<!-- https://go.dev/issue/54451, CL 491175 -->
Triển khai interface
[`io/fs.FileInfo`](/pkg/io/fs/#FileInfo)
được trả bởi
[`FileHeader.FileInfo`](/pkg/archive/zip/#FileHeader.FileInfo)
giờ triển khai phương thức `String` gọi
[`io/fs.FormatFileInfo`](/pkg/io/fs/#FormatFileInfo).

<!-- https://go.dev/issue/54451, CL 491175 -->
Triển khai interface
[`io/fs.DirEntry`](/pkg/io/fs/#DirEntry)
được trả bởi phương thức
[`io/fs.ReadDirFile.ReadDir`](/pkg/io/fs/#ReadDirFile.ReadDir)
của
[`io/fs.File`](/pkg/io/fs/#File)
được trả bởi
[`Reader.Open`](/pkg/archive/zip/#Reader.Open)
giờ triển khai phương thức `String` gọi
[`io/fs.FormatDirEntry`](/pkg/io/fs/#FormatDirEntry).

<!-- archive/zip -->

#### [bytes](/pkg/bytes/)

<!-- https://go.dev/issue/53685, CL 474635 -->
Kiểu [`Buffer`](/pkg/bytes/#Buffer)
có hai phương thức mới:
[`Available`](/pkg/bytes/#Buffer.Available)
và [`AvailableBuffer`](/pkg/bytes/#Buffer.AvailableBuffer).
Chúng có thể dùng cùng với phương thức
[`Write`](/pkg/bytes/#Buffer.Write)
để append trực tiếp vào `Buffer`.

<!-- bytes -->

#### [context](/pkg/context/)

<!-- https://go.dev/issue/40221, CL 479918 -->
Hàm mới [`WithoutCancel`](/pkg/context/#WithoutCancel)
trả về bản sao của một context không bị hủy khi context gốc
bị hủy.

<!-- https://go.dev/issue/56661, CL 449318 -->
Các hàm mới [`WithDeadlineCause`](/pkg/context/#WithDeadlineCause)
và [`WithTimeoutCause`](/pkg/context/#WithTimeoutCause)
cung cấp cách đặt nguyên nhân hủy context khi deadline hoặc
bộ đếm thời gian hết hạn. Nguyên nhân có thể lấy lại bằng hàm
[`Cause`](/pkg/context/#Cause).

<!-- https://go.dev/issue/57928, CL 482695 -->
Hàm mới [`AfterFunc`](/pkg/context/#AfterFunc)
đăng ký một hàm chạy sau khi context bị hủy.

<!-- CL 455455 -->
Một tối ưu hóa có nghĩa là kết quả của việc gọi
[`Background`](/pkg/context/#Background)
và [`TODO`](/pkg/context/#TODO) và
chuyển đổi chúng sang một kiểu dùng chung có thể được coi là bằng nhau.
Trong các bản phát hành trước chúng luôn khác nhau. So sánh
các giá trị [`Context`](/pkg/context/#Context)
cho tính bằng nhau chưa bao giờ được định nghĩa rõ ràng, vì vậy đây không
được coi là thay đổi không tương thích.

#### [crypto/ecdsa](/pkg/crypto/ecdsa/)

<!-- CL 492955 -->
[`PublicKey.Equal`](/pkg/crypto/ecdsa/#PublicKey.Equal) và
[`PrivateKey.Equal`](/pkg/crypto/ecdsa/#PrivateKey.Equal)
giờ thực thi trong thời gian không đổi.

<!-- crypto/ecdsa -->

#### [crypto/elliptic](/pkg/crypto/elliptic/)

<!-- CL 459977 -->
Tất cả phương thức [`Curve`](/pkg/crypto/elliptic/#Curve) đã bị deprecated, cùng với [`GenerateKey`](/pkg/crypto/elliptic/#GenerateKey), [`Marshal`](/pkg/crypto/elliptic/#Marshal), và [`Unmarshal`](/pkg/crypto/elliptic/#Unmarshal). Cho các thao tác ECDH, nên dùng gói [`crypto/ecdh`](/pkg/crypto/ecdh/) mới thay thế. Cho các thao tác cấp thấp hơn, hãy dùng các module bên thứ ba như [filippo.io/nistec](https://pkg.go.dev/filippo.io/nistec).

<!-- crypto/elliptic -->

#### [crypto/rand](/pkg/crypto/rand/)

<!-- CL 463123 -->
Gói [`crypto/rand`](/pkg/crypto/rand/) giờ dùng system call `getrandom` trên NetBSD 10.0 trở lên.

<!-- crypto/rand -->

#### [crypto/rsa](/pkg/crypto/rsa/)

<!-- CL 471259, CL 492935 -->
Hiệu năng của các phép tính RSA riêng (giải mã và ký) giờ tốt hơn Go 1.19 cho `GOARCH=amd64` và `GOARCH=arm64`. Nó đã bị thoái lui trong Go 1.20.

Do việc thêm các trường riêng vào [`PrecomputedValues`](/pkg/crypto/rsa/#PrecomputedValues), [`PrivateKey.Precompute`](/pkg/crypto/rsa/#PrivateKey.Precompute) phải được gọi để có hiệu năng tối ưu ngay cả khi deserializing (ví dụ từ JSON) một khóa riêng đã được tính trước.

<!-- CL 492955 -->
[`PublicKey.Equal`](/pkg/crypto/rsa/#PublicKey.Equal) và
[`PrivateKey.Equal`](/pkg/crypto/rsa/#PrivateKey.Equal)
giờ thực thi trong thời gian không đổi.

<!-- https://go.dev/issue/56921, CL 459976 -->
Hàm [`GenerateMultiPrimeKey`](/pkg/crypto/rsa/#GenerateMultiPrimeKey) và trường [`PrecomputedValues.CRTValues`](/pkg/crypto/rsa/#PrecomputedValues.CRTValues) đã bị deprecated. [`PrecomputedValues.CRTValues`](/pkg/crypto/rsa/#PrecomputedValues.CRTValues) vẫn sẽ được điền khi [`PrivateKey.Precompute`](/pkg/crypto/rsa/#PrivateKey.Precompute) được gọi, nhưng các giá trị sẽ không được sử dụng trong quá trình giải mã.

<!-- crypto/rsa -->

<!-- CL 483815 reverted -->

#### [crypto/sha256](/pkg/crypto/sha256/)

<!-- https://go.dev/issue/50543, CL 408795 -->
Các phép tính SHA-224 và SHA-256 giờ dùng các hướng dẫn native khi có sẵn với `GOARCH=amd64`, cải thiện hiệu năng khoảng 3-4 lần.

<!-- crypto/sha256 -->

<!-- CL 481478 reverted -->
<!-- CL 483816 reverted -->

#### [crypto/tls](/pkg/crypto/tls/)

<!-- CL 497895 -->
Các máy chủ giờ bỏ qua việc xác minh chứng chỉ client (bao gồm không chạy
[`Config.VerifyPeerCertificate`](/pkg/crypto/tls/#Config.VerifyPeerCertificate))
cho các kết nối được tiếp tục, ngoài việc kiểm tra thời gian hết hạn. Điều này làm cho
session ticket lớn hơn khi chứng chỉ client được sử dụng. Các client đã
bỏ qua việc xác minh khi tiếp tục, nhưng giờ kiểm tra thời gian hết hạn
ngay cả khi [`Config.InsecureSkipVerify`](/pkg/crypto/tls/#Config.InsecureSkipVerify)
được đặt.

<!-- https://go.dev/issue/60105, CL 496818, CL 496820, CL 496822, CL 496821, CL 501675 -->
Các ứng dụng giờ có thể kiểm soát nội dung của session ticket.

  - Kiểu mới [`SessionState`](/pkg/crypto/tls/#SessionState)
    mô tả một phiên có thể tiếp tục.
  - Phương thức [`SessionState.Bytes`](/pkg/crypto/tls/#SessionState.Bytes)
    và hàm [`ParseSessionState`](/pkg/crypto/tls/#ParseSessionState)
    serialize và deserialize một `SessionState`.
  - Các hook [`Config.WrapSession`](/pkg/crypto/tls/#Config.WrapSession) và
    [`Config.UnwrapSession`](/pkg/crypto/tls/#Config.UnwrapSession)
    chuyển đổi `SessionState` thành và từ ticket ở phía máy chủ.
  - Các phương thức [`Config.EncryptTicket`](/pkg/crypto/tls/#Config.EncryptTicket)
    và [`Config.DecryptTicket`](/pkg/crypto/tls/#Config.DecryptTicket)
    cung cấp triển khai mặc định của `WrapSession` và
    `UnwrapSession`.
  - Phương thức [`ClientSessionState.ResumptionState`](/pkg/crypto/tls/#ClientSessionState.ResumptionState) và
    hàm [`NewResumptionState`](/pkg/crypto/tls/#NewResumptionState)
    có thể được dùng bởi triển khai `ClientSessionCache` để lưu trữ và
    tiếp tục các phiên ở phía client.


<!-- CL 496817 -->
Để giảm khả năng session ticket được dùng như cơ chế theo dõi
giữa các kết nối, máy chủ giờ phát hành ticket mới ở mỗi
lần tiếp tục (nếu chúng được hỗ trợ và không bị tắt) và ticket không còn mang
định danh cho khóa mã hóa chúng nữa. Nếu truyền số lượng lớn
khóa cho [`Conn.SetSessionTicketKeys`](/pkg/crypto/tls/#Conn.SetSessionTicketKeys),
điều này có thể dẫn đến chi phí hiệu năng đáng chú ý.

<!-- CL 497376 -->
Cả client và máy chủ giờ triển khai extension Extended Master Secret (RFC 7627).
Việc deprecated của [`ConnectionState.TLSUnique`](/pkg/crypto/tls/#ConnectionState.TLSUnique)
đã được hoàn nguyên, và giờ được đặt cho các kết nối được tiếp tục hỗ trợ Extended Master Secret.

<!-- https://go.dev/issue/44886, https://go.dev/issue/60107, CL 493655, CL 496995, CL 514997 -->
Kiểu mới [`QUICConn`](/pkg/crypto/tls/#QUICConn)
cung cấp hỗ trợ cho các triển khai QUIC, bao gồm hỗ trợ 0-RTT. Lưu ý
rằng đây không tự nó là triển khai QUIC, và 0-RTT vẫn chưa
được hỗ trợ trong TLS.

<!-- https://go.dev/issue/46308, CL 497377 -->
Hàm mới [`VersionName`](/pkg/crypto/tls/#VersionName)
trả về tên cho một số phiên bản TLS.

<!-- https://go.dev/issue/52113, CL 410496 -->
Các mã cảnh báo TLS được gửi từ máy chủ cho các lỗi xác thực client đã
được cải thiện. Trước đây, các lỗi này luôn dẫn đến cảnh báo "bad certificate".
Giờ, một số lỗi sẽ dẫn đến các mã cảnh báo phù hợp hơn,
được định nghĩa bởi RFC 5246 và RFC 8446:

  - Cho các kết nối TLS 1.3, nếu máy chủ được cấu hình yêu cầu xác thực client bằng
    [RequireAnyClientCert](/pkg/crypto/tls/#RequireAnyClientCert) hoặc
    [RequireAndVerifyClientCert](/pkg/crypto/tls/#RequireAndVerifyClientCert),
    và client không cung cấp bất kỳ chứng chỉ nào, máy chủ giờ sẽ trả về cảnh báo "certificate required".
  - Nếu client cung cấp chứng chỉ không được ký bởi tập hợp CA tin cậy
    được cấu hình trên máy chủ, máy chủ sẽ trả về cảnh báo "unknown certificate authority".
  - Nếu client cung cấp chứng chỉ đã hết hạn hoặc chưa có hiệu lực,
    máy chủ sẽ trả về cảnh báo "expired certificate".
  - Trong tất cả các tình huống khác liên quan đến lỗi xác thực client, máy chủ vẫn trả về "bad certificate".


<!-- crypto/tls -->

#### [crypto/x509](/pkg/crypto/x509/)

<!-- https://go.dev/issue/53573, CL 468875 -->
[`RevocationList.RevokedCertificates`](/pkg/crypto/x509/#RevocationList.RevokedCertificates) đã bị deprecated và được thay thế bằng trường [`RevokedCertificateEntries`](/pkg/crypto/x509/#RevocationList.RevokedCertificateEntries) mới, là một slice của [`RevocationListEntry`](/pkg/crypto/x509/#RevocationListEntry). [`RevocationListEntry`](/pkg/crypto/x509/#RevocationListEntry) chứa tất cả các trường trong [`pkix.RevokedCertificate`](/pkg/crypto/x509/pkix#RevokedCertificate), cũng như mã lý do thu hồi.

<!-- CL 478216 -->
Các ràng buộc tên giờ được thực thi đúng cách trên các chứng chỉ không phải leaf, và
không áp dụng cho các chứng chỉ nơi chúng được biểu đạt.

<!-- crypto/x509 -->

#### [debug/elf](/pkg/debug/elf/)

<!-- https://go.dev/issue/56892, CL 452617 -->
Phương thức mới
[`File.DynValue`](/pkg/debug/elf/#File.DynValue)
có thể dùng để lấy các giá trị số được liệt kê với một
dynamic tag đã cho.

<!-- https://go.dev/issue/56887, CL 452496 -->
Các cờ hằng số được phép trong dynamic tag `DT_FLAGS_1`
giờ được định nghĩa với kiểu
[`DynFlag1`](/pkg/debug/elf/#DynFlag1). Các tag này
có tên bắt đầu bằng `DF_1`.

<!-- CL 473256 -->
Gói giờ định nghĩa hằng số
[`COMPRESS_ZSTD`](/pkg/debug/elf/#COMPRESS_ZSTD).

<!-- https://go.dev/issue/60348, CL 496918 -->
Gói giờ định nghĩa hằng số
[`R_PPC64_REL24_P9NOTOC`](/pkg/debug/elf/#R_PPC64_REL24_P9NOTOC).

<!-- debug/elf -->

#### [debug/pe](/pkg/debug/pe/)

<!-- CL 488475 -->
Các lần thử đọc từ một phần chứa dữ liệu chưa khởi tạo
bằng cách dùng
[`Section.Data`](/pkg/debug/pe/#Section.Data)
hoặc reader được trả bởi [`Section.Open`](/pkg/debug/pe/#Section.Open)
giờ trả về lỗi.

<!-- debug/pe -->

#### [embed](/pkg/embed/)

<!-- https://go.dev/issue/57803, CL 483235 -->
[`io/fs.File`](/pkg/io/fs/#File)
được trả bởi
[`FS.Open`](/pkg/embed/#FS.Open) giờ
có phương thức `ReadAt` triển khai
[`io.ReaderAt`](/pkg/io/#ReaderAt).

<!-- https://go.dev/issue/54451, CL 491175 -->
Gọi <code>[FS.Open](/pkg/embed/FS.Open).[Stat](/pkg/io/fs/#File.Stat)</code>
sẽ trả về một kiểu giờ triển khai phương thức `String`
gọi
[`io/fs.FormatFileInfo`](/pkg/io/fs/#FormatFileInfo).

<!-- embed -->

#### [encoding/binary](/pkg/encoding/binary/)

<!-- https://go.dev/issue/57237, CL 463218, CL 463985 -->
Biến mới
[`NativeEndian`](/pkg/encoding/binary/#NativeEndian)
có thể dùng để chuyển đổi giữa byte slice và số nguyên
sử dụng endianness native của máy hiện tại.

<!-- encoding/binary -->

#### [errors](/pkg/errors/)

<!-- https://go.dev/issue/41198, CL 473935 -->
Lỗi mới
[`ErrUnsupported`](/pkg/errors/#ErrUnsupported)
cung cấp cách chuẩn hóa để cho biết rằng một
thao tác được yêu cầu có thể không được thực hiện vì nó không được hỗ trợ.
Ví dụ: lời gọi đến
[`os.Link`](/pkg/os/#Link) khi sử dụng hệ thống tệp không hỗ trợ hard link.

<!-- errors -->

#### [flag](/pkg/flag/)

<!-- https://go.dev/issue/53747, CL 476015 -->
Hàm mới [`BoolFunc`](/pkg/flag/#BoolFunc)
và phương thức
[`FlagSet.BoolFunc`](/pkg/flag/#FlagSet.BoolFunc)
định nghĩa một flag không yêu cầu đối số và gọi
một hàm khi flag được sử dụng. Điều này tương tự như
[`Func`](/pkg/flag/#Func) nhưng cho boolean flag.

<!-- CL 480215 -->
Định nghĩa flag
(qua [`Bool`](/pkg/flag/#Bool),
[`BoolVar`](/pkg/flag/#BoolVar),
[`Int`](/pkg/flag/#Int),
[`IntVar`](/pkg/flag/#IntVar), v.v.)
sẽ panic nếu [`Set`](/pkg/flag/#Set) đã
được gọi trên một flag có cùng tên. Thay đổi này được thiết kế
để phát hiện các trường hợp [thay đổi trong
thứ tự khởi tạo](#language) gây ra các thao tác flag xảy ra theo
thứ tự khác với mong đợi. Trong nhiều trường hợp, cách sửa vấn đề
này là giới thiệu một dependency gói tường minh để
sắp xếp đúng việc định nghĩa trước bất kỳ
thao tác [`Set`](/pkg/flag/#Set) nào.

<!-- flag -->

#### [go/ast](/pkg/go/ast/)

<!-- https://go.dev/issue/28089, CL 487935 -->
Predicate mới [`IsGenerated`](/pkg/go/ast/#IsGenerated)
báo cáo liệu một cây cú pháp tệp có chứa
[comment đặc biệt](/s/generatedcode)
theo quy ước cho biết tệp được tạo bởi một công cụ hay không.

<!-- https://go.dev/issue/59033, CL 476276 -->
Trường mới
[`File.GoVersion`](/pkg/go/ast/#File.GoVersion)
ghi lại phiên bản Go tối thiểu yêu cầu bởi
bất kỳ directive `//go:build` hoặc `// +build` nào.

<!-- go/ast -->

#### [go/build](/pkg/go/build/)

<!-- https://go.dev/issue/56986, CL 453603 -->
Gói giờ phân tích các build directive (comment bắt đầu
bằng `//go:`) trong header tệp (trước
khai báo `package`). Các directive này
có sẵn trong các trường mới
[`Package`](/pkg/go/build#Package)
[`Directives`](/pkg/go/build#Package.Directives),
[`TestDirectives`](/pkg/go/build#Package.TestDirectives),
và
[`XTestDirectives`](/pkg/go/build#Package.XTestDirectives).

<!-- go/build -->

#### [go/build/constraint](/pkg/go/build/constraint/)

<!-- https://go.dev/issue/59033, CL 476275 -->
Hàm mới
[`GoVersion`](/pkg/go/build/constraint/#GoVersion)
trả về phiên bản Go tối thiểu được ngụ ý bởi một biểu thức build.

<!-- go/build/constraint -->

#### [go/token](/pkg/go/token/)

<!-- https://go.dev/issue/57708, CL 464515 -->
Phương thức mới [`File.Lines`](/pkg/go/token/#File.Lines)
trả về bảng số dòng của tệp theo cùng dạng được chấp nhận bởi
`File.SetLines`.

<!-- go/token -->

#### [go/types](/pkg/go/types/)

<!-- https://go.dev/issue/61175, CL 507975 -->
Phương thức mới [`Package.GoVersion`](/pkg/go/types/#Package.GoVersion)
trả về phiên bản ngôn ngữ Go được dùng để kiểm tra gói.

<!-- go/types -->

#### [hash/maphash](/pkg/hash/maphash/)

<!-- https://go.dev/issue/47342, CL 468795 -->
Gói `hash/maphash` giờ có triển khai Go thuần túy, có thể chọn bằng build tag `purego`.

<!-- hash/maphash -->

#### [html/template](/pkg/html/template/)

<!-- https://go.dev/issue/59584, CL 496395 -->
Lỗi mới
[`ErrJSTemplate`](/pkg/html/template/#ErrJSTemplate)
được trả khi một action xuất hiện trong JavaScript template
literal. Trước đây một lỗi không được export sẽ được trả về.

<!-- html/template -->

#### [io/fs](/pkg/io/fs/)

<!-- https://go.dev/issue/54451, CL 489555 -->
Hàm mới
[`FormatFileInfo`](/pkg/io/fs/#FormatFileInfo)
trả về phiên bản được định dạng của
[`FileInfo`](/pkg/io/fs/#FileInfo).
Hàm mới
[`FormatDirEntry`](/pkg/io/fs/#FormatDirEntry)
trả về phiên bản được định dạng của
[`DirEntry`](/pkg/io/fs/#FileInfo).
Triển khai của
[`DirEntry`](/pkg/io/fs/#DirEntry)
được trả bởi
[`ReadDir`](/pkg/io/fs/#ReadDir) giờ
triển khai phương thức `String` gọi
[`FormatDirEntry`](/pkg/io/fs/#FormatDirEntry),
và tương tự với
giá trị [`DirEntry`](/pkg/io/fs/#DirEntry)
được truyền cho
[`WalkDirFunc`](/pkg/io/fs/#WalkDirFunc).

<!-- io/fs -->

<!-- https://go.dev/issue/56491 rolled back by https://go.dev/issue/60519 -->
<!-- CL 459435 reverted by CL 467255 -->
<!-- CL 467515 reverted by CL 499416 -->

#### [math/big](/pkg/math/big/)

<!-- https://go.dev/issue/56984, CL 453115, CL 500116 -->
Phương thức mới [`Int.Float64`](/pkg/math/big/#Int.Float64)
trả về giá trị dấu phẩy động gần nhất với
số nguyên đa độ chính xác, cùng với thông tin về bất kỳ
làm tròn nào đã xảy ra.

<!-- math/big -->

#### [net](/pkg/net/)

<!-- https://go.dev/issue/59166, https://go.dev/issue/56539 -->
<!-- CL 471136, CL 471137, CL 471140 -->
Trên Linux, gói [net](/pkg/net/) giờ có thể dùng
Multipath TCP khi kernel hỗ trợ nó. Nó không được dùng
theo mặc định. Để dùng Multipath TCP khi có sẵn trên client, hãy gọi
phương thức
[`Dialer.SetMultipathTCP`](/pkg/net/#Dialer.SetMultipathTCP)
trước khi gọi phương thức
[`Dialer.Dial`](/pkg/net/#Dialer.Dial) hoặc
[`Dialer.DialContext`](/pkg/net/#Dialer.DialContext).
Để dùng Multipath TCP khi có sẵn trên máy chủ, hãy gọi
phương thức
[`ListenConfig.SetMultipathTCP`](/pkg/net/#ListenConfig.SetMultipathTCP)
trước khi gọi phương thức
[`ListenConfig.Listen`](/pkg/net/#ListenConfig.Listen).
Chỉ định network là `"tcp"` hoặc
`"tcp4"` hoặc `"tcp6"` như thường lệ. Nếu
Multipath TCP không được kernel hoặc remote host hỗ trợ,
kết nối sẽ lặng lẽ fall back về TCP. Để kiểm tra liệu một
kết nối cụ thể có đang sử dụng Multipath TCP hay không, hãy dùng phương thức
[`TCPConn.MultipathTCP`](/pkg/net/#TCPConn.MultipathTCP).

Trong bản phát hành Go tương lai, chúng tôi có thể bật Multipath TCP theo mặc định trên
các hệ thống hỗ trợ nó.

<!-- net -->

#### [net/http](/pkg/net/http/)

<!-- CL 472636 -->
Phương thức mới [`ResponseController.EnableFullDuplex`](/pkg/net/http#ResponseController.EnableFullDuplex)
cho phép các handler máy chủ đọc đồng thời từ body yêu cầu HTTP/1
trong khi viết phản hồi. Thông thường, HTTP/1 server
tự động tiêu thụ bất kỳ body yêu cầu còn lại nào trước khi bắt đầu
viết phản hồi, để tránh deadlock client cố viết yêu cầu đầy đủ trước khi đọc phản hồi.
Phương thức `EnableFullDuplex` tắt hành vi này.

<!-- https://go.dev/issue/44855, CL 382117 -->
Lỗi mới [`ErrSchemeMismatch`](/pkg/net/http/#ErrSchemeMismatch) được trả bởi [`Client`](/pkg/net/http/#Client) và [`Transport`](/pkg/net/http/#Transport) khi máy chủ phản hồi yêu cầu HTTPS bằng phản hồi HTTP.

<!-- CL 494122 -->
Gói [net/http](/pkg/net/http/) giờ hỗ trợ
[`errors.ErrUnsupported`](/pkg/errors/#ErrUnsupported),
sao cho biểu thức
`errors.Is(http.ErrNotSupported, errors.ErrUnsupported)`
sẽ trả về true.

<!-- net/http -->

#### [os](/pkg/os/)

<!-- https://go.dev/issue/32558, CL 219638 -->
Các chương trình giờ có thể truyền giá trị `time.Time` rỗng cho
hàm [`Chtimes`](/pkg/os/#Chtimes)
để giữ nguyên thời gian truy cập hoặc thời gian sửa đổi.

<!-- CL 480135 -->
Trên Windows,
phương thức [`File.Chdir`](/pkg/os#File.Chdir)
giờ thay đổi thư mục hiện tại thành tệp, thay vì
luôn trả về lỗi.

<!-- CL 495079 -->
Trên các hệ thống Unix, nếu descriptor không chặn được truyền
cho [`NewFile`](/pkg/os/#NewFile), gọi
phương thức [`File.Fd`](/pkg/os/#File.Fd)
giờ sẽ trả về descriptor không chặn. Trước đây
descriptor được chuyển đổi sang chế độ blocking.

<!-- CL 477215 -->
Trên Windows, gọi
[`Truncate`](/pkg/os/#Truncate) trên
tệp không tồn tại trước đây tạo tệp rỗng. Giờ nó trả về
lỗi cho biết tệp không tồn tại.

<!-- https://go.dev/issue/56899, CL 463219 -->
Trên Windows, gọi
[`TempDir`](/pkg/os/#TempDir) giờ dùng
GetTempPath2W khi có sẵn, thay vì GetTempPathW.
Hành vi mới là biện pháp tăng cường bảo mật ngăn chặn
các tệp tạm thời được tạo bởi các quá trình chạy với tư cách SYSTEM
bị truy cập bởi các quá trình không phải SYSTEM.

<!-- CL 493036 -->
Trên Windows, gói os giờ hỗ trợ làm việc với các tệp có
tên, được lưu trữ dưới dạng UTF-16, không thể được biểu diễn dưới dạng UTF-8 hợp lệ.

<!-- CL 463177 -->
Trên Windows, [`Lstat`](/pkg/os/#Lstat) giờ giải quyết
symbolic link cho các đường dẫn kết thúc bằng dấu phân cách đường dẫn, nhất quán với hành vi của nó
trên các nền tảng POSIX.

<!-- https://go.dev/issue/54451, CL 491175 -->
Triển khai interface
[`io/fs.DirEntry`](/pkg/io/fs/#DirEntry)
được trả bởi hàm [`ReadDir`](/pkg/os/#ReadDir) và
phương thức [`File.ReadDir`](/pkg/os/#File.ReadDir)
giờ triển khai phương thức `String` gọi
[`io/fs.FormatDirEntry`](/pkg/io/fs/#FormatDirEntry).

<!-- https://go.dev/issue/53761, CL 416775, CL 498015-->
Triển khai interface
[`io/fs.FS`](/pkg/io/fs/#FS) được trả bởi
hàm [`DirFS`](/pkg/os/#DirFS) giờ triển khai
các interface [`io/fs.ReadFileFS`](/pkg/io/fs/#ReadFileFS) và
[`io/fs.ReadDirFS`](/pkg/io/fs/#ReadDirFS).

<!-- os -->

#### [path/filepath](/pkg/path/filepath/)

Triển khai interface
[`io/fs.DirEntry`](/pkg/io/fs/#DirEntry)
được truyền cho đối số hàm của
[`WalkDir`](/pkg/path/filepath/#WalkDir)
giờ triển khai phương thức `String` gọi
[`io/fs.FormatDirEntry`](/pkg/io/fs/#FormatDirEntry).

<!-- path/filepath -->

<!-- CL 459455 reverted -->

#### [reflect](/pkg/reflect/)

<!-- CL 408826, CL 413474 -->
Trong Go 1.21, [`ValueOf`](/pkg/reflect/#ValueOf)
không còn ép buộc đối số của nó được cấp phát trên heap, cho phép
nội dung của `Value` được cấp phát trên stack. Hầu hết
các thao tác trên `Value` cũng cho phép giá trị bên dưới
được cấp phát trên stack.

<!-- https://go.dev/issue/55002 -->
Phương thức [`Value`](/pkg/reflect/#Value) mới
[`Value.Clear`](/pkg/reflect/#Value.Clear)
xóa nội dung của map hoặc đặt nội dung của slice về zero.
Điều này tương ứng với hàm built-in `clear` mới
[được thêm vào ngôn ngữ](#language).

<!-- https://go.dev/issue/56906, CL 452762 -->
Các kiểu [`SliceHeader`](/pkg/reflect/#SliceHeader)
và [`StringHeader`](/pkg/reflect/#StringHeader)
giờ bị deprecated. Trong code mới, ưu tiên dùng
[`unsafe.Slice`](/pkg/unsafe/#Slice),
[`unsafe.SliceData`](/pkg/unsafe/#SliceData),
[`unsafe.String`](/pkg/unsafe/#String),
hoặc [`unsafe.StringData`](/pkg/unsafe/#StringData).

<!-- reflect -->

#### [regexp](/pkg/regexp/)

<!-- https://go.dev/issue/46159, CL 479401 -->
[`Regexp`](/pkg/regexp#Regexp) giờ định nghĩa
các phương thức [`MarshalText`](/pkg/regexp#Regexp.MarshalText)
và [`UnmarshalText`](/pkg/regexp#Regexp.UnmarshalText).
Chúng triển khai
[`encoding.TextMarshaler`](/pkg/encoding#TextMarshaler)
và
[`encoding.TextUnmarshaler`](/pkg/encoding#TextUnmarshaler)
và sẽ được dùng bởi các gói như
[encoding/json](/pkg/encoding/json).

<!-- regexp -->

#### [runtime](/pkg/runtime/)

<!-- https://go.dev/issue/38651, CL 435337 -->
Các stack trace dạng văn bản được tạo bởi chương trình Go, chẳng hạn như
những gì được tạo khi crash, gọi `runtime.Stack`, hoặc
thu thập goroutine profile với `debug=2`, giờ
bao gồm ID của các goroutine đã tạo mỗi goroutine trong
stack trace.

<!-- https://go.dev/issue/57441, CL 474915 -->
Các ứng dụng Go bị crash giờ có thể opt-in vào Windows Error Reporting (WER) bằng cách đặt biến môi trường
`GOTRACEBACK=wer` hoặc gọi [`debug.SetTraceback("wer")`](/pkg/runtime/debug/#SetTraceback)
trước khi crash. Ngoài việc bật WER, runtime sẽ hoạt động như với `GOTRACEBACK=crash`.
Trên các hệ thống không phải Windows, `GOTRACEBACK=wer` bị bỏ qua.

<!-- CL 447778 -->
`GODEBUG=cgocheck=2`, một trình kiểm tra kỹ lưỡng các quy tắc truyền con trỏ cgo,
không còn có sẵn như một [tùy chọn debug](/pkg/runtime#hdr-Environment_Variables) nữa.
Thay vào đó, nó có sẵn như một thử nghiệm sử dụng `GOEXPERIMENT=cgocheck2`.
Đặc biệt điều này có nghĩa là chế độ này phải được chọn tại thời điểm build thay vì thời gian khởi động.

`GODEBUG=cgocheck=1` vẫn có sẵn (và vẫn là mặc định).

<!-- https://go.dev/issue/46787, CL 367296 -->
Kiểu mới `Pinner` đã được thêm vào gói
runtime. `Pinner` có thể dùng để "pin" bộ nhớ Go
để nó có thể được sử dụng tự do hơn bởi code không phải Go. Ví dụ:
truyền các giá trị Go tham chiếu đến bộ nhớ Go được pin cho code C giờ
được cho phép. Trước đây, truyền bất kỳ tham chiếu lồng nhau nào như vậy
bị cấm bởi
[quy tắc truyền con trỏ cgo.](https://pkg.go.dev/cmd/cgo#hdr-Passing_pointers)
Xem [tài liệu](/pkg/runtime#Pinner) để biết thêm chi tiết.

<!-- CL 472195 no release note needed -->

<!-- runtime -->

#### [runtime/metrics](/pkg/runtime/metrics/)

<!-- https://go.dev/issue/56857, CL 497315 -->
Một số metric GC nội bộ trước đây, như kích thước live heap, giờ
có sẵn.
`GOGC` và `GOMEMLIMIT` cũng giờ
có sẵn như metric.

<!-- runtime/metrics -->

#### [runtime/trace](/pkg/runtime/trace/)

<!-- https://go.dev/issue/16638 -->
Thu thập trace trên amd64 và arm64 giờ phát sinh chi phí CPU
nhỏ hơn đáng kể: cải thiện lên đến 10 lần so với bản phát hành trước.

<!-- CL 494495 -->
Các trace giờ chứa các sự kiện stop-the-world tường minh cho mỗi lý do
Go runtime có thể stop-the-world, không chỉ bộ gom rác.

<!-- runtime/trace -->

#### [sync](/pkg/sync/)

<!-- https://go.dev/issue/56102, CL 451356 -->
Các hàm mới [`OnceFunc`](/pkg/sync/#OnceFunc),
[`OnceValue`](/pkg/sync/#OnceValue), và
[`OnceValues`](/pkg/sync/#OnceValues)
nắm bắt một cách sử dụng phổ biến của [Once](/pkg/sync/#Once) để
khởi tạo lười biếng một giá trị khi sử dụng lần đầu.

#### [syscall](/pkg/syscall/)

<!-- CL 480135 -->
Trên Windows,
hàm [`Fchdir`](/pkg/syscall#Fchdir)
giờ thay đổi thư mục hiện tại thành đối số của nó, thay vì
luôn trả về lỗi.

<!-- https://go.dev/issue/46259, CL 458335 -->
Trên FreeBSD,
[`SysProcAttr`](/pkg/syscall#SysProcAttr)
có trường mới `Jail` có thể dùng để đặt
tiến trình mới tạo vào môi trường jail.

<!-- CL 493036 -->
Trên Windows, gói syscall giờ hỗ trợ làm việc với các tệp có
tên, được lưu trữ dưới dạng UTF-16, không thể được biểu diễn dưới dạng UTF-8 hợp lệ.
Các hàm [`UTF16ToString`](/pkg/syscall#UTF16ToString)
và [`UTF16FromString`](/pkg/syscall#UTF16FromString)
giờ chuyển đổi giữa dữ liệu UTF-16 và
chuỗi [WTF-8](https://simonsapin.github.io/wtf-8/).
Điều này tương thích ngược vì WTF-8 là siêu tập của định dạng UTF-8
được dùng trong các bản phát hành trước.

<!-- CL 476578, CL 476875, CL 476916 -->
Một số giá trị lỗi khớp với
[`errors.ErrUnsupported`](/pkg/errors/#ErrUnsupported) mới,
sao cho `errors.Is(err, errors.ErrUnsupported)`
trả về true.

  - `ENOSYS`
  - `ENOTSUP`
  - `EOPNOTSUPP`
  - `EPLAN9` (chỉ Plan 9)
  - `ERROR_CALL_NOT_IMPLEMENTED` (chỉ Windows)
  - `ERROR_NOT_SUPPORTED` (chỉ Windows)
  - `EWINDOWS` (chỉ Windows)


<!-- syscall -->

#### [testing](/pkg/testing/)

<!-- https://go.dev/issue/37708, CL 463837 -->
Tùy chọn mới `-test.fullpath` sẽ in tên đường dẫn đầy đủ
trong các thông báo log test, thay vì chỉ tên cơ sở.

<!-- https://go.dev/issue/52600, CL 475496 -->
Hàm mới [`Testing`](/pkg/testing/#Testing) báo cáo liệu chương trình có phải là test được tạo bởi `go` `test` hay không.

<!-- testing -->

#### [testing/fstest](/pkg/testing/fstest/)

<!-- https://go.dev/issue/54451, CL 491175 -->
Gọi <code>[Open](/pkg/testing/fstest/MapFS.Open).[Stat](/pkg/io/fs/#File.Stat)</code>
sẽ trả về một kiểu giờ triển khai phương thức `String`
gọi
[`io/fs.FormatFileInfo`](/pkg/io/fs/#FormatFileInfo).

<!-- testing/fstest -->

#### [unicode](/pkg/unicode/)

<!-- CL 456837 -->
Gói [`unicode`](/pkg/unicode/) và
hỗ trợ liên quan trong toàn bộ hệ thống đã được nâng cấp lên
[Unicode 15.0.0](https://www.unicode.org/versions/Unicode15.0.0/).

<!-- unicode -->

## Các nền tảng {#ports}

### Darwin {#darwin}

<!-- https://go.dev/issue/57125 -->
Như đã [thông báo](go1.20#darwin) trong ghi chú phát hành Go 1.20,
Go 1.21 yêu cầu macOS 10.15 Catalina trở lên;
hỗ trợ các phiên bản trước đã bị ngừng.

### Windows {#windows}

<!-- https://go.dev/issue/57003, https://go.dev/issue/57004 -->
Như đã [thông báo](go1.20#windows) trong ghi chú phát hành Go 1.20,
Go 1.21 yêu cầu ít nhất Windows 10 hoặc Windows Server 2016;
hỗ trợ các phiên bản trước đã bị ngừng.

### ARM

<!-- CL 470695 -->

Khi build bản phân phối Go với `GOARCH=arm` khi không chạy
trên hệ thống ARM (tức là khi build cross-compiler sang ARM), giá trị
mặc định cho biến môi trường `GOARM` giờ luôn được đặt
thành `7`.
Trước đây giá trị mặc định phụ thuộc vào đặc điểm của hệ thống build.

Khi không build cross-compiler, giá trị mặc định được xác định
bằng cách kiểm tra hệ thống build.
Điều đó đúng trước đây và vẫn đúng trong Go 1.21.
Điều đã thay đổi là hành vi khi build cross-compiler.

### WebAssembly {#wasm}

<!-- https://go.dev/issue/38248, https://go.dev/issue/59149, CL 489255 -->
Directive `go:wasmimport` mới giờ có thể được dùng trong chương trình Go
để import các hàm từ WebAssembly host.

<!-- https://go.dev/issue/56100 -->

Go scheduler giờ tương tác hiệu quả hơn nhiều với
JavaScript event loop, đặc biệt trong các ứng dụng chặn
thường xuyên trên các sự kiện bất đồng bộ.

### WebAssembly System Interface {#wasip1}

<!-- https://go.dev/issue/58141 -->
Go 1.21 thêm một cổng thử nghiệm cho [
WebAssembly System Interface (WASI)](https://wasi.dev/), Preview 1
(`GOOS=wasip1`, `GOARCH=wasm`).

Kết quả của việc thêm giá trị `GOOS` mới
"`wasip1`", các tệp Go có tên `*_wasip1.go`
giờ sẽ bị [bỏ qua
bởi các công cụ Go](/pkg/go/build/#hdr-Build_Constraints) ngoại trừ khi giá trị `GOOS` đó đang được
sử dụng.
Nếu bạn có tên tệp hiện có khớp với mẫu đó, bạn sẽ
cần đổi tên chúng.

### ppc64/ppc64le {#PPC64}

<!-- go.dev/issue/44549 -->
Trên Linux, `GOPPC64=power10` giờ tạo các hướng dẫn PC-relative, hướng dẫn có tiền tố,
và các hướng dẫn Power10 mới khác. Trên AIX, `GOPPC64=power10`
tạo các hướng dẫn Power10, nhưng không tạo các hướng dẫn PC-relative.

Khi build các binary position-independent cho `GOPPC64=power10`
`GOOS=linux` `GOARCH=ppc64le`, người dùng có thể thấy kích thước binary giảm
trong hầu hết các trường hợp, trong một số trường hợp là 3.5%. Các binary position-independent được build cho
ppc64le với các giá trị `-buildmode` sau:
`c-archive`, `c-shared`, `shared`, `pie`, `plugin`.

### loong64 {#loong64}

<!-- go.dev/issue/53301, CL 455075, CL 425474, CL 425476, CL 425478, CL 489576 -->
Cổng `linux/loong64` giờ hỗ trợ `-buildmode=c-archive`,
`-buildmode=c-shared` và `-buildmode=pie`.

<!--
 proposals for x repos that don't need to be mentioned here but
     are picked up by the relnote tool.
 -->
<!-- https://go.dev/issue/54232 -->
<!-- https://go.dev/issue/57051 -->
<!-- https://go.dev/issue/57792 -->
<!-- https://go.dev/issue/57906 -->
<!-- https://go.dev/issue/58668 -->
<!-- https://go.dev/issue/59016 -->
<!-- https://go.dev/issue/59676 -->
<!-- https://go.dev/issue/60409 -->
<!-- https://go.dev/issue/61176 -->
<!-- changes to cmd/api that don't need release notes. -->
<!-- CL 469115, CL 469135, CL 499981 -->
<!-- proposals that don't need release notes. -->
<!-- https://go.dev/issue/10275 -->
<!-- https://go.dev/issue/59719 -->
