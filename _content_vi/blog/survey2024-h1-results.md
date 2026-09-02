---
title: Kết quả khảo sát Go Developer Survey 2024 H1
date: 2024-4-09
by:
- Alice Merrick
- Todd Kulesza
tags:
- survey
- community
- developer experience research
summary: Những gì chúng tôi tìm hiểu được từ khảo sát lập trình viên H1 2024
---

<style type="text/css" scoped>
  .chart {
    margin-left: 1.5rem;
    margin-right: 1.5rem;
    width: 800px;
  }
  blockquote p {
    color: var(--color-text-subtle) !important;
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

## Bối cảnh

Bài viết này chia sẻ kết quả của Go Developer Survey gần đây nhất, được thực hiện vào tháng 1 và tháng 2 năm 2024. Ngoài việc ghi lại tâm lý và thách thức xung quanh việc sử dụng Go và hệ thống công cụ Go, trọng tâm chính của chúng tôi trong khảo sát này là về cách các lập trình viên bắt đầu dùng Go (hoặc các ngôn ngữ khác) cho các trường hợp sử dụng liên quan đến AI, và các thách thức cụ thể đối với những người đang học Go hoặc muốn mở rộng kỹ năng Go.

Chúng tôi tuyển dụng người tham gia từ blog Go và thông qua các lời nhắc ngẫu nhiên trong plugin VS Code Go. Năm nay, với sự trợ giúp của [JetBrains](https://jetbrains.com), chúng tôi cũng đưa vào một lời nhắc khảo sát ngẫu nhiên trong [GoLand IDE](https://jetbrains.com/go/), cho phép chúng tôi tuyển chọn được mẫu đại diện hơn của các lập trình viên Go. Chúng tôi nhận được tổng cộng 6.224 phản hồi! Xin cảm ơn rất nhiều tất cả những người đã đóng góp để điều này trở thành hiện thực.

## Điểm nổi bật {#tldr}

* Tâm lý lập trình viên vẫn cao, với 93% người trả lời bày tỏ sự hài lòng với Go trong năm qua.
* Đa số người trả lời (80%) cho biết họ tin tưởng nhóm Go sẽ "làm điều tốt nhất" cho các lập trình viên như bản thân họ khi duy trì và phát triển ngôn ngữ.
* Trong số những người trả lời khảo sát xây dựng các ứng dụng và dịch vụ được hỗ trợ bởi AI, có một nhận thức chung rằng Go là nền tảng mạnh mẽ để chạy các loại ứng dụng này trong môi trường production. Ví dụ, đa số người trả lời đang làm việc với các ứng dụng dựa trên AI đã dùng Go hoặc muốn chuyển sang Go cho các workload AI của họ, và những thách thức nghiêm trọng nhất mà lập trình viên gặp phải liên quan đến hệ sinh thái thư viện và tài liệu hơn là ngôn ngữ cốt lõi và runtime. Tuy nhiên, các con đường được ghi nhận phổ biến nhất để bắt đầu hiện nay đang tập trung vào Python, dẫn đến nhiều tổ chức bắt đầu công việc dựa trên AI trong Python trước khi chuyển sang một ngôn ngữ sẵn sàng cho môi trường production hơn.
* Các loại dịch vụ dựa trên AI phổ biến nhất mà người trả lời đang xây dựng bao gồm công cụ tóm tắt, công cụ tạo văn bản và chatbot. Các phản hồi cho thấy nhiều trường hợp sử dụng này hướng vào nội bộ, chẳng hạn như chatbot được huấn luyện trên tài liệu nội bộ của tổ chức và nhằm mục đích trả lời câu hỏi của nhân viên. Chúng tôi giả thuyết rằng các tổ chức đang cố ý bắt đầu với các trường hợp sử dụng nội bộ để phát triển chuyên môn trong nhà với LLM trong khi tránh những tình huống xấu hổ công khai khi các agent dựa trên AI hoạt động bất ngờ.
* Thiếu thời gian hoặc cơ hội là thách thức được đề cập phổ biến nhất đối với người trả lời trong việc đạt được mục tiêu học tập liên quan đến Go, cho thấy rằng việc học ngôn ngữ khó có thể được ưu tiên mà không có mục tiêu cụ thể hoặc lý do kinh doanh. Thách thức phổ biến tiếp theo là học các thực hành tốt nhất, khái niệm và idiom mới đặc thù của Go khi đến từ các hệ sinh thái ngôn ngữ khác.

## Mục lục

- <a href="#sentiment">Tâm lý lập trình viên</a>
- <a href="#devenv">Môi trường phát triển</a>
- <a href="#priorities">Ưu tiên về tài nguyên và hiệu năng</a>
- <a href="#mlai">Hiểu về các trường hợp sử dụng AI trong Go</a>
- <a href="#learn">Thách thức học tập</a>
- <a href="#demographics">Thông tin nhân khẩu học</a>
- <a href="#firmographics">Thông tin về tổ chức</a>
- <a href="#methodology">Phương pháp luận</a>
- <a href="#closing">Kết luận</a>

## Tâm lý lập trình viên {#sentiment}

Mức độ hài lòng tổng thể vẫn cao trong khảo sát với 93% người trả lời cho biết họ đã phần nào hoặc rất hài lòng với Go trong năm qua. Điều này không có gì đáng ngạc nhiên, xét đến việc đối tượng của chúng tôi là những người tự nguyện tham gia khảo sát. Nhưng ngay cả trong số những người được chọn ngẫu nhiên từ cả VS Code và GoLand, chúng tôi vẫn thấy tỷ lệ hài lòng tương đương (92%). Mặc dù các phần trăm chính xác dao động nhẹ từ khảo sát này sang khảo sát khác, chúng tôi không thấy bất kỳ sự khác biệt có ý nghĩa thống kê nào so với [2023 H2](/blog/survey2023-h2-results), khi tỷ lệ hài lòng là 90%.

<img src="survey2024h1/csat.svg" alt="Biểu đồ mức độ hài lòng của lập trình viên với Go"
class="chart" />

### Tin tưởng {#trust}

Năm nay chúng tôi giới thiệu một thước đo mới để đo lường sự tin tưởng của lập trình viên. Đây là câu hỏi thử nghiệm và cách diễn đạt có thể thay đổi theo thời gian khi chúng tôi tìm hiểu thêm về cách người trả lời hiểu nó. Vì đây là lần đầu tiên chúng tôi đặt câu hỏi này, chúng tôi không có các năm trước để cung cấp ngữ cảnh cho kết quả. Chúng tôi phát hiện rằng 80% người trả lời phần nào hoặc hoàn toàn đồng ý rằng họ tin tưởng nhóm Go sẽ làm điều tốt nhất cho những người dùng như họ. Người trả lời có 5 năm kinh nghiệm với Go trở lên có xu hướng đồng ý nhiều hơn (83%) so với những người có ít hơn 2 năm kinh nghiệm (77%). Điều này có thể phản ánh [sự thiên vị sống sót](https://en.wikipedia.org/wiki/Survivorship_bias) ở chỗ những người tin tưởng nhóm Go nhiều hơn có nhiều khả năng tiếp tục dùng Go hơn, hoặc có thể phản ánh cách sự tin tưởng được hiệu chỉnh theo thời gian.

<img src="survey2024h1/trust_go.svg" alt="Biểu đồ mức độ tin tưởng của lập trình viên với nhóm Go" class="chart" />

### Sự hài lòng với cộng đồng {#community}

Trong năm qua, gần một phần ba người trả lời (32%) cho biết họ đã tham gia vào cộng đồng lập trình viên Go trực tuyến hoặc tại các sự kiện trực tiếp. Các lập trình viên Go có kinh nghiệm hơn có nhiều khả năng tham gia vào sự kiện cộng đồng và hài lòng hơn với các sự kiện cộng đồng nói chung. Mặc dù chúng tôi không thể rút ra kết luận nhân quả từ dữ liệu này, chúng tôi đã thấy mối tương quan tích cực giữa sự hài lòng với cộng đồng và sự hài lòng tổng thể với Go. Có thể là việc tham gia vào cộng đồng Go tăng sự hài lòng thông qua tăng cường tương tác xã hội hoặc hỗ trợ kỹ thuật. Nói chung, chúng tôi cũng thấy rằng người trả lời có ít kinh nghiệm hơn có ít khả năng tham gia vào các sự kiện trong năm qua. Điều này có thể có nghĩa là họ chưa khám phá ra các sự kiện hoặc chưa tìm thấy cơ hội để tham gia.

<img src="survey2024h1/community_events.svg" alt="Biểu đồ tham gia vào các sự kiện cộng đồng" class="chart" /> <img src="survey2024h1/community_sat.svg"
alt="Biểu đồ sự hài lòng với cộng đồng" class="chart" />

### Thách thức lớn nhất {#biggestchallenge}

Trong vài năm qua, khảo sát này đã hỏi người tham gia về thách thức lớn nhất của họ khi dùng Go. Điều này luôn ở dạng ô nhập văn bản mở và đã thu hút nhiều loại phản hồi khác nhau. Trong chu kỳ này chúng tôi đã giới thiệu dạng đóng của câu hỏi, nơi chúng tôi cung cấp các câu trả lời viết tay phổ biến nhất từ các năm trước. Người trả lời được hiển thị ngẫu nhiên dạng mở hoặc dạng đóng của câu hỏi. Dạng đóng giúp chúng tôi xác nhận cách chúng tôi đã giải thích các phản hồi này trong lịch sử, đồng thời tăng số lượng lập trình viên Go chúng tôi nghe từ: năm nay người tham gia nhìn thấy dạng đóng có khả năng trả lời cao hơn 2,5 lần so với những người nhìn thấy dạng mở. Con số phản hồi cao hơn này thu hẹp sai số biên của chúng tôi và tăng độ tin cậy khi giải thích kết quả khảo sát.

Trong dạng đóng, chỉ có 8% người trả lời chọn "Khác", điều này cho thấy chúng tôi đã nắm bắt được phần lớn các thách thức phổ biến với các lựa chọn phản hồi của mình.
Thú vị là, 13% người trả lời cho biết họ không gặp bất kỳ thách thức nào khi dùng Go. Trong phiên bản văn bản mở của câu hỏi này, chỉ có 2% người trả lời đưa ra phản hồi này. Các phản hồi hàng đầu trong dạng đóng là học cách viết Go hiệu quả (15%) và sự dài dòng của xử lý lỗi (13%). Điều này khớp với những gì chúng tôi thấy trong dạng văn bản mở, nơi 11% phản hồi đề cập đến học Go, học các thực hành tốt nhất, hoặc các vấn đề với tài liệu là thách thức lớn nhất của họ, và 11% khác đề cập đến xử lý lỗi.

<img src="survey2024h1/biggest_challenge_closed.svg" alt="Biểu đồ dạng đóng về thách thức lớn nhất khi dùng Go" class="chart" /> <img
src="survey2024h1/text_biggest_challenge.svg" alt="Biểu đồ văn bản mở về thách thức lớn nhất khi dùng Go" class="chart" />

Những người trả lời nhìn thấy dạng đóng của câu hỏi cũng nhận được một câu hỏi văn bản mở tiếp theo để cho họ cơ hội nói thêm về thách thức lớn nhất của họ trong trường hợp họ muốn cung cấp câu trả lời sắc thái hơn, các thách thức bổ sung, hoặc bất cứ điều gì khác họ cảm thấy quan trọng. Phản hồi phổ biến nhất đề cập đến hệ thống kiểu của Go, và thường yêu cầu cụ thể là enum, kiểu option, hoặc kiểu sum trong Go. Thường chúng tôi không nhận được nhiều ngữ cảnh cho những yêu cầu này, nhưng chúng tôi nghi ngờ điều này là do một số đề xuất và cuộc thảo luận cộng đồng gần đây liên quan đến enum, sự gia tăng của những người đến từ các hệ sinh thái ngôn ngữ khác nơi những tính năng này phổ biến, hoặc kỳ vọng rằng những tính năng này sẽ giảm việc viết code boilerplate. Một trong những nhận xét toàn diện hơn liên quan đến hệ thống kiểu giải thích như sau:

> "Đây không phải là những thách thức lớn, mà là những tiện lợi tôi nhớ đến trong ngôn ngữ.
> Có cách giải quyết cho tất cả chúng, nhưng sẽ tốt nếu không phải nghĩ về điều đó.

> Kiểu sum/enum đóng có thể được mô phỏng nhưng rất rườm rà. Đây là tính năng rất hữu ích khi tương tác với các API chỉ có một tập hợp hạn chế các giá trị cho một phần tử/trường cụ thể trong phản hồi và giá trị ngoài tập đó là lỗi. Nó giúp xác thực và bắt các vấn đề tại điểm nhập và thường có thể được tạo trực tiếp từ các đặc tả API như JSON Schema, OpenAPI hoặc XML Schema Definitions.

> Tôi không phiền gì với sự dài dòng của kiểm tra lỗi, nhưng việc kiểm tra nil với con trỏ trở nên tẻ nhạt đặc biệt khi cần khoan vào một struct lồng nhau sâu với các trường con trỏ. Một số dạng kiểu Optional/Result hoặc khả năng theo dõi qua chuỗi con trỏ và đơn giản nhận nil về thay vì kích hoạt runtime panic sẽ được đánh giá cao."

<img src="survey2024h1/text_biggest_challenge_anything.svg" alt="Biểu đồ về bất cứ điều gì liên quan đến thách thức lớn nhất khi dùng Go" class="chart" />

## Môi trường phát triển {#devenv}
Giống như các năm trước, hầu hết người trả lời khảo sát phát triển với Go trên hệ thống Linux (61%) và macOS (58%). Mặc dù các con số không thay đổi nhiều từ năm này sang năm khác, chúng tôi đã thấy một số khác biệt thú vị trong mẫu tự chọn của mình. Các nhóm được lấy mẫu ngẫu nhiên từ JetBrains và VS Code có nhiều khả năng hơn (31% và 33%, tương ứng) để phát triển trên Windows so với nhóm tự chọn (19%). Chúng tôi không biết chính xác tại sao nhóm tự chọn lại khác biệt như vậy, nhưng chúng tôi giả thuyết rằng vì họ có thể đã gặp khảo sát khi đọc blog Go, những người trả lời này là một số lập trình viên tích cực và có kinh nghiệm nhất trong cộng đồng. Sở thích hệ điều hành của họ có thể phản ánh các ưu tiên lịch sử của nhóm phát triển cốt lõi, những người thường phát triển trên Linux và macOS. May mắn thay, chúng tôi có các mẫu ngẫu nhiên từ JetBrains và VS Code để cung cấp cái nhìn đại diện hơn về sở thích của lập trình viên.

<img src="survey2024h1/os_dev.svg" alt="Biểu đồ hệ điều hành người trả lời dùng khi phát triển phần mềm Go" class="chart" /> <img
src="survey2024h1/os_dev_src.svg" alt="Biểu đồ hệ điều hành người trả lời dùng khi phát triển phần mềm Go, phân chia theo nguồn mẫu khác nhau"
class="chart" /> <img src="survey2024h1/os_dev_exp.svg" alt="Biểu đồ hệ điều hành người trả lời dùng khi phát triển phần mềm Go, phân chia theo thời gian kinh nghiệm" class="chart" />

Là câu hỏi tiếp theo cho 17% người trả lời phát triển trên WSL, chúng tôi hỏi phiên bản nào họ đang dùng. 93% người trả lời phát triển trên WSL đang dùng phiên bản 2, vì vậy [nhóm Go tại Microsoft đã quyết định tập trung nỗ lực vào WSL2.](/issue/63503)

<img src="survey2024h1/wsl_version.svg" alt="Biểu đồ việc sử dụng các phiên bản WSL"
class="chart" />

Vì hai trong số các nhóm mẫu của chúng tôi được tuyển từ trong VS Code hoặc GoLand, họ có sự thiên vị mạnh mẽ về việc ưu tiên các trình soạn thảo đó. Để tránh làm lệch kết quả, chúng tôi chỉ hiển thị dữ liệu ở đây từ nhóm tự chọn. Tương tự như các năm trước, các trình soạn thảo code phổ biến nhất trong số người trả lời Go Developer Survey tiếp tục là [VS Code](https://code.visualstudio.com/) (43%) và [GoLand](https://www.jetbrains.com/go/) (33%). Chúng tôi không thấy bất kỳ sự khác biệt có ý nghĩa thống kê nào so với giữa năm 2023 (44% và 31%, tương ứng).

<img src="survey2024h1/editor.svg" alt="Biểu đồ trình soạn thảo code người trả lời ưu tiên dùng với Go" class="chart" />

Với sự phổ biến của Go trong phát triển đám mây và workload được đóng gói, không có gì ngạc nhiên khi các lập trình viên Go chủ yếu triển khai trên môi trường Linux (93%). Chúng tôi không thấy bất kỳ thay đổi đáng kể nào từ năm ngoái.

<img src="survey2024h1/os_deploy.svg" alt="Biểu đồ nền tảng người trả lời triển khai phần mềm Go lên" class="chart" />

Go là ngôn ngữ phổ biến cho phát triển dựa trên đám mây hiện đại, vì vậy chúng tôi thường bao gồm các câu hỏi khảo sát để giúp chúng tôi hiểu các nền tảng đám mây nào các lập trình viên Go đang dùng và mức độ hài lòng của họ với ba nền tảng phổ biến nhất: Amazon Web Services (AWS), Microsoft Azure và Google Cloud. Phần này chỉ được hiển thị cho những người trả lời cho biết họ dùng Go cho công việc chính của mình, khoảng 76% tổng số người trả lời. 98% số người nhìn thấy câu hỏi này làm việc trên phần mềm Go tích hợp với dịch vụ đám mây. Hơn một nửa người trả lời dùng AWS (52%), trong khi 27% dùng GCP cho việc phát triển và triển khai Go của họ. Đối với cả AWS và Google Cloud, chúng tôi không thấy bất kỳ sự khác biệt nào giữa các công ty nhỏ hay lớn về khả năng sử dụng một trong hai nhà cung cấp. Microsoft Azure là nhà cung cấp đám mây duy nhất có khả năng đáng kể hơn được dùng trong các tổ chức lớn (công ty có hơn 1.000 nhân viên) so với các công ty nhỏ hơn. Chúng tôi không thấy bất kỳ sự khác biệt đáng kể nào về việc sử dụng dựa trên quy mô tổ chức đối với bất kỳ nhà cung cấp đám mây nào khác.

Tỷ lệ hài lòng khi dùng Go với AWS và Google Cloud đều là 77%. Trong lịch sử, các tỷ lệ này xấp xỉ nhau. Giống như các năm trước, tỷ lệ hài lòng đối với Microsoft Azure thấp hơn (57%).

<img src="survey2024h1/cloud_platform.svg" alt="Biểu đồ nền tảng đám mây người trả lời sử dụng" class="chart" /> <img src="survey2024h1/cloud_sat_aws.svg"
alt="Biểu đồ mức độ hài lòng với Go trên AWS trong năm qua" class="chart" />
<img src="survey2024h1/cloud_sat_gcp.svg" alt="Biểu đồ mức độ hài lòng khi dùng Go trên Google Cloud trong năm qua" class="chart" /> <img
src="survey2024h1/cloud_sat_azure.svg" alt="Biểu đồ mức độ hài lòng khi dùng Go trên Microsoft Azure trong năm qua" class="chart" />

## Ưu tiên về tài nguyên và bảo mật {#priorities}

Để giúp ưu tiên công việc của nhóm Go, chúng tôi muốn hiểu những lo ngại hàng đầu về chi phí tài nguyên và bảo mật đối với các nhóm dùng Go. Khoảng một nửa người trả lời dùng Go trong công việc báo cáo có ít nhất một lo ngại về chi phí tài nguyên trong năm qua (52%). Chi phí kỹ thuật khi viết và duy trì các dịch vụ Go phổ biến hơn (28%) so với lo ngại về chi phí chạy các dịch vụ Go (10%) hoặc cả hai gần như bằng nhau (12%). Chúng tôi không thấy bất kỳ sự khác biệt đáng kể nào về lo ngại tài nguyên giữa các tổ chức nhỏ và lớn. Để giải quyết các lo ngại về chi phí tài nguyên, nhóm Go đang tiếp tục tối ưu hóa Go và nâng cao tối ưu hóa dựa trên hồ sơ thực thi (PGO).

<img src="survey2024h1/cost_concern.svg" alt="Biểu đồ về các lo ngại chi phí người trả lời có liên quan đến việc dùng Go trong năm qua" class="chart" />

Về các ưu tiên bảo mật, chúng tôi đã yêu cầu người trả lời cho chúng tôi biết tối đa ba lo ngại hàng đầu của họ. Trong số những người có lo ngại bảo mật, nhìn chung lo ngại hàng đầu là thực hành lập trình không an toàn (42%), tiếp theo là cấu hình hệ thống sai (29%). Điều chúng tôi rút ra chính là người trả lời đặc biệt quan tâm đến hệ thống công cụ giúp tìm và sửa các vấn đề bảo mật tiềm năng trong khi họ đang viết code. Điều này phù hợp với những gì chúng tôi đã tìm hiểu từ nghiên cứu trước đây về cách các lập trình viên tìm và xử lý các lỗ hổng bảo mật.

<img src="survey2024h1/security_concern.svg" alt="Biểu đồ về các lo ngại chi phí người trả lời có liên quan đến việc dùng Go trong năm qua" class="chart" />

### Hệ thống công cụ hiệu năng {#perf}

Mục tiêu của chúng tôi cho phần này là đo lường cách người trả lời nhận thức về sự dễ dàng hay khó khăn của việc chẩn đoán các vấn đề hiệu năng và xác định liệu nhiệm vụ này có khó hơn hay dễ hơn tùy thuộc vào việc sử dụng trình soạn thảo hay IDE của họ.
Cụ thể, chúng tôi muốn biết liệu có khó hơn khi chẩn đoán các vấn đề hiệu năng từ dòng lệnh hay không, và liệu chúng tôi có nên đầu tư vào việc cải thiện tích hợp của hệ thống công cụ chẩn đoán hiệu năng trong VS Code để làm cho nhiệm vụ này dễ dàng hơn không. Trong các phân tích của mình, chúng tôi hiển thị các so sánh giữa người trả lời ưu tiên VS Code hoặc GoLand để nêu bật những gì chúng tôi tìm hiểu được về trải nghiệm khi dùng VS Code so với trình soạn thảo phổ biến khác.

Chúng tôi đầu tiên đặt câu hỏi chung về các loại công cụ và kỹ thuật khác nhau mà người trả lời dùng với Go để có một số điểm so sánh. Chúng tôi phát hiện chỉ có 40% người trả lời dùng công cụ để cải thiện hiệu năng hoặc hiệu quả code. Chúng tôi không thấy bất kỳ sự khác biệt đáng kể nào dựa trên sở thích trình soạn thảo hay IDE, nghĩa là người dùng VS Code và GoLand có xác suất sử dụng công cụ để cải thiện hiệu năng hoặc hiệu quả code tương đương nhau.

<img src="survey2024h1/dev_techniques.svg" alt="Biểu đồ các kỹ thuật khác nhau dùng cho bảo mật, chất lượng và hiệu năng" class="chart" />

Đa số người trả lời (73%) cho chúng tôi biết rằng việc xác định và giải quyết các vấn đề hiệu năng ít nhất là quan trọng ở mức vừa phải. Một lần nữa, chúng tôi không thấy bất kỳ sự khác biệt đáng kể nào ở đây giữa người dùng GoLand và VS Code về mức độ quan trọng của việc chẩn đoán vấn đề hiệu năng.

<img src="survey2024h1/perf_importance.svg" alt="Biểu đồ về tầm quan trọng của việc xác định và giải quyết các vấn đề hiệu năng" class="chart" />

Nhìn chung, người trả lời không thấy việc chẩn đoán vấn đề hiệu năng dễ dàng, với 30% báo cáo nó là khá khó hoặc rất khó và 46% nói không dễ cũng không khó. Trái với giả thuyết của chúng tôi, người dùng VS Code không có nhiều khả năng hơn để báo cáo các thách thức khi chẩn đoán vấn đề hiệu năng so với người trả lời khác. Những người dùng dòng lệnh để chẩn đoán vấn đề hiệu năng, bất kể trình soạn thảo ưu tiên của họ, cũng không báo cáo nhiệm vụ này khó hơn so với những người dùng IDE. Số năm kinh nghiệm là yếu tố đáng kể duy nhất chúng tôi quan sát được, nơi các lập trình viên Go ít kinh nghiệm thấy khó hơn nhìn chung để chẩn đoán vấn đề hiệu năng so với các lập trình viên Go có kinh nghiệm hơn.

<img src="survey2024h1/perf_easiness.svg" alt="Biểu đồ mức độ dễ hay khó của việc chẩn đoán vấn đề hiệu năng" class="chart" /> <img
src="survey2024h1/perf_easiness_exp.svg" alt="Biểu đồ mức độ dễ hay khó của việc chẩn đoán vấn đề hiệu năng phân chia theo thời gian kinh nghiệm" class="chart" /> <img src="survey2024h1/perf_easiness_where.svg"
alt="Biểu đồ mức độ dễ hay khó của việc chẩn đoán vấn đề hiệu năng phân chia theo nơi sử dụng công cụ chẩn đoán hiệu năng" class="chart" />

Để trả lời câu hỏi ban đầu của chúng tôi, hầu hết lập trình viên thấy khó để chẩn đoán vấn đề hiệu năng trong Go, bất kể trình soạn thảo hay hệ thống công cụ ưu tiên của họ. Điều này đặc biệt đúng với các lập trình viên có ít hơn hai năm kinh nghiệm trong Go.

Chúng tôi cũng bao gồm một câu hỏi tiếp theo cho những người trả lời đánh giá việc chẩn đoán vấn đề hiệu năng là ít nhất quan trọng một chút để hiểu vấn đề nào quan trọng nhất với họ. Độ trễ, tổng bộ nhớ và tổng CPU là những lo ngại hàng đầu. Có thể có một số giải thích cho tầm quan trọng của các lĩnh vực này. Thứ nhất, chúng có thể đo lường được và dễ dàng chuyển đổi thành chi phí kinh doanh. Thứ hai, tổng bộ nhớ và mức sử dụng CPU đại diện cho các giới hạn vật lý đòi hỏi nâng cấp phần cứng hoặc tối ưu hóa phần mềm để cải thiện. Hơn nữa, độ trễ, tổng bộ nhớ và tổng CPU dễ quản lý hơn bởi lập trình viên và có thể ảnh hưởng ngay cả các dịch vụ đơn giản. Ngược lại, hiệu năng GC và phân bổ bộ nhớ có thể chỉ liên quan trong các trường hợp hiếm gặp hoặc đối với các workload nặng đặc biệt. Ngoài ra, độ trễ nổi bật là thước đo hiện rõ nhất với người dùng, vì độ trễ cao dẫn đến dịch vụ chậm và người dùng không hài lòng.

<img src="survey2024h1/perf_concerns.svg" alt="Biểu đồ các vấn đề hiệu năng nào là lo ngại cao nhất đối với người trả lời" class="chart" />

## Hiểu về các trường hợp sử dụng AI cho Go {#mlai}

[Khảo sát trước đây](/blog/survey2023-h2-results#mlai) của chúng tôi đã hỏi các lập trình viên Go về những trải nghiệm sớm của họ với các hệ thống AI tạo sinh. Để đi sâu hơn trong chu kỳ này, chúng tôi đặt một số câu hỏi liên quan đến AI để hiểu cách người trả lời đang xây dựng các dịch vụ dựa trên AI (cụ thể hơn là dựa trên LLM). Chúng tôi phát hiện rằng một nửa người trả lời khảo sát (50%) làm việc tại các tổ chức đang xây dựng hoặc khám phá các dịch vụ dựa trên AI. Trong số đó, hơn một nửa (56%) cho biết họ tham gia vào việc bổ sung khả năng AI cho các dịch vụ của tổ chức mình. Các câu hỏi liên quan đến AI còn lại chỉ được hiển thị cho phần người trả lời này.

Hãy thận trọng khi tổng quát hóa các phản hồi của người tham gia này sang toàn bộ dân số lập trình viên Go. Vì chỉ khoảng 1/4 người trả lời khảo sát đang làm việc với các dịch vụ dựa trên AI, chúng tôi đề nghị dùng dữ liệu này để hiểu những người dùng sớm trong không gian này, với lưu ý rằng người dùng sớm có xu hướng hơi khác so với đa số người cuối cùng sẽ chấp nhận một công nghệ. Ví dụ, chúng tôi kỳ vọng rằng đối tượng này đang thử nghiệm nhiều mô hình và SDK hơn có thể sẽ là trường hợp một hoặc hai năm nữa, và gặp nhiều thách thức hơn liên quan đến việc tích hợp các dịch vụ đó vào codebase hiện có của họ.

<img src="survey2024h1/ai_org.svg" alt="Biểu đồ người trả lời có tổ chức đang xây dựng hoặc khám phá các dịch vụ dựa trên ML/AI" class="chart" /> <img
src="survey2024h1/ai_involved.svg" alt="Biểu đồ người trả lời hiện đang tham gia vào việc phát triển AI của tổ chức" class="chart" />

Trong số đối tượng lập trình viên Go làm việc chuyên nghiệp với các hệ thống AI tạo sinh (GenAI), đa số (81%) báo cáo sử dụng ChatGPT hoặc các mô hình DALL-E của OpenAI. Một tập hợp các mô hình mã nguồn mở cũng thấy tỷ lệ chấp nhận cao, với đa số người trả lời (53%) dùng ít nhất một trong Llama, Mistral, hoặc mô hình OSS khác. Chúng tôi thấy một số bằng chứng sớm rằng các tổ chức lớn hơn (1.000+ nhân viên) ít có khả năng dùng các mô hình OpenAI (74% so với 83%) và có nhiều khả năng hơn một chút để dùng các mô hình độc quyền khác (22% so với 11%). Tuy nhiên, chúng tôi không thấy bất kỳ bằng chứng nào về sự khác biệt trong việc chấp nhận mô hình OSS dựa trên quy mô tổ chức, cả các công ty nhỏ hơn và doanh nghiệp lớn hơn đều cho thấy số đông nhỏ chấp nhận mô hình OSS (51% và 53%, tương ứng). Nhìn chung, chúng tôi thấy đa số người trả lời ưu tiên dùng các mô hình mã nguồn mở (47%) với chỉ 19% ưu tiên các mô hình độc quyền; 37% cho biết họ không có sở thích.

<img src="survey2024h1/generative_models.svg" alt="Biểu đồ các mô hình AI tạo sinh mà tổ chức người trả lời đang dùng" class="chart" /> <img
src="survey2024h1/ai_libs.svg" alt="Biểu đồ các dịch vụ và thư viện liên quan đến AI mà tổ chức người trả lời đang dùng" class="chart" />

Các loại dịch vụ phổ biến nhất mà người trả lời đang xây dựng bao gồm công cụ tóm tắt (56%), công cụ tạo văn bản (55%) và chatbot (46%).
Các phản hồi văn bản mở cho thấy nhiều trường hợp sử dụng này hướng vào nội bộ, chẳng hạn như chatbot được huấn luyện trên tài liệu nội bộ của tổ chức và nhằm trả lời câu hỏi của nhân viên. Người trả lời đã nêu ra một số lo ngại về các tính năng AI hướng ra ngoài, đặc biệt là do các vấn đề về độ tin cậy (ví dụ: liệu những thay đổi nhỏ trong câu hỏi của tôi có dẫn đến kết quả rất khác nhau không?) và độ chính xác (ví dụ: kết quả có đáng tin cậy không?). Một chủ đề thú vị chạy xuyên suốt các phản hồi này là cảm giác căng thẳng giữa rủi ro không áp dụng hệ thống công cụ AI hoàn toàn (và do đó bỏ lỡ lợi thế cạnh tranh tiềm năng nếu AI tạo sinh trở nên cần thiết trong tương lai), cân bằng với rủi ro về công khai tiêu cực hoặc vi phạm quy định/luật pháp khi dùng AI chưa được kiểm tra trong các lĩnh vực hướng đến khách hàng có mức độ quan trọng cao.

Chúng tôi tìm thấy bằng chứng rằng Go đã được dùng trong không gian GenAI, và có vẻ như có nhu cầu nhiều hơn. Khoảng 1/3 người trả lời đang xây dựng các tính năng dựa trên AI cho chúng tôi biết họ đã dùng Go cho nhiều nhiệm vụ GenAI, bao gồm prototyping các tính năng mới và tích hợp dịch vụ với LLM.
Tỷ lệ này tăng lên một chút đối với hai lĩnh vực mà chúng tôi tin rằng Go là công cụ đặc biệt phù hợp: đường ống dữ liệu cho hệ thống ML/AI (37%) và hosting API endpoint cho mô hình ML/AI (41%). Ngoài những người chấp nhận (có thể là sớm) này, chúng tôi thấy rằng khoảng 1/4 người trả lời _muốn_ dùng Go cho những loại sử dụng này, nhưng hiện đang bị chặn bởi điều gì đó. Chúng tôi sẽ quay lại những rào cản này sau, sau khi khám phá lý do tại sao người trả lời muốn dùng Go cho những nhiệm vụ này ngay từ đầu.

<img src="survey2024h1/ai_apps.svg" alt="Biểu đồ các loại ứng dụng AI tạo sinh người trả lời làm việc" class="chart" /> <img
src="survey2024h1/ai_uses_interest.svg" alt="Biểu đồ các loại ứng dụng AI mà tổ chức người trả lời hiện đang làm việc hoặc đang xem xét" class="chart" />

### Lý do dùng Go với các hệ thống AI tạo sinh

Để giúp chúng tôi hiểu những lợi ích mà lập trình viên hy vọng thu được khi dùng Go trong các dịch vụ AI/ML của họ, chúng tôi đã hỏi lập trình viên tại sao họ cảm thấy Go là lựa chọn tốt cho lĩnh vực này. Đa số rõ ràng (61%) người trả lời đề cập đến một hoặc nhiều nguyên tắc hoặc tính năng cốt lõi của Go, chẳng hạn như tính đơn giản, an toàn runtime, concurrency, hoặc triển khai binary đơn. Một phần ba người trả lời trích dẫn sự quen thuộc hiện có với Go, bao gồm mong muốn tránh giới thiệu các ngôn ngữ mới nếu có thể. Hoàn chỉnh các phản hồi phổ biến nhất là các thách thức khác nhau với Python (đặc biệt là để chạy các dịch vụ production) ở mức 14%.

> "Tôi nghĩ rằng sự bền vững, đơn giản, hiệu năng và binary gốc mà ngôn ngữ cung cấp làm cho nó trở thành lựa chọn mạnh mẽ hơn nhiều cho các workload AI."
> <span class="quote_source">--- Lập trình viên Go mã nguồn mở tại một tổ chức lớn với ít hơn 1 năm kinh nghiệm</span>

> "Chúng tôi muốn giữ stack công nghệ đồng nhất nhất có thể trên toàn tổ chức để mọi người dễ dàng phát triển trên mọi lĩnh vực hơn. Vì chúng tôi đã viết tất cả các backend bằng Go, nên chúng tôi quan tâm đến việc có thể viết các triển khai mô hình ML bằng Go và tránh phải viết lại các phần của stack cho logging, monitoring, v.v... bằng ngôn ngữ riêng biệt [như] Python." <span class="quote_source">--- Lập trình viên Go chuyên nghiệp tại một tổ chức cỡ trung với 5 đến 7 năm kinh nghiệm</span>

> "Go tốt hơn cho chúng tôi trong việc chạy các máy chủ API và các tác vụ nền trên worker pool. Việc sử dụng tài nguyên thấp hơn của Go đã cho phép chúng tôi phát triển mà không dùng thêm tài nguyên. Và chúng tôi nhận thấy rằng các dự án Go dễ duy trì hơn theo thời gian cả trong thay đổi code và khi cập nhật dependency. Chúng tôi chạy các mô hình như một dịch vụ riêng biệt viết bằng Python và tương tác với chúng trong Go."
> <span class="quote_source">--- Lập trình viên Go chuyên nghiệp tại một tổ chức lớn với 5 đến 7 năm kinh nghiệm</span>

Có vẻ như trong số các lập trình viên Go quan tâm đến ML/AI, có một nhận thức chung rằng 1) Go vốn là ngôn ngữ tốt cho lĩnh vực này (vì những lý do được trình bày ở trên), và 2) có sự do dự khi giới thiệu một ngôn ngữ mới sau khi các tổ chức đã đầu tư vào Go (điểm này hợp lý tổng quát cho bất kỳ ngôn ngữ nào). Một số người trả lời cũng bày tỏ sự thất vọng với Python vì những lý do như an toàn kiểu, chất lượng code và triển khai khó khăn.

<img src="survey2024h1/text_ml_interest.svg" alt="Biểu đồ lý do của người trả lời tại sao Go là lựa chọn tốt cho trường hợp sử dụng AI của họ"
class="chart" />

### Thách thức khi dùng Go với các hệ thống GenAI

Người trả lời nhất quán về những gì hiện ngăn họ dùng Go với các dịch vụ dựa trên AI: hệ sinh thái tập trung vào Python, các thư viện/framework yêu thích của họ đều bằng Python, tài liệu hướng dẫn bắt đầu giả định sự quen thuộc với Python, và các nhà khoa học dữ liệu hoặc nhà nghiên cứu khám phá các mô hình này đã quen với Python.

> "Python dường như có tất cả các thư viện. PyTorch chẳng hạn được dùng rộng rãi để chạy các mô hình. Nếu có các framework trong Go để chạy các mô hình này, chúng tôi sẽ thích làm vậy hơn." <span class="quote_source">--- Lập trình viên Go chuyên nghiệp tại một tổ chức lớn với 2 đến 4 năm kinh nghiệm</span>

> "Các công cụ Python trưởng thành hơn đáng kể và có thể dùng ngay từ đầu, làm cho chúng có chi phí triển khai thấp hơn đáng kể."
>  <span class="quote_source">--- Lập trình viên Go chuyên nghiệp tại một tổ chức nhỏ với 2 đến 4 năm kinh nghiệm</span>

> "[Thế giới] Go thiếu nhiều thư viện AI. Nếu tôi có mô hình LLM PyTorch, tôi thậm chí không thể phục vụ nó (hoặc tôi không biết làm thế nào). Với Python về cơ bản chỉ là vài dòng code." <span class="quote_source">--- Lập trình viên Go chuyên nghiệp tại một tổ chức nhỏ với ít hơn 1 năm kinh nghiệm</span>

Những phát hiện này củng cố tốt quan sát trên của chúng tôi rằng các lập trình viên Go tin rằng Go _nên_ là ngôn ngữ tuyệt vời để xây dựng các dịch vụ AI sẵn sàng cho môi trường production: chỉ có 3% người trả lời nói rằng có điều gì đó cụ thể với Go đang chặn con đường của họ, và chỉ có 2% trích dẫn các thách thức tương tác cụ thể với Python. Nói cách khác, hầu hết các rào cản mà lập trình viên đối mặt có thể được giải quyết trong hệ sinh thái module và tài liệu, thay vì yêu cầu thay đổi ngôn ngữ cốt lõi hoặc runtime.

<img src="survey2024h1/text_ml_blockers.svg" alt="Biểu đồ những gì đang chặn người trả lời dùng Go với các ứng dụng dựa trên AI của họ" class="chart" />

Chúng tôi cũng hỏi người tham gia khảo sát liệu họ đã làm việc với Python cho GenAI chưa, và nếu có, liệu họ có thích dùng Go không. Những người trả lời cho biết họ thích dùng Go hơn Python cũng nhận được câu hỏi tiếp theo về những gì sẽ cho phép họ dùng Go với các hệ thống GenAI.

Đa số (62%) người trả lời báo cáo đã dùng Python để tích hợp với các mô hình AI tạo sinh; trong nhóm này, 57% thích dùng Go hơn. Vì đối tượng khảo sát của chúng tôi đều là lập trình viên Go, chúng tôi nên kỳ vọng đây là giới hạn trên gần đúng về tỷ lệ lập trình viên tổng thể quan tâm đến việc chuyển từ Python sang Go cho các nhiệm vụ GenAI, trong trạng thái hiện tại của mỗi hệ sinh thái.

Trong số những người trả lời đã dùng Python nhưng muốn dùng Go, đa số áp đảo (92%) cho biết sự có sẵn của các tương đương Go cho các thư viện Python sẽ cho phép họ tích hợp Go với các hệ thống GenAI.
Tuy nhiên, chúng tôi nên thận trọng khi giải thích kết quả này; các phản hồi văn bản mở và một bộ phỏng vấn ngữ cảnh riêng biệt với các lập trình viên làm việc trên các dịch vụ GenAI mô tả hệ sinh thái tập trung vào Python xung quanh GenAI; không chỉ là Go thiếu nhiều thư viện so với hệ sinh thái Python, mà còn là mức độ đầu tư được nhận thức vào các thư viện Go thấp hơn, tài liệu và ví dụ chủ yếu bằng Python, và mạng lưới các chuyên gia làm việc trong lĩnh vực này đã quen với Python.
Thử nghiệm và xây dựng proof-of-concept trong Python gần như chắc chắn sẽ tiếp tục, và việc thiếu các biến thể Go của các thư viện Python (ví dụ, [pandas](https://pandas.pydata.org/)) chỉ là rào cản đầu tiên mà các lập trình viên sẽ gặp phải khi cố gắng chuyển từ Python sang Go. Các thư viện và SDK là cần thiết, nhưng khó có thể đủ một mình để xây dựng hệ sinh thái Go mạnh mẽ cho các ứng dụng ML/AI production.

Hơn nữa, các phỏng vấn ngữ cảnh với các lập trình viên Go xây dựng dịch vụ dựa trên AI cho thấy rằng _gọi_ API từ Go không phải là vấn đề lớn, đặc biệt với các mô hình được hosting như [GPT-4](https://openai.com/gpt-4) hoặc [Gemini](https://gemini.google.com/). Xây dựng, đánh giá và hosting các mô hình tùy chỉnh được coi là thách thức trong Go (chủ yếu do thiếu framework và thư viện hỗ trợ điều này trong Python), nhưng người tham gia phỏng vấn đã phân biệt giữa các trường hợp sử dụng sở thích (ví dụ: chơi với các mô hình tùy chỉnh ở nhà) và các trường hợp sử dụng kinh doanh. Các trường hợp sở thích bị thống trị bởi Python vì tất cả các lý do đã liệt kê ở trên, nhưng các trường hợp kinh doanh tập trung hơn vào độ tin cậy, độ chính xác và hiệu năng khi gọi các mô hình được hosting. Đây là lĩnh vực mà Go có thể tỏa sáng _mà không cần_ xây dựng hệ sinh thái lớn của các thư viện ML/AI/khoa học dữ liệu, mặc dù chúng tôi kỳ vọng lập trình viên vẫn sẽ được hưởng lợi từ tài liệu, hướng dẫn thực hành tốt nhất và ví dụ.

Bởi vì lĩnh vực GenAI còn rất mới, các thực hành tốt nhất vẫn đang được xác định và kiểm tra. Các phỏng vấn ngữ cảnh ban đầu với các lập trình viên đã cho thấy rằng một trong những mục tiêu của họ là chuẩn bị cho tương lai trong đó GenAI trở thành lợi thế cạnh tranh; bằng cách đầu tư một chút vào lĩnh vực này ngay bây giờ, họ hy vọng giảm thiểu rủi ro trong tương lai. Họ cũng vẫn đang cố gắng hiểu xem các hệ thống GenAI có thể hữu ích cho điều gì và lợi tức đầu tư (nếu có) trông như thế nào. Do những điều chưa biết này, dữ liệu ban đầu của chúng tôi cho thấy rằng các tổ chức (đặc biệt bên ngoài ngành công nghệ) có thể do dự khi đưa ra các cam kết dài hạn ở đây, và thay vào đó sẽ theo đuổi cách tiếp cận tinh gọn hoặc linh hoạt cho đến khi một trường hợp sử dụng đáng tin cậy với lợi ích rõ ràng nổi lên, hoặc các đồng nghiệp trong ngành bắt đầu đầu tư lớn và công khai vào lĩnh vực này.

<img src="survey2024h1/python_usage.svg" alt="Biểu đồ cho thấy mức độ sử dụng Python cao để tích hợp với các mô hình gen AI" class="chart" /> <img
src="survey2024h1/go_python_pref.svg" alt="Biểu đồ cho thấy sở thích dùng Go hơn Python để tích hợp với các mô hình gen AI" class="chart" /> <img
src="survey2024h1/enable_go.svg" alt="Biểu đồ những gì sẽ cho phép người trả lời dùng Go ở nơi họ hiện đang dùng Python" class="chart" /> <img
src="survey2024h1/text_ml_challenge.svg" alt="Biểu đồ thách thức lớn nhất cho người trả lời tích hợp dịch vụ backend với các mô hình gen AI" class="chart" />

## Thách thức học tập {#learn}

Để cải thiện trải nghiệm học Go, chúng tôi muốn nghe từ các lập trình viên Go không có kinh nghiệm, cũng như những người có thể đã nắm được những điều cơ bản về những gì họ coi là thách thức lớn nhất để đạt được mục tiêu học tập của mình. Chúng tôi cũng muốn nghe từ các lập trình viên có thể chủ yếu tập trung vào việc giúp người khác bắt đầu với Go hơn là mục tiêu học tập của chính họ, vì họ có thể có một số hiểu biết về những thách thức phổ biến mà họ thấy khi giới thiệu lập trình viên.

Chỉ có 3% người trả lời cho biết họ hiện đang học những kiến thức cơ bản về Go. Điều này không quá ngạc nhiên, xét đến việc hầu hết người trả lời khảo sát của chúng tôi có ít nhất một năm kinh nghiệm với Go. Trong khi đó, 40% người trả lời cho biết họ đã học những điều cơ bản nhưng muốn học các chủ đề nâng cao hơn và 40% khác cho biết họ giúp các lập trình viên khác học Go. Chỉ có 15% cho biết họ không có mục tiêu học tập nào liên quan đến Go.

<img src="survey2024h1/learning_goal.svg" alt="Biểu đồ mục tiêu học tập của người trả lời về Go" class="chart" />

Khi chúng tôi xem xét các phân đoạn thời gian chi tiết hơn về kinh nghiệm Go, chúng tôi thấy rằng 30% những người đã dùng Go dưới ba tháng cho biết họ đang học những điều cơ bản về Go, trong khi khoảng hai phần ba trong số họ cho biết họ đã học được những điều cơ bản. Đó là bằng chứng tốt rằng ai đó có thể ít nhất cảm thấy mình đã học được những điều cơ bản về Go trong một thời gian ngắn, nhưng cũng có nghĩa là chúng tôi không có nhiều phản hồi từ nhóm này khi họ đang ở đầu hành trình học tập của mình.

<img src="survey2024h1/learning_goal_go_exp.svg" alt="Biểu đồ mục tiêu học tập của người trả lời về Go phân chia theo các đơn vị thời gian nhỏ hơn" class="chart" />

Để xác định loại tài liệu học tập nào có thể cần nhất trong cộng đồng, chúng tôi đã hỏi loại nội dung học tập nào người trả lời ưu tiên cho các chủ đề liên quan đến phát triển phần mềm. Họ có thể chọn nhiều tùy chọn, do đó các con số ở đây vượt quá 100%. 87% người trả lời cho biết họ ưu tiên nội dung viết, đó là định dạng được ưu tiên nhất. 52% cho biết họ ưu tiên nội dung video, và đặc biệt định dạng này được ưu tiên hơn bởi các lập trình viên có ít kinh nghiệm hơn. Điều này có thể cho thấy nhu cầu ngày càng tăng về nội dung học tập theo định dạng video. Nhóm nhân khẩu học ít kinh nghiệm hơn không ưu tiên nội dung viết ít hơn các nhóm khác. [Việc cung cấp cả định dạng viết và video cùng nhau đã được chứng minh là cải thiện kết quả học tập](https://www.sciencedirect.com/science/article/abs/pii/S0360131514001353) và [giúp các lập trình viên có sở thích và khả năng học tập khác nhau](https://udlguidelines.cast.org/representation/perception), điều này có thể tăng khả năng tiếp cận của nội dung học tập trong cộng đồng Go.

<img src="survey2024h1/learning_content_exp.svg" alt="Biểu đồ định dạng ưu tiên của người trả lời cho nội dung học tập, phân chia theo số năm kinh nghiệm Go"
class="chart" />

Chúng tôi đã hỏi những người trả lời cho biết họ có mục tiêu học tập liên quan đến Go thách thức lớn nhất của họ là gì để đạt được mục tiêu đó. Điều này có chủ ý được giữ đủ rộng để ai đó đang bắt đầu hoặc đã nắm được những điều cơ bản đều có thể trả lời câu hỏi này. Chúng tôi cũng muốn cho người trả lời cơ hội nói với chúng tôi về nhiều loại thách thức, không chỉ các chủ đề họ thấy khó.

Thách thức phổ biến nhất được đề cập áp đảo là thiếu thời gian hoặc các hạn chế cá nhân khác như sự tập trung hoặc động lực để học (44%).
Mặc dù chúng tôi không thể cho người trả lời thêm thời gian, chúng tôi nên lưu ý khi sản xuất tài liệu học tập hoặc giới thiệu các thay đổi trong hệ sinh thái rằng người dùng có thể đang hoạt động trong các ràng buộc thời gian đáng kể. Cũng có thể có các cơ hội cho các nhà giáo dục để sản xuất các tài nguyên [có thể tiêu hóa thành các phần nhỏ hơn](https://web.cortland.edu/frieda/id/IDtheories/26.html) hoặc [theo nhịp đều đặn](https://psychology.ucsd.edu/undergraduate-program/undergraduate-resources/academic-writing-resources/effective-studying/spaced-practice.html#:~:text=This%20is%20known%20as%20spaced,information%20and%20retain%20it%20longer.) để giữ cho người học có động lực.

Ngoài thời gian, thách thức hàng đầu là học các khái niệm, idiom hoặc thực hành tốt nhất mới đặc thù của Go (11%). Đặc biệt, thích nghi với ngôn ngữ biên dịch kiểu tĩnh từ Python hoặc JavaScript và học cách tổ chức code Go có thể đặc biệt thách thức. Người trả lời cũng yêu cầu thêm ví dụ (6%), cả trong tài liệu và các ứng dụng thực tế để học từ đó. Các lập trình viên đến từ cộng đồng lập trình viên lớn hơn kỳ vọng có thể tìm thấy nhiều giải pháp và ví dụ có sẵn hơn.

> "Chuyển từ ngôn ngữ như Python sang ngôn ngữ biên dịch kiểu tĩnh là thách thức, nhưng bản thân Go thì không. Tôi thích học qua phản hồi nhanh, vì vậy REPL của Python rất tuyệt cho điều đó. Vì vậy, bây giờ tôi cần tập trung vào việc thực sự đọc tài liệu và ví dụ để có thể học. Một số tài liệu cho Go khá thưa thớt và có thể cần thêm ví dụ."
> <span class="quote_source">--- Người trả lời có ít hơn 3 năm kinh nghiệm với Go.</span>

> "Thách thức chính của tôi là thiếu các dự án ví dụ cho các ứng dụng cấp doanh nghiệp. Cách tổ chức một dự án Go lớn là điều tôi muốn có thêm ví dụ làm tham chiếu. Tôi muốn tái cấu trúc dự án hiện tại mà tôi đang làm theo kiến trúc module/sạch hơn, và tôi thấy khó trong Go do thiếu ví dụ / tham chiếu 'thư mục/gói' theo hướng ý kiến hơn." <span class="quote_source">--- Người trả lời có 1 đến 2 năm kinh nghiệm với Go.</span>

> "Đây là hệ sinh thái nhỏ hơn tôi quen, vì vậy tìm kiếm trực tuyến không cho nhiều kết quả cho các vấn đề cụ thể. Các tài nguyên có sẵn thực sự rất hữu ích và tôi thường có thể giải quyết vấn đề cuối cùng, chỉ mất thêm một chút thời gian hơn."<span class="quote_source">--- Người trả lời có ít hơn 3 tháng kinh nghiệm với Go.</span>

<img src="survey2024h1/text_learning_challenge.svg" alt="Biểu đồ thách thức lớn nhất để đạt được mục tiêu học tập của người trả lời" class="chart" />

Đối với những người trả lời có mục tiêu học tập chính là giúp người khác bắt đầu với Go, chúng tôi đã hỏi điều gì có thể giúp lập trình viên dễ dàng bắt đầu với Go hơn. Chúng tôi nhận được nhiều loại phản hồi bao gồm gợi ý tài liệu, nhận xét về các chủ đề khó (ví dụ: dùng con trỏ hoặc concurrency), cũng như yêu cầu thêm các tính năng quen thuộc từ các ngôn ngữ khác. Đối với các danh mục chiếm ít hơn 2% phản hồi, chúng tôi gộp chúng vào phản hồi "Khác". Thú vị là, không ai đề cập đến "thêm thời gian." Chúng tôi nghĩ điều này là vì thiếu thời gian hoặc động lực thường là thách thức khi không có sự cần thiết ngay lập tức để học điều gì đó mới liên quan đến Go. Đối với những người giúp người khác bắt đầu với Go, có thể có lý do kinh doanh để làm vậy, làm cho việc ưu tiên dễ dàng hơn, và do đó "thiếu thời gian" không phải là thách thức lớn.

Nhất quán với các kết quả trước đó, 16% trong số những người giúp người khác bắt đầu với Go cho chúng tôi biết rằng các lập trình viên Go mới sẽ được hưởng lợi từ việc có nhiều ví dụ thực tế hơn hoặc các bài tập dựa trên dự án để học từ đó. Họ cũng thấy cần thiết phải giúp các lập trình viên đến từ các hệ sinh thái ngôn ngữ khác thông qua so sánh giữa chúng. [Nghiên cứu trước đây cho chúng ta biết rằng kinh nghiệm với một ngôn ngữ lập trình có thể cản trở việc học một ngôn ngữ mới](https://dl.acm.org/doi/abs/10.1145/3377811.3380352), đặc biệt khi các khái niệm và hệ thống công cụ mới khác với những gì lập trình viên quen thuộc. Có các tài nguyên hiện có nhằm giải quyết vấn đề này (chỉ cần tìm kiếm "Golang for [language] developers" làm ví dụ), nhưng có thể khó để các lập trình viên Go mới tìm kiếm các khái niệm mà họ chưa có từ vựng hoặc những loại tài nguyên này có thể không giải quyết đầy đủ các nhiệm vụ cụ thể. Trong tương lai chúng tôi muốn tìm hiểu thêm về cách và khi nào nên trình bày so sánh ngôn ngữ để tạo điều kiện học các khái niệm mới.

Một nhu cầu liên quan mà nhóm này báo cáo là cần nhiều giải thích hơn đằng sau triết lý và thực hành tốt nhất của Go. Có thể là việc học không chỉ _những gì_ làm Go khác biệt mà còn _tại sao_ sẽ giúp các lập trình viên Go mới hiểu các khái niệm mới hoặc cách thực hiện các nhiệm vụ khác với kinh nghiệm trước của họ.

<img src="survey2024h1/text_onboard_others.svg" alt="Biểu đồ ý tưởng từ người trả lời giúp người khác bắt đầu với Go" class="chart" />

## Thông tin nhân khẩu học {#demographics}

Chúng tôi đặt các câu hỏi nhân khẩu học tương tự trong mỗi chu kỳ khảo sát này để chúng tôi có thể hiểu các kết quả từ năm này sang năm khác có thể so sánh như thế nào. Ví dụ, nếu đa số người trả lời báo cáo có ít hơn một năm kinh nghiệm với Go trong một chu kỳ khảo sát, có thể rất có khả năng rằng bất kỳ sự khác biệt nào khác trong kết quả từ các chu kỳ trước xuất phát từ sự thay đổi nhân khẩu học lớn này. Chúng tôi cũng dùng những câu hỏi này để so sánh giữa các nhóm, chẳng hạn như sự hài lòng theo thời gian người trả lời đã dùng Go.

Năm nay chúng tôi đã giới thiệu một số thay đổi nhỏ về cách chúng tôi hỏi về kinh nghiệm với Go để phù hợp với khảo sát lập trình viên JetBrains. Điều này cho phép chúng tôi thực hiện so sánh giữa các nhóm khảo sát và tạo điều kiện cho việc phân tích dữ liệu.

<img src="survey2024h1/go_exp.svg" alt="Biểu đồ thời gian người trả lời đã làm việc với Go" class="chart" />

Chúng tôi thấy một số khác biệt về mức độ kinh nghiệm tùy thuộc vào cách các lập trình viên khám phá khảo sát của chúng tôi. Nhóm đã phản hồi thông báo khảo sát trong VS Code có xu hướng ít kinh nghiệm với Go hơn; chúng tôi nghi ngờ điều này là phản ánh sự phổ biến của VS Code với các lập trình viên Go mới, những người có thể chưa sẵn sàng đầu tư vào giấy phép IDE trong khi họ vẫn đang học. Về số năm kinh nghiệm Go, những người trả lời được chọn ngẫu nhiên từ GoLand giống hơn với nhóm tự chọn của chúng tôi, những người tìm thấy khảo sát qua blog Go. Việc thấy sự nhất quán giữa các mẫu như thế này cho phép chúng tôi tổng quát hóa các phát hiện đến phần còn lại của cộng đồng một cách tự tin hơn.

<img src="survey2024h1/go_exp_src.svg" alt="Biểu đồ thời gian người trả lời đã làm việc với Go, phân chia theo nguồn mẫu khác nhau" class="chart" />

Ngoài số năm kinh nghiệm với Go, năm nay chúng tôi cũng đo lường số năm kinh nghiệm lập trình chuyên nghiệp. Chúng tôi ngạc nhiên khi thấy rằng 26% người trả lời có 16 năm kinh nghiệm lập trình chuyên nghiệp trở lên. Để so sánh, [đối tượng JetBrains Developer Survey](https://www.jetbrains.com/lp/devecosystem-2023/demographics/#code_yrs) từ năm 2023 có đa số người trả lời với 3 đến 5 năm kinh nghiệm chuyên nghiệp. Có nhân khẩu học có kinh nghiệm hơn có thể ảnh hưởng đến sự khác biệt trong phản hồi. Ví dụ, chúng tôi thấy sự khác biệt đáng kể về loại nội dung học tập mà người trả lời với các mức kinh nghiệm khác nhau ưa thích.

<img src="survey2024h1/dev_exp.svg" alt="Biểu đồ số năm kinh nghiệm lập trình viên chuyên nghiệp của người trả lời" class="chart" />

Khi chúng tôi xem xét các mẫu khác nhau của mình, nhóm tự chọn thậm chí còn có kinh nghiệm hơn so với các nhóm được chọn ngẫu nhiên, với 29% có 16 năm kinh nghiệm chuyên nghiệp trở lên. Điều này cho thấy rằng nhóm tự chọn của chúng tôi nhìn chung có kinh nghiệm hơn các nhóm được chọn ngẫu nhiên và có thể giúp giải thích một số sự khác biệt chúng tôi thấy trong nhóm này.

<img src="survey2024h1/dev_exp_src.svg" alt="Biểu đồ số năm kinh nghiệm lập trình viên chuyên nghiệp của người trả lời" class="chart" />

Chúng tôi đã giới thiệu một câu hỏi nhân khẩu học khác trong chu kỳ này về trạng thái việc làm để giúp chúng tôi thực hiện so sánh với [Khảo sát lập trình viên của JetBrains](https://www.jetbrains.com/lp/devecosystem-2023/demographics/#employment_status).
Chúng tôi thấy rằng 81% người trả lời được tuyển dụng toàn thời gian, nhiều hơn đáng kể so với 63% trong khảo sát JetBrains. Chúng tôi cũng thấy ít sinh viên hơn đáng kể trong dân số của mình (4%) so với 15% trong khảo sát JetBrains. Khi chúng tôi xem xét các mẫu riêng lẻ của mình, chúng tôi thấy sự khác biệt nhỏ nhưng đáng kể trong số người trả lời từ VS Code, những người có khả năng được tuyển dụng toàn thời gian thấp hơn một chút và có khả năng là sinh viên cao hơn một chút. Điều này có ý nghĩa vì VS Code miễn phí.

<img src="survey2024h1/employment.svg" alt="Biểu đồ trạng thái việc làm của người trả lời" class="chart" />

Tương tự như các năm trước, các trường hợp sử dụng phổ biến nhất cho Go là dịch vụ API/RPC (74%) và công cụ dòng lệnh (63%). Chúng tôi đã nghe nói rằng máy chủ HTTP tích hợp và các primitive concurrency của Go, sự dễ dàng của biên dịch chéo và triển khai binary đơn làm cho Go là lựa chọn tốt cho các loại ứng dụng này.

Chúng tôi cũng tìm kiếm sự khác biệt dựa trên mức độ kinh nghiệm của người trả lời với Go và quy mô tổ chức. Các lập trình viên Go có kinh nghiệm hơn báo cáo xây dựng nhiều loại ứng dụng hơn trong Go. Xu hướng này nhất quán trên mọi danh mục ứng dụng hoặc dịch vụ. Chúng tôi không tìm thấy bất kỳ sự khác biệt đáng chú ý nào về những gì người trả lời đang xây dựng dựa trên quy mô tổ chức của họ.

<img src="survey2024h1/what.svg" alt="Biểu đồ các loại thứ người trả lời đang xây dựng với Go" class="chart" />

## Thông tin về tổ chức {#firmographics}

Chúng tôi đã nghe từ người trả lời tại nhiều tổ chức khác nhau. Khoảng 27% làm việc tại các tổ chức lớn với 1.000 nhân viên trở lên, 25% đến từ các tổ chức cỡ trung với 100 đến 1.000 nhân viên, và 43% làm việc tại các tổ chức nhỏ hơn với ít hơn 100 nhân viên. Giống như các năm trước, ngành phổ biến nhất mà mọi người làm việc là công nghệ (48%) trong khi ngành phổ biến thứ hai là dịch vụ tài chính (13%).

Điều này không thay đổi về mặt thống kê so với các Go Developer Survey trước đây, chúng tôi tiếp tục nghe từ những người ở các quốc gia và trong các tổ chức có quy mô và ngành công nghiệp khác nhau với tỷ lệ nhất quán từ năm này sang năm khác.

<img src="survey2024h1/org_size.svg" alt="Biểu đồ quy mô tổ chức khác nhau nơi người trả lời dùng Go" class="chart" />

<img src="survey2024h1/industry.svg" alt="Biểu đồ các ngành công nghiệp khác nhau nơi người trả lời dùng Go" class="chart" />

<img src="survey2024h1/location.svg" alt="Biểu đồ các quốc gia hoặc khu vực nơi người trả lời đang ở" class="chart" />

## Phương pháp luận {#methodology}

Trước năm 2021, chúng tôi công bố khảo sát chủ yếu qua blog Go, nơi nó được lan truyền trên các kênh xã hội khác nhau như Twitter, Reddit hoặc Hacker News. Năm 2021 chúng tôi giới thiệu cách mới để tuyển dụng người trả lời bằng cách dùng plugin VS Code Go để ngẫu nhiên chọn người dùng được hiển thị lời nhắc hỏi họ có muốn tham gia khảo sát không. Điều này tạo ra một mẫu ngẫu nhiên mà chúng tôi dùng để so sánh với những người tự chọn từ các kênh truyền thống của chúng tôi và giúp xác định các tác động tiềm năng của [thiên vị tự chọn](https://en.wikipedia.org/wiki/Self-selection_bias). Trong chu kỳ này, các bạn của chúng tôi tại JetBrains đã hào phóng cung cấp cho chúng tôi một mẫu ngẫu nhiên bổ sung bằng cách nhắc một tập con ngẫu nhiên của người dùng GoLand tham gia khảo sát!

64% người trả lời khảo sát "tự chọn" để tham gia khảo sát, nghĩa là họ tìm thấy nó trên blog Go hoặc các kênh Go xã hội khác. Những người không theo dõi các kênh này ít có khả năng tìm hiểu về khảo sát từ chúng, và trong một số trường hợp, họ phản hồi khác với những người theo dõi chặt chẽ. Ví dụ, họ có thể là người mới đến cộng đồng Go và chưa biết đến blog Go. Khoảng 36% người trả lời được lấy mẫu ngẫu nhiên, nghĩa là họ phản hồi khảo sát sau khi nhìn thấy lời nhắc trong VS Code (25%) hoặc GoLand (11%). Trong giai đoạn từ ngày 23 tháng 1 đến ngày 13 tháng 2, có khoảng 10% cơ hội người dùng đã nhìn thấy lời nhắc này. Bằng cách kiểm tra cách các nhóm được lấy mẫu ngẫu nhiên khác với các phản hồi tự chọn, cũng như khác với nhau, chúng tôi có thể tổng quát hóa các phát hiện đến cộng đồng lập trình viên Go lớn hơn một cách tự tin hơn.

<img src="survey2024h1/source.svg" alt="Biểu đồ các nguồn người trả lời khảo sát khác nhau" class="chart" />

### Cách đọc các kết quả này

Trong báo cáo này, chúng tôi dùng các biểu đồ phản hồi khảo sát để cung cấp bằng chứng hỗ trợ cho các phát hiện của chúng tôi. Tất cả các biểu đồ này dùng định dạng tương tự. Tiêu đề là câu hỏi chính xác mà người trả lời khảo sát đã thấy. Trừ khi có ghi chú khác, câu hỏi là lựa chọn nhiều và người tham gia chỉ có thể chọn một phản hồi; phụ đề của mỗi biểu đồ sẽ cho người đọc biết nếu câu hỏi cho phép nhiều lựa chọn phản hồi hoặc là ô văn bản mở thay vì câu hỏi trắc nghiệm. Đối với các biểu đồ phản hồi văn bản mở, một thành viên nhóm Go đã đọc và phân loại thủ công tất cả các phản hồi. Nhiều câu hỏi mở thu hút nhiều loại phản hồi khác nhau; để giữ kích thước biểu đồ hợp lý, chúng tôi rút gọn chúng xuống tối đa 10 đến 12 chủ đề hàng đầu, với các chủ đề bổ sung được nhóm dưới "Khác". Nhãn phần trăm hiển thị trong biểu đồ được làm tròn đến số nguyên gần nhất (ví dụ: 1,4% và 0,8% đều sẽ được hiển thị là 1%), nhưng độ dài của mỗi thanh và thứ tự hàng dựa trên các giá trị chưa làm tròn.

Để giúp người đọc hiểu trọng lượng bằng chứng cơ bản cho mỗi phát hiện, chúng tôi đã bao gồm các thanh lỗi hiển thị [khoảng tin cậy](https://en.wikipedia.org/wiki/Confidence_interval) 95% cho các phản hồi; các thanh hẹp hơn cho thấy độ tin cậy tăng. Đôi khi hai hoặc nhiều phản hồi có thanh lỗi chồng chéo, điều đó có nghĩa là thứ tự tương đối của các phản hồi đó không có ý nghĩa thống kê (nghĩa là các phản hồi về cơ bản là ngang nhau). Phía dưới bên phải của mỗi biểu đồ hiển thị số người có phản hồi được đưa vào biểu đồ, theo dạng "n = [số người trả lời]". Trong các trường hợp chúng tôi thấy sự khác biệt thú vị trong phản hồi giữa các nhóm (ví dụ: số năm kinh nghiệm, quy mô tổ chức, hoặc nguồn mẫu), chúng tôi đã hiển thị phân tích màu sắc của sự khác biệt.

## Kết luận {#closing}

Đó là kết thúc của Go Developer Survey nửa năm của chúng tôi. Xin cảm ơn rất nhiều đến tất cả những người đã chia sẻ suy nghĩ về Go và tất cả những người đã đóng góp để thực hiện khảo sát này! Điều đó có ý nghĩa rất lớn với chúng tôi và thực sự giúp chúng tôi cải thiện Go.

Năm nay chúng tôi cũng vui mừng thông báo về việc phát hành sắp tới của tập dữ liệu khảo sát này. Chúng tôi dự kiến chia sẻ dữ liệu ẩn danh này vào cuối tháng 4, cho phép bất kỳ ai phân tích và lọc các phản hồi khảo sát theo nhu cầu để trả lời câu hỏi của riêng họ về hệ sinh thái Go.

Cập nhật ngày 03-05-2024: Thật tiếc, chúng tôi cần trì hoãn việc phát hành tập dữ liệu này. Chúng tôi vẫn đang nỗ lực để thực hiện điều này, nhưng không kỳ vọng có thể chia sẻ nó cho đến nửa sau năm 2024.

--- Alice và Todd (thay mặt nhóm Go tại Google)
