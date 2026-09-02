---
title: Ghi chú phát hành Go 1.8
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

## Giới thiệu về Go 1.8 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.8, ra đời sáu tháng sau [Go 1.7](go1.7).
Phần lớn các thay đổi nằm ở phần triển khai bộ công cụ, runtime và các thư viện.
Có [hai thay đổi nhỏ](#language) đối với đặc tả ngôn ngữ.
Như thường lệ, bản phát hành duy trì [cam kết tương thích](/doc/go1compat.html) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

Bản phát hành [bổ sung hỗ trợ cho MIPS 32-bit](#ports),
[cập nhật backend trình biên dịch](#compiler) để tạo mã hiệu quả hơn,
[giảm thời gian tạm dừng GC](#gc) bằng cách loại bỏ quét stack stop-the-world,
[bổ sung hỗ trợ HTTP/2 Push](#h2push),
[bổ sung tắt nhẹ nhàng HTTP](#http_shutdown),
[bổ sung hỗ trợ context nhiều hơn](#more_context),
[cho phép profiling mutex](#mutex_prof),
và [đơn giản hóa sắp xếp slice](#sort_slice).

## Thay đổi về ngôn ngữ {#language}

Khi chuyển đổi tường minh một giá trị từ kiểu struct này sang kiểu struct khác,
từ Go 1.8 các tag bị bỏ qua. Do đó, hai struct chỉ khác nhau về tag có thể được chuyển đổi từ cái này sang cái kia:

	func example() {
		type T1 struct {
			X int `json:"foo"`
		}
		type T2 struct {
			X int `json:"bar"`
		}
		var v1 T1
		var v2 T2
		v1 = T1(v2) // hợp lệ từ bây giờ
	}

<!-- CL 17711 -->
Đặc tả ngôn ngữ giờ chỉ yêu cầu các triển khai
hỗ trợ tối đa số mũ 16-bit trong các hằng số dấu phẩy động. Điều này không ảnh hưởng
đến cả trình biên dịch "[`gc`](/cmd/compile/)" hay
trình biên dịch `gccgo`, cả hai vẫn hỗ trợ số mũ 32-bit.

## Các port {#ports}

Go giờ hỗ trợ MIPS 32-bit trên Linux cho cả máy big-endian
(`linux/mips`) và little-endian
(`linux/mipsle`) triển khai tập lệnh MIPS32r1 với FPU
hoặc mô phỏng FPU kernel. Lưu ý rằng nhiều bộ định tuyến dựa trên MIPS phổ biến không có FPU và
có firmware không bật mô phỏng FPU kernel; Go sẽ không chạy trên các máy đó.

Trên DragonFly BSD, Go giờ yêu cầu DragonFly 4.4.4 hoặc mới hơn. <!-- CL 29491, CL 29971 -->

Trên OpenBSD, Go giờ yêu cầu OpenBSD 5.9 hoặc mới hơn. <!-- CL 34093 -->

Hỗ trợ mạng của port Plan 9 giờ đầy đủ hơn nhiều
và khớp với hành vi của Unix và Windows về deadline
và hủy. Về yêu cầu kernel Plan 9, xem
[trang wiki Plan 9](/wiki/Plan9).

Go 1.8 giờ chỉ hỗ trợ OS X 10.8 hoặc mới hơn. Đây có thể là
bản phát hành Go cuối cùng hỗ trợ 10.8. Việc biên dịch Go hoặc chạy
các tệp nhị phân trên các phiên bản OS X cũ hơn chưa được kiểm thử.

Go 1.8 sẽ là bản phát hành cuối cùng hỗ trợ Linux trên bộ xử lý ARMv5E và ARMv6:
Go 1.9 có thể sẽ yêu cầu ARMv6K (như được tìm thấy trong Raspberry Pi 1) hoặc mới hơn.
Để xác định xem một hệ thống Linux có phải ARMv6K hay mới hơn hay không, chạy
"`go` `tool` `dist` `-check-armv6k`"
(để tạo điều kiện kiểm thử, cũng có thể chỉ sao chép lệnh `dist` sang
hệ thống mà không cần cài đặt bản sao đầy đủ của Go 1.8)
và nếu chương trình kết thúc với đầu ra "ARMv6K supported." thì hệ thống
triển khai ARMv6K hoặc mới hơn.
Go trên các hệ thống ARM không phải Linux đã yêu cầu ARMv6K hoặc mới hơn.

<!-- CL 31596, go.dev/issue/17528 -->
`zos` giờ là giá trị được nhận dạng cho `GOOS`,
được dành riêng cho hệ điều hành z/OS.

### Các vấn đề đã biết {#known_issues}

Có một số bất ổn trên FreeBSD và NetBSD đã biết nhưng chưa được hiểu rõ.
Điều này có thể dẫn đến crash chương trình trong các trường hợp hiếm gặp.
Xem
[issue 15658](/issue/15658) và
[issue 16511](/issue/16511).
Mọi sự giúp đỡ trong việc giải quyết các vấn đề này đều được hoan nghênh.

## Công cụ {#tools}

### Assembler {#cmd_asm}

Đối với các hệ thống x86 64-bit, các lệnh sau đã được thêm vào:
`VBROADCASTSD`,
`BROADCASTSS`,
`MOVDDUP`,
`MOVSHDUP`,
`MOVSLDUP`,
`VMOVDDUP`,
`VMOVSHDUP`, và
`VMOVSLDUP`.

Đối với các hệ thống PPC 64-bit, các lệnh scalar vector chung đã được
thêm vào:
`LXS`,
`LXSDX`,
`LXSI`,
`LXSIWAX`,
`LXSIWZX`,
`LXV`,
`LXVD2X`,
`LXVDSX`,
`LXVW4X`,
`MFVSR`,
`MFVSRD`,
`MFVSRWZ`,
`MTVSR`,
`MTVSRD`,
`MTVSRWA`,
`MTVSRWZ`,
`STXS`,
`STXSDX`,
`STXSI`,
`STXSIWX`,
`STXV`,
`STXVD2X`,
`STXVW4X`,
`XSCV`,
`XSCVDPSP`,
`XSCVDPSPN`,
`XSCVDPSXDS`,
`XSCVDPSXWS`,
`XSCVDPUXDS`,
`XSCVDPUXWS`,
`XSCVSPDP`,
`XSCVSPDPN`,
`XSCVSXDDP`,
`XSCVSXDSP`,
`XSCVUXDDP`,
`XSCVUXDSP`,
`XSCVX`,
`XSCVXP`,
`XVCV`,
`XVCVDPSP`,
`XVCVDPSXDS`,
`XVCVDPSXWS`,
`XVCVDPUXDS`,
`XVCVDPUXWS`,
`XVCVSPDP`,
`XVCVSPSXDS`,
`XVCVSPSXWS`,
`XVCVSPUXDS`,
`XVCVSPUXWS`,
`XVCVSXDDP`,
`XVCVSXDSP`,
`XVCVSXWDP`,
`XVCVSXWSP`,
`XVCVUXDDP`,
`XVCVUXDSP`,
`XVCVUXWDP`,
`XVCVUXWSP`,
`XVCVX`,
`XVCVXP`,
`XXLAND`,
`XXLANDC`,
`XXLANDQ`,
`XXLEQV`,
`XXLNAND`,
`XXLNOR`,
`XXLOR`,
`XXLORC`,
`XXLORQ`,
`XXLXOR`,
`XXMRG`,
`XXMRGHW`,
`XXMRGLW`,
`XXPERM`,
`XXPERMDI`,
`XXSEL`,
`XXSI`,
`XXSLDWI`,
`XXSPLT`, và
`XXSPLTW`.

### Yacc {#tool_yacc}

<!-- CL 27324, CL 27325 -->
Công cụ `yacc` (trước đây có sẵn bằng cách chạy
"`go` `tool` `yacc`") đã bị loại bỏ.
Từ Go 1.7, nó không còn được sử dụng bởi trình biên dịch Go.
Nó đã được chuyển sang kho "tools" và hiện có sẵn tại
[`golang.org/x/tools/cmd/goyacc`](https://godoc.org/golang.org/x/tools/cmd/goyacc).

### Fix {#tool_fix}

<!-- CL 28872 -->
Công cụ `fix` có sửa chữa "`context`" mới
để thay đổi các import từ "`golang.org/x/net/context`"
thành "[`context`](/pkg/context/)".

### Pprof {#tool_pprof}

<!-- CL 33157 -->
Công cụ `pprof` giờ có thể profile các server TLS
và bỏ qua xác thực chứng chỉ bằng cách sử dụng lược đồ URL "`https+insecure`".

<!-- CL 23781 -->
Đầu ra callgrind giờ có độ chi tiết cấp lệnh.

### Trace {#tool_trace}

<!-- CL 23324 -->
Công cụ `trace` có cờ mới `-pprof` để
tạo ra các profile blocking và latency tương thích với pprof từ một
execution trace.

<!-- CL 30017, CL 30702 -->
Các sự kiện thu gom rác giờ được hiển thị rõ ràng hơn trong
trình xem execution trace. Hoạt động thu gom rác được hiển thị trên
hàng riêng của nó và các goroutine GC helper được chú thích với vai trò của chúng.

### Vet {#tool_vet}

Vet nghiêm ngặt hơn theo một số cách và ít nghiêm ngặt hơn ở những nơi
trước đây gây ra dương tính giả.

Vet giờ kiểm tra việc sao chép mảng khóa, các tag struct JSON và XML trùng lặp,
các tag struct không phân cách bằng khoảng trắng,
các lời gọi trì hoãn đến `Response.Body.Close` của HTTP
trước khi kiểm tra lỗi, và
các đối số được đánh chỉ mục trong `Printf`.
Nó cũng cải thiện các kiểm tra hiện có.


### Bộ công cụ biên dịch {#compiler}

Go 1.7 đã giới thiệu một backend trình biên dịch mới cho các hệ thống x86 64-bit.
Trong Go 1.8, backend đó đã được phát triển thêm và giờ được sử dụng cho
tất cả kiến trúc.

Backend mới, dựa trên
[dạng gán đơn tĩnh](https://en.wikipedia.org/wiki/Static_single_assignment_form) (SSA),
tạo ra mã nhỏ gọn, hiệu quả hơn
và cung cấp nền tảng tốt hơn cho các tối ưu hóa
như loại bỏ kiểm tra giới hạn.
Backend mới giảm thời gian CPU cần thiết bởi
các chương trình benchmark của chúng tôi từ 20-30%
trên các hệ thống ARM 32-bit. Đối với các hệ thống x86 64-bit, vốn đã sử dụng backend SSA trong
Go 1.7, mức tăng khiêm tốn hơn là 0-10%. Các kiến trúc khác có thể sẽ
thấy các cải tiến gần hơn với con số ARM 32-bit.

Cờ trình biên dịch `-ssa=0` tạm thời được giới thiệu trong Go 1.7
để tắt backend mới đã bị loại bỏ trong Go 1.8.

Ngoài việc bật backend trình biên dịch mới cho tất cả hệ thống,
Go 1.8 cũng giới thiệu một frontend trình biên dịch mới. Frontend trình biên dịch mới
sẽ không được người dùng chú ý nhưng là nền tảng cho
các công việc hiệu suất trong tương lai.

Trình biên dịch và trình liên kết đã được tối ưu hóa và chạy nhanh hơn trong bản phát hành này
so với Go 1.7, mặc dù chúng vẫn chậm hơn mức mong muốn
và sẽ tiếp tục được tối ưu hóa trong các bản phát hành tương lai.
So với bản phát hành trước, Go 1.8
[nhanh hơn khoảng 15%](https://dave.cheney.net/2016/11/19/go-1-8-toolchain-improvements).

### Cgo {#cmd_cgo}

<!-- CL 31141 -->
Công cụ Go giờ ghi nhớ giá trị của biến môi trường `CGO_ENABLED`
được đặt trong quá trình `make.bash` và áp dụng nó cho tất cả các lần biên dịch trong tương lai
theo mặc định để sửa issue [#12808](/issue/12808).
Khi thực hiện biên dịch native, hiếm khi cần đặt tường minh
biến môi trường `CGO_ENABLED` vì `make.bash`
sẽ phát hiện cài đặt đúng tự động. Lý do chính để đặt tường minh
biến môi trường `CGO_ENABLED` là khi môi trường của bạn
hỗ trợ cgo, nhưng bạn tường minh không muốn hỗ trợ cgo, trong trường hợp đó, đặt
`CGO_ENABLED=0` trong quá trình `make.bash` hoặc `all.bash`.

<!-- CL 29991 -->
Biến môi trường `PKG_CONFIG` giờ có thể được sử dụng để
đặt chương trình chạy để xử lý các chỉ thị `#cgo` `pkg-config`.
Giá trị mặc định là `pkg-config`, chương trình
luôn được sử dụng bởi các bản phát hành trước. Điều này nhằm giúp việc cross-compile
mã [cgo](/cmd/cgo/) dễ dàng hơn.

<!-- CL 32354 -->
Công cụ [cgo](/cmd/cgo/) giờ hỗ trợ tùy chọn `-srcdir`,
được sử dụng bởi lệnh [go](/cmd/go/).

<!-- CL 31768, 31811 -->
Nếu mã [cgo](/cmd/cgo/) gọi `C.malloc`, và
`malloc` trả về `NULL`, chương trình sẽ
crash với lỗi hết bộ nhớ.
`C.malloc` sẽ không bao giờ trả về nil.
Không giống như hầu hết các hàm C, `C.malloc` không thể được sử dụng trong
dạng hai kết quả trả về giá trị errno.

<!-- CL 33237 -->
Nếu mã [cgo](/cmd/cgo/) gọi một hàm C truyền
con trỏ đến một C union, và nếu C union có thể chứa bất kỳ giá trị con trỏ nào,
và nếu [kiểm tra con trỏ
cgo](/cmd/cgo/#hdr-Passing_pointers) được bật (như theo mặc định), giá trị union giờ
được kiểm tra cho các con trỏ Go.

### Gccgo {#gccgo}

Do sự căn chỉnh lịch phát hành sáu tháng một lần của Go với lịch phát hành hàng năm của GCC,
GCC phiên bản 6 chứa phiên bản Go 1.6.1 của gccgo.
Chúng tôi kỳ vọng bản phát hành tiếp theo, GCC 7, sẽ chứa phiên bản Go 1.8
của gccgo.

### GOPATH mặc định {#gopath}

[Biến môi trường `GOPATH`](/cmd/go/#hdr-GOPATH_environment_variable)
giờ có giá trị mặc định nếu nó
không được đặt. Nó mặc định là
`$HOME/go` trên Unix và
`%USERPROFILE%/go` trên Windows.

### Go get {#go_get}

<!-- CL 34818 -->
Lệnh "`go` `get`" giờ luôn tôn trọng
các biến môi trường proxy HTTP, bất kể cờ
<code style="white-space:nowrap">-insecure</code> có được sử dụng hay không. Trong các bản phát hành trước, cờ
<code style="white-space:nowrap">-insecure</code> có tác dụng phụ là không sử dụng proxy.

### Go bug {#go_bug}

Lệnh mới
"[`go` `bug`](/cmd/go/#hdr-Print_information_for_bug_reports)"
bắt đầu một báo cáo lỗi trên GitHub, được điền sẵn
với thông tin về hệ thống hiện tại.

### Go doc {#cmd_doc}

<!-- CL 25419 -->
Lệnh
"[`go` `doc`](/cmd/go/#hdr-Show_documentation_for_package_or_symbol)"
giờ nhóm các hằng số và biến với kiểu của chúng,
theo hành vi của
[`godoc`](/cmd/godoc/).

<!-- CL 25420 -->
Để cải thiện tính dễ đọc của đầu ra `doc`,
mỗi tóm tắt của các mục cấp đầu được đảm bảo
chiếm một dòng.

<!-- CL 31852 -->
Tài liệu cho một phương thức cụ thể trong định nghĩa interface giờ có thể được yêu cầu, như trong
"`go` `doc` `net.Conn.SetDeadline`".

### Plugins {#plugin}

Go giờ cung cấp hỗ trợ sơ bộ cho plugin với chế độ build "`plugin`"
để tạo ra các plugin viết bằng Go, và một
package [`plugin`](/pkg/plugin/) mới để
tải các plugin đó trong thời gian chạy. Hỗ trợ plugin hiện chỉ
có sẵn trên Linux. Vui lòng báo cáo bất kỳ vấn đề nào.

## Runtime {#runtime}

### Thời gian sống của đối số {#liveness}

<!-- Issue 15843 -->
Bộ gom rác không còn coi
các đối số là sống trong suốt toàn bộ hàm. Để biết thêm
thông tin, và cách buộc một biến vẫn sống, xem
hàm [`runtime.KeepAlive`](/pkg/runtime/#KeepAlive)
được thêm vào trong Go 1.7.

_Cập nhật:_
Mã đặt finalizer trên một đối tượng được cấp phát có thể cần thêm
các lời gọi đến `runtime.KeepAlive` trong các hàm hoặc phương thức
sử dụng đối tượng đó.
Đọc
[tài liệu `KeepAlive`](/pkg/runtime/#KeepAlive) và ví dụ của nó để biết thêm chi tiết.

### Sử dụng sai Map đồng thời {#mapiter}

Trong Go 1.6, runtime
[đã thêm tính năng phát hiện nhẹ, theo khả năng tốt nhất việc sử dụng đồng thời sai của map](/doc/go1.6#runtime). Bản phát hành này
cải thiện bộ phát hiện đó với hỗ trợ để phát hiện các chương trình
viết vào và lặp qua map đồng thời.

Như thường lệ, nếu một goroutine đang ghi vào một map, không goroutine nào khác nên
đọc (bao gồm cả lặp) hoặc ghi map đó đồng thời.
Nếu runtime phát hiện điều kiện này, nó sẽ in chẩn đoán và làm dừng chương trình.
Cách tốt nhất để tìm hiểu thêm về vấn đề là chạy chương trình
dưới
[bộ phát hiện race](/blog/race-detector),
sẽ xác định race đáng tin cậy hơn
và cung cấp thêm chi tiết.

### Tài liệu MemStats {#memstats}

<!-- CL 28972 -->
Kiểu [`runtime.MemStats`](/pkg/runtime/#MemStats)
đã được ghi lại đầy đủ hơn.

## Hiệu suất {#performance}

Như thường lệ, các thay đổi rất chung chung và đa dạng nên rất khó đưa ra tuyên bố chính xác
về hiệu suất.
Hầu hết các chương trình nên chạy nhanh hơn một chút,
do tăng tốc trong bộ gom rác và
các tối ưu hóa trong thư viện chuẩn.

Đã có các tối ưu hóa cho các triển khai trong các package
[`bytes`](/pkg/bytes/),
[`crypto/aes`](/pkg/crypto/aes/),
[`crypto/cipher`](/pkg/crypto/cipher/),
[`crypto/elliptic`](/pkg/crypto/elliptic/),
[`crypto/sha256`](/pkg/crypto/sha256/),
[`crypto/sha512`](/pkg/crypto/sha512/),
[`encoding/asn1`](/pkg/encoding/asn1/),
[`encoding/csv`](/pkg/encoding/csv/),
[`encoding/hex`](/pkg/encoding/hex/),
[`encoding/json`](/pkg/encoding/json/),
[`hash/crc32`](/pkg/hash/crc32/),
[`image/color`](/pkg/image/color/),
[`image/draw`](/pkg/image/draw/),
[`math`](/pkg/math/),
[`math/big`](/pkg/math/big/),
[`reflect`](/pkg/reflect/),
[`regexp`](/pkg/regexp/),
[`runtime`](/pkg/runtime/),
[`strconv`](/pkg/strconv/),
[`strings`](/pkg/strings/),
[`syscall`](/pkg/syscall/),
[`text/template`](/pkg/text/template/) và
[`unicode/utf8`](/pkg/unicode/utf8/).

### Bộ gom rác {#gc}

Thời gian tạm dừng của bộ gom rác nên ngắn hơn đáng kể so với
Go 1.7, thường dưới 100 micro giây và thường thấp tới
10 micro giây.
Xem
[tài liệu về việc loại bỏ quét stack stop-the-world](https://github.com/golang/proposal/blob/master/design/17503-eliminate-rescan.md)
để biết chi tiết. Vẫn còn nhiều công việc cho Go 1.9.

### Defer {#defer}

<!-- CL 29656, CL 29656 -->

Chi phí của [các lời gọi hàm trì hoãn](/ref/spec/#Defer_statements) đã giảm khoảng một nửa.

### Cgo {#cgoperf}

Chi phí của các lời gọi từ Go vào C đã giảm khoảng một nửa.

## Thư viện chuẩn {#library}

### Ví dụ {#examples}

Các ví dụ đã được thêm vào tài liệu trên nhiều package.

### Sort {#sort_slice}

Package [sort](/pkg/sort/)
giờ bao gồm một hàm tiện lợi
[`Slice`](/pkg/sort/#Slice) để sắp xếp một
slice cho hàm _less_ đã cho.
Trong nhiều trường hợp, điều này có nghĩa là không cần thiết phải viết một kiểu sorter mới.

Cũng có mới là
[`SliceStable`](/pkg/sort/#SliceStable) và
[`SliceIsSorted`](/pkg/sort/#SliceIsSorted).

### HTTP/2 Push {#h2push}

Package [net/http](/pkg/net/http/) giờ bao gồm một
cơ chế để
gửi các HTTP/2 server push từ một
[`Handler`](/pkg/net/http/#Handler).
Tương tự như các interface `Flusher` và `Hijacker` hiện có,
một
[`ResponseWriter`](/pkg/net/http/#ResponseWriter) HTTP/2
giờ triển khai interface
[`Pusher`](/pkg/net/http/#Pusher) mới.

### Tắt nhẹ nhàng HTTP Server {#http_shutdown}

<!-- CL 32329 -->
HTTP Server giờ có hỗ trợ tắt nhẹ nhàng bằng phương thức mới
[`Server.Shutdown`](/pkg/net/http/#Server.Shutdown)
và tắt đột ngột bằng phương thức mới
[`Server.Close`](/pkg/net/http/#Server.Close).

### Hỗ trợ Context nhiều hơn {#more_context}

Tiếp tục [việc áp dụng của Go 1.7](/doc/go1.7#context)
[`context.Context`](/pkg/context/#Context)
vào thư viện chuẩn, Go 1.8 bổ sung nhiều hỗ trợ context hơn
vào các package hiện có:

  - [`Server.Shutdown`](/pkg/net/http/#Server.Shutdown) mới
    nhận đối số context.
  - Đã có [các bổ sung đáng kể](#database_sql) cho package
    [database/sql](/pkg/database/sql/) với hỗ trợ context.
  - Tất cả chín phương thức `Lookup` mới trên
    [`net.Resolver`](/pkg/net/#Resolver) mới
    đều nhận context.

### Profiling tranh chấp Mutex {#mutex_prof}

Runtime và các công cụ giờ hỗ trợ profiling các mutex tranh chấp.

Hầu hết người dùng sẽ muốn sử dụng cờ `-mutexprofile` mới với "[`go` `test`](/cmd/go/#hdr-Description_of_testing_flags)",
và sau đó sử dụng [pprof](/cmd/pprof/) trên tệp kết quả.

Hỗ trợ cấp thấp hơn cũng có sẵn thông qua
[`MutexProfile`](/pkg/runtime/#MutexProfile) và
[`SetMutexProfileFraction`](/pkg/runtime/#SetMutexProfileFraction) mới.

Một hạn chế đã biết cho Go 1.8 là profile chỉ báo cáo tranh chấp cho
[`sync.Mutex`](/pkg/sync/#Mutex),
không phải
[`sync.RWMutex`](/pkg/sync/#RWMutex).

### Thay đổi nhỏ đối với thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ cho thư viện,
được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1
trong tâm trí. Các phần sau liệt kê các thay đổi và bổ sung có thể nhìn thấy với người dùng.
Các tối ưu hóa và sửa lỗi nhỏ không được liệt kê.

#### [archive/tar](/pkg/archive/tar/)

<!-- CL 28471, CL 31440, CL 31441, CL 31444, CL 28418, CL 31439 -->
Việc triển khai tar sửa nhiều lỗi trong các trường hợp góc cạnh của định dạng tệp.
[`Reader`](/pkg/archive/tar/#Reader)
giờ có thể xử lý các tệp tar ở định dạng PAX với các mục lớn hơn 8GB.
[`Writer`](/pkg/archive/tar/#Writer)
không còn tạo ra các tệp tar không hợp lệ trong một số tình huống liên quan đến tên đường dẫn dài.

#### [compress/flate](/pkg/compress/flate/)

<!-- CL 31640, CL 31174, CL 32149 -->
Đã có một số sửa chữa nhỏ cho encoder để cải thiện
tỷ lệ nén trong một số tình huống. Do đó, đầu ra mã hóa chính xác
của `DEFLATE` có thể khác với Go 1.7. Vì
`DEFLATE` là nén bên dưới của gzip, png, zlib và zip,
các định dạng đó có thể có đầu ra đã thay đổi.

<!-- CL 31174 -->
Encoder, khi hoạt động ở chế độ
[`NoCompression`](/pkg/compress/flate/#NoCompression),
giờ tạo ra đầu ra nhất quán không phụ thuộc vào
kích thước của các slice được truyền cho phương thức
[`Write`](/pkg/compress/flate/#Writer.Write).

<!-- CL 28216 -->
Bộ giải mã, khi gặp lỗi, giờ trả về bất kỳ
dữ liệu đã giải nén nào nó có cùng với lỗi.

#### [compress/gzip](/pkg/compress/gzip/)

[`Writer`](/pkg/compress/gzip/#Writer)
giờ mã hóa trường `MTIME` bằng không khi
trường [`Header.ModTime`](/pkg/compress/gzip/#Header)
là giá trị không.
Trong các bản phát hành Go trước, `Writer` sẽ mã hóa
một giá trị vô nghĩa.
Tương tự,
[`Reader`](/pkg/compress/gzip/#Reader)
giờ báo cáo trường `MTIME` được mã hóa bằng không là một
`Header.ModTime` bằng không.

#### [context](/pkg/context/)

<!-- CL 30370 -->
Lỗi [`DeadlineExceeded`](/pkg/context#DeadlineExceeded)
giờ triển khai
[`net.Error`](/pkg/net/#Error)
và báo cáo true cho cả hai phương thức `Timeout` và
`Temporary`.

#### [crypto/tls](/pkg/crypto/tls/)

<!-- CL 25159, CL 31318 -->
Phương thức mới
[`Conn.CloseWrite`](/pkg/crypto/tls/#Conn.CloseWrite)
cho phép các kết nối TLS đóng một nửa.

<!-- CL 28075 -->
Phương thức mới
[`Config.Clone`](/pkg/crypto/tls/#Config.Clone)
sao chép một cấu hình TLS.

<!-- CL 30790 -->
Callback mới [`Config.GetConfigForClient`](/pkg/crypto/tls/#Config.GetConfigForClient)
cho phép chọn cấu hình cho client một cách động, dựa trên
[`ClientHelloInfo`](/pkg/crypto/tls/#ClientHelloInfo) của client. <!-- CL 31391, CL 32119 -->
Struct [`ClientHelloInfo`](/pkg/crypto/tls/#ClientHelloInfo)
giờ có các trường mới `Conn`, `SignatureSchemes` (sử dụng kiểu mới
[`SignatureScheme`](/pkg/crypto/tls/#SignatureScheme)),
`SupportedProtos` và `SupportedVersions`.

<!-- CL 32115 -->
Callback mới [`Config.GetClientCertificate`](/pkg/crypto/tls/#Config.GetClientCertificate)
cho phép chọn chứng chỉ client dựa trên
thông điệp TLS `CertificateRequest` của server, được biểu diễn bởi
[`CertificateRequestInfo`](/pkg/crypto/tls/#CertificateRequestInfo) mới.

<!-- CL 27434 -->
[`Config.KeyLogWriter`](/pkg/crypto/tls/#Config.KeyLogWriter) mới
cho phép gỡ lỗi các kết nối TLS
trong [WireShark](https://www.wireshark.org/) và
các công cụ tương tự.

<!-- CL 32115 -->
Callback mới
[`Config.VerifyPeerCertificate`](/pkg/crypto/tls/#Config.VerifyPeerCertificate)
cho phép xác thực bổ sung của chứng chỉ được trình bày của peer.

<!-- CL 18130 -->
Package `crypto/tls` giờ triển khai các biện pháp đối phó cơ bản
chống lại các oracle đệm CBC. Không nên có các thời gian phụ thuộc bí mật tường minh, nhưng nó không cố gắng
chuẩn hóa các lần truy cập bộ nhớ để ngăn rò rỉ thời gian cache.

Package `crypto/tls` giờ hỗ trợ
X25519 và <!-- CL 30824, CL 30825 -->
ChaCha20-Poly1305. <!-- CL 30957, CL 30958 -->
ChaCha20-Poly1305 giờ được ưu tiên trừ khi <!-- CL 32871 -->
hỗ trợ phần cứng cho AES-GCM có mặt.

<!-- CL 27315, CL 35290 -->
Các bộ mã hóa AES-128-CBC với SHA-256 cũng
giờ được hỗ trợ, nhưng bị tắt theo mặc định.

#### [crypto/x509](/pkg/crypto/x509/)

<!-- CL 24743 -->
Các chữ ký PSS giờ được hỗ trợ.

<!-- CL 32644 -->
[`UnknownAuthorityError`](/pkg/crypto/x509/#UnknownAuthorityError)
giờ có trường `Cert`, báo cáo chứng chỉ
không đáng tin cậy.

Xác thực chứng chỉ có nhiều quyền hạn hơn trong một số trường hợp và
nghiêm ngặt hơn trong một số trường hợp khác. <!--
crypto/x509: allow a leaf certificate to be specified directly as root (CL 27393)
crypto/x509: check that the issuer name matches the issuer's subject name (CL 23571)
crypto/x509: don't accept a root that already appears in a chain. (CL 32121)
crypto/x509: fix name constraints handling (CL 30155)
crypto/x509: parse all names in an RDN (CL 30810)
crypto/x509: recognise ISO OID for RSA+SHA1 (CL 27394)
crypto/x509: require a NULL parameters for RSA public keys (CL 16166, CL 27312)
crypto/x509: return error for missing SerialNumber (CL 27238)
-->

<!-- CL 30375 -->
Chứng chỉ root giờ cũng được tìm kiếm tại
`/etc/pki/ca-trust/extracted/pem/tls-ca-bundle.pem`
trên Linux, để hỗ trợ RHEL và CentOS.

#### [database/sql](/pkg/database/sql/)

Package giờ hỗ trợ `context.Context`. Có các phương thức mới
kết thúc bằng `Context` như
[`DB.QueryContext`](/pkg/database/sql/#DB.QueryContext) và
[`DB.PrepareContext`](/pkg/database/sql/#DB.PrepareContext)
nhận các đối số context. Sử dụng các phương thức `Context` mới đảm bảo rằng
các kết nối được đóng và trả về pool kết nối khi
yêu cầu hoàn thành; cho phép hủy các truy vấn đang tiến hành
nếu driver hỗ trợ điều đó; và cho phép pool cơ sở dữ liệu
hủy việc chờ đợi kết nối tiếp theo có sẵn.

[`IsolationLevel`](/pkg/database/sql#IsolationLevel)
giờ có thể được đặt khi bắt đầu một giao dịch bằng cách đặt mức độ cô lập
trên [`TxOptions.Isolation`](/pkg/database/sql#TxOptions.Isolation) và truyền nó
cho [`DB.BeginTx`](/pkg/database/sql#DB.BeginTx).
Một lỗi sẽ được trả về nếu chọn một mức độ cô lập mà driver
không hỗ trợ. Thuộc tính chỉ đọc cũng có thể được đặt trên giao dịch
bằng cách đặt [`TxOptions.ReadOnly`](/pkg/database/sql/#TxOptions.ReadOnly)
thành true.

Các truy vấn giờ hiển thị thông tin kiểu cột SQL cho các driver hỗ trợ.
Rows có thể trả về [`ColumnTypes`](/pkg/database/sql#Rows.ColumnTypes)
có thể bao gồm thông tin kiểu SQL, độ dài kiểu cột và kiểu Go.

[`Rows`](/pkg/database/sql/#Rows)
giờ có thể biểu diễn nhiều tập kết quả. Sau khi
[`Rows.Next`](/pkg/database/sql/#Rows.Next) trả về false,
[`Rows.NextResultSet`](/pkg/database/sql/#Rows.NextResultSet)
có thể được gọi để chuyển sang tập kết quả tiếp theo. `Rows` hiện có
nên tiếp tục được sử dụng sau khi nó chuyển sang tập kết quả tiếp theo.

[`NamedArg`](/pkg/database/sql/#NamedArg) có thể được sử dụng
làm đối số truy vấn. Hàm mới [`Named`](/pkg/database/sql/#Named)
giúp tạo [`NamedArg`](/pkg/database/sql/#NamedArg)
ngắn gọn hơn.

Nếu một driver hỗ trợ interface
[`Pinger`](/pkg/database/sql/driver/#Pinger) mới,
các phương thức
[`DB.Ping`](/pkg/database/sql/#DB.Ping)
và
[`DB.PingContext`](/pkg/database/sql/#DB.PingContext)
sẽ sử dụng interface đó để kiểm tra xem kết nối cơ sở dữ liệu còn hợp lệ không.

Các phương thức truy vấn `Context` mới hoạt động cho tất cả driver, nhưng
việc hủy `Context` không phản hồi trừ khi driver đã được
cập nhật để sử dụng chúng. Các tính năng khác yêu cầu hỗ trợ driver trong
[`database/sql/driver`](/pkg/database/sql/driver).
Tác giả driver nên xem xét các interface mới. Người dùng driver hiện có
nên xem tài liệu driver để xem nó
hỗ trợ gì và tài liệu cụ thể cho hệ thống về mỗi tính năng.

#### [debug/pe](/pkg/debug/pe/)

<!-- CL 22720, CL 27212, CL 22181, CL 22332, CL 22336, Issue 15345 -->
Package đã được mở rộng và giờ được sử dụng bởi
[trình liên kết Go](/cmd/link/) để đọc các tệp đối tượng do `gcc` tạo ra.
[`File.StringTable`](/pkg/debug/pe/#File.StringTable) và
[`Section.Relocs`](/pkg/debug/pe/#Section.Relocs) mới
cung cấp quyền truy cập vào bảng chuỗi COFF và các relocation COFF.
[`File.COFFSymbols`](/pkg/debug/pe/#File.COFFSymbols) mới
cho phép truy cập cấp thấp vào bảng ký hiệu COFF.

#### [encoding/base64](/pkg/encoding/base64/)

<!-- CL 24964 -->
Phương thức [`Encoding.Strict`](/pkg/encoding/base64/#Encoding.Strict) mới
trả về một `Encoding` khiến bộ giải mã
trả về lỗi khi các bit đệm ở cuối không phải bằng không.

#### [encoding/binary](/pkg/encoding/binary/)

<!-- CL 28514 -->
[`Read`](/pkg/encoding/binary/#Read)
và
[`Write`](/pkg/encoding/binary/#Write)
giờ hỗ trợ boolean.

#### [encoding/json](/pkg/encoding/json/)

<!-- CL 18692  -->
[`UnmarshalTypeError`](/pkg/encoding/json/#UnmarshalTypeError)
giờ bao gồm tên struct và trường.

<!-- CL 31932 -->
Một [`Marshaler`](/pkg/encoding/json/#Marshaler) nil
giờ marshal thành giá trị JSON `null`.

<!-- CL 21811 -->
Một giá trị [`RawMessage`](/pkg/encoding/json/#RawMessage) giờ
marshal giống như kiểu con trỏ của nó.

<!-- CL 30371 -->
[`Marshal`](/pkg/encoding/json/#Marshal)
mã hóa các số dấu phẩy động sử dụng cùng định dạng như trong ES6,
ưu tiên ký hiệu thập phân (không phải mũ) cho phạm vi giá trị rộng hơn.
Đặc biệt, tất cả các số nguyên dấu phẩy động tới 2<sup>64</sup> định dạng
giống như biểu diễn `int64` tương đương.

<!-- CL 30944 -->
Trong các phiên bản Go trước, việc unmarshal JSON `null` vào một
[`Unmarshaler`](/pkg/encoding/json/#Unmarshaler)
được coi là no-op; giờ phương thức `UnmarshalJSON` của `Unmarshaler`
được gọi với literal JSON `null` và có thể định nghĩa ngữ nghĩa của trường hợp đó.

#### [encoding/pem](/pkg/encoding/pem/)

<!-- CL 27391 -->
[`Decode`](/pkg/encoding/pem/#Decode)
giờ nghiêm ngặt về định dạng của dòng kết thúc.

#### [encoding/xml](/pkg/encoding/xml/)

<!-- CL 30946 -->
[`Unmarshal`](/pkg/encoding/xml/#Unmarshal)
giờ có hỗ trợ wildcard để thu thập tất cả thuộc tính bằng cách sử dụng
tag struct `",any,attr"` mới.

#### [expvar](/pkg/expvar/)

<!-- CL 30917 -->
Các phương thức mới
[`Int.Value`](/pkg/expvar/#Int.Value),
[`String.Value`](/pkg/expvar/#String.Value),
[`Float.Value`](/pkg/expvar/#Float.Value) và
[`Func.Value`](/pkg/expvar/#Func.Value)
báo cáo giá trị hiện tại của một biến được xuất khẩu.

<!-- CL 24722 -->
Hàm mới [`Handler`](/pkg/expvar/#Handler)
trả về trình xử lý HTTP của package, để bật việc cài đặt nó ở
các vị trí không chuẩn.

#### [fmt](/pkg/fmt/)

<!-- CL 30611 -->
[`Scanf`](/pkg/fmt/#Scanf),
[`Fscanf`](/pkg/fmt/#Fscanf) và
[`Sscanf`](/pkg/fmt/#Sscanf) giờ
xử lý khoảng trắng khác nhau và nhất quán hơn so với
các bản phát hành trước. Xem
[tài liệu quét](/pkg/fmt/#hdr-Scanning)
để biết chi tiết.

#### [go/doc](/pkg/go/doc/)

<!-- CL 29870 -->
Hàm mới [`IsPredeclared`](/pkg/go/doc/#IsPredeclared)
báo cáo liệu một chuỗi có phải là định danh được định nghĩa sẵn hay không.

#### [go/types](/pkg/go/types/)

<!-- CL 30715 -->
Hàm mới
[`Default`](/pkg/go/types/#Default)
trả về kiểu "có kiểu" mặc định cho một kiểu "không có kiểu".

<!-- CL 31939 -->
Căn chỉnh của `complex64` giờ khớp với
[trình biên dịch Go](/cmd/compile/).

#### [html/template](/pkg/html/template/)

<!-- CL 14336 -->
Package giờ xác thực
thuộc tính `"type"` trên
tag `<script>`.

#### [image/png](/pkg/image/png/)

<!-- CL 32143, CL 32140 -->
[`Decode`](/pkg/image/png/#Decode)
(và `DecodeConfig`)
giờ hỗ trợ độ trong suốt True Color và grayscale.

<!-- CL 29872 -->
[`Encoder`](/pkg/image/png/#Encoder)
giờ nhanh hơn và tạo ra đầu ra nhỏ hơn
khi mã hóa hình ảnh được phân bảng màu.

#### [math/big](/pkg/math/big/)

<!-- CL 30706 -->
Phương thức mới
[`Int.Sqrt`](/pkg/math/big/#Int.Sqrt)
tính ⌊√x⌋.

Phương thức mới
[`Float.Scan`](/pkg/math/big/#Float.Scan)
là một routine hỗ trợ cho
[`fmt.Scanner`](/pkg/fmt/#Scanner).

[`Int.ModInverse`](/pkg/math/big/#Int.ModInverse)
giờ hỗ trợ các số âm.

#### [math/rand](/pkg/math/rand/)

<!-- CL 27253, CL 33456 -->
Phương thức mới [`Rand.Uint64`](/pkg/math/rand/#Rand.Uint64)
trả về các giá trị `uint64`. Interface
[`Source64`](/pkg/math/rand/#Source64) mới
mô tả các nguồn có khả năng tạo ra các giá trị đó
trực tiếp; nếu không, phương thức `Rand.Uint64`
xây dựng `uint64` từ hai lời gọi
đến phương thức `Int63` của [`Source`](/pkg/math/rand/#Source).

#### [mime](/pkg/mime/)

<!-- CL 32175 -->
[`ParseMediaType`](/pkg/mime/#ParseMediaType)
giờ bảo tồn các thoát backslash không cần thiết như các literal,
để hỗ trợ MSIE.
Khi MSIE gửi đường dẫn tệp đầy đủ (trong "chế độ intranet"), nó không
thoát backslash: "`C:\dev\go\foo.txt`", không phải
"`C:\\dev\\go\\foo.txt`".
Nếu chúng ta thấy một thoát backslash không cần thiết, chúng ta giờ giả định nó từ MSIE
và được dùng như backslash literal.
Không có generator MIME đã biết nào phát ra các thoát backslash không cần thiết
cho các ký tự token đơn giản như số và chữ cái.

#### [mime/quotedprintable](/pkg/mime/quotedprintable/)

Việc phân tích của
[`Reader`](/pkg/mime/quotedprintable/#Reader)
đã được nới lỏng theo hai cách để chấp nhận
nhiều đầu vào thấy trong thực tế hơn. <!-- CL 32174 -->
Đầu tiên, nó chấp nhận dấu bằng (`=`) không theo sau bởi
hai chữ số hex là một dấu bằng literal. <!-- CL 27530 -->
Thứ hai, nó im lặng bỏ qua dấu bằng ở cuối
của đầu vào được mã hóa.

#### [net](/pkg/net/)

<!-- CL 30164, CL 33473 -->
Tài liệu của [`Conn`](/pkg/net/#Conn)
đã được cập nhật để làm rõ kỳ vọng của việc triển khai interface.
Các cập nhật trong các package `net/http` phụ thuộc vào các triển khai tuân theo tài liệu.

_Cập nhật:_ các triển khai của interface `Conn` nên xác minh
chúng triển khai ngữ nghĩa đã ghi lại. Package
[golang.org/x/net/nettest](https://godoc.org/golang.org/x/net/nettest)
sẽ kiểm thử một `Conn` và xác thực nó hoạt động đúng cách.

<!-- CL 32099 -->
Phương thức mới
[`UnixListener.SetUnlinkOnClose`](/pkg/net/#UnixListener.SetUnlinkOnClose)
đặt xem tệp socket bên dưới có nên bị xóa khỏi hệ thống tệp khi
listener được đóng hay không.

<!-- CL 29951 -->
Kiểu mới [`Buffers`](/pkg/net/#Buffers) cho phép
ghi vào mạng hiệu quả hơn từ nhiều buffer không liền kề
trong bộ nhớ. Trên một số máy, đối với một số loại kết nối nhất định,
điều này được tối ưu hóa thành một hoạt động ghi batch dành riêng cho OS (như `writev`).

<!-- CL 29440 -->
[`Resolver`](/pkg/net/#Resolver) mới tìm kiếm tên và số
và hỗ trợ [`context.Context`](/pkg/context/#Context).
[`Dialer`](/pkg/net/#Dialer) giờ có trường
[`Resolver`](/pkg/net/#Dialer.Resolver) tùy chọn.

<!-- CL 29892 -->
[`Interfaces`](/pkg/net/#Interfaces) giờ được hỗ trợ trên Solaris.

<!-- CL 29233, CL 24901 -->
Bộ giải quyết DNS Go giờ hỗ trợ các tùy chọn "`rotate`"
và "`option` `ndots:0`" của `resolv.conf`. Tùy chọn "`ndots`" giờ
được tôn trọng theo cùng cách như `libresolve`.

#### [net/http](/pkg/net/http/)

Thay đổi server:

  - Server giờ hỗ trợ tắt nhẹ nhàng, [được đề cập ở trên](#http_shutdown).
  - <!-- CL 32024 -->
    [`Server`](/pkg/net/http/#Server)
    thêm các tùy chọn cấu hình
    `ReadHeaderTimeout` và `IdleTimeout`
    và ghi lại `WriteTimeout`.
  - <!-- CL 32014 -->
    [`FileServer`](/pkg/net/http/#FileServer)
    và
    [`ServeContent`](/pkg/net/http/#ServeContent)
    giờ hỗ trợ các yêu cầu điều kiện HTTP `If-Match`,
    ngoài hỗ trợ `If-None-Match` trước đây
    cho các ETag được định dạng đúng theo RFC 7232, phần 2.3.

Có một số bổ sung cho những gì `Handler` của server có thể làm:

  - <!-- CL 31173 -->
    [`Context`](/pkg/context/#Context) được trả về
    bởi [`Request.Context`](/pkg/net/http/#Request.Context)
    bị hủy nếu `net.Conn` bên dưới
    đóng. Ví dụ, nếu người dùng đóng trình duyệt của họ trong
    giữa một yêu cầu chậm, `Handler` giờ có thể
    phát hiện rằng người dùng đã đi. Điều này bổ sung cho
    hỗ trợ [`CloseNotifier`](/pkg/net/http/#CloseNotifier)
    hiện có. Chức năng này yêu cầu rằng
    [`net.Conn`](/pkg/net/#Conn) bên dưới triển khai
    [tài liệu interface được làm rõ gần đây](#net).
  - <!-- CL 32479 -->
    Để phục vụ các trailer được tạo ra sau khi header đã được ghi,
    xem cơ chế
    [`TrailerPrefix`](/pkg/net/http/#TrailerPrefix) mới.
  - <!-- CL 33099 -->
    `Handler` giờ có thể hủy bỏ một phản hồi bằng cách panic
    với lỗi
    [`ErrAbortHandler`](/pkg/net/http/#ErrAbortHandler).
  - <!-- CL 30812 -->
    `Write` bằng không byte đến một
    [`ResponseWriter`](/pkg/net/http/#ResponseWriter)
    giờ được định nghĩa là
    một cách để kiểm tra xem `ResponseWriter` có bị hijack hay không:
    nếu có, `Write` trả về
    [`ErrHijacked`](/pkg/net/http/#ErrHijacked)
    mà không in lỗi
    vào log lỗi của server.

Thay đổi Client & Transport:

  - <!-- CL 28930, CL 31435 -->
    [`Client`](/pkg/net/http/#Client)
    giờ sao chép hầu hết các header yêu cầu khi chuyển hướng. Xem
    [tài liệu](/pkg/net/http/#Client)
    về kiểu `Client` để biết chi tiết.
  - <!-- CL 29072 -->
    [`Transport`](/pkg/net/http/#Transport)
    giờ hỗ trợ tên miền quốc tế. Do đó,
    [Get](/pkg/net/http/#Get) và các helper khác cũng vậy.
  - <!-- CL 31733, CL 29852 -->
    `Client` giờ hỗ trợ chuyển hướng 301, 307 và 308.
    Ví dụ, `Client.Post` giờ theo dõi các chuyển hướng 301,
    chuyển đổi chúng thành các yêu cầu `GET` không có thân,
    giống như nó đã làm cho các phản hồi chuyển hướng 302 và 303
    trước đây.
    `Client` giờ cũng theo dõi các chuyển hướng 307 và 308,
    giữ nguyên phương thức yêu cầu gốc và thân, nếu có.
    Nếu việc chuyển hướng yêu cầu gửi lại thân yêu cầu, yêu cầu
    phải có trường
    [`Request.GetBody`](/pkg/net/http/#Request) mới được định nghĩa.
    [`NewRequest`](/pkg/net/http/#NewRequest)
    đặt `Request.GetBody` tự động cho
    các kiểu thân phổ biến.
  - <!-- CL 32482 -->
    `Transport` giờ từ chối các yêu cầu cho các URL với
    các port chứa ký tự không phải chữ số.
  - <!-- CL 27117 -->
    `Transport` giờ sẽ thử lại các yêu cầu không idempotent
    nếu không có byte nào được ghi trước khi mạng lỗi
    và yêu cầu không có thân.
  - <!-- CL 32481 -->
    [`Transport.ProxyConnectHeader`](/pkg/net/http/#Transport) mới
    cho phép cấu hình các giá trị header để gửi đến proxy
    trong một yêu cầu `CONNECT`.
  - <!-- CL 28077 -->
    [`DefaultTransport.Dialer`](/pkg/net/http/#DefaultTransport)
    giờ bật hỗ trợ `DualStack` ("[Happy Eyeballs](https://tools.ietf.org/html/rfc6555)"),
    cho phép sử dụng IPv4 như backup nếu có vẻ như IPv6 có thể
    thất bại.
  - <!-- CL 31726 -->
    [`Transport`](/pkg/net/http/#Transport)
    không còn đọc một byte của
    [`Request.Body`](/pkg/net/http/#Request.Body) không nil
    khi
    [`Request.ContentLength`](/pkg/net/http/#Request.ContentLength)
    bằng không để xác định xem `ContentLength`
    thực sự bằng không hay chỉ không xác định.
    Để tường minh báo hiệu rằng thân có độ dài bằng không,
    hoặc đặt nó thành `nil`, hoặc đặt nó thành giá trị mới
    [`NoBody`](/pkg/net/http/#NoBody).
    Giá trị `NoBody` mới được dùng bởi các hàm constructor `Request`;
    nó được sử dụng bởi
    [`NewRequest`](/pkg/net/http/#NewRequest).

#### [net/http/httptrace](/pkg/net/http/httptrace/)

<!-- CL 30359 -->
Giờ có hỗ trợ để theo dõi quá trình bắt tay TLS của yêu cầu client với
[`ClientTrace.TLSHandshakeStart`](/pkg/net/http/httptrace/#ClientTrace.TLSHandshakeStart) và
[`ClientTrace.TLSHandshakeDone`](/pkg/net/http/httptrace/#ClientTrace.TLSHandshakeDone) mới.

#### [net/http/httputil](/pkg/net/http/httputil/)

<!-- CL 32356 -->
[`ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy)
có hook tùy chọn mới,
[`ModifyResponse`](/pkg/net/http/httputil/#ReverseProxy.ModifyResponse),
để sửa đổi phản hồi từ backend trước khi proxy nó đến client.

#### [net/mail](/pkg/net/mail/)

<!-- CL 32176 -->
Các chuỗi được trích dẫn rỗng lại được cho phép trong phần tên của
địa chỉ. Nghĩa là, Go 1.4 và trước đây chấp nhận
`""` `<gopher@example.com>`,
nhưng Go 1.5 đã giới thiệu một lỗi từ chối địa chỉ này.
Địa chỉ lại được nhận dạng.

<!-- CL 31581 -->
Phương thức [`Header.Date`](/pkg/net/mail/#Header.Date)
luôn cung cấp một cách để phân tích
header `Date:`.
Hàm mới
[`ParseDate`](/pkg/net/mail/#ParseDate)
cho phép phân tích ngày được tìm thấy trong các
dòng header khác, như header `Resent-Date:`.

#### [net/smtp](/pkg/net/smtp/)

<!-- CL 33143 -->
Nếu việc triển khai phương thức
[`Auth.Start`](/pkg/net/smtp/#Auth)
trả về giá trị `toServer` rỗng,
package không còn gửi
khoảng trắng ở cuối trong lệnh SMTP `AUTH`,
mà một số server từ chối.

#### [net/url](/pkg/net/url/)

<!-- CL 31322 -->
Các hàm mới
[`PathEscape`](/pkg/net/url/#PathEscape)
và
[`PathUnescape`](/pkg/net/url/#PathUnescape)
tương tự như các hàm thoát và bỏ thoát truy vấn nhưng
cho các phần tử đường dẫn.

<!-- CL 28933 -->
Các phương thức mới
[`URL.Hostname`](/pkg/net/url/#URL.Hostname)
và
[`URL.Port`](/pkg/net/url/#URL.Port)
trả về các trường tên máy chủ và port của URL,
xử lý đúng trường hợp port có thể không có mặt.

<!-- CL 28343 -->
Phương thức hiện có
[`URL.ResolveReference`](/pkg/net/url/#URL.ResolveReference)
giờ xử lý đúng các đường dẫn với các byte được thoát mà không mất
ký tự thoát.

<!-- CL 31467 -->
Kiểu `URL` giờ triển khai
[`encoding.BinaryMarshaler`](/pkg/encoding/#BinaryMarshaler) và
[`encoding.BinaryUnmarshaler`](/pkg/encoding/#BinaryUnmarshaler),
cho phép xử lý URL trong [dữ liệu gob](/pkg/encoding/gob/).

<!-- CL 29610, CL 31582 -->
Theo RFC 3986,
[`Parse`](/pkg/net/url/#Parse)
giờ từ chối các URL như `this_that:other/thing` thay vì
diễn giải chúng là các đường dẫn tương đối (`this_that` không phải là lược đồ hợp lệ).
Để buộc diễn giải là đường dẫn tương đối,
các URL đó nên được tiền tố bằng "`./`".
Phương thức `URL.String` giờ chèn tiền tố này khi cần.

#### [os](/pkg/os/)

<!-- CL 16551 -->
Hàm mới
[`Executable`](/pkg/os/#Executable) trả về
tên đường dẫn của tệp thực thi đang chạy.

<!-- CL 30614 -->
Một nỗ lực gọi một phương thức trên
[`os.File`](/pkg/os/#File)
đã được đóng trước đây giờ sẽ trả về giá trị lỗi mới
[`os.ErrClosed`](/pkg/os/#ErrClosed).
Trước đây nó trả về một lỗi dành riêng cho hệ thống như
`syscall.EBADF`.

<!-- CL 31358 -->
Trên các hệ thống Unix, [`os.Rename`](/pkg/os/#Rename)
giờ sẽ trả về lỗi khi được sử dụng để đổi tên thư mục thành một
thư mục rỗng hiện có.
Trước đây nó sẽ thất bại khi đổi tên thành thư mục không rỗng
nhưng thành công khi đổi tên thành thư mục rỗng.
Điều này làm cho hành vi trên Unix tương ứng với các hệ thống khác.

<!-- CL 32451 -->
Trên Windows, các đường dẫn tuyệt đối dài giờ được chuyển đổi trong suốt thành
các đường dẫn có độ dài mở rộng (các đường dẫn bắt đầu bằng "`\\?\`").
Điều này cho phép package làm việc với các tệp có tên đường dẫn
dài hơn 260 ký tự.

<!-- CL 29753 -->
Trên Windows, [`os.IsExist`](/pkg/os/#IsExist)
giờ sẽ trả về `true` cho lỗi hệ thống
`ERROR_DIR_NOT_EMPTY`.
Điều này tương ứng với việc xử lý hiện có của lỗi Unix
`ENOTEMPTY`.

<!-- CL 32152 -->
Trên Plan 9, các tệp không được phục vụ bởi `#M` giờ sẽ
có [`ModeDevice`](/pkg/os/#ModeDevice) được đặt trong
giá trị được trả về
bởi [`FileInfo.Mode`](/pkg/os/#FileInfo).

#### [path/filepath](/pkg/path/filepath/)

Một số lỗi và trường hợp góc cạnh trên Windows đã được sửa:
[`Abs`](/pkg/path/filepath/#Abs) giờ gọi `Clean` như đã ghi lại,
[`Glob`](/pkg/path/filepath/#Glob) giờ khớp với
"`\\?\c:\*`",
[`EvalSymlinks`](/pkg/path/filepath/#EvalSymlinks) giờ
xử lý đúng "`C:.`", và
[`Clean`](/pkg/path/filepath/#Clean) giờ xử lý đúng
"`..`" đứng đầu trong đường dẫn.

#### [reflect](/pkg/reflect/)

<!-- CL 30088 -->
Hàm mới
[`Swapper`](/pkg/reflect/#Swapper) đã được
thêm vào để hỗ trợ [`sort.Slice`](#sortslice).

#### [strconv](/pkg/strconv/)

<!-- CL 31210 -->
Hàm [`Unquote`](/pkg/strconv/#Unquote)
giờ loại bỏ carriage return (`\r`) trong
các chuỗi raw được trích dẫn backquote, theo
[ngữ nghĩa ngôn ngữ Go](/ref/spec#String_literals).

#### [syscall](/pkg/syscall/)

<!-- CL 25050, CL 25022 -->
[`Getpagesize`](/pkg/syscall/#Getpagesize)
giờ trả về kích thước của hệ thống, thay vì giá trị hằng số.
Trước đây nó luôn trả về 4KB.

<!-- CL 31446 -->
Chữ ký
của [`Utimes`](/pkg/syscall/#Utimes) đã
thay đổi trên Solaris để khớp với tất cả các chữ ký của các hệ thống Unix khác.
Mã di động nên tiếp tục sử dụng
[`os.Chtimes`](/pkg/os/#Chtimes) thay thế.

<!-- CL 32319 -->
Trường `X__cmsg_data` đã bị xóa khỏi
[`Cmsghdr`](/pkg/syscall/#Cmsghdr).

#### [text/template](/pkg/text/template/)

<!-- CL 31462 -->
[`Template.Execute`](/pkg/text/template/#Template.Execute)
giờ có thể nhận một
[`reflect.Value`](/pkg/reflect/#Value) làm đối số dữ liệu của nó, và
các hàm [`FuncMap`](/pkg/text/template/#FuncMap)
cũng có thể chấp nhận và trả về `reflect.Value`.

#### [time](/pkg/time/)

<!-- CL 20118 -->
Hàm mới
[`Until`](/pkg/time/#Until) bổ sung cho
hàm `Since` tương tự.

<!-- CL 29338 -->
[`ParseDuration`](/pkg/time/#ParseDuration)
giờ chấp nhận các phần thập phân dài.

<!-- CL 33429 -->
[`Parse`](/pkg/time/#Parse)
giờ từ chối các ngày trước đầu của tháng, như ngày 0 tháng 6;
nó đã từ chối các ngày ngoài cuối tháng, như
ngày 31 tháng 6 và ngày 32 tháng 7.

<!-- CL 33029 -->
<!-- CL 34816 -->
Cơ sở dữ liệu `tzdata` đã được cập nhật lên phiên bản
2016j cho các hệ thống không có cơ sở dữ liệu múi giờ cục bộ.


#### [testing](/pkg/testing/)

<!-- CL 29970 -->
Phương thức mới
[`T.Name`](/pkg/testing/#T.Name)
(và `B.Name`) trả về tên của test hoặc benchmark hiện tại.

<!-- CL 32483 -->
Hàm mới
[`CoverMode`](/pkg/testing/#CoverMode)
báo cáo chế độ kiểm tra độ bao phủ.

<!-- CL 32615 -->
Các test và benchmark giờ được đánh dấu là thất bại nếu bộ phát hiện race
được bật và một data race xảy ra trong quá trình thực thi.
Trước đây, các trường hợp test riêng lẻ có vẻ như đã vượt qua,
và chỉ việc thực thi tổng thể của tệp nhị phân test mới thất bại.

<!-- CL 32455 -->
Chữ ký của hàm
[`MainStart`](/pkg/testing/#MainStart)
đã thay đổi, như được cho phép bởi tài liệu. Đây là chi tiết nội bộ và không phải là một phần của cam kết tương thích Go 1.
Nếu bạn không gọi `MainStart` trực tiếp nhưng thấy
các lỗi, điều đó có thể có nghĩa là bạn đặt
biến môi trường `GOROOT` thường rỗng và nó
không khớp với phiên bản của nhị phân lệnh `go` của bạn.

#### [unicode](/pkg/unicode/)

<!-- CL 30935 -->
[`SimpleFold`](/pkg/unicode/#SimpleFold)
giờ trả về đối số của nó không thay đổi nếu đầu vào được cung cấp là một rune không hợp lệ.
Trước đây, việc triển khai thất bại với panic kiểm tra giới hạn chỉ mục.
