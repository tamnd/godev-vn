---
title: JSON và Go
date: 2011-01-25
by:
- Andrew Gerrand
tags:
- json
- technical
summary: Cách tạo và sử dụng dữ liệu được định dạng JSON trong Go.
---

[_**Lưu ý, tháng 8 năm 2026**: Go 1.27 giới thiệu gói `encoding/json/v2` mới.  
Để xem phần giới thiệu được cập nhật về cách làm việc với JSON trong Go, hãy xem “[Hướng dẫn: Làm việc với JSON](/doc/tutorial/json).”_]

## Giới thiệu

JSON (JavaScript Object Notation) là một định dạng trao đổi dữ liệu đơn giản.
Về mặt cú pháp, nó giống với các đối tượng và danh sách của JavaScript.
Nó thường được sử dụng nhất để giao tiếp giữa các phần phụ trợ web và các
chương trình JavaScript chạy trong trình duyệt,
nhưng cũng được sử dụng ở nhiều nơi khác.
Trang chủ của nó, [json.org](http://json.org),
cung cấp một định nghĩa tiêu chuẩn rất rõ ràng và súc tích.

Với [gói json](/pkg/encoding/json/) bạn có thể dễ dàng đọc và ghi dữ liệu JSON
từ các chương trình Go của mình.

## Mã hóa

Để mã hóa dữ liệu JSON, chúng ta sử dụng hàm [`Marshal`](/pkg/encoding/json/#Marshal).

	func Marshal(v interface{}) ([]byte, error)

Với cấu trúc dữ liệu Go, `Message`,

	type Message struct {
	    Name string
	    Body string
	    Time int64
	}

và một thực thể của `Message`

	m := Message{"Alice", "Hello", 1294706395881547000}

chúng ta có thể marshal phiên bản được mã hóa JSON của m bằng `json.Marshal`:

	b, err := json.Marshal(m)

Nếu mọi thứ đều đúng, `err` sẽ là `nil` và `b` sẽ là một `[]byte` chứa dữ liệu JSON này:

	b == []byte(`{"Name":"Alice","Body":"Hello","Time":1294706395881547000}`)

Chỉ các cấu trúc dữ liệu có thể được biểu diễn dưới dạng JSON hợp lệ mới được mã hóa:

  - Đối tượng JSON chỉ hỗ trợ chuỗi làm khóa;
    để mã hóa một kiểu map của Go, nó phải có dạng `map[string]T` (trong đó `T`
    là bất kỳ kiểu Go nào được gói json hỗ trợ).

  - Các kiểu channel, complex và function không thể được mã hóa.

  - Các cấu trúc dữ liệu tuần hoàn không được hỗ trợ; chúng sẽ khiến `Marshal` rơi vào vòng lặp vô hạn.

  - Con trỏ sẽ được mã hóa thành các giá trị mà chúng trỏ tới (hoặc 'null' nếu con trỏ là `nil`).

Gói json chỉ truy cập các trường được export của kiểu struct (những trường
bắt đầu bằng một chữ cái viết hoa).
Do đó, chỉ các trường được export của một struct mới xuất hiện trong đầu ra JSON.

## Giải mã

Để giải mã dữ liệu JSON, chúng ta sử dụng hàm [`Unmarshal`](/pkg/encoding/json/#Unmarshal).

	func Unmarshal(data []byte, v interface{}) error

Trước tiên, chúng ta phải tạo một nơi để lưu trữ dữ liệu đã được giải mã

	var m Message

và gọi `json.Unmarshal`, truyền cho nó một `[]byte` chứa dữ liệu JSON cùng với con trỏ tới `m`

	err := json.Unmarshal(b, &m)

Nếu `b` chứa JSON hợp lệ phù hợp với `m`,
sau lời gọi `err` sẽ là `nil` và dữ liệu từ `b` sẽ được
lưu vào struct `m`,
giống như thực hiện phép gán:

	m = Message{
	    Name: "Alice",
	    Body: "Hello",
	    Time: 1294706395881547000,
	}

Làm thế nào `Unmarshal` xác định các trường để lưu dữ liệu đã giải mã?
Với một khóa JSON cụ thể `"Foo"`,
`Unmarshal` sẽ tìm qua các trường của struct đích để tìm (theo
thứ tự ưu tiên):

  - Một trường được export có tag là `"Foo"` (xem [đặc tả Go](/ref/spec#Struct_types)
    để biết thêm về tag của struct),

  - Một trường được export có tên `"Foo"`, hoặc

  - Một trường được export có tên `"FOO"` hoặc `"FoO"` hoặc một tên khác khớp với `"Foo"` mà không phân biệt chữ hoa chữ thường.

Điều gì xảy ra khi cấu trúc của dữ liệu JSON không hoàn toàn khớp với kiểu Go?

	b := []byte(`{"Name":"Bob","Food":"Pickle"}`)
	var m Message
	err := json.Unmarshal(b, &m)

`Unmarshal` sẽ chỉ giải mã các trường mà nó có thể tìm thấy trong kiểu đích.
Trong trường hợp này, chỉ trường Name của m được điền giá trị,
còn trường Food sẽ bị bỏ qua.
Hành vi này đặc biệt hữu ích khi bạn muốn chỉ lấy một vài trường cụ thể
từ một khối JSON lớn.
Điều này cũng có nghĩa là mọi trường không được export trong struct đích sẽ
không bị ảnh hưởng bởi `Unmarshal`.

Nhưng nếu bạn không biết trước cấu trúc của dữ liệu JSON thì sao?

## JSON tổng quát với interface{}

Kiểu `interface{}` (interface rỗng) mô tả một interface không có phương thức nào.
Mọi kiểu Go đều triển khai ít nhất không phương thức nào và do đó thỏa mãn interface rỗng.

Interface rỗng đóng vai trò là một kiểu chứa tổng quát:

	var i interface{}
	i = "a string"
	i = 2011
	i = 2.777

Phép khẳng định kiểu truy cập kiểu cụ thể bên dưới:

	r := i.(float64)
	fmt.Println("the circle's area", math.Pi*r*r)

Hoặc, nếu kiểu bên dưới chưa biết, phép chuyển đổi kiểu xác định kiểu đó:

	switch v := i.(type) {
	case int:
	    fmt.Println("twice i is", v*2)
	case float64:
	    fmt.Println("the reciprocal of i is", 1/v)
	case string:
	    h := len(v) / 2
	    fmt.Println("i swapped by halves is", v[h:]+v[:h])
	default:
	    // i không phải là một trong các kiểu ở trên
	}

Gói json sử dụng các giá trị `map[string]interface{}` và
`[]interface{}` để lưu trữ các đối tượng và mảng JSON tùy ý;
nó sẽ giải mã thành công mọi khối JSON hợp lệ vào một giá trị
`interface{}` thuần. Các kiểu Go cụ thể mặc định là:

  - `bool` cho các giá trị boolean JSON,

  - `float64` cho các số JSON,

  - `string` cho các chuỗi JSON, và

  - `nil` cho giá trị null JSON.

## Giải mã dữ liệu tùy ý

Hãy xem xét dữ liệu JSON này, được lưu trong biến `b`:

	b := []byte(`{"Name":"Wednesday","Age":6,"Parents":["Gomez","Morticia"]}`)

Không biết cấu trúc của dữ liệu này, chúng ta có thể giải mã nó thành một giá trị `interface{}` bằng `Unmarshal`:

	var f interface{}
	err := json.Unmarshal(b, &f)

Tại thời điểm này, giá trị Go trong `f` sẽ là một map có các khóa là chuỗi
và các giá trị của nó được lưu dưới dạng các giá trị interface rỗng:

	f = map[string]interface{}{
	    "Name": "Wednesday",
	    "Age":  6,
	    "Parents": []interface{}{
	        "Gomez",
	        "Morticia",
	    },
	}

Để truy cập dữ liệu này, chúng ta có thể sử dụng phép khẳng định kiểu để truy cập `map[string]interface{}` bên dưới của `f`:

	m := f.(map[string]interface{})

Sau đó, chúng ta có thể lặp qua map bằng câu lệnh range và sử dụng type
switch để truy cập các giá trị của nó theo các kiểu cụ thể:

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

Bằng cách này, bạn có thể làm việc với dữ liệu JSON chưa biết trước trong khi vẫn tận dụng được lợi ích của an toàn kiểu.

## Các kiểu tham chiếu

Hãy định nghĩa một kiểu Go để chứa dữ liệu từ ví dụ trước:

	type FamilyMember struct {
	    Name    string
	    Age     int
	    Parents []string
	}

	var m FamilyMember
	err := json.Unmarshal(b, &m)

Việc giải mã dữ liệu đó vào một giá trị `FamilyMember` hoạt động như mong đợi,
nhưng nếu xem xét kỹ, chúng ta có thể thấy một điều đáng chú ý đã xảy ra.
Với câu lệnh var, chúng ta đã cấp phát một struct `FamilyMember`,
sau đó cung cấp một con trỏ đến giá trị đó cho `Unmarshal`,
nhưng tại thời điểm đó trường `Parents` là một giá trị slice `nil`.
Để điền dữ liệu vào trường `Parents`, `Unmarshal` đã cấp phát một slice mới ở phía sau.
Đây là cách `Unmarshal` thường hoạt động với các kiểu tham chiếu được hỗ trợ
(con trỏ, slice và map).

Hãy xem xét việc giải mã vào cấu trúc dữ liệu này:

	type Foo struct {
	    Bar *Bar
	}

Nếu có trường `Bar` trong đối tượng JSON,
`Unmarshal` sẽ cấp phát một `Bar` mới và điền dữ liệu cho nó.
Nếu không, `Bar` sẽ được giữ nguyên dưới dạng một con trỏ `nil`.

Từ đây xuất hiện một mẫu hữu ích: nếu bạn có một ứng dụng nhận
một vài kiểu thông báo riêng biệt,
bạn có thể định nghĩa cấu trúc "receiver" như sau

	type IncomingMessage struct {
	    Cmd *Command
	    Msg *Message
	}

và bên gửi có thể điền vào trường `Cmd` và/hoặc trường `Msg`
của đối tượng JSON cấp cao nhất,
tùy thuộc vào kiểu thông báo mà họ muốn truyền đạt.
`Unmarshal`, khi giải mã JSON vào một struct `IncomingMessage`,
sẽ chỉ cấp phát các cấu trúc dữ liệu có trong dữ liệu JSON.
Để biết cần xử lý những thông báo nào, lập trình viên chỉ cần kiểm tra
rằng `Cmd` hoặc `Msg` không phải là `nil`.

## Bộ mã hóa và giải mã luồng

Gói json cung cấp các kiểu `Decoder` và `Encoder` để hỗ trợ thao tác phổ biến là đọc và ghi các luồng dữ liệu JSON.
Các hàm `NewDecoder` và `NewEncoder` bao bọc các kiểu interface [`io.Reader`](/pkg/io/#Reader)
và [`io.Writer`](/pkg/io/#Writer).

	func NewDecoder(r io.Reader) *Decoder
	func NewEncoder(w io.Writer) *Encoder

Sau đây là một chương trình ví dụ đọc một chuỗi đối tượng JSON từ đầu vào chuẩn,
loại bỏ tất cả các trường ngoại trừ trường `Name` khỏi mỗi đối tượng,
sau đó ghi các đối tượng ra đầu ra chuẩn:

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

Do tính phổ biến của các Reader và Writer,
các kiểu `Encoder` và `Decoder` này có thể được sử dụng trong nhiều tình huống khác nhau,
chẳng hạn như đọc và ghi tới các kết nối HTTP,
WebSockets hoặc tệp.

## Tài liệu tham khảo

Để biết thêm thông tin, hãy xem [tài liệu về gói json](/pkg/encoding/json/).
Để xem ví dụ sử dụng json, hãy xem các tệp mã nguồn của [gói jsonrpc](/pkg/net/rpc/jsonrpc/).
