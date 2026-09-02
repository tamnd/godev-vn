---
title: Ghi chú phát hành Go 1.24
---

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu Go 1.24 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.24,
ra mắt vào [tháng 2 năm 2025](/doc/devel/release#go1.24.0),
sáu tháng sau [Go 1.23](/doc/go1.23).
Phần lớn các thay đổi nằm ở phần triển khai toolchain, runtime và thư viện.
Như thường lệ, bản phát hành duy trì [cam kết tương thích](/doc/go1compat) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

## Thay đổi ngôn ngữ {#language}

<!-- go.dev/issue/46477 -->
Go 1.24 giờ hỗ trợ đầy đủ [generic type alias](/issue/46477): một type alias
có thể được tham số hóa giống như kiểu được định nghĩa.
Xem [đặc tả ngôn ngữ](/ref/spec#Alias_declarations) để biết chi tiết.
Hiện tại, tính năng này có thể bị vô hiệu hóa bằng cách đặt `GOEXPERIMENT=noaliastypeparams`;
nhưng cài đặt `aliastypeparams` sẽ bị xóa cho Go 1.25.

## Công cụ {#tools}

### Lệnh Go {#go-command}

<!-- go.dev/issue/48429 -->

Các module Go giờ có thể theo dõi các dependency thực thi bằng cách dùng directive `tool` trong
go.mod. Điều này loại bỏ nhu cầu về giải pháp thay thế trước đây là thêm các công cụ như
import trắng vào tệp thường được đặt tên là "tools.go". Lệnh `go tool`
giờ có thể chạy các công cụ này ngoài các công cụ đi kèm với bản phân phối Go.
Để biết thêm thông tin, xem [tài liệu](/doc/modules/managing-dependencies#tools).

Cờ mới `-tool` cho `go get` khiến directive tool được thêm vào
module hiện tại cho các gói được đặt tên ngoài việc thêm directive require.

[Meta-pattern `tool`](/cmd/go#hdr-Package_lists_and_patterns) mới tham chiếu đến
tất cả các công cụ trong module hiện tại. Điều này có thể được dùng để nâng cấp tất cả chúng với `go get tool` hoặc cài đặt chúng vào thư mục GOBIN với `go install tool`.

<!-- go.dev/issue/69290 -->

Các tệp thực thi được tạo bởi `go run` và hành vi mới của `go tool` giờ được
lưu trong Go build cache. Điều này làm cho việc thực thi lặp lại nhanh hơn với
chi phí làm cache lớn hơn. Xem [#69290](/issue/69290).

<!-- go.dev/issue/62067 -->

Các lệnh `go build` và `go install` giờ chấp nhận cờ `-json` báo cáo
đầu ra build và lỗi dưới dạng JSON có cấu trúc trên đầu ra chuẩn.
Để biết chi tiết về định dạng báo cáo, xem `go help buildjson`.

Ngoài ra, `go test -json` giờ báo cáo đầu ra build và lỗi trong JSON,
xen kẽ với JSON kết quả test.
Chúng được phân biệt bởi các kiểu `Action` mới, nhưng nếu chúng gây vấn đề trong
hệ thống tích hợp test, bạn có thể quay lại đầu ra build văn bản với
[cài đặt GODEBUG](/doc/godebug) `gotestjsonbuildtext=1`.

<!-- go.dev/issue/26232 -->

Biến môi trường mới `GOAUTH` cung cấp cách linh hoạt để xác thực
việc tải xuống module riêng tư. Xem `go help goauth` để biết thêm thông tin.

<!-- go.dev/issue/50603 -->

Lệnh `go build` giờ đặt [phiên bản của module chính](/pkg/runtime/debug#BuildInfo.Main) trong tệp nhị phân được biên dịch
dựa trên tag và/hoặc commit của hệ thống quản lý phiên bản.
Hậu tố `+dirty` sẽ được thêm nếu có các thay đổi chưa commit.
Dùng cờ `-buildvcs=false` để bỏ qua thông tin quản lý phiên bản khỏi tệp nhị phân.

<!-- go.dev/issue/63939 -->

[Cài đặt GODEBUG](/doc/godebug) mới [`toolchaintrace=1`](/doc/toolchain#select)
có thể được dùng để theo dõi quá trình chọn toolchain của lệnh `go`.

### Cgo {#cgo}

<!-- go.dev/issue/56378, CL 579955 -->
Cgo hỗ trợ các chú thích mới cho các hàm C để cải thiện hiệu suất runtime.
`#cgo noescape cFunctionName` cho trình biên dịch biết rằng bộ nhớ được truyền cho
hàm C `cFunctionname` không thoát.
`#cgo nocallback cFunctionName` cho trình biên dịch biết rằng hàm C
`cFunctionName` không gọi lại bất kỳ hàm Go nào.
Để biết thêm thông tin, xem [tài liệu cgo](/pkg/cmd/cgo#hdr-Optimizing_calls_of_C_code).

<!-- go.dev/issue/67699 -->
Cgo hiện từ chối biên dịch các lệnh gọi đến hàm C có nhiều
khai báo không tương thích. Ví dụ, nếu `f` được khai báo là cả `void f(int)`
và `void f(double)`, cgo sẽ báo lỗi thay vì có thể tạo ra
chuỗi lệnh gọi không chính xác cho `f(0)`. Mới trong bản phát hành này là bộ phát hiện tốt hơn cho
điều kiện lỗi này khi các khai báo không tương thích xuất hiện trong các tệp khác nhau.
Xem [#67699](/issue/67699).

### Objdump

<!-- go.dev/issue/15255, go.dev/issue/36738 -->
Công cụ [objdump](/cmd/objdump) giờ hỗ trợ disassembly trên LoongArch 64-bit
(`GOARCH=loong64`), RISC-V (`GOARCH=riscv64`), và S390X (`GOARCH=s390x`).

### Vet

<!-- go.dev/issue/44251 -->
Trình phân tích `tests` mới báo cáo các lỗi phổ biến trong khai báo
test, fuzzer, benchmark và ví dụ trong các gói test, như
tên không đúng định dạng, chữ ký không chính xác, hoặc ví dụ tài liệu
các định danh không tồn tại. Một số lỗi này có thể khiến test không
chạy.
Trình phân tích này nằm trong tập con các trình phân tích được chạy bởi `go test`.

<!-- go.dev/issue/60529 -->
Trình phân tích `printf` hiện tại giờ báo cáo chẩn đoán cho các lệnh gọi có
dạng `fmt.Printf(s)`, trong đó `s` là chuỗi định dạng không hằng,
không có đối số khác. Các lệnh gọi như vậy hầu như luôn là lỗi
vì giá trị của `s` có thể chứa ký tự `%`; hãy dùng `fmt.Print` thay thế.
Xem [#60529](/issue/60529). Kiểm tra này có xu hướng tạo ra kết quả trong mã hiện có,
và vì vậy chỉ được áp dụng khi phiên bản ngôn ngữ (được chỉ định bởi directive `go` trong go.mod
hoặc các comment `//go:build`) ít nhất là Go 1.24, để tránh
gây ra lỗi tích hợp liên tục khi cập nhật lên toolchain Go 1.24.

<!-- go.dev/issue/64127 -->
Trình phân tích `buildtag` hiện tại giờ báo cáo chẩn đoán khi
có [ràng buộc build phiên bản chính Go](/pkg/cmd/go#hdr-Build_constraints) không hợp lệ
trong directive `//go:build`. Ví dụ, `//go:build go1.23.1` tham chiếu đến
bản phát hành điểm; hãy dùng `//go:build go1.23` thay thế.
Xem [#64127](/issue/64127).

<!-- go.dev/issue/66387 -->
Trình phân tích `copylock` hiện tại giờ báo cáo chẩn đoán khi một
biến được khai báo trong vòng lặp `for` 3 mệnh đề như
`for i := iter(); done(i); i = next(i) { ... }` chứa `sync.Locker`,
như `sync.Mutex`. [Go 1.22](/doc/go1.22#language) đã thay đổi hành vi
của các vòng lặp này để tạo biến mới cho mỗi lần lặp, sao chép
giá trị từ lần lặp trước; thao tác sao chép này không an toàn với khóa.
Xem [#66387](/issue/66387).

### GOCACHEPROG

<!-- go.dev/issue/64876 -->
Cơ chế lưu cache tệp nhị phân và test nội bộ `cmd/go` giờ có thể được triển khai
bởi các tiến trình con thực hiện giao thức JSON giữa công cụ `cmd/go`
và tiến trình con được đặt tên bởi biến môi trường `GOCACHEPROG`.
Trước đây, đây là GOEXPERIMENT.
Để biết chi tiết giao thức, xem [tài liệu](/cmd/go/internal/cacheprog).

## Runtime {#runtime}

<!-- go.dev/issue/54766 -->
<!-- go.dev/cl/614795 -->
<!-- go.dev/issue/68578 -->

Một số cải tiến hiệu suất đối với runtime đã giảm chi phí CPU
trung bình 2-3% trên bộ benchmark đại diện.
Kết quả có thể khác nhau tùy theo ứng dụng.
Những cải tiến này bao gồm một triển khai `map` tích hợp mới dựa trên
[Swiss Tables](https://abseil.io/about/design/swisstables), cấp phát bộ nhớ hiệu quả hơn
cho các đối tượng nhỏ, và một triển khai mutex nội bộ runtime mới.

Triển khai `map` tích hợp mới và mutex nội bộ runtime mới có thể
bị vô hiệu hóa bằng cách đặt `GOEXPERIMENT=noswissmap` và `GOEXPERIMENT=nospinbitmutex`
tại thời điểm build tương ứng.

## Trình biên dịch {#compiler}

<!-- go.dev/issue/60725, go.dev/issue/57926 -->
Trình biên dịch đã không cho phép định nghĩa các phương thức mới với kiểu receiver được tạo bởi
cgo, nhưng có thể vượt qua hạn chế đó thông qua kiểu alias.
Go 1.24 giờ luôn báo lỗi nếu receiver ký hiệu kiểu được tạo bởi cgo,
dù trực tiếp hay gián tiếp (qua kiểu alias).

## Linker {#linker}

<!-- go.dev/issue/68678, go.dev/issue/68652, CL 618598, CL 618601 -->
Linker giờ tạo GNU build ID (note ELF `NT_GNU_BUILD_ID`) trên các nền tảng ELF
và UUID (lệnh tải Mach-O `LC_UUID`) trên macOS theo mặc định.
Build ID hoặc UUID được dẫn xuất từ Go build ID.
Nó có thể bị vô hiệu hóa bởi cờ linker `-B none`, hoặc được ghi đè bởi cờ linker `-B 0xNNNN`
với giá trị thập lục phân do người dùng chỉ định.

## Bootstrap {#bootstrap}

<!-- go.dev/issue/64751 -->
Như đã đề cập trong [ghi chú phát hành Go 1.22](/doc/go1.22#bootstrap), Go 1.24 giờ yêu cầu
Go 1.22.6 trở lên để bootstrap.
Chúng tôi kỳ vọng Go 1.26 sẽ yêu cầu bản phát hành điểm của Go 1.24 trở lên để bootstrap.

## Thư viện chuẩn {#library}

### Truy cập hệ thống tệp bị giới hạn trong thư mục

<!-- go.dev/issue/67002 -->
Kiểu mới [`os.Root`](/pkg/os#Root) cung cấp khả năng thực hiện các thao tác
hệ thống tệp trong một thư mục cụ thể.

Hàm [`os.OpenRoot`](/pkg/os#OpenRoot) mở một thư mục và trả về [`os.Root`](/pkg/os#Root).
Các phương thức trên [`os.Root`](/pkg/os#Root) hoạt động trong thư mục và không cho phép
các đường dẫn tham chiếu đến các vị trí ngoài thư mục, bao gồm
các đường dẫn theo symbolic link ra ngoài thư mục.
Các phương thức trên `os.Root` phản chiếu hầu hết các thao tác hệ thống tệp có sẵn trong gói
`os`, bao gồm ví dụ [`os.Root.Open`](/pkg/os#Root.Open),
[`os.Root.Create`](/pkg/os#Root.Create),
[`os.Root.Mkdir`](/pkg/os#Root.Mkdir),
và [`os.Root.Stat`](/pkg/os#Root.Stat),

### Hàm benchmark mới

Benchmark giờ có thể dùng phương thức [`testing.B.Loop`](/pkg/testing#B.Loop) nhanh hơn và ít dễ gây lỗi hơn để thực hiện các lần lặp benchmark như `for b.Loop() { ... }` thay cho các cấu trúc vòng lặp điển hình liên quan đến `b.N` như `for range b.N`. Điều này cung cấp hai lợi thế đáng kể:
 - Hàm benchmark sẽ thực thi đúng một lần mỗi -count, vì vậy các bước thiết lập và dọn dẹp tốn kém chỉ thực thi một lần.
 - Các tham số lệnh gọi hàm và kết quả được giữ còn sống, ngăn trình biên dịch tối ưu hóa hoàn toàn thân vòng lặp.

### Cơ chế finalizer được cải thiện

<!-- go.dev/issue/67535 -->
Hàm mới [`runtime.AddCleanup`](/pkg/runtime#AddCleanup) là một
cơ chế finalization linh hoạt hơn, hiệu quả hơn và ít dễ gây lỗi hơn
so với [`runtime.SetFinalizer`](/pkg/runtime#SetFinalizer).
`AddCleanup` gắn hàm cleanup vào đối tượng sẽ chạy một khi
đối tượng không còn được truy cập.
Tuy nhiên, không giống như `SetFinalizer`,
nhiều cleanup có thể được gắn vào một đối tượng duy nhất,
cleanup có thể được gắn vào các con trỏ nội bộ,
cleanup thường không gây ra rò rỉ khi các đối tượng tạo thành chu kỳ, và
cleanup không làm trễ việc giải phóng đối tượng hoặc các đối tượng nó trỏ đến.
Mã mới nên ưu tiên `AddCleanup` hơn `SetFinalizer`.

### Gói weak mới {#weak}

Gói mới [`weak`](/pkg/weak/) cung cấp các con trỏ yếu.

Con trỏ yếu là nguyên thủy cấp thấp được cung cấp để kích hoạt
việc tạo các cấu trúc hiệu quả bộ nhớ, như weak map để
liên kết các giá trị, canonicalization map cho bất cứ điều gì không
được bao phủ bởi gói [`unique`](/pkg/unique/), và nhiều loại
cache khác nhau.
Để hỗ trợ các trường hợp sử dụng này, bản phát hành này cũng cung cấp
[`runtime.AddCleanup`](/pkg/runtime/#AddCleanup) và
[`maphash.Comparable`](/pkg/maphash/#Comparable).

### Gói crypto/mlkem mới {#crypto-mlkem}

<!-- go.dev/issue/70122 -->

Gói mới [`crypto/mlkem`](/pkg/crypto/mlkem/) triển khai
ML-KEM-768 và ML-KEM-1024.

ML-KEM là cơ chế trao đổi khóa hậu lượng tử trước đây được gọi là Kyber và
được chỉ định trong [FIPS 203](https://doi.org/10.6028/NIST.FIPS.203).

### Các gói crypto/hkdf, crypto/pbkdf2, và crypto/sha3 mới {#crypto-packages}

<!-- go.dev/issue/61477, go.dev/issue/69488, go.dev/issue/69982, go.dev/issue/65269, CL 629176 -->

Gói mới [`crypto/hkdf`](/pkg/crypto/hkdf/) triển khai
hàm dẫn xuất khóa Extract-and-Expand dựa trên HMAC HKDF,
như được định nghĩa trong [RFC 5869](https://www.rfc-editor.org/rfc/rfc5869.html).

Gói mới [`crypto/pbkdf2`](/pkg/crypto/pbkdf2/) triển khai
hàm dẫn xuất khóa dựa trên mật khẩu PBKDF2,
như được định nghĩa trong [RFC 8018](https://www.rfc-editor.org/rfc/rfc8018.html).

Gói mới [`crypto/sha3`](/pkg/crypto/sha3/) triển khai
hàm hash SHA-3 và các hàm đầu ra mở rộng SHAKE và cSHAKE,
như được định nghĩa trong [FIPS 202](http://doi.org/10.6028/NIST.FIPS.202).

Cả ba gói đều dựa trên các gói `golang.org/x/crypto/...` đã có trước.

### Tuân thủ FIPS 140-3 {#fips140}

Bản phát hành này bao gồm [một tập hợp các cơ chế mới để tạo điều kiện tuân thủ FIPS 140-3](/doc/security/fips140).

Go Cryptographic Module là một tập hợp các gói thư viện chuẩn nội bộ được
sử dụng trong suốt để triển khai các thuật toán được phê duyệt theo FIPS 140-3. Các ứng dụng
không cần thay đổi để sử dụng Go Cryptographic Module cho các thuật toán được phê duyệt.

Biến môi trường mới `GOFIPS140` có thể được dùng để chọn phiên bản Go
Cryptographic Module để dùng trong bản build. [Cài đặt GODEBUG](/doc/godebug) mới `fips140`
có thể được dùng để bật chế độ FIPS 140-3 tại runtime.

Go 1.24 bao gồm Go Cryptographic Module phiên bản v1.0.0, hiện đang
được kiểm thử với phòng thí nghiệm được CMVP công nhận.

### Gói testing/synctest thử nghiệm mới {#testing-synctest}

Gói thử nghiệm mới [`testing/synctest`](/pkg/testing/synctest/) cung cấp hỗ trợ
để test mã đồng thời.
- Hàm [`synctest.Run`](/pkg/testing/synctest/#Run) khởi động một
  nhóm goroutine trong một "bong bóng" biệt lập.
  Trong bong bóng, các hàm của gói [`time`](/pkg/time) hoạt động trên
  đồng hồ giả.
- Hàm [`synctest.Wait`](/pkg/testing/synctest#Wait) chờ cho đến khi
  tất cả goroutine trong bong bóng hiện tại bị chặn.

Xem tài liệu gói để biết thêm chi tiết.

Gói `synctest` là thử nghiệm và phải được bật bằng cách
đặt `GOEXPERIMENT=synctest` tại thời điểm build.
API của gói có thể thay đổi trong các bản phát hành tương lai.
Xem [issue #67434](/issue/67434) để biết thêm thông tin và
cung cấp phản hồi.

### Thay đổi nhỏ trong thư viện {#minor_library_changes}

#### [`archive`](/pkg/archive/)

Các triển khai `(*Writer).AddFS` trong cả `archive/zip` và `archive/tar`
giờ ghi header thư mục cho thư mục rỗng.

#### [`bytes`](/pkg/bytes/)

Gói [`bytes`](/pkg/bytes) thêm một số hàm hoạt động với iterator:
- [`Lines`](/pkg/bytes#Lines) trả về iterator qua các
  dòng kết thúc bằng ký tự newline trong byte slice.
- [`SplitSeq`](/pkg/bytes#SplitSeq) trả về iterator qua
  tất cả các sub-slice của byte slice được chia xung quanh dấu phân cách.
- [`SplitAfterSeq`](/pkg/bytes#SplitAfterSeq) trả về iterator
  qua các sub-slice của byte slice được chia sau mỗi lần xuất hiện dấu phân cách.
- [`FieldsSeq`](/pkg/bytes#FieldsSeq) trả về iterator qua
  các sub-slice của byte slice được chia xung quanh các chuỗi ký tự khoảng trắng,
  như được định nghĩa bởi [`unicode.IsSpace`](/pkg/unicode#IsSpace).
- [`FieldsFuncSeq`](/pkg/bytes#FieldsFuncSeq) trả về iterator
  qua các sub-slice của byte slice được chia xung quanh các chuỗi code point Unicode
  thỏa mãn một vị từ.

#### [`crypto/aes`](/pkg/crypto/aes/)

Giá trị được trả về bởi [`NewCipher`](/pkg/crypto/aes#NewCipher) không còn
triển khai các phương thức `NewCTR`, `NewGCM`, `NewCBCEncrypter`, và `NewCBCDecrypter`.
Các phương thức này không có tài liệu và không có sẵn trên tất cả kiến trúc.
Thay vào đó, giá trị [`Block`](/pkg/crypto/cipher#Block) nên được truyền
trực tiếp cho các hàm [`crypto/cipher`](/pkg/crypto/cipher/) liên quan.
Hiện tại, `crypto/cipher` vẫn kiểm tra các phương thức đó trên các giá trị `Block`,
ngay cả khi chúng không còn được thư viện chuẩn sử dụng.

#### [`crypto/cipher`](/pkg/crypto/cipher/)

Hàm mới [`NewGCMWithRandomNonce`](/pkg/crypto/cipher#NewGCMWithRandomNonce)
trả về [`AEAD`](/pkg/crypto/cipher#AEAD) triển khai AES-GCM bằng cách
tạo nonce ngẫu nhiên trong Seal và thêm nó vào đầu văn bản mã hóa.

Triển khai [`Stream`](/pkg/crypto/cipher#Stream) được trả về bởi
[`NewCTR`](/pkg/crypto/cipher#NewCTR) khi dùng với
[`crypto/aes`](/pkg/crypto/aes/) giờ nhanh hơn vài lần trên amd64 và arm64.

[`NewOFB`](/pkg/crypto/cipher#NewOFB),
[`NewCFBEncrypter`](/pkg/crypto/cipher#NewCFBEncrypter), và
[`NewCFBDecrypter`](/pkg/crypto/cipher#NewCFBDecrypter) giờ bị deprecated.
Chế độ OFB và CFB không được xác thực, điều này thường cho phép các cuộc tấn công chủ động
để thao túng và khôi phục văn bản gốc. Khuyến nghị rằng các ứng dụng nên sử dụng
chế độ [`AEAD`](/pkg/crypto/cipher#AEAD) thay thế. Nếu cần chế độ [`Stream`](/pkg/crypto/cipher#Stream)
không được xác thực, hãy dùng [`NewCTR`](/pkg/crypto/cipher#NewCTR) thay thế.

#### [`crypto/ecdsa`](/pkg/crypto/ecdsa/)

<!-- go.dev/issue/64802 -->
[`PrivateKey.Sign`](/pkg/crypto/ecdsa#PrivateKey.Sign) giờ tạo ra
chữ ký xác định theo
[RFC 6979](https://www.rfc-editor.org/rfc/rfc6979.html) nếu nguồn ngẫu nhiên là nil.

#### [`crypto/md5`](/pkg/crypto/md5/)

Giá trị được trả về bởi [`md5.New`](/pkg/md5#New) giờ cũng triển khai
interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`crypto/rand`](/pkg/crypto/rand/)

<!-- go.dev/issue/66821 -->
Hàm [`Read`](/pkg/crypto/rand#Read) giờ được đảm bảo không thất bại.
Nó sẽ luôn trả về `nil` làm kết quả `error`.
Nếu `Read` gặp lỗi khi đọc từ
[`Reader`](/pkg/crypto/rand#Reader), chương trình sẽ crash không thể khôi phục.
Lưu ý rằng các API nền tảng được dùng bởi `Reader` mặc định được tài liệu hóa để
luôn thành công, vì vậy thay đổi này chỉ ảnh hưởng đến các chương trình ghi đè biến
`Reader`. Một ngoại lệ là các kernel Linux trước phiên bản 3.17, nơi
`Reader` mặc định vẫn mở `/dev/urandom` và có thể thất bại.

<!-- go.dev/issue/69577 -->
Trên Linux 6.11 và sau đó, `Reader` giờ dùng lệnh gọi hệ thống `getrandom` thông qua vDSO.
Điều này nhanh hơn vài lần, đặc biệt cho các lần đọc nhỏ.

<!-- CL 608395 -->
Trên OpenBSD, `Reader` giờ dùng `arc4random_buf(3)`.

<!-- go.dev/issue/67057 -->
Hàm mới [`Text`](/pkg/crypto/rand#Text) có thể được dùng để tạo
các chuỗi văn bản ngẫu nhiên an toàn về mật mã.

#### [`crypto/rsa`](/pkg/crypto/rsa/)

[`GenerateKey`](/pkg/crypto/rsa#GenerateKey) giờ trả về lỗi nếu khóa nhỏ hơn
1024 bit được yêu cầu.
Tất cả các phương thức Sign, Verify, Encrypt và Decrypt giờ trả về lỗi nếu được dùng với
khóa nhỏ hơn 1024 bit. Các khóa như vậy không an toàn và không nên được sử dụng.
[Cài đặt GODEBUG](/doc/godebug) `rsa1024min=0` khôi phục hành vi cũ, nhưng chúng tôi
khuyến nghị chỉ làm điều này khi cần thiết và chỉ trong các test, ví dụ bằng cách thêm
dòng `//go:debug rsa1024min=0` vào tệp test.
[Ví dụ](/pkg/crypto/rsa#example-GenerateKey-TestKey) mới về `GenerateKey`
cung cấp khóa test 2048-bit chuẩn dễ sử dụng.

Giờ an toàn và hiệu quả hơn để gọi
[`PrivateKey.Precompute`](/pkg/crypto/rsa#PrivateKey.Precompute) trước
[`PrivateKey.Validate`](/pkg/crypto/rsa#PrivateKey.Validate).
`Precompute` giờ nhanh hơn khi có [`PrecomputedValues`](/pkg/crypto/rsa#PrecomputedValues)
được điền một phần, như khi unmarshal khóa từ JSON.

Gói giờ từ chối nhiều khóa không hợp lệ hơn, ngay cả khi `Validate` không được gọi,
và [`GenerateKey`](/pkg/crypto/rsa#GenerateKey) có thể trả về lỗi mới cho
các nguồn ngẫu nhiên bị hỏng.
Các trường [`Primes`](/pkg/crypto/rsa#PrivateKey.Primes) và
[`Precomputed`](/pkg/crypto/rsa#PrivateKey.Precomputed) của
[`PrivateKey`](/pkg/crypto/rsa#PrivateKey) giờ được sử dụng và xác thực ngay cả khi
một số giá trị bị thiếu.
Xem thêm các thay đổi đối với phân tích và marshaling khóa RSA của `crypto/x509`
[được mô tả dưới đây](#cryptox509pkgcryptox509).

<!-- go.dev/issue/43923 -->
[`SignPKCS1v15`](/pkg/crypto/rsa#SignPKCS1v15) và
[`VerifyPKCS1v15`](/pkg/crypto/rsa#VerifyPKCS1v15) giờ hỗ trợ
SHA-512/224, SHA-512/256, và SHA-3.

<!-- CL 639936 -->
[`GenerateKey`](/pkg/crypto/rsa#GenerateKey) giờ sử dụng phương pháp hơi khác
để tạo số mũ riêng (phần tử Carmichael thay vì Euler).
Các ứng dụng hiếm tái tạo khóa bên ngoài chỉ từ các thừa số nguyên tố
có thể tạo ra kết quả khác nhau nhưng tương thích.

<!-- CL 626957 -->
Các thao tác khóa công khai và riêng tư giờ nhanh hơn đến hai lần trên wasm.

#### [`crypto/sha1`](/pkg/crypto/sha1/)

Giá trị được trả về bởi [`sha1.New`](/pkg/sha1#New) giờ cũng triển khai
interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`crypto/sha256`](/pkg/crypto/sha256/)

Các giá trị được trả về bởi [`sha256.New`](/pkg/sha256#New) và
[`sha256.New224`](/pkg/sha256#New224) giờ cũng triển khai
interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`crypto/sha512`](/pkg/crypto/sha512/)

Các giá trị được trả về bởi [`sha512.New`](/pkg/sha512#New),
[`sha512.New384`](/pkg/sha512#New384),
[`sha512.New512_224`](/pkg/sha512#New512_224) và
[`sha512.New512_256`](/pkg/sha512#New512_256) giờ cũng triển khai
interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`crypto/subtle`](/pkg/crypto/subtle/)

Hàm mới [`WithDataIndependentTiming`](/pkg/crypto/subtle#WithDataIndependentTiming)
cho phép người dùng chạy một hàm với các tính năng kiến trúc cụ thể được
kích hoạt đảm bảo các lệnh cụ thể là bất biến về thời gian giá trị dữ liệu.
Điều này có thể được dùng để đảm bảo rằng mã được thiết kế để chạy trong thời gian hằng số không bị
tối ưu hóa bởi các tính năng cấp CPU đến mức hoạt động trong thời gian biến đổi.
Hiện tại, `WithDataIndependentTiming` sử dụng bit PSTATE.DIT trên arm64, và là
no-op trên tất cả các kiến trúc khác. [Cài đặt GODEBUG](/doc/godebug)
`dataindependenttiming=1` kích hoạt chế độ DIT cho toàn bộ chương trình Go.

<!-- CL 622276 -->
Đầu ra của [`XORBytes`](/pkg/crypto/subtle#XORBytes) phải chồng chéo chính xác hoặc
hoàn toàn không với các đầu vào. Trước đây, hành vi ngược lại là không xác định, trong khi
giờ `XORBytes` sẽ panic.

#### [`crypto/tls`](/pkg/crypto/tls/)

TLS server giờ hỗ trợ Encrypted Client Hello (ECH). Tính năng này có thể được
bật bằng cách điền trường [`Config.EncryptedClientHelloKeys`](/pkg/crypto/tls#Config.EncryptedClientHelloKeys).

Cơ chế trao đổi khóa hậu lượng tử mới [`X25519MLKEM768`](/pkg/crypto/tls#X25519MLKEM768)
giờ được hỗ trợ và được bật theo mặc định khi
[`Config.CurvePreferences`](/pkg/crypto/tls#Config.CurvePreferences) là nil.
[Cài đặt GODEBUG](/doc/godebug) `tlsmlkem=0` hoàn nguyên mặc định.
Điều này có thể hữu ích khi xử lý các TLS server có lỗi không xử lý các bản ghi lớn đúng cách,
gây ra timeout trong quá trình bắt tay (xem [TLS post-quantum TL;DR fail](https://tldr.fail/)).

Hỗ trợ cho trao đổi khóa thử nghiệm `X25519Kyber768Draft00` đã bị xóa.

<!-- go.dev/issue/69393, CL 630775 -->
Thứ tự trao đổi khóa giờ được xử lý hoàn toàn bởi gói `crypto/tls`. Thứ tự
của [`Config.CurvePreferences`](/pkg/crypto/tls#Config.CurvePreferences) giờ bị bỏ qua,
và nội dung chỉ được dùng để xác định các trao đổi khóa nào cần bật khi trường được điền.

<!-- go.dev/issue/32936 -->
Trường mới [`ClientHelloInfo.Extensions`](/pkg/crypto/tls#ClientHelloInfo.Extensions)
liệt kê các ID của extension nhận trong thông điệp Client Hello.
Điều này có thể hữu ích để lấy dấu vân tay TLS client.

<!-- go.dev/issue/72111, documented retro-actively after Go 1.27 release -->

Thiết lập `GODEBUG` tên `tlskyber` (được thêm trong [Go 1.23](/doc/godebug#go-123))
đã bị loại bỏ.

#### [`crypto/x509`](/pkg/crypto/x509/)

<!-- go.dev/issue/41682 -->
[Cài đặt GODEBUG](/doc/godebug) `x509sha1` đã bị xóa.
[`Certificate.Verify`](/pkg/crypto/x509#Certificate.Verify) không còn
hỗ trợ các chữ ký dựa trên SHA-1.

[`OID`](/pkg/crypto/x509#OID) giờ triển khai các interface
[`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender) và
[`encoding.TextAppender`](/pkg/encoding#TextAppender).

Trường chính sách chứng chỉ mặc định đã thay đổi từ
[`Certificate.PolicyIdentifiers`](/pkg/crypto/x509#Certificate.PolicyIdentifiers)
thành [`Certificate.Policies`](/pkg/crypto/x509#Certificate.Policies). Khi phân tích
chứng chỉ, cả hai trường sẽ được điền, nhưng khi tạo chứng chỉ
các chính sách giờ được lấy từ trường `Certificate.Policies` thay vì
trường `Certificate.PolicyIdentifiers`. Thay đổi này có thể được hoàn nguyên với
[cài đặt GODEBUG](/doc/godebug) `x509usepolicies=0`.

<!-- go.dev/issue/67675 -->
[`CreateCertificate`](/pkg/crypto/x509#CreateCertificate) giờ sẽ tạo một
số serial bằng phương pháp tuân thủ RFC 5280 khi được truyền template với
trường [`Certificate.SerialNumber`](/pkg/crypto/x509#Certificate.SerialNumber) nil,
thay vì thất bại.

[`Certificate.Verify`](/pkg/crypto/x509#Certificate.Verify) giờ hỗ trợ xác thực
chính sách, như được định nghĩa trong RFC 5280 và RFC 9618. Trường
mới [`VerifyOptions.CertificatePolicies`](/pkg/crypto/x509#VerifyOptions.CertificatePolicies)
có thể được đặt thành tập hợp các OID chính sách có thể chấp nhận.
Chỉ các chuỗi chứng chỉ với biểu đồ chính sách hợp lệ mới được trả về từ
[`Certificate.Verify`](/pkg/crypto/x509#Certificate.Verify).

[`MarshalPKCS8PrivateKey`](/pkg/crypto/x509#MarshalPKCS8PrivateKey) giờ trả về
lỗi thay vì marshaling khóa RSA không hợp lệ.
([`MarshalPKCS1PrivateKey`](/pkg/crypto/x509#MarshalPKCS1PrivateKey) không
có giá trị trả về lỗi, và hành vi của nó khi cung cấp khóa không hợp lệ vẫn
không xác định.)

[`ParsePKCS1PrivateKey`](/pkg/crypto/x509#ParsePKCS1PrivateKey) và
[`ParsePKCS8PrivateKey`](/pkg/crypto/x509#ParsePKCS8PrivateKey) giờ sử dụng và
xác thực các giá trị CRT được mã hóa, vì vậy có thể từ chối các khóa RSA không hợp lệ đã
được chấp nhận trước đây. Dùng [cài đặt GODEBUG](/doc/godebug) `x509rsacrt=0` để
quay lại tính toán lại các giá trị CRT.

#### [`debug/elf`](/pkg/debug/elf/)

<!-- go.dev/issue/63952 -->

Gói [`debug/elf`](/pkg/debug/elf) thêm hỗ trợ để xử lý các phiên bản symbol
trong các tệp ELF động (Executable and Linkable Format).
Phương thức mới [`File.DynamicVersions`](/pkg/debug/elf#File.DynamicVersions)
trả về danh sách các phiên bản động được định nghĩa trong tệp ELF.
Phương thức mới [`File.DynamicVersionNeeds`](/pkg/debug/elf#File.DynamicVersionNeeds)
trả về danh sách các phiên bản động được yêu cầu bởi tệp ELF này được
định nghĩa trong các đối tượng ELF khác.
Cuối cùng, các trường mới [`Symbol.HasVersion`](/pkg/debug/elf#Symbol) và
[`Symbol.VersionIndex`](/pkg/debug/elf#Symbol) cho biết phiên bản của
symbol.

#### [`encoding`](/pkg/encoding/)

Hai interface mới, [`TextAppender`](/pkg/encoding#TextAppender) và [`BinaryAppender`](/pkg/encoding#BinaryAppender), đã được
giới thiệu để thêm biểu diễn văn bản hoặc nhị phân của đối tượng
vào byte slice. Các interface này cung cấp cùng chức năng với
[`TextMarshaler`](/pkg/encoding#TextMarshaler) và [`BinaryMarshaler`](/pkg/encoding#BinaryMarshaler), nhưng thay vì cấp phát slice mới
mỗi lần, chúng thêm dữ liệu trực tiếp vào slice hiện có.
Các interface này giờ được triển khai bởi các kiểu thư viện chuẩn đã
triển khai `TextMarshaler` và/hoặc `BinaryMarshaler`.

#### [`encoding/json`](/pkg/encoding/json/)

<!-- go.dev/issue/45669 -->
Khi marshaling, trường struct với tùy chọn `omitzero` mới trong thẻ trường struct
sẽ bị bỏ qua nếu giá trị của nó là zero. Nếu kiểu trường có phương thức `IsZero() bool`,
phương thức đó sẽ được dùng để xác định liệu giá trị có phải zero không. Nếu không,
giá trị là zero nếu nó là [giá trị zero cho kiểu của nó](/ref/spec#The_zero_value).
Thẻ trường `omitzero` rõ ràng hơn và ít dễ gây lỗi hơn `omitempty` khi
mục đích là bỏ qua các giá trị zero.
Đặc biệt, không giống `omitempty`, `omitzero` bỏ qua các giá trị
[`time.Time`](/pkg/time#Time) có giá trị zero, đây là nguồn gốc phổ biến của vấn đề.

Nếu cả `omitempty` và `omitzero` được chỉ định, trường sẽ bị bỏ qua nếu giá trị
là rỗng hoặc zero (hoặc cả hai).

[`UnmarshalTypeError.Field`](/pkg/encoding/json#UnmarshalTypeError.Field) giờ bao gồm các struct nhúng để cung cấp thông báo lỗi chi tiết hơn.

#### [`go/types`](/pkg/go/types/)

Tất cả các cấu trúc dữ liệu `go/types` hiển thị các chuỗi bằng một cặp
phương thức như `Len() int` và `At(int) T` giờ cũng có các phương thức trả về
iterator, cho phép bạn đơn giản hóa mã như:

```
params := fn.Type.(*types.Signature).Params()
for i := 0; i < params.Len(); i++ {
   use(params.At(i))
}
```

thành:

```
for param := range fn.Signature().Params().Variables() {
   use(param)
}
```

Các phương thức là:
[`Interface.EmbeddedTypes`](/pkg/go/types#Interface.EmbeddedTypes),
[`Interface.ExplicitMethods`](/pkg/go/types#Interface.ExplicitMethods),
[`Interface.Methods`](/pkg/go/types#Interface.Methods),
[`MethodSet.Methods`](/pkg/go/types#MethodSet.Methods),
[`Named.Methods`](/pkg/go/types#Named.Methods),
[`Scope.Children`](/pkg/go/types#Scope.Children),
[`Struct.Fields`](/pkg/go/types#Struct.Fields),
[`Tuple.Variables`](/pkg/go/types#Tuple.Variables),
[`TypeList.Types`](/pkg/go/types#TypeList.Types),
[`TypeParamList.TypeParams`](/pkg/go/types#TypeParamList.TypeParams),
[`Union.Terms`](/pkg/go/types#Union.Terms).

#### [`hash/adler32`](/pkg/hash/adler32/)

Giá trị được trả về bởi [`New`](/pkg/hash/adler32#New) giờ cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`hash/crc32`](/pkg/hash/crc32/)

Các giá trị được trả về bởi [`New`](/pkg/hash/crc32#New) và [`NewIEEE`](/pkg/hash/crc32#NewIEEE) giờ cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`hash/crc64`](/pkg/hash/crc64/)

Giá trị được trả về bởi [`New`](/pkg/hash/crc64#New) giờ cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`hash/fnv`](/pkg/hash/fnv/)

Các giá trị được trả về bởi [`New32`](/pkg/hash/fnv#New32), [`New32a`](/pkg/hash/fnv#New32a), [`New64`](/pkg/hash/fnv#New64), [`New64a`](/pkg/hash/fnv#New64a), [`New128`](/pkg/hash/fnv#New128) và [`New128a`](/pkg/hash/fnv#New128a) giờ cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`hash/maphash`](/pkg/hash/maphash/)

Các hàm mới [`Comparable`](/pkg/hash/maphash#Comparable) và
[`WriteComparable`](/pkg/hash/maphash#WriteComparable) có thể tính toán
hash của bất kỳ giá trị có thể so sánh nào.
Điều này giúp có thể hash bất cứ thứ gì có thể được dùng làm key của map Go.

#### [`log/slog`](/pkg/log/slog/)

[`DiscardHandler`](/pkg/log/slog#DiscardHandler) mới là handler không bao giờ được bật và luôn loại bỏ đầu ra của nó.

[`Level`](/pkg/log/slog#Level) và [`LevelVar`](/pkg/log/slog#LevelVar) giờ triển khai interface [`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`math/big`](/pkg/math/big/)

[`Float`](/pkg/math/big#Float), [`Int`](/pkg/math/big#Int) và [`Rat`](/pkg/math/big#Rat) giờ triển khai interface [`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`math/rand`](/pkg/math/rand/)

Các lệnh gọi đến hàm [`Seed`](/pkg/math/rand#Seed) cấp cao nhất bị deprecated không còn có tác dụng gì. Để
khôi phục hành vi cũ, hãy dùng [cài đặt GODEBUG](/doc/godebug) `randseednop=0`. Để biết thêm thông tin xem
[đề xuất #67273](/issue/67273).

#### [`math/rand/v2`](/pkg/math/rand/v2/)

[`ChaCha8`](/pkg/math/rand/v2#ChaCha8) và [`PCG`](/pkg/math/rand/v2#PCG) giờ triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`net`](/pkg/net/)

[`ListenConfig`](/pkg/net#ListenConfig) giờ sử dụng MPTCP theo mặc định trên các hệ thống hỗ trợ nó
(hiện chỉ trên Linux).

[`IP`](/pkg/net#IP) giờ triển khai interface [`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`net/http`](/pkg/net/http/)

Giới hạn của [`Transport`](/pkg/net/http#Transport) đối với các phản hồi thông tin 1xx nhận được
trong phản hồi yêu cầu đã thay đổi.
Trước đây, nó hủy bỏ yêu cầu và trả về lỗi sau khi
nhận được hơn 5 phản hồi 1xx.
Giờ nó trả về lỗi nếu tổng kích thước của tất cả phản hồi 1xx
vượt quá cài đặt cấu hình [`Transport.MaxResponseHeaderBytes`](/pkg/net/http#Transport.MaxResponseHeaderBytes).

Ngoài ra, khi một yêu cầu có
hook trace [`net/http/httptrace.ClientTrace.Got1xxResponse`](/pkg/net/http/httptrace#ClientTrace.Got1xxResponse),
giờ không có giới hạn về tổng số phản hồi 1xx.
Hook `Got1xxResponse` có thể trả về lỗi để hủy bỏ yêu cầu.

[`Transport`](/pkg/net/http#Transport) và [`Server`](/pkg/net/http#Server) giờ có trường HTTP2 cho phép
cấu hình cài đặt giao thức HTTP/2.

Các trường mới [`Server.Protocols`](/pkg/net/http#Server.Protocols) và [`Transport.Protocols`](/pkg/net/http#Transport.Protocols) cung cấp
cách đơn giản để cấu hình các giao thức HTTP mà server hoặc client sử dụng.

Server và client có thể được cấu hình để hỗ trợ các kết nối HTTP/2 không mã hóa.

Khi [`Server.Protocols`](/pkg/net/http#Server.Protocols) chứa UnencryptedHTTP2, server sẽ chấp nhận
kết nối HTTP/2 trên các cổng không mã hóa. Server có thể chấp nhận cả
HTTP/1 và HTTP/2 không mã hóa trên cùng một cổng.

Khi [`Transport.Protocols`](/pkg/net/http#Transport.Protocols) chứa UnencryptedHTTP2 và không chứa
HTTP1, transport sẽ sử dụng HTTP/2 không mã hóa cho các URL http://.
Nếu transport được cấu hình để sử dụng cả HTTP/1 và HTTP/2 không mã hóa,
nó sẽ dùng HTTP/1.

Hỗ trợ HTTP/2 không mã hóa sử dụng "HTTP/2 with Prior Knowledge"
(RFC 9113, mục 3.3). Header "Upgrade: h2c" bị deprecated
không được hỗ trợ.

#### [`net/netip`](/pkg/net/netip/)

[`Addr`](/pkg/net/netip#Addr), [`AddrPort`](/pkg/net/netip#AddrPort) và [`Prefix`](/pkg/net/netip#Prefix) giờ triển khai các interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender) và
[`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`net/url`](/pkg/net/url/)

[`URL`](/pkg/net/url#URL) giờ cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`os/user`](/pkg/os/user/)

Trên Windows, [`Current`](/pkg/os/user#Current) giờ có thể được dùng trong Windows Nano Server.
Triển khai đã được cập nhật để tránh sử dụng các hàm
từ thư viện `NetApi32`, không có sẵn trong Nano Server.

Trên Windows, [`Current`](/pkg/os/user#Current), [`Lookup`](/pkg/os/user#Lookup) và [`LookupId`](/pkg/os/user#LookupId) giờ hỗ trợ các
tài khoản người dùng dịch vụ tích hợp sau:
- `NT AUTHORITY\SYSTEM`
- `NT AUTHORITY\LOCAL SERVICE`
- `NT AUTHORITY\NETWORK SERVICE`

Trên Windows, [`Current`](/pkg/os/user#Current) đã được thực hiện nhanh hơn đáng kể khi
người dùng hiện tại tham gia vào domain chậm, đây là
trường hợp thông thường cho nhiều người dùng doanh nghiệp. Hiệu suất triển khai mới
giờ ở mức mili giây, so với triển khai trước có thể mất vài giây,
hoặc thậm chí phút, để hoàn thành.

Trên Windows, [`Current`](/pkg/os/user#Current) giờ trả về người dùng chủ sở hữu tiến trình khi
luồng hiện tại đang mạo danh người dùng khác. Trước đây,
nó trả về lỗi.

#### [`regexp`](/pkg/regexp/)

[`Regexp`](/pkg/regexp#Regexp) giờ triển khai interface [`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`runtime`](/pkg/runtime/)

Hàm [`GOROOT`](/pkg/runtime#GOROOT) giờ bị deprecated.
Trong mã mới, hãy ưu tiên dùng đường dẫn hệ thống để định vị tệp nhị phân "go",
và dùng `go env GOROOT` để tìm GOROOT của nó.

#### [`strings`](/pkg/strings/)

Gói [`strings`](/pkg/strings) thêm một số hàm hoạt động với iterator:
- [`Lines`](/pkg/strings#Lines) trả về iterator qua
  các dòng kết thúc bằng ký tự newline trong chuỗi.
- [`SplitSeq`](/pkg/strings#SplitSeq) trả về iterator qua
  tất cả các chuỗi con của một chuỗi được chia xung quanh dấu phân cách.
- [`SplitAfterSeq`](/pkg/strings#SplitAfterSeq) trả về iterator
  qua các chuỗi con của một chuỗi được chia sau mỗi lần xuất hiện dấu phân cách.
- [`FieldsSeq`](/pkg/strings#FieldsSeq) trả về iterator qua
  các chuỗi con của một chuỗi được chia xung quanh các chuỗi ký tự khoảng trắng,
  như được định nghĩa bởi [`unicode.IsSpace`](/pkg/unicode#IsSpace).
- [`FieldsFuncSeq`](/pkg/strings#FieldsFuncSeq) trả về iterator
  qua các chuỗi con của một chuỗi được chia xung quanh các chuỗi code point Unicode
  thỏa mãn một vị từ.

#### [`sync`](/pkg/sync/)

Triển khai của [`sync.Map`](/pkg/sync#Map) đã được thay đổi, cải thiện hiệu suất,
đặc biệt là cho các sửa đổi map.
Ví dụ, các sửa đổi của các tập hợp khóa tách rời ít có khả năng tranh chấp hơn trên
các map lớn hơn, và không còn cần thời gian tăng tốc nào để đạt được các lần đọc ít tranh chấp
từ map.

Nếu bạn gặp bất kỳ vấn đề nào, hãy đặt `GOEXPERIMENT=nosynchashtriemap` tại thời điểm build
để chuyển lại triển khai cũ và hãy [file một issue](/issue/new).

#### [`testing`](/pkg/testing/)

Các phương thức mới [`T.Context`](/pkg/testing#T.Context) và [`B.Context`](/pkg/testing#B.Context) trả về context bị hủy
sau khi test hoàn thành và trước khi các hàm dọn dẹp test chạy.

<!-- testing.B.Loop mentioned in 6-stdlib/6-testing-bloop.md. -->

Các phương thức mới [`T.Chdir`](/pkg/testing#T.Chdir) và [`B.Chdir`](/pkg/testing#B.Chdir) có thể được dùng để thay đổi thư mục làm việc
trong suốt thời gian của một test hoặc benchmark.

#### [`text/template`](/pkg/text/template/)

Các template giờ hỗ trợ range-over-func và range-over-int.

#### [`time`](/pkg/time/)

[`Time`](/pkg/time#Time) giờ triển khai các interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender) và [`encoding.TextAppender`](/pkg/encoding#TextAppender).

## Các cổng {#ports}

### Linux {#linux}

<!-- go.dev/issue/67001 -->
Như đã [thông báo](go1.23#linux) trong ghi chú phát hành Go 1.23, Go 1.24 yêu cầu phiên bản
kernel Linux 3.2 trở lên.

### Darwin {#darwin}

<!-- go.dev/issue/69839 -->
Go 1.24 là bản phát hành cuối cùng sẽ chạy trên macOS 11 Big Sur.
Go 1.25 sẽ yêu cầu macOS 12 Monterey trở lên.

### WebAssembly {#wasm}

<!-- go.dev/issue/65199, CL 603055 -->
Directive trình biên dịch `go:wasmexport` được thêm để các chương trình Go xuất các hàm
sang WebAssembly host.

Trên WebAssembly System Interface Preview 1 (`GOOS=wasip1 GOARCH=wasm`), Go 1.24 hỗ trợ
xây dựng chương trình Go như một
[reactor/library](https://github.com/WebAssembly/WASI/blob/63a46f61052a21bfab75a76558485cf097c0dbba/legacy/application-abi.md#current-unstable-abi),
bằng cách chỉ định cờ build `-buildmode=c-shared`.

<!-- go.dev/issue/66984, CL 626615 -->
Nhiều kiểu hơn giờ được phép làm kiểu đối số hoặc kiểu kết quả cho các hàm `go:wasmimport`.
Cụ thể, `bool`, `string`, `uintptr`, và con trỏ đến một số kiểu nhất định được cho phép
(xem [tài liệu](/pkg/cmd/compile#hdr-WebAssembly_Directives) để biết chi tiết),
cùng với các kiểu số nguyên và float 32-bit và 64-bit, và `unsafe.Pointer`, đã
được cho phép.
Các kiểu này cũng được phép làm kiểu đối số hoặc kiểu kết quả cho các hàm `go:wasmexport`.

<!-- go.dev/issue/68024 -->
Các tệp hỗ trợ cho WebAssembly đã được chuyển sang `lib/wasm` từ `misc/wasm`.

<!-- CL 621635, CL 621636 -->
Kích thước bộ nhớ ban đầu giảm đáng kể, đặc biệt cho các ứng dụng WebAssembly nhỏ.

### Windows {#windows}

<!-- go.dev/issue/70705 -->
Cổng windows/arm 32-bit (`GOOS=windows GOARCH=arm`) đã bị đánh dấu là lỗi.
Xem [issue #70705](/issue/70705) để biết chi tiết.

<!-- Items that don't need to be mentioned in Go 1.24 release notes but are picked up by relnote todo.

accepted proposal https://go.dev/issue/25309 (from https://go.dev/cl/594018, https://go.dev/cl/595120, https://go.dev/cl/595564, https://go.dev/cl/601778) - new x/crypto package; doesn't seem to need to be mentioned
accepted proposal https://go.dev/issue/43744 (from https://go.dev/cl/357530) - no change in the standard library
accepted proposal https://go.dev/issue/60905 (from https://go.dev/cl/610195) - CL 610195 reverted
accepted proposal https://go.dev/issue/61395 (from https://go.dev/cl/594738, https://go.dev/cl/594976) - CL 594738 made sync/atomic AND/OR operations intrinsic on amd64, but the API was already added in Go 1.23; CL 594976 is a fix; probably doesn't require a Go 1.24 release note (performance change only)
accepted proposal https://go.dev/issue/51269 (from https://go.dev/cl/627035) - may be worth mentioning in Go 1.24 release notes, or may be fine to leave out; commented at https://go.dev/issue/51269#issuecomment-2501802763; Ian confirmed it's fine to leave out
accepted proposal https://go.dev/issue/66540 (from https://go.dev/cl/603958) - a Go language spec clarification; might not need to be mentioned in Go 1.24 release notes; left a comment at https://go.dev/issue/66540#issuecomment-2502051684; Robert confirmed it indeed doesn't
accepted proposal https://go.dev/issue/34208 (from https://go.dev/cl/586241) - CL 586241 implements a fix for a Go 1.23 feature, doesn't seem to be need anything in Go 1.24 release notes
accepted proposal https://go.dev/issue/43993 (from https://go.dev/cl/626116) - CL 626116 prepares the tree towards the vet change but the vet change itself isn't implemented in Go 1.24, so nothing to say in Go 1.24 release notes
accepted proposal https://go.dev/issue/44505 (from https://go.dev/cl/609955) - CL 609955 is an internal cleanup in x/tools, no need for Go 1.24 release note
accepted proposal https://go.dev/issue/61476 (from https://go.dev/cl/608255) - CL 608255 builds on GORISCV64 added in Go 1.23; nothing to mention in Go 1.24 release notes
accepted proposal https://go.dev/issue/66315 (from https://go.dev/cl/577996) - adding Pass.Module field to x/tools/go/analysis doesn't seem like something that needs to be mentioned in Go 1.24 release notes
accepted proposal https://go.dev/issue/57786 (from https://go.dev/cl/472717) - CL 472717 is in x/net/http2 and mentions a Go 1.21 proposal; it doesn't seem to need anything in Go 1.24 release notes
accepted proposal https://go.dev/issue/54265 (from https://go.dev/cl/609915, https://go.dev/cl/610675) - CLs that refer to a Go 1.22 proposal, nothing more is needed in Go 1.24 release notes
accepted proposal https://go.dev/issue/53021 (from https://go.dev/cl/622276) - CL 622276 improves docs; proposal 53021 was in Go 1.20 so nothing more is needed in Go 1.24 release notes
accepted proposal https://go.dev/issue/51430 (from https://go.dev/cl/613375) - CL 613375 is an internal documentation comment; proposal 51430 happened in Go 1.20/1.21 so nothing more is needed in Go 1.24 release notes
accepted proposal https://go.dev/issue/38445 (from https://go.dev/cl/626495) - CL 626495 works on proposal 38445, which is about x/tools/go/package, doesn't need anything in Go 1.24 release notes
accepted proposal https://go.dev/issue/56986 (from https://go.dev/cl/618115) - CL 618115 adds documentation; it doesn't need to be mentioned in Go 1.24 release notes
accepted proposal https://go.dev/issue/60061 (from https://go.dev/cl/612038) - CL 612038 is a CL that deprecates something in x/tools/go/ast and mentions a Go 1.22 proposal; doesn't need anything in Go 1.24 release notes
accepted proposal https://go.dev/issue/61324 (from https://go.dev/cl/411907) - CL 411907 is an x/tools CL that implements a proposal for a new package there; doesn't need anything in Go 1.24 release notes
accepted proposal https://go.dev/issue/61777 (from https://go.dev/cl/601496) - CL 601496 added a WriteByteTimeout field to x/net/http2.Server; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/61940 (from https://go.dev/cl/600997) - CL 600997 deleted obsolete code in x/build and mentioned an accepted proposal; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/62113 (from https://go.dev/cl/594195) - CL 594195 made iterator-related additions in x/net/html; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/62484 (from https://go.dev/cl/600775) - CL 600775 documents CopyFS symlink behavior and mentions the Go 1.23 proposal; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/64207 (from https://go.dev/cl/605875) - an x/website CL that follows up on a Go 1.23 proposal; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/65236 (from https://go.dev/cl/596135) - CL 596135 adds tests for the Go 1.23 proposal 65236; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/67795 (from https://go.dev/cl/616218) - iteratior support for x/tools/go/ast/inspector; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/67812 (from https://go.dev/cl/601497) - configurable server pings for x/net/http2.Server; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/68232 (from https://go.dev/cl/595676) - x/sys/unix additions; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/68898 (from https://go.dev/cl/607495, https://go.dev/cl/620036, https://go.dev/cl/620135, https://go.dev/cl/623638) - a proposal for x/tools/go/gcexportdata to document 2 releases + tip support policy; since the change is in x/tools it doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/69095 (from https://go.dev/cl/593683, https://go.dev/cl/608955, https://go.dev/cl/610716) - a proposal that affects maintenance and support of golang.org/x repositories; doesn't need to be mentioned in Go 1.24 release notes
accepted proposal https://go.dev/issue/68384 (from https://go.dev/cl/611875) - expanding the scope of Go Telemetry to include Delve isn't directly tied to Go 1.24 and doesn't seem to need to be mentioned in Go 1.24 release notes
accepted proposal https://go.dev/issue/69291 (from https://go.dev/cl/610939) - CL 610939 refactors code in x/tools and mentions the still-open proposal #69291 to add Reachable to x/tools/go/ssa/ssautil; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/69360 (from https://go.dev/cl/614158, https://go.dev/cl/614159, https://go.dev/cl/614635, https://go.dev/cl/614675) - proposal 69360 is to tag and delete gorename from x/tools; doesn't need a Go 1.24 release note
accepted proposal https://go.dev/issue/61417 (from https://go.dev/cl/605955) - a new field in x/oauth2; nothing to mention in Go 1.24 release notes
accepted proposal https://go.dev/issue/29266 (from https://go.dev/cl/632897) - a documentation-only proposal for go.dev/doc/contribute; doesn't need a Go 1.24 release note
-->
