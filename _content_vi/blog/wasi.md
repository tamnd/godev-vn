---
title: Hỗ trợ WASI trong Go
date: 2023-09-13
by:
- Johan Brandhorst-Satzkorn, Julien Fabre, Damian Gryski, Evan Phoenix, and Achille Roussel
summary: Go 1.21 bổ sung một port mới nhắm vào API syscall WASI preview 1
---

Go 1.21 bổ sung một port mới nhắm vào API syscall WASI preview 1 thông qua giá trị `GOOS` mới là `wasip1`. Port này được xây dựng dựa trên port WebAssembly hiện có được giới thiệu trong Go 1.11.

## WebAssembly là gì?

[WebAssembly (Wasm)](https://webassembly.org/) là một định dạng lệnh nhị phân ban đầu được thiết kế cho web. Nó đại diện cho một tiêu chuẩn cho phép các nhà phát triển chạy code hiệu suất cao, cấp thấp trực tiếp trong trình duyệt web với tốc độ gần bằng native.

Go lần đầu thêm hỗ trợ biên dịch sang Wasm trong phiên bản 1.11, thông qua port `js/wasm`. Điều này cho phép code Go được biên dịch bằng trình biên dịch Go được thực thi trong trình duyệt web, nhưng nó yêu cầu môi trường thực thi JavaScript.

Khi việc sử dụng Wasm ngày càng tăng, các trường hợp sử dụng ngoài trình duyệt cũng tăng theo. Nhiều nhà cung cấp đám mây hiện đang cung cấp các dịch vụ cho phép người dùng thực thi các file thực thi Wasm trực tiếp, tận dụng API syscall [WebAssembly System Interface (WASI)](https://wasi.dev/) mới.

## WebAssembly System Interface

WASI định nghĩa API syscall cho các file thực thi Wasm, cho phép chúng tương tác với các tài nguyên hệ thống như hệ thống tệp, đồng hồ hệ thống, các tiện ích dữ liệu ngẫu nhiên và hơn thế nữa. Phiên bản mới nhất của đặc tả WASI được gọi là `wasi_snapshot_preview1`, từ đó chúng ta có tên `GOOS` là `wasip1`. Các phiên bản mới của API đang được phát triển, và việc hỗ trợ chúng trong trình biên dịch Go trong tương lai có thể sẽ đồng nghĩa với việc thêm `GOOS` mới.

Việc tạo ra WASI đã cho phép một số Wasm runtime (host) chuẩn hóa API syscall của họ xung quanh nó. Các ví dụ về Wasm/WASI host bao gồm [Wasmtime](https://wasmtime.dev), [Wazero](https://wazero.io/), [WasmEdge](https://wasmedge.org/), [Wasmer](https://wasmer.io/) và [NodeJS](https://nodejs.org). Cũng có nhiều nhà cung cấp đám mây cung cấp dịch vụ host các file thực thi Wasm/WASI.

## Làm thế nào để sử dụng với Go?

Đảm bảo rằng bạn đã cài đặt ít nhất phiên bản 1.21 của Go. Cho bài demo này, chúng ta sẽ dùng [Wasmtime host](https://docs.wasmtime.dev/cli-install.html) để thực thi binary. Hãy bắt đầu với một `main.go` đơn giản:

```go
package main

import "fmt"

func main() {
	fmt.Println("Hello world!")
}
```

Chúng ta có thể build nó cho `wasip1` bằng lệnh:

```shell
$ GOOS=wasip1 GOARCH=wasm go build -o main.wasm main.go
```

Điều này sẽ tạo ra một tệp `main.wasm` mà chúng ta có thể thực thi bằng `wasmtime`:

```shell
$ wasmtime main.wasm
Hello world!
```

Đó là tất cả những gì cần làm để bắt đầu với Wasm/WASI! Bạn có thể mong đợi hầu hết các tính năng của Go hoạt động bình thường với `wasip1`. Để tìm hiểu thêm về chi tiết cách WASI hoạt động với Go, hãy xem [đề xuất](/issue/58141).

## Chạy go test với wasip1

> Go 1.24 đã chuyển các tệp hỗ trợ Wasm sang `lib/wasm`. Với Go 1.21 - 1.23, hãy dùng thư mục `misc/wasm`.

Build và chạy một binary thì dễ dàng, nhưng đôi khi chúng ta muốn có thể chạy `go test` trực tiếp mà không cần build và thực thi binary thủ công. Tương tự như port `js/wasm`, bản phân phối thư viện chuẩn trong bản cài đặt Go của bạn đi kèm với một tệp giúp điều này rất dễ dàng. Thêm thư mục `lib/wasm` vào `PATH` của bạn khi chạy Go test và nó sẽ chạy test bằng Wasm host mà bạn chọn. Điều này hoạt động bằng cách `go test` [tự động thực thi](https://pkg.go.dev/cmd/go#hdr-Compile_and_run_Go_program) `lib/wasm/go_wasip1_wasm_exec` khi tìm thấy tệp này trong `PATH`.

```shell
$ export PATH=$PATH:$(go env GOROOT)/lib/wasm
$ GOOS=wasip1 GOARCH=wasm go test ./...
```

Điều này sẽ chạy `go test` bằng Wasmtime. Wasm host được sử dụng có thể được kiểm soát bằng biến môi trường `GOWASIRUNTIME`. Các giá trị hiện được hỗ trợ cho biến này là `wazero`, `wasmedge`, `wasmtime` và `wasmer`. Script này có thể thay đổi không tương thích giữa các phiên bản Go. Lưu ý rằng binary `wasip1` của Go chưa thực thi hoàn hảo trên tất cả các host (xem [#59907](/issue/59907) và [#60097](/issue/60097)).

Tính năng này cũng hoạt động khi sử dụng `go run`:

```shell
$ GOOS=wasip1 GOARCH=wasm go run ./main.go
Hello world!
```

## Bọc hàm Wasm trong Go với go:wasmimport

Ngoài port `wasip1/wasm` mới, Go 1.21 giới thiệu một chỉ thị trình biên dịch mới: `go:wasmimport`. Nó hướng dẫn trình biên dịch dịch các lời gọi đến hàm được chú thích thành lời gọi đến hàm được chỉ định bởi tên module host và tên hàm. Chức năng trình biên dịch mới này là thứ cho phép chúng tôi định nghĩa API syscall `wasip1` trong Go để hỗ trợ port mới, nhưng nó không bị giới hạn chỉ dùng trong thư viện chuẩn.

Ví dụ, API syscall wasip1 định nghĩa [hàm `random_get`](https://github.com/WebAssembly/WASI/blob/a51a66df5b1db01cf9e873f5537bc5bd552cf770/legacy/preview1/docs.md#-random_getbuf-pointeru8-buf_len-size---result-errno), và nó được cung cấp cho thư viện chuẩn Go thông qua [một wrapper hàm](https://cs.opensource.google/go/go/+/refs/tags/go1.21.0:src/runtime/os_wasip1.go;l=73-75) được định nghĩa trong gói runtime. Nó trông như thế này:

```go
//go:wasmimport wasi_snapshot_preview1 random_get
//go:noescape
func random_get(buf unsafe.Pointer, bufLen size) errno
```

Wrapper hàm này sau đó được bọc trong [một hàm tiện dụng hơn](https://cs.opensource.google/go/go/+/refs/tags/go1.21.0:src/runtime/os_wasip1.go;l=183-187) để sử dụng trong thư viện chuẩn:

```go
func getRandomData(r []byte) {
	if random_get(unsafe.Pointer(&r[0]), size(len(r))) != 0 {
		throw("random_get failed")
	}
}
```

Theo cách này, người dùng có thể gọi `getRandomData` với một byte slice và nó sẽ cuối cùng đến hàm `random_get` được định nghĩa bởi host. Tương tự, người dùng có thể định nghĩa các wrapper của riêng họ cho các hàm host.

Để tìm hiểu thêm về sự phức tạp của việc bọc hàm Wasm trong Go, hãy xem [đề xuất `go:wasmimport`](/issue/59149).

## Hạn chế

Mặc dù port `wasip1` vượt qua tất cả test thư viện chuẩn, có một số hạn chế cơ bản đáng chú ý của kiến trúc Wasm có thể khiến người dùng ngạc nhiên.

Wasm là kiến trúc đơn luồng không có tính song song. Bộ lập lịch vẫn có thể lập lịch các goroutine để chạy đồng thời, và đầu vào/ra/lỗi chuẩn không bị chặn, vì vậy một goroutine có thể thực thi trong khi goroutine khác đọc hoặc ghi, nhưng bất kỳ lời gọi hàm host nào (như yêu cầu dữ liệu ngẫu nhiên sử dụng ví dụ ở trên) sẽ khiến tất cả goroutine bị chặn cho đến khi lời gọi hàm host trả về.

Một tính năng còn thiếu đáng chú ý trong API `wasip1` là cài đặt đầy đủ network socket. `wasip1` chỉ định nghĩa các hàm hoạt động trên các socket đã mở, khiến không thể hỗ trợ một số tính năng phổ biến nhất của thư viện chuẩn Go, như HTTP server. Các host như Wasmer và WasmEdge cài đặt các extension cho API `wasip1`, cho phép mở network socket. Mặc dù các extension này không được cài đặt bởi trình biên dịch Go, nhưng có một thư viện bên thứ ba, [`github.com/stealthrocket/net`](https://github.com/stealthrocket/net), sử dụng `go:wasmimport` để cho phép dùng `net.Dial` và `net.Listen` trên các Wasm host được hỗ trợ. Điều này cho phép tạo ra `net/http` server và các tính năng liên quan đến mạng khác khi sử dụng gói này.

## Tương lai của Wasm trong Go

Việc bổ sung port `wasip1/wasm` chỉ là khởi đầu của các khả năng Wasm mà chúng tôi muốn mang đến cho Go. Hãy để ý đến [issue tracker](https://github.com/golang/go/issues?q=is%3Aopen+is%3Aissue+label%3Aarch-wasm) để biết các đề xuất về xuất hàm Go sang Wasm (`go:wasmexport`), port 32-bit và tương thích với các phiên bản API WASI trong tương lai.

## Tham gia đóng góp

Nếu bạn đang thử nghiệm và muốn đóng góp cho Wasm và Go, hãy tham gia! Issue tracker của Go theo dõi tất cả công việc đang tiến hành và kênh #webassembly trên [Gophers Slack](https://invite.slack.golangbridge.org/) là nơi tuyệt vời để thảo luận về Go và WebAssembly. Chúng tôi mong muốn nghe từ bạn!
