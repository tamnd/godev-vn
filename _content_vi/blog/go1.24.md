---
title: Go 1.24 đã được phát hành!
date: 2025-02-11
by:
- Junyang Shao, on behalf of the Go team
summary:
  Go 1.24 mang đến type alias generic, cải thiện hiệu năng map, tuân thủ FIPS 140 và nhiều hơn nữa.
---

Hôm nay nhóm Go vui mừng phát hành Go 1.24,
bạn có thể tải về từ [trang download](/dl/).

Go 1.24 có nhiều cải tiến so với Go 1.23. Dưới đây là một số thay đổi đáng chú ý;
để xem danh sách đầy đủ, tham khảo [ghi chú phát hành](/doc/go1.24).

## Thay đổi ngôn ngữ

<!-- go.dev/issue/46477 -->
Go 1.24 nay hỗ trợ đầy đủ [type alias generic](/issue/46477): một type alias
có thể được tham số hóa như một kiểu được định nghĩa.
Xem [đặc tả ngôn ngữ](/ref/spec#Alias_declarations) để biết chi tiết.

## Cải thiện hiệu năng

<!-- go.dev/issue/54766, go.dev/cl/614795, go.dev/issue/68578 -->
Một số cải tiến hiệu năng trong runtime đã giảm chi phí CPU
từ 2-3% tính trung bình trên một bộ benchmark đại diện. Các
cải tiến này bao gồm triển khai `map` built-in mới dựa trên
[Swiss Tables](https://abseil.io/about/design/swisstables), cấp phát bộ nhớ hiệu quả hơn
cho các object nhỏ, và triển khai mutex nội bộ runtime mới.

## Cải tiến công cụ

- <!-- go.dev/issue/48429 -->
  Lệnh `go` nay cung cấp cơ chế theo dõi dependency tool cho một
  module. Dùng `go get -tool` để thêm chỉ thị `tool` vào module hiện tại. Dùng
  `go tool [tên tool]` để chạy các tool được khai báo với chỉ thị `tool`.
  Đọc thêm về [lệnh go](/doc/go1.24#go-command) trong ghi chú phát hành.
- <!-- go.dev/issue/44251 -->
  Trình phân tích `test` mới trong lệnh con `go vet` báo cáo các lỗi phổ biến trong
  khai báo test, fuzzer, benchmark và example trong các gói kiểm thử.
  Đọc thêm về [vet](/doc/go1.24#vet) trong ghi chú phát hành.

## Bổ sung thư viện chuẩn

- Thư viện chuẩn nay bao gồm [một tập cơ chế mới để hỗ trợ
  tuân thủ FIPS 140-3](/doc/security/fips140). Các ứng dụng không cần thay đổi mã nguồn
  để dùng các cơ chế mới cho các thuật toán được chấp thuận. Đọc thêm
  về [tuân thủ FIPS 140-3](/doc/go1.24#fips140) trong ghi chú phát hành.
  Ngoài FIPS 140, một số gói trước đây trong module
  [x/crypto](/pkg/golang.org/x/crypto) nay có sẵn trong
  [thư viện chuẩn](/doc/go1.24#crypto-mlkem).

- Benchmark nay có thể dùng phương thức
  [`testing.B.Loop`](/pkg/testing#B.Loop) nhanh hơn và ít xảy ra lỗi hơn để thực hiện các lần lặp benchmark
  như `for b.Loop() { ... }` thay cho các cấu trúc vòng lặp thông thường liên quan đến
  `b.N` như `for range b.N`. Đọc thêm về
  [hàm benchmark mới](/doc/go1.24#new-benchmark-function) trong ghi chú phát hành.

- Kiểu [`os.Root`](/pkg/os#Root) mới cung cấp khả năng thực hiện
  các thao tác filesystem được cô lập trong một thư mục cụ thể. Đọc thêm về
  [truy cập filesystem](/doc/go1.24#directory-limited-filesystem-access) trong ghi chú phát hành.

- Runtime cung cấp cơ chế finalization mới,
  [`runtime.AddCleanup`](/pkg/runtime#AddCleanup), linh hoạt hơn,
  hiệu quả hơn và ít xảy ra lỗi hơn so với
  [`runtime.SetFinalizer`](/pkg/runtime#SetFinalizer). Đọc thêm về
  [cleanup](/doc/go1.24#improved-finalizers) trong ghi chú phát hành.

## Cải thiện hỗ trợ WebAssembly

<!-- go.dev/issue/65199, CL 603055 -->
Go 1.24 thêm chỉ thị `go:wasmexport` mới cho chương trình Go để export
hàm tới WebAssembly host, và hỗ trợ build một chương trình Go như một WASI
[reactor/library](https://github.com/WebAssembly/WASI/blob/63a46f61052a21bfab75a76558485cf097c0dbba/legacy/application-abi.md#current-unstable-abi).
Đọc thêm về [WebAssembly](/doc/go1.24#wasm) trong ghi chú phát hành.

---

Hãy đọc [ghi chú phát hành Go 1.24](/doc/go1.24) để biết thông tin đầy đủ và
chi tiết. Đừng quên theo dõi các bài đăng blog tiếp theo sẽ đi sâu hơn
về một số chủ đề được đề cập ở đây!

Cảm ơn tất cả những người đã đóng góp cho bản phát hành này bằng cách viết code và
tài liệu, báo cáo lỗi, chia sẻ phản hồi, và kiểm thử các release candidate.
Nỗ lực của bạn giúp đảm bảo Go 1.24 ổn định nhất có thể.
Như thường lệ, nếu bạn phát hiện bất kỳ vấn đề nào, hãy [tạo issue](/issue/new).

Tận hưởng Go 1.24!
