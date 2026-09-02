---
title: Kết quả Khảo sát Nhà phát triển Go 2023 Q1
date: 2023-05-11
by:
- Alice Merrick
tags:
- survey
- community
summary: Phân tích kết quả từ Khảo sát Nhà phát triển Go 2023 Q1.
---

<style type="text/css" scoped>
  .chart {
    margin-left: 1.5rem;
    margin-right: 1.5rem;
    width: 800px;
  }
  .quote {
    margin-left: 2rem;
    margin-right: 2rem;
    color: #999;
    font-style: italic;
    font-size: 120%;
  }
  @media (prefers-color-scheme: dark) {
    .chart {
      border-radius: 8px;
    }
  }
</style>

## Cảm ơn những người tham gia đã mang lại cho chúng tôi những hiểu biết này!

Chúng tôi rất vui được chia sẻ với bạn kết quả của phiên bản tháng 1 năm 2023 của Khảo sát Nhà phát triển Go. Cảm ơn 5.844 người tham gia đã chia sẻ với chúng tôi cách họ đang sử dụng Go, những thách thức lớn nhất khi sử dụng Go, và những ưu tiên hàng đầu để cải tiến trong tương lai. Những kết quả này giúp nhóm Go tập trung nỗ lực vào các lĩnh vực quan trọng nhất với cộng đồng, và chúng tôi hy vọng những hiểu biết này cũng giúp ích cho những người khác đóng góp và hỗ trợ hệ sinh thái Go.

### Những phát hiện chính

- __Các nhà phát triển Go mới thích phát triển web__. Năm nay chúng tôi đã giới thiệu một cách phân khúc mới dựa trên mức độ kinh nghiệm tự xác định. Người mới bày tỏ một số khác biệt thú vị so với các mức kinh nghiệm khác. Đáng chú ý nhất là họ cho thấy sự quan tâm nhiều hơn đến việc sử dụng Go để phát triển web.
- __Xử lý lỗi và học tập là những thách thức hàng đầu của người tham gia__. Trước đây, thiếu generics là thách thức lớn nhất khi sử dụng Go, nhưng kể từ khi generics được giới thiệu, số nhận xét về generics đã giảm. Các ý kiến về xử lý lỗi (liên quan đến khả năng đọc và sự dài dòng) và khó khăn trong việc học các phương pháp hay nhất hiện là những thách thức được báo cáo phổ biến nhất.
- __Hướng dẫn tối ưu hóa là cách cải thiện hiệu suất Go được đánh giá cao nhất__. Khi được hỏi họ sẽ phân bổ nguồn lực cho các cải tiến khác nhau về trình biên dịch và runtime của Go như thế nào, người tham gia chi nhiều nhất cho hướng dẫn tối ưu hóa thay vì các cải tiến hiệu suất cụ thể, cho thấy tầm quan trọng của tài liệu trong lĩnh vực này.
- __Quản lý dependency và phiên bản là những thách thức hàng đầu cho người duy trì module Go mã nguồn mở__. Những người duy trì module mã nguồn mở gặp khó khăn trong việc cập nhật dependency và tránh gián đoạn do phiên bản và các thay đổi phá vỡ tương thích. Đây là lĩnh vực chúng tôi sẽ khám phá thêm để giúp người duy trì cung cấp một hệ sinh thái ổn định và lành mạnh.

### Cách đọc kết quả này

Xuyên suốt bài đăng này, chúng tôi sử dụng các biểu đồ phản hồi khảo sát để cung cấp bằng chứng hỗ trợ cho các phát hiện của mình. Tất cả các biểu đồ này sử dụng định dạng tương tự. Tiêu đề là câu hỏi chính xác mà người tham gia khảo sát đã thấy. Trừ khi có ghi chú khác, các câu hỏi là câu hỏi nhiều lựa chọn và người tham gia chỉ có thể chọn một câu trả lời duy nhất; phụ đề của mỗi biểu đồ sẽ cho bạn biết liệu câu hỏi có cho phép nhiều lựa chọn hoặc là hộp văn bản mở thay vì câu hỏi nhiều lựa chọn hay không. Đối với các biểu đồ câu trả lời văn bản mở, một thành viên nhóm Go đã đọc và phân loại thủ công tất cả các câu trả lời. Nhiều câu hỏi mở thu hút nhiều loại phản hồi; để giữ kích thước biểu đồ hợp lý, chúng tôi rút gọn xuống 10-15 chủ đề hàng đầu, với các chủ đề bổ sung được nhóm dưới "Khác". Chúng tôi cũng đã thêm danh mục "Không có" khi thích hợp.

Để giúp người đọc hiểu được mức độ bằng chứng đằng sau mỗi phát hiện, chúng tôi bao gồm các thanh lỗi hiển thị khoảng tin cậy 95% cho các phản hồi; thanh hẹp hơn cho thấy độ tin cậy tăng. Đôi khi hai hoặc nhiều phản hồi có các thanh lỗi chồng lên nhau, có nghĩa là thứ tự tương đối của các phản hồi đó không có ý nghĩa thống kê (tức là, các phản hồi thực tế bằng nhau). Góc dưới bên phải của mỗi biểu đồ hiển thị số người có phản hồi được đưa vào biểu đồ, dưới dạng "_n = [số người tham gia]_".

### Ghi chú về phương pháp luận

Hầu hết người tham gia khảo sát "tự chọn" tham gia khảo sát bằng cách truy cập qua một liên kết trên [blog Go](/blog), [@golang trên Twitter](https://twitter.com/golang), hoặc các kênh Go xã hội khác. Những người không theo dõi các kênh này có thể phản hồi khác với những người _có_ theo dõi chúng chặt chẽ. Khoảng một phần tư người tham gia được lấy mẫu ngẫu nhiên, có nghĩa là họ đã phản hồi khảo sát sau khi thấy lời nhắc trong VS Code (mọi người sử dụng plugin VS Code Go từ ngày 18 tháng 1 - 8 tháng 2 năm 2023 đều có 10% cơ hội nhận được lời nhắc ngẫu nhiên này). Nhóm được lấy mẫu ngẫu nhiên này giúp chúng tôi khái quát hóa những phát hiện này cho cộng đồng lớn hơn của các nhà phát triển Go. Hầu hết các câu hỏi khảo sát không cho thấy sự khác biệt có ý nghĩa giữa các nhóm này, nhưng trong một số ít trường hợp có sự khác biệt quan trọng, người đọc sẽ thấy các biểu đồ phân tích phản hồi thành các nhóm "Mẫu ngẫu nhiên" và "Tự chọn".

## Nhìn kỹ hơn vào các nhóm người tham gia khác nhau

Nhân khẩu học người tham gia của chúng tôi không thay đổi đáng kể so với [khảo sát trước](/blog/survey2022-q2-results). Nhất quán với các chu kỳ trước, Go chủ yếu được sử dụng trong ngành công nghệ, và khoảng 80% người tham gia cho biết họ lập trình bằng Go trong công việc. Nhìn chung, người tham gia khảo sát có xu hướng hài lòng với Go trong năm qua, với 92% cho biết họ hài lòng phần nào hoặc rất hài lòng.

<img src="survey2023q1/where.svg" alt="Biểu đồ thanh hiển thị nơi người tham gia sử dụng Go" class="chart"/> <img
src="survey2023q1/csat.svg" alt="Biểu đồ thanh hiển thị tỷ lệ người tham gia hài lòng" class="chart"/>

Người tham gia của chúng tôi dành nhiều thời gian lập trình bằng Go so với các ngôn ngữ khác. Khoảng một phần ba người tham gia thậm chí duy trì một module Go mã nguồn mở. Chúng tôi nhận ra rằng đối tượng khảo sát của chúng tôi bao gồm những người đã áp dụng thành công Go, thường xuyên sử dụng Go, và hầu hết đều hài lòng khi sử dụng Go. Để xác định những khoảng cách tiềm ẩn trong việc đáp ứng nhu cầu cộng đồng, chúng tôi xem xét các nhóm con khác nhau của người tham gia để xem họ có thể đang sử dụng Go khác nhau hoặc có các ưu tiên khác nhau như thế nào. Ví dụ, năm nay chúng tôi đã xem xét cách phản hồi khác nhau giữa các nguồn mẫu khác nhau (tức là, Blog Go hoặc thông qua plugin VS Code), vai trò công việc khác nhau, quy mô tổ chức, và mức độ kinh nghiệm Go. Những điểm khác biệt thú vị nhất là giữa các mức kinh nghiệm.

## Hiểu biết từ người tham gia mới

<img src="survey2023q1/go_exp.svg" alt="Biểu đồ thanh về số năm kinh nghiệm sử dụng Go" class="chart"/>

Trước đây, chúng tôi đã dùng thời gian (tính theo tháng/năm) người tham gia sử dụng Go như một thước đo để hiểu kết quả thay đổi như thế nào giữa các mức độ kinh nghiệm. Năm nay chúng tôi đã thử nghiệm với một câu hỏi phân khúc mới, "Mức độ kinh nghiệm của bạn với Go là gì?", để xem liệu tự xác định có thể là cách hữu ích hơn để xem xét kinh nghiệm Go so với việc gộp các khoảng thời gian khác nhau. Vì các thuật ngữ phân loại như "mới" hoặc "chuyên gia" có thể khác nhau giữa người này và người kia, chúng tôi đã cung cấp mô tả để giúp làm cho các nhóm này khách quan hơn. Các lựa chọn là:

* Nhận thức: Tôi biết về Go, nhưng không thể viết một chương trình Go đơn giản mà không có sự hỗ trợ
* Mới: Tôi có thể hoàn thành các dự án lập trình đơn giản bằng Go, có thể với sự hỗ trợ
* Trung cấp: Tôi có thể hoàn thành các dự án lập trình quan trọng bằng Go với một số hỗ trợ
* Nâng cao: Tôi có thể hoàn thành các dự án lập trình quan trọng bằng Go mà không cần hỗ trợ
* Chuyên gia: Tôi có thể cung cấp hướng dẫn, khắc phục sự cố, và trả lời các câu hỏi liên quan đến Go từ các kỹ sư khác

<img src="survey2023q1/exp_level.svg" alt="Biểu đồ thanh về mức độ kinh nghiệm sử dụng Go" class="chart"/>

Chúng tôi tìm thấy mối tương quan vừa phải (⍴ = .66) giữa thời gian người tham gia sử dụng Go và mức độ kinh nghiệm tự xác định của họ. Điều này có nghĩa là thang đo mức độ kinh nghiệm, mặc dù tương tự như thang đo thời gian, có thể cung cấp cho chúng tôi một số hiểu biết mới về cách người tham gia khác nhau theo kinh nghiệm. Ví dụ, tỷ lệ thời gian mà người tham gia dành để viết bằng Go so với thời gian họ dành để viết bằng các ngôn ngữ khác có tương quan mạnh hơn với mức độ kinh nghiệm tự xác định của họ so với thời gian họ đã sử dụng Go.

Trong các phân tích chúng tôi sử dụng phân khúc này, chúng tôi thường loại bỏ danh mục Nhận thức vì họ sẽ không được coi là có đủ kinh nghiệm để trả lời câu hỏi và chỉ chiếm khoảng 1% người tham gia.

### Người tham gia mới có nhiều khả năng thích Windows hơn so với người tham gia có kinh nghiệm hơn

Nhóm được lấy mẫu ngẫu nhiên của chúng tôi có tỷ lệ người tham gia mới cao hơn so với nhóm tự chọn, cho thấy có nhiều Gopher mới hơn ngoài kia mà chúng tôi không thường nghe từ họ. Vì họ được lấy mẫu qua plugin Go VS Code, chúng tôi có thể kỳ vọng nhóm này có nhiều khả năng thích VS Code hơn hoặc phát triển trên Windows nhiều hơn so với các mức kinh nghiệm khác. Mặc dù điều này đúng, người mới cũng có nhiều khả năng phát triển trên Windows hơn so với các mức kinh nghiệm khác, bất kể họ có phản hồi qua plugin VS Code hay không.

<img src="survey2023q1/exp_level_s.svg" alt="Biểu đồ thanh về mức độ kinh nghiệm sử dụng Go cho mẫu tự chọn và ngẫu nhiên" class="chart"/> <img
src="survey2023q1/editor_self_select_exp.svg" alt="Biểu đồ thanh về sở thích trình soạn thảo phân tích theo mức kinh nghiệm chỉ cho nhóm tự chọn" class="chart"/> <img src="survey2023q1/os_dev_exp_s.svg" alt="Biểu đồ thanh về mức độ kinh nghiệm sử dụng Go" class="chart"/>

Có thể có nhiều lý do tại sao chúng tôi không thấy tỷ lệ người dùng Windows cao hơn ở các mức kinh nghiệm cao hơn. Ví dụ, người dùng Windows có thể có nhiều khả năng gặp khó khăn và ngừng sử dụng Go hơn, hoặc có thể có xu hướng rộng hơn trong việc sử dụng hệ điều hành không liên quan đến Go. Trong mọi trường hợp, chúng tôi nên đưa thêm người dùng Windows vào nghiên cứu trong tương lai về việc bắt đầu với Go để đảm bảo chúng tôi cung cấp trải nghiệm onboarding toàn diện.

### Cách các mức kinh nghiệm khác nhau hiện đang sử dụng Go (và các lĩnh vực họ muốn sử dụng)

<img src="survey2023q1/go_app.svg" alt="Biểu đồ thanh về các trường hợp sử dụng" class="chart"/> <img src="survey2023q1/go_app_exp.svg" alt="Biểu đồ thanh về các trường hợp sử dụng phân tích theo mức kinh nghiệm" class="chart"/>

Theo cách người tham gia hiện đang sử dụng Go, các Gopher có nhiều kinh nghiệm hơn có xu hướng sử dụng Go cho nhiều loại ứng dụng hơn. Ví dụ, chuyên gia trung bình sử dụng Go trong ít nhất bốn lĩnh vực trong khi người mới trung bình sử dụng Go chỉ trong hai lĩnh vực. Đó là lý do tại sao có sự khác biệt lớn về tỷ lệ người mới và chuyên gia sử dụng Go cho mỗi trường hợp sử dụng. Tuy nhiên, hai công dụng hàng đầu, dịch vụ API/RPC và CLI, là các trường hợp sử dụng phổ biến nhất ở tất cả các mức kinh nghiệm.

Chúng tôi thấy các xu hướng thú vị hơn đối với GUI và Website/Dịch vụ Web (trả về HTML). Tất cả các mức kinh nghiệm đều sử dụng Go cho ứng dụng Desktop/GUI ở khoảng cùng một tỷ lệ. Điều này cho thấy mong muốn có GUI không chỉ đến từ các Gopher mới đang tìm kiếm một dự án khởi đầu thú vị, mà từ toàn bộ phổ kinh nghiệm.

Websites/dịch vụ trả về HTML cũng cho thấy xu hướng tương tự. Một giải thích có thể là đây là một trường hợp sử dụng phổ biến sớm trong hành trình Go của ai đó (vì nó nằm trong top 3 phổ biến nhất cho người mới), hoặc người mới có nhiều khả năng làm việc trên websites hoặc dịch vụ web trả về HTML hơn. Ở phần sau của khảo sát, chúng tôi hỏi người tham gia, "Trong lĩnh vực nào (nếu có) bạn không sử dụng Go, nhưng muốn sử dụng nhất?" Mặc dù nhiều người tham gia (29%) cho biết họ đã sử dụng Go ở khắp mọi nơi họ muốn, hai lĩnh vực hàng đầu để mở rộng việc sử dụng là ứng dụng GUI/Desktop và AI/ML. Điều này nhất quán giữa các nhóm ở quy mô tổ chức và vai trò công việc khác nhau, nhưng không phải ở các mức kinh nghiệm. Lĩnh vực số một mà người mới muốn sử dụng Go nhiều hơn là cho websites/dịch vụ web trả về HTML.

<img src="survey2023q1/app_opportunities_exp.svg" alt="Biểu đồ thanh về mức độ kinh nghiệm sử dụng Go" class="chart"/>

Trong một câu hỏi văn bản mở, 12 trong số 29 người tham gia cho biết họ muốn sử dụng Go cho websites/dịch vụ web trả về HTML nói rằng họ bị chặn vì các ngôn ngữ khác có framework để hỗ trợ trường hợp sử dụng này tốt hơn. Có thể là các nhà phát triển Go có kinh nghiệm hơn không cố gắng hoặc kỳ vọng sử dụng Go cho trường hợp sử dụng này khi các ngôn ngữ khác đã có framework đáp ứng những nhu cầu đó. Như một người tham gia đã diễn đạt:

>"Thường thì dễ hơn khi thực hiện điều này trong các ngôn ngữ khác như PHP hoặc Ruby. Một phần do các framework xuất sắc tồn tại trong các ngôn ngữ đó."

Một giải thích đóng góp khác cho sự quan tâm của người mới đến phát triển web có thể liên quan đến việc họ sử dụng JavaScript/TypeScript. Người mới dành nhiều thời gian hơn để viết bằng JavaScript/TypeScript so với người tham gia có kinh nghiệm hơn. Sự quan tâm cao hơn đến web có thể có liên quan đến những gì người tham gia mới hiện đang làm trong các ngôn ngữ khác hoặc có thể cho thấy sự quan tâm chung đến công nghệ web. Trong tương lai, chúng tôi muốn tìm hiểu thêm về trường hợp sử dụng này và cách chúng tôi có thể giúp các Gopher mới bắt đầu sử dụng Go trong các lĩnh vực hữu ích nhất với họ.

<img src="survey2023q1/language_time_exp.svg" alt="Biểu đồ thanh về mức độ kinh nghiệm sử dụng Go" class="chart"/>

## Người tham gia đối mặt với danh sách dài các thách thức

Mỗi chu kỳ khảo sát chúng tôi hỏi người tham gia thách thức lớn nhất của họ khi sử dụng Go là gì. Trước đây, thiếu generics là thách thức được đề cập nhiều nhất, ví dụ, đó là câu trả lời phổ biến nhất vào năm 2020, và được đề cập bởi khoảng 18% người tham gia. Kể từ khi generics được giới thiệu, xử lý lỗi (12%) và học tập/phương pháp hay nhất/tài liệu (11%) đã nổi lên ở đầu danh sách dài các vấn đề thay vì bất kỳ vấn đề đơn lẻ nào trở nên thường xuyên hơn.

<img src="survey2023q1/text_biggest_challenge.svg" alt="Biểu đồ thanh về những thách thức lớn nhất" class="chart"/>

### Tại sao xử lý lỗi lại là một thách thức như vậy?

Phản hồi về xử lý lỗi thường mô tả vấn đề là sự dài dòng. Bề ngoài, điều này có thể phản ánh rằng việc viết code lặp đi lặp lại thật nhàm chán hoặc khó chịu. Tuy nhiên, hơn là chỉ là phiền phức khi viết boilerplate, xử lý lỗi cũng có thể ảnh hưởng đến khả năng debug của người tham gia.

Một người tham gia đã minh họa vấn đề này một cách ngắn gọn:

>"Xử lý lỗi tạo ra sự lộn xộn và dễ dàng che giấu các vấn đề nếu không được thực hiện đúng (không có stack trace)"

### Cuộc vật lộn để học các phương pháp hay nhất

>"Sử dụng Go hiệu quả. Dễ học, khó thành thạo."

Chúng tôi đã nghe rằng Go dễ học, và [một khảo sát trước đây cho thấy hơn 70%](/blog/survey2020-results#TOC_6.2) người tham gia cảm thấy làm việc hiệu quả khi sử dụng Go trong năm đầu tiên, nhưng việc học các phương pháp hay nhất của Go đã được nêu ra là một trong những thách thức lớn nhất khi sử dụng Go. Người tham gia năm nay cho chúng tôi biết rằng các phương pháp hay nhất xung quanh **cấu trúc code** và **các công cụ và thư viện được khuyến nghị** không được ghi chép đầy đủ, tạo ra thách thức cho người mới bắt đầu và các nhóm để giữ code nhất quán. Học cách viết Go idiomatic có thể đặc biệt thách thức đối với những người đến từ các mô hình lập trình khác. Người tham gia có kinh nghiệm hơn với Go đã xác nhận rằng khi các nhà phát triển không tuân theo các phương pháp hay nhất để viết Go idiomatic, nó làm giảm tính nhất quán và chất lượng của các dự án chia sẻ.

## Những thách thức lớn nhất cho người duy trì module

Người duy trì module Go là những thành viên quan trọng của cộng đồng Go, giúp phát triển và duy trì sức khỏe của hệ sinh thái package của chúng tôi. Năm nay chúng tôi dự định thực hiện nghiên cứu với những người duy trì module để xác định cơ hội hỗ trợ sự ổn định và phát triển của hệ sinh thái package và giúp tăng cường việc áp dụng Go trong các tổ chức. Để thông tin cho nghiên cứu này, chúng tôi đã giới thiệu một câu hỏi trong khảo sát để có ý tưởng về những thách thức hàng đầu hiện tại cho những người duy trì mã nguồn mở.

<img src="survey2023q1/text_maintainer_challenge.svg" alt="Biểu đồ thanh về những thách thức cho người duy trì module mã nguồn mở" class="chart"/>

Những thách thức hàng đầu cho người duy trì là giữ cho dependency được cập nhật và khó khăn xung quanh việc quản lý phiên bản, bao gồm tránh, xác định, hoặc biết khi nào nên giới thiệu các thay đổi phá vỡ tương thích. Những hiểu biết này, cùng với kết quả của nghiên cứu trong tương lai, sẽ giúp thông tin cho các chiến lược hỗ trợ người duy trì trong việc giữ hệ sinh thái Go ổn định và an toàn.

## Những thách thức lớn nhất khi triển khai code Go

Năm nay chúng tôi hỏi thách thức lớn nhất của người tham gia khi triển khai code Go là gì. "Dễ triển khai" thường được trích dẫn là lý do sử dụng Go, nhưng chúng tôi nhận được phản hồi mâu thuẫn trong một nghiên cứu gần đây khiến chúng tôi khám phá các vấn đề tiềm ẩn khi triển khai code Go. Trong các câu trả lời văn bản mở của chúng tôi, chủ đề phổ biến nhất là khó khăn khi cross-compiling với cgo (16%), và hỗ trợ cho WebAssembly hoặc WASI đứng ở vị trí xa thứ hai (7%).

<img src="survey2023q1/text_deploy_challenge.svg" alt="Biểu đồ thanh về những thách thức cho người duy trì module mã nguồn mở" class="chart"/>

## Ưu tiên cộng đồng: điều người tham gia muốn nhất

Năm nay chúng tôi đã sử dụng câu hỏi ưu tiên hóa mà chúng tôi đã sử dụng trong các khảo sát trước dựa trên phương pháp buy-a-feature. Người tham gia được cho 10 "gophercoin" và được yêu cầu phân phối chúng cho các lĩnh vực họ muốn thấy cải tiến. Người tham gia được ngẫu nhiên giao một trong ba câu hỏi có thể, mỗi câu chứa bảy mục liên quan đến hệ thống công cụ, bảo mật, hoặc trình biên dịch & runtime. Cách tiếp cận này cho phép chúng tôi hỏi về các mục liên quan đến từng lĩnh vực trọng tâm mà không làm quá tải người tham gia với ba bộ câu hỏi ưu tiên đòi hỏi nhận thức cao.

Vào cuối bài tập, chúng tôi đã cho người tham gia một câu hỏi văn bản mở để nói với chúng tôi về bất kỳ lĩnh vực nào họ nghĩ nên là ưu tiên hàng đầu của nhóm Go trong năm tới, bất kể mục nào họ đã dành coin. Ví dụ, nếu một người tham gia được hiển thị phần bảo mật, nhưng họ không quan tâm nhiều đến bảo mật, họ vẫn có cơ hội nói với chúng tôi điều đó trong phần văn bản mở.

### Bảo mật

Chúng tôi đã chọn các mục này để kiểm tra các giả định chúng tôi có về tầm quan trọng tương đối của các thực hành bảo mật đối với cộng đồng. Đây là bảy mục được mô tả cho người tham gia:

* pkg.go.dev xác định các package được bảo trì kém (ví dụ: không phản hồi với các issue, không cập nhật dependency, vẫn còn lỗ hổng bảo mật trong thời gian dài)
* pkg.go.dev xác định các package thực hiện thay đổi API phá vỡ tương thích (tức là, yêu cầu sửa các cách sử dụng API đó khi nâng cấp lên phiên bản mới hơn)
* Hỗ trợ cho việc loại bỏ các lỗ hổng bảo mật trong govulncheck
* Công cụ theo dõi cách dữ liệu nhạy cảm chảy qua một chương trình Go (phát hiện rò rỉ PII)
* Hướng dẫn phương pháp hay nhất về bảo mật (ví dụ: cách chọn và cập nhật dependency; cách thiết lập fuzzing, kiểm tra lỗ hổng bảo mật, và thread sanitizer; cách sử dụng crypto)
* Thư viện Web & SQL an toàn theo mặc định giúp người dùng tránh đưa ra các lỗ hổng bảo mật trong code máy chủ web
* Thư viện mã hóa tuân thủ FIPS-140

<img src="survey2023q1/prioritization_security.svg" alt="Biểu đồ thanh về nơi người tham gia chi nhiều nhất cho các vấn đề bảo mật" class="chart"/>

Tính năng bảo mật được tài trợ cao nhất là thư viện web & SQL an toàn theo mặc định để tránh đưa ra các lỗ hổng bảo mật trong code máy chủ web, nhưng bốn tính năng hàng đầu đều liên quan đến việc tránh đưa ra các lỗ hổng bảo mật. Mong muốn về mặc định an toàn nhất quán với nghiên cứu bảo mật trước đây cho thấy các nhà phát triển muốn "shift left" về bảo mật: các nhóm phát triển thường không có thời gian hoặc nguồn lực để giải quyết các vấn đề bảo mật, và do đó coi trọng các công cụ giảm khả năng đưa chúng vào ngay từ đầu. Mục phổ biến thứ hai là hướng dẫn phương pháp hay nhất về bảo mật, làm nổi bật giá trị cao của tài liệu phương pháp hay nhất so với các công cụ hoặc tính năng mới đối với đa số người tham gia.

### Công cụ

Các mục chúng tôi đưa vào câu hỏi này được lấy cảm hứng từ phản hồi của người dùng plugin VS Code. Chúng tôi muốn biết những cải tiến hệ thống công cụ và IDE nào sẽ hữu ích nhất cho đối tượng rộng hơn có thể sử dụng các IDE hoặc trình soạn thảo khác.
* Công cụ refactoring tốt hơn (ví dụ: hỗ trợ chuyển đổi code tự động: đổi tên, trích xuất hàm, di chuyển API, v.v.)
* Hỗ trợ tốt hơn cho việc testing trong trình soạn thảo/IDE của bạn (ví dụ: Test Explorer UI mạnh mẽ và có thể mở rộng, framework test bên thứ ba, hỗ trợ subtest, phạm vi bao phủ code)
* Hỗ trợ tốt hơn để làm việc trên nhiều module trong trình soạn thảo/IDE của bạn (ví dụ: chỉnh sửa module A và B, trong đó module A phụ thuộc vào module B)
* Thông tin chi tiết về dependency trong pkg.go.dev (ví dụ: lỗ hổng bảo mật, thay đổi phá vỡ tương thích, scorecard)
* Thông tin chi tiết về dependency trong trình soạn thảo/IDE của bạn (ví dụ: lỗ hổng bảo mật, thay đổi phá vỡ tương thích, scorecard)
* Hỗ trợ xuất bản module với đường dẫn module mới (ví dụ: chuyển giao quyền sở hữu repository)
* Hỗ trợ tìm các kiểu triển khai một interface & các interface được triển khai bởi một kiểu trong trình soạn thảo/IDE của bạn

<img
src="survey2023q1/prioritization_tooling.svg" alt="Biểu đồ thanh về nơi người tham gia chi nhiều nhất cho hệ thống công cụ" class="chart"/>

Tính năng trình soạn thảo được tài trợ nhiều nhất là *hỗ trợ tìm các kiểu triển khai một interface và các interface được triển khai bởi một kiểu* và *công cụ refactoring*. Chúng tôi cũng thấy sự khác biệt thú vị trong cách người tham gia chi gophercoin của họ theo sở thích sử dụng trình soạn thảo. Đáng chú ý nhất, người dùng VS Code chi nhiều gophercoin hơn cho refactoring so với người dùng GoLand, cho thấy rằng các chuyển đổi code tự động hiện được hỗ trợ tốt hơn trong GoLand so với VS Code.

### Trình biên dịch & runtime

Câu hỏi chính của chúng tôi cho phần này là xác định liệu người tham gia có muốn hiệu suất tốt hơn theo mặc định, công cụ tối ưu hóa tốt hơn, hay chỉ là hiểu rõ hơn về cách viết code Go có hiệu suất cao.
* Giảm chi phí tính toán
* Giảm sử dụng bộ nhớ
* Giảm kích thước nhị phân
* Giảm thời gian build
* Công cụ debug hiệu suất tốt hơn
* Hướng dẫn tối ưu hóa (cách cải thiện hiệu suất và giảm chi phí, bao gồm việc triển khai Go và các công cụ debug hiệu suất)
* Hỗ trợ tốt hơn cho việc sử dụng cgo khi cross-compiling

<img src="survey2023q1/prioritization_core.svg" alt="Biểu đồ thanh về nơi người tham gia chi nhiều nhất cho các cải tiến trình biên dịch và runtime" class="chart"/>

Mục được tài trợ nhiều nhất trong danh sách này là hướng dẫn tối ưu hóa. Điều này nhất quán giữa quy mô tổ chức, vai trò công việc, và mức độ kinh nghiệm. Chúng tôi đã hỏi thêm một câu hỏi về việc người tham gia có mối lo ngại về chi phí tài nguyên hay không. Hầu hết người tham gia (55%) cho biết họ không có mối lo ngại chi phí nào, nhưng những người có lo ngại về chi phí tài nguyên đã chi nhiều gophercoin hơn (trung bình là 2,0) để giảm chi phí tính toán và bộ nhớ so với những người không có. Tuy nhiên, ngay cả những người lo ngại về chi phí tài nguyên vẫn chi khoảng nhiều cho hướng dẫn tối ưu hóa (trung bình 1,9 gophercoin). Đây là tín hiệu mạnh mẽ rằng cung cấp hướng dẫn cho các nhà phát triển Go để hiểu và tối ưu hóa hiệu suất Go hiện có giá trị hơn so với các cải tiến hiệu suất trình biên dịch và runtime bổ sung.

## Kết luận {#conclusion}

Cảm ơn bạn đã tham gia cùng chúng tôi để xem lại kết quả của cuộc khảo sát nhà phát triển đầu tiên năm 2023 của chúng tôi! Hiểu được kinh nghiệm và thách thức của nhà phát triển giúp chúng tôi ưu tiên cách phục vụ tốt nhất cho cộng đồng Go. Một số điểm rút ra mà chúng tôi thấy đặc biệt hữu ích:

* Các nhà phát triển Go mới có nhiều xu hướng phát triển web hơn so với người tham gia ở các mức kinh nghiệm khác. Đây là lĩnh vực chúng tôi muốn khám phá thêm để đảm bảo chúng tôi đáp ứng nhu cầu của các nhà phát triển Go mới.
* Mặc định an toàn, hướng dẫn phương pháp hay nhất về bảo mật và tối ưu hóa, và hỗ trợ refactoring nhiều hơn trong IDE sẽ là những bổ sung có giá trị cho cộng đồng.
* Xử lý lỗi là vấn đề ưu tiên cao cho cộng đồng và tạo ra thách thức về sự dài dòng và khả năng debug. Nhóm Go chưa có đề xuất công khai để chia sẻ vào lúc này nhưng đang tiếp tục khám phá các tùy chọn để cải thiện xử lý lỗi.
* Onboarding và học các phương pháp hay nhất là một trong những thách thức hàng đầu của người tham gia và sẽ là các lĩnh vực nghiên cứu trong tương lai.
* Đối với những người duy trì module Go, giữ cho dependency được cập nhật, quản lý phiên bản module, và xác định hoặc tránh các thay đổi phá vỡ tương thích là những thách thức lớn nhất. Giúp người duy trì cung cấp một hệ sinh thái ổn định và lành mạnh là một chủ đề quan tâm khác cho nghiên cứu UX tiếp theo.

Cảm ơn một lần nữa tất cả những người đã phản hồi và đóng góp cho cuộc khảo sát này, chúng tôi không thể làm được điều đó nếu không có bạn. Chúng tôi hy vọng gặp lại bạn vào cuối năm nay trong cuộc khảo sát tiếp theo.
