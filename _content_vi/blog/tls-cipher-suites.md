---
title: "Tự động sắp xếp cipher suite trong crypto/tls"
date: 2021-09-15
by:
- Filippo Valsorda
summary: Go 1.17 giúp việc cấu hình TLS dễ dàng và an toàn hơn bằng cách tự động hóa thứ tự ưu tiên cipher suite TLS.
---

Thư viện chuẩn Go cung cấp `crypto/tls`,
một triển khai mạnh mẽ của Transport Layer Security (TLS),
giao thức bảo mật quan trọng nhất trên Internet,
và là thành phần cơ bản của HTTPS.
Trong Go 1.17 chúng tôi đã làm cho cấu hình của nó dễ dàng hơn, an toàn hơn,
và hiệu quả hơn bằng cách tự động hóa thứ tự ưu tiên của các cipher suite.


## Cipher suite hoạt động như thế nào

Cipher suite có từ thời tiền thân của TLS là Secure Socket Layer (SSL),
[gọi chúng là "cipher kinds"](https://datatracker.ietf.org/doc/html/draft-hickman-netscape-ssl-00#appendix-C.4).
Chúng là các định danh có vẻ đáng sợ như
`TLS_RSA_WITH_AES_256_CBC_SHA` và
`TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256`
liệt kê các thuật toán được dùng để trao đổi khóa,
xác thực chứng chỉ và mã hóa bản ghi trong kết nối TLS.

Cipher suite được _đàm phán_ trong quá trình TLS handshake:
client gửi danh sách các cipher suite nó hỗ trợ trong tin nhắn đầu tiên,
Client Hello, và server chọn một từ danh sách đó,
thông báo lựa chọn của mình cho client.
Client gửi danh sách các cipher suite được hỗ trợ theo thứ tự ưu tiên của chính nó,
và server tự do chọn từ đó theo cách nó muốn.
Thông thường nhất, server sẽ chọn cipher suite được hỗ trợ chung đầu tiên
theo thứ tự ưu tiên của client hoặc theo thứ tự ưu tiên của server,
dựa trên cấu hình của nó.

Cipher suite thực sự chỉ là một trong nhiều tham số được đàm phán, các
đường cong/nhóm được hỗ trợ và thuật toán chữ ký cũng được đàm phán thêm thông qua
các phần mở rộng riêng của chúng, nhưng chúng là những thứ phức tạp và nổi tiếng nhất,
và là những thứ duy nhất mà các lập trình viên và quản trị viên được huấn luyện qua
nhiều năm để có ý kiến.

Trong TLS 1.0-1.2, tất cả các tham số này tương tác trong một mạng lưới phức tạp của các phụ thuộc lẫn nhau:
ví dụ các chứng chỉ được hỗ trợ phụ thuộc vào các thuật toán chữ ký được hỗ trợ,
các đường cong được hỗ trợ, và các cipher suite được hỗ trợ.
Trong TLS 1.3, tất cả điều này được đơn giản hóa triệt để:
cipher suite chỉ chỉ định các thuật toán mã hóa đối xứng,
trong khi các đường cong/nhóm được hỗ trợ điều chỉnh trao đổi khóa và các thuật toán chữ ký
được hỗ trợ áp dụng cho chứng chỉ.


## Một lựa chọn phức tạp được giao cho lập trình viên

Hầu hết các HTTPS và TLS server ủy thác việc lựa chọn cipher suite và thứ tự ưu tiên
cho người vận hành server hoặc lập trình viên ứng dụng.
Đây là một lựa chọn phức tạp đòi hỏi kiến thức cập nhật và chuyên sâu vì nhiều lý do.

Một số cipher suite cũ có các thành phần không an toàn,
một số yêu cầu triển khai cực kỳ cẩn thận và phức tạp để bảo mật,
và một số chỉ an toàn nếu client áp dụng các biện pháp giảm thiểu nhất định hoặc thậm chí
có phần cứng nhất định.
Ngoài bảo mật của các thành phần riêng lẻ,
các cipher suite khác nhau có thể cung cấp các thuộc tính bảo mật hoàn toàn khác nhau
cho toàn bộ kết nối,
vì các cipher suite không có ECDHE hoặc DHE không cung cấp forward secrecy, tức là
thuộc tính mà các kết nối không thể bị giải mã hồi tố hoặc thụ động
bằng khóa của chứng chỉ.
Cuối cùng, việc lựa chọn các cipher suite được hỗ trợ ảnh hưởng đến tính tương thích và hiệu năng,
và việc thực hiện các thay đổi mà không có hiểu biết cập nhật về hệ sinh thái
có thể dẫn đến việc phá vỡ các kết nối từ các client cũ,
tăng tài nguyên tiêu thụ của server,
hoặc làm hao pin của các thiết bị di động.

Lựa chọn này phức tạp và tinh tế đến mức có các công cụ chuyên dụng để hướng dẫn người vận hành,
chẳng hạn như [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/) xuất sắc.

Chúng ta đã đến đây như thế nào và tại sao lại như vậy?

Để bắt đầu, các thành phần mật mã riêng lẻ từng bị phá vỡ thường xuyên hơn nhiều.
Năm 2011, khi cuộc tấn công BEAST phá vỡ các cipher suite CBC theo cách chỉ
client mới có thể giảm thiểu cuộc tấn công,
các server chuyển sang ưu tiên RC4, không bị ảnh hưởng.
Năm 2013, khi rõ ràng rằng RC4 bị phá vỡ,
các server quay lại CBC.
Khi Lucky Thirteen cho thấy rõ ràng rằng việc triển khai các cipher suite CBC cực kỳ khó
do thiết kế MAC-then-encrypt ngược của chúng...
À, không có gì khác trên bàn, vì vậy các triển khai phải
[vượt qua cẩn thận](https://www.imperialviolet.org/2013/02/04/luckythirteen.html)
để triển khai CBC và [tiếp tục thất bại ở nhiệm vụ đáng sợ đó trong nhiều năm](https://blog.cloudflare.com/yet-another-padding-oracle-in-openssl-cbc-ciphersuites/).
Cipher suite có thể cấu hình và [tính linh hoạt mật mã](https://www.imperialviolet.org/2016/05/16/agility.html)
từng cung cấp một sự đảm bảo rằng khi một thành phần bị phá vỡ, nó có thể được
thay thế ngay lập tức.

Mật mã học hiện đại về cơ bản khác.
Các giao thức vẫn có thể bị phá vỡ không thường xuyên,
nhưng hiếm khi là một thành phần riêng lẻ trừu tượng bị thất bại.
_Không có cipher suite nào dựa trên AEAD được giới thiệu bắt đầu từ TLS 1.2 vào
năm 2008 bị phá vỡ._ Ngày nay tính linh hoạt mật mã là một gánh nặng:
nó tạo ra sự phức tạp có thể dẫn đến điểm yếu hoặc downgrade,
và chỉ cần thiết vì lý do hiệu năng và tuân thủ.

Việc vá lỗi cũng khác trước. Ngày nay chúng ta thừa nhận rằng việc kịp thời áp dụng
các bản vá phần mềm cho các lỗ hổng bảo mật được tiết lộ là nền tảng của triển khai phần mềm an toàn,
nhưng mười năm trước đó không phải là thực hành chuẩn.
Thay đổi cấu hình được xem là tùy chọn phản hồi nhanh hơn nhiều đối với
các cipher suite bị tổn thương,
vì vậy người vận hành, thông qua cấu hình, được đặt hoàn toàn phụ trách chúng.
Bây giờ chúng ta có vấn đề ngược lại: có các server được vá lỗi và cập nhật đầy đủ
vẫn hoạt động kỳ lạ,
dưới mức tối ưu, hoặc không an toàn, vì các cấu hình của chúng chưa được chỉnh sửa trong nhiều năm.

Cuối cùng, người ta hiểu rằng server có xu hướng cập nhật chậm hơn client,
và do đó là những người đánh giá kém tin cậy hơn về lựa chọn tốt nhất của cipher suite.
Tuy nhiên, chính server là người có tiếng nói cuối cùng về việc chọn cipher suite,
vì vậy mặc định trở thành việc làm cho server nhường theo thứ tự ưu tiên của client,
thay vì có ý kiến mạnh mẽ.
Điều này vẫn đúng một phần: trình duyệt đã quản lý để cập nhật tự động xảy ra
và cập nhật thường xuyên hơn nhiều so với server trung bình.
Mặt khác, một số thiết bị cũ bây giờ không còn được hỗ trợ và
bị kẹt ở các cấu hình TLS client cũ,
điều này thường làm cho một server cập nhật được trang bị tốt hơn để chọn hơn một số client của nó.

Bất kể chúng ta đã đến đây như thế nào, đây là sự thất bại của kỹ thuật mật mã
khi yêu cầu các lập trình viên ứng dụng và người vận hành server phải trở thành chuyên gia
về các sắc thái của việc chọn cipher suite,
và phải cập nhật các diễn biến mới nhất để giữ các cấu hình của họ hiện tại.
Nếu họ đang triển khai các bản vá bảo mật của chúng tôi,
điều đó nên là đủ.

Mozilla SSL Configuration Generator rất tốt, và nó không nên tồn tại.

Điều này có đang cải thiện không?

Có tin tốt và tin xấu về xu hướng trong vài năm qua.
Tin xấu là thứ tự đang trở nên phức tạp hơn,
vì có các tập hợp cipher suite có thuộc tính bảo mật tương đương.
Lựa chọn tốt nhất trong tập hợp như vậy phụ thuộc vào phần cứng có sẵn và
khó thể hiện trong tệp cấu hình.
Trong các hệ thống khác, những gì bắt đầu là một danh sách cipher suite đơn giản giờ đây phụ thuộc
vào [cú pháp phức tạp hơn](https://boringssl.googlesource.com/boringssl/+/c3b373bf4f4b2e2fba2578d1d5b5fe04e410f7cb/include/openssl/ssl.h#1457)
hoặc các cờ bổ sung như [SSL\_OP\_PRIORITIZE\_CHACHA](https://www.openssl.org/docs/man1.1.1/man3/SSL_CTX_clear_options.html#:~:text=session-,ssl_op_prioritize_chacha,-When).

Tin tốt là TLS 1.3 đã đơn giản hóa triệt để cipher suite,
và nó sử dụng một tập hợp tách biệt với TLS 1.0-1.2.
Tất cả các cipher suite TLS 1.3 đều an toàn, vì vậy các lập trình viên ứng dụng và người vận hành server
không cần phải lo lắng về chúng.
Thực sự, một số thư viện TLS như BoringSSL và `crypto/tls` của Go
không cho phép cấu hình chúng.


## crypto/tls của Go và cipher suite

Go cho phép cấu hình cipher suite trong TLS 1.0-1.2.
Các ứng dụng luôn có thể thiết lập các cipher suite được bật và
thứ tự ưu tiên với [`Config.CipherSuites`](https://pkg.go.dev/crypto/tls#Config.CipherSuites).
Server ưu tiên thứ tự ưu tiên của client theo mặc định,
trừ khi [`Config.PreferServerCipherSuites`](https://pkg.go.dev/crypto/tls#Config.PreferServerCipherSuites) được thiết lập.

Khi chúng tôi triển khai TLS 1.3 trong Go 1.12, [chúng tôi không làm cho cipher suite TLS 1.3 có thể cấu hình](/issue/29349),
vì chúng là một tập hợp tách biệt với cipher suite TLS 1.0-1.2 và quan trọng nhất là
tất cả đều an toàn,
vì vậy không cần ủy thác lựa chọn cho ứng dụng.
`Config.PreferServerCipherSuites` vẫn kiểm soát bên nào có thứ tự ưu tiên được sử dụng,
và ưu tiên của phía địa phương phụ thuộc vào phần cứng có sẵn.

Trong Go 1.14 chúng tôi [tiết lộ các cipher suite được hỗ trợ](https://pkg.go.dev/crypto/tls#CipherSuites),
nhưng đã chọn rõ ràng trả về chúng theo thứ tự trung lập (được sắp xếp theo ID của chúng),
để chúng tôi sẽ không bị buộc phải đại diện logic ưu tiên của mình
theo cách một thứ tự sắp xếp tĩnh.

Trong Go 1.16, chúng tôi bắt đầu tích cực [ưu tiên cipher suite ChaCha20Poly1305 hơn AES-GCM trên server](/cl/262857)
khi phát hiện rằng client hoặc server thiếu hỗ trợ phần cứng cho AES-GCM.
Điều này là vì AES-GCM khó triển khai hiệu quả và an toàn mà không có
hỗ trợ phần cứng chuyên dụng (chẳng hạn như các bộ lệnh AES-NI và CLMUL).

**Go 1.17, mới được phát hành, tiếp quản thứ tự ưu tiên cipher suite cho tất cả người dùng Go.**
Trong khi `Config.CipherSuites` vẫn kiểm soát cipher suite TLS 1.0-1.2 nào được bật,
nó không được dùng để sắp xếp, và `Config.PreferServerCipherSuites` bây giờ bị bỏ qua.
Thay vào đó, `crypto/tls` [đưa ra tất cả các quyết định sắp xếp](/cl/314609),
dựa trên các cipher suite có sẵn, phần cứng địa phương,
và khả năng phần cứng từ xa được suy ra.

[Logic sắp xếp TLS 1.0-1.2 hiện tại](https://cs.opensource.google/go/go/+/9d0819b27ca248f9949e7cf6bf7cb9fe7cf574e8:src/crypto/tls/cipher_suites.go;l=206-270)
tuân theo các quy tắc sau:



1. ECDHE được ưu tiên hơn trao đổi khóa RSA tĩnh.

    Thuộc tính quan trọng nhất của một cipher suite là cho phép forward secrecy.
    Chúng tôi không triển khai Diffie-Hellman trường hữu hạn "cổ điển",
    vì nó phức tạp, chậm hơn, yếu hơn, và [bị phá vỡ tinh tế](https://datatracker.ietf.org/doc/draft-bartle-tls-deprecate-ffdh/) trong TLS 1.0-1.2,
    vì vậy điều đó có nghĩa là ưu tiên trao đổi khóa Elliptic Curve Diffie-Hellman
    hơn trao đổi khóa RSA tĩnh kế thừa.
    (Cái sau đơn giản là mã hóa bí mật của kết nối bằng khóa công khai của chứng chỉ,
    làm cho nó có thể giải mã nếu chứng chỉ bị xâm phạm trong tương lai.)

2. Chế độ AEAD được ưu tiên hơn CBC để mã hóa.

    Ngay cả khi chúng tôi triển khai các biện pháp phản công một phần cho Lucky13
    ([đóng góp đầu tiên của tôi cho thư viện chuẩn Go, vào năm 2015!](/cl/18130)),
    các cipher suite CBC là [một cơn ác mộng để thực hiện đúng](https://blog.cloudflare.com/yet-another-padding-oracle-in-openssl-cbc-ciphersuites/),
    vì vậy mọi thứ khác quan trọng hơn đều ngang nhau,
    chúng tôi chọn AES-GCM và ChaCha20Poly1305 thay thế.

3. 3DES, CBC-SHA256, và RC4 chỉ được dùng nếu không có gì khác, theo thứ tự ưu tiên đó.

    3DES có các khối 64-bit, khiến nó về cơ bản dễ bị tổn thương với
    [tấn công sinh nhật](https://sweet32.info) khi có đủ lưu lượng.
    3DES được liệt kê dưới [`InsecureCipherSuites`](https://pkg.go.dev/crypto/tls#InsecureCipherSuites),
    nhưng nó được bật theo mặc định để tương thích.
    (Một lợi ích bổ sung của việc kiểm soát thứ tự ưu tiên là
    chúng tôi có thể giữ các cipher suite ít an toàn hơn được bật theo mặc định
    mà không lo ngại về các ứng dụng hoặc client chọn chúng
    ngoại trừ như là phương án cuối cùng.
    Điều này an toàn vì không có tấn công downgrade nào dựa trên
    sự sẵn có của cipher suite yếu hơn để tấn công các peer
    hỗ trợ các lựa chọn tốt hơn.)


    Các cipher suite CBC dễ bị tấn công side channel kiểu Lucky13
    và chúng tôi chỉ triển khai một phần [các biện pháp phản công phức tạp](https://www.imperialviolet.org/2013/02/04/luckythirteen.html)
    được thảo luận ở trên cho hàm băm SHA-1, không phải SHA-256.
    Các cipher suite CBC-SHA1 có giá trị tương thích, biện minh cho độ phức tạp bổ sung,
    trong khi các cipher suite CBC-SHA256 thì không, vì vậy chúng bị tắt theo mặc định.


    RC4 có [các bias thực sự có thể khai thác](https://www.rc4nomore.com)
    có thể dẫn đến phục hồi plaintext mà không có side channel.
    Không có gì tệ hơn thế này, vì vậy RC4 bị tắt theo mặc định.

4. ChaCha20Poly1305 được ưu tiên hơn AES-GCM để mã hóa,
    trừ khi cả hai phía đều có hỗ trợ phần cứng cho cái sau.

    Như đã thảo luận ở trên, AES-GCM khó triển khai hiệu quả và
    an toàn mà không có hỗ trợ phần cứng.
    Nếu chúng tôi phát hiện rằng không có hỗ trợ phần cứng địa phương hoặc (trên server)
    rằng client không ưu tiên AES-GCM,
    chúng tôi chọn ChaCha20Poly1305 thay thế.

5. AES-128 được ưu tiên hơn AES-256 để mã hóa.

    AES-256 có khóa lớn hơn AES-128, thường tốt hơn,
    nhưng nó cũng thực hiện nhiều vòng của hàm mã hóa cốt lõi hơn,
    làm cho nó chậm hơn.
    (Các vòng bổ sung trong AES-256 độc lập với thay đổi kích thước khóa;
    chúng là một nỗ lực cung cấp biên độ rộng hơn chống lại phân tích mật mã.)
    Khóa lớn hơn chỉ hữu ích trong các cài đặt đa người dùng và post-quantum,
    không liên quan đến TLS, vốn tạo ra các IV đủ ngẫu nhiên
    và không có hỗ trợ trao đổi khóa post-quantum.
    Vì khóa lớn hơn không có lợi ích,
    chúng tôi ưu tiên AES-128 vì tốc độ của nó.


[Logic sắp xếp TLS 1.3](https://cs.opensource.google/go/go/+/9d0819b27ca248f9949e7cf6bf7cb9fe7cf574e8:src/crypto/tls/cipher_suites.go;l=342-355)
chỉ cần hai quy tắc cuối,
vì TLS 1.3 đã loại bỏ các thuật toán có vấn đề mà ba quy tắc đầu
đang bảo vệ chống lại.


## Câu hỏi thường gặp

_Nếu một cipher suite bị phá vỡ thì sao?_ Giống như bất kỳ lỗ hổng bảo mật nào khác,
nó sẽ được sửa trong một bản phát hành bảo mật cho tất cả các phiên bản Go được hỗ trợ.
Tất cả các ứng dụng cần được chuẩn bị để áp dụng các bản vá bảo mật để hoạt động an toàn.
Trong lịch sử, các cipher suite bị phá vỡ ngày càng hiếm.

_Tại sao vẫn để các cipher suite TLS 1.0-1.2 có thể cấu hình?_ Có một
sự đánh đổi có ý nghĩa giữa bảo mật _cơ bản_ và tương thích kế thừa
trong việc chọn cipher suite nào để bật,
và đó là lựa chọn chúng tôi không thể tự thực hiện mà không cắt đứt
một phần hệ sinh thái không thể chấp nhận,
hoặc giảm đảm bảo bảo mật của người dùng hiện đại.

_Tại sao không làm cho cipher suite TLS 1.3 có thể cấu hình?_ Ngược lại,
không có sự đánh đổi nào với TLS 1.3,
vì tất cả các cipher suite của nó đều cung cấp bảo mật mạnh.
Điều này cho phép chúng tôi giữ tất cả chúng được bật và chọn cái nhanh nhất dựa trên các đặc điểm cụ thể
của kết nối mà không cần sự tham gia của lập trình viên.


## Điểm chính cần ghi nhớ

Bắt đầu từ Go 1.17, `crypto/tls` đang tiếp quản thứ tự trong đó các cipher suite có sẵn
được chọn.
Với phiên bản Go được cập nhật thường xuyên, điều này an toàn hơn so với việc để các client có thể
lỗi thời chọn thứ tự,
cho phép chúng tôi tối ưu hóa hiệu năng, và nó giải phóng sự phức tạp đáng kể từ các lập trình viên Go.

Điều này nhất quán với triết lý chung của chúng tôi về việc đưa ra các quyết định mật mã bất cứ khi nào có thể,
thay vì ủy thác chúng cho lập trình viên,
và với [các nguyên tắc mật mã của chúng tôi](/design/cryptography-principles).
Hy vọng các thư viện TLS khác sẽ áp dụng các thay đổi tương tự,
biến việc cấu hình cipher suite tinh tế thành điều của quá khứ.
