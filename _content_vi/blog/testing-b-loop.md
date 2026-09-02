---
title: "Benchmark dự đoán được hơn với testing.B.Loop"
date: 2025-04-02
by:
- Junyang Shao
tags:
- benchmark
- testing
- compile
summary: Cải thiện vòng lặp benchmark trong Go 1.24.
---

Các lập trình viên Go đã viết benchmark bằng gói
[`testing`](https://pkg.go.dev/testing) có thể đã gặp phải một số
bẫy phổ biến. Go 1.24 giới thiệu một cách viết benchmark mới vừa dễ sử dụng,
vừa mạnh mẽ hơn nhiều:
[`testing.B.Loop`](https://pkg.go.dev/testing#B.Loop).

Theo truyền thống, benchmark Go được viết bằng vòng lặp từ 0 đến `b.N`:
```
func Benchmark(b *testing.B) {
  for range b.N {
    ... code to measure ...
  }
}
```
Thay thế bằng `b.Loop` chỉ là một thay đổi nhỏ:
```
func Benchmark(b *testing.B) {
  for b.Loop() {
    ... code to measure ...
  }
}
```

`testing.B.Loop` có nhiều lợi ích:
* Nó ngăn chặn các tối ưu hóa trình biên dịch không mong muốn bên trong vòng lặp benchmark.
* Nó tự động loại trừ mã thiết lập và dọn dẹp khỏi thời gian đo benchmark.
* Mã không thể vô tình phụ thuộc vào tổng số lần lặp hoặc lần lặp hiện tại.

Tất cả những điều này đều là lỗi dễ mắc phải với benchmark kiểu `b.N`
và sẽ dẫn đến kết quả benchmark sai mà không có cảnh báo nào. Ngoài ra, benchmark kiểu `b.Loop`
còn hoàn thành trong thời gian ngắn hơn!

Hãy khám phá những ưu điểm của `testing.B.Loop` và cách sử dụng hiệu quả.

## Các vấn đề với vòng lặp benchmark cũ

Trước Go 1.24, dù cấu trúc cơ bản của benchmark đơn giản, các benchmark
phức tạp hơn đòi hỏi nhiều sự chú ý hơn:
```
func Benchmark(b *testing.B) {
  ... setup ...
  b.ResetTimer() // if setup may be expensive
  for range b.N {
    ... code to measure ...
    ... use sinks or accumulation to prevent dead-code elimination ...
  }
  b.StopTimer() // if cleanup or reporting may be expensive
  ... cleanup ...
  ... report ...
}
```
Nếu thiết lập hoặc dọn dẹp không tầm thường, lập trình viên cần bao quanh vòng lặp benchmark
bằng các lời gọi `ResetTimer` và/hoặc `StopTimer`. Những thứ này dễ bị quên,
và ngay cả khi lập trình viên nhớ chúng có thể cần thiết, cũng khó để đánh giá xem liệu thiết lập hoặc
dọn dẹp có "đủ tốn kém" để cần chúng hay không.

Nếu không có những thứ này, gói `testing` chỉ có thể đo thời gian toàn bộ hàm benchmark. Nếu một
hàm benchmark bỏ qua chúng, mã thiết lập và dọn dẹp sẽ được tính vào
thời gian đo tổng thể, làm sai lệch kết quả benchmark cuối cùng mà không có thông báo.


Có một bẫy khác, tinh tế hơn, đòi hỏi hiểu biết sâu hơn:
([Nguồn ví dụ](https://eli.thegreenplace.net/2023/common-pitfalls-in-go-benchmarking/))

```
func isCond(b byte) bool {
  if b%3 == 1 && b%7 == 2 && b%17 == 11 && b%31 == 9 {
    return true
  }
  return false
}

func BenchmarkIsCondWrong(b *testing.B) {
  for range b.N {
    isCond(201)
  }
}
```
Trong ví dụ này, người dùng có thể thấy `isCond` thực thi trong thời gian dưới nanosecond.
CPU nhanh, nhưng không nhanh đến vậy! Kết quả bất thường này xuất phát
từ thực tế rằng `isCond` được inline, và vì kết quả của nó không bao giờ được dùng, trình biên dịch
loại nó ra như là dead code. Kết quả là, benchmark này không đo `isCond`
chút nào; nó đo thời gian để không làm gì. Trong trường hợp này, kết quả dưới nanosecond
là một dấu hiệu cảnh báo rõ ràng, nhưng trong các benchmark phức tạp hơn, việc loại một phần dead code
có thể cho ra kết quả trông hợp lý nhưng vẫn không đo đúng thứ cần đo.

## `testing.B.Loop` giúp ích như thế nào

Khác với benchmark kiểu `b.N`, `testing.B.Loop` có thể theo dõi khi nào nó được gọi lần đầu
trong một benchmark và khi nào lần lặp cuối kết thúc. `b.ResetTimer` ở đầu vòng lặp
và `b.StopTimer` ở cuối được tích hợp vào `testing.B.Loop`, loại bỏ nhu cầu
quản lý timer benchmark thủ công cho mã thiết lập và dọn dẹp.

Hơn nữa, trình biên dịch Go giờ đây phát hiện các vòng lặp mà điều kiện chỉ là một lời gọi đến
`testing.B.Loop` và ngăn chặn việc loại dead code trong vòng lặp. Trong Go 1.24, điều này
được thực hiện bằng cách không cho phép inline vào thân của vòng lặp như vậy, nhưng chúng tôi dự định
[cải thiện](/issue/73137) điều này trong tương lai.

Một tính năng hay khác của `testing.B.Loop` là cách tiếp cận khởi động một lần. Với benchmark kiểu `b.N`,
gói testing phải gọi hàm benchmark nhiều lần với các giá trị khác nhau
của `b.N`, tăng dần cho đến khi thời gian đo đạt ngưỡng. Ngược lại, `b.Loop`
có thể đơn giản chạy vòng lặp benchmark cho đến khi đạt ngưỡng thời gian, và chỉ cần gọi
hàm benchmark một lần. Nội bộ, `b.Loop` vẫn sử dụng quá trình khởi động để phân bổ
chi phí đo lường, nhưng điều này được ẩn khỏi bên gọi và có thể hiệu quả hơn.

Một số ràng buộc của vòng lặp kiểu `b.N` vẫn áp dụng cho vòng lặp kiểu `b.Loop`.
Người dùng vẫn có trách nhiệm quản lý timer trong vòng lặp benchmark,
khi cần thiết:
([Nguồn ví dụ](https://eli.thegreenplace.net/2023/common-pitfalls-in-go-benchmarking/))

```
func BenchmarkSortInts(b *testing.B) {
  ints := make([]int, N)
  for b.Loop() {
    b.StopTimer()
    fillRandomInts(ints)
    b.StartTimer()
    slices.Sort(ints)
  }
}
```
Trong ví dụ này, để benchmark hiệu năng sắp xếp tại chỗ của `slices.Sort`, cần một mảng
được khởi tạo ngẫu nhiên cho mỗi lần lặp. Người dùng vẫn phải
quản lý timer thủ công trong những trường hợp như vậy.

Hơn nữa, vẫn cần có đúng một vòng lặp như vậy trong thân hàm benchmark
(vòng lặp kiểu `b.N` không thể cùng tồn tại với vòng lặp kiểu `b.Loop`), và mỗi lần lặp của
vòng lặp nên làm cùng một việc.

## Khi nào nên dùng

Phương thức `testing.B.Loop` hiện là cách ưu tiên để viết benchmark:
```
func Benchmark(b *testing.B) {
  ... setup ...
  for b.Loop() {
    // optional timer control for in-loop setup/cleanup
    ... code to measure ...
  }
  ... cleanup ...
}
```

`testing.B.Loop` cung cấp benchmark nhanh hơn, chính xác hơn và
trực quan hơn.

## Lời cảm ơn

Xin gửi lời cảm ơn chân thành đến tất cả mọi người trong cộng đồng đã đưa ra phản hồi về vấn đề đề xuất
và báo cáo lỗi khi tính năng này được phát hành! Tôi cũng biết ơn Eli
Bendersky vì những tổng kết blog hữu ích của ông. Và cuối cùng xin cảm ơn Austin Clements,
Cherry Mui và Michael Pratt vì sự đánh giá, công việc chu đáo về các lựa chọn thiết kế và
cải thiện tài liệu. Cảm ơn tất cả vì những đóng góp của bạn!
