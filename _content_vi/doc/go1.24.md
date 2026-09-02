---
title: Ghi chú bản phát hành Go 1.24
---

<style>
  main ul li { margin: 0.5em 0; }
</style>

## Giới thiệu về Go 1.24 {#introduction}

Bản phát hành Go mới nhất, phiên bản 1.24,
ra mắt vào [tháng 2 năm 2025](/doc/devel/release#go1.24.0),
sáu tháng sau [Go 1.23](/doc/go1.23).
Phần lớn các thay đổi của bản phát hành này nằm trong việc triển khai toolchain, runtime và các thư viện.
Như thường lệ, bản phát hành duy trì [cam kết về khả năng tương thích](/doc/go1compat) của Go 1.
Chúng tôi kỳ vọng hầu hết các chương trình Go sẽ tiếp tục biên dịch và chạy như trước.

## Các thay đổi đối với ngôn ngữ {#language}

<!-- go.dev/issue/46477 -->
Go 1.24 hiện hỗ trợ đầy đủ [bí danh kiểu generic](/issue/46477): một bí danh kiểu
có thể được tham số hóa giống như một kiểu được định nghĩa.
Xem [đặc tả ngôn ngữ](/ref/spec#Alias_declarations) để biết chi tiết.
Hiện tại, tính năng này có thể bị vô hiệu hóa bằng cách đặt `GOEXPERIMENT=noaliastypeparams`;
nhưng thiết lập `aliastypeparams` sẽ bị xóa trong Go 1.25.

## Công cụ {#tools}

### Lệnh Go {#go-command}

<!-- go.dev/issue/48429 -->

Các module Go giờ đây có thể theo dõi các dependency thực thi bằng cách sử dụng các chỉ thị `tool` trong
go.mod. Điều này loại bỏ nhu cầu về giải pháp tạm thời trước đây là thêm các công cụ dưới dạng
blank import vào một tệp thường được đặt tên là "tools.go". Lệnh `go tool`
giờ đây có thể chạy các công cụ này bên cạnh các công cụ được cung cấp cùng với bản phân phối Go.
Để biết thêm thông tin, xem [tài liệu](/doc/modules/managing-dependencies#tools).

Cờ `-tool` mới cho `go get` khiến một chỉ thị tool được thêm vào
module hiện tại cho các gói được chỉ định, đồng thời thêm các chỉ thị require.

[Meta-pattern `tool` mới](/cmd/go#hdr-Package_lists_and_patterns) tham chiếu đến
tất cả các công cụ trong module hiện tại. Có thể sử dụng nó để nâng cấp tất cả chúng bằng `go get tool` hoặc cài đặt chúng vào thư mục GOBIN của bạn bằng `go install tool`.

<!-- go.dev/issue/69290 -->

Các tệp thực thi được tạo bởi `go run` và hành vi mới của `go tool` giờ đây
được lưu vào Go build cache. Điều này giúp các lần thực thi lặp lại nhanh hơn
đổi lại làm cache lớn hơn. Xem [#69290](/issue/69290).

<!-- go.dev/issue/62067 -->

Các lệnh `go build` và `go install` giờ đây chấp nhận cờ `-json`, cờ này báo cáo
đầu ra build và lỗi dưới dạng đầu ra JSON có cấu trúc trên đầu ra chuẩn.
Để biết chi tiết về định dạng báo cáo, xem `go help buildjson`.

Ngoài ra, `go test -json` giờ đây báo cáo đầu ra build và lỗi ở dạng JSON,
xen kẽ với JSON kết quả kiểm thử.
Các dữ liệu này được phân biệt bằng các kiểu `Action` mới, nhưng nếu chúng gây ra vấn đề
trong hệ thống tích hợp kiểm thử, bạn có thể quay lại đầu ra build dạng văn bản bằng
[thiết lập GODEBUG](/doc/godebug) `gotestjsonbuildtext=1`.

<!-- go.dev/issue/26232 -->

Biến môi trường `GOAUTH` mới cung cấp một cách linh hoạt để xác thực
các lần tải module riêng tư. Xem `go help goauth` để biết thêm thông tin.

<!-- go.dev/issue/50603 -->

Lệnh `go build` giờ đây đặt
[phiên bản của module chính](/pkg/runtime/debug#BuildInfo.Main) trong binary đã biên dịch
dựa trên thẻ của hệ thống quản lý phiên bản và/hoặc commit.
Hậu tố `+dirty` sẽ được thêm vào nếu có các thay đổi chưa commit.
Sử dụng cờ `-buildvcs=false` để loại bỏ thông tin quản lý phiên bản khỏi binary.

<!-- go.dev/issue/63939 -->

[Thiết lập GODEBUG mới](/doc/godebug) [`toolchaintrace=1`](/doc/toolchain#select)
có thể được sử dụng để theo dõi quá trình lựa chọn toolchain của lệnh `go`.

### Cgo {#cgo}

<!-- go.dev/issue/56378, CL 579955 -->
Cgo hỗ trợ các chú thích mới cho các hàm C nhằm cải thiện hiệu năng thời gian chạy.
`#cgo noescape cFunctionName` cho trình biên dịch biết rằng bộ nhớ được truyền đến
hàm C `cFunctionname` không thoát ra ngoài.
`#cgo nocallback cFunctionName` cho trình biên dịch biết rằng hàm C
`cFunctionName` không gọi ngược đến bất kỳ hàm Go nào.
Để biết thêm thông tin, xem [tài liệu cgo](/pkg/cmd/cgo#hdr-Optimizing_calls_of_C_code).

<!-- go.dev/issue/67699 -->
Hiện tại Cgo từ chối biên dịch các lời gọi đến một hàm C có nhiều khai báo
không tương thích. Ví dụ, nếu `f` được khai báo vừa là `void f(int)`
vừa là `void f(double)`, cgo sẽ báo lỗi thay vì có thể tạo ra một chuỗi lời gọi
không chính xác cho `f(0)`. Điểm mới trong bản phát hành này là bộ phát hiện tốt hơn
cho điều kiện lỗi này khi các khai báo không tương thích xuất hiện trong các
tệp khác nhau. Xem [#67699](/issue/67699).

### Objdump

<!-- go.dev/issue/15255, go.dev/issue/36738 -->
Công cụ [objdump](/cmd/objdump) hiện hỗ trợ việc disassembly trên 64-bit
LoongArch (`GOARCH=loong64`), RISC-V (`GOARCH=riscv64`) và S390X (`GOARCH=s390x`).

### Vet

<!-- go.dev/issue/44251 -->
Bộ phân tích `tests` mới báo cáo các lỗi thường gặp trong khai báo của
test, fuzzers, benchmark và ví dụ trong các gói test, chẳng hạn như
tên không đúng định dạng, chữ ký không chính xác hoặc các ví dụ ghi lại
những định danh không tồn tại. Một số lỗi này có thể khiến các test không
chạy.
Bộ phân tích này nằm trong tập hợp con các bộ phân tích được chạy bởi `go test`.

<!-- go.dev/issue/60529 -->
Bộ phân tích `printf` hiện có giờ báo cáo chẩn đoán cho các lời gọi có
dạng `fmt.Printf(s)`, trong đó `s` là một chuỗi định dạng không phải hằng số,
không có đối số nào khác. Những lời gọi như vậy gần như luôn là lỗi
vì giá trị của `s` có thể chứa ký hiệu `%`; thay vào đó hãy dùng `fmt.Print`.
Xem [#60529](/issue/60529). Kiểm tra này có xu hướng phát hiện lỗi trong
mã hiện có, vì vậy chỉ được áp dụng khi phiên bản ngôn ngữ (được chỉ định bởi
chỉ thị `go` trong go.mod hoặc các chú thích `//go:build`) ít nhất là Go 1.24,
nhằm tránh gây lỗi tích hợp liên tục khi cập nhật lên bộ công cụ Go 1.24.

<!-- go.dev/issue/64127 -->
Bộ phân tích `buildtag` hiện có giờ báo cáo chẩn đoán khi có một ràng buộc
xây dựng phiên bản chính [không hợp lệ của Go](/pkg/cmd/go#hdr-Build_constraints)
trong chỉ thị `//go:build`. Ví dụ, `//go:build go1.23.1` đề cập đến một
bản phát hành điểm; thay vào đó hãy dùng `//go:build go1.23`.
Xem [#64127](/issue/64127).

<!-- go.dev/issue/66387 -->
Bộ phân tích `copylock` hiện có giờ báo cáo chẩn đoán khi một
biến được khai báo trong vòng lặp "for" 3 mệnh đề như
`for i := iter(); done(i); i = next(i) { ... }` chứa một `sync.Locker`,
chẳng hạn như `sync.Mutex`. [Go 1.22](/doc/go1.22#language) đã thay đổi hành vi
của các vòng lặp này để tạo một biến mới cho mỗi lần lặp, sao chép
giá trị từ lần lặp trước; thao tác sao chép này không an toàn đối với khóa.
Xem [#66387](/issue/66387).

### GOCACHEPROG

<!-- go.dev/issue/64876 -->
Cơ chế lưu đệm nhị phân nội bộ và lưu đệm kiểm thử của `cmd/go` giờ đây có thể được triển khai bởi các tiến trình con thực hiện một giao thức JSON giữa công cụ `cmd/go` và tiến trình con được đặt tên bởi biến môi trường `GOCACHEPROG`.
Trước đây, tính năng này nằm sau một GOEXPERIMENT.
Để biết chi tiết về giao thức, xem [tài liệu](/cmd/go/internal/cacheprog).

## Runtime {#runtime}

<!-- go.dev/issue/54766 -->
<!-- go.dev/cl/614795 -->
<!-- go.dev/issue/68578 -->

Một số cải tiến hiệu năng đối với runtime đã làm giảm chi phí CPU trung bình 2–3% trên một bộ benchmark đại diện.
Kết quả có thể khác nhau tùy theo ứng dụng.
Các cải tiến này bao gồm một triển khai `map` builtin mới dựa trên
[Swiss Tables](https://abseil.io/about/design/swisstables), cấp phát bộ nhớ hiệu quả hơn cho các đối tượng nhỏ và một triển khai mutex mới nội bộ của runtime.

Triển khai `map` builtin mới và mutex mới nội bộ của runtime có thể lần lượt được vô hiệu hóa bằng cách đặt `GOEXPERIMENT=noswissmap` và `GOEXPERIMENT=nospinbitmutex` tại thời điểm build.

## Compiler {#compiler}

<!-- go.dev/issue/60725, go.dev/issue/57926 -->
Trình biên dịch trước đây đã không cho phép định nghĩa các phương thức mới với kiểu receiver được tạo bởi cgo, nhưng có thể vượt qua hạn chế đó thông qua một kiểu bí danh.
Go 1.24 giờ đây luôn báo lỗi nếu receiver biểu thị một kiểu được tạo bởi cgo, dù trực tiếp hay gián tiếp (thông qua một kiểu bí danh).

## Linker {#linker}

<!-- go.dev/issue/68678, go.dev/issue/68652, CL 618598, CL 618601 -->
Linker giờ đây tạo GNU build ID (ghi chú ELF `NT_GNU_BUILD_ID`) trên các nền tảng ELF và UUID (lệnh nạp Mach-O `LC_UUID`) trên macOS theo mặc định.
Build ID hoặc UUID được suy ra từ Go build ID.
Nó có thể được vô hiệu hóa bằng cờ linker `-B none`, hoặc được ghi đè bằng cờ linker `-B 0xNNNN` với giá trị thập lục phân do người dùng chỉ định.

## Bootstrap {#bootstrap}

<!-- go.dev/issue/64751 -->
Như đã đề cập trong [ghi chú bản phát hành Go 1.22](/doc/go1.22#bootstrap), Go 1.24 giờ đây yêu cầu
Go 1.22.6 hoặc mới hơn để bootstrap.
Chúng tôi dự kiến rằng Go 1.26 sẽ yêu cầu một point release của Go 1.24 hoặc mới hơn để bootstrap.

## Thư viện chuẩn {#library}

### Truy cập hệ thống tệp giới hạn theo thư mục

<!-- go.dev/issue/67002 -->
Kiểu mới [`os.Root`](/pkg/os#Root) cung cấp khả năng thực hiện các thao tác hệ thống tệp trong một thư mục cụ thể.

Hàm [`os.OpenRoot`](/pkg/os#OpenRoot) mở một thư mục và trả về một [`os.Root`](/pkg/os#Root).
Các phương thức trên [`os.Root`](/pkg/os#Root) hoạt động trong phạm vi thư mục và không cho phép các đường dẫn tham chiếu đến vị trí bên ngoài thư mục, bao gồm cả
những đường dẫn theo liên kết tượng trưng ra ngoài thư mục.
Các phương thức trên `os.Root` mô phỏng hầu hết các thao tác hệ thống tệp có trong gói
`os`, bao gồm ví dụ [`os.Root.Open`](/pkg/os#Root.Open),
[`os.Root.Create`](/pkg/os#Root.Create),
[`os.Root.Mkdir`](/pkg/os#Root.Mkdir),
và [`os.Root.Stat`](/pkg/os#Root.Stat),

### Hàm benchmark mới

Benchmark hiện có thể sử dụng phương thức [`testing.B.Loop`](/pkg/testing#B.Loop) nhanh hơn và ít gây lỗi hơn để thực hiện các lần lặp benchmark như `for b.Loop() { ... }` thay cho các cấu trúc vòng lặp thông thường liên quan đến `b.N` như `for range b.N`. Điều này mang lại hai lợi ích đáng kể:
- Hàm benchmark sẽ thực thi chính xác một lần cho mỗi -count, vì vậy các bước thiết lập và dọn dẹp tốn kém chỉ được thực hiện một lần.
- Các tham số và kết quả của lệnh gọi hàm được giữ sống, ngăn trình biên dịch tối ưu hóa hoàn toàn phần thân vòng lặp.

### Cải thiện finalizer

<!-- go.dev/issue/67535 -->
Hàm mới [`runtime.AddCleanup`](/pkg/runtime#AddCleanup) là một
cơ chế hoàn tất linh hoạt hơn, hiệu quả hơn và ít gây lỗi hơn so với [`runtime.SetFinalizer`](/pkg/runtime#SetFinalizer).
`AddCleanup` gắn một hàm dọn dẹp vào một đối tượng, hàm này sẽ chạy khi
đối tượng không còn được tham chiếu đến.
Tuy nhiên, không giống như `SetFinalizer`,
có thể gắn nhiều hàm dọn dẹp vào một đối tượng,
có thể gắn hàm dọn dẹp vào các con trỏ bên trong,
các hàm dọn dẹp nhìn chung không gây rò rỉ khi các đối tượng tạo thành một chu kỳ, và
các hàm dọn dẹp không trì hoãn việc giải phóng một đối tượng hoặc các đối tượng mà nó trỏ tới.
Mã mới nên ưu tiên `AddCleanup` thay vì `SetFinalizer`.

### Gói weak mới {#weak}

Gói [`weak`](/pkg/weak/) mới cung cấp các con trỏ yếu.

Con trỏ yếu là một nguyên thủy cấp thấp được cung cấp để cho phép
tạo các cấu trúc tiết kiệm bộ nhớ, chẳng hạn như các weak map để
liên kết giá trị, các map chuẩn hóa cho mọi thứ không được
gói [`unique`](/pkg/unique/) bao phủ, và nhiều loại
bộ nhớ đệm khác nhau.
Để hỗ trợ các trường hợp sử dụng này, bản phát hành này cũng cung cấp
[`runtime.AddCleanup`](/pkg/runtime/#AddCleanup) và
[`maphash.Comparable`](/pkg/maphash/#Comparable).

### Gói crypto/mlkem mới {#crypto-mlkem}

<!-- go.dev/issue/70122 -->

Gói [`crypto/mlkem`](/pkg/crypto/mlkem/) mới triển khai
ML-KEM-768 và ML-KEM-1024.

ML-KEM là một cơ chế trao đổi khóa hậu lượng tử trước đây được biết đến với tên Kyber và
được đặc tả trong [FIPS 203](https://doi.org/10.6028/NIST.FIPS.203).

### Các gói crypto/hkdf, crypto/pbkdf2 và crypto/sha3 mới {#crypto-packages}

<!-- go.dev/issue/61477, go.dev/issue/69488, go.dev/issue/69982, go.dev/issue/65269, CL 629176 -->

Gói [`crypto/hkdf`](/pkg/crypto/hkdf/) mới triển khai
hàm dẫn xuất khóa HKDF dựa trên HMAC Extract-and-Expand,
như được định nghĩa trong [RFC 5869](https://www.rfc-editor.org/rfc/rfc5869.html).

Gói [`crypto/pbkdf2`](/pkg/crypto/pbkdf2/) mới triển khai
hàm dẫn xuất khóa dựa trên mật khẩu PBKDF2,
như được định nghĩa trong [RFC 8018](https://www.rfc-editor.org/rfc/rfc8018.html).

Gói [`crypto/sha3`](/pkg/crypto/sha3/) mới triển khai
hàm băm SHA-3 và các hàm đầu ra có thể mở rộng SHAKE và cSHAKE,
như được định nghĩa trong [FIPS 202](http://doi.org/10.6028/NIST.FIPS.202).

Cả ba gói đều dựa trên các gói `golang.org/x/crypto/...` đã có từ trước.

### Tuân thủ FIPS 140-3 {#fips140}

Bản phát hành này bao gồm [một tập hợp cơ chế mới để hỗ trợ việc tuân thủ FIPS 140-3](/doc/security/fips140).

Go Cryptographic Module là một tập hợp các gói thư viện chuẩn nội bộ
được sử dụng một cách minh bạch để triển khai các thuật toán được FIPS 140-3 phê duyệt. Ứng dụng
không cần thay đổi gì để sử dụng Go Cryptographic Module cho các thuật toán được phê duyệt.

Biến môi trường mới `GOFIPS140` có thể được sử dụng để chọn phiên bản Go
Cryptographic Module sẽ dùng trong một bản build. Thiết lập `fips140` [GODEBUG
setting](/doc/godebug) mới có thể được sử dụng để bật chế độ FIPS 140-3 khi chạy.

Go 1.24 bao gồm Go Cryptographic Module phiên bản v1.0.0, hiện đang
được kiểm thử với một phòng thí nghiệm được CMVP công nhận.

### Gói thử nghiệm mới testing/synctest {#testing-synctest}

Gói thử nghiệm mới [`testing/synctest`](/pkg/testing/synctest/) cung cấp hỗ trợ để kiểm thử mã đồng thời.
- Hàm [`synctest.Run`](/pkg/testing/synctest/#Run) khởi động một
  nhóm goroutine trong một "bubble" cô lập.
  Bên trong bubble, các hàm của gói [`time`](/pkg/time) hoạt động trên
  một đồng hồ giả.
- Hàm [`synctest.Wait`](/pkg/testing/synctest#Wait) chờ
  tất cả goroutine trong bubble hiện tại bị chặn.

Xem tài liệu của gói để biết thêm chi tiết.

Gói `synctest` là thử nghiệm và phải được bật bằng cách
thiết lập `GOEXPERIMENT=synctest` tại thời điểm build.
API của gói có thể thay đổi trong các bản phát hành tương lai.
Xem [issue #67434](/issue/67434) để biết thêm thông tin và
cung cấp phản hồi.

### Các thay đổi nhỏ đối với thư viện {#minor_library_changes}

#### [`archive`](/pkg/archive/)

Các triển khai `(*Writer).AddFS` trong cả `archive/zip` và `archive/tar`
giờ đây ghi header thư mục cho một thư mục rỗng.

#### [`bytes`](/pkg/bytes/)

Gói [`bytes`](/pkg/bytes) bổ sung một số hàm làm việc với iterator:
- [`Lines`](/pkg/bytes#Lines) trả về một iterator duyệt qua các
  dòng kết thúc bằng ký tự xuống dòng trong một lát byte.
- [`SplitSeq`](/pkg/bytes#SplitSeq) trả về một iterator duyệt qua
  tất cả lát con của một lát byte được tách quanh một dấu phân cách.
- [`SplitAfterSeq`](/pkg/bytes#SplitAfterSeq) trả về một iterator
  duyệt qua các lát con của một lát byte được tách sau mỗi lần xuất hiện của một
  dấu phân cách.
- [`FieldsSeq`](/pkg/bytes#FieldsSeq) trả về một iterator duyệt qua
  các lát con của một lát byte được tách quanh các chuỗi ký tự khoảng trắng,
  như được định nghĩa bởi [`unicode.IsSpace`](/pkg/unicode#IsSpace).
- [`FieldsFuncSeq`](/pkg/bytes#FieldsFuncSeq) trả về một iterator
  duyệt qua các lát con của một lát byte được tách quanh các chuỗi điểm mã Unicode
  thỏa mãn một vị từ.

#### [`crypto/aes`](/pkg/crypto/aes/)

Giá trị được trả về bởi [`NewCipher`](/pkg/crypto/aes#NewCipher) không còn
triển khai các phương thức `NewCTR`, `NewGCM`, `NewCBCEncrypter`, và `NewCBCDecrypter`.
Các phương thức này không được ghi chép và không khả dụng trên mọi kiến trúc.
Thay vào đó, giá trị [`Block`](/pkg/crypto/cipher#Block) nên được truyền
trực tiếp vào các hàm [`crypto/cipher`](/pkg/crypto/cipher/) tương ứng.
Hiện tại, `crypto/cipher` vẫn kiểm tra các phương thức đó trên các giá trị `Block`,
ngay cả khi chúng không còn được thư viện chuẩn sử dụng nữa.

#### [`crypto/cipher`](/pkg/crypto/cipher/)

Hàm mới [`NewGCMWithRandomNonce`](/pkg/crypto/cipher#NewGCMWithRandomNonce) trả về một [`AEAD`](/pkg/crypto/cipher#AEAD) triển khai AES-GCM bằng cách tạo một nonce ngẫu nhiên trong quá trình Seal và thêm nó vào trước ciphertext.

Triển khai [`Stream`](/pkg/crypto/cipher#Stream) được trả về bởi [`NewCTR`](/pkg/crypto/cipher#NewCTR) khi sử dụng với [`crypto/aes`](/pkg/crypto/aes/) hiện nhanh hơn vài lần trên amd64 và arm64.

[`NewOFB`](/pkg/crypto/cipher#NewOFB),
[`NewCFBEncrypter`](/pkg/crypto/cipher#NewCFBEncrypter), và
[`NewCFBDecrypter`](/pkg/crypto/cipher#NewCFBDecrypter) hiện đã bị không khuyến nghị sử dụng.
Chế độ OFB và CFB không được xác thực, điều này nhìn chung cho phép các cuộc tấn công chủ động thao túng và khôi phục plaintext. Khuyến nghị các ứng dụng sử dụng các chế độ [`AEAD`](/pkg/crypto/cipher#AEAD) thay thế. Nếu cần chế độ [`Stream`](/pkg/crypto/cipher#Stream) không xác thực, hãy sử dụng [`NewCTR`](/pkg/crypto/cipher#NewCTR) thay thế.

#### [`crypto/ecdsa`](/pkg/crypto/ecdsa/)

<!-- go.dev/issue/64802 -->
[`PrivateKey.Sign`](/pkg/crypto/ecdsa#PrivateKey.Sign) hiện tạo chữ ký xác định theo
[RFC 6979](https://www.rfc-editor.org/rfc/rfc6979.html) nếu nguồn ngẫu nhiên là nil.

#### [`crypto/md5`](/pkg/crypto/md5/)

Giá trị được trả về bởi [`md5.New`](/pkg/md5#New) hiện cũng triển khai interface
[`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`crypto/rand`](/pkg/crypto/rand/)

<!-- go.dev/issue/66821 -->
Hàm [`Read`](/pkg/crypto/rand#Read) hiện được đảm bảo không thất bại.
Hàm này sẽ luôn trả về `nil` làm kết quả `error`.
Nếu [`Read`](/pkg/crypto/rand#Read) gặp lỗi khi đọc từ [`Reader`](/pkg/crypto/rand#Reader), chương trình sẽ bị sự cố không thể khôi phục.
Lưu ý rằng các API nền tảng được `Reader` mặc định sử dụng được ghi nhận là luôn thành công, vì vậy thay đổi này chỉ ảnh hưởng đến các chương trình ghi đè biến `Reader`. Một ngoại lệ là các kernel Linux trước phiên bản 3.17, trong đó `Reader` mặc định vẫn mở `/dev/urandom` và có thể thất bại.

<!-- go.dev/issue/69577 -->
Trên Linux 6.11 trở lên, `Reader` hiện sử dụng lời gọi hệ thống `getrandom` thông qua vDSO.
Điều này nhanh hơn vài lần, đặc biệt với các lần đọc nhỏ.

<!-- CL 608395 -->
Trên OpenBSD, `Reader` hiện sử dụng `arc4random_buf(3)`.

<!-- go.dev/issue/67057 -->
Hàm mới [`Text`](/pkg/crypto/rand#Text) có thể được sử dụng để tạo các chuỗi văn bản ngẫu nhiên an toàn về mặt mật mã.

#### [`crypto/rsa`](/pkg/crypto/rsa/)

[`GenerateKey`](/pkg/crypto/rsa#GenerateKey) hiện trả về lỗi nếu yêu cầu tạo khóa có độ dài nhỏ hơn 1024 bit.
Tất cả các phương thức Sign, Verify, Encrypt và Decrypt hiện trả về lỗi nếu được sử dụng với khóa nhỏ hơn 1024 bit. Các khóa như vậy không an toàn và không nên được sử dụng.
[Cài đặt GODEBUG](/doc/godebug) `rsa1024min=0` khôi phục hành vi cũ, nhưng chúng tôi khuyến nghị chỉ thực hiện việc này khi cần thiết và chỉ trong các bài kiểm thử, ví dụ bằng cách thêm dòng `//go:debug rsa1024min=0` vào một tệp kiểm thử.
Một [ví dụ](/pkg/crypto/rsa#example-GenerateKey-TestKey) mới về `GenerateKey` cung cấp một khóa kiểm thử chuẩn 2048 bit dễ sử dụng.

Hiện đã an toàn và hiệu quả hơn khi gọi
[`PrivateKey.Precompute`](/pkg/crypto/rsa#PrivateKey.Precompute) trước
[`PrivateKey.Validate`](/pkg/crypto/rsa#PrivateKey.Validate).
`Precompute` hiện nhanh hơn khi có các giá trị [`PrecomputedValues`](/pkg/crypto/rsa#PrecomputedValues) được điền một phần, chẳng hạn như khi giải mã hóa một khóa từ JSON.

Gói này hiện từ chối nhiều khóa không hợp lệ hơn, ngay cả khi không gọi `Validate`,
và [`GenerateKey`](/pkg/crypto/rsa#GenerateKey) có thể trả về các lỗi mới đối với nguồn ngẫu nhiên bị lỗi.
Các trường [`Primes`](/pkg/crypto/rsa#PrivateKey.Primes) và
[`Precomputed`](/pkg/crypto/rsa#PrivateKey.Precomputed) của
[`PrivateKey`](/pkg/crypto/rsa#PrivateKey) hiện được sử dụng và xác thực ngay cả khi thiếu một số giá trị.
Xem thêm các thay đổi đối với việc phân tích cú pháp và sắp xếp tuần tự khóa RSA của `crypto/x509`
[được mô tả bên dưới](#cryptox509pkgcryptox509).

<!-- go.dev/issue/43923 -->
[`SignPKCS1v15`](/pkg/crypto/rsa#SignPKCS1v15) và
[`VerifyPKCS1v15`](/pkg/crypto/rsa#VerifyPKCS1v15) hiện hỗ trợ
SHA-512/224, SHA-512/256 và SHA-3.

<!-- CL 639936 -->
[`GenerateKey`](/pkg/crypto/rsa#GenerateKey) hiện sử dụng một phương pháp hơi khác
để tạo số mũ riêng tư (hàm Carmichael thay vì hàm Euler). Các ứng dụng hiếm gặp tạo lại khóa bên ngoài chỉ từ các thừa số nguyên tố có thể tạo ra các kết quả khác nhưng tương thích.

<!-- CL 626957 -->
Các thao tác khóa công khai và riêng tư hiện nhanh hơn tới hai lần trên wasm.

#### [`crypto/sha1`](/pkg/crypto/sha1/)

Giá trị được trả về bởi [`sha1.New`](/pkg/sha1#New) hiện cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`crypto/sha256`](/pkg/crypto/sha256/)

Các giá trị được trả về bởi [`sha256.New`](/pkg/sha256#New) và
[`sha256.New224`](/pkg/sha256#New224) hiện cũng triển khai interface
[`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`crypto/sha512`](/pkg/crypto/sha512/)

Các giá trị được trả về bởi [`sha512.New`](/pkg/sha512#New),
[`sha512.New384`](/pkg/sha512#New384),
[`sha512.New512_224`](/pkg/sha512#New512_224) và
[`sha512.New512_256`](/pkg/sha512#New512_256) hiện cũng triển khai interface
[`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`crypto/subtle`](/pkg/crypto/subtle/)

Hàm mới [`WithDataIndependentTiming`](/pkg/crypto/subtle#WithDataIndependentTiming)
cho phép người dùng chạy một hàm với các tính năng cụ thể của kiến trúc được
bật, nhằm đảm bảo các lệnh cụ thể có thời gian thực thi không phụ thuộc vào giá trị dữ liệu.
Điều này có thể được dùng để đảm bảo mã được thiết kế để chạy trong thời gian hằng
không bị các tính năng ở mức CPU tối ưu hóa theo cách khiến nó hoạt động với thời
gian thay đổi.
Hiện tại, `WithDataIndependentTiming` sử dụng bit PSTATE.DIT trên arm64 và
không thực hiện thao tác nào trên tất cả kiến trúc khác. [Thiết lập GODEBUG](/doc/godebug)
`dataindependenttiming=1` bật chế độ DIT cho toàn bộ chương trình Go.

<!-- CL 622276 -->
Đầu ra của [`XORBytes`](/pkg/crypto/subtle#XORBytes) phải chồng lấp chính xác
hoặc hoàn toàn không chồng lấp với các đầu vào. Trước đây, hành vi này không được
xác định, còn hiện tại `XORBytes` sẽ panic.

#### [`crypto/tls`](/pkg/crypto/tls/)

Máy chủ TLS hiện hỗ trợ Encrypted Client Hello (ECH). Tính năng này có thể được
bật bằng cách điền trường [`Config.EncryptedClientHelloKeys`](/pkg/crypto/tls#Config.EncryptedClientHelloKeys).

Cơ chế trao đổi khóa hậu lượng tử mới [`X25519MLKEM768`](/pkg/crypto/tls#X25519MLKEM768)
hiện được hỗ trợ và được bật theo mặc định khi
[`Config.CurvePreferences`](/pkg/crypto/tls#Config.CurvePreferences) là nil.
[Thiết lập GODEBUG](/doc/godebug) `tlsmlkem=0` khôi phục mặc định.
Điều này có thể hữu ích khi xử lý các máy chủ TLS bị lỗi không xử lý đúng các bản ghi lớn,
gây ra hết thời gian chờ trong quá trình bắt tay (xem [TLS post-quantum TL;DR fail](https://tldr.fail/)).

Hỗ trợ cho cơ chế trao đổi khóa thử nghiệm `X25519Kyber768Draft00` đã bị xóa.

<!-- go.dev/issue/69393, CL 630775 -->
Thứ tự trao đổi khóa hiện được xử lý hoàn toàn bởi gói `crypto/tls`. Thứ tự của
[`Config.CurvePreferences`](/pkg/crypto/tls#Config.CurvePreferences) hiện bị bỏ qua,
và nội dung của trường này chỉ được dùng để xác định những cơ chế trao đổi khóa nào
sẽ được bật khi trường được điền.

<!-- go.dev/issue/32936 -->
Trường mới [`ClientHelloInfo.Extensions`](/pkg/crypto/tls#ClientHelloInfo.Extensions)
liệt kê ID của các phần mở rộng nhận được trong thông báo Client Hello.
Điều này có thể hữu ích cho việc nhận dạng dấu vân tay của máy khách TLS.

<!-- go.dev/issue/72111, documented retro-actively after Go 1.27 release -->

Thiết lập `tlskyber` `GODEBUG` (được thêm trong [Go 1.23](/doc/godebug#go-123))
đã bị xóa.

#### [`crypto/x509`](/pkg/crypto/x509/)

<!-- go.dev/issue/41682 -->
Cài đặt `x509sha1` trong [GODEBUG setting](/doc/godebug) đã bị xóa.
[`Certificate.Verify`](/pkg/crypto/x509#Certificate.Verify) không còn hỗ trợ
chữ ký dựa trên SHA-1.

[`OID`](/pkg/crypto/x509#OID) hiện triển khai các interface
[`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender) và
[`encoding.TextAppender`](/pkg/encoding#TextAppender).

Trường chính sách chứng chỉ mặc định đã thay đổi từ
[`Certificate.PolicyIdentifiers`](/pkg/crypto/x509#Certificate.PolicyIdentifiers)
thành [`Certificate.Policies`](/pkg/crypto/x509#Certificate.Policies). Khi phân tích
chứng chỉ, cả hai trường sẽ được điền, nhưng khi tạo chứng chỉ, các chính sách
giờ đây sẽ được lấy từ trường `Certificate.Policies` thay vì trường
`Certificate.PolicyIdentifiers`. Thay đổi này có thể được hoàn nguyên bằng
[GODEBUG setting](/doc/godebug) `x509usepolicies=0`.

<!-- go.dev/issue/67675 -->
[`CreateCertificate`](/pkg/crypto/x509#CreateCertificate) giờ đây sẽ tạo số sê-ri
bằng phương thức tuân thủ RFC 5280 khi được truyền một mẫu có trường
[`Certificate.SerialNumber`](/pkg/crypto/x509#Certificate.SerialNumber) là nil,
thay vì thất bại.

[`Certificate.Verify`](/pkg/crypto/x509#Certificate.Verify) hiện hỗ trợ xác thực
chính sách, như được định nghĩa trong RFC 5280 và RFC 9618. Trường mới
[`VerifyOptions.CertificatePolicies`](/pkg/crypto/x509#VerifyOptions.CertificatePolicies)
có thể được đặt thành tập hợp chính sách [`OIDs`](/pkg/crypto/x509#OID) được chấp
nhận. Chỉ các chuỗi chứng chỉ có đồ thị chính sách hợp lệ mới được trả về từ
[`Certificate.Verify`](/pkg/crypto/x509#Certificate.Verify).

[`MarshalPKCS8PrivateKey`](/pkg/crypto/x509#MarshalPKCS8PrivateKey) giờ đây trả về
lỗi thay vì marshal một khóa RSA không hợp lệ.
([`MarshalPKCS1PrivateKey`](/pkg/crypto/x509#MarshalPKCS1PrivateKey) không
có giá trị trả về lỗi, và hành vi của nó khi được cung cấp các khóa không hợp lệ
tiếp tục là không xác định.)

[`ParsePKCS1PrivateKey`](/pkg/crypto/x509#ParsePKCS1PrivateKey) và
[`ParsePKCS8PrivateKey`](/pkg/crypto/x509#ParsePKCS8PrivateKey) hiện sử dụng và
xác thực các giá trị CRT đã mã hóa, do đó có thể từ chối các khóa RSA không hợp lệ
trước đây đã được chấp nhận. Sử dụng [GODEBUG setting](/doc/godebug)
`x509rsacrt=0` để hoàn nguyên về việc tính toán lại các giá trị CRT.

#### [`debug/elf`](/pkg/debug/elf/)

<!-- go.dev/issue/63952 -->

Gói [`debug/elf`](/pkg/debug/elf) bổ sung hỗ trợ xử lý các phiên bản ký hiệu trong các tệp ELF (Executable and Linkable Format) động.
Phương thức mới [`File.DynamicVersions`](/pkg/debug/elf#File.DynamicVersions) trả về danh sách các phiên bản động được định nghĩa trong tệp ELF.
Phương thức mới [`File.DynamicVersionNeeds`](/pkg/debug/elf#File.DynamicVersionNeeds) trả về danh sách các phiên bản động mà tệp ELF này yêu cầu, được định nghĩa trong các đối tượng ELF khác.
Cuối cùng, các trường mới [`Symbol.HasVersion`](/pkg/debug/elf#Symbol) và [`Symbol.VersionIndex`](/pkg/debug/elf#Symbol) cho biết phiên bản của một ký hiệu.

#### [`encoding`](/pkg/encoding/)

Hai interface mới, [`TextAppender`](/pkg/encoding#TextAppender) và [`BinaryAppender`](/pkg/encoding#BinaryAppender), đã được giới thiệu để thêm biểu diễn dạng văn bản hoặc nhị phân của một đối tượng vào một byte slice. Các interface này cung cấp cùng chức năng như [`TextMarshaler`](/pkg/encoding#TextMarshaler) và [`BinaryMarshaler`](/pkg/encoding#BinaryMarshaler), nhưng thay vì cấp phát một slice mới mỗi lần, chúng thêm dữ liệu trực tiếp vào một slice hiện có.
Các interface này hiện được triển khai bởi các kiểu trong thư viện chuẩn vốn đã triển khai `TextMarshaler` và/hoặc `BinaryMarshaler`.

#### [`encoding/json`](/pkg/encoding/json/)

<!-- go.dev/issue/45669 -->
Khi marshal, một trường struct có tùy chọn `omitzero` mới trong tag của trường struct sẽ bị bỏ qua nếu giá trị của nó là zero. Nếu kiểu của trường có phương thức `IsZero() bool`, phương thức đó sẽ được dùng để xác định liệu giá trị có phải là zero hay không. Nếu không, giá trị là zero nếu nó là [giá trị zero cho kiểu của nó](/ref/spec#The_zero_value).
Tag trường `omitzero` rõ ràng hơn và ít dễ gây lỗi hơn `omitempty` khi mục đích là bỏ qua các giá trị zero.
Cụ thể, không giống `omitempty`, `omitzero` bỏ qua các giá trị zero của [`time.Time`](/pkg/time#Time), vốn là một nguồn gây khó khăn phổ biến.

Nếu cả `omitempty` và `omitzero` được chỉ định, trường sẽ bị bỏ qua nếu giá trị là rỗng hoặc zero (hoặc cả hai).

[`UnmarshalTypeError.Field`](/pkg/encoding/json#UnmarshalTypeError.Field) hiện bao gồm các struct nhúng để cung cấp thông báo lỗi chi tiết hơn.

#### [`go/types`](/pkg/go/types/)

Tất cả cấu trúc dữ liệu `go/types` cung cấp các chuỗi thông qua một cặp phương thức như `Len() int` và `At(int) T` giờ đây cũng có các phương thức trả về iterator, cho phép bạn đơn giản hóa mã như sau:

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

Giá trị được trả về bởi [`New`](/pkg/hash/adler32#New) giờ đây cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`hash/crc32`](/pkg/hash/crc32/)

Các giá trị được trả về bởi [`New`](/pkg/hash/crc32#New) và [`NewIEEE`](/pkg/hash/crc32#NewIEEE) giờ đây cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`hash/crc64`](/pkg/hash/crc64/)

Giá trị được trả về bởi [`New`](/pkg/hash/crc64#New) giờ đây cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`hash/fnv`](/pkg/hash/fnv/)

Các giá trị được trả về bởi [`New32`](/pkg/hash/fnv#New32), [`New32a`](/pkg/hash/fnv#New32a), [`New64`](/pkg/hash/fnv#New64), [`New64a`](/pkg/hash/fnv#New64a), [`New128`](/pkg/hash/fnv#New128) và [`New128a`](/pkg/hash/fnv#New128a) giờ đây cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`hash/maphash`](/pkg/hash/maphash/)

Các hàm mới [`Comparable`](/pkg/hash/maphash#Comparable) và
[`WriteComparable`](/pkg/hash/maphash#WriteComparable) có thể tính toán
giá trị băm của bất kỳ giá trị có thể so sánh nào.
Các hàm này cho phép băm mọi thứ có thể được dùng làm khóa của Go map.

#### [`log/slog`](/pkg/log/slog/)

[`DiscardHandler`](/pkg/log/slog#DiscardHandler) mới là một handler không bao giờ được bật và luôn loại bỏ đầu ra của nó.

[`Level`](/pkg/log/slog#Level) và [`LevelVar`](/pkg/log/slog#LevelVar) hiện triển khai interface [`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`math/big`](/pkg/math/big/)

[`Float`](/pkg/math/big#Float), [`Int`](/pkg/math/big#Int) và [`Rat`](/pkg/math/big#Rat) hiện triển khai interface [`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`math/rand`](/pkg/math/rand/)

Các lệnh gọi đến hàm cấp cao nhất [`Seed`](/pkg/math/rand#Seed) đã lỗi thời không còn có tác dụng. Để
khôi phục hành vi cũ, hãy dùng [`cài đặt GODEBUG`](/doc/godebug) `randseednop=0`. Để biết thêm thông tin nền, xem
[đề xuất #67273](/issue/67273).

#### [`math/rand/v2`](/pkg/math/rand/v2/)

[`ChaCha8`](/pkg/math/rand/v2#ChaCha8) và [`PCG`](/pkg/math/rand/v2#PCG) hiện triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`net`](/pkg/net/)

[`ListenConfig`](/pkg/net#ListenConfig) hiện sử dụng MPTCP theo mặc định trên các hệ thống được hỗ trợ
(hiện chỉ trên Linux).

[`IP`](/pkg/net#IP) hiện triển khai interface [`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`net/http`](/pkg/net/http/)

Giới hạn của [`Transport`](/pkg/net/http#Transport) đối với các phản hồi thông tin 1xx nhận được
khi phản hồi một yêu cầu đã thay đổi.
Trước đây, nó hủy một yêu cầu và trả về lỗi sau khi
nhận được hơn 5 phản hồi 1xx.
Hiện tại, nó trả về lỗi nếu tổng kích thước của tất cả phản hồi 1xx
vượt quá thiết lập cấu hình [`Transport.MaxResponseHeaderBytes`](/pkg/net/http#Transport.MaxResponseHeaderBytes).

Ngoài ra, khi một yêu cầu có hook theo dõi
[`net/http/httptrace.ClientTrace.Got1xxResponse`](/pkg/net/http/httptrace#ClientTrace.Got1xxResponse),
hiện không còn giới hạn về tổng số phản hồi 1xx.
Hook `Got1xxResponse` có thể trả về lỗi để hủy một yêu cầu.

[`Transport`](/pkg/net/http#Transport) và [`Server`](/pkg/net/http#Server) hiện có trường HTTP2 cho phép
cấu hình các thiết lập giao thức HTTP/2.

Các trường mới [`Server.Protocols`](/pkg/net/http#Server.Protocols) và [`Transport.Protocols`](/pkg/net/http#Transport.Protocols) cung cấp
một cách đơn giản để cấu hình các giao thức HTTP mà máy chủ hoặc client sử dụng.

Máy chủ và client có thể được cấu hình để hỗ trợ các kết nối HTTP/2
không mã hóa.

Khi [`Server.Protocols`](/pkg/net/http#Server.Protocols) chứa UnencryptedHTTP2, máy chủ sẽ chấp nhận
các kết nối HTTP/2 trên các cổng không mã hóa. Máy chủ có thể chấp nhận cả
HTTP/1 và HTTP/2 không mã hóa trên cùng một cổng.

Khi [`Transport.Protocols`](/pkg/net/http#Transport.Protocols) chứa UnencryptedHTTP2 và không chứa
HTTP1, transport sẽ sử dụng HTTP/2 không mã hóa cho các URL http://.
Nếu transport được cấu hình để sử dụng cả HTTP/1 và HTTP/2 không mã hóa,
nó sẽ sử dụng HTTP/1.

Hỗ trợ HTTP/2 không mã hóa sử dụng "HTTP/2 with Prior Knowledge"
(RFC 9113, mục 3.3). Header "Upgrade: h2c" đã lỗi thời
không được hỗ trợ.

#### [`net/netip`](/pkg/net/netip/)

[`Addr`](/pkg/net/netip#Addr), [`AddrPort`](/pkg/net/netip#AddrPort) và [`Prefix`](/pkg/net/netip#Prefix) hiện triển khai các interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender) và
[`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`net/url`](/pkg/net/url/)

[`URL`](/pkg/net/url#URL) hiện cũng triển khai interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender).

#### [`os/user`](/pkg/os/user/)

Trên Windows, [`Current`](/pkg/os/user#Current) hiện có thể được sử dụng trong Windows Nano Server.
Triển khai này đã được cập nhật để tránh sử dụng các hàm
từ thư viện `NetApi32`, thư viện này không có trong Nano Server.

Trên Windows, [`Current`](/pkg/os/user#Current), [`Lookup`](/pkg/os/user#Lookup) và [`LookupId`](/pkg/os/user#LookupId) hiện hỗ trợ các
tài khoản người dùng dịch vụ tích hợp sẵn sau:
- `NT AUTHORITY\SYSTEM`
- `NT AUTHORITY\LOCAL SERVICE`
- `NT AUTHORITY\NETWORK SERVICE`

Trên Windows, [`Current`](/pkg/os/user#Current) đã được làm nhanh hơn đáng kể khi
người dùng hiện tại được tham gia vào một domain chậm, đây là
trường hợp thường gặp đối với nhiều người dùng doanh nghiệp. Hiệu năng
của triển khai mới hiện ở mức mili giây, so với
triển khai trước đây có thể mất vài giây,
hoặc thậm chí vài phút, để hoàn tất.

Trên Windows, [`Current`](/pkg/os/user#Current) hiện trả về người dùng sở hữu tiến trình khi
luồng hiện tại đang mạo danh một người dùng khác. Trước đây,
nó trả về lỗi.

#### [`regexp`](/pkg/regexp/)

[`Regexp`](/pkg/regexp#Regexp) hiện triển khai interface [`encoding.TextAppender`](/pkg/encoding#TextAppender).

#### [`runtime`](/pkg/runtime/)

Hàm [`GOROOT`](/pkg/runtime#GOROOT) hiện đã bị loại bỏ khuyến nghị sử dụng.
Trong mã mới, ưu tiên sử dụng đường dẫn hệ thống để định vị binary “go”,
và sử dụng `go env GOROOT` để tìm GOROOT của nó.

#### [`strings`](/pkg/strings/)

Gói [`strings`](/pkg/strings) bổ sung một số hàm hoạt động với iterator:
- [`Lines`](/pkg/strings#Lines) trả về một iterator duyệt qua
  các dòng kết thúc bằng ký tự xuống dòng trong một chuỗi.
- [`SplitSeq`](/pkg/strings#SplitSeq) trả về một iterator duyệt qua
  tất cả chuỗi con của một chuỗi được tách quanh một dấu phân cách.
- [`SplitAfterSeq`](/pkg/strings#SplitAfterSeq) trả về một iterator
  duyệt qua các chuỗi con của một chuỗi được tách sau mỗi lần xuất hiện của
  một dấu phân cách.
- [`FieldsSeq`](/pkg/strings#FieldsSeq) trả về một iterator duyệt qua
  các chuỗi con của một chuỗi được tách quanh các dãy ký tự khoảng trắng,
  như được định nghĩa bởi [`unicode.IsSpace`](/pkg/unicode#IsSpace).
- [`FieldsFuncSeq`](/pkg/strings#FieldsFuncSeq) trả về một iterator
  duyệt qua các chuỗi con của một chuỗi được tách quanh các dãy điểm mã Unicode
  thỏa mãn một vị từ.

#### [`sync`](/pkg/sync/)

Việc triển khai [`sync.Map`](/pkg/sync#Map) đã được thay đổi, cải thiện hiệu năng,
đặc biệt đối với các thao tác sửa đổi map.
Ví dụ, các thao tác sửa đổi trên những tập khóa rời nhau ít có khả năng tranh chấp hơn nhiều trên
các map lớn hơn, và không còn cần thời gian tăng dần để đạt được các lần tải có mức tranh chấp thấp
từ map.

Nếu bạn gặp bất kỳ vấn đề nào, hãy đặt `GOEXPERIMENT=nosynchashtriemap` tại thời điểm build
để chuyển lại sang triển khai cũ và vui lòng [gửi
issue](/issue/new).

#### [`testing`](/pkg/testing/)

Các phương thức mới [`T.Context`](/pkg/testing#T.Context) và [`B.Context`](/pkg/testing#B.Context) trả về một context bị hủy
sau khi bài kiểm thử hoàn tất và trước khi các hàm dọn dẹp bài kiểm thử chạy.

<!-- testing.B.Loop mentioned in 6-stdlib/6-testing-bloop.md. -->

Các phương thức mới [`T.Chdir`](/pkg/testing#T.Chdir) và [`B.Chdir`](/pkg/testing#B.Chdir) có thể được dùng để thay đổi thư mục
làm việc trong khoảng thời gian thực hiện một bài kiểm thử hoặc benchmark.

#### [`text/template`](/pkg/text/template/)

Các template hiện hỗ trợ range-over-func và range-over-int.

#### [`time`](/pkg/time/)

[`Time`](/pkg/time#Time) hiện triển khai các interface [`encoding.BinaryAppender`](/pkg/encoding#BinaryAppender) và [`encoding.TextAppender`](/pkg/encoding#TextAppender).

## Các cổng {#ports}

### Linux {#linux}

<!-- go.dev/issue/67001 -->
Như đã [được công bố](go1.23#linux) trong ghi chú bản phát hành Go 1.23, Go 1.24 yêu cầu phiên bản
kernel Linux 3.2 trở lên.

### Darwin {#darwin}

<!-- go.dev/issue/69839 -->
Go 1.24 là bản phát hành cuối cùng chạy trên macOS 11 Big Sur.
Go 1.25 sẽ yêu cầu macOS 12 Monterey trở lên.

### WebAssembly {#wasm}

<!-- go.dev/issue/65199, CL 603055 -->
Chỉ thị trình biên dịch `go:wasmexport` được thêm vào để các chương trình Go xuất các hàm
sang máy chủ WebAssembly.

Trên WebAssembly System Interface Preview 1 (`GOOS=wasip1 GOARCH=wasm`), Go 1.24 hỗ trợ
xây dựng một chương trình Go dưới dạng
[reactor/library](https://github.com/WebAssembly/WASI/blob/63a46f61052a21bfab75a76558485cf097c0dbba/legacy/application-abi.md#current-unstable-abi),
bằng cách chỉ định cờ build `-buildmode=c-shared`.

<!-- go.dev/issue/66984, CL 626615 -->
Hiện có thêm nhiều kiểu được phép làm kiểu đối số hoặc kiểu kết quả cho các hàm `go:wasmimport`.
Cụ thể, `bool`, `string`, `uintptr` và con trỏ đến một số kiểu nhất định được cho phép
(xem [tài liệu](/pkg/cmd/compile#hdr-WebAssembly_Directives) để biết chi tiết),
cùng với các kiểu số nguyên và số thực 32-bit và 64-bit, và `unsafe.Pointer`, vốn
đã được cho phép.
Các kiểu này cũng được phép làm kiểu đối số hoặc kiểu kết quả cho các hàm `go:wasmexport`.

<!-- go.dev/issue/68024 -->
Các tệp hỗ trợ cho WebAssembly đã được chuyển từ `misc/wasm` sang `lib/wasm`.

<!-- CL 621635, CL 621636 -->
Kích thước bộ nhớ ban đầu được giảm đáng kể, đặc biệt đối với các ứng dụng WebAssembly nhỏ.

### Windows {#windows}

<!-- go.dev/issue/70705 -->
Cổng windows/arm 32-bit (`GOOS=windows GOARCH=arm`) đã được đánh dấu là bị hỏng.
Xem [vấn đề #70705](/issue/70705) để biết chi tiết.

<!-- Các mục không cần được đề cập trong ghi chú bản phát hành Go 1.24 nhưng được relnote todo thu thập. -->

accepted proposal [https://go.dev/issue/25309](https://go.dev/issue/25309) (từ [https://go.dev/cl/594018](https://go.dev/cl/594018), [https://go.dev/cl/595120](https://go.dev/cl/595120), [https://go.dev/cl/595564](https://go.dev/cl/595564), [https://go.dev/cl/601778](https://go.dev/cl/601778)) - gói x/crypto mới; dường như không cần đề cập
accepted proposal [https://go.dev/issue/43744](https://go.dev/issue/43744) (từ [https://go.dev/cl/357530](https://go.dev/cl/357530)) - không có thay đổi trong thư viện chuẩn
accepted proposal [https://go.dev/issue/60905](https://go.dev/issue/60905) (từ [https://go.dev/cl/610195](https://go.dev/cl/610195)) - CL 610195 đã được hoàn nguyên
accepted proposal [https://go.dev/issue/61395](https://go.dev/issue/61395) (từ [https://go.dev/cl/594738](https://go.dev/cl/594738), [https://go.dev/cl/594976](https://go.dev/cl/594976)) - CL 594738 đã làm cho các phép toán AND/OR của sync/atomic trở thành intrinsic trên amd64, nhưng API đã được thêm vào trong Go 1.23; CL 594976 là bản sửa lỗi; có lẽ không cần ghi chú bản phát hành Go 1.24 (chỉ thay đổi hiệu năng)
accepted proposal [https://go.dev/issue/51269](https://go.dev/issue/51269) (từ [https://go.dev/cl/627035](https://go.dev/cl/627035)) - có thể đáng đề cập trong ghi chú bản phát hành Go 1.24, hoặc cũng có thể bỏ qua; đã nhận xét tại [https://go.dev/issue/51269#issuecomment-2501802763](https://go.dev/issue/51269#issuecomment-2501802763); Ian xác nhận có thể bỏ qua
accepted proposal [https://go.dev/issue/66540](https://go.dev/issue/66540) (từ [https://go.dev/cl/603958](https://go.dev/cl/603958)) - làm rõ đặc tả ngôn ngữ Go; có thể không cần đề cập trong ghi chú bản phát hành Go 1.24; đã để lại nhận xét tại [https://go.dev/issue/66540#issuecomment-2502051684](https://go.dev/issue/66540#issuecomment-2502051684); Robert xác nhận đúng là không cần
accepted proposal [https://go.dev/issue/34208](https://go.dev/issue/34208) (từ [https://go.dev/cl/586241](https://go.dev/cl/586241)) - CL 586241 triển khai bản sửa lỗi cho một tính năng Go 1.23, dường như không cần gì trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/43993](https://go.dev/issue/43993) (từ [https://go.dev/cl/626116](https://go.dev/cl/626116)) - CL 626116 chuẩn bị cây mã nguồn cho thay đổi của vet nhưng bản thân thay đổi của vet chưa được triển khai trong Go 1.24, vì vậy không có gì để nói trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/44505](https://go.dev/issue/44505) (từ [https://go.dev/cl/609955](https://go.dev/cl/609955)) - CL 609955 là dọn dẹp nội bộ trong x/tools, không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/61476](https://go.dev/issue/61476) (từ [https://go.dev/cl/608255](https://go.dev/cl/608255)) - CL 608255 xây dựng dựa trên GORISCV64 được thêm trong Go 1.23; không có gì cần đề cập trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/66315](https://go.dev/issue/66315) (từ [https://go.dev/cl/577996](https://go.dev/cl/577996)) - thêm trường Pass.Module vào x/tools/go/analysis dường như không phải điều cần được đề cập trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/57786](https://go.dev/issue/57786) (từ [https://go.dev/cl/472717](https://go.dev/cl/472717)) - CL 472717 nằm trong x/net/http2 và đề cập một đề xuất Go 1.21; dường như không cần gì trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/54265](https://go.dev/issue/54265) (từ [https://go.dev/cl/609915](https://go.dev/cl/609915), [https://go.dev/cl/610675](https://go.dev/cl/610675)) - các CL tham chiếu đến một đề xuất Go 1.22, không cần thêm gì trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/53021](https://go.dev/issue/53021) (từ [https://go.dev/cl/622276](https://go.dev/cl/622276)) - CL 622276 cải thiện tài liệu; đề xuất 53021 thuộc Go 1.20 nên không cần thêm gì trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/51430](https://go.dev/issue/51430) (từ [https://go.dev/cl/613375](https://go.dev/cl/613375)) - CL 613375 là nhận xét tài liệu nội bộ; đề xuất 51430 xảy ra trong Go 1.20/1.21 nên không cần thêm gì trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/38445](https://go.dev/issue/38445) (từ [https://go.dev/cl/626495](https://go.dev/cl/626495)) - CL 626495 làm việc trên đề xuất 38445, liên quan đến x/tools/go/package, không cần gì trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/56986](https://go.dev/issue/56986) (từ [https://go.dev/cl/618115](https://go.dev/cl/618115)) - CL 618115 thêm tài liệu; không cần đề cập trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/60061](https://go.dev/issue/60061) (từ [https://go.dev/cl/612038](https://go.dev/cl/612038)) - CL 612038 là một CL loại bỏ tính năng khỏi x/tools/go/ast và đề cập một đề xuất Go 1.22; không cần gì trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/61324](https://go.dev/issue/61324) (từ [https://go.dev/cl/411907](https://go.dev/cl/411907)) - CL 411907 là một CL x/tools triển khai đề xuất cho một gói mới ở đó; không cần gì trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/61777](https://go.dev/issue/61777) (từ [https://go.dev/cl/601496](https://go.dev/cl/601496)) - CL 601496 thêm trường WriteByteTimeout vào x/net/http2.Server; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/61940](https://go.dev/issue/61940) (từ [https://go.dev/cl/600997](https://go.dev/cl/600997)) - CL 600997 xóa mã lỗi thời trong x/build và đề cập một đề xuất đã được chấp nhận; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/62113](https://go.dev/issue/62113) (từ [https://go.dev/cl/594195](https://go.dev/cl/594195)) - CL 594195 thêm các bổ sung liên quan đến iterator trong x/net/html; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/62484](https://go.dev/issue/62484) (từ [https://go.dev/cl/600775](https://go.dev/cl/600775)) - CL 600775 ghi lại hành vi symlink của CopyFS và đề cập đề xuất Go 1.23; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/64207](https://go.dev/issue/64207) (từ [https://go.dev/cl/605875](https://go.dev/cl/605875)) - một CL x/website tiếp nối một đề xuất Go 1.23; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/65236](https://go.dev/issue/65236) (từ [https://go.dev/cl/596135](https://go.dev/cl/596135)) - CL 596135 thêm các bài kiểm thử cho đề xuất Go 1.23 65236; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/67795](https://go.dev/issue/67795) (từ [https://go.dev/cl/616218](https://go.dev/cl/616218)) - hỗ trợ iterator cho x/tools/go/ast/inspector; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/67812](https://go.dev/issue/67812) (từ [https://go.dev/cl/601497](https://go.dev/cl/601497)) - ping máy chủ có thể cấu hình cho x/net/http2.Server; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/68232](https://go.dev/issue/68232) (từ [https://go.dev/cl/595676](https://go.dev/cl/595676)) - các bổ sung cho x/sys/unix; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/68898](https://go.dev/issue/68898) (từ [https://go.dev/cl/607495](https://go.dev/cl/607495), [https://go.dev/cl/620036](https://go.dev/cl/620036), [https://go.dev/cl/620135](https://go.dev/cl/620135), [https://go.dev/cl/623638](https://go.dev/cl/623638)) - một đề xuất cho x/tools/go/gcexportdata để ghi lại chính sách hỗ trợ 2 bản phát hành + tip; vì thay đổi nằm trong x/tools nên không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/69095](https://go.dev/issue/69095) (từ [https://go.dev/cl/593683](https://go.dev/cl/593683), [https://go.dev/cl/608955](https://go.dev/cl/608955), [https://go.dev/cl/610716](https://go.dev/cl/610716)) - một đề xuất ảnh hưởng đến việc bảo trì và hỗ trợ các repository golang.org/x; không cần được đề cập trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/68384](https://go.dev/issue/68384) (từ [https://go.dev/cl/611875](https://go.dev/cl/611875)) - mở rộng phạm vi Go Telemetry để bao gồm Delve không liên quan trực tiếp đến Go 1.24 và dường như không cần được đề cập trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/69291](https://go.dev/issue/69291) (từ [https://go.dev/cl/610939](https://go.dev/cl/610939)) - CL 610939 tái cấu trúc mã trong x/tools và đề cập đề xuất #69291 vẫn đang mở để thêm Reachable vào x/tools/go/ssa/ssautil; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/69360](https://go.dev/issue/69360) (từ [https://go.dev/cl/614158](https://go.dev/cl/614158), [https://go.dev/cl/614159](https://go.dev/cl/614159), [https://go.dev/cl/614635](https://go.dev/cl/614635), [https://go.dev/cl/614675](https://go.dev/cl/614675)) - đề xuất 69360 là gắn thẻ và xóa gorename khỏi x/tools; không cần ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/61417](https://go.dev/issue/61417) (từ [https://go.dev/cl/605955](https://go.dev/cl/605955)) - một trường mới trong x/oauth2; không có gì cần đề cập trong ghi chú bản phát hành Go 1.24
accepted proposal [https://go.dev/issue/29266](https://go.dev/issue/29266) (từ [https://go.dev/cl/632897](https://go.dev/cl/632897)) - đề xuất chỉ liên quan đến tài liệu cho go.dev/doc/contribute; không cần ghi chú bản phát hành Go 1.24
-->
