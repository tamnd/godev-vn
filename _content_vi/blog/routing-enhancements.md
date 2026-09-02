---
title: Cải tiến routing trong Go 1.22
date: 2024-02-13
by:
- Jonathan Amsterdam, on behalf of the Go team
summary: Các bổ sung cho pattern HTTP route trong Go 1.22.
---

Go 1.22 mang đến hai cải tiến cho router của package `net/http`: khớp method
và wildcard. Các tính năng này cho phép bạn biểu diễn các route phổ biến dưới dạng
pattern thay vì code Go. Mặc dù chúng đơn giản để giải thích và sử dụng,
việc đưa ra các quy tắc đúng để chọn pattern chiến thắng khi có nhiều pattern khớp với một request là thách thức.

Chúng tôi thực hiện các thay đổi này như một phần trong nỗ lực liên tục để làm cho Go trở thành
ngôn ngữ tuyệt vời để xây dựng các hệ thống production. Chúng tôi đã nghiên cứu nhiều web framework
của bên thứ ba, trích xuất những gì chúng tôi cảm thấy là các tính năng được sử dụng nhiều nhất, và tích hợp
chúng vào `net/http`. Sau đó chúng tôi xác nhận lựa chọn và cải thiện thiết kế bằng cách
hợp tác với cộng đồng trong một [GitHub discussion](
https://github.com/golang/go/discussions/60227) và một [proposal issue](/issue/61410).
Thêm các tính năng này vào thư viện chuẩn có nghĩa là ít dependency hơn cho
nhiều dự án. Nhưng các web framework bên thứ ba vẫn là lựa chọn tốt cho người dùng hiện tại
hoặc các chương trình có nhu cầu routing nâng cao.

## Các cải tiến

Các tính năng routing mới hầu như chỉ ảnh hưởng đến chuỗi pattern được truyền
vào hai phương thức `net/http.ServeMux` là `Handle` và `HandleFunc`, và
các hàm cấp cao nhất tương ứng `http.Handle` và `http.HandleFunc`. Thay đổi API
duy nhất là hai phương thức mới trên `net/http.Request` để làm việc với các kết quả khớp wildcard.

Chúng ta sẽ minh họa các thay đổi với một blog server giả định trong đó mỗi bài đăng
có một định danh số nguyên. Request như `GET /posts/234` lấy bài đăng có
ID 234. Trước Go 1.22, code xử lý các request đó sẽ bắt đầu bằng một
dòng như thế này:

    http.HandleFunc("/posts/", handlePost)

Dấu gạch chéo ở cuối định tuyến tất cả các request bắt đầu bằng `/posts/` đến hàm `handlePost`,
hàm này sẽ phải kiểm tra method HTTP là GET, trích xuất
định danh, và lấy bài đăng. Vì việc kiểm tra method không thực sự cần thiết
để thỏa mãn request, sẽ là lỗi tự nhiên khi bỏ qua nó. Điều đó
có nghĩa là một request như `DELETE /posts/234` sẽ lấy bài đăng, điều này
ít nhất là đáng ngạc nhiên.

Trong Go 1.22, code hiện có sẽ tiếp tục hoạt động, hoặc bạn có thể viết thế này:

    http.HandleFunc("GET /posts/{id}", handlePost2)

Pattern này khớp với request GET có đường dẫn bắt đầu bằng "/posts/" và có hai
segment. (Theo trường hợp đặc biệt, GET cũng khớp với HEAD; tất cả các method khác khớp chính xác.)
Hàm `handlePost2` không cần kiểm tra method nữa, và
việc trích xuất chuỗi định danh có thể được viết bằng phương thức `PathValue` mới
trên `Request`:

    idString := req.PathValue("id")

Phần còn lại của `handlePost2` sẽ hoạt động giống như `handlePost`, chuyển đổi chuỗi
định danh thành số nguyên và lấy bài đăng.

Các request như `DELETE /posts/234` sẽ thất bại nếu không có pattern khớp nào khác được
đăng ký. Theo [ngữ nghĩa HTTP](
https://httpwg.org/specs/rfc9110.html#status.405), server `net/http` sẽ trả lời
request đó với lỗi `405 Method Not Allowed` liệt kê các method có sẵn
trong header `Allow`.

Một wildcard có thể khớp toàn bộ một segment, như `{id}` trong ví dụ trên, hoặc nếu
nó kết thúc bằng `...` nó có thể khớp tất cả các segment còn lại của đường dẫn, như trong
pattern `/files/{pathname...}`.

Có thêm một chút cú pháp cuối cùng. Như chúng ta đã chỉ ở trên, các pattern kết thúc bằng dấu gạch chéo,
như `/posts/`, khớp với tất cả các đường dẫn bắt đầu bằng chuỗi đó. Để chỉ khớp
đường dẫn có dấu gạch chéo cuối, bạn có thể viết `/posts/{$}`. Điều đó sẽ khớp
`/posts/` nhưng không khớp `/posts` hoặc `/posts/234`.

Và có thêm một chút API cuối cùng: `net/http.Request` có phương thức `SetPathValue`
để các router bên ngoài thư viện chuẩn có thể làm cho kết quả phân tích đường dẫn của chúng
có sẵn qua `Request.PathValue`.

## Ưu tiên

Mọi HTTP router đều phải xử lý các pattern chồng lấp, như `/posts/{id}` và
`/posts/latest`. Cả hai pattern này đều khớp với đường dẫn "posts/latest", nhưng nhiều nhất
một cái có thể phục vụ request. Pattern nào được ưu tiên?

Một số router không cho phép chồng lấp; những router khác sử dụng pattern được đăng ký sau cùng.
Go luôn cho phép chồng lấp, và đã chọn pattern dài hơn bất kể
thứ tự đăng ký. Bảo toàn tính độc lập về thứ tự rất quan trọng với chúng tôi (và
cần thiết để tương thích ngược), nhưng chúng tôi cần quy tắc tốt hơn
"dài nhất thắng". Quy tắc đó sẽ chọn `/posts/latest` hơn `/posts/{id}`, nhưng
sẽ chọn `/posts/{identifier}` hơn cả hai. Điều đó có vẻ sai: tên wildcard không
quan trọng. Cảm giác như `/posts/latest` luôn nên thắng cuộc cạnh tranh này,
vì nó khớp một đường dẫn duy nhất thay vì nhiều đường dẫn.

Hành trình tìm kiếm quy tắc ưu tiên tốt đã khiến chúng tôi xem xét nhiều thuộc tính của
các pattern. Ví dụ, chúng tôi đã xem xét ưu tiên pattern có tiền tố literal (không phải wildcard)
dài nhất. Điều đó sẽ chọn `/posts/latest` hơn `/posts/{id}`. Nhưng nó sẽ không phân biệt
giữa `/users/{u}/posts/latest` và `/users/{u}/posts/{id}`, và có vẻ như cái trước
nên được ưu tiên.

Cuối cùng chúng tôi đã chọn một quy tắc dựa trên ý nghĩa của các pattern thay vì cách chúng
trông. Mỗi pattern hợp lệ khớp với một tập hợp các request. Ví dụ,
`/posts/latest` khớp với các request có đường dẫn `/posts/latest`, trong khi `/posts/{id}`
khớp với các request có bất kỳ đường dẫn hai segment nào mà segment đầu tiên là "posts". Chúng ta
nói rằng một pattern _cụ thể hơn_ một pattern khác nếu nó khớp với một tập hợp con chặt chẽ
của các request. Pattern `/posts/latest` cụ thể hơn `/posts/{id}`
vì cái sau khớp với mọi request mà cái trước khớp, và nhiều hơn.

Quy tắc ưu tiên đơn giản: pattern cụ thể nhất thắng. Quy tắc này
khớp với trực giác của chúng ta rằng `posts/latest` nên được ưu tiên hơn `posts/{id}`,
và `/users/{u}/posts/latest` nên được ưu tiên hơn `/users/{u}/posts/{id}`.
Nó cũng có ý nghĩa với các method. Ví dụ, `GET /posts/{id}` được
ưu tiên hơn `/posts/{id}` vì cái đầu chỉ khớp với các request GET và HEAD,
trong khi cái thứ hai khớp với các request với bất kỳ method nào.

Quy tắc "cụ thể nhất thắng" tổng quát hóa quy tắc "dài nhất thắng" ban đầu cho
các phần đường dẫn của các pattern ban đầu, những pattern không có wildcard hoặc `{$}`. Các
pattern như vậy chỉ chồng lấp khi một cái là tiền tố của cái kia, và cái dài hơn là
cụ thể hơn.

Nếu hai pattern chồng lấp nhau nhưng không cái nào cụ thể hơn thì sao? Ví dụ, `/posts/{id}`
và `/{resource}/latest` đều khớp với `/posts/latest`. Không có câu trả lời rõ ràng về
cái nào được ưu tiên, vì vậy chúng tôi coi các pattern này là xung đột với nhau.
Đăng ký cả hai (theo thứ tự nào!) sẽ gây panic.

Quy tắc ưu tiên hoạt động chính xác như trên đối với các method và đường dẫn, nhưng chúng tôi phải
thực hiện một ngoại lệ cho host để bảo toàn tính tương thích: nếu hai pattern sẽ
xung đột với nhau và một cái có host trong khi cái kia không có, thì pattern
có host được ưu tiên.

Sinh viên khoa học máy tính có thể nhớ lý thuyết đẹp đẽ về biểu thức chính quy
và ngôn ngữ chính quy. Mỗi biểu thức chính quy chọn ra một ngôn ngữ chính quy,
tập hợp các chuỗi được khớp bởi biểu thức. Một số câu hỏi dễ đặt ra và trả lời hơn
khi nói về ngôn ngữ thay vì biểu thức.
Quy tắc ưu tiên của chúng tôi được lấy cảm hứng từ lý thuyết này. Thực sự, mỗi routing pattern
tương ứng với một biểu thức chính quy, và các tập hợp request khớp đóng vai trò là
các ngôn ngữ chính quy.

Định nghĩa ưu tiên bằng ngôn ngữ thay vì biểu thức giúp dễ dàng phát biểu
và hiểu. Nhưng có một nhược điểm của việc có quy tắc dựa trên các tập hợp tiềm năng
vô hạn: không rõ cách triển khai nó hiệu quả. Hóa ra chúng ta
có thể xác định xem hai pattern có xung đột không bằng cách duyệt từng segment.
Đại khái, nếu một pattern có segment literal ở mọi nơi pattern kia có
wildcard, thì nó cụ thể hơn; nhưng nếu literal căn chỉnh với wildcard theo cả hai
chiều, các pattern xung đột.

Khi các pattern mới được đăng ký trên `ServeMux`, nó kiểm tra xung đột với các pattern đã đăng ký trước.
Nhưng kiểm tra mọi cặp pattern sẽ mất thời gian bình phương. Chúng tôi sử dụng một index để bỏ qua các pattern
không thể xung đột với pattern mới; trong thực tế, nó hoạt động khá tốt. Trong mọi trường hợp, kiểm tra này
xảy ra khi các pattern được đăng ký, thường là lúc khởi động server. Thời gian khớp các request đến
trong Go 1.22 không thay đổi nhiều so với các phiên bản trước.

## Tương thích

Chúng tôi đã nỗ lực hết sức để giữ chức năng mới tương thích với các
phiên bản cũ của Go. Cú pháp pattern mới là tập hợp cha của cũ, và quy tắc
ưu tiên mới tổng quát hóa quy tắc cũ. Nhưng có một vài trường hợp biên. Ví dụ,
các phiên bản Go trước chấp nhận các pattern có dấu ngoặc nhọn và xử lý
chúng theo nghĩa đen, nhưng Go 1.22 sử dụng dấu ngoặc nhọn cho wildcard. Cài đặt GODEBUG
`httpmuxgo121` khôi phục hành vi cũ.

Để biết thêm chi tiết về các cải tiến routing này, xem [tài liệu `net/http.ServeMux`
](/pkg/net/http#ServeMux).
