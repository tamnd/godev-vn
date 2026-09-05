<!--{
  "Title": "Quản lý dependency"
}-->

Khi code của bạn sử dụng các package bên ngoài, những package đó (được phân phối dưới dạng module) trở thành dependency. Theo thời gian, bạn có thể cần nâng cấp hoặc thay thế chúng. Go cung cấp các công cụ quản lý dependency giúp bạn giữ cho các ứng dụng Go an toàn khi tích hợp các dependency bên ngoài.

Chủ đề này mô tả cách thực hiện các tác vụ quản lý dependency trong code của bạn. Bạn có thể thực hiện hầu hết các tác vụ này bằng công cụ Go. Chủ đề này cũng mô tả cách thực hiện một số tác vụ khác liên quan đến dependency mà bạn có thể thấy hữu ích.

**Xem thêm**

*   Nếu bạn mới làm quen với việc làm việc với dependency dưới dạng module, hãy xem [Hướng dẫn bắt đầu](/doc/tutorial/getting-started) để có phần giới thiệu ngắn gọn.
*   Sử dụng lệnh `go` để quản lý dependency giúp đảm bảo rằng các yêu cầu của bạn luôn nhất quán và nội dung tệp go.mod hợp lệ. Để tham khảo về các lệnh, xem [Lệnh go](/cmd/go/). Bạn cũng có thể nhận trợ giúp từ dòng lệnh bằng cách gõ `go help` _tên-lệnh_, ví dụ `go help mod tidy`.
*   Các lệnh Go bạn dùng để thay đổi dependency sẽ chỉnh sửa tệp go.mod của bạn. Để biết thêm về nội dung tệp, xem [Tài liệu tham chiếu tệp go.mod](/doc/modules/gomod-ref).
*   Làm cho trình soạn thảo hoặc IDE của bạn nhận thức được Go module có thể giúp việc quản lý chúng trở nên dễ dàng hơn. Để biết thêm về các trình soạn thảo hỗ trợ Go, xem [Plugin trình soạn thảo và IDE](/doc/editors.html).
*   Chủ đề này không mô tả cách phát triển, xuất bản và đánh số phiên bản module cho người khác sử dụng. Để biết thêm về điều đó, xem [Phát triển và xuất bản module](developing).

## Quy trình sử dụng và quản lý dependency {#workflow}

Bạn có thể lấy và sử dụng các package hữu ích với công cụ Go. Trên [pkg.go.dev](https://pkg.go.dev), bạn có thể tìm kiếm các package có thể hữu ích, sau đó dùng lệnh `go` để import các package đó vào code của bạn để gọi các hàm của chúng.

Danh sách dưới đây liệt kê các bước quản lý dependency phổ biến nhất. Để biết thêm về từng bước, xem các phần trong chủ đề này.

1. [Tìm kiếm các package hữu ích](#locating_packages) trên [pkg.go.dev](https://pkg.go.dev).
1. [Import các package](#locating_packages) bạn muốn vào code của mình.
1. Thêm code của bạn vào một module để theo dõi dependency (nếu nó chưa nằm trong module). Xem [Bật tính năng theo dõi dependency](#enable_tracking).
1. [Thêm các package bên ngoài làm dependency](#adding_dependency) để bạn có thể quản lý chúng.
1. [Nâng cấp hoặc hạ cấp các phiên bản dependency](#upgrading) khi cần theo thời gian.

## Quản lý dependency dưới dạng module {#modules}

Trong Go, bạn quản lý dependency dưới dạng module chứa các package bạn import. Quy trình này được hỗ trợ bởi:

*   Một **hệ thống xuất bản phi tập trung** cho module và truy xuất mã nguồn. Các lập trình viên đưa module của họ lên từ kho lưu trữ của bản thân và xuất bản kèm theo số phiên bản để các lập trình viên khác sử dụng.
*   Một **công cụ tìm kiếm package** và trình duyệt tài liệu (pkg.go.dev) để bạn có thể tìm kiếm module. Xem [Tìm kiếm và import các package hữu ích](#locating_packages).
*   Một **quy ước đánh số phiên bản** module giúp bạn hiểu mức độ ổn định và các đảm bảo tương thích ngược của module. Xem [Đánh số phiên bản module](version-numbers).
*   **Công cụ Go** giúp bạn quản lý dependency dễ dàng hơn, bao gồm tải mã nguồn module, nâng cấp và nhiều hơn nữa. Xem các phần trong chủ đề này để biết thêm.

## Tìm kiếm và import các package hữu ích {#locating_packages}

Bạn có thể tìm kiếm trên [pkg.go.dev](https://pkg.go.dev) để tìm các package có hàm mà bạn có thể thấy hữu ích.

Khi bạn tìm thấy một package muốn sử dụng trong code, hãy xác định đường dẫn package ở đầu trang và nhấn nút Copy path để sao chép đường dẫn vào clipboard. Trong code của bạn, dán đường dẫn vào một câu lệnh import, như trong ví dụ sau:

```
import "rsc.io/quote"
```

Sau khi code của bạn import package, hãy bật theo dõi dependency và lấy code của package để biên dịch. Để biết thêm, xem [Bật tính năng theo dõi dependency trong code](#enable_tracking) và [Thêm dependency](#adding_dependency).

## Bật tính năng theo dõi dependency trong code {#enable_tracking}

Để theo dõi và quản lý các dependency bạn thêm vào, hãy bắt đầu bằng cách đưa code của bạn vào module riêng của nó. Điều này tạo ra một tệp go.mod tại gốc của cây mã nguồn. Các dependency bạn thêm vào sẽ được liệt kê trong tệp đó.

Để thêm code vào module riêng của nó, dùng [lệnh `go mod init`](/ref/mod#go-mod-init). Ví dụ, từ dòng lệnh, chuyển đến thư mục gốc của code, sau đó chạy lệnh như trong ví dụ sau:

```
$ go mod init example/mymodule
```

Đối số của lệnh `go mod init` là đường dẫn module của module. Nếu có thể, đường dẫn module nên là vị trí kho lưu trữ của mã nguồn.

Nếu lúc đầu bạn chưa biết vị trí kho lưu trữ cuối cùng của module, hãy dùng một giá trị thay thế an toàn. Đây có thể là tên một tên miền bạn sở hữu hoặc một tên khác bạn kiểm soát (như tên công ty), cùng với một đường dẫn theo tên hoặc thư mục nguồn của module. Để biết thêm, xem [Đặt tên cho module](#naming_module).

Khi bạn dùng công cụ Go để quản lý dependency, công cụ đó sẽ cập nhật tệp go.mod để duy trì danh sách dependency hiện tại của bạn.

Khi bạn thêm dependency, công cụ Go cũng tạo một tệp go.sum chứa các checksum của các module mà bạn phụ thuộc. Go dùng tệp này để xác minh tính toàn vẹn của các tệp module được tải xuống, đặc biệt quan trọng với các lập trình viên khác cùng làm việc trên dự án của bạn.

Hãy đưa cả tệp go.mod và go.sum vào kho lưu trữ cùng với code của bạn.

Xem [tài liệu tham chiếu go.mod](/doc/modules/gomod-ref) để biết thêm.

## Đặt tên cho module {#naming_module}

Khi bạn chạy `go mod init` để tạo module theo dõi dependency, bạn chỉ định đường dẫn module đóng vai trò tên của module. Đường dẫn module trở thành tiền tố đường dẫn import cho các package trong module. Hãy đảm bảo chỉ định đường dẫn module không xung đột với đường dẫn module của các module khác.

Ở mức tối thiểu, một đường dẫn module chỉ cần chỉ ra điều gì đó về nguồn gốc của nó, chẳng hạn tên công ty, tác giả hoặc chủ sở hữu. Nhưng đường dẫn cũng có thể mô tả rõ hơn module là gì hoặc làm gì.

Đường dẫn module thường có dạng sau:

```
<prefix>/<descriptive-text>
```

* _Prefix_ thường là một chuỗi mô tả một phần về module, chẳng hạn một chuỗi mô tả nguồn gốc của nó. Đây có thể là:

    *   Vị trí kho lưu trữ nơi công cụ Go có thể tìm thấy mã nguồn module (bắt buộc nếu bạn xuất bản module).

        Ví dụ, có thể là `github.com/<tên-dự-án>/`.

        Hãy dùng thực hành tốt nhất này nếu bạn nghĩ mình có thể xuất bản module cho người khác sử dụng. Để biết thêm về xuất bản, xem [Phát triển và xuất bản module](/doc/modules/developing).

    *   Một tên bạn kiểm soát.

        Nếu bạn không dùng tên kho lưu trữ, hãy chọn một prefix mà bạn chắc chắn người khác sẽ không dùng. Tên công ty của bạn là lựa chọn tốt. Tránh các thuật ngữ phổ biến như `widgets`, `utilities` hoặc `app`.

* Để chọn _descriptive text_, một dự án tốt sẽ là tên dự án. Nhớ rằng tên package đảm nhận phần lớn trách nhiệm mô tả chức năng. Đường dẫn module tạo ra namespace cho các tên package đó.

**Các tiền tố đường dẫn module được đặt trước**

Go đảm bảo rằng các chuỗi sau sẽ không được dùng trong tên package.

- `test` -- Bạn có thể dùng `test` làm tiền tố đường dẫn module cho một module mà code được thiết kế để kiểm thử cục bộ các hàm trong module khác.

    Dùng tiền tố đường dẫn `test` cho các module được tạo như một phần của quá trình kiểm thử. Ví dụ, bản thân bài kiểm thử có thể chạy `go mod init test` và sau đó thiết lập module đó theo một cách cụ thể để kiểm thử với một công cụ phân tích mã nguồn Go.

- `example` -- Được dùng làm tiền tố đường dẫn module trong một số tài liệu Go, chẳng hạn trong các hướng dẫn nơi bạn tạo module chỉ để theo dõi dependency.

    Lưu ý rằng tài liệu Go cũng dùng `example.com` để minh họa khi ví dụ có thể là một module được xuất bản.

## Thêm dependency {#adding_dependency}

Sau khi bạn import các package từ một module đã xuất bản, bạn có thể thêm module đó để quản lý như một dependency bằng cách dùng [lệnh `go get`](/cmd/go/#hdr-Add_dependencies_to_current_module_and_install_them).

Lệnh thực hiện các việc sau:

*   Nếu cần, nó thêm các chỉ thị `require` vào tệp go.mod cho các module cần thiết để build các package được đặt tên trên dòng lệnh. Một chỉ thị `require` theo dõi phiên bản tối thiểu của module mà module của bạn phụ thuộc. Xem [tài liệu tham chiếu go.mod](/doc/modules/gomod-ref) để biết thêm.
*   Nếu cần, nó tải mã nguồn module để bạn có thể biên dịch các package phụ thuộc vào chúng. Nó có thể tải module từ một proxy module như proxy.golang.org hoặc trực tiếp từ kho lưu trữ quản lý phiên bản. Mã nguồn được lưu trong bộ nhớ cache cục bộ.

    Bạn có thể đặt vị trí mà công cụ Go tải module về từ đó. Để biết thêm, xem [Chỉ định một proxy server module](#proxy_server).

Dưới đây mô tả một vài ví dụ.

*   Để thêm tất cả dependency cho một package trong module, chạy lệnh như sau ("." đề cập đến package trong thư mục hiện tại):

    ```
    $ go get .
    ```

*   Để thêm một dependency cụ thể, chỉ định đường dẫn module của nó làm đối số cho lệnh.

    ```
    $ go get example.com/theirmodule
    ```

Lệnh cũng xác thực mỗi module mà nó tải xuống. Điều này đảm bảo rằng module không bị thay đổi so với khi nó được xuất bản. Nếu module đã thay đổi kể từ khi xuất bản -- ví dụ, lập trình viên đã thay đổi nội dung của commit -- công cụ Go sẽ hiển thị lỗi bảo mật. Việc kiểm tra xác thực này bảo vệ bạn khỏi các module có thể đã bị giả mạo.

## Lấy phiên bản dependency cụ thể {#getting_version}

Bạn có thể lấy một phiên bản cụ thể của module dependency bằng cách chỉ định phiên bản trong lệnh `go get`. Lệnh này cập nhật chỉ thị `require` trong tệp go.mod (mặc dù bạn cũng có thể cập nhật thủ công).

Bạn có thể muốn làm điều này nếu:

*   Bạn muốn lấy một phiên bản pre-release cụ thể của module để thử nghiệm.
*   Bạn phát hiện ra phiên bản bạn đang yêu cầu không hoạt động với bạn, vì vậy bạn muốn lấy một phiên bản bạn biết mình có thể dựa vào.
*   Bạn muốn nâng cấp hoặc hạ cấp một module bạn đang yêu cầu.

Dưới đây là các ví dụ sử dụng [lệnh `go get`](/ref/mod#go-get):

*   Để lấy một phiên bản có số cụ thể, thêm dấu @ vào đường dẫn module theo sau là phiên bản bạn muốn:

    ```
    $ go get example.com/theirmodule@v1.3.4
    ```

*   Để lấy phiên bản mới nhất, thêm `@latest` vào đường dẫn module:

    ```
    $ go get example.com/theirmodule@latest
    ```

Ví dụ sau về chỉ thị `require` trong tệp go.mod (xem [tài liệu tham chiếu go.mod](/doc/modules/gomod-ref) để biết thêm) minh họa cách yêu cầu một số phiên bản cụ thể:

```
require example.com/theirmodule v1.3.4
```

## Khám phá các bản cập nhật hiện có {#discovering_updates}

Bạn có thể kiểm tra xem có các phiên bản mới hơn của các dependency bạn đang dùng trong module hiện tại hay không. Dùng lệnh `go list` để hiển thị danh sách các dependency của module, cùng với phiên bản mới nhất hiện có cho mỗi module. Sau khi phát hiện các bản nâng cấp hiện có, bạn có thể thử chúng với code của mình để quyết định có nên nâng cấp lên phiên bản mới hay không.

Để biết thêm về lệnh `go list`, xem [`go list -m`](/ref/mod#go-list-m).

Dưới đây là một vài ví dụ.

*   Liệt kê tất cả các module là dependency của module hiện tại, cùng với phiên bản mới nhất hiện có cho mỗi module:

    ```
    $ go list -m -u all
    ```

*   Hiển thị phiên bản mới nhất hiện có cho một module cụ thể:

    ```
    $ go list -m -u example.com/theirmodule
    ```

## Nâng cấp hoặc hạ cấp dependency {#upgrading}

Bạn có thể nâng cấp hoặc hạ cấp một module dependency bằng cách dùng công cụ Go để khám phá các phiên bản hiện có, sau đó thêm một phiên bản khác làm dependency.

1. Để khám phá các phiên bản mới, dùng lệnh `go list` như mô tả trong [Khám phá các bản cập nhật hiện có](#discovering_updates).

1. Để thêm một phiên bản cụ thể làm dependency, dùng lệnh `go get` như mô tả trong [Lấy phiên bản dependency cụ thể](#getting_version).

## Đồng bộ hóa các dependency trong code {#synchronizing}

Bạn có thể đảm bảo rằng bạn đang quản lý dependency cho tất cả các package được import trong code, đồng thời xóa các dependency cho các package không còn được import nữa.

Điều này có thể hữu ích khi bạn đã thực hiện nhiều thay đổi đối với code và các dependency, có thể tạo ra một tập hợp các dependency được quản lý và các module được tải xuống không còn khớp chính xác với tập hợp được yêu cầu bởi các package được import trong code.

Để giữ tập dependency được quản lý gọn gàng, dùng lệnh `go mod tidy`. Dựa trên tập hợp các package được import trong code, lệnh này chỉnh sửa tệp go.mod để thêm các module cần thiết nhưng còn thiếu. Nó cũng xóa các module không dùng đến mà không cung cấp bất kỳ package liên quan nào.

Lệnh này không có đối số ngoại trừ một cờ, -v, in thông tin về các module bị xóa.

```
$ go mod tidy
```

## Phát triển và kiểm thử dựa trên mã nguồn module chưa xuất bản {#unpublished}

Bạn có thể chỉ định rằng code nên dùng các module dependency có thể chưa được xuất bản. Code cho các module này có thể nằm trong kho lưu trữ của chúng, trong một fork của các kho lưu trữ đó, hoặc trên một ổ đĩa cùng với module hiện tại đang sử dụng chúng.

Bạn có thể muốn làm điều này khi:

*   Bạn muốn thực hiện các thay đổi của riêng mình đối với code của một module bên ngoài, chẳng hạn sau khi fork và/hoặc clone nó. Ví dụ, bạn có thể muốn chuẩn bị một bản sửa lỗi cho module, sau đó gửi nó như một pull request đến lập trình viên của module.
*   Bạn đang build một module mới và chưa xuất bản nó, vì vậy nó không có sẵn trên kho lưu trữ nơi lệnh `go get` có thể truy cập.

### Yêu cầu mã nguồn module từ thư mục cục bộ {#local_directory}

Bạn có thể chỉ định rằng code cho một module bắt buộc nằm trên cùng ổ đĩa cục bộ với code đang yêu cầu nó. Bạn có thể thấy điều này hữu ích khi bạn:

*   Đang phát triển module riêng của mình và muốn kiểm thử từ module hiện tại.
*   Đang sửa lỗi hoặc thêm tính năng cho một module bên ngoài và muốn kiểm thử từ module hiện tại. (Lưu ý rằng bạn cũng có thể yêu cầu module bên ngoài từ fork kho lưu trữ của bạn. Để biết thêm, xem [Yêu cầu mã nguồn module bên ngoài từ fork kho lưu trữ của bạn](#external_fork).)

Để yêu cầu các lệnh Go sử dụng bản sao cục bộ của mã nguồn module, dùng chỉ thị `replace` trong tệp go.mod để thay thế đường dẫn module được cho trong chỉ thị `require`. Xem [tài liệu tham chiếu go.mod](/doc/modules/gomod-ref) để biết thêm về các chỉ thị.

Trong ví dụ tệp go.mod sau, module hiện tại yêu cầu module bên ngoài `example.com/theirmodule`, với số phiên bản không tồn tại (`v0.0.0-unpublished`) được dùng để đảm bảo việc thay thế hoạt động đúng. Chỉ thị `replace` sau đó thay thế đường dẫn module gốc bằng `../theirmodule`, một thư mục nằm cùng cấp với thư mục của module hiện tại.

```
module example.com/mymodule

go 1.23.0

require example.com/theirmodule v0.0.0-unpublished

replace example.com/theirmodule v0.0.0-unpublished => ../theirmodule
```

Khi thiết lập cặp `require`/`replace`, dùng các lệnh [`go mod edit`](/ref/mod#go-mod-edit) và [`go get`](/ref/mod#go-get) để đảm bảo các yêu cầu được mô tả trong tệp vẫn nhất quán:

```
$ go mod edit -replace=example.com/theirmodule@v0.0.0-unpublished=../theirmodule
$ go get example.com/theirmodule@v0.0.0-unpublished
```

**Lưu ý:** Khi bạn dùng chỉ thị `replace`, công cụ Go không xác thực các module bên ngoài như mô tả trong [Thêm dependency](#adding_dependency).

Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

Go 1.18 bổ sung [workspace mode](/blog/get-familiar-with-workspaces) vào Go, cho phép bạn làm việc trên nhiều module đồng thời. Xem [Hướng dẫn: Bắt đầu với workspace nhiều module](/doc/tutorial/workspaces).

### Yêu cầu mã nguồn module bên ngoài từ fork kho lưu trữ của bạn {#external_fork}

Khi bạn đã fork kho lưu trữ của một module bên ngoài (chẳng hạn để sửa lỗi trong code của module hoặc thêm tính năng), bạn có thể yêu cầu công cụ Go sử dụng fork của bạn làm nguồn cho module. Điều này có thể hữu ích để kiểm thử các thay đổi từ code của bạn. (Lưu ý rằng bạn cũng có thể yêu cầu mã nguồn module trong một thư mục trên ổ đĩa cục bộ cùng với module đang yêu cầu nó. Để biết thêm, xem [Yêu cầu mã nguồn module từ thư mục cục bộ](#local_directory).)

Bạn thực hiện điều này bằng cách dùng chỉ thị `replace` trong tệp go.mod để thay thế đường dẫn module gốc của module bên ngoài bằng đường dẫn đến fork trong kho lưu trữ của bạn. Điều này hướng dẫn công cụ Go sử dụng đường dẫn thay thế (vị trí fork) khi biên dịch, ví dụ, trong khi vẫn cho phép bạn giữ nguyên các câu lệnh `import` theo đường dẫn module gốc.

Để biết thêm về chỉ thị `replace`, xem [tài liệu tham chiếu tệp go.mod](gomod-ref).

Trong ví dụ tệp go.mod sau, module hiện tại yêu cầu module bên ngoài `example.com/theirmodule`. Chỉ thị `replace` sau đó thay thế đường dẫn module gốc bằng `example.com/myfork/theirmodule`, một fork của kho lưu trữ chính của module.

```
module example.com/mymodule

go 1.23.0

require example.com/theirmodule v1.2.3

replace example.com/theirmodule v1.2.3 => example.com/myfork/theirmodule v1.2.3-fixed
```

Khi thiết lập cặp `require`/`replace`, dùng các lệnh công cụ Go để đảm bảo các yêu cầu được mô tả trong tệp vẫn nhất quán. Dùng lệnh [`go list`](/ref/mod#go-list-m) để lấy phiên bản đang được dùng bởi module hiện tại. Sau đó dùng lệnh [`go mod edit`](/ref/mod#go-mod-edit) để thay thế module bắt buộc bằng fork:

```
$ go list -m example.com/theirmodule
example.com/theirmodule v1.2.3
$ go mod edit -replace=example.com/theirmodule@v1.2.3=example.com/myfork/theirmodule@v1.2.3-fixed
```

**Lưu ý:** Khi bạn dùng chỉ thị `replace`, công cụ Go không xác thực các module bên ngoài như mô tả trong [Thêm dependency](#adding_dependency).

Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

## Lấy một commit cụ thể bằng định danh kho lưu trữ {#repo_identifier}

Bạn có thể dùng lệnh `go get` để thêm code chưa xuất bản cho một module từ một commit cụ thể trong kho lưu trữ của nó.

Để làm điều này, bạn dùng lệnh `go get`, chỉ định code bạn muốn với dấu `@`. Khi bạn dùng `go get`, lệnh sẽ thêm vào tệp go.mod của bạn một chỉ thị `require` yêu cầu module bên ngoài, sử dụng số pseudo-version dựa trên chi tiết về commit.

Các ví dụ sau đây cung cấp một vài minh họa. Chúng dựa trên một module mà mã nguồn nằm trong kho lưu trữ git.

*   Để lấy module tại một commit cụ thể, thêm dạng @<em>commithash</em>:

    ```
    $ go get example.com/theirmodule@4cf76c2
    ```

*   Để lấy module tại một nhánh cụ thể, thêm dạng @<em>branchname</em>:

    ```
    $ go get example.com/theirmodule@bugfixes
    ```

## Xóa một dependency {#removing_dependency}

Khi code của bạn không còn sử dụng bất kỳ package nào trong một module, bạn có thể ngừng theo dõi module đó như một dependency.

Để ngừng theo dõi tất cả các module không dùng, chạy [lệnh `go mod tidy`](/ref/mod#go-mod-tidy). Lệnh này cũng có thể thêm các dependency còn thiếu cần thiết để build các package trong module của bạn.

```
$ go mod tidy
```

Để xóa một dependency cụ thể, dùng [lệnh `go get`](/ref/mod#go-get), chỉ định đường dẫn module của module và thêm `@none`, như trong ví dụ sau:

```
$ go get example.com/theirmodule@none
```

Lệnh `go get` cũng sẽ hạ cấp hoặc xóa các dependency khác phụ thuộc vào module đã xóa.

## Tool dependencies {#tools}

Tool dependencies cho phép bạn quản lý các công cụ dành cho lập trình viên được viết bằng Go và được sử dụng khi làm việc trên module của bạn. Ví dụ, bạn có thể dùng [`stringer`](https://pkg.go.dev/golang.org/x/tools/cmd/stringer) với [`go generate`](/blog/generate), hoặc một linter hay formatter cụ thể như một phần của việc chuẩn bị thay đổi để nộp.

Từ Go 1.24 trở lên, bạn có thể thêm tool dependency với:

```
$ go get -tool golang.org/x/tools/cmd/stringer
```

Lệnh này sẽ thêm [chỉ thị `tool`](/ref/mod/#go-mod-file-tool) vào tệp `go.mod` của bạn và đảm bảo các chỉ thị require cần thiết có mặt. Sau khi chỉ thị này được thêm vào, bạn có thể chạy tool bằng cách truyền thành phần cuối cùng [không phải phiên bản chính](/ref/mod#major-version-suffixes) của đường dẫn import của tool cho `go tool`:

```
$ go tool stringer
```

Trong trường hợp nhiều tool chia sẻ phần cuối cùng của đường dẫn, hoặc phần đó trùng với một trong các tool được cung cấp cùng bản phân phối Go, bạn phải truyền đường dẫn package đầy đủ:

```
$ go tool golang.org/x/tools/cmd/stringer
```

Để xem danh sách tất cả các tool hiện có, chạy `go tool` không có đối số:

```
$ go tool
```

Bạn có thể thêm thủ công chỉ thị `tool` vào tệp `go.mod`, nhưng bạn phải đảm bảo có chỉ thị `require` cho module định nghĩa tool. Cách dễ nhất để thêm bất kỳ chỉ thị `require` còn thiếu là chạy:

```
$ go mod tidy
```

Các yêu cầu cần thiết để thỏa mãn tool dependencies hoạt động như bất kỳ yêu cầu nào khác trong [đồ thị module](/ref/mod#glos-module-graph) của bạn. Chúng tham gia [lựa chọn phiên bản tối thiểu](/ref/mod#minimal-version-selection) và tôn trọng các chỉ thị `require`, `replace` và `exclude`. Do module pruning, khi bạn phụ thuộc vào một module bản thân nó có tool dependency, các yêu cầu chỉ tồn tại để thỏa mãn tool dependency đó thường không trở thành yêu cầu của module bạn.

[Meta-pattern](/cmd/go#hdr-Package_lists_and_patterns) `tool` cung cấp cách thực hiện các thao tác trên tất cả tool cùng lúc. Ví dụ bạn có thể nâng cấp tất cả tool với `go get tool`,
tương đương với `go get tool@upgrade`, hoặc cài đặt tất cả vào $GOBIN với `go install tool`.

Trong các phiên bản Go trước 1.24, bạn có thể đạt được điều gì đó tương tự với chỉ thị `tool` bằng cách thêm một blank import vào một tệp go trong module được loại trừ khỏi bản build sử dụng [build constraints](/pkg/go/build/#hdr-Build_Constraints). Nếu bạn làm vậy, bạn có thể dùng `go run` với đường dẫn package đầy đủ để chạy tool.

## Chỉ định một proxy server module {#proxy_server}

Khi bạn dùng công cụ Go để làm việc với module, theo mặc định các công cụ tải module từ proxy.golang.org (một mirror module công khai do Google điều hành) hoặc trực tiếp từ kho lưu trữ của module. Bạn có thể chỉ định rằng công cụ Go nên dùng một proxy server khác để tải xuống và xác thực module.

Bạn có thể muốn làm điều này nếu bạn (hoặc nhóm của bạn) đã thiết lập hoặc chọn một proxy server module khác mà bạn muốn sử dụng. Ví dụ, một số người thiết lập một proxy server module để có khả năng kiểm soát tốt hơn cách các dependency được sử dụng.

Để chỉ định một proxy server module khác cho công cụ Go sử dụng, đặt biến môi trường `GOPROXY` thành URL của một hoặc nhiều server. Công cụ Go sẽ thử từng URL theo thứ tự bạn chỉ định. Theo mặc định, `GOPROXY` chỉ định proxy module công khai do Google điều hành trước, sau đó tải trực tiếp từ kho lưu trữ của module (như được chỉ định trong đường dẫn module của nó):

```
GOPROXY="https://proxy.golang.org,direct"
```

Để biết thêm về biến môi trường `GOPROXY`, bao gồm các giá trị để hỗ trợ các hành vi khác, xem [tài liệu tham chiếu lệnh `go`](/cmd/go/#hdr-Module_downloading_and_verification).

Bạn có thể đặt biến thành các URL cho các proxy server module khác, phân tách URL bằng dấu phẩy hoặc dấu ống.

*   Khi bạn dùng dấu phẩy, công cụ Go sẽ thử URL tiếp theo trong danh sách chỉ khi URL hiện tại trả về HTTP 404 hoặc 410.

    ```
    GOPROXY="https://proxy.example.com,https://proxy2.example.com"
    ```

*   Khi bạn dùng dấu ống, công cụ Go sẽ thử URL tiếp theo trong danh sách bất kể mã lỗi HTTP.

    ```
    GOPROXY="https://proxy.example.com|https://proxy2.example.com"
    ```


Các module Go thường được phát triển và phân phối trên các server quản lý phiên bản và proxy module không có sẵn trên internet công cộng. Bạn có thể đặt biến môi trường `GOPRIVATE` để cấu hình lệnh `go` tải xuống và build module từ các nguồn riêng tư. Khi đó lệnh `go` có thể tải xuống và build module từ các nguồn riêng tư.

Các biến môi trường `GOPRIVATE` hoặc `GONOPROXY` có thể được đặt thành danh sách các pattern glob khớp với các tiền tố module là riêng tư và không nên được yêu cầu từ bất kỳ proxy nào. Ví dụ:

```
GOPRIVATE=*.corp.example.com,*.research.example.com
```
