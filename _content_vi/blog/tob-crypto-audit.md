---
title: "Kiểm tra Bảo mật Mật mã học Go"
date: 2025-05-19
by:
- Roland Shoemaker and Filippo Valsorda
summary: Các thư viện mật mã học của Go đã được kiểm toán bởi Trail of Bits.
---

Go đi kèm với một bộ đầy đủ các gói mật mã học trong thư viện chuẩn để giúp lập trình viên xây dựng các ứng dụng bảo mật. Google gần đây đã hợp đồng với công ty bảo mật độc lập [Trail of Bits](https://www.trailofbits.com/) để thực hiện kiểm toán tập hợp cốt lõi các gói cũng được xác thực như một phần của [module FIPS 140-3 native mới](/doc/go1.24#fips140). Kiểm toán tạo ra một phát hiện có mức độ nghiêm trọng thấp, trong [tích hợp Go+BoringCrypto kế thừa và không được hỗ trợ](/doc/security/fips140#goboringcrypto), và một số phát hiện mang tính thông tin. Toàn văn báo cáo kiểm toán có thể tìm thấy [tại đây](https://github.com/trailofbits/publications/blob/d47e8fafa7e3323e5620d228f2f3f3bf58ed5978/reviews/2025-03-google-gocryptographiclibraries-securityreview.pdf).

Phạm vi kiểm toán bao gồm các triển khai trao đổi khóa (ECDH và ML-KEM post-quantum), chữ ký số (ECDSA, RSA và Ed25519), mã hóa (AES-GCM, AES-CBC và AES-CTR), băm (SHA-1, SHA-2 và SHA-3), dẫn xuất khóa (HKDF và PBKDF2) và xác thực (HMAC), cũng như bộ tạo số ngẫu nhiên mật mã. Các triển khai số nguyên lớn cấp thấp và đường cong elliptic, với các lõi assembly tinh tế, đã được đưa vào. Các giao thức cấp cao hơn như TLS và X.509 không nằm trong phạm vi. Ba kỹ sư của Trail of Bits đã làm việc về kiểm toán trong một tháng.

Chúng tôi tự hào về lịch sử bảo mật của các gói mật mã học Go, và về kết quả của cuộc kiểm toán này, chỉ là một trong nhiều cách chúng tôi đạt được sự đảm bảo về tính đúng đắn của các gói. Đầu tiên, chúng tôi giới hạn tích cực độ phức tạp của chúng, được hướng dẫn bởi [Nguyên tắc Mật mã học](/design/cryptography-principles) ưu tiên bảo mật hơn hiệu năng. Hơn nữa, chúng tôi [kiểm thử kỹ lưỡng chúng](https://www.youtube.com/watch?v=lkEH3V3PkS0) với một loạt các kỹ thuật khác nhau. Chúng tôi chú trọng đến việc tận dụng các API an toàn ngay cả đối với các gói nội bộ, và tự nhiên chúng tôi có thể dựa vào các thuộc tính ngôn ngữ Go để tránh các vấn đề quản lý bộ nhớ. Cuối cùng, chúng tôi tập trung vào khả năng đọc để giúp việc bảo trì dễ dàng hơn và việc đánh giá mã cũng như kiểm toán hiệu quả hơn.

## Một phát hiện có mức độ nghiêm trọng thấp

Vấn đề có thể khai thác duy nhất, TOB-GOCL-3, có *mức độ nghiêm trọng thấp*, có nghĩa là nó có tác động nhỏ và khó kích hoạt. Vấn đề này đã được sửa trong Go 1.24.

Quan trọng là, TOB-GOCL-3 ([thảo luận thêm bên dưới](#cgo-memory-management)) liên quan đến quản lý bộ nhớ trong [GOEXPERIMENT Go+BoringCrypto kế thừa](/doc/security/fips140#goboringcrypto), không được bật theo mặc định và không được hỗ trợ để sử dụng bên ngoài Google.

## Năm phát hiện mang tính thông tin

Các phát hiện còn lại là *thông tin*, có nghĩa là chúng không đặt ra rủi ro ngay lập tức nhưng có liên quan đến các thực hành tốt nhất về bảo mật. Chúng tôi đã giải quyết những điều này trong cây phát triển Go 1.25 hiện tại.

Phát hiện TOB-GOCL-1, TOB-GOCL-2 và TOB-GOCL-6 liên quan đến các timing side-channel có thể trong các thao tác mật mã khác nhau. Trong số ba phát hiện này, chỉ TOB-GOCL-2 ảnh hưởng đến các thao tác được kỳ vọng là constant time do hoạt động trên các giá trị bí mật, nhưng nó chỉ ảnh hưởng đến các mục tiêu Power ISA (GOARCH ppc64 và ppc64le). TOB-GOCL-4 nêu bật rủi ro lạm dụng trong một API nội bộ, nếu nó được tái sử dụng ngoài trường hợp sử dụng hiện tại. TOB-GOCL-5 chỉ ra một kiểm tra bị thiếu cho giới hạn không thể đạt được trong thực tế.

## Timing Side-Channel

Phát hiện TOB-GOCL-1, TOB-GOCL-2 và TOB-GOCL-6 liên quan đến các timing side-channel nhỏ. TOB-GOCL-1 và TOB-GOCL-6 liên quan đến các hàm chúng tôi không dùng cho các giá trị nhạy cảm, nhưng có thể được dùng cho các giá trị như vậy trong tương lai, và TOB-GOCL-2 liên quan đến triển khai assembly của P-256 ECDSA trên Power ISA.

### `crypto/ecdh,crypto/ecdsa`: chuyển đổi từ bytes sang field element không phải constant time (TOB-GOCL-1)

Triển khai nội bộ của các đường cong elliptic NIST cung cấp một phương thức để chuyển đổi các field element giữa biểu diễn nội bộ và bên ngoài hoạt động trong thời gian biến thiên.

Tất cả các cách dùng của phương thức này đã hoạt động trên các đầu vào công khai không được coi là bí mật (các giá trị ECDH công khai và khóa công khai ECDSA), vì vậy chúng tôi xác định đây không phải là vấn đề bảo mật. Mặc dù vậy, chúng tôi đã quyết định [làm cho phương thức constant time dù sao](/cl/650579), để ngăn chặn việc vô tình sử dụng phương thức này trong tương lai với các giá trị bí mật, và để chúng tôi không phải suy nghĩ về việc liệu đây có phải là vấn đề hay không.

### `crypto/ecdsa`: Phủ định có điều kiện P-256 không phải constant time trong assembly Power ISA (TOB-GOCL-2, CVE-2025-22866)

Ngoài [các nền tảng Go hạng nhất](/wiki/PortingPolicy#first-class-ports), Go cũng hỗ trợ một số nền tảng bổ sung, bao gồm một số kiến trúc ít phổ biến hơn. Trong quá trình xem xét các triển khai assembly của các nguyên thủy mật mã cơ bản, nhóm Trail of Bits đã tìm thấy một vấn đề ảnh hưởng đến triển khai ECDSA trên các kiến trúc ppc64 và ppc64le.

Do việc sử dụng lệnh phân nhánh có điều kiện trong triển khai phủ định có điều kiện của các điểm P-256, hàm hoạt động trong thời gian biến thiên, thay vì constant time, như mong đợi. Sửa chữa cho vấn đề này tương đối đơn giản, [thay thế lệnh phân nhánh có điều kiện](/cl/643735) bằng một mẫu chúng tôi đã dùng ở nơi khác để có điều kiện chọn kết quả đúng trong constant time. Chúng tôi đã gán CVE-2025-22866 cho vấn đề này.

Để ưu tiên mã tiếp cận hầu hết người dùng của chúng tôi, và do kiến thức chuyên biệt cần thiết để nhắm mục tiêu ISA cụ thể, chúng tôi thường dựa vào đóng góp cộng đồng để duy trì assembly cho các nền tảng không phải hạng nhất. Chúng tôi cảm ơn các đối tác tại IBM đã giúp cung cấp đánh giá cho bản sửa lỗi của chúng tôi.

### `crypto/ed25519`: Scalar.SetCanonicalBytes không phải constant time (TOB-GOCL-6)

Gói edwards25519 nội bộ cung cấp một phương thức để chuyển đổi giữa biểu diễn nội bộ và bên ngoài của scalars hoạt động trong thời gian biến thiên.

Phương thức này chỉ được dùng trên các đầu vào chữ ký cho ed25519.Verify, không được coi là bí mật, vì vậy chúng tôi xác định đây không phải là vấn đề bảo mật. Mặc dù vậy, tương tự như phát hiện TOB-GOCL-1, chúng tôi đã quyết định [làm cho phương thức constant time dù sao](/cl/648035), để ngăn chặn việc vô tình sử dụng phương thức này trong tương lai với các giá trị bí mật, và vì chúng tôi biết rằng mọi người fork code này bên ngoài thư viện chuẩn, và có thể đang sử dụng nó với các giá trị bí mật.

## Quản lý Bộ nhớ Cgo

Phát hiện TOB-GOCL-3 liên quan đến một vấn đề quản lý bộ nhớ trong tích hợp Go+BoringCrypto.

### `crypto/ecdh`: finalizer tùy chỉnh có thể giải phóng bộ nhớ ở đầu lời gọi hàm C sử dụng bộ nhớ này (TOB-GOCL-3)

Trong quá trình xem xét, có một số câu hỏi về tích hợp Go+BoringCrypto dựa trên cgo của chúng tôi, cung cấp chế độ mật mã học tuân thủ FIPS 140-2 để sử dụng nội bộ tại Google. Code Go+BoringCrypto không được nhóm Go hỗ trợ để sử dụng bên ngoài, nhưng rất quan trọng cho việc sử dụng Go nội bộ tại Google.

Nhóm Trail of Bits đã tìm thấy một lỗ hổng bảo mật và một [lỗi không liên quan đến bảo mật](/cl/644120), cả hai đều là kết quả của việc quản lý bộ nhớ thủ công cần thiết để tương tác với thư viện C. Vì nhóm Go không hỗ trợ việc sử dụng code này bên ngoài Google, chúng tôi đã chọn không phát hành CVE hay mục nhập cơ sở dữ liệu lỗ hổng bảo mật Go cho vấn đề này, nhưng chúng tôi [đã sửa nó trong Go 1.24](/cl/644119).

Kiểu bẫy này là một trong nhiều lý do chúng tôi quyết định rời xa tích hợp Go+BoringCrypto. Chúng tôi đã làm việc trên [chế độ FIPS 140-3 native](/doc/security/fips140) sử dụng các gói mật mã học Go thuần túy thông thường, cho phép chúng tôi tránh ngữ nghĩa cgo phức tạp thay cho mô hình bộ nhớ Go truyền thống.

## Tính Đầy đủ của Triển khai

Phát hiện TOB-GOCL-4 và TOB-GOCL-5 liên quan đến các triển khai giới hạn của hai đặc tả, [NIST SP 800-90A](https://csrc.nist.gov/pubs/sp/800/90/a/r1/final) và [RFC 8018](https://datatracker.ietf.org/doc/html/rfc8018).

### `crypto/internal/fips140/drbg`: API CTR\_DRBG có nhiều rủi ro lạm dụng (TOB-GOCL-4)

Là một phần của [chế độ FIPS 140-3 native](/doc/security/fips140) mà chúng tôi đang giới thiệu, chúng tôi cần một triển khai NIST CTR\_DRBG (bộ tạo bit ngẫu nhiên xác định dựa trên AES-CTR) để cung cấp tính ngẫu nhiên tuân thủ.

Vì chúng tôi chỉ cần một tập hợp nhỏ chức năng của NIST SP 800-90A Rev. 1 CTR\_DRBG cho mục đích của mình, chúng tôi không triển khai đầy đủ đặc tả, đặc biệt bỏ qua hàm dẫn xuất và các chuỗi cá nhân hóa. Các tính năng này có thể quan trọng để sử dụng DRBG an toàn trong các bối cảnh chung.

Vì triển khai của chúng tôi được phạm vi chặt chẽ đến trường hợp sử dụng cụ thể chúng tôi cần, và vì triển khai không được xuất công khai, chúng tôi xác định điều này là chấp nhận được và đáng để giảm độ phức tạp của triển khai. Chúng tôi không mong đợi triển khai này sẽ bao giờ được sử dụng cho các mục đích khác nội bộ, và đã [thêm cảnh báo vào tài liệu](/cl/647815) chi tiết về các hạn chế này.

### `crypto/pbkdf2`: PBKDF2 không áp dụng giới hạn độ dài đầu ra (TOB-GOCL-5)

Trong Go 1.24, chúng tôi bắt đầu quá trình chuyển các gói từ [golang.org/x/crypto](https://golang.org/x/crypto) vào thư viện chuẩn, kết thúc một mẫu khó hiểu trong đó các gói mật mã học Go hạng nhất, chất lượng cao và ổn định được giữ bên ngoài thư viện chuẩn mà không có lý do cụ thể.

Là một phần của quá trình này, chúng tôi đã chuyển gói [golang.org/x/crypto/pbkdf2](https://golang.org/x/crypto/pbkdf2) vào thư viện chuẩn, thành crypto/pbkdf2. Trong khi xem xét gói này, nhóm Trail of Bits nhận thấy rằng chúng tôi không áp dụng giới hạn về kích thước khóa được dẫn xuất được định nghĩa trong [RFC 8018](https://datatracker.ietf.org/doc/html/rfc8018).

Giới hạn là `(2^32 - 1) * <hash length>`, sau đó khóa sẽ lặp lại. Khi sử dụng SHA-256, vượt quá giới hạn sẽ cần một khóa có độ dài hơn 137GB. Chúng tôi không mong đợi ai từng sử dụng PBKDF2 để tạo khóa lớn như vậy, đặc biệt là vì PBKDF2 chạy các lần lặp ở mỗi khối, nhưng vì sự đúng đắn, chúng tôi [bây giờ áp dụng giới hạn theo định nghĩa của tiêu chuẩn](/cl/644122).

# Tiếp theo là gì

Kết quả của cuộc kiểm toán này xác nhận nỗ lực mà nhóm Go đã đầu tư vào việc phát triển các thư viện mật mã học chất lượng cao, dễ sử dụng và nên mang lại sự tự tin cho người dùng của chúng tôi, những người dựa vào chúng để xây dựng phần mềm an toàn và bảo mật.

Chúng tôi không tự mãn: các người đóng góp Go đang tiếp tục phát triển và cải thiện các thư viện mật mã học chúng tôi cung cấp cho người dùng.

Go 1.24 hiện bao gồm chế độ FIPS 140-3 được viết bằng Go thuần túy, hiện đang trải qua kiểm tra CMVP. Điều này sẽ cung cấp chế độ tuân thủ FIPS 140-3 được hỗ trợ cho tất cả người dùng Go, thay thế tích hợp Go+BoringCrypto hiện không được hỗ trợ.

Chúng tôi cũng đang làm việc để triển khai mật mã học post-quantum hiện đại, giới thiệu triển khai ML-KEM-768 và ML-KEM-1024 trong Go 1.24 trong [gói crypto/mlkem](/pkg/crypto/mlkem), và thêm hỗ trợ cho gói crypto/tls cho trao đổi khóa X25519MLKEM768 lai ghép.

Cuối cùng, chúng tôi đang lên kế hoạch giới thiệu các API mật mã học cấp cao mới dễ sử dụng hơn, được thiết kế để giảm rào cản trong việc chọn và sử dụng các thuật toán chất lượng cao cho các trường hợp sử dụng cơ bản. Chúng tôi dự định bắt đầu bằng cách cung cấp API băm mật khẩu đơn giản loại bỏ nhu cầu người dùng phải quyết định trong số vô số thuật toán có thể họ nên dựa vào, với các cơ chế để tự động di chuyển sang các thuật toán mới hơn khi trạng thái nghệ thuật thay đổi.
