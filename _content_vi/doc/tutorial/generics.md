<!--{
  "Title": "Hướng dẫn: Bắt đầu với generics",
  "Breadcrumb": true
}-->

Hướng dẫn này giới thiệu những kiến thức cơ bản về generics trong Go. Với generics, bạn có thể
khai báo và sử dụng các hàm hoặc kiểu dữ liệu được viết để hoạt động với bất kỳ kiểu nào
trong một tập hợp các kiểu được cung cấp bởi code gọi hàm.

Trong hướng dẫn này, bạn sẽ khai báo hai hàm non-generic đơn giản, sau đó
gói gọn logic tương tự trong một hàm generic duy nhất.

Bạn sẽ thực hiện lần lượt các phần sau:

1. Tạo thư mục cho code của bạn.
2. Thêm các hàm non-generic.
3. Thêm một hàm generic để xử lý nhiều kiểu.
4. Bỏ các type argument khi gọi hàm generic.
5. Khai báo một type constraint.

**Lưu ý:** Để xem các hướng dẫn khác, truy cập [Hướng dẫn](/doc/tutorial/index.html).

**Lưu ý:** Nếu bạn thích, bạn có thể dùng
[Go playground ở chế độ "Go dev branch"](/play/?v=gotip)
để chỉnh sửa và chạy chương trình thay thế.

## Điều kiện tiên quyết

*   **Đã cài đặt Go 1.18 hoặc mới hơn.** Để biết hướng dẫn cài đặt, xem
    [Cài đặt Go](/doc/install).
*   **Một công cụ để chỉnh sửa code.** Bất kỳ trình soạn thảo văn bản nào bạn có đều dùng được.
*   **Một cửa sổ dòng lệnh.** Go hoạt động tốt trên bất kỳ terminal nào trên Linux và Mac,
    cũng như trên PowerShell hoặc cmd trong Windows.

## Tạo thư mục cho code của bạn {#create_folder}

Để bắt đầu, hãy tạo một thư mục cho code bạn sẽ viết.

1. Mở dấu nhắc lệnh và chuyển đến thư mục home của bạn.

    Trên Linux hoặc Mac:

    ```
    $ cd
    ```

    Trên Windows:

    ```
    C:\> cd %HOMEPATH%
    ```

    Phần còn lại của hướng dẫn sẽ dùng $ làm dấu nhắc lệnh. Các lệnh bạn sử dụng
    cũng hoạt động trên Windows.

2. Từ dấu nhắc lệnh, tạo một thư mục có tên generics.

    ```
    $ mkdir generics
    $ cd generics
    ```

3. Tạo một module để chứa code của bạn.

    Chạy lệnh `go mod init`, cung cấp đường dẫn module cho code mới của bạn.

    ```
    $ go mod init example/generics
    go: creating new go.mod: module example/generics
    ```

    **Lưu ý:** Với code production, bạn sẽ chỉ định đường dẫn module cụ thể hơn
    theo nhu cầu của mình. Để biết thêm, hãy xem
    [Quản lý dependency](/doc/modules/managing-dependencies).

Tiếp theo, bạn sẽ thêm một ít code đơn giản để làm việc với map.

## Thêm các hàm non-generic {#non_generic_functions}

Trong bước này, bạn sẽ thêm hai hàm, mỗi hàm cộng tổng các giá trị của một
map và trả về tổng.

Bạn khai báo hai hàm thay vì một vì bạn đang làm việc với hai
kiểu map khác nhau: một kiểu lưu trữ giá trị `int64` và một kiểu lưu trữ giá trị `float64`.

#### Viết code

1. Dùng trình soạn thảo văn bản của bạn, tạo một file có tên main.go trong thư mục generics.
    Bạn sẽ viết code Go của mình trong file này.
2. Vào main.go, ở đầu file, dán phần khai báo package sau.

    ```
    package main
    ```

    Một chương trình độc lập (trái với thư viện) luôn ở trong gói `main`.

3. Bên dưới khai báo package, dán hai phần khai báo hàm sau.

    ```
    // SumInts adds together the values of m.
    func SumInts(m map[string]int64) int64 {
    	var s int64
    	for _, v := range m {
    		s += v
    	}
    	return s
    }

    // SumFloats adds together the values of m.
    func SumFloats(m map[string]float64) float64 {
    	var s float64
    	for _, v := range m {
    		s += v
    	}
    	return s
    }
    ```

    Trong đoạn code này, bạn:

    *   Khai báo hai hàm để cộng tổng các giá trị của một map và trả về
        tổng.
        *   `SumFloats` nhận một map từ `string` đến giá trị `float64`.
        *   `SumInts` nhận một map từ `string` đến giá trị `int64`.

4. Ở đầu main.go, bên dưới khai báo package, dán hàm `main` sau để khởi tạo hai map và sử dụng chúng làm đối số khi gọi các hàm đã khai báo ở bước trên.

    ```
    func main() {
    	// Initialize a map for the integer values
    	ints := map[string]int64{
    		"first":  34,
    		"second": 12,
    	}

    	// Initialize a map for the float values
    	floats := map[string]float64{
    		"first":  35.98,
    		"second": 26.99,
    	}

    	fmt.Printf("Non-Generic Sums: %v and %v\n",
    		SumInts(ints),
    		SumFloats(floats))
    }
    ```

    Trong đoạn code này, bạn:

    *   Khởi tạo một map giá trị `float64` và một map giá trị `int64`, mỗi map có hai mục.
    *   Gọi hai hàm đã khai báo trước đó để tìm tổng các giá trị của mỗi map.
    *   In kết quả.

5. Gần đầu main.go, ngay bên dưới khai báo package, import
    gói bạn cần để hỗ trợ code vừa viết.

    Các dòng đầu tiên của code nên trông như sau:

    ```
    package main

    import "fmt"
    ```

6. Lưu main.go.

#### Chạy code

Từ dòng lệnh trong thư mục chứa main.go, chạy code.

```
$ go run .
Non-Generic Sums: 46 and 62.97
```

Với generics, bạn có thể viết một hàm ở đây thay vì hai. Tiếp theo, bạn sẽ
thêm một hàm generic duy nhất cho các map chứa giá trị integer hoặc float.

## Thêm một hàm generic để xử lý nhiều kiểu {#add_generic_function}

Trong phần này, bạn sẽ thêm một hàm generic duy nhất có thể nhận một map
chứa giá trị integer hoặc float, thực sự thay thế hai
hàm bạn vừa viết bằng một hàm duy nhất.

Để hỗ trợ giá trị của cả hai kiểu, hàm duy nhất đó sẽ cần một cách
khai báo những kiểu nó hỗ trợ. Ngược lại, code gọi hàm sẽ cần một cách
để chỉ định liệu nó đang gọi với map integer hay float.

Để hỗ trợ điều này, bạn sẽ viết một hàm khai báo _type parameter_ bổ sung
cho các tham số hàm thông thường của nó. Các type parameter này làm cho
hàm trở nên generic, cho phép nó hoạt động với các đối số kiểu khác nhau. Bạn sẽ
gọi hàm với _type argument_ và các đối số hàm thông thường.

Mỗi type parameter có một _type constraint_ hoạt động như một loại meta-type
cho type parameter đó. Mỗi type constraint chỉ định các type argument được phép
mà code gọi hàm có thể sử dụng cho type parameter tương ứng.

Trong khi một type constraint thường đại diện cho một tập hợp các kiểu, tại
thời điểm biên dịch, type parameter đại diện cho một kiểu duy nhất, là kiểu được cung cấp
làm type argument bởi code gọi hàm. Nếu kiểu của type argument không
được phép bởi type constraint của type parameter, code sẽ không biên dịch được.

Hãy nhớ rằng một type parameter phải hỗ trợ tất cả các thao tác mà code generic
đang thực hiện trên nó. Ví dụ, nếu code hàm của bạn cố gắng thực hiện
các thao tác `string` (chẳng hạn như indexing) trên một type parameter mà
constraint bao gồm các kiểu số, code sẽ không biên dịch được.

Trong code bạn sắp viết, bạn sẽ sử dụng một constraint cho phép
kiểu integer hoặc float.

#### Viết code

1. Bên dưới hai hàm bạn đã thêm trước đó, dán hàm generic sau.

    ```
    // SumIntsOrFloats sums the values of map m. It supports both int64 and float64
    // as types for map values.
    func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
        var s V
        for _, v := range m {
            s += v
        }
        return s
    }
    ```

    Trong đoạn code này, bạn:

    *   Khai báo hàm `SumIntsOrFloats` với hai type parameter (bên trong
        dấu ngoặc vuông), `K` và `V`, và một đối số sử dụng các type
        parameter, `m` kiểu `map[K]V`. Hàm trả về một giá trị kiểu `V`.
    *   Chỉ định cho type parameter `K` type constraint `comparable`.
        Được thiết kế đặc biệt cho các trường hợp như thế này, constraint `comparable`
        được khai báo sẵn trong Go. Nó cho phép bất kỳ kiểu nào mà các giá trị có thể được dùng làm
        toán hạng của các toán tử so sánh `==` và `!=`. Go yêu cầu khóa map
        phải có thể so sánh được. Vì vậy, khai báo `K` là `comparable` là cần thiết để bạn
        có thể dùng `K` làm khóa trong biến map. Nó cũng đảm bảo rằng code gọi hàm
        sử dụng kiểu được phép cho khóa map.
    *   Chỉ định cho type parameter `V` một constraint là hợp của hai
        kiểu: `int64` và `float64`. Dùng `|` chỉ định hợp của hai
        kiểu, có nghĩa là constraint này cho phép cả hai kiểu. Cả hai kiểu
        sẽ được compiler chấp nhận làm đối số trong code gọi hàm.
    *   Chỉ định rằng đối số `m` kiểu `map[K]V`, trong đó `K` và `V`
        là các kiểu đã chỉ định cho các type parameter. Lưu ý rằng chúng ta
        biết `map[K]V` là kiểu map hợp lệ vì `K` là kiểu có thể so sánh được. Nếu
        chúng ta không khai báo `K` là comparable, compiler sẽ từ chối
        tham chiếu đến `map[K]V`.

2. Trong main.go, bên dưới code bạn đã có, dán đoạn code sau.

    ```
    fmt.Printf("Generic Sums: %v and %v\n",
    	SumIntsOrFloats[string, int64](ints),
    	SumIntsOrFloats[string, float64](floats))
    ```

    Trong đoạn code này, bạn:

    *   Gọi hàm generic vừa khai báo, truyền vào mỗi map bạn đã tạo.
    *   Chỉ định type argument, là tên kiểu trong dấu ngoặc vuông, để
        làm rõ các kiểu nên thay thế type parameter trong
        hàm bạn đang gọi.

        Như bạn sẽ thấy trong phần tiếp theo, thường bạn có thể bỏ qua các type
        argument trong lần gọi hàm. Go thường có thể suy ra chúng từ code của bạn.
    *   In các tổng được hàm trả về.

#### Chạy code

Từ dòng lệnh trong thư mục chứa main.go, chạy code.

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
```

Để chạy code của bạn, trong mỗi lần gọi, compiler thay thế các type parameter bằng
các kiểu cụ thể được chỉ định trong lần gọi đó.

Khi gọi hàm generic bạn đã viết, bạn đã chỉ định type argument cho compiler biết
loại kiểu nào sẽ thay thế các type parameter của hàm.
Như bạn sẽ thấy trong phần tiếp theo, trong nhiều trường hợp bạn có thể bỏ qua các type
argument này vì compiler có thể suy ra chúng.

## Bỏ type argument khi gọi hàm generic {#remove_type_arguments}

Trong phần này, bạn sẽ thêm một phiên bản sửa đổi của lần gọi hàm generic,
thực hiện một thay đổi nhỏ để đơn giản hóa code gọi hàm. Bạn sẽ bỏ
type argument, vốn không cần thiết trong trường hợp này.

Bạn có thể bỏ qua type argument trong code gọi hàm khi compiler Go có thể suy ra
các kiểu bạn muốn sử dụng. Compiler suy ra type argument từ các kiểu của
đối số hàm.

Lưu ý rằng điều này không phải lúc nào cũng có thể. Ví dụ, nếu bạn cần gọi một
hàm generic không có đối số, bạn sẽ cần bao gồm type argument trong lần gọi hàm.

#### Viết code

*   Trong main.go, bên dưới code bạn đã có, dán đoạn code sau.

    ```
    fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
    	SumIntsOrFloats(ints),
    	SumIntsOrFloats(floats))
    ```

    Trong đoạn code này, bạn:

    *   Gọi hàm generic, bỏ qua type argument.

#### Chạy code

Từ dòng lệnh trong thư mục chứa main.go, chạy code.

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
```

Tiếp theo, bạn sẽ đơn giản hóa hơn nữa hàm bằng cách gói gọn hợp của integer
và float vào một type constraint có thể tái sử dụng, chẳng hạn từ code khác.

## Khai báo một type constraint {#declare_type_constraint}

Trong phần cuối này, bạn sẽ chuyển constraint đã định nghĩa trước đó vào
interface riêng của nó để có thể tái sử dụng ở nhiều nơi. Việc khai báo
constraint theo cách này giúp đơn giản hóa code, chẳng hạn khi constraint
phức tạp hơn.

Bạn khai báo _type constraint_ dưới dạng interface. Constraint cho phép bất kỳ
kiểu nào triển khai interface. Ví dụ, nếu bạn khai báo một interface type constraint
với ba phương thức, rồi dùng nó với một type parameter trong hàm generic,
các type argument được dùng để gọi hàm phải có tất cả các phương thức đó.

Constraint interface cũng có thể tham chiếu đến các kiểu cụ thể, như bạn sẽ thấy trong
phần này.

#### Viết code

1. Ngay phía trên `main`, ngay sau các câu lệnh import, dán
    đoạn code sau để khai báo một type constraint.

    ```
    type Number interface {
        int64 | float64
    }
    ```

    Trong đoạn code này, bạn:

    *   Khai báo kiểu interface `Number` để dùng làm type constraint.
    *   Khai báo hợp của `int64` và `float64` bên trong interface.

        Về bản chất, bạn đang chuyển hợp từ khai báo hàm
        vào một type constraint mới. Bằng cách đó, khi bạn muốn giới hạn một type
        parameter là `int64` hoặc `float64`, bạn có thể dùng type constraint `Number`
        này thay vì viết ra `int64 | float64`.

2. Bên dưới các hàm bạn đã có, dán hàm generic `SumNumbers` sau.

    ```
    // SumNumbers sums the values of map m. It supports both integers
    // and floats as map values.
    func SumNumbers[K comparable, V Number](m map[K]V) V {
        var s V
        for _, v := range m {
            s += v
        }
        return s
    }
    ```

    Trong đoạn code này, bạn:

    *   Khai báo một hàm generic với logic giống hàm generic
        đã khai báo trước đó, nhưng với kiểu interface mới thay vì
        hợp làm type constraint. Như trước, bạn dùng type parameter
        cho kiểu đối số và trả về.

3. Trong main.go, bên dưới code bạn đã có, dán đoạn code sau.

    ```
    fmt.Printf("Generic Sums with Constraint: %v and %v\n",
    	SumNumbers(ints),
    	SumNumbers(floats))
    ```

    Trong đoạn code này, bạn:

    *   Gọi `SumNumbers` với mỗi map, in tổng các giá trị của từng map.

        Như ở phần trước, bạn bỏ qua type argument (tên kiểu trong dấu ngoặc vuông)
        trong các lần gọi hàm generic. Compiler Go có thể suy ra type argument
        từ các đối số khác.

#### Chạy code

Từ dòng lệnh trong thư mục chứa main.go, chạy code.

```
$ go run .
Non-Generic Sums: 46 and 62.97
Generic Sums: 46 and 62.97
Generic Sums, type parameters inferred: 46 and 62.97
Generic Sums with Constraint: 46 and 62.97
```

## Kết luận {#conclusion}

Làm tốt lắm! Bạn vừa làm quen với generics trong Go.

Các chủ đề đề xuất tiếp theo:

*   [Go Tour](/tour/) là phần giới thiệu từng bước tuyệt vời
    về các khái niệm cơ bản của Go.
*   Bạn sẽ tìm thấy các thực hành tốt về Go hữu ích được mô tả trong
    [Effective Go](/doc/effective_go) và
    [Cách viết code Go](/doc/code).

## Code hoàn chỉnh {#completed_code}

<!--TODO: Update text and link after release.-->
Bạn có thể chạy chương trình này trong
[Go playground](/play/p/apNmfVwogK0?v=gotip). Trên
playground, đơn giản là nhấn nút **Run**.

```
package main

import "fmt"

type Number interface {
	int64 | float64
}

func main() {
	// Initialize a map for the integer values
	ints := map[string]int64{
		"first": 34,
		"second": 12,
	}

	// Initialize a map for the float values
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

// SumInts adds together the values of m.
func SumInts(m map[string]int64) int64 {
	var s int64
	for _, v := range m {
		s += v
	}
	return s
}

// SumFloats adds together the values of m.
func SumFloats(m map[string]float64) float64 {
	var s float64
	for _, v := range m {
		s += v
	}
	return s
}

// SumIntsOrFloats sums the values of map m. It supports both floats and integers
// as map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}

// SumNumbers sums the values of map m. Its supports both integers
// and floats as map values.
func SumNumbers[K comparable, V Number](m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}
```
