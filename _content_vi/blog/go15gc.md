---
title: "Go GC: Ưu tiên độ trễ thấp và sự đơn giản"
date: 2015-08-31
by:
- Richard Hudson
summary: Go 1.5 là bước đầu tiên hướng tới tương lai độ trễ thấp cho bộ thu gom rác của Go.
---

## Bối cảnh

Go đang xây dựng bộ thu gom rác (GC) không chỉ cho năm 2015 mà cho cả năm 2025
và xa hơn: một GC hỗ trợ phát triển phần mềm ngày nay và mở rộng cùng với
phần mềm và phần cứng mới trong thập kỷ tới. Tương lai như vậy không còn chỗ
cho các khoảng dừng stop-the-world của GC, vốn đã là rào cản cho việc sử dụng
rộng rãi hơn các ngôn ngữ an toàn và bảo mật như Go.

Go 1.5, cái nhìn đầu tiên về tương lai này, đạt được độ trễ GC thấp hơn nhiều so với
mục tiêu 10 mili giây chúng tôi đặt ra một năm trước. Chúng tôi đã trình bày một số con số
ấn tượng trong [một bài nói chuyện tại Gophercon](/talks/2015/go-gc.pdf).
Những cải tiến độ trễ đã thu hút rất nhiều sự chú ý;
bài đăng blog của Robin Verlangen
[_Hàng tỷ request mỗi ngày gặp Go 1.5_](https://medium.com/@robin.verlangen/billions-of-request-per-day-meet-go-1-5-362bfefa0911)
xác nhận hướng đi của chúng tôi với kết quả đầu cuối.
Chúng tôi cũng đặc biệt thích
[đồ thị server production của Alan Shreve](https://twitter.com/inconshreveable/status/620650786662555648)
và bình luận "Holy 85% reduction" của anh ấy.

Ngày nay 16 gigabyte RAM có giá 100 đô la và CPU đi kèm nhiều lõi, mỗi lõi với
nhiều luồng phần cứng. Trong một thập kỷ, phần cứng này sẽ có vẻ lỗi thời nhưng
phần mềm đang được xây dựng bằng Go ngày nay sẽ cần mở rộng để đáp ứng nhu cầu
ngày càng tăng và điều tiếp theo lớn lao. Vì phần cứng sẽ cung cấp sức mạnh để
tăng thông lượng, bộ thu gom rác của Go được thiết kế để ưu tiên độ trễ thấp và
điều chỉnh thông qua chỉ một tham số duy nhất. Go 1.5 là bước lớn đầu tiên trên
con đường này và những bước đầu tiên này sẽ mãi mãi ảnh hưởng đến Go và các ứng
dụng mà nó hỗ trợ tốt nhất. Bài đăng này cung cấp tổng quan cấp cao về những gì
chúng tôi đã làm cho bộ thu gom rác Go 1.5.

## Chi tiết kỹ thuật

Để tạo ra bộ thu gom rác cho thập kỷ tới, chúng tôi đã sử dụng một thuật toán
từ nhiều thập kỷ trước. Bộ thu gom rác mới của Go là bộ thu gom _concurrent_,
_tri-color_, _mark-sweep_, một ý tưởng được đề xuất lần đầu bởi
[Dijkstra năm 1978](http://dl.acm.org/citation.cfm?id=359655).
Đây là sự khác biệt có chủ ý so với hầu hết các bộ thu gom rác "doanh nghiệp" hiện nay,
và một bộ mà chúng tôi tin rằng phù hợp tốt với các đặc tính của phần cứng hiện đại
và yêu cầu độ trễ của phần mềm hiện đại.

Trong bộ thu gom tri-color, mỗi đối tượng có màu trắng, xám hoặc đen và chúng tôi
xem heap như một đồ thị các đối tượng được kết nối. Khi bắt đầu một chu kỳ GC, tất cả
các đối tượng đều màu trắng. GC ghé thăm tất cả các _gốc_ (roots), là các đối tượng
được truy cập trực tiếp bởi ứng dụng như các biến global và những thứ trên stack,
và tô màu xám cho chúng. GC sau đó chọn một đối tượng xám, tô đen nó, và sau đó
quét nó để tìm con trỏ đến các đối tượng khác. Khi quét này tìm thấy con trỏ đến
một đối tượng trắng, nó chuyển đối tượng đó sang màu xám. Quá trình này lặp lại
cho đến khi không còn đối tượng xám nào. Lúc này, các đối tượng trắng được biết là
không thể truy cập và có thể được tái sử dụng.

Tất cả điều này diễn ra đồng thời với ứng dụng, được gọi là _mutator_, thay đổi
con trỏ trong khi bộ thu gom đang chạy. Do đó, mutator phải duy trì bất biến rằng
không có đối tượng đen nào trỏ đến đối tượng trắng, để bộ thu gom rác không mất
dấu một đối tượng được cài đặt trong phần heap mà nó đã ghé thăm. Duy trì bất biến
này là công việc của _write barrier_, là một hàm nhỏ chạy bởi mutator mỗi khi một
con trỏ trong heap được sửa đổi. Write barrier của Go tô màu xám cho đối tượng giờ
có thể truy cập nếu nó hiện đang trắng, đảm bảo rằng bộ thu gom rác cuối cùng sẽ
quét nó để tìm con trỏ.

Quyết định khi nào công việc tìm tất cả đối tượng xám hoàn thành là tinh tế và có
thể tốn kém và phức tạp nếu chúng ta muốn tránh chặn các mutator. Để giữ mọi thứ
đơn giản, Go 1.5 thực hiện nhiều công việc nhất có thể một cách đồng thời và sau
đó tạm dừng thế giới ngắn gọn để kiểm tra tất cả các nguồn tiềm năng của đối tượng
xám. Tìm ra điểm ngọt giữa thời gian cần cho lần stop-the-world cuối cùng này và
tổng lượng công việc mà GC này thực hiện là một kết quả chính cho Go 1.6.

Tất nhiên, chi tiết là điều quan trọng. Khi nào chúng ta bắt đầu một chu kỳ GC?
Chúng ta dùng các chỉ số nào để đưa ra quyết định đó? GC nên tương tác với bộ lập
lịch Go như thế nào? Làm thế nào để tạm dừng một luồng mutator đủ lâu để quét
stack của nó? Làm thế nào để biểu diễn trắng, xám và đen để chúng ta có thể tìm
và quét đối tượng xám một cách hiệu quả? Làm thế nào để chúng ta biết đâu là các
gốc? Làm thế nào để chúng ta biết trong một đối tượng con trỏ ở đâu? Làm thế nào
để giảm thiểu phân mảnh bộ nhớ? Làm thế nào để xử lý các vấn đề hiệu năng cache?
Heap nên lớn bao nhiêu? Và còn nhiều nữa, một số liên quan đến cấp phát, một số
đến việc tìm đối tượng có thể truy cập, một số liên quan đến lập lịch, nhưng nhiều
liên quan đến hiệu năng. Thảo luận chi tiết về từng lĩnh vực này vượt ngoài phạm
vi của bài đăng này.

Ở cấp độ cao hơn, một cách tiếp cận để giải quyết vấn đề hiệu năng là thêm các
tham số GC, một cho mỗi vấn đề hiệu năng. Lập trình viên sau đó có thể điều chỉnh
các tham số để tìm cài đặt phù hợp cho ứng dụng của họ. Nhược điểm là sau một thập
kỷ với một hoặc hai tham số mới mỗi năm bạn sẽ có một đạo luật "GC Knobs Turner".
Go không đi theo con đường đó. Thay vào đó chúng tôi cung cấp một tham số duy nhất,
được gọi là GOGC. Giá trị này kiểm soát tổng kích thước heap so với kích thước các
đối tượng có thể truy cập. Giá trị mặc định là 100 có nghĩa là tổng kích thước heap
lớn hơn 100% (tức là gấp đôi) so với kích thước các đối tượng có thể truy cập sau
lần thu gom cuối. 200 có nghĩa là tổng kích thước heap lớn hơn 200% (tức là gấp ba)
so với kích thước các đối tượng có thể truy cập. Nếu bạn muốn giảm tổng thời gian
dành cho GC, hãy tăng GOGC. Nếu bạn muốn đổi nhiều thời gian GC hơn để lấy ít bộ
nhớ hơn, hãy giảm GOGC.

Quan trọng hơn, khi RAM tăng gấp đôi với thế hệ phần cứng tiếp theo, chỉ cần tăng
gấp đôi GOGC sẽ giảm một nửa số chu kỳ GC. Mặt khác, vì GOGC dựa trên kích thước
đối tượng có thể truy cập, tăng gấp đôi tải bằng cách tăng gấp đôi đối tượng có
thể truy cập không cần điều chỉnh lại. Ứng dụng chỉ mở rộng. Hơn nữa, không bị
ràng buộc bởi việc hỗ trợ hàng tá tham số liên tục, nhóm runtime có thể tập trung
vào việc cải thiện runtime dựa trên phản hồi từ các ứng dụng khách hàng thực tế.

## Kết luận

GC của Go 1.5 mở ra một tương lai trong đó các khoảng dừng stop-the-world không còn
là rào cản để chuyển sang ngôn ngữ an toàn và bảo mật. Đó là tương lai nơi các ứng
dụng mở rộng dễ dàng cùng với phần cứng và khi phần cứng trở nên mạnh mẽ hơn, GC
sẽ không còn là trở ngại cho phần mềm tốt hơn, có khả năng mở rộng hơn. Đó là nơi
tốt để ở trong thập kỷ tới và xa hơn.
Để biết thêm chi tiết về GC 1.5 và cách chúng tôi loại bỏ các vấn đề độ trễ, hãy
xem bài trình bày [Go GC: Latency Problem Solved](https://www.youtube.com/watch?v=aiv1JOfMjm0)
hoặc [các slide](/talks/2015/go-gc.pdf).
