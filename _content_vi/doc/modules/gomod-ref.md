<!--{
  "Title": "Tài liệu tham chiếu tệp go.mod"
}-->

Mỗi module Go được định nghĩa bởi một tệp go.mod mô tả các thuộc tính của module, bao gồm các dependency của nó đối với các module khác và đối với các phiên bản Go.

Các thuộc tính đó bao gồm:

* **Đường dẫn module** của module hiện tại. Đây phải là một vị trí mà công cụ Go có thể dùng để tải module về, chẳng hạn vị trí kho lưu trữ chứa mã nguồn module. Giá trị này đóng vai trò định danh duy nhất khi kết hợp với số phiên bản của module. Nó cũng là tiền tố của đường dẫn package cho tất cả các package trong module. Để biết thêm về cách Go xác định vị trí module, xem <a href="/ref/mod#vcs-find">Tài liệu tham chiếu Go Modules</a>.
* **Phiên bản Go tối thiểu** mà module hiện tại yêu cầu.
* Danh sách các phiên bản tối thiểu của các **module khác mà module hiện tại yêu cầu**.
* Các chỉ thị tùy chọn để **thay thế** một module bắt buộc bằng một phiên bản module khác hoặc một thư mục cục bộ, hoặc để **loại trừ** một phiên bản cụ thể của module bắt buộc.

Go tạo tệp go.mod khi bạn chạy [lệnh `go mod init`](/ref/mod#go-mod-init). Ví dụ sau tạo một tệp go.mod, đặt đường dẫn module thành example/mymodule:

```
$ go mod init example/mymodule
```

Sử dụng các lệnh `go` để quản lý dependency. Các lệnh này đảm bảo rằng các yêu cầu được mô tả trong tệp go.mod luôn nhất quán và nội dung tệp go.mod hợp lệ. Các lệnh đó bao gồm [`go get`](/ref/mod#go-get), [`go mod tidy`](/ref/mod#go-mod-tidy) và [`go mod edit`](/ref/mod#go-mod-edit).

Để tham khảo về các lệnh `go`, xem [Lệnh go](/cmd/go/).
Bạn cũng có thể nhận trợ giúp từ dòng lệnh bằng cách gõ `go help` _tên-lệnh_, ví dụ `go help mod tidy`.

**Xem thêm**

* Công cụ Go sẽ thay đổi tệp go.mod khi bạn dùng chúng để quản lý dependency. Để biết thêm, xem [Quản lý dependency](/doc/modules/managing-dependencies).
* Để biết thêm chi tiết và các ràng buộc liên quan đến tệp go.mod, xem [Tài liệu tham chiếu Go modules](/ref/mod#go-mod-file).

## Ví dụ {#example}

Một tệp go.mod bao gồm các chỉ thị như trong ví dụ sau. Chúng được mô tả ở các phần khác trong chủ đề này.

```
module example.com/mymodule

go 1.14

require (
    example.com/othermodule v1.2.3
    example.com/thismodule v1.2.3
    example.com/thatmodule v1.2.3
)

replace example.com/thatmodule => ../thatmodule
exclude example.com/thismodule v1.3.0
```

## module {#module}

Khai báo đường dẫn module của module, là định danh duy nhất của module (khi kết hợp với số phiên bản module). Đường dẫn module trở thành tiền tố import cho tất cả các package mà module chứa.

Để biết thêm, xem [chỉ thị `module`](/ref/mod#go-mod-file-module) trong Tài liệu tham chiếu Go Modules.

### Cú pháp {#module-syntax}

<pre>module <var>module-path</var></pre>

<dl>
    <dt>module-path</dt>
    <dd>Đường dẫn module của module, thường là vị trí kho lưu trữ mà công cụ Go
      có thể dùng để tải module. Với các phiên bản module v2 trở lên, giá trị
      này phải kết thúc bằng số phiên bản chính, chẳng hạn <code>/v2</code>.</dd>
</dl>

### Ví dụ {#module-examples}

Các ví dụ sau thay thế `example.com` bằng một tên miền kho lưu trữ mà module có thể được tải về từ đó.

* Khai báo module cho module v0 hoặc v1:
  ```
  module example.com/mymodule
  ```
* Đường dẫn module cho module v2:
  ```
  module example.com/mymodule/v2
  ```

### Lưu ý {#module-notes}

Đường dẫn module phải định danh duy nhất cho module của bạn. Với hầu hết các module, đường dẫn là URL mà lệnh `go` có thể tìm thấy code (hoặc chuyển hướng đến code). Với các module sẽ không bao giờ được tải trực tiếp, đường dẫn module có thể chỉ là một tên nào đó mà bạn kiểm soát và đảm bảo tính duy nhất. Tiền tố `example/` cũng được dành riêng cho các ví dụ như thế này.

Để biết thêm chi tiết, xem [Quản lý dependency](/doc/modules/managing-dependencies#naming_module).

Trên thực tế, đường dẫn module thường là tên miền kho lưu trữ và đường dẫn đến code module trong kho lưu trữ đó. Lệnh `go` dựa vào dạng này khi tải các phiên bản module về để giải quyết dependency thay mặt người dùng module.

Dù bạn chưa có kế hoạch đưa module lên để người khác sử dụng ngay, thực hành tốt nhất vẫn là dùng đường dẫn kho lưu trữ, để tránh phải đổi tên module sau này nếu bạn quyết định xuất bản.

Nếu lúc đầu bạn chưa biết vị trí kho lưu trữ cuối cùng của module, hãy tạm thời dùng một giá trị thay thế an toàn, chẳng hạn tên một tên miền bạn sở hữu hoặc một tên bạn kiểm soát (như tên công ty), kết hợp với đường dẫn theo tên hoặc thư mục nguồn của module. Để biết thêm, xem [Quản lý dependency](/doc/modules/managing-dependencies#naming_module).

Ví dụ, nếu bạn đang phát triển trong thư mục `stringtools`, đường dẫn module tạm thời có thể là `<tên-công-ty>/stringtools`, như trong ví dụ sau, trong đó _tên-công-ty_ là tên công ty của bạn:

```
go mod init <tên-công-ty>/stringtools
```

## go {#go}

Cho biết module được viết theo ngữ nghĩa của phiên bản Go được chỉ định trong chỉ thị.

Để biết thêm, xem [chỉ thị `go`](/ref/mod#go-mod-file-go) trong Tài liệu tham chiếu Go Modules.

### Cú pháp {#go-syntax}

<pre>go <var>minimum-go-version</var></pre>

<dl>
    <dt>minimum-go-version</dt>
    <dd>Phiên bản Go tối thiểu cần thiết để biên dịch các package trong module này.</dd>
</dl>

### Ví dụ {#go-examples}

* Module phải chạy trên Go phiên bản 1.14 trở lên:
  ```
  go 1.14
  ```

### Lưu ý {#go-notes}

Chỉ thị `go` đặt phiên bản Go tối thiểu cần thiết để sử dụng module này. Trước Go 1.21, chỉ thị này chỉ mang tính tham khảo; từ Go 1.21, nó là yêu cầu bắt buộc: các toolchain Go từ chối sử dụng các module khai báo phiên bản Go mới hơn.

Chỉ thị `go` là một đầu vào để chọn toolchain Go cần chạy. Xem "[Go toolchains](/doc/toolchain)" để biết chi tiết.

Chỉ thị `go` ảnh hưởng đến việc sử dụng các tính năng ngôn ngữ mới:

* Với các package trong module, trình biên dịch từ chối các tính năng ngôn ngữ được giới thiệu sau phiên bản được chỉ định trong chỉ thị `go`. Ví dụ, nếu module có chỉ thị `go 1.12`, các package của nó không được dùng các literal số như `1_000_000` vốn được giới thiệu trong Go 1.13.
* Nếu một phiên bản Go cũ hơn biên dịch một trong các package của module và gặp lỗi biên dịch, lỗi đó sẽ ghi chú rằng module được viết cho phiên bản Go mới hơn. Ví dụ, giả sử module có `go 1.13` và một package dùng literal số `1_000_000`. Nếu package đó được biên dịch với Go 1.12, trình biên dịch ghi chú rằng code được viết cho Go 1.13.

Chỉ thị `go` cũng ảnh hưởng đến hành vi của lệnh `go`:

* Từ `go 1.14` trở lên, tính năng [vendoring](/ref/mod#vendoring) tự động có thể được bật. Nếu tệp `vendor/modules.txt` tồn tại và nhất quán với `go.mod`, không cần dùng cờ `-mod=vendor` một cách tường minh.
* Từ `go 1.16` trở lên, pattern package `all` chỉ khớp các package được import bắc cầu bởi các package và test trong [module chính](/ref/mod#glos-main-module). Đây là tập package giống với những gì [`go mod vendor`](/ref/mod#go-mod-vendor) giữ lại từ khi modules được giới thiệu. Ở các phiên bản thấp hơn, `all` còn bao gồm cả test của các package được import bởi các package trong module chính, test của các package đó, và tiếp tục như vậy.
* Từ `go 1.17` trở lên:
   * Tệp `go.mod` bao gồm chỉ thị [`require`](/ref/mod#go-mod-file-require) tường minh cho mỗi module cung cấp bất kỳ package nào được import bắc cầu bởi một package hoặc test trong module chính. (Ở `go 1.16` và thấp hơn, một dependency gián tiếp chỉ được đưa vào nếu [lựa chọn phiên bản tối thiểu](/ref/mod#minimal-version-selection) sẽ chọn một phiên bản khác.) Thông tin bổ sung này cho phép [cắt tỉa đồ thị module](/ref/mod#graph-pruning) và [tải module lười](/ref/mod#lazy-loading).
   * Do có thể có nhiều dependency `// indirect` hơn so với các phiên bản `go` trước, các dependency gián tiếp được ghi riêng trong một khối trong tệp `go.mod`.
   * `go mod vendor` bỏ qua các tệp `go.mod` và `go.sum` của các dependency được vendor. (Điều này cho phép các lệnh `go` được gọi trong các thư mục con của `vendor` xác định đúng module chính.)
   * `go mod vendor` ghi lại phiên bản `go` từ tệp `go.mod` của mỗi dependency vào `vendor/modules.txt`.
* Từ `go 1.21` trở lên:
   * Dòng `go` khai báo phiên bản Go tối thiểu bắt buộc để sử dụng module này.
   * Dòng `go` phải lớn hơn hoặc bằng dòng `go` của tất cả các dependency.
   * Lệnh `go` không còn cố gắng duy trì tính tương thích với phiên bản Go cũ hơn trước đó.
   * Lệnh `go` cẩn thận hơn trong việc giữ checksum của các tệp `go.mod` trong tệp `go.sum`.
<!-- Nếu bạn cập nhật danh sách này, hãy cập nhật cả /ref/mod#go-mod-file-go. -->

Một tệp `go.mod` chỉ có thể chứa tối đa một chỉ thị `go`. Hầu hết các lệnh sẽ thêm chỉ thị `go` với phiên bản Go hiện tại nếu chưa có.

## toolchain {#toolchain}

Khai báo một toolchain Go được khuyến nghị để sử dụng với module này. Chỉ có hiệu lực khi module là module chính và toolchain mặc định cũ hơn toolchain được khuyến nghị.

Để biết thêm, xem "[Go toolchains](/doc/toolchain)" và [chỉ thị `toolchain`](/ref/mod/#go-mod-file-toolchain) trong Tài liệu tham chiếu Go Modules.

### Cú pháp {#toolchain-syntax}

<pre>toolchain <var>toolchain-name</var></pre>

<dl>
    <dt>toolchain-name</dt>
    <dd>Tên của toolchain Go được khuyến nghị. Tên toolchain chuẩn có dạng
      <code>go<i>V</i></code> cho phiên bản Go <i>V</i>, ví dụ
      <code>go1.21.0</code> và <code>go1.18rc1</code>.
      Giá trị đặc biệt <code>default</code> vô hiệu hóa tính năng tự động chuyển toolchain.</dd>
</dl>

### Ví dụ {#toolchain-examples}

* Khuyến nghị sử dụng Go 1.21.0 trở lên:
    ```
    toolchain go1.21.0
    ```

### Lưu ý {#toolchain-notes}

Xem "[Go toolchains](/doc/toolchain)" để biết chi tiết về cách dòng `toolchain` ảnh hưởng đến việc chọn toolchain Go.

## godebug {#godebug}

Chỉ định các cài đặt [GODEBUG](/doc/godebug) mặc định được áp dụng cho các package main của module này. Các cài đặt này ghi đè mọi giá trị mặc định của toolchain và bị ghi đè bởi các dòng `//go:debug` tường minh trong các package main.

### Cú pháp {#godebug-syntax}

<pre>godebug <var>debug-key</var>=<var>debug-value</var></pre>

<dl>
    <dt>debug-key</dt>
    <dd>Tên của cài đặt cần áp dụng. Danh sách các cài đặt và phiên bản chúng được giới thiệu có thể được tìm thấy tại
      <a href="/doc/godebug#history">Lịch sử GODEBUG</a>.
    </dd>
    <dt>debug-value</dt>
    <dd>Giá trị cung cấp cho cài đặt. Nếu không được chỉ định rõ, <code>0</code> để tắt và <code>1</code> để bật hành vi được đặt tên.</dd>
</dl>

### Ví dụ {#godebug-examples}

* Sử dụng hành vi mới `asynctimerchan=0` của phiên bản 1.23:
  ```
  godebug asynctimerchan=0
  ```
* Sử dụng các GODEBUG mặc định từ Go 1.21, nhưng giữ hành vi cũ `panicnil=1`:
  ```
  godebug (
      default=go1.21
      panicnil=1
  )
  ```

### Lưu ý {#godebug-notes}

Các cài đặt GODEBUG chỉ áp dụng cho các bản build package main và binary test trong module hiện tại. Chúng không có hiệu lực khi module được dùng là một dependency.

Xem "[Go, Tính tương thích ngược và GODEBUG](/doc/godebug)" để biết chi tiết về tính tương thích ngược.

## require {#require}

Khai báo một module là dependency của module hiện tại, chỉ định phiên bản tối thiểu của module được yêu cầu.

Để biết thêm, xem [chỉ thị `require`](/ref/mod#go-mod-file-require) trong Tài liệu tham chiếu Go Modules.

### Cú pháp {#require-syntax}

<pre>require <var>module-path</var> <var>module-version</var></pre>

<dl>
    <dt>module-path</dt>
    <dd>Đường dẫn module của module, thường là ghép nối giữa tên miền kho lưu trữ nguồn và tên module. Với các phiên bản module v2 trở lên, giá trị này phải kết thúc bằng số phiên bản chính, chẳng hạn <code>/v2</code>.</dd>
    <dt>module-version</dt>
    <dd>Phiên bản của module. Có thể là số phiên bản phát hành, chẳng hạn v1.2.3, hoặc số pseudo-version do Go tạo ra, chẳng hạn v0.0.0-20200921210052-fa0125251cc4.</dd>
</dl>

### Ví dụ {#require-examples}

* Yêu cầu một phiên bản phát hành v1.2.3:
    ```
    require example.com/othermodule v1.2.3
    ```
* Yêu cầu một phiên bản chưa được gắn tag trong kho lưu trữ bằng cách dùng số pseudo-version do công cụ Go tạo ra:
    ```
    require example.com/othermodule v0.0.0-20200921210052-fa0125251cc4
    ```

### Lưu ý {#require-notes}

Khi bạn chạy một lệnh `go` như `go get`, Go thêm các chỉ thị `require` cho mỗi module chứa các package được import. Khi một module chưa được gắn tag trong kho lưu trữ, Go gán cho nó một số pseudo-version được tạo khi bạn chạy lệnh.

Bạn có thể yêu cầu Go sử dụng module từ một vị trí khác thay vì kho lưu trữ của nó bằng cách dùng [chỉ thị `replace`](#replace).

Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

Để biết thêm về quản lý dependency, xem:

* [Thêm dependency](/doc/modules/managing-dependencies#adding_dependency)
* [Lấy phiên bản dependency cụ thể](/doc/modules/managing-dependencies#getting_version)
* [Khám phá các bản cập nhật hiện có](/doc/modules/managing-dependencies#discovering_updates)
* [Nâng cấp hoặc hạ cấp dependency](/doc/modules/managing-dependencies#upgrading)
* [Đồng bộ hóa các dependency trong code](/doc/modules/managing-dependencies#synchronizing)

## tool {#tool}

Thêm một package vào danh sách dependency của module hiện tại và cho phép chạy nó bằng `go tool` khi thư mục làm việc hiện tại nằm trong module này.

### Cú pháp {#tool-syntax}

<pre>tool <var>package-path</var></pre>

<dl>
    <dt>package-path</dt>
    <dd>Đường dẫn package của tool, là sự kết hợp giữa module chứa tool và đường dẫn (có thể rỗng) đến package cài đặt tool trong module đó.</dd>
</dl>

### Ví dụ {#tool-examples}

* Khai báo một tool được cài đặt trong module hiện tại:
    ```
    module example.com/mymodule

    tool example.com/mymodule/cmd/mytool
    ```
* Khai báo một tool được cài đặt trong module riêng biệt:
    ```
    module example.com/mymodule

    tool example.com/atool/cmd/atool

    require example.com/atool v1.2.3
    ```

### Lưu ý {#tool-notes}

Bạn có thể dùng `go tool` để chạy các tool được khai báo trong module bằng đường dẫn package đầy đủ hoặc, nếu không có sự mơ hồ, bằng phần cuối cùng của đường dẫn. Trong ví dụ đầu tiên ở trên, bạn có thể chạy `go tool mytool` hoặc `go tool example.com/mymodule/cmd/mytool`.

Trong workspace mode, bạn có thể dùng `go tool` để chạy một tool được khai báo trong bất kỳ module nào trong workspace.

Các tool được build bằng cùng đồ thị module với bản thân module đó. Một [chỉ thị `require`](#require) là cần thiết để chọn phiên bản của module cài đặt tool. Bất kỳ [chỉ thị `replace`](#replace) hay [chỉ thị `exclude`](#exclude) nào cũng áp dụng cho tool và các dependency của nó.

Để biết thêm thông tin, xem [Tool dependencies](/doc/modules/managing-dependencies#tools).

## replace {#replace}

Thay thế nội dung của một module ở một phiên bản cụ thể (hoặc tất cả các phiên bản) bằng một phiên bản module khác hoặc một thư mục cục bộ. Công cụ Go sẽ sử dụng đường dẫn thay thế khi giải quyết dependency.

Để biết thêm, xem [chỉ thị `replace`](/ref/mod#go-mod-file-replace) trong Tài liệu tham chiếu Go Modules.

### Cú pháp {#replace-syntax}

<pre>replace <var>module-path</var> <var>[module-version]</var> => <var>replacement-path</var> <var>[replacement-version]</var></pre>

<dl>
    <dt>module-path</dt>
    <dd>Đường dẫn module của module cần thay thế.</dd>
    <dt>module-version</dt>
    <dd>Tùy chọn. Một phiên bản cụ thể cần thay thế. Nếu số phiên bản này bị bỏ qua, tất cả các phiên bản của module sẽ được thay thế bằng nội dung ở phía bên phải mũi tên.</dd>
    <dt>replacement-path</dt>
    <dd>Đường dẫn mà Go sẽ tìm module được yêu cầu. Có thể là đường dẫn module hoặc đường dẫn đến một thư mục trên hệ thống tệp cục bộ của module thay thế. Nếu đây là đường dẫn module, bạn phải chỉ định giá trị <em>replacement-version</em>. Nếu đây là đường dẫn cục bộ, bạn không được dùng giá trị <em>replacement-version</em>.</dd>
    <dt>replacement-version</dt>
    <dd>Phiên bản của module thay thế. Phiên bản thay thế chỉ được chỉ định nếu <em>replacement-path</em> là đường dẫn module (không phải thư mục cục bộ).</dd>
</dl>

### Ví dụ {#replace-examples}

* Thay thế bằng một fork của kho lưu trữ module

  Trong ví dụ sau, bất kỳ phiên bản nào của example.com/othermodule đều được thay thế bằng fork đã chỉ định.

  ```
  require example.com/othermodule v1.2.3

  replace example.com/othermodule => example.com/myfork/othermodule v1.2.3-fixed
  ```

  Khi bạn thay thế một đường dẫn module bằng đường dẫn khác, đừng thay đổi các câu lệnh import cho các package trong module đang được thay thế.

  Để biết thêm về việc dùng bản fork của mã nguồn module, xem [Yêu cầu mã nguồn module bên ngoài từ fork kho lưu trữ của bạn](/doc/modules/managing-dependencies#external_fork).

* Thay thế bằng một số phiên bản khác

  Ví dụ sau chỉ định rằng phiên bản v1.2.3 sẽ được dùng thay cho bất kỳ phiên bản nào khác của module.

  ```
  require example.com/othermodule v1.2.2

  replace example.com/othermodule => example.com/othermodule v1.2.3
  ```

  Ví dụ sau thay thế phiên bản module v1.2.5 bằng phiên bản v1.2.3 của cùng module đó.

  ```
  replace example.com/othermodule v1.2.5 => example.com/othermodule v1.2.3
  ```

* Thay thế bằng code cục bộ

  Ví dụ sau chỉ định rằng một thư mục cục bộ sẽ được dùng thay thế cho tất cả các phiên bản của module.

  ```
  require example.com/othermodule v1.2.3

  replace example.com/othermodule => ../othermodule
  ```

  Ví dụ sau chỉ định rằng một thư mục cục bộ sẽ được dùng thay thế chỉ cho v1.2.5.

  ```
  require example.com/othermodule v1.2.5

  replace example.com/othermodule v1.2.5 => ../othermodule
  ```

  Để biết thêm về việc dùng bản sao cục bộ của mã nguồn module, xem [Yêu cầu mã nguồn module từ thư mục cục bộ](/doc/modules/managing-dependencies#local_directory).

### Lưu ý {#replace-notes}

Dùng chỉ thị `replace` để tạm thời thay thế một giá trị đường dẫn module bằng một giá trị khác khi bạn muốn Go dùng đường dẫn đó để tìm mã nguồn module. Điều này có tác dụng chuyển hướng tìm kiếm module của Go đến vị trí thay thế. Bạn không cần thay đổi các đường dẫn import package để dùng đường dẫn thay thế.

Dùng các chỉ thị `exclude` và `replace` để kiểm soát việc giải quyết dependency tại thời điểm build khi build module hiện tại. Các chỉ thị này bị bỏ qua trong các module phụ thuộc vào module hiện tại.

Chỉ thị `replace` có thể hữu ích trong các tình huống như sau:

* Bạn đang phát triển một module mới mà code chưa có trong kho lưu trữ. Bạn muốn kiểm thử với các client dùng phiên bản cục bộ.
* Bạn đã phát hiện ra một vấn đề với một dependency, đã clone kho lưu trữ của dependency đó, và đang kiểm thử một bản sửa lỗi với kho lưu trữ cục bộ.

Lưu ý rằng một chỉ thị `replace` đơn độc không thêm module vào [đồ thị module](/ref/mod#glos-module-graph). Cũng cần có một [chỉ thị `require`](#require) tham chiếu đến phiên bản module được thay thế, trong tệp `go.mod` của module chính hoặc tệp `go.mod` của dependency. Nếu bạn không có phiên bản cụ thể để thay thế, bạn có thể dùng một phiên bản giả, như trong ví dụ dưới đây. Lưu ý rằng điều này sẽ làm hỏng các module phụ thuộc vào module của bạn, vì các chỉ thị `replace` chỉ được áp dụng trong module chính.

```
require example.com/mod v0.0.0-replace

replace example.com/mod v0.0.0-replace => ./mod
```

Để biết thêm về việc thay thế một module bắt buộc, bao gồm cách dùng công cụ Go để thực hiện thay đổi, xem:

* [Yêu cầu mã nguồn module bên ngoài từ fork kho lưu trữ của bạn](/doc/modules/managing-dependencies#external_fork)
* [Yêu cầu mã nguồn module từ thư mục cục bộ](/doc/modules/managing-dependencies#local_directory)

Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

## exclude {#exclude}

Chỉ định một module hoặc phiên bản module cần loại trừ khỏi đồ thị dependency của module hiện tại.

Để biết thêm, xem [chỉ thị `exclude`](/ref/mod#go-mod-file-exclude) trong Tài liệu tham chiếu Go Modules.

### Cú pháp {#exclude-syntax}

<pre>exclude <var>module-path</var> <var>module-version</var></pre>

<dl>
    <dt>module-path</dt>
    <dd>Đường dẫn module của module cần loại trừ.</dd>
    <dt>module-version</dt>
    <dd>Phiên bản cụ thể cần loại trừ.</dd>
</dl>

### Ví dụ {#exclude-example}

* Loại trừ example.com/theirmodule phiên bản v1.3.0

  ```
  exclude example.com/theirmodule v1.3.0
  ```

### Lưu ý {#exclude-notes}

Dùng chỉ thị `exclude` để loại trừ một phiên bản cụ thể của một module được yêu cầu gián tiếp nhưng không thể tải vì một lý do nào đó. Ví dụ, bạn có thể dùng nó để loại trừ một phiên bản module có checksum không hợp lệ.

Dùng các chỉ thị `exclude` và `replace` để kiểm soát việc giải quyết dependency tại thời điểm build khi build module hiện tại (module chính mà bạn đang build). Các chỉ thị này bị bỏ qua trong các module phụ thuộc vào module hiện tại.

Bạn có thể dùng lệnh [`go mod edit`](/ref/mod#go-mod-edit) để loại trừ một module, như trong ví dụ sau.

```
go mod edit -exclude=example.com/theirmodule@v1.3.0
```

Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

## retract {#retract}

Chỉ định rằng một phiên bản hoặc một dải phiên bản của module được định nghĩa bởi `go.mod` không nên được phụ thuộc vào. Chỉ thị `retract` hữu ích khi một phiên bản được xuất bản sớm hoặc khi phát hiện ra vấn đề nghiêm trọng sau khi phiên bản đã được xuất bản.

Để biết thêm, xem [chỉ thị `retract`](/ref/mod#go-mod-file-retract) trong Tài liệu tham chiếu Go Modules.

### Cú pháp {#retract-syntax}

<pre>
retract <var>version</var> // <var>rationale</var>
retract [<var>version-low</var>,<var>version-high</var>] // <var>rationale</var>
</pre>

<dl>
  <dt>version</dt>
  <dd>Một phiên bản duy nhất cần thu hồi.</dd>
  <dt>version-low</dt>
  <dd>Giới hạn dưới của một dải phiên bản cần thu hồi.</dd>
  <dt>version-high</dt>
  <dd>
    Giới hạn trên của một dải phiên bản cần thu hồi. Cả <var>version-low</var>
    và <var>version-high</var> đều được bao gồm trong dải.
  </dd>
  <dt>rationale</dt>
  <dd>
    Chú thích tùy chọn giải thích lý do thu hồi. Có thể được hiển thị trong các thông báo cho người dùng.
  </dd>
</dl>

### Ví dụ {#retract-example}

* Thu hồi một phiên bản duy nhất

  ```
  retract v1.1.0 // Published accidentally.
  ```

* Thu hồi một dải phiên bản

  ```
  retract [v1.0.0,v1.0.5] // Build broken on some platforms.
  ```

### Lưu ý {#retract-notes}

Dùng chỉ thị `retract` để chỉ ra rằng phiên bản trước đó của module không nên được sử dụng. Người dùng sẽ không tự động nâng cấp lên phiên bản bị thu hồi khi dùng `go get`, `go mod tidy` hoặc các lệnh khác. Người dùng sẽ không thấy phiên bản bị thu hồi như một bản cập nhật hiện có khi dùng `go list -m -u`.

Các phiên bản bị thu hồi vẫn nên tiếp tục có sẵn để người dùng đang phụ thuộc vào chúng có thể build được các package của họ. Dù một phiên bản bị thu hồi có thể bị xóa khỏi kho lưu trữ nguồn, nó vẫn có thể tiếp tục tồn tại trên các mirror như [proxy.golang.org](https://proxy.golang.org). Người dùng đang phụ thuộc vào các phiên bản bị thu hồi có thể nhận được thông báo khi họ chạy `go get` hoặc `go list -m -u` trên các module liên quan.

Lệnh `go` phát hiện các phiên bản bị thu hồi bằng cách đọc các chỉ thị `retract` trong tệp `go.mod` của phiên bản mới nhất của module. Phiên bản mới nhất được xác định theo thứ tự ưu tiên:

1. Phiên bản phát hành cao nhất của nó, nếu có
2. Phiên bản pre-release cao nhất của nó, nếu có
3. Một pseudo-version cho đỉnh của nhánh mặc định của kho lưu trữ.

Khi bạn thêm một lần thu hồi, bạn hầu như luôn cần gắn tag một phiên bản mới, cao hơn để lệnh có thể thấy nó trong phiên bản mới nhất của module.

Bạn có thể xuất bản một phiên bản mà mục đích duy nhất là báo hiệu việc thu hồi. Trong trường hợp này, phiên bản mới cũng có thể tự thu hồi chính nó.

Ví dụ, nếu bạn vô tình gắn tag `v1.0.0`, bạn có thể gắn tag `v1.0.1` với các chỉ thị sau:

```
retract v1.0.0 // Published accidentally.
retract v1.0.1 // Contains retraction only.
```

Tiếc là, khi một phiên bản đã được xuất bản, nó không thể thay đổi được. Nếu bạn sau đó gắn tag `v1.0.0` ở một commit khác, lệnh `go` có thể phát hiện ra checksum không khớp trong `go.sum` hoặc trong [cơ sở dữ liệu checksum](/ref/mod#checksum-database).

Các phiên bản bị thu hồi của module thường không xuất hiện trong kết quả của `go list -m -versions`, nhưng bạn có thể dùng `-retracted` để hiển thị chúng. Để biết thêm, xem [`go list -m`](/ref/mod#go-list-m) trong Tài liệu tham chiếu Go Modules.
