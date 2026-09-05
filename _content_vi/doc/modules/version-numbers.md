<!--{
  "Title": "Đánh số phiên bản module"
}-->

Lập trình viên của module dùng từng phần của số phiên bản module để báo hiệu mức độ ổn định và tính tương thích ngược của phiên bản. Với mỗi bản phát hành mới, số phiên bản của module phản ánh cụ thể bản chất của các thay đổi kể từ bản phát hành trước.

Khi bạn đang phát triển code sử dụng các module bên ngoài, bạn có thể dùng số phiên bản để hiểu mức độ ổn định của một module bên ngoài khi cân nhắc nâng cấp. Khi bạn đang phát triển các module của riêng mình, số phiên bản của bạn sẽ thông báo mức độ ổn định và tính tương thích ngược của các module đó cho các lập trình viên khác.

Chủ đề này mô tả ý nghĩa của số phiên bản module.

**Xem thêm**

* Khi bạn đang dùng các package bên ngoài trong code, bạn có thể quản lý các dependency đó bằng công cụ Go. Để biết thêm, xem [Quản lý dependency](managing-dependencies).
* Nếu bạn đang phát triển module cho người khác sử dụng, bạn áp dụng số phiên bản khi xuất bản module, gắn tag module trong kho lưu trữ của nó. Để biết thêm, xem [Xuất bản một module](publishing).

Một module được phát hành với số phiên bản theo mô hình semantic versioning, như trong minh họa sau:

<img src="images/version-number.png"
     alt="Diagram illustrating a semantic version number showing major version 1, minor version 4, patch version 0, and pre-release version beta 2"
     style="width: 300px;" />

Bảng sau mô tả cách các phần của số phiên bản báo hiệu mức độ ổn định và tính tương thích ngược của module.

<table class="DocTable">
  <thead>
    <tr class="DocTable-head">
      <th class="DocTable-cell" width="20%">Giai đoạn phiên bản</th>
      <th class="DocTable-cell">Ví dụ</th>
      <th class="DocTable-cell">Thông điệp tới lập trình viên</th>
    </tr>
  </thead>
  <tbody>
    <tr class="DocTable-row">
      <td class="DocTable-cell"><a href="#in-development">Đang phát triển</a></td>
      <td class="DocTable-cell">Số pseudo-version tự động
      <p>v<strong>0</strong>.x.x</td>
      <td class="DocTable-cell">Báo hiệu rằng module vẫn <strong>đang phát triển và không ổn định</strong>. Bản phát hành này không có đảm bảo tương thích ngược hoặc tính ổn định.</td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell"><a href="#major">Phiên bản chính</a></td>
      <td class="DocTable-cell">v<strong>1</strong>.x.x</td>
      <td class="DocTable-cell">Báo hiệu <strong>các thay đổi API công khai không tương thích ngược</strong>. Bản phát hành này không đảm bảo tương thích ngược với các phiên bản chính trước đó.</td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell"><a href="#minor">Phiên bản phụ</a></td>
      <td class="DocTable-cell">vx.<strong>4</strong>.x</td>
      <td class="DocTable-cell">Báo hiệu <strong>các thay đổi API công khai tương thích ngược</strong>. Bản phát hành này đảm bảo tính tương thích ngược và ổn định.</td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell"><a href="#patch">Phiên bản vá lỗi</a></td>
      <td class="DocTable-cell">vx.x.<strong>1</strong></td>
      <td class="DocTable-cell">Báo hiệu <strong>các thay đổi không ảnh hưởng đến API công khai của module</strong> hoặc các dependency của nó. Bản phát hành này đảm bảo tính tương thích ngược và ổn định.</td>
    </tr>
    <tr class="DocTable-row">
      <td class="DocTable-cell"><a href="#pre-release">Phiên bản pre-release</a></td>
      <td class="DocTable-cell">vx.x.x-<strong>beta.2</strong></td>
      <td class="DocTable-cell">Báo hiệu đây là <strong>mốc pre-release, chẳng hạn alpha hoặc beta</strong>. Bản phát hành này không có đảm bảo tính ổn định.</td>
    </tr>
  </tbody>
</table>

<a id="in-development" ></a>
## Đang phát triển

Báo hiệu rằng module vẫn đang phát triển và **không ổn định**. Bản phát hành này không có đảm bảo tương thích ngược hoặc tính ổn định.

Số phiên bản có thể có một trong các dạng sau:

**Số pseudo-version**

> v0.0.0-20170915032832-14c0d48ead0c

**Số v0**

> v0.x.x

<a id="pseudo" ></a>
### Số pseudo-version

Khi một module chưa được gắn tag trong kho lưu trữ, công cụ Go sẽ tạo ra một số pseudo-version để dùng trong tệp go.mod của code gọi các hàm trong module.

**Lưu ý:** Theo thực hành tốt nhất, luôn để công cụ Go tạo ra số pseudo-version thay vì tự tạo.

Pseudo-version hữu ích khi một lập trình viên của code sử dụng các hàm của module cần phát triển dựa trên một commit chưa được gắn tag semantic version.

Một số pseudo-version gồm ba phần được phân tách bằng dấu gạch ngang, như trong dạng sau:

#### Cú pháp

_baseVersionPrefix_-_timestamp_-_revisionIdentifier_

#### Các phần

* **baseVersionPrefix** (vX.0.0 hoặc vX.Y.Z-0) là một giá trị được lấy từ tag semantic version đứng trước revision hoặc từ vX.0.0 nếu không có tag như vậy.

* **timestamp** (yymmddhhmmss) là thời gian UTC khi revision được tạo. Trong Git, đây là thời gian commit, không phải thời gian tác giả.

* **revisionIdentifier** (abcdefabcdef) là tiền tố 12 ký tự của hash commit, hoặc trong Subversion là số revision được đệm bằng số không.

<a id="v0" ></a>
### Số v0

Một module được xuất bản với số v0 sẽ có số phiên bản semantic chính thức với phần chính, phụ và vá lỗi, cùng với định danh pre-release tùy chọn.

Mặc dù phiên bản v0 có thể được dùng trong môi trường production, nó không đảm bảo tính ổn định hoặc tương thích ngược. Ngoài ra, các phiên bản v1 và sau đó được phép phá vỡ tính tương thích ngược với code đang dùng các phiên bản v0. Vì lý do này, một lập trình viên với code sử dụng các hàm trong module v0 có trách nhiệm thích nghi với các thay đổi không tương thích cho đến khi v1 được phát hành.

<a id="pre-release" ></a>
## Phiên bản pre-release

Báo hiệu đây là mốc pre-release, chẳng hạn alpha hoặc beta. Bản phát hành này không có đảm bảo tính ổn định.

#### Ví dụ

```
vx.x.x-beta.2
```

Lập trình viên của module có thể dùng định danh pre-release với bất kỳ kết hợp major.minor.patch nào bằng cách thêm dấu gạch ngang và định danh pre-release.

<a id="minor" ></a>
## Phiên bản phụ

Báo hiệu các thay đổi tương thích ngược với API công khai của module. Bản phát hành này đảm bảo tính tương thích ngược và ổn định.

#### Ví dụ

```
vx.4.x
```

Phiên bản này thay đổi API công khai của module, nhưng không theo cách phá vỡ code gọi. Điều này có thể bao gồm các thay đổi đối với dependency của module hoặc việc bổ sung các hàm, phương thức, trường struct hoặc kiểu mới.

Nói cách khác, phiên bản này có thể bao gồm các cải tiến thông qua các hàm mới mà một lập trình viên khác có thể muốn sử dụng. Tuy nhiên, lập trình viên đang dùng các phiên bản phụ trước đó không cần thay đổi code của họ.

<a id="patch" ></a>
## Phiên bản vá lỗi

Báo hiệu các thay đổi không ảnh hưởng đến API công khai của module hoặc các dependency của nó. Bản phát hành này đảm bảo tính tương thích ngược và ổn định.

#### Ví dụ

```
vx.x.1
```

Một bản cập nhật tăng con số này chỉ dành cho các thay đổi nhỏ như sửa lỗi. Các lập trình viên của code sử dụng module có thể nâng cấp lên phiên bản này một cách an toàn mà không cần thay đổi code của họ.

<a id="major" ></a>
## Phiên bản chính

Báo hiệu các thay đổi không tương thích ngược trong API công khai của module. Bản phát hành này không đảm bảo tương thích ngược với các phiên bản chính trước đó.

#### Ví dụ

v1.x.x

Số phiên bản v1 trở lên báo hiệu rằng module ổn định để sử dụng (với ngoại lệ là các phiên bản pre-release của nó).

Lưu ý rằng vì phiên bản 0 không đảm bảo tính ổn định hoặc tương thích ngược, một lập trình viên nâng cấp module từ v0 lên v1 có trách nhiệm thích nghi với các thay đổi phá vỡ tính tương thích ngược.

Lập trình viên module chỉ nên tăng con số này vượt quá v1 khi thực sự cần thiết vì bản nâng cấp phiên bản đại diện cho sự gián đoạn đáng kể với các lập trình viên có code dùng hàm trong module được nâng cấp. Sự gián đoạn này bao gồm các thay đổi không tương thích ngược với API công khai, cũng như nhu cầu của các lập trình viên dùng module phải cập nhật đường dẫn package ở bất kỳ đâu họ import các package từ module.

Một bản cập nhật phiên bản chính lên số cao hơn v1 cũng sẽ có đường dẫn module mới. Lý do là đường dẫn module sẽ có số phiên bản chính được thêm vào, như trong ví dụ sau:

```
module example.com/mymodule/v2 v2.0.0
```

Một bản cập nhật phiên bản chính biến đây thành một module mới với lịch sử riêng biệt so với phiên bản trước của module. Nếu bạn đang phát triển module để xuất bản cho người khác, xem "Xuất bản các thay đổi API phá vỡ tương thích" trong [Quy trình phát hành và đánh số phiên bản module](release-workflow).

Để biết thêm về chỉ thị module, xem [tài liệu tham chiếu go.mod](gomod-ref).
