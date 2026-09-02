---
title: Go Fuzzing
layout: article
breadcrumb: true
---

Go hỗ trợ fuzzing trong toolchain tiêu chuẩn bắt đầu từ Go 1.18. Các bài kiểm thử fuzz Go gốc
[được hỗ trợ bởi OSS-Fuzz](https://google.github.io/oss-fuzz/getting-started/new-project-guide/go-lang/#native-go-fuzzing-support).


**Hãy thử [hướng dẫn về fuzzing với Go](/doc/tutorial/fuzz).**

## Tổng quan

Fuzzing là một loại kiểm thử tự động liên tục thao túng các đầu vào cho
một chương trình để tìm lỗi. Go fuzzing sử dụng hướng dẫn độ phủ để thông minh duyệt qua
mã đang được fuzz để tìm và báo cáo lỗi cho người dùng. Vì nó
có thể tiếp cận các trường hợp biên mà con người thường bỏ qua, kiểm thử fuzz có thể đặc biệt
có giá trị cho việc tìm ra các khai thác bảo mật và lỗ hổng bảo mật.

Dưới đây là ví dụ về [fuzz test](#glos-fuzz-test), nổi bật các thành phần chính của nó.

<img class="DarkMode-img" alt="Example code showing the overall fuzz test, with a fuzz target within
it. Before the fuzz target is a corpus addition with f.Add, and the parameters
of the fuzz target are highlighted as the fuzzing arguments."
src="/security/fuzz/example-dark.png" style="width: 600px; height:
auto;"/>
<img alt="Example code showing the overall fuzz test, with a fuzz target within
it. Before the fuzz target is a corpus addition with f.Add, and the parameters
of the fuzz target are highlighted as the fuzzing arguments."
src="/security/fuzz/example.png" style="width: 600px; height:
auto;" class="LightMode-img"/>

## Viết fuzz test

### Yêu cầu

Dưới đây là các quy tắc mà fuzz test phải tuân theo.

- Một fuzz test phải là một hàm có tên như `FuzzXxx`, chỉ chấp nhận một
  `*testing.F`, và không có giá trị trả về.
- Fuzz test phải nằm trong các tệp \*\_test.go để chạy.
- Một [fuzz target](#glos-fuzz-target) phải là lời gọi phương thức đến
  <code>[(\*testing.F).Fuzz](https://pkg.go.dev/testing#F.Fuzz)</code> nhận
  một `*testing.T` làm tham số đầu tiên, theo sau là các đối số fuzzing.
  Không có giá trị trả về.
- Phải có đúng một fuzz target cho mỗi fuzz test.
- Tất cả các mục [seed corpus](#glos-seed-corpus) phải có các kiểu dữ liệu giống hệt
  với các [đối số fuzzing](#glos-fuzzing-arguments), theo cùng thứ tự.
  Điều này đúng với các lời gọi
  <code>[(\*testing.F).Add](https://pkg.go.dev/testing#F.Add)</code> và bất kỳ
  tệp corpus nào trong thư mục testdata/fuzz của fuzz test.
- Các đối số fuzzing chỉ có thể là các kiểu dữ liệu sau:
  - `string`, `[]byte`
  - `int`, `int8`, `int16`, `int32`/`rune`, `int64`
  - `uint`, `uint8`/`byte`, `uint16`, `uint32`, `uint64`
  - `float32`, `float64`
  - `bool`

### Gợi ý {#suggestions}

Dưới đây là các gợi ý giúp bạn tận dụng tối đa fuzzing.

- Fuzz target nên nhanh và xác định để fuzzing engine có thể hoạt động
  hiệu quả, và các lỗi mới cũng như độ phủ mã có thể được tái tạo dễ dàng.
- Vì fuzz target được gọi song song trên nhiều worker và theo
  thứ tự không xác định, trạng thái của một fuzz target không nên tồn tại sau
  khi kết thúc mỗi lần gọi, và hành vi của fuzz target không nên phụ thuộc vào
  trạng thái toàn cục.

## Chạy fuzz test

Có hai chế độ để chạy fuzz test của bạn: như một unit test (mặc định `go test`), hoặc
với fuzzing (`go test -fuzz=FuzzTestName`).

Theo mặc định, fuzz test được chạy giống như một unit test. Mỗi mục [seed corpus
entry](#glos-seed-corpus) sẽ được kiểm thử với fuzz target, báo cáo bất kỳ
lỗi nào trước khi thoát.

Để bật fuzzing, chạy `go test` với cờ `-fuzz`, cung cấp một regex
khớp với một fuzz test duy nhất. Theo mặc định, tất cả các bài kiểm thử khác trong gói đó sẽ
chạy trước khi fuzzing bắt đầu. Điều này để đảm bảo rằng fuzzing sẽ không báo cáo bất kỳ
vấn đề nào đã có thể được phát hiện bởi một bài kiểm thử hiện có.

Lưu ý rằng quyết định chạy fuzzing trong bao lâu là tùy bạn. Rất có khả năng
việc thực thi fuzzing có thể chạy vô thời hạn nếu nó không tìm thấy lỗi nào.
Sẽ có hỗ trợ để chạy các fuzz test này liên tục bằng các công cụ như OSS-Fuzz
trong tương lai, xem [Issue #50192](/issue/50192).

**Lưu ý:** Fuzzing nên được chạy trên một nền tảng hỗ trợ đo độ phủ
(hiện tại là AMD64 và ARM64) để corpus có thể phát triển có ý nghĩa khi nó chạy,
và nhiều mã hơn có thể được phủ trong quá trình fuzzing.

### Đầu ra dòng lệnh

Khi fuzzing đang tiến hành, [fuzzing engine](#glos-fuzzing-engine)
tạo ra các đầu vào mới và chạy chúng với fuzz target được cung cấp. Theo mặc định,
nó tiếp tục chạy cho đến khi tìm thấy một [failing input](#glos-failing-input), hoặc
người dùng hủy quá trình (ví dụ với Ctrl^C).

Đầu ra sẽ trông như thế này:

```
~ go test -fuzz FuzzFoo
fuzz: elapsed: 0s, gathering baseline coverage: 0/192 completed
fuzz: elapsed: 0s, gathering baseline coverage: 192/192 completed, now fuzzing with 8 workers
fuzz: elapsed: 3s, execs: 325017 (108336/sec), new interesting: 11 (total: 202)
fuzz: elapsed: 6s, execs: 680218 (118402/sec), new interesting: 12 (total: 203)
fuzz: elapsed: 9s, execs: 1039901 (119895/sec), new interesting: 19 (total: 210)
fuzz: elapsed: 12s, execs: 1386684 (115594/sec), new interesting: 21 (total: 212)
PASS
ok      foo 12.692s
```

Các dòng đầu tiên cho biết "baseline coverage" được thu thập trước
khi fuzzing bắt đầu.

Để thu thập baseline coverage, fuzzing engine thực thi cả [seed
corpus](#glos-seed-corpus) và [generated corpus](#glos-generated-corpus),
để đảm bảo không có lỗi nào xảy ra và để hiểu độ phủ mã mà corpus hiện có
đã cung cấp.

Các dòng tiếp theo cung cấp cái nhìn về việc thực thi fuzzing đang diễn ra:

  - elapsed: lượng thời gian đã trôi qua kể từ khi quá trình bắt đầu
  - execs: tổng số đầu vào đã được chạy với fuzz target
    (với trung bình execs/sec kể từ dòng log cuối cùng)
  - new interesting: tổng số đầu vào "thú vị" đã được
    thêm vào generated corpus trong lần thực thi fuzzing này (cùng với tổng
    kích thước của toàn bộ corpus)

Để một đầu vào được coi là "thú vị", nó phải mở rộng độ phủ mã vượt ra ngoài những gì
generated corpus hiện có có thể tiếp cận. Thông thường số lượng đầu vào thú vị mới
tăng nhanh lúc ban đầu và cuối cùng chậm lại,
với những đợt tăng đột ngột khi các nhánh mới được khám phá.

Bạn nên thấy số "new interesting" giảm dần theo thời gian khi các
đầu vào trong corpus bắt đầu bao phủ nhiều dòng mã hơn, với những đợt tăng đột ngột
nếu fuzzing engine tìm thấy một đường dẫn mã mới.

### Đầu vào thất bại

Một lỗi có thể xảy ra trong quá trình fuzzing vì nhiều lý do:

  - Một panic đã xảy ra trong mã hoặc bài kiểm thử.
  - Fuzz target đã gọi `t.Fail`, trực tiếp hoặc thông qua các phương thức như
  `t.Error` hoặc `t.Fatal`.
  - Một lỗi không thể khôi phục đã xảy ra, chẳng hạn như `os.Exit` hoặc tràn stack.
  - Fuzz target mất quá nhiều thời gian để hoàn thành. Hiện tại, timeout cho một
  lần thực thi của fuzz target là 1 giây. Điều này có thể thất bại do deadlock hoặc
  vòng lặp vô hạn, hoặc do hành vi có chủ đích trong mã. Đây là một lý do tại sao
  [gợi ý rằng fuzz target của bạn nên nhanh](#suggestions).

Nếu một lỗi xảy ra, fuzzing engine sẽ cố gắng thu nhỏ đầu vào thành
giá trị nhỏ nhất có thể và dễ đọc nhất đối với con người mà vẫn tạo ra
lỗi. Để cấu hình điều này, xem phần [cài đặt tùy chỉnh](#custom-settings).

Sau khi thu nhỏ hoàn tất, thông báo lỗi sẽ được ghi lại, và đầu ra
sẽ kết thúc với nội dung như thế này:

```
    Failing input written to testdata/fuzz/FuzzFoo/a878c3134fe0404d44eb1e662e5d8d4a24beb05c3d68354903670ff65513ff49
    To re-run:
    go test -run=FuzzFoo/a878c3134fe0404d44eb1e662e5d8d4a24beb05c3d68354903670ff65513ff49
FAIL
exit status 1
FAIL    foo 0.839s
```

Fuzzing engine đã ghi [failing input](#glos-failing-input) này vào seed
corpus của fuzz test đó, và nó sẽ được chạy theo mặc định với `go test`,
đóng vai trò là bài kiểm thử hồi quy sau khi lỗi đã được sửa.

Bước tiếp theo của bạn sẽ là chẩn đoán vấn đề, sửa lỗi, xác minh
bản vá bằng cách chạy lại `go test`, và gửi bản vá với tệp testdata mới
đóng vai trò là bài kiểm thử hồi quy của bạn.

### Cài đặt tùy chỉnh {#custom-settings}

Cài đặt mặc định của lệnh go nên hoạt động tốt cho hầu hết các trường hợp sử dụng fuzzing. Vì vậy
thông thường, một lần thực thi fuzzing trên dòng lệnh sẽ trông như thế này:

```
$ go test -fuzz={FuzzTestName}
```

Tuy nhiên, lệnh `go` cung cấp một số cài đặt khi chạy fuzzing.
Những cài đặt này được tài liệu hóa trong [tài liệu gói `cmd/go`](https://pkg.go.dev/cmd/go).

Để nổi bật một vài:

- `-fuzztime`: tổng thời gian hoặc số lần lặp mà fuzz target
  sẽ được thực thi trước khi thoát, mặc định là vô thời hạn.
- `-fuzzminimizetime`: thời gian hoặc số lần lặp mà fuzz target
  sẽ được thực thi trong mỗi lần cố gắng thu nhỏ, mặc định 60 giây. Bạn có thể
  hoàn toàn tắt chức năng thu nhỏ bằng cách đặt `-fuzzminimizetime 0` khi fuzzing.
- `-parallel`: số lượng quá trình fuzzing chạy cùng một lúc, mặc định
  `$GOMAXPROCS`. Hiện tại, việc đặt -cpu trong quá trình fuzzing không có tác dụng.

## Định dạng tệp corpus

Các tệp corpus được mã hóa theo một định dạng đặc biệt. Đây là định dạng giống nhau cho cả
[seed corpus](#glos-seed-corpus) và [generated
corpus](#glos-generated-corpus).

Dưới đây là ví dụ về tệp corpus:

```
go test fuzz v1
[]byte("hello\\xbd\\xb2=\\xbc ⌘")
int64(572293)
```

Dòng đầu tiên được sử dụng để thông báo cho fuzzing engine về phiên bản mã hóa của tệp.
Mặc dù không có phiên bản tương lai nào của định dạng mã hóa hiện được
lên kế hoạch, nhưng thiết kế phải hỗ trợ khả năng này.

Mỗi dòng tiếp theo là các giá trị tạo nên mục corpus, và
có thể được sao chép trực tiếp vào mã Go nếu muốn.

Trong ví dụ trên, chúng ta có `[]byte` theo sau là `int64`. Các kiểu dữ liệu này
phải khớp chính xác với các đối số fuzzing, theo thứ tự đó. Fuzz target cho các
kiểu dữ liệu này sẽ trông như thế này:

```
f.Fuzz(func(*testing.T, []byte, int64) {})
```

Cách dễ nhất để chỉ định các giá trị seed corpus của riêng bạn là sử dụng
phương thức `(*testing.F).Add`. Trong ví dụ trên, sẽ trông như thế này:

```
f.Add([]byte("hello\\xbd\\xb2=\\xbc ⌘"), int64(572293))
```

Tuy nhiên, bạn có thể có các tệp nhị phân lớn mà bạn không muốn sao chép vào mã
vào bài kiểm thử của mình, và thay vào đó vẫn là các mục seed corpus riêng lẻ trong
thư mục testdata/fuzz/{FuzzTestName}. Công cụ
[`file2fuzz`](https://pkg.go.dev/golang.org/x/tools/cmd/file2fuzz) tại
golang.org/x/tools/cmd/file2fuzz có thể được sử dụng để chuyển đổi các tệp nhị phân này thành
các tệp corpus được mã hóa cho `[]byte`.

Để sử dụng công cụ này:

```
$ go install golang.org/x/tools/cmd/file2fuzz@latest
$ file2fuzz -h
```

## Tài nguyên

- **Hướng dẫn**:
  - Hãy thử [hướng dẫn về fuzzing với Go](/doc/tutorial/fuzz) để có cái nhìn sâu
    về các khái niệm mới.
  - Để có hướng dẫn ngắn hơn, giới thiệu về fuzzing với Go, hãy xem [bài đăng trên blog](/blog/fuzz-beta).
- **Tài liệu**:
  - Tài liệu gói [`testing`](https://pkg.go.dev//testing#hdr-Fuzzing)
    mô tả kiểu `testing.F` được sử dụng khi viết fuzz test.
  - Tài liệu gói [`cmd/go`](https://pkg.go.dev/cmd/go) mô tả các cờ
    liên quan đến fuzzing.
- **Chi tiết kỹ thuật**:
  - [Bản thảo thiết kế](/s/draft-fuzzing-design)
  - [Đề xuất](/issue/44551)

## Bảng thuật ngữ {#glossary}

<a id="glos-corpus-entry"></a>
**corpus entry:** Một đầu vào trong corpus có thể được sử dụng trong quá trình fuzzing. Đây
có thể là một tệp được định dạng đặc biệt, hoặc một lời gọi đến
<code>[(\*testing.F).Add](https://pkg.go.dev/testing#F.Add)</code>.

<a id="glos-coverage-guidance"></a>
**coverage guidance:** Một phương pháp fuzzing sử dụng việc mở rộng trong độ phủ mã
để xác định các mục corpus nào đáng giữ lại để sử dụng trong tương lai.

<a id="glos-failing-input"></a>
**failing input:** Một failing input là một mục corpus sẽ gây ra lỗi
hoặc panic khi chạy với [fuzz target](#glos-fuzz-target).

<a id="glos-fuzz-target"></a>
**fuzz target:** Hàm của fuzz test được thực thi cho các mục corpus
và các giá trị được tạo trong quá trình fuzzing. Nó được cung cấp cho fuzz test bằng cách
truyền hàm vào
<code>[(\*testing.F).Fuzz](https://pkg.go.dev/testing#F.Fuzz)</code>.

<a id="glos-fuzz-test"></a>
**fuzz test:** Một hàm trong tệp kiểm thử có dạng `func FuzzXxx(*testing.F)`
có thể được sử dụng cho fuzzing.

<a id="glos-fuzzing"></a>
**fuzzing:** Một loại kiểm thử tự động liên tục thao túng các đầu vào
cho một chương trình để tìm ra các vấn đề như lỗi hoặc
[lỗ hổng bảo mật](#glos-vulnerability) mà mã có thể dễ bị tổn thương.

<a id="glos-fuzzing-arguments"></a>
**fuzzing arguments:** Các kiểu dữ liệu sẽ được truyền cho fuzz target, và
bị biến đổi bởi [mutator](#glos-mutator).

<a id="glos-fuzzing-engine"></a>
**fuzzing engine:** Một công cụ quản lý fuzzing, bao gồm duy trì corpus,
gọi mutator, xác định độ phủ mới, và báo cáo lỗi.

<a id="glos-generated-corpus"></a>
**generated corpus:** Một corpus được fuzzing engine duy trì theo
thời gian trong quá trình fuzzing để theo dõi tiến trình. Nó được lưu trữ trong `$GOCACHE`/fuzz.
Các mục này chỉ được sử dụng trong quá trình fuzzing.

<a id="glos-mutator"></a>
**mutator:** Một công cụ được sử dụng trong quá trình fuzzing ngẫu nhiên biến đổi các mục corpus
trước khi truyền chúng cho fuzz target.

<a id="glos-package"></a>
**package:** Một tập hợp các tệp nguồn trong cùng thư mục được
biên dịch cùng nhau. Xem [phần Packages](/ref/spec#Packages) trong Đặc tả ngôn ngữ Go.

<a id="glos-seed-corpus"></a>
**seed corpus:** Một corpus do người dùng cung cấp cho một fuzz test có thể được sử dụng để
hướng dẫn fuzzing engine. Nó bao gồm các mục corpus được cung cấp bởi các lời gọi f.Add
trong fuzz test, và các tệp trong thư mục testdata/fuzz/{FuzzTestName}
trong gói. Các mục này được chạy theo mặc định với `go test`,
dù đang fuzzing hay không.

<a id="glos-test-file"></a>
**test file:** Một tệp có định dạng xxx_test.go có thể chứa các bài kiểm thử,
benchmark, ví dụ và fuzz test.

<a id="glos-vulnerability"></a>
**vulnerability:** Một điểm yếu nhạy cảm về bảo mật trong mã có thể bị
khai thác bởi kẻ tấn công.

## Phản hồi

Nếu bạn gặp bất kỳ vấn đề nào hoặc có ý tưởng cho một tính năng, hãy [gửi
vấn đề](/issue/new?&labels=fuzz).

Để thảo luận và phản hồi chung về tính năng, bạn cũng có thể tham gia
vào [kênh #fuzzing](https://gophers.slack.com/archives/CH5KV1AKE) trong
Gophers Slack.
