<!--{
  "Title": "Hướng dẫn: Bắt đầu với generics",
  "Breadcrumb": true
}-->

Hướng dẫn này giới thiệu các khái niệm cơ bản về generics trong Go. Với generics, bạn có thể khai báo và sử dụng các hàm hoặc kiểu được viết để hoạt động với bất kỳ kiểu nào trong một tập hợp các kiểu do mã gọi cung cấp.

Trong hướng dẫn này, bạn sẽ khai báo hai hàm đơn giản không dùng generics, sau đó gói gọn cùng một logic trong một hàm generic duy nhất.

Bạn sẽ lần lượt thực hiện các phần sau:

1. Tạo một thư mục cho mã của bạn.
2. Thêm các hàm không dùng generics.
3. Thêm một hàm generic để xử lý nhiều kiểu.
4. Loại bỏ các đối số kiểu khi gọi hàm generic.
5. Khai báo một ràng buộc kiểu.

**Lưu ý:** Đối với các hướng dẫn khác, xem [Tutorials](/doc/tutorial/index.html).

## Điều kiện tiên quyết

*   **Go.** Chúng tôi khuyên bạn nên sử dụng phiên bản Go mới nhất để làm theo hướng dẫn này.
    Để biết hướng dẫn cài đặt, xem [Installing Go](/doc/install).
*   **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt.
*   **Một terminal lệnh.** Go hoạt động tốt khi sử dụng bất kỳ terminal nào trên Linux và Mac,
    cũng như PowerShell hoặc cmd trên Windows.

## Tạo một thư mục cho mã của bạn {#create_folder}

Để bắt đầu, hãy tạo một thư mục cho mã bạn sẽ viết.

1. Mở dấu nhắc lệnh và chuyển đến thư mục nhà của bạn.

    Trên Linux hoặc Mac:

    ```
    $ cd
    ```

    Trên Windows:

    ```
    C:\> cd %HOMEPATH%
    ```

    Phần còn lại của hướng dẫn sẽ hiển thị $ làm dấu nhắc. Các lệnh bạn sử dụng
    cũng sẽ hoạt động trên Windows.

2. Từ dấu nhắc lệnh, tạo một thư mục cho mã của bạn có tên là generics.

    ```
    $ mkdir generics
    $ cd generics
    ```

3. Tạo một module để chứa mã của bạn.

    Chạy lệnh `go mod init`, cung cấp cho nó đường dẫn module của mã mới của bạn.

    ```
    $ go mod init example/generics
    go: creating new go.mod: module example/generics
    ```

    **Lưu ý:** Đối với mã dùng trong sản xuất, bạn nên chỉ định một đường dẫn module cụ thể hơn
    phù hợp với nhu cầu của riêng bạn. Để biết thêm, hãy xem
    [Managing dependencies](/doc/modules/managing-dependencies).

Tiếp theo, bạn sẽ thêm một số mã đơn giản để làm việc với các map.

## Thêm các hàm không phải generics {#non_generic_functions}

Trong bước này, bạn sẽ thêm hai hàm, mỗi hàm cộng các giá trị của một `map` và trả về tổng.

Bạn khai báo hai hàm thay vì một hàm vì bạn đang làm việc với hai kiểu `map` khác nhau: một kiểu lưu các giá trị `int64` và một kiểu lưu các giá trị `float64`.

#### Viết mã

1. Sử dụng trình soạn thảo văn bản của bạn, tạo một tệp có tên main.go trong thư mục generics. Bạn sẽ viết mã Go của mình trong tệp này.
2. Trong main.go, ở đầu tệp, dán khai báo package sau.

    ```
    package main
    ```

    Một chương trình độc lập (trái với thư viện) luôn nằm trong package `main`.

3. Bên dưới khai báo package, dán hai khai báo hàm sau.

    ```
    // SumInts cộng các giá trị của m.
    func SumInts(m map[string]int64) int64 {
    	var s int64
    	for _, v := range m {
    		s += v
    	}
    	return s
    }

    // SumFloats cộng các giá trị của m.
    func SumFloats(m map[string]float64) float64 {
    	var s float64
    	for _, v := range m {
    		s += v
    	}
    	return s
    }
    ```

    Trong mã này, bạn:

    * Khai báo hai hàm để cộng các giá trị của một `map` và trả về tổng.
        * `SumFloats` nhận một `map` từ `string` đến các giá trị `float64`.
        * `SumInts` nhận một `map` từ `string` đến các giá trị `int64`.

4. Ở đầu main.go, bên dưới khai báo package, dán hàm `main` sau để khởi tạo hai `map` và sử dụng chúng làm đối số khi gọi các hàm bạn đã khai báo trong bước trước.

    ```
    func main() {
    	// Khởi tạo một map cho các giá trị số nguyên
    	ints := map[string]int64{
    		"first":  34,
    		"second": 12,
    	}

    	// Khởi tạo một map cho các giá trị số thực
    	floats := map[string]float64{
    		"first":  35.98,
    		"second": 26.99,
    	}

    	fmt.Printf("Non-Generic Sums: %v and %v\n",
    		SumInts(ints),
    		SumFloats(floats))
    }
    ```

    Trong mã này, bạn:

    * Khởi tạo một `map` chứa các giá trị `float64` và một `map` chứa các giá trị `int64`, mỗi `map` có hai mục.
    * Gọi hai hàm bạn đã khai báo trước đó để tìm tổng các giá trị của mỗi `map`.
    * In kết quả.

5. Gần đầu main.go, ngay bên dưới khai báo package, nhập package bạn cần để hỗ trợ mã vừa viết.

    Các dòng mã đầu tiên sẽ có dạng như sau:

    ```
    package main

    import "fmt"
    ```

6. Lưu main.go.

#### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.

```
$ go run .
Non-Generic Sums: 46 and 62.97
```

Với generics, bạn có thể viết một hàm duy nhất ở đây thay vì hai hàm. Tiếp theo, bạn sẽ thêm một hàm generics duy nhất cho các map chứa giá trị kiểu số nguyên hoặc số thực.

## Thêm một hàm generics để xử lý nhiều kiểu {#add_generic_function}

Trong phần này, bạn sẽ thêm một hàm generics duy nhất có thể nhận một map chứa giá trị kiểu số nguyên hoặc số thực, thực chất thay thế hai hàm bạn vừa viết bằng một hàm duy nhất.

Để hỗ trợ các giá trị thuộc một trong hai kiểu, hàm duy nhất đó sẽ cần một cách để khai báo các kiểu mà nó hỗ trợ. Mặt khác, mã gọi hàm sẽ cần một cách để chỉ định liệu nó đang gọi với một map số nguyên hay map số thực.

Để hỗ trợ điều này, bạn sẽ viết một hàm khai báo _tham số kiểu_ bên cạnh các tham số hàm thông thường. Các tham số kiểu này làm cho hàm trở thành generics, cho phép hàm hoạt động với các đối số thuộc các kiểu khác nhau. Bạn sẽ gọi hàm bằng _đối số kiểu_ và các đối số hàm thông thường.

Mỗi tham số kiểu có một _ràng buộc kiểu_ đóng vai trò như một loại siêu kiểu cho tham số kiểu đó. Mỗi ràng buộc kiểu chỉ định các đối số kiểu được phép mà mã gọi hàm có thể sử dụng cho tham số kiểu tương ứng.

Mặc dù ràng buộc của một tham số kiểu thường biểu diễn một tập hợp các kiểu, tại thời điểm biên dịch, tham số kiểu đại diện cho một kiểu duy nhất – kiểu được cung cấp dưới dạng đối số kiểu bởi mã gọi hàm. Nếu kiểu của đối số kiểu không được ràng buộc của tham số kiểu cho phép, mã sẽ không biên dịch được.

Hãy nhớ rằng một tham số kiểu phải hỗ trợ tất cả các thao tác mà mã generics thực hiện trên nó. Ví dụ, nếu mã của hàm cố thực hiện các thao tác `string` (chẳng hạn như lập chỉ mục) trên một tham số kiểu có ràng buộc bao gồm các kiểu số, mã sẽ không biên dịch được.

Trong mã bạn sắp viết, bạn sẽ sử dụng một ràng buộc cho phép các kiểu số nguyên hoặc số thực.

#### Viết mã

1. Bên dưới hai hàm bạn đã thêm trước đó, dán hàm generic sau.

    ```
 // SumIntsOrFloats tính tổng các giá trị của map m. Nó hỗ trợ cả int64 và float64
 // làm các kiểu cho giá trị của map.
    func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
        var s V
        for _, v := range m {
            s += v
        }
        return s
    }
    ```

Trong mã này, bạn:

*   Khai báo hàm `SumIntsOrFloats` với hai tham số kiểu (bên trong
    dấu ngoặc vuông), `K` và `V`, cùng một đối số sử dụng các tham số kiểu,
    `m` có kiểu `map[K]V`. Hàm trả về một giá trị có kiểu `V`.
*   Chỉ định ràng buộc kiểu `comparable` cho tham số kiểu `K`.
    Được thiết kế riêng cho các trường hợp như thế này, ràng buộc
    `comparable` được khai báo sẵn trong Go. Nó cho phép mọi kiểu có giá trị
    có thể được dùng làm toán hạng của các toán tử so sánh `==` và `!=`. Go
    yêu cầu khóa của map phải có thể so sánh được. Vì vậy, khai báo `K` là
    `comparable` là cần thiết để bạn có thể dùng `K` làm khóa trong biến map.
    Nó cũng đảm bảo mã gọi sử dụng một kiểu hợp lệ cho khóa map.
*   Chỉ định cho tham số kiểu `V` một ràng buộc là hợp của hai kiểu:
    `int64` và `float64`. Việc dùng `|` chỉ định một hợp của hai kiểu,
    nghĩa là ràng buộc này cho phép một trong hai kiểu. Cả hai kiểu đều
    sẽ được trình biên dịch cho phép làm đối số trong mã gọi.
*   Chỉ định rằng đối số `m` có kiểu `map[K]V`, trong đó `K` và `V`
    là các kiểu đã được chỉ định cho các tham số kiểu. Lưu ý rằng chúng ta
    biết `map[K]V` là một kiểu map hợp lệ vì `K` là một kiểu có thể so sánh.
    Nếu chúng ta không khai báo `K` là `comparable`, trình biên dịch sẽ từ chối
    tham chiếu đến `map[K]V`.

2. Trong main.go, bên dưới mã bạn đã có, dán đoạn mã sau.

    ```
    fmt.Printf("Generic Sums: %v and %v\n",
    	SumIntsOrFloats[string, int64](ints),
    	SumIntsOrFloats[string, float64](floats))
    ```

Trong mã này, bạn:

*   Gọi hàm generic vừa khai báo, truyền vào từng map bạn đã tạo.
*   Chỉ định các đối số kiểu – tên các kiểu trong dấu ngoặc vuông – để
    làm rõ các kiểu nào sẽ thay thế các tham số kiểu trong hàm bạn đang gọi.

    Như bạn sẽ thấy trong phần tiếp theo, bạn thường có thể bỏ qua các đối số
    kiểu trong lời gọi hàm. Go thường có thể suy ra chúng từ mã của bạn.
*   In các tổng được hàm trả về.

#### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
```

Để chạy mã của bạn, trong mỗi lần gọi, trình biên dịch đã thay thế các tham số kiểu bằng các kiểu cụ thể được chỉ định trong lần gọi đó.

Khi gọi hàm generic mà bạn đã viết, bạn đã chỉ định các đối số kiểu cho trình biên dịch biết cần dùng kiểu nào thay cho các tham số kiểu của hàm. Như bạn sẽ thấy trong phần tiếp theo, trong nhiều trường hợp bạn có thể bỏ qua các đối số kiểu này vì trình biên dịch có thể suy luận chúng.

## Xóa các đối số kiểu khi gọi hàm generic {#remove_type_arguments}

Trong phần này, bạn sẽ thêm một phiên bản đã sửa đổi của lời gọi hàm generic, thực hiện một thay đổi nhỏ để đơn giản hóa mã gọi. Bạn sẽ xóa các đối số kiểu, vì chúng không cần thiết trong trường hợp này.

Bạn có thể bỏ qua các đối số kiểu trong mã gọi khi trình biên dịch Go có thể suy luận các kiểu bạn muốn dùng. Trình biên dịch suy luận các đối số kiểu từ các kiểu của đối số hàm.

Lưu ý rằng điều này không phải lúc nào cũng có thể thực hiện được. Ví dụ: nếu bạn cần gọi một hàm generic không có đối số, bạn sẽ cần đưa các đối số kiểu vào lời gọi hàm.

#### Viết mã

*   Trong main.go, bên dưới mã bạn đã có, dán đoạn mã sau.

    ```
    fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
    	SumIntsOrFloats(ints),
    	SumIntsOrFloats(floats))
    ```

    Trong mã này, bạn:

    *   Gọi hàm generic, bỏ qua các đối số kiểu.

#### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
```

Tiếp theo, bạn sẽ tiếp tục đơn giản hóa hàm bằng cách gom hợp của các số nguyên và số thực vào một ràng buộc kiểu mà bạn có thể tái sử dụng, chẳng hạn như từ mã khác.

## Khai báo ràng buộc kiểu {#declare_type_constraint}

Trong phần cuối này, bạn sẽ chuyển ràng buộc đã định nghĩa trước đó vào interface riêng để có thể tái sử dụng ở nhiều nơi. Khai báo các ràng buộc theo cách này giúp đơn giản hóa mã, chẳng hạn khi một ràng buộc phức tạp hơn.

Bạn khai báo một _ràng buộc kiểu_ dưới dạng một interface. Ràng buộc cho phép bất kỳ kiểu nào triển khai interface đó. Ví dụ: nếu bạn khai báo một interface ràng buộc kiểu có ba phương thức, sau đó dùng nó với một tham số kiểu trong một hàm generic, các đối số kiểu được dùng để gọi hàm phải có tất cả các phương thức đó.

Các interface ràng buộc cũng có thể tham chiếu đến các kiểu cụ thể, như bạn sẽ thấy trong phần này.

#### Viết mã

1. Ngay phía trên `main`, ngay sau các câu lệnh import, dán đoạn mã sau để khai báo một ràng buộc kiểu.

    ```
    type Number interface {
        int64 | float64
    }
    ```

Trong đoạn mã này, bạn:

*   Khai báo kiểu `interface` `Number` để sử dụng làm ràng buộc kiểu.
*   Khai báo một phép hợp của `int64` và `float64` bên trong interface.

    Về cơ bản, bạn đang di chuyển phép hợp từ khai báo hàm vào một ràng buộc kiểu mới. Bằng cách đó, khi muốn giới hạn một tham số kiểu chỉ có thể là `int64` hoặc `float64`, bạn có thể sử dụng ràng buộc kiểu `Number` này thay vì viết trực tiếp `int64 | float64`.

2. Bên dưới các hàm bạn đã có, dán hàm generic `SumNumbers` sau.

    ```
// SumNumbers tính tổng các giá trị của map m. Nó hỗ trợ cả số nguyên
// và số thực làm giá trị của map.
    func SumNumbers[K comparable, V Number](m map[K]V) V {
        var s V
        for _, v := range m {
            s += v
        }
        return s
    }
    ```

Trong đoạn mã này, bạn:

*   Khai báo một hàm generic có cùng logic với hàm generic bạn đã khai báo trước đó, nhưng sử dụng kiểu `interface` mới thay vì phép hợp làm ràng buộc kiểu. Như trước đây, bạn sử dụng các tham số kiểu cho kiểu của đối số và kiểu trả về.

3. Trong main.go, bên dưới mã bạn đã có, dán đoạn mã sau.

    ```
    fmt.Printf("Generic Sums with Constraint: %v and %v\n",
    	SumNumbers(ints),
    	SumNumbers(floats))
    ```

Trong đoạn mã này, bạn:

*   Gọi `SumNumbers` với từng map, in ra tổng từ các giá trị của từng map.

    Giống như trong phần trước, bạn bỏ qua các đối số kiểu (tên kiểu trong dấu ngoặc vuông) trong các lệnh gọi đến hàm generic. Trình biên dịch Go có thể suy ra đối số kiểu từ các đối số khác.

#### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
Generic Sums with Constraint: 46 and 62.97
```

## Kết luận {#conclusion}

Làm tốt lắm! Bạn vừa làm quen với generics trong Go.

Các chủ đề tiếp theo được đề xuất:

* [Go Tour](/tour/) là một bước tiếp theo tuyệt vời để giới thiệu từng bước về các khái niệm cơ bản của Go.
* Bạn sẽ tìm thấy các phương pháp hay hữu ích của Go được mô tả trong [Effective Go](/doc/effective_go) và [Cách viết mã Go](/doc/code).

## Mã đã hoàn thành {#completed_code}

Bạn có thể chạy chương trình này trong [Go playground](/play/p/apNmfVwogK0). Trên playground, chỉ cần nhấp vào nút **Run**.

```
package main

import "fmt"

type Number interface {
	int64 | float64
}

func main() {
	// Khởi tạo một map cho các giá trị số nguyên
	ints := map[string]int64{
		"first": 34,
		"second": 12,
	}

	// Khởi tạo một map cho các giá trị số thực
	floats := map[string]float64{
		"first": 35.98,
		"second": 26.99,
	}

	fmt.Printf("Non-Generic Sums: %v and %v\n",
		SumInts(ints),
		SumFloats(floats))

	fmt.Printf("Generic Sums: %v and %v\n",
		SumIntsOrFloats[string, int64](ints),
		SumIntsOrFloats[string, float64](floats))

	fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
		SumIntsOrFloats(ints),
		SumIntsOrFloats(floats))

	fmt.Printf("Generic Sums with Constraint: %v and %v\n",
		SumNumbers(ints),
		SumNumbers(floats))
}

// SumInts cộng các giá trị của m với nhau.
func SumInts(m map[string]int64) int64 {
	var s int64
	for _, v := range m {
		s += v
	}
	return s
}

// SumFloats cộng các giá trị của m với nhau.
func SumFloats(m map[string]float64) float64 {
	var s float64
	for _, v := range m {
		s += v
	}
	return s
}

// SumIntsOrFloats tính tổng các giá trị của map m. Nó hỗ trợ cả số thực và số nguyên
// làm giá trị của map.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}

// SumNumbers tính tổng các giá trị của map m. Nó hỗ trợ cả số nguyên
// và số thực làm giá trị của map.
func SumNumbers[K comparable, V Number](m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}

```
