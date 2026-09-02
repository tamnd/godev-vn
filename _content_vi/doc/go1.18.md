---
path: /doc/go1.18
title: Ghi chú phát hành Go 1.18
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

## Giới thiệu về Go 1.18 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.18,
là một bản phát hành quan trọng, bao gồm các thay đổi về ngôn ngữ,
triển khai toolchain, runtime và thư viện.
Go 1.18 ra đời bảy tháng sau [Go 1.17](/doc/go1.17).
Như thường lệ, bản phát hành này duy trì [cam kết tương thích](/doc/go1compat) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước đây.

## Thay đổi về ngôn ngữ {#language}

### Generics {#generics}

<!-- https://golang.org/issue/43651, https://golang.org/issue/45346 -->
Go 1.18 bao gồm một triển khai các tính năng generics như được mô tả trong
[Đề xuất Type Parameters](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md).
Điều này bao gồm các thay đổi lớn nhưng hoàn toàn tương thích ngược đối với ngôn ngữ.

Những thay đổi ngôn ngữ mới này đòi hỏi một lượng lớn code mới chưa
được kiểm tra đáng kể trong môi trường sản xuất. Điều đó sẽ
chỉ xảy ra khi ngày càng nhiều người viết và sử dụng code generic. Chúng tôi tin
rằng tính năng này được triển khai tốt và chất lượng cao. Tuy nhiên,
không như hầu hết các khía cạnh của Go, chúng tôi không thể hỗ trợ niềm tin đó bằng kinh nghiệm thực tế.
Do đó, trong khi chúng tôi khuyến khích việc sử dụng generics
khi phù hợp, hãy thận trọng khi triển khai code generic trong môi trường sản xuất.

Mặc dù chúng tôi tin rằng các tính năng ngôn ngữ mới được thiết kế tốt
và được đặc tả rõ ràng, có thể chúng tôi đã mắc lỗi.
Chúng tôi muốn nhấn mạnh rằng [cam kết tương thích Go 1](/doc/go1compat) ghi rằng
"Nếu cần thiết phải giải quyết sự không nhất quán hoặc không hoàn chỉnh trong đặc tả, việc giải quyết
vấn đề có thể ảnh hưởng đến ý nghĩa hoặc tính hợp lệ của các chương trình hiện có.
Chúng tôi bảo lưu quyền giải quyết các vấn đề như vậy, bao gồm cập nhật các triển khai."
Nó cũng nói rằng "Nếu trình biên dịch hoặc thư viện có lỗi vi phạm đặc tả,
một chương trình phụ thuộc vào hành vi lỗi đó có thể bị hỏng nếu lỗi được sửa.
Chúng tôi bảo lưu quyền sửa các lỗi như vậy." Nói cách khác, có thể
sẽ có code sử dụng generics hoạt động được với bản phát hành 1.18
nhưng bị hỏng trong các bản phát hành sau. Chúng tôi không có kế hoạch hoặc kỳ vọng
thực hiện bất kỳ thay đổi như vậy. Tuy nhiên, việc phá vỡ các chương trình 1.18 trong các bản phát hành tương lai có thể
trở nên cần thiết vì những lý do chúng tôi hiện không thể lường trước. Chúng tôi sẽ
giảm thiểu bất kỳ sự phá vỡ như vậy càng nhiều càng tốt, nhưng chúng tôi không thể đảm bảo rằng sự phá vỡ sẽ bằng không.

Sau đây là danh sách các thay đổi nổi bật nhất. Để có cái nhìn tổng quan đầy đủ hơn, xem
[đề xuất](https://go.googlesource.com/proposal/+/refs/heads/master/design/43651-type-parameters.md).
Để biết chi tiết, xem [đặc tả ngôn ngữ](/ref/spec).

  - Cú pháp cho các khai báo
    [hàm](/ref/spec#Function_declarations) và
    [kiểu](/ref/spec#Type_declarations)
    hiện chấp nhận
    [các tham số kiểu](/ref/spec#Type_parameter_declarations).
  - Các hàm và kiểu được tham số hóa có thể được khởi tạo bằng cách theo sau chúng với danh sách
    các đối số kiểu trong dấu ngoặc vuông.
  - Token mới `~` đã được thêm vào tập hợp
    [các toán tử và dấu câu](/ref/spec#Operators_and_punctuation).
  - Cú pháp cho
    [các kiểu Interface](/ref/spec#Interface_types)
    hiện cho phép nhúng các kiểu tùy ý (không chỉ tên kiểu của các interface)
    cũng như các phần tử kiểu union và `~T`. Các interface như vậy chỉ có thể được sử dụng
    làm ràng buộc kiểu.
    Một interface hiện định nghĩa một tập hợp các kiểu cũng như một tập hợp các phương thức.
  - [Định danh được khai báo trước](/ref/spec#Predeclared_identifiers) mới
    `any` là bí danh cho interface rỗng. Nó có thể được sử dụng thay vì
    `interface{}`.
  - [Định danh được khai báo trước](/ref/spec#Predeclared_identifiers) mới
    `comparable` là một interface biểu thị tập hợp tất cả các kiểu có thể
    được so sánh bằng `==` hoặc `!=`. Nó chỉ có thể được sử dụng như là (hoặc được nhúng trong)
    một ràng buộc kiểu.

Có ba package thử nghiệm sử dụng generics có thể
hữu ích.
Các package này nằm trong kho x/exp; API của chúng không được bao phủ bởi
cam kết Go 1 và có thể thay đổi khi chúng tôi có thêm kinh nghiệm với
generics.

#### [`golang.org/x/exp/constraints`](https://pkg.go.dev/golang.org/x/exp/constraints)

Các ràng buộc hữu ích cho code generic, chẳng hạn như
[`constraints.Ordered`](https://pkg.go.dev/golang.org/x/exp/constraints#Ordered).

#### [`golang.org/x/exp/slices`](https://pkg.go.dev/golang.org/x/exp/slices)

Tập hợp các hàm generic hoạt động trên các slice của
bất kỳ kiểu phần tử nào.

#### [`golang.org/x/exp/maps`](https://pkg.go.dev/golang.org/x/exp/maps)

Tập hợp các hàm generic hoạt động trên các map với
bất kỳ kiểu key hoặc kiểu phần tử nào.


Triển khai generics hiện tại có các giới hạn đã biết sau:

  - <!-- https://golang.org/issue/47631 -->
    Trình biên dịch Go không thể xử lý các khai báo kiểu bên trong các hàm
    hoặc phương thức generic. Chúng tôi hy vọng cung cấp hỗ trợ cho tính năng này trong một
    bản phát hành trong tương lai.
  - <!-- https://golang.org/issue/50937 -->
    Trình biên dịch Go không chấp nhận các đối số có kiểu tham số kiểu với
    các hàm được khai báo trước `real`, `imag` và `complex`.
    Chúng tôi hy vọng loại bỏ hạn chế này trong một bản phát hành trong tương lai.
  - <!-- https://golang.org/issue/51183 -->
    Trình biên dịch Go chỉ hỗ trợ gọi phương thức `m` trên giá trị
    `x` của kiểu tham số kiểu `P` nếu `m` được khai báo tường minh
    bởi interface ràng buộc của `P`.
    Tương tự, các giá trị phương thức `x.m` và các biểu thức phương thức
    `P.m` cũng chỉ được hỗ trợ nếu `m` được khai báo tường minh
    bởi `P`, mặc dù `m` có thể nằm trong tập phương thức
    của `P` do tất cả các kiểu trong `P` đều triển khai
    `m`. Chúng tôi hy vọng loại bỏ hạn chế này trong một bản phát hành
    trong tương lai.
  - <!-- https://golang.org/issue/51576 -->
    Trình biên dịch Go không hỗ trợ truy cập trường struct `x.f`
    khi `x` có kiểu tham số kiểu ngay cả khi tất cả các kiểu trong
    tập kiểu của tham số kiểu có trường `f`.
    Chúng tôi có thể loại bỏ hạn chế này trong một bản phát hành trong tương lai.
  - <!-- https://golang.org/issue/49030 -->
    Việc nhúng một tham số kiểu, hoặc con trỏ đến tham số kiểu, như là
    một trường không tên trong kiểu struct là không được phép. Tương tự,
    việc nhúng tham số kiểu trong kiểu interface là không được phép.
    Hiện tại chưa rõ liệu những điều này có bao giờ được cho phép hay không.
  - Một phần tử union với nhiều hơn một term có thể không chứa
    kiểu interface với tập phương thức không rỗng. Hiện tại chưa rõ liệu điều này có bao giờ được cho phép hay không.


Generics cũng đại diện cho một thay đổi lớn đối với hệ sinh thái Go. Mặc dù chúng tôi đã
cập nhật một số công cụ cốt lõi với hỗ trợ generics, vẫn còn nhiều việc phải làm.
Sẽ cần thời gian để các công cụ, tài liệu và thư viện còn lại theo kịp
những thay đổi ngôn ngữ này.

### Sửa lỗi {#bug_fixes}

Trình biên dịch Go 1.18 hiện báo cáo đúng các lỗi `declared but not used`
cho các biến được đặt bên trong một function literal nhưng không bao giờ được sử dụng. Trước Go 1.18,
trình biên dịch không báo lỗi trong các trường hợp như vậy. Điều này sửa lỗi trình biên dịch
tồn tại lâu dài [#8560](/issue/8560). Do thay đổi này,
các chương trình (có thể không đúng) có thể không biên dịch được nữa. Cách sửa cần thiết
rất đơn giản: sửa chương trình nếu thực sự nó không đúng, hoặc sử dụng biến vi phạm,
ví dụ bằng cách gán nó cho định danh trống `_`.
Vì `go vet` luôn chỉ ra lỗi này, số lượng chương trình bị ảnh hưởng có thể rất nhỏ.

Trình biên dịch Go 1.18 hiện báo cáo tràn số khi truyền một biểu thức hằng rune
như `'1' << 32` làm đối số cho các hàm được khai báo trước
`print` và `println`, nhất quán với hành vi của
các hàm do người dùng định nghĩa. Trước Go 1.18, trình biên dịch không báo lỗi
trong các trường hợp như vậy nhưng chấp nhận im lặng các đối số hằng như vậy nếu chúng vừa trong
`int64`. Do thay đổi này, các chương trình (có thể không đúng)
có thể không biên dịch được nữa. Cách sửa cần thiết rất đơn giản: sửa chương trình nếu
thực sự nó không đúng, hoặc chuyển đổi tường minh đối số vi phạm sang kiểu đúng.
Vì `go vet` luôn chỉ ra lỗi này, số lượng chương trình bị ảnh hưởng có thể rất nhỏ.

## Các nền tảng {#ports}

### AMD64 {#amd64}

<!-- CL 349595 -->
Go 1.18 giới thiệu biến môi trường `GOAMD64` mới, chọn tại thời điểm biên dịch
phiên bản mục tiêu tối thiểu của kiến trúc AMD64. Các giá trị được phép là `v1`,
`v2`, `v3` hoặc `v4`. Mỗi cấp độ cao hơn yêu cầu
và tận dụng các tính năng bộ xử lý bổ sung. Mô tả chi tiết
có thể tìm thấy [tại đây](/wiki/MinimumRequirements#amd64).

Biến môi trường `GOAMD64` mặc định là `v1`.

### RISC-V {#riscv}

<!-- golang.org/issue/47100, CL 334872 -->
Kiến trúc RISC-V 64-bit trên Linux (cổng `linux/riscv64`)
hiện hỗ trợ các chế độ build `c-archive` và `c-shared`.

### Linux {#linux}

<!-- golang.org/issue/45964 -->
Go 1.18 yêu cầu Linux kernel phiên bản 2.6.32 trở lên.

### Windows {#windows}

<!-- https://golang.org/issue/49759 -->
Các cổng `windows/arm` và `windows/arm64` hiện hỗ trợ
preemption không cộng tác, đưa khả năng này đến tất cả bốn cổng Windows,
điều này hy vọng sẽ giải quyết các lỗi khó phát hiện khi gọi
vào các hàm Win32 bị chặn trong thời gian dài.

### iOS {#ios}

<!-- golang.org/issue/48076, golang.org/issue/49616 -->
Trên iOS (cổng `ios/arm64`)
và trình giả lập iOS chạy trên macOS dựa trên AMD64 (cổng `ios/amd64`),
Go 1.18 hiện yêu cầu iOS 12 trở lên; hỗ trợ cho các phiên bản trước đã bị ngừng.

### FreeBSD {#freebsd}

Go 1.18 là bản phát hành cuối cùng được hỗ trợ trên FreeBSD 11.x, vốn đã
đạt đến ngày kết thúc vòng đời. Go 1.19 sẽ yêu cầu FreeBSD 12.2+ hoặc FreeBSD
13.0+.
FreeBSD 13.0+ sẽ yêu cầu một kernel với tùy chọn COMPAT\_FREEBSD12 được đặt (đây là mặc định).

## Công cụ {#tools}

### Fuzzing {#fuzzing}

Go 1.18 bao gồm một triển khai fuzzing như được mô tả trong
[đề xuất fuzzing](/issue/44551).

Xem [trang landing fuzzing](/security/fuzz) để bắt đầu.

Hãy lưu ý rằng fuzzing có thể tiêu thụ nhiều bộ nhớ và có thể ảnh hưởng đến
hiệu suất máy của bạn trong khi chạy. Cũng lưu ý rằng engine fuzzing
ghi các giá trị mở rộng phủ sóng test vào thư mục fuzz cache trong
`$GOCACHE/fuzz` trong khi chạy. Hiện tại không có giới hạn về
số file hoặc tổng số byte có thể được ghi vào fuzz cache, vì vậy nó
có thể chiếm nhiều dung lượng lưu trữ (có thể vài GB).

### Lệnh go {#go-command}

#### `go` `get` {#go-get}

<!-- golang.org/issue/43684 -->
`go` `get` không còn build hoặc cài đặt các package trong
chế độ nhận biết module. `go` `get` hiện dành riêng cho
việc điều chỉnh các dependency trong `go.mod`. Thực tế là cờ
`-d` luôn được bật. Để cài đặt phiên bản mới nhất
của một tệp thực thi ngoài ngữ cảnh của module hiện tại, hãy sử dụng
[`go` `install` `example.com/cmd@latest`](/ref/mod#go-install). Bất kỳ
[truy vấn phiên bản](/ref/mod#version-queries) nào
có thể được sử dụng thay vì `latest`. Dạng `go`
`install` này đã được thêm vào trong Go 1.16, vì vậy các dự án hỗ trợ các phiên bản cũ hơn
có thể cần cung cấp hướng dẫn cài đặt cho cả `go`
`install` lẫn `go` `get`. `go`
`get` hiện báo lỗi khi được sử dụng ngoài một module, vì không có
file `go.mod` để cập nhật. Trong chế độ GOPATH (với
`GO111MODULE=off`), `go` `get` vẫn build
và cài đặt các package như trước.

#### Cập nhật tự động `go.mod` và `go.sum` {#go-mod-updates}

<!-- https://go.dev/issue/45551 -->
Các lệnh con `go` `mod` `graph`,
`go` `mod` `vendor`,
`go` `mod` `verify` và
`go` `mod` `why`
không còn tự động cập nhật các file `go.mod` và
`go.sum` nữa.
(Các file đó có thể được cập nhật tường minh bằng `go` `get`,
`go` `mod` `tidy` hoặc
`go` `mod` `download`.)

#### `go` `version` {#go-version}

<!-- golang.org/issue/37475 -->
Lệnh `go` hiện nhúng thông tin kiểm soát phiên bản vào
các nhị phân. Nó bao gồm revision hiện đang được checkout, thời gian commit và một
cờ chỉ ra liệu có các file đã chỉnh sửa hoặc chưa được theo dõi hay không. Thông tin
kiểm soát phiên bản được nhúng nếu lệnh `go` được gọi trong
một thư mục trong kho lưu trữ Git, Mercurial, Fossil hoặc Bazaar, và
package `main` cùng module main chứa nó nằm trong cùng
kho lưu trữ. Thông tin này có thể bị bỏ qua bằng cờ
`-buildvcs=false`.

<!-- golang.org/issue/37475 -->
Ngoài ra, lệnh `go` nhúng thông tin về build,
bao gồm các build tag và tool tag (được đặt với `-tags`), trình biên dịch,
assembler và các cờ linker (như `-gcflags`), liệu cgo có
được bật không, và nếu có, các giá trị của các biến môi trường cgo
(như `CGO_CFLAGS`).
Cả thông tin VCS và build có thể được đọc cùng với thông tin module
bằng cách sử dụng
`go` `version` `-m` `file` hoặc
`runtime/debug.ReadBuildInfo` (cho nhị phân đang chạy hiện tại)
hoặc package [`debug/buildinfo`](#debug/buildinfo) mới.

<!-- CL 369977 -->
Định dạng dữ liệu cơ bản của thông tin build được nhúng có thể thay đổi với
các bản phát hành Go mới, vì vậy một phiên bản `go` cũ hơn có thể không xử lý được
thông tin build được tạo ra với phiên bản `go` mới hơn.
Để đọc thông tin phiên bản từ nhị phân được build với `go` 1.18,
hãy sử dụng lệnh `go` `version` và
package `debug/buildinfo` từ `go` 1.18+.

#### `go` `mod` `download` {#go-mod-download}

<!-- https://golang.org/issue/44435 -->
Nếu file `go.mod` của module chính
chỉ định [`go` `1.17`](/ref/mod#go-mod-file-go)
hoặc cao hơn, `go` `mod` `download` không có
đối số hiện chỉ tải mã nguồn cho các module
được [yêu cầu](/ref/mod#go-mod-file-require) tường minh trong file
`go.mod` của module chính. (Trong module `go` `1.17` hoặc
cao hơn, tập hợp đó đã bao gồm tất cả các dependency cần thiết để build các
package và test trong module chính.)
Để cũng tải mã nguồn cho các dependency bắc cầu, hãy sử dụng
`go` `mod` `download` `all`.

#### `go` `mod` `vendor` {#go-mod-vendor}

<!-- https://golang.org/issue/47327 -->
Lệnh con `go` `mod` `vendor` hiện
hỗ trợ cờ `-o` để đặt thư mục đầu ra.
(Các lệnh `go` khác vẫn đọc từ thư mục `vendor`
tại thư mục gốc module khi tải các package
với `-mod=vendor`, vì vậy việc sử dụng chính của cờ này là cho
các công cụ bên thứ ba cần thu thập mã nguồn package.)

#### `go` `mod` `tidy` {#go-mod-tidy}

<!-- https://golang.org/issue/47738, CL 344572 -->
Lệnh `go` `mod` `tidy` hiện giữ lại
các checksum bổ sung trong file `go.sum` cho các module có mã nguồn
cần thiết để xác minh rằng mỗi package được import chỉ được cung cấp bởi một
module trong [danh sách build](/ref/mod#glos-build-list). Vì điều kiện này
hiếm gặp và việc không áp dụng nó dẫn đến lỗi build, thay đổi này _không_
phụ thuộc vào phiên bản `go` trong file `go.mod` của module chính.

#### `go` `work` {#go-work}

<!-- https://golang.org/issue/45713 -->
Lệnh `go` hiện hỗ trợ chế độ "Workspace". Nếu một
file `go.work` được tìm thấy trong thư mục làm việc hoặc một
thư mục cha, hoặc một file được chỉ định bằng biến môi trường `GOWORK`,
nó sẽ đặt lệnh `go` vào chế độ workspace.
Trong chế độ workspace, file `go.work` sẽ được sử dụng để
xác định tập hợp các module chính được sử dụng làm gốc cho việc giải quyết module,
thay vì sử dụng file `go.mod` thường được tìm thấy để chỉ định module chính duy nhất.
Để biết thêm thông tin, xem tài liệu
[`go work`](/pkg/cmd/go#hdr-Workspace_maintenance).

#### `go` `build` `-asan` {#go-build-asan}

<!-- CL 298612 -->
Lệnh `go` `build` và các lệnh liên quan
hiện hỗ trợ cờ `-asan` bật khả năng tương tác
với code C (hoặc C++) được biên dịch với address sanitizer (tùy chọn trình biên dịch C
`-fsanitize=address`).

#### `go` `test` {#go-test}

<!-- CL 251441 -->
Lệnh `go` hiện hỗ trợ các tùy chọn dòng lệnh bổ sung
cho [hỗ trợ fuzzing mới được mô tả ở trên](#fuzzing):

  - `go test` hỗ trợ
    các tùy chọn `-fuzz`, `-fuzztime` và
    `-fuzzminimizetime`.
    Để biết tài liệu về những điều này, xem
    [`go help testflag`](/pkg/cmd/go#hdr-Testing_flags).
  - `go clean` hỗ trợ tùy chọn `-fuzzcache`.
    Để biết tài liệu, xem
    [`go help clean`](/pkg/cmd/go#hdr-Remove_object_files_and_cached_files).


#### Các dòng `//go:build` {#go-build-lines}

<!-- CL 240611 -->
Go 1.17 đã giới thiệu các dòng `//go:build` như một cách dễ đọc hơn để viết các ràng buộc build,
thay vì các dòng `//` `+build`.
Kể từ Go 1.17, `gofmt` thêm các dòng `//go:build`
để khớp với các dòng `+build` hiện có và giữ chúng đồng bộ,
trong khi `go` `vet` chẩn đoán khi chúng không đồng bộ.

Vì bản phát hành Go 1.18 đánh dấu kết thúc hỗ trợ cho Go 1.16,
tất cả các phiên bản Go được hỗ trợ hiện hiểu các dòng `//go:build`.
Trong Go 1.18, `go` `fix` hiện loại bỏ các dòng
`//` `+build` đã lỗi thời trong các module khai báo
`go` `1.18` hoặc mới hơn trong các file `go.mod` của chúng.

Để biết thêm thông tin, xem [go.dev/design/draft-gobuild](/design/draft-gobuild).

### Gofmt {#gofmt}

<!-- https://golang.org/issue/43566 -->
`gofmt` hiện đọc và định dạng các file đầu vào đồng thời, với một
giới hạn bộ nhớ tỷ lệ với `GOMAXPROCS`. Trên máy có
nhiều CPU, `gofmt` hiện sẽ nhanh hơn đáng kể.

### Vet {#vet}

#### Cập nhật cho Generics {#vet-generics}

<!-- https://golang.org/issue/48704 -->
Công cụ `vet` được cập nhật để hỗ trợ code generic. Trong hầu hết các trường hợp,
nó báo lỗi trong code generic bất cứ khi nào nó sẽ báo lỗi trong
code tương đương không generic sau khi thay thế các tham số kiểu bằng một
kiểu từ [tập kiểu](/ref/spec#Interface_types) của chúng.
Ví dụ, `vet` báo lỗi format trong

	func Print[T ~int|~string](t T) {
		fmt.Printf("%d", t)
	}

vì nó sẽ báo lỗi format trong phiên bản không generic tương đương của
`Print[string]`:

	func PrintString(x string) {
		fmt.Printf("%d", x)
	}


#### Cải tiến độ chính xác cho các trình kiểm tra hiện có {#vet-precision}

<!-- CL 323589 356830 319689 355730 351553 338529 -->
Các trình kiểm tra `cmd/vet` `copylock`, `printf`,
`sortslice`, `testinggoroutine` và `tests`
đều có những cải tiến độ chính xác vừa phải để xử lý các mẫu code bổ sung.
Điều này có thể dẫn đến các lỗi mới được báo cáo trong các package hiện có. Ví dụ, trình kiểm tra
`printf` hiện theo dõi các chuỗi định dạng được tạo ra bằng cách
nối các hằng chuỗi. Vì vậy `vet` sẽ báo lỗi trong:

	  // fmt.Printf formatting directive %d is being passed to Println.
	  fmt.Println("%d"+` ≡ x (mod 2)`+"\n", x%2)


## Runtime {#runtime}

<!-- https://golang.org/issue/44167 -->
Bộ gom rác hiện bao gồm các nguồn công việc bộ gom rác không phải heap
(ví dụ: quét stack) khi xác định tần suất chạy. Do đó,
chi phí bộ gom rác có thể dự đoán hơn khi các nguồn này
đáng kể. Đối với hầu hết các ứng dụng, những thay đổi này sẽ không đáng kể; tuy nhiên,
một số ứng dụng Go hiện có thể sử dụng ít bộ nhớ hơn và dành nhiều thời gian hơn cho việc gom rác,
hoặc ngược lại, so với trước đây. Cách khắc phục dự kiến là điều chỉnh
`GOGC` khi cần thiết.

<!-- CL 358675, CL 353975, CL 353974 -->
Runtime hiện trả bộ nhớ về hệ điều hành hiệu quả hơn và đã
được điều chỉnh để hoạt động tích cực hơn.

<!-- CL 352057, https://golang.org/issue/45728 -->
Go 1.17 nhìn chung đã cải thiện định dạng của các đối số trong stack trace,
nhưng có thể in các giá trị không chính xác cho các đối số được truyền qua register.
Điều này được cải thiện trong Go 1.18 bằng cách in dấu hỏi (`?`)
sau mỗi giá trị có thể không chính xác.

<!-- CL 347917 -->
Hàm built-in `append` hiện sử dụng một công thức hơi khác
khi quyết định cần grow slice bao nhiêu khi phải cấp phát mảng nền mới.
Công thức mới ít bị đột biến trong hành vi cấp phát hơn.

## Trình biên dịch {#compiler}

<!-- https://golang.org/issue/40724 -->
Go 1.17 đã [triển khai](go1.17#compiler) một cách mới để truyền
các đối số và kết quả hàm bằng register thay vì stack
trên kiến trúc x86 64-bit trên các hệ điều hành được chọn.
Go 1.18 mở rộng các nền tảng được hỗ trợ để bao gồm ARM 64-bit (`GOARCH=arm64`),
PowerPC 64-bit big-endian và little-endian (`GOARCH=ppc64`, `ppc64le`),
cũng như kiến trúc x86 64-bit (`GOARCH=amd64`)
trên tất cả các hệ điều hành.
Trên các hệ thống ARM 64-bit và PowerPC 64-bit, benchmark cho thấy
cải thiện hiệu suất điển hình là 10% hoặc hơn.

Như đã [đề cập](go1.17#compiler) trong ghi chú phát hành Go 1.17,
thay đổi này không ảnh hưởng đến chức năng của bất kỳ code Go an toàn nào và
được thiết kế để không tác động đến hầu hết code assembly. Xem
[ghi chú phát hành Go 1.17](go1.17#compiler) để biết thêm chi tiết.

<!-- CL 355497, CL 356869 -->
Trình biên dịch hiện có thể inline các hàm chứa vòng lặp range hoặc
vòng lặp for có nhãn.

<!-- CL 298611 -->
Tùy chọn trình biên dịch `-asan` mới hỗ trợ tùy chọn
`-asan` của lệnh `go` mới.

<!-- https://golang.org/issue/50954 -->
Vì trình kiểm tra kiểu của trình biên dịch đã được thay thế hoàn toàn để
hỗ trợ generics, một số thông báo lỗi hiện có thể dùng từ ngữ khác
so với trước đây. Trong một số trường hợp, các thông báo lỗi trước Go 1.18 cung cấp nhiều
chi tiết hơn hoặc được diễn đạt theo cách hữu ích hơn.
Chúng tôi dự định giải quyết các trường hợp này trong Go 1.19.

<!-- /issue/49569 -->
Do các thay đổi trong trình biên dịch liên quan đến hỗ trợ generics,
tốc độ biên dịch Go 1.18 có thể chậm hơn khoảng 15% so với tốc độ biên dịch Go 1.17.
Thời gian thực thi của code đã biên dịch không bị ảnh hưởng. Chúng tôi
dự định cải thiện tốc độ của trình biên dịch trong các bản phát hành tương lai.

## Linker {#linker}

Linker phát ra [ít relocation hơn nhiều](https://tailscale.com/blog/go-linker/).
Do đó, hầu hết các codebase sẽ link nhanh hơn, yêu cầu ít bộ nhớ hơn để link,
và tạo ra các nhị phân nhỏ hơn.
Các công cụ xử lý nhị phân Go nên sử dụng package `debug/gosym` của Go 1.18
để xử lý trong suốt cả các nhị phân cũ và mới.

<!-- CL 298610 -->
Tùy chọn linker `-asan` mới hỗ trợ tùy chọn
`-asan` của lệnh `go` mới.

## Bootstrap {#bootstrap}

<!-- CL 369914, CL 370274 -->
Khi build một bản phát hành Go từ nguồn và `GOROOT_BOOTSTRAP`
không được đặt, các phiên bản Go trước đây tìm kiếm toolchain bootstrap Go 1.4 trở lên
trong thư mục `$HOME/go1.4` (`%HOMEDRIVE%%HOMEPATH%\go1.4` trên Windows).
Go hiện tìm kiếm đầu tiên cho `$HOME/go1.17` hoặc `$HOME/sdk/go1.17`
trước khi quay trở lại `$HOME/go1.4`.
Chúng tôi dự định Go 1.19 yêu cầu Go 1.17 trở lên để bootstrap,
và thay đổi này sẽ làm cho quá trình chuyển đổi mượt mà hơn.
Để biết thêm chi tiết, xem [go.dev/issue/44505](/issue/44505).

## Thư viện chuẩn {#library}

### Package `debug/buildinfo` mới {#debug_buildinfo}

<!-- golang.org/issue/39301 -->
Package mới [`debug/buildinfo`](/pkg/debug/buildinfo) cung cấp quyền truy cập vào
các phiên bản module, thông tin kiểm soát phiên bản và
các cờ build được nhúng vào các file thực thi được build bởi lệnh `go`.
Thông tin tương tự cũng có sẵn thông qua
[`runtime/debug.ReadBuildInfo`](/pkg/runtime/debug#ReadBuildInfo)
cho nhị phân đang chạy hiện tại và thông qua `go`
`version` `-m` trên dòng lệnh.

### Package `net/netip` mới {#netip}

Package mới [`net/netip`](/pkg/net/netip/)
định nghĩa kiểu địa chỉ IP mới, [`Addr`](/pkg/net/netip/#Addr).
So với kiểu [`net.IP`](/pkg/net/#IP) hiện có, kiểu `netip.Addr` chiếm ít
bộ nhớ hơn, bất biến và có thể so sánh nên nó hỗ trợ `==`
và có thể được sử dụng làm key map.

Ngoài `Addr`, package định nghĩa
[`AddrPort`](/pkg/net/netip/#AddrPort), đại diện cho
một IP và port, và
[`Prefix`](/pkg/net/netip/#Prefix), đại diện cho
một prefix CIDR mạng.

Package cũng định nghĩa một số hàm để tạo và kiểm tra
các kiểu mới này:
[`AddrFrom4`](/pkg/net/netip#AddrFrom4),
[`AddrFrom16`](/pkg/net/netip#AddrFrom16),
[`AddrFromSlice`](/pkg/net/netip#AddrFromSlice),
[`AddrPortFrom`](/pkg/net/netip#AddrPortFrom),
[`IPv4Unspecified`](/pkg/net/netip#IPv4Unspecified),
[`IPv6LinkLocalAllNodes`](/pkg/net/netip#IPv6LinkLocalAllNodes),
[`IPv6Unspecified`](/pkg/net/netip#IPv6Unspecified),
[`MustParseAddr`](/pkg/net/netip#MustParseAddr),
[`MustParseAddrPort`](/pkg/net/netip#MustParseAddrPort),
[`MustParsePrefix`](/pkg/net/netip#MustParsePrefix),
[`ParseAddr`](/pkg/net/netip#ParseAddr),
[`ParseAddrPort`](/pkg/net/netip#ParseAddrPort),
[`ParsePrefix`](/pkg/net/netip#ParsePrefix),
[`PrefixFrom`](/pkg/net/netip#PrefixFrom).

Package [`net`](/pkg/net/) bao gồm các phương thức mới song song với các phương thức hiện có, nhưng
trả về `netip.AddrPort` thay vì kiểu
nặng hơn [`net.IP`](/pkg/net/#IP) hoặc
[`*net.UDPAddr`](/pkg/net/#UDPAddr):
[`Resolver.LookupNetIP`](/pkg/net/#Resolver.LookupNetIP),
[`UDPConn.ReadFromUDPAddrPort`](/pkg/net/#UDPConn.ReadFromUDPAddrPort),
[`UDPConn.ReadMsgUDPAddrPort`](/pkg/net/#UDPConn.ReadMsgUDPAddrPort),
[`UDPConn.WriteToUDPAddrPort`](/pkg/net/#UDPConn.WriteToUDPAddrPort),
[`UDPConn.WriteMsgUDPAddrPort`](/pkg/net/#UDPConn.WriteMsgUDPAddrPort).
Các phương thức `UDPConn` mới hỗ trợ I/O không có cấp phát.

Package `net` cũng hiện bao gồm các hàm và phương thức
để chuyển đổi giữa các kiểu
[`TCPAddr`](/pkg/net/#TCPAddr)/[`UDPAddr`](/pkg/net/#UDPAddr)
hiện có và `netip.AddrPort`:
[`TCPAddrFromAddrPort`](/pkg/net/#TCPAddrFromAddrPort),
[`UDPAddrFromAddrPort`](/pkg/net/#UDPAddrFromAddrPort),
[`TCPAddr.AddrPort`](/pkg/net/#TCPAddr.AddrPort),
[`UDPAddr.AddrPort`](/pkg/net/#UDPAddr.AddrPort).

### TLS 1.0 và 1.1 bị tắt mặc định ở phía client {#tls10}

<!-- CL 359779, golang.org/issue/45428 -->
Nếu [`Config.MinVersion`](/pkg/crypto/tls/#Config.MinVersion)
không được đặt, nó hiện mặc định là TLS 1.2 cho các kết nối client. Bất kỳ server
cập nhật an toàn nào đều dự kiến hỗ trợ TLS 1.2, và các trình duyệt đã yêu cầu
nó kể từ năm 2020. TLS 1.0 và 1.1 vẫn được hỗ trợ bằng cách đặt
`Config.MinVersion` thành `VersionTLS10`.
Giá trị mặc định phía server không thay đổi ở TLS 1.0.

Mặc định có thể tạm thời đặt lại về TLS 1.0 bằng cách đặt
biến môi trường `GODEBUG=tls10default=1`.
Tùy chọn này sẽ được xóa trong Go 1.19.

### Từ chối chứng chỉ SHA-1 {#sha1}

<!-- CL 359777, golang.org/issue/41682 -->
`crypto/x509` hiện sẽ
từ chối các chứng chỉ được ký bằng hàm băm SHA-1. Điều này không
áp dụng cho các chứng chỉ root tự ký. Các cuộc tấn công thực tế chống lại SHA-1
[đã được chứng minh kể từ năm 2017](https://shattered.io/) và các
Cơ quan chứng nhận được tin cậy công khai đã không cấp chứng chỉ SHA-1 kể từ năm 2015.

Điều này có thể tạm thời đặt lại bằng cách đặt
biến môi trường `GODEBUG=x509sha1=1`.
Tùy chọn này sẽ được xóa trong một bản phát hành trong tương lai.

### Các thay đổi nhỏ trong thư viện {#minor_library_changes}

Như thường lệ, có nhiều thay đổi và cập nhật nhỏ cho thư viện,
được thực hiện với [cam kết tương thích](/doc/go1compat) của Go 1 trong tâm trí.

#### [bufio](/pkg/bufio/)

<!-- CL 345569 -->
Phương thức mới [`Writer.AvailableBuffer`](/pkg/bufio#Writer.AvailableBuffer)
trả về một buffer rỗng với dung lượng có thể không rỗng để sử dụng
với các API kiểu append. Sau khi append, buffer có thể được cung cấp cho
lời gọi `Write` tiếp theo và có thể tránh bất kỳ sao chép nào.

<!-- CL 345570 -->
Các phương thức [`Reader.Reset`](/pkg/bufio#Reader.Reset) và
[`Writer.Reset`](/pkg/bufio#Writer.Reset)
hiện sử dụng kích thước buffer mặc định khi được gọi trên các đối tượng với
buffer `nil`.

<!-- bufio -->

#### [bytes](/pkg/bytes/)

<!-- CL 351710 -->
Hàm mới [`Cut`](/pkg/bytes/#Cut)
cắt `[]byte` quanh một dấu phân cách. Nó có thể thay thế
và đơn giản hóa nhiều cách sử dụng phổ biến của
[`Index`](/pkg/bytes/#Index),
[`IndexByte`](/pkg/bytes/#IndexByte),
[`IndexRune`](/pkg/bytes/#IndexRune)
và [`SplitN`](/pkg/bytes/#SplitN).

<!-- CL 323318, CL 332771 -->
[`Trim`](/pkg/bytes/#Trim), [`TrimLeft`](/pkg/bytes/#TrimLeft)
và [`TrimRight`](/pkg/bytes/#TrimRight) hiện không cấp phát bộ nhớ và, đặc biệt với
các cutset ASCII nhỏ, nhanh hơn tới 10 lần.

<!-- CL 359485 -->
Hàm [`Title`](/pkg/bytes/#Title) hiện bị deprecated. Nó không
xử lý dấu câu Unicode và các quy tắc viết hoa theo ngôn ngữ, và được thay thế bởi
package [golang.org/x/text/cases](https://golang.org/x/text/cases).

<!-- bytes -->

#### [crypto/elliptic](/pkg/crypto/elliptic/)

<!-- CL 320071, CL 320072, CL 320074, CL 361402, CL 360014 -->
Các triển khai đường cong [`P224`](/pkg/crypto/elliptic#P224),
[`P384`](/pkg/crypto/elliptic#P384) và
[`P521`](/pkg/crypto/elliptic#P521)
hiện tất cả được hỗ trợ bởi code được tạo ra bởi các dự án
[addchain](https://github.com/mmcloughlin/addchain) và
[fiat-crypto](https://github.com/mit-plv/fiat-crypto),
dự án sau dựa trên một mô hình được xác minh chính thức
về các thao tác số học. Chúng hiện sử dụng các công thức đầy đủ an toàn hơn
và các API nội bộ. P-224 và P-384 hiện nhanh hơn khoảng bốn lần.
Tất cả các triển khai đường cong cụ thể hiện có thời gian cố định.

Hoạt động trên các điểm đường cong không hợp lệ (những điểm mà
phương thức `IsOnCurve` trả về false, và không bao giờ được trả về
bởi [`Unmarshal`](/pkg/crypto/elliptic#Unmarshal) hoặc
một phương thức `Curve` hoạt động trên một điểm hợp lệ) luôn là
hành vi không xác định, có thể dẫn đến các cuộc tấn công khôi phục key và hiện
không được hỗ trợ bởi backend mới. Nếu một điểm không hợp lệ được cung cấp cho một
phương thức `P224`, `P384` hoặc `P521`, phương thức đó
hiện sẽ trả về một điểm ngẫu nhiên. Hành vi có thể thay đổi thành
một panic tường minh trong một bản phát hành tương lai.

<!-- crypto/elliptic -->

#### [crypto/tls](/pkg/crypto/tls/)

<!-- CL 325250 -->
Phương thức mới [`Conn.NetConn`](/pkg/crypto/tls/#Conn.NetConn)
cho phép truy cập vào
[`net.Conn`](/pkg/net#Conn) nền.

<!-- crypto/tls -->

#### [crypto/x509](/pkg/crypto/x509)

<!-- CL 353132, CL 353403 -->
[`Certificate.Verify`](/pkg/crypto/x509/#Certificate.Verify)
hiện sử dụng các API nền tảng để xác minh tính hợp lệ của chứng chỉ trên macOS và iOS khi nó
được gọi với [`VerifyOpts.Roots`](/pkg/crypto/x509/#VerifyOpts.Roots) nil
hoặc khi sử dụng pool root được trả về từ
[`SystemCertPool`](/pkg/crypto/x509/#SystemCertPool).

<!-- CL 353589 -->
[`SystemCertPool`](/pkg/crypto/x509/#SystemCertPool)
hiện có sẵn trên Windows.

Trên Windows, macOS và iOS, khi một
[`CertPool`](/pkg/crypto/x509/#CertPool) được trả về bởi
[`SystemCertPool`](/pkg/crypto/x509/#SystemCertPool)
có thêm các chứng chỉ được thêm vào,
[`Certificate.Verify`](/pkg/crypto/x509/#Certificate.Verify)
sẽ thực hiện hai xác minh: một sử dụng các API xác minh nền tảng và
các root hệ thống, và một sử dụng verifier Go và các root bổ sung.
Các chuỗi được trả về bởi các API xác minh nền tảng sẽ được ưu tiên.

[`CertPool.Subjects`](/pkg/crypto/x509/#CertPool.Subjects)
bị deprecated. Trên Windows, macOS và iOS,
[`CertPool`](/pkg/crypto/x509/#CertPool) được trả về bởi
[`SystemCertPool`](/pkg/crypto/x509/#SystemCertPool)
sẽ trả về một pool không bao gồm các root hệ thống trong slice
được trả về bởi `Subjects`, vì danh sách tĩnh không thể đại diện đúng
cho các chính sách nền tảng và có thể không có sẵn từ các
API nền tảng.

Hỗ trợ ký các chứng chỉ bằng các thuật toán chữ ký phụ thuộc vào
hàm băm MD5 (`MD5WithRSA`) có thể bị xóa trong Go 1.19.

#### [debug/dwarf](/pkg/debug/dwarf/)

<!-- CL 380714 -->
Các struct [`StructField`](/pkg/debug/dwarf#StructField)
và [`BasicType`](/pkg/debug/dwarf#BasicType)
giờ đều có trường `DataBitOffset`, giữ
giá trị của thuộc tính `DW_AT_data_bit_offset`
nếu có.

#### [debug/elf](/pkg/debug/elf/)

<!-- CL 352829 -->
Hằng số [`R_PPC64_RELATIVE`](/pkg/debug/elf/#R_PPC64_RELATIVE)
đã được thêm vào.

<!-- debug/elf -->

#### [debug/plan9obj](/pkg/debug/plan9obj/)

<!-- CL 350229 -->
Phương thức [File.Symbols](/pkg/debug/plan9obj#File.Symbols)
hiện trả về giá trị lỗi được xuất ra mới
[ErrNoSymbols](/pkg/debug/plan9obj#ErrNoSymbols)
nếu file không có phần symbol.

<!-- debug/plan9obj -->

#### [embed](/pkg/embed/)

<!-- CL 359413 -->
Một chỉ thị [`go:embed`](/pkg/embed#hdr-Directives)
hiện có thể bắt đầu với `all:` để bao gồm các file
có tên bắt đầu bằng dấu chấm hoặc dấu gạch dưới.

<!-- debug/plan9obj -->

#### [go/ast](/pkg/go/ast/)

<!-- https://golang.org/issue/47781, CL 325689, CL 327149, CL 348375, CL 348609 -->
Theo đề xuất
[Bổ sung vào go/ast và go/token để hỗ trợ các hàm và kiểu được tham số hóa](https://go.googlesource.com/proposal/+/master/design/47781-parameterized-go-ast.md)
các bổ sung sau được thực hiện vào package [`go/ast`](/pkg/go/ast):

  - Các node [`FuncType`](/pkg/go/ast/#FuncType)
    và [`TypeSpec`](/pkg/go/ast/#TypeSpec)
    có trường `TypeParams` mới để giữ các tham số kiểu, nếu có.
  - Node biểu thức mới [`IndexListExpr`](/pkg/go/ast/#IndexListExpr)
    đại diện cho các biểu thức chỉ mục với nhiều chỉ mục, được sử dụng cho các lần khởi tạo hàm và kiểu
    với nhiều hơn một đối số kiểu tường minh.


#### [go/constant](/pkg/go/constant/)

<!-- https://golang.org/issue/46211, CL 320491 -->
Phương thức mới [`Kind.String`](/pkg/go/constant/#Kind.String)
trả về tên đọc được bởi con người cho kind của receiver.

#### [go/token](/pkg/go/token/)

<!-- https://golang.org/issue/47781, CL 324992 -->
Hằng số mới [`TILDE`](/pkg/go/token/#TILDE)
đại diện cho token `~` theo đề xuất
[Bổ sung vào go/ast và go/token để hỗ trợ các hàm và kiểu được tham số hóa](https://go.googlesource.com/proposal/+/master/design/47781-parameterized-go-ast.md).

#### [go/types](/pkg/go/types/)

<!-- https://golang.org/issue/46648 -->
Trường mới [`Config.GoVersion`](/pkg/go/types/#Config.GoVersion)
đặt phiên bản ngôn ngữ Go được chấp nhận.

<!-- https://golang.org/issue/47916 -->
Theo đề xuất
[Bổ sung vào go/types để hỗ trợ các tham số kiểu](https://go.googlesource.com/proposal/+/master/design/47916-parameterized-go-types.md)
các bổ sung sau được thực hiện vào package [`go/types`](/pkg/go/types):

  - Kiểu mới
    [`TypeParam`](/pkg/go/types/#TypeParam), hàm factory
    [`NewTypeParam`](/pkg/go/types/#NewTypeParam)
    và các phương thức liên quan được thêm vào để đại diện cho tham số kiểu.
  - Kiểu mới
    [`TypeParamList`](/pkg/go/types/#TypeParamList) giữ danh sách
    tham số kiểu.
  - Kiểu mới
    [`TypeList`](/pkg/go/types/#TypeList) giữ danh sách các kiểu.
  - Hàm factory mới
    [`NewSignatureType`](/pkg/go/types/#NewSignatureType) cấp phát một
    [`Signature`](/pkg/go/types/#Signature) với
    các tham số kiểu (receiver hoặc hàm).
    Để truy cập các tham số kiểu đó, kiểu `Signature` có hai phương thức mới
    [`Signature.RecvTypeParams`](/pkg/go/types/#Signature.RecvTypeParams) và
    [`Signature.TypeParams`](/pkg/go/types/#Signature.TypeParams).
  - Các kiểu [`Named`](/pkg/go/types/#Named) có bốn phương thức mới:
    [`Named.Origin`](/pkg/go/types/#Named.Origin) để lấy các kiểu tham số hóa ban đầu của các kiểu được khởi tạo,
    [`Named.TypeArgs`](/pkg/go/types/#Named.TypeArgs) và
    [`Named.TypeParams`](/pkg/go/types/#Named.TypeParams) để lấy
    các đối số kiểu hoặc tham số kiểu của kiểu được khởi tạo hoặc tham số hóa, và
    [`Named.SetTypeParams`](/pkg/go/types/#Named.TypeParams) để đặt
    các tham số kiểu (ví dụ, khi import một kiểu có tên nơi mà việc cấp phát kiểu có tên
    và đặt các tham số kiểu không thể được thực hiện đồng thời do các chu kỳ có thể xảy ra).
  - Kiểu [`Interface`](/pkg/go/types/#Interface) có bốn phương thức mới:
    [`Interface.IsComparable`](/pkg/go/types/#Interface.IsComparable) và
    [`Interface.IsMethodSet`](/pkg/go/types/#Interface.IsMethodSet) để
    truy vấn các thuộc tính của tập kiểu được định nghĩa bởi interface, và
    [`Interface.MarkImplicit`](/pkg/go/types/#Interface.MarkImplicit) và
    [`Interface.IsImplicit`](/pkg/go/types/#Interface.IsImplicit) để đặt
    và kiểm tra liệu interface có phải là interface ngầm quanh một type constraint literal hay không.
  - Các kiểu mới
    [`Union`](/pkg/go/types/#Union) và
    [`Term`](/pkg/go/types/#Term), các hàm factory
    [`NewUnion`](/pkg/go/types/#NewUnion) và
    [`NewTerm`](/pkg/go/types/#NewTerm) và các phương thức liên quan được thêm vào để đại diện cho các tập kiểu trong interface.
  - Hàm mới
    [`Instantiate`](/pkg/go/types/#Instantiate)
    khởi tạo một kiểu được tham số hóa.
  - Map [`Info.Instances`](/pkg/go/types/#Info.Instances) mới
    ghi lại các lần khởi tạo hàm và kiểu thông qua kiểu
    [`Instance`](/pkg/go/types/#Instance) mới.
  - <!-- CL 342671 -->
    Kiểu mới [`ArgumentError`](/pkg/go/types/#ArgumentError)
    và các phương thức liên quan được thêm vào để đại diện cho lỗi liên quan đến đối số kiểu.
  - <!-- CL 353089 -->
    Kiểu mới [`Context`](/pkg/go/types/#Context) và hàm factory
    [`NewContext`](/pkg/go/types/#NewContext)
    được thêm vào để tạo điều kiện chia sẻ các instance kiểu giống hệt nhau
    giữa các package đã được kiểm tra kiểu, thông qua trường
    [`Config.Context`](/pkg/go/types/#Config.Context) mới.

Các vị từ
[`AssignableTo`](/pkg/go/types/#AssignableTo),
[`ConvertibleTo`](/pkg/go/types/#ConvertibleTo),
[`Implements`](/pkg/go/types/#Implements),
[`Identical`](/pkg/go/types/#Identical),
[`IdenticalIgnoreTags`](/pkg/go/types/#IdenticalIgnoreTags) và
[`AssertableTo`](/pkg/go/types/#AssertableTo)
hiện cũng hoạt động với các đối số là hoặc chứa các interface tổng quát,
tức là các interface chỉ có thể được sử dụng làm ràng buộc kiểu trong code Go.
Lưu ý rằng hành vi của `AssignableTo`,
`ConvertibleTo`, `Implements` và
`AssertableTo` không xác định với các đối số là
các kiểu generic chưa được khởi tạo, và `AssertableTo` không xác định
nếu đối số đầu tiên là một interface tổng quát.

#### [html/template](/pkg/html/template/)

<!-- CL 321491 -->
Trong pipeline `range`, lệnh mới
`{{break}}` sẽ kết thúc vòng lặp sớm và
lệnh mới `{{continue}}` sẽ ngay lập tức bắt đầu
vòng lặp tiếp theo.

<!-- CL 321490 -->
Hàm `and` không còn luôn luôn đánh giá tất cả các đối số; nó
ngừng đánh giá các đối số sau đối số đầu tiên đánh giá thành
false. Tương tự, hàm `or` hiện ngừng đánh giá
các đối số sau đối số đầu tiên đánh giá thành true. Điều này tạo ra
sự khác biệt nếu bất kỳ đối số nào là một lời gọi hàm.

<!-- html/template -->

#### [image/draw](/pkg/image/draw/)

<!-- CL 340049 -->
Các triển khai dự phòng `Draw` và `DrawMask`
(được sử dụng khi các đối số không phải là các kiểu image phổ biến nhất) hiện
nhanh hơn khi các đối số đó triển khai tùy chọn
[`draw.RGBA64Image`](/pkg/image/draw/#RGBA64Image)
và [`image.RGBA64Image`](/pkg/image/#RGBA64Image)
được thêm vào trong Go 1.17.

<!-- image/draw -->

#### [net](/pkg/net/)

<!-- CL 340261 -->
[`net.Error.Temporary`](/pkg/net#Error) đã bị deprecated.

<!-- net -->

#### [net/http](/pkg/net/http/)

<!-- CL 330852 -->
Trên các mục tiêu WebAssembly, các trường phương thức `Dial`, `DialContext`,
`DialTLS` và `DialTLSContext` trong
[`Transport`](/pkg/net/http/#Transport)
hiện sẽ được sử dụng đúng cách, nếu được chỉ định, để thực hiện các yêu cầu HTTP.

<!-- CL 338590 -->
Phương thức mới
[`Cookie.Valid`](/pkg/net/http#Cookie.Valid)
báo cáo liệu cookie có hợp lệ hay không.

<!-- CL 346569 -->
Hàm mới
[`MaxBytesHandler`](/pkg/net/http#MaxBytesHandler)
tạo một `Handler` bao bọc `ResponseWriter` và
`Request.Body` của nó với một
[`MaxBytesReader`](/pkg/net/http#MaxBytesReader).

<!-- CL 359634, CL 360381, CL 362735 -->
Khi tra cứu tên miền chứa các ký tự không phải ASCII,
việc chuyển đổi Unicode sang ASCII hiện được thực hiện theo
Nontransitional Processing như được định nghĩa trong tiêu chuẩn
[Xử lý tương thích Unicode IDNA](https://unicode.org/reports/tr46/) (UTS #46). Cách diễn giải của bốn rune riêng biệt bị thay đổi: ß, ς,
zero-width joiner U+200D và zero-width non-joiner
U+200C. Nontransitional Processing nhất quán với hầu hết
các ứng dụng và trình duyệt web.

<!-- net/http -->

#### [os/user](/pkg/os/user/)

<!-- CL 330753 -->
[`User.GroupIds`](/pkg/os/user#User.GroupIds)
hiện sử dụng triển khai Go gốc khi cgo không có sẵn.

<!-- os/user -->

#### [reflect](/pkg/reflect/)

<!-- CL 356049, CL 320929 -->
Các phương thức mới
[`Value.SetIterKey`](/pkg/reflect/#Value.SetIterKey)
và [`Value.SetIterValue`](/pkg/reflect/#Value.SetIterValue)
đặt Value bằng cách sử dụng map iterator như nguồn. Chúng tương đương với
`Value.Set(iter.Key())` và `Value.Set(iter.Value())`, nhưng
thực hiện ít cấp phát hơn.

<!-- CL 350691 -->
Phương thức mới
[`Value.UnsafePointer`](/pkg/reflect/#Value.UnsafePointer)
trả về giá trị của Value như một [`unsafe.Pointer`](/pkg/unsafe/#Pointer).
Điều này cho phép người gọi migrate từ [`Value.UnsafeAddr`](/pkg/reflect/#Value.UnsafeAddr)
và [`Value.Pointer`](/pkg/reflect/#Value.Pointer)
để loại bỏ nhu cầu thực hiện chuyển đổi uintptr sang unsafe.Pointer tại callsite (như các quy tắc unsafe.Pointer yêu cầu).

<!-- CL 321891 -->
Phương thức mới
[`MapIter.Reset`](/pkg/reflect/#MapIter.Reset)
thay đổi receiver của nó để lặp qua một
map khác. Việc sử dụng
[`MapIter.Reset`](/pkg/reflect/#MapIter.Reset)
cho phép lặp không cấp phát
qua nhiều map.

<!-- CL 352131 -->
Một số phương thức (
[`Value.CanInt`](/pkg/reflect#Value.CanInt),
[`Value.CanUint`](/pkg/reflect#Value.CanUint),
[`Value.CanFloat`](/pkg/reflect#Value.CanFloat),
[`Value.CanComplex`](/pkg/reflect#Value.CanComplex)
)
đã được thêm vào
[`Value`](/pkg/reflect#Value)
để kiểm tra liệu một chuyển đổi có an toàn không.

<!-- CL 357962 -->
[`Value.FieldByIndexErr`](/pkg/reflect#Value.FieldByIndexErr)
đã được thêm vào để tránh panic xảy ra trong
[`Value.FieldByIndex`](/pkg/reflect#Value.FieldByIndex)
khi bước qua con trỏ nil đến một struct được nhúng.

<!-- CL 341333 -->
[`reflect.Ptr`](/pkg/reflect#Ptr) và
[`reflect.PtrTo`](/pkg/reflect#PtrTo)
đã được đổi tên thành
[`reflect.Pointer`](/pkg/reflect#Pointer) và
[`reflect.PointerTo`](/pkg/reflect#PointerTo),
tương ứng, để nhất quán với phần còn lại của package reflect.
Các tên cũ sẽ tiếp tục hoạt động, nhưng sẽ bị deprecated trong một
bản phát hành Go trong tương lai.

<!-- reflect -->

#### [regexp](/pkg/regexp/)

<!-- CL 354569 -->
[`regexp`](/pkg/regexp/)
hiện coi mỗi byte không hợp lệ của chuỗi UTF-8 là `U+FFFD`.

<!-- regexp -->

#### [runtime/debug](/pkg/runtime/debug/)

<!-- CL 354569 -->
Struct [`BuildInfo`](/pkg/runtime/debug#BuildInfo)
có hai trường mới, chứa thông tin bổ sung
về cách nhị phân được build:

  - [`GoVersion`](/pkg/runtime/debug#BuildInfo.GoVersion)
    giữ phiên bản Go được sử dụng để build nhị phân.
  - [`Settings`](/pkg/runtime/debug#BuildInfo.Settings)
    là một slice của các struct
    [`BuildSettings`](/pkg/runtime/debug#BuildSettings)
    giữ các cặp key/value mô tả build.


<!-- runtime/debug -->

#### [runtime/pprof](/pkg/runtime/pprof/)

<!-- CL 324129 -->
CPU profiler hiện sử dụng timer theo luồng trên Linux. Điều này tăng
mức sử dụng CPU tối đa mà một profile có thể quan sát, và giảm một số dạng
thiên lệch.

<!-- runtime/pprof -->

#### [strconv](/pkg/strconv/)

<!-- CL 343877 -->
[`strconv.Unquote`](/pkg/strconv/#strconv.Unquote)
hiện từ chối các nửa surrogate Unicode.

<!-- strconv -->

#### [strings](/pkg/strings/)

<!-- CL 351710 -->
Hàm mới [`Cut`](/pkg/strings/#Cut)
cắt `string` quanh một dấu phân cách. Nó có thể thay thế
và đơn giản hóa nhiều cách sử dụng phổ biến của
[`Index`](/pkg/strings/#Index),
[`IndexByte`](/pkg/strings/#IndexByte),
[`IndexRune`](/pkg/strings/#IndexRune)
và [`SplitN`](/pkg/strings/#SplitN).

<!-- CL 345849 -->
Hàm mới [`Clone`](/pkg/strings/#Clone) sao chép
`string` đầu vào mà không để `string` được clone được trả về tham chiếu
bộ nhớ của chuỗi đầu vào.

<!-- CL 323318, CL 332771 -->
[`Trim`](/pkg/strings/#Trim), [`TrimLeft`](/pkg/strings/#TrimLeft)
và [`TrimRight`](/pkg/strings/#TrimRight) hiện không cấp phát bộ nhớ và, đặc biệt với
các cutset ASCII nhỏ, nhanh hơn tới 10 lần.

<!-- CL 359485 -->
Hàm [`Title`](/pkg/strings/#Title) hiện bị deprecated. Nó không
xử lý dấu câu Unicode và các quy tắc viết hoa theo ngôn ngữ, và được thay thế bởi
package [golang.org/x/text/cases](https://golang.org/x/text/cases).

<!-- strings -->

#### [sync](/pkg/sync/)

<!-- CL 319769 -->
Các phương thức mới
[`Mutex.TryLock`](/pkg/sync#Mutex.TryLock),
[`RWMutex.TryLock`](/pkg/sync#RWMutex.TryLock) và
[`RWMutex.TryRLock`](/pkg/sync#RWMutex.TryRLock),
sẽ giành được lock nếu nó không đang được giữ.

<!-- sync -->

#### [syscall](/pkg/syscall/)

<!-- CL 336550 -->
Hàm mới [`SyscallN`](/pkg/syscall/?GOOS=windows#SyscallN)
đã được giới thiệu cho Windows, cho phép gọi với số lượng đối số tùy ý. Do đó,
[`Syscall`](/pkg/syscall/?GOOS=windows#Syscall),
[`Syscall6`](/pkg/syscall/?GOOS=windows#Syscall6),
[`Syscall9`](/pkg/syscall/?GOOS=windows#Syscall9),
[`Syscall12`](/pkg/syscall/?GOOS=windows#Syscall12),
[`Syscall15`](/pkg/syscall/?GOOS=windows#Syscall15) và
[`Syscall18`](/pkg/syscall/?GOOS=windows#Syscall18) bị deprecated
để ủng hộ [`SyscallN`](/pkg/syscall/?GOOS=windows#SyscallN).

<!-- CL 355570 -->
[`SysProcAttr.Pdeathsig`](/pkg/syscall/?GOOS=freebsd#SysProcAttr.Pdeathsig)
hiện được hỗ trợ trên FreeBSD.

<!-- syscall -->

#### [syscall/js](/pkg/syscall/js/)

<!-- CL 356430 -->
Interface `Wrapper` đã bị xóa.

<!-- syscall/js -->

#### [testing](/pkg/testing/)

<!-- CL 343883 -->
Ưu tiên của `/` trong đối số cho `-run` và
`-bench` đã được tăng lên. `A/B|C/D` từng được
coi là `A/(B|C)/D` và hiện được coi là
`(A/B)|(C/D)`.

<!-- CL 356669 -->
Nếu tùy chọn `-run` không chọn bất kỳ test nào, tùy chọn
`-count` bị bỏ qua. Điều này có thể thay đổi hành vi của
các test hiện có trong trường hợp khó xảy ra khi một test thay đổi tập hợp các subtest
được chạy mỗi lần hàm test chính nó được chạy.

<!-- CL 251441 -->
Kiểu mới [`testing.F`](/pkg/testing#F)
được sử dụng bởi [hỗ trợ fuzzing mới được mô tả ở trên](#fuzzing). Các test hiện cũng hỗ trợ các tùy chọn dòng lệnh
`-test.fuzz`, `-test.fuzztime` và
`-test.fuzzminimizetime`.

<!-- testing -->

#### [text/template](/pkg/text/template/)

<!-- CL 321491 -->
Trong pipeline `range`, lệnh mới
`{{break}}` sẽ kết thúc vòng lặp sớm và
lệnh mới `{{continue}}` sẽ ngay lập tức bắt đầu
vòng lặp tiếp theo.

<!-- CL 321490 -->
Hàm `and` không còn luôn luôn đánh giá tất cả các đối số; nó
ngừng đánh giá các đối số sau đối số đầu tiên đánh giá thành
false. Tương tự, hàm `or` hiện ngừng đánh giá
các đối số sau đối số đầu tiên đánh giá thành true. Điều này tạo ra
sự khác biệt nếu bất kỳ đối số nào là một lời gọi hàm.

<!-- text/template -->

#### [text/template/parse](/pkg/text/template/parse/)

<!-- CL 321491 -->
Package hỗ trợ lệnh `{{break}}` mới của
[text/template](/pkg/text/template/) và
[html/template](/pkg/html/template/)
thông qua hằng số mới
[`NodeBreak`](/pkg/text/template/parse#NodeBreak)
và kiểu mới
[`BreakNode`](/pkg/text/template/parse#BreakNode),
và tương tự hỗ trợ lệnh `{{continue}}` mới
thông qua hằng số mới
[`NodeContinue`](/pkg/text/template/parse#NodeContinue)
và kiểu mới
[`ContinueNode`](/pkg/text/template/parse#ContinueNode).

<!-- text/template -->

#### [unicode/utf8](/pkg/unicode/utf8/)

<!-- CL 345571 -->
Hàm mới [`AppendRune`](/pkg/unicode/utf8/#AppendRune) thêm mã hóa UTF-8
của một `rune` vào một `[]byte`.

<!-- unicode/utf8 -->
