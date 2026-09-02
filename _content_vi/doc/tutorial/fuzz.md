<!--{
  "Template": true,
  "Title": "Hướng dẫn: Bắt đầu với fuzzing",
  "HideTOC": true,
  "Breadcrumb": true,
  "template": true
}-->

Hướng dẫn này giới thiệu những kiến thức cơ bản về fuzzing trong Go. Với fuzzing, dữ liệu ngẫu nhiên
được chạy với test của bạn nhằm tìm ra các lỗ hổng bảo mật hoặc các đầu vào gây ra crash.
Một số ví dụ về lỗ hổng bảo mật có thể được phát hiện qua fuzzing là SQL
injection, tràn bộ đệm, tấn công từ chối dịch vụ và cross-site scripting.

Trong hướng dẫn này, bạn sẽ viết một fuzz test cho một hàm đơn giản, chạy lệnh go
và debug, sửa các vấn đề trong code.

Để tham khảo thuật ngữ trong suốt hướng dẫn này, xem [Bảng thuật ngữ Go Fuzzing
](/security/fuzz/#glossary).

Bạn sẽ thực hiện lần lượt các phần sau:

1. [Tạo thư mục cho code của bạn.](#create_folder)
2. [Thêm code để test.](#code_to_test)
3. [Thêm unit test.](#unit_test)
4. [Thêm fuzz test.](#fuzz_test)
5. [Sửa hai lỗi.](#fix_invalid_string_error)
6. [Khám phá thêm tài liệu tham khảo.](#conclusion)

**Lưu ý:** Để xem các hướng dẫn khác, truy cập [Hướng dẫn](/doc/tutorial/index.html).

**Lưu ý:** Fuzzing trong Go hiện hỗ trợ một tập con các kiểu dữ liệu có sẵn, được liệt kê trong
[tài liệu Go Fuzzing](/security/fuzz/#requirements), với sự hỗ trợ cho nhiều kiểu dữ liệu có sẵn hơn sẽ được thêm vào trong tương lai.

## Điều kiện tiên quyết

- **Đã cài đặt Go 1.18 hoặc mới hơn.** Để biết hướng dẫn cài đặt, xem
  [Cài đặt Go](/doc/install).
- **Một công cụ để chỉnh sửa code.** Bất kỳ trình soạn thảo văn bản nào bạn có đều dùng được.
- **Một cửa sổ dòng lệnh.** Go hoạt động tốt trên bất kỳ terminal nào trên Linux và Mac, cũng như
  trên PowerShell hoặc cmd trong Windows.
- **Môi trường hỗ trợ fuzzing.** Fuzzing trong Go với công cụ đo phạm vi phủ hiện chỉ
  có sẵn trên kiến trúc AMD64 và ARM64.

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

2. Từ dấu nhắc lệnh, tạo một thư mục có tên fuzz.

   ```
   $ mkdir fuzz
   $ cd fuzz
   ```

3. Tạo một module để chứa code của bạn.

   Chạy lệnh `go mod init`, cung cấp đường dẫn module cho code mới của bạn.

   ```
   $ go mod init example/fuzz
   go: creating new go.mod: module example/fuzz
   ```

   **Lưu ý:** Với code production, bạn sẽ chỉ định đường dẫn module cụ thể hơn
   theo nhu cầu của mình. Để biết thêm, hãy xem [Quản lý
   dependency](/doc/modules/managing-dependencies).

Tiếp theo, bạn sẽ thêm một ít code đơn giản để đảo ngược chuỗi, mà chúng ta sẽ fuzz sau.

## Thêm code để test {#code_to_test}

Trong bước này, bạn sẽ thêm một hàm để đảo ngược một chuỗi.

### Viết code

1.  Dùng trình soạn thảo văn bản của bạn, tạo một file có tên main.go trong thư mục fuzz.
2.  Vào main.go, ở đầu file, dán phần khai báo package sau.

    ```
    package main
    ```

    Một chương trình độc lập (trái với thư viện) luôn ở trong gói `main`.

3.  Bên dưới khai báo package, dán phần khai báo hàm sau.

    ```
    func Reverse(s string) string {
        b := []byte(s)
        for i, j := 0, len(b)-1; i {{raw "<"}} len(b)/2; i, j = i+1, j-1 {
            b[i], b[j] = b[j], b[i]
        }
        return string(b)
    }
    ```

    Hàm này sẽ nhận một `string`, duyệt qua nó từng `byte` một và
    trả về chuỗi đã đảo ngược ở cuối.

    _Lưu ý:_ Code này dựa trên hàm `stringutil.Reverse` trong
    golang.org/x/example.

4.  Ở đầu main.go, bên dưới khai báo package, dán hàm `main` sau để khởi tạo một chuỗi, đảo ngược nó, in kết quả và lặp lại.

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

    Hàm này sẽ thực hiện một vài thao tác `Reverse`, rồi in kết quả ra
    dòng lệnh. Điều này có thể giúp xem code đang hoạt động như thế nào và
    hỗ trợ việc debug.

5.  Hàm `main` dùng gói fmt, vì vậy bạn cần import nó.

    Các dòng đầu tiên của code nên trông như sau:

    ```
    package main

    import "fmt"
    ```

### Chạy code

Từ dòng lệnh trong thư mục chứa main.go, chạy code.

```
$ go run .
original: "The quick brown fox jumped over the lazy dog"
reversed: "god yzal eht revo depmuj xof nworb kciuq ehT"
reversed again: "The quick brown fox jumped over the lazy dog"
```

Bạn có thể thấy chuỗi gốc, kết quả của việc đảo ngược nó, rồi kết quả của
việc đảo ngược lần nữa, tương đương với chuỗi gốc.

Bây giờ code đang chạy, đã đến lúc test nó.

## Thêm unit test {#unit_test}

Trong bước này, bạn sẽ viết một unit test cơ bản cho hàm `Reverse`.

### Viết code

1. Dùng trình soạn thảo văn bản của bạn, tạo một file có tên reverse_test.go trong thư mục fuzz.
2. Dán đoạn code sau vào reverse_test.go.

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

   Unit test đơn giản này sẽ kiểm tra rằng các chuỗi đầu vào được liệt kê sẽ được
   đảo ngược đúng cách.

### Chạy code

Chạy unit test bằng `go test`

```
$ go test
PASS
ok      example/fuzz  0.013s
```

Tiếp theo, bạn sẽ chuyển đổi unit test thành fuzz test.

## Thêm fuzz test {#fuzz_test}

Unit test có những hạn chế, cụ thể là mỗi đầu vào phải được thêm vào test
bởi lập trình viên. Một lợi ích của fuzzing là nó tạo ra các đầu vào cho
code của bạn và có thể xác định các trường hợp biên mà các test case bạn nghĩ ra
không đạt tới.

Trong phần này, bạn sẽ chuyển đổi unit test thành fuzz test để có thể
tạo thêm nhiều đầu vào với ít công sức hơn!

Lưu ý rằng bạn có thể giữ unit test, benchmark và fuzz test trong cùng
file *_test.go, nhưng trong ví dụ này bạn sẽ chuyển đổi unit test thành fuzz test.

### Viết code

Trong trình soạn thảo văn bản của bạn, thay thế unit test trong reverse_test.go bằng
fuzz test sau.

```
func FuzzReverse(f *testing.F) {
    testcases := []string{"Hello, world", " ", "!12345"}
    for _, tc := range testcases {
        f.Add(tc)  // Use f.Add to provide a seed corpus
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

Fuzzing cũng có một số hạn chế. Trong unit test của bạn, bạn có thể dự đoán
kết quả mong đợi của hàm `Reverse` và xác minh rằng kết quả thực tế đáp ứng
những kỳ vọng đó.

Ví dụ, trong test case `Reverse("Hello, world")`, unit test chỉ định
giá trị trả về là `"dlrow ,olleH"`.

Khi fuzzing, bạn không thể dự đoán kết quả mong đợi vì bạn không
kiểm soát được các đầu vào.

Tuy nhiên, có một vài thuộc tính của hàm `Reverse` mà bạn có thể
kiểm tra trong fuzz test. Hai thuộc tính được kiểm tra trong fuzz test này là:

1.  Đảo ngược một chuỗi hai lần giữ nguyên giá trị ban đầu
2.  Chuỗi đã đảo ngược giữ nguyên trạng thái là UTF-8 hợp lệ.

Lưu ý sự khác biệt về cú pháp giữa unit test và fuzz test:

- Hàm bắt đầu bằng FuzzXxx thay vì TestXxx, và nhận `*testing.F`
  thay vì `*testing.T`
- Thay vì thấy một lần thực thi `t.Run`, bạn thấy `f.Fuzz`
  nhận một hàm fuzz target với các tham số là `*testing.T` và các
  kiểu cần fuzz. Các đầu vào từ unit test của bạn được cung cấp như các đầu vào seed corpus
  bằng `f.Add`.

Đảm bảo gói mới, `unicode/utf8` đã được import.

```
package main

import (
    "testing"
    "unicode/utf8"
)
```

Sau khi đã chuyển đổi unit test thành fuzz test, đã đến lúc chạy lại test.

### Chạy code

1. Chạy fuzz test mà không bật fuzzing để đảm bảo các đầu vào seed pass.

   ```
   $ go test
   PASS
   ok      example/fuzz  0.013s
   ```

   Bạn cũng có thể chạy `go test -run=FuzzReverse` nếu bạn có các test khác trong
   file đó và chỉ muốn chạy fuzz test.

2. Chạy `FuzzReverse` với fuzzing để xem liệu có chuỗi đầu vào được tạo ngẫu nhiên nào
   gây ra lỗi không. Điều này được thực thi bằng `go test` với cờ mới
   `-fuzz`, đặt thành tham số `Fuzz`. Sao chép lệnh dưới đây.

    ```
    $ go test -fuzz=Fuzz
    ```

    Một cờ hữu ích khác là `-fuzztime`, giới hạn thời gian fuzzing chạy.
    Ví dụ, chỉ định `-fuzztime 10s` trong test dưới đây có nghĩa là,
    miễn là không có lỗi xảy ra trước đó, test sẽ thoát theo mặc định
    sau khi 10 giây đã trôi qua. Xem [phần
    này](https://pkg.go.dev/cmd/go#hdr-Testing_flags) của tài liệu cmd/go
    để xem các cờ test khác.

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

   Đã xảy ra lỗi trong khi fuzzing, và đầu vào gây ra vấn đề được
   ghi vào file seed corpus, file này sẽ được chạy vào lần tiếp theo `go test` được
   gọi, ngay cả khi không có cờ `-fuzz`. Để xem đầu vào gây ra
   lỗi, hãy mở file corpus được ghi vào thư mục testdata/fuzz/FuzzReverse
   trong trình soạn thảo văn bản. File seed corpus của bạn có thể chứa một chuỗi khác,
   nhưng định dạng sẽ giống nhau.

   ```
   go test fuzz v1
   string("泃")
   ```

   Dòng đầu tiên của file corpus cho biết phiên bản mã hóa. Mỗi
   dòng tiếp theo đại diện cho giá trị của mỗi kiểu tạo nên mục corpus.
   Vì fuzz target chỉ nhận 1 đầu vào, chỉ có 1 giá trị sau
   phiên bản.

3. Chạy lại `go test` mà không có cờ `-fuzz`; mục seed corpus thất bại mới
   sẽ được sử dụng:

   ```
   $ go test
   --- FAIL: FuzzReverse (0.00s)
       --- FAIL: FuzzReverse/af69258a12129d6cbba438df5d5f25ba0ec050461c116f777e77ea7c9a0d217a (0.00s)
           reverse_test.go:20: Reverse produced invalid string
   FAIL
   exit status 1
   FAIL    example/fuzz  0.016s
   ```

   Vì test đã thất bại, đã đến lúc debug.

## Sửa lỗi chuỗi không hợp lệ {#fix_invalid_string_error}

Trong phần này, bạn sẽ debug lỗi và sửa bug.

Hãy dành chút thời gian suy nghĩ về vấn đề này và thử tự sửa trước khi tiếp tục.

### Chẩn đoán lỗi

Có một vài cách khác nhau để debug lỗi này. Nếu bạn đang dùng VS
Code làm trình soạn thảo văn bản, bạn có thể [thiết lập
debugger](https://github.com/golang/vscode-go/blob/master/docs/debugging.md) để
điều tra.

Trong hướng dẫn này, chúng ta sẽ ghi thông tin debug hữu ích ra terminal.

Trước tiên, hãy xem tài liệu của
[`utf8.ValidString`](https://pkg.go.dev/unicode/utf8).

```
ValidString reports whether s consists entirely of valid UTF-8-encoded runes.
```

Hàm `Reverse` hiện tại đảo ngược chuỗi theo từng byte, và đó chính là vấn đề của chúng ta.
Để bảo toàn các rune được mã hóa UTF-8 của chuỗi gốc,
chúng ta phải đảo ngược chuỗi theo từng rune.

Để kiểm tra tại sao đầu vào (trong trường hợp này là ký tự Trung Quốc `泃`) khiến
`Reverse` tạo ra một chuỗi không hợp lệ khi đảo ngược, bạn có thể kiểm tra số
lượng rune trong chuỗi đã đảo ngược.

#### Viết code

Trong trình soạn thảo văn bản của bạn, thay thế fuzz target trong `FuzzReverse` bằng
đoạn code sau.

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

Dòng `t.Logf` này sẽ in ra dòng lệnh nếu có lỗi xảy ra, hoặc nếu
chạy test với `-v`, điều này có thể giúp bạn debug vấn đề cụ thể này.

#### Chạy code

Chạy test bằng go test

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

Toàn bộ seed corpus sử dụng các chuỗi trong đó mỗi ký tự là một byte duy nhất.
Tuy nhiên, các ký tự như 泃 có thể yêu cầu nhiều byte. Do đó, việc đảo ngược
chuỗi theo từng byte sẽ làm hỏng các ký tự đa byte.

**Lưu ý:** Nếu bạn tò mò về cách Go xử lý chuỗi, hãy đọc bài viết blog
[Strings, bytes, runes and characters in Go](/blog/strings) để
hiểu sâu hơn.

Với hiểu biết tốt hơn về bug, hãy sửa lỗi trong hàm `Reverse`.

### Sửa lỗi

Để sửa hàm `Reverse`, hãy duyệt qua chuỗi theo rune thay vì theo byte.

#### Viết code

Trong trình soạn thảo văn bản của bạn, thay thế hàm Reverse() hiện có bằng đoạn sau.

```
func Reverse(s string) string {
    r := []rune(s)
    for i, j := 0, len(r)-1; i {{raw "<"}} len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    return string(r)
}
```

Điểm khác biệt chính là `Reverse` bây giờ duyệt qua từng `rune` trong
chuỗi, thay vì từng `byte`. Lưu ý đây chỉ là ví dụ và không
xử lý [ký tự kết hợp](https://en.wikipedia.org/wiki/Combining_character) đúng cách.

#### Chạy code

1. Chạy test bằng `go test`

   ```
   $ go test
   PASS
   ok      example/fuzz  0.016s
   ```

   Test đã pass!

2. Fuzz lại với `go test -fuzz`, để xem còn bug nào mới không.

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

   Chúng ta thấy rằng chuỗi khác với chuỗi gốc sau khi đảo ngược hai lần. Lần này chính đầu vào là unicode không hợp lệ. Điều này có thể xảy ra như thế nào nếu chúng ta đang fuzzing với chuỗi?

   Hãy debug lại.

## Sửa lỗi đảo ngược kép {#fix_double_reverse_error}

Trong phần này, bạn sẽ debug lỗi đảo ngược kép và sửa bug.

Hãy dành chút thời gian suy nghĩ về vấn đề này và thử tự sửa trước khi tiếp tục.

### Chẩn đoán lỗi

Giống như trước, có nhiều cách để debug lỗi này. Trong trường hợp này, dùng
[debugger](https://github.com/golang/vscode-go/blob/master/docs/debugging.md)
là một cách tiếp cận tốt.

Trong hướng dẫn này, chúng ta sẽ ghi thông tin debug hữu ích trong hàm `Reverse`.

Hãy xem kỹ chuỗi đã đảo ngược để phát hiện lỗi. Trong Go, [một chuỗi là
một slice byte chỉ đọc](/blog/strings), và có thể chứa các byte
không phải UTF-8 hợp lệ. Chuỗi gốc là một slice byte với một byte,
`'\x91'`. Khi chuỗi đầu vào được đặt thành `[]rune`, Go mã hóa slice byte thành
UTF-8 và thay thế byte bằng ký tự UTF-8 �. Khi chúng ta so sánh
ký tự UTF-8 thay thế với slice byte đầu vào, chúng rõ ràng không bằng nhau.

#### Viết code

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

   Điều này sẽ giúp chúng ta hiểu điều gì đang xảy ra sai khi chuyển đổi chuỗi
   thành một slice rune.

#### Chạy code

Lần này, chúng ta chỉ muốn chạy test thất bại để kiểm tra logs. Để
làm điều này, chúng ta sẽ dùng `go test -run`.

Để chạy một mục corpus cụ thể trong FuzzXxx/testdata, bạn có thể cung cấp
{FuzzTestName}/{filename} cho `-run`. Điều này có thể hữu ích khi debug.
Trong trường hợp này, đặt cờ `-run` bằng với hash chính xác của test thất bại.
Sao chép và dán hash duy nhất từ terminal của bạn;
nó sẽ khác với hash bên dưới.

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

Biết rằng đầu vào là unicode không hợp lệ, hãy sửa lỗi trong hàm `Reverse` của chúng ta.

### Sửa lỗi

Để sửa vấn đề này, hãy trả về lỗi nếu đầu vào của `Reverse` không phải là
UTF-8 hợp lệ.

#### Viết code

1. Trong trình soạn thảo văn bản của bạn, thay thế hàm `Reverse` hiện có bằng
   đoạn sau.

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

   Thay đổi này sẽ trả về lỗi nếu chuỗi đầu vào chứa các ký tự
   không phải UTF-8 hợp lệ.

1. Vì hàm Reverse bây giờ trả về lỗi, hãy sửa đổi hàm `main` để
   bỏ qua giá trị lỗi thêm vào. Thay thế hàm `main` hiện có bằng đoạn sau.

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

    Các lần gọi `Reverse` này nên trả về lỗi nil, vì chuỗi đầu vào
    là UTF-8 hợp lệ.

1. Bạn sẽ cần import các gói errors và unicode/utf8.
   Câu lệnh import trong main.go nên trông như sau.

   ```
   import (
       "errors"
       "fmt"
       "unicode/utf8"
   )
   ```

1. Sửa đổi file reverse_test.go để kiểm tra lỗi và bỏ qua test nếu
   có lỗi được tạo ra bằng cách return.

   ```
   func FuzzReverse(f *testing.F) {
       testcases := []string {"Hello, world", " ", "!12345"}
       for _, tc := range testcases {
           f.Add(tc)  // Use f.Add to provide a seed corpus
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

   Thay vì return, bạn cũng có thể gọi `t.Skip()` để dừng thực thi
   đầu vào fuzz đó.

#### Chạy code

1. Chạy test bằng go test

   ```
   $ go test
   PASS
   ok      example/fuzz  0.019s
   ```

2.  Fuzz với `go test -fuzz=Fuzz`, sau đó khi vài giây đã trôi qua, dừng
    fuzzing bằng `ctrl-C`. Fuzz test sẽ chạy cho đến khi gặp đầu vào thất bại
    trừ khi bạn truyền cờ `-fuzztime`. Mặc định là chạy mãi nếu không
    có lỗi xảy ra, và có thể ngắt bằng `ctrl-C`.

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

3. Fuzz với `go test -fuzz=Fuzz -fuzztime 30s`, sẽ fuzz trong 30
   giây trước khi thoát nếu không tìm thấy lỗi nào.

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

   Fuzzing đã pass!

   Ngoài cờ `-fuzz`, một số cờ mới đã được thêm vào `go
   test` và có thể xem trong [tài liệu](/security/fuzz/#custom-settings).

   Xem [Go Fuzzing](/security/fuzz/#command-line-output) để biết thêm
   thông tin về các thuật ngữ được sử dụng trong đầu ra fuzzing. Ví dụ, "new interesting"
   đề cập đến các đầu vào mở rộng phạm vi phủ code của fuzz test corpus hiện có.
   Số lượng đầu vào "new interesting" có thể tăng mạnh khi fuzzing bắt đầu, tăng đột biến
   nhiều lần khi các đường code mới được phát hiện, sau đó giảm dần theo thời gian.

## Kết luận {#conclusion}

Làm tốt lắm! Bạn vừa làm quen với fuzzing trong Go.

Bước tiếp theo là chọn một hàm trong code của bạn mà bạn muốn fuzz và
thử xem! Nếu fuzzing tìm thấy bug trong code của bạn, hãy cân nhắc thêm nó vào
[danh sách thành tích](/wiki/Fuzzing-trophy-case).

Nếu bạn gặp vấn đề hoặc có ý tưởng về tính năng, hãy [tạo
issue](/issue/new/?&labels=fuzz).

Để thảo luận và phản hồi chung về tính năng này, bạn cũng có thể tham gia
vào [kênh #fuzzing](https://gophers.slack.com/archives/CH5KV1AKE) trong
Gophers Slack.

Xem tài liệu tại [go.dev/security/fuzz](/security/fuzz/#requirements) để
đọc thêm.

## Code hoàn chỉnh

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
        f.Add(tc) // Use f.Add to provide a seed corpus
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

[Trở về đầu trang](#top)
