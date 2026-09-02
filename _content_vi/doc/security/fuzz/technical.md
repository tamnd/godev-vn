<!--{
  "Title": "Chi tiết kỹ thuật về Go Fuzzing",
  "Breadcrumb": true
}-->

Tài liệu này cung cấp tổng quan về các chi tiết kỹ thuật của việc triển khai fuzzing gốc, và nhằm mục đích là tài nguyên cho các người đóng góp.

## Kiến trúc tổng thể

Fuzzer sử dụng một tiến trình điều phối duy nhất, quản lý corpus, và nhiều tiến trình worker, biến đổi đầu vào và thực thi fuzz target. Tiến trình điều phối và các tiến trình worker giao tiếp bằng giao thức RPC dựa trên JSON qua pipe, và các vùng bộ nhớ ảo dùng chung.

Đối với mỗi worker, tiến trình điều phối tạo một goroutine khởi động tiến trình worker và thiết lập giao tiếp xuyên tiến trình. Mỗi goroutine sau đó đọc từ một kênh dùng chung được cung cấp bởi vòng lặp điều phối chính, gửi các hướng dẫn mà nó đọc từ kênh đến các tiến trình worker tương ứng.

Vòng lặp điều phối chính chọn các đầu vào từ corpus, gửi chúng đến kênh worker dùng chung. Worker nào nhận đầu vào đó từ kênh sẽ gửi yêu cầu fuzzing đến tiến trình worker tương ứng. Tiến trình này nằm trong một vòng lặp, biến đổi đầu vào và thực thi fuzz target cho đến khi một lần thực thi gây ra tăng bộ đếm độ phủ, gây ra panic hoặc crash, hoặc vượt qua thời hạn được xác định trước.

Nếu tiến trình worker thực thi một đầu vào đã biến đổi gây ra tăng bộ đếm độ phủ hoặc panic có thể khôi phục, nó báo hiệu điều này cho tiến trình điều phối, tiến trình này sau đó có thể tái cấu trúc đầu vào đã biến đổi. Tiến trình điều phối sẽ cố gắng [thu nhỏ đầu vào](#input-minimization), sau đó thêm nó vào corpus để fuzzing thêm, trong trường hợp tìm thấy độ phủ tăng, hoặc ghi nó vào thư mục testdata, trong trường hợp đầu vào gây ra lỗi hoặc panic.

Nếu một lỗi không thể khôi phục xảy ra trong quá trình fuzzing khiến tiến trình worker tắt (ví dụ vòng lặp vô hạn, os.Exit, hết bộ nhớ, v.v.), thu nhỏ sẽ không được thực hiện, và đầu vào thất bại sẽ được ghi vào thư mục testdata và báo cáo.

<img alt="Sequence diagram of the interaction between coordinator and worker, as described above." src="/security/fuzz/seq-diagram.png"/>

### Giao tiếp xuyên tiến trình

Khi khởi động các tiến trình worker con, tiến trình điều phối thiết lập hai phương thức giao tiếp: một pipe, được sử dụng để truyền các thông điệp RPC dựa trên JSON, và một vùng bộ nhớ dùng chung, được sử dụng để truyền đầu vào và trạng thái RNG. Mỗi tiến trình worker có pipe và vùng bộ nhớ dùng chung riêng của nó.

Pipe RPC được sử dụng bởi tiến trình điều phối để kiểm soát tiến trình worker, gửi cho nó các hướng dẫn fuzzing hoặc thu nhỏ, và bởi worker để chuyển tiếp kết quả của các hoạt động của nó đến tiến trình điều phối (tức là liệu đầu vào có mở rộng độ phủ, gây ra crash, đã được thu nhỏ thành công, v.v.).

Vùng bộ nhớ dùng chung được sử dụng để truyền thông tin cụ thể qua lại với các worker. Tiến trình điều phối sử dụng vùng này để truyền mục corpus để fuzz cho worker, và được worker sử dụng để lưu trữ trạng thái RNG hiện tại của nó. Trạng thái RNG được tiến trình điều phối sử dụng để tái cấu trúc các biến đổi đã được áp dụng cho đầu vào bởi worker khi nó hoàn thành thực thi target (việc tái cấu trúc này xảy ra cả khi worker thoát sạch và khi nó crash.)

## Lựa chọn đầu vào

Tiến trình điều phối hiện tại không triển khai bất kỳ hình thức ưu tiên đầu vào nâng cao nào. Nó lặp qua toàn bộ corpus, vòng lặp sau khi cạn kiệt các mục.

Tương tự, tiến trình điều phối không triển khai bất kỳ loại thu nhỏ corpus nào (không được nhầm lẫn với thu nhỏ đầu vào, [được thảo luận bên dưới](#input-minimization)).

## Hướng dẫn độ phủ

Fuzzer sử dụng [bộ đếm nội tuyến 8 bit tương thích với libFuzzer](https://clang.llvm.org/docs/SanitizerCoverage.html#inline-8bit-counters). Các bộ đếm này được chèn trong quá trình biên dịch tại mỗi cạnh mã, và được tăng khi vào. Các bộ đếm không được bảo vệ chống tràn, để chúng không bị bão hòa.

Tương tự như AFL và libFuzzer, khi theo dõi độ phủ, các bộ đếm được lượng tử hóa thành lũy thừa gần nhất của hai. Điều này cho phép fuzzer phân biệt giữa các thay đổi không đáng kể và đáng kể trong luồng thực thi. Để theo dõi các thay đổi này, fuzzer giữ một slice byte ánh xạ đến các bộ đếm nội tuyến, các bit của đó cho biết liệu có ít nhất một đầu vào trong corpus tăng bộ đếm liên quan ít nhất 2^bit-position lần hay không. Các byte này có thể bị bão hòa, nếu có các đầu vào khiến bộ đếm đạt đến mỗi giá trị lượng tử hóa, tại thời điểm đó bộ đếm liên quan không cung cấp thêm thông tin độ phủ hữu ích.

Vì các bộ đếm độ phủ được thêm vào mọi cạnh trong quá trình biên dịch, mã không được fuzz cũng được đo, điều này có thể khiến worker phát hiện mở rộng độ phủ không liên quan đến target đang được thực thi (ví dụ nếu một đường dẫn mã mới được kích hoạt trong một goroutine không liên quan đến fuzz target). Worker cố gắng giảm điều này theo hai cách: đầu tiên nó đặt lại tất cả bộ đếm ngay trước khi thực thi fuzz target và sau đó chụp ảnh bộ đếm ngay sau khi target trả về, và thứ hai bằng cách bỏ qua rõ ràng một tập hợp các gói có khả năng "ồn ào".

Một số gói rõ ràng không có bộ đếm được chèn, vì chúng có khả năng đưa vào nhiễu bộ đếm không liên quan đến target đang được thực thi. Các gói này là:

* `context`
* `internal/fuzz`
* `reflect`
* `runtime`
* `sync`
* `sync/atomic`
* `syscall`
* `testing`
* `time`

## Động cơ biến đổi

Khi worker nhận được một đầu vào mới, nó áp dụng các biến đổi cho đầu vào trước khi thực thi target với đầu vào. Sau mỗi biến đổi, fuzz target được thực thi với đầu vào mới, và nếu độ phủ không được mở rộng, các biến đổi tiếp theo sẽ được áp dụng. Để ngăn đầu vào phân kỳ quá nhiều so với trạng thái ban đầu của chúng, sau khi năm biến đổi được áp dụng cho một đầu vào, nó được đặt lại về trạng thái ban đầu trước khi các biến đổi tiếp theo được áp dụng. Ví dụ với đầu vào `hello world`, chiến lược biến đổi có thể trông như sau:

```
0. hello world [initial state]
1. kello world [replace first byte]
2. world kello [swap two chunks]
3. world ke    [delete last three bytes]
4. owrld ke    [shuffle first three bytes]
5. owrldx ke   [insert random byte]
6. ello world  [reset to initial state, delete first byte]
...
```

Các mutator cố gắng thiên về tạo ra các đầu vào nhỏ hơn, thay vì đầu vào lớn hơn, để ngăn sự tăng trưởng nhanh của kích thước corpus.

Có nhiều mutator cho kiểu `[]byte` và `string`, và ít mutator hơn cho tất cả các kiểu `int`, `uint` và `float`.

Hiện tại không có chiến lược biến đổi dựa trên thực thi nào được triển khai (chẳng hạn như tương ứng đầu vào-so sánh), cũng không có mutator dựa trên từ điển.

## Thu nhỏ đầu vào

Để ngăn corpus phình to (làm chậm fuzzer cả về hiệu suất, và giảm xác suất rằng một biến đổi sẽ thực sự chạm vào dữ liệu thú vị) chúng tôi cố gắng thu nhỏ mỗi đầu vào được khám phá có mở rộng độ phủ hoặc gây ra một crash có thể khôi phục (các crash không thể khôi phục, chẳng hạn như những crash do hết bộ nhớ, không được thu nhỏ, vì quá trình sẽ cực kỳ chậm). Chiến lược được sử dụng để thu nhỏ khá đơn giản, tuần tự cố gắng loại bỏ các byte khỏi đầu vào trong khi duy trì độ phủ ban đầu được tìm thấy. Cụ thể, cơ chế thu nhỏ sử dụng chiến lược sau:

1. Cố gắng cắt một đoạn byte nhỏ hơn theo cấp số nhân ở cuối đầu vào
2. Cố gắng loại bỏ từng byte riêng lẻ
3. Cố gắng loại bỏ từng tập hợp byte có thể có
4. Cố gắng thay thế từng byte không thể đọc được bằng con người bằng một byte có thể đọc được bằng con người (tức là một thứ gì đó trong tập hợp byte ASCII)
