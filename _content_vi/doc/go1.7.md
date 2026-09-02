---
title: Ghi chú phát hành Go 1.7
---

<!--
for acme:
Edit .,s;^PKG:([a-z][A-Za-z0-9_/]+);<a href="/pkg/\1/"><code>\1</code></a>;g
Edit .,s;^([a-z][A-Za-z0-9_/]+)\.([A-Z][A-Za-z0-9_]+\.)?([A-Z][A-Za-z0-9_]+)([ .',)]|$);<a href="/pkg/\1/#\2\3"><code>\3</code></a>\4;g
Edit .,s;^FULL:([a-z][A-Za-z0-9_/]+)\.([A-Z][A-Za-z0-9_]+\.)?([A-Z][A-Za-z0-9_]+)([ .',)]|$);<a href="/pkg/\1/#\2\3"><code>\1.\2\3</code></a>\4;g
Edit .,s;^DPKG:([a-z][A-Za-z0-9_/]+);<dl id="\1"><a href="/pkg/\1/">\1</a></dl>;g
rsc last updated through 6729576
-->

<!--
NOTE: In this document and others in this directory, the convention is to
set fixed-width phrases with non-fixed-width spaces, as in
`hello` `world`.
Do not send CLs removing the interior tags from such phrases.
-->

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu về Go 1.7 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.7, ra đời sáu tháng sau phiên bản 1.6.
Phần lớn các thay đổi nằm ở phần triển khai bộ công cụ, runtime và các thư viện.
Có một thay đổi nhỏ đối với đặc tả ngôn ngữ.
Như thường lệ, bản phát hành duy trì [cam kết tương thích](/doc/go1compat.html) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

Bản phát hành [bổ sung một port cho IBM LinuxOne](#ports);
[cập nhật backend trình biên dịch x86-64](#compiler) để tạo mã hiệu quả hơn;
bao gồm [package context](#context), được nâng cấp từ
[x/net subrepository](https://golang.org/x/net/context)
và hiện được sử dụng trong thư viện chuẩn;
và [bổ sung hỗ trợ trong package testing](#testing) cho
việc tạo phân cấp test và benchmark.
Bản phát hành cũng [hoàn thiện hỗ trợ vendoring](#cmd_go)
được bắt đầu từ Go 1.5, biến nó thành một tính năng chuẩn.

## Thay đổi về ngôn ngữ {#language}

Có một thay đổi nhỏ về ngôn ngữ trong bản phát hành này.
Phần về [các câu lệnh kết thúc](/ref/spec#Terminating_statements)
làm rõ rằng để xác định xem một danh sách câu lệnh có kết thúc bằng câu lệnh kết thúc không,
"câu lệnh không rỗng cuối cùng" được coi là phần kết,
khớp với hành vi hiện tại của bộ công cụ biên dịch gc và gccgo.
Trong các bản phát hành trước, định nghĩa chỉ đề cập đến "câu lệnh cuối cùng,"
để lại tác động của các câu lệnh rỗng ở cuối ít nhất là không rõ ràng.
Package [`go/types`](/pkg/go/types/)
đã được cập nhật để khớp với bộ công cụ biên dịch gc và gccgo
trong khía cạnh này.
Thay đổi này không có tác động đến tính đúng đắn của các chương trình hiện có.

## Các port {#ports}

Go 1.7 bổ sung hỗ trợ cho macOS 10.12 Sierra.
Các tệp nhị phân được build với các phiên bản Go trước 1.7 sẽ không hoạt động
đúng trên Sierra.

Go 1.7 bổ sung một port thử nghiệm cho [Linux trên z Systems](https://en.wikipedia.org/wiki/Linux_on_z_Systems) (`linux/s390x`)
và bắt đầu của một port cho Plan 9 trên ARM (`plan9/arm`).

Các port thử nghiệm cho Linux trên MIPS 64-bit (`linux/mips64` và `linux/mips64le`)
được thêm vào trong Go 1.6 hiện có hỗ trợ đầy đủ cho cgo và liên kết ngoài.

Port thử nghiệm cho Linux trên PowerPC 64-bit little-endian (`linux/ppc64le`)
giờ yêu cầu kiến trúc POWER8 trở lên.
PowerPC 64-bit big-endian (`linux/ppc64`) chỉ yêu cầu
kiến trúc POWER5.

Port OpenBSD hiện yêu cầu OpenBSD 5.6 trở lên, để truy cập lời gọi hệ thống [_getentropy_(2)](https://man.openbsd.org/getentropy.2).

### Các vấn đề đã biết {#known_issues}

Có một số bất ổn trên FreeBSD đã biết nhưng chưa được hiểu rõ.
Điều này có thể dẫn đến crash chương trình trong các trường hợp hiếm gặp.
Xem [issue 16136](/issue/16136),
[issue 15658](/issue/15658),
và [issue 16396](/issue/16396).
Mọi sự giúp đỡ trong việc giải quyết các vấn đề cụ thể của FreeBSD này đều được hoan nghênh.

## Công cụ {#tools}

### Assembler {#cmd_asm}

Đối với các hệ thống ARM 64-bit, tên thanh ghi vector đã được
sửa thành `V0` đến `V31`;
các bản phát hành trước đã tham chiếu sai thành `V32` đến `V63`.

Đối với các hệ thống x86 64-bit, các lệnh sau đã được thêm vào:
`PCMPESTRI`,
`RORXL`,
`RORXQ`,
`VINSERTI128`,
`VPADDD`,
`VPADDQ`,
`VPALIGNR`,
`VPBLENDD`,
`VPERM2F128`,
`VPERM2I128`,
`VPOR`,
`VPSHUFB`,
`VPSHUFD`,
`VPSLLD`,
`VPSLLDQ`,
`VPSLLQ`,
`VPSRLD`,
`VPSRLDQ`,
và
`VPSRLQ`.

### Bộ công cụ biên dịch {#compiler}

Bản phát hành này bao gồm một backend tạo mã mới cho các hệ thống x86 64-bit,
theo [đề xuất từ năm 2015](/s/go17ssa)
đã được phát triển từ đó.
Backend mới, dựa trên
[SSA](https://en.wikipedia.org/wiki/Static_single_assignment_form),
tạo ra mã nhỏ gọn, hiệu quả hơn
và cung cấp nền tảng tốt hơn cho các tối ưu hóa
như loại bỏ kiểm tra giới hạn.
Backend mới giảm thời gian CPU cần thiết bởi
các chương trình benchmark của chúng tôi từ 5-35%.

Đối với bản phát hành này, backend mới có thể bị tắt bằng cách truyền
`-ssa=0` cho trình biên dịch.
Nếu bạn thấy rằng chương trình của bạn chỉ biên dịch hoặc chạy thành công
với backend mới bị tắt, hãy
[gửi báo cáo lỗi](/issue/new).

Định dạng của siêu dữ liệu được xuất khẩu được ghi bởi trình biên dịch trong các archive package đã thay đổi:
định dạng văn bản cũ đã được thay thế bằng định dạng nhị phân nhỏ gọn hơn.
Điều này dẫn đến các archive package nhỏ hơn và sửa một vài
lỗi trường hợp góc cạnh lâu dài.

Đối với bản phát hành này, định dạng xuất khẩu mới có thể bị tắt bằng cách truyền
`-newexport=0` cho trình biên dịch.
Nếu bạn thấy rằng chương trình của bạn chỉ biên dịch hoặc chạy thành công
với định dạng xuất khẩu mới bị tắt, hãy
[gửi báo cáo lỗi](/issue/new).

Tùy chọn `-X` của trình liên kết không còn hỗ trợ dạng hai đối số bất thường
`-X` `name` `value`,
như đã [thông báo](/doc/go1.6#compiler) trong bản phát hành Go 1.6
và trong các cảnh báo được in bởi trình liên kết.
Sử dụng `-X` `name=value` thay thế.

Trình biên dịch và trình liên kết đã được tối ưu hóa và chạy nhanh hơn đáng kể trong bản phát hành này so với Go 1.6,
mặc dù chúng vẫn chậm hơn mức mong muốn và sẽ tiếp tục được tối ưu hóa trong các bản phát hành tương lai.

Do các thay đổi trên bộ công cụ biên dịch và thư viện chuẩn,
các tệp nhị phân được build với bản phát hành này thường nhỏ hơn các tệp nhị phân
được build với Go 1.6,
đôi khi tới 20-30%.

Trên các hệ thống x86-64, các chương trình Go giờ duy trì các con trỏ stack frame
như được mong đợi bởi các công cụ profiling như perf của Linux và VTune của Intel,
giúp việc phân tích và tối ưu hóa các chương trình Go với các công cụ này dễ dàng hơn.
Việc duy trì con trỏ frame có một chi phí thời gian chạy nhỏ
trung bình khoảng 2%. Chúng tôi hy vọng giảm chi phí này trong các bản phát hành tương lai.
Để build một bộ công cụ không sử dụng con trỏ frame, đặt
`GOEXPERIMENT=noframepointer` khi chạy
`make.bash`, `make.bat` hoặc `make.rc`.

### Cgo {#cmd_cgo}

Các package sử dụng [cgo](/cmd/cgo/) giờ có thể bao gồm
các tệp nguồn Fortran (ngoài C, C++, Objective C và SWIG),
mặc dù các liên kết Go vẫn phải sử dụng API ngôn ngữ C.

Các liên kết Go giờ có thể sử dụng hàm helper mới `C.CBytes`.
Trái ngược với `C.CString`, lấy một `string` Go
và trả về `*C.byte` (một `char*` C),
`C.CBytes` lấy một `[]byte` Go
và trả về `unsafe.Pointer` (một `void*` C).

Các package và tệp nhị phân được build bằng `cgo` trong các bản phát hành trước
đã tạo đầu ra khác nhau trong mỗi lần build,
do việc nhúng tên thư mục tạm thời.
Khi sử dụng bản phát hành này với
các phiên bản GCC hoặc Clang đủ mới
(những phiên bản hỗ trợ tùy chọn `-fdebug-prefix-map`),
các build đó cuối cùng nên mang tính xác định.

### Gccgo {#gccgo}

Do sự căn chỉnh lịch phát hành sáu tháng một lần của Go với lịch phát hành hàng năm của GCC,
GCC phiên bản 6 chứa phiên bản Go 1.6.1 của gccgo.
Bản phát hành tiếp theo, GCC 7, có thể sẽ có phiên bản Go 1.8 của gccgo.

### Lệnh go {#cmd_go}

Hoạt động cơ bản của lệnh [`go`](/cmd/go/)
không thay đổi, nhưng có một số thay đổi đáng chú ý.

Bản phát hành này bỏ hỗ trợ cho biến môi trường `GO15VENDOREXPERIMENT`,
như đã [thông báo](/doc/go1.6#go_command) trong bản phát hành Go 1.6.
[Hỗ trợ vendoring](/s/go15vendor)
giờ là một tính năng chuẩn của lệnh và bộ công cụ `go`.

Cấu trúc dữ liệu `Package` được cung cấp cho
"`go` `list`" giờ bao gồm một trường
`StaleReason` giải thích tại sao một package cụ thể
được coi là stale (cần được rebuild) hay không.
Trường này có sẵn cho các tùy chọn `-f` hoặc `-json`
và hữu ích để hiểu tại sao một mục tiêu đang được rebuild.

Lệnh "`go` `get`" giờ hỗ trợ
các đường dẫn import tham chiếu đến `git.openstack.org`.

Bản phát hành này thêm hỗ trợ thử nghiệm, tối thiểu để build các chương trình sử dụng
[các package chỉ có nhị phân](/pkg/go/build#hdr-Binary_Only_Packages),
các package được phân phối ở dạng nhị phân
không có mã nguồn tương ứng.
Tính năng này cần thiết trong một số môi trường thương mại
nhưng không có ý định được tích hợp đầy đủ vào phần còn lại của bộ công cụ.
Ví dụ, các công cụ giả định quyền truy cập vào mã nguồn hoàn chỉnh
sẽ không hoạt động với các package đó, và không có kế hoạch hỗ trợ
các package đó trong lệnh "`go` `get`".

### Go doc {#cmd_doc}

Lệnh "`go` `doc`"
giờ nhóm các constructor với kiểu mà chúng tạo ra,
theo [`godoc`](/cmd/godoc/).

### Go vet {#cmd_vet}

Lệnh "`go` `vet`"
có phân tích chính xác hơn trong các kiểm tra `-copylock` và `-printf` của nó,
và một kiểm tra `-tests` mới kiểm tra tên và chữ ký của các hàm test có khả năng.
Để tránh nhầm lẫn với kiểm tra `-tests` mới, tùy chọn
`-test` cũ không được quảng cáo đã bị loại bỏ; nó tương đương với `-all` `-shadow`.

Lệnh `vet` cũng có một kiểm tra mới,
`-lostcancel`, phát hiện việc không gọi
hàm hủy được trả về bởi các hàm `WithCancel`,
`WithTimeout` và `WithDeadline` trong
package `context` mới của Go 1.7 (xem [bên dưới](#context)).
Không gọi hàm ngăn `Context` mới
được thu hồi cho đến khi parent của nó bị hủy.
(Context nền không bao giờ bị hủy.)

### Go tool dist {#cmd_dist}

Lệnh con mới "`go` `tool` `dist` `list`"
in tất cả các cặp hệ điều hành/kiến trúc được hỗ trợ.

### Go tool trace {#cmd_trace}

Lệnh "`go` `tool` `trace`",
[được giới thiệu trong Go 1.5](/doc/go1.5#trace_command),
đã được tinh chỉnh theo nhiều cách.

Đầu tiên, việc thu thập trace hiệu quả hơn đáng kể so với các bản phát hành trước.
Trong bản phát hành này, chi phí thời gian thực thi điển hình của việc thu thập trace là khoảng 25%;
trong các bản phát hành trước, nó ít nhất là 400%.
Thứ hai, các tệp trace giờ bao gồm thông tin tệp và số dòng,
làm cho chúng độc lập hơn và làm cho
tệp thực thi gốc là tùy chọn khi chạy công cụ trace.
Thứ ba, công cụ trace giờ chia nhỏ các trace lớn để tránh giới hạn
trong trình xem dựa trên trình duyệt.

Mặc dù định dạng tệp trace đã thay đổi trong bản phát hành này,
các công cụ Go 1.7 vẫn có thể đọc các trace từ các bản phát hành trước.

## Hiệu suất {#performance}

Như thường lệ, các thay đổi rất chung chung và đa dạng nên rất khó đưa ra tuyên bố chính xác
về hiệu suất.
Hầu hết các chương trình nên chạy nhanh hơn một chút,
do tăng tốc trong bộ gom rác và
các tối ưu hóa trong thư viện lõi.
Trên các hệ thống x86-64, nhiều chương trình sẽ chạy nhanh hơn đáng kể,
do các cải tiến trong mã được tạo ra bởi
backend trình biên dịch mới.
Như đã lưu ý ở trên, trong các benchmark của chúng tôi,
các thay đổi tạo mã đơn lẻ thường giảm thời gian CPU của chương trình xuống 5-35%.

<!-- git log -''-grep '-[0-9][0-9]\.[0-9][0-9]%' go1.6.. -->
Đã có những tối ưu hóa đáng kể mang lại cải thiện hơn 10% cho
các triển khai trong các package
[`crypto/sha1`](/pkg/crypto/sha1/),
[`crypto/sha256`](/pkg/crypto/sha256/),
[`encoding/binary`](/pkg/encoding/binary/),
[`fmt`](/pkg/fmt/),
[`hash/adler32`](/pkg/hash/adler32/),
[`hash/crc32`](/pkg/hash/crc32/),
[`hash/crc64`](/pkg/hash/crc64/),
[`image/color`](/pkg/image/color/),
[`math/big`](/pkg/math/big/),
[`strconv`](/pkg/strconv/),
[`strings`](/pkg/strings/),
[`unicode`](/pkg/unicode/),
và
[`unicode/utf16`](/pkg/unicode/utf16/).

Thời gian tạm dừng của bộ gom rác nên ngắn hơn đáng kể so với
Go 1.6 đối với các chương trình có số lượng lớn goroutine nhàn rỗi,
biến động kích thước stack đáng kể hoặc các biến cấp package lớn.

## Thư viện chuẩn {#library}

### Context {#context}

Go 1.7 chuyển package `golang.org/x/net/context`
vào thư viện chuẩn là [`context`](/pkg/context/).
Điều này cho phép sử dụng các context để hủy, timeout và truyền
dữ liệu phạm vi yêu cầu trong các package thư viện chuẩn khác,
bao gồm
[net](#net),
[net/http](#net_http),
và
[os/exec](#os_exec),
như được ghi chú dưới đây.

Để biết thêm thông tin về context, xem
[tài liệu package](/pkg/context/)
và bài đăng blog Go
"[Go Concurrent Patterns: Context](/blog/context)."

### HTTP Tracing {#httptrace}

Go 1.7 giới thiệu [`net/http/httptrace`](/pkg/net/http/httptrace/),
một package cung cấp các cơ chế để theo dõi các sự kiện trong các yêu cầu HTTP.

### Testing {#testing}

Package `testing` giờ hỗ trợ định nghĩa
các test với subtest và benchmark với sub-benchmark.
Hỗ trợ này giúp dễ dàng viết các benchmark hướng bảng
và tạo các test phân cấp.
Nó cũng cung cấp một cách để chia sẻ mã thiết lập và dọn dẹp chung.
Xem [tài liệu package](/pkg/testing/#hdr-Subtests_and_Sub_benchmarks) để biết chi tiết.

### Runtime {#runtime}

Tất cả các panic được khởi tạo bởi runtime giờ sử dụng các giá trị panic
triển khai cả
[`error`](/ref/spec#Errors) dựng sẵn,
và
[`runtime.Error`](/pkg/runtime/#Error),
như
[được yêu cầu bởi đặc tả ngôn ngữ](/ref/spec#Run_time_panics).

Trong quá trình panic, nếu tên của tín hiệu được biết, nó sẽ được in trong stack trace.
Ngoài ra, số của tín hiệu sẽ được sử dụng, như trước Go 1.7.

Hàm mới
[`KeepAlive`](/pkg/runtime/#KeepAlive)
cung cấp một cơ chế tường minh để khai báo
rằng một đối tượng được cấp phát phải được coi là có thể truy cập
tại một điểm cụ thể trong chương trình,
thường để trì hoãn việc thực thi của finalizer liên quan.

Hàm mới
[`CallersFrames`](/pkg/runtime/#CallersFrames)
dịch một slice PC thu được từ
[`Callers`](/pkg/runtime/#Callers)
thành một chuỗi các frame tương ứng với call stack.
API mới này nên được ưu tiên thay vì sử dụng trực tiếp
[`FuncForPC`](/pkg/runtime/#FuncForPC),
vì chuỗi frame có thể mô tả chính xác hơn
các call stack với các lời gọi hàm được inlined.

Hàm mới
[`SetCgoTraceback`](/pkg/runtime/#SetCgoTraceback)
tạo điều kiện cho tích hợp chặt chẽ hơn giữa mã Go và C thực thi
trong cùng một quá trình được gọi bằng cgo.

Trên các hệ thống 32-bit, runtime giờ có thể sử dụng bộ nhớ được cấp phát
bởi hệ điều hành ở bất kỳ đâu trong không gian địa chỉ,
loại bỏ lỗi
"bộ nhớ được cấp phát bởi OS không ở trong phạm vi có thể sử dụng"
phổ biến trong một số môi trường.

Runtime giờ có thể trả lại bộ nhớ không sử dụng cho hệ điều hành trên
tất cả kiến trúc.
Trong Go 1.6 và trước đó, runtime không thể
giải phóng bộ nhớ trên ARM64, PowerPC 64-bit hoặc MIPS.

Trên Windows, các chương trình Go trong Go 1.5 và trước đó đã buộc
độ phân giải bộ hẹn giờ Windows toàn cục xuống 1ms khi khởi động
bằng cách gọi `timeBeginPeriod(1)`.
Việc thay đổi độ phân giải bộ hẹn giờ toàn cục đã gây ra vấn đề trên một số hệ thống,
và kiểm tra cho thấy lời gọi không cần thiết để có hiệu suất bộ lập lịch tốt,
vì vậy Go 1.6 đã loại bỏ lời gọi.
Go 1.7 khôi phục lại lời gọi: trong một số khối lượng công việc,
lời gọi vẫn cần thiết để có hiệu suất bộ lập lịch tốt.

### Thay đổi nhỏ đối với thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ cho thư viện,
được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1
trong tâm trí.

#### [bufio](/pkg/bufio/)

Trong các bản phát hành Go trước, nếu
phương thức [`Peek`](/pkg/bufio/#Reader.Peek) của
[`Reader`](/pkg/bufio/#Reader)
được yêu cầu nhiều byte hơn mức phù hợp với buffer bên dưới,
nó sẽ trả về một slice rỗng và lỗi `ErrBufferFull`.
Giờ nó trả về toàn bộ buffer bên dưới, vẫn kèm theo lỗi `ErrBufferFull`.

#### [bytes](/pkg/bytes/)

Các hàm mới
[`ContainsAny`](/pkg/bytes/#ContainsAny) và
[`ContainsRune`](/pkg/bytes/#ContainsRune)
đã được thêm vào để đối xứng với
package [`strings`](/pkg/strings/).

Trong các bản phát hành Go trước, nếu
phương thức [`Read`](/pkg/bytes/#Reader.Read) của
[`Reader`](/pkg/bytes/#Reader)
được yêu cầu không byte nào với không còn dữ liệu, nó sẽ
trả về số đếm là 0 và không có lỗi.
Giờ nó trả về số đếm là 0 và lỗi
[`io.EOF`](/pkg/io/#EOF).

Kiểu [`Reader`](/pkg/bytes/#Reader) có phương thức mới
[`Reset`](/pkg/bytes/#Reader.Reset) để cho phép tái sử dụng `Reader`.

#### [compress/flate](/pkg/compress/flate/)

Có nhiều tối ưu hóa hiệu suất trên toàn package.
Tốc độ giải nén được cải thiện khoảng 10%,
trong khi nén cho `DefaultCompression` nhanh gấp đôi.

Ngoài những cải tiến chung đó,
compressor `BestSpeed` đã được thay thế hoàn toàn và sử dụng một
thuật toán tương tự như [Snappy](https://github.com/google/snappy),
dẫn đến tăng tốc độ khoảng 2,5 lần,
mặc dù đầu ra có thể lớn hơn 5-10% so với thuật toán trước đó.

Cũng có mức nén mới
`HuffmanOnly`
áp dụng Huffman nhưng không áp dụng mã hóa Lempel-Ziv.
[Bỏ qua mã hóa Lempel-Ziv](https://blog.klauspost.com/constant-time-gzipzip-compression/) có nghĩa là
`HuffmanOnly` chạy nhanh hơn khoảng 3 lần so với `BestSpeed` mới
nhưng với cái giá là tạo ra các đầu ra nén lớn hơn 20-40% so với những đầu ra
được tạo ra bởi `BestSpeed` mới.

Quan trọng là lưu ý rằng cả
`BestSpeed` và `HuffmanOnly` đều tạo ra đầu ra nén tuân thủ
[RFC 1951](https://tools.ietf.org/html/rfc1951).
Nói cách khác, bất kỳ bộ giải nén DEFLATE hợp lệ nào vẫn có thể giải nén các đầu ra này.

Cuối cùng, có một thay đổi nhỏ đối với việc triển khai
[`io.Reader`](/pkg/io/#Reader) của bộ giải nén. Trong các phiên bản trước,
bộ giải nén trì hoãn báo cáo
[`io.EOF`](/pkg/io/#EOF) cho đến khi chính xác không còn byte nào có thể được đọc.
Giờ nó báo cáo
[`io.EOF`](/pkg/io/#EOF) sớm hơn khi đọc tập hợp byte cuối cùng.

#### [crypto/tls](/pkg/crypto/tls/)

Việc triển khai TLS gửi vài gói dữ liệu đầu tiên trên mỗi kết nối
sử dụng kích thước bản ghi nhỏ, dần dần tăng lên kích thước bản ghi TLS tối đa.
Heuristic này giảm lượng dữ liệu cần nhận trước
khi gói đầu tiên có thể được giải mã, cải thiện độ trễ truyền thông trên
các mạng băng thông thấp.
Đặt trường `DynamicRecordSizingDisabled` của
[`Config`](/pkg/crypto/tls/#Config)
thành true buộc hành vi của Go 1.6 và trước đó, nơi các gói là
lớn nhất có thể ngay từ đầu kết nối.

Client TLS hiện có hỗ trợ tùy chọn, hạn chế cho việc đàm phán lại do server khởi tạo,
được kích hoạt bằng cách đặt trường `Renegotiation` của
[`Config`](/pkg/crypto/tls/#Config).
Điều này cần thiết để kết nối với nhiều server Microsoft Azure.

Các lỗi được trả về bởi package giờ nhất quán bắt đầu với tiền tố
`tls:`.
Trong các bản phát hành trước, một số lỗi sử dụng tiền tố `crypto/tls:`,
một số sử dụng tiền tố `tls:`, và một số không có tiền tố nào cả.

Khi tạo các chứng chỉ tự ký, package không còn đặt
trường "Authority Key Identifier" theo mặc định.

#### [crypto/x509](/pkg/crypto/x509/)

Hàm mới
[`SystemCertPool`](/pkg/crypto/x509/#SystemCertPool)
cung cấp quyền truy cập vào toàn bộ pool chứng chỉ hệ thống nếu có.
Cũng có một kiểu lỗi liên quan mới
[`SystemRootsError`](/pkg/crypto/x509/#SystemRootsError).

#### [debug/dwarf](/pkg/debug/dwarf/)

Phương thức mới [`SeekPC`](/pkg/debug/dwarf/#Reader.SeekPC) của kiểu
[`Reader`](/pkg/debug/dwarf/#Reader) và
phương thức mới [`Ranges`](/pkg/debug/dwarf/#Ranges) của kiểu
[`Data`](/pkg/debug/dwarf/#Data)
giúp tìm đơn vị biên dịch để truyền cho một
[`LineReader`](/pkg/debug/dwarf/#LineReader)
và để xác định hàm cụ thể cho một bộ đếm chương trình đã cho.

#### [debug/elf](/pkg/debug/elf/)

Kiểu [`R_390`](/pkg/debug/elf/#R_390) relocation mới
và nhiều hằng số được định nghĩa sẵn
hỗ trợ port S390.

#### [encoding/asn1](/pkg/encoding/asn1/)

Bộ giải mã ASN.1 giờ từ chối các mã hóa số nguyên không tối thiểu.
Điều này có thể khiến package từ chối một số dữ liệu ASN.1 không hợp lệ nhưng trước đây được chấp nhận.

#### [encoding/json](/pkg/encoding/json/)

Phương thức mới [`SetIndent`](/pkg/encoding/json/#Encoder.SetIndent) của
[`Encoder`](/pkg/encoding/json/#Encoder)
đặt các tham số thụt lề cho mã hóa JSON,
giống như trong hàm cấp cao nhất
[`Indent`](/pkg/encoding/json/#Indent).

Phương thức mới [`SetEscapeHTML`](/pkg/encoding/json/#Encoder.SetEscapeHTML) của
[`Encoder`](/pkg/encoding/json/#Encoder)
kiểm soát xem các ký tự
`&`, `<`, và `>`
trong các chuỗi được trích dẫn có nên được thoát thành
`\u0026`, `\u003c`, và `\u003e`,
tương ứng hay không.
Như trong các bản phát hành trước, encoder mặc định áp dụng thoát này,
để tránh một số vấn đề có thể phát sinh khi nhúng JSON vào HTML.

Trong các phiên bản Go trước, package này chỉ hỗ trợ mã hóa và giải mã
các map sử dụng khóa có kiểu string.
Go 1.7 bổ sung hỗ trợ cho các map sử dụng khóa có kiểu số nguyên:
mã hóa sử dụng biểu diễn thập phân có trích dẫn làm khóa JSON.
Go 1.7 cũng bổ sung hỗ trợ mã hóa các map sử dụng khóa không phải string triển khai phương thức
`MarshalText`
(xem
[`encoding.TextMarshaler`](/pkg/encoding/#TextMarshaler)),
cũng như hỗ trợ giải mã các map sử dụng khóa không phải string triển khai phương thức
`UnmarshalText`
(xem
[`encoding.TextUnmarshaler`](/pkg/encoding/#TextUnmarshaler)).
Các phương thức này bị bỏ qua đối với khóa có kiểu string để bảo tồn
mã hóa và giải mã được sử dụng trong các phiên bản Go trước.

Khi mã hóa một slice của các byte được định kiểu,
[`Marshal`](/pkg/encoding/json/#Marshal)
giờ tạo ra một mảng các phần tử được mã hóa bằng cách sử dụng
phương thức `MarshalJSON` hoặc `MarshalText`
của kiểu byte đó nếu có,
chỉ quay lại dữ liệu chuỗi được mã hóa base64 mặc định nếu không có phương thức nào.
Các phiên bản Go trước chấp nhận cả mã hóa chuỗi base64 gốc
và mã hóa mảng (giả sử kiểu byte cũng triển khai
`UnmarshalJSON` hoặc `UnmarshalText`
tương ứng),
vì vậy thay đổi này nên tương thích ngược về mặt ngữ nghĩa với các phiên bản Go trước,
mặc dù nó có thay đổi mã hóa được chọn.

#### [go/build](/pkg/go/build/)

Để triển khai hỗ trợ mới của lệnh go cho các package chỉ nhị phân
và mã Fortran trong các package dựa trên cgo,
kiểu [`Package`](/pkg/go/build/#Package)
bổ sung các trường mới `BinaryOnly`, `CgoFFLAGS` và `FFiles`.

#### [go/doc](/pkg/go/doc/)

Để hỗ trợ thay đổi tương ứng trong `go` `test` được mô tả ở trên,
struct [`Example`](/pkg/go/doc/#Example) bổ sung một trường `Unordered`
chỉ ra liệu ví dụ có thể tạo ra các dòng đầu ra của nó theo bất kỳ thứ tự nào hay không.

#### [io](/pkg/io/)

Package bổ sung các hằng số mới
`SeekStart`, `SeekCurrent` và `SeekEnd`,
để sử dụng với các triển khai
[`Seeker`](/pkg/io/#Seeker).
Các hằng số này được ưu tiên hơn `os.SEEK_SET`, `os.SEEK_CUR` và `os.SEEK_END`,
nhưng các hằng số sau sẽ được bảo tồn để tương thích.

#### [math/big](/pkg/math/big/)

Kiểu [`Float`](/pkg/math/big/#Float) bổ sung các phương thức
[`GobEncode`](/pkg/math/big/#Float.GobEncode) và
[`GobDecode`](/pkg/math/big/#Float.GobDecode),
sao cho các giá trị kiểu `Float` giờ có thể được mã hóa và giải mã bằng cách sử dụng package
[`encoding/gob`](/pkg/encoding/gob/).

#### [math/rand](/pkg/math/rand/)

Hàm [`Read`](/pkg/math/rand/#Read) và
phương thức [`Read`](/pkg/math/rand/#Rand.Read) của
[`Rand`](/pkg/math/rand/#Rand)
giờ tạo ra một luồng byte giả ngẫu nhiên nhất quán và không
phụ thuộc vào kích thước của buffer đầu vào.

Tài liệu làm rõ rằng
các phương thức [`Seed`](/pkg/math/rand/#Rand.Seed) và [`Read`](/pkg/math/rand/#Rand.Read) của Rand
không an toàn để gọi đồng thời, mặc dù các hàm toàn cục
[`Seed`](/pkg/math/rand/#Seed) và [`Read`](/pkg/math/rand/#Read) là (và luôn luôn) an toàn.

#### [mime/multipart](/pkg/mime/multipart/)

Việc triển khai [`Writer`](/pkg/mime/multipart/#Writer)
giờ phát ra header của mỗi phần multipart được sắp xếp theo khóa.
Trước đây, việc lặp qua map dẫn đến header phần sử dụng thứ tự không xác định.

#### [net](/pkg/net/)

Là một phần của việc giới thiệu [context](#context), kiểu
[`Dialer`](/pkg/net/#Dialer) có phương thức mới
[`DialContext`](/pkg/net/#Dialer.DialContext), giống như
[`Dial`](/pkg/net/#Dialer.Dial) nhưng thêm
[`context.Context`](/pkg/context/#Context)
cho hoạt động dial.
Context được dùng để thay thế các trường `Cancel` và `Deadline` của `Dialer`,
nhưng việc triển khai vẫn tôn trọng chúng,
để tương thích ngược.

Phương thức [`String`](/pkg/net/#IP.String) của kiểu [`IP`](/pkg/net/#IP) đã thay đổi kết quả của nó cho các địa chỉ `IP` không hợp lệ.
Trong các bản phát hành trước, nếu một slice byte `IP` có độ dài khác 0, 4 hoặc 16, `String`
trả về `"?"`.
Go 1.7 thêm mã hóa thập lục phân của các byte, như trong `"?12ab"`.

Việc triển khai [phân giải tên](/pkg/net/#hdr-Name_Resolution) thuần Go
giờ tôn trọng ưu tiên của `nsswitch.conf` về ưu tiên của tra cứu DNS so với
tra cứu tệp cục bộ (tức là `/etc/hosts`).

#### [net/http](/pkg/net/http/)

Tài liệu của [`ResponseWriter`](/pkg/net/http/#ResponseWriter)
giờ làm rõ rằng bắt đầu ghi phản hồi
có thể ngăn các lần đọc trong tương lai trên thân yêu cầu.
Để tương thích tối đa, các triển khai được khuyến khích
đọc toàn bộ thân yêu cầu trước khi ghi bất kỳ phần nào của phản hồi.

Là một phần của việc giới thiệu [context](#context),
[`Request`](/pkg/net/http/#Request) có các phương thức mới
[`Context`](/pkg/net/http/#Request.Context), để truy xuất context liên quan, và
[`WithContext`](/pkg/net/http/#Request.WithContext), để xây dựng một bản sao của `Request`
với context đã sửa đổi.

Trong việc triển khai [`Server`](/pkg/net/http/#Server),
[`Serve`](/pkg/net/http/#Server.Serve) ghi vào context yêu cầu
cả `*Server` bên dưới bằng khóa `ServerContextKey`
và địa chỉ cục bộ mà yêu cầu được nhận (một
[`Addr`](/pkg/net/#Addr)) bằng khóa `LocalAddrContextKey`.
Ví dụ, địa chỉ nhận được một yêu cầu là
`req.Context().Value(http.LocalAddrContextKey).(net.Addr)`.

Phương thức [`Serve`](/pkg/net/http/#Server.Serve) của server
giờ chỉ bật hỗ trợ HTTP/2 nếu trường `Server.TLSConfig` là `nil`
hoặc bao gồm `"h2"` trong `TLSConfig.NextProtos` của nó.

Việc triển khai server giờ
đệm mã phản hồi nhỏ hơn 100 thành ba chữ số
như được yêu cầu bởi giao thức,
sao cho `w.WriteHeader(5)` sử dụng trạng thái phản hồi HTTP
`005`, không chỉ `5`.

Việc triển khai server giờ chỉ gửi một header "Transfer-Encoding" khi "chunked"
được đặt tường minh, theo [RFC 7230](https://tools.ietf.org/html/rfc7230#section-3.3.1).

Việc triển khai server giờ nghiêm ngặt hơn về việc từ chối các yêu cầu với phiên bản HTTP không hợp lệ.
Các yêu cầu không hợp lệ tuyên bố là HTTP/0.x giờ bị từ chối (HTTP/0.9 chưa bao giờ được hỗ trợ đầy đủ),
và các yêu cầu HTTP/2 dạng văn bản thuần khác ngoài yêu cầu nâng cấp "PRI \* HTTP/2.0" giờ cũng bị từ chối.
Server tiếp tục xử lý các yêu cầu HTTP/2 được mã hóa.

Trong server, mã trạng thái 200 được gửi lại bởi timeout handler trên một
thân phản hồi rỗng, thay vì gửi lại 0 là mã trạng thái.

Trong client, việc triển khai
[`Transport`](/pkg/net/http/#Transport) truyền context yêu cầu
cho bất kỳ hoạt động dial nào kết nối với server từ xa.
Nếu cần một dialer tùy chỉnh, trường `Transport` mới
`DialContext` được ưu tiên hơn trường `Dial` hiện có,
để cho phép transport cung cấp context.

[`Transport`](/pkg/net/http/#Transport) cũng thêm các trường
`IdleConnTimeout`,
`MaxIdleConns`,
và
`MaxResponseHeaderBytes`
để giúp kiểm soát tài nguyên client được sử dụng bởi
các server nhàn rỗi hoặc nói chuyện nhiều.

Hàm `CheckRedirect` được cấu hình của [`Client`](/pkg/net/http/#Client) giờ có thể
trả về `ErrUseLastResponse` để chỉ ra rằng
phản hồi chuyển hướng gần đây nhất nên được trả về là
kết quả của yêu cầu HTTP.
Phản hồi đó giờ có sẵn cho hàm `CheckRedirect`
là `req.Response`.

Từ Go 1, hành vi mặc định của HTTP client là
yêu cầu nén phía server
sử dụng header yêu cầu `Accept-Encoding`
và sau đó giải nén thân phản hồi một cách trong suốt,
và hành vi này có thể điều chỉnh bằng trường `DisableCompression` của
[`Transport`](/pkg/net/http/#Transport).
Trong Go 1.7, để hỗ trợ việc triển khai proxy HTTP, trường mới
`Uncompressed` của [`Response`](/pkg/net/http/#Response)
báo cáo xem giải nén trong suốt này có xảy ra hay không.

[`DetectContentType`](/pkg/net/http/#DetectContentType)
bổ sung hỗ trợ cho một số loại nội dung âm thanh và video mới.

#### [net/http/cgi](/pkg/net/http/cgi/)

[`Handler`](/pkg/net/http/cgi/#Handler)
thêm trường mới
`Stderr`
cho phép chuyển hướng lỗi chuẩn của quá trình con ra khỏi
lỗi chuẩn của quá trình máy chủ.

#### [net/http/httptest](/pkg/net/http/httptest/)

Hàm mới
[`NewRequest`](/pkg/net/http/httptest/#NewRequest)
chuẩn bị một
[`http.Request`](/pkg/net/http/#Request) mới
phù hợp để truyền cho một
[`http.Handler`](/pkg/net/http/#Handler) trong quá trình kiểm thử.

Phương thức mới [`Result`](/pkg/net/http/httptest/#ResponseRecorder.Result) của
[`ResponseRecorder`](/pkg/net/http/httptest/#ResponseRecorder)
trả về
[`http.Response`](/pkg/net/http/#Response) đã ghi lại.
Các kiểm thử cần kiểm tra header hoặc trailer của phản hồi
nên gọi `Result` và kiểm tra các trường phản hồi
thay vì truy cập trực tiếp vào `HeaderMap` của `ResponseRecorder`.

#### [net/http/httputil](/pkg/net/http/httputil/)

Việc triển khai [`ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy) giờ phản hồi với "502 Bad Gateway"
khi nó không thể tiếp cận backend; trong các bản phát hành trước nó phản hồi với "500 Internal Server Error."

Cả
[`ClientConn`](/pkg/net/http/httputil/#ClientConn) và
[`ServerConn`](/pkg/net/http/httputil/#ServerConn) đã được ghi lại là không dùng nữa.
Chúng là mức thấp, cũ, và không được sử dụng bởi stack HTTP hiện tại của Go
và sẽ không còn được cập nhật.
Các chương trình nên sử dụng
[`http.Client`](/pkg/net/http/#Client),
[`http.Transport`](/pkg/net/http/#Transport),
và
[`http.Server`](/pkg/net/http/#Server)
thay thế.

#### [net/http/pprof](/pkg/net/http/pprof/)

Trình xử lý HTTP trace runtime, được cài đặt để xử lý đường dẫn `/debug/pprof/trace`,
giờ chấp nhận một số phân số trong tham số truy vấn `seconds` của nó,
cho phép thu thập các trace cho các khoảng thời gian nhỏ hơn một giây.
Điều này đặc biệt hữu ích trên các server bận rộn.

#### [net/mail](/pkg/net/mail/)

Bộ phân tích địa chỉ giờ cho phép văn bản UTF-8 không được thoát trong địa chỉ
theo [RFC 6532](https://tools.ietf.org/html/rfc6532),
nhưng nó không áp dụng bất kỳ chuẩn hóa nào cho kết quả.
Để tương thích với các bộ phân tích thư cũ hơn,
bộ mã hóa địa chỉ, cụ thể là
phương thức [`String`](/pkg/net/mail/#Address.String) của
[`Address`](/pkg/net/mail/#Address),
tiếp tục thoát tất cả văn bản UTF-8 theo [RFC 5322](https://tools.ietf.org/html/rfc5322).

Hàm [`ParseAddress`](/pkg/net/mail/#ParseAddress)
và
phương thức [`AddressParser.Parse`](/pkg/net/mail/#AddressParser.Parse)
nghiêm ngặt hơn.
Trước đây chúng bỏ qua bất kỳ ký tự nào theo sau địa chỉ email, nhưng
giờ sẽ trả về lỗi cho bất cứ thứ gì khác ngoài khoảng trắng.

#### [net/url](/pkg/net/url/)

Trường mới `ForceQuery` của
[`URL`](/pkg/net/url/#URL)
ghi lại xem URL có phải có chuỗi truy vấn hay không,
để phân biệt các URL không có chuỗi truy vấn (như `/search`)
với các URL có chuỗi truy vấn rỗng (như `/search?`).

#### [os](/pkg/os/)

[`IsExist`](/pkg/os/#IsExist) giờ trả về true cho `syscall.ENOTEMPTY`,
trên các hệ thống có lỗi đó.

Trên Windows,
[`Remove`](/pkg/os/#Remove) giờ xóa các tệp chỉ đọc khi có thể,
làm cho việc triển khai hoạt động như trên
các hệ thống không phải Windows.

#### [os/exec](/pkg/os/exec/)

Là một phần của việc giới thiệu [context](#context),
constructor mới
[`CommandContext`](/pkg/os/exec/#CommandContext)
giống như
[`Command`](/pkg/os/exec/#Command) nhưng bao gồm một context có thể được sử dụng để hủy việc thực thi lệnh.

#### [os/user](/pkg/os/user/)

Hàm [`Current`](/pkg/os/user/#Current)
giờ được triển khai ngay cả khi cgo không có sẵn.

Kiểu [`Group`](/pkg/os/user/#Group) mới,
cùng với các hàm tra cứu
[`LookupGroup`](/pkg/os/user/#LookupGroup) và
[`LookupGroupId`](/pkg/os/user/#LookupGroupId)
và trường mới `GroupIds` trong struct `User`,
cung cấp quyền truy cập vào thông tin nhóm người dùng cụ thể cho hệ thống.

#### [reflect](/pkg/reflect/)

Mặc dù
phương thức [`Field`](/pkg/reflect/#Value.Field) của
[`Value`](/pkg/reflect/#Value) luôn được ghi lại là panic
nếu số trường `i` đã cho nằm ngoài phạm vi, thay vào đó nó đã
im lặng trả về một
[`Value`](/pkg/reflect/#Value) bằng không.
Go 1.7 thay đổi phương thức để hoạt động như đã ghi lại.

Hàm mới
[`StructOf`](/pkg/reflect/#StructOf)
xây dựng một kiểu struct trong thời gian chạy.
Nó hoàn thiện tập hợp các constructor kiểu, tham gia cùng
[`ArrayOf`](/pkg/reflect/#ArrayOf),
[`ChanOf`](/pkg/reflect/#ChanOf),
[`FuncOf`](/pkg/reflect/#FuncOf),
[`MapOf`](/pkg/reflect/#MapOf),
[`PtrTo`](/pkg/reflect/#PtrTo),
và
[`SliceOf`](/pkg/reflect/#SliceOf).

Phương thức mới [`Lookup`](/pkg/reflect/#StructTag.Lookup) của
[`StructTag`](/pkg/reflect/#StructTag)
giống như
[`Get`](/pkg/reflect/#StructTag.Get)
nhưng phân biệt tag không chứa khóa đã cho
với tag liên kết chuỗi rỗng với khóa đã cho.

Các phương thức [`Method`](/pkg/reflect/#Type.Method) và
[`NumMethod`](/pkg/reflect/#Type.NumMethod)
của
[`Type`](/pkg/reflect/#Type) và
[`Value`](/pkg/reflect/#Value)
không còn trả về hoặc đếm các phương thức không được xuất khẩu.

#### [strings](/pkg/strings/)

Trong các bản phát hành Go trước, nếu
phương thức [`Read`](/pkg/strings/#Reader.Read) của
[`Reader`](/pkg/strings/#Reader)
được yêu cầu không byte nào với không còn dữ liệu, nó sẽ
trả về số đếm là 0 và không có lỗi.
Giờ nó trả về số đếm là 0 và lỗi
[`io.EOF`](/pkg/io/#EOF).

Kiểu [`Reader`](/pkg/strings/#Reader) có phương thức mới
[`Reset`](/pkg/strings/#Reader.Reset) để cho phép tái sử dụng `Reader`.

#### [time](/pkg/time/)

Phương thức `time.Duration.String` của
[`Duration`](/pkg/time/#Duration) giờ báo cáo duration bằng không là `"0s"`, không phải `"0"`.
[`ParseDuration`](/pkg/time/#ParseDuration) tiếp tục chấp nhận cả hai dạng.

Lời gọi phương thức `time.Local.String()` giờ trả về `"Local"` trên tất cả hệ thống;
trong các bản phát hành trước, nó trả về chuỗi rỗng trên Windows.

Cơ sở dữ liệu múi giờ trong
`$GOROOT/lib/time` đã được cập nhật
lên IANA phiên bản 2016d.
Cơ sở dữ liệu dự phòng này chỉ được sử dụng khi cơ sở dữ liệu múi giờ hệ thống
không thể được tìm thấy, ví dụ trên Windows.
Danh sách viết tắt múi giờ Windows cũng đã được cập nhật.

#### [syscall](/pkg/syscall/)

Trên Linux, struct
[`SysProcAttr`](/pkg/syscall/#SysProcAttr)
(được sử dụng trong
trường `SysProcAttr` của [`os/exec.Cmd`](/pkg/os/exec/#Cmd))
có trường mới `Unshareflags`.
Nếu trường khác không, quá trình con được tạo bởi
[`ForkExec`](/pkg/syscall/#ForkExec)
(được sử dụng trong phương thức `Run` của `exec.Cmd`)
sẽ gọi lời gọi hệ thống
[_unshare_(2)](https://man7.org/linux/man-pages/man2/unshare.2.html)
trước khi thực thi chương trình mới.

#### [unicode](/pkg/unicode/)

Package [`unicode`](/pkg/unicode/) và hỗ trợ liên quan
trên toàn hệ thống đã được nâng cấp từ phiên bản 8.0 lên
[Unicode 9.0](https://www.unicode.org/versions/Unicode9.0.0/).
