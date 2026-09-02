<!--{
  "Title": "Tutorial: Working with JSON",
  "Breadcrumb": true
}-->

JSON (JavaScript Object Notation) is a simple data interchange format.
Syntactically it resembles the objects and lists of JavaScript.
It is commonly used for communication with networked API services,
but it is used in many other places, too.
Its home page, [json.org](http://json.org),
provides a wonderfully clear and concise definition of the standard.

With the [encoding/json/v2 package](/pkg/encoding/json/v2) it's a snap to read
and write JSON data from your Go programs.
This package provides a cleaner API and better defaults than the older
[encoding/json](/pkg/encoding/json) package.

## Encoding

To encode JSON data we use the [`Marshal`](/pkg/encoding/json/v2#Marshal) function.

```
func Marshal(in any, opts ...Options) (out []byte, err error)
```

Given the Go data structure, `Message`,

```
type Message struct {
	Name string
	Body string
	Time time.Time
}
```

and an instance of `Message`

```
m := Message{"Alice", "Hello", time.Date(2011, 1, 25, 0, 0, 0, 0, time.UTC)}
```

we can marshal a JSON-encoded version of m using `json.Marshal`:

```
b, err := json.Marshal(m)
```

If all is well, `err` will be `nil` and `b` will be a `[]byte` containing this JSON data:

```
b == []byte(`{"Name":"Alice","Body":"Hello","Time":"2011-01-25T00:00:00Z"}`)
```

Only data structures that can be represented as valid JSON will be encoded:

  - Pointers will be encoded as the values they point to (or `null` if the pointer is `nil`).

  - JSON objects only support strings as keys.
    Thus, to encode a Go map, it must have a key that encodes as a JSON string,
    such as `map[string]T` (where `T` is any Go type supported by the json
    package).

  - Channel, complex, and function types cannot be encoded.

  - Cyclic data structures are not supported.

  - [`Marshal`](/pkg/encoding/json/v2#Marshal) documents the full set of encoding semantics.

The json package only accesses the exported fields of struct types (those
that begin with an uppercase letter).
Therefore only the exported fields of a struct will be present in the JSON output.

## Giải mã

Để giải mã dữ liệu JSON, chúng ta sử dụng hàm [`Unmarshal`](/pkg/encoding/json/v2#Unmarshal).

```
func Unmarshal(in []byte, out any, opts ...Options) (err error)
```

Trước tiên, chúng ta phải tạo một nơi để lưu trữ dữ liệu đã được giải mã

```
var m Message
```

và gọi `json.Unmarshal`, truyền vào đó một `[]byte` chứa dữ liệu JSON và một con trỏ tới `m`

```
err := json.Unmarshal(b, &m)
```

Nếu `b` chứa JSON hợp lệ phù hợp với `m`,
sau lời gọi này `err` sẽ là `nil` và dữ liệu từ `b` sẽ được
lưu trữ trong struct `m`,
như thể bằng một phép gán như:

```
m = Message{
	Name: "Alice",
	Body: "Hello",
	Time: time.Date(2011, 1, 25, 0, 0, 0, 0, time.UTC),
}
```

Làm thế nào `Unmarshal` xác định các trường để lưu dữ liệu đã giải mã?
Với một khóa JSON cụ thể `"Foo"`,
`Unmarshal` sẽ duyệt qua các trường của struct đích để tìm (theo
thứ tự ưu tiên):

  - Một trường đã export có tag `json:"Foo"` (xem [đặc tả Go](/ref/spec#Struct_types)
    để biết thêm về tag của struct), hoặc

  - Một trường đã export có tên `Foo`.

Điều gì xảy ra khi cấu trúc của dữ liệu JSON không khớp chính xác với kiểu Go?

```
b := []byte(`{"Name":"Bob","Food":"Pickle"}`)
var m Message
err := json.Unmarshal(b, &m)
```

`Unmarshal` sẽ chỉ giải mã các trường mà nó có thể tìm thấy trong kiểu đích.
Trong trường hợp này, chỉ trường `Name` của `m` được điền giá trị,
còn trường `Food` sẽ bị bỏ qua.
Hành vi này đặc biệt hữu ích khi bạn muốn chỉ lấy một vài trường cụ thể
từ một khối JSON lớn.
Điều này cũng có nghĩa là mọi trường chưa export trong struct đích sẽ
không bị ảnh hưởng bởi `Unmarshal`.

Nhưng nếu bạn không biết trước cấu trúc của dữ liệu JSON thì sao?

## JSON tổng quát với `any`

Kiểu `any` mô tả bất kỳ kiểu Go nào.
`any` được định nghĩa là `interface{}` (interface rỗng), mô tả một interface không có phương thức nào.
Mọi kiểu Go đều triển khai ít nhất không phương thức nào và do đó thỏa mãn interface rỗng.

`any` đóng vai trò là một kiểu vùng chứa tổng quát:

	var a any
	a = "a string"
	a = 2011
	a = 2.777

Một phép khẳng định kiểu truy cập kiểu cụ thể bên dưới:

	r := a.(float64)
	fmt.Println("the circle's area", math.Pi*r*r)

Hoặc, nếu kiểu bên dưới không xác định, một phép chuyển kiểu xác định kiểu đó:

	switch v := a.(type) {
	case int:
	    fmt.Println("twice a is", v*2)
	case float64:
	    fmt.Println("the reciprocal of a is", 1/v)
	case string:
	    h := len(v) / 2
	    fmt.Println("a swapped by halves is", v[h:]+v[:h])
	default:
	    // a isn't one of the types above
	}

Gói json sử dụng các giá trị `map[string]any` và
`[]any` để lưu trữ các đối tượng và mảng JSON tùy ý;
nó sẽ dễ dàng giải mã bất kỳ khối JSON hợp lệ nào vào một giá trị
`any` thuần túy. Các kiểu Go cụ thể mặc định là:

  - `bool` cho các giá trị boolean JSON,

  - `float64` cho các số JSON,

  - `string` cho các chuỗi JSON, và

  - `nil` cho giá trị null JSON.

## Decoding arbitrary data

Consider this JSON data, stored in the variable `b`:

```
b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)
```

Without knowing this data's structure, we can decode it into an `any` value with `Unmarshal`:

```
var f any
err := json.Unmarshal(b, &f)
```

At this point the Go value in `f` would be a map whose keys are strings
and whose values are themselves stored as `any` values:

```
f = map[string]any{
	"Name": "Wednesday",
	"Age":  6,
	"Parents": []any{
		"Gomez",
		"Morticia",
	},
}
```

To access this data we can use a type assertion to access `f`'s underlying `map[string]any`:

```
m := f.(map[string]any)
```

We can then iterate through the map with a range statement and use a type
switch to access its values as their concrete types:

```
for k, v := range m {
	switch vv := v.(type) {
	case string:
		fmt.Println(k, "is string", vv)
	case float64:
		fmt.Println(k, "is float64", vv)
	case []any:
		fmt.Println(k, "is an array:")
		for i, u := range vv {
			fmt.Println(i, u)
		}
	default:
		fmt.Println(k, "is of a type I don't know how to handle")
	}
}
```

In this way you can work with unknown JSON data while still enjoying the benefits of type safety.

## Reference Types

Let's define a Go type to contain the data from the previous example:

```
type FamilyMember struct {
	Name    string
	Age     int
	Parents []string
}

var m FamilyMember
err := json.Unmarshal(b, &m)
```

Unmarshaling that data into a `FamilyMember` value works as expected,
but if we look closely we can see a remarkable thing has happened.
With the var statement we allocated a `FamilyMember` struct,
and then provided a pointer to that value to `Unmarshal`,
but at that time the `Parents` field was a `nil` slice value.
To populate the `Parents` field, `Unmarshal` allocated a new slice behind the scenes.
This is typical of how `Unmarshal` works with the supported reference types
(pointers, slices, and maps).

Consider unmarshaling into this data structure:

```
type Foo struct {
	Bar *Bar
}
```

If there were a `Bar` field in the JSON object,
`Unmarshal` would allocate a new `Bar` and populate it.
If not, `Bar` would be left as a `nil` pointer.

From this a useful pattern arises: if you have an application that receives
a few distinct message types,
you might define "receiver" structure like

```
type IncomingMessage struct {
	Cmd *Command
	Msg *Message
}
```

and the sending party can populate the `Cmd` field and/or the `Msg` field
of the top-level JSON object,
depending on the type of message they want to communicate.
`Unmarshal`, when decoding the JSON into an `IncomingMessage` struct,
will only allocate the data structures present in the JSON data.
To know which messages to process, the programmer need simply test that
either `Cmd` or `Msg` is not `nil`.

## Truyền trực tiếp Marshal và Unmarshal

Các interface [`io.Reader`](/pkg/io#Reader) và [`io.Writer`](/pkg/io#Writer) xuất hiện phổ biến trong Go, cung cấp quyền truy cập dạng stream đến các tài nguyên như kết nối HTTP, WebSocket hoặc tệp.
Các hàm `MarshalWrite` và ``UnmarshalRead` cho phép marshal và unmarshal trực tiếp đến và từ các stream này mà không cần một `[]byte` trung gian chứa toàn bộ thông điệp.

```
func MarshalWrite(out io.Writer, in any, opts ...Options) (err error)
func UnmarshalRead(in io.Reader, out any, opts ...Options) (err error)
```

Ví dụ, chúng ta có thể ghi trực tiếp kết quả marshal ra đầu ra chuẩn:

```
err := json.MarshalWrite(os.Stdout, m)
```

## Marshal và Unmarshal tùy chỉnh

Đôi khi hành vi marshal mặc định không phù hợp với kiểu của bạn.

Ví dụ, giả sử chúng ta có một kiểu mô tả phiên bản phần mềm.

```
type Version struct {
	Major, Minor, Patch int64
}

v := Version{1, 2, 3}
b, err := json.Marshal(s)
```

Giá trị này được marshal thành `{"Major":1,"Minor":2,"Patch":3}`, như mong đợi dựa trên định nghĩa struct, nhưng một chuỗi phiên bản như "1.2.3" có thể sẽ là biểu diễn tốt hơn cho các bên dùng JSON.

Bằng cách triển khai interface [`MarshalerTo`](/pkg/encoding/json/v2#MarshalerTo), chúng ta có thể cung cấp một biểu diễn JSON tùy chỉnh.
[`UnmarshalerFrom`](/pkg/encoding/json/v2#UnmarshalerFrom) cung cấp khả năng unmarshal biểu diễn JSON tùy chỉnh.

```
type MarshalerTo interface {
	MarshalJSONTo(*jsontext.Encoder) error
}

type UnmarshalerFrom interface {
	UnmarshalJSONFrom(*jsontext.Decoder) error
}
```

Chúng ta có thể triển khai các phương thức này để chuyển đổi sang và từ các biểu diễn chuỗi.

```
func (v Version) MarshalJSONTo(enc *jsontext.Encoder) error {
	return json.MarshalEncode(enc, fmt.Sprintf("%d.%d.%d", v.Major, v.Minor, v.Patch))
}
```

Với marshal tùy chỉnh, giá trị `Version{1, 2, 3}` hiện được marshal thành `"1.2.3"`.

```
func (v *Version) UnmarshalJSONFrom(dec *jsontext.Decoder) error {
	if k := dec.PeekKind(); k != jsontext.KindString {
		// Giá trị phải là một chuỗi.
		return &json.SemanticError{JSONKind: k}
	}

	var s string
	if err := json.UnmarshalDecode(dec, &s); err != nil {
		return err
	}

	_, err := fmt.Sscanf(s, "%d.%d.%d", &v.Major, &v.Minor, &v.Patch)
	return err
}
```

Với unmarshal tùy chỉnh, `"1.2.3"` hiện được unmarshal thành `Version{1, 2, 3}`.
Ví dụ đơn giản này yêu cầu định dạng phiên bản chính xác `"major.minor.patch"`, nhưng một `UnmarshalJSONFrom` phức tạp hơn có thể chọn bổ sung tính linh hoạt, chẳng hạn như cho phép phiên bản minor và patch là tùy chọn.

## Tài liệu tham khảo

Để biết thêm thông tin, hãy xem tài liệu về gói [encoding/json/v2](/pkg/encoding/json/v2) và [encoding/json/jsontext](/pkg/encoding/json/jsontext).
