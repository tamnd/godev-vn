---
title: Go, Mã nguồn mở, Cộng đồng
date: 2015-07-08
by:
- Russ Cox
tags:
- community
summary: Tại sao Go là mã nguồn mở, và làm thế nào chúng ta có thể củng cố cộng đồng mã nguồn mở của mình?
---

## Chào mừng

[Đây là nội dung bài phát biểu khai mạc của tôi tại Gophercon 2015.
[Video có sẵn tại đây](https://www.youtube.com/watch?v=XvZOdpd_9tc).]

Cảm ơn tất cả mọi người đã đến Denver để có mặt ở đây,
và cảm ơn tất cả những người đang xem qua video.
Nếu đây là Gophercon đầu tiên của bạn, chào mừng bạn.
Nếu bạn đã ở đây năm ngoái, chào mừng trở lại.
Cảm ơn các nhà tổ chức
vì tất cả công sức
để tổ chức một hội nghị như thế này.
Tôi rất vui được ở đây và được nói chuyện với tất cả các bạn.

Tôi là trưởng nhóm kỹ thuật của dự án Go
và nhóm Go tại Google.
Tôi chia sẻ vai trò đó với Rob Pike.
Trong vai trò đó, tôi dành nhiều thời gian suy nghĩ về
dự án mã nguồn mở Go tổng thể,
đặc biệt là cách nó vận hành,
ý nghĩa của việc là mã nguồn mở là gì,
và sự tương tác giữa
các contributor bên trong và bên ngoài Google.
Hôm nay tôi muốn chia sẻ với bạn
cách tôi nhìn nhận dự án Go như một tổng thể
và sau đó dựa vào đó giải thích
cách tôi nhìn thấy dự án mã nguồn mở Go
phát triển.

## Tại sao Go?

Để bắt đầu,
chúng ta phải quay lại từ đầu.
Tại sao chúng tôi bắt đầu làm việc trên Go?

Go là nỗ lực để làm cho các lập trình viên năng suất hơn.
Chúng tôi muốn cải thiện quy trình phát triển phần mềm
tại Google,
nhưng các vấn đề Google gặp phải
không phải là duy nhất với Google.

Có hai mục tiêu bao trùm.

Mục tiêu đầu tiên là tạo ra một ngôn ngữ tốt hơn
để đáp ứng những thách thức của tính đồng thời có khả năng mở rộng.
Tính đồng thời có khả năng mở rộng có nghĩa là
phần mềm xử lý nhiều mối quan tâm đồng thời,
chẳng hạn như phối hợp hàng nghìn máy chủ phía sau
bằng cách gửi và nhận lưu lượng mạng.

Ngày nay, loại phần mềm đó có tên ngắn hơn:
chúng ta gọi nó là phần mềm đám mây.
Có thể nói rằng Go được thiết kế cho đám mây
trước khi các đám mây chạy phần mềm.

Mục tiêu lớn hơn là tạo ra một môi trường tốt hơn
để đáp ứng những thách thức của phát triển phần mềm có khả năng mở rộng,
phần mềm được làm việc và sử dụng bởi nhiều người,
với sự phối hợp hạn chế giữa họ,
và được duy trì trong nhiều năm.
Tại Google, chúng tôi có hàng nghìn kỹ sư
viết và chia sẻ code của họ với nhau,
cố gắng hoàn thành công việc,
tái sử dụng công việc của người khác nhiều nhất có thể,
và làm việc trong một codebase có lịch sử
kéo dài hơn mười năm.
Các kỹ sư thường làm việc hoặc ít nhất xem xét
code ban đầu được viết bởi người khác,
hoặc code họ đã viết nhiều năm trước,
thường là cùng một điều.

Tình huống bên trong Google
có nhiều điểm chung với
phát triển mã nguồn mở hiện đại, quy mô lớn
như được thực hành trên các trang như GitHub.
Vì điều này,
Go rất phù hợp với các dự án mã nguồn mở,
giúp chúng chấp nhận và quản lý
đóng góp từ một cộng đồng lớn
trong một thời gian dài.

Tôi tin rằng phần lớn thành công của Go được giải thích bởi thực tế rằng
Go rất phù hợp với phần mềm đám mây,
Go rất phù hợp với các dự án mã nguồn mở,
và, thật may mắn, cả hai đều đang
ngày càng phổ biến và quan trọng hơn
trong ngành công nghiệp phần mềm.

Những người khác cũng có những quan sát tương tự.
Đây là hai ví dụ.
Năm ngoái, trên RedMonk.com, Donnie Berkholz
đã viết về
"[Go as the emerging language of cloud infrastructure](http://redmonk.com/dberkholz/2014/03/18/go-the-emerging-language-of-cloud-infrastructure/),"
nhận xét rằng
"các dự án nổi bật của [Go] ... là tập trung vào đám mây hoặc
được tạo ra để xử lý các hệ thống phân tán
hoặc môi trường tạm thời."

Năm nay, trên Texlution.com, tác giả
đã viết một bài có tiêu đề
"[Why Golang is doomed to succeed](https://texlution.com/post/why-go-is-doomed-to-succeed/),"
chỉ ra rằng sự tập trung vào phát triển quy mô lớn này
thậm chí có thể phù hợp hơn với mã nguồn mở
so với bản thân Google: "Sự phù hợp mã nguồn mở này là lý do tại sao tôi nghĩ
bạn sắp thấy ngày càng nhiều Go hơn ..."

## Sự cân bằng của Go

Go hoàn thành những điều đó như thế nào?

Nó làm cho tính đồng thời có khả năng mở rộng
và phát triển phần mềm có khả năng mở rộng dễ dàng hơn như thế nào?

Hầu hết mọi người trả lời câu hỏi này bằng cách nói về
channel và goroutine, và interface, và build nhanh,
và lệnh go, và hỗ trợ công cụ tốt.
Đó đều là những phần quan trọng của câu trả lời,
nhưng tôi nghĩ có một ý tưởng rộng hơn đằng sau chúng.

Tôi nghĩ về ý tưởng đó như sự cân bằng của Go.
Có những mối quan tâm cạnh tranh trong bất kỳ thiết kế phần mềm nào,
và có xu hướng rất tự nhiên để cố gắng giải quyết
tất cả các vấn đề mà bạn thấy trước.
Trong Go, chúng tôi đã cố gắng không giải quyết mọi thứ.
Thay vào đó, chúng tôi đã cố gắng làm đủ để bạn có thể xây dựng
các giải pháp tùy chỉnh của riêng mình một cách dễ dàng.

Cách tôi tóm tắt sự cân bằng đã chọn của Go là: **Làm Ít Hơn. Cho Phép Nhiều Hơn.**

Làm ít hơn, nhưng cho phép nhiều hơn.

Go không thể làm mọi thứ.
Chúng ta không nên cố gắng.
Nhưng nếu chúng ta nỗ lực,
Go có thể làm
một vài thứ tốt.
Nếu chúng ta chọn những thứ đó cẩn thận,
chúng ta có thể đặt nền tảng
mà trên đó các nhà phát triển có thể _dễ dàng_ xây dựng
các giải pháp và công cụ họ cần,
và lý tưởng là có thể tương tác với
các giải pháp và công cụ được xây dựng bởi người khác.

### Ví dụ

Hãy để tôi minh họa điều này với một số ví dụ.

Đầu tiên, kích thước của chính ngôn ngữ Go.
Chúng tôi đã làm việc chăm chỉ để đưa vào càng ít khái niệm càng tốt,
để tránh vấn đề các phương ngữ không thể hiểu lẫn nhau
hình thành ở các phần khác nhau của một cộng đồng nhà phát triển lớn.
Không có ý tưởng nào được đưa vào Go cho đến khi
nó được đơn giản hóa đến bản chất
và sau đó có lợi ích rõ ràng
biện hộ cho độ phức tạp được thêm vào.

Nói chung, nếu chúng ta có 100 thứ
chúng ta muốn Go làm tốt,
chúng ta không thể thực hiện 100 thay đổi riêng lẻ.
Thay vào đó, chúng ta cố gắng nghiên cứu và hiểu
không gian thiết kế
và sau đó xác định một vài thay đổi
hoạt động tốt cùng nhau
và cho phép khoảng 90 trong số những thứ đó.
Chúng ta sẵn sàng hy sinh 10 còn lại
để tránh làm phình ngôn ngữ,
để tránh thêm độ phức tạp
chỉ để giải quyết các trường hợp sử dụng cụ thể
có vẻ quan trọng ngày hôm nay
nhưng có thể biến mất vào ngày mai.

Giữ ngôn ngữ nhỏ
cho phép các mục tiêu quan trọng hơn.
Nhỏ làm cho Go
dễ học hơn,
dễ hiểu hơn,
dễ triển khai hơn,
dễ tái triển khai hơn,
dễ debug hơn,
dễ điều chỉnh hơn,
và dễ phát triển hơn.
Làm ít hơn cho phép nhiều hơn.

Tôi nên chỉ ra rằng
điều này có nghĩa là chúng tôi nói không
với nhiều ý tưởng của người khác,
nhưng tôi đảm bảo với bạn
chúng tôi đã nói không
với thậm chí nhiều ý tưởng của chính mình hơn.

Tiếp theo, channel và goroutine.
Làm thế nào chúng ta nên cấu trúc và phối hợp
các tính toán đồng thời và song song?
Mutex và biến điều kiện rất chung chung
nhưng quá cấp thấp đến mức khó sử dụng đúng.
Các framework thực thi song song như OpenMP quá cấp cao
đến mức chúng chỉ có thể được sử dụng để giải quyết một phạm vi hẹp các vấn đề.
Channel và goroutine nằm giữa hai cực này.
Bản thân chúng không phải là giải pháp cho nhiều thứ.
Nhưng chúng đủ mạnh để dễ dàng sắp xếp
để cho phép giải pháp cho nhiều vấn đề phổ biến
trong phần mềm đồng thời.
Làm ít hơn, thực sự làm vừa đủ, cho phép nhiều hơn.

Tiếp theo, kiểu và interface.
Có kiểu tĩnh cho phép kiểm tra tại thời điểm biên dịch hữu ích,
điều mà các ngôn ngữ kiểu động thiếu
như Python hoặc Ruby.
Đồng thời,
kiểu tĩnh của Go tránh được
phần lớn sự lặp lại
của các ngôn ngữ kiểu tĩnh truyền thống,
làm cho nó cảm thấy nhẹ nhàng hơn,
giống các ngôn ngữ kiểu động hơn.
Đây là một trong những điều đầu tiên người ta nhận thấy,
và nhiều người dùng sớm của Go đến từ
các ngôn ngữ kiểu động.

Interface của Go là một phần quan trọng của điều đó.
Đặc biệt,
bỏ qua các khai báo ``implements''
của Java hoặc các ngôn ngữ khác có phân cấp tĩnh
làm cho interface nhẹ hơn và linh hoạt hơn.
Không có phân cấp cứng nhắc đó
cho phép các idiom như interface kiểm tra mô tả
các triển khai production hiện có, không liên quan.
Làm ít hơn cho phép nhiều hơn.

Tiếp theo, kiểm thử và benchmark.
Có sự thiếu hụt nào về các framework kiểm thử
và benchmark trong hầu hết các ngôn ngữ không?
Có sự đồng thuận nào giữa chúng không?

Package testing của Go không nhằm mục đích
giải quyết mọi khía cạnh có thể của các chủ đề này.
Thay vào đó, nó nhằm cung cấp
các khái niệm cơ bản cần thiết
cho hầu hết các công cụ cấp cao hơn.
Các gói có các test case pass, fail, hoặc bị bỏ qua.
Các gói có benchmark chạy và có thể được đo lường
bằng các số liệu khác nhau.

Làm ít hơn ở đây là nỗ lực
để rút gọn các khái niệm này về bản chất của chúng,
để tạo ra một từ vựng chung
để các công cụ phong phú hơn có thể tương tác.
Sự đồng thuận đó cho phép phần mềm kiểm thử cấp cao hơn
như trình chuyển đổi go2xunit của Miki Tebeka,
hoặc các công cụ phân tích benchmark benchcmp và benchstat.

Vì _có_ sự đồng thuận
về biểu diễn các khái niệm cơ bản,
các công cụ cấp cao hơn này hoạt động cho tất cả các gói Go,
không chỉ những gói có nỗ lực để tham gia,
và chúng tương tác với nhau,
theo nghĩa sử dụng, chẳng hạn, go2xunit
không ngăn cản việc sử dụng benchstat,
theo cách nó sẽ xảy ra nếu các công cụ này là, chẳng hạn,
plugin cho các framework kiểm thử cạnh tranh.
Làm ít hơn cho phép nhiều hơn.

Tiếp theo, tái cấu trúc và phân tích chương trình.
Vì Go dành cho các codebase lớn,
chúng tôi biết nó sẽ cần hỗ trợ tự động
bảo trì và cập nhật source code.
Chúng tôi cũng biết rằng chủ đề này quá lớn
để xây dựng trực tiếp vào.
Nhưng chúng tôi biết một điều chúng tôi phải làm.
Trong kinh nghiệm của chúng tôi khi cố gắng
thay đổi chương trình tự động trong các môi trường khác,
rào cản quan trọng nhất chúng tôi gặp phải
thực sự là việc viết chương trình đã sửa đổi ra
theo định dạng mà các nhà phát triển có thể chấp nhận.

Trong các ngôn ngữ khác,
thường thấy các nhóm khác nhau sử dụng
các quy ước định dạng khác nhau.
Nếu một chỉnh sửa bởi chương trình sử dụng quy ước sai,
nó hoặc viết một phần của tệp nguồn trông không giống gì
so với phần còn lại của tệp, hoặc nó định dạng lại toàn bộ tệp,
gây ra các diff không cần thiết và không mong muốn.

Go không có vấn đề này.
Chúng tôi đã thiết kế ngôn ngữ để làm cho gofmt có thể thực hiện được,
chúng tôi đã làm việc chăm chỉ
để làm cho định dạng của gofmt được chấp nhận
cho tất cả các chương trình Go,
và chúng tôi đảm bảo gofmt đã có ở đó
từ ngày đầu tiên của bản phát hành công khai ban đầu.
Gofmt áp đặt sự thống nhất đến mức
các thay đổi tự động hòa vào phần còn lại của tệp.
Bạn không thể biết liệu một thay đổi cụ thể
được thực hiện bởi người hay máy tính.
Chúng tôi không xây dựng hỗ trợ tái cấu trúc rõ ràng.
Thiết lập một thuật toán định dạng được thống nhất
là đủ một nền tảng chung
cho các công cụ độc lập phát triển và tương tác.
Gofmt đã cho phép gofix, goimports, eg, và các công cụ khác.
Tôi tin rằng công việc ở đây chỉ mới bắt đầu.
Còn có thể làm được nhiều hơn nữa.

Cuối cùng, xây dựng và chia sẻ phần mềm.
Trong quá trình chuẩn bị cho Go 1, chúng tôi đã xây dựng goinstall,
trở thành thứ mà tất cả chúng ta biết là "go get".
Công cụ đó xác định một cách tiêu chuẩn không cần cấu hình
để giải quyết các đường dẫn import trên các trang như github.com,
và sau đó là cách giải quyết đường dẫn trên các trang khác
bằng cách thực hiện các yêu cầu HTTP.
Thuật toán giải quyết được thống nhất này
đã cho phép các công cụ khác hoạt động theo các đường dẫn đó,
nổi bật nhất là trang godoc.org được tạo bởi Gary Burd.
Trong trường hợp bạn chưa sử dụng nó,
bạn truy cập godoc.org/the-import-path
cho bất kỳ đường dẫn import "go get" hợp lệ nào,
và trang web sẽ tải code
và hiển thị tài liệu cho nó.
Một tác dụng phụ tốt là
godoc.org phục vụ như một danh sách chính thô
các gói Go công khai có sẵn.
Tất cả những gì chúng tôi đã làm là cho các đường dẫn import có ý nghĩa rõ ràng.
Làm ít hơn, cho phép nhiều hơn.

Bạn sẽ nhận thấy rằng nhiều ví dụ công cụ này
là về việc thiết lập một quy ước chung.
Đôi khi mọi người gọi đây là Go "có chủ kiến,"
nhưng có điều gì đó sâu hơn đang diễn ra.
Đồng ý với những giới hạn
của một quy ước chung
là cách để cho phép
một lớp công cụ rộng tương tác với nhau,
vì chúng đều nói cùng một ngôn ngữ cơ sở.
Đây là cách rất hiệu quả
để làm ít hơn nhưng cho phép nhiều hơn.
Cụ thể, trong nhiều trường hợp
chúng ta có thể làm tối thiểu cần thiết
để thiết lập sự hiểu biết chung
về một khái niệm cụ thể, như import từ xa,
hoặc định dạng đúng của tệp nguồn,
và từ đó cho phép
việc tạo ra các gói và công cụ
hoạt động cùng nhau
vì chúng đều đồng ý
về các chi tiết cốt lõi đó.

Tôi sẽ quay lại ý tưởng đó sau.

## Tại sao Go là mã nguồn mở?

Nhưng trước tiên, như tôi đã nói trước đó,
tôi muốn giải thích cách tôi nhìn nhận
sự cân bằng Làm Ít Hơn và Cho Phép Nhiều Hơn
hướng dẫn công việc của chúng tôi
trên dự án
mã nguồn mở Go rộng lớn hơn.
Để làm điều đó, tôi cần bắt đầu với
lý do tại sao Go là mã nguồn mở.

Google trả tiền cho tôi và những người khác để làm việc trên Go, vì,
nếu các lập trình viên của Google năng suất hơn,
Google có thể xây dựng sản phẩm nhanh hơn,
duy trì chúng dễ dàng hơn,
và vân vân.
Nhưng tại sao lại mã nguồn mở Go?
Tại sao Google nên chia sẻ lợi ích này với thế giới?

Tất nhiên, nhiều người trong chúng ta
đã làm việc trên các dự án mã nguồn mở trước Go,
và chúng ta tự nhiên muốn Go
là một phần của thế giới mã nguồn mở đó.
Nhưng các ưu tiên của chúng ta không phải là lý do kinh doanh.
Lý do kinh doanh là
Go là mã nguồn mở
vì đó là cách duy nhất
Go có thể thành công.
Chúng tôi, nhóm đã xây dựng Go trong Google,
biết điều này từ ngày đầu tiên.
Chúng tôi biết rằng Go phải được cung cấp
cho càng nhiều người càng tốt
để thành công.

Các ngôn ngữ đóng chết.

Một ngôn ngữ cần cộng đồng lớn, rộng rãi.

Một ngôn ngữ cần nhiều người viết nhiều phần mềm,
để khi bạn cần một công cụ hoặc thư viện cụ thể,
có khả năng cao là nó đã được viết,
bởi ai đó hiểu chủ đề hơn bạn,
và dành nhiều thời gian hơn bạn để làm cho nó tuyệt vời.

Một ngôn ngữ cần nhiều người báo cáo lỗi,
để các vấn đề được xác định và sửa nhanh chóng.
Vì cơ sở người dùng lớn hơn nhiều,
các trình biên dịch Go mạnh mẽ hơn nhiều và tuân thủ spec hơn
so với các trình biên dịch Plan 9 C mà chúng dựa trên.

Một ngôn ngữ cần nhiều người sử dụng nó
cho nhiều mục đích khác nhau,
để ngôn ngữ không quá phù hợp với một trường hợp sử dụng
và trở nên vô dụng khi bối cảnh công nghệ thay đổi.

Một ngôn ngữ cần nhiều người muốn học nó,
để có thị trường cho người viết sách
hoặc dạy các khóa học,
hoặc tổ chức các hội nghị như thế này.

Không có điều nào trong số này có thể xảy ra
nếu Go ở lại trong Google.
Go sẽ ngạt thở bên trong Google,
hoặc bên trong bất kỳ công ty đơn lẻ nào
hoặc môi trường đóng.

Về cơ bản,
Go phải mở,
và Go cần bạn.
Go không thể thành công nếu không có tất cả các bạn,
không có tất cả những người sử dụng Go
cho tất cả các loại dự án khác nhau
trên khắp thế giới.

Đổi lại, nhóm Go tại Google
không bao giờ có thể đủ lớn
để hỗ trợ toàn bộ cộng đồng Go.
Để tiếp tục mở rộng quy mô,
chúng ta
cần cho phép tất cả "nhiều hơn" này
trong khi làm ít hơn.
Mã nguồn mở là một phần lớn của điều đó.

## Mã nguồn mở của Go

Mã nguồn mở có nghĩa là gì?
Yêu cầu tối thiểu là mở source code,
làm cho nó có sẵn dưới giấy phép mã nguồn mở,
và chúng tôi đã làm điều đó.

Nhưng chúng tôi cũng mở quy trình phát triển của mình:
kể từ khi công bố Go,
chúng tôi đã thực hiện tất cả phát triển của mình công khai,
trên các danh sách thư công khai mở cho tất cả mọi người.
Chúng tôi chấp nhận và xem xét
đóng góp source code từ bất kỳ ai.
Quy trình là như nhau
dù bạn có làm việc cho Google hay không.
Chúng tôi duy trì trình theo dõi lỗi của mình công khai,
chúng tôi thảo luận và phát triển các đề xuất thay đổi công khai,
và chúng tôi làm việc hướng tới các bản phát hành công khai.
Cây source code công khai là bản sao có thẩm quyền.
Các thay đổi xảy ra ở đó trước.
Chúng chỉ được đưa vào
cây source code nội bộ của Google sau đó.
Đối với Go, là mã nguồn mở có nghĩa là
đây là nỗ lực tập thể
mở rộng ra ngoài Google, mở cho tất cả.

Bất kỳ dự án mã nguồn mở nào cũng bắt đầu với một vài người,
thường chỉ một, nhưng với Go đó là ba:
Robert Griesemer, Rob Pike, và Ken Thompson.
Họ có tầm nhìn về
những gì họ muốn Go trở thành,
những gì họ nghĩ Go có thể làm tốt hơn
các ngôn ngữ hiện có, và
Robert sẽ nói thêm về điều đó vào sáng mai.
Tôi là người tiếp theo tham gia nhóm,
rồi Ian Taylor,
và rồi, từng người một,
chúng tôi đã kết thúc ở đây ngày hôm nay,
với hàng trăm contributor.

Cảm ơn
nhiều người đã đóng góp
code
hoặc ý tưởng
hoặc báo cáo lỗi
cho dự án Go cho đến nay.
Chúng tôi đã cố gắng liệt kê mọi người chúng tôi có thể
trong không gian của chúng tôi trong chương trình hôm nay.
Nếu tên bạn không ở đó,
tôi xin lỗi,
nhưng cảm ơn bạn.

Tôi tin rằng
hàng trăm contributor cho đến nay
đang làm việc hướng tới tầm nhìn chung
về những gì Go có thể trở thành.
Thật khó để diễn đạt những điều này bằng lời,
nhưng tôi đã cố hết sức
để giải thích một phần tầm nhìn
trước đó:
Làm Ít Hơn, Cho Phép Nhiều Hơn.

## Vai trò của Google

Một câu hỏi tự nhiên là:
Vai trò của nhóm Go tại Google là gì,
so với các contributor khác?
Tôi tin rằng vai trò đó
đã thay đổi theo thời gian,
và nó tiếp tục thay đổi.
Xu hướng chung là
theo thời gian
nhóm Go tại Google
nên làm ít hơn
và cho phép nhiều hơn.

Trong những ngày đầu tiên,
trước khi Go được biết đến công khai,
nhóm Go tại Google
rõ ràng là đang làm việc một mình.
Chúng tôi đã viết bản nháp đầu tiên của mọi thứ:
đặc tả,
trình biên dịch,
runtime,
thư viện chuẩn.

Nhưng khi Go được mã nguồn mở,
vai trò của chúng tôi bắt đầu thay đổi.
Điều quan trọng nhất
chúng tôi cần làm
là truyền đạt tầm nhìn của chúng tôi cho Go.
Điều đó khó khăn,
và chúng tôi vẫn đang làm việc với nó.
Việc triển khai ban đầu
là một cách quan trọng
để truyền đạt tầm nhìn đó,
cũng như công việc phát triển chúng tôi dẫn dắt
dẫn đến Go 1,
và các bài đăng blog khác nhau,
và bài báo,
và các bài nói chuyện chúng tôi đã xuất bản.

Nhưng như Rob đã nói tại Gophercon năm ngoái,
"ngôn ngữ đã xong."
Bây giờ chúng ta cần xem nó hoạt động như thế nào,
xem mọi người sử dụng nó như thế nào,
xem mọi người xây dựng gì.
Trọng tâm bây giờ là
mở rộng loại công việc
mà Go có thể hỗ trợ.

Vai trò chính của Google bây giờ là
cho phép cộng đồng,
phối hợp,
đảm bảo các thay đổi hoạt động tốt cùng nhau,
và giữ Go trung thành với tầm nhìn ban đầu.

Vai trò chính của Google là:
Làm Ít Hơn. Cho Phép Nhiều Hơn.

Tôi đã đề cập trước đó
rằng chúng tôi thà có một số lượng nhỏ tính năng
cho phép, chẳng hạn, 90% các trường hợp sử dụng mục tiêu,
và tránh số lượng
tính năng lớn hơn theo cấp số nhân cần thiết
để đạt 99 hoặc 100%.
Chúng tôi đã thành công trong việc áp dụng chiến lược đó
cho các lĩnh vực phần mềm mà chúng tôi biết rõ.
Nhưng nếu Go muốn trở nên hữu ích trong nhiều lĩnh vực mới,
chúng ta cần các chuyên gia trong những lĩnh vực đó
mang chuyên môn của họ
vào các cuộc thảo luận của chúng ta,
để cùng nhau
chúng ta có thể thiết kế các điều chỉnh nhỏ
cho phép nhiều ứng dụng mới cho Go.

Sự chuyển dịch này áp dụng không chỉ cho thiết kế
mà còn cho phát triển.
Vai trò của nhóm Go tại Google
tiếp tục chuyển dịch
nhiều hơn sang hướng dẫn
và ít hơn là phát triển thuần túy.
Tôi chắc chắn dành nhiều thời gian hơn
làm review code hơn là viết code,
nhiều thời gian hơn xử lý báo cáo lỗi
hơn là tự nộp báo cáo lỗi.
Chúng ta cần làm ít hơn và cho phép nhiều hơn.

Khi thiết kế và phát triển chuyển dịch
sang cộng đồng Go rộng hơn,
một trong những điều quan trọng nhất
chúng tôi
các tác giả gốc của Go
có thể cung cấp
là sự nhất quán của tầm nhìn,
để giúp giữ Go
là Go.
Sự cân bằng mà chúng ta phải đạt được
chắc chắn là chủ quan.
Ví dụ, một cơ chế cho cú pháp có thể mở rộng
sẽ là cách để
cho phép nhiều hơn
cách viết code Go,
nhưng điều đó sẽ đi ngược lại mục tiêu của chúng ta
là có một ngôn ngữ nhất quán
không có các phương ngữ khác nhau.

Chúng ta phải đôi khi nói không,
có lẽ nhiều hơn trong các cộng đồng ngôn ngữ khác,
nhưng khi chúng ta làm vậy,
chúng ta hướng đến làm vậy
một cách xây dựng và tôn trọng,
lấy đó làm cơ hội
để làm rõ tầm nhìn cho Go.

Tất nhiên, không phải tất cả đều là phối hợp và tầm nhìn.
Google vẫn tài trợ cho công việc phát triển Go.
Rick Hudson sẽ nói sau hôm nay
về công việc của anh ấy trong việc giảm độ trễ bộ gom rác,
và Hana Kim sẽ nói vào ngày mai
về công việc của cô ấy trong việc đưa Go đến các thiết bị di động.
Nhưng tôi muốn làm rõ rằng,
nhiều nhất có thể,
chúng tôi nhằm mục đích đối xử
với phát triển được tài trợ bởi Google
như nhau với
phát triển được tài trợ bởi các công ty khác
hoặc được đóng góp bởi các cá nhân sử dụng thời gian rảnh của họ.
Chúng tôi làm điều này vì chúng tôi không biết
ý tưởng tuyệt vời tiếp theo sẽ đến từ đâu.
Mọi contributor cho Go
nên có cơ hội được lắng nghe.

### Ví dụ

Tôi muốn chia sẻ một số bằng chứng cho tuyên bố này
rằng, theo thời gian,
nhóm Go gốc tại Google
đang tập trung nhiều hơn vào
phối hợp hơn là phát triển trực tiếp.

Đầu tiên, nguồn tài trợ
cho phát triển Go đang mở rộng.
Trước khi phát hành mã nguồn mở,
rõ ràng Google đã trả cho tất cả phát triển Go.
Sau khi phát hành mã nguồn mở,
nhiều cá nhân bắt đầu đóng góp thời gian của họ,
và chúng tôi đã từ từ nhưng ổn định
tăng số lượng contributor
được hỗ trợ bởi các công ty khác
để làm việc trên Go ít nhất bán thời gian,
đặc biệt là liên quan đến
làm cho Go hữu ích hơn cho các công ty đó.
Ngày nay, danh sách đó bao gồm
Canonical, Dropbox, Intel, Oracle, và các công ty khác.
Và tất nhiên Gophercon và các
hội nghị Go khu vực khác được tổ chức
hoàn toàn bởi người bên ngoài Google,
và họ có nhiều nhà tài trợ doanh nghiệp
ngoài Google.

Thứ hai, chiều sâu khái niệm
của phát triển Go
được thực hiện bên ngoài nhóm gốc
đang mở rộng.

Ngay sau khi phát hành mã nguồn mở,
một trong những đóng góp lớn đầu tiên
là port sang Microsoft Windows,
được bắt đầu bởi Hector Chu
và hoàn thành bởi Alex Brainman và những người khác.
Nhiều contributor đã port Go
sang các hệ điều hành khác.
Thậm chí nhiều contributor hơn
đã viết lại hầu hết code số học của chúng tôi
để nhanh hơn hoặc chính xác hơn hoặc cả hai.
Đây là tất cả những đóng góp quan trọng,
và rất được đánh giá cao,
nhưng
hầu hết chúng
không liên quan đến các thiết kế mới.

Gần đây hơn,
một nhóm contributor do Aram Hăvărneanu dẫn dắt
đã port Go sang kiến trúc ARM 64,
Đây là port kiến trúc đầu tiên
bởi các contributor bên ngoài Google.
Điều này có ý nghĩa, vì
nói chung
hỗ trợ cho một kiến trúc mới
đòi hỏi nhiều công việc thiết kế hơn
so với hỗ trợ cho một hệ điều hành mới.
Có nhiều biến thể hơn giữa các kiến trúc
so với giữa các hệ điều hành.

Một ví dụ khác là việc giới thiệu
qua vài bản phát hành gần đây
hỗ trợ sơ bộ
cho việc xây dựng các chương trình Go bằng thư viện chia sẻ.
Tính năng này quan trọng cho nhiều bản phân phối Linux
nhưng không quan trọng đối với Google,
vì chúng tôi triển khai các binary tĩnh.
Chúng tôi đã giúp hướng dẫn chiến lược tổng thể,
nhưng hầu hết thiết kế
và gần như tất cả triển khai
đã được thực hiện bởi các contributor bên ngoài Google,
đặc biệt là Michael Hudson-Doyle.

Ví dụ cuối cùng của tôi là cách tiếp cận của lệnh go
đối với vendoring.
Tôi định nghĩa vendoring là
sao chép source code cho các dependency bên ngoài
vào cây của bạn
để đảm bảo rằng chúng không biến mất
hoặc thay đổi bất ngờ.

Vendoring không phải là vấn đề Google gặp phải,
ít nhất không theo cách phần còn lại của thế giới gặp.
Chúng tôi sao chép các thư viện mã nguồn mở chúng tôi muốn sử dụng
vào cây source chung của chúng tôi,
ghi lại phiên bản nào chúng tôi đã sao chép,
và chỉ cập nhật bản sao
khi có nhu cầu làm vậy.
Chúng tôi có quy tắc
rằng chỉ có thể có một phiên bản
của một thư viện cụ thể trong cây source,
và đó là công việc của ai muốn nâng cấp thư viện đó
để đảm bảo nó tiếp tục hoạt động như mong đợi
bởi code Google phụ thuộc vào nó.
Không có điều nào trong số này xảy ra thường xuyên.
Đây là cách tiếp cận lười biếng với vendoring.

Ngược lại, hầu hết các dự án bên ngoài Google
áp dụng cách tiếp cận hăng hái hơn,
import và cập nhật code
bằng cách sử dụng các công cụ tự động
và đảm bảo rằng họ
luôn sử dụng các phiên bản mới nhất.

Vì Google có tương đối ít kinh nghiệm
với vấn đề vendoring này,
chúng tôi để người dùng bên ngoài Google phát triển giải pháp.
Trong năm năm qua,
mọi người đã xây dựng một loạt công cụ.
Những công cụ chính được sử dụng ngày nay là
godep của Keith Rarick,
nut của Owen Ou,
và plugin gb-vendor cho gb của Dave Cheney.

Có hai vấn đề với tình huống hiện tại.
Vấn đề đầu tiên là các công cụ này
không tương thích
ngay lập tức
với lệnh "go get" của lệnh go.
Vấn đề thứ hai là các công cụ
thậm chí không tương thích với nhau.
Cả hai vấn đề này
phân mảnh cộng đồng nhà phát triển theo công cụ.

Mùa thu năm ngoái, chúng tôi bắt đầu một cuộc thảo luận thiết kế công khai
để cố gắng xây dựng đồng thuận về
một số điều cơ bản về
cách tất cả các công cụ này hoạt động,
để chúng có thể làm việc cùng với "go get"
và với nhau.

Đề xuất cơ bản của chúng tôi là tất cả các công cụ đồng ý
về cách tiếp cận viết lại các đường dẫn import trong khi vendoring,
để phù hợp với mô hình của "go get",
và cũng là tất cả các công cụ đồng ý về định dạng tệp
mô tả nguồn và phiên bản của code được sao chép,
để các công cụ vendoring khác nhau
có thể được sử dụng cùng nhau
ngay cả bởi một dự án duy nhất.
Nếu bạn sử dụng một công cụ ngày hôm nay,
bạn vẫn có thể sử dụng công cụ khác vào ngày mai.

Tìm kiếm điểm chung theo cách này
rất phù hợp với tinh thần Làm Ít Hơn, Cho Phép Nhiều Hơn.
Nếu chúng ta có thể xây dựng đồng thuận
về các khía cạnh ngữ nghĩa cơ bản này,
điều đó sẽ cho phép "go get" và tất cả các công cụ này tương tác,
và nó sẽ cho phép chuyển đổi giữa các công cụ,
theo cách tương tự
thỏa thuận về cách các chương trình Go
được lưu trữ trong các tệp văn bản
cho phép trình biên dịch Go và tất cả các trình soạn thảo văn bản tương tác.
Vì vậy chúng tôi đã gửi đề xuất về điểm chung.

Hai điều đã xảy ra.

Đầu tiên, Daniel Theophanes
đã bắt đầu một dự án vendor-spec trên GitHub
với một đề xuất mới
và tiếp quản việc phối hợp và thiết kế
của spec cho metadata vendoring.

Thứ hai, cộng đồng đã lên tiếng
với về cơ bản một giọng nói
để nói rằng
việc viết lại các đường dẫn import trong khi vendoring
là không khả thi.
Vendoring hoạt động trơn tru hơn nhiều
nếu code có thể được sao chép mà không thay đổi.

Keith Rarick đã đăng một đề xuất thay thế
cho một thay đổi tối thiểu đối với lệnh go
để hỗ trợ vendoring mà không viết lại các đường dẫn import.
Đề xuất của Keith không cần cấu hình
và phù hợp tốt với phần còn lại của cách tiếp cận của lệnh go.
Đề xuất đó sẽ được phát hành
như một tính năng thử nghiệm trong Go 1.5
và có khả năng được bật mặc định trong Go 1.6.
Và tôi tin rằng các tác giả công cụ vendoring khác nhau
đã đồng ý áp dụng spec của Daniel khi nó được hoàn thiện.

Kết quả
là tại Gophercon tiếp theo
chúng ta nên có khả năng tương tác rộng rãi
giữa các công cụ vendoring và lệnh go,
và thiết kế để điều đó xảy ra
đã được thực hiện hoàn toàn bởi các contributor
bên ngoài nhóm Go gốc.

Không chỉ vậy,
đề xuất của nhóm Go về cách làm điều này
về cơ bản là hoàn toàn sai.
Cộng đồng Go đã nói với chúng tôi điều đó
rất rõ ràng.
Chúng tôi đã nghe lời khuyên đó,
và bây giờ có một kế hoạch hỗ trợ vendoring
mà tôi tin rằng
mọi người liên quan đều hài lòng.

Đây cũng là ví dụ tốt
về cách tiếp cận chung của chúng tôi với thiết kế.
Chúng tôi cố gắng không thực hiện bất kỳ thay đổi nào đối với Go
cho đến khi chúng tôi cảm thấy có sự đồng thuận rộng rãi
về một giải pháp được hiểu rõ.
Đối với vendoring,
phản hồi và thiết kế
từ cộng đồng Go
rất quan trọng để đạt đến điểm đó.

Xu hướng chung này
hướng tới cả code và thiết kế
đến từ cộng đồng Go rộng hơn
rất quan trọng cho Go.
Bạn, cộng đồng Go rộng hơn,
biết những gì đang hoạt động
và những gì không
trong các môi trường nơi bạn sử dụng Go.
Chúng tôi tại Google thì không.
Ngày càng nhiều,
chúng tôi sẽ dựa vào chuyên môn của bạn,
và chúng tôi sẽ cố gắng giúp bạn phát triển
các thiết kế và code
mở rộng Go để hữu ích trong nhiều cài đặt hơn
và phù hợp tốt với tầm nhìn ban đầu của Go.
Đồng thời,
chúng tôi sẽ tiếp tục chờ đợi
sự đồng thuận rộng rãi
về các giải pháp được hiểu rõ.

Điều này đưa tôi đến điểm cuối cùng.

## Quy tắc ứng xử

Tôi đã lập luận rằng Go phải mở,
và Go cần sự giúp đỡ của bạn.

Nhưng trên thực tế Go cần sự giúp đỡ của mọi người.
Và không phải mọi người đều ở đây.

Go cần ý tưởng từ càng nhiều người càng tốt.

Để làm cho điều đó trở thành thực tế,
cộng đồng Go cần phải
bao gồm,
chào đón,
hữu ích,
và tôn trọng nhất có thể.

Cộng đồng Go đủ lớn để bây giờ,
thay vì giả định rằng mọi người liên quan
đều biết những gì được mong đợi,
tôi và những người khác tin rằng sẽ có ý nghĩa
khi viết những kỳ vọng đó một cách rõ ràng.
Giống như spec Go
đặt kỳ vọng cho tất cả các trình biên dịch Go,
chúng ta có thể viết một spec
đặt kỳ vọng cho hành vi của chúng ta
trong các cuộc thảo luận trực tuyến
và trong các cuộc họp ngoại tuyến như cuộc này.

Như bất kỳ spec tốt nào,
nó phải đủ chung chung
để cho phép nhiều triển khai
nhưng đủ cụ thể
để có thể xác định các vấn đề quan trọng.
Khi hành vi của chúng ta không đáp ứng spec,
mọi người có thể chỉ ra điều đó cho chúng ta,
và chúng ta có thể khắc phục vấn đề.
Đồng thời,
điều quan trọng là phải hiểu rằng
loại spec này
không thể chính xác như một spec ngôn ngữ.
Chúng ta phải bắt đầu với giả định
rằng tất cả chúng ta sẽ hợp lý trong việc áp dụng nó.

Loại spec này
thường được gọi là
Quy tắc ứng xử.
Gophercon có một,
mà tất cả chúng ta đã đồng ý tuân theo
bằng cách có mặt ở đây,
nhưng cộng đồng Go thì không có.
Tôi và những người khác
tin rằng cộng đồng Go
cần một Quy tắc ứng xử.

Nhưng nó nên nói gì?

Tôi tin rằng
tuyên bố tổng thể quan trọng nhất
chúng ta có thể đưa ra là
nếu bạn muốn sử dụng hoặc thảo luận về Go,
thì bạn được chào đón ở đây,
trong cộng đồng của chúng ta.
Đó là tiêu chuẩn
mà tôi tin chúng ta khát vọng hướng tới.

Nếu không có lý do nào khác
(và, để nói rõ, có những lý do tuyệt vời khác),
Go cần cộng đồng càng lớn càng tốt.
Trong chừng mực mà hành vi
giới hạn kích thước của cộng đồng,
nó cản trở Go.
Và hành vi có thể dễ dàng
giới hạn kích thước của cộng đồng.

Cộng đồng công nghệ nói chung
và cộng đồng Go nói riêng
có xu hướng hướng tới những người giao tiếp thẳng thắn.
Tôi không tin điều này là cơ bản.
Tôi không tin điều này là cần thiết.
Nhưng nó đặc biệt dễ dàng làm vậy
trong các cuộc thảo luận trực tuyến như email và IRC,
nơi văn bản thuần túy không được bổ sung
bởi các tín hiệu và dấu hiệu khác chúng ta có
trong các tương tác mặt đối mặt.

Ví dụ, tôi đã học được
rằng khi tôi bị thiếu thời gian
tôi có xu hướng viết ít từ hơn,
với kết quả cuối cùng là
email của tôi không chỉ có vẻ vội vàng
mà còn thẳng thắn, thiếu kiên nhẫn, thậm chí coi thường.
Đó không phải là cảm giác của tôi,
nhưng đó là cách tôi có thể xuất hiện,
và ấn tượng đó có thể đủ
để khiến mọi người suy nghĩ lại
về việc sử dụng hoặc đóng góp
cho Go.
Tôi nhận ra rằng mình đang làm điều này
khi một số contributor Go
đã gửi cho tôi email riêng để cho tôi biết.
Bây giờ, khi tôi bị thiếu thời gian,
tôi chú ý thêm đến những gì tôi đang viết,
và tôi thường viết nhiều hơn tôi tự nhiên sẽ làm,
để đảm bảo
tôi đang gửi đi thông điệp tôi muốn truyền đạt.

Tôi tin rằng
việc sửa chữa những phần
trong các tương tác hàng ngày của chúng ta,
dù có chủ ý hay không,
khiến người dùng và contributor tiềm năng rời đi
là một trong những điều quan trọng nhất
tất cả chúng ta có thể làm
để đảm bảo cộng đồng Go
tiếp tục phát triển.
Một Quy tắc ứng xử tốt có thể giúp chúng ta làm điều đó.

Chúng tôi không có kinh nghiệm viết Quy tắc ứng xử,
vì vậy chúng tôi đã đọc các quy tắc hiện có,
và chúng tôi có thể sẽ áp dụng một quy tắc hiện có,
có thể với các điều chỉnh nhỏ.
Quy tắc tôi thích nhất là Quy tắc ứng xử Django,
có nguồn gốc từ một dự án khác có tên SpeakUp!
Nó được cấu trúc như một sự mở rộng của danh sách
nhắc nhở cho tương tác hàng ngày.

"Hãy thân thiện và kiên nhẫn.
Hãy chào đón.
Hãy chu đáo.
Hãy tôn trọng.
Hãy cẩn thận với những từ bạn chọn.
Khi chúng ta không đồng ý, hãy cố gắng hiểu tại sao."

Tôi tin rằng điều này nắm bắt được giai điệu chúng ta muốn thiết lập,
thông điệp chúng ta muốn gửi đi,
môi trường chúng ta muốn tạo ra
cho các contributor mới.
Tôi chắc chắn muốn trở thành
thân thiện,
kiên nhẫn,
chào đón,
chu đáo,
và tôn trọng.
Tôi sẽ không làm đúng hoàn toàn mọi lúc,
và tôi hoan nghênh một ghi chú hữu ích
nếu tôi không sống đúng với điều đó.
Tôi tin rằng hầu hết chúng ta
cũng cảm thấy như vậy.

Tôi chưa đề cập đến
việc loại trừ tích cực dựa trên
hoặc ảnh hưởng không cân xứng đến
chủng tộc, giới tính, khuyết tật,
hoặc các đặc điểm cá nhân khác,
và tôi chưa đề cập đến quấy rối.
Đối với tôi,
nó xuất phát từ những gì tôi vừa nói
rằng hành vi loại trừ
hoặc quấy rối rõ ràng
là hoàn toàn không thể chấp nhận được,
trực tuyến và ngoại tuyến.
Mọi Quy tắc ứng xử đều nói điều này một cách rõ ràng,
và tôi kỳ vọng quy tắc của chúng ta cũng vậy.
Nhưng tôi tin rằng những nhắc nhở của SpeakUp!
về các tương tác hàng ngày
là một tuyên bố quan trọng không kém.
Tôi tin rằng
đặt tiêu chuẩn cao
cho những tương tác hàng ngày đó
làm cho hành vi cực đoan
rõ ràng hơn nhiều
và dễ xử lý hơn.

Tôi không có nghi ngờ gì rằng
cộng đồng Go có thể là
một trong những cộng đồng
thân thiện,
chào đón,
chu đáo,
và
tôn trọng nhất
trong ngành công nghiệp công nghệ.
Chúng ta có thể làm điều đó xảy ra,
và nó sẽ là
một lợi ích và tín dụng cho tất cả chúng ta.

Andrew Gerrand
đã dẫn dắt nỗ lực
áp dụng một Quy tắc ứng xử phù hợp
cho cộng đồng Go.
Nếu bạn có đề xuất,
hoặc mối quan tâm,
hoặc kinh nghiệm với Quy tắc ứng xử,
hoặc muốn tham gia,
vui lòng tìm Andrew hoặc tôi
trong hội nghị.
Nếu bạn vẫn còn ở đây vào ngày thứ Sáu,
Andrew và tôi sẽ dành ra
một chút thời gian cho các cuộc thảo luận Quy tắc ứng xử
trong Hack Day.

Một lần nữa, chúng ta không biết
ý tưởng tuyệt vời tiếp theo sẽ đến từ đâu.
Chúng ta cần tất cả sự giúp đỡ có thể có.
Chúng ta cần một cộng đồng Go lớn, đa dạng.

## Cảm ơn

Tôi xem xét nhiều người
phát hành phần mềm để tải xuống bằng "go get,"
chia sẻ những hiểu biết của họ qua các bài đăng blog,
hoặc giúp đỡ người khác trên danh sách thư hoặc IRC
là một phần của nỗ lực mã nguồn mở rộng rãi này,
là một phần của cộng đồng Go.
Mọi người ở đây hôm nay cũng là một phần của cộng đồng đó.

Cảm ơn trước
các diễn giả
những người trong vài ngày tới
sẽ dành thời gian để chia sẻ kinh nghiệm của họ
sử dụng và mở rộng Go.

Cảm ơn trước
tất cả các bạn trong khán giả
vì đã dành thời gian để có mặt ở đây,
đặt câu hỏi,
và cho chúng tôi biết
Go đang hoạt động như thế nào với bạn.
Khi bạn trở về nhà,
hãy tiếp tục chia sẻ những gì bạn đã học.
Ngay cả khi bạn không sử dụng Go
cho công việc hàng ngày,
chúng tôi rất muốn thấy những gì đang hoạt động cho Go
được áp dụng trong các bối cảnh khác,
cũng như chúng tôi luôn tìm kiếm những ý tưởng hay
để đưa trở lại vào Go.

Cảm ơn tất cả các bạn một lần nữa
vì đã nỗ lực có mặt ở đây
và là một phần của cộng đồng Go.

Trong vài ngày tới, hãy:
cho chúng tôi biết những gì chúng tôi đang làm đúng,
cho chúng tôi biết những gì chúng tôi đang làm sai,
và giúp tất cả chúng ta cùng làm việc
để làm cho Go tốt hơn nữa.

Hãy nhớ
hãy thân thiện,
kiên nhẫn,
chào đón,
chu đáo,
và tôn trọng.

Trên tất cả, hãy thích thú với hội nghị.
