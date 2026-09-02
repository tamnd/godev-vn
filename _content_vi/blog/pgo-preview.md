---
title: Xem trước tối ưu hóa dựa trên profile
date: 2023-02-08
by:
- Michael Pratt
summary: Giới thiệu về tối ưu hóa dựa trên profile, có sẵn dưới dạng xem trước trong Go 1.20.
---

Khi bạn build một binary Go, trình biên dịch Go thực hiện các tối ưu hóa để cố gắng tạo ra binary hoạt động tốt nhất có thể.
Ví dụ, lan truyền hằng số có thể đánh giá các biểu thức hằng số tại thời điểm biên dịch, tránh chi phí đánh giá tại runtime.
Phân tích escape tránh cấp phát heap cho các đối tượng có phạm vi cục bộ, tránh overhead của GC.
Inlining sao chép thân hàm của các hàm đơn giản vào nơi gọi, thường cho phép tối ưu hóa thêm tại nơi gọi (như lan truyền hằng số bổ sung hoặc phân tích escape tốt hơn).

Go cải thiện các tối ưu hóa từ bản phát hành này sang bản phát hành khác, nhưng đây không phải lúc nào cũng là nhiệm vụ dễ dàng.
Một số tối ưu hóa có thể điều chỉnh, nhưng trình biên dịch không thể chỉ "tăng hết mức" trên mọi hàm vì các tối ưu hóa quá tích cực thực sự có thể gây hại cho hiệu năng hoặc gây ra thời gian build quá lâu.
Các tối ưu hóa khác yêu cầu trình biên dịch đưa ra phán đoán về đường dẫn "phổ biến" và "không phổ biến" trong một hàm là gì.
Trình biên dịch phải đưa ra phỏng đoán tốt nhất dựa trên các heuristic tĩnh vì nó không thể biết các trường hợp nào sẽ phổ biến tại thời điểm chạy.

Hay có thể biết được không?

Không có thông tin xác định về cách code được sử dụng trong môi trường production, trình biên dịch chỉ có thể hoạt động trên source code của các package.
Nhưng chúng ta có một công cụ để đánh giá hành vi production: [profiling](/doc/diagnostics#profiling).
Nếu chúng ta cung cấp một profile cho trình biên dịch, nó có thể đưa ra các quyết định sáng suốt hơn: tối ưu hóa tích cực hơn các hàm được sử dụng thường xuyên nhất, hoặc chọn các trường hợp phổ biến chính xác hơn.

Sử dụng các profile về hành vi ứng dụng để tối ưu hóa trình biên dịch được gọi là _Tối ưu hóa dựa trên hồ sơ thực thi (PGO)_ (còn được gọi là Tối ưu hóa dựa trên Phản hồi (FDO)).

Go 1.20 bao gồm hỗ trợ ban đầu cho PGO dưới dạng xem trước.
Xem [hướng dẫn người dùng tối ưu hóa dựa trên hồ sơ thực thi](/doc/pgo) để có tài liệu đầy đủ.
Vẫn còn một số điểm thô mà có thể ngăn cản việc sử dụng trong môi trường production, nhưng chúng tôi rất muốn bạn thử và [gửi cho chúng tôi bất kỳ phản hồi hoặc vấn đề nào bạn gặp phải](/issue/new).

## Ví dụ

Hãy xây dựng một dịch vụ chuyển đổi Markdown sang HTML: người dùng tải Markdown nguồn lên `/render`, trả về kết quả chuyển đổi HTML.
Chúng ta có thể sử dụng [`gitlab.com/golang-commonmark/markdown`](https://pkg.go.dev/gitlab.com/golang-commonmark/markdown) để triển khai điều này một cách dễ dàng.

### Thiết lập

```
$ go mod init example.com/markdown
$ go get gitlab.com/golang-commonmark/markdown@bf3e522c626a
```

Trong `main.go`:

```
package main

import (
	"bytes"
	"io"
	"log"
	"net/http"
	_ "net/http/pprof"

	"gitlab.com/golang-commonmark/markdown"
)

func render(w http.ResponseWriter, r *http.Request) {
	if r.Method != "POST" {
		http.Error(w, "Only POST allowed", http.StatusMethodNotAllowed)
		return
	}

	src, err := io.ReadAll(r.Body)
	if err != nil {
		log.Printf("error reading body: %v", err)
		http.Error(w, "Internal Server Error", http.StatusInternalServerError)
		return
	}

	md := markdown.New(
		markdown.XHTMLOutput(true),
		markdown.Typographer(true),
		markdown.Linkify(true),
		markdown.Tables(true),
	)

	var buf bytes.Buffer
	if err := md.Render(&buf, src); err != nil {
		log.Printf("error converting markdown: %v", err)
		http.Error(w, "Malformed markdown", http.StatusBadRequest)
		return
	}

	if _, err := io.Copy(w, &buf); err != nil {
		log.Printf("error writing response: %v", err)
		http.Error(w, "Internal Server Error", http.StatusInternalServerError)
		return
	}
}

func main() {
	http.HandleFunc("/render", render)
	log.Printf("Serving on port 8080...")
	log.Fatal(http.ListenAndServe(":8080", nil))
}

```

Build và chạy server:

```
$ go build -o markdown.nopgo.exe
$ ./markdown.nopgo.exe
2023/01/19 14:26:24 Serving on port 8080...
```

Hãy thử gửi một số Markdown từ terminal khác.
Chúng ta có thể sử dụng README từ dự án Go làm tài liệu mẫu:

```
$ curl -o README.md -L "https://raw.githubusercontent.com/golang/go/c16c2c49e2fa98ae551fc6335215fadd62d33542/README.md"
$ curl --data-binary @README.md http://localhost:8080/render
<h1>The Go Programming Language</h1>
<p>Go is an open source programming language that makes it easy to build simple,
reliable, and efficient software.</p>
...
```

### Profiling

Bây giờ chúng ta có một dịch vụ hoạt động, hãy thu thập một profile và rebuild với PGO để xem liệu chúng ta có hiệu năng tốt hơn không.

Trong `main.go`, chúng ta đã import [net/http/pprof](https://pkg.go.dev/net/http/pprof) tự động thêm endpoint `/debug/pprof/profile` vào server để lấy CPU profile.

Thông thường bạn muốn thu thập profile từ môi trường production của mình để trình biên dịch nhận được cái nhìn đại diện về hành vi trong production.
Vì ví dụ này không có môi trường "production", chúng ta sẽ tạo một chương trình đơn giản để tạo tải trong khi chúng ta thu thập profile.
Sao chép nguồn của [chương trình này](/play/p/yYH0kfsZcpL) vào `load/main.go` và khởi động bộ tạo tải (đảm bảo server vẫn đang chạy!).

```
$ go run example.com/markdown/load
```

Trong khi đó đang chạy, tải xuống một profile từ server:

```
$ curl -o cpu.pprof "http://localhost:8080/debug/pprof/profile?seconds=30"
```

Khi hoàn thành, dừng bộ tạo tải và server.

### Sử dụng profile

Chúng ta có thể yêu cầu Go toolchain build với PGO bằng cờ `-pgo` cho `go build`.
`-pgo` nhận đường dẫn đến profile cần sử dụng, hoặc `auto`, sẽ sử dụng tệp `default.pgo` trong thư mục package chính.

Chúng tôi khuyến nghị commit các profile `default.pgo` vào repository của bạn.
Lưu trữ các profile cùng với source code đảm bảo người dùng tự động có quyền truy cập vào profile chỉ bằng cách lấy repository (qua hệ thống kiểm soát phiên bản, hoặc qua `go get`) và các build vẫn có thể tái tạo.
Trong Go 1.20, `-pgo=off` là mặc định, vì vậy người dùng vẫn cần thêm `-pgo=auto`, nhưng phiên bản tương lai của Go dự kiến sẽ thay đổi mặc định thành `-pgo=auto`, tự động cấp cho bất kỳ ai build binary lợi ích của PGO.

Hãy build:

```
$ mv cpu.pprof default.pgo
$ go build -pgo=auto -o markdown.withpgo.exe
```

### Đánh giá

Chúng ta sẽ sử dụng phiên bản benchmark Go của bộ tạo tải để đánh giá tác động của PGO lên hiệu năng.
Sao chép [benchmark này](/play/p/6FnQmHfRjbh) vào `load/bench_test.go`.

Đầu tiên, chúng ta sẽ benchmark server không có PGO. Khởi động server đó:

```
$ ./markdown.nopgo.exe
```

Trong khi đó đang chạy, chạy một số lần lặp benchmark:

```
$ go test example.com/markdown/load -bench=. -count=20 -source ../README.md > nopgo.txt
```

Khi hoàn thành, dừng server gốc và khởi động phiên bản có PGO:

```
$ ./markdown.withpgo.exe
```

Trong khi đó đang chạy, chạy một số lần lặp benchmark:

```
$ go test example.com/markdown/load -bench=. -count=20 -source ../README.md > withpgo.txt
```

Khi hoàn thành, hãy so sánh kết quả:

```
$ go install golang.org/x/perf/cmd/benchstat@latest
$ benchstat nopgo.txt withpgo.txt
goos: linux
goarch: amd64
pkg: example.com/markdown/load
cpu: Intel(R) Xeon(R) W-2135 CPU @ 3.70GHz
        │  nopgo.txt  │            withpgo.txt             │
        │   sec/op    │   sec/op     vs base               │
Load-12   393.8µ ± 1%   383.6µ ± 1%  -2.59% (p=0.000 n=20)
```

Phiên bản mới nhanh hơn khoảng 2,6%!
Trong Go 1.20, các workload thường đạt được cải thiện mức sử dụng CPU từ 2% đến 4% khi bật PGO.
Các profile chứa nhiều thông tin về hành vi ứng dụng và Go 1.20 chỉ mới bắt đầu khai thác bề mặt bằng cách sử dụng thông tin này để inlining.
Các bản phát hành tương lai sẽ tiếp tục cải thiện hiệu năng khi nhiều phần của trình biên dịch tận dụng PGO.

## Các bước tiếp theo

Trong ví dụ này, sau khi thu thập profile, chúng ta đã rebuild server sử dụng chính xác source code giống nhau được sử dụng trong build gốc.
Trong kịch bản thực tế, luôn có sự phát triển liên tục.
Vì vậy, chúng ta có thể thu thập profile từ production đang chạy code của tuần trước và sử dụng nó để build với source code ngày hôm nay.
Điều đó hoàn toàn ổn!
PGO trong Go có thể xử lý các thay đổi nhỏ đối với source code mà không gặp vấn đề gì.

Để biết thêm nhiều thông tin về việc sử dụng PGO, các thực hành tốt nhất và những điều cần lưu ý, hãy xem [hướng dẫn người dùng tối ưu hóa dựa trên hồ sơ thực thi](/doc/pgo).

Hãy gửi cho chúng tôi phản hồi của bạn!
PGO vẫn đang trong giai đoạn xem trước và chúng tôi rất muốn nghe về bất cứ điều gì khó sử dụng, không hoạt động đúng, v.v.
Vui lòng nộp issue tại [go.dev/issue/new](/issue/new).

## Lời cảm ơn

Thêm tối ưu hóa dựa trên hồ sơ thực thi vào Go là nỗ lực của cả nhóm, và tôi đặc biệt muốn nêu bật những đóng góp từ Raj Barik và Jin Lin tại Uber, và Cherry Mui và Austin Clements tại Google.
Loại hợp tác liên cộng đồng này là phần quan trọng để làm cho Go trở nên tuyệt vời.
