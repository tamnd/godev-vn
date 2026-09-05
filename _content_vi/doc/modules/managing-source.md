<!--{
  "Title": "Quản lý mã nguồn module"
}-->

Khi bạn phát triển các module để xuất bản cho người khác sử dụng, bạn có thể giúp đảm bảo rằng module của mình dễ sử dụng hơn cho các lập trình viên khác bằng cách tuân theo các quy ước kho lưu trữ được mô tả trong chủ đề này.

Chủ đề này mô tả các hành động bạn có thể thực hiện khi quản lý kho lưu trữ module. Để biết thông tin về trình tự các bước trong quy trình làm việc khi cập nhật từ phiên bản này sang phiên bản khác, xem [Quy trình phát hành và đánh số phiên bản module](release-workflow).

Một số quy ước được mô tả ở đây là bắt buộc trong các module, trong khi những quy ước khác là thực hành tốt nhất. Nội dung này giả định bạn đã quen với các thực hành sử dụng module cơ bản được mô tả trong [Quản lý dependency](/doc/modules/managing-dependencies).

Go hỗ trợ các kho lưu trữ sau để xuất bản module: Git, Subversion, Mercurial, Bazaar và Fossil.

Để xem tổng quan về phát triển module, xem [Phát triển và xuất bản module](developing).

## Cách công cụ Go tìm module đã xuất bản của bạn {#tools}

Trong hệ thống phi tập trung của Go để xuất bản module và truy xuất mã nguồn, bạn có thể xuất bản module trong khi giữ code trong kho lưu trữ của mình. Công cụ Go dựa vào các quy tắc đặt tên bao gồm đường dẫn kho lưu trữ và tag kho lưu trữ biểu thị tên và số phiên bản của module. Khi kho lưu trữ của bạn tuân theo các yêu cầu này, mã nguồn module của bạn có thể được tải về từ kho lưu trữ bằng công cụ Go như [lệnh `go get`](/ref/mod#go-get).

Khi một lập trình viên dùng lệnh `go get` để lấy mã nguồn cho các package mà code của họ import, lệnh thực hiện như sau:

1. Từ các câu lệnh `import` trong mã nguồn Go, `go get` xác định đường dẫn module trong đường dẫn package.
1. Sử dụng URL được lấy từ đường dẫn module, lệnh tìm kiếm mã nguồn module trên một proxy server module hoặc trực tiếp tại kho lưu trữ của nó.
1. Xác định vị trí mã nguồn của phiên bản module cần tải xuống bằng cách khớp số phiên bản của module với một tag kho lưu trữ để tìm code trong kho lưu trữ. Khi số phiên bản cần dùng chưa được biết, `go get` tìm phiên bản phát hành mới nhất.
1. Truy xuất mã nguồn module và tải xuống vào bộ nhớ cache module cục bộ của lập trình viên.

## Tổ chức code trong kho lưu trữ {#repository}

Bạn có thể giữ cho việc bảo trì đơn giản và cải thiện trải nghiệm của lập trình viên với module bằng cách tuân theo các quy ước được mô tả ở đây. Việc đưa mã nguồn module vào kho lưu trữ nhìn chung cũng đơn giản như với các code khác.

Sơ đồ sau minh họa một cấu trúc cây mã nguồn cho một module đơn giản gồm hai package.

<img src="images/source-hierarchy.png"
     alt="Diagram illustrating a module source code hierarchy"
     style="width: 250px;" />

Commit đầu tiên của bạn nên bao gồm các tệp được liệt kê trong bảng sau:

<table id="module-files" class="DocTable">
  <thead>
    <tr class="DocTable-head">
      <th class="DocTable-cell" width="20%">Tệp</td>
      <th class="DocTable-cell">Mô tả</th>
    </tr>
  </thead>
  <tbody>
    <tr class="DocTable-row">
      <td class="DocTable-cell">LICENSE</td>
      <td class="DocTable-cell">Giấy phép của module.</td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">go.mod</td>
      <td class="DocTable-cell"><p>Mô tả module, bao gồm đường dẫn module (trên thực tế là tên của nó) và các dependency. Để biết thêm, xem
        <a href="gomod-ref">tài liệu tham chiếu go.mod</a>.</p>
      <p>Đường dẫn module sẽ được đưa ra trong chỉ thị module, chẳng hạn:</p>
      <pre>module example.com/mymodule</pre>
      <p>Để biết thêm về cách chọn đường dẫn module, xem
          <a href="/doc/modules/managing-dependencies#naming_module">Quản lý
          dependency</a>.</p>
      <p>Mặc dù bạn có thể chỉnh sửa tệp go.mod, bạn sẽ thấy việc thực hiện thay đổi thông qua các lệnh <code>go</code> đáng tin cậy hơn.</p>
      </td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">go.sum</td>
      <td class="DocTable-cell"><p>Chứa các hash mật mã đại diện cho các dependency của module. Công cụ Go dùng các hash này để xác thực các module được tải xuống, cố gắng xác nhận rằng module được tải xuống là xác thực. Khi việc xác nhận này thất bại, Go sẽ hiển thị lỗi bảo mật.<p>
      <p>Tệp sẽ rỗng hoặc không tồn tại khi không có dependency. Bạn không nên chỉnh sửa tệp này trừ khi dùng lệnh <code>go mod tidy</code>, lệnh này sẽ xóa các mục không cần thiết.</p>
      </td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell">Các thư mục package và mã nguồn .go.</td>
      <td class="DocTable-cell">Các thư mục và tệp .go bao gồm các package Go và mã nguồn trong module.</td>
    </tr>
  </tbody>
</table>

Từ dòng lệnh, bạn có thể tạo một kho lưu trữ rỗng, thêm các tệp sẽ là một phần trong commit đầu tiên và commit với một thông điệp. Đây là ví dụ dùng git:


```
$ git init
$ git add --all
$ git commit -m "mycode: initial commit"
$ git push
```

## Chọn phạm vi kho lưu trữ {#repository-scope}

Bạn xuất bản code trong một module khi code đó cần được đánh số phiên bản độc lập với code trong các module khác.

Thiết kế kho lưu trữ của bạn để lưu trữ một module duy nhất tại thư mục gốc sẽ giúp việc bảo trì đơn giản hơn, đặc biệt theo thời gian khi bạn xuất bản các phiên bản phụ và vá lỗi mới, tạo nhánh vào các phiên bản chính mới và nhiều hơn nữa. Tuy nhiên, nếu nhu cầu của bạn đòi hỏi, bạn có thể duy trì một tập hợp module trong một kho lưu trữ duy nhất.

### Lưu trữ một module trên mỗi kho lưu trữ {#one-module-source}

Bạn có thể duy trì một kho lưu trữ chứa mã nguồn của một module duy nhất. Trong mô hình này, bạn đặt tệp go.mod tại gốc kho lưu trữ, với các thư mục con chứa mã nguồn Go nằm bên dưới.

Đây là cách tiếp cận đơn giản nhất, giúp module của bạn dễ quản lý hơn theo thời gian. Nó giúp bạn tránh phải thêm tiền tố đường dẫn thư mục vào số phiên bản module.

<img src="images/single-module.png"
     alt="Diagram illustrating a single module's source in its repository"
     style="width: 425px;" />

### Lưu trữ nhiều module trong một kho lưu trữ duy nhất {#multiple-module-source}

Bạn có thể xuất bản nhiều module từ một kho lưu trữ duy nhất. Ví dụ, bạn có thể có code trong một kho lưu trữ tạo thành nhiều module, nhưng muốn đánh số phiên bản các module đó riêng biệt.

Mỗi thư mục con là thư mục gốc của module phải có tệp go.mod riêng.

Việc lưu trữ mã nguồn module trong các thư mục con làm thay đổi dạng tag phiên bản bạn phải dùng khi xuất bản module. Bạn phải thêm tiền tố là tên thư mục con là thư mục gốc module vào phần số phiên bản của tag. Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

Ví dụ, với module `example.com/mymodules/module1` bên dưới, bạn sẽ có những điều sau cho phiên bản v1.2.3:

*   Đường dẫn module: `example.com/mymodules/module1`
*   Tag phiên bản: `module1/v1.2.3`
*   Đường dẫn package được import bởi người dùng: `example.com/mymodules/module1/package1`
*   Đường dẫn module và phiên bản như được chỉ định trong chỉ thị require của người dùng: `example.com/mymodules/module1 v1.2.3`

<img src="images/multiple-modules.png"
     alt="Diagram illustrating two modules in a single repository"
     style="width: 480px;" />
