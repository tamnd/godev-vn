---
Title: Tham chiếu tệp go.mod
---

Mỗi mô-đun Go được xác định bởi một tệp go.mod mô tả các thuộc tính của mô-đun, bao gồm các dependency của mô-đun đó với các mô-đun khác và với các phiên bản Go.

Các thuộc tính này bao gồm:

* **đường dẫn mô-đun** của mô-đun hiện tại. Đây phải là một vị trí mà từ đó các công cụ Go có thể tải xuống mô-đun, chẳng hạn như vị trí kho lưu trữ của mã mô-đun. Vị trí này đóng vai trò là một mã định danh duy nhất khi được kết hợp với số phiên bản của mô-đun. Đây cũng là tiền tố của đường dẫn gói cho tất cả các gói trong mô-đun. Để biết thêm về cách Go định vị mô-đun, xem <a href="/ref/mod#vcs-find">Tài liệu tham khảo về Go Modules</a>.
* **phiên bản Go** tối thiểu được mô-đun hiện tại yêu cầu.
* Danh sách các phiên bản tối thiểu của các **mô-đun khác được yêu cầu** bởi mô-đun hiện tại.
* Các hướng dẫn, nếu có, để **thay thế** một mô-đun được yêu cầu bằng một phiên bản mô-đun khác hoặc một thư mục cục bộ, để **loại trừ** một phiên bản cụ thể của một mô-đun được yêu cầu, hoặc để **bỏ qua** các thư mục cụ thể trong mô-đun khi khớp các mẫu gói.

Go tạo tệp go.mod khi bạn chạy [lệnh `go mod init`](/ref/mod#go-mod-init). Ví dụ sau tạo một tệp go.mod, đặt đường dẫn mô-đun của mô-đun thành example/mymodule:

```
$ go mod init example/mymodule
```

Sử dụng các lệnh `go` để quản lý các dependency. Các lệnh này đảm bảo rằng các yêu cầu được mô tả trong tệp go.mod của bạn vẫn nhất quán và nội dung của tệp go.mod vẫn hợp lệ. Các lệnh này bao gồm các lệnh [`go get`](/ref/mod#go-get), [`go mod tidy`](/ref/mod#go-mod-tidy) và [`go mod edit`](/ref/mod#go-mod-edit).

Để tham khảo về các lệnh `go`, xem [Command go](/cmd/go/).
Bạn có thể nhận trợ giúp từ dòng lệnh bằng cách nhập `go help` _command-name_, như với `go help mod tidy`.

**Xem thêm**

* Các công cụ Go thực hiện thay đổi đối với tệp go.mod của bạn khi bạn sử dụng chúng để quản lý dependency. Để biết thêm, xem [Quản lý dependency](/doc/modules/managing-dependencies).
* Để biết thêm chi tiết và các ràng buộc liên quan đến tệp go.mod, xem [tài liệu tham khảo về Go modules](/ref/mod#go-mod-file).

## Ví dụ {#example}

Tệp go.mod bao gồm các chỉ thị như trong ví dụ sau. Các chỉ thị này được mô tả ở phần khác trong chủ đề này.

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

Khai báo đường dẫn module của module, là mã định danh duy nhất của module
(khi kết hợp với số phiên bản module). Đường dẫn module trở thành tiền tố
import cho tất cả các gói mà module chứa.

Để biết thêm, xem chỉ thị [`module`](/ref/mod#go-mod-file-module) trong
Tham chiếu Go Modules.

### Cú pháp {#module-syntax}

<pre>module <var>module-path</var></pre>

<dl>
    <dt>module-path</dt>
    <dd>Đường dẫn module của module, thường là vị trí kho lưu trữ từ đó
      module có thể được tải xuống bằng các công cụ Go. Đối với các phiên bản
      module v2 trở lên, giá trị này phải kết thúc bằng số phiên bản chính,
      chẳng hạn như <code>/v2</code>.</dd>
</dl>

### Ví dụ {#module-examples}

Các ví dụ sau thay thế `example.com` cho miền repository mà từ đó module
có thể được tải xuống.

* Khai báo module cho module v0 hoặc v1:
  ```
  module example.com/mymodule
  ```
* Đường dẫn module cho module v2:
  ```
  module example.com/mymodule/v2
  ```

### Lưu ý {#module-notes}

Đường dẫn module phải nhận diện duy nhất module của bạn. Với hầu hết module,
đường dẫn này là một URL nơi lệnh `go` có thể tìm thấy mã nguồn (hoặc một
chuyển hướng đến mã nguồn). Với các module sẽ không bao giờ được tải xuống
trực tiếp, đường dẫn module có thể chỉ là một tên do bạn kiểm soát để đảm bảo
tính duy nhất. Tiền tố `example/` cũng được dành riêng để sử dụng trong các
ví dụ như thế này.

Để biết thêm chi tiết, xem [Quản lý dependency](/doc/modules/managing-dependencies#naming_module).

Trong thực tế, đường dẫn module thường là miền repository của mã nguồn module
và đường dẫn đến mã module bên trong repository. Lệnh `go` dựa vào dạng này
khi tải xuống các phiên bản module để thay mặt người dùng module giải quyết
các dependency.

Ngay cả khi ban đầu bạn không định cung cấp module của mình để mã nguồn khác
sử dụng, việc sử dụng đường dẫn repository của module là một thực hành tốt,
giúp bạn tránh phải đổi tên module nếu sau này xuất bản nó.

Nếu ban đầu bạn chưa biết vị trí repository cuối cùng của module, hãy cân nhắc
sử dụng tạm thời một giá trị thay thế an toàn, chẳng hạn như tên của một miền
mà bạn sở hữu hoặc một tên do bạn kiểm soát (chẳng hạn tên công ty của bạn),
cùng với đường dẫn bắt nguồn từ tên module hoặc thư mục mã nguồn. Để biết thêm,
xem [Quản lý dependency](/doc/modules/managing-dependencies#naming_module).

Ví dụ: nếu bạn đang phát triển trong thư mục `stringtools`, đường dẫn module
tạm thời của bạn có thể là `<company-name>/stringtools`, như trong ví dụ sau,
trong đó _company-name_ là tên công ty của bạn:

```
go mod init <company-name>/stringtools
```

## go {#go}

Cho biết module được viết với giả định về ngữ nghĩa của phiên bản Go được chỉ định bởi chỉ thị này.

Để biết thêm, xem [chỉ thị `go`](/ref/mod#go-mod-file-go) trong Tài liệu tham khảo Go Modules.

### Syntax {#go-syntax}

<pre>go <var>minimum-go-version</var></pre>

<dl>
    <dt>minimum-go-version</dt>
    <dd>Phiên bản Go tối thiểu cần thiết để biên dịch các gói trong module này.</dd>
</dl>

### Examples {#go-examples}

* Module phải chạy trên phiên bản Go 1.14 trở lên:
  ```
  go 1.14
  ```

### Notes {#go-notes}

Chỉ thị `go` đặt phiên bản Go tối thiểu cần thiết để sử dụng module này.
Trước Go 1.21, chỉ thị này chỉ mang tính khuyến nghị; hiện tại đây là một yêu cầu bắt buộc:
các bộ công cụ Go từ chối sử dụng các module khai báo phiên bản Go mới hơn.

Chỉ thị `go` là đầu vào để lựa chọn bộ công cụ Go sẽ chạy.
Xem “[các bộ công cụ Go](/doc/toolchain)” để biết chi tiết.

Chỉ thị `go` ảnh hưởng đến việc sử dụng các tính năng ngôn ngữ mới:

* Đối với các gói trong module, trình biên dịch từ chối việc sử dụng các tính năng ngôn ngữ
  được giới thiệu sau phiên bản được chỉ định bởi chỉ thị `go`. Ví dụ, nếu
  một module có chỉ thị `go 1.12`, các gói của nó không được sử dụng các
  literal số như `1_000_000`, vốn được giới thiệu trong Go 1.13.
* Nếu một phiên bản Go cũ hơn xây dựng một trong các gói của module và gặp
  lỗi biên dịch, lỗi đó sẽ ghi chú rằng module được viết cho một phiên bản Go
  mới hơn. Ví dụ, giả sử một module có `go 1.13` và một gói sử dụng
  literal số `1_000_000`. Nếu gói đó được xây dựng bằng Go 1.12, trình
  biên dịch ghi chú rằng mã được viết cho Go 1.13.

Chỉ thị `go` cũng ảnh hưởng đến hành vi của lệnh `go`:

* Ở `go 1.14` trở lên, [vendoring](/ref/mod#vendoring) tự động có thể được
  bật. Nếu tệp `vendor/modules.txt` tồn tại và nhất quán với
  `go.mod`, không cần sử dụng rõ ràng cờ `-mod=vendor`.
* Ở `go 1.16` trở lên, mẫu gói `all` chỉ khớp với các gói
  được import bắc cầu bởi các gói và bài kiểm thử trong [module
  chính](/ref/mod#glos-main-module). Đây là cùng tập các gói được giữ lại
  bởi [`go mod vendor`](/ref/mod#go-mod-vendor) kể từ khi module được giới thiệu. Trong
  các phiên bản thấp hơn, `all` cũng bao gồm các bài kiểm thử của các gói được
  import bởi các gói trong module chính, các bài kiểm thử của những gói đó, và cứ tiếp tục như vậy.
* Ở `go 1.17` trở lên:
  * Tệp `go.mod` bao gồm một [chỉ thị
    `require`](/ref/mod#go-mod-file-require) rõ ràng cho mỗi module cung cấp bất kỳ
    gói nào được import bắc cầu bởi một gói hoặc bài kiểm thử trong module chính. (Ở
    `go 1.16` trở xuống, một dependency gián tiếp chỉ được đưa vào nếu [lựa chọn
    phiên bản tối thiểu](/ref/mod#minimal-version-selection) nếu không sẽ
    chọn một phiên bản khác.) Thông tin bổ sung này cho phép [loại bỏ
    đồ thị module](/ref/mod#graph-pruning) và [tải module
    lười](/ref/mod#lazy-loading).
  * Vì có thể có nhiều dependency `// indirect` hơn đáng kể so với các
    phiên bản `go` trước đây, các dependency gián tiếp được ghi lại trong một khối
    riêng trong tệp `go.mod`.
  * `go mod vendor` bỏ qua các tệp `go.mod` và `go.sum` của các dependency được
    vendoring. (Điều đó cho phép các lần gọi lệnh `go` trong các
    thư mục con của `vendor` xác định module chính chính xác.)
  * `go mod vendor` ghi lại phiên bản `go` từ tệp `go.mod` của mỗi dependency
    trong `vendor/modules.txt`.
* Ở `go 1.21` trở lên:
  * Dòng `go` khai báo phiên bản Go tối thiểu bắt buộc để sử dụng với module này.
  * Dòng `go` phải lớn hơn hoặc bằng dòng `go` của tất cả dependency.
  * Lệnh `go` không còn cố gắng duy trì khả năng tương thích với phiên bản Go cũ hơn trước đó.
  * Lệnh `go` cẩn thận hơn trong việc giữ các checksum của tệp `go.mod` trong tệp `go.sum`.
<!-- Nếu bạn cập nhật danh sách này, cũng hãy cập nhật /ref/mod#go-mod-file-go. -->

Một tệp `go.mod` có thể chứa nhiều nhất một chỉ thị `go`. Hầu hết các lệnh sẽ thêm một
chỉ thị `go` với phiên bản Go hiện tại nếu chưa có.

## toolchain {#toolchain}

Khai báo một Go toolchain được đề xuất để sử dụng với module này.
Chỉ có hiệu lực khi module là module chính
và toolchain mặc định cũ hơn toolchain được đề xuất.

Để biết thêm, xem “[Go toolchains](/doc/toolchain)” và
chỉ thị [`toolchain`](/ref/mod/#go-mod-file-toolchain) trong
Tài liệu tham khảo Go Modules.

### Syntax {#toolchain-syntax}

<pre>toolchain <var>toolchain-name</var></pre>

<dl>
    <dt>toolchain-name</dt>
    <dd>Tên của Go toolchain được đề xuất. Tên toolchain tiêu chuẩn có dạng
      <code>go<i>V</i></code> cho một phiên bản Go <i>V</i>, như
      <code>go1.21.0</code> và <code>go1.18rc1</code>.
      Giá trị đặc biệt <code>default</code> vô hiệu hóa việc chuyển đổi toolchain tự động.</dd>
</dl>

### Examples {#toolchain-examples}

* Đề xuất sử dụng Go 1.21.0 hoặc mới hơn:
    ```
    toolchain go1.21.0
    ```

### Notes {#toolchain-notes}

Xem “[Go toolchains](/doc/toolchain)” để biết chi tiết về cách dòng `toolchain`
ảnh hưởng đến việc lựa chọn Go toolchain.

## godebug {#godebug}

Chỉ ra các thiết lập [GODEBUG](/doc/godebug) mặc định được áp dụng cho các gói chính của module này.
Các thiết lập này ghi đè mọi giá trị mặc định của toolchain và bị ghi đè bởi các dòng `//go:debug` rõ ràng trong các gói chính.

### Syntax {#godebug-syntax}

<pre>godebug <var>debug-key</var>=<var>debug-value</var></pre>

<dl>
    <dt>debug-key</dt>
    <dd>Tên của thiết lập được áp dụng.
      Danh sách các thiết lập và các phiên bản mà chúng được giới thiệu có thể được tìm thấy tại
      <a href="/doc/godebug#history">Lịch sử GODEBUG</a>.
    </dd>
    <dt>debug-value</dt>
    <dd>Giá trị được cung cấp cho thiết lập.
      Nếu không được chỉ định khác, <code>0</code> để vô hiệu hóa và <code>1</code> để bật hành vi được đặt tên.</dd>
</dl>

### Examples {#godebug-examples}

* Sử dụng hành vi `asynctimerchan=0` mới trong 1.23:
  ```
  godebug asynctimerchan=0
  ```
* Sử dụng GODEBUG mặc định từ Go 1.21, nhưng sử dụng hành vi `panicnil=1` cũ:
  ```
  godebug (
      default=go1.21
      panicnil=1
  )
  ```

### Notes {#godebug-notes}

Các thiết lập GODEBUG chỉ áp dụng cho các bản dựng của gói chính và các binary kiểm thử trong module hiện tại.
Chúng không có tác dụng khi một module được sử dụng làm dependency.

Xem “[Go, Backwards Compatibility, and GODEBUG](/doc/godebug)” để biết chi tiết về khả năng tương thích ngược.

## require {#require}

Khai báo một module là dependency của module hiện tại, chỉ định phiên bản tối thiểu
của module được yêu cầu.

Để biết thêm, xem [chỉ thị `require`](/ref/mod#go-mod-file-require) trong
Tài liệu tham chiếu Go Modules.

### Syntax {#require-syntax}

<pre>require <var>module-path</var> <var>module-version</var></pre>

<dl>
    <dt>module-path</dt>
    <dd>Đường dẫn module của module, thường là sự kết hợp giữa miền của kho lưu trữ
      nguồn module và tên module. Đối với các phiên bản module v2 trở lên,
      giá trị này phải kết thúc bằng số phiên bản chính, chẳng hạn như <code>/v2</code>.</dd>
    <dt>module-version</dt>
    <dd>Phiên bản của module. Giá trị này có thể là số phiên bản bản phát hành,
      chẳng hạn như v1.2.3, hoặc số phiên bản giả do Go tạo, chẳng hạn như
      v0.0.0-20200921210052-fa0125251cc4.</dd>
</dl>

### Examples {#require-examples}

* Yêu cầu một phiên bản đã phát hành v1.2.3:
    ```
    require example.com/othermodule v1.2.3
    ```
* Yêu cầu một phiên bản chưa được gắn thẻ trong kho lưu trữ của nó bằng cách sử dụng số
  phiên bản giả do các công cụ Go tạo:
    ```
    require example.com/othermodule v0.0.0-20200921210052-fa0125251cc4
    ```

### Notes {#require-notes}

Khi bạn chạy một lệnh `go` như `go get`, Go chèn các chỉ thị `require`
cho mỗi module chứa các gói đã nhập. Khi một module chưa được gắn thẻ trong
kho lưu trữ của nó, Go gán một số phiên bản giả được tạo khi bạn chạy
lệnh đó.

Bạn có thể yêu cầu Go sử dụng một module từ vị trí khác với kho lưu trữ của nó bằng
cách sử dụng [chỉ thị `replace`](#replace).

Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

Để biết thêm về quản lý dependency, xem các nội dung sau:

* [Thêm một dependency](/doc/modules/managing-dependencies#adding_dependency)
* [Lấy một phiên bản dependency cụ thể](/doc/modules/managing-dependencies#getting_version)
* [Khám phá các bản cập nhật có sẵn](/doc/modules/managing-dependencies#discovering_updates)
* [Nâng cấp hoặc hạ cấp một dependency](/doc/modules/managing-dependencies#upgrading)
* [Đồng bộ hóa dependency của mã nguồn](/doc/modules/managing-dependencies#synchronizing)

## tool {#tool}

Thêm một gói làm dependency của module hiện tại và cho phép chạy gói đó bằng `go tool` khi thư mục làm việc hiện tại nằm trong module này.

### Cú pháp {#tool-syntax}

<pre>tool <var>package-path</var></pre>

<dl>
    <dt>package-path</dt>
    <dd>Đường dẫn gói của công cụ, là sự kết hợp giữa module chứa
        công cụ và đường dẫn (có thể rỗng) đến gói triển khai
        công cụ trong module đó.</dd>
</dl>

### Ví dụ {#tool-examples}

* Khai báo một công cụ được triển khai trong module hiện tại:
    ```
    module example.com/mymodule

    tool example.com/mymodule/cmd/mytool
    ```
* Khai báo một công cụ được triển khai trong một module riêng:
    ```
    module example.com/mymodule

    tool example.com/atool/cmd/atool

    require example.com/atool v1.2.3
    ```

### Ghi chú {#tool-notes}

Bạn có thể sử dụng `go tool` để chạy các công cụ được khai báo trong module của mình bằng đường dẫn gói đầy đủ hoặc, nếu không có sự mơ hồ, bằng phần cuối cùng của đường dẫn. Trong ví dụ đầu tiên ở trên, bạn có thể chạy `go tool mytool` hoặc `go tool example.com/mymodule/cmd/mytool`.

Trong chế độ workspace, bạn có thể sử dụng `go tool` để chạy một công cụ được khai báo trong bất kỳ module workspace nào.

Các công cụ được xây dựng bằng cùng đồ thị module với chính module đó. Cần có [chỉ thị `require`](#require) để chọn phiên bản của module triển khai công cụ. Bất kỳ [chỉ thị `replace`](#replace) hoặc [chỉ thị `exclude`](#exclude) nào cũng áp dụng cho công cụ và các dependency của nó.

Để biết thêm thông tin, xem [Dependency của công cụ](/doc/modules/managing-dependencies#tools).

## replace {#replace}

Thay thế nội dung của một module tại một phiên bản cụ thể (hoặc tất cả phiên bản) bằng một phiên bản module khác hoặc bằng một thư mục cục bộ. Các công cụ Go sẽ sử dụng đường dẫn thay thế khi phân giải dependency.

Để biết thêm, xem [chỉ thị `replace`](/ref/mod#go-mod-file-replace) trong Tài liệu tham khảo Go Modules.

### Cú pháp {#replace-syntax}

<pre>replace <var>module-path</var> <var>[module-version]</var> => <var>replacement-path</var> <var>[replacement-version]</var></pre>

<dl>
    <dt>module-path</dt>
    <dd>Đường dẫn module của module cần thay thế.</dd>
    <dt>module-version</dt>
    <dd>Tùy chọn. Một phiên bản cụ thể cần thay thế. Nếu số phiên bản này
      bị bỏ qua, tất cả phiên bản của module được thay thế bằng nội dung ở
      phía bên phải của mũi tên.</dd>
    <dt>replacement-path</dt>
    <dd>Đường dẫn mà Go sẽ tìm module được yêu cầu. Đây có thể là đường dẫn
      module hoặc đường dẫn đến một thư mục trên hệ thống tệp cục bộ của
      module thay thế. Nếu đây là đường dẫn module, bạn phải chỉ định giá trị
      <em>replacement-version</em>. Nếu đây là đường dẫn cục bộ, bạn không được sử dụng giá trị
      <em>replacement-version</em>.</dd>
    <dt>replacement-version</dt>
    <dd>Phiên bản của module thay thế. Phiên bản thay thế chỉ có thể
      được chỉ định nếu <em>replacement-path</em> là đường dẫn module (không phải thư mục cục bộ).</dd>
</dl>

### Ví dụ {#replace-examples}

* Thay thế bằng một fork của kho lưu trữ module

  Trong ví dụ sau, mọi phiên bản của example.com/othermodule được thay thế
  bằng fork được chỉ định của mã nguồn của nó.

  ```
  require example.com/othermodule v1.2.3

  replace example.com/othermodule => example.com/myfork/othermodule v1.2.3-fixed
  ```

  Khi bạn thay thế một đường dẫn module bằng một đường dẫn khác, không thay đổi
  các câu lệnh import cho các gói trong module đang được thay thế.

  Để biết thêm về việc sử dụng bản sao fork của mã module, xem [Yêu cầu mã
  module bên ngoài từ fork kho lưu trữ của riêng bạn](/doc/modules/managing-dependencies#external_fork).

* Thay thế bằng số phiên bản khác

  Ví dụ sau chỉ định rằng phiên bản v1.2.3 nên được sử dụng thay vì bất kỳ
  phiên bản nào khác của module.

  ```
  require example.com/othermodule v1.2.2

  replace example.com/othermodule => example.com/othermodule v1.2.3
  ```

  Ví dụ sau thay thế phiên bản module v1.2.5 bằng phiên bản v1.2.3 của cùng
  module đó.

  ```
  replace example.com/othermodule v1.2.5 => example.com/othermodule v1.2.3
  ```

* Thay thế bằng mã cục bộ

  Ví dụ sau chỉ định rằng một thư mục cục bộ nên được sử dụng làm thay thế cho
  tất cả các phiên bản của module.

  ```
  require example.com/othermodule v1.2.3

  replace example.com/othermodule => ../othermodule
  ```

  Ví dụ sau chỉ định rằng một thư mục cục bộ nên được sử dụng làm thay thế chỉ
  cho v1.2.5.

  ```
  require example.com/othermodule v1.2.5

  replace example.com/othermodule v1.2.5 => ../othermodule
  ```

  Để biết thêm về việc sử dụng bản sao cục bộ của mã module, xem [Yêu cầu mã
  module trong thư mục cục bộ](/doc/modules/managing-dependencies#local_directory).

### Ghi chú {#replace-notes}

Sử dụng chỉ thị `replace` để tạm thời thay thế giá trị đường dẫn module bằng một
giá trị khác khi bạn muốn Go sử dụng đường dẫn khác để tìm mã nguồn của module.
Điều này có tác dụng chuyển hướng việc tìm kiếm module của Go đến vị trí của
bản thay thế. Bạn không cần thay đổi các đường dẫn import gói để sử dụng đường
dẫn thay thế.

Sử dụng các chỉ thị `exclude` và `replace` để kiểm soát việc phân giải
dependency tại thời điểm build khi xây dựng module hiện tại. Các chỉ thị này bị
bỏ qua trong những module phụ thuộc vào module hiện tại.

Chỉ thị `replace` có thể hữu ích trong các tình huống như sau:

* Bạn đang phát triển một module mới có mã nguồn chưa có trong kho lưu trữ. Bạn
  muốn kiểm thử với các client bằng cách sử dụng một phiên bản cục bộ.
* Bạn đã xác định một vấn đề với một dependency, đã clone kho lưu trữ của
  dependency đó, và đang kiểm thử bản sửa lỗi với kho lưu trữ cục bộ.

Lưu ý rằng chỉ một chỉ thị `replace` không tự thêm một module vào [đồ thị
module](/ref/mod#glos-module-graph). Cũng cần có một [chỉ thị
`require`](#require) tham chiếu đến phiên bản module đã được thay thế, trong
tệp `go.mod` của module chính hoặc tệp `go.mod` của một dependency. Nếu bạn
không có một phiên bản cụ thể để thay thế, bạn có thể sử dụng một phiên bản giả,
như trong ví dụ dưới đây. Lưu ý rằng điều này sẽ làm hỏng các module phụ thuộc
vào module của bạn, vì các chỉ thị `replace` chỉ được áp dụng trong module
chính.

```
require example.com/mod v0.0.0-replace

replace example.com/mod v0.0.0-replace => ./mod
```

Để biết thêm về việc thay thế một module được yêu cầu, bao gồm việc sử dụng các
công cụ Go để thực hiện thay đổi, xem:

* [Yêu cầu mã module bên ngoài từ fork
  kho lưu trữ của riêng bạn](/doc/modules/managing-dependencies#external_fork)
* [Yêu cầu mã module trong thư mục
  cục bộ](/doc/modules/managing-dependencies#local_directory)

Để biết thêm về số phiên bản, xem [Đánh số phiên bản
module](/doc/modules/version-numbers).

## exclude {#exclude}

Chỉ định một module hoặc phiên bản module cần loại trừ khỏi đồ thị dependency của module hiện tại.

Để biết thêm, xem [chỉ thị `exclude`](/ref/mod#go-mod-file-exclude) trong
Tài liệu tham khảo về Go Modules.

### Syntax {#exclude-syntax}

<pre>exclude <var>module-path</var> <var>module-version</var></pre>

<dl>
    <dt>module-path</dt>
    <dd>Đường dẫn module của module cần loại trừ.</dd>
    <dt>module-version</dt>
    <dd>Phiên bản cụ thể cần loại trừ.</dd>
</dl>

### Example {#exclude-example}

* Loại trừ example.com/theirmodule phiên bản v1.3.0

  ```
  exclude example.com/theirmodule v1.3.0
  ```

### Notes {#exclude-notes}

Sử dụng chỉ thị `exclude` để loại trừ một phiên bản cụ thể của một module được yêu cầu gián tiếp nhưng không thể tải vì một lý do nào đó. Ví dụ: bạn có thể sử dụng chỉ thị này để loại trừ một phiên bản module có checksum không hợp lệ.

Sử dụng các chỉ thị `exclude` và `replace` để kiểm soát việc phân giải dependency tại thời điểm build khi xây dựng module hiện tại (module chính mà bạn đang xây dựng). Các chỉ thị này bị bỏ qua trong các module phụ thuộc vào module hiện tại.

Bạn có thể sử dụng lệnh [`go mod edit`](/ref/mod#go-mod-edit)
để loại trừ một module, như trong ví dụ sau.

```
go mod edit -exclude=example.com/theirmodule@v1.3.0
```

Để biết thêm về số phiên bản, xem
[Đánh số phiên bản module](/doc/modules/version-numbers).

## retract {#retract}

Cho biết rằng một phiên bản hoặc một khoảng phiên bản của module được định nghĩa bởi `go.mod` không nên được phụ thuộc vào. Chỉ thị `retract` hữu ích khi một phiên bản được phát hành quá sớm hoặc một vấn đề nghiêm trọng được phát hiện sau khi phiên bản đó được phát hành.

Để biết thêm, xem [chỉ thị `retract`](/ref/mod#go-mod-file-retract) trong
Tài liệu tham khảo về Go Modules.

### Syntax {#retract-syntax}

<pre>
retract <var>version</var> // <var>rationale</var>
retract [<var>version-low</var>,<var>version-high</var>] // <var>rationale</var>
</pre>

<dl>
  <dt>version</dt>
  <dd>Một phiên bản đơn lẻ cần thu hồi.</dd>
  <dt>version-low</dt>
  <dd>Giới hạn dưới của một khoảng phiên bản cần thu hồi.</dd>
  <dt>version-high</dt>
  <dd>
    Giới hạn trên của một khoảng phiên bản cần thu hồi. Cả <var>version-low</var>
    và <var>version-high</var> đều được bao gồm trong khoảng này.
  </dd>
  <dt>rationale</dt>
  <dd>
    Chú thích tùy chọn giải thích lý do thu hồi. Có thể được hiển thị trong thông báo gửi đến
    người dùng.
  </dd>
</dl>

### Ví dụ {#retract-example}

* Thu hồi một phiên bản duy nhất

  ```
  retract v1.1.0 // Được phát hành do nhầm lẫn.
  ```

* Thu hồi một phạm vi phiên bản

  ```
  retract [v1.0.0,v1.0.5] // Bản build bị lỗi trên một số nền tảng.
  ```

### Ghi chú {#retract-notes}

Sử dụng chỉ thị `retract` để cho biết rằng một phiên bản trước đó của module không nên được sử dụng. Người dùng sẽ không tự động nâng cấp lên một phiên bản đã bị thu hồi bằng `go get`, `go mod tidy` hoặc các lệnh khác. Người dùng sẽ không thấy phiên bản đã bị thu hồi là một bản cập nhật khả dụng với `go list -m -u`.

Các phiên bản đã bị thu hồi nên vẫn được giữ khả dụng để những người dùng đã phụ thuộc vào chúng có thể build các gói của họ. Ngay cả khi một phiên bản đã bị thu hồi bị xóa khỏi kho lưu trữ nguồn, nó vẫn có thể khả dụng trên các mirror như [proxy.golang.org](https://proxy.golang.org). Người dùng phụ thuộc vào các phiên bản đã bị thu hồi có thể được thông báo khi họ chạy `go get` hoặc `go list -m -u` trên các module liên quan.

Lệnh `go` phát hiện các phiên bản đã bị thu hồi bằng cách đọc các chỉ thị `retract` trong tệp `go.mod` của phiên bản mới nhất của một module. Phiên bản mới nhất là, theo thứ tự ưu tiên:

1. Phiên bản release cao nhất, nếu có
2. Phiên bản pre-release cao nhất, nếu có
3. Một pseudo-version cho đầu của nhánh mặc định trong kho lưu trữ.

Khi thêm một lần thu hồi, hầu như bạn luôn cần gắn thẻ một phiên bản mới cao hơn để lệnh có thể thấy nó trong phiên bản mới nhất của module.

Bạn có thể phát hành một phiên bản chỉ có mục đích duy nhất là báo hiệu các lần thu hồi. Trong trường hợp này, phiên bản mới cũng có thể tự thu hồi chính nó.

Ví dụ, nếu bạn vô tình gắn thẻ `v1.0.0`, bạn có thể gắn thẻ `v1.0.1` với các chỉ thị sau:

```
retract v1.0.0 // Được phát hành do nhầm lẫn.
retract v1.0.1 // Chỉ chứa thông tin thu hồi.
```

Đáng tiếc là một khi một phiên bản đã được phát hành, nó không thể được thay đổi. Nếu sau đó bạn gắn thẻ `v1.0.0` tại một commit khác, lệnh `go` có thể phát hiện tổng kiểm tra không khớp trong `go.sum` hoặc trong [cơ sở dữ liệu checksum](/ref/mod#checksum-database).

Các phiên bản đã bị thu hồi của một module thường không xuất hiện trong đầu ra của `go list -m -versions`, nhưng bạn có thể sử dụng `-retracted` để hiển thị chúng. Để biết thêm, xem [`go list -m`](/ref/mod#go-list-m) trong Tài liệu tham khảo Go Modules.

## ignore {#ignore}

Chỉ định các đường dẫn thư mục trong module mà lệnh `go` nên bỏ qua khi khớp các mẫu gói.

Để biết thêm, xem [chỉ thị `ignore`](/ref/mod#go-mod-file-ignore) trong Tham chiếu Go Modules.

### Syntax {#ignore-syntax}

<pre>
ignore <var>path</var>
</pre>

<dl>
  <dt>path</dt>
  <dd>
    Đường dẫn được phân tách bằng dấu gạch chéo cần bỏ qua. Nếu đường dẫn bắt đầu bằng <code>./</code>,
    nó được hiểu là tương đối với thư mục gốc của module. Nếu không, mọi thư mục có tên đó
    ở bất kỳ cấp độ nào trong module sẽ bị bỏ qua.
  </dd>
</dl>

### Examples {#ignore-examples}

* Bỏ qua một thư mục cục bộ tương đối với thư mục gốc của module

  ```
  ignore ./node_modules
  ```

* Bỏ qua mọi thư mục có tên `generated` ở bất kỳ đâu trong module

  ```
  ignore generated
  ```

* Bỏ qua nhiều đường dẫn bằng một block

  ```
  ignore (
      static
      content/html
      ./third_party/javascript
  )
  ```

### Notes {#ignore-notes}

Sử dụng chỉ thị `ignore` để ngăn lệnh `go` khớp các gói trong các thư mục được tạo tự động hoặc không phải Go khi sử dụng các mẫu ký tự đại diện như `./...`. Các thư mục bị bỏ qua và nội dung của chúng bị loại khỏi các mẫu gói nhưng vẫn hiện diện trong cây tệp của module.

Chỉ thị `ignore` chỉ áp dụng trong tệp `go.mod` của module chính. Nó không có tác dụng trong các tệp `go.mod` của các dependency.
