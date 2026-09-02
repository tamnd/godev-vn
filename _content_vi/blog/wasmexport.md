---
title: Ứng dụng Wasm mở rộng với Go
date: 2025-02-13
by:
- Cherry Mui
summary: Go 1.24 nâng cao khả năng WebAssembly với xuất hàm và chế độ reactor
---

Go 1.24 nâng cao khả năng WebAssembly (Wasm) với việc bổ sung chỉ thị `go:wasmexport` và khả năng build reactor cho WebAssembly System Interface (WASI). Các tính năng này cho phép các nhà phát triển Go xuất hàm Go sang Wasm, tạo điều kiện tích hợp tốt hơn với Wasm host và mở rộng các khả năng cho ứng dụng Wasm dựa trên Go.

## WebAssembly và WebAssembly System Interface

[WebAssembly (Wasm)](https://webassembly.org/) là một định dạng lệnh nhị phân ban đầu được tạo ra cho trình duyệt web, cung cấp khả năng thực thi code hiệu suất cao, cấp thấp ở tốc độ gần bằng native. Kể từ đó, tính hữu ích của Wasm đã mở rộng, và nó hiện được sử dụng trong nhiều môi trường khác nhau ngoài trình duyệt. Đáng chú ý, các nhà cung cấp đám mây cung cấp các dịch vụ thực thi trực tiếp các file thực thi Wasm, tận dụng API syscall [WebAssembly System Interface (WASI)](https://wasi.dev/). WASI cho phép các file thực thi này tương tác với tài nguyên hệ thống.

Go lần đầu thêm hỗ trợ biên dịch sang Wasm trong phiên bản 1.11, thông qua port `js/wasm`. Go 1.21 bổ sung một port mới nhắm vào API syscall WASI preview 1 thông qua port `GOOS=wasip1` mới.

## Xuất hàm Go sang Wasm với `go:wasmexport`

Go 1.24 giới thiệu một chỉ thị trình biên dịch mới, `go:wasmexport`, cho phép các nhà phát triển xuất hàm Go để gọi từ bên ngoài module Wasm, thường từ một ứng dụng host chạy Wasm runtime. Chỉ thị này hướng dẫn trình biên dịch làm cho hàm được chú thích có sẵn như một [export](https://webassembly.github.io/spec/core/valid/modules.html?highlight=export#exports) Wasm trong binary Wasm kết quả.

Để sử dụng chỉ thị `go:wasmexport`, chỉ cần thêm nó vào định nghĩa hàm:

```
//go:wasmexport add
func add(a, b int32) int32 { return a + b }
```

Với điều này, module Wasm sẽ có một hàm được xuất tên `add` có thể được gọi từ host.

Điều này tương tự với [chỉ thị `export` của cgo](/cmd/cgo#hdr-C_references_to_Go), làm cho hàm có sẵn để gọi từ C, mặc dù `go:wasmexport` sử dụng cơ chế khác, đơn giản hơn.

## Build WASI Reactor

WASI reactor là một module WebAssembly hoạt động liên tục, và có thể được gọi nhiều lần để phản ứng với các sự kiện hoặc yêu cầu. Không giống như module "command", kết thúc sau khi hàm main của nó hoàn thành, một thực thể reactor vẫn còn sống sau khi khởi tạo, và các export của nó vẫn có thể truy cập.

Với Go 1.24, có thể build một WASI reactor với cờ build `-buildmode=c-shared`.

```
$ GOOS=wasip1 GOARCH=wasm go build -buildmode=c-shared -o reactor.wasm
```

Cờ build này báo hiệu cho linker không tạo hàm `_start` (điểm vào cho module command), mà thay vào đó tạo hàm `_initialize`, thực hiện khởi tạo runtime và gói, cùng với bất kỳ hàm được xuất nào và các dependency của chúng. Hàm `_initialize` phải được gọi trước bất kỳ hàm được xuất nào khác. Hàm `main` sẽ không được gọi tự động.

Để sử dụng một WASI reactor, ứng dụng host trước tiên khởi tạo nó bằng cách gọi `_initialize`, sau đó chỉ cần gọi các hàm được xuất. Đây là một ví dụ sử dụng [Wazero](https://wazero.io/), một cài đặt Wasm runtime dựa trên Go:

```
// Tạo Wasm runtime, thiết lập WASI.
r := wazero.NewRuntime(ctx)
defer r.Close(ctx)
wasi_snapshot_preview1.MustInstantiate(ctx, r)

// Cấu hình module để khởi tạo reactor.
config := wazero.NewModuleConfig().WithStartFunctions("_initialize")

// Khởi tạo module.
wasmModule, _ := r.InstantiateWithConfig(ctx, wasmFile, config)

// Gọi hàm được xuất.
fn := wasmModule.ExportedFunction("add")
var a, b int32 = 1, 2
res, _ := fn.Call(ctx, api.EncodeI32(a), api.EncodeI32(b))
c := api.DecodeI32(res[0])
fmt.Printf("add(%d, %d) = %d\n", a, b, c)

// Thực thể vẫn còn sống. Chúng ta có thể gọi hàm lại.
res, _ = fn.Call(ctx, api.EncodeI32(b), api.EncodeI32(c))
fmt.Printf("add(%d, %d) = %d\n", b, c, api.DecodeI32(res[0]))
```

Chỉ thị `go:wasmexport` và chế độ build reactor cho phép các ứng dụng được mở rộng bằng cách gọi vào code Wasm dựa trên Go. Điều này đặc biệt có giá trị đối với các ứng dụng đã áp dụng Wasm như một cơ chế plugin hoặc extension với các giao diện được định nghĩa rõ ràng. Bằng cách xuất hàm Go, các ứng dụng có thể tận dụng các module Wasm Go để cung cấp chức năng mà không cần biên dịch lại toàn bộ ứng dụng. Hơn nữa, việc build như một reactor đảm bảo rằng các hàm được xuất có thể được gọi nhiều lần mà không cần khởi tạo lại, làm cho nó phù hợp cho các ứng dụng hoặc dịch vụ chạy dài.

## Hỗ trợ kiểu phong phú giữa host và client

Go 1.24 cũng nới lỏng các ràng buộc về các kiểu có thể được sử dụng làm tham số đầu vào và kết quả với các hàm `go:wasmimport`. Ví dụ, có thể truyền một bool, một string, một con trỏ đến `int32`, hoặc một con trỏ đến một struct nhúng `structs.HostLayout` và chứa các kiểu trường được hỗ trợ (xem [tài liệu](/cmd/compile#hdr-WebAssembly_Directives) để biết chi tiết). Điều này cho phép các ứng dụng Wasm Go được viết theo cách tự nhiên và tiện dụng hơn, đồng thời loại bỏ một số chuyển đổi kiểu không cần thiết.

## Hạn chế

Mặc dù Go 1.24 đã có những cải tiến đáng kể về khả năng Wasm, vẫn còn một số hạn chế đáng chú ý.

Wasm là kiến trúc đơn luồng không có tính song song. Một hàm `go:wasmexport` có thể tạo ra các goroutine mới. Nhưng nếu một hàm tạo ra một goroutine nền, nó sẽ không tiếp tục thực thi khi hàm `go:wasmexport` trả về, cho đến khi gọi lại vào module Wasm dựa trên Go.

Mặc dù một số hạn chế về kiểu đã được nới lỏng trong Go 1.24, vẫn còn các hạn chế về các kiểu có thể được sử dụng với các hàm `go:wasmimport` và `go:wasmexport`. Do sự không khớp đáng tiếc giữa kiến trúc 64-bit của client và kiến trúc 32-bit của host, không thể truyền con trỏ trong bộ nhớ. Ví dụ, hàm `go:wasmimport` không thể nhận con trỏ đến struct chứa trường kiểu con trỏ.

## Kết luận

Việc bổ sung khả năng build WASI reactor và xuất hàm Go sang Wasm trong Go 1.24 đại diện cho một bước tiến đáng kể cho khả năng WebAssembly của Go. Các tính năng này trao quyền cho các nhà phát triển tạo ra các ứng dụng Wasm dựa trên Go linh hoạt và mạnh mẽ hơn, mở ra những khả năng mới cho Go trong hệ sinh thái Wasm.
