---
title: Go tròn 10 tuổi
date: 2019-11-08
by:
- Russ Cox, for the Go team
summary: Chúc mừng sinh nhật 10 tuổi, Go!
template: true
---


Chúc mừng sinh nhật, Go!

Cuối tuần này chúng ta kỷ niệm 10 năm
[bản phát hành Go](https://opensource.googleblog.com/2009/11/hey-ho-lets-go.html),
đánh dấu sinh nhật lần thứ 10 của Go với tư cách là một ngôn ngữ lập trình
mã nguồn mở và hệ sinh thái để xây dựng phần mềm mạng hiện đại.

Để đánh dấu dịp này,
[Renee French](https://twitter.com/reneefrench),
người tạo ra
[Go gopher](/blog/gopher),
đã vẽ nên khung cảnh tuyệt đẹp này:

<a href="10years/gopher10th-large.jpg">
{{image "10years/gopher10th-small.jpg" 850}}
</a>

Việc kỷ niệm 10 năm của Go khiến tôi nhớ lại đầu tháng 11 năm 2009,
khi chúng tôi đang chuẩn bị chia sẻ Go với thế giới.
Chúng tôi không biết sẽ nhận được phản ứng như thế nào,
liệu có ai quan tâm đến ngôn ngữ nhỏ bé này hay không.
Tôi hy vọng rằng ngay cả nếu cuối cùng không ai sử dụng Go,
ít nhất chúng tôi cũng đã thu hút sự chú ý đến một số ý tưởng hay,
đặc biệt là cách tiếp cận của Go đối với tính đồng thời và các interface,
những điều có thể ảnh hưởng đến các ngôn ngữ tiếp nối.

Khi nhận ra rằng mọi người hào hứng với Go,
tôi đã xem xét lịch sử của các ngôn ngữ phổ biến
như C, C++, Perl, Python và Ruby,
xem mỗi ngôn ngữ mất bao lâu để được chấp nhận rộng rãi.
Ví dụ, với tôi Perl dường như đã xuất hiện hoàn chỉnh
vào giữa đến cuối thập niên 1990, cùng với các tập lệnh CGI và web,
nhưng nó được bản phát hành đầu tiên vào năm 1987.
Mô hình này lặp lại với gần như mọi ngôn ngữ mà tôi xem xét:
dường như cần khoảng một thập kỷ cải tiến âm thầm, đều đặn
và phổ biến trước khi một ngôn ngữ mới thực sự phát triển mạnh.

Tôi tự hỏi: Go sẽ như thế nào sau một thập kỷ?

Ngày nay, chúng ta có thể trả lời câu hỏi đó:
Go có mặt ở khắp nơi, được sử dụng bởi ít nhất [một triệu nhà phát triển trên toàn thế giới](https://research.swtch.com/gophercount).

Mục tiêu ban đầu của Go là hạ tầng hệ thống mạng,
mà ngày nay chúng ta gọi là phần mềm đám mây.
Mọi nhà cung cấp đám mây lớn hiện nay đều sử dụng hạ tầng đám mây cốt lõi được viết bằng Go,
chẳng hạn như Docker, Etcd, Istio, Kubernetes, Prometheus và Terraform;
phần lớn các
[dự án của Cloud Native Computing Foundation](https://www.cncf.io/projects/)
được viết bằng Go.
Vô số công ty cũng đang sử dụng Go để chuyển công việc của họ lên đám mây,
từ các startup xây dựng từ đầu
đến các doanh nghiệp hiện đại hóa ngăn xếp phần mềm của mình.
Go cũng đã được áp dụng vượt xa mục tiêu đám mây ban đầu,
với các trường hợp sử dụng trải rộng
từ
việc điều khiển các hệ thống nhúng nhỏ với
[GoBot](https://gobot.io) và [TinyGo](https://tinygo.org/)
đến phát hiện ung thư bằng
[phân tích dữ liệu lớn quy mô lớn và học máy tại GRAIL](https://medium.com/grail-eng/bigslice-a-cluster-computing-system-for-go-7e03acd2419b),
và mọi thứ ở giữa.

Tất cả những điều này cho thấy Go đã thành công vượt xa những gì chúng tôi từng mơ ước.
Và thành công của Go không chỉ là về ngôn ngữ.
Nó là về ngôn ngữ, hệ sinh thái, và đặc biệt là cộng đồng cùng làm việc với nhau.

Năm 2009, ngôn ngữ này là một ý tưởng tốt với một bản phác thảo triển khai đang hoạt động.
Lệnh `go` chưa tồn tại:
chúng tôi chạy các lệnh như `6g` để biên dịch và `6l` để liên kết binary,
được tự động hóa bằng makefile.
Chúng tôi gõ dấu chấm phẩy ở cuối các câu lệnh.
Toàn bộ chương trình dừng lại trong quá trình bộ gom rác,
vốn khi đó gặp khó khăn trong việc tận dụng tốt hai lõi xử lý.
Go chỉ chạy trên Linux và Mac, trên x86 32 bit, 64 bit và ARM 32 bit.

Trong thập kỷ vừa qua, với sự giúp đỡ của các nhà phát triển Go trên khắp thế giới,
chúng tôi đã phát triển ý tưởng và bản phác thảo này thành một ngôn ngữ hiệu quả
với hệ thống công cụ tuyệt vời,
một triển khai đạt chất lượng sản phẩm,
một
[bộ gom rác hiện đại nhất](/blog/ismmkeynote),
và [các bản chuyển sang 12 hệ điều hành và 10 kiến trúc](/doc/install/source#introduction).

Mọi ngôn ngữ lập trình đều cần sự hỗ trợ của một hệ sinh thái phát triển mạnh.
Bản phát hành mã nguồn mở là hạt giống cho hệ sinh thái đó,
nhưng kể từ đó, nhiều người đã đóng góp thời gian và tài năng của họ
để lấp đầy hệ sinh thái Go bằng các hướng dẫn, sách, khóa học, bài đăng blog,
podcast, công cụ, tích hợp tuyệt vời, và tất nhiên là các gói Go có thể tái sử dụng được nhập bằng `go` `get`.
Go sẽ không bao giờ có thể thành công nếu thiếu sự hỗ trợ của hệ sinh thái này.

Tất nhiên, hệ sinh thái cần sự hỗ trợ của một cộng đồng phát triển mạnh.
Vào năm 2019 có hàng chục hội nghị Go trên khắp thế giới,
cùng với
[hơn 150 nhóm meetup Go với hơn 90.000 thành viên](https://www.meetup.com/pro/go).
[GoBridge](https://golangbridge.org)
và
[Women Who Go](https://medium.com/@carolynvs/www-loves-gobridge-ccb26309f667)
giúp đưa những tiếng nói mới vào cộng đồng Go,
thông qua việc cố vấn, đào tạo và học bổng hội nghị.
Chỉ riêng năm nay, họ đã giảng dạy
cho hàng trăm người thuộc các nhóm thường ít được đại diện
tại các hội thảo nơi các thành viên cộng đồng giảng dạy và cố vấn cho những người mới làm quen với Go.

Có
[hơn một triệu nhà phát triển Go](https://research.swtch.com/gophercount)
trên toàn thế giới,
và các công ty trên khắp toàn cầu đang tìm cách tuyển thêm người.
Trên thực tế, mọi người thường nói với chúng tôi rằng việc học Go
đã giúp họ có được công việc đầu tiên trong ngành công nghệ.
Cuối cùng, điều chúng tôi tự hào nhất về Go
không phải là một tính năng được thiết kế tốt hay một đoạn mã thông minh
mà là tác động tích cực mà Go đã tạo ra trong cuộc sống của rất nhiều người.
Chúng tôi hướng tới việc tạo ra một ngôn ngữ giúp chúng tôi trở thành những nhà phát triển tốt hơn,
và chúng tôi rất vui mừng khi Go đã giúp đỡ rất nhiều người khác.

Nhân dịp
[\#GoTurns10](https://twitter.com/search?q=%23GoTurns10),
tôi hy vọng mọi người sẽ dành một chút thời gian để kỷ niệm
cộng đồng Go và tất cả những gì chúng ta đã đạt được.
Thay mặt toàn bộ đội ngũ Go tại Google,
cảm ơn tất cả mọi người đã đồng hành cùng chúng tôi trong thập kỷ vừa qua.
Hãy cùng biến thập kỷ tiếp theo trở nên tuyệt vời hơn nữa!

<div>
<center>
<a href="10years/gopher10th-pin-large.jpg">
{{image "10years/gopher10th-pin-small.jpg" 150}}
</center>
</div>
