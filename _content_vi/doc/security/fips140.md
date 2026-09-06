---
title: Tuân thủ FIPS 140-3
layout: article
---

Bắt đầu từ Go 1.24, các binary Go có thể hoạt động nguyên bản ở chế độ hỗ trợ việc tuân thủ FIPS 140-3. Hơn nữa, toolchain có thể xây dựng dựa trên các phiên bản đã được đóng băng của những gói mật mã tạo thành Mô-đun Mật mã Go.

## FIPS 140-3

NIST FIPS 140-3 là một chế độ tuân thủ của Chính phủ Hoa Kỳ dành cho các ứng dụng mật mã, trong đó ngoài các yêu cầu khác còn yêu cầu sử dụng một tập hợp thuật toán được phê duyệt và sử dụng các mô-đun mật mã được
[CMVP](https://csrc.nist.gov/projects/cryptographic-module-validation-program)-validated
kiểm định bởi CMVP đã được kiểm thử trong các môi trường vận hành mục tiêu.

Các cơ chế được mô tả trong trang này hỗ trợ việc tuân thủ cho các ứng dụng Go.

Các ứng dụng không cần tuân thủ FIPS 140-3 có thể bỏ qua chúng một cách an toàn và không nên bật chế độ FIPS 140-3.

**LƯU Ý:** Chỉ sử dụng một mô-đun mật mã tuân thủ và đã được kiểm định theo FIPS 140-3 có thể không—tự nó—đáp ứng tất cả các yêu cầu pháp lý liên quan. Đội ngũ Go không thể cung cấp bất kỳ đảm bảo hoặc hỗ trợ nào về việc cách sử dụng chế độ FIPS 140-3 được cung cấp có thể hoặc không thể đáp ứng các yêu cầu pháp lý cụ thể cho từng người dùng. Cần cẩn thận khi xác định liệu việc sử dụng mô-đun này có đáp ứng các yêu cầu cụ thể của bạn hay không.

## Mô-đun Mật mã Go

Mô-đun Mật mã Go là một tập hợp các gói Go trong thư viện chuẩn dưới `crypto/internal/fips140/...` triển khai các thuật toán được phê duyệt theo FIPS 140-3.

Các gói API công khai như `crypto/ecdsa` và `crypto/rand` sử dụng Mô-đun Mật mã Go một cách minh bạch để triển khai các thuật toán FIPS 140-3.

## Chế độ FIPS 140-3

Khi hoạt động ở chế độ FIPS 140-3:

 - Mô-đun Mật mã Go tự động thực hiện kiểm tra toàn vẹn tại thời điểm `init`, bằng cách so sánh checksum của tệp đối tượng của mô-đun được tính toán tại thời điểm build với các symbol được nạp vào bộ nhớ.

 - Tất cả thuật toán thực hiện các bài tự kiểm tra với đáp án đã biết theo Hướng dẫn Triển khai FIPS 140-3 liên quan, tại thời điểm `init` hoặc trong lần sử dụng đầu tiên.

 - Các bài kiểm tra tính nhất quán theo cặp được thực hiện trên các khóa mật mã được tạo ra. Lưu ý rằng điều này có thể khiến một số loại khóa chậm hơn tới 2 lần, đặc biệt đáng chú ý đối với các khóa tạm thời.

 - [`crypto/rand.Reader`](/pkg/crypto/rand/#Reader) được triển khai dựa trên một DRBG NIST SP 800-90A. Để đảm bảo cùng mức độ bảo mật như các chương trình không chạy ở chế độ FIPS 140-3, các byte ngẫu nhiên cũng được lấy từ CSPRNG của nền tảng ở mỗi lần `Read` và được trộn vào đầu ra dưới dạng dữ liệu bổ sung chưa được ghi nhận.

 - Gói [`crypto/tls`](/pkg/crypto/tls/) sẽ bỏ qua và không thương lượng bất kỳ phiên bản giao thức, bộ mã hóa, thuật toán chữ ký hoặc cơ chế trao đổi khóa nào không được FIPS 140-3 phê duyệt. (Điều này tương đương với cơ chế Go+BoringCrypto `crypto/tls/fipsonly` chọn tham gia kiểu cũ.)

 - [`crypto/rsa.SignPSS`](/pkg/crypto/rsa/#SignPSS) với [`PSSSaltLengthAuto`](/pkg/crypto/rsa/#PSSSaltLengthAuto) sẽ giới hạn độ dài của salt ở độ dài của hàm băm.

Chế độ FIPS 140-3 không được hỗ trợ trên OpenBSD, Wasm, AIX và Windows 32-bit.

## Gói `crypto/fips140`

Hàm [`crypto/fips140.Enabled`](/pkg/crypto/fips140/#Enabled) cho biết liệu chế độ FIPS 140-3 có đang hoạt động hay không.

Hàm [`crypto/fips140.Version`](/pkg/crypto/fips140/#Version) trả về phiên bản của Mô-đun Mật mã Go đang được sử dụng.

## Biến môi trường `GOFIPS140`

Biến môi trường `GOFIPS140` có thể được sử dụng với `go build`, `go install`, và `go test` để chọn phiên bản của Mô-đun Mật mã Go sẽ được liên kết vào chương trình thực thi, đồng thời bật chế độ FIPS 140-3 theo mặc định.

- `off` là mặc định và sử dụng các gói `crypto/internal/fips140/...` trong cây thư viện chuẩn đang được sử dụng.

- `latest` giống như `off`, nhưng bật chế độ FIPS 140-3 theo mặc định.

- `v1.0.0` hoặc `v1.26.0` chọn các phiên bản Mô-đun Mật mã Go tương ứng cụ thể. Chúng bật chế độ FIPS 140-3 theo mặc định.

- `inprocess` và `certified` tương đương với việc chỉ định phiên bản mới nhất đã đạt đến [CMVP Modules In Process List][] và phiên bản mới nhất đã nhận được [CMVP validation certificate][], tương ứng.

[CMVP Modules In Process List]: https://csrc.nist.gov/Projects/cryptographic-module-validation-program/modules-in-process/modules-in-process-list
[CMVP validation certificate]: https://csrc.nist.gov/projects/cryptographic-module-validation-program/validated-modules/search?SearchMode=Basic&ModuleName=Go+Cryptographic+Module&CertificateStatus=Active&ValidationYear=0

## Tùy chọn `fips140` của GODEBUG

Tùy chọn [GODEBUG](/doc/godebug) `fips140` trong thời gian chạy điều khiển việc Mô-đun Mật mã Go có hoạt động ở chế độ FIPS 140-3 hay không. Không thể thay đổi tùy chọn này sau khi chương trình đã khởi động.

Mặc định là `off` trừ khi `GOFIPS140` được đặt tại thời điểm build.

Nếu được đặt thành `on`, chế độ FIPS 140-3 được bật. Điều này có thể xảy ra ngay cả khi `GOFIPS140` không được đặt tại thời điểm build.

Nếu được đặt thành `only`, các thuật toán mật mã không tuân thủ FIPS 140-3 sẽ trả về lỗi hoặc panic. Lưu ý rằng đây là chế độ nỗ lực tốt nhất dành cho việc kiểm thử, đánh giá và gỡ lỗi. *Chế độ này không được thiết kế để sử dụng trong môi trường production*, không được Security Policy yêu cầu, cố ý tạo ra các lỗi crash và các lỗi có khả năng không được xử lý theo thiết kế, đồng thời có thể có dương tính giả hoặc âm tính giả.

Hầu hết chương trình không nên đặt trực tiếp tùy chọn này, mà thay vào đó nên sử dụng `GOFIPS140` tại thời điểm build.

## Phiên bản mô-đun, quá trình xác thực và khả năng tương thích

Google hiện có mối quan hệ hợp đồng với [Geomys](https://geomys.org/)
để hỗ trợ việc xác thực CMVP ít nhất hằng năm cho Mô-đun mật mã Go.
Tại thời điểm xác thực, chúng tôi sẽ đóng băng Mô-đun mật mã Go và tạo
một phiên bản mô-đun mới để gửi đi.

Các quá trình xác thực này được kiểm thử trên một tập hợp toàn diện các
Môi trường vận hành, hỗ trợ nhiều tổ hợp hệ điều hành và nền tảng phần cứng
phổ biến.

Các phiên bản cũ hơn của Mô-đun mật mã Go tiếp tục được hỗ trợ và cung cấp
miễn là chưa có phiên bản mới hơn nhận được chứng chỉ xác thực CMVP.
Khi một phiên bản mới hơn đã nhận được chứng chỉ xác thực CMVP, các phiên bản
cũ hơn sẽ bị gỡ bỏ.

Một số tính năng của thư viện chuẩn có thể không khả dụng và trả về lỗi nếu sử dụng
Mô-đun mật mã Go được đóng băng từ một phiên bản Go cũ hơn.

### Mô-đun mật mã Go v1.26.0

Mô-đun mật mã Go v1.26.0 được đóng băng vào đầu năm 2026 từ Go 1.26.

Mô-đun này có sẵn trong Go 1.26+.

Tính đến ngày 2026-04-28, mô-đun này đang ở trạng thái Đang chờ xem xét trong Danh sách các mô-đun đang được xử lý của CMVP.
Mô-đun này được bao phủ bởi [CAVP Certificate A8028](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program/details?validation=40638).

#### Thay đổi từ v1.0.0

  - Đã triển khai ML-DSA.

  - [testing/cryptotest.SetGlobalRandom](/pkg/testing/cryptotest#SetGlobalRandom) hiện được hỗ trợ.

  - Đã giới thiệu các API tuân thủ AES-GCM mới, được sử dụng trong `crypto/hpke` và các API được công khai trong tương lai.

  - Mô-đun mật mã Go hiện sử dụng Nguồn entropy CPU jitter, với
  [ESV Certificate #E318](https://csrc.nist.gov/projects/cryptographic-module-validation-program/entropy-validations/certificate/318)
  và [CAVP Certificate A7715](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program/details?product=20498).
  (CSPRNG của nền tảng vẫn được sử dụng như một nguồn dữ liệu bổ sung không được ghi nhận cho tất cả các byte ngẫu nhiên.)

  - Các cải tiến khác về tính an toàn và hiệu năng.

### Go Cryptographic Module v1.0.0

Go Cryptographic Module v1.0.0 được đóng băng vào đầu năm 2024 từ Go 1.24.

Nó có sẵn trong Go 1.24+.

Nó được bao phủ bởi [Chứng chỉ CMVP #5247](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/5247)
và [Chứng chỉ CAVP A6650](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program/details?product=19371).

## Go+BoringCrypto

Cơ chế trước đây, không được hỗ trợ, để sử dụng mô-đun BoringCrypto cho một số thuật toán được FIPS 140-3 phê duyệt hiện vẫn còn khả dụng, nhưng dự kiến sẽ bị loại bỏ và thay thế bằng cơ chế được mô tả trong trang này trong một bản phát hành tương lai.

Go+BoringCrypto không tương thích với chế độ FIPS 140-3 gốc.
