<!--{
  "Title": "Phát triển bản cập nhật phiên bản chính"
}-->

Bạn phải cập nhật lên phiên bản chính khi những thay đổi bạn đang thực hiện trong phiên bản mới tiềm năng không thể đảm bảo khả năng tương thích ngược cho người dùng module. Ví dụ, bạn sẽ thực hiện thay đổi này nếu bạn thay đổi API công khai của module theo cách phá vỡ code client đang dùng các phiên bản trước của module.

> **Lưu ý:** Mỗi loại bản phát hành -- phiên bản chính, phụ, vá lỗi hoặc pre-release -- có ý nghĩa khác nhau đối với người dùng module. Những người dùng đó dựa vào sự khác biệt này để hiểu mức độ rủi ro mà một bản phát hành đại diện cho code của họ. Nói cách khác, khi chuẩn bị một bản phát hành, hãy đảm bảo số phiên bản của nó phản ánh chính xác bản chất của các thay đổi kể từ bản phát hành trước đó. Để biết thêm về số phiên bản, xem [Đánh số phiên bản module](/doc/modules/version-numbers).

**Xem thêm**

* Để xem tổng quan về phát triển module, xem [Phát triển và xuất bản module](developing).
* Để xem toàn bộ quy trình, xem [Quy trình phát hành và đánh số phiên bản module](release-workflow).

## Các cân nhắc khi cập nhật phiên bản chính {#considerations}

Bạn chỉ nên cập nhật lên phiên bản chính mới khi thực sự cần thiết. Một bản cập nhật phiên bản chính đòi hỏi nhiều công sức thay đổi cho cả bạn và người dùng module. Khi cân nhắc cập nhật phiên bản chính, hãy suy nghĩ về những điều sau:

* Hãy rõ ràng với người dùng về ý nghĩa của việc phát hành phiên bản chính mới đối với sự hỗ trợ của bạn cho các phiên bản chính trước đó.

  Các phiên bản trước có bị deprecated không? Có được hỗ trợ như trước không? Bạn có duy trì các phiên bản trước, bao gồm cả việc sửa lỗi không?

* Hãy sẵn sàng đảm nhận việc bảo trì hai phiên bản: phiên bản cũ và phiên bản mới. Ví dụ, nếu bạn sửa lỗi trong một phiên bản, bạn thường sẽ phải chuyển các bản sửa lỗi đó sang phiên bản kia.

* Hãy nhớ rằng phiên bản chính mới là một module mới từ góc độ quản lý dependency. Người dùng của bạn sẽ cần cập nhật để sử dụng module mới sau khi bạn phát hành, thay vì chỉ đơn giản là nâng cấp.

  Lý do là phiên bản chính mới có đường dẫn module khác với phiên bản chính trước đó. Ví dụ, với module có đường dẫn là example.com/mymodule, phiên bản v2 sẽ có đường dẫn module là example.com/mymodule/v2.

* Khi bạn đang phát triển phiên bản chính mới, bạn cũng phải cập nhật các đường dẫn import ở bất kỳ đâu code import các package từ module mới. Người dùng module của bạn cũng phải cập nhật các đường dẫn import của họ nếu muốn nâng cấp lên phiên bản chính mới.

## Tạo nhánh cho bản phát hành chính {#branching}

Cách đơn giản nhất để xử lý mã nguồn khi chuẩn bị phát triển phiên bản chính mới là tạo nhánh kho lưu trữ tại phiên bản mới nhất của phiên bản chính trước đó.

Ví dụ, trong dấu nhắc lệnh, bạn có thể chuyển đến thư mục gốc của module, sau đó tạo nhánh v2 mới tại đó.

```
$ cd mymodule
$ git checkout -b v2
Switched to a new branch "v2"
```

<img src="images/v2-branch-module.png"
     alt="Diagram illustrating a repository branched from master to v2"
     style="width: 600px;" />


Sau khi đã tạo nhánh mã nguồn, bạn cần thực hiện các thay đổi sau đối với mã nguồn của phiên bản mới:

* Trong tệp go.mod của phiên bản mới, thêm số phiên bản chính mới vào đường dẫn module, như trong ví dụ sau:
  * Phiên bản hiện tại: `example.com/mymodule`
  * Phiên bản mới: `example.com/mymodule/v2`

* Trong code Go của bạn, cập nhật mọi đường dẫn package được import từ module, thêm số phiên bản chính vào phần đường dẫn module.
  * Câu lệnh import cũ: `import "example.com/mymodule/package1"`
  * Câu lệnh import mới: `import "example.com/mymodule/v2/package1"`

Để xem các bước xuất bản, hãy xem [Xuất bản một module](/doc/modules/publishing).
