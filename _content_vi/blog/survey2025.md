---
title: Kết Quả Go Developer Survey 2025
date: 2026-01-21
by:
- Todd Kulesza, on behalf of the Go team
tags:
- survey
- community
- research
summary: Kết quả Go Developer Survey 2025, tập trung vào tâm lý của nhà phát triển đối với Go, các trường hợp sử dụng, thách thức và môi trường phát triển.
---

<style type="text/css" scoped>
  blockquote p {
    color: var(--color-text-subtle) !important;
  }

  .chart {
    margin-left: 1.5rem;
    margin-right: 1.5rem;
    width: 800px;
  }

  .quote_source {
    font-style: italic;
  }

  @media (prefers-color-scheme: dark) {
    .chart {
      border-radius: 8px;
    }
  }
</style>

Xin chào! Trong bài viết này, chúng tôi sẽ thảo luận về kết quả của Go Developer Survey 2025, được tiến hành trong tháng 9 năm 2025.

Cảm ơn 5.379 nhà phát triển Go đã phản hồi lời mời khảo sát của chúng tôi năm nay. Phản hồi của bạn giúp cả nhóm Go tại Google và cộng đồng Go rộng lớn hơn hiểu trạng thái hiện tại của hệ sinh thái Go và ưu tiên các dự án cho năm tới.

Ba phát hiện lớn nhất của chúng tôi là:

- Nhìn chung, các nhà phát triển Go yêu cầu được giúp đỡ trong việc xác định và áp dụng các thực hành tốt nhất, tận dụng tối đa thư viện chuẩn, và mở rộng ngôn ngữ cũng như hệ thống công cụ tích hợp sẵn với nhiều khả năng hiện đại hơn.
- Hầu hết các nhà phát triển Go hiện đang sử dụng các công cụ phát triển được hỗ trợ bởi AI khi tìm kiếm thông tin (ví dụ: tìm hiểu cách sử dụng một module) hoặc thực hiện các công việc lặp đi lặp lại (ví dụ: viết các khối mã tương tự lặp đi lặp lại), nhưng mức độ hài lòng của họ với những công cụ này ở mức trung bình, một phần là do lo ngại về chất lượng.
- Một tỷ lệ đáng ngạc nhiên cao của người trả lời cho biết họ thường xuyên cần xem tài liệu cho các lệnh con `go` cốt lõi, bao gồm `go build`, `go run` và `go mod`, gợi ý có nhiều cơ hội đáng kể để cải thiện hệ thống trợ giúp của lệnh `go`.

Đọc tiếp để biết chi tiết về các phát hiện này và nhiều hơn nữa.

## Các phần

- <a href="#demographics">Chúng tôi đã lắng nghe từ ai?</a>
- <a href="#sentiment">Mọi người cảm thấy như thế nào về Go?</a>
- <a href="#uses">Mọi người đang xây dựng gì với Go?</a>
- <a href="#challenges">Những thách thức lớn nhất mà các nhà phát triển Go đang đối mặt là gì?</a>
- <a href="#devenv">Môi trường phát triển của họ trông như thế nào?</a>
- <a href="#methodology">Phương pháp khảo sát</a>

## Chúng tôi đã lắng nghe từ ai? {#demographics}

Hầu hết người trả lời khảo sát tự xác định là nhà phát triển chuyên nghiệp (87%) sử dụng Go cho công việc chính của họ (82%). Đa số lớn cũng sử dụng Go cho các dự án cá nhân hoặc mã nguồn mở (72%). Hầu hết người trả lời nằm trong độ tuổi 25 &ndash; 45 (68%) với ít nhất sáu năm kinh nghiệm phát triển chuyên nghiệp (75%). Đi sâu hơn, 81% người trả lời cho chúng tôi biết họ có nhiều kinh nghiệm phát triển chuyên nghiệp hơn kinh nghiệm đặc thù với Go, bằng chứng mạnh mẽ rằng Go thường không phải là ngôn ngữ đầu tiên mà các nhà phát triển làm việc. Thực tế, một trong những chủ đề liên tục xuất hiện trong phân tích khảo sát năm nay dường như bắt nguồn từ thực tế này: khi cách thực hiện một nhiệm vụ trong Go khác biệt đáng kể so với một ngôn ngữ quen thuộc hơn, nó tạo ra ma sát cho các nhà phát triển khi họ phải học mẫu thuần ngữ Go mới (đối với họ), và sau đó nhớ lại những khác biệt này một cách nhất quán khi tiếp tục làm việc với nhiều ngôn ngữ. Chúng tôi sẽ quay lại chủ đề này sau.

Ngành phổ biến nhất mà người trả lời làm việc là "Công nghệ" (46%), nhưng đa số người trả lời làm việc bên ngoài ngành công nghệ (54%). Chúng tôi thấy sự đại diện của tất cả quy mô tổ chức, với đa số nhỏ làm việc ở nơi có 2 &ndash; 500 nhân viên (51%), 9% làm việc một mình, và 30% làm việc tại các doanh nghiệp có hơn 1.000 nhân viên. Như các năm trước, đa số câu trả lời đến từ Bắc Mỹ và Châu Âu.

Năm nay chúng tôi quan sát thấy sự giảm trong tỷ lệ người trả lời cho biết họ khá mới với Go, đã làm việc với nó chưa đến một năm (13%, so với 21% năm 2024). Chúng tôi nghi ngờ điều này liên quan đến <a href="https://digitaleconomy.stanford.edu/wp-content/uploads/2025/08/Canaries_BrynjolfssonChandarChen.pdf">sự suy giảm trên toàn ngành trong các vai trò kỹ thuật phần mềm cấp độ đầu vào</a>; chúng tôi thường nghe từ mọi người rằng họ học Go cho một công việc cụ thể, vì vậy sự suy giảm tuyển dụng sẽ được kỳ vọng sẽ giảm số lượng nhà phát triển học Go trong năm đó. Giả thuyết này được hỗ trợ thêm bởi phát hiện của chúng tôi rằng hơn 80% người trả lời đã học Go _sau_ khi bắt đầu sự nghiệp chuyên nghiệp của họ.

Ngoài những điều trên, chúng tôi không tìm thấy bất kỳ thay đổi đáng kể nào trong các nhân khẩu học khác kể từ cuộc khảo sát năm 2024.

<img src="survey2025/where.svg" alt="Trong năm qua, bạn đã sử dụng Go trong những loại tình huống nào?" class="chart" /> <img
src="survey2025/go_exp.svg" alt="Bạn đã sử dụng Go được bao lâu?"
class="chart" /> <img src="survey2025/role.svg" alt="Điều nào sau đây mô tả tốt nhất cách hoặc lý do bạn làm việc với Go?" class="chart" /> <img
src="survey2025/age.svg" alt="Bạn bao nhiêu tuổi?" class="chart" /> <img
src="survey2025/pro_dev_exp.svg" alt="Bạn có bao nhiêu năm kinh nghiệm lập trình chuyên nghiệp?" class="chart" /> <img src="survey2025/org_size.svg"
alt="Có bao nhiêu người làm việc trong tổ chức của bạn?" class="chart" /> <img
src="survey2025/industry.svg" alt="Điều nào sau đây mô tả tốt nhất ngành mà tổ chức của bạn hoạt động?" class="chart" /> <img
src="survey2025/location.svg" alt="Bạn sống ở đâu?" class="chart" />

## Mọi người cảm thấy như thế nào về Go? {#sentiment}

Đa số lớn người trả lời (91%) cho biết họ cảm thấy hài lòng khi làm việc với Go. Gần ⅔ là "rất hài lòng", đây là mức đánh giá cao nhất. Cả hai số liệu này đều rất tích cực và ổn định kể từ khi chúng tôi bắt đầu đặt câu hỏi này vào năm 2019. Tính ổn định theo thời gian thực sự là điều chúng tôi theo dõi từ số liệu này, vì chúng tôi coi nó như một chỉ số trễ, có nghĩa là đến khi số liệu hài lòng này cho thấy sự thay đổi có ý nghĩa, chúng tôi sẽ kỳ vọng đã thấy các tín hiệu sớm hơn từ các báo cáo vấn đề, danh sách gửi thư, hoặc phản hồi cộng đồng khác.

<img src="survey2025/csat.svg" alt="Nhìn chung, bạn hài lòng hay không hài lòng khi sử dụng Go trong năm qua như thế nào?" class="chart" />

Tại sao người trả lời lại tích cực như vậy về Go? Nhìn vào các câu trả lời văn bản mở cho một số câu hỏi khảo sát khác nhau cho thấy đó là tổng thể, chứ không phải bất kỳ một điều nào. Những người này đang nói với chúng tôi rằng họ thấy giá trị to lớn trong Go như một nền tảng toàn diện. Điều đó không có nghĩa là nó hỗ trợ tất cả các lĩnh vực lập trình đều tốt như nhau (chắc chắn là không), nhưng các nhà phát triển đánh giá cao các lĩnh vực mà nó _thực sự_ hỗ trợ tốt thông qua stdlib và hệ thống công cụ tích hợp sẵn.

Dưới đây là một số trích dẫn đại diện từ người trả lời. Để cung cấp bối cảnh cho mỗi trích dẫn, chúng tôi cũng xác định mức độ hài lòng, số năm kinh nghiệm với Go và ngành của người trả lời.

> "Go là ngôn ngữ yêu thích của tôi cho đến nay; các ngôn ngữ khác cảm thấy quá phức tạp và không hữu ích. Thực tế là Go tương đối nhỏ, đơn giản, với ít tính năng hào nhoáng đóng một vai trò lớn trong việc khiến nó trở thành nền tảng lâu bền tốt để xây dựng chương trình với nó. Tôi yêu rằng nó mở rộng tốt để được sử dụng bởi một lập trình viên duy nhất và trong các nhóm lớn." <span
> class="quote_source">&mdash; Rất hài lòng / 10+ năm / Công ty công nghệ</span>

> "Lý do duy nhất tôi sử dụng Go là hệ thống công cụ tuyệt vời và thư viện chuẩn. Tôi rất biết ơn nhóm đã tập trung vào HTTP, crypto, math, sync và các công cụ tuyệt vời khác giúp việc phát triển các ứng dụng hướng dịch vụ dễ dàng và đáng tin cậy." <span class="quote_source">&mdash; Rất hài lòng / 10+ năm / Công ty năng lượng</span>

> "Hệ sinh thái Go là lý do tại sao tôi thực sự thích ngôn ngữ lập trình này. Có nhiều vấn đề npm gần đây nhưng không có với Go." <span
> class="quote_source">&mdash; Rất hài lòng / 3 &ndash; 10 năm / Dịch vụ tài chính</span>

Năm nay chúng tôi cũng hỏi về các ngôn ngữ khác mà mọi người sử dụng. Người trả lời khảo sát cho biết ngoài Go, họ thích làm việc với Python, Rust và TypeScript, trong số nhiều ngôn ngữ khác. Một số đặc điểm chung của các ngôn ngữ này phù hợp với các điểm ma sát phổ biến được báo cáo bởi các nhà phát triển Go, bao gồm các lĩnh vực như xử lý lỗi, enum và các mẫu thiết kế hướng đối tượng. Ví dụ, khi chúng tôi cộng tỷ lệ người trả lời nói rằng ngôn ngữ yêu thích tiếp theo của họ bao gồm một trong các yếu tố sau, chúng tôi thấy rằng đa số người trả lời thích sử dụng các ngôn ngữ có kế thừa, enum an toàn kiểu và ngoại lệ, với chỉ đa số nhỏ của các ngôn ngữ này bao gồm hệ thống kiểu tĩnh theo mặc định.

| Khái niệm hoặc tính năng | Tỷ lệ người trả lời |
| --- | --- |
| Kế thừa | 71% |
| Enum an toàn kiểu | 65% |
| Ngoại lệ | 60% |
| Kiểu tĩnh | 51% |

Chúng tôi nghĩ điều này quan trọng vì nó tiết lộ môi trường rộng lớn hơn mà các nhà phát triển hoạt động, gợi ý rằng mọi người cần sử dụng các mẫu thiết kế khác nhau cho các nhiệm vụ khá bình thường, tùy thuộc vào ngôn ngữ của codebase họ đang làm việc. Điều này dẫn đến tải nhận thức bổ sung và nhầm lẫn, không chỉ trong số các nhà phát triển mới với Go (những người phải học các mẫu thiết kế thuần ngữ Go), mà còn trong số nhiều nhà phát triển làm việc trong nhiều codebase hoặc dự án. Một cách để giảm bớt tải bổ sung này là hướng dẫn theo ngữ cảnh cụ thể, chẳng hạn như hướng dẫn về "Xử lý lỗi trong Go cho các nhà phát triển Java". Thậm chí có thể có cơ hội để xây dựng một số hướng dẫn này vào các trình phân tích mã, giúp dễ dàng hiển thị trực tiếp trong IDE.

<img src="survey2025/fav_lang.svg" alt="Không tính Go, ngôn ngữ lập trình yêu thích của bạn là gì?" class="chart" />

Năm nay chúng tôi đã yêu cầu cộng đồng Go chia sẻ tâm lý của họ đối với dự án Go. Các kết quả này khá khác với tỷ lệ hài lòng 91% mà chúng tôi đã thảo luận ở trên, và chỉ ra các lĩnh vực mà nhóm Go có kế hoạch đầu tư năng lượng trong năm 2026. Đặc biệt, chúng tôi muốn khuyến khích nhiều người đóng góp hơn tham gia, và đảm bảo nhóm Go hiểu chính xác những thách thức mà các nhà phát triển Go hiện đang đối mặt. Chúng tôi hy vọng sự tập trung này sẽ giúp tăng sự tin tưởng của nhà phát triển vào cả dự án Go và lãnh đạo nhóm Go. Như một người trả lời đã giải thích vấn đề:

> "Bây giờ khi thế hệ đầu tiên thành lập nhóm Go không còn tham gia nhiều vào việc ra quyết định, tôi hơi lo lắng về tương lai của Go về chất lượng bảo trì, và các quyết định cân bằng cho đến nay liên quan đến các thay đổi trong ngôn ngữ và thư viện chuẩn. Sự hiện diện nhiều hơn dưới dạng các bài nói chuyện của các thành viên nhóm cốt lõi mới về trạng thái hiện tại và kế hoạch tương lai có thể hữu ích để củng cố niềm tin." <span
> class="quote_source">&mdash; Rất hài lòng / 10+ năm / Công ty công nghệ</span>

<img src="survey2025/trust.svg" alt="Bạn đồng ý hay không đồng ý với các nhận định sau đây ở mức độ nào?" class="chart" />

## Mọi người đang xây dựng gì với Go? {#uses}

Chúng tôi đã sửa lại danh sách "các loại thứ bạn xây dựng với Go?" từ năm 2024 với mục đích phân tích có ích hơn những gì mọi người đang xây dựng với Go, và tránh nhầm lẫn xung quanh các thuật ngữ đang phát triển như "agent". Các trường hợp sử dụng hàng đầu của người trả lời vẫn là CLI và dịch vụ API, không có thay đổi đáng kể nào kể từ năm 2024. Thực tế, đa số người trả lời (55%) cho biết họ xây dựng _cả_ CLI và dịch vụ API với Go. Hơn ⅓ người trả lời đặc biệt xây dựng hệ thống công cụ cơ sở hạ tầng đám mây (một danh mục mới), và 11% làm việc với các mô hình, công cụ hoặc agent ML (một danh mục được mở rộng). Thật không may, các trường hợp sử dụng nhúng đã bị bỏ qua trong danh sách sửa đổi, nhưng chúng tôi sẽ khắc phục điều này cho cuộc khảo sát năm tới.

<img src="survey2025/build.svg" alt="Các loại thứ bạn xây dựng với Go là gì?" class="chart" />

Hầu hết người trả lời cho biết họ hiện không xây dựng các tính năng hỗ trợ AI vào phần mềm Go họ làm việc (78%), với ⅔ báo cáo rằng phần mềm của họ không sử dụng chức năng AI nào cả (66%). Điều này có vẻ là sự giảm trong mức sử dụng AI liên quan đến production so với năm trước; vào năm 2024, 59% người trả lời không tham gia vào công việc tính năng AI, trong khi 39% cho biết một mức độ tham gia nào đó. Điều đó đánh dấu sự dịch chuyển 14 điểm ra xa việc xây dựng các hệ thống hỗ trợ AI trong số người trả lời khảo sát, và có thể phản ánh một số sự rút lui tự nhiên khỏi sự cường điệu ban đầu xung quanh các ứng dụng hỗ trợ AI: có thể nhiều người đã thử xem họ có thể làm gì với công nghệ này trong lần triển khai ban đầu, với một tỷ lệ nhất định quyết định không tiếp tục khám phá (ít nhất là lúc này).

<img src="survey2025/genai.svg" alt="Hãy nghĩ về phần mềm Go mà bạn đã làm việc nhiều nhất trong tháng qua. Nó có sử dụng AI cho bất kỳ chức năng nào của nó không?" class="chart" />

Trong số những người trả lời đang xây dựng chức năng hỗ trợ AI hoặc LLM, trường hợp sử dụng phổ biến nhất là tạo tóm tắt nội dung hiện có (45%). Nhìn chung, tuy nhiên, có ít sự khác biệt giữa hầu hết các cách sử dụng, với 28% &ndash; 33% người trả lời thêm chức năng AI để hỗ trợ phân loại, tạo nội dung, xác định giải pháp, chatbot và phát triển phần mềm.

<img src="survey2025/genai_use.svg" alt="Phần mềm Go tôi xây dựng sử dụng AI hoặc LLM để:" class="chart" />

## Những thách thức lớn nhất mà các nhà phát triển Go đang đối mặt là gì? {#challenges}

Một trong những loại phản hồi hữu ích nhất mà chúng tôi nhận được từ các nhà phát triển là chi tiết về những thách thức mà mọi người gặp phải khi làm việc với Go. Nhóm Go xem xét thông tin này một cách tổng thể và theo các khoảng thời gian dài, vì thường có sự căng thẳng giữa việc cải thiện các khía cạnh còn thô ráp của Go và giữ ngôn ngữ và hệ thống công cụ nhất quán cho các nhà phát triển. Ngoài các yếu tố kỹ thuật, mỗi thay đổi cũng phát sinh một chi phí nhất định về mặt sự chú ý của nhà phát triển và sự gián đoạn nhận thức. Giảm thiểu sự gián đoạn có vẻ hơi nhàm chán, nhưng chúng tôi coi đây là điểm mạnh quan trọng của Go. Như Russ Cox đã viết năm 2023, ["Nhàm chán là tốt... Nhàm chán có nghĩa là có thể tập trung vào công việc của bạn, không phải vào những gì khác biệt về Go."](/blog/compat).

Với tinh thần đó, các thách thức hàng đầu của năm nay không khác biệt căn bản so với năm ngoái. Ba khó chịu hàng đầu mà người trả lời báo cáo là "Đảm bảo mã Go của chúng tôi tuân theo các thực hành tốt nhất / thành ngữ Go" (33% người trả lời), "Một tính năng tôi đánh giá cao từ ngôn ngữ khác không có trong Go" (28%), và "Tìm các module và gói Go đáng tin cậy" (26%). Chúng tôi đã xem xét các câu trả lời văn bản mở để hiểu rõ hơn ý nghĩa của mọi người. Hãy dành một phút để đào sâu vào từng vấn đề.

Những người trả lời thất vọng nhất khi viết Go thuần ngữ thường tìm kiếm hướng dẫn chính thức hơn, cũng như hỗ trợ hệ thống công cụ để giúp thực thi hướng dẫn này trong codebase của họ. Như trong các cuộc khảo sát trước, các câu hỏi về cách cấu trúc dự án Go cũng là chủ đề phổ biến. Ví dụ:

> "Sự đơn giản của Go giúp đọc và hiểu mã từ các nhà phát triển khác, nhưng vẫn còn một số khía cạnh có thể khác nhau khá nhiều giữa các lập trình viên. Đặc biệt nếu các nhà phát triển đến từ các ngôn ngữ khác, ví dụ Java." <span class="quote_source">&mdash; Rất hài lòng / 3 &ndash; 10 năm / Chăm sóc sức khỏe và khoa học đời sống</span>

> "Cách viết mã Go có quan điểm hơn. Như cách cấu trúc dự án Go cho các dịch vụ/công cụ CLI." <span class="quote_source">&mdash; Rất hài lòng /
> < 3 năm / Công nghệ</span>

> "Khó tìm ra các thành ngữ tốt là gì. Đặc biệt vì nhóm cốt lõi không cập nhật Effective Go." <span
> class="quote_source">&mdash; Rất hài lòng / 3 &ndash; 10 năm /
> Công nghệ</span>

Danh mục thất vọng lớn thứ hai là các tính năng ngôn ngữ mà các nhà phát triển thích làm việc trong các hệ sinh thái khác. Các bình luận văn bản mở này tập trung chủ yếu vào các mẫu xử lý và báo cáo lỗi, enum và các loại tổng hợp, an toàn con trỏ nil, và tính biểu đạt / tính dài dòng chung:

> "Vẫn chưa chắc cách xử lý lỗi tốt nhất là gì." <span
> class="quote_source">&mdash; Rất hài lòng / 3 &ndash; 10 năm / Bán lẻ và hàng tiêu dùng</span>

> "Enum của Rust rất tuyệt, và dẫn đến việc viết mã an toàn kiểu tuyệt vời." <span
> class="quote_source">&mdash; Hơi hài lòng / 3 &ndash; 10 năm /
> Chăm sóc sức khỏe và khoa học đời sống</span>

> "Không có gì (trong trình biên dịch) ngăn tôi sử dụng con trỏ có thể nil, hoặc sử dụng giá trị mà không kiểm tra err trước. Điều đó nên được tích hợp vào hệ thống kiểu." <span class="quote_source">&mdash; Hơi hài lòng
> / < 3 năm / Công nghệ</span>

> "Tôi thích [Go] nhưng tôi không ngờ nó lại có ngoại lệ con trỏ nil :)" <span
> class="quote_source">&mdash; Hơi hài lòng / 3 &ndash; 10 năm /
> Dịch vụ tài chính</span>

> "Tôi thường thấy khó xây dựng các trừu tượng và cung cấp ý định rõ ràng cho những người đọc mã của tôi trong tương lai." <span class="quote_source">&mdash;
> Hơi không hài lòng / < 3 năm / Công nghệ</span>

Thất vọng lớn thứ ba là tìm kiếm các module Go đáng tin cậy. Người trả lời thường mô tả hai khía cạnh của vấn đề này. Một là họ coi nhiều module bên thứ ba có chất lượng trung bình, khiến việc nổi bật của các module thực sự tốt trở nên khó khăn. Thứ hai là xác định module nào được sử dụng phổ biến và trong điều kiện nào (bao gồm cả xu hướng gần đây theo thời gian). Đây đều là các vấn đề có thể được giải quyết bằng cách hiển thị những gì chúng tôi sẽ gọi mơ hồ là "tín hiệu chất lượng" trên pkg.go.dev. Người trả lời đã cung cấp các giải thích hữu ích về các tín hiệu họ sử dụng để xác định các module đáng tin cậy, bao gồm hoạt động dự án, chất lượng mã, xu hướng áp dụng gần đây, hoặc các tổ chức cụ thể hỗ trợ hoặc dựa vào module đó.

> "Có thể lọc theo các tiêu chí như phiên bản ổn định, số lượng người dùng và tuổi bản cập nhật cuối cùng tại pkg.go.dev có thể giúp mọi thứ dễ dàng hơn một chút." <span
> class="quote_source">&mdash; Rất hài lòng / < 3 năm / Công nghệ</span>

> "Nhiều gói chỉ là bản sao/fork hoặc dự án một lần không có lịch sử/bảo trì." <span class="quote_source">&mdash; Rất
> hài lòng / 10+ năm / Dịch vụ tài chính</span>

> "Có thể gắn cờ các gói đáng tin cậy dựa trên kinh nghiệm, độ trưởng thành và phản hồi cộng đồng?" <span class="quote_source">&mdash; Rất hài lòng / 3
> &ndash; 10 năm / Chăm sóc sức khỏe và khoa học đời sống</span>

Chúng tôi đồng ý rằng đây đều là các lĩnh vực mà trải nghiệm nhà phát triển với Go có thể được cải thiện. Thách thức, như đã thảo luận trước đó, là làm như vậy theo cách không dẫn đến các thay đổi đột phá, tăng nhầm lẫn trong số các nhà phát triển Go, hoặc cản trở mọi người cố gắng hoàn thành công việc của họ với Go. Phản hồi từ cuộc khảo sát này là một nguồn thông tin chính mà chúng tôi sử dụng khi thảo luận về các đề xuất, nhưng nếu bạn muốn tham gia trực tiếp hơn hoặc theo dõi với các người đóng góp khác, hãy truy cập [các đề xuất Go trên GitHub](https://github.com/golang/go/issues?q=state%3Aopen%20label%3AProposal); hãy đảm bảo [tuân theo quy trình này](https://github.com/golang/proposal) nếu bạn muốn thêm đề xuất mới.

<img src="survey2025/frustrations.svg" alt="Ba điều khó chịu nhất của bạn khi làm việc với Go là gì?" class="chart" />

Ngoài những thách thức (có thể) trên toàn hệ sinh thái này, năm nay chúng tôi cũng hỏi cụ thể về việc làm việc với lệnh `go`. Chúng tôi đã nghe không chính thức từ các nhà phát triển rằng hệ thống trợ giúp của công cụ này có thể gây nhầm lẫn khi điều hướng, nhưng chúng tôi chưa có cảm nhận rõ ràng về tần suất mọi người thấy mình cần xem tài liệu này.

Người trả lời cho chúng tôi biết rằng trừ `go test`, 15% &ndash; 25% trong số họ cảm thấy họ "thường xuyên cần xem tài liệu" khi làm việc với các công cụ này. Điều này thật ngạc nhiên, đặc biệt là đối với các lệnh con thường được sử dụng như `build` và `run`. Các lý do phổ biến bao gồm việc nhớ các cờ cụ thể, hiểu các tùy chọn khác nhau làm gì, và điều hướng chính hệ thống trợ giúp. Người tham gia cũng xác nhận rằng việc sử dụng không thường xuyên là một lý do gây thất vọng, nhưng việc điều hướng và phân tích cú pháp trợ giúp lệnh có vẻ là nguyên nhân cơ bản. Nói cách khác, tất cả chúng ta đều kỳ vọng phải xem tài liệu đôi khi, nhưng chúng ta không kỳ vọng phải cần giúp đỡ để điều hướng chính hệ thống tài liệu. Như một người trả lời đã mô tả hành trình của họ:

> "Truy cập trợ giúp thật đau đớn. go test --help # không hoạt động, nhưng cho tôi biết phải gõ `go help test` thay thế... go help test # ồ, thực ra, thông tin tôi đang tìm kiếm có trong `testflag` go help testflag # phân tích cú pháp qua văn bản trông tất cả đều giống nhau mà không có nhiều định dạng... Tôi chỉ không có thời gian để đào sâu vào cái hố thỏ này." <span class="quote_source">&mdash; Rất
> hài lòng / 10+ năm / Công nghệ</span>

<img src="survey2025/go_help.svg" alt="Bạn có thấy mình thường xuyên xem tài liệu cho bất kỳ lệnh con Go nào sau đây không?"
class="chart" />

## Môi trường phát triển của họ trông như thế nào? {#devenv}

### Hệ điều hành và kiến trúc

Nhìn chung, người trả lời cho chúng tôi biết nền tảng phát triển của họ là kiểu UNIX. Hầu hết người trả lời phát triển trên macOS (60%) hoặc Linux (58%) và triển khai lên các hệ thống dựa trên Linux, bao gồm container (96%). Thay đổi lớn nhất từ năm trước đến năm nay là trong các triển khai "thiết bị nhúng / IoT", tăng từ 2% lên 8% người trả lời; đây là thay đổi có ý nghĩa duy nhất trong các nền tảng triển khai kể từ năm 2024.

Đa số lớn người trả lời phát triển trên kiến trúc x86-64 hoặc ARM64, với một nhóm đáng kể (25%) vẫn có thể đang làm việc trên các hệ thống x86 32-bit. Tuy nhiên, chúng tôi tin rằng cách đặt câu hỏi này gây nhầm lẫn cho người trả lời; năm tới chúng tôi sẽ làm rõ sự phân biệt 32-bit so với 64-bit cho mỗi kiến trúc.

<img src="survey2025/os_dev.svg" alt="Các nền tảng nào bạn sử dụng khi viết mã Go?" class="chart" /> <img src="survey2025/deploy.svg" alt="Các hệ thống nào bạn triển khai phần mềm Go?" class="chart" /> <img
src="survey2025/arch.svg" alt="Các kiến trúc nào bạn triển khai phần mềm Go?" class="chart" />

### Trình soạn thảo mã

Một số trình soạn thảo mã mới đã có sẵn trong hai năm qua, và chúng tôi đã mở rộng câu hỏi khảo sát để bao gồm những cái phổ biến nhất. Trong khi chúng tôi thấy một số bằng chứng về việc áp dụng sớm, hầu hết người trả lời tiếp tục ưa thích [VS Code](https://code.visualstudio.com/) (37%) hoặc [GoLand](https://www.jetbrains.com/go/) (28%). Trong số các trình soạn thảo mới hơn, Zed và Cursor được xếp hạng cao nhất, mỗi loại trở thành trình soạn thảo ưa thích của 4% người trả lời. Để đặt những con số đó vào ngữ cảnh, chúng tôi đã nhìn lại khi VS Code và GoLand được giới thiệu lần đầu. VS Code (ra mắt năm 2015) được ưa thích bởi 16% người trả lời một năm sau khi ra mắt. IntelliJ có plugin Go do cộng đồng dẫn dắt lâu hơn kể từ khi chúng tôi khảo sát các nhà phát triển Go (💙), nhưng nếu chúng tôi nhìn vào khi JetBrains bắt đầu hỗ trợ Go chính thức trong IntelliJ (2016), trong vòng một năm IntelliJ được ưa thích bởi 20% người trả lời.

Lưu ý: Phân tích này về các trình soạn thảo mã không bao gồm những người trả lời được giới thiệu trực tiếp đến cuộc khảo sát từ VS Code hoặc GoLand.

<img src="survey2025/editor.svg" alt="Trình soạn thảo mã yêu thích của bạn cho Go là gì?" class="chart" />

### Môi trường đám mây

Các môi trường triển khai phổ biến nhất cho Go tiếp tục là Amazon Web Services (AWS) với 46% người trả lời, các máy chủ do công ty sở hữu (44%), và Google Cloud Platform (GCP) với 26%. Những con số này cho thấy sự thay đổi nhỏ kể từ năm 2024, nhưng không có gì có ý nghĩa thống kê. Chúng tôi thấy rằng danh mục "Khác" đã tăng lên 11% năm nay, và điều này chủ yếu do Hetzner (20% câu trả lời "Khác"); chúng tôi có kế hoạch bao gồm Hetzner như một lựa chọn câu trả lời trong cuộc khảo sát năm tới.

Chúng tôi cũng hỏi người trả lời về trải nghiệm phát triển của họ khi làm việc với các nhà cung cấp đám mây khác nhau. Các câu trả lời phổ biến nhất, tuy nhiên, cho thấy rằng người trả lời không thực sự chắc chắn (46%) hoặc không tương tác trực tiếp với các nhà cung cấp đám mây công cộng (21%). Yếu tố chính đằng sau các câu trả lời này là một chủ đề mà chúng tôi đã nghe thường xuyên trước đây: với container, có thể trừu tượng hóa nhiều chi tiết của môi trường đám mây khỏi nhà phát triển, vì vậy họ không tương tác có ý nghĩa với hầu hết các công nghệ đặc thù của nhà cung cấp. Kết quả này gợi ý rằng ngay cả các nhà phát triển có công việc được _triển khai_ lên đám mây cũng có thể có kinh nghiệm hạn chế với bộ công cụ và công nghệ lớn hơn liên quan đến từng nhà cung cấp đám mây. Ví dụ:

> "Khá trừu tượng với nền tảng, Go rất dễ đặt vào container và vì vậy khá dễ triển khai ở bất cứ đâu: đây là một trong những điểm mạnh lớn của nó." <span
> class="quote_source">&mdash; [không có câu trả lời hài lòng] / 3 &ndash; 10 năm
> / Công nghệ</span>

> "Nhà cung cấp đám mây thực sự không tạo ra nhiều sự khác biệt với tôi. Tôi viết mã và triển khai nó vào container, vì vậy dù đó là AWS hay GCP, tôi thực sự không quan tâm." <span class="quote_source">&mdash; Hơi hài lòng / 3 &ndash; 10 năm / Dịch vụ tài chính</span>

Chúng tôi nghi ngờ mức độ trừu tượng này phụ thuộc vào trường hợp sử dụng và yêu cầu của dịch vụ được triển khai, có thể không phải lúc nào cũng có lý hoặc có thể giữ nó ở mức độ trừu tượng cao. Trong tương lai, chúng tôi có kế hoạch điều tra thêm cách các nhà phát triển Go thường tương tác với các nền tảng nơi phần mềm của họ cuối cùng được triển khai.

<img src="survey2025/work_deploy.svg" alt="Nhóm của tôi tại công việc triển khai chương trình Go đến:" class="chart" />

<img src="survey2025/favorite_go_cloud.svg" alt="Theo kinh nghiệm của bạn, nhà cung cấp đám mây công cộng nào cung cấp trải nghiệm tốt nhất cho các nhà phát triển Go?"
class="chart" />

### Phát triển với AI

Cuối cùng, chúng tôi không thể thảo luận về môi trường phát triển vào năm 2025 mà không đề cập đến các công cụ phát triển phần mềm hỗ trợ AI. Cuộc khảo sát của chúng tôi gợi ý việc áp dụng phân chia, với đa số người trả lời (53%) cho biết họ sử dụng các công cụ như vậy hàng ngày, nhưng cũng có một nhóm lớn (29%) không sử dụng chúng chút nào, hoặc chỉ sử dụng chúng một vài lần trong tháng qua. Chúng tôi kỳ vọng điều này có tương quan nghịch với tuổi hoặc kinh nghiệm phát triển, nhưng không thể tìm thấy bằng chứng mạnh hỗ trợ lý thuyết này ngoại trừ các nhà phát triển *rất* mới: những người trả lời có ít hơn một năm kinh nghiệm phát triển chuyên nghiệp (không đặc thù cho Go) thực sự báo cáo sử dụng AI nhiều hơn mọi nhóm khác, nhưng nhóm này chỉ chiếm 2% người trả lời khảo sát.

<img src="survey2025/ai_freq.svg" alt="Trong tháng qua, bạn thường xuyên sử dụng các công cụ hỗ trợ AI khi viết Go như thế nào?" class="chart" />

Hiện tại, việc sử dụng AI theo kiểu agent trong các công cụ phát triển hỗ trợ AI có vẻ còn sơ khai trong số các nhà phát triển Go, với chỉ 17% người trả lời cho biết đây là cách chính họ sử dụng các công cụ đó, mặc dù một nhóm lớn hơn (40%) thỉnh thoảng đang thử các chế độ hoạt động theo kiểu agent.

<img src="survey2025/ai_agent.svg" alt="Khi làm việc với các công cụ phát triển hỗ trợ AI, bạn có xu hướng sử dụng chúng như các agent không có giám sát không?"
class="chart" />

Các trợ lý AI được sử dụng phổ biến nhất vẫn là ChatGPT, GitHub Copilot và Claude. Hầu hết các agent này cho thấy số lượng sử dụng thấp hơn [so với cuộc khảo sát năm 2024 của chúng tôi](/blog/survey2024-h2-results#ai-assistance) (Claude và Cursor là những ngoại lệ đáng chú ý), nhưng do thay đổi phương pháp, đây không phải là so sánh táo với táo. Tuy nhiên, hoàn toàn có thể là các nhà phát triển đang "mua sắm ít hơn" so với khi các công cụ này lần đầu ra mắt, dẫn đến nhiều người hơn sử dụng một trợ lý duy nhất cho hầu hết công việc của họ.

<img src="survey2025/ai_asst.svg" alt="Khi viết mã Go, những trợ lý hoặc agent AI nào bạn đã sử dụng trong tháng qua?" class="chart" />

Chúng tôi cũng hỏi về mức độ hài lòng tổng thể với các công cụ phát triển hỗ trợ AI. Đa số (55%) báo cáo hài lòng, nhưng điều này có trọng số mạnh hướng tới danh mục "Hơi hài lòng" (42%) so với nhóm "Rất hài lòng" (13%). Hãy nhớ rằng bản thân Go liên tục cho thấy tỷ lệ hài lòng 90%+ mỗi năm; năm nay, 62% người trả lời nói họ "Rất hài lòng" với Go. Chúng tôi thêm ngữ cảnh này để cho thấy rằng trong khi hệ thống công cụ hỗ trợ AI đang bắt đầu được áp dụng và tìm thấy một số trường hợp sử dụng thành công, tâm lý của nhà phát triển đối với chúng vẫn mềm mỏng hơn nhiều so với hệ thống công cụ đã được thiết lập tốt hơn (ít nhất là trong số các nhà phát triển Go).

Điều gì đang thúc đẩy tỷ lệ hài lòng thấp hơn này? Trong một từ: chất lượng. Chúng tôi đã yêu cầu người trả lời cho chúng tôi biết điều tốt họ đã hoàn thành với những công cụ này, cũng như điều gì đó không diễn ra tốt. Đa số cho biết tạo mã không hoạt động là vấn đề chính của họ với các công cụ nhà phát triển AI (53%), với 30% than thở rằng ngay cả mã hoạt động cũng có chất lượng kém. Các lợi ích được đề cập thường xuyên nhất, ngược lại, là tạo bài kiểm thử đơn vị, viết mã boilerplate, hoàn thành mã được cải thiện, tái cấu trúc và tạo tài liệu. Những điều này có vẻ là các trường hợp mà chất lượng mã được coi là ít quan trọng hơn, nghiêng cán cân có lợi cho việc để AI thực hiện lần đầu tiên. Tuy nhiên, người trả lời cũng cho chúng tôi biết rằng mã do AI tạo ra trong các trường hợp thành công này vẫn cần được xem xét cẩn thận (và thường là sửa chữa), vì nó có thể có lỗi, không an toàn, hoặc thiếu ngữ cảnh.

> "Tôi không bao giờ hài lòng với chất lượng mã hoặc tính nhất quán, nó không bao giờ tuân theo các thực hành tôi muốn." <span class="quote_source">&mdash; [không có câu trả lời hài lòng]
> / 3 &ndash; 10 năm / Dịch vụ tài chính</span>

> "Tất cả các công cụ AI có xu hướng ảo giác nhanh chóng khi làm việc với các codebase vừa đến lớn (10k+ dòng mã). Chúng có thể giải thích mã hiệu quả nhưng gặp khó khăn khi tạo các tính năng phức tạp mới" <span
> class="quote_source">&mdash; Hơi hài lòng / 3 &ndash; 10 năm /
> Bán lẻ và hàng tiêu dùng</span>

> "Dù có nhiều nỗ lực để khiến nó viết mã trong một codebase đã thiết lập, cần quá nhiều công sức để hướng dẫn nó tuân theo các thực hành trong dự án, và nó sẽ thêm các đường dẫn hành vi tinh tế, tức là nếu nó bỏ lỡ một phương thức, nó sẽ cố gắng tìm cách xung quanh nó hoặc dựa vào một số tác dụng phụ. Đôi khi những điều đó khó nhận ra trong quá trình xem xét mã. Tôi cũng thấy việc xem xét mã do AI tạo ra thật mệt mỏi về mặt tinh thần và chi phí đó giết chết tiềm năng năng suất khi viết mã." <span
> class="quote_source">&mdash; Rất hài lòng / 10+ năm / Công nghệ</span>

<img src="survey2025/ai_csat.svg" alt="Nhìn chung, bạn hài lòng hay không hài lòng khi làm việc với các công cụ phát triển hỗ trợ AI trong tháng qua như thế nào?" class="chart">

Khi chúng tôi hỏi các nhà phát triển họ sử dụng những công cụ này cho mục đích gì, một mẫu nổi lên nhất quán với những lo ngại về chất lượng này. Các nhiệm vụ được áp dụng nhiều nhất (màu xanh lá trong biểu đồ bên dưới) và ít kháng cự nhất (màu đỏ) liên quan đến việc thu hẹp khoảng cách kiến thức, cải thiện mã cục bộ và tránh công việc nhàm chán. Những thất vọng mà các nhà phát triển nói về với các công cụ tạo mã ít rõ ràng hơn nhiều khi họ đang tìm kiếm thông tin, như cách sử dụng một API cụ thể hoặc cấu hình độ bao phủ kiểm thử, và có thể do đó, chúng tôi thấy mức sử dụng AI cao hơn trong các lĩnh vực này. Một điểm nổi bật khác là xem xét mã _cục bộ_ và các gợi ý liên quan, mọi người ít quan tâm đến việc sử dụng AI để xem xét mã của người khác hơn là xem xét mã của chính họ. Đáng ngạc nhiên, "kiểm thử mã" cho thấy mức độ áp dụng AI thấp hơn các nhiệm vụ nhàm chán khác, mặc dù chúng tôi chưa có hiểu biết sâu sắc về lý do tại sao.

Trong tất cả các nhiệm vụ chúng tôi hỏi, "Viết mã" có sự phân chia mạnh nhất, với 66% người trả lời đã hoặc hy vọng sớm sử dụng AI cho việc này, trong khi ¼ người trả lời không muốn AI tham gia chút nào. Các câu trả lời mở gợi ý rằng các nhà phát triển chủ yếu sử dụng điều này cho các mã nhàm chán, lặp đi lặp lại, và tiếp tục lo ngại về chất lượng của mã do AI tạo ra.

<img src="survey2025/ai_tools_what.svg" alt="Bạn đang sử dụng các công cụ hỗ trợ AI với Go như thế nào ngày nay?" class="chart" />

## Kết luận

Một lần nữa, xin cảm ơn rất nhiều tất cả những ai đã phản hồi Go Developer Survey năm nay!

Chúng tôi có kế hoạch chia sẻ bộ dữ liệu khảo sát thô trong Q1 năm 2026, để cộng đồng lớn hơn cũng có thể khám phá dữ liệu cơ bản của các phát hiện này. Điều này sẽ chỉ bao gồm câu trả lời từ những người đã chọn chia sẻ dữ liệu này (82% trong tổng số người trả lời), vì vậy có thể có một số khác biệt so với các con số chúng tôi đề cập trong bài đăng này.

## Phương pháp khảo sát {#methodology}

Cuộc khảo sát này được tiến hành từ ngày 9 tháng 9 đến ngày 30 tháng 9 năm 2025. Người tham gia được mời công khai phản hồi qua Blog Go, lời mời trên các kênh truyền thông xã hội (bao gồm Bluesky, Mastodon, Reddit và X), cũng như các lời mời trong sản phẩm ngẫu nhiên cho những người sử dụng VS Code và GoLand để viết phần mềm Go. Chúng tôi nhận được tổng cộng 7.070 phản hồi. Sau khi làm sạch dữ liệu để loại bỏ bot và các phản hồi chất lượng rất thấp khác, 5.379 phản hồi được sử dụng cho phần còn lại của phân tích. Thời gian phản hồi khảo sát trung bình là 12 &ndash; 13 phút.

Trong suốt báo cáo này, chúng tôi sử dụng biểu đồ câu trả lời khảo sát để cung cấp bằng chứng hỗ trợ cho các phát hiện của chúng tôi. Tất cả các biểu đồ này sử dụng định dạng tương tự. Tiêu đề là câu hỏi chính xác mà người trả lời khảo sát đã thấy. Trừ khi có ghi chú khác, các câu hỏi là nhiều lựa chọn và người tham gia chỉ có thể chọn một câu trả lời duy nhất; phụ đề của mỗi biểu đồ sẽ cho người đọc biết nếu câu hỏi cho phép nhiều câu trả lời hoặc là hộp văn bản mở thay vì câu hỏi nhiều lựa chọn. Đối với biểu đồ của câu trả lời văn bản mở, một thành viên nhóm Go đã đọc và phân loại thủ công tất cả các câu trả lời. Nhiều câu hỏi mở đã gợi ra nhiều câu trả lời khác nhau; để giữ kích thước biểu đồ hợp lý, chúng tôi đã nén chúng thành tối đa 10-12 chủ đề hàng đầu, với các chủ đề bổ sung tất cả được nhóm dưới "Khác". Nhãn phần trăm được hiển thị trong biểu đồ được làm tròn đến số nguyên gần nhất (ví dụ, 1,4% và 0,8% sẽ cả hai đều được hiển thị là 1%), nhưng độ dài của mỗi thanh và thứ tự hàng dựa trên các giá trị chưa làm tròn.

Để giúp người đọc hiểu trọng lượng bằng chứng cơ bản mỗi phát hiện, chúng tôi đã thêm các thanh lỗi hiển thị [khoảng tin cậy](https://en.wikipedia.org/wiki/Confidence_interval) 95% cho các câu trả lời; thanh hẹp hơn cho thấy độ tin cậy tăng. Đôi khi hai hoặc nhiều câu trả lời có các thanh lỗi chồng chéo, điều đó có nghĩa là thứ tự tương đối của các câu trả lời đó không có ý nghĩa thống kê (tức là các câu trả lời thực sự bằng nhau). Góc dưới bên phải của mỗi biểu đồ hiển thị số người có câu trả lời được bao gồm trong biểu đồ, ở dạng "n = [số người trả lời]".
