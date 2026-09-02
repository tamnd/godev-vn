---
title: API JSON thử nghiệm mới cho Go
date: 2025-09-09
by:
- Joe Tsai
- Daniel Martí
- Johan Brandhorst-Satzkorn
- Roger Peppe
- Chris Hines
- Damien Neil
tags:
- json
- technical
summary: Go 1.25 giới thiệu hỗ trợ thử nghiệm cho các package encoding/json/jsontext và encoding/json/v2.
---

## Giới thiệu

[JavaScript Object Notation (JSON)](https://datatracker.ietf.org/doc/html/rfc8259)
là một định dạng trao đổi dữ liệu đơn giản. Gần 15 năm trước,
chúng tôi đã viết về [hỗ trợ JSON trong Go](/blog/json),
giới thiệu khả năng serialize và deserialize các kiểu Go sang và từ dữ liệu JSON.
Kể từ đó, JSON đã trở thành định dạng dữ liệu phổ biến nhất được sử dụng trên Internet.
Nó được đọc và ghi rộng rãi bởi các chương trình Go,
và encoding/json hiện xếp hạng là package Go được import nhiều thứ 5.

Theo thời gian, các package phát triển theo nhu cầu của người dùng,
và `encoding/json` cũng không ngoại lệ. Bài viết blog này nói về các package thử nghiệm
`encoding/json/v2` và `encoding/json/jsontext` mới trong Go 1.25,
đem lại những cải tiến và sửa chữa được mong đợi từ lâu.
Bài viết này lập luận cho một phiên bản API chính mới,
cung cấp tổng quan về các package mới,
và giải thích cách bạn có thể sử dụng chúng.
Các package thử nghiệm không hiển thị theo mặc định và
có thể thay đổi API trong tương lai.

## Vấn đề với `encoding/json`

Nhìn chung, `encoding/json` đã hoạt động tốt.
Ý tưởng marshal và unmarshal các kiểu Go tùy ý
với một số biểu diễn mặc định trong JSON, kết hợp với khả năng
tùy chỉnh biểu diễn, đã được chứng minh là linh hoạt cao.
Tuy nhiên, trong những năm kể từ khi nó được giới thiệu,
nhiều người dùng đã xác định nhiều điểm hạn chế.

### Lỗi hành vi

Có nhiều lỗi hành vi trong `encoding/json`:

* **Xử lý cú pháp JSON không chính xác**: Theo năm tháng, JSON đã có
sự chuẩn hóa ngày càng tăng để các chương trình có thể giao tiếp đúng đắn.
Nói chung, các bộ giải mã đã trở nên nghiêm ngặt hơn trong việc từ chối các đầu vào mơ hồ,
để giảm khả năng hai triển khai sẽ có
các diễn giải (thành công) khác nhau về một giá trị JSON cụ thể.

    * `encoding/json` hiện chấp nhận UTF-8 không hợp lệ,
    trong khi Tiêu chuẩn Internet mới nhất (RFC 8259) yêu cầu UTF-8 hợp lệ.
    Hành vi mặc định nên báo lỗi khi có UTF-8 không hợp lệ,
    thay vì âm thầm gây hỏng dữ liệu,
    điều có thể gây ra vấn đề ở hạ nguồn.

    * `encoding/json` hiện chấp nhận các object có tên thành viên trùng lặp.
    RFC 8259 không chỉ định cách xử lý tên trùng lặp,
    vì vậy một triển khai có thể tự do chọn một giá trị tùy ý,
    hợp nhất các giá trị, loại bỏ các giá trị, hoặc báo lỗi.
    Sự hiện diện của tên trùng lặp dẫn đến giá trị JSON
    không có ý nghĩa được đồng thuận chung.
    Điều này có thể [bị khai thác bởi kẻ tấn công trong các ứng dụng bảo mật](https://www.youtube.com/watch?v=avilmOcHKHE&t=1057s)
    và đã bị khai thác trước đây (như trong [CVE-2017-12635](https://nvd.nist.gov/vuln/detail/CVE-2017-12635)).
    Hành vi mặc định nên nghiêng về phía an toàn và từ chối tên trùng lặp.

* **Rò rỉ nilness của slice và map**: JSON thường được sử dụng để giao tiếp với
các chương trình sử dụng triển khai JSON không cho phép `null` được unmarshal
vào kiểu dữ liệu được mong đợi là JSON array hoặc object.
Vì `encoding/json` marshal nil slice hoặc map thành JSON `null`,
điều này có thể dẫn đến lỗi khi unmarshal bởi các triển khai khác.
[Một khảo sát](/issue/63397#discussioncomment-7201222)
chỉ ra rằng hầu hết người dùng Go thích nil slice và map
được marshal thành JSON array hoặc object rỗng theo mặc định.

* **Unmarshal không phân biệt hoa thường**: Khi unmarshal, tên thành viên JSON object
được giải quyết sang tên trường struct Go bằng khớp không phân biệt hoa thường.
Đây là mặc định gây ngạc nhiên, có thể là lỗ hổng bảo mật tiềm ẩn, và là giới hạn hiệu suất.

* **Gọi phương thức không nhất quán**: Do một chi tiết triển khai,
các phương thức `MarshalJSON` được khai báo trên receiver con trỏ
[không được `encoding/json` gọi nhất quán](/issue/22967). Mặc dù được coi là lỗi,
điều này không thể sửa vì quá nhiều ứng dụng phụ thuộc vào hành vi hiện tại.

### Thiếu sót API

API của `encoding/json` có thể phức tạp hoặc hạn chế:

* Khó unmarshal chính xác từ `io.Reader`.
Người dùng thường viết `json.NewDecoder(r).Decode(v)`,
không từ chối dữ liệu rác ở cuối đầu vào.

* Các tùy chọn có thể được đặt trên kiểu `Encoder` và `Decoder`,
nhưng không thể sử dụng với các hàm `Marshal` và `Unmarshal`.
Tương tự, các kiểu triển khai interface `Marshaler` và `Unmarshaler`
không thể sử dụng các tùy chọn và không có cách để truyền tùy chọn xuống call stack.
Ví dụ, tùy chọn `Decoder.DisallowUnknownFields` mất hiệu lực
khi gọi phương thức `UnmarshalJSON` tùy chỉnh.

* Các hàm `Compact`, `Indent` và `HTMLEscape` ghi vào `bytes.Buffer`
thay vì thứ gì đó linh hoạt hơn như `[]byte` hoặc `io.Writer`.
Điều này giới hạn khả năng sử dụng của các hàm đó.

### Giới hạn hiệu suất

Gạt bỏ các chi tiết triển khai nội bộ,
API công khai cam kết với một số giới hạn hiệu suất nhất định:

* **MarshalJSON**: Phương thức interface `MarshalJSON` buộc triển khai
phải cấp phát `[]byte` được trả về. Ngoài ra, ngữ nghĩa yêu cầu
`encoding/json` xác minh rằng kết quả là JSON hợp lệ
và định dạng lại nó để khớp với thụt lề đã chỉ định.

* **UnmarshalJSON**: Phương thức interface `UnmarshalJSON` yêu cầu
một giá trị JSON hoàn chỉnh được cung cấp (không có dữ liệu trailing).
Điều này buộc `encoding/json` phân tích toàn bộ giá trị JSON cần unmarshal
để xác định vị trí kết thúc trước khi có thể gọi `UnmarshalJSON`.
Sau đó, chính phương thức `UnmarshalJSON` phải phân tích lại giá trị JSON được cung cấp.

* **Thiếu streaming**: Mặc dù các kiểu `Encoder` và `Decoder` hoạt động
trên `io.Writer` hoặc `io.Reader`, chúng đệm toàn bộ giá trị JSON trong bộ nhớ.
Phương thức `Decoder.Token` để đọc từng token riêng lẻ rất tốn bộ nhớ
và không có API tương ứng để ghi token.

Hơn nữa, nếu triển khai của phương thức `MarshalJSON` hoặc `UnmarshalJSON`
đệ quy gọi hàm `Marshal` hoặc `Unmarshal`,
thì hiệu suất trở nên bậc hai.

## Cố gắng sửa `encoding/json` trực tiếp

Giới thiệu một phiên bản chính mới, không tương thích của package là điều cân nhắc nặng nề.
Nếu có thể, chúng ta nên cố gắng sửa package hiện tại.

Mặc dù thêm tính năng mới tương đối dễ,
nhưng thay đổi các tính năng hiện có lại khó.
Thật không may, những vấn đề này là hậu quả cố hữu của API hiện tại,
khiến chúng gần như không thể sửa trong [cam kết tương thích Go 1](/doc/go1compat).

Về mặt nguyên tắc, chúng ta có thể khai báo các tên riêng biệt, chẳng hạn như `MarshalV2` hay `UnmarshalV2`,
nhưng điều đó tương đương với việc tạo ra một không gian tên song song trong cùng một package.
Điều này dẫn chúng ta đến `encoding/json/v2` (sau đây gọi là `v2`),
nơi chúng ta có thể thực hiện những thay đổi này trong không gian tên `v2` riêng biệt
trái ngược với `encoding/json` (sau đây gọi là `v1`).

## Lập kế hoạch cho `encoding/json/v2`

Việc lập kế hoạch cho phiên bản chính mới của `encoding/json` kéo dài nhiều năm.
Vào cuối năm 2020, thúc đẩy bởi sự không thể sửa các vấn đề trong package hiện tại,
Daniel Martí (một trong những người duy trì `encoding/json`) lần đầu tiên phác thảo suy nghĩ của mình về
[một package `v2` giả thuyết nên trông như thế nào](https://docs.google.com/document/d/1WQGoM44HLinH4NGBEv5drGlw5_RNW-GP7DdGEpm7Y3o).
Riêng biệt, sau công việc trước đây trên [API Go cho Protocol Buffers](/blog/protobuf-apiv2),
Joe Tsai thất vọng rằng [package `protojson`](/pkg/google.golang.org/protobuf/encoding/protojson)
cần sử dụng triển khai JSON tùy chỉnh vì `encoding/json` không
có khả năng tuân thủ tiêu chuẩn JSON nghiêm ngặt hơn mà
đặc tả Protocol Buffer yêu cầu,
cũng không serialize JSON theo cách streaming hiệu quả.

Tin rằng một tương lai tươi sáng hơn cho JSON vừa có lợi vừa có thể đạt được,
Daniel và Joe đã hợp tác để động não về `v2` và
[bắt đầu xây dựng một prototype](https://github.com/go-json-experiment/json)
(với mã ban đầu là phiên bản được đánh bóng của logic serialization JSON từ module protobuf Go).
Theo thời gian, một số người khác (Roger Peppe, Chris Hines, Johan Brandhorst-Satzkorn và Damien Neil)
tham gia nỗ lực này bằng cách cung cấp đánh giá thiết kế, đánh giá mã và kiểm tra hồi quy.
Nhiều cuộc thảo luận ban đầu có sẵn công khai trong
[các cuộc họp được ghi lại](https://www.youtube.com/playlist?list=PLZgrQPcV8W8EChkaAvv-3NUu6PYmnGG3b) và
[ghi chú cuộc họp](https://docs.google.com/document/d/1rovrOTd-wTawGMPPlPuKhwXaYBg9VszTXR9AQQL5LfI).

Công việc này đã được công khai từ đầu,
và chúng tôi ngày càng tham gia cộng đồng Go rộng lớn hơn,
đầu tiên với
[bài nói tại GopherCon](https://www.youtube.com/watch?v=avilmOcHKHE) và
[thảo luận được đăng vào cuối năm 2023](/issue/63397),
[đề xuất chính thức được đăng vào đầu năm 2025](/issue/71497),
và gần đây nhất là [áp dụng `encoding/json/v2` như một thử nghiệm Go](/issue/71845)
(có sẵn trong Go 1.25) để kiểm tra quy mô rộng hơn bởi tất cả người dùng Go.

Nỗ lực `v2` đã diễn ra trong 5 năm,
kết hợp phản hồi từ nhiều người đóng góp và thu được kinh nghiệm thực tế
có giá trị từ việc sử dụng trong môi trường production.

Đáng chú ý rằng phần lớn đã được phát triển và thúc đẩy bởi những người
không được Google tuyển dụng, cho thấy dự án Go là một nỗ lực hợp tác
với cộng đồng toàn cầu sôi động tận tâm cải thiện hệ sinh thái Go.

## Xây dựng trên `encoding/json/jsontext`

Trước khi thảo luận về API `v2`, chúng tôi trước tiên giới thiệu package thử nghiệm
[`encoding/json/jsontext`](/pkg/encoding/json/jsontext)
đặt nền tảng cho những cải tiến JSON trong Go trong tương lai.

Serialization JSON trong Go có thể được chia thành hai thành phần chính:

* *Chức năng cú pháp* liên quan đến việc xử lý JSON dựa trên ngữ pháp của nó, và
* *Chức năng ngữ nghĩa* định nghĩa mối quan hệ giữa các giá trị JSON và giá trị Go.

Chúng tôi sử dụng thuật ngữ "encode" và "decode" để mô tả chức năng cú pháp và
thuật ngữ "marshal" và "unmarshal" để mô tả chức năng ngữ nghĩa.
Chúng tôi nhằm cung cấp sự phân biệt rõ ràng giữa chức năng
thuần túy liên quan đến encoding so với chức năng marshaling.

<img src="jsonv2-exp/api.png" width=100%>

Sơ đồ này cung cấp tổng quan về sự phân tách này.
Các khối màu tím biểu diễn kiểu, trong khi các khối màu xanh biểu diễn hàm hoặc phương thức.
Hướng của các mũi tên gần đúng biểu diễn luồng dữ liệu.
Nửa dưới của sơ đồ, được triển khai bởi package `jsontext`,
chứa chức năng chỉ liên quan đến cú pháp,
trong khi nửa trên, được triển khai bởi package `json/v2`,
chứa chức năng gán ý nghĩa ngữ nghĩa cho dữ liệu cú pháp
được xử lý bởi nửa dưới.

API cơ bản của `jsontext` như sau:

```
package jsontext

type Encoder struct { ... }
func NewEncoder(io.Writer, ...Options) *Encoder
func (*Encoder) WriteValue(Value) error
func (*Encoder) WriteToken(Token) error

type Decoder struct { ... }
func NewDecoder(io.Reader, ...Options) *Decoder
func (*Decoder) ReadValue() (Value, error)
func (*Decoder) ReadToken() (Token, error)

type Kind byte
type Value []byte
func (Value) Kind() Kind
type Token struct { ... }
func (Token) Kind() Kind
```

Package `jsontext` cung cấp chức năng tương tác với JSON
ở cấp độ cú pháp và lấy tên từ
[RFC 8259, phần 2](https://datatracker.ietf.org/doc/html/rfc8259#section-2)
nơi ngữ pháp cho dữ liệu JSON được gọi theo đúng nghĩa là `JSON-text`.
Vì nó chỉ tương tác với JSON ở cấp độ cú pháp,
nó không phụ thuộc vào reflection của Go.

[`Encoder`](/pkg/encoding/json/jsontext#Encoder) và
[`Decoder`](/pkg/encoding/json/jsontext#Decoder)
cung cấp hỗ trợ encode và decode các giá trị và token JSON.
Các hàm khởi tạo
[chấp nhận tùy chọn variadic](/pkg/encoding/json/jsontext#Options)
ảnh hưởng đến hành vi cụ thể của encoding và decoding.
Khác với các kiểu `Encoder` và `Decoder` được khai báo trong `v1`,
các kiểu mới trong `jsontext` tránh lẫn lộn sự phân biệt giữa cú pháp và
ngữ nghĩa và hoạt động theo cách streaming thực sự.

Một giá trị JSON là một đơn vị dữ liệu hoàn chỉnh và được biểu diễn trong Go là
[một `[]byte` được đặt tên](/pkg/encoding/json/jsontext#Value).
Nó giống với [`RawMessage`](/pkg/encoding/json#RawMessage) trong `v1`.
Một giá trị JSON được cấu tạo cú pháp từ một hoặc nhiều token JSON.
Một token JSON được biểu diễn trong Go là [kiểu `Token` mờ đục](/pkg/encoding/json/jsontext#Token)
với các hàm khởi tạo và phương thức accessor.
Nó tương tự với [`Token`](/pkg/encoding/json#Token) trong `v1`
nhưng được thiết kế để biểu diễn các token JSON tùy ý mà không cần cấp phát.

Để giải quyết các vấn đề hiệu suất cơ bản với
các phương thức interface `MarshalJSON` và `UnmarshalJSON`,
chúng ta cần một cách hiệu quả để encode và decode JSON
như một chuỗi streaming của token và giá trị.
Trong `v2`, chúng tôi giới thiệu các phương thức interface `MarshalJSONTo` và `UnmarshalJSONFrom`
hoạt động trên `Encoder` hoặc `Decoder`, cho phép các triển khai của phương thức
xử lý JSON theo cách streaming thuần túy. Do đó, package `json` không cần
chịu trách nhiệm xác thực hoặc định dạng một giá trị JSON được trả về bởi `MarshalJSON`,
cũng không cần chịu trách nhiệm xác định ranh giới của giá trị JSON
được cung cấp cho `UnmarshalJSON`. Những trách nhiệm này thuộc về `Encoder` và `Decoder`.

## Giới thiệu `encoding/json/v2`

Xây dựng trên package `jsontext`, chúng tôi giờ giới thiệu package thử nghiệm
[`encoding/json/v2`](/pkg/encoding/json/v2).
Nó được thiết kế để sửa các vấn đề đã đề cập,
trong khi vẫn quen thuộc với người dùng của package `v1`.
Mục tiêu của chúng tôi là các cách dùng `v1` sẽ hoạt động *hầu như* giống nhau nếu được chuyển sang `v2`.

Trong bài viết này, chúng tôi sẽ chủ yếu đề cập đến API cấp cao của `v2`.
Để xem ví dụ về cách sử dụng nó, chúng tôi khuyến khích độc giả
nghiên cứu [các ví dụ trong package `v2`](/pkg/encoding/json/v2#pkg-examples) hoặc
đọc [blog của Anton Zhiyanov về chủ đề này](https://antonz.org/go-json-v2/).

API cơ bản của `v2` như sau:
```
package json

func Marshal(in any, opts ...Options) (out []byte, err error)
func MarshalWrite(out io.Writer, in any, opts ...Options) error
func MarshalEncode(out *jsontext.Encoder, in any, opts ...Options) error

func Unmarshal(in []byte, out any, opts ...Options) error
func UnmarshalRead(in io.Reader, out any, opts ...Options) error
func UnmarshalDecode(in *jsontext.Decoder, out any, opts ...Options) error
```

Các hàm [`Marshal`](/pkg/encoding/json/v2#Marshal)
và [`Unmarshal`](/pkg/encoding/json/v2#Unmarshal)
có chữ ký tương tự `v1`, nhưng chấp nhận tùy chọn để cấu hình hành vi của chúng.
Các hàm [`MarshalWrite`](/pkg/encoding/json/v2#MarshalWrite)
và [`UnmarshalRead`](/pkg/encoding/json/v2#UnmarshalRead)
trực tiếp hoạt động trên `io.Writer` hoặc `io.Reader`,
tránh cần xây dựng tạm thời một `Encoder` hoặc `Decoder`
chỉ để ghi hoặc đọc từ các kiểu đó.
Các hàm [`MarshalEncode`](/pkg/encoding/json/v2#MarshalEncode)
và [`UnmarshalDecode`](/pkg/encoding/json/v2#UnmarshalDecode)
hoạt động trên `jsontext.Encoder` và `jsontext.Decoder` và
thực sự là triển khai cơ bản của các hàm đã đề cập trước đó.
Không giống `v1`, các tùy chọn là đối số hạng nhất cho mỗi hàm marshal và unmarshal,
mở rộng đáng kể tính linh hoạt và khả năng cấu hình của `v2`.
Có [nhiều tùy chọn sẵn có](/pkg/encoding/json/v2#Options)
trong `v2` mà bài viết này không đề cập.

### Tùy chỉnh theo kiểu

Tương tự `v1`, `v2` cho phép các kiểu định nghĩa biểu diễn JSON của chúng
bằng cách thỏa mãn các interface cụ thể.

```
type Marshaler interface {
	MarshalJSON() ([]byte, error)
}
type MarshalerTo interface {
	MarshalJSONTo(*jsontext.Encoder) error
}

type Unmarshaler interface {
	UnmarshalJSON([]byte) error
}
type UnmarshalerFrom interface {
	UnmarshalJSONFrom(*jsontext.Decoder) error
}
```

Các interface [`Marshaler`](/pkg/encoding/json/v2#Marshaler)
và [`Unmarshaler`](/pkg/encoding/json/v2#Unmarshaler)
giống với những interface trong `v1`.
Các interface mới [`MarshalerTo`](/pkg/encoding/json/v2#MarshalerTo)
và [`UnmarshalerFrom`](/pkg/encoding/json/v2#UnmarshalerFrom)
cho phép một kiểu biểu diễn chính nó dưới dạng JSON bằng cách dùng `jsontext.Encoder` hoặc `jsontext.Decoder`.
Điều này cho phép các tùy chọn được chuyển tiếp xuống call stack, vì các tùy chọn
có thể được lấy qua phương thức accessor `Options` trên `Encoder` hoặc `Decoder`.

Xem [ví dụ `OrderedObject`](/pkg/encoding/json/v2#example-package-OrderedObject)
để biết cách triển khai kiểu tùy chỉnh duy trì thứ tự của các thành viên JSON object.

### Tùy chỉnh theo người gọi

Trong `v2`, người gọi `Marshal` và `Unmarshal` cũng có thể chỉ định
biểu diễn JSON tùy chỉnh cho bất kỳ kiểu tùy ý nào,
trong đó các hàm do người gọi chỉ định có ưu tiên hơn các phương thức được kiểu định nghĩa
hoặc biểu diễn mặc định cho một kiểu cụ thể.

```
func WithMarshalers(*Marshalers) Options

type Marshalers struct { ... }
func MarshalFunc[T any](fn func(T) ([]byte, error)) *Marshalers
func MarshalToFunc[T any](fn func(*jsontext.Encoder, T) error) *Marshalers

func WithUnmarshalers(*Unmarshalers) Options

type Unmarshalers struct { ... }
func UnmarshalFunc[T any](fn func([]byte, T) error) *Unmarshalers
func UnmarshalFromFunc[T any](fn func(*jsontext.Decoder, T) error) *Unmarshalers
```

[`MarshalFunc`](/pkg/encoding/json/v2#MarshalFunc) và
[`MarshalToFunc`](/pkg/encoding/json/v2#MarshalToFunc)
xây dựng một marshaler tùy chỉnh có thể được truyền đến lệnh gọi `Marshal`
bằng `WithMarshalers` để ghi đè marshaling của các kiểu cụ thể.
Tương tự,
[`UnmarshalFunc`](/pkg/encoding/json/v2#UnmarshalFunc) và
[`UnmarshalFromFunc`](/pkg/encoding/json/v2#UnmarshalFromFunc)
hỗ trợ tùy chỉnh tương tự cho `Unmarshal`.

[Ví dụ `ProtoJSON`](/pkg/encoding/json/v2#example-package-ProtoJSON)
minh họa cách tính năng này cho phép serialization của tất cả
các kiểu [`proto.Message`](/pkg/google.golang.org/protobuf/proto#Message)
được xử lý bởi package [`protojson`](/pkg/google.golang.org/protobuf/encoding/protojson).

### Sự khác biệt hành vi

Mặc dù `v2` nhằm mục đích hoạt động *hầu như* giống `v1`,
hành vi của nó đã thay đổi [theo một số cách](/pkg/github.com/go-json-experiment/json/v1#hdr-Migrating_to_v2)
để giải quyết các vấn đề trong `v1`, đáng chú ý nhất là:

* `v2` báo lỗi khi có UTF-8 không hợp lệ.

* `v2` báo lỗi nếu JSON object chứa tên trùng lặp.

* `v2` marshal nil Go slice hoặc Go map thành JSON array hoặc JSON object rỗng tương ứng.

* `v2` unmarshal JSON object vào Go struct bằng cách
khớp phân biệt hoa thường từ tên thành viên JSON đến tên trường Go.

* `v2` định nghĩa lại tùy chọn tag `omitempty` để bỏ qua một trường nếu nó sẽ được
mã hóa thành giá trị JSON "rỗng" (là `null`, `""`, `[]` và `{}`).

* `v2` báo lỗi khi cố serialize `time.Duration`,
hiện không có [biểu diễn mặc định](/issue/71631),
nhưng cung cấp tùy chọn để cho phép người gọi quyết định.

Đối với hầu hết các thay đổi hành vi, có tùy chọn struct tag hoặc tùy chọn do người gọi chỉ định
có thể cấu hình hành vi để hoạt động theo ngữ nghĩa `v1` hoặc `v2`,
hoặc thậm chí hành vi khác do người gọi xác định.
Xem ["Chuyển sang v2"](/pkg/github.com/go-json-experiment/json/v1#hdr-Migrating_to_v2) để biết thêm thông tin.

### Tối ưu hóa hiệu suất

Hiệu suất `Marshal` của `v2` xấp xỉ ngang bằng `v1`.
Đôi khi nó nhanh hơn một chút, nhưng lần khác lại chậm hơn một chút.
Hiệu suất `Unmarshal` của `v2` nhanh hơn đáng kể so với `v1`,
với các benchmark cho thấy cải thiện lên đến 10 lần.

Để có được lợi ích hiệu suất lớn hơn,
các triển khai hiện có của
[`Marshaler`](/pkg/encoding/json/v2#Marshaler) và
[`Unmarshaler`](/pkg/encoding/json/v2#Unmarshaler) nên
chuyển sang triển khai thêm
[`MarshalerTo`](/pkg/encoding/json/v2#MarshalerTo) và
[`UnmarshalerFrom`](/pkg/encoding/json/v2#UnmarshalerFrom),
để chúng có thể hưởng lợi từ việc xử lý JSON theo cách streaming thuần túy.
Ví dụ, phân tích đệ quy các đặc tả OpenAPI trong các phương thức `UnmarshalJSON`
đã ảnh hưởng đáng kể đến hiệu suất trong một dịch vụ cụ thể của Kubernetes
(xem [kubernetes/kube-openapi#315](https://github.com/kubernetes/kube-openapi/issues/315)),
trong khi chuyển sang `UnmarshalJSONFrom` đã cải thiện hiệu suất theo bậc độ lớn.

Để biết thêm thông tin, xem kho lưu trữ
[`go-json-experiment/jsonbench`](https://github.com/go-json-experiment/jsonbench).

## Cải thiện `encoding/json` theo chiều ngược

Chúng tôi muốn tránh có hai triển khai JSON riêng biệt trong thư viện chuẩn Go,
vì vậy điều quan trọng là, dưới lớp bề mặt, `v1` được triển khai theo `v2`.

Có một số lợi ích của cách tiếp cận này:

1. **Di chuyển dần dần**: Các hàm `Marshal` và `Unmarshal` trong `v1` hoặc `v2`
biểu diễn một tập hợp hành vi mặc định hoạt động theo ngữ nghĩa `v1` hoặc `v2`.
Các tùy chọn có thể được chỉ định để cấu hình `Marshal` hoặc `Unmarshal` hoạt động với
ngữ nghĩa hoàn toàn `v1`, hầu hết `v1` với một chút `v2`, pha trộn `v1` hoặc `v2`,
hầu hết `v2` với một chút `v1`, hoặc hoàn toàn `v2`.
Điều này cho phép di chuyển dần dần giữa các hành vi mặc định của hai phiên bản.

2. **Kế thừa tính năng**: Khi các tính năng tương thích ngược được thêm vào `v2`,
chúng sẽ được cung cấp sẵn trong `v1`. Ví dụ, `v2` thêm
hỗ trợ cho một số tùy chọn struct tag mới như `inline` hoặc `format` và cũng
hỗ trợ cho các phương thức interface `MarshalJSONTo` và `UnmarshalJSONFrom`,
vốn hiệu quả hơn và linh hoạt hơn.
Khi `v1` được triển khai theo `v2`, nó sẽ kế thừa hỗ trợ cho các tính năng này.

3. **Giảm bảo trì**: Việc bảo trì một package được sử dụng rộng rãi đòi hỏi nỗ lực đáng kể.
Bằng cách để `v1` và `v2` sử dụng cùng một triển khai, gánh nặng bảo trì được giảm.
Nhìn chung, một thay đổi duy nhất sẽ sửa lỗi, cải thiện hiệu suất, hoặc thêm chức năng cho cả hai phiên bản.
Không cần backport một thay đổi `v2` với một thay đổi `v1` tương đương.

Mặc dù một số phần của `v1` có thể bị deprecated theo thời gian (giả sử `v2` tốt nghiệp khỏi thử nghiệm),
nhưng package nói chung sẽ không bao giờ bị deprecated.
Di chuyển sang `v2` sẽ được khuyến khích, nhưng không bắt buộc.
Dự án Go sẽ không ngừng hỗ trợ `v1`.

## Thử nghiệm với `jsonv2`

API mới hơn trong các package `encoding/json/jsontext` và `encoding/json/v2` không hiển thị theo mặc định.
Để sử dụng chúng, hãy build mã của bạn với `GOEXPERIMENT=jsonv2` được đặt trong môi trường hoặc với build tag `goexperiment.jsonv2`.
Bản chất của thử nghiệm là API không ổn định và có thể thay đổi trong tương lai.
Mặc dù API không ổn định, triển khai có chất lượng cao và
đã được sử dụng thành công trong môi trường production bởi một số dự án lớn.

Thực tế là `v1` được triển khai theo `v2` có nghĩa là triển khai cơ bản của `v1`
hoàn toàn khác khi build dưới thử nghiệm `jsonv2`.
Không thay đổi bất kỳ mã nào, bạn có thể chạy các bài test của mình
dưới `jsonv2` và về mặt lý thuyết không có gì mới sẽ thất bại:

```
GOEXPERIMENT=jsonv2 go test ./...
```

Việc triển khai lại `v1` theo `v2` nhằm cung cấp hành vi giống hệt nhau
trong phạm vi [cam kết tương thích Go 1](/doc/go1compat),
mặc dù một số khác biệt có thể quan sát được như cách diễn đạt chính xác của thông báo lỗi.
Chúng tôi khuyến khích bạn chạy các bài test của mình dưới `jsonv2` và
báo cáo bất kỳ hồi quy nào [trên trình theo dõi vấn đề](/issues).

Trở thành thử nghiệm trong Go 1.25 là một cột mốc quan trọng trên con đường
chính thức áp dụng `encoding/json/jsontext` và `encoding/json/v2` vào thư viện chuẩn.
Tuy nhiên, mục đích của thử nghiệm `jsonv2` là để có được kinh nghiệm rộng hơn.
Phản hồi của bạn sẽ xác định các bước tiếp theo của chúng tôi, và kết quả của thử nghiệm này,
có thể dẫn đến từ việc từ bỏ nỗ lực đến việc áp dụng như các package ổn định của Go 1.26.
Hãy chia sẻ trải nghiệm của bạn trên [go.dev/issue/71497](/issue/71497), và giúp xác định tương lai của Go.
