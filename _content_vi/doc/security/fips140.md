---
title: Tuân thủ FIPS 140-3
layout: article
---

Bắt đầu từ Go 1.24, các tệp nhị phân Go có thể hoạt động nguyên bản ở chế độ hỗ trợ tuân thủ FIPS 140-3. Hơn nữa, toolchain có thể xây dựng với các phiên bản đóng băng của các gói mật mã tạo thành Module Mật mã Go.

## FIPS 140-3

NIST FIPS 140-3 là chế độ tuân thủ của Chính phủ Hoa Kỳ đối với các ứng dụng mật mã, trong số các yêu cầu khác đòi hỏi sử dụng một tập hợp các thuật toán được phê duyệt và sử dụng các module mật mã được xác nhận bởi
[CMVP](https://csrc.nist.gov/projects/cryptographic-module-validation-program)
và được kiểm thử trong các môi trường vận hành mục tiêu.

Các cơ chế được mô tả trong trang này hỗ trợ việc tuân thủ cho các ứng dụng Go.

Các ứng dụng không cần tuân thủ FIPS 140-3 có thể bỏ qua chúng một cách an toàn,
và không nên bật chế độ FIPS 140-3.

**LƯU Ý:** Chỉ đơn giản sử dụng một module mật mã tuân thủ và được xác nhận FIPS 140-3 có thể không tự nó đáp ứng tất cả các yêu cầu quy định liên quan. Nhóm Go không thể cung cấp bất kỳ đảm bảo hoặc hỗ trợ nào về việc sử dụng chế độ FIPS 140-3 được cung cấp có thể hoặc không thể đáp ứng các yêu cầu quy định cụ thể cho từng người dùng. Cần thận trọng khi xác định xem việc sử dụng module này có đáp ứng các yêu cầu cụ thể của bạn hay không.

## Module Mật mã Go

Module Mật mã Go là tập hợp các gói thư viện chuẩn Go
trong `crypto/internal/fips140/...` triển khai các thuật toán được phê duyệt theo FIPS 140-3.

Các gói API công khai như `crypto/ecdsa` và `crypto/rand` sử dụng
Module Mật mã Go một cách minh bạch để triển khai các thuật toán FIPS 140-3.

## Chế độ FIPS 140-3

Tùy chọn [GODEBUG](/doc/godebug) `fips140` thời gian chạy kiểm soát xem Module Mật mã Go
có hoạt động ở chế độ FIPS 140-3 hay không. Giá trị mặc định là `off`. Nó không thể
thay đổi sau khi chương trình đã khởi động.

Khi hoạt động ở chế độ FIPS 140-3 (cài đặt GODEBUG `fips140` là `on`):

 - Module Mật mã Go tự động thực hiện kiểm tra tính toàn vẹn tự kiểm tra tại
   thời điểm `init`, so sánh checksum của tệp đối tượng của module được tính tại
   thời điểm xây dựng với các ký hiệu được nạp vào bộ nhớ.

 - Tất cả các thuật toán thực hiện kiểm tra tự kiểm tra known-answer theo Hướng dẫn Triển khai FIPS
   140-3 liên quan, tại thời điểm `init`, hoặc khi sử dụng lần đầu.

 - Kiểm thử nhất quán theo cặp được thực hiện trên các khóa mật mã được tạo.
   Lưu ý rằng điều này có thể gây ra chậm trễ tới 2x cho một số loại khóa,
   đặc biệt liên quan đến các khóa tạm thời.

 - [`crypto/rand.Reader`](/pkg/crypto/rand/#Reader) được triển khai dưới dạng
   NIST SP 800-90A DRBG. Để đảm bảo cùng mức độ bảo mật như
   `GODEBUG=fips140=off`, các byte ngẫu nhiên cũng được lấy từ CSPRNG của nền tảng tại
   mỗi lần `Read` và được trộn vào đầu ra như dữ liệu bổ sung không được tính điểm.

 - Gói [`crypto/tls`](/pkg/crypto/tls/) sẽ bỏ qua và không thương lượng
   bất kỳ phiên bản giao thức, bộ mã hóa, thuật toán chữ ký hoặc cơ chế trao đổi khóa
   nào không được FIPS 140-3 phê duyệt.

 - [`crypto/rsa.SignPSS`](/pkg/crypto/rsa/#SignPSS) với
   [`PSSSaltLengthAuto`](/pkg/crypto/rsa/#PSSSaltLengthAuto) sẽ giới hạn độ dài
   của salt bằng độ dài của hàm băm.

Khi sử dụng `GODEBUG=fips140=only`, ngoài các điều trên, các thuật toán mật mã
không tuân thủ FIPS 140-3 sẽ trả về lỗi hoặc panic. Lưu ý
rằng chế độ này là cố gắng tốt nhất và không thể đảm bảo tuân thủ tất cả các yêu cầu FIPS
140-3.

`GODEBUG=fips140=on` và `only` không được hỗ trợ trên OpenBSD, Wasm, AIX và
các nền tảng Windows 32-bit.

## Gói `crypto/fips140`

Hàm [`crypto/fips140.Enabled`](/pkg/crypto/fips140/#Enabled) báo cáo
xem chế độ FIPS 140-3 có đang hoạt động hay không.

## Biến môi trường `GOFIPS140`

Biến môi trường `GOFIPS140` có thể được sử dụng với `go build`, `go install`,
và `go test` để chọn phiên bản của Module Mật mã Go được liên kết
vào chương trình thực thi.

 - `off` là giá trị mặc định, sử dụng các gói `crypto/internal/fips140/...` trong
   cây thư viện chuẩn đang dùng.

 - `latest` giống như `off`, nhưng bật chế độ FIPS 140-3 theo mặc định.

 - `v1.0.0` sử dụng Module Mật mã Go phiên bản v1.0.0, được đóng băng vào đầu năm 2025
   và được vận chuyển lần đầu với Go 1.24. Nó bật chế độ FIPS 140-3 theo mặc định.

 - `v1.26.0` sử dụng Module Mật mã Go phiên bản v1.26.0, được đóng băng vào đầu năm 2026
   và được vận chuyển lần đầu với Go 1.26. Nó bật chế độ FIPS 140-3 theo mặc định.

## Xác nhận Module

Google hiện có quan hệ hợp đồng với [Geomys](https://geomys.org/)
để thực hiện ít nhất hàng năm các xác nhận CMVP của Module Mật mã Go.
Tại thời điểm xác nhận, chúng tôi sẽ đóng băng Module Mật mã Go và tạo
một phiên bản module mới để nộp.

Các xác nhận này được kiểm thử trên một tập hợp toàn diện các Môi trường vận hành,
hỗ trợ nhiều kết hợp hệ điều hành và nền tảng phần cứng phổ biến.

Các xác nhận ngoài chu kỳ có thể được thực hiện nếu các vấn đề bảo mật được phát hiện trong
module.

### Các phiên bản Module đã được xác nhận

Danh sách các phiên bản module đã hoàn thành [xác nhận CMVP](https://csrc.nist.gov/projects/cryptographic-module-validation-program/validated-modules/search?SearchMode=Basic&ModuleName=Go+Cryptographic+Module&CertificateStatus=Active&ValidationYear=0):

_Hiện tại chưa có phiên bản module nào hoàn thành xác nhận._

### Các phiên bản Module đang trong quá trình xử lý

Danh sách các phiên bản module hiện đang có trong [Danh sách Module đang xử lý CMVP](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/modules-in-process/modules-in-process-list):

* v1.0.0 ([Chứng chỉ CAVP A6650](https://csrc.nist.gov/projects/cryptographic-algorithm-validation-program/details?validation=39260)), Đang xem xét, có trong Go 1.24+

### Các phiên bản Module đang được kiểm thử triển khai

Danh sách các phiên bản module hiện đang có trong [Danh sách Triển khai đang kiểm thử CMVP](https://csrc.nist.gov/Projects/cryptographic-module-validation-program/modules-in-process/iut-list):

* v1.26.0, có trong Go 1.26+

## Go+BoringCrypto

Cơ chế cũ không được hỗ trợ để sử dụng module BoringCrypto cho một số
thuật toán được phê duyệt theo FIPS 140-3 hiện vẫn còn khả dụng, nhưng nó dự kiến
sẽ bị xóa và thay thế bằng cơ chế được mô tả trong trang này trong một bản phát hành tương lai.

Go+BoringCrypto không tương thích với chế độ FIPS 140-3 gốc.
