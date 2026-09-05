<!--{
  "Title": "Quy trình phát hành và đánh số phiên bản module"
}-->

Khi bạn phát triển module để các lập trình viên khác sử dụng, bạn có thể tuân theo một quy trình làm việc giúp đảm bảo trải nghiệm đáng tin cậy và nhất quán cho các lập trình viên dùng module. Chủ đề này mô tả các bước cấp cao trong quy trình đó.

Để xem tổng quan về phát triển module, xem [Phát triển và xuất bản module](developing).

**Xem thêm**

* Nếu bạn chỉ muốn dùng các package bên ngoài trong code, hãy xem [Quản lý dependency](/doc/modules/managing-dependencies).
* Với mỗi phiên bản mới, bạn thông báo các thay đổi cho module qua số phiên bản. Để biết thêm, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

## Các bước quy trình làm việc phổ biến {#common-steps}

Trình tự sau minh họa các bước quy trình phát hành và đánh số phiên bản cho một module mới ví dụ. Để biết thêm về từng bước, xem các phần trong chủ đề này.

1.  **Bắt đầu một module** và tổ chức mã nguồn của nó để giúp các lập trình viên dễ sử dụng và bạn dễ bảo trì hơn.

    Nếu bạn mới bắt đầu phát triển module, hãy xem [Hướng dẫn: Tạo một module Go](/doc/tutorial/create-module).

    Trong hệ thống xuất bản module phi tập trung của Go, cách bạn tổ chức code rất quan trọng. Để biết thêm, xem [Quản lý mã nguồn module](/doc/modules/managing-source).

1.  Thiết lập để **viết code client cục bộ** gọi các hàm trong module chưa xuất bản.

    Trước khi bạn xuất bản một module, nó không có sẵn cho quy trình quản lý dependency thông thường sử dụng các lệnh như `go get`. Một cách tốt để kiểm thử mã nguồn module ở giai đoạn này là thử nó khi nó nằm trong một thư mục cục bộ với code gọi của bạn.

    Xem [Viết code dựa trên module chưa xuất bản](#unpublished) để biết thêm về phát triển cục bộ.

1.  Khi mã nguồn của module sẵn sàng để các lập trình viên khác thử nghiệm, **bắt đầu xuất bản các pre-release v0** như alpha và beta. Xem [Xuất bản các phiên bản pre-release](#pre-release) để biết thêm.

1.  **Phát hành v0** không được đảm bảo ổn định, nhưng người dùng có thể thử nghiệm. Để biết thêm, xem [Xuất bản phiên bản đầu tiên (không ổn định)](#first-unstable).

1.  Sau khi phiên bản v0 được xuất bản, bạn có thể (và nên!) tiếp tục **phát hành các phiên bản mới** của nó.

    Các phiên bản mới này có thể bao gồm sửa lỗi (bản vá lỗi), bổ sung vào API công khai của module (bản phát hành phụ), và thậm chí các thay đổi phá vỡ tương thích. Vì phiên bản v0 không đảm bảo tính ổn định hoặc tương thích ngược, bạn có thể thực hiện các thay đổi phá vỡ tương thích trong các phiên bản của nó.

    Để biết thêm, xem [Xuất bản bản sửa lỗi](#bug-fixes) và [Xuất bản các thay đổi API không phá vỡ tương thích](#non-breaking).

1.  Khi bạn chuẩn bị một phiên bản ổn định để phát hành, hãy **xuất bản các pre-release dưới dạng alpha và beta**. Để biết thêm, xem [Xuất bản các phiên bản pre-release](#pre-release).

1.  Phát hành v1 là **bản phát hành ổn định đầu tiên**.

    Đây là bản phát hành đầu tiên đưa ra các cam kết về tính ổn định của module. Để biết thêm, xem [Xuất bản phiên bản ổn định đầu tiên](#first-stable).

1.  Trong phiên bản v1, **tiếp tục sửa lỗi** và, khi cần thiết, thực hiện các bổ sung vào API công khai của module.

    Để biết thêm, xem [Xuất bản bản sửa lỗi](#bug-fixes) và [Xuất bản các thay đổi API không phá vỡ tương thích](#non-breaking).

1.  Khi không thể tránh khỏi, xuất bản các thay đổi phá vỡ tương thích trong một **phiên bản chính mới**.

    Một bản cập nhật phiên bản chính -- chẳng hạn từ v1.x.x lên v2.x.x -- có thể là một bản nâng cấp rất gây khó chịu cho người dùng module. Đây nên là phương án cuối cùng. Để biết thêm, xem [Xuất bản các thay đổi API phá vỡ tương thích](#breaking).

## Viết code dựa trên module chưa xuất bản {#unpublished}

Khi bạn bắt đầu phát triển một module hoặc phiên bản mới của module, bạn chưa xuất bản nó. Trước khi xuất bản module, bạn sẽ không thể dùng các lệnh Go để thêm module như một dependency. Thay vào đó, ban đầu, khi viết code client trong một module khác gọi các hàm trong module chưa xuất bản, bạn cần tham chiếu đến một bản sao của module trên hệ thống tệp cục bộ.

Bạn có thể tham chiếu đến một module cục bộ từ tệp go.mod của module client bằng cách dùng chỉ thị `replace` trong tệp go.mod của module client. Để biết thêm thông tin, xem [Yêu cầu mã nguồn module từ thư mục cục bộ](managing-dependencies#local_directory).

## Xuất bản các phiên bản pre-release {#pre-release}

Bạn có thể xuất bản các phiên bản pre-release để đưa module lên cho người khác thử nghiệm và cho bạn phản hồi. Một phiên bản pre-release không đảm bảo tính ổn định.

Số phiên bản pre-release được thêm vào sau cùng một định danh pre-release. Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

Đây là hai ví dụ:

```
v0.2.1-beta.1
v1.2.3-alpha
```

Khi đưa một phiên bản pre-release lên, hãy nhớ rằng các lập trình viên sử dụng bản pre-release sẽ cần chỉ định nó rõ ràng theo phiên bản với lệnh `go get`. Lý do là, theo mặc định, lệnh `go` ưu tiên các phiên bản phát hành hơn các phiên bản pre-release khi tìm kiếm module bạn yêu cầu. Vì vậy, lập trình viên phải lấy bản pre-release bằng cách chỉ định rõ ràng, như trong ví dụ sau:

```
go get example.com/theirmodule@v1.2.3-alpha
```

Bạn xuất bản một phiên bản pre-release bằng cách gắn tag mã nguồn module trong kho lưu trữ, chỉ định định danh pre-release trong tag. Để biết thêm, xem [Xuất bản một module](publishing).

## Xuất bản phiên bản đầu tiên (không ổn định) {#first-unstable}

Cũng như khi bạn xuất bản một phiên bản pre-release, bạn có thể xuất bản các phiên bản phát hành không đảm bảo tính ổn định hoặc tương thích ngược, nhưng cho người dùng cơ hội thử nghiệm module và cung cấp phản hồi cho bạn.

Các phiên bản không ổn định là những phiên bản có số phiên bản trong phạm vi v0.x.x. Một phiên bản v0 không đảm bảo tính ổn định hoặc tương thích ngược. Nhưng nó cho bạn cách thu thập phản hồi và tinh chỉnh API trước khi đưa ra các cam kết ổn định với v1 và sau đó. Để biết thêm, xem [Đánh số phiên bản module](version-numbers).

Cũng như với các phiên bản đã xuất bản khác, bạn có thể tăng các phần phụ và vá lỗi của số phiên bản v0 khi thực hiện các thay đổi hướng đến việc phát hành v1 ổn định. Ví dụ, sau khi phát hành v.0.0.0, bạn có thể phát hành v0.0.1 với bộ sửa lỗi đầu tiên.

Đây là ví dụ số phiên bản:

```
v0.1.3
```

Bạn xuất bản bản phát hành không ổn định bằng cách gắn tag mã nguồn module trong kho lưu trữ, chỉ định số phiên bản v0 trong tag. Để biết thêm, xem [Xuất bản một module](publishing).

## Xuất bản phiên bản ổn định đầu tiên {#first-stable}

Phiên bản ổn định đầu tiên của bạn sẽ có số phiên bản v1.x.x. Bản phát hành ổn định đầu tiên theo sau các bản phát hành pre-release và v0 mà qua đó bạn nhận được phản hồi, sửa lỗi và ổn định module cho người dùng.

Với bản phát hành v1, bạn đưa ra các cam kết sau với các lập trình viên dùng module:

* Họ có thể nâng cấp lên các phiên bản phụ và vá lỗi tiếp theo của phiên bản chính mà không làm hỏng code của họ.
* Bạn sẽ không thực hiện thêm các thay đổi đối với API công khai của module -- bao gồm chữ ký hàm và phương thức -- phá vỡ tính tương thích ngược.
* Bạn sẽ không xóa bất kỳ kiểu được xuất nào, điều đó sẽ phá vỡ tính tương thích ngược.
* Các thay đổi trong tương lai đối với API (chẳng hạn như thêm trường mới vào struct) sẽ tương thích ngược và sẽ được bao gồm trong bản phát hành phụ mới.
* Các bản sửa lỗi (chẳng hạn như bản sửa lỗi bảo mật) sẽ được bao gồm trong bản vá lỗi hoặc như một phần của bản phát hành phụ.

**Lưu ý:** Mặc dù phiên bản chính đầu tiên của bạn có thể là phiên bản v0, nhưng phiên bản v0 không đảm bảo tính ổn định hoặc tương thích ngược. Do đó, khi bạn tăng từ v0 lên v1, bạn không cần phải lưu ý đến việc phá vỡ tính tương thích ngược vì phiên bản v0 không được coi là ổn định.

Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

Đây là ví dụ số phiên bản ổn định:

```
v1.0.0
```

Bạn xuất bản bản phát hành ổn định đầu tiên bằng cách gắn tag mã nguồn module trong kho lưu trữ, chỉ định số phiên bản v1 trong tag. Để biết thêm, xem [Xuất bản một module](publishing).

## Xuất bản bản sửa lỗi {#bug-fixes}

Bạn có thể xuất bản một bản phát hành trong đó các thay đổi chỉ giới hạn ở việc sửa lỗi. Đây được gọi là bản vá lỗi.

Một _bản vá lỗi_ chỉ bao gồm các thay đổi nhỏ. Đặc biệt, nó không bao gồm các thay đổi đối với API công khai của module. Các lập trình viên của code sử dụng module có thể nâng cấp lên phiên bản này một cách an toàn mà không cần thay đổi code của họ.

**Lưu ý:** Bản vá lỗi của bạn nên cố gắng không nâng cấp bất kỳ dependency bắc cầu nào của module đó quá một bản vá lỗi. Nếu không, ai đó nâng cấp lên bản vá lỗi của module bạn có thể vô tình kéo theo một thay đổi xâm phạm hơn vào một dependency bắc cầu mà họ đang sử dụng.

Bản vá lỗi tăng phần vá lỗi của số phiên bản module. Để biết thêm, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

Trong ví dụ sau, v1.0.1 là bản vá lỗi.

Phiên bản cũ: `v1.0.0`

Phiên bản mới: `v1.0.1`

Bạn xuất bản bản vá lỗi bằng cách gắn tag mã nguồn module trong kho lưu trữ, tăng số phiên bản vá lỗi trong tag. Để biết thêm, xem [Xuất bản một module](publishing).

## Xuất bản các thay đổi API không phá vỡ tương thích {#non-breaking}

Bạn có thể thực hiện các thay đổi không phá vỡ tương thích đối với API công khai của module và xuất bản những thay đổi đó trong bản phát hành _phụ_.

Phiên bản này thay đổi API, nhưng không theo cách phá vỡ code gọi. Điều này có thể bao gồm các thay đổi đối với dependency của module hoặc việc bổ sung các hàm, phương thức, trường struct hoặc kiểu mới. Dù có các thay đổi bao gồm trong đó, loại bản phát hành này đảm bảo tính tương thích ngược và ổn định cho code hiện tại gọi các hàm của module.

Bản phát hành phụ tăng phần phụ của số phiên bản module. Để biết thêm, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

Trong ví dụ sau, v1.1.0 là bản phát hành phụ.

Phiên bản cũ: `v1.0.1`

Phiên bản mới: `v1.1.0`

Bạn xuất bản bản phát hành phụ bằng cách gắn tag mã nguồn module trong kho lưu trữ, tăng số phiên bản phụ trong tag. Để biết thêm, xem [Xuất bản một module](publishing).

## Xuất bản các thay đổi API phá vỡ tương thích {#breaking}

Bạn có thể xuất bản một phiên bản phá vỡ tính tương thích ngược bằng cách xuất bản bản phát hành _chính_.

Bản phát hành phiên bản chính không đảm bảo tính tương thích ngược, thường là vì nó bao gồm các thay đổi đối với API công khai của module sẽ làm hỏng code đang dùng các phiên bản trước của module.

Do ảnh hưởng gây gián đoạn mà bản nâng cấp phiên bản chính có thể gây ra cho code phụ thuộc vào module, bạn nên tránh cập nhật phiên bản chính nếu có thể. Để biết thêm về cập nhật phiên bản chính, xem [Phát triển bản cập nhật phiên bản chính](/doc/modules/major-version). Để biết các chiến lược tránh thực hiện các thay đổi phá vỡ tương thích, xem bài đăng blog [Giữ cho module của bạn tương thích](/blog/module-compatibility).

Trong khi việc xuất bản các loại phiên bản khác về cơ bản chỉ yêu cầu gắn tag mã nguồn module với số phiên bản, việc xuất bản bản cập nhật phiên bản chính đòi hỏi nhiều bước hơn.

1.  Trước khi bắt đầu phát triển phiên bản chính mới, trong kho lưu trữ hãy tạo một nơi cho mã nguồn của phiên bản mới.

    Một cách để làm điều này là tạo một nhánh mới trong kho lưu trữ dành riêng cho phiên bản chính mới và các phiên bản phụ và vá lỗi tiếp theo của nó. Để biết thêm, xem [Quản lý mã nguồn module](/doc/modules/managing-source).

1.  Trong tệp go.mod của module, sửa đường dẫn module để thêm số phiên bản chính mới, như trong ví dụ sau:

    ```
    example.com/mymodule/v2
    ```

    Do đường dẫn module là định danh của module, sự thay đổi này thực sự tạo ra một module mới. Nó cũng thay đổi đường dẫn package, đảm bảo các lập trình viên sẽ không vô tình import một phiên bản làm hỏng code của họ. Thay vào đó, những người muốn nâng cấp sẽ thay thế rõ ràng các xuất hiện của đường dẫn cũ bằng đường dẫn mới.

1.  Trong code của bạn, hãy thay đổi bất kỳ đường dẫn package nào mà bạn đang import các package từ module bạn đang cập nhật, bao gồm các package trong module bạn đang cập nhật. Bạn cần làm điều này vì bạn đã thay đổi đường dẫn module.

1.  Cũng như với bất kỳ bản phát hành mới nào, bạn nên xuất bản các phiên bản pre-release để nhận phản hồi và báo cáo lỗi trước khi xuất bản bản phát hành chính thức.

1.  Xuất bản phiên bản chính mới bằng cách gắn tag mã nguồn module trong kho lưu trữ, tăng số phiên bản chính trong tag -- chẳng hạn từ v1.5.2 lên v2.0.0.

    Để biết thêm, xem [Xuất bản một module](/doc/modules/publishing).
