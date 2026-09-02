---
path: /doc/go1.20
title: Ghi chú phát hành Go 1.20
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

## Giới thiệu Go 1.20 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.20, ra mắt sáu tháng sau [Go 1.19](/doc/go1.19).
Phần lớn thay đổi nằm ở phần triển khai của toolchain, runtime và thư viện.
Như mọi khi, bản phát hành duy trì [cam kết tương thích](/doc/go1compat) của Go 1.
Chúng tôi kỳ vọng hầu như tất cả chương trình Go vẫn tiếp tục biên dịch và chạy như trước.

## Thay đổi ngôn ngữ {#language}

Go 1.20 bao gồm bốn thay đổi ngôn ngữ.

<!-- https://go.dev/issue/46505 -->
Go 1.17 đã thêm [chuyển đổi từ slice sang con trỏ mảng](/ref/spec#Conversions_from_slice_to_array_or_array_pointer).
Go 1.20 mở rộng điều này để cho phép chuyển đổi từ slice sang mảng:
với một slice `x`, giờ có thể viết `[4]byte(x)`
thay vì `*(*[4]byte)(x)`.

<!-- https://go.dev/issue/53003 -->
Gói [`unsafe`](/ref/spec/#Package_unsafe) định nghĩa
ba hàm mới: `SliceData`, `String`, và `StringData`.
Cùng với `Slice` từ Go 1.17, các hàm này giờ cung cấp khả năng hoàn chỉnh để
xây dựng và phân tách giá trị slice và string mà không phụ thuộc vào biểu diễn nội bộ của chúng.

<!-- https://go.dev/issue/8606 -->
Đặc tả ngôn ngữ giờ định nghĩa rằng giá trị struct được so sánh từng trường một,
theo thứ tự các trường xuất hiện trong định nghĩa kiểu struct,
và dừng lại ở sự khác biệt đầu tiên.
Trước đây đặc tả có thể hiểu theo nghĩa rằng
tất cả các trường cần được so sánh kể cả sau sự khác biệt đầu tiên.
Tương tự, đặc tả giờ định nghĩa rằng giá trị mảng được so sánh
từng phần tử một, theo thứ tự chỉ số tăng dần.
Trong cả hai trường hợp, sự khác biệt này ảnh hưởng đến việc liệu một số phép so sánh nhất định có phải panic hay không.
Các chương trình hiện tại không thay đổi: cách diễn đạt mới trong đặc tả mô tả
những gì các triển khai đã luôn làm.

<!-- https://go.dev/issue/56548 -->
[Các kiểu comparable](/ref/spec#Comparison_operators) (như interface thông thường)
giờ có thể thỏa mãn ràng buộc `comparable`, ngay cả khi các đối số kiểu
không strictly comparable (phép so sánh có thể panic khi chạy).
Điều này cho phép khởi tạo một tham số kiểu bị ràng buộc bởi `comparable`
(ví dụ: tham số kiểu cho khóa map generic do người dùng định nghĩa) với một đối số kiểu không strictly comparable
như kiểu interface, hoặc kiểu tổng hợp chứa kiểu interface.

## Các nền tảng {#ports}

### Windows {#windows}

<!-- https://go.dev/issue/57003, https://go.dev/issue/57004 -->
Go 1.20 là bản phát hành cuối cùng chạy trên mọi phiên bản Windows 7, 8, Server 2008 và Server 2012.
Go 1.21 sẽ yêu cầu ít nhất Windows 10 hoặc Server 2016.

### Darwin và iOS {#darwin}

<!-- https://go.dev/issue/23011 -->
Go 1.20 là bản phát hành cuối cùng chạy trên macOS 10.13 High Sierra hoặc 10.14 Mojave.
Go 1.21 sẽ yêu cầu macOS 10.15 Catalina trở lên.

### FreeBSD/RISC-V {#freebsd-riscv}

<!-- https://go.dev/issue/53466 -->
Go 1.20 thêm hỗ trợ thử nghiệm cho FreeBSD trên RISC-V (`GOOS=freebsd`, `GOARCH=riscv64`).

## Công cụ {#tools}

### Lệnh go {#go-command}

<!-- CL 432535, https://go.dev/issue/47257 -->
Thư mục `$GOROOT/pkg` không còn lưu trữ
các gói thư viện chuẩn được biên dịch sẵn nữa:
`go` `install` không còn ghi chúng,
`go` build không còn kiểm tra chúng,
và bản phân phối Go không còn đóng gói chúng.
Thay vào đó, các gói trong thư viện chuẩn được biên dịch khi cần
và lưu vào build cache, giống như các gói ngoài `GOROOT`.
Thay đổi này giảm kích thước bản phân phối Go và cũng
tránh sai lệch C toolchain cho các gói sử dụng cgo.

<!-- CL 448357: cmd/go: print test2json start events -->
Triển khai của `go` `test` `-json`
đã được cải thiện để ổn định hơn.
Các chương trình chạy `go` `test` `-json`
không cần cập nhật gì.
Các chương trình gọi trực tiếp `go` `tool` `test2json`
giờ nên chạy test binary với `-v=test2json`
(ví dụ: `go` `test` `-v=test2json`
hoặc `./pkg.test` `-test.v=test2json`)
thay vì chỉ `-v`.

<!-- CL 448357: cmd/go: print test2json start events -->
Thay đổi liên quan đến `go` `test` `-json`
là việc thêm một sự kiện với `Action` được đặt thành `start`
ở đầu quá trình thực thi mỗi chương trình test.
Khi chạy nhiều test bằng lệnh `go`,
các sự kiện start này được đảm bảo phát ra theo cùng thứ tự với
các gói được đặt tên trên dòng lệnh.

<!-- https://go.dev/issue/45454, CL 421434 -->
Lệnh `go` giờ định nghĩa
các build tag theo tính năng kiến trúc, chẳng hạn `amd64.v2`,
để cho phép chọn tệp triển khai gói dựa trên sự có mặt
hoặc vắng mặt của một tính năng kiến trúc cụ thể.
Xem [`go` `help` `buildconstraint`](/cmd/go#hdr-Build_constraints) để biết chi tiết.

<!-- https://go.dev/issue/50332 -->
Các lệnh con `go` giờ chấp nhận
`-C` `<dir>` để chuyển thư mục làm việc sang \<dir>
trước khi thực hiện lệnh, có thể hữu ích cho các script cần
thực thi lệnh trong nhiều module khác nhau.

<!-- https://go.dev/issue/41696, CL 416094 -->
Các lệnh `go` `build` và `go` `test`
không còn chấp nhận cờ `-i` nữa,
vốn đã [bị deprecated từ Go 1.16](/issue/41696).

<!-- https://go.dev/issue/38687, CL 421440 -->
Lệnh `go` `generate` giờ chấp nhận
`-skip` `<pattern>` để bỏ qua các directive `//go:generate`
khớp với `<pattern>`.

<!-- https://go.dev/issue/41583 -->
Lệnh `go` `test` giờ chấp nhận
`-skip` `<pattern>` để bỏ qua các test, subtest hoặc example
khớp với `<pattern>`.

<!-- https://go.dev/issue/37015 -->
Khi main module nằm trong `GOPATH/src`,
`go` `install` không còn cài đặt thư viện cho
các gói không phải `main` vào `GOPATH/pkg`,
và `go` `list` không còn báo cáo trường `Target`
cho các gói đó nữa. (Trong chế độ module, các gói đã biên dịch chỉ được lưu trong
[build cache](https://pkg.go.dev/cmd/go#hdr-Build_and_test_caching),
nhưng [một lỗi](/issue/37015) đã khiến
các đích cài đặt `GOPATH` vẫn còn hiệu lực một cách không mong muốn.)

<!-- https://go.dev/issue/55022 -->
Các lệnh `go` `build`, `go` `install`,
và các lệnh liên quan đến build giờ hỗ trợ cờ `-pgo` bật
tối ưu hóa dựa trên hồ sơ thực thi, được mô tả chi tiết hơn trong phần
[Compiler](#compiler) bên dưới.
Cờ `-pgo` chỉ định đường dẫn tệp hồ sơ.
Khi dùng `-pgo=auto`, lệnh `go` sẽ tìm kiếm
tệp tên `default.pgo` trong thư mục của main package và
sử dụng nếu có.
Chế độ này hiện yêu cầu chỉ định một main package duy nhất trên
dòng lệnh, nhưng chúng tôi có kế hoạch bỏ hạn chế này trong bản phát hành tương lai.
Khi dùng `-pgo=off` sẽ tắt tối ưu hóa dựa trên hồ sơ thực thi.

<!-- https://go.dev/issue/51430 -->
Các lệnh `go` `build`, `go` `install`,
và các lệnh liên quan đến build giờ hỗ trợ cờ `-cover`
để biên dịch đích đã chỉ định cùng với đo đạc độ phủ code.
Điều này được mô tả chi tiết hơn trong phần
[Cover](#cover) bên dưới.

#### `go` `version` {#go-version}

<!-- https://go.dev/issue/48187 -->
Lệnh `go` `version` `-m`
giờ hỗ trợ đọc nhiều loại Go binary hơn, đặc biệt là Windows DLL
được xây dựng với `go` `build` `-buildmode=c-shared`
và Linux binary không có quyền thực thi.

### Cgo {#cgo}

<!-- CL 450739 -->
Lệnh `go` giờ mặc định tắt `cgo`
trên các hệ thống không có C toolchain.
Cụ thể hơn, khi biến môi trường `CGO_ENABLED` không được đặt,
biến môi trường `CC` không được đặt,
và trình biên dịch C mặc định (thường là `clang` hoặc `gcc`)
không tìm thấy trong PATH,
`CGO_ENABLED` mặc định là `0`.
Như mọi khi, bạn có thể ghi đè giá trị mặc định bằng cách đặt `CGO_ENABLED` tường minh.

Tác động quan trọng nhất của thay đổi mặc định này là khi Go được cài đặt
trên hệ thống không có trình biên dịch C, nó giờ sẽ sử dụng bản build Go thuần túy cho các gói
trong thư viện chuẩn có sử dụng cgo, thay vì dùng các gói đã phân phối sẵn
(đã bị xóa, như [đã lưu ý ở trên](#go-command))
hoặc cố gắng sử dụng cgo và thất bại.
Điều này giúp Go hoạt động tốt hơn trong một số môi trường container tối giản
cũng như trên macOS, nơi các gói phân phối sẵn đã không được dùng cho
các gói dựa trên cgo kể từ Go 1.16.

Các gói trong thư viện chuẩn sử dụng cgo là [`net`](/pkg/net/),
[`os/user`](/pkg/os/user/), và
[`plugin`](/pkg/plugin/).
Trên macOS, các gói `net` và `os/user` đã được viết lại để không dùng cgo:
code giống nhau giờ được dùng cho cả build có cgo và không có cgo, cũng như cross-compiled build.
Trên Windows, các gói `net` và `os/user` chưa bao giờ dùng cgo.
Trên các hệ thống khác, các build với cgo bị tắt sẽ sử dụng phiên bản Go thuần túy của các gói này.

Hệ quả là, trên macOS, nếu code Go sử dụng
gói `net` được build với `-buildmode=c-archive`,
việc liên kết archive kết quả vào chương trình C yêu cầu truyền `-lresolv` khi
liên kết code C.

Trên macOS, race detector đã được viết lại để không dùng cgo:
các chương trình bật race detector có thể được build và chạy mà không cần Xcode.
Trên Linux và các hệ thống Unix khác, cũng như Windows, cần có C toolchain của máy chủ
để sử dụng race detector.

### Cover {#cover}

<!-- CL 436236, CL 401236, CL 438503 -->
Go 1.20 hỗ trợ thu thập dữ liệu phủ code cho chương trình
(ứng dụng và test tích hợp), chứ không chỉ unit test.

Để thu thập dữ liệu phủ code cho một chương trình, hãy build nó với cờ `-cover` của `go`
`build`, sau đó chạy binary kết quả với biến môi trường `GOCOVERDIR` được đặt
thành thư mục đầu ra cho các hồ sơ phủ code.
Xem [trang landing 'coverage for integration tests'](/doc/build-cover) để biết cách bắt đầu.
Để biết chi tiết về thiết kế và triển khai, xem
[đề xuất](/issue/51430).

### Vet {#vet}

#### Phát hiện tốt hơn việc capture biến vòng lặp bởi hàm lồng nhau {#vet-loopclosure}

<!-- CL 447256, https://go.dev/issue/55972: extend the loopclosure analysis to parallel subtests -->
Công cụ `vet` giờ báo cáo các tham chiếu đến biến vòng lặp theo sau
lệnh gọi [`T.Parallel()`](/pkg/testing/#T.Parallel)
trong thân hàm subtest. Các tham chiếu như vậy có thể quan sát giá trị của
biến từ một lần lặp khác (thường gây ra các trường hợp test bị bỏ qua) hoặc trạng thái không hợp lệ do truy cập đồng thời không đồng bộ.

<!-- CL 452615 -->
Công cụ cũng phát hiện lỗi tham chiếu ở nhiều nơi hơn. Trước đây nó chỉ
xem xét câu lệnh cuối cùng của thân vòng lặp, nhưng giờ nó kiểm tra đệ quy
các câu lệnh cuối cùng trong các câu lệnh if, switch và select.

#### Chuẩn đoán mới cho định dạng thời gian sai {#vet-timeformat}

<!-- CL 354010, https://go.dev/issue/48801: check for time formats with 2006-02-01 -->
Công cụ vet giờ báo cáo việc sử dụng định dạng thời gian 2006-02-01 (yyyy-dd-mm)
với [`Time.Format`](/pkg/time/#Time.Format) và
[`time.Parse`](/pkg/time/#Parse).
Định dạng này không xuất hiện trong các chuẩn ngày phổ biến, nhưng thường
bị sử dụng nhầm khi cố dùng định dạng ngày ISO 8601
(yyyy-mm-dd).

## Runtime {#runtime}

<!-- CL 422634 -->
Một số cấu trúc dữ liệu nội bộ của bộ gom rác đã được tổ chức lại để
hiệu quả hơn cả về không gian lẫn CPU.
Thay đổi này giảm chi phí bộ nhớ và cải thiện hiệu năng CPU tổng thể lên đến
2%.

<!-- CL 417558, https://go.dev/issue/53892 -->
Bộ gom rác hoạt động ít thất thường hơn liên quan đến
goroutine assist trong một số trường hợp.

<!-- https://go.dev/issue/51430 -->
Go 1.20 thêm gói `runtime/coverage` mới
chứa các API để ghi dữ liệu hồ sơ phủ code khi
chạy từ các chương trình dài hạn và/hoặc máy chủ không kết thúc qua `os.Exit()`.

## Compiler {#compiler}

<!-- https://go.dev/issue/55022 -->
Go 1.20 thêm hỗ trợ xem trước cho tối ưu hóa dựa trên hồ sơ thực thi (PGO).
PGO cho phép toolchain thực hiện các tối ưu hóa đặc thù theo ứng dụng và khối lượng công việc
dựa trên thông tin hồ sơ khi chạy.
Hiện tại, compiler hỗ trợ hồ sơ CPU pprof, có thể thu thập
qua các cách thông thường, chẳng hạn gói `runtime/pprof` hoặc
`net/http/pprof`.
Để bật PGO, truyền đường dẫn tệp hồ sơ pprof qua
cờ `-pgo` cho `go` `build`,
như đã đề cập [ở trên](#go-command).
Go 1.20 sử dụng PGO để inline tích cực hơn các hàm tại các điểm gọi nóng.
Benchmark cho một tập đại diện các chương trình Go cho thấy bật
tối ưu hóa inline dựa trên hồ sơ cải thiện hiệu năng khoảng 3-4%.
Xem [hướng dẫn sử dụng PGO](/doc/pgo) để biết tài liệu chi tiết.
Chúng tôi có kế hoạch thêm nhiều tối ưu hóa dựa trên hồ sơ hơn trong các bản phát hành tương lai.
Lưu ý rằng tối ưu hóa dựa trên hồ sơ là tính năng xem trước, vì vậy hãy sử dụng
với sự thận trọng phù hợp.

Compiler Go 1.20 đã nâng cấp frontend để sử dụng cách xử lý dữ liệu nội bộ mới,
vốn sửa một số vấn đề liên quan đến kiểu generic và cho phép
khai báo kiểu trong các hàm và phương thức generic.

<!-- https://go.dev/issue/56103, CL 445598 -->
Compiler giờ [từ chối các vòng lặp interface ẩn danh](/issue/56103)
bằng lỗi compiler theo mặc định.
Chúng phát sinh từ các cách dùng phức tạp của [embedded interface](/ref/spec#Embedded_interfaces)
và luôn có vấn đề về tính đúng đắn tinh tế,
nhưng chúng tôi không có bằng chứng về việc chúng được dùng trong thực tế.
Giả sử không có báo cáo từ người dùng bị ảnh hưởng xấu bởi thay đổi này,
chúng tôi có kế hoạch cập nhật đặc tả ngôn ngữ cho Go 1.22 để cấm chúng một cách chính thức
để các tác giả công cụ cũng có thể dừng hỗ trợ chúng.

<!-- https://go.dev/issue/49569 -->
Go 1.18 và 1.19 có sự thoái lui về tốc độ build, chủ yếu do việc thêm
hỗ trợ generics và các công việc tiếp theo. Go 1.20 cải thiện tốc độ build lên
đến 10%, đưa nó trở lại ngang với Go 1.17.
So với Go 1.19, hiệu năng code được tạo ra cũng được cải thiện nhẹ nói chung.

## Linker {#linker}

<!-- https://go.dev/issue/54197, CL 420774 -->
Trên Linux, linker giờ chọn dynamic interpreter cho `glibc`
hoặc `musl` tại thời điểm link.

<!-- https://go.dev/issue/35006 -->
Trên Windows, Go linker giờ hỗ trợ các C toolchain dựa trên LLVM hiện đại.

<!-- https://go.dev/issue/37762, CL 317917 -->
Go 1.20 dùng tiền tố `go:` và `type:` cho các
ký hiệu được tạo bởi compiler thay vì `go.` và `type.`.
Điều này tránh nhầm lẫn cho các gói người dùng có tên bắt đầu bằng `go.`.
Gói [`debug/gosym`](/pkg/debug/gosym) hiểu
quy ước đặt tên mới này cho các binary được build với Go 1.20 trở lên.

## Bootstrap {#bootstrap}

<!-- https://go.dev/issue/44505 -->
Khi build bản phát hành Go từ source và `GOROOT_BOOTSTRAP` không được đặt,
các phiên bản Go trước tìm kiếm bootstrap toolchain Go 1.4 hoặc mới hơn trong thư mục
`$HOME/go1.4` (`%HOMEDRIVE%%HOMEPATH%\go1.4` trên Windows).
Go 1.18 và Go 1.19 tìm kiếm `$HOME/go1.17` hoặc `$HOME/sdk/go1.17` trước
trước khi dùng `$HOME/go1.4`,
để chuẩn bị cho việc yêu cầu Go 1.17 khi bootstrap Go 1.20.
Go 1.20 thực sự yêu cầu bản phát hành Go 1.17 để bootstrap, nhưng chúng tôi nhận ra nên
dùng điểm phát hành mới nhất của bootstrap toolchain, vì vậy nó yêu cầu Go 1.17.13.
Go 1.20 tìm kiếm `$HOME/go1.17.13` hoặc `$HOME/sdk/go1.17.13`
trước khi dùng `$HOME/go1.4`
(để hỗ trợ các hệ thống đã hard-code đường dẫn $HOME/go1.4 nhưng đã cài đặt
Go toolchain mới hơn ở đó).
Trong tương lai, chúng tôi có kế hoạch cập nhật bootstrap toolchain mỗi năm một lần,
và đặc biệt chúng tôi kỳ vọng Go 1.22 sẽ yêu cầu điểm phát hành cuối cùng của Go 1.20 để bootstrap.

## Thư viện chuẩn {#library}

### Gói crypto/ecdh mới {#crypto_ecdh}

<!-- https://go.dev/issue/52221, CL 398914, CL 450335, https://go.dev/issue/56052 -->
Go 1.20 thêm gói [`crypto/ecdh`](/pkg/crypto/ecdh/) mới
để cung cấp hỗ trợ tường minh cho trao đổi khóa Elliptic Curve Diffie-Hellman
qua các đường cong NIST và Curve25519.

Các chương trình nên dùng `crypto/ecdh` thay vì chức năng cấp thấp hơn trong
[`crypto/elliptic`](/pkg/crypto/elliptic/) cho ECDH, và
các module bên thứ ba cho các trường hợp sử dụng nâng cao hơn.

### Bọc nhiều lỗi {#errors}

<!-- CL 432898 -->
Go 1.20 mở rộng hỗ trợ bọc lỗi để cho phép một lỗi
bọc nhiều lỗi khác.

Một lỗi `e` có thể bọc nhiều hơn một lỗi bằng cách cung cấp
phương thức `Unwrap` trả về `[]error`.

Các hàm [`errors.Is`](/pkg/errors/#Is) và
[`errors.As`](/pkg/errors/#As)
đã được cập nhật để kiểm tra các lỗi được bọc nhiều lần.

Hàm [`fmt.Errorf`](/pkg/fmt/#Errorf)
giờ hỗ trợ nhiều lần xuất hiện của động từ định dạng `%w`,
sẽ khiến nó trả về một lỗi bọc tất cả các toán hạng lỗi đó.

Hàm mới [`errors.Join`](/pkg/errors/#Join)
trả về một lỗi bọc danh sách các lỗi.

### HTTP ResponseController {#http_responsecontroller}

<!-- CL 436890, https://go.dev/issue/54136 -->
Kiểu
[`"net/http".ResponseController`](/pkg/net/http/#ResponseController) mới
cung cấp quyền truy cập vào chức năng mở rộng theo từng yêu cầu không được xử lý bởi
interface [`"net/http".ResponseWriter`](/pkg/net/http/#ResponseWriter).

Trước đây, chúng tôi đã thêm chức năng theo từng yêu cầu mới bằng cách định nghĩa các
interface tùy chọn mà `ResponseWriter` có thể triển khai, chẳng hạn như
[`Flusher`](/pkg/net/http/#Flusher). Các interface này
không thể khám phá và cồng kềnh khi sử dụng.

Kiểu `ResponseController` cung cấp cách thêm điều khiển theo handler rõ ràng hơn, dễ khám phá hơn.
Hai điều khiển như vậy cũng được thêm trong Go 1.20 là
`SetReadDeadline` và `SetWriteDeadline`, cho phép đặt
deadline đọc và ghi theo từng yêu cầu. Ví dụ:

	func RequestHandler(w ResponseWriter, r *Request) {
	  rc := http.NewResponseController(w)
	  rc.SetWriteDeadline(time.Time{}) // vô hiệu hóa Server.WriteTimeout khi gửi response lớn
	  io.Copy(w, bigData)
	}

### Hook Rewrite mới cho ReverseProxy {#reverseproxy_rewrite}

<!-- https://go.dev/issue/53002, CL 407214 -->
Proxy chuyển tiếp [`httputil.ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy)
bao gồm hàm hook
[`Rewrite`](/pkg/net/http/httputil/#ReverseProxy.Rewrite) mới,
thay thế hook `Director` trước đây.

Hook `Rewrite` nhận tham số
[`ProxyRequest`](/pkg/net/http/httputil/#ProxyRequest),
bao gồm cả yêu cầu đến nhận bởi proxy và yêu cầu đi mà nó sẽ gửi.
Khác với hook `Director`, chỉ hoạt động trên yêu cầu đi,
điều này cho phép hook `Rewrite` tránh một số tình huống nhất định khi
yêu cầu đến độc hại có thể khiến header được thêm bởi hook
bị xóa trước khi chuyển tiếp.
Xem [issue #50580](/issue/50580).

Phương thức [`ProxyRequest.SetURL`](/pkg/net/http/httputil/#ProxyRequest.SetURL)
định tuyến yêu cầu đi đến đích đã cung cấp
và thay thế hàm `NewSingleHostReverseProxy`.
Khác với `NewSingleHostReverseProxy`, `SetURL`
cũng đặt header `Host` của yêu cầu đi.

<!-- https://go.dev/issue/50465, CL 407414 -->
Phương thức
[`ProxyRequest.SetXForwarded`](/pkg/net/http/httputil/#ProxyRequest.SetXForwarded)
đặt các header `X-Forwarded-For`, `X-Forwarded-Host`,
và `X-Forwarded-Proto` của yêu cầu đi.
Khi dùng `Rewrite`, các header này không được thêm theo mặc định.

Ví dụ về hook `Rewrite` sử dụng các tính năng này là:

	proxyHandler := &httputil.ReverseProxy{
	  Rewrite: func(r *httputil.ProxyRequest) {
	    r.SetURL(outboundURL) // Chuyển tiếp yêu cầu đến outboundURL.
	    r.SetXForwarded()     // Đặt các header X-Forwarded-*.
	    r.Out.Header.Set("X-Additional-Header", "header được đặt bởi proxy")
	  },
	}

<!-- CL 407375 -->
[`ReverseProxy`](/pkg/net/http/httputil/#ReverseProxy) không còn thêm header `User-Agent`
vào các yêu cầu được chuyển tiếp khi yêu cầu đến không có header đó.

### Các thay đổi nhỏ trong thư viện {#minor_library_changes}

Như mọi khi, có nhiều thay đổi và cập nhật nhỏ trong thư viện,
được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1
được ghi nhớ.
Ngoài ra cũng có nhiều cải thiện hiệu năng, không được liệt kê ở đây.

#### [archive/tar](/pkg/archive/tar/)

<!-- https://go.dev/issue/55356, CL 449937 -->
Khi biến môi trường `GODEBUG=tarinsecurepath=0` được đặt,
phương thức [`Reader.Next`](/pkg/archive/tar/#Reader.Next)
giờ sẽ trả về lỗi [`ErrInsecurePath`](/pkg/archive/tar/#ErrInsecurePath)
cho một mục có tên tệp là đường dẫn tuyệt đối,
tham chiếu đến vị trí ngoài thư mục hiện tại, chứa ký tự không hợp lệ,
hoặc (trên Windows) là tên dành riêng như `NUL`.
Phiên bản Go tương lai có thể tắt đường dẫn không an toàn theo mặc định.

<!-- archive/tar -->

#### [archive/zip](/pkg/archive/zip/)

<!-- https://go.dev/issue/55356 -->
Khi biến môi trường `GODEBUG=zipinsecurepath=0` được đặt,
[`NewReader`](/pkg/archive/zip/#NewReader) giờ sẽ trả về lỗi
[`ErrInsecurePath`](/pkg/archive/zip/#ErrInsecurePath)
khi mở một archive chứa bất kỳ tên tệp nào là đường dẫn tuyệt đối,
tham chiếu đến vị trí ngoài thư mục hiện tại, chứa ký tự không hợp lệ,
hoặc (trên Windows) là tên dành riêng như `NUL`.
Phiên bản Go tương lai có thể tắt đường dẫn không an toàn theo mặc định.

<!-- CL 449955 -->
Đọc từ tệp thư mục có chứa dữ liệu tệp giờ sẽ trả về lỗi.
Đặc tả zip không cho phép tệp thư mục chứa dữ liệu tệp,
vì vậy thay đổi này chỉ ảnh hưởng khi đọc từ các archive không hợp lệ.

<!-- archive/zip -->

#### [bytes](/pkg/bytes/)

<!-- CL 407176 -->
Các hàm mới
[`CutPrefix`](/pkg/bytes/#CutPrefix) và
[`CutSuffix`](/pkg/bytes/#CutSuffix)
giống như [`TrimPrefix`](/pkg/bytes/#TrimPrefix)
và [`TrimSuffix`](/pkg/bytes/#TrimSuffix)
nhưng cũng báo cáo liệu string có được cắt hay không.

<!-- CL 359675, https://go.dev/issue/45038 -->
Hàm mới [`Clone`](/pkg/bytes/#Clone)
cấp phát bản sao của một byte slice.

<!-- bytes -->

#### [context](/pkg/context/)

<!-- https://go.dev/issue/51365, CL 375977 -->
Hàm mới [`WithCancelCause`](/pkg/context/#WithCancelCause)
cung cấp cách hủy một context với lỗi đã cho.
Lỗi đó có thể lấy lại bằng cách gọi hàm mới [`Cause`](/pkg/context/#Cause).

<!-- context -->

#### [crypto/ecdsa](/pkg/crypto/ecdsa/)

<!-- CL 353849 -->
Khi sử dụng các đường cong được hỗ trợ, tất cả phép tính giờ được triển khai trong thời gian không đổi.
Điều này dẫn đến tăng thời gian CPU từ 5% đến 30%, chủ yếu ảnh hưởng đến P-384 và P-521.

<!-- https://go.dev/issue/56088, CL 450816 -->
Phương thức mới [`PrivateKey.ECDH`](/pkg/crypto/ecdsa/#PrivateKey.ECDH)
chuyển đổi `ecdsa.PrivateKey` sang `ecdh.PrivateKey`.

<!-- crypto/ecdsa -->

#### [crypto/ed25519](/pkg/crypto/ed25519/)

<!-- CL 373076, CL 404274, https://go.dev/issue/31804 -->
Phương thức [`PrivateKey.Sign`](/pkg/crypto/ed25519/#PrivateKey.Sign)
và hàm
[`VerifyWithOptions`](/pkg/crypto/ed25519/#VerifyWithOptions)
giờ hỗ trợ ký các thông điệp được băm trước với Ed25519ph,
được chỉ định bởi
[`Options.HashFunc`](/pkg/crypto/ed25519/#Options.HashFunc)
trả về
[`crypto.SHA512`](/pkg/crypto/#SHA512).
Chúng cũng hỗ trợ Ed25519ctx và Ed25519ph với context,
được chỉ định bằng cách đặt trường
[`Options.Context`](/pkg/crypto/ed25519/#Options.Context) mới.

<!-- crypto/ed25519 -->

#### [crypto/rsa](/pkg/crypto/rsa/)

<!-- CL 418874, https://go.dev/issue/19974 -->
Trường mới [`OAEPOptions.MGFHash`](/pkg/crypto/rsa/#OAEPOptions.MGFHash)
cho phép cấu hình MGF1 hash riêng biệt cho OAEP decryption.

<!-- https://go.dev/issue/20654 -->
crypto/rsa giờ sử dụng backend mới, an toàn hơn, với thời gian không đổi. Điều này gây ra tăng thời gian CPU
cho các phép giải mã từ khoảng 15%
(RSA-2048 trên amd64) đến 45% (RSA-4096 trên arm64), và nhiều hơn trên kiến trúc 32-bit.
Các phép mã hóa chậm hơn khoảng 20 lần so với trước (nhưng vẫn nhanh hơn 5-10 lần so với giải mã).
Hiệu năng dự kiến sẽ cải thiện trong các bản phát hành tương lai.
Các chương trình không được sửa đổi hoặc tạo thủ công các trường của
[`PrecomputedValues`](/pkg/crypto/rsa/#PrecomputedValues).

<!-- crypto/rsa -->

#### [crypto/subtle](/pkg/crypto/subtle/)

<!-- https://go.dev/issue/53021, CL 421435 -->
Hàm mới [`XORBytes`](/pkg/crypto/subtle/#XORBytes)
XOR hai byte slice với nhau.

<!-- crypto/subtle -->

#### [crypto/tls](/pkg/crypto/tls/)

<!-- CL 426455, CL 427155, CL 426454, https://go.dev/issue/46035 -->
Các chứng chỉ đã phân tích giờ được chia sẻ giữa tất cả các client đang tích cực sử dụng chứng chỉ đó.
Tiết kiệm bộ nhớ có thể đáng kể trong các chương trình thực hiện nhiều kết nối đồng thời đến một
máy chủ hoặc tập hợp máy chủ chia sẻ bất kỳ phần nào của chuỗi chứng chỉ của họ.

<!-- https://go.dev/issue/48152, CL 449336 -->
Khi handshake thất bại do lỗi xác minh chứng chỉ,
TLS client và server giờ trả về lỗi kiểu mới
[`CertificateVerificationError`](/pkg/crypto/tls/#CertificateVerificationError),
bao gồm các chứng chỉ được trình bày.

<!-- crypto/tls -->

#### [crypto/x509](/pkg/crypto/x509/)

<!-- CL 450816, CL 450815 -->
[`ParsePKCS8PrivateKey`](/pkg/crypto/x509/#ParsePKCS8PrivateKey)
và
[`MarshalPKCS8PrivateKey`](/pkg/crypto/x509/#MarshalPKCS8PrivateKey)
giờ hỗ trợ khóa kiểu [`*crypto/ecdh.PrivateKey`](/pkg/crypto/ecdh.PrivateKey).
[`ParsePKIXPublicKey`](/pkg/crypto/x509/#ParsePKIXPublicKey)
và
[`MarshalPKIXPublicKey`](/pkg/crypto/x509/#MarshalPKIXPublicKey)
giờ hỗ trợ khóa kiểu [`*crypto/ecdh.PublicKey`](/pkg/crypto/ecdh.PublicKey).
Phân tích khóa đường cong NIST vẫn trả về giá trị kiểu
`*ecdsa.PublicKey` và `*ecdsa.PrivateKey`.
Dùng các phương thức `ECDH` mới của chúng để chuyển đổi sang kiểu `crypto/ecdh`.

<!-- CL 449235 -->
Hàm mới [`SetFallbackRoots`](/pkg/crypto/x509/#SetFallbackRoots)
cho phép chương trình định nghĩa một tập chứng chỉ root dự phòng trong trường hợp
trình xác minh hệ điều hành hoặc gói root nền tảng chuẩn không có sẵn khi chạy.
Nó thường được dùng nhiều nhất với một gói mới, [golang.org/x/crypto/x509roots/fallback](/pkg/golang.org/x/crypto/x509roots/fallback),
sẽ cung cấp gói root được cập nhật.

<!-- crypto/x509 -->

#### [debug/elf](/pkg/debug/elf/)

<!-- CL 429601 -->
Các lần thử đọc từ phần `SHT_NOBITS` bằng cách dùng
[`Section.Data`](/pkg/debug/elf/#Section.Data)
hoặc reader được trả bởi [`Section.Open`](/pkg/debug/elf/#Section.Open)
giờ trả về lỗi.

<!-- CL 420982 -->
Các hằng số [`R_LARCH_*`](/pkg/debug/elf/#R_LARCH) bổ sung được định nghĩa để dùng với hệ thống LoongArch.

<!-- CL 420982, CL 435415, CL 425555 -->
Các hằng số [`R_PPC64_*`](/pkg/debug/elf/#R_PPC64) bổ sung được định nghĩa để dùng với các relocation PPC64 ELFv2.

<!-- CL 411915 -->
Giá trị hằng số cho [`R_PPC64_SECTOFF_LO_DS`](/pkg/debug/elf/#R_PPC64_SECTOFF_LO_DS) được sửa, từ 61 thành 62.

<!-- debug/elf -->

#### [debug/gosym](/pkg/debug/gosym/)

<!-- https://go.dev/issue/37762, CL 317917 -->
Do thay đổi [quy ước đặt tên ký hiệu của Go](#linker), các công cụ xử lý
Go binary nên dùng gói `debug/gosym` của Go 1.20 để
xử lý cả binary cũ và mới một cách trong suốt.

<!-- debug/gosym -->

#### [debug/pe](/pkg/debug/pe/)

<!-- CL 421357 -->
Các hằng số [`IMAGE_FILE_MACHINE_RISCV*`](/pkg/debug/pe/#IMAGE_FILE_MACHINE_RISCV128) bổ sung được định nghĩa để dùng với hệ thống RISC-V.

<!-- debug/pe -->

#### [encoding/binary](/pkg/encoding/binary/)

<!-- CL 420274 -->
Các hàm [`ReadVarint`](/pkg/encoding/binary/#ReadVarint) và
[`ReadUvarint`](/pkg/encoding/binary/#ReadUvarint)
giờ trả về `io.ErrUnexpectedEOF` sau khi đọc giá trị một phần,
thay vì `io.EOF`.

<!-- encoding/binary -->

#### [encoding/xml](/pkg/encoding/xml/)

<!-- https://go.dev/issue/53346, CL 424777 -->
Phương thức mới [`Encoder.Close`](/pkg/encoding/xml/#Encoder.Close)
có thể dùng để kiểm tra các phần tử chưa đóng khi kết thúc encoding.

<!-- CL 103875, CL 105636 -->
Decoder giờ từ chối các tên phần tử và thuộc tính có nhiều hơn một dấu hai chấm,
chẳng hạn `<a:b:c>`,
cũng như namespace phân giải thành chuỗi rỗng, chẳng hạn `xmlns:a=""`.

<!-- CL 107255 -->
Decoder giờ từ chối các phần tử sử dụng tiền tố namespace khác nhau trong thẻ mở và đóng,
ngay cả khi cả hai tiền tố đó đều ký hiệu cùng một namespace.

<!-- encoding/xml -->

#### [errors](/pkg/errors/)

<!-- https://go.dev/issue/53435 -->
Hàm mới [`Join`](/pkg/errors/#Join) trả về một lỗi bọc danh sách các lỗi.

<!-- errors -->

#### [fmt](/pkg/fmt/)

<!-- https://go.dev/issue/53435 -->
Hàm [`Errorf`](/pkg/fmt/#Errorf) hỗ trợ nhiều lần xuất hiện của
động từ định dạng `%w`, trả về một lỗi có thể unwrap thành danh sách tất cả đối số cho `%w`.

<!-- https://go.dev/issue/51668, CL 400875 -->
Hàm mới [`FormatString`](/pkg/fmt/#FormatString) khôi phục
directive định dạng tương ứng với [`State`](/pkg/fmt/#State),
có thể hữu ích trong các triển khai [`Formatter`](/pkg/fmt/#Formatter).

<!-- fmt -->

#### [go/ast](/pkg/go/ast/)

<!-- CL 426091, https://go.dev/issue/50429 -->
Trường mới [`RangeStmt.Range`](/pkg/go/ast/#RangeStmt.Range)
ghi lại vị trí của từ khóa `range` trong câu lệnh range.

<!-- CL 427955, https://go.dev/issue/53202 -->
Các trường mới [`File.FileStart`](/pkg/go/ast/#File.FileStart)
và [`File.FileEnd`](/pkg/go/ast/#File.FileEnd)
ghi lại vị trí bắt đầu và kết thúc của toàn bộ tệp source.

<!-- go/ast -->

#### [go/token](/pkg/go/token/)

<!-- CL 410114, https://go.dev/issue/53200 -->
Phương thức mới [`FileSet.RemoveFile`](/pkg/go/token/#FileSet.RemoveFile)
xóa một tệp khỏi `FileSet`.
Các chương trình chạy dài có thể dùng điều này để giải phóng bộ nhớ liên quan
đến các tệp không còn cần.

<!-- go/token -->

#### [go/types](/pkg/go/types/)

<!-- CL 454575 -->
Hàm mới [`Satisfies`](/pkg/go/types/#Satisfies) báo cáo
liệu một kiểu có thỏa mãn một ràng buộc hay không.
Thay đổi này phù hợp với [ngữ nghĩa ngôn ngữ mới](#language)
phân biệt việc thỏa mãn một ràng buộc với việc triển khai một interface.

<!-- go/types -->

#### [html/template](/pkg/html/template/)

<!-- https://go.dev/issue/59153 -->
<!-- CL 481993 -->
Go 1.20.3 trở lên
[không cho phép các action trong template literal ECMAScript 6.](/pkg/html/template#hdr-Security_Model)
Hành vi này có thể hoàn nguyên bằng cài đặt `GODEBUG=jstmpllitinterp=1`.

<!-- html/template -->

#### [io](/pkg/io/)

<!-- https://go.dev/issue/45899, CL 406776 -->
[`OffsetWriter`](/pkg/io/#OffsetWriter) mới bọc
[`WriterAt`](/pkg/io/#WriterAt) bên dưới
và cung cấp các phương thức `Seek`, `Write`, và `WriteAt`
điều chỉnh vị trí offset tệp hiệu quả của chúng bởi một lượng cố định.

<!-- io -->

#### [io/fs](/pkg/io/fs/)

<!-- CL 363814, https://go.dev/issue/47209 -->
Lỗi mới [`SkipAll`](/pkg/io/fs/#SkipAll)
kết thúc một [`WalkDir`](/pkg/io/fs/#WalkDir)
ngay lập tức nhưng thành công.

<!-- io -->

#### [math/big](/pkg/math/big/)

<!-- https://go.dev/issue/52182 -->
Phạm vi rộng và thời gian phụ thuộc đầu vào của gói [math/big](/pkg/math/big/) khiến nó không phù hợp để triển khai mật mã học.
Các gói mật mã học trong thư viện chuẩn không còn gọi các phương thức
[Int](/pkg/math/big#Int) không tầm thường trên các đầu vào do kẻ tấn công kiểm soát.
Trong tương lai, việc xác định xem một lỗi trong math/big có
được coi là lỗ hổng bảo mật hay không sẽ phụ thuộc vào tác động rộng hơn của nó đối với
thư viện chuẩn.

<!-- math/big -->

#### [math/rand](/pkg/math/rand/)

<!-- https://go.dev/issue/54880, CL 436955, https://go.dev/issue/56319 -->
Gói [math/rand](/pkg/math/rand/) giờ tự động seed
bộ tạo số ngẫu nhiên toàn cục
(được dùng bởi các hàm cấp cao nhất như `Float64` và `Int`) với một giá trị ngẫu nhiên,
và hàm [`Seed`](/pkg/math/rand/#Seed) cấp cao nhất đã bị deprecated.
Các chương trình cần chuỗi số ngẫu nhiên có thể tái tạo
nên ưu tiên cấp phát nguồn ngẫu nhiên riêng, dùng `rand.New(rand.NewSource(seed))`.

Các chương trình cần hành vi seed toàn cục nhất quán trước đó có thể đặt
`GODEBUG=randautoseed=0` trong môi trường của chúng.

<!-- https://go.dev/issue/20661 -->
Hàm [`Read`](/pkg/math/rand/#Read) cấp cao nhất đã bị deprecated.
Trong hầu hết các trường hợp, [`crypto/rand.Read`](/pkg/crypto/rand/#Read) phù hợp hơn.

<!-- math/rand -->

#### [mime](/pkg/mime/)

<!-- https://go.dev/issue/48866 -->
Hàm [`ParseMediaType`](/pkg/mime/#ParseMediaType) giờ cho phép tên tham số trùng lặp,
miễn là giá trị của các tên đó giống nhau.

<!-- mime -->

#### [mime/multipart](/pkg/mime/multipart/)

<!-- CL 431675 -->
Các phương thức của kiểu [`Reader`](/pkg/mime/multipart/#Reader) giờ bọc các lỗi
được trả bởi `io.Reader` bên dưới.

<!-- https://go.dev/issue/59153 -->
<!-- CL 481985 -->
Trong Go 1.19.8 trở lên, gói này giới hạn kích thước
dữ liệu MIME mà nó xử lý để bảo vệ trước các đầu vào độc hại.
`Reader.NextPart` và `Reader.NextRawPart` giới hạn
số header trong một phần ở 10000 và `Reader.ReadForm` giới hạn
tổng số header trong tất cả `FileHeaders` ở 10000.
Các giới hạn này có thể điều chỉnh với cài đặt `GODEBUG=multipartmaxheaders`.
`Reader.ReadForm` tiếp tục giới hạn số phần trong form ở 1000.
Giới hạn này có thể điều chỉnh với cài đặt `GODEBUG=multipartmaxparts`.

<!-- mime/multipart -->

#### [net](/pkg/net/)

<!-- https://go.dev/issue/50101, CL 446179 -->
Hàm [`LookupCNAME`](/pkg/net/#LookupCNAME)
giờ trả về nội dung của bản ghi `CNAME` một cách nhất quán
khi có một bản ghi. Trước đây trên hệ thống Unix và
khi dùng pure Go resolver, `LookupCNAME` sẽ trả về lỗi
nếu bản ghi `CNAME` tham chiếu đến tên không có bản ghi `A`,
`AAAA`, hoặc `CNAME`. Thay đổi này sửa
`LookupCNAME` để khớp với hành vi trước đó trên Windows,
cho phép `LookupCNAME` thành công bất cứ khi nào
`CNAME` tồn tại.

<!-- https://go.dev/issue/53482, CL 413454 -->
[`Interface.Flags`](/pkg/net/#Interface.Flags) giờ bao gồm cờ mới `FlagRunning`,
cho biết một interface đang hoạt động.
Một interface được cấu hình về mặt quản trị nhưng không hoạt động (ví dụ: cáp mạng chưa kết nối)
sẽ có `FlagUp` được đặt nhưng không có `FlagRunning`.

<!-- https://go.dev/issue/55301, CL 444955 -->
Trường mới [`Dialer.ControlContext`](/pkg/net/#Dialer.ControlContext) chứa hàm callback
tương tự hook [`Dialer.Control`](/pkg/net/#Dialer.Control) hiện có, nhưng bổ sung
chấp nhận context của dial làm tham số.
`Control` bị bỏ qua khi `ControlContext` khác nil.

<!-- CL 428955 -->
DNS resolver của Go nhận ra tùy chọn resolver `trust-ad`.
Khi `options trust-ad` được đặt trong `resolv.conf`,
Go resolver sẽ đặt bit AD trong các truy vấn DNS. Resolver không
sử dụng bit AD trong các phản hồi.

<!-- CL 448075 -->
Phân giải DNS sẽ phát hiện các thay đổi đối với `/etc/nsswitch.conf`
và tải lại tệp khi nó thay đổi. Các kiểm tra được thực hiện nhiều nhất mỗi
năm giây, khớp với cách xử lý trước đó của `/etc/hosts`
và `/etc/resolv.conf`.

<!-- net -->

#### [net/http](/pkg/net/http/)

<!-- https://go.dev/issue/51914 -->
Hàm [`ResponseWriter.WriteHeader`](/pkg/net/http/#ResponseWriter.WriteHeader) giờ hỗ trợ gửi
mã trạng thái `1xx`.

<!-- https://go.dev/issue/41773, CL 356410 -->
Cài đặt cấu hình mới [`Server.DisableGeneralOptionsHandler`](/pkg/net/http/#Server.DisableGeneralOptionsHandler)
cho phép tắt handler `OPTIONS *` mặc định.

<!-- https://go.dev/issue/54299, CL 447216 -->
Hook mới [`Transport.OnProxyConnectResponse`](/pkg/net/http/#Transport.OnProxyConnectResponse) được gọi
khi `Transport` nhận phản hồi HTTP từ proxy
cho yêu cầu `CONNECT`.

<!-- https://go.dev/issue/53960, CL 418614  -->
HTTP server giờ chấp nhận các yêu cầu HEAD có chứa body,
thay vì từ chối chúng như không hợp lệ.

<!-- https://go.dev/issue/53896 -->
Lỗi stream HTTP/2 được trả bởi các hàm `net/http` có thể được chuyển đổi
thành [`golang.org/x/net/http2.StreamError`](/pkg/golang.org/x/net/http2/#StreamError) bằng cách dùng
[`errors.As`](/pkg/errors/#As).

<!-- https://go.dev/cl/397734 -->
Các khoảng trắng ở đầu và cuối được cắt khỏi tên cookie,
thay vì bị từ chối như không hợp lệ.
Ví dụ: cài đặt cookie "name =value"
giờ được chấp nhận là đặt cookie "name".

<!-- https://go.dev/issue/52989 -->
Một [`Cookie`](/pkg/net/http#Cookie) có trường Expires rỗng giờ được coi là hợp lệ.
[`Cookie.Valid`](/pkg/net/http#Cookie.Valid) chỉ kiểm tra Expires khi nó được đặt.

<!-- net/http -->

#### [net/netip](/pkg/net/netip/)

<!-- https://go.dev/issue/51766, https://go.dev/issue/51777, CL 412475 -->
Các hàm mới [`IPv6LinkLocalAllRouters`](/pkg/net/netip/#IPv6LinkLocalAllRouters)
và [`IPv6Loopback`](/pkg/net/netip/#IPv6Loopback)
là các tương đương `net/netip` của
[`net.IPv6loopback`](/pkg/net/#IPv6loopback) và
[`net.IPv6linklocalallrouters`](/pkg/net/#IPv6linklocalallrouters).

<!-- net/netip -->

#### [os](/pkg/os/)

<!-- CL 448897 -->
Trên Windows, tên `NUL` không còn được xử lý như trường hợp đặc biệt trong
[`Mkdir`](/pkg/os/#Mkdir) và
[`Stat`](/pkg/os/#Stat).

<!-- https://go.dev/issue/52747, CL 405275 -->
Trên Windows, [`File.Stat`](/pkg/os/#File.Stat)
giờ dùng file handle để lấy các thuộc tính khi tệp là thư mục.
Trước đây nó sẽ dùng đường dẫn được truyền cho
[`Open`](/pkg/os/#Open), vốn có thể không còn là tệp
được đại diện bởi file handle nếu tệp đã bị di chuyển hoặc thay thế.
Thay đổi này sửa `Open` để mở thư mục mà không có
quyền truy cập `FILE_SHARE_DELETE`, khớp với hành vi của các tệp thông thường.

<!-- https://go.dev/issue/36019, CL 405275 -->
Trên Windows, [`File.Seek`](/pkg/os/#File.Seek) giờ hỗ trợ
seek đến đầu thư mục.

<!-- os -->

#### [os/exec](/pkg/os/exec/)

<!-- https://go.dev/issue/50436, CL 401835 -->
Các trường mới [`Cmd`](/pkg/os/exec/#Cmd)
[`Cancel`](/pkg/os/exec/#Cmd.Cancel) và
[`WaitDelay`](/pkg/os/exec/#Cmd.WaitDelay)
chỉ định hành vi của `Cmd` khi `Context` liên kết của nó bị hủy hoặc quá trình của nó thoát với các I/O pipe vẫn
được giữ mở bởi một tiến trình con.

<!-- os/exec -->

#### [path/filepath](/pkg/path/filepath/)

<!-- CL 363814, https://go.dev/issue/47209 -->
Lỗi mới [`SkipAll`](/pkg/path/filepath/#SkipAll)
kết thúc một [`Walk`](/pkg/path/filepath/#Walk)
ngay lập tức nhưng thành công.

<!-- https://go.dev/issue/56219, CL 449239 -->
Hàm mới [`IsLocal`](/pkg/path/filepath/#IsLocal) báo cáo liệu một đường dẫn có
cục bộ về mặt từ vựng với một thư mục hay không.
Ví dụ: nếu `IsLocal(p)` là `true`,
thì `Open(p)` sẽ tham chiếu đến một tệp nằm về mặt từ vựng
trong cây con có gốc tại thư mục hiện tại.

<!-- io -->

#### [reflect](/pkg/reflect/)

<!-- https://go.dev/issue/46746, CL 423794 -->
Các phương thức mới [`Value.Comparable`](/pkg/reflect/#Value.Comparable) và
[`Value.Equal`](/pkg/reflect/#Value.Equal)
có thể dùng để so sánh hai `Value` về tính bằng nhau.
`Comparable` báo cáo liệu `Equal` có phải là phép tính hợp lệ cho một receiver `Value` đã cho hay không.

<!-- https://go.dev/issue/48000, CL 389635 -->
Phương thức mới [`Value.Grow`](/pkg/reflect/#Value.Grow)
mở rộng một slice để đảm bảo chỗ cho thêm `n` phần tử.

<!-- https://go.dev/issue/52376, CL 411476 -->
Phương thức mới [`Value.SetZero`](/pkg/reflect/#Value.SetZero)
đặt một giá trị thành giá trị zero cho kiểu của nó.

<!-- CL 425184 -->
Go 1.18 đã giới thiệu các phương thức [`Value.SetIterKey`](/pkg/reflect/#Value.SetIterKey)
và [`Value.SetIterValue`](/pkg/reflect/#Value.SetIterValue).
Đây là các tối ưu hóa: `v.SetIterKey(it)` được thiết kế tương đương với `v.Set(it.Key())`.
Các triển khai không đúng cách đã bỏ qua kiểm tra việc sử dụng các trường không được export có trong các dạng không tối ưu.
Go 1.20 sửa các phương thức này để bao gồm kiểm tra trường không được export.

<!-- reflect -->

#### [regexp](/pkg/regexp/)

<!-- CL 444817 -->
Go 1.19.2 và Go 1.18.7 bao gồm bản sửa lỗi bảo mật cho parser biểu thức chính quy,
làm nó từ chối các biểu thức rất lớn tiêu thụ quá nhiều bộ nhớ.
Vì các bản Go patch không giới thiệu API mới,
parser trả về [`syntax.ErrInternalError`](/pkg/regexp/syntax/#ErrInternalError) trong trường hợp này.
Go 1.20 thêm lỗi cụ thể hơn, [`syntax.ErrLarge`](/pkg/regexp/syntax/#ErrLarge),
mà parser giờ trả về thay thế.

<!-- regexp -->

#### [runtime/cgo](/pkg/runtime/cgo/)

<!-- https://go.dev/issue/46731, CL 421879 -->
Go 1.20 thêm kiểu marker [`Incomplete`](/pkg/runtime/cgo/#Incomplete) mới.
Code được tạo bởi cgo sẽ dùng `cgo.Incomplete` để đánh dấu kiểu C không đầy đủ.

<!-- runtime/cgo -->

#### [runtime/metrics](/pkg/runtime/metrics/)

<!-- https://go.dev/issue/47216, https://go.dev/issue/49881 -->
Go 1.20 thêm các [metric được hỗ trợ](/pkg/runtime/metrics/#hdr-Supported_metrics) mới,
bao gồm cài đặt `GOMAXPROCS` hiện tại (`/sched/gomaxprocs:threads`),
số lời gọi cgo được thực thi (`/cgo/go-to-c-calls:calls`),
tổng thời gian chặn mutex (`/sync/mutex/wait/total:seconds`), và nhiều số liệu đo thời gian
dành cho bộ gom rác.

<!-- CL 427615 -->
Các metric histogram dựa trên thời gian giờ kém chính xác hơn, nhưng chiếm ít bộ nhớ hơn nhiều.

<!-- runtime/metrics -->

#### [runtime/pprof](/pkg/runtime/pprof/)

<!-- CL 443056 -->
Các mẫu mutex profile giờ được tính trước, sửa vấn đề khi các mẫu mutex profile cũ
sẽ bị tính sai nếu tỷ lệ lấy mẫu thay đổi trong quá trình thực thi.

<!-- CL 416975 -->
Các profile được thu thập trên Windows giờ bao gồm thông tin ánh xạ bộ nhớ để sửa
các vấn đề symbolization cho các binary position-independent.

<!-- runtime/pprof -->

#### [runtime/trace](/pkg/runtime/trace/)

<!-- CL 447135, https://go.dev/issue/55022 -->
Background sweeper của bộ gom rác giờ yield ít thường xuyên hơn,
dẫn đến ít sự kiện thừa hơn nhiều trong các execution trace.

<!-- runtime/trace -->

#### [strings](/pkg/strings/)

<!-- CL 407176, https://go.dev/issue/42537 -->
Các hàm mới
[`CutPrefix`](/pkg/strings/#CutPrefix) và
[`CutSuffix`](/pkg/strings/#CutSuffix)
giống như [`TrimPrefix`](/pkg/strings/#TrimPrefix)
và [`TrimSuffix`](/pkg/strings/#TrimSuffix)
nhưng cũng báo cáo liệu string có được cắt hay không.

<!-- strings -->

#### [sync](/pkg/sync/)

<!-- CL 399094, https://go.dev/issue/51972 -->
Các phương thức [`Map`](/pkg/sync/#Map) mới [`Swap`](/pkg/sync/#Map.Swap),
[`CompareAndSwap`](/pkg/sync/#Map.CompareAndSwap), và
[`CompareAndDelete`](/pkg/sync/#Map.CompareAndDelete)
cho phép cập nhật các mục map hiện có một cách nguyên tử.

<!-- sync -->

#### [syscall](/pkg/syscall/)

<!-- CL 411596 -->
Trên FreeBSD, các shim tương thích cần cho FreeBSD 11 và cũ hơn đã bị xóa.

<!-- CL 407574 -->
Trên Linux, các hằng số [`CLONE_*`](/pkg/syscall/#CLONE_CLEAR_SIGHAND) bổ sung
được định nghĩa để dùng với trường [`SysProcAttr.Cloneflags`](/pkg/syscall/#SysProcAttr.Cloneflags).

<!-- CL 417695 -->
Trên Linux, các trường [`SysProcAttr.CgroupFD`](/pkg/syscall/#SysProcAttr.CgroupFD) mới
và [`SysProcAttr.UseCgroupFD`](/pkg/syscall/#SysProcAttr.UseCgroupFD)
cung cấp cách đặt tiến trình con vào một cgroup cụ thể.

<!-- syscall -->

#### [testing](/pkg/testing/)

<!-- https://go.dev/issue/43620, CL 420254 -->
Phương thức mới [`B.Elapsed`](/pkg/testing/#B.Elapsed)
báo cáo thời gian đã trôi qua hiện tại của benchmark, có thể hữu ích để
tính tốc độ để báo cáo với `ReportMetric`.

<!-- https://go.dev/issue/48515, CL 352349 -->
Gọi [`T.Run`](/pkg/testing/#T.Run)
từ một hàm được truyền cho [`T.Cleanup`](/pkg/testing/#T.Cleanup)
chưa bao giờ được định nghĩa rõ ràng, và giờ sẽ panic.

<!-- testing -->

#### [time](/pkg/time/)

<!-- https://go.dev/issue/52746, CL 412495 -->
Các hằng số layout thời gian mới [`DateTime`](/pkg/time/#DateTime),
[`DateOnly`](/pkg/time/#DateOnly), và
[`TimeOnly`](/pkg/time/#TimeOnly)
cung cấp tên cho ba trong số các chuỗi layout phổ biến nhất được dùng trong cuộc khảo sát code Go công khai.

<!-- CL 382734, https://go.dev/issue/50770 -->
Phương thức mới [`Time.Compare`](/pkg/time/#Time.Compare)
so sánh hai thời điểm.

<!-- CL 425037 -->
[`Parse`](/pkg/time/#Parse)
giờ bỏ qua độ chính xác dưới nano giây trong đầu vào của nó,
thay vì báo cáo các chữ số đó như lỗi.

<!-- CL 444277 -->
Phương thức [`Time.MarshalJSON`](/pkg/time/#Time.MarshalJSON)
giờ nghiêm ngặt hơn về việc tuân thủ RFC 3339.

<!-- time -->

#### [unicode/utf16](/pkg/unicode/utf16/)

<!-- https://go.dev/issue/51896, CL 409054 -->
Hàm mới [`AppendRune`](/pkg/unicode/utf16/#AppendRune)
nối encoding UTF-16 của một rune đã cho vào slice uint16,
tương tự như [`utf8.AppendRune`](/pkg/unicode/utf8/#AppendRune).

<!-- unicode/utf16 -->

<!-- Silence false positives from x/build/cmd/relnote: -->
<!-- https://go.dev/issue/45964 was documented in Go 1.18 release notes but closed recently -->
<!-- https://go.dev/issue/52114 is an accepted proposal to add golang.org/x/net/http2.Transport.DialTLSContext; it's not a part of the Go release -->
<!-- CL 431335: cmd/api: make check pickier about api/*.txt -->
<!-- CL 447896 api: add newline to 55301.txt; modified api/next/55301.txt -->
<!-- CL 449215 api/next/54299: add missing newline; modified api/next/54299.txt -->
<!-- CL 433057 cmd: update vendored golang.org/x/tools for multiple error wrapping -->
<!-- CL 423362 crypto/internal/boring: update to newer boringcrypto, add arm64 -->
<!-- https://go.dev/issue/53481 x/cryptobyte ReadUint64, AddUint64 -->
<!-- https://go.dev/issue/51994 x/crypto/ssh -->
<!-- https://go.dev/issue/55358 x/exp/slices -->
<!-- https://go.dev/issue/54714 x/sys/unix -->
<!-- https://go.dev/issue/50035 https://go.dev/issue/54237 x/time/rate -->
<!-- CL 345488 strconv optimization -->
<!-- CL 428757 reflect deprecation, rolled back -->
<!-- https://go.dev/issue/49390 compile -l -N is fully supported -->
<!-- https://go.dev/issue/54619 x/tools -->
<!-- CL 448898 reverted -->
<!-- https://go.dev/issue/54850 x/net/http2 Transport.MaxReadFrameSize -->
<!-- https://go.dev/issue/56054 x/net/http2 SETTINGS_HEADER_TABLE_SIZE -->
<!-- CL 450375 reverted -->
<!-- CL 453259 tracking deprecations in api -->
<!-- CL 453260 tracking darwin port in api -->
<!-- CL 453615 fix deprecation comment in archive/tar -->
<!-- CL 453616 fix deprecation comment in archive/zip -->
<!-- CL 453617 fix deprecation comment in encoding/csv -->
<!-- https://go.dev/issue/54661 x/tools/go/analysis -->
<!-- CL 423359, https://go.dev/issue/51317 arena -->
