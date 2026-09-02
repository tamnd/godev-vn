---
title: API tệp kháng traversal
date: 2025-03-12
by:
- Damien Neil
tags:
- file
- os
summary: API truy cập tệp mới trong Go 1.24.
---

Một *lỗ hổng path traversal* phát sinh khi kẻ tấn công có thể lừa chương trình
mở một tệp khác với tệp mà nó định mở.
Bài đăng này giải thích loại lỗ hổng này,
một số biện pháp phòng thủ hiện có chống lại nó, và mô tả cách API mới
[`os.Root`](/pkg/os#Root) được thêm vào trong Go 1.24 cung cấp
một biện pháp phòng thủ đơn giản và mạnh mẽ chống lại path traversal không chủ ý.

## Tấn công path traversal

"Path traversal" bao gồm một số tấn công liên quan tuân theo một mẫu chung:
Chương trình cố gắng mở một tệp ở một vị trí đã biết, nhưng kẻ tấn công khiến
nó mở một tệp ở vị trí khác.

Nếu kẻ tấn công kiểm soát một phần của tên tệp, họ có thể sử dụng các
thành phần thư mục tương đối ("..") để thoát khỏi vị trí dự định:

    f, err := os.Open(filepath.Join(trustedLocation, "../../../../etc/passwd"))

Trên các hệ thống Windows, một số tên có ý nghĩa đặc biệt:

    // f sẽ in ra console.
    f, err := os.Create(filepath.Join(trustedLocation, "CONOUT$"))

Nếu kẻ tấn công kiểm soát một phần của hệ thống tệp cục bộ, họ có thể sử dụng
liên kết tượng trưng để khiến chương trình truy cập sai tệp:

    // Kẻ tấn công liên kết /home/user/.config đến /home/otheruser/.config:
    err := os.WriteFile("/home/user/.config/foo", config, 0o666)

Nếu chương trình bảo vệ chống lại traversal symlink bằng cách xác minh trước rằng tệp dự định
không chứa bất kỳ symlink nào, nó vẫn có thể dễ bị tổn thương bởi
[các cuộc đua time-of-check/time-of-use (TOCTOU)](https://en.wikipedia.org/wiki/Time-of-check_to_time-of-use),
nơi kẻ tấn công tạo một symlink sau khi chương trình kiểm tra:

    // Xác thực đường dẫn trước khi sử dụng.
    cleaned, err := filepath.EvalSymlinks(unsafePath)
    if err != nil {
      return err
    }
    if !filepath.IsLocal(cleaned) {
      return errors.New("unsafe path")
    }

    // Kẻ tấn công thay thế một phần của đường dẫn bằng symlink.
    // Cuộc gọi Open theo dõi symlink:
    f, err := os.Open(cleaned)

Một loại cuộc đua TOCTOU khác liên quan đến việc di chuyển một thư mục tạo thành một phần của đường dẫn
trong quá trình traversal. Ví dụ, kẻ tấn công cung cấp đường dẫn như "a/b/c/../../etc/passwd",
và đổi tên "a/b/c" thành "a/b" trong khi thao tác mở đang diễn ra.

## Vệ sinh đường dẫn

Trước khi giải quyết các tấn công path traversal nói chung, hãy bắt đầu với vệ sinh đường dẫn.
Khi mô hình mối đe dọa của chương trình không bao gồm những kẻ tấn công có quyền truy cập vào hệ thống tệp cục bộ,
có thể đủ để xác thực các đường dẫn đầu vào không đáng tin trước khi sử dụng.

Thật không may, việc vệ sinh đường dẫn có thể khá phức tạp một cách bất ngờ,
đặc biệt đối với các chương trình di động phải xử lý cả đường dẫn Unix và Windows.
Ví dụ, trên Windows ```filepath.IsAbs(`\foo`)``` báo cáo `false`,
vì đường dẫn "\foo" là tương đối với ổ đĩa hiện tại.

Trong Go 1.20, chúng tôi đã thêm hàm [`path/filepath.IsLocal`](/pkg/path/filepath#IsLocal),
báo cáo liệu một đường dẫn có "cục bộ" không. Đường dẫn "cục bộ" là đường dẫn:

  - không thoát khỏi thư mục mà nó được đánh giá ("../etc/passwd" không được phép);
  - không phải đường dẫn tuyệt đối ("/etc/passwd" không được phép);
  - không rỗng ("" không được phép);
  - trên Windows, không phải tên dành riêng ("COM1" không được phép).

Trong Go 1.23, chúng tôi đã thêm hàm [`path/filepath.Localize`](/pkg/path/filepath#Localize),
chuyển đổi một đường dẫn được phân tách bằng / thành đường dẫn hệ điều hành cục bộ.

Các chương trình chấp nhận và hoạt động trên các đường dẫn có thể bị kiểm soát bởi kẻ tấn công nên hầu như
luôn sử dụng `filepath.IsLocal` hoặc `filepath.Localize` để xác thực hoặc vệ sinh các đường dẫn đó.

## Vượt xa vệ sinh

Vệ sinh đường dẫn không đủ khi kẻ tấn công có thể có quyền truy cập vào một phần
của hệ thống tệp cục bộ.

Hệ thống nhiều người dùng không phổ biến ngày nay, nhưng quyền truy cập của kẻ tấn công vào hệ thống tệp
vẫn có thể xảy ra theo nhiều cách.
Một tiện ích giải nén có thể giải nén tệp tar hoặc zip có thể bị
buộc giải nén một liên kết tượng trưng rồi giải nén tên tệp đi qua liên kết đó.
Một container runtime có thể cấp cho code không đáng tin quyền truy cập vào một phần của hệ thống tệp cục bộ.

Các chương trình có thể bảo vệ chống lại traversal symlink không chủ ý bằng cách sử dụng
hàm [`path/filepath.EvalSymlinks`](/pkg/path/filepath#EvalSymlinks)
để giải quyết các liên kết trong tên không đáng tin trước khi xác thực, nhưng như đã mô tả
ở trên, quy trình hai bước này dễ bị tổn thương bởi các cuộc đua TOCTOU.

Trước Go 1.24, lựa chọn an toàn hơn là sử dụng một gói như
[github.com/google/safeopen](/pkg/github.com/google/safeopen),
cung cấp các hàm kháng path traversal để mở một
tên tệp có thể không đáng tin trong một thư mục cụ thể.

## Giới thiệu `os.Root`

Trong Go 1.24, chúng tôi đang giới thiệu các API mới trong gói `os` để an toàn mở
một tệp ở một vị trí theo cách kháng traversal.

Kiểu [`os.Root`](/pkg/os#Root) mới đại diện cho một thư mục ở đâu đó
trong hệ thống tệp cục bộ. Mở một root với hàm [`os.OpenRoot`](/pkg/os#OpenRoot):

    root, err := os.OpenRoot("/some/root/directory")
    if err != nil {
      return err
    }
    defer root.Close()

`Root` cung cấp các phương thức để thao tác trên các tệp trong root.
Tất cả các phương thức này chấp nhận tên tệp tương đối với root,
và không cho phép bất kỳ thao tác nào có thể thoát khỏi root
bằng cách sử dụng các thành phần đường dẫn tương đối ("..") hoặc symlink.

    f, err := root.Open("path/to/file")

`Root` cho phép các thành phần đường dẫn tương đối và symlink không thoát khỏi root.
Ví dụ, `root.Open("a/../b")` được cho phép. Tên tệp được giải quyết sử dụng
ngữ nghĩa của nền tảng cục bộ: Trên các hệ thống Unix, điều này sẽ theo dõi
bất kỳ symlink nào trong "a" (miễn là liên kết đó không thoát khỏi root);
trong khi trên các hệ thống Windows điều này sẽ mở "b" (ngay cả khi "a" không tồn tại).

`Root` hiện cung cấp tập hợp các thao tác sau:

    func (*Root) Create(string) (*File, error)
    func (*Root) Lstat(string) (fs.FileInfo, error)
    func (*Root) Mkdir(string, fs.FileMode) error
    func (*Root) Open(string) (*File, error)
    func (*Root) OpenFile(string, int, fs.FileMode) (*File, error)
    func (*Root) OpenRoot(string) (*Root, error)
    func (*Root) Remove(string) error
    func (*Root) Stat(string) (fs.FileInfo, error)

Ngoài kiểu `Root`, hàm mới
[`os.OpenInRoot`](/pkg/os#OpenInRoot)
cung cấp một cách đơn giản để mở một tên tệp có thể không đáng tin trong một
thư mục cụ thể:

    f, err := os.OpenInRoot("/some/root/directory", untrustedFilename)

Kiểu `Root` cung cấp một API đơn giản, an toàn, di động để hoạt động với các tên tệp không đáng tin.

## Lưu ý và cân nhắc

### Unix

Trên các hệ thống Unix, `Root` được triển khai bằng họ các lời gọi hệ thống `openat`.
Một `Root` chứa một file descriptor tham chiếu đến thư mục root của nó và sẽ theo dõi
thư mục đó qua các lần đổi tên hoặc xóa.

`Root` bảo vệ chống lại traversal symlink nhưng không giới hạn traversal
của các điểm mount. Ví dụ, `Root` không ngăn traversal của
các Linux bind mount. Mô hình mối đe dọa của chúng tôi là `Root` bảo vệ chống lại
các cấu trúc hệ thống tệp có thể được tạo bởi người dùng thông thường (chẳng hạn
như symlink), nhưng không xử lý những cấu trúc yêu cầu quyền root để
tạo ra (chẳng hạn như bind mount).

### Windows

Trên Windows, `Root` mở một handle tham chiếu đến thư mục root của nó.
Handle mở ngăn thư mục đó bị đổi tên hoặc xóa cho đến khi `Root` được đóng.

`Root` ngăn truy cập vào các tên thiết bị Windows dành riêng như `NUL` và `COM1`.

### WASI

Trên WASI, gói `os` sử dụng API hệ thống tệp WASI preview 1,
được thiết kế để cung cấp quyền truy cập hệ thống tệp kháng traversal.
Tuy nhiên, không phải tất cả các triển khai WASI đều hỗ trợ đầy đủ sandboxing hệ thống tệp,
và biện pháp phòng thủ traversal của `Root` bị giới hạn bởi khả năng cung cấp
của triển khai WASI.

### GOOS=js

Khi GOOS=js, gói `os` sử dụng API hệ thống tệp Node.js.
API này không bao gồm họ các hàm openat,
và do đó `os.Root` dễ bị tổn thương bởi các cuộc đua TOCTOU (time-of-check-time-of-use) trong xác thực symlink
trên nền tảng này.

Khi GOOS=js, một `Root` tham chiếu đến tên thư mục thay vì file descriptor,
và không theo dõi các thư mục qua các lần đổi tên.

### Plan 9

Plan 9 không có symlink.
Trên Plan 9, một `Root` tham chiếu đến tên thư mục và thực hiện vệ sinh từ vựng
của tên tệp.

### Hiệu năng

Các thao tác `Root` trên tên tệp có nhiều thành phần thư mục có thể đắt hơn nhiều
so với thao tác không phải `Root` tương đương. Giải quyết các thành phần ".." cũng có thể tốn kém.
Các chương trình muốn giới hạn chi phí của các thao tác hệ thống tệp có thể sử dụng `filepath.Clean` để
loại bỏ các thành phần ".." khỏi tên tệp đầu vào, và có thể muốn giới hạn số lượng
thành phần thư mục.

## Ai nên sử dụng os.Root?

Bạn nên sử dụng `os.Root` hoặc `os.OpenInRoot` nếu:

  - bạn đang mở một tệp trong một thư mục; VÀ
  - thao tác không nên truy cập tệp bên ngoài thư mục đó.

Ví dụ, một chương trình giải nén archive viết các tệp vào một thư mục đầu ra nên sử dụng
`os.Root`, vì các tên tệp có thể không đáng tin và sẽ không đúng
khi viết tệp bên ngoài thư mục đầu ra.

Tuy nhiên, một chương trình dòng lệnh viết đầu ra đến một vị trí được chỉ định bởi người dùng
không nên sử dụng `os.Root`, vì tên tệp không đáng ngờ và có thể
tham chiếu bất cứ đâu trên hệ thống tệp.

Như một quy tắc tốt, code gọi `filepath.Join` để kết hợp một thư mục cố định
và tên tệp được cung cấp bên ngoài có thể nên sử dụng `os.Root` thay thế.

    // Điều này có thể mở một tệp không nằm trong baseDirectory.
    f, err := os.Open(filepath.Join(baseDirectory, filename))

    // Điều này sẽ chỉ mở các tệp trong baseDirectory.
    f, err := os.OpenInRoot(baseDirectory, filename)

## Công việc tương lai

API `os.Root` là mới trong Go 1.24.
Chúng tôi dự kiến sẽ thực hiện các bổ sung và tinh chỉnh cho nó trong các bản phát hành tương lai.

Triển khai hiện tại ưu tiên tính đúng đắn và an toàn hơn hiệu năng.
Các phiên bản tương lai sẽ tận dụng các API đặc thù của nền tảng, chẳng hạn như
`openat2` của Linux, để cải thiện hiệu năng nơi có thể.

Có một số thao tác hệ thống tệp mà `Root` chưa hỗ trợ, chẳng hạn như
tạo liên kết tượng trưng và đổi tên tệp. Nơi có thể, chúng tôi sẽ thêm hỗ trợ cho các
thao tác này. Danh sách các hàm bổ sung đang trong tiến trình có trong
[go.dev/issue/67002](/issue/67002).
