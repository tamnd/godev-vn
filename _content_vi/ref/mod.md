<!--{
  "Template": true,
  "Title": "Tham chiếu về Go Modules"
}-->
<!-- TODO(golang.org/issue/33637): Viết các bài viết "hướng dẫn" tập trung vào những chủ đề và tác vụ cụ thể về module. Liên kết đến các bài đó thay vì blog, vì blog có thể sẽ không được cập nhật theo thời gian. -->

## Giới thiệu {#introduction}

Module là cách Go quản lý các dependency.

Tài liệu này là sổ tay tham khảo chi tiết về hệ thống module của Go. Để giới thiệu về việc tạo các dự án Go, xem [Cách viết mã Go](/doc/code.html). Để biết thông tin về việc sử dụng module, chuyển đổi dự án sang module và các chủ đề khác, xem chuỗi bài viết trên blog bắt đầu với [Sử dụng Go Modules](/blog/using-go-modules).

## Module, gói và phiên bản {#modules-overview}

Một <dfn>module</dfn> là tập hợp các gói được phát hành, quản lý phiên bản và phân phối cùng nhau. Module có thể được tải xuống trực tiếp từ các kho lưu trữ quản lý phiên bản hoặc từ các máy chủ proxy module.

Một module được xác định bằng [đường dẫn module](#glos-module-path), được khai báo trong [tệp `go.mod`](#go-mod-file), cùng với thông tin về các dependency của module. <dfn>Thư mục gốc của module</dfn> là thư mục chứa tệp `go.mod`. <dfn>Module chính</dfn> là module chứa thư mục nơi lệnh `go` được gọi.

Mỗi <dfn>gói</dfn> trong một module là tập hợp các tệp mã nguồn trong cùng một thư mục được biên dịch cùng nhau. <dfn>Đường dẫn gói</dfn> là đường dẫn module được nối với thư mục con chứa gói đó (tính tương đối với thư mục gốc của module). Ví dụ, module `"golang.org/x/net"` chứa một gói trong thư mục `"html"`. Đường dẫn của gói đó là `"golang.org/x/net/html"`.

### Đường dẫn module {#module-path}

<dfn>Đường dẫn module</dfn> là tên chính thức của một module, được khai báo bằng [chỉ thị `module`](#go-mod-file-module) trong [tệp `go.mod`](#glos-go-mod-file) của module. Đường dẫn của module là tiền tố cho các đường dẫn gói trong module.

Đường dẫn module nên mô tả cả chức năng của module và nơi tìm thấy nó. Thông thường, đường dẫn module bao gồm đường dẫn gốc của kho lưu trữ, một thư mục trong kho lưu trữ (thường là rỗng) và hậu tố phiên bản chính (chỉ dành cho phiên bản chính 2 trở lên).

* <dfn>Đường dẫn gốc của kho lưu trữ</dfn> là phần của đường dẫn module tương ứng với thư mục gốc của kho lưu trữ quản lý phiên bản nơi module được phát triển. Hầu hết module được định nghĩa trong thư mục gốc của kho lưu trữ, vì vậy đây thường là toàn bộ đường dẫn. Ví dụ, `golang.org/x/net` là đường dẫn gốc của kho lưu trữ cho module cùng tên. Xem [Tìm kho lưu trữ cho đường dẫn module](#vcs-find) để biết thông tin về cách lệnh `go` định vị kho lưu trữ bằng các yêu cầu HTTP được tạo từ đường dẫn module.
* Nếu module không được định nghĩa trong thư mục gốc của kho lưu trữ, <dfn>thư mục con của module</dfn> là phần của đường dẫn module đặt tên cho thư mục đó, không bao gồm hậu tố phiên bản chính. Phần này cũng đóng vai trò là tiền tố cho các thẻ phiên bản ngữ nghĩa. Ví dụ, module `golang.org/x/tools/gopls` nằm trong thư mục con `gopls` của kho lưu trữ có đường dẫn gốc `golang.org/x/tools`, vì vậy nó có thư mục con module là `gopls`. Xem [Ánh xạ phiên bản thành commit](#vcs-version) và [Các thư mục module trong một kho lưu trữ](#vcs-dir).
* Nếu module được phát hành ở phiên bản chính 2 trở lên, đường dẫn module phải kết thúc bằng [hậu tố phiên bản chính](#major-version-suffixes) như `/v2`. Hậu tố này có thể có hoặc không là một phần của tên thư mục con. Ví dụ, module có đường dẫn `golang.org/x/repo/sub/v2` có thể nằm trong thư mục con `/sub` hoặc `/sub/v2` của kho lưu trữ `golang.org/x/repo`.

Nếu một module có thể được các module khác sử dụng làm dependency, các quy tắc này phải được tuân theo để lệnh `go` có thể tìm và tải xuống module. Ngoài ra còn có một số [hạn chế về cú pháp](#go-mod-file-ident) đối với các ký tự được phép trong đường dẫn module.

Một module sẽ không bao giờ được tải xuống làm dependency của bất kỳ module nào khác có thể sử dụng bất kỳ đường dẫn gói hợp lệ nào cho đường dẫn module của nó, nhưng phải cẩn thận để không xung đột với các đường dẫn có thể được dependency của module hoặc thư viện chuẩn Go sử dụng. Thư viện chuẩn Go sử dụng các đường dẫn gói không chứa dấu chấm trong phần tử đường dẫn đầu tiên, và lệnh `go` không cố gắng phân giải các đường dẫn như vậy từ các máy chủ mạng. Các đường dẫn `example` và `test` được dành riêng cho người dùng: chúng sẽ không được sử dụng trong thư viện chuẩn và phù hợp để dùng trong các module độc lập, chẳng hạn như các module được định nghĩa trong hướng dẫn hoặc mã ví dụ, hoặc được tạo và thao tác như một phần của bài kiểm thử.

### Các phiên bản {#versions}

Một <dfn>phiên bản</dfn> xác định một ảnh chụp bất biến của một mô-đun, có thể là một [bản phát hành](#glos-release-version) hoặc một [pre-release](#glos-pre-release-version). Mỗi phiên bản bắt đầu bằng chữ cái `v`, theo sau là một phiên bản semantic. Xem [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html) để biết chi tiết về cách các phiên bản được định dạng, diễn giải và so sánh.

Tóm lại, một phiên bản semantic bao gồm ba số nguyên không âm (các phiên bản major, minor và patch, từ trái sang phải) được phân cách bằng dấu chấm. Phiên bản patch có thể được theo sau bởi một chuỗi pre-release tùy chọn bắt đầu bằng dấu gạch nối. Chuỗi pre-release hoặc phiên bản patch có thể được theo sau bởi một chuỗi metadata bản dựng bắt đầu bằng dấu cộng. Ví dụ, `v0.0.0`, `v1.12.134`, `v8.0.5-pre` và `v2.0.9+meta` là các phiên bản hợp lệ.

Mỗi phần của một phiên bản cho biết phiên bản đó có ổn định hay không và có tương thích với các phiên bản trước hay không.

* [Phiên bản major](#glos-major-version) phải được tăng lên và các phiên bản minor cùng patch phải được đặt thành 0 sau khi có thay đổi không tương thích ngược đối với interface công khai của mô-đun hoặc chức năng được ghi chép, ví dụ như sau khi một gói bị xóa.
* [Phiên bản minor](#glos-minor-version) phải được tăng lên và phiên bản patch phải được đặt thành 0 sau một thay đổi tương thích ngược, ví dụ như sau khi thêm một hàm mới.
* [Phiên bản patch](#glos-patch-version) phải được tăng lên sau một thay đổi không ảnh hưởng đến interface công khai của mô-đun, chẳng hạn như sửa lỗi hoặc tối ưu hóa.
* Hậu tố pre-release cho biết một phiên bản là một [pre-release](#glos-pre-release-version). Các phiên bản pre-release được sắp xếp trước các phiên bản phát hành tương ứng. Ví dụ, `v1.2.3-pre` đứng trước `v1.2.3`.
* Hậu tố metadata bản dựng bị bỏ qua khi so sánh các phiên bản. Lệnh go chấp nhận các phiên bản có metadata bản dựng và chuyển đổi chúng thành các pseudo-version để duy trì thứ tự tổng thể giữa các phiên bản.
  * Hậu tố đặc biệt `+incompatible` biểu thị một phiên bản được phát hành trước khi di chuyển sang mô-đun phiên bản major 2 hoặc mới hơn (xem [Compatibility with non-module repositories](#non-module-compat)).
  * Hậu tố đặc biệt `+dirty` được thêm vào thông tin phiên bản của một binary khi nó được xây dựng bằng một bộ công cụ Go 1.24 hoặc mới hơn trong một kho lưu trữ Version Control System (VCS) cục bộ hợp lệ có chứa các thay đổi chưa commit trong thư mục làm việc.

Một phiên bản được xem là không ổn định nếu phiên bản major của nó là 0 hoặc nó có hậu tố pre-release. Các phiên bản không ổn định không chịu các yêu cầu về tính tương thích. Ví dụ, `v0.2.0` có thể không tương thích với `v0.1.0`, và `v1.5.0-beta` có thể không tương thích với `v1.5.0`.

Go có thể truy cập các mô-đun trong hệ thống quản lý phiên bản bằng các tag, nhánh hoặc revision không tuân theo các quy ước này. Tuy nhiên, trong mô-đun chính, lệnh `go` sẽ tự động chuyển đổi các tên revision không tuân theo tiêu chuẩn này thành các phiên bản chuẩn. Lệnh `go` cũng sẽ loại bỏ các hậu tố metadata bản dựng (ngoại trừ `+incompatible`) trong quá trình này. Điều này có thể tạo ra một [pseudo-version](#glos-pseudo-version), một phiên bản pre-release mã hóa một mã định danh revision (chẳng hạn như Git commit hash) và một dấu thời gian từ hệ thống quản lý phiên bản. Ví dụ, lệnh `go get golang.org/x/net@daa7c041` sẽ chuyển đổi commit hash `daa7c041` thành pseudo-version `v0.0.0-20191109021931-daa7c04131f5`. Các phiên bản chuẩn là bắt buộc bên ngoài mô-đun chính, và lệnh `go` sẽ báo lỗi nếu một phiên bản không chuẩn như `master` xuất hiện trong tệp `go.mod`.

### Pseudo-versions {#pseudo-versions}

Một <dfn>pseudo-version</dfn> là một [version](#glos-version) [pre-release](#glos-pre-release-version) được định dạng đặc biệt, mã hóa thông tin về một revision cụ thể trong một kho lưu trữ quản lý phiên bản. Ví dụ, `v0.0.0-20191109021931-daa7c04131f5` là một pseudo-version.

Pseudo-version có thể tham chiếu đến các revision không có [semantic version tags](#glos-semantic-version-tag) khả dụng. Chúng có thể được dùng để kiểm thử các commit trước khi tạo thẻ version, ví dụ trên một nhánh phát triển.

Mỗi pseudo-version có ba phần:

* Tiền tố version cơ sở (`vX.0.0` hoặc `vX.Y.Z-0`), được suy ra từ một thẻ semantic version đứng trước revision hoặc là `vX.0.0` nếu không có thẻ như vậy.
* Dấu thời gian (`yyyymmddhhmmss`), là thời điểm UTC revision được tạo. Trong Git, đây là thời điểm commit, không phải thời điểm của tác giả.
* Mã định danh revision (`abcdefabcdef`), là tiền tố gồm 12 ký tự của mã băm commit, hoặc trong Subversion là số revision được đệm bằng số 0.

Mỗi pseudo-version có thể có một trong ba dạng, tùy thuộc vào version cơ sở. Các dạng này đảm bảo rằng một pseudo-version được so sánh cao hơn version cơ sở của nó, nhưng thấp hơn version được gắn thẻ tiếp theo.

* `vX.0.0-yyyymmddhhmmss-abcdefabcdef` được dùng khi không có version cơ sở đã biết. Giống như mọi version khác, version chính `X` phải khớp với [major version suffix](#glos-major-version-suffix) của module.
* `vX.Y.Z-pre.0.yyyymmddhhmmss-abcdefabcdef` được dùng khi version cơ sở là một pre-release version như `vX.Y.Z-pre`.
* `vX.Y.(Z+1)-0.yyyymmddhhmmss-abcdefabcdef` được dùng khi version cơ sở là một bản phát hành như `vX.Y.Z`. Ví dụ, nếu version cơ sở là `v1.2.3`, một pseudo-version có thể là `v1.2.4-0.20191109021931-daa7c04131f5`.

Nhiều hơn một pseudo-version có thể tham chiếu đến cùng một commit bằng cách sử dụng các version cơ sở khác nhau. Điều này xảy ra tự nhiên khi một version thấp hơn được gắn thẻ sau khi một pseudo-version được tạo.

Các dạng này cung cấp cho pseudo-version hai thuộc tính hữu ích:

* Pseudo-version có version cơ sở đã biết được sắp xếp cao hơn các version đó nhưng thấp hơn các pre-release khác của các version sau.
* Pseudo-version có cùng tiền tố version cơ sở được sắp xếp theo thứ tự thời gian.

Lệnh `go` thực hiện một số kiểm tra để đảm bảo các tác giả module có quyền kiểm soát cách pseudo-version được so sánh với các version khác và pseudo-version tham chiếu đến các revision thực sự thuộc lịch sử commit của module.

* Nếu một version cơ sở được chỉ định, phải có một thẻ semantic version tương ứng là tổ tiên của revision được mô tả bởi pseudo-version. Điều này ngăn các nhà phát triển vượt qua [minimal version selection](#glos-minimal-version-selection) bằng cách sử dụng một pseudo-version được so sánh cao hơn mọi version đã gắn thẻ như `v1.999.999-99999999999999-daa7c04131f5`.
* Dấu thời gian phải khớp với dấu thời gian của revision. Điều này ngăn kẻ tấn công làm tràn [module proxies](#glos-module-proxy) bằng một số lượng không giới hạn các pseudo-version vốn giống nhau về mọi mặt khác. Điều này cũng ngăn người dùng module thay đổi thứ tự tương đối của các version.
* Revision phải là tổ tiên của một trong các nhánh hoặc thẻ của kho lưu trữ module. Điều này ngăn kẻ tấn công tham chiếu đến các thay đổi chưa được phê duyệt hoặc các yêu cầu pull.

Không bao giờ cần phải tự nhập pseudo-version bằng tay. Nhiều lệnh chấp nhận mã băm commit hoặc tên nhánh và sẽ tự động chuyển đổi chúng thành pseudo-version (hoặc version đã gắn thẻ nếu có). Ví dụ:

```
go get example.com/mod@master
go list -m -json example.com/mod@abcd1234
```

### Hậu tố phiên bản chính {#major-version-suffixes}

Bắt đầu từ phiên bản chính 2, đường dẫn module phải có <dfn>hậu tố phiên bản
chính</dfn> như `/v2` khớp với phiên bản chính. Ví dụ, nếu một module có đường dẫn
`example.com/mod` tại `v1.0.0`, thì module đó phải có đường dẫn
`example.com/mod/v2` ở phiên bản `v2.0.0`.

Các hậu tố phiên bản chính triển khai [<dfn>quy tắc tương thích
khi import</dfn>](https://research.swtch.com/vgo-import):

> Nếu một gói cũ và một gói mới có cùng đường dẫn import,
> gói mới phải tương thích ngược với gói cũ.

Theo định nghĩa, các gói trong một phiên bản chính mới của module không tương thích
ngược với các gói tương ứng trong phiên bản chính trước đó. Do đó, bắt đầu từ
`v2`, các gói cần có đường dẫn import mới. Điều này được thực hiện bằng cách thêm
hậu tố phiên bản chính vào đường dẫn module. Vì đường dẫn module là tiền tố của
đường dẫn import cho mỗi gói bên trong module, việc thêm hậu tố phiên bản chính
vào đường dẫn module cung cấp một đường dẫn import riêng biệt cho mỗi phiên bản
không tương thích.

Hậu tố phiên bản chính không được phép ở các phiên bản chính `v0` hoặc `v1`. Không
cần thay đổi đường dẫn module giữa `v0` và `v1` vì các phiên bản `v0` không ổn định
và không có đảm bảo về tính tương thích. Ngoài ra, với hầu hết các module, `v1`
tương thích ngược với phiên bản `v0` cuối cùng; một phiên bản `v1` thể hiện cam
kết về tính tương thích, thay vì chỉ ra các thay đổi không tương thích so với
`v0`.

Một trường hợp đặc biệt là các đường dẫn module bắt đầu bằng `gopkg.in/` phải
luôn có hậu tố phiên bản chính, ngay cả ở `v0` và `v1`. Hậu tố phải bắt đầu bằng
dấu chấm thay vì dấu gạch chéo (ví dụ, `gopkg.in/yaml.v2`).

Các hậu tố phiên bản chính cho phép nhiều phiên bản chính của một module cùng tồn
tại trong một bản build. Điều này có thể cần thiết do [vấn đề dependency
hình kim cương](https://research.swtch.com/vgo-import#dependency_story). Thông
thường, nếu một module được yêu cầu ở hai phiên bản khác nhau bởi các dependency
bắc cầu, phiên bản cao hơn sẽ được sử dụng. Tuy nhiên, nếu hai phiên bản không
tương thích, không phiên bản nào có thể đáp ứng tất cả client. Vì các phiên bản
không tương thích phải có số phiên bản chính khác nhau, chúng cũng phải có đường
dẫn module khác nhau do hậu tố phiên bản chính. Điều này giải quyết xung đột:
các module có hậu tố khác nhau được xem là các module riêng biệt, và các gói của
chúng—ngay cả các gói trong cùng thư mục con tính tương đối so với gốc module—
đều là các gói khác nhau.

Nhiều dự án Go đã phát hành các phiên bản `v2` hoặc cao hơn mà không sử dụng hậu
tố phiên bản chính trước khi chuyển sang module (có thể là trước cả khi module
được giới thiệu). Các phiên bản này được đánh dấu bằng build tag
`+incompatible` (ví dụ, `v2.0.0+incompatible`). Xem [Tính tương thích với các
repository không phải module](#non-module-compat) để biết thêm thông tin.

### Xác định một gói thành một module {#resolve-pkg-mod}

Khi lệnh `go` tải một gói bằng cách sử dụng [đường dẫn
gói](#glos-package-path), nó cần xác định module nào cung cấp gói đó.

Lệnh `go` bắt đầu bằng cách tìm kiếm [danh sách build](#glos-build-list) để
tìm các module có đường dẫn là tiền tố của đường dẫn gói. Ví dụ, nếu gói
`example.com/a/b` được nhập, và module `example.com/a` có trong danh sách
build, lệnh `go` sẽ kiểm tra xem `example.com/a` có chứa gói đó hay không,
trong thư mục `b`. Một thư mục phải có ít nhất một tệp có phần mở rộng `.go`
để được xem là một gói. [Các ràng buộc
build](/pkg/go/build/#hdr-Build_Constraints) không được áp dụng cho mục đích
này. Nếu chính xác một module trong danh sách build cung cấp gói đó, module đó
sẽ được sử dụng. Nếu không có module nào cung cấp gói hoặc có hai hay nhiều
module cung cấp gói, lệnh `go` sẽ báo lỗi. Cờ `-mod=mod` chỉ dẫn lệnh `go` cố
gắng tìm các module mới cung cấp các gói bị thiếu và cập nhật `go.mod` cùng
`go.sum`. Các lệnh [`go get`](#go-get) và [`go mod tidy`](#go-mod-tidy) thực
hiện việc này tự động.

<!-- LƯU Ý(golang.org/issue/27899): lệnh go báo lỗi khi hai
hoặc nhiều module cung cấp một gói có cùng đường dẫn như trên. Trong tương lai,
chúng tôi có thể thử nâng cấp một (hoặc tất cả) các module xung đột.
-->

Khi lệnh `go` tra cứu một module mới cho đường dẫn gói, nó kiểm tra biến môi
trường `GOPROXY`, đây là danh sách các URL proxy được phân tách bằng dấu phẩy
hoặc các từ khóa `direct` hay `off`. Một URL proxy cho biết lệnh `go` nên liên
hệ với một [module proxy](#glos-module-proxy) bằng cách sử dụng [giao thức
`GOPROXY`](#goproxy-protocol). `direct` cho biết lệnh `go` nên
[liên lạc với một hệ thống quản lý phiên bản](#vcs). `off` cho biết không nên
thực hiện liên lạc nào. Các [biến môi trường](#environment-variables)
`GOPRIVATE` và `GONOPROXY` cũng có thể được sử dụng để kiểm soát hành vi này.

Đối với mỗi mục trong danh sách `GOPROXY`, lệnh `go` yêu cầu phiên bản mới nhất
của từng đường dẫn module có thể cung cấp gói (nghĩa là từng tiền tố của đường
dẫn gói). Với mỗi đường dẫn module được yêu cầu thành công, lệnh `go` sẽ tải
module ở phiên bản mới nhất và kiểm tra xem module có chứa gói được yêu cầu
hay không. Nếu một hoặc nhiều module chứa gói được yêu cầu, module có đường
dẫn dài nhất sẽ được sử dụng. Nếu tìm thấy một hoặc nhiều module nhưng không
module nào chứa gói được yêu cầu, lỗi sẽ được báo cáo. Nếu không tìm thấy
module nào, lệnh `go` thử mục tiếp theo trong danh sách `GOPROXY`. Nếu không
còn mục nào, lỗi sẽ được báo cáo.

Ví dụ, giả sử lệnh `go` đang tìm một module cung cấp gói
`golang.org/x/net/html`, và `GOPROXY` được đặt thành
`https://corp.example.com,https://proxy.golang.org`. Lệnh `go` có thể thực hiện
các yêu cầu sau:

* Đến `https://corp.example.com/` (song song):
  * Yêu cầu phiên bản mới nhất của `golang.org/x/net/html`
  * Yêu cầu phiên bản mới nhất của `golang.org/x/net`
  * Yêu cầu phiên bản mới nhất của `golang.org/x`
  * Yêu cầu phiên bản mới nhất của `golang.org`
* Đến `https://proxy.golang.org/`, nếu tất cả các yêu cầu đến `https://corp.example.com/`
  đã thất bại với 404 hoặc 410:
  * Yêu cầu phiên bản mới nhất của `golang.org/x/net/html`
  * Yêu cầu phiên bản mới nhất của `golang.org/x/net`
  * Yêu cầu phiên bản mới nhất của `golang.org/x`
  * Yêu cầu phiên bản mới nhất của `golang.org`

Sau khi tìm thấy một module phù hợp, lệnh `go` sẽ thêm một
[yêu cầu](#go-mod-file-require) mới với đường dẫn và phiên bản của module mới
vào tệp `go.mod` của module chính. Điều này đảm bảo rằng khi cùng một gói được
tải trong tương lai, cùng một module sẽ được sử dụng ở cùng một phiên bản. Nếu
gói đã xác định không được nhập bởi một gói trong module chính, yêu cầu mới sẽ
có một chú thích `// indirect`.

## `go.mod` files {#go-mod-file}

Một module được định nghĩa bằng một tệp văn bản được mã hóa UTF-8 có tên `go.mod` trong thư mục gốc của nó. Tệp `go.mod` có cấu trúc theo dòng. Mỗi dòng chứa một chỉ thị duy nhất, bao gồm một từ khóa theo sau bởi các đối số. Ví dụ:

```
module example.com/my/thing

go 1.23.0

require example.com/other/thing v1.0.2
require example.com/new/thing/v2 v2.3.4
exclude example.com/old/thing v1.2.3
replace example.com/bad/thing v1.4.5 => example.com/good/thing v1.4.5
retract [v1.9.0, v1.9.5]
```

Từ khóa ở đầu có thể được tách ra khỏi các dòng liền kề để tạo thành một khối, tương tự như các lệnh import trong Go.

```
require (
    example.com/new/thing/v2 v2.3.4
    example.com/old/thing v1.2.3
)
```

Tệp `go.mod` được thiết kế để con người có thể đọc và máy có thể ghi. Lệnh `go` cung cấp một số lệnh con thay đổi các tệp `go.mod`. Ví dụ, [`go get`](#go-get) có thể nâng cấp hoặc hạ cấp các dependency cụ thể. Các lệnh tải đồ thị module sẽ [tự động cập nhật](#go-mod-file-updates) `go.mod` khi cần. [`go mod edit`](#go-mod-edit) có thể thực hiện các chỉnh sửa cấp thấp. Gói [`golang.org/x/mod/modfile`](https://pkg.go.dev/golang.org/x/mod/modfile?tab=doc) có thể được các chương trình Go sử dụng để thực hiện các thay đổi tương tự theo cách lập trình.

Một tệp `go.mod` là bắt buộc đối với [module chính](#glos-main-module), và đối với bất kỳ [module thay thế](#go-mod-file-replace) nào được chỉ định bằng đường dẫn tệp cục bộ. Tuy nhiên, một module không có tệp `go.mod` rõ ràng vẫn có thể được [yêu cầu](#go-mod-file-require) làm dependency, hoặc được dùng làm module thay thế được chỉ định bằng đường dẫn module và phiên bản; xem [Tính tương thích với các repository không phải module](#non-module-compat).

### Các phần tử từ vựng {#go-mod-file-lexical}

Khi một tệp `go.mod` được phân tích cú pháp, nội dung của nó được chia thành một chuỗi token. Có một số loại token: khoảng trắng, chú thích, dấu câu, từ khóa, định danh và chuỗi.

*Khoảng trắng* bao gồm dấu cách (U+0020), tab (U+0009), ký tự xuống dòng kiểu carriage return (U+000D) và dòng mới (U+000A). Các ký tự khoảng trắng khác dòng mới không có tác dụng nào ngoài việc phân tách các token vốn sẽ được kết hợp nếu không có chúng. Dòng mới là các token có ý nghĩa.

*Chú thích* bắt đầu bằng `//` và kéo dài đến cuối dòng. Không cho phép chú thích dạng `/* */`.

*Token dấu câu* bao gồm `(`, `)`, và `=>`.

*Từ khóa* phân biệt các loại chỉ thị khác nhau trong tệp `go.mod`. Các từ khóa được phép là `module`, `go`, `require`, `replace`, `exclude`, và `retract`.

*Định danh* là các chuỗi ký tự không phải khoảng trắng, chẳng hạn như đường dẫn module hoặc phiên bản ngữ nghĩa.

*Chuỗi* là các chuỗi ký tự được đặt trong dấu ngoặc. Có hai loại chuỗi: chuỗi được diễn giải bắt đầu và kết thúc bằng dấu ngoặc kép (`"`, U+0022) và chuỗi thô bắt đầu và kết thúc bằng dấu huyền (`<code>&#x60;</code>`, U+0060). Chuỗi được diễn giải có thể chứa các chuỗi thoát bao gồm dấu gạch chéo ngược (`\`, U+005C) theo sau bởi một ký tự khác. Dấu ngoặc kép được thoát (`\"`) không kết thúc một chuỗi được diễn giải. Giá trị không có dấu ngoặc của một chuỗi được diễn giải là chuỗi ký tự nằm giữa các dấu ngoặc kép với mỗi chuỗi thoát được thay thế bằng ký tự đứng sau dấu gạch chéo ngược (ví dụ, `\"` được thay thế bằng `"`, `\n` được thay thế bằng `n`). Ngược lại, giá trị không có dấu ngoặc của một chuỗi thô chỉ đơn giản là chuỗi ký tự nằm giữa các dấu huyền; dấu gạch chéo ngược không có ý nghĩa đặc biệt trong chuỗi thô.

Định danh và chuỗi có thể thay thế lẫn nhau trong ngữ pháp `go.mod`.

### Đường dẫn module và phiên bản {#go-mod-file-ident}

Hầu hết các định danh và chuỗi trong tệp `go.mod` là đường dẫn module hoặc phiên bản.

Đường dẫn module phải đáp ứng các yêu cầu sau:

* Đường dẫn phải bao gồm một hoặc nhiều phần tử đường dẫn được phân tách bằng dấu gạch chéo (`/`, U+002F). Đường dẫn không được bắt đầu hoặc kết thúc bằng dấu gạch chéo.
* Mỗi phần tử đường dẫn là một chuỗi không rỗng được tạo thành từ các chữ cái ASCII, chữ số ASCII và một số dấu câu ASCII giới hạn (`-`, `.`, `_` và `~`).
* Một phần tử đường dẫn không được bắt đầu hoặc kết thúc bằng dấu chấm (`.`, U+002E).
* Tiền tố của phần tử tính đến dấu chấm đầu tiên không được là tên tệp dành riêng trên Windows, không phân biệt chữ hoa chữ thường (`CON`, `com1`, `NuL`, v.v.).
* Tiền tố của phần tử tính đến dấu chấm đầu tiên không được kết thúc bằng dấu ngã theo sau bởi một hoặc nhiều chữ số (như `EXAMPL~1.COM`).

Nếu đường dẫn module xuất hiện trong chỉ thị `require` và không được thay thế, hoặc nếu đường dẫn module xuất hiện ở phía bên phải của chỉ thị `replace`, lệnh `go` có thể cần tải xuống các module có đường dẫn đó, và một số yêu cầu bổ sung phải được đáp ứng.

* Phần tử đường dẫn đứng đầu (đến dấu gạch chéo đầu tiên, nếu có), theo quy ước là tên miền, chỉ được chứa các chữ cái ASCII viết thường, chữ số ASCII, dấu chấm (`.`, U+002E) và dấu gạch ngang (`-`, U+002D); nó phải chứa ít nhất một dấu chấm và không được bắt đầu bằng dấu gạch ngang.
* Đối với phần tử đường dẫn cuối cùng có dạng `/vN` trong đó `N` có dạng số (các chữ số ASCII và dấu chấm), `N` không được bắt đầu bằng số 0 ở đầu, không được là `/v1` và không được chứa bất kỳ dấu chấm nào.
  * Đối với các đường dẫn bắt đầu bằng `gopkg.in/`, yêu cầu này được thay thế bằng yêu cầu đường dẫn phải tuân theo các quy ước của dịch vụ [gopkg.in](https://gopkg.in).

Các phiên bản trong tệp `go.mod` có thể là [chuẩn](`#glos-canonical-version`) hoặc không chuẩn.

Một phiên bản chuẩn bắt đầu bằng chữ cái `v`, theo sau là một phiên bản semantic tuân theo đặc tả [Semantic Versioning 2.0.0](https://semver.org/spec/v2.0.0.html). Xem [Versions](`#versions`) để biết thêm thông tin.

Hầu hết các định danh và chuỗi khác có thể được dùng làm phiên bản không chuẩn, mặc dù có một số hạn chế để tránh các vấn đề với hệ thống tệp, repository và [module proxies](`#glos-module-proxy`). Các phiên bản không chuẩn chỉ được phép trong tệp `go.mod` của module chính. Lệnh `go` sẽ cố gắng thay thế mỗi phiên bản không chuẩn bằng một phiên bản chuẩn tương đương khi tự động [cập nhật](`#go-mod-file-updates`) tệp `go.mod`.

Ở những nơi đường dẫn module được liên kết với một phiên bản (như trong các chỉ thị `require`, `replace` và `exclude`), phần tử đường dẫn cuối cùng phải nhất quán với phiên bản. Xem [hậu tố phiên bản chính](`#major-version-suffixes`).

### Ngữ pháp {#go-mod-file-grammar}

Cú pháp `go.mod` được chỉ định bên dưới bằng Biểu mẫu Backus-Naur mở rộng (EBNF).
Xem [phần Ký hiệu trong Đặc tả Ngôn ngữ Go](/ref/spec#Notation)
để biết chi tiết về cú pháp EBNF.

```
GoMod = { Directive } .
Directive = ModuleDirective |
            GoDirective |
            ToolDirective |
            IgnoreDirective |
            RequireDirective |
            ExcludeDirective |
            ReplaceDirective |
            RetractDirective .
```

Các dòng mới, định danh và chuỗi lần lượt được ký hiệu bằng `newline`, `ident` và
`string`.

Đường dẫn module và phiên bản được ký hiệu bằng `ModulePath` và `Version`.

```
ModulePath = ident | string . /* xem các hạn chế ở trên */
Version = ident | string .    /* xem các hạn chế ở trên */
```

### Chỉ thị `module` {#go-mod-file-module}

Chỉ thị `module` xác định [đường dẫn](#glos-module-path) của module chính. Một
tệp `go.mod` phải chứa chính xác một chỉ thị `module`.

```
ModuleDirective = "module" ( ModulePath | "(" newline ModulePath newline ")" ) newline .
```

Ví dụ:

```
module golang.org/x/net
```

#### Ngừng hỗ trợ {#go-mod-file-module-deprecation}

Một module có thể được đánh dấu là ngừng hỗ trợ trong một khối chú thích chứa
chuỗi `Deprecated:` (phân biệt chữ hoa chữ thường) ở đầu một đoạn văn. Thông báo
ngừng hỗ trợ bắt đầu sau dấu hai chấm và kéo dài đến hết đoạn văn. Các chú thích
có thể xuất hiện ngay trước chỉ thị `module` hoặc sau đó trên cùng một dòng.

Ví dụ:

```
// Deprecated: use example.com/mod/v2 instead.
module example.com/mod
```

Kể từ Go 1.17, [`go list -m -u`](#go-list-m) kiểm tra thông tin về tất cả các
module đã ngừng hỗ trợ trong [danh sách build](#glos-build-list). [`go get`](#go-get)
kiểm tra các module đã ngừng hỗ trợ cần thiết để build các gói được đặt tên trên
dòng lệnh.

Khi lệnh `go` lấy thông tin ngừng hỗ trợ cho một module, nó tải tệp `go.mod` từ
phiên bản khớp với [truy vấn phiên bản](#version-queries) `@latest` mà không xem
xét [việc thu hồi](#go-mod-file-retract) hoặc [loại trừ](#go-mod-file-exclude).
Lệnh `go` tải danh sách các [phiên bản đã thu hồi](#glos-retracted-version) từ
cùng tệp `go.mod`.

Để ngừng hỗ trợ một module, tác giả có thể thêm chú thích `// Deprecated:` và gắn
thẻ một bản phát hành mới. Tác giả có thể thay đổi hoặc xóa thông báo ngừng hỗ
trợ trong một bản phát hành cao hơn.

Việc ngừng hỗ trợ áp dụng cho tất cả các phiên bản minor của một module. Các
phiên bản major cao hơn `v2` được xem là các module riêng biệt cho mục đích này,
vì [hậu tố phiên bản major](#glos-major-version-suffix) của chúng cung cấp cho
chúng các đường dẫn module khác nhau.

Thông báo ngừng hỗ trợ nhằm thông báo cho người dùng rằng module không còn được
hỗ trợ và cung cấp hướng dẫn di chuyển, ví dụ sang phiên bản major mới nhất.
Các phiên bản minor và patch riêng lẻ không thể được đánh dấu ngừng hỗ trợ;
[`retract`](#go-mod-file-retract) có thể phù hợp hơn cho trường hợp đó.

### `go` directive {#go-mod-file-go}

Một chỉ thị `go` cho biết một module được viết với giả định về ngữ nghĩa của một phiên bản Go cụ thể. Phiên bản đó phải là một [phiên bản Go](/doc/toolchain#version) hợp lệ, chẳng hạn như `1.14`, `1.21rc1` hoặc `1.23.0`.

Chỉ thị `go` đặt phiên bản Go tối thiểu cần thiết để sử dụng module này. Trước Go 1.21, chỉ thị này chỉ mang tính khuyến nghị; hiện nay nó là một yêu cầu bắt buộc: các toolchain Go từ chối sử dụng những module khai báo phiên bản Go mới hơn.

Chỉ thị `go` là đầu vào để lựa chọn toolchain Go nào sẽ chạy.
Xem “[các toolchain Go](/doc/toolchain)” để biết chi tiết.

Chỉ thị `go` ảnh hưởng đến việc sử dụng các tính năng ngôn ngữ mới:

* Đối với các gói trong module, trình biên dịch từ chối sử dụng các tính năng ngôn ngữ được giới thiệu sau phiên bản được chỉ định bởi chỉ thị `go`. Ví dụ, nếu một module có chỉ thị `go 1.12`, các gói của module đó không được sử dụng các literal số như `1_000_000`, vốn được giới thiệu trong Go 1.13.
* Nếu một phiên bản Go cũ hơn xây dựng một trong các gói của module và gặp lỗi biên dịch, lỗi đó cho biết module được viết cho một phiên bản Go mới hơn. Ví dụ, giả sử một module có `go 1.13` và một gói sử dụng literal số `1_000_000`. Nếu gói đó được xây dựng bằng Go 1.12, trình biên dịch sẽ cho biết mã được viết cho Go 1.13.

Chỉ thị `go` cũng ảnh hưởng đến hành vi của lệnh `go`:

* Với `go 1.14` trở lên, [vendoring](#vendoring) tự động có thể được bật. Nếu tệp `vendor/modules.txt` tồn tại và nhất quán với `go.mod`, không cần sử dụng rõ ràng cờ `-mod=vendor`.
* Với `go 1.16` trở lên, mẫu gói `all` chỉ khớp với các gói được nhập gián tiếp bởi các gói và bài kiểm thử trong [module chính](#glos-main-module). Đây là cùng tập hợp các gói được giữ lại bởi [`go mod vendor`](#go-mod-vendor) kể từ khi module được giới thiệu. Trong các phiên bản thấp hơn, `all` cũng bao gồm các bài kiểm thử của những gói được nhập bởi các gói trong module chính, các bài kiểm thử của những gói đó, v.v.
* Với `go 1.17` trở lên:
  * Tệp `go.mod` bao gồm một [chỉ thị `require` rõ ràng](#go-mod-file-require) cho mỗi module cung cấp bất kỳ gói nào được nhập gián tiếp bởi một gói hoặc bài kiểm thử trong module chính. (Ở `go 1.16` trở xuống, một [dependency gián tiếp](#glos-direct-dependency) chỉ được đưa vào nếu [lựa chọn phiên bản tối thiểu](#minimal-version-selection) nếu không sẽ chọn một phiên bản khác.) Thông tin bổ sung này cho phép [cắt tỉa đồ thị module](#graph-pruning) và [tải module lười](#lazy-loading).
  * Vì có thể có nhiều `// indirect` dependency hơn đáng kể so với các phiên bản `go` trước đó, các dependency gián tiếp được ghi lại trong một khối riêng trong tệp `go.mod`.
  * `go mod vendor` bỏ qua các tệp `go.mod` và `go.sum` của các dependency được vendor. (Điều đó cho phép các lần gọi lệnh `go` bên trong các thư mục con của `vendor` xác định module chính chính xác.)
  * `go mod vendor` ghi lại phiên bản `go` từ tệp `go.mod` của mỗi dependency vào `vendor/modules.txt`.
* Với `go 1.21` trở lên:
  * Dòng `go` khai báo phiên bản Go tối thiểu bắt buộc để sử dụng với module này.
  * Dòng `go` phải lớn hơn hoặc bằng dòng `go` của tất cả dependency.
  * Lệnh `go` không còn cố gắng duy trì khả năng tương thích với phiên bản Go cũ hơn trước đó.
  * Lệnh `go` cẩn thận hơn trong việc giữ checksum của các tệp `go.mod` trong tệp `go.sum`.
<!-- Nếu bạn cập nhật danh sách này, cũng cập nhật /doc/modules/gomod-ref#go-notes. -->

Một tệp `go.mod` có thể chứa nhiều nhất một chỉ thị `go`. Hầu hết các lệnh sẽ thêm một chỉ thị `go` với phiên bản Go hiện tại nếu chưa có.

Nếu thiếu chỉ thị `go`, `go 1.16` được giả định.

```
GoDirective = "go" GoVersion newline .
GoVersion = string | ident .  /* phiên bản bản phát hành hợp lệ; xem ở trên */
```

Ví dụ:

```
go 1.23.0
```

### `toolchain` directive {#go-mod-file-toolchain}

Chỉ thị `toolchain` khai báo một Go toolchain được đề xuất để sử dụng với một module.
Phiên bản của Go toolchain được đề xuất không được thấp hơn phiên bản Go bắt buộc
được khai báo trong chỉ thị `go`.
Chỉ thị `toolchain`
chỉ có tác dụng khi module là module chính và phiên bản của toolchain mặc định
thấp hơn phiên bản của toolchain được đề xuất.

Để đảm bảo khả năng tái tạo, lệnh `go` ghi tên toolchain của chính nó trong một dòng `toolchain` bất cứ khi nào
nó cập nhật phiên bản `go` trong tệp `go.mod` (thường là trong quá trình `go get`).

Để biết chi tiết, xem “[Go toolchains](/doc/toolchain)”.

```
ToolchainDirective = "toolchain" ToolchainName newline .
ToolchainName = string | ident .  /* tên toolchain hợp lệ; xem “Go toolchains” */
```

Ví dụ:

```
toolchain go1.21.0
```

### `godebug` directive {#go-mod-file-godebug}

Chỉ thị `godebug` khai báo một [thiết lập GODEBUG](/doc/godebug) duy nhất
để áp dụng khi module này là module chính.
Có thể có nhiều hơn một dòng như vậy và chúng có thể được gom nhóm.
Sẽ là lỗi nếu module chính đặt tên một khóa GODEBUG không tồn tại.
Hiệu ứng của `godebug key=value` tương tự như việc mọi gói chính đang được biên dịch
chứa một tệp nguồn liệt kê `//go:debug key=value`.

```
GodebugDirective = "godebug" ( GodebugSpec | "(" newline { GodebugSpec } ")" newline ) .
GodebugSpec = GodebugKey "=" GodebugValue newline.
GodebugKey = GodebugChar { GodebugChar }.
GodebugValue = GodebugChar { GodebugChar }.
GodebugChar = any non-space character except , " ` ' (comma and quotes).
```

Ví dụ:

```
godebug default=go1.21
godebug (
	panicnil=1
	asynctimerchan=0
)
```

### `require` directive {#go-mod-file-require}

Chỉ thị `require` khai báo phiên bản tối thiểu bắt buộc của một module
dependency nhất định. Với mỗi phiên bản module được yêu cầu, lệnh `go` tải
tệp `go.mod` của phiên bản đó và tích hợp các yêu cầu từ
tệp đó. Sau khi tất cả yêu cầu đã được tải, lệnh `go` giải quyết chúng
bằng [minimal version selection (MVS)](#minimal-version-selection) để tạo ra
[build list](#glos-build-list).

Lệnh `go` tự động thêm các chú thích `// indirect` cho một số
yêu cầu. Chú thích `// indirect` cho biết rằng không có gói nào từ
module được yêu cầu được bất kỳ gói nào trong [module
chính](#glos-main-module) nhập trực tiếp.

Nếu [chỉ thị `go`](#go-mod-file-go) chỉ định `go 1.16` hoặc thấp hơn, lệnh `go`
thêm một yêu cầu gián tiếp khi phiên bản được chọn của một module
cao hơn phiên bản đã được ngầm định (theo bắc cầu) bởi các dependency khác của
module chính. Điều này có thể xảy ra do một lần nâng cấp rõ ràng (`go get -u ./...`),
loại bỏ một dependency khác trước đó áp đặt yêu cầu (`go
mod tidy`), hoặc một dependency nhập một gói mà không có yêu cầu tương ứng
trong tệp `go.mod` của chính nó (chẳng hạn một dependency hoàn toàn không có tệp `go.mod`).

Tại `go 1.17` trở lên, lệnh `go` thêm một yêu cầu gián tiếp cho mỗi
module cung cấp bất kỳ gói nào được nhập (ngay cả
[gián tiếp](#glos-indirect-dependency)) bởi một gói hoặc bài kiểm thử trong module chính
hoặc được truyền làm đối số cho `go get`. Các yêu cầu đầy đủ hơn này
cho phép [cắt tỉa đồ thị module](#graph-pruning) và [tải module
lười](#lazy-loading).

```
RequireDirective = "require" ( RequireSpec | "(" newline { RequireSpec } ")" newline ) .
RequireSpec = ModulePath Version newline .
```

Ví dụ:

```
require golang.org/x/net v1.2.3

require (
    golang.org/x/crypto v1.4.5 // indirect
    golang.org/x/text v1.6.7
)
```

### `tool` directive {#go-mod-file-tool}

Kể từ Go 1.24, chỉ thị `tool` thêm một gói làm dependency của module hiện tại. Chỉ thị này cũng
cho phép chạy gói đó bằng `go tool` khi thư mục làm việc hiện tại nằm
trong module này, hoặc trong một workspace chứa module này.

Nếu gói công cụ không nằm trong module hiện tại, phải có chỉ thị `require`
chỉ định phiên bản của công cụ cần sử dụng.

Mẫu meta `tool` phân giải thành danh sách các công cụ được định nghĩa trong
`go.mod` của module hiện tại, hoặc trong chế độ workspace thành hợp của tất cả công cụ được định nghĩa trong tất cả module trong
workspace.

```
ToolDirective = "tool" ( ToolSpec | "(" newline { ToolSpec } ")" newline ) .
ToolSpec = ModulePath newline .
```

Ví dụ:

```
tool golang.org/x/tools/cmd/stringer

tool (
    example.com/module/cmd/a
    example.com/module/cmd/b
)
```

### `ignore` directive {#go-mod-file-ignore}

Chỉ thị `ignore` khiến lệnh go bỏ qua các đường dẫn thư mục được phân tách bằng dấu gạch chéo
và mọi tệp hoặc thư mục được chứa đệ quy trong chúng,
khi khớp các mẫu gói.

Nếu đường dẫn bắt đầu bằng `./`, đường dẫn được hiểu là tương đối so với
thư mục gốc của module, và thư mục đó cùng mọi thư mục hoặc tệp
được chứa đệ quy trong nó sẽ bị bỏ qua khi khớp các mẫu gói.

Nếu không, mọi thư mục có đường dẫn đó ở bất kỳ độ sâu nào trong module, cùng
mọi thư mục hoặc tệp được chứa đệ quy trong chúng sẽ bị bỏ qua.

```
IgnoreDirective = "ignore" ( IgnoreSpec | "(" newline { IgnoreSpec } ")" newline ) .
IgnoreSpec = RelativeFilePath newline .
RelativeFilePath = /* slash-separated relative file path */ .
```

Ví dụ
```
ignore ./node_modules

ignore (
    static
    content/html
    ./third_party/javascript
)
```

### `exclude` directive {#go-mod-file-exclude}

Chỉ thị `exclude` ngăn một phiên bản module được lệnh `go`
tải.

Kể từ Go 1.16, nếu một phiên bản được tham chiếu bởi chỉ thị `require` trong bất kỳ tệp `go.mod`
nào bị loại trừ bởi chỉ thị `exclude` trong tệp `go.mod` của module chính,
yêu cầu đó sẽ bị bỏ qua. Điều này có thể khiến các lệnh như [`go get`](#go-get)
và [`go mod tidy`](#go-mod-tidy) thêm các yêu cầu mới lên các phiên bản cao hơn
vào `go.mod`, với chú thích `// indirect` nếu phù hợp.

Trước Go 1.16, nếu một phiên bản bị loại trừ được tham chiếu bởi chỉ thị `require`,
lệnh `go` liệt kê các phiên bản khả dụng cho module (như được hiển thị với [`go
list -m -versions`](#go-list-m)) và tải phiên bản không bị loại trừ tiếp theo cao hơn
thay thế. Điều này có thể dẫn đến việc lựa chọn phiên bản không xác định, vì
phiên bản cao hơn tiếp theo có thể thay đổi theo thời gian. Cả phiên bản bản phát hành và pre-release
đều được xem xét cho mục đích này, nhưng pseudo-version thì không. Nếu
không có phiên bản cao hơn, lệnh `go` báo cáo lỗi.

Các chỉ thị `exclude` chỉ áp dụng trong tệp `go.mod` của module chính và bị
bỏ qua trong các module khác. Xem [Minimal version
selection](#minimal-version-selection) để biết chi tiết.

```
ExcludeDirective = "exclude" ( ExcludeSpec | "(" newline { ExcludeSpec } ")" newline ) .
ExcludeSpec = ModulePath Version newline .
```

Ví dụ:

```
exclude golang.org/x/net v1.2.3

exclude (
    golang.org/x/crypto v1.4.5
    golang.org/x/text v1.6.7
)
```

### `replace` directive {#go-mod-file-replace}

Chỉ thị `replace` thay thế nội dung của một phiên bản cụ thể của một module,
hoặc tất cả các phiên bản của một module, bằng nội dung được tìm thấy ở nơi
khác. Phần thay thế có thể được chỉ định bằng một module path và phiên bản khác,
hoặc bằng một đường dẫn tệp dành riêng cho nền tảng.

Nếu có phiên bản ở phía bên trái của mũi tên (`=>`), chỉ phiên bản cụ thể đó
của module được thay thế; các phiên bản khác sẽ được truy cập bình thường.
Nếu phiên bản bên trái bị bỏ qua, tất cả các phiên bản của module đều được thay
thế.

Nếu path ở phía bên phải của mũi tên là một đường dẫn tuyệt đối hoặc tương đối
(bắt đầu bằng `./` hoặc `../`), nó được hiểu là đường dẫn tệp cục bộ đến thư mục
gốc của module thay thế, thư mục này phải chứa tệp `go.mod`. Trong trường hợp
này, phiên bản thay thế phải được bỏ qua.

Nếu path ở phía bên phải không phải là đường dẫn cục bộ, nó phải là một module
path hợp lệ. Trong trường hợp này, phiên bản là bắt buộc. Cùng một phiên bản
module không được đồng thời xuất hiện trong danh sách build.

Bất kể việc thay thế được chỉ định bằng đường dẫn cục bộ hay module path, nếu
module thay thế có tệp `go.mod`, chỉ thị `module` của nó phải khớp với module
path mà nó thay thế.

Các chỉ thị `replace` chỉ áp dụng trong tệp `go.mod` của module chính
và bị bỏ qua trong các module khác. Xem [Minimal version
selection](#minimal-version-selection) để biết chi tiết.

Nếu có nhiều module chính, các tệp `go.mod` của tất cả module chính đều được áp
dụng. Các chỉ thị `replace` xung đột giữa các module chính không được phép,
và phải được xóa hoặc ghi đè trong
[một `replace` trong tệp `go.work`](#go-work-file-replace).

Lưu ý rằng chỉ một chỉ thị `replace` không tự thêm module vào [module
graph](#glos-module-graph). Cũng cần có một [chỉ thị
`require`](#go-mod-file-require) tham chiếu đến phiên bản module đã thay thế,
hoặc trong tệp `go.mod` của module chính hoặc trong tệp `go.mod` của một
dependency. Một chỉ thị `replace` không có tác dụng nếu phiên bản module ở
phía bên trái không được yêu cầu.

```
ReplaceDirective = "replace" ( ReplaceSpec | "(" newline { ReplaceSpec } ")" newline ) .
ReplaceSpec = ModulePath [ Version ] "=>" FilePath newline
            | ModulePath [ Version ] "=>" ModulePath Version newline .
FilePath = /* platform-specific relative or absolute file path */
```

Ví dụ:

```
replace golang.org/x/net v1.2.3 => example.com/fork/net v1.4.5

replace (
    golang.org/x/net v1.2.3 => example.com/fork/net v1.4.5
    golang.org/x/net => example.com/fork/net v1.4.5
    golang.org/x/net v1.2.3 => ./fork/net
    golang.org/x/net => ./fork/net
)
```

### `retract` directive {#go-mod-file-retract}

Một chỉ thị `retract` cho biết rằng không nên phụ thuộc vào một phiên bản hoặc một dải phiên bản của module được định nghĩa bởi `go.mod`. Một chỉ thị `retract` hữu ích khi một phiên bản được xuất bản quá sớm hoặc một vấn đề nghiêm trọng được phát hiện sau khi phiên bản đó được xuất bản. Các phiên bản bị thu hồi vẫn nên có sẵn trong các repository quản lý phiên bản và trên [module
proxies](#glos-module-proxy) để đảm bảo rằng các bản dựng phụ thuộc vào chúng không bị hỏng. Từ *retract* được mượn từ tài liệu học thuật: một bài nghiên cứu bị thu hồi vẫn có sẵn, nhưng nó có vấn đề và không nên là cơ sở cho công việc trong tương lai.

Khi một phiên bản module bị thu hồi, người dùng sẽ không tự động nâng cấp lên phiên bản đó bằng cách sử dụng [`go get`](#go-get), [`go mod tidy`](#go-mod-tidy), hoặc các lệnh khác. Các bản dựng phụ thuộc vào những phiên bản bị thu hồi vẫn nên tiếp tục hoạt động, nhưng người dùng sẽ được thông báo về việc thu hồi khi họ kiểm tra bản cập nhật bằng [`go list -m -u`](#go-list-m) hoặc cập nhật một module liên quan bằng [`go get`](#go-get).

Để thu hồi một phiên bản, tác giả module nên thêm một chỉ thị `retract` vào `go.mod`, sau đó xuất bản một phiên bản mới chứa chỉ thị đó. Phiên bản mới phải cao hơn các phiên bản bản phát hành hoặc pre-release khác; nghĩa là truy vấn phiên bản `@latest` [version query](#version-queries) phải phân giải thành phiên bản mới trước khi xem xét các lần thu hồi. Lệnh `go` tải và áp dụng các lần thu hồi từ phiên bản được hiển thị bởi `go list -m -retracted $modpath@latest` (trong đó `$modpath` là đường dẫn module).

Các phiên bản bị thu hồi được ẩn khỏi danh sách phiên bản được in bởi [`go list -m -versions`](#go-list-m) trừ khi sử dụng cờ `-retracted`. Các phiên bản bị thu hồi bị loại trừ khi phân giải các truy vấn phiên bản như `@>=v1.2.3` hoặc `@latest`.

Một phiên bản chứa các lần thu hồi có thể tự thu hồi chính nó. Nếu phiên bản bản phát hành hoặc pre-release cao nhất của một module tự thu hồi chính nó, truy vấn `@latest` sẽ phân giải thành một phiên bản thấp hơn sau khi loại trừ các phiên bản bị thu hồi.

Ví dụ, hãy xem xét trường hợp tác giả của module `example.com/m` vô tình xuất bản phiên bản `v1.0.0`. Để ngăn người dùng nâng cấp lên `v1.0.0`, tác giả có thể thêm hai chỉ thị `retract` vào `go.mod`, sau đó gắn thẻ `v1.0.1` với các lần thu hồi.

```
retract (
    v1.0.0 // Được xuất bản do nhầm lẫn.
    v1.0.1 // Chỉ chứa các lần thu hồi.
)
```

Khi người dùng chạy `go get example.com/m@latest`, lệnh `go` đọc các lần thu hồi từ `v1.0.1`, hiện là phiên bản cao nhất. Cả `v1.0.0` và `v1.0.1` đều bị thu hồi, vì vậy lệnh `go` sẽ nâng cấp (hoặc hạ cấp!) xuống phiên bản cao nhất tiếp theo, có thể là `v0.9.5`.

Các chỉ thị `retract` có thể được viết với một phiên bản duy nhất (như `v1.0.0`) hoặc với một khoảng đóng của các phiên bản có giới hạn trên và giới hạn dưới, được phân cách bằng `[` và `]` (như `[v1.1.0, v1.2.0]`). Một phiên bản duy nhất tương đương với một khoảng mà giới hạn trên và giới hạn dưới giống nhau. Giống như các chỉ thị khác, nhiều chỉ thị `retract` có thể được nhóm lại trong một khối được phân cách bởi `(` ở cuối dòng và `)` trên một dòng riêng.

Mỗi chỉ thị `retract` nên có một chú thích giải thích lý do thu hồi, dù điều này không bắt buộc. Lệnh `go` có thể hiển thị các chú thích lý do trong cảnh báo về các phiên bản bị thu hồi và trong đầu ra của `go list`. Chú thích lý do có thể được viết ngay phía trên một chỉ thị `retract` (không có dòng trống ở giữa) hoặc sau đó trên cùng dòng. Nếu một chú thích xuất hiện phía trên một khối, nó áp dụng cho tất cả các chỉ thị `retract` trong khối không có chú thích riêng. Một chú thích lý do có thể trải dài trên nhiều dòng.

```
RetractDirective = "retract" ( RetractSpec | "(" newline { RetractSpec } ")" newline ) .
RetractSpec = ( Version | "[" Version "," Version "]" ) newline .
```

Ví dụ:

* Thu hồi tất cả phiên bản giữa `v1.0.0` và `v1.9.9`:

```
retract v1.0.0
retract [v1.0.0, v1.9.9]
retract (
    v1.0.0
    [v1.0.0, v1.9.9]
)
```

* Trở về trạng thái không có phiên bản sau khi phát hành sớm một phiên bản `v1.0.0`:

```
retract [v0.0.0, v1.0.1] // giả sử v1.0.1 chứa lần thu hồi này.
```

* Xóa một module bao gồm tất cả pseudo-version và phiên bản được gắn thẻ:

```
retract [v0.0.0-0, v0.15.2]  // giả sử v0.15.2 chứa lần thu hồi này.
```

Chỉ thị `retract` được thêm vào trong Go 1.16. Go 1.15 trở xuống sẽ báo lỗi nếu một chỉ thị `retract` được viết trong tệp `go.mod` của [main
module](#glos-main-module) và sẽ bỏ qua các chỉ thị `retract` trong các tệp `go.mod` của dependency.

### Cập nhật tự động {#go-mod-file-updates}

Hầu hết các lệnh đều báo lỗi nếu `go.mod` thiếu thông tin hoặc không phản ánh chính xác thực tế. Các lệnh [`go get`](#go-get) và
[`go mod tidy`](#go-mod-tidy) có thể được sử dụng để khắc phục phần lớn các vấn đề này. Ngoài ra, cờ `-mod=mod` có thể được dùng với hầu hết các lệnh nhận biết module (`go build`, `go test`, v.v.) để yêu cầu lệnh `go` tự động khắc phục các vấn đề trong `go.mod` và `go.sum`.

Ví dụ, hãy xem xét tệp `go.mod` này:

```
module example.com/M

go 1.23.0

require (
    example.com/A v1
    example.com/B v1.0.0
    example.com/C v1.0.0
    example.com/D v1.2.3
    example.com/E dev
)

exclude example.com/D v1.2.3
```

Bản cập nhật được kích hoạt với `-mod=mod` sẽ viết lại các mã định danh phiên bản không chuẩn về dạng semver [chuẩn](#glos-canonical-version), vì vậy `v1` của `example.com/A` trở thành `v1.0.0`, và `dev` của `example.com/E` trở thành phiên bản giả cho commit mới nhất trên nhánh `dev`, có thể là `v0.0.0-20180523231146-b3f5c0f6e5f1`.

Bản cập nhật sửa đổi các yêu cầu để tuân theo các loại trừ, vì vậy yêu cầu đối với `example.com/D v1.2.3` đã bị loại trừ được cập nhật để sử dụng phiên bản tiếp theo có sẵn của `example.com/D`, có thể là `v1.2.4` hoặc `v1.3.0`.

Bản cập nhật loại bỏ các yêu cầu dư thừa hoặc gây hiểu nhầm. Ví dụ, nếu bản thân `example.com/A v1.0.0` yêu cầu `example.com/B v1.2.0` và `example.com/C v1.0.0`, thì yêu cầu của `go.mod` đối với `example.com/B v1.0.0` là gây hiểu nhầm (đã bị thay thế bởi nhu cầu `v1.2.0` của `example.com/A`), và yêu cầu của nó đối với `example.com/C v1.0.0` là dư thừa (được suy ra từ nhu cầu cùng phiên bản của `example.com/A`), vì vậy cả hai sẽ bị loại bỏ. Nếu module chính chứa các gói nhập trực tiếp các gói từ `example.com/B` hoặc `example.com/C`, thì các yêu cầu sẽ được giữ lại nhưng được cập nhật thành các phiên bản thực tế đang được sử dụng.

Cuối cùng, bản cập nhật định dạng lại `go.mod` theo định dạng chuẩn, để các thay đổi cơ học trong tương lai sẽ tạo ra các diff tối thiểu. Lệnh `go` sẽ không cập nhật `go.mod` nếu chỉ cần thay đổi định dạng.

Vì đồ thị module xác định ý nghĩa của các câu lệnh import, mọi lệnh tải gói cũng sử dụng `go.mod` và do đó có thể cập nhật nó, bao gồm `go build`, `go get`, `go install`, `go list`, `go test`, `go mod tidy`.

Trong Go 1.15 trở xuống, cờ `-mod=mod` được bật theo mặc định, vì vậy các bản cập nhật được thực hiện tự động. Kể từ Go 1.16, lệnh `go` hoạt động như thể `-mod=readonly` được đặt thay thế: nếu cần bất kỳ thay đổi nào đối với `go.mod`, lệnh `go` sẽ báo lỗi và đề xuất cách khắc phục.

## Lựa chọn phiên bản tối thiểu (MVS) {#minimal-version-selection}

Go sử dụng một thuật toán có tên là <dfn>Lựa chọn phiên bản tối thiểu (MVS)</dfn> để chọn một tập hợp các phiên bản module được sử dụng khi xây dựng các gói. MVS được mô tả chi tiết trong [Minimal Version Selection](https://research.swtch.com/vgo-mvs) của Russ Cox.

Về mặt khái niệm, MVS hoạt động trên một đồ thị có hướng của các module, được chỉ định bằng các [tệp `go.mod`](#glos-go-mod-file). Mỗi đỉnh trong đồ thị biểu diễn một phiên bản module. Mỗi cạnh biểu diễn phiên bản tối thiểu được yêu cầu của một dependency, được chỉ định bằng một chỉ thị [`require`](#go-mod-file-require). Đồ thị có thể được sửa đổi bởi các chỉ thị [`exclude`](#go-mod-file-exclude) và [`replace`](#go-mod-file-replace) trong (các) tệp `go.mod` của (các) module chính và bởi các chỉ thị [`replace`](#go-work-file-replace) trong tệp `go.work`.

MVS tạo ra [danh sách build](#glos-build-list) làm đầu ra, là danh sách các phiên bản module được sử dụng cho một bản build.

MVS bắt đầu từ các module chính (các đỉnh đặc biệt trong đồ thị không có phiên bản) và duyệt qua đồ thị, theo dõi phiên bản được yêu cầu cao nhất của từng module. Khi kết thúc quá trình duyệt, các phiên bản được yêu cầu cao nhất tạo thành danh sách build: chúng là các phiên bản tối thiểu đáp ứng mọi yêu cầu.

Có thể kiểm tra danh sách build bằng lệnh [`go list -m all`](#go-list-m). Không giống các hệ thống quản lý dependency khác, danh sách build không được lưu trong tệp "lock". MVS có tính xác định, và danh sách build không thay đổi khi các phiên bản mới của dependency được phát hành, vì vậy MVS được dùng để tính toán danh sách này ở đầu mỗi lệnh nhận biết module.

Hãy xem ví dụ trong sơ đồ bên dưới. Module chính yêu cầu module A ở phiên bản 1.2 trở lên và module B ở phiên bản 1.2 trở lên. A 1.2 và B 1.2 lần lượt yêu cầu C 1.3 và C 1.4. C 1.3 và C 1.4 đều yêu cầu D 1.2.

![Đồ thị phiên bản module với các phiên bản đã truy cập được đánh dấu](/doc/mvs/buildlist.svg "Đồ thị danh sách build của MVS")

MVS truy cập và tải tệp `go.mod` cho từng phiên bản module được đánh dấu màu xanh lam. Khi kết thúc quá trình duyệt đồ thị, MVS trả về một danh sách build chứa các phiên bản được in đậm: A 1.2, B 1.2, C 1.4 và D 1.2. Lưu ý rằng các phiên bản cao hơn của B và D có sẵn nhưng MVS không chọn chúng, vì không có gì yêu cầu chúng.

### Thay thế {#mvs-replace}

Nội dung của một module (bao gồm tệp `go.mod` của nó) có thể được thay thế bằng [chỉ thị `replace`](#go-mod-file-replace) trong tệp `go.mod` của module chính hoặc tệp `go.work` của một workspace. Một chỉ thị `replace` có thể áp dụng cho một phiên bản cụ thể của một module hoặc cho tất cả phiên bản của một module.

Các thay thế làm thay đổi đồ thị module, vì một module thay thế có thể có các dependency khác với các phiên bản bị thay thế.

Xét ví dụ bên dưới, trong đó C 1.4 đã được thay thế bằng R. R phụ thuộc vào D 1.3 thay vì D 1.2, vì vậy MVS trả về danh sách build chứa A 1.2, B 1.2, C 1.4 (được thay thế bằng R) và D 1.3.

![Đồ thị phiên bản module với một thay thế](/doc/mvs/replace.svg "MVS replacement")

### Loại trừ {#mvs-exclude}

Một module cũng có thể bị loại trừ ở các phiên bản cụ thể bằng [chỉ thị `exclude`](#go-mod-file-exclude) trong tệp `go.mod` của module chính.

Các loại trừ cũng làm thay đổi đồ thị module. Khi một phiên bản bị loại trừ, nó sẽ bị xóa khỏi đồ thị module, và các yêu cầu đối với nó được chuyển hướng đến phiên bản cao hơn tiếp theo.

Xét ví dụ bên dưới. C 1.3 đã bị loại trừ. MVS sẽ hoạt động như thể A 1.2 yêu cầu C 1.4 (phiên bản cao hơn tiếp theo) thay vì C 1.3.

![Đồ thị phiên bản module với một loại trừ](/doc/mvs/exclude.svg "MVS exclude")

### Nâng cấp {#mvs-upgrade}

Lệnh [`go get`](#go-get) có thể được dùng để nâng cấp một tập hợp module. Để thực hiện nâng cấp, lệnh `go` thay đổi đồ thị module trước khi chạy MVS bằng cách thêm các cạnh từ các phiên bản đã duyệt đến các phiên bản đã nâng cấp.

Xét ví dụ bên dưới. Module B có thể được nâng cấp từ 1.2 lên 1.3, C có thể được nâng cấp từ 1.3 lên 1.4, và D có thể được nâng cấp từ 1.2 lên 1.3.

![Đồ thị phiên bản module với các nâng cấp](/doc/mvs/upgrade.svg "MVS upgrade")

Các nâng cấp (và hạ cấp) có thể thêm hoặc xóa các dependency gián tiếp. Trong trường hợp này, E 1.1 và F 1.1 xuất hiện trong danh sách build sau khi nâng cấp, vì E 1.1 được B 1.3 yêu cầu.

Để duy trì các nâng cấp, lệnh `go` cập nhật các yêu cầu trong `go.mod`. Nó sẽ thay đổi yêu cầu đối với B thành phiên bản 1.3. Nó cũng sẽ thêm các yêu cầu đối với C 1.4 và D 1.3 với các chú thích `// indirect`, vì nếu không thì các phiên bản đó sẽ không được chọn.

### Hạ cấp {#mvs-downgrade}

Lệnh [`go get`](#go-get) cũng có thể được dùng để hạ cấp một tập hợp module. Để thực hiện việc hạ cấp, lệnh `go` thay đổi đồ thị module bằng cách loại bỏ các phiên bản cao hơn những phiên bản được hạ cấp. Lệnh này cũng loại bỏ các phiên bản của những module khác phụ thuộc vào các phiên bản đã bị loại bỏ, vì chúng có thể không tương thích với các phiên bản đã hạ cấp của dependency của chúng. Nếu module chính yêu cầu một phiên bản module bị loại bỏ do hạ cấp, yêu cầu đó được thay đổi thành một phiên bản trước đó chưa bị loại bỏ. Nếu không có phiên bản trước đó, yêu cầu sẽ bị loại bỏ.

Hãy xem xét ví dụ bên dưới. Giả sử phát hiện một vấn đề với C 1.4, vì vậy chúng ta hạ cấp xuống C 1.3. C 1.4 bị loại bỏ khỏi đồ thị module. B 1.2 cũng bị loại bỏ vì nó yêu cầu C 1.4 trở lên. Yêu cầu của module chính đối với B được thay đổi thành 1.1.

![Đồ thị phiên bản module khi hạ cấp](/doc/mvs/downgrade.svg "MVS hạ cấp")

[`go get`](#go-get) cũng có thể loại bỏ hoàn toàn các dependency bằng cách sử dụng hậu tố `@none` sau một đối số. Cách này hoạt động tương tự như hạ cấp. Tất cả phiên bản của module được chỉ định đều bị loại bỏ khỏi đồ thị module.

## Cắt tỉa đồ thị module {#graph-pruning}

Nếu module chính ở `go 1.17` trở lên, [đồ thị module](#glos-module-graph) được dùng cho [lựa chọn phiên bản tối thiểu](#minimal-version-selection) chỉ bao gồm các yêu cầu _trực tiếp_ đối với từng dependency module chỉ định `go 1.17` trở lên trong tệp `go.mod` của chính nó, trừ khi phiên bản đó của module cũng được một dependency _khác_ tại `go 1.16` trở xuống yêu cầu (theo cách bắc cầu). (Các dependency _bắc cầu_ của các dependency `go 1.17` được _cắt bỏ_ khỏi đồ thị module.)

Vì tệp `go.mod` của `go 1.17` bao gồm một [chỉ thị require](#go-mod-file-require) cho mọi dependency cần thiết để xây dựng bất kỳ package hoặc bài kiểm thử nào trong module đó, đồ thị module đã được cắt tỉa bao gồm tất cả dependency cần thiết để `go build` hoặc `go test` các package trong bất kỳ dependency nào được [module chính](#glos-main-module) yêu cầu rõ ràng. Một module _không_ cần thiết để xây dựng bất kỳ package hoặc bài kiểm thử nào trong một module cụ thể không thể ảnh hưởng đến hành vi thời gian chạy của các package trong module đó, vì vậy các dependency bị cắt khỏi đồ thị module chỉ gây ra sự can thiệp giữa các module vốn không liên quan.

Các module có yêu cầu đã bị cắt tỉa vẫn xuất hiện trong đồ thị module và vẫn được `go list -m all` báo cáo: [phiên bản được chọn](#glos-selected-version) của chúng đã được biết và xác định rõ ràng, và các package có thể được tải từ những module đó (ví dụ, dưới dạng dependency bắc cầu của các bài kiểm thử được tải từ các module khác). Tuy nhiên, vì lệnh `go` không thể dễ dàng xác định dependency nào của các module này được thỏa mãn, các đối số của `go build` và `go test` không thể bao gồm package từ các module có yêu cầu đã bị cắt tỉa. [`go get`](#go-get) nâng module chứa mỗi package được chỉ định thành một dependency rõ ràng, cho phép gọi `go build` hoặc `go test` trên package đó.

Vì Go 1.16 và các phiên bản trước đó không hỗ trợ cắt tỉa đồ thị module, toàn bộ bao đóng bắc cầu của các dependency — bao gồm cả các dependency `go 1.17` bắc cầu — vẫn được bao gồm cho mỗi module chỉ định `go 1.16` trở xuống. (Ở `go 1.16` trở xuống, tệp `go.mod` chỉ bao gồm [dependency trực tiếp](#glos-direct-dependency), vì vậy phải tải một đồ thị lớn hơn nhiều để đảm bảo tất cả dependency gián tiếp được bao gồm.)

Tệp [`go.sum`](#go-sum-files) được ghi bởi [`go mod tidy`](#go-mod-tidy) cho một module theo mặc định bao gồm các checksum cần thiết bởi phiên bản Go _thấp hơn một phiên bản_ so với phiên bản được chỉ định trong [chỉ thị `go`](#go-mod-file-go) của nó. Vì vậy, một module `go 1.17` bao gồm các checksum cần thiết cho toàn bộ đồ thị module được Go 1.16 tải, nhưng một module `go 1.18` sẽ chỉ bao gồm các checksum cần thiết cho đồ thị module đã được cắt tỉa được Go 1.17 tải. Cờ `-compat` có thể được dùng để ghi đè phiên bản mặc định (ví dụ, để cắt tỉa tệp `go.sum` mạnh hơn trong module `go 1.17`).

Xem [tài liệu thiết kế](https://go.googlesource.com/proposal/+/master/design/36460-lazy-module-loading.md) để biết thêm chi tiết.

### Tải module lười {#lazy-loading}

Các yêu cầu toàn diện hơn được bổ sung cho việc cắt tỉa đồ thị module cũng cho phép một tối ưu hóa khác khi làm việc trong một module. Nếu module chính ở mức `go 1.17` trở lên, lệnh `go` tránh tải toàn bộ đồ thị module cho đến khi (và trừ khi) cần đến nó. Thay vào đó, nó chỉ tải tệp `go.mod` của module chính, sau đó cố gắng tải các gói cần được xây dựng chỉ bằng các yêu cầu đó. Nếu không tìm thấy một gói cần import (ví dụ: một dependency của một bài kiểm thử cho một gói bên ngoài module chính) trong các yêu cầu đó, thì phần còn lại của đồ thị module sẽ được tải theo yêu cầu.

Nếu tất cả các gói được import có thể được tìm thấy mà không cần tải đồ thị module, thì lệnh `go` sau đó chỉ tải các tệp `go.mod` của _các_ module chứa những gói đó, và các yêu cầu của chúng được kiểm tra với các yêu cầu của module chính để đảm bảo rằng chúng nhất quán cục bộ. (Sự không nhất quán có thể phát sinh do việc hợp nhất trong kiểm soát phiên bản, chỉnh sửa thủ công và các thay đổi trong các module đã được [thay thế](#go-mod-file-replace) bằng cách sử dụng đường dẫn hệ thống tệp cục bộ.)

## Workspace {#workspaces}

Một <dfn>workspace</dfn> là một tập hợp các module trên đĩa được sử dụng làm các module chính khi chạy [minimal version selection (MVS)](#minimal-version-selection).

Một workspace có thể được khai báo trong [tệp `go.work`](#go-work-file), tệp này chỉ định các đường dẫn tương đối đến thư mục module của từng module trong workspace. Khi không tồn tại tệp `go.work`, workspace bao gồm một module duy nhất chứa thư mục hiện tại.

Hầu hết các lệnh con `go` làm việc với các module hoạt động trên tập hợp các module được xác định bởi workspace hiện tại. `go mod init`, `go mod why`, `go mod edit`, `go mod tidy`, `go mod vendor` và `go get` luôn hoạt động trên một module chính duy nhất.

Một lệnh xác định liệu nó có đang ở trong ngữ cảnh workspace hay không bằng cách trước tiên kiểm tra biến môi trường `GOWORK`. Nếu `GOWORK` được đặt thành `off`, lệnh sẽ ở trong ngữ cảnh một module duy nhất. Nếu biến này rỗng hoặc không được cung cấp, lệnh sẽ tìm kiếm tệp `go.work` trong thư mục làm việc hiện tại, sau đó trong các thư mục cha liên tiếp. Nếu tìm thấy một tệp, lệnh sẽ hoạt động trong workspace mà tệp đó định nghĩa; nếu không, workspace sẽ chỉ bao gồm module chứa thư mục làm việc.
Nếu `GOWORK` đặt tên một đường dẫn đến một tệp hiện có kết thúc bằng .work, chế độ workspace sẽ được bật. Mọi giá trị khác đều là lỗi. Bạn có thể sử dụng lệnh `go env GOWORK` để xác định tệp `go.work` nào mà lệnh `go` đang sử dụng. `go env GOWORK` sẽ rỗng nếu lệnh `go` không ở chế độ workspace.

### `go.work` files {#go-work-file}

Một workspace được xác định bởi một tệp văn bản được mã hóa UTF-8 có tên `go.work`. Tệp
`go.work` có cấu trúc theo dòng. Mỗi dòng chứa một chỉ thị duy nhất, gồm một
từ khóa theo sau bởi các đối số. Ví dụ:

```
go 1.23.0

use ./my/first/thing
use ./my/second/thing

replace example.com/bad/thing v1.4.5 => example.com/good/thing v1.4.5
```

Tương tự như các tệp `go.mod`, một từ khóa ở đầu có thể được tách ra khỏi các dòng
liền kề để tạo thành một khối.

```
use (
    ./my/first/thing
    ./my/second/thing
)
```

Lệnh `go` cung cấp một số lệnh con để thao tác với các tệp `go.work`.
[`go work init`](#go-work-init) tạo các tệp `go.work` mới. [`go work use`](#go-work-use) thêm các thư mục module vào
tệp `go.work`. [`go work edit`](#go-work-edit) thực hiện các chỉnh sửa
cấp thấp. Gói
[`golang.org/x/mod/modfile`](https://pkg.go.dev/golang.org/x/mod/modfile?tab=doc)
có thể được các chương trình Go sử dụng để thực hiện các thay đổi tương tự bằng mã.

Lệnh go sẽ duy trì một tệp `go.work.sum` để theo dõi các hash được workspace sử dụng
nhưng không có trong các tệp go.sum của các module workspace tập hợp.

Thông thường không nên commit các tệp go.work vào các
hệ thống kiểm soát phiên bản, vì hai lý do:

* Một tệp `go.work` được đưa vào kho có thể ghi đè lên tệp `go.work` riêng của nhà phát triển
  từ thư mục cha, gây nhầm lẫn khi các chỉ thị `use` của họ không
  được áp dụng.
* Một tệp `go.work` được đưa vào kho có thể khiến hệ thống tích hợp liên tục (CI)
  chọn và do đó kiểm thử sai các phiên bản dependency của một module. Các hệ thống CI
  nhìn chung không nên được phép sử dụng tệp `go.work` để chúng có thể kiểm thử
  hành vi của module như khi module đó được các module khác yêu cầu,
  trong trường hợp này tệp `go.work` bên trong module không có tác dụng.

Tuy nhiên, có một số trường hợp việc commit một tệp `go.work` là hợp lý.
Ví dụ, khi các module trong một repository được phát triển riêng biệt với
nhau nhưng không cùng với các module bên ngoài, có thể không có lý do để
nhà phát triển muốn sử dụng một tổ hợp module khác trong một workspace.
Trong trường hợp đó, tác giả module nên đảm bảo các module riêng lẻ được kiểm thử
và phát hành đúng cách.

### Các phần tử từ vựng {#go-work-file-lexical}

Các phần tử từ vựng trong tệp `go.work` được định nghĩa hoàn toàn giống như [đối với `go.mod files`](#go-mod-file-lexical).

### Ngữ pháp {#go-work-file-grammar}

Cú pháp `go.work` được chỉ định bên dưới bằng cách sử dụng Dạng Backus-Naur mở rộng (EBNF).  
Xem [phần Ký hiệu trong Đặc tả Ngôn ngữ Go](/ref/spec#Notation) để biết chi tiết về cú pháp EBNF.

```
GoWork = { Directive } .
Directive = GoDirective |
            ToolchainDirective |
            UseDirective |
            ReplaceDirective .
```

Dòng mới, định danh và chuỗi lần lượt được biểu diễn bằng `newline`, `ident` và `string`.

Đường dẫn module và phiên bản được biểu diễn bằng `ModulePath` và `Version`.  
Đường dẫn module và phiên bản được chỉ định hoàn toàn giống như [đối với `go.mod files`](#go-mod-file-lexical).

```
ModulePath = ident | string . /* xem các hạn chế ở trên */
Version = ident | string .    /* xem các hạn chế ở trên */
```

### Chỉ thị `go` {#go-work-file-go}

Chỉ thị `go` là bắt buộc trong một tệp `go.work` hợp lệ. Phiên bản phải là một phiên bản Go bản phát hành hợp lệ: một số nguyên dương theo sau bởi dấu chấm và một số nguyên không âm (ví dụ: `1.18`, `1.19`).

Chỉ thị `go` cho biết phiên bản chuỗi công cụ `go` mà tệp `go.work` được thiết kế để hoạt động cùng. Nếu có thay đổi đối với định dạng tệp `go.work`, các phiên bản tương lai của chuỗi công cụ sẽ diễn giải tệp dựa theo phiên bản được chỉ định.

Một tệp `go.work` có thể chứa nhiều nhất một chỉ thị `go`.

```
GoDirective = "go" GoVersion newline .
GoVersion = string | ident .  /* phiên bản Go bản phát hành hợp lệ; xem ở trên */
```

Ví dụ:

```
go 1.23.0
```

### Chỉ thị `toolchain` {#go-work-file-toolchain}

Chỉ thị `toolchain` khai báo một chuỗi công cụ Go được đề xuất để sử dụng trong một workspace.  
Chỉ thị này chỉ có hiệu lực khi chuỗi công cụ mặc định cũ hơn chuỗi công cụ được đề xuất.

Để biết chi tiết, xem “[Chuỗi công cụ Go](/doc/toolchain)”.

```
ToolchainDirective = "toolchain" ToolchainName newline .
ToolchainName = string | ident .  /* tên chuỗi công cụ hợp lệ; xem “Chuỗi công cụ Go” */
```

Ví dụ:

```
toolchain go1.21.0
```

### `godebug` directive {#go-work-file-godebug}

Chỉ thị `godebug` khai báo một [thiết lập GODEBUG](/doc/godebug) duy nhất
để áp dụng khi làm việc trong workspace này.
Cú pháp và tác động giống với [chỉ thị `godebug` của tệp `go.mod`](/#go-mod-file-godebug).
Khi một workspace đang được sử dụng, các chỉ thị `godebug` trong các tệp `go.mod` sẽ bị bỏ qua.

### `use` directive {#go-work-file-use}

Một `use` thêm một module trên đĩa vào tập hợp các module chính trong một workspace.
Đối số của nó là một đường dẫn tương đối đến thư mục chứa tệp
`go.mod` của module. Chỉ thị `use` không thêm các module nằm trong
các thư mục con của thư mục được chỉ định. Các module đó có thể được thêm bởi
thư mục chứa tệp `go.mod` của chúng trong các chỉ thị `use` riêng biệt.

```
UseDirective = "use" ( UseSpec | "(" newline { UseSpec } ")" newline ) .
UseSpec = FilePath newline .
FilePath = /* platform-specific relative or absolute file path */

```

Ví dụ:

```
use ./mymod  // example.com/mymod

use (
    ../othermod
    ./subdir/thirdmod
)
```

### `replace` directive {#go-work-file-replace}

Tương tự như chỉ thị `replace` trong tệp `go.mod`, chỉ thị `replace` trong
tệp `go.work` thay thế nội dung của một phiên bản cụ thể của một module,
hoặc tất cả các phiên bản của một module, bằng nội dung được tìm thấy ở nơi khác. Một thay thế
ký tự đại diện trong `go.work` ghi đè lên một `replace` dành riêng cho phiên bản trong tệp `go.mod`.

Các chỉ thị `replace` trong các tệp `go.work` ghi đè mọi thay thế của cùng
module hoặc phiên bản module trong các module thuộc workspace.

```
ReplaceDirective = "replace" ( ReplaceSpec | "(" newline { ReplaceSpec } ")" newline ) .
ReplaceSpec = ModulePath [ Version ] "=>" FilePath newline
            | ModulePath [ Version ] "=>" ModulePath Version newline .
FilePath = /* platform-specific relative or absolute file path */
```

Ví dụ:

```
replace golang.org/x/net v1.2.3 => example.com/fork/net v1.4.5

replace (
    golang.org/x/net v1.2.3 => example.com/fork/net v1.4.5
    golang.org/x/net => example.com/fork/net v1.4.5
    golang.org/x/net v1.2.3 => ./fork/net
    golang.org/x/net => ./fork/net
)
```

## Khả năng tương thích với các kho lưu trữ không sử dụng module {#non-module-compat}

Để đảm bảo quá trình chuyển đổi suôn sẻ từ `GOPATH` sang module, lệnh `go` có thể tải xuống và xây dựng các gói ở chế độ nhận biết module từ các kho lưu trữ chưa di chuyển sang module bằng cách thêm một [tệp `go.mod`](#glos-go-mod-file).

Khi lệnh `go` tải xuống một module ở một phiên bản cụ thể [trực tiếp](#vcs) từ một kho lưu trữ, nó tìm URL kho lưu trữ cho đường dẫn module, ánh xạ phiên bản tới một revision trong kho lưu trữ, sau đó trích xuất một bản lưu trữ của kho lưu trữ tại revision đó. Nếu [đường dẫn của module](#glos-module-path) bằng với [đường dẫn gốc của kho lưu trữ](#glos-repository-root-path), và thư mục gốc của kho lưu trữ không chứa tệp `go.mod`, lệnh `go` tạo tổng hợp một tệp `go.mod` trong bộ nhớ đệm module chứa [chỉ thị `module`](#go-mod-file-module) và không có gì khác. Vì các tệp `go.mod` tổng hợp không chứa [các chỉ thị `require`](#go-mod-file-require) cho các dependency của chúng, các module khác phụ thuộc vào chúng có thể cần thêm các chỉ thị `require` (với các chú thích `// indirect`) để đảm bảo mỗi dependency được tải ở cùng một phiên bản trong mọi lần xây dựng.

Khi lệnh `go` tải xuống một module từ một [proxy](#communicating-with-proxies), nó tải tệp `go.mod` riêng biệt với phần nội dung còn lại của module. Proxy được kỳ vọng sẽ cung cấp một tệp `go.mod` tổng hợp nếu module ban đầu không có tệp này.

### Các phiên bản `+incompatible` {#incompatible-versions}

Một module được phát hành ở phiên bản major 2 trở lên phải có [hậu tố phiên bản major](#major-version-suffixes) tương ứng trên đường dẫn module của nó. Ví dụ, nếu một module được phát hành ở `v2.0.0`, đường dẫn của nó phải có hậu tố `/v2`. Điều này cho phép lệnh `go` xem nhiều phiên bản major của một dự án là các module riêng biệt, ngay cả khi chúng được phát triển trong cùng một kho lưu trữ.

Yêu cầu về hậu tố phiên bản major được đưa ra khi hỗ trợ module được thêm vào lệnh `go`, và nhiều kho lưu trữ đã gắn thẻ các bản phát hành với phiên bản major `2` hoặc cao hơn trước thời điểm đó. Để duy trì khả năng tương thích với các kho lưu trữ này, lệnh `go` thêm hậu tố `+incompatible` vào các phiên bản có phiên bản major 2 trở lên mà không có tệp `go.mod`. `+incompatible` cho biết một phiên bản là một phần của cùng module với các phiên bản có số phiên bản major thấp hơn; do đó, lệnh `go` có thể tự động nâng cấp lên các phiên bản `+incompatible` cao hơn mặc dù điều đó có thể làm hỏng quá trình xây dựng.

Xem xét yêu cầu ví dụ dưới đây:

```
require example.com/m v4.1.2+incompatible
```

Phiên bản `v4.1.2+incompatible` tham chiếu đến [thẻ phiên bản ngữ nghĩa](#glos-semantic-version-tag) `v4.1.2` trong kho lưu trữ cung cấp module `example.com/m`. Module phải nằm trong thư mục gốc của kho lưu trữ (nghĩa là [đường dẫn gốc của kho lưu trữ](#glos-module-path) cũng phải là `example.com/m`), và không được có tệp `go.mod`. Module có thể có các phiên bản với số phiên bản major thấp hơn như `v1.5.2`, và lệnh `go` có thể tự động nâng cấp lên `v4.1.2+incompatible` từ các phiên bản đó (xem [lựa chọn phiên bản tối thiểu (MVS)](#minimal-version-selection) để biết thông tin về cách hoạt động của việc nâng cấp).

Một kho lưu trữ di chuyển sang module sau khi phiên bản `v2.0.0` được gắn thẻ thường nên phát hành một phiên bản major mới. Trong ví dụ trên, tác giả nên tạo một module với đường dẫn `example.com/m/v5` và nên phát hành phiên bản `v5.0.0`. Tác giả cũng nên cập nhật các import của các gói trong module để sử dụng tiền tố `example.com/m/v5` thay vì `example.com/m`. Xem [Go Modules: v2 and Beyond](/blog/v2-go-modules) để có ví dụ chi tiết hơn.

Lưu ý rằng hậu tố `+incompatible` không nên xuất hiện trên một thẻ trong kho lưu trữ; một thẻ như `v4.1.2+incompatible` sẽ bị bỏ qua. Hậu tố này chỉ xuất hiện trong các phiên bản được lệnh `go` sử dụng. Xem [Ánh xạ phiên bản tới các commit](#vcs-version) để biết chi tiết về sự khác biệt giữa phiên bản và thẻ.

Cũng lưu ý rằng hậu tố `+incompatible` có thể xuất hiện trên [các phiên bản giả](#glos-pseudo-version). Ví dụ, `v2.0.1-20200722182040-012345abcdef+incompatible` có thể là một phiên bản giả hợp lệ.

### Tương thích module tối thiểu {#minimal-module-compatibility}

Một module được phát hành ở phiên bản chính 2 trở lên bắt buộc phải có [hậu tố phiên bản chính](#glos-major-version-suffix) trong [đường dẫn module](#glos-module-path) của nó. Module có thể được phát triển hoặc không được phát triển trong một [thư mục con phiên bản chính](#glos-major-version-subdirectory) bên trong kho lưu trữ của nó. Điều này ảnh hưởng đến các gói nhập các gói bên trong module khi xây dựng ở chế độ `GOPATH`.

Thông thường trong chế độ `GOPATH`, một gói được lưu trong một thư mục khớp với [đường dẫn gốc của kho lưu trữ](#glos-repository-root-path) của nó, nối với thư mục của gói đó bên trong kho lưu trữ. Ví dụ: một gói trong kho lưu trữ có đường dẫn gốc `example.com/repo` ở thư mục con `sub` sẽ được lưu trong `$GOPATH/src/example.com/repo/sub` và được nhập bằng `example.com/repo/sub`.

Đối với một module có hậu tố phiên bản chính, có thể mong đợi tìm thấy gói `example.com/repo/v2/sub` trong thư mục `$GOPATH/src/example.com/repo/v2/sub`. Điều này yêu cầu module được phát triển trong thư mục con `v2` của kho lưu trữ. Lệnh `go` hỗ trợ điều này nhưng không yêu cầu như vậy (xem [Ánh xạ phiên bản tới các commit](#vcs-version)).

Nếu một module *không* được phát triển trong thư mục con phiên bản chính, thì thư mục của nó trong `GOPATH` sẽ không chứa hậu tố phiên bản chính, và các gói của nó có thể được nhập mà không có hậu tố phiên bản chính. Trong ví dụ trên, gói sẽ được tìm thấy trong thư mục `$GOPATH/src/example.com/repo/sub` và được nhập bằng `example.com/repo/sub`.

Điều này tạo ra một vấn đề cho các gói được dự định xây dựng trong cả chế độ module và chế độ `GOPATH`: chế độ module yêu cầu hậu tố, trong khi chế độ `GOPATH` thì không.

Để khắc phục điều này, <dfn>tương thích module tối thiểu</dfn> đã được thêm vào Go 1.11 và được backport về Go 1.9.7 và 1.10.3. Khi đường dẫn nhập được phân giải thành một thư mục trong chế độ `GOPATH`:

* Khi phân giải một import có dạng `$modpath/$vn/$dir` trong đó:
  * `$modpath` là một đường dẫn module hợp lệ,
  * `$vn` là một hậu tố phiên bản chính,
  * `$dir` là một thư mục con có thể rỗng,
* Nếu tất cả các điều kiện sau đều đúng:
  * Gói `$modpath/$vn/$dir` không có trong bất kỳ [thư mục `vendor`](#glos-vendor-directory) liên quan nào.
  * Tệp `go.mod` có trong cùng thư mục với tệp đang nhập hoặc trong bất kỳ thư mục cha nào cho đến thư mục gốc `$GOPATH/src`,
  * Không tồn tại thư mục `$GOPATH[i]/src/$modpath/$vn/$suffix` (với bất kỳ thư mục gốc `$GOPATH[i]` nào),
  * Tệp `$GOPATH[d]/src/$modpath/go.mod` tồn tại (với một thư mục gốc `$GOPATH[d]` nào đó) và khai báo đường dẫn module là `$modpath/$vn`,
* Khi đó import `$modpath/$vn/$dir` được phân giải thành thư mục `$GOPATH[d]/src/$modpath/$dir`.

Quy tắc này cho phép các gói đã được chuyển sang module nhập các gói khác đã được chuyển sang module khi được xây dựng trong chế độ `GOPATH`, ngay cả khi không sử dụng thư mục con phiên bản chính.

## Các lệnh nhận biết module {#mod-commands}

Hầu hết các lệnh `go` có thể chạy ở *chế độ nhận biết module* hoặc *chế độ `GOPATH`*. Ở chế độ nhận biết module, lệnh `go` sử dụng các tệp `go.mod` để tìm các dependency có phiên bản, và thường tải các gói từ [bộ nhớ đệm module](#glos-module-cache), tải xuống các module nếu chúng chưa có. Ở chế độ `GOPATH`, lệnh `go` bỏ qua các module; nó tìm dependency trong các [thư mục `vendor`](#glos-vendor-directory) và trong `GOPATH`.

Kể từ Go 1.16, chế độ nhận biết module được bật theo mặc định, bất kể có tệp `go.mod` hay không. Ở các phiên bản thấp hơn, chế độ nhận biết module được bật khi có tệp `go.mod` trong thư mục hiện tại hoặc bất kỳ thư mục cha nào.

Chế độ nhận biết module có thể được điều khiển bằng biến môi trường `GO111MODULE`, có thể được đặt thành `on`, `off` hoặc `auto`.

* Nếu `GO111MODULE=off`, lệnh `go` bỏ qua các tệp `go.mod` và chạy ở chế độ `GOPATH`.
* Nếu `GO111MODULE=on` hoặc không được đặt, lệnh `go` chạy ở chế độ nhận biết module, ngay cả khi không có tệp `go.mod`. Không phải tất cả lệnh đều hoạt động khi không có tệp `go.mod`: xem [Các lệnh module bên ngoài một module](#commands-outside).
* Nếu `GO111MODULE=auto`, lệnh `go` chạy ở chế độ nhận biết module nếu có tệp `go.mod` trong thư mục hiện tại hoặc bất kỳ thư mục cha nào. Trong Go 1.15 trở xuống, đây là hành vi mặc định. Các lệnh con `go mod` và `go install` với [truy vấn phiên bản](#version-queries) chạy ở chế độ nhận biết module ngay cả khi không có tệp `go.mod`.

Ở chế độ nhận biết module, `GOPATH` không còn xác định ý nghĩa của các import trong quá trình build, nhưng nó vẫn lưu các dependency đã tải xuống (trong `GOPATH/pkg/mod`; xem [Bộ nhớ đệm module](#module-cache)) và các lệnh đã cài đặt (trong `GOPATH/bin`, trừ khi `GOBIN` được đặt).

### Các lệnh build {#build-commands}

Tất cả các lệnh tải thông tin về gói đều nhận biết module. Bao gồm:

* `go build`
* `go fix`
* `go generate`
* `go install`
* `go list`
* `go run`
* `go test`
* `go vet`

Khi chạy ở chế độ nhận biết module, các lệnh này sử dụng các tệp `go.mod` để diễn giải đường dẫn import được liệt kê trên dòng lệnh hoặc được viết trong các tệp mã nguồn Go. Các lệnh này chấp nhận các cờ sau, dùng chung cho tất cả các lệnh module.

* Cờ `-mod` điều khiển việc `go.mod` có thể được tự động cập nhật hay không và thư mục `vendor` có được sử dụng hay không.
  * `-mod=mod` yêu cầu lệnh `go` bỏ qua thư mục vendor và [tự động cập nhật](#go-mod-file-updates) `go.mod`, ví dụ khi một gói được import không được cung cấp bởi bất kỳ module đã biết nào.
  * `-mod=readonly` yêu cầu lệnh `go` bỏ qua thư mục `vendor` và báo lỗi nếu `go.mod` cần được cập nhật.
  * `-mod=vendor` yêu cầu lệnh `go` sử dụng thư mục `vendor`. Ở chế độ này, lệnh `go` sẽ không sử dụng mạng hoặc bộ nhớ đệm module.
  * Theo mặc định, nếu [phiên bản `go`](#go-mod-file-go) trong `go.mod` là `1.14` trở lên và có thư mục `vendor`, lệnh `go` hoạt động như thể `-mod=vendor` được sử dụng. Nếu không, lệnh `go` hoạt động như thể `-mod=readonly` được sử dụng.
  * `go get` từ chối cờ này vì mục đích của lệnh là sửa đổi dependency, việc này chỉ được cho phép bởi `-mod=mod`.
* Cờ `-modcacherw` yêu cầu lệnh `go` tạo các thư mục mới trong bộ nhớ đệm module với quyền đọc-ghi thay vì chỉ đọc. Khi cờ này được sử dụng nhất quán (thường bằng cách đặt `GOFLAGS=-modcacherw` trong môi trường hoặc chạy `go env -w GOFLAGS=-modcacherw`), bộ nhớ đệm module có thể được xóa bằng các lệnh như `rm -r` mà không cần thay đổi quyền trước. Lệnh [`go clean -modcache`](#go-clean-modcache) có thể được dùng để xóa bộ nhớ đệm module, bất kể có sử dụng `-modcacherw` hay không.
* Cờ `-modfile=file.mod` yêu cầu lệnh `go` đọc (và có thể ghi) một tệp thay thế thay cho `go.mod` trong thư mục gốc của module. Tên tệp phải kết thúc bằng `.mod`. Vẫn phải có một tệp tên là `go.mod` để xác định thư mục gốc của module, nhưng tệp này không được truy cập. Khi `-modfile` được chỉ định, một tệp `go.sum` thay thế cũng được sử dụng: đường dẫn của nó được suy ra từ cờ `-modfile` bằng cách loại bỏ phần mở rộng `.mod` và thêm `.sum`.

### Vendoring {#vendoring}

Khi sử dụng module, lệnh `go` thường thỏa mãn các dependency bằng cách tải xuống các module từ nguồn của chúng vào bộ nhớ đệm module, sau đó tải các gói từ các bản sao đã tải xuống đó. <dfn>Vendoring</dfn> có thể được sử dụng để cho phép tương tác với các phiên bản Go cũ hơn, hoặc để đảm bảo rằng tất cả tệp được sử dụng cho một bản build được lưu trữ trong một cây tệp duy nhất.

Lệnh [`go mod vendor`](#go-mod-vendor) tạo một thư mục có tên `vendor` trong thư mục gốc của [module chính](#glos-main-module), chứa các bản sao của tất cả gói cần thiết để build và kiểm thử các gói trong module chính. Các gói chỉ được import bởi các bài kiểm thử của gói bên ngoài module chính sẽ không được đưa vào. Giống như [`go mod tidy`](#go-mod-tidy) và các lệnh module khác, [ràng buộc build](#glos-build-constraint) ngoại trừ `ignore` không được xem xét khi tạo thư mục `vendor`.

`go mod vendor` cũng tạo tệp `vendor/modules.txt`, chứa danh sách các gói được vendoring và các phiên bản module mà chúng được sao chép từ đó. Khi vendoring được bật, tệp kê khai này được sử dụng làm nguồn thông tin phiên bản module, như được báo cáo bởi [`go list -m`](#go-list-m) và [`go version -m`](#go-version-m). Khi lệnh `go` đọc `vendor/modules.txt`, lệnh này kiểm tra rằng các phiên bản module nhất quán với `go.mod`. Nếu `go.mod` đã thay đổi kể từ khi `vendor/modules.txt` được tạo, lệnh `go` sẽ báo lỗi. Cần chạy lại `go mod vendor` để cập nhật thư mục `vendor`.

Nếu thư mục `vendor` tồn tại trong thư mục gốc của module chính, thư mục này sẽ được tự động sử dụng nếu [phiên bản `go`](#go-mod-file-go) trong [tệp `go.mod`](#glos-go-mod-file) của module chính là `1.14` hoặc cao hơn. Để bật vendoring một cách rõ ràng, hãy gọi lệnh `go` với cờ `-mod=vendor`. Để tắt vendoring, hãy sử dụng cờ `-mod=readonly` hoặc `-mod=mod`.

Khi vendoring được bật, các [lệnh build](#build-commands) như `go build` và `go test` tải các gói từ thư mục `vendor` thay vì truy cập mạng hoặc bộ nhớ đệm module cục bộ. Lệnh [`go list -m`](#go-list-m) chỉ in thông tin về các module được liệt kê trong `go.mod`. Các lệnh `go mod` như [`go mod download`](#go-mod-download) và [`go mod tidy`](#go-mod-tidy) không hoạt động khác đi khi vendoring được bật và vẫn sẽ tải xuống module cũng như truy cập bộ nhớ đệm module. [`go get`](#go-get) cũng không hoạt động khác đi khi vendoring được bật.

Khác với [vendoring trong chế độ `GOPATH` mode](/s/go15vendor), lệnh `go` bỏ qua các thư mục vendor ở những vị trí khác ngoài thư mục gốc của module chính. Ngoài ra, vì các thư mục vendor trong các module khác không được sử dụng, lệnh `go` không đưa các thư mục vendor vào khi xây dựng [tệp zip module](#zip-files) (nhưng xem các lỗi đã biết [#31562](/issue/31562) và [#37397](/issue/37397)).

### `go get` {#go-get}

Cách dùng:

```
go get [-d] [-t] [-u] [-tool] [build flags] [packages]
```

Ví dụ:

```
# Nâng cấp một module cụ thể.
$ go get golang.org/x/net

# Nâng cấp các module cung cấp gói được nhập bởi các gói trong module chính.
$ go get -u ./...

# Nâng cấp hoặc hạ cấp xuống một phiên bản cụ thể của một module.
$ go get golang.org/x/text@v0.3.2

# Cập nhật đến commit trên nhánh master của module.
$ go get golang.org/x/text@master

# Xóa dependency đến một module và hạ cấp các module yêu cầu module đó
# xuống các phiên bản không yêu cầu module đó.
$ go get golang.org/x/text@none

# Nâng cấp phiên bản Go tối thiểu được yêu cầu cho module chính.
$ go get go

# Nâng cấp toolchain Go được đề xuất, giữ nguyên phiên bản Go tối thiểu.
$ go get toolchain

# Nâng cấp lên bản phát hành vá mới nhất của toolchain Go được đề xuất.
$ go get toolchain@patch
```

Lệnh `go get` cập nhật các dependency của module trong [tệp `go.mod`](#go-mod-file) của [module chính](#glos-main-module), sau đó xây dựng và cài đặt các gói được liệt kê trên dòng lệnh.

Bước đầu tiên là xác định những module nào cần được cập nhật. `go get` nhận một danh sách các gói, mẫu gói và đường dẫn module làm đối số. Nếu một đối số gói được chỉ định, `go get` cập nhật module cung cấp gói đó. Nếu một mẫu gói được chỉ định (ví dụ: `all` hoặc một đường dẫn có ký tự đại diện `...`), `go get` mở rộng mẫu thành một tập hợp các gói, sau đó cập nhật các module cung cấp các gói đó. Nếu một đối số đặt tên một module nhưng không phải một gói (ví dụ: module `golang.org/x/net` không có gói nào trong thư mục gốc), `go get` sẽ cập nhật module nhưng không xây dựng một gói. Nếu không có đối số nào được chỉ định, `go get` hoạt động như thể `.` đã được chỉ định (gói trong thư mục hiện tại); cách này có thể được dùng cùng cờ `-u` để cập nhật các module cung cấp các gói đã nhập.

Mỗi đối số có thể bao gồm một <dfn>hậu tố truy vấn phiên bản</dfn> cho biết phiên bản mong muốn, như trong `go get golang.org/x/text@v0.3.0`. Hậu tố truy vấn phiên bản bao gồm một ký hiệu `@` theo sau bởi một [truy vấn phiên bản](#version-queries), có thể chỉ định một phiên bản cụ thể (`v0.3.0`), tiền tố phiên bản (`v0.3`), tên nhánh hoặc thẻ (`master`), một bản sửa đổi (`1234abcd`), hoặc một trong các truy vấn đặc biệt `latest`, `upgrade`, `patch` hay `none`. Nếu không có phiên bản nào được cung cấp, `go get` sử dụng truy vấn `@upgrade`.

Sau khi `go get` đã phân giải các đối số của nó thành các module và phiên bản cụ thể, `go get` sẽ thêm, thay đổi hoặc xóa các [chỉ thị `require`](#go-mod-file-require) trong tệp `go.mod` của module chính để đảm bảo các module vẫn ở những phiên bản mong muốn trong tương lai. Lưu ý rằng các phiên bản được yêu cầu trong tệp `go.mod` là *phiên bản tối thiểu* và có thể được tự động tăng lên khi các dependency mới được thêm vào. Xem [Lựa chọn phiên bản tối thiểu (MVS)](#minimal-version-selection) để biết chi tiết về cách các phiên bản được lựa chọn và xung đột được giải quyết bởi các lệnh nhận biết module.

Các module khác có thể được nâng cấp khi một module được đặt tên trên dòng lệnh được thêm, nâng cấp hoặc hạ cấp nếu phiên bản mới của module được đặt tên yêu cầu các module khác ở phiên bản cao hơn. Ví dụ, giả sử module `example.com/a` được nâng cấp lên phiên bản `v1.5.0`, và phiên bản đó yêu cầu module `example.com/b` ở phiên bản `v1.2.0`. Nếu module `example.com/b` hiện được yêu cầu ở phiên bản `v1.1.0`, `go get example.com/a@v1.5.0` cũng sẽ nâng cấp `example.com/b` lên `v1.2.0`.

![go get nâng cấp một yêu cầu bắc cầu](/doc/mvs/get-upgrade.svg)

Các module khác có thể bị hạ cấp khi một module được đặt tên trên dòng lệnh bị hạ cấp hoặc bị xóa. Tiếp tục ví dụ trên, giả sử module `example.com/b` bị hạ cấp xuống `v1.1.0`. Module `example.com/a` cũng sẽ bị hạ cấp xuống một phiên bản yêu cầu `example.com/b` ở phiên bản `v1.1.0` hoặc thấp hơn.

![go get hạ cấp một yêu cầu bắc cầu](/doc/mvs/get-downgrade.svg)

Một yêu cầu module có thể được xóa bằng cách sử dụng hậu tố phiên bản `@none`. Đây là một dạng hạ cấp đặc biệt. Các module phụ thuộc vào module đã xóa sẽ bị hạ cấp hoặc xóa khi cần thiết. Một yêu cầu module có thể được xóa ngay cả khi một hoặc nhiều gói của nó được các gói trong module chính nhập. Trong trường hợp này, lệnh build tiếp theo có thể thêm một yêu cầu module mới.

Nếu một module cần thiết ở hai phiên bản khác nhau (được chỉ định rõ ràng trong các đối số dòng lệnh hoặc để đáp ứng việc nâng cấp và hạ cấp), `go get` sẽ báo lỗi.

Sau khi `go get` đã chọn một tập hợp phiên bản mới, nó kiểm tra xem bất kỳ phiên bản module mới được chọn nào hoặc bất kỳ module nào cung cấp các gói được đặt tên trên dòng lệnh có [bị thu hồi](#glos-retracted-version) hoặc [bị phản đối](#glos-deprecated-module) hay không. `go get` in ra cảnh báo cho mỗi phiên bản bị thu hồi hoặc module bị phản đối mà nó tìm thấy. [`go list -m -u all`](#go-list-m) có thể được sử dụng để kiểm tra việc thu hồi và phản đối trong tất cả dependency.

Sau khi `go get` cập nhật tệp `go.mod`, nó xây dựng các gói được đặt tên trên dòng lệnh. Các tệp thực thi sẽ được cài đặt trong thư mục được đặt tên bởi biến môi trường `GOBIN`, mặc định là `$GOPATH/bin` hoặc `$HOME/go/bin` nếu biến môi trường `GOPATH` chưa được thiết lập.

`go get` hỗ trợ các cờ sau:

* Cờ `-d` cho `go get` biết không xây dựng hoặc cài đặt các gói. Khi sử dụng `-d`, `go get` sẽ chỉ quản lý các dependency trong `go.mod`. Việc sử dụng `go get` không có `-d` để xây dựng và cài đặt các gói đã bị phản đối (kể từ Go 1.17). Trong Go 1.18, `-d` sẽ luôn được bật.
* Cờ `-u` cho `go get` biết nâng cấp các module cung cấp các gói được nhập trực tiếp hoặc gián tiếp bởi các gói được đặt tên trên dòng lệnh. Mỗi module được `-u` chọn sẽ được nâng cấp lên phiên bản mới nhất trừ khi nó đã được yêu cầu ở phiên bản cao hơn (một pre-release).
* Cờ `-u=patch` (không phải `-u patch`) cũng cho `go get` biết nâng cấp các dependency, nhưng `go get` sẽ nâng cấp từng dependency lên phiên bản patch mới nhất (tương tự truy vấn phiên bản `@patch`).
* Cờ `-t` cho `go get` biết xem xét các module cần thiết để xây dựng bài kiểm thử của các gói được đặt tên trên dòng lệnh. Khi `-t` và `-u` được sử dụng cùng nhau, `go get` cũng sẽ cập nhật các dependency của bài kiểm thử.
* Không nên sử dụng cờ `-insecure` nữa. Cờ này cho phép `go get` phân giải các đường dẫn import tùy chỉnh và lấy dữ liệu từ các repository cũng như proxy module bằng các lược đồ không an toàn như HTTP. [Biến môi trường](#environment-variables) `GOINSECURE` cung cấp khả năng kiểm soát chi tiết hơn và nên được sử dụng thay thế.
* Cờ `-tool` hướng dẫn go thêm một dòng tool tương ứng vào `go.mod` cho mỗi gói được liệt kê. Nếu `-tool` được sử dụng với `@none`, dòng đó sẽ bị xóa.

Kể từ Go 1.16, [`go install`](#go-install) là lệnh được khuyến nghị để xây dựng và cài đặt chương trình. Khi được sử dụng với hậu tố phiên bản (như `@latest` hoặc `@v1.4.6`), `go install` xây dựng các gói ở chế độ nhận biết module, bỏ qua tệp `go.mod` trong thư mục hiện tại hoặc bất kỳ thư mục cha nào nếu có.

`go get` tập trung hơn vào việc quản lý các yêu cầu trong `go.mod`. Cờ `-d` đã bị phản đối, và kể từ Go 1.18, cờ này luôn được bật.

### `go install` {#go-install}

Cách dùng:

```
go install [build flags] [packages]
```

Ví dụ:

```
# Cài đặt phiên bản mới nhất của một chương trình,
# bỏ qua go.mod trong thư mục hiện tại (nếu có).
$ go install golang.org/x/tools/gopls@latest

# Cài đặt một phiên bản cụ thể của một chương trình.
$ go install golang.org/x/tools/gopls@v0.6.4

# Cài đặt một chương trình ở phiên bản được module trong thư mục hiện tại chọn.
$ go install golang.org/x/tools/gopls

# Cài đặt tất cả chương trình trong một thư mục.
$ go install ./cmd/...
```

Lệnh `go install` xây dựng và cài đặt các gói được đặt tên bởi các đường dẫn trên dòng lệnh. Các tệp thực thi (các gói `main`) được cài đặt vào thư mục được đặt tên bởi biến môi trường `GOBIN`, mặc định là `$GOPATH/bin` hoặc `$HOME/go/bin` nếu biến môi trường `GOPATH` chưa được đặt. Các tệp thực thi trong `GOROOT` được cài đặt vào `$GOROOT/bin` hoặc `GOTOOLDIR` thay vì `GOBIN`. Các gói không phải tệp thực thi được xây dựng và lưu vào bộ nhớ đệm nhưng không được cài đặt.

Kể từ Go 1.16, nếu các đối số có hậu tố phiên bản (như `@latest` hoặc `@v1.0.0`), `go install` xây dựng các gói ở chế độ nhận biết module, bỏ qua tệp `go.mod` trong thư mục hiện tại hoặc bất kỳ thư mục cha nào nếu có. Điều này hữu ích để cài đặt các tệp thực thi mà không ảnh hưởng đến các dependency của module chính.

Để loại bỏ sự mơ hồ về việc phiên bản module nào được sử dụng trong quá trình xây dựng, nếu bất kỳ đối số nào có hậu tố phiên bản, các đối số phải thỏa mãn các ràng buộc sau:

* Các đối số phải là đường dẫn gói hoặc mẫu gói (với ký tự đại diện "`...`"). Chúng không được là các gói chuẩn (như `fmt`), các meta-pattern (`std`, `cmd`, `all`, `work`, `tool`), hoặc các đường dẫn tệp tương đối hay tuyệt đối. Lưu ý rằng `go install tool` có thể được sử dụng mà không có hậu tố phiên bản: xem bên dưới.
* Tất cả đối số phải có cùng hậu tố phiên bản. Không cho phép các truy vấn khác nhau, ngay cả khi chúng tham chiếu đến cùng một phiên bản.
* Tất cả đối số phải tham chiếu đến các gói trong cùng một module ở cùng một phiên bản.
* Các đối số là đường dẫn gói phải tham chiếu đến các gói `main`. Các đối số dạng mẫu chỉ khớp với các gói `main`.
* Không có module nào được xem là [module chính](#glos-main-module).
  * Nếu module chứa các gói được đặt tên trên dòng lệnh có tệp `go.mod`, tệp này không được chứa các chỉ thị (`replace` và `exclude`) khiến nó được diễn giải khác đi nếu nó là module chính.
  * Module không được yêu cầu một phiên bản cao hơn của chính nó.
  * Các thư mục vendor không được sử dụng trong bất kỳ module nào. (Các thư mục vendor không được bao gồm trong [tệp zip module](#zip-files), vì vậy `go install` không tải xuống chúng.)

Xem [Truy vấn phiên bản](#version-queries) để biết cú pháp truy vấn phiên bản được hỗ trợ. Go 1.15 trở xuống không hỗ trợ sử dụng truy vấn phiên bản với `go install`.

Nếu các đối số không có hậu tố phiên bản, `go install` có thể chạy ở chế độ nhận biết module hoặc chế độ `GOPATH`, tùy thuộc vào biến môi trường `GO111MODULE` và sự hiện diện của tệp `go.mod`. Xem [các lệnh nhận biết module](#mod-commands) để biết chi tiết. Nếu chế độ nhận biết module được bật, `go install` chạy trong ngữ cảnh của module chính, có thể khác với module chứa gói đang được cài đặt. Ở chế độ nhận biết module, `go install tool` có thể được sử dụng từ một module để cài đặt tất cả tool trong module.

### `go tool` {#go-tool}

Cách dùng:

```
go tool [-n] command [args...]
```

Ví dụ:

```
$ go tool golang.org/x/tools/cmd/stringer
$ go tool stringer
```

Ở chế độ module, lệnh `go tool` có thể được dùng để xây dựng và chạy các công cụ được khai báo trong các tệp `go.mod` bằng [chỉ thị `tool`](#go-mod-file-tool).
Lệnh có thể được chỉ định bằng đường dẫn gói đầy đủ tới một công cụ được khai báo bằng chỉ thị tool. Tên binary mặc định của công cụ, là thành phần cuối cùng của đường dẫn gói, không bao gồm hậu tố phiên bản chính, cũng có thể được sử dụng nếu tên đó là duy nhất trong số các công cụ đã cài đặt.

### `go list -m` {#go-list-m}

Cách dùng:

```
go list -m [-u] [-retracted] [-versions] [list flags] [modules]
```

Ví dụ:

```
$ go list -m all
$ go list -m -versions example.com/m
$ go list -m -json example.com/m@latest
```

Cờ `-m` khiến `go list` liệt kê các module thay vì các gói. Ở chế độ này, các đối số của `go list` có thể là module, mẫu module (chứa ký tự đại diện `...`), [truy vấn phiên bản](#version-queries), hoặc mẫu đặc biệt `all`, khớp với tất cả module trong [danh sách build](#glos-build-list). Nếu không chỉ định đối số nào, [module chính](#glos-main-module) được liệt kê.

Khi liệt kê các module, cờ `-f` vẫn chỉ định một mẫu định dạng được áp dụng cho một cấu trúc Go, nhưng giờ đây là cấu trúc `Module`:

```
type Module struct {
    Path       string        // đường dẫn module
    Version    string        // phiên bản module
    Versions   []string      // các phiên bản module có sẵn
    Replace    *Module       // được thay thế bởi module này
    Time       *time.Time    // thời điểm tạo phiên bản
    Update     *Module       // bản cập nhật có sẵn (với -u)
    Main       bool          // đây có phải là module chính không?
    Indirect   bool          // module chỉ được module chính cần đến gián tiếp
    Dir        string        // thư mục chứa bản sao cục bộ của tệp, nếu có
    GoMod      string        // đường dẫn đến tệp go.mod mô tả module, nếu có
    GoVersion  string        // phiên bản Go được dùng trong module
    Retracted  []string      // thông tin thu hồi, nếu có (với -retracted hoặc -u)
    Deprecated string        // thông báo không còn được khuyến nghị, nếu có (với -u)
    Error      *ModuleError  // lỗi khi tải module
}

type ModuleError struct {
    Err string // chính lỗi đó
}
```

Đầu ra mặc định là in đường dẫn module rồi đến thông tin về phiên bản và việc thay thế nếu có. Ví dụ, `go list -m all` có thể in:

```
example.com/main/module
golang.org/x/net v0.1.0
golang.org/x/text v0.3.0 => /tmp/text
rsc.io/pdf v0.1.1
```

Cấu trúc `Module` có phương thức `String` định dạng dòng đầu ra này, vì vậy định dạng mặc định tương đương với {{raw "`-f '{{.String}}'`"}}.

Lưu ý rằng khi một module đã được thay thế, trường `Replace` của nó mô tả module thay thế, và trường `Dir` của nó được đặt thành mã nguồn của module thay thế nếu có. (Nghĩa là, nếu `Replace` khác nil, thì `Dir` được đặt thành `Replace.Dir`, không có quyền truy cập vào mã nguồn đã bị thay thế.)

Cờ `-u` thêm thông tin về các bản nâng cấp có sẵn. Khi phiên bản mới nhất của một module đã cho mới hơn phiên bản hiện tại, `list -u` đặt trường `Update` của module thành thông tin về module mới hơn. `list -u` cũng in liệu phiên bản hiện được chọn có bị [thu hồi](#glos-retracted-version) hay không và liệu module có [không còn được khuyến nghị](#go-mod-file-module-deprecation) hay không. Phương thức `String` của module cho biết một bản nâng cấp có sẵn bằng cách định dạng phiên bản mới hơn trong dấu ngoặc vuông sau phiên bản hiện tại. Ví dụ, `go list -m -u all` có thể in:

```
example.com/main/module
golang.org/x/old v1.9.9 (deprecated)
golang.org/x/net v0.1.0 (retracted) [v0.2.0]
golang.org/x/text v0.3.0 [v0.4.0] => /tmp/text
rsc.io/pdf v0.1.1 [v0.1.2]
```

(Đối với các công cụ, `go list -m -u -json all` có thể thuận tiện hơn để phân tích.)

Cờ `-versions` khiến `list` đặt trường `Versions` của module thành danh sách tất cả phiên bản đã biết của module đó, được sắp xếp theo quy tắc lập phiên bản ngữ nghĩa, từ thấp đến cao. Cờ này cũng thay đổi định dạng đầu ra mặc định để hiển thị đường dẫn module theo sau là danh sách phiên bản được phân tách bằng khoảng trắng. Các phiên bản bị thu hồi được loại khỏi danh sách này trừ khi cờ `-retracted` cũng được chỉ định.

Cờ `-retracted` yêu cầu `list` hiển thị các phiên bản bị thu hồi trong danh sách được in bằng cờ `-versions` và xem xét các phiên bản bị thu hồi khi phân giải [truy vấn phiên bản](#version-queries). Ví dụ, `go list -m -retracted example.com/m@latest` hiển thị phiên bản bản phát hành hoặc pre-release cao nhất của module `example.com/m`, ngay cả khi phiên bản đó đã bị thu hồi.
Các [chỉ thị `retract`](#go-mod-file-retract) và [thông tin không còn được khuyến nghị](#go-mod-file-module-deprecation) được tải từ tệp `go.mod` ở phiên bản này. Cờ `-retracted` được thêm vào trong Go 1.16.

Hàm mẫu `module` nhận một đối số chuỗi duy nhất, đối số này phải là đường dẫn module hoặc truy vấn, và trả về module được chỉ định dưới dạng cấu trúc `Module`. Nếu xảy ra lỗi, kết quả sẽ là một cấu trúc `Module` có trường `Error` khác nil.

### `go mod download` {#go-mod-download}

Cách dùng:

```
go mod download [-x] [-json] [-reuse=old.json] [modules]
```

Ví dụ:

```
$ go mod download
$ go mod download golang.org/x/mod@v0.2.0
```

Lệnh `go mod download` tải các module được chỉ định vào [bộ nhớ đệm module](#glos-module-cache). Đối số có thể là đường dẫn module hoặc mẫu module chọn các dependency của module chính, hoặc [truy vấn phiên bản](#version-queries) có dạng `path@version`. Khi không có đối số, `download` áp dụng cho tất cả dependency của [module chính](#glos-main-module).

Lệnh `go` sẽ tự động tải các module khi cần trong quá trình thực thi thông thường. Lệnh `go mod download` chủ yếu hữu ích để điền sẵn bộ nhớ đệm module hoặc để tải dữ liệu sẽ được cung cấp bởi [module proxy](#glos-module-proxy).

Theo mặc định, `download` không ghi gì ra đầu ra tiêu chuẩn. Lệnh này in các thông báo tiến trình và lỗi ra luồng lỗi tiêu chuẩn.

Cờ `-json` khiến `download` in một chuỗi các đối tượng JSON ra đầu ra tiêu chuẩn, mô tả từng module đã tải xuống (hoặc lỗi), tương ứng với struct Go sau:

```
type Module struct {
    Path     string // đường dẫn module
    Query    string // truy vấn phiên bản tương ứng với phiên bản này
    Version  string // phiên bản module
    Error    string // lỗi khi tải module
    Info     string // đường dẫn tuyệt đối đến tệp .info đã lưu trong bộ nhớ đệm
    GoMod    string // đường dẫn tuyệt đối đến tệp .mod đã lưu trong bộ nhớ đệm
    Zip      string // đường dẫn tuyệt đối đến tệp .zip đã lưu trong bộ nhớ đệm
    Dir      string // đường dẫn tuyệt đối đến thư mục gốc mã nguồn đã lưu trong bộ nhớ đệm
    Sum      string // checksum cho đường dẫn, phiên bản (như trong go.sum)
    GoModSum string // checksum cho go.mod (như trong go.sum)
    Origin   any    // nguồn gốc của module
    Reuse    bool   // việc tái sử dụng thông tin module cũ có an toàn hay không
}
```

Cờ `-x` khiến `download` in các lệnh mà `download` thực thi ra luồng lỗi tiêu chuẩn.

Cờ -reuse nhận tên của tệp chứa đầu ra JSON của một lần gọi 'go mod download -json' trước đó. Lệnh go có thể sử dụng tệp này để xác định rằng một module không thay đổi kể từ lần gọi trước đó và tránh tải lại module. Các module không được tải lại sẽ được đánh dấu trong đầu ra mới bằng cách đặt trường Reuse thành true. Thông thường bộ nhớ đệm module cung cấp kiểu tái sử dụng này một cách tự động; cờ -reuse có thể hữu ích trên các hệ thống không duy trì bộ nhớ đệm module.

### `go mod edit` {#go-mod-edit}

Cách dùng:

```
go mod edit [editing flags] [-fmt|-print|-json] [go.mod]
```

Ví dụ:

```
# Thêm một chỉ thị replace.
$ go mod edit -replace example.com/a@v1.0.0=./a

# Xóa một chỉ thị replace.
$ go mod edit -dropreplace example.com/a@v1.0.0

# Đặt phiên bản go, thêm một yêu cầu, và in tệp
# thay vì ghi tệp vào đĩa.
$ go mod edit -go=1.14 -require=example.com/m@v1.0.0 -print

# Định dạng tệp go.mod.
$ go mod edit -fmt

# Định dạng và in một tệp .mod khác.
$ go mod edit -print tools.mod

# In biểu diễn JSON của tệp go.mod.
$ go mod edit -json
```

Lệnh `go mod edit` cung cấp một giao diện dòng lệnh để chỉnh sửa và định dạng các tệp `go.mod`, chủ yếu được sử dụng bởi các công cụ và script. `go mod edit` chỉ đọc một tệp `go.mod`; nó không tra cứu thông tin về các module khác. Theo mặc định, `go mod edit` đọc và ghi tệp `go.mod` của module chính, nhưng có thể chỉ định một tệp đích khác sau các cờ chỉnh sửa.

Các cờ chỉnh sửa chỉ định một chuỗi thao tác chỉnh sửa.

* Cờ `-module` thay đổi đường dẫn của module (dòng module trong tệp `go.mod`).
* Cờ `-go=version` đặt phiên bản ngôn ngữ Go được mong đợi.
* Các cờ `-require=path@version` và `-droprequire=path` thêm và xóa một yêu cầu đối với đường dẫn module và phiên bản đã cho. Lưu ý rằng `-require` ghi đè mọi yêu cầu hiện có trên `path`. Các cờ này chủ yếu dành cho các công cụ hiểu đồ thị module. Người dùng nên ưu tiên `go get path@version` hoặc `go get path@none`, vì các lệnh này thực hiện những điều chỉnh `go.mod` khác khi cần để đáp ứng các ràng buộc do các module khác áp đặt. Xem [`go get`](#go-get).
* Các cờ `-exclude=path@version` và `-dropexclude=path@version` thêm và xóa một loại trừ đối với đường dẫn module và phiên bản đã cho. Lưu ý rằng `-exclude=path@version` không thực hiện thao tác nào nếu loại trừ đó đã tồn tại.
* Cờ `-replace=old[@v]=new[@v]` thêm một thay thế cho cặp đường dẫn module và phiên bản đã cho. Nếu `@v` trong `old@v` bị bỏ qua, một thay thế không có phiên bản ở phía bên trái sẽ được thêm vào, áp dụng cho mọi phiên bản của đường dẫn module cũ. Nếu `@v` trong `new@v` bị bỏ qua, đường dẫn mới phải là thư mục gốc của một module cục bộ, không phải là đường dẫn module. Lưu ý rằng `-replace` ghi đè mọi thay thế dư thừa cho `old[@v]`, vì vậy việc bỏ qua `@v` sẽ xóa các thay thế cho các phiên bản cụ thể.
* Cờ `-dropreplace=old[@v]` xóa một thay thế của cặp đường dẫn module và phiên bản đã cho. Nếu cung cấp `@v`, một thay thế với phiên bản đã cho sẽ bị xóa. Một thay thế hiện có không có phiên bản ở phía bên trái vẫn có thể thay thế module. Nếu bỏ qua `@v`, một thay thế không có phiên bản sẽ bị xóa.
* Các cờ `-retract=version` và `-dropretract=version` thêm và xóa một khai báo thu hồi cho phiên bản đã cho, có thể là một phiên bản đơn (như `v1.2.3`) hoặc một khoảng (như `[v1.1.0,v1.2.0]`). Lưu ý rằng cờ `-retract` không thể thêm chú thích lý do cho chỉ thị `retract`. Các chú thích lý do được khuyến nghị và có thể được hiển thị bởi `go list -m -u` và các lệnh khác.
* Các cờ `-tool=path` và `-droptool=path` thêm và xóa một chỉ thị `tool` cho các đường dẫn đã cho. Lưu ý rằng thao tác này sẽ không thêm các dependency cần thiết vào đồ thị build. Người dùng nên ưu tiên `go get -tool path` để thêm một công cụ, hoặc `go get -tool path@none` để xóa một công cụ.

Các cờ chỉnh sửa có thể được lặp lại. Các thay đổi được áp dụng theo thứ tự đã cho.

`go mod edit` có thêm các cờ kiểm soát đầu ra của nó.

* Cờ `-fmt` định dạng lại tệp `go.mod` mà không thực hiện các thay đổi khác. Việc định dạng lại này cũng được ngầm thực hiện bởi bất kỳ sửa đổi nào khác sử dụng hoặc ghi lại tệp `go.mod`. Chỉ cần dùng cờ này khi không chỉ định cờ nào khác, như trong `go mod edit -fmt`.
* Cờ `-print` in `go.mod` cuối cùng ở định dạng văn bản thay vì ghi lại vào đĩa.
* Cờ `-json` in `go.mod` cuối cùng ở định dạng JSON thay vì ghi lại vào đĩa ở định dạng văn bản. Đầu ra JSON tương ứng với các kiểu Go sau:

```
type Module struct {
    Path    string
    Version string
}

type GoMod struct {
    Module  ModPath
    Go      string
    Require []Require
    Exclude []Module
    Replace []Replace
    Retract []Retract
}

type ModPath struct {
    Path       string
    Deprecated string
}

type Require struct {
    Path     string
    Version  string
    Indirect bool
}

type Replace struct {
    Old Module
    New Module
}

type Retract struct {
    Low       string
    High      string
    Rationale string
}

type Tool struct {
    Path      string
}
```

Lưu ý rằng điều này chỉ mô tả chính tệp `go.mod`, không phải các module khác được tham chiếu gián tiếp. Để có toàn bộ tập hợp module có sẵn cho một bản build, hãy dùng `go list -m -json all`. Xem [`go list -m`](#go-list-m).

Ví dụ, một công cụ có thể lấy tệp `go.mod` dưới dạng cấu trúc dữ liệu bằng cách phân tích đầu ra của `go mod edit -json`, sau đó có thể thực hiện thay đổi bằng cách gọi `go mod edit` với `-require`, `-exclude`, v.v.

Các công cụ cũng có thể sử dụng package [`golang.org/x/mod/modfile`](https://pkg.go.dev/golang.org/x/mod/modfile?tab=doc) để phân tích cú pháp, chỉnh sửa và định dạng các tệp `go.mod`.

### `go mod graph` {#go-mod-graph}

Cách dùng:

```
go mod graph [-go=version]
```

Lệnh `go mod graph` in [đồ thị yêu cầu module](#glos-module-graph) (đã áp dụng các thay thế) dưới dạng văn bản. Ví dụ:

```
example.com/main example.com/a@v1.1.0
example.com/main example.com/b@v1.2.0
example.com/a@v1.1.0 example.com/b@v1.1.1
example.com/a@v1.1.0 example.com/c@v1.3.0
example.com/b@v1.1.0 example.com/c@v1.1.0
example.com/b@v1.2.0 example.com/c@v1.2.0
```

Mỗi đỉnh trong đồ thị module đại diện cho một phiên bản cụ thể của một module. Mỗi cạnh trong đồ thị đại diện cho một yêu cầu về phiên bản tối thiểu của một dependency.

`go mod graph` in các cạnh của đồ thị, mỗi cạnh trên một dòng. Mỗi dòng có hai trường được phân tách bằng khoảng trắng: một phiên bản module và một trong các dependency của nó. Mỗi phiên bản module được xác định dưới dạng chuỗi có dạng `path@version`. Module chính không có hậu tố `@version`, vì nó không có phiên bản.

Cờ `-go` khiến `go mod graph` báo cáo đồ thị module được tải bởi phiên bản Go đã cho, thay vì phiên bản được chỉ ra bởi [chỉ thị `go`](#go-mod-file-go) trong tệp `go.mod`.

Xem [Lựa chọn phiên bản tối thiểu (MVS)](#minimal-version-selection) để biết thêm thông tin về cách các phiên bản được chọn. Xem thêm [`go list -m`](#go-list-m) để in các phiên bản đã chọn và [`go mod why`](#go-mod-why) để hiểu tại sao một module là cần thiết.

### `go mod init` {#go-mod-init}

Cách dùng:

```
go mod init [module-path]
```

Ví dụ:

```
go mod init
go mod init example.com/m
```

Lệnh `go mod init` khởi tạo và ghi một tệp `go.mod` mới trong thư mục hiện tại, về bản chất tạo một module mới có gốc tại thư mục hiện tại. Tệp `go.mod` không được tồn tại trước đó.

`init` chấp nhận một đối số tùy chọn, là [đường dẫn module](#glos-module-path) cho module mới. Xem [Đường dẫn module](#module-path) để biết hướng dẫn về cách chọn đường dẫn module. Nếu đối số đường dẫn module bị bỏ qua, `init` sẽ cố gắng suy ra đường dẫn module bằng cách sử dụng các chú thích import trong các tệp `.go` và thư mục hiện tại (nếu nằm trong `GOPATH`).

### `go mod tidy` {#go-mod-tidy}

Cách dùng:

```
go mod tidy [-e] [-v] [-x] [-diff] [-go=version] [-compat=version]
```

`go mod tidy` đảm bảo tệp `go.mod` khớp với mã nguồn trong module. Lệnh này thêm mọi yêu cầu module còn thiếu cần thiết để xây dựng các gói và dependency của module hiện tại, đồng thời loại bỏ các yêu cầu đối với những module không cung cấp bất kỳ gói liên quan nào. Lệnh cũng thêm mọi mục còn thiếu vào `go.sum` và loại bỏ các mục không cần thiết.

Cờ `-e` (được thêm trong Go 1.16) khiến `go mod tidy` cố gắng tiếp tục thực hiện mặc dù gặp lỗi trong khi tải các gói.

Cờ `-v` khiến `go mod tidy` in thông tin về các module đã bị loại bỏ ra lỗi chuẩn.

Cờ `-x` khiến `go mod tidy` in các lệnh mà `tidy` thực thi.

Cờ `-diff` khiến `go mod tidy` không sửa đổi go.mod hoặc go.sum mà thay vào đó in các thay đổi cần thiết dưới dạng unified diff. Lệnh thoát với mã khác không nếu diff không rỗng.

`go mod tidy` hoạt động bằng cách tải tất cả các gói trong [module chính](#glos-main-module), tất cả công cụ của module đó và tất cả các gói mà chúng import, theo cách đệ quy. Điều này bao gồm các gói được import bởi các bài kiểm thử (bao gồm cả các bài kiểm thử trong những module khác). `go mod tidy` hoạt động như thể tất cả build tag đều được bật, vì vậy nó sẽ xem xét các tệp mã nguồn dành riêng cho nền tảng và các tệp yêu cầu build tag tùy chỉnh, ngay cả khi những tệp mã nguồn đó thường không được xây dựng. Có một ngoại lệ: build tag `ignore` không được bật, vì vậy tệp có ràng buộc build `// +build ignore` sẽ không được xem xét. Lưu ý rằng `go mod tidy` sẽ không xem xét các gói trong module chính nằm trong các thư mục có tên `testdata` hoặc có tên bắt đầu bằng `.` hay `_`, trừ khi các gói đó được các gói khác import một cách rõ ràng.

Sau khi `go mod tidy` đã tải tập hợp các gói này, nó đảm bảo rằng mỗi module cung cấp một hoặc nhiều gói có một chỉ thị `require` trong tệp `go.mod` của module chính hoặc — nếu module chính ở phiên bản `go 1.16` trở xuống — được một module được yêu cầu khác yêu cầu. `go mod tidy` sẽ thêm yêu cầu đối với phiên bản mới nhất của mỗi module còn thiếu (xem [Truy vấn phiên bản](#version-queries) để biết định nghĩa của phiên bản `latest`). `go mod tidy` sẽ loại bỏ các chỉ thị `require` đối với những module không cung cấp bất kỳ gói nào trong tập hợp được mô tả ở trên.

`go mod tidy` cũng có thể thêm hoặc xóa các chú thích `// indirect` trên các chỉ thị `require`. Một chú thích `// indirect` biểu thị một module không cung cấp gói được import bởi một gói trong module chính. (Xem [chỉ thị `require`](#go-mod-file-require) để biết thêm chi tiết về thời điểm các dependency và chú thích `// indirect` được thêm vào.)

Nếu cờ `-go` được đặt, `go mod tidy` sẽ cập nhật [chỉ thị `go`](#go-mod-file-go) thành phiên bản được chỉ định, bật hoặc tắt [cắt tỉa đồ thị module](#graph-pruning) và [tải module lười](#lazy-loading) (đồng thời thêm hoặc xóa các yêu cầu gián tiếp khi cần) theo phiên bản đó.

Theo mặc định, `go mod tidy` sẽ kiểm tra rằng [các phiên bản được chọn](#glos-selected-version) của các module không thay đổi khi đồ thị module được tải bởi phiên bản Go ngay trước phiên bản được chỉ định trong chỉ thị `go`. Phiên bản được kiểm tra để tương thích cũng có thể được chỉ định rõ ràng thông qua cờ `-compat`.

### `go mod vendor` {#go-mod-vendor}

Cách dùng:

```
go mod vendor [-e] [-v] [-o]
```

Lệnh `go mod vendor` tạo một thư mục có tên `vendor` trong thư mục gốc của [module chính](#glos-main-module), chứa các bản sao của tất cả gói cần thiết để hỗ trợ việc xây dựng và kiểm thử các gói trong module chính. Các gói chỉ được nhập bởi các bài kiểm thử của những gói bên ngoài module chính sẽ không được đưa vào. Tương tự như [`go mod tidy`](#go-mod-tidy) và các lệnh module khác, [ràng buộc xây dựng](#glos-build-constraint) ngoại trừ `ignore` không được xem xét khi tạo thư mục `vendor`.

Khi vendoring được bật, lệnh `go` sẽ tải các gói từ thư mục `vendor` thay vì tải xuống các module từ nguồn của chúng vào bộ nhớ đệm module và sử dụng các gói từ những bản sao đã tải xuống đó. Xem [Vendoring](#vendoring) để biết thêm thông tin.

`go mod vendor` cũng tạo tệp `vendor/modules.txt`, chứa danh sách các gói đã được vendoring và các phiên bản module mà chúng được sao chép từ đó. Khi vendoring được bật, tệp kê khai này được dùng làm nguồn thông tin về phiên bản module, như được báo cáo bởi [`go list -m`](#go-list-m) và [`go version -m`](#go-version-m). Khi lệnh `go` đọc `vendor/modules.txt`, nó kiểm tra rằng các phiên bản module nhất quán với `go.mod`. Nếu `go.mod` đã thay đổi kể từ khi `vendor/modules.txt` được tạo, cần chạy lại `go mod vendor`.

Lưu ý rằng `go mod vendor` xóa thư mục `vendor` nếu thư mục này tồn tại trước khi tạo lại nó. Không nên thực hiện các thay đổi cục bộ đối với các gói đã được vendoring. Lệnh `go` không kiểm tra rằng các gói trong thư mục `vendor` có bị sửa đổi hay không, nhưng bạn có thể xác minh tính toàn vẹn của thư mục `vendor` bằng cách chạy `go mod vendor` và kiểm tra rằng không có thay đổi nào được thực hiện.

Cờ `-e` (được thêm trong Go 1.16) khiến `go mod vendor` cố gắng tiếp tục thực hiện bất chấp các lỗi gặp phải khi tải các gói.

Cờ `-v` khiến `go mod vendor` in tên của các module và gói đã được vendoring ra lỗi chuẩn.

Cờ `-o` (được thêm trong Go 1.18) khiến `go mod vendor` xuất cây vendor tại thư mục được chỉ định thay vì `vendor`. Đối số có thể là một đường dẫn tuyệt đối hoặc một đường dẫn tương đối so với thư mục gốc của module.

### `go mod verify` {#go-mod-verify}

Cách dùng:

```
go mod verify
```

`go mod verify` kiểm tra rằng các dependency của [module chính](#glos-main-module) được lưu trữ trong [module cache](#glos-module-cache) chưa bị sửa đổi kể từ khi chúng được tải xuống. Để thực hiện việc kiểm tra này, `go mod verify` băm từng [tệp `.zip`](#zip-files) module đã tải xuống và thư mục đã giải nén, sau đó so sánh các giá trị băm đó với một giá trị băm được ghi lại khi module được tải xuống lần đầu. `go mod verify` kiểm tra từng module trong [danh sách build](#glos-build-list) (có thể được in bằng [`go list -m all`](#go-list-m)).

Nếu tất cả module không bị sửa đổi, `go mod verify` in ra "all modules verified". Nếu không, lệnh báo cáo những module đã bị thay đổi và thoát với trạng thái khác không.

Lưu ý rằng tất cả lệnh nhận biết module đều xác minh rằng các giá trị băm trong tệp `go.sum` của module chính khớp với các giá trị băm được ghi lại cho các module đã tải xuống vào module cache. Nếu thiếu một giá trị băm trong `go.sum` (ví dụ vì module đang được sử dụng lần đầu), lệnh `go` sẽ xác minh giá trị băm của nó bằng [cơ sở dữ liệu checksum](#checksum-database) (trừ khi đường dẫn module khớp với `GOPRIVATE` hoặc `GONOSUMDB`). Xem [Xác thực module](#authenticating) để biết chi tiết.

Ngược lại, `go mod verify` kiểm tra rằng các tệp `.zip` của module và các thư mục đã giải nén của chúng có giá trị băm khớp với các giá trị băm được ghi lại trong module cache khi chúng được tải xuống lần đầu. Điều này hữu ích để phát hiện các thay đổi đối với tệp trong module cache *sau khi* một module đã được tải xuống và xác minh. `go mod verify` không tải xuống nội dung cho các module không có trong cache và không sử dụng các tệp `go.sum` để xác minh nội dung module. Tuy nhiên, `go mod verify` có thể tải xuống các tệp `go.mod` để thực hiện [lựa chọn phiên bản tối thiểu](#minimal-version-selection). Nó sẽ sử dụng `go.sum` để xác minh các tệp đó và có thể thêm các mục `go.sum` cho những giá trị băm bị thiếu.

### `go mod why` {#go-mod-why}

Cách dùng:

```
go mod why [-m] [-vendor] packages...
```

`go mod why` hiển thị đường dẫn ngắn nhất trong đồ thị import từ module chính đến từng gói được liệt kê.

Đầu ra là một chuỗi các đoạn, mỗi đoạn dành cho một gói hoặc module được đặt tên trên dòng lệnh, được phân tách bằng các dòng trống. Mỗi đoạn bắt đầu bằng một dòng chú thích bắt đầu bằng `#` cho biết gói hoặc module đích. Các dòng tiếp theo cung cấp một đường dẫn qua đồ thị import, mỗi dòng một gói. Nếu gói hoặc module không được tham chiếu từ module chính, đoạn đó sẽ hiển thị một ghi chú duy nhất trong dấu ngoặc đơn cho biết điều đó.

Ví dụ:

```
$ go mod why golang.org/x/text/language golang.org/x/text/encoding
# golang.org/x/text/language
rsc.io/quote
rsc.io/sampler
golang.org/x/text/language

# golang.org/x/text/encoding
(main module does not need package golang.org/x/text/encoding)
```

Cờ `-m` khiến `go mod why` xử lý các đối số của nó như một danh sách module. `go mod why` sẽ in một đường dẫn đến bất kỳ gói nào trong mỗi module. Lưu ý rằng ngay cả khi sử dụng `-m`, `go mod why` truy vấn đồ thị gói, không phải đồ thị module được in bởi [`go mod graph`](#go-mod-graph).

Cờ `-vendor` khiến `go mod why` bỏ qua các import trong các bài kiểm tra của những gói bên ngoài module chính (như [`go mod vendor`](#go-mod-vendor) thực hiện). Theo mặc định, `go mod why` xem xét đồ thị của các gói được khớp bởi mẫu `all`. Cờ này không có tác dụng sau Go 1.16 trong các module khai báo `go 1.16` trở lên (sử dụng [chỉ thị `go`](#go-mod-file-go) trong `go.mod`), vì ý nghĩa của `all` đã thay đổi để khớp với tập hợp các gói được khớp bởi `go mod vendor`.

### `go version -m` {#go-version-m}

Cách dùng:

```
go version [-m] [-v] [file ...]
```

Ví dụ:

```
# In phiên bản Go được dùng để xây dựng go.
$ go version

# In phiên bản Go được dùng để xây dựng một tệp thực thi cụ thể.
$ go version ~/go/bin/gopls

# In phiên bản Go và các phiên bản module được dùng để xây dựng một tệp thực thi cụ thể.
$ go version -m ~/go/bin/gopls

# In phiên bản Go và các phiên bản module được dùng để xây dựng các tệp thực thi trong một thư mục.
$ go version -m ~/go/bin/
```

`go version` báo cáo phiên bản Go được dùng để xây dựng từng tệp thực thi có tên trên dòng lệnh.

Nếu không có tệp nào được đặt tên trên dòng lệnh, `go version` in thông tin phiên bản của chính nó.

Nếu một thư mục được đặt tên, `go version` duyệt đệ quy qua thư mục đó, tìm các binary Go được nhận dạng và báo cáo phiên bản của chúng. Theo mặc định, `go version` không báo cáo các tệp không được nhận dạng được tìm thấy trong quá trình quét thư mục. Cờ `-v` khiến nó báo cáo các tệp không được nhận dạng.

Cờ `-m` khiến `go version` in thông tin phiên bản module được nhúng của mỗi tệp thực thi, khi có sẵn. Với mỗi tệp thực thi, `go version -m` in một bảng có các cột được phân tách bằng tab như bảng dưới đây.

```
$ go version -m ~/go/bin/goimports
/home/jrgopher/go/bin/goimports: go1.14.3
        path    golang.org/x/tools/cmd/goimports
        mod     golang.org/x/tools      v0.0.0-20200518203908-8018eb2c26ba      h1:0Lcy64USfQQL6GAJma8BdHCgeofcchQj+Z7j0SXYAzU=
        dep     golang.org/x/mod        v0.2.0          h1:KU7oHjnv3XNWfa5COkzUifxZmxp1TyI7ImMXqFxLwvQ=
        dep     golang.org/x/xerrors    v0.0.0-20191204190536-9bdfabe68543      h1:E7g+9GITq07hpfrRu66IVDexMakfv52eLZ2CXBWiKr4=
```

Định dạng của bảng có thể thay đổi trong tương lai. Có thể lấy được cùng thông tin từ
[`runtime/debug.ReadBuildInfo`](https://pkg.go.dev/runtime/debug?tab=doc#ReadBuildInfo).

Ý nghĩa của mỗi hàng trong bảng được xác định bởi từ trong cột đầu tiên.

* **`path`**: đường dẫn của gói `main` được dùng để xây dựng tệp thực thi.
* **`mod`**: module chứa gói `main`. Các cột lần lượt là đường dẫn module, phiên bản và tổng kiểm tra. [module chính](#glos-main-module) có phiên bản `(devel)` và không có tổng kiểm tra.
* **`dep`**: một module cung cấp một hoặc nhiều gói được liên kết vào tệp thực thi. Cùng định dạng với `mod`.
* **`=>`**: một [thay thế](#go-mod-file-replace) cho module ở dòng trước đó. Nếu thay thế là một thư mục cục bộ, chỉ có đường dẫn thư mục được liệt kê (không có phiên bản hoặc tổng kiểm tra). Nếu thay thế là một phiên bản module, đường dẫn, phiên bản và tổng kiểm tra được liệt kê, giống như với `mod` và `dep`. Một module được thay thế không có tổng kiểm tra.

### `go clean -modcache` {#go-clean-modcache}

Cách dùng:

```
go clean [-modcache]
```

Cờ `-modcache` khiến [`go
clean`](/cmd/go/#hdr-Remove_object_files_and_cached_files) xóa toàn bộ [bộ nhớ đệm mô-đun](#glos-module-cache), bao gồm mã nguồn chưa giải nén của các dependency có phiên bản.

Đây thường là cách tốt nhất để xóa bộ nhớ đệm mô-đun. Theo mặc định, hầu hết tệp và thư mục trong bộ nhớ đệm mô-đun là chỉ đọc để ngăn các bài kiểm thử và trình soạn thảo vô tình thay đổi tệp sau khi chúng đã được [xác thực](#authenticating). Tuy nhiên, điều này khiến các lệnh như `rm -r` thất bại, vì không thể xóa tệp nếu chưa làm cho các thư mục cha của chúng có quyền ghi.

Cờ `-modcacherw` (được chấp nhận bởi [`go
build`](/cmd/go/#hdr-Compile_packages_and_dependencies) và các lệnh khác có hỗ trợ mô-đun) khiến các thư mục mới trong bộ nhớ đệm mô-đun có quyền ghi. Để truyền `-modcacherw` cho tất cả các lệnh có hỗ trợ mô-đun, hãy thêm cờ này vào biến `GOFLAGS`. `GOFLAGS` có thể được đặt trong môi trường hoặc bằng [`go env
-w`](/cmd/go/#hdr-Print_Go_environment_information). Ví dụ, lệnh dưới đây đặt giá trị này vĩnh viễn:

```
go env -w GOFLAGS=-modcacherw
```

Nên sử dụng `-modcacherw` một cách thận trọng; các nhà phát triển nên cẩn thận không thực hiện thay đổi đối với các tệp trong bộ nhớ đệm mô-đun. Có thể dùng [`go mod verify`](#go-mod-verify) để kiểm tra rằng các tệp trong bộ nhớ đệm khớp với các hash trong tệp `go.sum` của mô-đun chính.

### Truy vấn phiên bản {#version-queries}

Một số lệnh cho phép bạn chỉ định phiên bản của một mô-đun bằng cách sử dụng *truy vấn phiên bản*, xuất hiện sau ký tự `@` theo sau đường dẫn mô-đun hoặc gói trên dòng lệnh.

Ví dụ:

```
go get example.com/m@latest
go mod download example.com/m@master
go list -m -json example.com/m@e3702bed2
```

Một truy vấn phiên bản có thể là một trong các dạng sau:

* Một phiên bản semantic đầy đủ, chẳng hạn như `v1.2.3`, chọn một phiên bản cụ thể. Xem [Các phiên bản](#versions) để biết cú pháp.
* Một tiền tố phiên bản semantic, chẳng hạn như `v1` hoặc `v1.2`, chọn phiên bản cao nhất hiện có với tiền tố đó.
* Một phép so sánh phiên bản semantic, chẳng hạn như {{raw "`<v1.2.3` or `>=v1.5.6`"}}, chọn phiên bản hiện có gần nhất với mục tiêu so sánh (phiên bản thấp nhất cho `>` và `>=`, và phiên bản cao nhất cho {{raw "`<` and `<=`"}}).
* Một mã định danh bản sửa đổi cho repository nguồn bên dưới, chẳng hạn như tiền tố hash commit, thẻ bản sửa đổi hoặc tên nhánh. Nếu bản sửa đổi được gắn thẻ bằng một phiên bản semantic, truy vấn này chọn phiên bản đó. Nếu không, truy vấn này chọn một [pseudo-version](#glos-pseudo-version) cho commit bên dưới. Lưu ý rằng các nhánh và thẻ có tên khớp với các truy vấn phiên bản khác không thể được chọn theo cách này. Ví dụ, truy vấn `v2` chọn phiên bản mới nhất bắt đầu bằng `v2`, không phải nhánh có tên `v2`.
* Chuỗi `latest`, chọn bản phát hành có phiên bản cao nhất hiện có. Nếu không có phiên bản bản phát hành, `latest` chọn phiên bản pre-release cao nhất. Nếu không có phiên bản được gắn thẻ, `latest` chọn một pseudo-version cho commit tại đầu nhánh mặc định của repository.
* Chuỗi `upgrade`, tương tự như `latest` ngoại trừ việc nếu mô-đun hiện đang được yêu cầu ở phiên bản cao hơn phiên bản mà `latest` sẽ chọn (ví dụ: một pre-release), `upgrade` sẽ chọn phiên bản hiện tại.
* Chuỗi `patch`, chọn phiên bản hiện có mới nhất có cùng số phiên bản chính và phụ với phiên bản hiện đang được yêu cầu. Nếu hiện không có phiên bản nào được yêu cầu, `patch` tương đương với `latest`. Kể từ Go 1.16, [`go get`](#go-get) yêu cầu có phiên bản hiện tại khi sử dụng `patch` (nhưng cờ `-u=patch` không có yêu cầu này).

Ngoại trừ các truy vấn cho phiên bản hoặc bản sửa đổi được đặt tên cụ thể, tất cả truy vấn đều xem xét các phiên bản hiện có được báo cáo bởi `go list -m -versions` (xem [`go list
-m`](#go-list-m)). Danh sách này chỉ chứa các phiên bản được gắn thẻ, không chứa pseudo-version. Các phiên bản mô-đun bị loại trừ bởi [chỉ thị `exclude`](#go-mod-file-exclude) trong [tệp `go.mod`](#glos-go-mod-file) của mô-đun chính sẽ không được xem xét. Các phiên bản được bao phủ bởi [chỉ thị `retract`](#go-mod-file-retract) trong tệp `go.mod` từ phiên bản `latest` của cùng mô-đun cũng bị bỏ qua, ngoại trừ khi cờ `-retracted` được sử dụng với [`go list -m`](#go-list-m) và ngoại trừ khi tải các chỉ thị `retract`.

[Các phiên bản bản phát hành](#glos-release-version) được ưu tiên hơn các phiên bản pre-release. Ví dụ: nếu có các phiên bản `v1.2.2` và `v1.2.3-pre`, truy vấn `latest` sẽ chọn `v1.2.2`, dù `v1.2.3-pre` cao hơn. Truy vấn {{raw "`<v1.2.4`"}} cũng sẽ chọn `v1.2.2`, dù `v1.2.3-pre` gần với `v1.2.4` hơn. Nếu không có phiên bản bản phát hành hoặc pre-release nào, các truy vấn `latest`, `upgrade` và `patch` sẽ chọn một pseudo-version cho commit tại đầu nhánh mặc định của repository. Các truy vấn khác sẽ báo lỗi.

### Lệnh module bên ngoài một module {#commands-outside}

Các lệnh Go nhận biết module thường chạy trong ngữ cảnh của [module chính](#glos-main-module) được xác định bởi tệp `go.mod` trong thư mục làm việc hoặc thư mục cha. Một số lệnh có thể được chạy ở chế độ nhận biết module mà không cần tệp `go.mod`, nhưng hầu hết lệnh hoạt động khác đi hoặc báo lỗi khi không có tệp `go.mod`.

Xem [các lệnh nhận biết module](#mod-commands) để biết thông tin về việc bật và tắt chế độ nhận biết module.

<table class="ModTable">
  <thead>
    <tr>
      <th>Lệnh</th>
      <th>Hành vi</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        <code>go build</code><br>
        <code>go doc</code><br>
        <code>go fix</code><br>
        <code>go fmt</code><br>
        <code>go generate</code><br>
        <code>go install</code><br>
        <code>go list</code><br>
        <code>go run</code><br>
        <code>go test</code><br>
        <code>go vet</code>
      </td>
      <td>
        Chỉ các gói trong thư viện chuẩn và các gói được chỉ định dưới dạng tệp
        <code>.go</code> trên dòng lệnh mới có thể được tải, nhập và xây dựng.
        Không thể xây dựng các gói từ những module khác, vì không có nơi để ghi
        lại các yêu cầu module và đảm bảo các bản dựng xác định.
      </td>
    </tr>
    <tr>
      <td><code>go get</code></td>
      <td>
        Các gói và tệp thực thi có thể được xây dựng và cài đặt như bình thường.
        Lưu ý rằng không có module chính khi <code>go get</code> được chạy mà
        không có tệp <code>go.mod</code>, vì vậy các chỉ thị <code>replace</code>
        và <code>exclude</code> không được áp dụng.
      </td>
    </tr>
    <tr>
      <td><code>go list -m</code></td>
      <td>
        Cần có <a href="#version-queries">truy vấn phiên bản</a> rõ ràng cho
        hầu hết đối số, ngoại trừ khi cờ <code>-versions</code> được sử dụng.
      </td>
    </tr>
    <tr>
      <td><code>go mod download</code></td>
      <td>
        Cần có <a href="#version-queries">truy vấn phiên bản</a> rõ ràng cho
        hầu hết đối số.
      </td>
    </tr>
    <tr>
      <td><code>go mod edit</code></td>
      <td>Cần có đối số tệp rõ ràng.</td>
    </tr>
    <tr>
      <td>
        <code>go mod graph</code><br>
        <code>go mod tidy</code><br>
        <code>go mod vendor</code><br>
        <code>go mod verify</code><br>
        <code>go mod why</code>
      </td>
      <td>
        Các lệnh này yêu cầu tệp <code>go.mod</code> và sẽ báo lỗi nếu tệp đó
        không tồn tại.
      </td>
    </tr>
  </tbody>
</table>

### `go work init` {#go-work-init}

Cách dùng:

```
go work init [moddirs]
```

Init khởi tạo và ghi một tệp go.work mới trong thư mục
hiện tại, về cơ bản tạo một workspace mới tại thư mục hiện tại.

go work init tùy chọn chấp nhận các đường dẫn đến các module trong
workspace làm đối số. Nếu bỏ qua đối số, một workspace trống không có
module nào sẽ được tạo.

Mỗi đường dẫn đối số được thêm vào một chỉ thị use trong tệp go.work. Phiên
bản Go hiện tại cũng sẽ được liệt kê trong tệp go.work.

### `go work edit` {#go-work-edit}

Cách dùng:

```
go work edit [editing flags] [go.work]
```

Lệnh `go work edit` cung cấp giao diện dòng lệnh để chỉnh sửa `go.work`,
chủ yếu được dùng bởi các công cụ hoặc script. Lệnh này chỉ đọc `go.work`;
nó không tra cứu thông tin về các module liên quan.
Nếu không chỉ định tệp, Edit sẽ tìm tệp `go.work` trong thư mục hiện tại
và các thư mục cha của nó

Các cờ chỉnh sửa chỉ định một chuỗi thao tác chỉnh sửa.
* Cờ `-fmt` định dạng lại tệp go.work mà không thực hiện các thay đổi khác.
  Việc định dạng lại này cũng được ngầm thực hiện bởi bất kỳ sửa đổi nào khác
  sử dụng hoặc ghi lại tệp `go.work`. Cờ này chỉ cần thiết khi không có cờ
  nào khác được chỉ định, như trong 'go work edit `-fmt`'.
* Các cờ `-use=path` và `-dropuse=path`
  thêm và xóa một chỉ thị use khỏi tập hợp thư mục module của tệp `go.work`.
* Cờ `-replace=old[@v]=new[@v]` thêm một thay thế cho cặp đường dẫn module
  và phiên bản đã cho. Nếu `@v` trong `old@v` bị bỏ qua, một thay thế
  không có phiên bản ở phía bên trái sẽ được thêm vào, áp dụng cho
  tất cả phiên bản của đường dẫn module cũ. Nếu `@v` trong `new@v` bị bỏ qua,
  đường dẫn mới phải là một thư mục gốc module cục bộ, không phải đường dẫn
  module. Lưu ý rằng `-replace` ghi đè mọi thay thế dư thừa cho `old[@v]`,
  vì vậy việc bỏ qua `@v` sẽ xóa các thay thế hiện có cho các phiên bản cụ thể.
* Cờ `-dropreplace=old[@v]` xóa một thay thế của cặp đường dẫn module
  và phiên bản đã cho. Nếu bỏ qua `@v`, một thay thế không có phiên bản
  ở phía bên trái sẽ bị xóa.
* Cờ `-go=version` đặt phiên bản ngôn ngữ Go mong muốn.

Các cờ chỉnh sửa có thể được lặp lại. Các thay đổi được áp dụng theo thứ tự đã cho.

`go work edit` có thêm các cờ kiểm soát đầu ra của nó

* Cờ -print in go.work cuối cùng ở định dạng văn bản thay vì
  ghi ngược lại vào go.mod.
* Cờ -json in tệp go.work cuối cùng ở định dạng JSON thay vì
  ghi ngược lại vào go.mod. Đầu ra JSON tương ứng với các kiểu Go sau:

```
type Module struct {
    Path    string
    Version string
}

type GoWork struct {
    Go        string
    Directory []Directory
    Replace   []Replace
}

type Use struct {
    Path       string
    ModulePath string
}

type Replace struct {
    Old Module
    New Module
}
```

### `go work use` {#go-work-use}

Cách dùng:

```
go work use [-r] [moddirs]
```

Lệnh `go work use` cung cấp giao diện dòng lệnh để thêm các thư mục, tùy chọn đệ quy, vào tệp `go.work`.

Một [chỉ thị `use`](#go-work-file-use) sẽ được thêm vào tệp `go.work` cho mỗi thư mục đối số được liệt kê trên dòng lệnh của tệp `go.work`, nếu thư mục đó tồn tại trên đĩa, hoặc bị xóa khỏi tệp `go.work` nếu thư mục đó không tồn tại trên đĩa.

Cờ `-r` tìm kiếm đệ quy các module trong các thư mục đối số, và lệnh use hoạt động như thể mỗi thư mục đó được chỉ định làm đối số.

### `go work sync` {#go-work-sync}

Cách dùng:

```
go work sync
```

Lệnh `go work sync` đồng bộ danh sách build của workspace trở lại các module của workspace.

Danh sách build của workspace là tập hợp các phiên bản của tất cả các module dependency (bắc cầu) được sử dụng để thực hiện build trong workspace. `go work sync` tạo danh sách build đó bằng cách sử dụng thuật toán [Lựa chọn phiên bản tối thiểu (MVS)](#glos-minimal-version-selection), sau đó đồng bộ các phiên bản đó trở lại từng module được chỉ định trong workspace (bằng các chỉ thị `use`).

Sau khi danh sách build của workspace được tính toán, tệp `go.mod` của từng module trong workspace được viết lại với các dependency liên quan đến module đó được nâng cấp để khớp với danh sách build của workspace.
Lưu ý rằng [Lựa chọn phiên bản tối thiểu](#glos-minimal-version-selection) đảm bảo rằng phiên bản của mỗi module trong danh sách build luôn bằng hoặc cao hơn phiên bản đó trong từng module của workspace.

## Proxy module {#module-proxy}

### Giao thức `GOPROXY` {#goproxy-protocol}

Một <dfn>proxy module</dfn> là một máy chủ HTTP có thể phản hồi các yêu cầu `GET` cho những đường dẫn được chỉ định bên dưới. Các yêu cầu không có tham số truy vấn và không yêu cầu tiêu đề cụ thể, vì vậy ngay cả một trang phục vụ từ hệ thống tệp cố định (bao gồm URL `file://`) cũng có thể là một proxy module.

Các phản hồi HTTP thành công phải có mã trạng thái 200 (OK). Các chuyển hướng (3xx) được thực hiện theo. Các phản hồi có mã trạng thái 4xx và 5xx được xem là lỗi. Các mã lỗi 404 (Not Found) và 410 (Gone) cho biết module hoặc phiên bản được yêu cầu không có trên proxy, nhưng có thể được tìm thấy ở nơi khác. Các phản hồi lỗi nên có kiểu nội dung `text/plain` với `charset` là `utf-8` hoặc `us-ascii`.

Lệnh `go` có thể được cấu hình để liên hệ với các proxy hoặc máy chủ quản lý mã nguồn bằng biến môi trường `GOPROXY`, biến này nhận một danh sách URL proxy. Danh sách này có thể bao gồm các từ khóa `direct` hoặc `off` (xem [Biến môi trường](#environment-variables) để biết chi tiết). Các phần tử trong danh sách có thể được phân tách bằng dấu phẩy (`,`) hoặc dấu gạch đứng (`|`), các dấu này quyết định hành vi dự phòng khi lỗi. Khi một URL theo sau bởi dấu phẩy, lệnh `go` chỉ chuyển sang các nguồn sau khi nhận phản hồi 404 (Not Found) hoặc 410 (Gone). Khi một URL theo sau bởi dấu gạch đứng, lệnh `go` chuyển sang các nguồn sau sau bất kỳ lỗi nào, bao gồm cả các lỗi không phải HTTP như hết thời gian chờ. Hành vi xử lý lỗi này cho phép proxy hoạt động như một cổng kiểm soát đối với các module chưa biết. Ví dụ, một proxy có thể phản hồi lỗi 403 (Forbidden) cho các module không nằm trong danh sách được phê duyệt (xem [Proxy riêng tư phục vụ các module riêng tư](#private-module-proxy-private)).

Bảng bên dưới chỉ định các truy vấn mà một proxy module phải phản hồi. Với mỗi đường dẫn, `$base` là phần đường dẫn của URL proxy, `$module` là đường dẫn module, và `$version` là một phiên bản. Ví dụ, nếu URL proxy là `https://example.com/mod`, và máy khách đang yêu cầu tệp `go.mod` cho module `golang.org/x/text` ở phiên bản `v0.3.2`, máy khách sẽ gửi một yêu cầu `GET` cho `https://example.com/mod/golang.org/x/text/@v/v0.3.2.mod`.

Để tránh sự không rõ ràng khi phục vụ từ các hệ thống tệp không phân biệt chữ hoa chữ thường, các phần tử `$module` và `$version` được mã hóa chữ hoa chữ thường bằng cách thay thế mỗi chữ cái viết hoa bằng một dấu chấm than theo sau bởi chữ cái viết thường tương ứng. Điều này cho phép các module `example.com/M` và `example.com/m` đều được lưu trữ trên đĩa, vì module trước được mã hóa thành `example.com/!m`.

<table class="ModTable">
  <thead>
    <tr>
      <th>Đường dẫn</th>
      <th>Mô tả</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>$base/$module/@v/list</code></td>
      <td>
        Trả về danh sách các phiên bản đã biết của module đã cho dưới dạng văn bản
        thuần, mỗi phiên bản trên một dòng. Danh sách này không nên bao gồm các
        pseudo-version.
      </td>
    </tr>
    <tr>
      <td><code>$base/$module/@v/$version.info</code></td>
      <td>
        <p>
          Trả về metadata được định dạng JSON về một phiên bản cụ thể của một
          module. Phản hồi phải là một đối tượng JSON tương ứng với cấu trúc dữ
          liệu Go bên dưới:
        </p>
        <pre>
type Info struct {
    Version string    // chuỗi phiên bản
    Time    time.Time // thời gian commit
}
</pre>
        <p>
          Trường <code>Version</code> là bắt buộc và phải chứa một
          <a href="#glos-canonical-version">phiên bản chuẩn</a> hợp lệ (xem
          <a href="#versions">Các phiên bản</a>). <code>$version</code> trong
          đường dẫn yêu cầu không cần phải là cùng phiên bản hoặc thậm chí là
          một phiên bản hợp lệ; endpoint này có thể được dùng để tìm các phiên
          bản cho tên nhánh hoặc mã định danh bản sửa đổi. Tuy nhiên, nếu
          <code>$version</code> là một phiên bản chuẩn có phiên bản chính tương
          thích với <code>$module</code>, trường <code>Version</code> trong phản
          hồi thành công phải giống nhau.
        </p>
        <p>
          Trường <code>Time</code> là tùy chọn. Nếu có, trường này phải là một
          chuỗi ở định dạng RFC 3339. Nó cho biết thời điểm phiên bản được tạo.
        </p>
        <p>
          Các trường khác có thể được thêm vào trong tương lai, vì vậy các tên
          khác được dành riêng.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>$base/$module/@v/$version.mod</code></td>
      <td>
        Trả về tệp <code>go.mod</code> cho một phiên bản cụ thể của một module.
        Nếu module không có tệp <code>go.mod</code> ở phiên bản được yêu cầu,
        phải trả về một tệp chỉ chứa câu lệnh <code>module</code> với đường dẫn
        module được yêu cầu. Nếu không, phải trả về tệp <code>go.mod</code>
        nguyên bản, chưa được sửa đổi.
      </td>
    </tr>
    <tr>
      <td><code>$base/$module/@v/$version.zip</code></td>
      <td>
        Trả về một tệp zip chứa nội dung của một phiên bản cụ thể của một
        module. Xem <a href="#zip-files">Các tệp zip của module</a> để biết chi
        tiết về cách định dạng tệp zip này.
      </td>
    </tr>
    <tr>
      <td><code>$base/$module/@latest</code></td>
      <td>
        Trả về metadata được định dạng JSON về phiên bản mới nhất đã biết của
        một module theo cùng định dạng với
        <code>$base/$module/@v/$version.info</code>. Phiên bản mới nhất nên là
        phiên bản của module mà lệnh <code>go</code> nên sử dụng nếu
        <code>$base/$module/@v/list</code> trống hoặc không có phiên bản nào
        được liệt kê phù hợp. Endpoint này là tùy chọn và các proxy module
        không bắt buộc phải triển khai nó.
      </td>
    </tr>
  </tbody>
</table>

Khi xác định phiên bản mới nhất của một module, lệnh `go` sẽ yêu cầu
`$base/$module/@v/list`, sau đó, nếu không tìm thấy phiên bản phù hợp,
`$base/$module/@latest`. Lệnh `go` ưu tiên theo thứ tự: phiên bản bản phát hành
cao nhất về mặt ngữ nghĩa, phiên bản pre-release cao nhất về mặt ngữ nghĩa và
pseudo-version gần đây nhất theo thời gian. Trong Go 1.12 trở về trước, lệnh
`go` xem các pseudo-version trong `$base/$module/@v/list` là các phiên bản
pre-release, nhưng điều này không còn đúng kể từ Go 1.13.

Một module proxy phải luôn cung cấp cùng một nội dung cho các phản hồi thành công
đối với các truy vấn `$base/$module/$version.mod` và `$base/$module/$version.zip`.
Nội dung này được [xác thực bằng mật mã](#authenticating) bằng cách sử dụng các
[tệp `go.sum`](#go-sum-files) và theo mặc định là [cơ sở dữ liệu checksum](#checksum-database).

Lệnh `go` lưu vào bộ nhớ đệm hầu hết nội dung mà nó tải xuống từ các module proxy
trong bộ nhớ đệm module tại `$GOPATH/pkg/mod/cache/download`. Ngay cả khi tải
trực tiếp từ các hệ thống quản lý phiên bản, lệnh `go` tạo ra các tệp `info`,
`mod` và `zip` rõ ràng rồi lưu chúng trong thư mục này, giống như khi nó tải
chúng trực tiếp từ một proxy. Bố cục bộ nhớ đệm giống với không gian URL của
proxy, vì vậy việc cung cấp `$GOPATH/pkg/mod/cache/download` tại (hoặc sao chép
nó tới) `https://example.com/proxy` sẽ cho phép
người dùng truy cập các phiên bản module đã lưu trong bộ nhớ đệm bằng cách đặt
`GOPROXY` thành `https://example.com/proxy`.

### Giao tiếp với các proxy {#communicating-with-proxies}

Lệnh `go` có thể tải mã nguồn module và metadata từ một [module proxy](#glos-module-proxy). Biến [môi trường](#environment-variables) `GOPROXY` có thể được dùng để cấu hình các proxy mà lệnh `go` có thể kết nối tới và liệu lệnh có thể giao tiếp trực tiếp với [hệ thống quản lý phiên bản](#vcs) hay không. Dữ liệu module đã tải xuống được lưu trong [bộ nhớ đệm module](#glos-module-cache). Lệnh `go` chỉ liên hệ với một proxy khi cần thông tin chưa có trong bộ nhớ đệm.

Phần [giao thức `GOPROXY`](#goproxy-protocol) mô tả các yêu cầu có thể được gửi tới máy chủ `GOPROXY`. Tuy nhiên, cũng hữu ích khi hiểu thời điểm lệnh `go` thực hiện các yêu cầu này. Ví dụ, `go build` thực hiện theo quy trình dưới đây:

* Tính toán [danh sách build](#glos-build-list) bằng cách đọc các [tệp `go.mod`](#glos-go-mod-file) và thực hiện [lựa chọn phiên bản tối thiểu (MVS)](#glos-minimal-version-selection).
* Đọc các gói được đặt tên trên dòng lệnh và các gói mà chúng import.
* Nếu một gói không được cung cấp bởi bất kỳ module nào trong danh sách build, tìm một module cung cấp gói đó. Thêm yêu cầu module với phiên bản mới nhất của nó vào `go.mod`, rồi bắt đầu lại.
* Build các gói sau khi mọi thứ đã được tải.

Khi lệnh `go` tính toán danh sách build, nó tải tệp `go.mod` cho mỗi module trong [đồ thị module](#glos-module-graph). Nếu tệp `go.mod` không có trong bộ nhớ đệm, lệnh `go` sẽ tải nó từ proxy bằng yêu cầu `$module/@v/$version.mod` (trong đó `$module` là đường dẫn module và `$version` là phiên bản). Các yêu cầu này có thể được kiểm thử bằng một công cụ như `curl`. Ví dụ, lệnh dưới đây tải tệp `go.mod` cho `golang.org/x/mod` tại phiên bản `v0.2.0`:

```
$ curl https://proxy.golang.org/golang.org/x/mod/@v/v0.2.0.mod
module golang.org/x/mod

go 1.12

require (
    golang.org/x/crypto v0.0.0-20191011191535-87dc89f01550
    golang.org/x/tools v0.0.0-20191119224855-298f0cb1881e
    golang.org/x/xerrors v0.0.0-20191011141410-1b5146add898
)
```

Để tải một gói, lệnh `go` cần mã nguồn của module cung cấp gói đó. Mã nguồn module được phân phối trong các tệp `.zip`, sau đó được giải nén vào bộ nhớ đệm module. Nếu tệp `.zip` của một module không có trong bộ nhớ đệm, lệnh `go` sẽ tải nó bằng yêu cầu `$module/@v/$version.zip`.

```
$ curl -O https://proxy.golang.org/golang.org/x/mod/@v/v0.2.0.zip
$ unzip -l v0.2.0.zip | head
Archive:  v0.2.0.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
     1479  00-00-1980 00:00   golang.org/x/mod@v0.2.0/LICENSE
     1303  00-00-1980 00:00   golang.org/x/mod@v0.2.0/PATENTS
      559  00-00-1980 00:00   golang.org/x/mod@v0.2.0/README
       21  00-00-1980 00:00   golang.org/x/mod@v0.2.0/codereview.cfg
      214  00-00-1980 00:00   golang.org/x/mod@v0.2.0/go.mod
     1476  00-00-1980 00:00   golang.org/x/mod@v0.2.0/go.sum
     5224  00-00-1980 00:00   golang.org/x/mod@v0.2.0/gosumcheck/main.go
```

Lưu ý rằng các yêu cầu `.mod` và `.zip` là riêng biệt, mặc dù các tệp `go.mod` thường nằm trong các tệp `.zip`. Lệnh `go` có thể cần tải các tệp `go.mod` cho nhiều module khác nhau, và các tệp `.mod` nhỏ hơn nhiều so với các tệp `.zip`. Ngoài ra, nếu một dự án Go không có tệp `go.mod`, proxy sẽ phục vụ một tệp `go.mod` tổng hợp chỉ chứa một [chỉ thị `module`](#go-mod-file-module). Các tệp `go.mod` tổng hợp được lệnh `go` tạo ra khi tải xuống từ một [hệ thống quản lý phiên bản](#vcs).

Nếu lệnh `go` cần tải một gói không được cung cấp bởi bất kỳ module nào trong danh sách build, nó sẽ cố gắng tìm một module mới cung cấp gói đó. Phần [Giải quyết một gói thành một module](#resolve-pkg-mod) mô tả quy trình này. Tóm lại, lệnh `go` yêu cầu thông tin về phiên bản mới nhất của từng đường dẫn module có khả năng chứa gói đó. Ví dụ, với gói `golang.org/x/net/html`, lệnh `go` sẽ thử tìm các phiên bản mới nhất của các module `golang.org/x/net/html`, `golang.org/x/net`, `golang.org/x/`, và `golang.org`. Chỉ `golang.org/x/net` thực sự tồn tại và cung cấp gói đó, vì vậy lệnh `go` sử dụng phiên bản mới nhất của module đó. Nếu có nhiều hơn một module cung cấp gói, lệnh `go` sẽ sử dụng module có đường dẫn dài nhất.

Khi lệnh `go` yêu cầu phiên bản mới nhất của một module, trước tiên nó gửi yêu cầu cho `$module/@v/list`. Nếu danh sách trống hoặc không có phiên bản nào được trả về có thể sử dụng, nó gửi yêu cầu cho `$module/@latest`. Sau khi chọn được một phiên bản, lệnh `go` gửi yêu cầu `$module/@v/$version.info` để lấy metadata. Sau đó, nó có thể gửi các yêu cầu `$module/@v/$version.mod` và `$module/@v/$version.zip` để tải tệp `go.mod` và mã nguồn.

```
$ curl https://proxy.golang.org/golang.org/x/mod/@v/list
v0.1.0
v0.2.0

$ curl https://proxy.golang.org/golang.org/x/mod/@v/v0.2.0.info
{"Version":"v0.2.0","Time":"2020-01-02T17:33:45Z"}
```

Sau khi tải tệp `.mod` hoặc `.zip`, lệnh `go` tính toán một hàm băm mật mã và kiểm tra rằng nó khớp với một hàm băm trong tệp `go.sum` của module chính. Nếu hàm băm không có trong `go.sum`, theo mặc định, lệnh `go` sẽ lấy nó từ [cơ sở dữ liệu checksum](#checksum-database). Nếu hàm băm được tính toán không khớp, lệnh `go` báo cáo lỗi bảo mật và không cài đặt tệp vào bộ nhớ đệm module. Các [biến môi trường](#environment-variables) `GOPRIVATE` và `GONOSUMDB` có thể được dùng để vô hiệu hóa các yêu cầu tới cơ sở dữ liệu checksum cho các module cụ thể. Biến môi trường `GOSUMDB` cũng có thể được đặt thành `off` để vô hiệu hóa hoàn toàn các yêu cầu tới cơ sở dữ liệu checksum. Xem [Xác thực module](#authenticating) để biết thêm thông tin. Lưu ý rằng danh sách phiên bản và metadata phiên bản được trả về cho các yêu cầu `.info` không được xác thực và có thể thay đổi theo thời gian.

### Cung cấp module trực tiếp từ một proxy {#serving-from-proxy}

Hầu hết module được phát triển và cung cấp từ một kho lưu trữ kiểm soát phiên bản. Ở
[chế độ trực tiếp](#glos-direct-mode), lệnh `go` tải xuống một module như vậy bằng
một công cụ kiểm soát phiên bản (xem [Hệ thống kiểm soát phiên bản](#vcs)). Cũng có thể
cung cấp một module trực tiếp từ một module proxy. Điều này hữu ích cho các tổ chức
muốn cung cấp module mà không để lộ các máy chủ kiểm soát phiên bản của họ và
cho các tổ chức sử dụng các công cụ kiểm soát phiên bản mà lệnh `go` không
hỗ trợ.

Khi lệnh `go` tải xuống một module ở chế độ trực tiếp, trước tiên nó tra cứu URL
của máy chủ module bằng một yêu cầu HTTP GET dựa trên đường dẫn module. Nó tìm
một thẻ `<meta>` có tên `go-import` trong phản hồi HTML. Nội dung của thẻ phải
chứa [đường dẫn gốc của kho lưu trữ](#glos-repository-root-path), hệ thống kiểm soát
phiên bản và URL, được phân tách bằng dấu cách. Xem [Tìm kho lưu trữ cho một đường
dẫn module](#vcs-find) để biết chi tiết.

Nếu hệ thống kiểm soát phiên bản là `mod`, lệnh `go` tải xuống module từ URL đã
cho bằng cách sử dụng [giao thức `GOPROXY`](#goproxy-protocol).

Ví dụ, giả sử lệnh `go` đang cố tải xuống module
`example.com/gopher` tại phiên bản `v1.0.0`. Nó gửi một yêu cầu tới
`https://example.com/gopher?go-get=1`. Máy chủ phản hồi bằng một tài liệu HTML
chứa thẻ:

```
<meta name="go-import" content="example.com/gopher mod https://modproxy.example.com">
```

Dựa trên phản hồi này, lệnh `go` tải xuống module bằng cách gửi các yêu cầu tới
`https://modproxy.example.com/example.com/gopher/@v/v1.0.0.info`,
`v1.0.0.mod` và `v1.0.0.zip`.

Lưu ý rằng các module được cung cấp trực tiếp từ một proxy không thể được tải
xuống bằng `go get` ở chế độ GOPATH.

## Hệ thống kiểm soát phiên bản {#vcs}

Lệnh `go` có thể tải xuống mã nguồn module và siêu dữ liệu trực tiếp từ một
kho lưu trữ kiểm soát phiên bản. Việc tải xuống một module từ một
[proxy](#communicating-with-proxies) thường nhanh hơn, nhưng kết nối trực tiếp
tới một kho lưu trữ là cần thiết nếu không có proxy hoặc nếu kho lưu trữ của một
module không thể được proxy truy cập (thường đúng với các kho lưu trữ riêng tư).
Git, Subversion, Mercurial, Bazaar và Fossil được hỗ trợ. Một công cụ kiểm soát
phiên bản phải được cài đặt trong một thư mục thuộc `PATH` để lệnh
`go` có thể sử dụng nó.

Để tải xuống các module cụ thể từ các kho lưu trữ mã nguồn thay vì một proxy, hãy đặt
các biến môi trường `GOPRIVATE` hoặc `GONOPROXY`. Để cấu hình lệnh `go`
tải xuống trực tiếp tất cả module từ các kho lưu trữ mã nguồn, hãy đặt `GOPROXY`
thành `direct`. Xem [Các biến môi trường](#environment-variables) để biết thêm
thông tin.

### Tìm kho lưu trữ cho đường dẫn mô-đun {#vcs-find}

Khi lệnh `go` tải xuống một mô-đun ở chế độ `direct`, trước tiên nó bắt đầu bằng việc xác định kho lưu trữ chứa mô-đun đó.

Nếu đường dẫn mô-đun có bộ định danh VCS (một trong `.bzr`, `.fossil`, `.git`, `.hg`, `.svn`) ở cuối một thành phần đường dẫn, lệnh `go` sẽ sử dụng mọi thứ trước bộ định danh đường dẫn đó làm URL của kho lưu trữ. Ví dụ, với mô-đun `example.com/foo.git/bar`, lệnh `go` tải xuống kho lưu trữ tại `example.com/foo` bằng git, và mong đợi tìm thấy mô-đun trong thư mục con `bar`. Lệnh `go` sẽ đoán giao thức cần dùng dựa trên các giao thức được công cụ quản lý phiên bản hỗ trợ.

Nếu đường dẫn mô-đun không có bộ định danh, lệnh `go` gửi một yêu cầu HTTP `GET` đến URL được tạo từ đường dẫn mô-đun với chuỗi truy vấn `?go-get=1`. Ví dụ, với mô-đun `golang.org/x/mod`, lệnh `go` có thể gửi các yêu cầu sau:

```
https://golang.org/x/mod?go-get=1 (preferred)
http://golang.org/x/mod?go-get=1  (fallback, only with GOINSECURE)
```

Lệnh `go` theo dõi các chuyển hướng nhưng bỏ qua mã trạng thái phản hồi, vì vậy máy chủ có thể phản hồi bằng 404 hoặc bất kỳ trạng thái lỗi nào khác. Biến môi trường `GOINSECURE` có thể được đặt để cho phép dự phòng và chuyển hướng sang HTTP không mã hóa cho các mô-đun cụ thể.

Máy chủ phải phản hồi bằng một tài liệu HTML chứa thẻ `<meta>` trong phần `<head>` của tài liệu. Thẻ `<meta>` nên xuất hiện sớm trong tài liệu để tránh gây nhầm lẫn cho bộ phân tích cú pháp bị giới hạn của lệnh `go`. Cụ thể, thẻ này nên xuất hiện trước mọi JavaScript hoặc CSS thô. Thẻ `<meta>` phải có dạng:

```
<meta name="go-import" content="root-path vcs repo-url [subdirectory]">
```

`root-path` là đường dẫn gốc của kho lưu trữ, phần của đường dẫn mô-đun tương ứng với thư mục gốc của kho lưu trữ, hoặc với `subdirectory` nếu có và sử dụng Go 1.25 trở lên (xem phần về `subdirectory` bên dưới). Nó phải là tiền tố hoặc khớp chính xác với đường dẫn mô-đun được yêu cầu. Nếu không khớp chính xác, một yêu cầu khác sẽ được thực hiện cho tiền tố đó để xác minh các thẻ `<meta>` khớp nhau.

`vcs` là hệ thống quản lý phiên bản. Nó phải là một trong các công cụ được liệt kê trong bảng bên dưới hoặc từ khóa `mod`, chỉ thị cho lệnh `go` tải mô-đun từ URL đã cho bằng giao thức [`GOPROXY`](#goproxy-protocol). Xem [Phục vụ mô-đun trực tiếp từ proxy](#serving-from-proxy) để biết chi tiết.

`repo-url` là URL của kho lưu trữ, chứa một lược đồ và không chứa bộ định danh .vcs. Các giao thức không an toàn (như `http://` và `git://`) chỉ có thể được sử dụng nếu đường dẫn mô-đun được khớp bởi biến môi trường `GOINSECURE`.

`subdirectory`, nếu có, là thư mục con được phân tách bằng dấu gạch chéo của kho lưu trữ mà `root-path` tương ứng, ghi đè lên mặc định là thư mục gốc của kho lưu trữ. Các thẻ meta `go-import` cung cấp `subdirectory` chỉ được Go 1.25 trở lên nhận dạng. Các lần thử tải xuống để phân giải mô-đun trên các phiên bản Go cũ hơn sẽ bỏ qua thẻ meta và dẫn đến lỗi phân giải nếu mô-đun không thể được phân giải ở nơi khác.

<table id="vcs-support" class="ModTable">
  <thead>
    <tr>
      <th>Name</th>
      <th>Command</th>
      <th>GOVCS default</th>
      <th>Secure schemes</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Bazaar</td>
      <td><code>bzr</code></td>
      <td>Private only</td>
      <td><code>https</code>, <code>bzr+ssh</code></td>
    </tr>
    <tr>
      <td>Fossil</td>
      <td><code>fossil</code></td>
      <td>Private only</td>
      <td><code>https</code></td>
    </tr>
    <tr>
      <td>Git</td>
      <td><code>git</code></td>
      <td>Public and private</td>
      <td><code>https</code>, <code>git+ssh</code>, <code>ssh</code></td>
    </tr>
    <tr>
      <td>Mercurial</td>
      <td><code>hg</code></td>
      <td>Public and private</td>
      <td><code>https</code>, <code>ssh</code></td>
    </tr>
    <tr>
      <td>Subversion</td>
      <td><code>svn</code></td>
      <td>Private only</td>
      <td><code>https</code>, <code>svn+ssh</code></td>
    </tr>
  </tbody>
</table>

Ví dụ, hãy xem lại `golang.org/x/mod`. Lệnh `go` gửi một yêu cầu đến `https://golang.org/x/mod?go-get=1`. Máy chủ phản hồi bằng một tài liệu HTML chứa thẻ:

```
<meta name="go-import" content="golang.org/x/mod git https://go.googlesource.com/mod">
```

Từ phản hồi này, lệnh `go` sẽ sử dụng kho lưu trữ Git tại URL từ xa `https://go.googlesource.com/mod`.

GitHub và các dịch vụ lưu trữ phổ biến khác phản hồi các truy vấn `?go-get=1` cho tất cả kho lưu trữ, vì vậy thông thường không cần cấu hình máy chủ cho các mô-đun được lưu trữ trên những trang đó.

Sau khi tìm thấy URL của kho lưu trữ, lệnh `go` sẽ sao chép kho lưu trữ vào bộ nhớ đệm mô-đun. Nhìn chung, lệnh `go` cố gắng tránh lấy dữ liệu không cần thiết từ một kho lưu trữ. Tuy nhiên, các lệnh thực tế được sử dụng thay đổi theo hệ thống quản lý phiên bản và có thể thay đổi theo thời gian. Với Git, lệnh `go` có thể liệt kê hầu hết các phiên bản có sẵn mà không cần tải xuống các commit. Nó thường sẽ lấy các commit mà không tải xuống các commit tổ tiên, nhưng đôi khi việc đó là cần thiết.

### Ánh xạ phiên bản tới các commit {#vcs-version}

Lệnh `go` có thể checkout một module trong một kho lưu trữ tại một [phiên bản chuẩn](#glos-canonical-version) cụ thể như `v1.2.3`, `v2.4.0-beta` hoặc `v3.0.0+incompatible`. Mỗi phiên bản module nên có một <dfn>thẻ phiên bản ngữ nghĩa</dfn> trong kho lưu trữ, cho biết revision nào cần được checkout cho một phiên bản nhất định.

Nếu một module được định nghĩa trong thư mục gốc của kho lưu trữ hoặc trong một thư mục con phiên bản major của thư mục gốc, thì tên của mỗi thẻ phiên bản bằng với phiên bản tương ứng. Ví dụ, module `golang.org/x/text` được định nghĩa trong thư mục gốc của kho lưu trữ, vì vậy phiên bản `v0.3.2` có thẻ `v0.3.2` trong kho lưu trữ đó. Điều này đúng với hầu hết các module.

Nếu một module được định nghĩa trong một thư mục con bên trong kho lưu trữ, tức là phần [thư mục con của module](#glos-module-subdirectory) trong đường dẫn module không rỗng, thì mỗi tên thẻ phải được thêm tiền tố là thư mục con của module, theo sau bởi dấu gạch chéo. Ví dụ, module `golang.org/x/tools/gopls` được định nghĩa trong thư mục con `gopls` của kho lưu trữ có đường dẫn gốc `golang.org/x/tools`. Phiên bản `v0.4.0` của module đó phải có thẻ có tên `gopls/v0.4.0` trong kho lưu trữ đó.

Số phiên bản major của một thẻ phiên bản ngữ nghĩa phải nhất quán với hậu tố phiên bản major của đường dẫn module (nếu có). Ví dụ, thẻ `v1.0.0` có thể thuộc về module `example.com/mod` nhưng không thể thuộc về `example.com/mod/v2`, vì module đó sẽ có các thẻ như `v2.0.0`.

Một thẻ có phiên bản major `v2` trở lên có thể thuộc về một module không có hậu tố phiên bản major nếu không có tệp `go.mod` và module nằm trong thư mục gốc của kho lưu trữ. Loại phiên bản này được biểu thị bằng hậu tố `+incompatible`. Bản thân thẻ phiên bản không được có hậu tố này. Xem [Tính tương thích với các kho lưu trữ không phải module](#non-module-compat).

Sau khi một thẻ được tạo, không nên xóa hoặc thay đổi nó sang một revision khác. Các phiên bản được [xác thực](#authenticating) để đảm bảo các bản build an toàn và có thể lặp lại. Nếu một thẻ bị thay đổi, client có thể gặp lỗi bảo mật khi tải xuống thẻ đó. Ngay cả sau khi một thẻ bị xóa, nội dung của nó vẫn có thể còn khả dụng trên [các module proxy](#glos-module-proxy).

### Ánh xạ pseudo-version tới commit {#vcs-pseudo}

Lệnh `go` có thể kiểm tra một module trong một repository tại một revision cụ thể, được mã hóa dưới dạng [pseudo-version](#glos-pseudo-version) như
`v1.3.2-0.20191109021931-daa7c04131f5`.

12 ký tự cuối của pseudo-version (`daa7c04131f5` trong ví dụ trên) chỉ ra một revision trong repository cần được kiểm tra. Ý nghĩa của phần này phụ thuộc vào hệ thống quản lý phiên bản. Với Git và Mercurial, đây là tiền tố của một commit hash. Với Subversion, đây là một số revision được đệm bằng số 0.

Trước khi kiểm tra một commit, lệnh `go` xác minh rằng dấu thời gian (`20191109021931` ở trên) khớp với ngày của commit. Lệnh cũng xác minh rằng phiên bản cơ sở (`v1.3.1`, phiên bản trước `v1.3.2` trong ví dụ trên) tương ứng với một thẻ phiên bản semantic là tổ tiên của commit. Các kiểm tra này đảm bảo rằng tác giả module có toàn quyền kiểm soát cách các pseudo-version được so sánh với các phiên bản đã phát hành khác.

Xem [Pseudo-versions](#pseudo-versions) để biết thêm thông tin.

### Ánh xạ branch và commit tới phiên bản {#vcs-branch}

Một module có thể được kiểm tra tại một branch, tag hoặc revision cụ thể bằng cách sử dụng [truy vấn phiên bản](#version-queries).

```
go get example.com/mod@master
```

Lệnh `go` chuyển đổi các tên này thành [phiên bản chuẩn](#glos-canonical-version) có thể được sử dụng với [lựa chọn phiên bản tối thiểu (MVS)](#minimal-version-selection). MVS phụ thuộc vào khả năng sắp xếp các phiên bản một cách không mơ hồ. Tên branch và revision không thể được so sánh đáng tin cậy theo thời gian, vì chúng phụ thuộc vào cấu trúc repository, vốn có thể thay đổi.

Nếu một revision được gắn một hoặc nhiều tag phiên bản semantic như `v1.2.3`, tag cho phiên bản hợp lệ cao nhất sẽ được sử dụng. Lệnh `go` chỉ xem xét các tag phiên bản semantic có thể thuộc về module đích; ví dụ, tag `v1.5.2` sẽ không được xem xét cho `example.com/mod/v2` vì phiên bản chính không khớp với hậu tố của đường dẫn module.

Nếu một revision không được gắn tag phiên bản semantic hợp lệ, lệnh `go` sẽ tạo một [pseudo-version](#glos-pseudo-version). Nếu revision có các tổ tiên với tag phiên bản semantic hợp lệ, phiên bản tổ tiên cao nhất sẽ được sử dụng làm cơ sở của pseudo-version. Xem [Pseudo-versions](#pseudo-versions).

### Thư mục module trong một kho lưu trữ {#vcs-dir}

Sau khi kho lưu trữ của một module đã được checkout tại một revision cụ thể, lệnh `go` phải tìm thư mục chứa tệp `go.mod` của module (thư mục gốc của module).

Hãy nhớ rằng [đường dẫn module](#module-path) bao gồm ba phần: đường dẫn gốc của kho lưu trữ (tương ứng với thư mục gốc của kho lưu trữ), một thư mục con của module và hậu tố phiên bản chính (chỉ dành cho các module được bản phát hành tại `v2` trở lên).

Với hầu hết module, đường dẫn module bằng với đường dẫn gốc của kho lưu trữ, vì vậy thư mục gốc của module là thư mục gốc của kho lưu trữ.

Các module đôi khi được định nghĩa trong các thư mục con của kho lưu trữ. Điều này thường được thực hiện cho các kho lưu trữ lớn có nhiều thành phần cần được bản phát hành và tạo phiên bản độc lập. Một module như vậy được kỳ vọng nằm trong một thư mục con khớp với phần trong đường dẫn module sau đường dẫn gốc của kho lưu trữ. Ví dụ: giả sử module `example.com/monorepo/foo/bar` nằm trong kho lưu trữ có đường dẫn gốc `example.com/monorepo`. Tệp `go.mod` của nó phải nằm trong thư mục con `foo/bar`.

Nếu một module được bản phát hành tại phiên bản chính `v2` trở lên, đường dẫn của nó phải có [hậu tố phiên bản chính](#major-version-suffixes). Một module có hậu tố phiên bản chính có thể được định nghĩa trong một trong hai thư mục con: một thư mục có hậu tố và một thư mục không có hậu tố. Ví dụ: giả sử một phiên bản mới của module ở trên được bản phát hành với đường dẫn `example.com/monorepo/foo/bar/v2`. Tệp `go.mod` của nó có thể nằm trong `foo/bar` hoặc `foo/bar/v2`.

Các thư mục con có hậu tố phiên bản chính là <dfn>thư mục con phiên bản chính</dfn>. Chúng có thể được dùng để phát triển nhiều phiên bản chính của một module trên cùng một branch. Điều này có thể không cần thiết khi việc phát triển nhiều phiên bản chính diễn ra trên các branch riêng biệt. Tuy nhiên, các thư mục con phiên bản chính có một thuộc tính quan trọng: trong chế độ `GOPATH`, đường dẫn import gói khớp chính xác với các thư mục bên dưới `GOPATH/src`. Lệnh `go` cung cấp khả năng tương thích module tối thiểu trong chế độ `GOPATH` (xem [Khả năng tương thích với các kho lưu trữ không phải module](#non-module-compat)), vì vậy các thư mục con phiên bản chính không phải lúc nào cũng cần thiết để tương thích với các dự án được xây dựng trong chế độ `GOPATH`. Tuy nhiên, các công cụ cũ không hỗ trợ khả năng tương thích module tối thiểu có thể gặp vấn đề.

Sau khi lệnh `go` tìm thấy thư mục gốc của module, nó tạo một tệp `.zip` chứa nội dung của thư mục, sau đó giải nén tệp `.zip` vào bộ đệm module. Xem [các ràng buộc về đường dẫn và kích thước tệp](#zip-path-size-constraints) để biết chi tiết về các tệp có thể được đưa vào tệp `.zip`. Nội dung của tệp `.zip` được [xác thực](#authenticating) trước khi giải nén vào bộ đệm module theo cùng cách như khi tệp `.zip` được tải xuống từ một proxy.

Các tệp zip của module không bao gồm nội dung của các thư mục `vendor` hoặc bất kỳ module lồng nhau nào (các thư mục con chứa tệp `go.mod`). Điều này có nghĩa là một module phải cẩn thận không tham chiếu đến các tệp bên ngoài thư mục của nó hoặc trong các module khác. Ví dụ: các mẫu [`//go:embed`](https://pkg.go.dev/embed#hdr-Directives) không được khớp với các tệp trong các module lồng nhau. Hành vi này có thể là một giải pháp thay thế hữu ích trong các trường hợp không nên đưa tệp vào một module. Ví dụ: nếu một kho lưu trữ có các tệp lớn được đưa vào thư mục `testdata`, tác giả module có thể thêm một tệp `go.mod` rỗng vào `testdata` để người dùng của họ không cần tải xuống các tệp đó. Tất nhiên, điều này có thể làm giảm độ phủ đối với người dùng đang kiểm thử các dependency của họ.

### Trường hợp đặc biệt đối với tệp LICENSE {#vcs-license}

Khi lệnh `go` tạo tệp `.zip` cho một mô-đun không nằm trong thư mục gốc của kho lưu trữ, nếu mô-đun không có tệp có tên `LICENSE` trong thư mục gốc của nó (cùng với `go.mod`), lệnh `go` sẽ sao chép tệp có tên `LICENSE` từ thư mục gốc của kho lưu trữ nếu tệp đó có trong cùng một bản sửa đổi.

Trường hợp đặc biệt này cho phép cùng một tệp `LICENSE` được áp dụng cho tất cả các mô-đun trong một kho lưu trữ. Điều này chỉ áp dụng cho các tệp có tên chính xác là `LICENSE`, không có phần mở rộng như `.txt`. Đáng tiếc, điều này không thể được mở rộng mà không làm hỏng các tổng kiểm tra mật mã của các mô-đun hiện có; xem [Xác thực mô-đun](#authenticating). Các công cụ và trang web khác như [pkg.go.dev](https://pkg.go.dev) có thể nhận ra các tệp có tên khác.

Cũng lưu ý rằng lệnh `go` không bao gồm các liên kết tượng trưng khi tạo tệp `.zip` của mô-đun; xem [Các ràng buộc về đường dẫn và kích thước tệp](#zip-path-size-constraints). Do đó, nếu một kho lưu trữ không có tệp `LICENSE` trong thư mục gốc, các tác giả có thể thay vào đó tạo các bản sao của tệp giấy phép trong các mô-đun được định nghĩa trong thư mục con để đảm bảo các tệp đó được đưa vào tệp `.zip` của mô-đun.

### Kiểm soát các công cụ quản lý phiên bản bằng `GOVCS` {#vcs-govcs}

Khả năng tải xuống mô-đun bằng các lệnh quản lý phiên bản như `git` của lệnh `go` rất quan trọng đối với hệ sinh thái gói phi tập trung, trong đó mã có thể được nhập từ bất kỳ máy chủ nào. Khả năng này cũng là một vấn đề bảo mật tiềm ẩn nếu một máy chủ độc hại tìm ra cách khiến lệnh quản lý phiên bản được gọi chạy mã không mong muốn.

Để cân bằng các mối quan tâm về chức năng và bảo mật, theo mặc định lệnh `go` chỉ sử dụng `git` và `hg` để tải xuống mã từ các máy chủ công khai. Nó sẽ sử dụng bất kỳ [hệ thống quản lý phiên bản đã biết](#vcs-support) nào để tải xuống mã từ các máy chủ riêng tư, được định nghĩa là những máy chủ lưu trữ các gói khớp với biến [môi trường `GOPRIVATE`](#environment-variables). Lý do chỉ cho phép Git và Mercurial là vì hai hệ thống này đã nhận được nhiều sự chú ý nhất về các vấn đề khi được chạy với vai trò máy khách của các máy chủ không đáng tin cậy. Ngược lại, Bazaar, Fossil và Subversion chủ yếu được sử dụng trong các môi trường đáng tin cậy, đã được xác thực và không được xem xét kỹ lưỡng về các bề mặt tấn công như vậy.

Các hạn chế đối với lệnh quản lý phiên bản chỉ áp dụng khi sử dụng quyền truy cập quản lý phiên bản trực tiếp để tải xuống mã. Khi tải xuống mô-đun từ proxy, lệnh `go` sử dụng [giao thức `GOPROXY`](#goproxy-protocol) thay thế, giao thức này luôn được cho phép. Theo mặc định, lệnh `go` sử dụng máy nhân bản mô-đun Go ([proxy.golang.org](https://proxy.golang.org)) cho các mô-đun công khai và chỉ chuyển sang quản lý phiên bản đối với các mô-đun riêng tư hoặc khi máy nhân bản từ chối phục vụ một gói công khai (thường vì lý do pháp lý). Do đó, theo mặc định, người dùng vẫn có thể truy cập mã công khai được phục vụ từ các kho lưu trữ Bazaar, Fossil hoặc Subversion, vì các lượt tải xuống đó sử dụng máy nhân bản mô-đun Go, nơi chịu rủi ro bảo mật khi chạy các lệnh quản lý phiên bản bằng một hộp cát tùy chỉnh.

Biến `GOVCS` có thể được sử dụng để thay đổi các hệ thống quản lý phiên bản được phép cho các mô-đun cụ thể. Biến `GOVCS` áp dụng khi xây dựng các gói ở cả chế độ nhận biết mô-đun và chế độ GOPATH. Khi sử dụng mô-đun, các mẫu được so khớp với đường dẫn mô-đun. Khi sử dụng GOPATH, các mẫu được so khớp với đường dẫn nhập tương ứng với gốc của kho lưu trữ quản lý phiên bản.

Dạng tổng quát của biến `GOVCS` là danh sách các quy tắc `pattern:vcslist` được phân tách bằng dấu phẩy. Mẫu là một [mẫu glob](/pkg/path#Match) phải khớp với một hoặc nhiều phần tử đầu tiên của đường dẫn mô-đun hoặc đường dẫn nhập. vcslist là danh sách các lệnh quản lý phiên bản được phép được phân tách bằng dấu gạch đứng, hoặc `all` để cho phép sử dụng bất kỳ lệnh nào đã biết, hoặc `off` để không cho phép gì cả. Lưu ý rằng nếu một mô-đun khớp với một mẫu có vcslist `off`, mô-đun đó vẫn có thể được tải xuống nếu máy chủ gốc sử dụng lược đồ `mod`, lược đồ này hướng dẫn lệnh go tải xuống mô-đun bằng [giao thức `GOPROXY`](#goproxy-protocol). Mẫu khớp sớm nhất trong danh sách sẽ được áp dụng, ngay cả khi các mẫu sau cũng có thể khớp.

Ví dụ, hãy xem xét:

```
GOVCS=github.com:git,evil.com:off,*:git|hg
```

Với thiết lập này, mã có mô-đun hoặc đường dẫn nhập bắt đầu bằng `github.com/` chỉ có thể sử dụng `git`; các đường dẫn trên `evil.com` không thể sử dụng bất kỳ lệnh quản lý phiên bản nào, và tất cả các đường dẫn khác (`*` khớp với mọi thứ) chỉ có thể sử dụng `git` hoặc `hg`.

Các mẫu đặc biệt `public` và `private` khớp với các đường dẫn mô-đun hoặc đường dẫn nhập công khai và riêng tư. Một đường dẫn là riêng tư nếu nó khớp với biến `GOPPRIVATE`; nếu không thì nó là công khai.

Nếu không có quy tắc nào trong biến `GOVCS` khớp với một mô-đun hoặc đường dẫn nhập cụ thể, lệnh `go` áp dụng quy tắc mặc định của nó, hiện có thể được tóm tắt bằng ký hiệu `GOVCS` là `public:git|hg,private:all`.

Để cho phép sử dụng không hạn chế bất kỳ hệ thống quản lý phiên bản nào cho bất kỳ gói nào, hãy dùng:

```
GOVCS=*:all
```

Để vô hiệu hóa toàn bộ việc sử dụng quản lý phiên bản, hãy dùng:

```
GOVCS=*:off
```

Có thể sử dụng [lệnh `go env -w` command](/cmd/go/#hdr-Print_Go_environment_information) để đặt biến `GOVCS` cho các lần gọi lệnh go trong tương lai.

`GOVCS` được giới thiệu trong Go 1.16. Các phiên bản Go trước đó có thể sử dụng bất kỳ công cụ quản lý phiên bản nào đã biết cho bất kỳ mô-đun nào.

## Tệp zip của mô-đun {#zip-files}

Các phiên bản mô-đun được phân phối dưới dạng tệp `.zip`. Hiếm khi cần tương tác trực tiếp với các tệp này, vì lệnh `go` tự động tạo, tải xuống và giải nén chúng từ [module proxies](#glos-module-proxy) và các repository kiểm soát phiên bản. Tuy nhiên, việc hiểu về các tệp này vẫn hữu ích để nắm được các ràng buộc tương thích đa nền tảng hoặc khi triển khai một module proxy.

Lệnh [`go mod download`](#go-mod-download) tải xuống các tệp zip cho một hoặc nhiều mô-đun, sau đó giải nén các tệp đó vào [bộ nhớ đệm mô-đun](#glos-module-cache). Tùy thuộc vào `GOPROXY` và các [biến môi trường](#environment-variables) khác, lệnh `go` có thể tải các tệp zip từ một proxy hoặc sao chép các repository mã nguồn và tạo tệp zip từ chúng. Cờ `-json` có thể được dùng để tìm vị trí của các tệp zip đã tải xuống và nội dung đã giải nén của chúng trong bộ nhớ đệm mô-đun.

Gói [`golang.org/x/mod/zip`](https://pkg.go.dev/golang.org/x/mod/zip?tab=doc) có thể được dùng để tạo, giải nén hoặc kiểm tra nội dung của các tệp zip theo chương trình.

### Ràng buộc về đường dẫn tệp và kích thước {#zip-path-size-constraints}

Có một số hạn chế đối với nội dung của các tệp zip mô-đun. Những ràng buộc này đảm bảo rằng các tệp zip có thể được giải nén an toàn và nhất quán trên nhiều nền tảng.

* Một tệp zip mô-đun có thể có kích thước tối đa 500 MiB. Tổng kích thước chưa nén của các tệp trong đó cũng bị giới hạn ở mức 500 MiB. Các tệp `go.mod` bị giới hạn ở 16 MiB. Các tệp `LICENSE` cũng bị giới hạn ở 16 MiB. Những giới hạn này tồn tại để giảm thiểu các cuộc tấn công từ chối dịch vụ nhắm vào người dùng, proxy và các phần khác của hệ sinh thái mô-đun. Các repository chứa hơn 500 MiB tệp trong cây thư mục mô-đun nên gắn thẻ các phiên bản mô-đun tại các commit chỉ bao gồm những tệp cần thiết để xây dựng các gói của mô-đun; video, mô hình và các tài sản lớn khác thường không cần thiết cho việc xây dựng.
* Mỗi tệp trong một tệp zip mô-đun phải bắt đầu bằng tiền tố ` $module@$version/` trong đó `$module` là đường dẫn mô-đun và `$version` là phiên bản, ví dụ, `golang.org/x/mod@v0.3.0/`. Đường dẫn mô-đun phải hợp lệ, phiên bản phải hợp lệ và chuẩn, đồng thời phiên bản phải khớp với hậu tố phiên bản chính của đường dẫn mô-đun. Xem [Đường dẫn và phiên bản mô-đun](#go-mod-file-ident) để biết các định nghĩa và hạn chế cụ thể.
* Chế độ tệp, dấu thời gian và các siêu dữ liệu khác bị bỏ qua.
* Các thư mục rỗng (các mục có đường dẫn kết thúc bằng dấu gạch chéo) có thể được bao gồm trong các tệp zip mô-đun nhưng không được giải nén. Lệnh `go` không bao gồm các thư mục rỗng trong các tệp zip mà nó tạo.
* Các liên kết tượng trưng và các tệp bất thường khác bị bỏ qua khi tạo tệp zip, vì chúng không thể di chuyển giữa các hệ điều hành và hệ thống tệp, đồng thời không có cách di động nào để biểu diễn chúng trong định dạng tệp zip.
* Các tệp bên trong các thư mục có tên `vendor` bị bỏ qua khi tạo tệp zip, vì các thư mục `vendor` bên ngoài mô-đun chính không bao giờ được sử dụng.
* Các tệp bên trong các thư mục chứa tệp `go.mod`, ngoài thư mục gốc của mô-đun, bị bỏ qua khi tạo tệp zip, vì chúng không thuộc về mô-đun. Lệnh `go` bỏ qua các thư mục con chứa tệp `go.mod` khi giải nén tệp zip.
* Không có hai tệp nào trong một tệp zip có thể có đường dẫn giống nhau theo quy tắc gấp chữ hoa chữ thường Unicode (xem [`strings.EqualFold`](https://pkg.go.dev/strings?tab=doc#EqualFold)). Điều này đảm bảo rằng các tệp zip có thể được giải nén trên các hệ thống tệp không phân biệt chữ hoa chữ thường mà không xảy ra xung đột.
* Một tệp `go.mod` có thể xuất hiện hoặc không xuất hiện trong thư mục cấp cao nhất (`$module@$version/go.mod`). Nếu có, nó phải có tên `go.mod` (toàn bộ chữ thường). Các tệp có tên `go.mod` không được phép xuất hiện trong bất kỳ thư mục nào khác.
* Tên tệp và thư mục trong một mô-đun có thể bao gồm các chữ cái Unicode, chữ số ASCII, ký tự khoảng trắng ASCII (U+0020) và các ký tự dấu câu ASCII `!#$%&()+,-.=@[]^_{}~`. Lưu ý rằng đường dẫn gói có thể không chứa tất cả các ký tự này. Xem [`module.CheckFilePath`](https://pkg.go.dev/golang.org/x/mod/module?tab=doc#CheckFilePath) và [`module.CheckImportPath`](https://pkg.go.dev/golang.org/x/mod/module?tab=doc#CheckImportPath) để biết sự khác biệt.
* Tên tệp hoặc thư mục cho đến dấu chấm đầu tiên không được là tên tệp dành riêng trên Windows, bất kể cách viết hoa chữ thường (`CON`, `com1`, `NuL`, v.v.).

## Mô-đun riêng tư {#private-modules}

Các mô-đun Go thường được phát triển và phân phối trên các máy chủ kiểm soát phiên bản và proxy mô-đun không có sẵn trên Internet công cộng. Lệnh `go` có thể tải xuống và xây dựng các mô-đun từ nguồn riêng tư, mặc dù thường yêu cầu một số cấu hình.

Các biến môi trường dưới đây có thể được sử dụng để cấu hình quyền truy cập vào các mô-đun riêng tư. Xem [Các biến môi trường](#environment-variables) để biết chi tiết. Xem thêm [Quyền riêng tư](#private-module-privacy) để biết thông tin về việc kiểm soát thông tin được gửi đến các máy chủ công cộng.

* `GOPROXY` — danh sách các URL proxy mô-đun. Lệnh `go` sẽ thử tải xuống mô-đun từ từng máy chủ theo thứ tự. Từ khóa `direct` yêu cầu lệnh `go` tải xuống mô-đun từ các repository kiểm soát phiên bản nơi chúng được phát triển thay vì sử dụng proxy.
* `GOPRIVATE` — danh sách các mẫu glob của tiền tố đường dẫn mô-đun nên được xem là riêng tư. Hoạt động như giá trị mặc định cho `GONOPROXY` và `GONOSUMDB`.
* `GONOPROXY` — danh sách các mẫu glob của tiền tố đường dẫn mô-đun không nên được tải xuống từ proxy. Lệnh `go` sẽ tải xuống các mô-đun khớp với mẫu từ các repository kiểm soát phiên bản nơi chúng được phát triển, bất kể `GOPROXY`.
* `GONOSUMDB` — danh sách các mẫu glob của tiền tố đường dẫn mô-đun không nên được kiểm tra bằng cơ sở dữ liệu checksum công cộng, [sum.golang.org](https://sum.golang.org).
* `GOINSECURE` — danh sách các mẫu glob của tiền tố đường dẫn mô-đun có thể được truy xuất qua HTTP và các giao thức không an toàn khác.

Các biến này có thể được đặt trong môi trường phát triển (ví dụ: trong tệp `.profile`), hoặc có thể được đặt vĩnh viễn bằng [`go env -w`](/cmd/go/#hdr-Print_Go_environment_information).

Phần còn lại của mục này mô tả các mẫu phổ biến để cung cấp quyền truy cập vào các proxy mô-đun riêng tư và repository kiểm soát phiên bản.

### Proxy riêng tư phục vụ tất cả mô-đun {#private-module-proxy-all}

Một máy chủ proxy riêng tư trung tâm phục vụ tất cả mô-đun (công cộng và riêng tư) cung cấp khả năng kiểm soát nhiều nhất cho quản trị viên và yêu cầu ít cấu hình nhất cho từng nhà phát triển.

Để cấu hình lệnh `go` sử dụng một máy chủ như vậy, hãy đặt các biến môi trường sau, thay thế `https://proxy.corp.example.com` bằng URL proxy của bạn và `corp.example.com` bằng tiền tố mô-đun của bạn:

```
GOPROXY=https://proxy.corp.example.com
GONOSUMDB=corp.example.com
```

Thiết lập `GOPROXY` yêu cầu lệnh `go` chỉ tải xuống mô-đun từ `https://proxy.corp.example.com`; lệnh `go` sẽ không kết nối với các proxy khác hoặc các repository kiểm soát phiên bản.

Thiết lập `GONOSUMDB` yêu cầu lệnh `go` không sử dụng cơ sở dữ liệu checksum công cộng để xác thực các mô-đun có đường dẫn bắt đầu bằng `corp.example.com`.

Một proxy chạy theo cấu hình này có thể cần quyền truy cập đọc vào các máy chủ kiểm soát phiên bản riêng tư. Proxy này cũng cần quyền truy cập Internet công cộng để tải xuống các phiên bản mới của các mô-đun công cộng.

Có một số triển khai máy chủ `GOPROXY` hiện có có thể được sử dụng theo cách này. Một triển khai tối thiểu sẽ phục vụ các tệp từ thư mục [bộ nhớ đệm mô-đun](#glos-module-cache) và sẽ sử dụng [`go mod download`](#go-mod-download) (với cấu hình phù hợp) để truy xuất các mô-đun còn thiếu.

### Proxy riêng tư phục vụ các mô-đun riêng tư {#private-module-proxy-private}

Một máy chủ proxy riêng tư có thể phục vụ các mô-đun riêng tư mà không đồng thời phục vụ các mô-đun có sẵn công khai. Lệnh `go` có thể được cấu hình để quay lại các nguồn công khai cho những mô-đun không có trên máy chủ riêng tư.

Để cấu hình lệnh `go` hoạt động theo cách này, hãy đặt các biến môi trường sau, thay thế `https://proxy.corp.example.com` bằng URL proxy và `corp.example.com` bằng tiền tố mô-đun:

```
GOPROXY=https://proxy.corp.example.com,https://proxy.golang.org,direct
GONOSUMDB=corp.example.com
```

Thiết lập `GOPROXY` hướng dẫn lệnh `go` thử tải xuống các mô-đun từ `https://proxy.corp.example.com` trước. Nếu máy chủ đó phản hồi với 404 (Not Found) hoặc 410 (Gone), lệnh `go` sẽ quay lại `https://proxy.golang.org`, sau đó kết nối trực tiếp đến các kho lưu trữ.

Thiết lập `GONOSUMDB` hướng dẫn lệnh `go` không sử dụng cơ sở dữ liệu tổng kiểm tra công khai để xác thực các mô-đun có đường dẫn bắt đầu bằng `corp.example.com`.

Lưu ý rằng một proxy được sử dụng trong cấu hình này vẫn có thể kiểm soát quyền truy cập vào các mô-đun công khai, dù nó không phục vụ các mô-đun đó. Nếu proxy phản hồi một yêu cầu bằng trạng thái lỗi khác ngoài 404 hoặc 410, lệnh `go` sẽ không quay lại các mục sau trong danh sách `GOPROXY`. Ví dụ: proxy có thể phản hồi 403 (Forbidden) đối với một mô-đun có giấy phép không phù hợp hoặc có lỗ hổng bảo mật đã biết.

### Truy cập trực tiếp vào các mô-đun riêng tư {#private-module-proxy-direct}

Lệnh `go` có thể được cấu hình để bỏ qua các proxy công khai và tải xuống các mô-đun riêng tư trực tiếp từ các máy chủ kiểm soát phiên bản. Điều này hữu ích khi việc chạy một máy chủ proxy riêng tư không khả thi.

Để cấu hình lệnh `go` hoạt động theo cách này, hãy đặt `GOPRIVATE`, thay thế `corp.example.com` bằng tiền tố mô-đun riêng tư:

```
GOPRIVATE=corp.example.com
```

Biến `GOPROXY` không cần thay đổi trong tình huống này. Giá trị mặc định của nó là `https://proxy.golang.org,direct`, giá trị này hướng dẫn lệnh `go` thử tải xuống các mô-đun từ `https://proxy.golang.org` trước, sau đó quay lại kết nối trực tiếp nếu proxy đó phản hồi với 404 (Not Found) hoặc 410 (Gone).

Thiết lập `GOPRIVATE` hướng dẫn lệnh `go` không kết nối tới proxy hoặc cơ sở dữ liệu tổng kiểm tra đối với các mô-đun bắt đầu bằng `corp.example.com`.

Một máy chủ HTTP nội bộ vẫn có thể cần thiết để [giải quyết đường dẫn mô-đun thành URL kho lưu trữ](#vcs-find). Ví dụ: khi lệnh `go` tải xuống mô-đun `corp.example.com/mod`, nó sẽ gửi một yêu cầu GET tới `https://corp.example.com/mod?go-get=1`, và tìm URL kho lưu trữ trong phản hồi. Để tránh yêu cầu này, hãy đảm bảo mỗi đường dẫn mô-đun riêng tư có hậu tố VCS (như `.git`) đánh dấu tiền tố gốc của kho lưu trữ. Ví dụ: khi lệnh `go` tải xuống mô-đun `corp.example.com/repo.git/mod`, nó sẽ sao chép kho lưu trữ Git tại `https://corp.example.com/repo.git` hoặc `ssh://corp.example.com/repo.git` mà không cần thực hiện thêm các yêu cầu khác.

Nhà phát triển sẽ cần quyền đọc đối với các kho lưu trữ chứa các mô-đun riêng tư. Điều này có thể được cấu hình trong các tệp cấu hình VCS toàn cục như `.gitconfig`. Tốt nhất là các công cụ VCS được cấu hình để không cần các lời nhắc xác thực tương tác. Theo mặc định, khi gọi Git, lệnh `go` vô hiệu hóa các lời nhắc tương tác bằng cách đặt `GIT_TERMINAL_PROMPT=0`, nhưng vẫn tôn trọng các thiết lập rõ ràng.

### Truyền thông tin xác thực đến các proxy riêng tư {#private-module-proxy-auth}

Lệnh `go` có thể xác thực với các máy chủ proxy bằng [xác thực cơ bản](https://en.wikipedia.org/wiki/Basic_access_authentication) HTTP hoặc các tiêu đề yêu cầu được cung cấp bởi `GOAUTH`.

Thông tin xác thực có thể được chỉ định trong một [tệp `.netrc`](https://www.gnu.org/software/inetutils/manual/html_node/The-_002enetrc-file.html).
Ví dụ: một tệp `.netrc` chứa các dòng bên dưới sẽ cấu hình lệnh `go` kết nối với máy `proxy.corp.example.com` bằng tên người dùng và mật khẩu đã cho.

```
machine proxy.corp.example.com
login jrgopher
password hunter2
```

Vị trí của tệp có thể được đặt bằng biến môi trường `NETRC`. Nếu `NETRC` không được đặt, lệnh `go` sẽ đọc `$HOME/.netrc` trên các nền tảng giống UNIX hoặc `%USERPROFILE%\_netrc` trên Windows.

Các trường trong `.netrc` được phân tách bằng dấu cách, tab và dòng mới. Đáng tiếc là các ký tự này không thể được sử dụng trong tên người dùng hoặc mật khẩu. Cũng lưu ý rằng tên máy không thể là một URL đầy đủ, vì vậy không thể chỉ định các tên người dùng và mật khẩu khác nhau cho các đường dẫn khác nhau trên cùng một máy.

Trong Go 1.24 trở lên, `GOAUTH` có thể sử dụng `.netrc`, chạy `git credential fill` trong một thư mục làm việc được chỉ định, hoặc gọi một lệnh tùy chỉnh. Một lệnh tùy chỉnh có thể cung cấp các tiêu đề yêu cầu HTTP tùy ý, chẳng hạn như mã thông báo Bearer, cho một hoặc nhiều tiền tố URL HTTPS. Điều này cho phép giới hạn phạm vi của thông tin xác thực theo một đường dẫn trên máy chủ proxy, điều mà `.netrc` không thể thực hiện. Xem `go help goauth` để biết giao thức lệnh và các tùy chọn cấu hình.

Lệnh `go` chỉ áp dụng thông tin xác thực thu được thông qua `GOAUTH`, bao gồm phương thức `netrc` mặc định của nó, cho các yêu cầu HTTPS.

Ngoài ra, thông tin xác thực có thể được chỉ định trực tiếp trong các URL `GOPROXY`. Ví dụ:

```
GOPROXY=https://jrgopher:hunter2@proxy.corp.example.com
```

Hãy cẩn thận khi sử dụng cách tiếp cận này: các biến môi trường có thể xuất hiện trong lịch sử shell và trong nhật ký.

### Truyền thông tin xác thực đến các kho lưu trữ riêng tư {#private-module-repo-auth}

Lệnh `go` có thể tải xuống một module trực tiếp từ một kho lưu trữ kiểm soát phiên bản. Điều này cần thiết cho các module riêng tư nếu không sử dụng proxy riêng tư. Xem [Truy cập trực tiếp vào các module riêng tư](#private-module-proxy-direct) để biết cấu hình.

Lệnh `go` chạy các công cụ kiểm soát phiên bản như `git` khi tải xuống các module trực tiếp. Các công cụ này thực hiện việc xác thực riêng của chúng, vì vậy bạn có thể cần cấu hình thông tin xác thực trong một tệp cấu hình dành riêng cho công cụ như `.gitconfig`.

Để đảm bảo hoạt động trơn tru, hãy đảm bảo lệnh `go` sử dụng đúng URL của kho lưu trữ và công cụ kiểm soát phiên bản không yêu cầu nhập mật khẩu tương tác. Lệnh `go` ưu tiên các URL `https://` hơn các lược đồ khác như `ssh://` trừ khi lược đồ đã được chỉ định khi [tra cứu URL của kho lưu trữ](#vcs-find). Riêng với các kho lưu trữ GitHub, lệnh `go` giả định sử dụng `https://`.

<!-- TODO(golang.org/issue/26134): nếu vấn đề này được khắc phục, chúng ta có thể xóa
đề cập về trường hợp đặc biệt cho GitHub ở trên. -->

Đối với hầu hết máy chủ, bạn có thể cấu hình máy khách để xác thực qua HTTP. Ví dụ: GitHub hỗ trợ sử dụng [mã thông báo truy cập cá nhân OAuth làm mật khẩu HTTP](https://docs.github.com/en/free-pro-team@latest/github/extending-github/git-automation-with-oauth-tokens).
Bạn có thể lưu trữ mật khẩu HTTP trong tệp `.netrc`, giống như khi [truyền thông tin xác thực đến các proxy riêng tư](#private-module-proxy-auth).

Ngoài ra, bạn có thể viết lại các URL `https://` thành một lược đồ khác. Ví dụ, trong `.gitconfig`:

```
[url "git@github.com:"]
    insteadOf = https://github.com/
```

Để biết thêm thông tin, xem [Tại sao "go get" sử dụng HTTPS khi sao chép một kho lưu trữ?](/doc/faq#git_https)

### Quyền riêng tư {#private-module-privacy}

Lệnh `go` có thể tải xuống các module và metadata từ các máy chủ proxy module và các hệ thống quản lý phiên bản. Biến môi trường `GOPROXY` kiểm soát các máy chủ nào được sử dụng. Các biến môi trường `GOPRIVATE` và `GONOPROXY` kiểm soát những module nào được lấy từ proxy.

Giá trị mặc định của `GOPROXY` là:

```
https://proxy.golang.org,direct
```

Với thiết lập này, khi lệnh `go` tải xuống một module hoặc metadata của module, trước tiên nó sẽ gửi một yêu cầu đến `proxy.golang.org`, một proxy module công khai do Google vận hành ([chính sách quyền riêng tư](https://proxy.golang.org/privacy)). Xem [giao thức `GOPROXY`](#goproxy-protocol) để biết chi tiết về thông tin được gửi trong mỗi yêu cầu. Lệnh `go` không truyền thông tin nhận dạng cá nhân, nhưng nó có truyền toàn bộ đường dẫn module đang được yêu cầu. Nếu proxy phản hồi với trạng thái 404 (Not Found) hoặc 410 (Gone), lệnh `go` sẽ cố gắng kết nối trực tiếp đến hệ thống quản lý phiên bản cung cấp module đó. Xem [Các hệ thống quản lý phiên bản](#vcs) để biết chi tiết.

Các biến môi trường `GOPRIVATE` hoặc `GONOPROXY` có thể được đặt thành danh sách các mẫu glob khớp với tiền tố module là riêng tư và không nên được yêu cầu từ bất kỳ proxy nào. Ví dụ:

```
GOPRIVATE=*.corp.example.com,*.research.example.com
```

`GOPRIVATE` chỉ đơn giản hoạt động như giá trị mặc định cho `GONOPROXY` và `GONOSUMDB`, vì vậy không cần thiết lập `GONOPROXY` trừ khi `GONOSUMDB` cần có một giá trị khác. Khi đường dẫn module khớp với `GONOPROXY`, lệnh `go` bỏ qua `GOPROXY` đối với module đó và lấy module trực tiếp từ kho lưu trữ quản lý phiên bản của nó. Điều này hữu ích khi không có proxy nào phục vụ các module riêng tư. Xem [Truy cập trực tiếp vào các module riêng tư](#private-module-proxy-direct).

Nếu có một [proxy đáng tin cậy phục vụ tất cả module](#private-module-proxy-all), thì không nên đặt `GONOPROXY`. Ví dụ, nếu `GOPROXY` được đặt thành một nguồn duy nhất, lệnh `go` sẽ không tải xuống module từ các nguồn khác. `GONOSUMDB` vẫn nên được đặt trong trường hợp này.

```
GOPROXY=https://proxy.corp.example.com
GONOSUMDB=*.corp.example.com,*.research.example.com
```

Nếu có một [proxy đáng tin cậy chỉ phục vụ các module riêng tư](#private-module-proxy-private), thì không nên đặt `GONOPROXY`, nhưng cần đảm bảo proxy phản hồi bằng các mã trạng thái chính xác. Ví dụ, hãy xem xét cấu hình sau:

```
GOPROXY=https://proxy.corp.example.com,https://proxy.golang.org
GONOSUMDB=*.corp.example.com,*.research.example.com
```

Giả sử rằng do một lỗi đánh máy, một nhà phát triển cố tải xuống một module không tồn tại.

```
go mod download corp.example.com/secret-product/typo@latest
```

Lệnh `go` trước tiên yêu cầu module này từ `proxy.corp.example.com`. Nếu proxy đó phản hồi với 404 (Not Found) hoặc 410 (Gone), lệnh `go` sẽ chuyển sang `proxy.golang.org`, truyền đường dẫn `secret-product` trong URL yêu cầu. Nếu proxy riêng tư phản hồi bằng bất kỳ mã lỗi nào khác, lệnh `go` sẽ in lỗi và không chuyển sang các nguồn khác.

Ngoài các proxy, lệnh `go` có thể kết nối đến cơ sở dữ liệu checksum để xác minh các hàm băm mật mã của những module không được liệt kê trong `go.sum`. Biến môi trường `GOSUMDB` thiết lập tên, URL và khóa công khai của cơ sở dữ liệu checksum. Giá trị mặc định của `GOSUMDB` là `sum.golang.org`, cơ sở dữ liệu checksum công khai do Google vận hành ([chính sách quyền riêng tư](https://sum.golang.org/privacy)). Xem [Cơ sở dữ liệu checksum](#checksum-database) để biết chi tiết về những gì được truyền trong mỗi yêu cầu. Tương tự như proxy, lệnh `go` không truyền thông tin nhận dạng cá nhân, nhưng nó có truyền toàn bộ đường dẫn module đang được yêu cầu, và cơ sở dữ liệu checksum không thể tính checksum cho các module không công khai.

Biến môi trường `GONOSUMDB` có thể được đặt thành các mẫu cho biết những module nào là riêng tư và không nên được yêu cầu từ cơ sở dữ liệu checksum. `GOPRIVATE` hoạt động như giá trị mặc định cho `GONOSUMDB` và `GONOPROXY`, vì vậy không cần thiết lập `GONOSUMDB` trừ khi `GONOPROXY` cần có một giá trị khác.

Một proxy có thể [phản chiếu cơ sở dữ liệu checksum](https://go.googlesource.com/proposal/+/master/design/25530-sumdb.md#proxying-a-checksum-database). Nếu một proxy trong `GOPROXY` thực hiện điều này, lệnh `go` sẽ không kết nối trực tiếp đến cơ sở dữ liệu checksum.

`GOSUMDB` có thể được đặt thành `off` để vô hiệu hóa hoàn toàn việc sử dụng cơ sở dữ liệu checksum. Với thiết lập này, lệnh `go` sẽ không xác thực các module đã tải xuống trừ khi chúng đã có trong `go.sum`. Xem [Xác thực module](#authenticating).

## Bộ nhớ đệm module {#module-cache}

The <dfn>bộ nhớ đệm module</dfn> là thư mục nơi lệnh `go` lưu trữ các tệp module đã tải xuống. Bộ nhớ đệm module khác với bộ nhớ đệm build, nơi chứa các gói đã biên dịch và các tạo tác build khác.

Vị trí mặc định của bộ nhớ đệm module là `$GOPATH/pkg/mod`. Để sử dụng vị trí khác, hãy đặt [biến môi trường](#environment-variables) `GOMODCACHE`.

Bộ nhớ đệm module không có kích thước tối đa và lệnh `go` không tự động xóa nội dung của nó.

Bộ nhớ đệm có thể được chia sẻ bởi nhiều dự án Go được phát triển trên cùng một máy. Lệnh `go` sẽ sử dụng cùng một bộ nhớ đệm bất kể vị trí của module chính. Nhiều thực thể của lệnh `go` có thể truy cập an toàn cùng một bộ nhớ đệm module cùng lúc.

Lệnh `go` tạo các tệp nguồn module và thư mục trong bộ nhớ đệm với quyền chỉ đọc để ngăn việc vô tình thay đổi các module sau khi chúng được tải xuống. Điều này có tác dụng phụ không mong muốn là khiến việc xóa bộ nhớ đệm bằng các lệnh như `rm -rf` trở nên khó khăn. Thay vào đó, có thể xóa bộ nhớ đệm bằng [`go clean -modcache`](#go-clean-modcache). Ngoài ra, khi sử dụng cờ `-modcacherw`, lệnh `go` sẽ tạo các thư mục mới với quyền đọc-ghi. Điều này làm tăng rủi ro trình soạn thảo, bài kiểm thử và các chương trình khác sửa đổi tệp trong bộ nhớ đệm module. Có thể sử dụng lệnh [`go mod verify`](#go-mod-verify) để phát hiện các sửa đổi đối với các dependency của module chính. Lệnh này quét nội dung đã giải nén của từng dependency module và xác nhận chúng khớp với hash mong đợi trong `go.sum`.

Bảng bên dưới giải thích mục đích của hầu hết các tệp trong bộ nhớ đệm module. Một số tệp tạm thời (tệp khóa, thư mục tạm thời) được bỏ qua. Với mỗi đường dẫn, `$module` là đường dẫn module và `$version` là một phiên bản. Các đường dẫn kết thúc bằng dấu gạch chéo (`/`) là thư mục. Các chữ cái viết hoa trong đường dẫn module và phiên bản được escape bằng dấu chấm than (`Azure` được escape thành `!azure`) để tránh xung đột trên các hệ thống tệp không phân biệt chữ hoa chữ thường.

<table class="ModTable">
  <thead>
    <tr>
      <th>Đường dẫn</th>
      <th>Mô tả</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>$module@$version/</code></td>
      <td>
        Thư mục chứa nội dung đã giải nén của tệp <code>.zip</code>
        module. Thư mục này đóng vai trò là thư mục gốc module cho một module đã
        tải xuống. Nó sẽ không chứa tệp <code>go.mod</code> nếu module gốc
        không có tệp đó.
      </td>
    </tr>
    <tr>
      <td><code>cache/download/</code></td>
      <td>
        Thư mục chứa các tệp được tải xuống từ proxy module và các tệp
        được tạo ra từ <a href="#vcs">hệ thống quản lý phiên bản</a>. Bố cục của
        thư mục này tuân theo
        <a href="#goproxy-protocol">giao thức <code>GOPROXY</code></a>, vì vậy
        thư mục này có thể được sử dụng làm proxy khi được phục vụ bởi một máy chủ
        tệp HTTP hoặc khi được tham chiếu bằng URL <code>file://</code>.
      </td>
    </tr>
    <tr>
      <td><code>cache/download/$module/@v/list</code></td>
      <td>
        Danh sách các phiên bản đã biết (xem
        <a href="#goproxy-protocol">giao thức <code>GOPROXY</code></a>). Tệp này
        có thể thay đổi theo thời gian, vì vậy lệnh <code>go</code> thường lấy
        một bản sao mới thay vì sử dụng lại tệp này.
      </td>
    </tr>
    <tr>
      <td><code>cache/download/$module/@v/$version.info</code></td>
      <td>
        Metadata JSON về phiên bản. (xem
        <a href="#goproxy-protocol">giao thức <code>GOPROXY</code></a>). Tệp này
        có thể thay đổi theo thời gian, vì vậy lệnh <code>go</code> thường lấy
        một bản sao mới thay vì sử dụng lại tệp này.
      </td>
    </tr>
    <tr>
      <td><code>cache/download/$module/@v/$version.mod</code></td>
      <td>
        Tệp <code>go.mod</code> cho phiên bản này (xem
        <a href="#goproxy-protocol">giao thức <code>GOPROXY</code></a>). Nếu
        module gốc không có tệp <code>go.mod</code>, đây là tệp được tổng hợp
        không có yêu cầu nào.
      </td>
    </tr>
    <tr>
      <td><code>cache/download/$module/@v/$version.zip</code></td>
      <td>
        Nội dung đã nén của module (xem
        <a href="#goproxy-protocol">giao thức <code>GOPROXY</code></a> và
        <a href="#zip-files">tệp zip module</a>).
      </td>
    </tr>
    <tr>
      <td><code>cache/download/$module/@v/$version.ziphash</code></td>
      <td>
        Một hash mật mã của các tệp trong tệp <code>.zip</code>.
        Lưu ý rằng bản thân tệp <code>.zip</code> không được băm, vì vậy thứ
        tự tệp, việc nén, căn chỉnh và metadata không ảnh hưởng đến hash.
        Khi sử dụng một module, lệnh <code>go</code> xác minh hash này
        khớp với dòng tương ứng trong
        <a href="#go-sum-files"><code>go.sum</code></a>. Lệnh
        <a href="#go-mod-verify"><code>go mod verify</code></a> kiểm tra
        rằng các hash của tệp <code>.zip</code> module và các thư mục đã giải
        nén khớp với các tệp này.
      </td>
    </tr>
    <tr>
      <td><code>cache/download/sumdb/</code></td>
      <td>
        Thư mục chứa các tệp được tải xuống từ một
        <a href="#checksum-database">cơ sở dữ liệu checksum</a> (thường là
        <code>sum.golang.org</code>).
      </td>
    </tr>
    <tr>
      <td><code>cache/vcs/</code></td>
      <td>
        Chứa các kho lưu trữ repository được clone của các module được lấy
        trực tiếp từ nguồn của chúng. Tên thư mục là các hash được mã hóa
        hexa, được tạo ra từ loại repository và URL. Các repository được tối ưu
        để giảm kích thước trên đĩa. Ví dụ, các repository Git được clone là
        dạng bare và shallow khi có thể.
      </td>
    </tr>
  </tbody>
</table>

## Xác thực module {#authenticating}

Khi lệnh `go` tải xuống [tệp zip](#zip-files) của module hoặc [tệp `go.mod`](#go-mod-file) vào [bộ nhớ đệm module](#module-cache), nó tính toán một hàm băm mật mã và so sánh với một giá trị đã biết để xác minh rằng tệp chưa bị thay đổi kể từ lần đầu được tải xuống. Lệnh `go` báo lỗi bảo mật nếu tệp đã tải xuống không có hàm băm chính xác.

Đối với các tệp `go.mod`, lệnh `go` tính hàm băm từ nội dung tệp. Đối với các tệp zip của module, lệnh `go` tính hàm băm từ tên và nội dung của các tệp bên trong kho lưu trữ theo một thứ tự xác định. Hàm băm không bị ảnh hưởng bởi thứ tự tệp, việc nén, căn chỉnh và các siêu dữ liệu khác. Xem [`golang.org/x/mod/sumdb/dirhash`](https://pkg.go.dev/golang.org/x/mod/sumdb/dirhash?tab=doc) để biết chi tiết về cách triển khai hàm băm.

Lệnh `go` so sánh từng hàm băm với dòng tương ứng trong [tệp `go.sum`](#go-sum-files) của module chính. Nếu hàm băm khác với hàm băm trong `go.sum`, lệnh `go` báo lỗi bảo mật và xóa tệp đã tải xuống mà không thêm tệp đó vào bộ nhớ đệm module.

Nếu tệp `go.sum` không tồn tại, hoặc không chứa hàm băm cho tệp đã tải xuống, lệnh `go` có thể xác minh hàm băm bằng cách sử dụng [cơ sở dữ liệu checksum](#checksum-database), một nguồn hàm băm toàn cục cho các module công khai. Sau khi hàm băm được xác minh, lệnh `go` thêm nó vào `go.sum` và thêm tệp đã tải xuống vào bộ nhớ đệm module. Nếu một module là riêng tư (được khớp bởi các biến môi trường `GOPRIVATE` hoặc `GONOSUMDB`) hoặc nếu cơ sở dữ liệu checksum bị vô hiệu hóa (bằng cách đặt `GOSUMDB=off`), lệnh `go` chấp nhận hàm băm và thêm tệp vào bộ nhớ đệm module mà không xác minh nó.

Bộ nhớ đệm module thường được chia sẻ bởi tất cả các dự án Go trên một hệ thống, và mỗi module có thể có tệp `go.sum` riêng với các hàm băm có khả năng khác nhau. Để tránh cần phải tin cậy các module khác, lệnh `go` xác minh các hàm băm bằng cách sử dụng `go.sum` của module chính mỗi khi truy cập một tệp trong bộ nhớ đệm module. Việc tính hàm băm cho tệp zip tốn nhiều chi phí, vì vậy lệnh `go` kiểm tra các hàm băm đã được tính trước được lưu bên cạnh các tệp zip thay vì tính lại hàm băm của các tệp. Lệnh [`go mod verify`](#go-mod-verify) có thể được sử dụng để kiểm tra rằng các tệp zip và thư mục đã giải nén chưa bị sửa đổi kể từ khi chúng được thêm vào bộ nhớ đệm module.

### go.sum files {#go-sum-files}

Một module có thể có một tệp văn bản tên là `go.sum` trong thư mục gốc, cùng với tệp `go.mod` của nó. Tệp `go.sum` chứa các hàm băm mật mã của các dependency trực tiếp và gián tiếp của module. Khi lệnh `go` tải xuống tệp `.mod` hoặc `.zip` của một module vào [bộ nhớ đệm module](#module-cache), nó tính toán một hàm băm và kiểm tra rằng hàm băm đó khớp với hàm băm tương ứng trong tệp `go.sum` của module chính. `go.sum` có thể trống hoặc không tồn tại nếu module không có dependency hoặc nếu tất cả dependency được thay thế bằng các thư mục cục bộ bằng cách sử dụng các [chỉ thị `replace`](#go-mod-file-replace).

Mỗi dòng trong `go.sum` có ba trường được phân tách bằng dấu cách: đường dẫn module, một phiên bản (có thể kết thúc bằng `/go.mod`) và một hàm băm.

* Đường dẫn module là tên của module mà hàm băm thuộc về.
* Phiên bản là phiên bản của module mà hàm băm thuộc về. Nếu phiên bản kết thúc bằng `/go.mod`, hàm băm chỉ dành cho tệp `go.mod` của module; nếu không, hàm băm dành cho các tệp bên trong tệp `.zip` của module.
* Cột hàm băm bao gồm tên của một thuật toán (như `h1`) và một hàm băm mật mã được mã hóa bằng base64, được phân tách bằng dấu hai chấm (`:`). Hiện tại, SHA-256 (`h1`) là thuật toán băm duy nhất được hỗ trợ. Nếu phát hiện lỗ hổng bảo mật trong SHA-256 trong tương lai, hỗ trợ sẽ được thêm cho một thuật toán khác (có tên `h2` và tương tự).

Tệp `go.sum` có thể chứa các hàm băm cho nhiều phiên bản của một module. Lệnh `go` có thể cần tải các tệp `go.mod` từ nhiều phiên bản của một dependency để thực hiện [lựa chọn phiên bản tối thiểu](#minimal-version-selection). `go.sum` cũng có thể chứa các hàm băm cho những phiên bản module không còn cần thiết nữa (ví dụ: sau khi nâng cấp). [`go mod tidy`](#go-mod-tidy) sẽ thêm các hàm băm bị thiếu và sẽ xóa các hàm băm không cần thiết khỏi `go.sum`.

### Cơ sở dữ liệu checksum {#checksum-database}

Cơ sở dữ liệu checksum là nguồn toàn cục của các dòng `go.sum`. Lệnh `go` có thể sử dụng nguồn này trong nhiều tình huống để phát hiện hành vi sai của proxy hoặc máy chủ gốc.

Cơ sở dữ liệu checksum cho phép tính nhất quán và độ tin cậy toàn cục cho tất cả các phiên bản module công khai. Nó làm cho các proxy không đáng tin cậy trở nên khả thi vì chúng không thể cung cấp mã sai mà không bị phát hiện. Nó cũng đảm bảo rằng các bit liên quan đến một phiên bản cụ thể không thay đổi từ ngày này sang ngày khác, ngay cả khi tác giả của module sau đó thay đổi các thẻ trong kho lưu trữ của họ.

Cơ sở dữ liệu checksum được cung cấp bởi [sum.golang.org](https://sum.golang.org), do Google vận hành. Đây là một [Nhật ký minh bạch](https://research.swtch.com/tlog) (hay “Cây Merkle”) của các hàm băm dòng `go.sum`, được hỗ trợ bởi [Trillian](https://github.com/google/trillian). Ưu điểm chính của cây Merkle là các kiểm toán viên độc lập có thể xác minh rằng nó chưa bị can thiệp, vì vậy nó đáng tin cậy hơn một cơ sở dữ liệu đơn giản.

Lệnh `go` tương tác với cơ sở dữ liệu checksum bằng giao thức được mô tả ban đầu trong [Đề xuất: Bảo mật Hệ sinh thái Module Go công khai](https://go.googlesource.com/proposal/+/master/design/25530-sumdb.md#checksum-database).

Bảng bên dưới chỉ rõ các truy vấn mà cơ sở dữ liệu checksum phải phản hồi. Với mỗi đường dẫn, `$base` là phần đường dẫn của URL cơ sở dữ liệu checksum, `$module` là một đường dẫn module và `$version` là một phiên bản. Ví dụ: nếu URL cơ sở dữ liệu checksum là `https://sum.golang.org`, và máy khách đang yêu cầu bản ghi cho module `golang.org/x/text` ở phiên bản `v0.3.2`, máy khách sẽ gửi một yêu cầu `GET` tới
`https://sum.golang.org/lookup/golang.org/x/text@v0.3.2`.

Để tránh sự không rõ ràng khi phục vụ từ các hệ thống tệp không phân biệt chữ hoa chữ thường, các phần tử `$module` và `$version` được [mã hóa phân biệt chữ hoa chữ thường](https://pkg.go.dev/golang.org/x/mod/module#EscapePath) bằng cách thay thế mọi chữ cái viết hoa bằng một dấu chấm than theo sau bởi chữ cái viết thường tương ứng. Điều này cho phép các module `example.com/M` và `example.com/m` đều được lưu trữ trên đĩa, vì module trước được mã hóa thành `example.com/!m`.

Các phần của đường dẫn được bao quanh bởi dấu ngoặc vuông, như `[.p/$W]`, biểu thị các giá trị tùy chọn.

<table class="ModTable">
  <thead>
    <tr>
      <th>Đường dẫn</th>
      <th>Mô tả</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>$base/latest</code></td>
      <td>
        Trả về mô tả cây đã mã hóa, được ký cho nhật ký mới nhất. Mô tả đã ký này có dạng
        <a href="https://pkg.go.dev/golang.org/x/mod/sumdb/note">note</a>,
        là văn bản đã được ký bởi một hoặc nhiều khóa máy chủ và có thể được xác minh bằng khóa công khai của máy chủ. Mô tả cây cung cấp kích thước của cây và hàm băm của đầu cây tại kích thước đó. Cách mã hóa này được mô tả trong
        <code><a href="https://pkg.go.dev/golang.org/x/mod/sumdb/tlog#FormatTree">golang.org/x/mod/sumdb/tlog#FormatTree</a></code>.
      </td>
    </tr>
    <tr>
      <td><code>$base/lookup/$module@$version</code></td>
      <td>
        Trả về số bản ghi nhật ký cho mục nhập về <code>$module</code>
        tại <code>$version</code>, tiếp theo là dữ liệu của bản ghi (nghĩa là
        các dòng <code>go.sum</code> cho <code>$module</code> tại
        <code>$version</code>) và mô tả cây đã mã hóa, được ký, chứa bản ghi đó.
      </td>
    </tr>
    <tr>
      <td><code>$base/tile/$H/$L/$K[.p/$W]</code></td>
      <td>
        Trả về một <a href="https://research.swtch.com/tlog#serving_tiles">mảnh nhật ký</a>,
        là một tập hợp các hàm băm tạo thành một phần của nhật ký. Mỗi mảnh
        được xác định trong tọa độ hai chiều tại mức mảnh
        <code>$L</code>, thứ <code>$K</code> tính từ bên trái, với chiều cao mảnh
        là <code>$H</code>. Hậu tố tùy chọn <code>.p/$W</code> chỉ ra một
        mảnh nhật ký từng phần chỉ có <code>$W</code> hàm băm. Máy khách phải
        quay lại lấy mảnh đầy đủ nếu không tìm thấy mảnh từng phần.
      </td>
    </tr>
    <tr>
      <td><code>$base/tile/$H/data/$K[.p/$W]</code></td>
      <td>
        Trả về dữ liệu bản ghi cho các hàm băm lá trong
        <code>/tile/$H/0/$K[.p/$W]</code> (với phần tử đường dẫn
        <code>data</code> theo nghĩa đen).
      </td>
    </tr>
  </tbody>
</table>

Nếu lệnh <code>go</code> tham vấn cơ sở dữ liệu tổng kiểm tra, thì bước đầu tiên là truy xuất dữ liệu bản ghi thông qua điểm cuối <code>/lookup</code>. Nếu phiên bản mô-đun chưa được ghi vào nhật ký, cơ sở dữ liệu tổng kiểm tra sẽ thử lấy nó từ máy chủ gốc trước khi trả lời. Dữ liệu <code>/lookup</code> này cung cấp tổng cho phiên bản mô-đun này cũng như vị trí của nó trong nhật ký, cho máy khách biết những mảnh nào cần được lấy để thực hiện các bằng chứng. Lệnh <code>go</code> thực hiện các bằng chứng “bao gồm” (rằng một bản ghi cụ thể tồn tại trong nhật ký) và các bằng chứng “nhất quán” (rằng cây chưa bị can thiệp) trước khi thêm các dòng <code>go.sum</code> mới vào tệp <code>go.sum</code> của mô-đun chính. Điều quan trọng là dữ liệu từ <code>/lookup</code> không bao giờ được sử dụng nếu chưa xác thực nó với hàm băm cây đã ký và xác thực hàm băm cây đã ký với chuỗi thời gian của các hàm băm cây đã ký của máy khách.

Các hàm băm cây đã ký và các mảnh mới do cơ sở dữ liệu tổng kiểm tra cung cấp được lưu trong bộ đệm mô-đun, vì vậy lệnh <code>go</code> chỉ cần lấy các mảnh còn thiếu.

Lệnh <code>go</code> không cần kết nối trực tiếp với cơ sở dữ liệu tổng kiểm tra. Nó có thể yêu cầu tổng mô-đun thông qua một proxy mô-đun
[phản chiếu cơ sở dữ liệu tổng kiểm tra](https://go.googlesource.com/proposal/+/master/design/25530-sumdb.md#proxying-a-checksum-database)
và hỗ trợ giao thức ở trên. Điều này có thể đặc biệt hữu ích cho các proxy riêng tư, proxy của doanh nghiệp chặn các yêu cầu bên ngoài tổ chức.

Biến môi trường <code>GOSUMDB</code> xác định tên của cơ sở dữ liệu tổng kiểm tra cần sử dụng và tùy chọn khóa công khai cùng URL của nó, như sau:

```
GOSUMDB="sum.golang.org"
GOSUMDB="sum.golang.org+<publickey>"
GOSUMDB="sum.golang.org+<publickey> https://sum.golang.org"
```

Lệnh <code>go</code> biết khóa công khai của <code>sum.golang.org</code>, đồng thời biết rằng tên <code>sum.golang.google.cn</code> (có sẵn bên trong Trung Quốc đại lục) kết nối với cơ sở dữ liệu tổng kiểm tra <code>sum.golang.org</code>; việc sử dụng bất kỳ cơ sở dữ liệu nào khác yêu cầu cung cấp rõ ràng khóa công khai. URL mặc định là <code>https://</code> theo sau bởi tên cơ sở dữ liệu.

<code>GOSUMDB</code> mặc định là <code>sum.golang.org</code>, cơ sở dữ liệu tổng kiểm tra Go do Google vận hành. Xem https://sum.golang.org/privacy để biết chính sách quyền riêng tư của dịch vụ.

Nếu <code>GOSUMDB</code> được đặt thành <code>off</code>, hoặc nếu <code>go get</code> được gọi với cờ <code>-insecure</code>, cơ sở dữ liệu tổng kiểm tra sẽ không được tham vấn và tất cả mô-đun chưa được nhận dạng sẽ được chấp nhận, phải đánh đổi bằng việc từ bỏ đảm bảo bảo mật về các lượt tải xuống có thể lặp lại đã được xác minh cho tất cả mô-đun. Cách tốt hơn để bỏ qua cơ sở dữ liệu tổng kiểm tra cho các mô-đun cụ thể là sử dụng các biến môi trường <code>GOPRIVATE</code> hoặc <code>GONOSUMDB</code>. Xem [Mô-đun riêng tư](#private-modules) để biết chi tiết.

Có thể sử dụng lệnh <code>go env -w</code> để
[đặt các biến này](/pkg/cmd/go/#hdr-Print_Go_environment_information)
cho các lần gọi lệnh <code>go</code> trong tương lai.

## Biến môi trường {#environment-variables}

Hành vi của module trong lệnh `go` có thể được cấu hình bằng các biến môi trường được liệt kê bên dưới. Danh sách này chỉ bao gồm các biến môi trường liên quan đến module. Xem [`go help
environment`](/cmd/go/#hdr-Environment_variables) để biết danh sách
tất cả các biến môi trường được lệnh `go` nhận dạng.

<table class="ModTable">
  <thead>
    <tr>
      <th>Biến</th>
      <th>Mô tả</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>GO111MODULE</code></td>
      <td>
        <p>
          Kiểm soát việc lệnh <code>go</code> chạy ở chế độ nhận biết module
          hay chế độ <code>GOPATH</code>. Có ba giá trị được nhận dạng:
        </p>
        <ul>
          <li>
            <code>off</code>: lệnh <code>go</code> bỏ qua các tệp
            <code>go.mod</code> và chạy ở chế độ <code>GOPATH</code>.
          </li>
          <li>
            <code>on</code> (hoặc không được đặt): lệnh <code>go</code> chạy ở
            chế độ nhận biết module, ngay cả khi không có tệp <code>go.mod</code>.
          </li>
          <li>
            <code>auto</code>: lệnh <code>go</code> chạy ở chế độ nhận biết module
            nếu có tệp <code>go.mod</code> trong thư mục hiện tại hoặc bất kỳ
            thư mục cha nào. Trong Go 1.15 trở xuống, đây là giá trị mặc định.
          </li>
        </ul>
        <p>
          Xem <a href="#mod-commands">Các lệnh nhận biết module</a> để biết thêm
          thông tin.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>GOMODCACHE</code></td>
      <td>
        <p>
          Thư mục nơi lệnh <code>go</code> sẽ lưu các module đã tải xuống
          và các tệp liên quan. Xem <a href="#module-cache">Bộ nhớ đệm
          module</a> để biết chi tiết về cấu trúc của thư mục này.
        </p>
        <p>
          Nếu <code>GOMODCACHE</code> không được đặt, giá trị mặc định là
          <code>$GOPATH/pkg/mod</code>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>GOINSECURE</code></td>
      <td>
        <p>
          Danh sách các mẫu glob được phân tách bằng dấu phẩy (theo cú pháp
          của <a href="/pkg/path/#Match"><code>path.Match</code></a> của Go)
          gồm các tiền tố đường dẫn module luôn có thể được tải xuống theo
          cách không an toàn. Chỉ áp dụng cho các dependency đang được tải
          xuống trực tiếp.
        </p>
        <p>
          Không giống cờ <code>-insecure</code> của <code>go get</code>,
          <code>GOINSECURE</code> không vô hiệu hóa việc xác thực cơ sở dữ liệu
          checksum của module. Có thể dùng <code>GOPRIVATE</code> hoặc
          <code>GONOSUMDB</code> để đạt được điều đó.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>GONOPROXY</code></td>
      <td>
        <p>
          Danh sách các mẫu glob được phân tách bằng dấu phẩy (theo cú pháp
          của <a href="/pkg/path/#Match"><code>path.Match</code></a> của Go)
          gồm các tiền tố đường dẫn module luôn phải được tải trực tiếp từ
          các repository kiểm soát phiên bản, không phải từ các proxy module.
        </p>
        <p>
          Nếu <code>GONOPROXY</code> không được đặt, giá trị mặc định là
          <code>GOPRIVATE</code>. Xem
          <a href="#private-module-privacy">Quyền riêng tư</a>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>GONOSUMDB</code></td>
      <td>
        <p>
          Danh sách các mẫu glob được phân tách bằng dấu phẩy (theo cú pháp
          của <a href="/pkg/path/#Match"><code>path.Match</code></a> của Go)
          gồm các tiền tố đường dẫn module mà tại đó <code>go</code> không
          nên xác minh checksum bằng cơ sở dữ liệu checksum.
        </p>
        <p>
          Nếu <code>GONOSUMDB</code> không được đặt, giá trị mặc định là
          <code>GOPRIVATE</code>. Xem
          <a href="#private-module-privacy">Quyền riêng tư</a>.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>GOPATH</code></td>
      <td>
        <p>
          Ở chế độ <code>GOPATH</code>, biến <code>GOPATH</code> là danh sách
          các thư mục có thể chứa mã Go.
        </p>
        <p>
          Ở chế độ nhận biết module, <a href="#glos-module-cache">bộ nhớ đệm
          module</a> được lưu trong thư mục con <code>pkg/mod</code> của thư mục
          <code>GOPATH</code> đầu tiên. Mã nguồn module bên ngoài bộ nhớ đệm có
          thể được lưu trong bất kỳ thư mục nào.
        </p>
        <p>
          Nếu <code>GOPATH</code> không được đặt, giá trị mặc định là thư mục
          con <code>go</code> trong thư mục nhà của người dùng.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>GOPRIVATE</code></td>
      <td>
        Danh sách các mẫu glob được phân tách bằng dấu phẩy (theo cú pháp của Go's
        <a href="/pkg/path/#Match"><code>path.Match</code></a>) gồm các tiền tố
        đường dẫn module nên được xem là riêng tư. <code>GOPRIVATE</code>
        là giá trị mặc định cho <code>GONOPROXY</code> và
        <code>GONOSUMDB</code>. Xem
        <a href="#private-module-privacy">Quyền riêng tư</a>. <code>GOPRIVATE</code>
        cũng xác định liệu một module có được xem là riêng tư đối với
        <code>GOVCS</code> hay không.
      </td>
    </tr>
    <tr>
      <td><code>GOPROXY</code></td>
      <td>
        <p>
          Danh sách các URL proxy module, được phân tách bằng dấu phẩy
          (<code>,</code>) hoặc dấu gạch đứng (<code>|</code>). Khi lệnh
          <code>go</code> tra cứu thông tin về một module, nó liên hệ với từng
          proxy trong danh sách theo thứ tự cho đến khi nhận được phản hồi thành
          công hoặc một lỗi kết thúc. Một proxy có thể trả về trạng thái 404
          (Not Found) hoặc 410 (Gone) để cho biết module không có trên máy chủ đó.
        </p>
        <p>
          Cách xử lý dự phòng khi gặp lỗi của lệnh <code>go</code> được xác định
          bởi ký tự phân tách giữa các URL. Nếu URL proxy được theo sau bởi dấu
          phẩy, lệnh <code>go</code> chuyển sang URL tiếp theo sau lỗi 404 hoặc
          410; mọi lỗi khác được xem là lỗi kết thúc. Nếu URL proxy được theo sau
          bởi dấu gạch đứng, lệnh <code>go</code> chuyển sang nguồn tiếp theo sau
          bất kỳ lỗi nào, bao gồm cả các lỗi không phải HTTP như hết thời gian chờ.
        </p>
        <p>
          URL <code>GOPROXY</code> có thể sử dụng các scheme <code>https</code>,
          <code>http</code> hoặc <code>file</code>. Nếu URL không có scheme,
          mặc định sử dụng <code>https</code>. Có thể dùng trực tiếp bộ nhớ đệm
          module làm proxy tệp:
        </p>
        <pre>GOPROXY=file://$(go env GOMODCACHE)/cache/download</pre>
        <p>Có thể dùng hai từ khóa thay cho URL proxy:</p>
        <ul>
          <li>
            <code>off</code>: không cho phép tải module từ bất kỳ nguồn nào.
          </li>
          <li>
            <code>direct</code>: tải trực tiếp từ các repository kiểm soát phiên
            bản thay vì sử dụng proxy module.
          </li>
        </ul>
        <p>
          <code>GOPROXY</code> mặc định là
          <code>https://proxy.golang.org,direct</code>. Với
          cấu hình đó, lệnh <code>go</code> trước tiên liên hệ với máy nhân bản
          module Go do Google vận hành, sau đó chuyển sang kết nối trực tiếp nếu
          máy nhân bản không có module. Xem
          <a href="https://proxy.golang.org/privacy">https://proxy.golang.org/privacy</a>
          để biết chính sách quyền riêng tư của máy nhân bản. Các biến môi trường
          <code>GOPRIVATE</code> và <code>GONOPROXY</code> có thể được đặt để
          ngăn các module cụ thể được tải xuống bằng proxy. Xem
          <a href="#private-module-privacy">Quyền riêng tư</a> để biết thông tin
          về cấu hình proxy riêng tư.
        </p>
        <p>
          Xem <a href="#module-proxy">Proxy module</a> và
          <a href="#resolve-pkg-mod">Giải quyết một gói thành một module</a> để
          biết thêm thông tin về cách proxy được sử dụng.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>GOSUMDB</code></td>
      <td>
        <p>
          Xác định tên của cơ sở dữ liệu checksum được sử dụng và tùy chọn cả
          khóa công khai cùng URL của nó. Ví dụ:
        </p>
        <pre>
GOSUMDB="sum.golang.org"
GOSUMDB="sum.golang.org+&lt;publickey&gt;"
GOSUMDB="sum.golang.org+&lt;publickey&gt; https://sum.golang.org"
</pre>
        <p>
          Lệnh <code>go</code> biết khóa công khai của <code>sum.golang.org</code>
          và cũng biết rằng tên <code>sum.golang.google.cn</code> (có sẵn bên
          trong Trung Quốc đại lục) kết nối với cơ sở dữ liệu
          <code>sum.golang.org</code>; việc sử dụng bất kỳ cơ sở dữ liệu nào khác
          yêu cầu cung cấp khóa công khai một cách rõ ràng. URL mặc định là
          <code>https://</code> theo sau bởi tên cơ sở dữ liệu.
        </p>
        <p>
          <code>GOSUMDB</code> mặc định là <code>sum.golang.org</code>, cơ sở
          dữ liệu checksum Go do Google vận hành. Xem
          <a href="https://sum.golang.org/privacy">https://sum.golang.org/privacy</a>
          để biết chính sách quyền riêng tư của dịch vụ.
        </p>
        <p>
          Nếu <code>GOSUMDB</code> được đặt thành <code>off</code> hoặc nếu
          <code>go get</code> được gọi với cờ <code>-insecure</code>, cơ sở dữ
          liệu checksum sẽ không được tham vấn, và mọi module chưa được nhận dạng
          sẽ được chấp nhận, đổi lại việc từ bỏ đảm bảo bảo mật về các lượt tải
          xuống có thể xác minh và lặp lại cho mọi module. Cách tốt hơn để bỏ qua
          cơ sở dữ liệu checksum cho các module cụ thể là sử dụng các biến môi
          trường <code>GOPRIVATE</code> hoặc <code>GONOSUMDB</code>.
        </p>
        <p>
          Xem <a href="#authenticating">Xác thực module</a> và
          <a href="#private-module-privacy">Quyền riêng tư</a> để biết thêm
          thông tin.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>GOVCS</code></td>
      <td>
        <p>
          Kiểm soát tập hợp các công cụ kiểm soát phiên bản mà lệnh
          <code>go</code> có thể sử dụng để tải xuống các module công khai và
          riêng tư (được xác định dựa trên việc đường dẫn của chúng có khớp với
          mẫu trong <code>GOPRIVATE</code> hay không) hoặc các module khác khớp
          với một mẫu glob.
        </p>
        <p>
          Nếu <code>GOVCS</code> không được đặt, hoặc nếu một module không khớp
          với bất kỳ mẫu nào trong <code>GOVCS</code>, lệnh <code>go</code> có
          thể sử dụng <code>git</code> và <code>hg</code> cho module công khai,
          hoặc bất kỳ công cụ kiểm soát phiên bản đã biết nào cho module riêng
          tư. Cụ thể, lệnh <code>go</code> hoạt động như thể <code>GOVCS</code>
          được đặt thành:
        </p>
        <pre>public:git|hg,private:all</pre>
        <p>
          Xem <a href="#vcs-govcs">Kiểm soát các công cụ kiểm soát phiên bản với
          <code>GOVCS</code></a> để biết giải thích đầy đủ.
        </p>
      </td>
    </tr>
    <tr>
      <td><code>GOWORK</code></td>
      <td>
       <p>
        Biến môi trường <code>GOWORK</code> hướng dẫn lệnh <code>go</code> vào
        chế độ workspace bằng cách sử dụng <a href="#go-work-file">tệp
        <code>go.work</code></a> được cung cấp để định nghĩa workspace.
        Nếu <code>GOWORK</code> được đặt thành <code>off</code>, chế độ
        workspace bị vô hiệu hóa. Có thể dùng cách này để chạy lệnh
        <code>go</code> ở chế độ một module: ví dụ,
        <code>GOWORK=off go build .</code> xây dựng gói <code>.</code> ở chế độ
        một module. Nếu <code>GOWORK</code> rỗng, lệnh <code>go</code> sẽ tìm
        kiếm tệp <code>go.work</code> như được mô tả trong phần
        <a href="#workspaces">Workspace</a>.
       </p>
      </td>
    </tr>
  </tbody>
</table>

## Bảng thuật ngữ {#glossary}

<a id="glos-build-constraint"></a>
**build constraint:** Một điều kiện xác định liệu tệp nguồn Go có được sử dụng khi biên dịch một package hay không. Các build constraint có thể được biểu diễn bằng hậu tố tên tệp (ví dụ: `foo_linux_amd64.go`) hoặc bằng các chú thích build constraint (ví dụ: `// +build linux,amd64`). Xem [Build
Constraints](/pkg/go/build/#hdr-Build_Constraints).

<a id="glos-build-list"></a>
**build list:** Danh sách các phiên bản module sẽ được sử dụng cho một lệnh build như `go build`, `go list` hoặc `go test`. Build list được xác định từ [module chính](#glos-main-module) [tệp `go.mod`](#glos-go-mod-file) và các tệp `go.mod` trong các module được yêu cầu bắc cầu bằng cách sử dụng [minimal version selection](#glos-minimal-version-selection). Build list chứa phiên bản của tất cả module trong [đồ thị module](#glos-module-graph), không chỉ những module liên quan đến một lệnh cụ thể.

<a id="glos-canonical-version"></a>
**canonical version:** Một [version](#glos-version) có định dạng đúng, không có hậu tố metadata build nào khác ngoài `+incompatible`. Ví dụ, `v1.2.3` là một canonical version, nhưng `v1.2.3+meta` thì không.

<a id="glos-current-module"></a>
**current module:** Từ đồng nghĩa với [module chính](#glos-main-module).

<a id="glos-deprecated-module"></a>
**deprecated module:** Một module không còn được các tác giả của nó hỗ trợ nữa (mặc dù các phiên bản major được xem là các module riêng biệt cho mục đích này). Một deprecated module được đánh dấu bằng [chú thích deprecated](#go-mod-file-module-deprecation) trong phiên bản mới nhất của [tệp `go.mod`](#glos-go-mod-file) của nó.

<a id="glos-direct-dependency"></a>
**direct dependency:** Một package có đường dẫn xuất hiện trong [khai báo `import`](/ref/spec#import_declarations) trong tệp nguồn `.go` của một package hoặc test trong [module chính](#glos-main-module), hoặc module chứa package đó. (So sánh với [indirect dependency](#glos-indirect-dependency).)

<a id="glos-direct-mode"></a>
**direct mode:** Một thiết lập của [biến môi trường](#environment-variables) khiến lệnh `go` tải xuống một module trực tiếp từ [hệ thống quản lý phiên bản](#vcs), thay vì từ [module proxy](#glos-module-proxy). `GOPROXY=direct` thực hiện điều này cho tất cả module. `GOPRIVATE` và `GONOPROXY` thực hiện điều này cho các module khớp với một danh sách mẫu.

<a id="glos-go-mod-file"></a>
**`go.mod` file:** Tệp định nghĩa đường dẫn, các yêu cầu và metadata khác của một module. Xuất hiện trong [thư mục gốc của module](#glos-module-root-directory). Xem phần về [các tệp `go.mod`](#go-mod-file).

<a id="glos-go-work-file"></a>
**`go.work` file** Tệp định nghĩa tập hợp các module được sử dụng trong một [workspace](#workspaces). Xem phần về [các tệp `go.work`](#go-work-file)

<a id="glos-import-path"></a>
**import path:** Một chuỗi được dùng để import một package trong tệp nguồn Go. Đồng nghĩa với [package path](#glos-package-path).

<a id="glos-indirect-dependency"></a>
**indirect dependency:** Một package được import bắc cầu bởi một package hoặc test trong [module chính](#glos-main-module), nhưng đường dẫn của nó không xuất hiện trong bất kỳ [khai báo `import`](/ref/spec#import_declarations) nào trong module chính; hoặc một module xuất hiện trong [đồ thị module](#glos-module-graph) nhưng không cung cấp package nào được module chính import trực tiếp.
(So sánh với [direct dependency](#glos-direct-dependency).)

<a id="glos-lazy-module-loading"></a>
**lazy module loading:** Một thay đổi trong Go 1.17 giúp tránh tải [đồ thị module](#glos-module-graph) cho các lệnh không cần đến nó trong những module chỉ định `go 1.17` hoặc cao hơn. Xem [Lazy module loading](#lazy-loading).

<a id="glos-main-module"></a>
**main module:** Module mà trong đó lệnh `go` được gọi. Module chính được định nghĩa bởi một [tệp `go.mod`](#glos-go-mod-file) trong thư mục hiện tại hoặc thư mục cha. Xem [Modules, packages, and
versions](#modules-overview).

<a id="glos-major-version"></a>
**major version:** Số đầu tiên trong một semantic version (`1` trong `v1.2.3`). Trong một bản phát hành có các thay đổi không tương thích, major version phải được tăng lên, còn minor version và patch version phải được đặt thành 0. Các semantic version có major version bằng 0 được xem là không ổn định.

<a id="glos-major-version-subdirectory"></a>
**major version subdirectory:** Một thư mục con trong một repository quản lý phiên bản khớp với [hậu tố major version](#glos-major-version-suffix) của một module, nơi module có thể được định nghĩa. Ví dụ, module `example.com/mod/v2` trong repository có [đường dẫn gốc](#glos-repository-root-path) `example.com/mod` có thể được định nghĩa trong thư mục gốc của repository hoặc thư mục con major version `v2`. Xem [Module directories within a repository](#vcs-dir).

<a id="glos-major-version-suffix"></a>
**major version suffix:** Một hậu tố đường dẫn module khớp với số major version. Ví dụ, `/v2` trong `example.com/mod/v2`. Các major version suffix được yêu cầu ở `v2.0.0` và các phiên bản sau đó, đồng thời không được phép ở các phiên bản trước đó. Xem phần về [Major version suffixes](#major-version-suffixes).

<a id="glos-minimal-version-selection"></a>
**minimal version selection (MVS):** Thuật toán được sử dụng để xác định phiên bản của tất cả module sẽ được dùng trong một bản build. Xem phần [Minimal version selection](#minimal-version-selection) để biết chi tiết.

<a id="glos-minor-version"></a>
**minor version:** Số thứ hai trong một semantic version (`2` trong `v1.2.3`). Trong một bản phát hành có chức năng mới tương thích ngược, minor version phải được tăng lên và patch version phải được đặt thành 0.

<a id="glos-module"></a>
**module:** Một tập hợp các package được phát hành, đánh version và phân phối cùng nhau.

<a id="glos-module-cache"></a>
**module cache:** Một thư mục cục bộ lưu trữ các module đã tải xuống, nằm tại `GOPATH/pkg/mod`. Xem [Module cache](#module-cache).

<a id="glos-module-graph"></a>
**đồ thị module:** Đồ thị có hướng của các yêu cầu module, bắt nguồn từ [module chính](#glos-main-module). Mỗi đỉnh trong đồ thị là một module; mỗi cạnh là một phiên bản từ câu lệnh `require` trong tệp `go.mod` (chịu sự chi phối của các câu lệnh `replace` và `exclude` trong tệp `go.mod` của module chính).

<a id="glos-module-graph-pruning"></a>
**cắt tỉa đồ thị module:** Một thay đổi trong Go 1.17 giúp giảm kích thước của đồ thị module bằng cách loại bỏ các dependency bắc cầu của những module chỉ định `go 1.17` hoặc cao hơn. Xem [Cắt tỉa đồ thị module](#graph-pruning).

<a id="glos-module-path"></a>
**đường dẫn module:** Một đường dẫn xác định một module và đóng vai trò tiền tố cho các đường dẫn import package bên trong module. Ví dụ: `"golang.org/x/net"`.

<a id="glos-module-proxy"></a>
**proxy module:** Một máy chủ web triển khai [giao thức `GOPROXY`](#goproxy-protocol). Lệnh `go` tải thông tin phiên bản, các tệp `go.mod` và các tệp zip module từ các proxy module.

<a id="glos-module-root-directory"></a>
**thư mục gốc module:** Thư mục chứa tệp `go.mod` định nghĩa một module.

<a id="glos-module-subdirectory"></a>
**thư mục con module:** Phần của [đường dẫn module](#glos-module-path) nằm sau [đường dẫn gốc repository](#glos-repository-root-path), cho biết thư mục con nơi module được định nghĩa. Khi không rỗng, thư mục con module cũng là tiền tố cho [thẻ phiên bản semantic](#glos-semantic-version-tag). Thư mục con module không bao gồm [hậu tố phiên bản chính](#glos-major-version-suffix), nếu có, ngay cả khi module nằm trong [thư mục con phiên bản chính](#glos-major-version-subdirectory). Xem [Đường dẫn module](#module-path).

<a id="glos-package"></a>
**package:** Một tập hợp các tệp mã nguồn trong cùng một thư mục được biên dịch cùng nhau. Xem [phần Packages](/ref/spec#Packages) trong Đặc tả ngôn ngữ Go.

<a id="glos-package-path"></a>
**đường dẫn package:** Đường dẫn xác định duy nhất một package. Đường dẫn package là một [đường dẫn module](#glos-module-path) được nối với một thư mục con bên trong module. Ví dụ, `"golang.org/x/net/html"` là đường dẫn package cho package trong module `"golang.org/x/net"` ở thư mục con `"html"`. Đồng nghĩa với [đường dẫn import](#glos-import-path).

<a id="glos-patch-version"></a>
**phiên bản patch:** Số thứ ba trong một phiên bản semantic (`3` trong `v1.2.3`). Trong một bản phát hành không có thay đổi đối với interface công khai của module, phiên bản patch phải được tăng lên.

<a id="glos-pre-release-version"></a>
**phiên bản pre-release:** Một phiên bản có dấu gạch ngang theo sau là một chuỗi định danh được phân tách bằng dấu chấm ngay sau phiên bản patch, ví dụ `v1.2.3-beta4`. Các phiên bản pre-release được xem là không ổn định và không được giả định là tương thích với các phiên bản khác. Một phiên bản pre-release được sắp xếp trước phiên bản phát hành tương ứng: `v1.2.3-pre` đứng trước `v1.2.3`. Xem thêm [phiên bản bản phát hành](#glos-release-version).

<a id="glos-pseudo-version"></a>
**phiên bản giả:** Một phiên bản mã hóa định danh revision (chẳng hạn như hash commit Git) và dấu thời gian từ một hệ thống quản lý phiên bản. Ví dụ, `v0.0.0-20191109021931-daa7c04131f5`. Được dùng để [tương thích với các repository không dùng module](#non-module-compat) và trong các tình huống khác khi không có phiên bản được gắn thẻ.

<a id="glos-release-version"></a>
**phiên bản bản phát hành:** Một phiên bản không có hậu tố pre-release. Ví dụ, `v1.2.3`, không phải `v1.2.3-pre`. Xem thêm [phiên bản pre-release](#glos-pre-release-version).

<a id="glos-repository-root-path"></a>
**đường dẫn gốc repository:** Phần của [đường dẫn module](#glos-module-path) tương ứng với thư mục gốc của một repository quản lý phiên bản. Xem [Đường dẫn module](#module-path).

<a id="glos-retracted-version"></a>
**phiên bản bị thu hồi:** Một phiên bản không nên được phụ thuộc vào, vì nó được phát hành quá sớm hoặc vì một vấn đề nghiêm trọng được phát hiện sau khi nó được phát hành. Xem [chỉ thị `retract`](#go-mod-file-retract).

<a id="glos-semantic-version-tag"></a>
**thẻ phiên bản semantic:** Một thẻ trong repository quản lý phiên bản ánh xạ một [phiên bản](#glos-version) tới một revision cụ thể. Xem [Ánh xạ phiên bản tới commit](#vcs-version).

<a id="glos-selected-version"></a>
**phiên bản được chọn:** Phiên bản của một module cụ thể được chọn bởi [lựa chọn phiên bản tối thiểu](#minimal-version-selection). Phiên bản được chọn là phiên bản cao nhất cho đường dẫn của module được tìm thấy trong [đồ thị module](#glos-module-graph).

<a id="glos-vendor-directory"></a>
**thư mục vendor:** Một thư mục có tên `vendor` chứa các package từ các module khác cần thiết để xây dựng các package trong module chính. Được duy trì bằng [`go mod vendor`](#go-mod-vendor). Xem [Vendoring](#vendoring).

<a id="glos-version"></a>
**phiên bản:** Một định danh cho ảnh chụp bất biến của một module, được viết dưới dạng chữ `v` theo sau bởi một phiên bản semantic. Xem phần [Các phiên bản](#versions).

<a id="glos-workspace"></a>
**workspace:** Một tập hợp các module trên đĩa được sử dụng làm các module chính khi chạy [lựa chọn phiên bản tối thiểu (MVS)](#minimal-version-selection). Xem phần [Workspace](#workspaces)
