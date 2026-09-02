---
title: "Go Protobuf: API Opaque mới"
date: 2024-12-16
by:
- Michael Stapelberg
tags:
- protobuf
summary: Chúng tôi đang thêm một API mới cho code được tạo ra trong Go Protobuf.
---

[[Protocol Buffers (Protobuf)](https://en.wikipedia.org/wiki/Protocol_Buffers)
là định dạng trao đổi dữ liệu trung lập với ngôn ngữ của Google. Xem
[protobuf.dev](https://protobuf.dev/).]

Vào tháng 3 năm 2020, chúng tôi đã phát hành module `google.golang.org/protobuf`, [một
bản cải tổ lớn của API Go Protobuf](/blog/protobuf-apiv2). Gói này giới thiệu
hỗ trợ hạng nhất cho
[reflection](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect),
cài đặt [`dynamicpb`](https://pkg.go.dev/google.golang.org/protobuf/types/dynamicpb)
và gói
[`protocmp`](https://pkg.go.dev/google.golang.org/protobuf/testing/protocmp)
để kiểm thử dễ dàng hơn.

Bản phát hành đó đã giới thiệu một module protobuf mới với một API mới. Hôm nay,
chúng tôi phát hành thêm một API cho code được tạo ra, tức là code Go trong các
tệp `.pb.go` được tạo ra bởi trình biên dịch protocol (`protoc`). Bài đăng này
giải thích lý do chúng tôi tạo ra một API mới và hướng dẫn bạn cách sử dụng
nó trong các dự án của mình.

Để rõ ràng: chúng tôi không xóa bất cứ thứ gì. Chúng tôi sẽ tiếp tục hỗ trợ
API hiện có cho code được tạo ra, giống như chúng tôi vẫn hỗ trợ module
protobuf cũ hơn (bằng cách bọc cài đặt `google.golang.org/protobuf`). Go
[cam kết duy trì tính tương thích ngược](/blog/compat) và điều này
cũng áp dụng cho Go Protobuf!

## Nền tảng: API Open Struct (hiện có) {#background}

Bây giờ chúng tôi gọi API hiện có là API Open Struct, vì các kiểu struct được
tạo ra đều mở để truy cập trực tiếp. Trong phần tiếp theo, chúng ta sẽ thấy nó
khác với API Opaque mới như thế nào.

Để làm việc với protocol buffers, trước tiên bạn tạo một tệp định nghĩa `.proto`
như thế này:

    edition = "2023";  // successor to proto2 and proto3

    package log;

    message LogEntry {
      string backend_server = 1;
      uint32 request_size = 2;
      string ip_address = 3;
    }

Sau đó, bạn [chạy trình biên dịch protocol
(`protoc`)](https://protobuf.dev/getting-started/gotutorial/) để tạo ra code
như sau (trong một tệp `.pb.go`):

    package logpb

    type LogEntry struct {
      BackendServer *string
      RequestSize   *uint32
      IPAddress     *string
      // …internal fields elided…
    }

    func (l *LogEntry) GetBackendServer() string { … }
    func (l *LogEntry) GetRequestSize() uint32   { … }
    func (l *LogEntry) GetIPAddress() string     { … }

Bây giờ bạn có thể import gói `logpb` được tạo ra từ code Go của mình và gọi
các hàm như
[`proto.Marshal`](https://pkg.go.dev/google.golang.org/protobuf/proto#Marshal)
để mã hóa các thông điệp `logpb.LogEntry` sang định dạng wire protobuf.

Bạn có thể tìm thêm chi tiết trong [tài liệu API Code được tạo ra
](https://protobuf.dev/reference/go/go-generated/).

### API Open Struct (hiện có): Sự hiện diện của trường {#presence}

Một khía cạnh quan trọng của code được tạo ra này là cách *sự hiện diện của
trường* (trường có được đặt hay không) được mô hình hóa. Chẳng hạn, ví dụ trên
mô hình hóa sự hiện diện bằng con trỏ, vì vậy bạn có thể đặt trường
`BackendServer` thành:

1. `proto.String("zrh01.prod")`: trường được đặt và chứa "zrh01.prod"
1. `proto.String("")`: trường được đặt (con trỏ không phải `nil`) nhưng chứa giá
   trị rỗng
1. Con trỏ `nil`: trường không được đặt

Nếu bạn quen với code được tạo ra không có con trỏ, thì bạn có thể đang sử
dụng các tệp `.proto` bắt đầu bằng `syntax = "proto3"`. Hành vi của sự hiện
diện trường đã thay đổi qua các năm:

* `syntax = "proto2"` sử dụng *explicit presence* theo mặc định
* `syntax = "proto3"` sử dụng *implicit presence* theo mặc định (trong đó trường
  hợp 2 và 3 không thể phân biệt được và đều được biểu diễn bằng chuỗi rỗng),
  nhưng sau đó đã được mở rộng để cho phép [chọn explicit presence với từ khóa
  `optional`](https://protobuf.dev/programming-guides/proto3/#field-labels)
* `edition = "2023"`, [người kế thừa của cả proto2 và
  proto3](https://protobuf.dev/editions/overview/), sử dụng [*explicit
  presence*](https://protobuf.dev/programming-guides/field_presence/) theo mặc
  định

## API Opaque mới {#opaqueapi}

Chúng tôi đã tạo ra *API Opaque* mới để tách rời [API Code được tạo ra
](https://protobuf.dev/reference/go/go-generated/) khỏi biểu diễn trong bộ
nhớ bên dưới. API Open Struct (hiện có) không có sự tách biệt như vậy: nó cho
phép các chương trình truy cập trực tiếp vào bộ nhớ thông điệp protobuf. Ví
dụ, có thể sử dụng gói `flag` để phân tích các giá trị cờ dòng lệnh vào các
trường thông điệp protobuf:

    var req logpb.LogEntry
    flag.StringVar(&req.BackendServer, "backend", os.Getenv("HOST"), "…")
    flag.Parse() // fills the BackendServer field from -backend flag

Vấn đề với sự kết hợp chặt chẽ như vậy là chúng ta không bao giờ có thể thay
đổi cách chúng ta bố trí các thông điệp protobuf trong bộ nhớ. Loại bỏ hạn chế
này cho phép nhiều cải tiến về cài đặt, mà chúng ta sẽ thấy bên dưới.

Điều gì thay đổi với API Opaque mới? Đây là cách code được tạo ra từ ví dụ
trên sẽ thay đổi:

    package logpb

    type LogEntry struct {
      xxx_hidden_BackendServer *string // no longer exported
      xxx_hidden_RequestSize   uint32  // no longer exported
      xxx_hidden_IPAddress     *string // no longer exported
      // …internal fields elided…
    }

    func (l *LogEntry) GetBackendServer() string { … }
    func (l *LogEntry) HasBackendServer() bool   { … }
    func (l *LogEntry) SetBackendServer(string)  { … }
    func (l *LogEntry) ClearBackendServer()      { … }
    // …

Với API Opaque, các trường struct bị ẩn và không còn có thể truy cập trực
tiếp. Thay vào đó, các phương thức accessor mới cho phép lấy, đặt hoặc xóa
một trường.

### Struct opaque dùng ít bộ nhớ hơn {#lessmemory}

Một thay đổi chúng tôi thực hiện đối với bố cục bộ nhớ là mô hình hóa sự
hiện diện của trường cho các trường sơ khai hiệu quả hơn:

* API Open Struct (hiện có) sử dụng con trỏ, điều này thêm một từ 64-bit vào
  chi phí không gian của trường.
* API Opaque sử dụng [bit
  fields](https://en.wikipedia.org/wiki/Bit_field), yêu cầu một bit mỗi trường
  (bỏ qua chi phí căn chỉnh).

Sử dụng ít biến và con trỏ hơn cũng giảm tải cho bộ cấp phát và bộ gom rác.

Cải tiến hiệu năng phụ thuộc nhiều vào hình dạng của các thông điệp protocol
của bạn: Thay đổi chỉ ảnh hưởng đến các trường sơ khai như số nguyên, bool,
enum và float, nhưng không ảnh hưởng đến chuỗi, trường repeated hoặc các
submessage (vì nó
[ít có lợi hơn](https://protobuf.dev/reference/go/opaque-faq/#memorylayout)
cho những kiểu đó).

Kết quả benchmark của chúng tôi cho thấy các thông điệp với ít trường sơ khai
có hiệu năng tốt như trước, trong khi các thông điệp với nhiều trường sơ khai
được giải mã với số lần cấp phát giảm đáng kể:

             │ Open Struct API │             Opaque API             │
             │    allocs/op    │  allocs/op   vs base               │
Prod#1          360.3k ± 0%       360.3k ± 0%  +0.00% (p=0.002 n=6)
Search#1       1413.7k ± 0%       762.3k ± 0%  -46.08% (p=0.002 n=6)
Search#2        314.8k ± 0%       132.4k ± 0%  -57.95% (p=0.002 n=6)

Giảm số lần cấp phát cũng làm cho việc giải mã các thông điệp protobuf hiệu
quả hơn:

             │ Open Struct API │             Opaque API            │
             │   user-sec/op   │ user-sec/op  vs base              │
Prod#1         55.55m ± 6%        55.28m ± 4%  ~ (p=0.180 n=6)
Search#1       324.3m ± 22%       292.0m ± 6%  -9.97% (p=0.015 n=6)
Search#2       67.53m ± 10%       45.04m ± 8%  -33.29% (p=0.002 n=6)

(Tất cả các phép đo được thực hiện trên AMD Castle Peak Zen 2. Kết quả trên
CPU ARM và Intel tương tự.)

Lưu ý: proto3 với implicit presence tương tự cũng không sử dụng con trỏ, vì
vậy bạn sẽ không thấy cải tiến hiệu năng nếu bạn đang dùng proto3. Nếu bạn
đã sử dụng implicit presence vì lý do hiệu năng, từ bỏ sự tiện lợi của việc
phân biệt trường rỗng với trường chưa đặt, thì API Opaque nay cho phép sử
dụng explicit presence mà không bị phạt về hiệu năng.

### Động lực: Lazy Decoding {#lazydecoding}

Lazy decoding là một tối ưu hóa hiệu năng trong đó nội dung của một submessage
được giải mã khi truy cập lần đầu thay vì trong quá trình
[`proto.Unmarshal`](https://pkg.go.dev/google.golang.org/protobuf/proto#Unmarshal).
Lazy decoding có thể cải thiện hiệu năng bằng cách tránh giải mã không cần
thiết các trường không bao giờ được truy cập.

Lazy decoding không thể được hỗ trợ an toàn bởi API Open Struct (hiện có). Mặc
dù API Open Struct cung cấp các getter, việc để lộ các trường struct (chưa được
giải mã) sẽ rất dễ gây lỗi. Để đảm bảo rằng logic giải mã chạy ngay trước khi
trường được truy cập lần đầu, chúng ta phải làm cho trường đó private và giám
sát tất cả quyền truy cập vào nó thông qua các hàm getter và setter.

Cách tiếp cận này giúp có thể cài đặt lazy decoding với API Opaque. Tất nhiên,
không phải mọi workload đều được hưởng lợi từ tối ưu hóa này, nhưng đối với
những workload có lợi, kết quả có thể rất ấn tượng: Chúng tôi đã thấy các
pipeline phân tích log loại bỏ các thông điệp dựa trên điều kiện thông điệp
cấp cao nhất (ví dụ: liệu `backend_server` có phải là một trong các máy chạy
phiên bản Linux kernel mới hay không) và có thể bỏ qua việc giải mã các cây
con thông điệp lồng sâu.

Là một ví dụ, đây là kết quả của micro-benchmark mà chúng tôi đã đưa vào,
chứng minh rằng lazy decoding tiết kiệm hơn 50% công việc và hơn 87% số lần
cấp phát!

                  │   nolazy    │                lazy                │
                  │   sec/op    │   sec/op     vs base               │
Unmarshal/lazy-24   6.742µ ± 0%   2.816µ ± 0%  -58.23% (p=0.002 n=6)

                  │    nolazy    │                lazy                 │
                  │     B/op     │     B/op      vs base               │
Unmarshal/lazy-24   3.666Ki ± 0%   1.814Ki ± 0%  -50.51% (p=0.002 n=6)

                  │   nolazy    │               lazy                │
                  │  allocs/op  │ allocs/op   vs base               │
Unmarshal/lazy-24   64.000 ± 0%   8.000 ± 0%  -87.50% (p=0.002 n=6)


### Động lực: Giảm lỗi so sánh con trỏ {#pointercomparison}

Mô hình hóa sự hiện diện của trường với con trỏ dẫn đến các lỗi liên quan
đến con trỏ.

Xem xét một enum, được khai báo trong thông điệp `LogEntry`:

    message LogEntry {
      enum DeviceType {
        DESKTOP = 0;
        MOBILE = 1;
        VR = 2;
      };
      DeviceType device_type = 1;
    }

Một lỗi đơn giản là so sánh trường enum `device_type` như sau:

    if cv.DeviceType == logpb.LogEntry_DESKTOP.Enum() { // incorrect!

Bạn có phát hiện ra lỗi không? Điều kiện so sánh địa chỉ bộ nhớ thay vì giá
trị. Vì accessor `Enum()` cấp phát một biến mới mỗi lần gọi, điều kiện này
không bao giờ có thể đúng. Phép kiểm tra nên là:

    if cv.GetDeviceType() == logpb.LogEntry_DESKTOP {

API Opaque mới ngăn chặn lỗi này: Vì các trường bị ẩn, tất cả quyền truy cập
phải đi qua getter.

### Động lực: Giảm lỗi chia sẻ vô tình {#accidentalsharing}

Hãy xem xét một lỗi liên quan đến con trỏ phức tạp hơn một chút. Giả sử bạn
đang cố gắng ổn định một dịch vụ RPC bị lỗi dưới tải cao. Phần sau của
middleware yêu cầu trông có vẻ đúng, nhưng toàn bộ dịch vụ vẫn bị sập khi
chỉ một khách hàng gửi lượng yêu cầu cao:

	logEntry.IPAddress = req.IPAddress
	logEntry.BackendServer = proto.String(hostname)
	// The redactIP() function redacts IPAddress to 127.0.0.1,
	// unexpectedly not just in logEntry *but also* in req!
	go auditlog(redactIP(logEntry))
	if quotaExceeded(req) {
		// BUG: All requests end up here, regardless of their source.
		return fmt.Errorf("server overloaded")
	}

Bạn có phát hiện ra lỗi không? Dòng đầu tiên vô tình sao chép con trỏ (qua đó
chia sẻ biến được trỏ đến giữa các thông điệp `logEntry` và `req`) thay vì giá
trị của nó. Lẽ ra nên là:

	logEntry.IPAddress = proto.String(req.GetIPAddress())

API Opaque mới ngăn chặn vấn đề này vì setter nhận một giá trị (`string`) thay
vì một con trỏ:

	logEntry.SetIPAddress(req.GetIPAddress())


### Động lực: Khắc phục điểm nhạy cảm: reflection {#reflection}

Để viết code không chỉ hoạt động với một kiểu thông điệp cụ thể
(ví dụ: `logpb.LogEntry`), mà với bất kỳ kiểu thông điệp nào, cần có một
dạng reflection nào đó. Ví dụ trước đã sử dụng một hàm để biên tập địa chỉ
IP. Để hoạt động với bất kỳ kiểu thông điệp nào, nó có thể được định nghĩa
là `func redactIP(proto.Message) proto.Message { … }`.

Nhiều năm trước, lựa chọn duy nhất để cài đặt một hàm như `redactIP` là sử dụng
[gói `reflect` của Go](/blog/laws-of-reflection),
dẫn đến sự kết hợp rất chặt chẽ: bạn chỉ có đầu ra của generator và phải
đảo ngược để suy ra định nghĩa thông điệp protobuf đầu vào có thể là gì như
thế nào. [Bản phát hành module `google.golang.org/protobuf`
](/blog/protobuf-apiv2) (từ tháng 3 năm 2020) đã giới thiệu
[Protobuf reflection](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect),
vốn luôn nên được ưu tiên hơn: Gói `reflect` của Go duyệt qua biểu diễn của
cấu trúc dữ liệu, vốn nên là chi tiết cài đặt. Protobuf reflection duyệt qua
cây logic của các thông điệp protocol mà không phụ thuộc vào biểu diễn của nó.

Thật không may, chỉ *cung cấp* protobuf reflection là chưa đủ và vẫn để lộ
một số điểm nhạy cảm: Trong một số trường hợp, người dùng có thể vô tình sử
dụng Go reflection thay vì protobuf reflection.

Ví dụ, mã hóa một thông điệp protobuf với gói `encoding/json` (sử dụng Go
reflection) về mặt kỹ thuật là khả thi, nhưng kết quả không phải là [mã hóa
JSON Protobuf chuẩn](https://protobuf.dev/programming-guides/proto3/#json).
Hãy sử dụng gói
[`protojson`](https://pkg.go.dev/google.golang.org/protobuf/encoding/protojson)
thay thế.

API Opaque mới ngăn chặn vấn đề này vì các trường struct thông điệp bị ẩn:
việc vô tình sử dụng Go reflection sẽ thấy một thông điệp rỗng. Điều này đủ
rõ ràng để hướng các nhà phát triển đến protobuf reflection.

### Động lực: Làm cho bố cục bộ nhớ lý tưởng trở nên khả thi {#idealmemory}

Kết quả benchmark từ phần [Biểu diễn bộ nhớ hiệu quả hơn](#lessmemory) đã
cho thấy hiệu năng protobuf phụ thuộc nhiều vào cách sử dụng cụ thể: Các thông
điệp được định nghĩa như thế nào? Các trường nào được đặt?

Để giữ Go Protobuf nhanh nhất có thể cho *mọi người*, chúng tôi không thể cài
đặt các tối ưu hóa chỉ giúp một chương trình nhưng lại làm giảm hiệu năng
của các chương trình khác.

Trình biên dịch Go trước đây cũng ở trong tình trạng tương tự, cho đến khi
[Go 1.20 giới thiệu Profile-Guided Optimization (PGO)](/blog/go1.20). Bằng
cách ghi lại hành vi production (thông qua [profiling](/blog/pprof)) và
đưa profile đó trở lại cho trình biên dịch, chúng ta cho phép trình biên dịch
đưa ra các đánh đổi tốt hơn *cho một chương trình hoặc workload cụ thể*.

Chúng tôi cho rằng sử dụng profile để tối ưu hóa cho các workload cụ thể là
một hướng tiếp cận hứa hẹn cho các tối ưu hóa Go Protobuf tiếp theo. API Opaque
làm cho những điều đó trở nên khả thi: Code chương trình sử dụng accessor và
không cần cập nhật khi biểu diễn bộ nhớ thay đổi, vì vậy chúng tôi có thể, ví
dụ, chuyển các trường ít khi được đặt vào một struct overflow.

## Di chuyển {#migration}

Bạn có thể di chuyển theo lịch trình của riêng mình, hoặc thậm chí không di
chuyển chút nào vì API Open Struct (hiện có) sẽ không bị xóa. Tuy nhiên, nếu
bạn không dùng API Opaque mới, bạn sẽ không được hưởng lợi từ hiệu năng cải
thiện hay các tối ưu hóa trong tương lai nhắm vào nó.

Chúng tôi khuyên bạn nên chọn API Opaque cho phát triển mới. Protobuf Edition
2024 (xem [Tổng quan về Protobuf Editions](https://protobuf.dev/editions/overview/)
nếu bạn chưa quen) sẽ làm cho API Opaque trở thành mặc định.

### API Hybrid {#hybridapi}

Ngoài API Open Struct và API Opaque, còn có API Hybrid, vốn giữ cho code hiện
có hoạt động bằng cách giữ các trường struct được export, nhưng cũng hỗ trợ
di chuyển sang API Opaque bằng cách thêm các phương thức accessor mới.

Với API Hybrid, trình biên dịch protobuf sẽ tạo ra code ở hai cấp độ API: tệp
`.pb.go` ở API Hybrid, trong khi phiên bản `_protoopaque.pb.go` ở API Opaque
và có thể được chọn bằng cách build với build tag `protoopaque`.

### Viết lại code sang API Opaque {#rewriting}

Xem [hướng dẫn di chuyển
](https://protobuf.dev/reference/go/opaque-migration/)
để biết hướng dẫn chi tiết. Các bước cấp cao là:

1. Bật API Hybrid.
1. Cập nhật code hiện có bằng công cụ di chuyển `open2opaque`.
1. Chuyển sang API Opaque.

### Khuyến nghị cho code được tạo ra đã công bố: Dùng API Hybrid {#publishing}

Các trường hợp sử dụng protobuf nhỏ có thể nằm hoàn toàn trong cùng một kho
lưu trữ, nhưng thường thì các tệp `.proto` được chia sẻ giữa các dự án khác
nhau thuộc sở hữu của các nhóm khác nhau. Một ví dụ rõ ràng là khi các công
ty khác nhau liên quan: Để gọi Google APIs (với protobuf), hãy sử dụng
[Google Cloud Client Libraries for Go](https://github.com/googleapis/google-cloud-go)
từ dự án của bạn. Chuyển Cloud Client Libraries sang API Opaque không phải là
một tùy chọn vì đó sẽ là thay đổi API breaking, nhưng chuyển sang API Hybrid
thì an toàn.

Lời khuyên của chúng tôi cho các gói như vậy công bố code được tạo ra (tệp
`.pb.go`) là hãy chuyển sang API Hybrid! Hãy công bố cả tệp `.pb.go` và
`_protoopaque.pb.go`. Phiên bản `protoopaque` cho phép người dùng di chuyển
theo lịch trình của riêng họ.

### Bật Lazy Decoding {#enablelazy}

Lazy decoding có sẵn (nhưng chưa được bật) sau khi bạn di chuyển sang API Opaque!

Để bật: trong tệp `.proto` của bạn, hãy chú thích các trường có kiểu thông
điệp với chú thích `[lazy = true]`.

Để từ chối lazy decoding (dù có chú thích `.proto`), [tài liệu gói `protolazy`
](https://pkg.go.dev/google.golang.org/protobuf/runtime/protolazy)
mô tả các tùy chọn từ chối có sẵn, ảnh hưởng đến một thao tác Unmarshal cụ
thể hoặc toàn bộ chương trình.

## Các bước tiếp theo {#nextsteps}

Bằng cách sử dụng công cụ open2opaque theo cách tự động trong vài năm qua,
chúng tôi đã chuyển đổi phần lớn các tệp `.proto` và code Go của Google sang
API Opaque. Chúng tôi liên tục cải thiện cài đặt API Opaque khi chuyển ngày
càng nhiều workload production sang nó.

Do đó, chúng tôi kỳ vọng bạn sẽ không gặp vấn đề khi thử API Opaque. Trong
trường hợp bạn gặp bất kỳ sự cố nào, hãy [cho chúng tôi biết trên Go Protobuf
issue tracker](https://github.com/golang/protobuf/issues/).

Tài liệu tham khảo cho Go Protobuf có thể tìm thấy tại [protobuf.dev → Go
Reference](https://protobuf.dev/reference/go/).
