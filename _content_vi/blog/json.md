---
title: JSON và Go
date: 2011-01-25
by:
- Andrew Gerrand
tags:
- json
- technical
summary: Cách tạo và sử dụng dữ liệu định dạng JSON trong Go.
---

[_**Lưu ý, tháng 8 năm 2026**: Go 1.27 giới thiệu gói `encoding/json/v2` mới.
Để có phần giới thiệu cập nhật về cách làm việc với JSON trong Go, xem “[Hướng dẫn: Làm việc với JSON](/doc/tutorial/json).”_]

## Giới thiệu

JSON (JavaScript Object Notation) là một định dạng trao đổi dữ liệu đơn giản.
Về mặt cú pháp, nó giống các đối tượng và danh sách trong JavaScript.
Nó được sử dụng phổ biến nhất cho giao tiếp giữa các backend web và các chương trình
JavaScript chạy trên trình duyệt,
nhưng nó cũng được sử dụng ở nhiều nơi khác.
Trang chủ của nó, [json.org](http://json.org),
cung cấp một định nghĩa rõ ràng và súc tích về tiêu chuẩn.

Với [package json](/pkg/encoding/json/), việc đọc và ghi dữ liệu JSON
từ chương trình Go của bạn rất dễ dàng.

## Mã hóa (Encoding)

Để mã hóa dữ liệu JSON, chúng ta sử dụng hàm [`Marshal`](/pkg/encoding/json/#Marshal).

	func Marshal(v interface{}) ([]byte, error)

Cho cấu trúc dữ liệu Go `Message`,

	type Message struct {
	    Name string
	    Body string
	    Time int64
	}

và một instance của `Message`

	m := Message{"Alice", "Hello", 1294706395881547000}

chúng ta có thể marshal một phiên bản mã hóa JSON của m bằng `json.Marshal`:

	b, err := json.Marshal(m)

Nếu mọi thứ ổn, `err` sẽ là `nil` và `b` sẽ là một `[]byte` chứa dữ liệu JSON này:

	b == []byte(`{"Name":"Alice","Body":"Hello","Time":1294706395881547000}`)

Chỉ các cấu trúc dữ liệu có thể biểu diễn dưới dạng JSON hợp lệ mới được mã hóa:

  - JSON object chỉ hỗ trợ string làm key;
    để mã hóa kiểu map của Go, nó phải có dạng `map[string]T` (trong đó `T`
    là bất kỳ kiểu Go nào được package json hỗ trợ).

  - Kiểu Channel, complex và function không thể mã hóa.

  - Cấu trúc dữ liệu vòng tròn (cyclic) không được hỗ trợ; chúng sẽ khiến `Marshal` đi vào vòng lặp vô hạn.

  - Con trỏ sẽ được mã hóa như các giá trị mà chúng trỏ đến (hoặc 'null' nếu con trỏ là `nil`).

Package json chỉ truy cập các trường exported của kiểu struct (những trường
bắt đầu bằng chữ cái in hoa).
Do đó, chỉ các trường exported của một struct mới xuất hiện trong kết quả JSON.

## Giải mã (Decoding)

Để giải mã dữ liệu JSON, chúng ta sử dụng hàm [`Unmarshal`](/pkg/encoding/json/#Unmarshal).

	func Unmarshal(data []byte, v interface{}) error

Đầu tiên chúng ta phải tạo một nơi để lưu dữ liệu đã giải mã

	var m Message

và gọi `json.Unmarshal`, truyền vào một `[]byte` dữ liệu JSON và một con trỏ đến `m`

	err := json.Unmarshal(b, &m)

Nếu `b` chứa JSON hợp lệ phù hợp với `m`,
sau lệnh gọi, `err` sẽ là `nil` và dữ liệu từ `b` sẽ được
lưu trong struct `m`,
như thể được gán bởi:

	m = Message{
	    Name: "Alice",
	    Body: "Hello",
	    Time: 1294706395881547000,
	}

`Unmarshal` xác định các trường cần lưu dữ liệu đã giải mã như thế nào?
Với một key JSON `"Foo"` đã cho,
`Unmarshal` sẽ tìm trong các trường của struct đích (theo thứ tự ưu tiên):

  - Một trường exported có tag `"Foo"` (xem [đặc tả Go](/ref/spec#Struct_types)
    để biết thêm về struct tag),

  - Một trường exported có tên `"Foo"`, hoặc

  - Một trường exported có tên `"FOO"` hay `"FoO"` hoặc khớp không phân biệt hoa thường nào khác của `"Foo"`.

Điều gì xảy ra khi cấu trúc của dữ liệu JSON không khớp chính xác với kiểu Go?

	b := []byte(`{"Name":"Bob","Food":"Pickle"}`)
	var m Message
	err := json.Unmarshal(b, &m)

`Unmarshal` sẽ chỉ giải mã các trường mà nó có thể tìm thấy trong kiểu đích.
Trong trường hợp này, chỉ trường Name của m sẽ được điền,
và trường Food sẽ bị bỏ qua.
Hành vi này đặc biệt hữu ích khi bạn muốn chỉ lấy một vài trường cụ thể
từ một JSON lớn.
Nó cũng có nghĩa là bất kỳ trường unexported nào trong struct đích sẽ
không bị ảnh hưởng bởi `Unmarshal`.

Nhưng nếu bạn không biết cấu trúc của dữ liệu JSON trước thì sao?

## JSON tổng quát với interface{}

Kiểu `interface{}` (interface rỗng) mô tả một interface với không có phương thức.
Mọi kiểu Go đều triển khai ít nhất không phương thức và do đó thỏa mãn interface rỗng.

Interface rỗng phục vụ như kiểu container tổng quát:

	var i interface{}
	i = "a string"
	i = 2011
	i = 2.777

Type assertion truy cập kiểu concrete bên dưới:

	r := i.(float64)
	fmt.Println("the circle's area", math.Pi*r*r)

Hoặc, nếu kiểu bên dưới không biết, một type switch xác định kiểu:

	switch v := i.(type) {
	case int:
	    fmt.Println("twice i is", v*2)
	case float64:
	    fmt.Println("the reciprocal of i is", 1/v)
	case string:
	    h := len(v) / 2
	    fmt.Println("i swapped by halves is", v[h:]+v[:h])
	default:
	    // i isn't one of the types above
	}

Package json sử dụng giá trị `map[string]interface{}` và
`[]interface{}` để lưu các JSON object và array tùy ý;
nó sẽ vui vẻ unmarshal bất kỳ JSON blob hợp lệ nào vào một
giá trị `interface{}` đơn giản. Các kiểu concrete Go mặc định là:

  - `bool` cho JSON boolean,

  - `float64` cho JSON number,

  - `string` cho JSON string, và

  - `nil` cho JSON null.

## Giải mã dữ liệu tùy ý

Xét dữ liệu JSON này, được lưu trong biến `b`:

	b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)

Không biết cấu trúc của dữ liệu này, chúng ta có thể giải mã nó vào một giá trị `interface{}` với `Unmarshal`:

	var f interface{}
	err := json.Unmarshal(b, &f)

Lúc này, giá trị Go trong `f` sẽ là một map có key là string
và các giá trị của chúng được lưu như các giá trị interface rỗng:

	f = map[string]interface{}{
	    "Name": "Wednesday",
	    "Age":  6,
	    "Parents": []interface{}{
	        "Gomez",
	        "Morticia",
	    },
	}

Để truy cập dữ liệu này, chúng ta có thể sử dụng type assertion để truy cập `map[string]interface{}` bên dưới của `f`:

	m := f.(map[string]interface{})

Sau đó chúng ta có thể lặp qua map với câu lệnh range và sử dụng type
switch để truy cập các giá trị của nó theo kiểu concrete:

	for k, v := range m {
	    switch vv := v.(type) {
	    case string:
	        fmt.Println(k, "is string", vv)
	    case float64:
	        fmt.Println(k, "is float64", vv)
	    case []interface{}:
	        fmt.Println(k, "is an array:")
	        for i, u := range vv {
	            fmt.Println(i, u)
	        }
	    default:
	        fmt.Println(k, "is of a type I don't know how to handle")
	    }
	}

Bằng cách này, bạn có thể làm việc với dữ liệu JSON không biết trước trong khi vẫn được hưởng lợi từ an toàn kiểu.

## Kiểu tham chiếu (Reference Types)

Hãy định nghĩa một kiểu Go để chứa dữ liệu từ ví dụ trước:

	type FamilyMember struct {
	    Name    string
	    Age     int
	    Parents []string
	}

	var m FamilyMember
	err := json.Unmarshal(b, &m)

Việc unmarshal dữ liệu đó vào một giá trị `FamilyMember` hoạt động như mong đợi,
nhưng nếu nhìn kỹ, chúng ta có thể thấy một điều đáng chú ý đã xảy ra.
Với câu lệnh var, chúng ta đã cấp phát một struct `FamilyMember`,
và sau đó cung cấp một con trỏ đến giá trị đó cho `Unmarshal`,
nhưng lúc đó trường `Parents` là một giá trị slice nil.
Để điền vào trường `Parents`, `Unmarshal` đã cấp phát một slice mới phía sau hậu trường.
Đây là điển hình cho cách `Unmarshal` hoạt động với các kiểu tham chiếu được hỗ trợ
(con trỏ, slice và map).

Xét việc unmarshal vào cấu trúc dữ liệu này:

	type Foo struct {
	    Bar *Bar
	}

Nếu có trường `Bar` trong JSON object,
`Unmarshal` sẽ cấp phát một `Bar` mới và điền vào nó.
Nếu không, `Bar` sẽ được để lại là con trỏ nil.

Từ đây xuất hiện một pattern hữu ích: nếu bạn có ứng dụng nhận
một vài loại tin nhắn khác nhau,
bạn có thể định nghĩa cấu trúc "receiver" như

	type IncomingMessage struct {
	    Cmd *Command
	    Msg *Message
	}

và bên gửi có thể điền vào trường `Cmd` và/hoặc trường `Msg`
của JSON object cấp cao nhất,
tùy thuộc vào loại tin nhắn họ muốn truyền đạt.
`Unmarshal`, khi giải mã JSON vào struct `IncomingMessage`,
sẽ chỉ cấp phát các cấu trúc dữ liệu có trong dữ liệu JSON.
Để biết tin nhắn nào cần xử lý, lập trình viên chỉ cần kiểm tra
xem `Cmd` hay `Msg` không phải `nil`.

## Encoder và Decoder streaming

Package json cung cấp các kiểu `Decoder` và `Encoder` để hỗ trợ thao tác phổ biến
là đọc và ghi luồng dữ liệu JSON.
Các hàm `NewDecoder` và `NewEncoder` bao bọc các kiểu interface [`io.Reader`](/pkg/io/#Reader)
và [`io.Writer`](/pkg/io/#Writer).

	func NewDecoder(r io.Reader) *Decoder
	func NewEncoder(w io.Writer) *Encoder

Đây là một chương trình ví dụ đọc một loạt JSON object từ đầu vào chuẩn,
xóa tất cả trừ trường `Name` khỏi mỗi object,
rồi ghi các object vào đầu ra chuẩn:

	package main

	import (
	    "encoding/json"
	    "log"
	    "os"
	)

	func main() {
	    dec := json.NewDecoder(os.Stdin)
	    enc := json.NewEncoder(os.Stdout)
	    for {
	        var v map[string]interface{}
	        if err := dec.Decode(&v); err != nil {
	            log.Println(err)
	            return
	        }
	        for k := range v {
	            if k != "Name" {
	                delete(v, k)
	            }
	        }
	        if err := enc.Encode(&v); err != nil {
	            log.Println(err)
	        }
	    }
	}

Do sự phổ biến của Reader và Writer,
các kiểu `Encoder` và `Decoder` này có thể được sử dụng trong nhiều tình huống,
chẳng hạn như đọc và ghi vào kết nối HTTP,
WebSocket hoặc tệp.

## Tài liệu tham khảo

Để biết thêm thông tin, xem [tài liệu package json](/pkg/encoding/json/).
Để xem ví dụ sử dụng json, xem tệp nguồn của [package jsonrpc](/pkg/net/rpc/jsonrpc/).
