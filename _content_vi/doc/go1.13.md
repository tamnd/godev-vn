---
title: Ghi chú phát hành Go 1.13
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

## Giới thiệu về Go 1.13 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.13, ra đời sáu tháng sau [Go 1.12](go1.12).
Hầu hết các thay đổi nằm ở phần triển khai của bộ công cụ, runtime và thư viện.
Như thường lệ, bản phát hành vẫn duy trì [cam kết tương thích](/doc/go1compat.html) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

Kể từ Go 1.13, lệnh go mặc định tải xuống và xác thực các module bằng mirror module Go và cơ sở dữ liệu checksum Go do Google vận hành. Xem
<https://proxy.golang.org/privacy>
để biết thông tin về quyền riêng tư của các dịch vụ này và [tài liệu lệnh go](/cmd/go/#hdr-Module_downloading_and_verification)
để biết chi tiết cấu hình, bao gồm cách tắt việc sử dụng các máy chủ này hoặc dùng máy chủ khác. Nếu bạn phụ thuộc vào các module không công khai, hãy xem [tài liệu cấu hình môi trường](/cmd/go/#hdr-Module_configuration_for_non_public_modules).

## Thay đổi ngôn ngữ {#language}

Theo [đề xuất về literal số](https://github.com/golang/proposal/blob/master/design/19308-number-literals.md),
Go 1.13 hỗ trợ tập hợp tiền tố literal số đồng nhất và hiện đại hơn.

  - [Literal số nguyên nhị phân](/ref/spec#Integer_literals):
    Tiền tố `0b` hoặc `0B` biểu thị literal số nguyên nhị phân
    như `0b1011`.
  - [Literal số nguyên bát phân](/ref/spec#Integer_literals):
    Tiền tố `0o` hoặc `0O` biểu thị literal số nguyên bát phân
    như `0o660`.
    Ký hiệu bát phân cũ dùng số `0` dẫn đầu tiếp theo là các chữ số bát phân vẫn hợp lệ.
  - [Literal số thực thập lục phân](/ref/spec#Floating-point_literals):
    Tiền tố `0x` hoặc `0X` giờ có thể dùng để biểu thị phần định trị của số thực ở dạng thập lục phân như `0x1.0p-1021`.
    Số thực thập lục phân phải luôn có số mũ, viết bằng chữ `p` hoặc `P` theo sau là số mũ dạng thập phân. Số mũ chia tỉ lệ phần định trị theo lũy thừa 2.
  - [Literal số phức](/ref/spec#Imaginary_literals):
    Hậu tố phần ảo `i` giờ có thể dùng với bất kỳ literal (nhị phân, thập phân, thập lục phân) số nguyên hoặc số thực nào.
  - Dấu phân cách chữ số:
    Các chữ số trong bất kỳ literal số nào đều có thể tách nhau (nhóm lại) bằng dấu gạch dưới, ví dụ
    `1_000_000`, `0b_1010_0110`, hoặc `3.1415_9265`.
    Dấu gạch dưới có thể xuất hiện giữa hai chữ số bất kỳ hoặc giữa tiền tố literal và chữ số đầu tiên.


Theo [đề xuất về đếm dịch có dấu](https://github.com/golang/proposal/blob/master/design/19113-signed-shift-counts.md),
Go 1.13 bỏ hạn chế yêu cầu [số đếm dịch](/ref/spec#Operators) phải là số không dấu. Thay đổi này loại bỏ nhu cầu dùng nhiều phép chuyển đổi `uint` giả tạo, vốn chỉ được thêm vào để thỏa mãn hạn chế (đã bị xóa) này của toán tử `<<` và `>>`.

Các thay đổi ngôn ngữ này được thực hiện thông qua thay đổi trong trình biên dịch, cùng với các thay đổi nội bộ trong các gói thư viện [`go/scanner`](#go/scanner) và
[`text/scanner`](#text/scanner) (literal số),
cũng như [`go/types`](#go/types) (số đếm dịch có dấu).

Nếu mã của bạn sử dụng module và tệp `go.mod` chỉ định phiên bản ngôn ngữ, hãy đảm bảo đặt ít nhất là `1.13` để có được các thay đổi ngôn ngữ này.
Bạn có thể thực hiện bằng cách chỉnh sửa tệp `go.mod` trực tiếp, hoặc chạy
`go mod edit -go=1.13`.

## Các nền tảng {#ports}

Go 1.13 là bản phát hành cuối cùng chạy trên Native Client (NaCl).

<!-- CL 170119, CL 168882 -->
Với `GOARCH=wasm`, biến môi trường mới `GOWASM` nhận danh sách phân cách bằng dấu phẩy các tính năng thực nghiệm mà tệp nhị phân được biên dịch cùng.
Các giá trị hợp lệ được ghi lại [tại đây](/cmd/go/#hdr-Environment_variables).

### AIX {#aix}

<!-- CL 164003, CL 169120 -->
AIX trên PPC64 (`aix/ppc64`) giờ hỗ trợ cgo, liên kết ngoài, và các chế độ build `c-archive` và `pie`.

### Android {#android}

<!-- CL 170127 -->
Các chương trình Go hiện tương thích với Android 10.

### Darwin {#darwin}

Như đã [thông báo](go1.12#darwin) trong ghi chú phát hành Go 1.12,
Go 1.13 giờ yêu cầu macOS 10.11 El Capitan hoặc mới hơn; hỗ trợ cho các phiên bản cũ hơn đã bị ngừng.

### FreeBSD {#freebsd}

Như đã [thông báo](go1.12#freebsd) trong ghi chú phát hành Go 1.12,
Go 1.13 giờ yêu cầu FreeBSD 11.2 hoặc mới hơn; hỗ trợ cho các phiên bản cũ hơn đã bị ngừng.
FreeBSD 12.0 hoặc mới hơn yêu cầu kernel có tùy chọn `COMPAT_FREEBSD11` được bật (đây là mặc định).

### Illumos {#illumos}

<!-- CL 174457 -->
Go giờ hỗ trợ Illumos với `GOOS=illumos`.
Build tag `illumos` bao hàm build tag `solaris`.

### Windows {#windows}

<!-- CL 178977 -->
Phiên bản Windows được chỉ định bởi các tệp nhị phân Windows liên kết nội bộ giờ là Windows 7 thay vì NT 4.0. Đây đã là phiên bản tối thiểu bắt buộc cho Go, nhưng có thể ảnh hưởng đến hành vi của các lệnh gọi hệ thống có chế độ tương thích ngược. Các lệnh này giờ sẽ hoạt động đúng như tài liệu mô tả. Các tệp nhị phân liên kết ngoài (bất kỳ chương trình nào dùng cgo) luôn chỉ định phiên bản Windows mới hơn.

## Công cụ {#tools}

### Modules {#modules}

#### Biến môi trường {#proxy-vars}

<!-- CL 176580 -->
Biến môi trường [`GO111MODULE`](/cmd/go/#hdr-Module_support)
tiếp tục mặc định là `auto`, nhưng cài đặt `auto` giờ kích hoạt chế độ module-aware của lệnh `go` bất cứ khi nào thư mục làm việc hiện tại chứa, hoặc nằm dưới một thư mục chứa, tệp `go.mod`, kể cả khi thư mục hiện tại nằm trong `GOPATH/src`. Thay đổi này đơn giản hóa việc di chuyển mã hiện có trong `GOPATH/src` và bảo trì liên tục các gói nhận biết module bên cạnh các importer không nhận biết module.

<!-- CL 181719 -->
Biến môi trường mới
[`GOPRIVATE`](/cmd/go/#hdr-Module_configuration_for_non_public_modules)
chỉ định các đường dẫn module không có sẵn công khai.
Nó đóng vai trò là giá trị mặc định cho các biến cấp thấp hơn `GONOPROXY`
và `GONOSUMDB`, cung cấp kiểm soát chi tiết hơn về các module nào được tải qua proxy và xác minh bằng cơ sở dữ liệu checksum.

<!-- CL 173441, CL 177958 -->
Biến môi trường [`GOPROXY`](/cmd/go/#hdr-Module_downloading_and_verification) giờ có thể được đặt thành danh sách URL proxy phân cách bằng dấu phẩy hoặc token đặc biệt `direct`, và [giá trị mặc định của nó](#introduction) giờ là `https://proxy.golang.org,direct`. Khi phân giải đường dẫn gói thành module chứa nó, lệnh `go` sẽ thử lần lượt tất cả các đường dẫn module ứng viên trên từng proxy trong danh sách. Proxy không thể tiếp cận hoặc mã HTTP khác 404 hoặc 410 sẽ kết thúc tìm kiếm mà không tham chiếu đến các proxy còn lại.

Biến môi trường mới
[`GOSUMDB`](/cmd/go/#hdr-Module_authentication_failures)
xác định tên, và tùy chọn khóa công khai và URL máy chủ, của cơ sở dữ liệu để tham chiếu checksum của các module chưa được liệt kê trong tệp `go.sum` của module chính.
Nếu `GOSUMDB` không bao gồm URL tường minh, URL được chọn bằng cách thăm dò các URL `GOPROXY` để tìm endpoint hỗ trợ cơ sở dữ liệu checksum, rồi truy cập trực tiếp đến cơ sở dữ liệu được đặt tên nếu không có proxy nào hỗ trợ. Nếu `GOSUMDB` được đặt thành `off`, cơ sở dữ liệu checksum không được tham chiếu và chỉ các checksum hiện có trong tệp `go.sum` được xác minh.

Người dùng không thể tiếp cận proxy mặc định và cơ sở dữ liệu checksum (ví dụ do cấu hình firewall hoặc sandbox) có thể tắt chúng bằng cách đặt `GOPROXY` thành `direct` và/hoặc `GOSUMDB` thành `off`.
[`go` `env` `-w`](#go-env-w)
có thể được dùng để đặt giá trị mặc định cho các biến này độc lập với nền tảng:

	go env -w GOPROXY=direct
	go env -w GOSUMDB=off

#### `go` `get` {#go-get}

<!-- CL 174099 -->
Trong chế độ module-aware,
[`go` `get`](/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them)
với cờ `-u` giờ cập nhật tập hợp module nhỏ hơn, nhất quán hơn với tập hợp gói được cập nhật bởi `go` `get` `-u` trong chế độ GOPATH.
`go` `get` `-u` tiếp tục cập nhật các module và gói được đặt tên trên dòng lệnh, nhưng ngoài ra chỉ cập nhật các module chứa các gói được _nhập bởi_ các gói được đặt tên, thay vì toàn bộ dependency module bắc cầu của các module chứa các gói được đặt tên.

Lưu ý rằng `go` `get` `-u` (không có tham số thêm) giờ chỉ cập nhật các import bắc cầu của gói trong thư mục hiện tại. Để thay vào đó cập nhật tất cả các gói được nhập bắc cầu bởi module chính (bao gồm cả dependency kiểm thử), dùng `go` `get` `-u` `all`.

<!-- CL 177879 -->
Kết quả của các thay đổi trên đối với `go` `get` `-u`, lệnh con `go` `get` không còn hỗ trợ cờ `-m`, vốn khiến `go` `get` dừng trước khi tải gói. Cờ `-d` vẫn được hỗ trợ, và tiếp tục khiến `go` `get` dừng sau khi tải mã nguồn cần thiết để build dependency của các gói được đặt tên.

<!-- CL 177677 -->
Mặc định, `go` `get` `-u` trong chế độ module chỉ nâng cấp các dependency không phải test, giống như trong chế độ GOPATH. Nó giờ cũng chấp nhận cờ `-t`, vốn (như trong chế độ GOPATH) khiến `go` `get` bao gồm các gói được nhập bởi _các test của_ các gói được đặt tên trên dòng lệnh.

<!-- CL 167747 -->
Trong chế độ module-aware, lệnh con `go` `get` giờ hỗ trợ hậu tố phiên bản `@patch`. Hậu tố `@patch` chỉ định rằng module được đặt tên, hoặc module chứa gói được đặt tên, phải được cập nhật lên phiên bản patch cao nhất có cùng major và minor với phiên bản tìm thấy trong build list.

<!-- CL 184440 -->
Nếu module được truyền làm tham số cho `go` `get` mà không có hậu tố phiên bản đã được yêu cầu ở phiên bản mới hơn phiên bản đã phát hành mới nhất, nó sẽ giữ nguyên phiên bản mới hơn. Điều này nhất quán với hành vi của cờ `-u` đối với module dependency. Điều này ngăn ngừa việc hạ cấp bất ngờ từ các pre-release. Hậu tố phiên bản mới `@upgrade` yêu cầu hành vi này một cách tường minh. `@latest` yêu cầu tường minh phiên bản mới nhất bất kể phiên bản hiện tại.

#### Xác thực phiên bản {#version-validation}

<!-- CL 181881 -->

Khi trích xuất module từ hệ thống kiểm soát phiên bản, lệnh `go` giờ thực hiện xác thực bổ sung trên chuỗi phiên bản được yêu cầu.

Chú thích phiên bản `+incompatible` bỏ qua yêu cầu [nhập phiên bản ngữ nghĩa](/cmd/go/#hdr-Module_compatibility_and_semantic_versioning) đối với các kho lưu trữ xuất hiện trước khi module được giới thiệu. Lệnh `go` giờ xác minh rằng phiên bản đó không bao gồm tệp `go.mod` tường minh.

Lệnh `go` giờ xác minh ánh xạ giữa [pseudo-version](/cmd/go/#hdr-Pseudo_versions) và metadata kiểm soát phiên bản. Cụ thể:

  - Tiền tố phiên bản phải có dạng `vX.0.0`, hoặc được dẫn xuất từ tag trên tổ tiên của revision được đặt tên, hoặc được dẫn xuất từ tag bao gồm [build metadata](https://semver.org/#spec-item-10) trên revision được đặt tên.
  - Chuỗi ngày phải khớp với timestamp UTC của revision.
  - Tên ngắn của revision phải dùng cùng số ký tự như những gì lệnh `go` sẽ tạo ra. (Với SHA-1 hash như dùng bởi `git`, tiền tố 12 chữ số.)


Nếu directive `require` trong [module chính](/cmd/go/#hdr-The_main_module_and_the_build_list) dùng pseudo-version không hợp lệ, thường có thể sửa bằng cách rút gọn phiên bản về chỉ còn commit hash và chạy lại lệnh `go`, ví dụ `go` `list` `-m` `all`
hoặc `go` `mod` `tidy`. Ví dụ,

	require github.com/docker/docker v1.14.0-0.20190319215453-e7b5f7dbe98c

có thể rút gọn thành

	require github.com/docker/docker e7b5f7dbe98c

hiện phân giải thành

	require github.com/docker/docker v0.7.3-0.20190319215453-e7b5f7dbe98c

Nếu một trong các dependency bắc cầu của module chính yêu cầu phiên bản hoặc pseudo-version không hợp lệ, phiên bản không hợp lệ có thể được thay thế bằng phiên bản hợp lệ bằng cách dùng [directive `replace`](/cmd/go/#hdr-The_go_mod_file) trong tệp `go.mod` của module chính. Nếu phần thay thế là commit hash, nó sẽ được phân giải thành pseudo-version phù hợp như trên. Ví dụ,

	replace github.com/docker/docker v1.14.0-0.20190319215453-e7b5f7dbe98c => github.com/docker/docker e7b5f7dbe98c

hiện phân giải thành

	replace github.com/docker/docker v1.14.0-0.20190319215453-e7b5f7dbe98c => github.com/docker/docker v0.7.3-0.20190319215453-e7b5f7dbe98c

### Lệnh Go {#go-command}

<!-- CL 171137 -->
Lệnh [`go` `env`](/cmd/go/#hdr-Environment_variables)
giờ chấp nhận cờ `-w` để đặt giá trị mặc định cho mỗi người dùng của một biến môi trường được lệnh `go` nhận diện, và cờ `-u` tương ứng để bỏ đặt giá trị mặc định đã đặt trước đó. Giá trị mặc định được đặt qua `go` `env` `-w` được lưu trong tệp `go/env` trong [`os.UserConfigDir()`](/pkg/os/#UserConfigDir).

<!-- CL 173343 -->
Lệnh [`go` `version`](/cmd/go/#hdr-Print_Go_version) giờ chấp nhận các tham số là tên file thực thi và thư mục. Khi gọi trên file thực thi, `go` `version` in ra phiên bản Go dùng để build file thực thi đó. Nếu dùng cờ `-m`, `go` `version` in ra thông tin phiên bản module nhúng của file thực thi, nếu có. Khi gọi trên thư mục, `go` `version` in thông tin về các file thực thi trong thư mục đó và các thư mục con của nó.

<!-- CL 173345 -->
Cờ [`go` `build`](/cmd/go/#hdr-Compile_packages_and_dependencies) mới `-trimpath` xóa tất cả đường dẫn hệ thống tệp khỏi file thực thi đã biên dịch, để cải thiện tính tái lập của build.

<!-- CL 167679 -->
Nếu cờ `-o` được truyền cho `go` `build` trỏ đến thư mục đã tồn tại, `go` `build` giờ sẽ ghi các file thực thi vào thư mục đó cho các gói `main` khớp với tham số gói của nó.

<!-- CL 173438 -->
Cờ `go` `build` `-tags` giờ nhận danh sách build tag phân cách bằng dấu phẩy, để cho phép nhiều tag trong [`GOFLAGS`](/cmd/go/#hdr-Environment_variables). Dạng phân cách bằng dấu cách đã bị deprecated nhưng vẫn được nhận diện và sẽ được duy trì.

<!-- CL 175983 -->
[`go` `generate`](/cmd/go/#hdr-Generate_Go_files_by_processing_source) giờ đặt build tag `generate` để các tệp có thể được tìm kiếm directive nhưng bị bỏ qua trong quá trình build.

<!-- CL 165746 -->
Như đã [thông báo](/doc/go1.12#binary-only) trong ghi chú phát hành Go 1.12, binary-only package không còn được hỗ trợ. Build một binary-only package (được đánh dấu bằng comment `//go:binary-only-package`) giờ sẽ gây ra lỗi.

### Bộ công cụ trình biên dịch {#compiler}

<!-- CL 170448 -->
Trình biên dịch có triển khai phân tích thoát mới chính xác hơn. Đối với hầu hết mã Go, đây là cải tiến (nói cách khác, nhiều biến và biểu thức Go hơn được cấp phát trên stack thay vì heap). Tuy nhiên, độ chính xác tăng lên này cũng có thể phá vỡ mã không hợp lệ vốn hoạt động được trước đây (ví dụ: mã vi phạm [quy tắc an toàn `unsafe.Pointer`](/pkg/unsafe/#Pointer)). Nếu bạn nhận thấy bất kỳ hồi quy nào có vẻ liên quan, có thể bật lại phân tích thoát cũ bằng `go` `build` `-gcflags=all=-newescape=false`. Tùy chọn dùng phân tích thoát cũ sẽ bị xóa trong bản phát hành tương lai.

<!-- CL 161904 -->
Trình biên dịch không còn phát ra các hằng số thực hoặc phức vào tệp `go_asm.h`. Chúng luôn được phát ra ở dạng không thể dùng làm hằng số số trong mã assembly.

### Assembler {#assembler}

<!-- CL 157001 -->
Assembler giờ hỗ trợ nhiều lệnh atomic được giới thiệu trong ARM v8.1.

### gofmt {#gofmt}

`gofmt` (và cùng với đó là `go fmt`) giờ chuẩn hóa tiền tố literal số và số mũ để dùng chữ thường, nhưng để nguyên các chữ số thập lục phân. Điều này cải thiện khả năng đọc khi dùng tiền tố bát phân mới (`0O` thành `0o`), và phép viết lại được áp dụng nhất quán. `gofmt` giờ cũng xóa các số 0 đứng đầu không cần thiết từ literal số nguyên thập phân phần ảo. (Để tương thích ngược, literal số nguyên phần ảo bắt đầu bằng `0` được coi là thập phân, không phải bát phân. Xóa các số 0 đứng đầu thừa tránh gây nhầm lẫn tiềm ẩn.)
Ví dụ, `0B1010`, `0XabcDEF`, `0O660`,
`1.2E3` và `01i` thành `0b1010`, `0xabcDEF`,
`0o660`, `1.2e3` và `1i` sau khi áp dụng `gofmt`.

### `godoc` và `go` `doc` {#godoc}

<!-- CL 174322 -->
Máy chủ web `godoc` không còn được bao gồm trong bản phân phối nhị phân chính. Để chạy máy chủ web `godoc` cục bộ, hãy cài đặt thủ công trước:

	go get golang.org/x/tools/cmd/godoc
	godoc


<!-- CL 177797 -->
Lệnh [`go` `doc`](/cmd/go/#hdr-Show_documentation_for_package_or_symbol)
giờ luôn bao gồm mệnh đề package trong đầu ra của nó, ngoại trừ các lệnh. Điều này thay thế hành vi trước đây nơi một heuristic được dùng, khiến mệnh đề package bị bỏ qua trong một số điều kiện nhất định.

## Runtime {#runtime}

<!-- CL 161477 -->
Thông báo panic do truy cập ngoài phạm vi giờ bao gồm chỉ số ngoài phạm vi và độ dài (hoặc dung lượng) của slice. Ví dụ, `s[3]` trên slice có độ dài 1 sẽ panic với "runtime error: index out of range [3] with length 1".

<!-- CL 171758 -->
Bản phát hành này cải thiện hiệu năng của hầu hết các lần dùng `defer` lên 30%.

<!-- CL 142960 -->
Runtime giờ tích cực hơn trong việc trả bộ nhớ về hệ điều hành để cung cấp cho các ứng dụng đồng thuê. Trước đây, runtime có thể giữ bộ nhớ trong năm phút hoặc hơn sau khi heap tăng đột biến. Giờ nó sẽ bắt đầu trả lại ngay sau khi heap thu nhỏ. Tuy nhiên, trên nhiều hệ điều hành, bao gồm Linux, chính hệ điều hành thu hồi bộ nhớ một cách lười biếng, vì vậy RSS của tiến trình sẽ không giảm cho đến khi hệ thống chịu áp lực bộ nhớ.

## Thư viện chuẩn {#library}

### TLS 1.3 {#tls_1_3}

Như đã thông báo trong Go 1.12, Go 1.13 bật hỗ trợ TLS 1.3 trong gói `crypto/tls` theo mặc định. Có thể tắt bằng cách thêm giá trị `tls13=0` vào biến môi trường `GODEBUG`. Tùy chọn opt-out sẽ bị xóa trong Go 1.14.

Xem [ghi chú phát hành Go 1.12](/doc/go1.12#tls_1_3) để biết thông tin tương thích quan trọng.

### [crypto/ed25519](/pkg/crypto/ed25519/) {#crypto_ed25519}

<!-- CL 174945, 182698 -->
Gói mới [`crypto/ed25519`](/pkg/crypto/ed25519/)
triển khai sơ đồ chữ ký Ed25519. Chức năng này trước đây được cung cấp bởi gói
[`golang.org/x/crypto/ed25519`](https://godoc.org/golang.org/x/crypto/ed25519),
gói này trở thành wrapper cho `crypto/ed25519` khi dùng với Go 1.13+.

### Bao bọc lỗi {#error_wrapping}

<!-- CL 163558, 176998 -->
Go 1.13 hỗ trợ bao bọc lỗi, như đã đề xuất lần đầu trong [đề xuất Error Values](https://go.googlesource.com/proposal/+/master/design/29934-error-values.md) và được thảo luận trên [issue liên quan](/issue/29934).

Một lỗi `e` có thể _bao bọc_ lỗi khác `w` bằng cách cung cấp phương thức `Unwrap` trả về `w`. Cả `e` và `w` đều có sẵn cho chương trình, cho phép `e` cung cấp thêm ngữ cảnh cho `w` hoặc diễn giải lại nó trong khi vẫn cho phép chương trình đưa ra quyết định dựa trên `w`.

Để hỗ trợ bao bọc, [`fmt.Errorf`](#fmt) giờ có verb `%w` để tạo lỗi được bao bọc, và ba hàm mới trong gói [`errors`](#errors) (
[`errors.Unwrap`](/pkg/errors/#Unwrap),
[`errors.Is`](/pkg/errors/#Is) và
[`errors.As`](/pkg/errors/#As)) đơn giản hóa việc mở bao bọc và kiểm tra lỗi được bao bọc.

Để biết thêm thông tin, đọc [tài liệu gói `errors`](/pkg/errors/), hoặc xem [Error Value FAQ](/wiki/ErrorValueFAQ). Sắp tới sẽ có một bài blog về chủ đề này.

### Thay đổi nhỏ trong thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ trong thư viện, được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1 trong tâm trí.

#### [bytes](/pkg/bytes/)

Hàm mới [`ToValidUTF8`](/pkg/bytes/#ToValidUTF8) trả về bản sao của slice byte cho trước với mỗi chuỗi byte UTF-8 không hợp lệ được thay thế bằng slice cho trước.

<!-- bytes -->

#### [context](/pkg/context/)

<!-- CL 169080 -->
Định dạng của các context được trả về bởi [`WithValue`](/pkg/context/#WithValue) không còn phụ thuộc vào `fmt` và sẽ không còn stringify theo cách tương tự. Mã phụ thuộc vào cách stringify trước đây có thể bị ảnh hưởng.

<!-- context -->

#### [crypto/tls](/pkg/crypto/tls/)

Hỗ trợ SSL phiên bản 3.0 (SSLv3) [hiện đã bị deprecated và sẽ bị xóa trong Go 1.14](/issue/32716). Lưu ý rằng SSLv3 là giao thức [bị phá vỡ về mặt mật mã](https://tools.ietf.org/html/rfc7568) có trước TLS.

SSLv3 luôn bị tắt theo mặc định, ngoại trừ trong Go 1.12, khi nó vô tình được bật theo mặc định ở phía máy chủ. Giờ nó lại bị tắt theo mặc định. (SSLv3 chưa bao giờ được hỗ trợ phía client.)

<!-- CL 177698 -->
Chứng chỉ Ed25519 giờ được hỗ trợ trong TLS phiên bản 1.2 và 1.3.

<!-- crypto/tls -->

#### [crypto/x509](/pkg/crypto/x509/)

<!-- CL 175478 -->
Khóa Ed25519 giờ được hỗ trợ trong chứng chỉ và yêu cầu chứng chỉ theo [RFC 8410](https://www.rfc-editor.org/info/rfc8410), cũng như bởi các hàm
[`ParsePKCS8PrivateKey`](/pkg/crypto/x509/#ParsePKCS8PrivateKey),
[`MarshalPKCS8PrivateKey`](/pkg/crypto/x509/#MarshalPKCS8PrivateKey),
và [`ParsePKIXPublicKey`](/pkg/crypto/x509/#ParsePKIXPublicKey).

<!-- CL 169238 -->
Các đường dẫn được tìm kiếm cho system roots giờ bao gồm `/etc/ssl/cert.pem`
để hỗ trợ vị trí mặc định trong Alpine Linux 3.7+.

<!-- crypto/x509 -->

#### [database/sql](/pkg/database/sql/)

<!-- CL 170699 -->
Kiểu mới [`NullTime`](/pkg/database/sql/#NullTime) biểu diễn `time.Time` có thể null.

<!-- CL 174178 -->
Kiểu mới [`NullInt32`](/pkg/database/sql/#NullInt32) biểu diễn `int32` có thể null.

<!-- database/sql -->

#### [debug/dwarf](/pkg/debug/dwarf/)

<!-- CL 158797 -->
Phương thức [`Data.Type`](/pkg/debug/dwarf/#Data.Type)
không còn panic nếu gặp DWARF tag không xác định trong đồ thị kiểu. Thay vào đó, nó biểu diễn thành phần đó của kiểu bằng đối tượng [`UnsupportedType`](/pkg/debug/dwarf/#UnsupportedType).

<!-- debug/dwarf -->

#### [errors](/pkg/errors/)

<!-- CL 163558 -->

Hàm mới [`As`](/pkg/errors/#As) tìm lỗi đầu tiên trong chuỗi lỗi của lỗi đã cho (chuỗi lỗi được bao bọc) khớp với kiểu của target đã cho, và nếu vậy, đặt target bằng giá trị lỗi đó.

Hàm mới [`Is`](/pkg/errors/#Is) báo cáo liệu giá trị lỗi đã cho có khớp với lỗi trong chuỗi của lỗi khác không.

Hàm mới [`Unwrap`](/pkg/errors/#Unwrap) trả về kết quả của việc gọi `Unwrap` trên lỗi đã cho, nếu có.

<!-- errors -->

#### [fmt](/pkg/fmt/)

<!-- CL 160245 -->

Các verb in `%x` và `%X` giờ định dạng số thực và số phức ở ký hiệu thập lục phân, lần lượt là chữ thường và chữ hoa.

<!-- CL 160246 -->

Verb in mới `%O` định dạng số nguyên ở cơ số 8, phát ra tiền tố `0o`.

<!-- CL 160247 -->

Scanner giờ chấp nhận các giá trị số thực thập lục phân, dấu gạch dưới phân cách chữ số và các tiền tố dẫn đầu `0b` và `0o`.
Xem [Thay đổi ngôn ngữ](#language) để biết chi tiết.

<!-- CL 176998 -->

Hàm [`Errorf`](/pkg/fmt/#Errorf)
có verb mới, `%w`, toán hạng của nó phải là lỗi.
Lỗi được trả về từ `Errorf` sẽ có phương thức `Unwrap` trả về toán hạng của `%w`.

<!-- fmt -->

#### [go/scanner](/pkg/go/scanner/)

<!-- CL 175218 -->
Scanner đã được cập nhật để nhận diện các literal số Go mới, cụ thể là literal nhị phân với tiền tố `0b`/`0B`, literal bát phân với tiền tố `0o`/`0O`, và số thực với phần định trị thập lục phân. Hậu tố phần ảo `i` giờ có thể dùng với bất kỳ literal số nào, và dấu gạch dưới có thể dùng làm dấu phân cách chữ số để nhóm.
Xem [Thay đổi ngôn ngữ](#language) để biết chi tiết.

<!-- go/scanner -->

#### [go/types](/pkg/go/types/)

Bộ kiểm tra kiểu đã được cập nhật để theo các quy tắc mới về dịch số nguyên.
Xem [Thay đổi ngôn ngữ](#language) để biết chi tiết.

<!-- go/types -->

#### [html/template](/pkg/html/template/)

<!-- CL 175218 -->
Khi dùng thẻ `<script>` với "module" được đặt làm thuộc tính type, mã giờ sẽ được hiểu là [JavaScript module script](https://html.spec.whatwg.org/multipage/scripting.html#the-script-element:module-script-2).

<!-- html/template -->

#### [log](/pkg/log/)

<!-- CL 168920 -->
Hàm mới [`Writer`](/pkg/log/#Writer) trả về đích đầu ra cho logger chuẩn.

<!-- log -->

#### [math/big](/pkg/math/big/)

<!-- CL 160682 -->
Phương thức mới [`Rat.SetUint64`](/pkg/math/big/#Rat.SetUint64) đặt `Rat` thành giá trị `uint64`.

<!-- CL 166157 -->
Với [`Float.Parse`](/pkg/math/big/#Float.Parse), nếu cơ số là 0, dấu gạch dưới có thể dùng giữa các chữ số để dễ đọc.
Xem [Thay đổi ngôn ngữ](#language) để biết chi tiết.

<!-- CL 166157 -->
Với [`Int.SetString`](/pkg/math/big/#Int.SetString), nếu cơ số là 0, dấu gạch dưới có thể dùng giữa các chữ số để dễ đọc.
Xem [Thay đổi ngôn ngữ](#language) để biết chi tiết.

<!-- CL 168237 -->
[`Rat.SetString`](/pkg/math/big/#Rat.SetString) giờ chấp nhận biểu diễn số thực không phải thập phân.

<!-- math/big -->

#### [math/bits](/pkg/math/bits/)

<!-- CL 178177 -->
Thời gian thực thi của [`Add`](/pkg/math/bits/#Add),
[`Sub`](/pkg/math/bits/#Sub),
[`Mul`](/pkg/math/bits/#Mul),
[`RotateLeft`](/pkg/math/bits/#RotateLeft), và
[`ReverseBytes`](/pkg/math/bits/#ReverseBytes) giờ được đảm bảo độc lập với đầu vào.

<!-- math/bits -->

#### [net](/pkg/net/)

<!-- CL 156366 -->
Trên hệ thống Unix nơi `use-vc` được đặt trong `resolv.conf`, TCP được dùng để phân giải DNS.

<!-- CL 170678 -->
Trường mới [`ListenConfig.KeepAlive`](/pkg/net/#ListenConfig.KeepAlive)
chỉ định khoảng thời gian keep-alive cho các kết nối mạng được listener chấp nhận.
Nếu trường này là 0 (mặc định), TCP keep-alive sẽ được bật.
Để tắt chúng, đặt giá trị âm.

Lưu ý rằng lỗi được trả về từ I/O trên kết nối bị đóng bởi timeout keep-alive sẽ có phương thức `Timeout` trả về `true` khi gọi.
Điều này có thể khiến lỗi keep-alive khó phân biệt với lỗi được trả về do bỏ lỡ deadline đặt bởi phương thức [`SetDeadline`](/pkg/net/#Conn) và các phương thức tương tự.
Mã dùng deadline và kiểm tra chúng bằng phương thức `Timeout` hoặc [`os.IsTimeout`](/pkg/os/#IsTimeout) có thể muốn tắt keep-alive, hoặc dùng `errors.Is(syscall.ETIMEDOUT)` (trên hệ thống Unix) vốn sẽ trả về true cho timeout keep-alive và false cho timeout deadline.

<!-- net -->

#### [net/http](/pkg/net/http/)

<!-- CL 76410 -->
Hai trường mới [`Transport.WriteBufferSize`](/pkg/net/http/#Transport.WriteBufferSize)
và [`Transport.ReadBufferSize`](/pkg/net/http/#Transport.ReadBufferSize)
cho phép chỉ định kích thước buffer ghi và đọc cho [`Transport`](/pkg/net/http/#Transport).
Nếu bất kỳ trường nào bằng không, kích thước mặc định 4KB được dùng.

<!-- CL 130256 -->
Trường mới [`Transport.ForceAttemptHTTP2`](/pkg/net/http/#Transport.ForceAttemptHTTP2)
kiểm soát liệu HTTP/2 có được bật không khi hàm `Dial`, `DialTLS`, hoặc `DialContext` không rỗng hoặc `TLSClientConfig` được cung cấp.

<!-- CL 140357 -->
[`Transport.MaxConnsPerHost`](/pkg/net/http/#Transport.MaxConnsPerHost) giờ hoạt động đúng với HTTP/2.

<!-- CL 154383 -->
[`ResponseWriter`](/pkg/net/http/#ResponseWriter) của [`TimeoutHandler`](/pkg/net/http/#TimeoutHandler)
giờ triển khai interface [`Pusher`](/pkg/net/http/#Pusher).

<!-- CL 157339 -->
`StatusCode` `103` `"Early Hints"` đã được thêm vào.

<!-- CL 163599 -->
[`Transport`](/pkg/net/http/#Transport) giờ dùng triển khai [`io.ReaderFrom`](/pkg/io/#ReaderFrom) của [`Request.Body`](/pkg/net/http/#Request.Body) nếu có, để tối ưu hóa việc ghi body.

<!-- CL 167017 -->
Khi gặp transfer-encoding không được hỗ trợ, [`http.Server`](/pkg/net/http/#Server) giờ trả về trạng thái "501 Unimplemented" như theo yêu cầu của đặc tả HTTP [RFC 7230 Section 3.3.1](https://tools.ietf.org/html/rfc7230#section-3.3.1).

<!-- CL 167681 -->
Các trường mới của [`Server`](/pkg/net/http/#Server)
[`BaseContext`](/pkg/net/http/#Server.BaseContext) và
[`ConnContext`](/pkg/net/http/#Server.ConnContext)
cho phép kiểm soát chi tiết hơn về các giá trị [`Context`](/pkg/context/#Context) được cung cấp cho request và kết nối.

<!-- CL 167781 -->
[`http.DetectContentType`](/pkg/net/http/#DetectContentType) giờ phát hiện đúng chữ ký RAR và có thể phát hiện cả chữ ký RAR v5.

<!-- CL 173658 -->
Phương thức mới [`Clone`](/pkg/net/http/#Header.Clone) của [`Header`](/pkg/net/http/#Header) trả về bản sao của receiver.

<!-- CL 174324 -->
Hàm mới [`NewRequestWithContext`](/pkg/net/http/#NewRequestWithContext) đã được thêm vào và chấp nhận [`Context`](/pkg/context/#Context) kiểm soát toàn bộ vòng đời của [`Request`](/pkg/net/http/#Request) ra ngoài được tạo, phù hợp để dùng với [`Client.Do`](/pkg/net/http/#Client.Do) và [`Transport.RoundTrip`](/pkg/net/http/#Transport.RoundTrip).

<!-- CL 179457 -->
[`Transport`](/pkg/net/http/#Transport) không còn ghi log lỗi khi máy chủ đóng nhẹ nhàng các kết nối rảnh bằng phản hồi `"408 Request Timeout"`.

<!-- net/http -->

#### [os](/pkg/os/)

<!-- CL 160877 -->
Hàm mới [`UserConfigDir`](/pkg/os/#UserConfigDir)
trả về thư mục mặc định để dùng cho dữ liệu cấu hình theo người dùng.

<!-- CL 166578 -->
Nếu [`File`](/pkg/os/#File) được mở bằng cờ O_APPEND, phương thức [`WriteAt`](/pkg/os/#File.WriteAt) của nó sẽ luôn trả về lỗi.

<!-- os -->

#### [os/exec](/pkg/os/exec/)

<!-- CL 174318 -->
Trên Windows, môi trường cho [`Cmd`](/pkg/os/exec/#Cmd) luôn kế thừa giá trị `%SYSTEMROOT%` của tiến trình cha trừ khi trường [`Cmd.Env`](/pkg/os/exec/#Cmd.Env) bao gồm giá trị tường minh cho nó.

<!-- os/exec -->

#### [reflect](/pkg/reflect/)

<!-- CL 171337 -->
Phương thức mới [`Value.IsZero`](/pkg/reflect/#Value.IsZero) báo cáo liệu `Value` có phải là giá trị zero cho kiểu của nó không.

<!-- CL 174531 -->
Hàm [`MakeFunc`](/pkg/reflect/#MakeFunc) giờ cho phép chuyển đổi gán trên các giá trị được trả về, thay vì yêu cầu khớp kiểu chính xác. Điều này đặc biệt hữu ích khi kiểu được trả về là interface, nhưng giá trị thực sự được trả về là giá trị cụ thể triển khai interface đó.

<!-- reflect -->

#### [runtime](/pkg/runtime/)

<!-- CL 167780 -->
Traceback, [`runtime.Caller`](/pkg/runtime/#Caller),
và [`runtime.Callers`](/pkg/runtime/#Callers) giờ tham chiếu đến hàm khởi tạo các biến toàn cục của `PKG`
là `PKG.init` thay vì `PKG.init.ializers`.

<!-- runtime -->

#### [strconv](/pkg/strconv/)

<!-- CL 160243 -->
Với [`strconv.ParseFloat`](/pkg/strconv/#ParseFloat),
[`strconv.ParseInt`](/pkg/strconv/#ParseInt)
và [`strconv.ParseUint`](/pkg/strconv/#ParseUint),
nếu cơ số là 0, dấu gạch dưới có thể dùng giữa các chữ số để dễ đọc.
Xem [Thay đổi ngôn ngữ](#language) để biết chi tiết.

<!-- strconv -->

#### [strings](/pkg/strings/)

<!-- CL 142003 -->
Hàm mới [`ToValidUTF8`](/pkg/strings/#ToValidUTF8) trả về bản sao của chuỗi đã cho với mỗi chuỗi byte UTF-8 không hợp lệ được thay thế bằng chuỗi đã cho.

<!-- strings -->

#### [sync](/pkg/sync/)

<!-- CL 148958, CL 148959, CL 152697, CL 152698 -->
Các fast path của [`Mutex.Lock`](/pkg/sync/#Mutex.Lock), [`Mutex.Unlock`](/pkg/sync/#Mutex.Unlock),
[`RWMutex.Lock`](/pkg/sync/#RWMutex.Lock), [`RWMutex.RUnlock`](/pkg/sync/#Mutex.RUnlock), và
[`Once.Do`](/pkg/sync/#Once.Do) giờ được inline vào caller của chúng.
Với các trường hợp không tranh chấp trên amd64, các thay đổi này làm cho [`Once.Do`](/pkg/sync/#Once.Do) nhanh gấp đôi, và các phương thức [`Mutex`](/pkg/sync/#Mutex)/[`RWMutex`](/pkg/sync/#RWMutex) nhanh hơn tới 10%.

<!-- CL 166960 -->
[`Pool`](/pkg/sync/#Pool) lớn không còn làm tăng thời gian dừng stop-the-world.

<!-- CL 166961 -->
`Pool` không còn cần được điền lại hoàn toàn sau mỗi GC. Nó giờ giữ lại một số đối tượng qua các GC, thay vì giải phóng tất cả đối tượng, giảm tải đột biến cho người dùng nhiều của `Pool`.

<!-- sync -->

#### [syscall](/pkg/syscall/)

<!-- CL 168479 -->
Việc sử dụng `_getdirentries64` đã được xóa khỏi các build Darwin, để cho phép upload tệp nhị phân Go lên macOS App Store.

<!-- CL 174197 -->
Các trường mới `ProcessAttributes` và `ThreadAttributes` trong [`SysProcAttr`](/pkg/syscall/?GOOS=windows#SysProcAttr) đã được giới thiệu cho Windows, công khai các cài đặt bảo mật khi tạo tiến trình mới.

<!-- CL 174320 -->
`EINVAL` không còn được trả về trong chế độ
[`Chmod`](/pkg/syscall/?GOOS=windows#Chmod) bằng không trên Windows.

<!-- CL 191337 -->
Các giá trị kiểu `Errno` có thể được kiểm tra so với các giá trị lỗi trong gói `os`, như [`ErrExist`](/pkg/os/#ErrExist), bằng cách dùng [`errors.Is`](/pkg/errors/#Is).

<!-- syscall -->

#### [syscall/js](/pkg/syscall/js/)

<!-- CL 177537 -->
`TypedArrayOf` đã được thay thế bởi
[`CopyBytesToGo`](/pkg/syscall/js/#CopyBytesToGo) và
[`CopyBytesToJS`](/pkg/syscall/js/#CopyBytesToJS) để sao chép byte giữa slice byte và `Uint8Array`.

<!-- syscall/js -->

#### [testing](/pkg/testing/)

<!-- CL 112155 -->
Khi chạy benchmark, [`B.N`](/pkg/testing/#B.N) không còn được làm tròn.

<!-- CL 166717 -->
Phương thức mới [`B.ReportMetric`](/pkg/testing/#B.ReportMetric) cho phép người dùng báo cáo các chỉ số benchmark tùy chỉnh và ghi đè các chỉ số tích hợp.

<!-- CL 173722 -->
Các cờ testing giờ được đăng ký trong hàm mới [`Init`](/pkg/testing/#Init), được gọi bởi hàm `main` được tạo ra cho test. Do đó, các cờ testing giờ chỉ được đăng ký khi chạy file nhị phân test, và các gói gọi `flag.Parse` trong quá trình khởi tạo gói có thể khiến test thất bại.

<!-- testing -->

#### [text/scanner](/pkg/text/scanner/)

<!-- CL 183077 -->
Scanner đã được cập nhật để nhận diện các literal số Go mới, cụ thể là literal nhị phân với tiền tố `0b`/`0B`, literal bát phân với tiền tố `0o`/`0O`, và số thực với phần định trị thập lục phân.
Ngoài ra, chế độ mới [`AllowDigitSeparators`](/pkg/text/scanner/#AllowDigitSeparators) cho phép literal số chứa dấu gạch dưới làm dấu phân cách chữ số (tắt theo mặc định để tương thích ngược).
Xem [Thay đổi ngôn ngữ](#language) để biết chi tiết.

<!-- text/scanner -->

#### [text/template](/pkg/text/template/)

<!-- CL 161762 -->
[Hàm slice](/pkg/text/template/#hdr-Functions) mới trả về kết quả của việc cắt tham số đầu tiên theo các tham số tiếp theo.

<!-- text/template -->

#### [time](/pkg/time/)

<!-- CL 122876 -->
Ngày trong năm giờ được hỗ trợ bởi [`Format`](/pkg/time/#Time.Format) và [`Parse`](/pkg/time/#Parse).

<!-- CL 167387 -->
Các phương thức [`Duration`](/pkg/time/#Duration) mới
[`Microseconds`](/pkg/time/#Duration.Microseconds) và
[`Milliseconds`](/pkg/time/#Duration.Milliseconds) trả về duration dưới dạng số nguyên theo đơn vị tương ứng.

<!-- time -->

#### [unicode](/pkg/unicode/)

Gói [`unicode`](/pkg/unicode/) và hỗ trợ liên quan trong toàn hệ thống đã được nâng cấp từ Unicode 10.0 lên [Unicode 11.0](https://www.unicode.org/versions/Unicode11.0.0/), bổ sung 684 ký tự mới, bao gồm bảy script mới và 66 emoji mới.

<!-- unicode -->
