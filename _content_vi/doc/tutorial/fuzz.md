<!--{
  "Template": true,
  "Title": "Hướng dẫn: Bắt đầu với fuzzing",
  "HideTOC": true,
  "Breadcrumb": true
}-->

Hướng dẫn này giới thiệu những kiến thức cơ bản về fuzzing trong Go. Với fuzzing, dữ liệu ngẫu nhiên được chạy qua bài kiểm thử của bạn nhằm cố gắng tìm ra các lỗ hổng hoặc đầu vào gây lỗi crash. Một số ví dụ về lỗ hổng có thể được tìm thấy bằng fuzzing là SQL injection, tràn bộ đệm, từ chối dịch vụ và các cuộc tấn công cross-site scripting.

Trong hướng dẫn này, bạn sẽ viết một fuzz test cho một hàm đơn giản, chạy lệnh go, đồng thời gỡ lỗi và sửa các vấn đề trong mã.

Để được trợ giúp về thuật ngữ trong suốt hướng dẫn này, hãy xem [bảng thuật ngữ Go Fuzzing](/security/fuzz/#glossary).

Bạn sẽ lần lượt thực hiện các phần sau:

1. [Tạo thư mục cho mã của bạn.](#create_folder)
2. [Thêm mã để kiểm thử.](#code_to_test)
3. [Thêm một unit test.](#unit_test)
4. [Thêm một fuzz test.](#fuzz_test)
5. [Sửa hai lỗi.](#fix_invalid_string_error)
6. [Khám phá các tài nguyên bổ sung.](#conclusion)

**Lưu ý:** Đối với các hướng dẫn khác, hãy xem [Hướng dẫn](/doc/tutorial/index.html).

**Lưu ý:** Fuzzing của Go hiện hỗ trợ một tập hợp con các kiểu dựng sẵn, được liệt kê trong [tài liệu Go Fuzzing](/security/fuzz/#requirements), với hỗ trợ thêm nhiều kiểu dựng sẵn sẽ được bổ sung trong tương lai.

## Điều kiện tiên quyết

- **Go.** Chúng tôi khuyến nghị sử dụng phiên bản Go mới nhất để thực hiện theo hướng dẫn này. Để biết hướng dẫn cài đặt, hãy xem [Cài đặt Go](/doc/install).
- **Một công cụ để chỉnh sửa mã của bạn.** Bất kỳ trình soạn thảo văn bản nào bạn có đều hoạt động tốt.
- **Một terminal lệnh.** Go hoạt động tốt với bất kỳ terminal nào trên Linux và Mac, cũng như PowerShell hoặc cmd trên Windows.
- **Một môi trường hỗ trợ fuzzing.** Fuzzing của Go với công cụ đo độ phủ hiện chỉ khả dụng trên các kiến trúc AMD64 và ARM64.

## Tạo thư mục cho mã của bạn {#create_folder}

Để bắt đầu, hãy tạo một thư mục cho mã mà bạn sẽ viết.

1. Mở lời nhắc lệnh và chuyển đến thư mục chính của bạn.

   Trên Linux hoặc Mac:

   ```
   $ cd
   ```

   Trên Windows:

   ```
   C:\> cd %HOMEPATH%
   ```

   Phần còn lại của hướng dẫn sẽ hiển thị dấu $ làm lời nhắc. Các lệnh bạn sử dụng cũng sẽ hoạt động trên Windows.

2. Từ lời nhắc lệnh, tạo một thư mục cho mã của bạn có tên là fuzz.

   ```
   $ mkdir fuzz
   $ cd fuzz
   ```

3. Tạo một module để chứa mã của bạn.

   Chạy lệnh `go mod init`, cung cấp cho nó đường dẫn module của mã mới của bạn.

   ```
   $ go mod init example/fuzz
   go: creating new go.mod: module example/fuzz
   ```

   **Lưu ý:** Đối với mã dùng trong môi trường sản xuất, bạn nên chỉ định một đường dẫn module cụ thể hơn phù hợp với nhu cầu của riêng mình. Để biết thêm, hãy xem [Quản lý dependency](/doc/modules/managing-dependencies).

Tiếp theo, bạn sẽ thêm một số mã đơn giản để đảo ngược một chuỗi, sau đó chúng ta sẽ fuzz mã này.

## Thêm mã để kiểm thử {#code_to_test}

Trong bước này, bạn sẽ thêm một hàm để đảo ngược một chuỗi.

### Viết mã

1.  Sử dụng trình soạn thảo văn bản của bạn, tạo một tệp có tên main.go trong thư mục fuzz.
2.  Trong main.go, ở đầu tệp, dán khai báo package sau.

    ```
    package main
    ```

    Một chương trình độc lập (trái với một thư viện) luôn nằm trong package `main`.

3.  Bên dưới khai báo package, dán khai báo hàm sau.

    ```
    func Reverse(s string) string {
        b := []byte(s)
        for i, j := 0, len(b)-1; i {{raw "<"}} len(b)/2; i, j = i+1, j-1 {
            b[i], b[j] = b[j], b[i]
        }
        return string(b)
    }
    ```

    Hàm này sẽ nhận một `string`, lặp qua nó từng `byte` một, rồi trả về chuỗi đã được đảo ngược ở cuối.

    _Lưu ý:_ Mã này dựa trên hàm `stringutil.Reverse` trong
    golang.org/x/example.

4.  Ở đầu main.go, bên dưới khai báo package, dán hàm `main` sau để khởi tạo một chuỗi, đảo ngược chuỗi đó, in kết quả và lặp lại.

    ```
    func main() {
        input := "The quick brown fox jumped over the lazy dog"
        rev := Reverse(input)
        doubleRev := Reverse(rev)
        fmt.Printf("original: %q\n", input)
        fmt.Printf("reversed: %q\n", rev)
        fmt.Printf("reversed again: %q\n", doubleRev)
    }
    ```

    Hàm này sẽ chạy một vài thao tác `Reverse`, sau đó in kết quả ra dòng lệnh. Điều này có thể hữu ích để xem mã hoạt động như thế nào và có khả năng hỗ trợ việc gỡ lỗi.

5.  Hàm `main` sử dụng package fmt, vì vậy bạn sẽ cần nhập package đó.

    Những dòng mã đầu tiên sẽ có dạng như sau:

    ```
    package main

    import "fmt"
    ```

### Chạy mã

Từ dòng lệnh trong thư mục chứa main.go, chạy mã.

```
$ go run .
original: "The quick brown fox jumped over the lazy dog"
reversed: "god yzal eht revo depmuj xof nworb kciuq ehT"
reversed again: "The quick brown fox jumped over the lazy dog"
```

Bạn có thể thấy chuỗi ban đầu, kết quả của việc đảo ngược chuỗi đó, sau đó là kết quả của việc đảo ngược lại lần nữa, tương đương với chuỗi ban đầu.

Bây giờ mã đã chạy được, đã đến lúc kiểm thử nó.

## Thêm một bài kiểm thử đơn vị {#unit_test}

Trong bước này, bạn sẽ viết một bài kiểm thử đơn vị cơ bản cho hàm `Reverse`.

### Viết mã

1. Sử dụng trình soạn thảo văn bản của bạn, tạo một tệp có tên reverse_test.go trong thư mục fuzz.
2. Dán đoạn mã sau vào reverse_test.go.

   ```
   package main

   import (
       "testing"
   )

   func TestReverse(t *testing.T) {
       testcases := []struct {
           in, want string
       }{
           {"Hello, world", "dlrow ,olleH"},
           {" ", " "},
           {"!12345", "54321!"},
       }
       for _, tc := range testcases {
           rev := Reverse(tc.in)
           if rev != tc.want {
                   t.Errorf("Reverse: %q, want %q", rev, tc.want)
           }
       }
   }
   ```

Bài kiểm thử đơn giản này sẽ xác nhận rằng các chuỗi đầu vào được liệt kê sẽ được đảo ngược đúng cách.

### Chạy mã

Chạy bài kiểm thử đơn vị bằng `go test`

```
$ go test
PASS
ok      example/fuzz  0.013s
```

Tiếp theo, bạn sẽ thay đổi bài kiểm thử đơn vị thành một bài fuzz test.

## Thêm một fuzz test {#fuzz_test}

Bài kiểm thử đơn vị có những giới hạn, cụ thể là mỗi đầu vào phải được nhà phát triển thêm vào bài kiểm thử. Một lợi ích của fuzzing là nó tự tạo ra các đầu vào cho mã của bạn và có thể xác định các trường hợp biên mà những trường hợp kiểm thử bạn tạo ra chưa chạm tới.

Trong phần này, bạn sẽ chuyển đổi bài kiểm thử đơn vị thành một fuzz test để bạn có thể tạo thêm nhiều đầu vào với ít công sức hơn!

Lưu ý rằng bạn có thể giữ các bài kiểm thử đơn vị, benchmark và fuzz test trong cùng một tệp `*_test.go`, nhưng trong ví dụ này bạn sẽ chuyển đổi bài kiểm thử đơn vị thành một fuzz test.

### Viết mã

Trong trình soạn thảo văn bản của bạn, thay thế bài kiểm thử đơn vị trong reverse_test.go bằng fuzz test sau.

```
func FuzzReverse(f *testing.F) {
    testcases := []string{"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc)  // Sử dụng f.Add để cung cấp seed corpus
    }
    f.Fuzz(func(t *testing.T, orig string) {
        rev := Reverse(orig)
        doubleRev := Reverse(rev)
        if orig != doubleRev {
            t.Errorf("Before: %q, after: %q", orig, doubleRev)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
        }
    })
}
```

Fuzzing cũng có một số giới hạn. Trong bài kiểm thử đơn vị, bạn có thể dự đoán đầu ra mong đợi của hàm `Reverse` và xác minh rằng đầu ra thực tế đáp ứng những mong đợi đó.

Ví dụ, trong trường hợp kiểm thử `Reverse("Hello, world")`, bài kiểm thử đơn vị chỉ định giá trị trả về là `"dlrow ,olleH"`.

Khi fuzzing, bạn không thể dự đoán đầu ra mong đợi, vì bạn không kiểm soát các đầu vào.

Tuy nhiên, có một số thuộc tính của hàm `Reverse` mà bạn có thể xác minh trong fuzz test. Hai thuộc tính được kiểm tra trong fuzz test này là:

1. Đảo ngược một chuỗi hai lần sẽ giữ nguyên giá trị ban đầu
2. Chuỗi đã đảo ngược giữ nguyên trạng thái là UTF-8 hợp lệ.

Lưu ý sự khác biệt về cú pháp giữa bài kiểm thử đơn vị và fuzz test:

- Hàm bắt đầu bằng FuzzXxx thay vì TestXxx và nhận `*testing.F` thay vì `*testing.T`
- Ở nơi bạn mong đợi thấy một lần thực thi `t.Run`, bạn lại thấy `f.Fuzz`, hàm này nhận một hàm mục tiêu fuzz có các tham số là `*testing.T` và các kiểu dữ liệu cần fuzz. Các đầu vào từ bài kiểm thử đơn vị của bạn được cung cấp làm đầu vào seed corpus bằng `f.Add`.

Đảm bảo gói mới, `unicode/utf8` đã được import.

```
package main

import (
    "testing"
    "unicode/utf8"
)
```

Sau khi chuyển đổi bài kiểm thử đơn vị thành fuzz test, đã đến lúc chạy lại bài kiểm thử.

### Chạy mã

1. Chạy bài kiểm thử fuzz mà không fuzz để đảm bảo các đầu vào hạt giống vượt qua.

   ```
   $ go test
   PASS
   ok      example/fuzz  0.013s
   ```

   Bạn cũng có thể chạy `go test -run=FuzzReverse` nếu bạn có các bài kiểm thử khác trong tệp đó và chỉ muốn chạy bài kiểm thử fuzz.

2. Chạy `FuzzReverse` với fuzzing để xem liệu bất kỳ đầu vào chuỗi nào được tạo ngẫu nhiên có gây ra lỗi hay không. Thao tác này được thực thi bằng cách sử dụng `go test` với một cờ mới, `-fuzz`, được đặt thành tham số `Fuzz`. Sao chép lệnh bên dưới.

    ```
    $ go test -fuzz=Fuzz
    ```

    Một cờ hữu ích khác là `-fuzztime`, cờ này giới hạn thời gian fuzzing diễn ra. Ví dụ, chỉ định `-fuzztime 10s` trong bài kiểm thử bên dưới có nghĩa là, miễn là không có lỗi nào xảy ra trước đó, bài kiểm thử sẽ thoát theo mặc định sau khi đã trôi qua 10 giây. Xem [phần này](https://pkg.go.dev/cmd/go#hdr-Testing_flags) trong tài liệu cmd/go để xem các cờ kiểm thử khác.

   Bây giờ, chạy lệnh bạn vừa sao chép.

   ```
   $ go test -fuzz=Fuzz
   fuzz: elapsed: 0s, gathering baseline coverage: 0/3 completed
   fuzz: elapsed: 0s, gathering baseline coverage: 3/3 completed, now fuzzing with 8 workers
   fuzz: minimizing 38-byte failing input file...
   --- FAIL: FuzzReverse (0.01s)
       --- FAIL: FuzzReverse (0.00s)
           reverse_test.go:20: Reverse produced invalid UTF-8 string "\x9c\xdd"

       Failing input written to testdata/fuzz/FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a
       To re-run:
       go test -run=FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a
   FAIL
   exit status 1
   FAIL    example/fuzz  0.030s
   ```

   Một lỗi đã xảy ra trong quá trình fuzzing và đầu vào gây ra vấn đề được ghi vào một tệp seed corpus sẽ được chạy vào lần tiếp theo khi `go test` được gọi, ngay cả khi không có cờ `-fuzz`. Để xem đầu vào gây ra lỗi, hãy mở tệp corpus được ghi vào thư mục testdata/fuzz/FuzzReverse bằng trình soạn thảo văn bản. Tệp seed corpus của bạn có thể chứa một chuỗi khác, nhưng định dạng sẽ giống nhau.

   ```
   go test fuzz v1
   string("泃")
   ```

   Dòng đầu tiên của tệp corpus cho biết phiên bản mã hóa. Mỗi dòng tiếp theo biểu thị giá trị của từng kiểu tạo nên mục corpus. Vì mục tiêu fuzz chỉ nhận 1 đầu vào, nên chỉ có 1 giá trị sau phiên bản.

3. Chạy lại `go test` mà không có cờ `-fuzz`; mục seed corpus lỗi mới sẽ được sử dụng:

   ```
   $ go test
   --- FAIL: FuzzReverse (0.00s)
       --- FAIL: FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a (0.00s)
           reverse_test.go:20: Reverse produced invalid string
   FAIL
   exit status 1
   FAIL    example/fuzz  0.016s
   ```

   Vì bài kiểm thử của chúng ta đã thất bại, đã đến lúc gỡ lỗi.

## Sửa lỗi chuỗi không hợp lệ {#fix_invalid_string_error}

Trong phần này, bạn sẽ gỡ lỗi nguyên nhân thất bại và sửa lỗi.

Bạn có thể dành thời gian suy nghĩ về vấn đề này và thử tự sửa lỗi trước khi tiếp tục.

### Chẩn đoán lỗi

Có một vài cách khác nhau để bạn có thể gỡ lỗi này. Nếu bạn đang sử dụng VS
Code làm trình soạn thảo văn bản, bạn có thể [thiết lập trình
gỡ lỗi](https://github.com/golang/vscode-go/blob/master/docs/debugging.md) để
điều tra.

Trong hướng dẫn này, chúng ta sẽ ghi lại thông tin gỡ lỗi hữu ích vào terminal của bạn.

Trước tiên, hãy xem xét tài liệu cho
[`utf8.ValidString`](https://pkg.go.dev/unicode/utf8).

```
ValidString reports whether s consists entirely of valid UTF-8-encoded runes.
```

Hàm `Reverse` hiện tại đảo ngược chuỗi theo từng byte, và đó chính là vấn đề của chúng ta. Để bảo toàn các rune được mã hóa UTF-8 của chuỗi ban đầu, thay vào đó chúng ta phải đảo ngược chuỗi theo từng rune.

Để kiểm tra lý do đầu vào (trong trường hợp này là ký tự tiếng Trung `泃`) khiến `Reverse` tạo ra một chuỗi không hợp lệ khi đảo ngược, bạn có thể kiểm tra số lượng rune trong chuỗi đã đảo ngược.

#### Viết mã

Trong trình soạn thảo văn bản của bạn, thay thế fuzz target bên trong `FuzzReverse` bằng đoạn sau.

```
f.Fuzz(func(t *testing.T, orig string) {
    rev := Reverse(orig)
    doubleRev := Reverse(rev)
    t.Logf("Number of runes: orig=%d, rev=%d, doubleRev=%d", utf8.RuneCountInString(orig), utf8.RuneCountInString(rev), utf8.RuneCountInString(doubleRev))
    if orig != doubleRev {
        t.Errorf("Before: %q, after: %q", orig, doubleRev)
    }
    if utf8.ValidString(orig) && !utf8.ValidString(rev) {
        t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
    }
})
```

Dòng `t.Logf` này sẽ in ra dòng lệnh nếu xảy ra lỗi hoặc nếu thực thi bài kiểm tra với `-v`, điều này có thể giúp bạn gỡ lỗi vấn đề cụ thể này.

#### Chạy mã

Chạy bài kiểm tra bằng go test

```
$ go test
--- FAIL: FuzzReverse (0.00s)
    --- FAIL: FuzzReverse/28f36ef487f23e6c7a81ebdaa9feffe2f2b02b4cddaa6252e87f69863046a5e0 (0.00s)
        reverse_test.go:16: Number of runes: orig=1, rev=3, doubleRev=1
        reverse_test.go:21: Reverse produced invalid UTF-8 string "\x83\xb3\xe6"
FAIL
exit status 1
FAIL    example/fuzz    0.598s
```

Toàn bộ seed corpus được sử dụng đều chứa các chuỗi trong đó mỗi ký tự chỉ là một byte.
Tuy nhiên, các ký tự như 泃 có thể yêu cầu nhiều byte. Vì vậy, đảo ngược chuỗi theo từng byte sẽ làm mất hiệu lực các ký tự nhiều byte.

**Lưu ý:** Nếu bạn tò mò về cách Go xử lý chuỗi, hãy đọc bài viết trên blog
[Strings, bytes, runes and characters in Go](/blog/strings) để hiểu
sâu hơn.

Với hiểu biết tốt hơn về lỗi, hãy sửa lỗi trong hàm `Reverse`.

### Sửa lỗi

Để sửa hàm `Reverse`, hãy duyệt qua chuỗi theo các rune thay vì theo byte.

#### Viết mã

Trong trình soạn thảo văn bản của bạn, hãy thay thế hàm Reverse() hiện có bằng đoạn sau.

```
func Reverse(s string) string {
    r := []rune(s)
    for i, j := 0, len(r)-1; i {{raw "<"}} len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r)
}
```

Điểm khác biệt chính là giờ đây `Reverse` lặp qua từng `rune` trong chuỗi, thay vì từng `byte`. Lưu ý rằng đây chỉ là một ví dụ và không xử lý đúng [các ký tự kết hợp](https://en.wikipedia.org/wiki/Combining_character).

#### Chạy mã

1. Chạy bài kiểm tra bằng `go test`

   ```
   $ go test
   PASS
   ok      example/fuzz  0.016s
   ```

   Bây giờ bài kiểm tra đã vượt qua!

2. Chạy fuzz lại bằng `go test -fuzz` để xem có lỗi mới nào không.

   ```
   $ go test -fuzz=Fuzz
   fuzz: elapsed: 0s, gathering baseline coverage: 0/37 completed
   fuzz: minimizing 506-byte failing input file...
   fuzz: elapsed: 0s, gathering baseline coverage: 5/37 completed
   --- FAIL: FuzzReverse (0.02s)
       --- FAIL: FuzzReverse (0.00s)
           reverse_test.go:33: Before: "\x91", after: "�"

       Failing input written to testdata/fuzz/FuzzReverse/1ffc28f7538e29d79fce69fef20ce5ea72648529a9ca10bea392bcff28cd015c
       To re-run:
       go test -run=FuzzReverse/1ffc28f7538e29d79fce69fef20ce5ea72648529a9ca10bea392bcff28cd015c
   FAIL
   exit status 1
   FAIL    example/fuzz  0.032s
   ```

   Có thể thấy rằng chuỗi khác với chuỗi ban đầu sau khi được đảo ngược hai lần. Lần này, chính đầu vào là unicode không hợp lệ. Điều này có thể xảy ra thế nào nếu chúng ta đang fuzz với các chuỗi?

   Hãy gỡ lỗi lần nữa.

## Sửa lỗi đảo ngược hai lần {#fix_double_reverse_error}

Trong phần này, bạn sẽ gỡ lỗi lỗi đảo ngược hai lần và sửa lỗi.

Hãy dành thời gian suy nghĩ về vấn đề này và thử tự sửa lỗi trước khi tiếp tục.

### Chẩn đoán lỗi

Giống như trước, có một số cách bạn có thể gỡ lỗi lỗi này. Trong trường hợp này, sử dụng một
[trình gỡ lỗi](https://github.com/golang/vscode-go/blob/master/docs/debugging.md)
sẽ là một cách tiếp cận tuyệt vời.

Trong hướng dẫn này, chúng ta sẽ ghi lại thông tin gỡ lỗi hữu ích trong hàm `Reverse`.

Hãy xem kỹ chuỗi sau khi đảo ngược để phát hiện lỗi. Trong Go, [một chuỗi là một lát chỉ đọc của các byte](/blog/strings), và có thể chứa các byte không phải UTF-8 hợp lệ. Chuỗi ban đầu là một lát byte với một byte duy nhất, `'\x91'`. Khi chuỗi đầu vào được chuyển thành `[]rune`, Go mã hóa lát byte thành UTF-8 và thay thế byte bằng ký tự UTF-8 �. Khi chúng ta so sánh ký tự UTF-8 thay thế với lát byte đầu vào, rõ ràng chúng không bằng nhau.

#### Viết mã

1. Trong trình soạn thảo văn bản của bạn, thay thế hàm `Reverse` bằng đoạn sau.

   ```
   func Reverse(s string) string {
       fmt.Printf("input: %q\n", s)
       r := []rune(s)
       fmt.Printf("runes: %q\n", r)
       for i, j := 0, len(r)-1; i {{raw "<"}} len(r)/2; i, j = i+1, j-1 {
           r[i], r[j] = r[j], r[i]
       }
       return string(r)
   }
   ```

   Điều này sẽ giúp chúng ta hiểu vấn đề xảy ra khi chuyển đổi chuỗi thành một lát cắt các rune.

#### Chạy mã

Lần này, chúng ta chỉ muốn chạy bài kiểm thử thất bại để kiểm tra các nhật ký. Để thực hiện việc này, chúng ta sẽ sử dụng `go test -run`.

Để chạy một mục cụ thể trong corpus bên trong FuzzXxx/testdata, bạn có thể cung cấp {FuzzTestName}/{filename} cho `-run`. Điều này có thể hữu ích khi gỡ lỗi. Trong trường hợp này, hãy đặt cờ `-run` bằng chính xác mã băm của bài kiểm thử thất bại. Sao chép và dán mã băm duy nhất từ terminal của bạn; nó sẽ khác với mã bên dưới.

```
$ go test -run=FuzzReverse/28f36ef487f23e6c7a81ebdaa9feffe2f2b02b4cddaa6252e87f69863046a5e0
input: "\x91"
runes: ['�']
input: "�"
runes: ['�']
--- FAIL: FuzzReverse (0.00s)
    --- FAIL: FuzzReverse/28f36ef487f23e6c7a81ebdaa9feffe2f2b02b4cddaa6252e87f69863046a5e0 (0.00s)
        reverse_test.go:16: Number of runes: orig=1, rev=1, doubleRev=1
        reverse_test.go:18: Before: "\x91", after: "�"
FAIL
exit status 1
FAIL    example/fuzz    0.145s
```

Biết rằng dữ liệu đầu vào là unicode không hợp lệ, hãy sửa lỗi trong hàm `Reverse` của chúng ta.

### Sửa lỗi

Để khắc phục vấn đề này, hãy trả về một lỗi nếu dữ liệu đầu vào của `Reverse` không phải UTF-8 hợp lệ.

#### Viết mã

1. Trong trình soạn thảo văn bản của bạn, thay thế hàm `Reverse` hiện có bằng đoạn sau.

   ```
   func Reverse(s string) (string, error) {
       if !utf8.ValidString(s) {
           return s, errors.New("input is not valid UTF-8")
       }
       r := []rune(s)
       for i, j := 0, len(r)-1; i {{raw "<"}} len(r)/2; i, j = i+1, j-1 {
           r[i], r[j] = r[j], r[i]
       }
       return string(r), nil
   }
   ```

   Thay đổi này sẽ trả về một lỗi nếu chuỗi đầu vào chứa các ký tự không phải UTF-8 hợp lệ.

1. Vì hàm `Reverse` hiện trả về một lỗi, hãy sửa đổi hàm `main` để loại bỏ giá trị lỗi bổ sung. Thay thế hàm `main` hiện có bằng đoạn sau.

   ```
   func main() {
       input := "The quick brown fox jumped over the lazy dog"
       rev, revErr := Reverse(input)
       doubleRev, doubleRevErr := Reverse(rev)
       fmt.Printf("original: %q\n", input)
       fmt.Printf("reversed: %q, err: %v\n", rev, revErr)
       fmt.Printf("reversed again: %q, err: %v\n", doubleRev, doubleRevErr)
   }
   ```

   Những lệnh gọi `Reverse` này sẽ trả về lỗi nil, vì chuỗi đầu vào là UTF-8 hợp lệ.

1. Bạn sẽ cần nhập các gói `errors` và `unicode/utf8`. Câu lệnh import trong main.go sẽ có dạng như sau.

   ```
   import (
       "errors"
       "fmt"
       "unicode/utf8"
   )
   ```

1. Sửa đổi tệp reverse_test.go để kiểm tra lỗi và bỏ qua bài kiểm thử nếu lỗi được tạo ra bằng cách trả về.

   ```
   func FuzzReverse(f *testing.F) {
       testcases := []string {"Hello, world", " ", "!12345"}
       for _, tc := range testcases {
           f.Add(tc)  // Sử dụng f.Add để cung cấp một seed corpus
       }
       f.Fuzz(func(t *testing.T, orig string) {
           rev, err1 := Reverse(orig)
           if err1 != nil {
               return
           }
           doubleRev, err2 := Reverse(rev)
           if err2 != nil {
                return
           }
           if orig != doubleRev {
               t.Errorf("Before: %q, after: %q", orig, doubleRev)
           }
           if utf8.ValidString(orig) && !utf8.ValidString(rev) {
               t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
           }
       })
   }
   ```

   Thay vì trả về, bạn cũng có thể gọi `t.Skip()` để dừng việc thực thi đầu vào fuzz đó.

#### Chạy mã

1. Chạy bài kiểm thử bằng go test

   ```
   $ go test
   PASS
   ok      example/fuzz  0.019s
   ```

2.  Chạy fuzz bằng `go test -fuzz=Fuzz`, sau đó sau khi đã trôi qua vài giây, dừng
    fuzz bằng `ctrl-C`. Bài kiểm thử fuzz sẽ chạy cho đến khi gặp một đầu vào
    gây lỗi, trừ khi bạn truyền cờ `-fuzztime`. Mặc định là chạy vô hạn nếu không
    xảy ra lỗi, và quá trình có thể bị ngắt bằng `ctrl-C`.

   ```
   $ go test -fuzz=Fuzz
   fuzz: elapsed: 0s, gathering baseline coverage: 0/38 completed
   fuzz: elapsed: 0s, gathering baseline coverage: 38/38 completed, now fuzzing with 4 workers
   fuzz: elapsed: 3s, execs: 86342 (28778/sec), new interesting: 2 (total: 35)
   fuzz: elapsed: 6s, execs: 193490 (35714/sec), new interesting: 4 (total: 37)
   fuzz: elapsed: 9s, execs: 304390 (36961/sec), new interesting: 4 (total: 37)
   ...
   fuzz: elapsed: 3m45s, execs: 7246222 (32357/sec), new interesting: 8 (total: 41)
   ^Cfuzz: elapsed: 3m48s, execs: 7335316 (31648/sec), new interesting: 8 (total: 41)
   PASS
   ok      example/fuzz  228.000s
   ```

3. Chạy fuzz bằng `go test -fuzz=Fuzz -fuzztime 30s`, thao tác này sẽ fuzz trong
   30 giây trước khi thoát nếu không tìm thấy lỗi.

   ```
   $ go test -fuzz=Fuzz -fuzztime 30s
   fuzz: elapsed: 0s, gathering baseline coverage: 0/5 completed
   fuzz: elapsed: 0s, gathering baseline coverage: 5/5 completed, now fuzzing with 4 workers
   fuzz: elapsed: 3s, execs: 80290 (26763/sec), new interesting: 12 (total: 12)
   fuzz: elapsed: 6s, execs: 210803 (43501/sec), new interesting: 14 (total: 14)
   fuzz: elapsed: 9s, execs: 292882 (27360/sec), new interesting: 14 (total: 14)
   fuzz: elapsed: 12s, execs: 371872 (26329/sec), new interesting: 14 (total: 14)
   fuzz: elapsed: 15s, execs: 517169 (48433/sec), new interesting: 15 (total: 15)
   fuzz: elapsed: 18s, execs: 663276 (48699/sec), new interesting: 15 (total: 15)
   fuzz: elapsed: 21s, execs: 771698 (36143/sec), new interesting: 15 (total: 15)
   fuzz: elapsed: 24s, execs: 924768 (50990/sec), new interesting: 16 (total: 16)
   fuzz: elapsed: 27s, execs: 1082025 (52427/sec), new interesting: 17 (total: 17)
   fuzz: elapsed: 30s, execs: 1172817 (30281/sec), new interesting: 17 (total: 17)
   fuzz: elapsed: 31s, execs: 1172817 (0/sec), new interesting: 17 (total: 17)
   PASS
   ok      example/fuzz  31.025s
   ```

   Fuzz đã thành công!

   Ngoài cờ `-fuzz`, một số cờ mới đã được thêm vào `go
   test` và có thể xem trong [tài liệu](/security/fuzz/#custom-settings).

   Xem [Go Fuzzing](/security/fuzz/#command-line-output) để biết thêm
   thông tin về các thuật ngữ được sử dụng trong đầu ra fuzz. Ví dụ, "new interesting"
   đề cập đến các đầu vào mở rộng độ phủ mã của corpus bài kiểm thử fuzz hiện có.
   Số lượng đầu vào "new interesting" có thể được dự kiến sẽ tăng mạnh khi fuzz
   bắt đầu, tăng vọt vài lần khi các đường dẫn mã mới được phát hiện, sau đó
   giảm dần theo thời gian.

## Kết luận {#conclusion}

Làm tốt lắm! Bạn vừa làm quen với fuzzing trong Go.

Bước tiếp theo là chọn một hàm trong mã của bạn mà bạn muốn fuzz, và thử chạy nó! Nếu fuzzing phát hiện một lỗi trong mã của bạn, hãy cân nhắc thêm nó vào [tủ thành tích](/wiki/Fuzzing-trophy-case).

Nếu bạn gặp bất kỳ vấn đề nào hoặc có ý tưởng về một tính năng, [tạo một issue](/issue/new/?&labels=fuzz).

Để thảo luận và gửi phản hồi chung về tính năng này, bạn cũng có thể tham gia [kênh #fuzzing](https://gophers.slack.com/archives/CH5KV1AKE) trong Gophers Slack.

Xem tài liệu tại [go.dev/security/fuzz](/security/fuzz/#requirements) để đọc thêm.

## Mã đã hoàn thành

--- main.go ---

```
package main

import (
    "errors"
    "fmt"
    "unicode/utf8"
)

func main() {
    input := "The quick brown fox jumped over the lazy dog"
    rev, revErr := Reverse(input)
    doubleRev, doubleRevErr := Reverse(rev)
    fmt.Printf("original: %q\n", input)
    fmt.Printf("reversed: %q, err: %v\n", rev, revErr)
    fmt.Printf("reversed again: %q, err: %v\n", doubleRev, doubleRevErr)
}

func Reverse(s string) (string, error) {
    if !utf8.ValidString(s) {
        return s, errors.New("input is not valid UTF-8")
    }
    r := []rune(s)
    for i, j := 0, len(r)-1; i {{raw "<"}} len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r), nil
}
```

--- reverse_test.go ---

```
package main

import (
    "testing"
    "unicode/utf8"
)

func FuzzReverse(f *testing.F) {
    testcases := []string{"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc) // Sử dụng f.Add để cung cấp corpus hạt giống
    }
    f.Fuzz(func(t *testing.T, orig string) {
        rev, err1 := Reverse(orig)
        if err1 != nil {
            return
        }
        doubleRev, err2 := Reverse(rev)
        if err2 != nil {
            return
        }
        if orig != doubleRev {
            t.Errorf("Before: %q, after: %q", orig, doubleRev)
        }
        if utf8.ValidString(orig) && !utf8.ValidString(rev) {
            t.Errorf("Reverse produced invalid UTF-8 string %q", rev)
        }
    })
}
```

[Quay lại đầu trang](#top)
