---
title: Kết Quả Khảo Sát Go Developer Survey 2024 H2
date: 2024-12-20
by:
- Alice Merrick
tags:
- survey
- community
- developer experience research
summary: Những gì chúng tôi học được từ khảo sát nhà phát triển H2 năm 2024
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

Go được thiết kế với trọng tâm là trải nghiệm nhà phát triển, và chúng tôi rất coi trọng những phản hồi nhận được qua các đề xuất, vấn đề và tương tác cộng đồng. Tuy nhiên, các kênh này thường chỉ đại diện cho tiếng nói của những người dùng có kinh nghiệm nhất hoặc tích cực nhất, một tập con nhỏ trong cộng đồng Go rộng lớn hơn. Để đảm bảo chúng tôi phục vụ các nhà phát triển ở mọi cấp độ kỹ năng, bao gồm cả những người có thể không có quan điểm mạnh mẽ về thiết kế ngôn ngữ, chúng tôi thực hiện khảo sát này một hoặc hai lần mỗi năm để thu thập phản hồi có hệ thống và bằng chứng định lượng. Cách tiếp cận toàn diện này cho phép chúng tôi lắng nghe từ nhiều nhà phát triển Go hơn, cung cấp những hiểu biết có giá trị về cách Go được sử dụng trong các ngữ cảnh và cấp độ kinh nghiệm khác nhau. Sự tham gia của bạn rất quan trọng trong việc định hướng các quyết định về thay đổi ngôn ngữ và phân bổ nguồn lực, từ đó định hình tương lai của Go. Cảm ơn tất cả những người đã đóng góp, và chúng tôi khuyến khích bạn tiếp tục tham gia vào các cuộc khảo sát trong tương lai. Trải nghiệm của bạn rất quan trọng với chúng tôi.

Bài đăng này chia sẻ kết quả của Go Developer Survey gần đây nhất, được tiến hành từ ngày 9–23 tháng 9 năm 2024. Chúng tôi tuyển dụng người tham gia từ blog Go và thông qua các lời mời ngẫu nhiên trong plugin [VS Code](https://code.visualstudio.com/) Go và [GoLand IDE](https://www.jetbrains.com/go/), cho phép chúng tôi tuyển dụng một mẫu đại diện hơn của các nhà phát triển Go. Chúng tôi nhận được tổng cộng 4.156 phản hồi. Xin cảm ơn rất nhiều tất cả những ai đã đóng góp để biến điều này thành hiện thực.

Cùng với việc ghi nhận cảm xúc và thách thức xung quanh việc sử dụng Go và hệ thống công cụ Go, các lĩnh vực trọng tâm chính của chúng tôi trong cuộc khảo sát này là khám phá các nguồn gây phiền phức, thách thức trong việc thực hiện các thực hành tốt nhất, và cách các nhà phát triển đang sử dụng hỗ trợ AI.

## Điểm nổi bật

* **Tâm lý của nhà phát triển đối với Go vẫn rất tích cực**, với 93% người trả lời khảo sát cho biết họ cảm thấy hài lòng khi làm việc với Go trong năm trước.
* **Dễ dàng triển khai và API/SDK dễ sử dụng** là những điều yêu thích nhất của người trả lời khi sử dụng Go trên ba nhà cung cấp đám mây hàng đầu. Hỗ trợ Go hạng nhất là điều quan trọng để theo kịp kỳ vọng của nhà phát triển.
* 70% người trả lời đang sử dụng trợ lý AI khi phát triển với Go. Các cách sử dụng phổ biến nhất là **hoàn thành mã dựa trên LLM**, viết bài kiểm thử, tạo mã Go từ mô tả ngôn ngữ tự nhiên và động não. Có sự chênh lệch đáng kể giữa những gì người trả lời nói họ muốn sử dụng AI năm ngoái và những gì họ hiện đang sử dụng AI.
* Thách thức lớn nhất đối với các nhóm sử dụng Go là **duy trì tiêu chuẩn mã hóa nhất quán** trong codebase của họ. Điều này thường là do các thành viên nhóm có các cấp độ kinh nghiệm Go khác nhau và đến từ các nền tảng lập trình khác nhau, dẫn đến sự không nhất quán trong phong cách mã và việc áp dụng các mẫu không thuần ngữ (non-idiomatic).

## Nội dung

- <a href="#sentiment">Mức độ hài lòng tổng thể</a>
- <a href="#devenv">Môi trường phát triển và công cụ</a>
- <a href="#cloud">Go trên đám mây</a>
- <a href="#ai-assistance">Hỗ trợ AI</a>
- <a href="#team-challenges">Thách thức cho các nhóm sử dụng Go</a>
- <a href="#simd">Single Instruction, Multiple Data (SIMD)</a>
- <a href="#demographics">Nhân khẩu học</a>
- <a href="#firmographics">Thông tin tổ chức</a>
- <a href="#methodology">Phương pháp</a>
- <a href="#how-to-read-these-results">Cách đọc các kết quả này</a>
- <a href="#closing">Kết luận</a>

### Mức độ hài lòng tổng thể {#sentiment}

Mức độ hài lòng tổng thể vẫn cao trong cuộc khảo sát, với 93% người trả lời cho biết họ khá hài lòng hoặc rất hài lòng với Go trong năm qua. Mặc dù tỷ lệ phần trăm chính xác dao động nhẹ từ chu kỳ này sang chu kỳ khác, chúng tôi không thấy bất kỳ sự khác biệt có ý nghĩa thống kê nào so với các Khảo sát [2023 H2](/blog/survey2023-h2-results) hoặc [2024 H1](/blog/survey2024-h1-results), khi tỷ lệ hài lòng lần lượt là 90% và 93%.

<img src="survey2024h2/csat.svg" alt="Biểu đồ mức độ hài lòng của nhà phát triển với Go"
class="chart" />

Các bình luận mở mà chúng tôi nhận được trong cuộc khảo sát tiếp tục làm nổi bật những gì các nhà phát triển thích nhất khi sử dụng Go, ví dụ như sự đơn giản, Go toolchain, và cam kết về khả năng tương thích ngược:

*"Tôi là người yêu thích ngôn ngữ lập trình (kiểu C) và tôi luôn quay lại Go vì sự đơn giản, biên dịch nhanh và toolchain mạnh mẽ. Tiếp tục phát triển nhé!"*

*"Cảm ơn vì đã tạo ra Go! Đó là ngôn ngữ yêu thích của tôi, vì nó khá tối giản, chu kỳ phát triển có chu kỳ build-test nhanh, và khi sử dụng một dự án mã nguồn mở ngẫu nhiên được viết bằng Go, có khả năng cao là nó sẽ hoạt động, ngay cả sau 10 năm. Tôi yêu cam kết tương thích 1.0."*

### Môi trường phát triển và công cụ {#devenv}

#### Hệ điều hành nhà phát triển {#developer-os}

Nhất quán với các năm trước, hầu hết người trả lời khảo sát phát triển với Go trên các hệ thống Linux (61%) và macOS (59%). Về mặt lịch sử, tỷ lệ người dùng Linux và macOS rất gần nhau, và chúng tôi không thấy bất kỳ thay đổi đáng kể nào so với cuộc khảo sát lần trước. Các nhóm được lấy mẫu ngẫu nhiên từ JetBrains và VS Code có nhiều khả năng hơn (lần lượt 33% và 36%) phát triển trên Windows so với nhóm tự chọn (16%).

<img src="survey2024h2/os_dev.svg" alt="Biểu đồ hệ điều hành người trả lời sử dụng khi phát triển phần mềm Go" class="chart" /> <img
src="survey2024h2/os_dev_src.svg" alt="Biểu đồ hệ điều hành người trả lời sử dụng khi phát triển phần mềm Go, phân chia theo nguồn mẫu khác nhau"
class="chart" />

#### Môi trường triển khai {#deployment-environments}

Với sự phổ biến của Go trong phát triển đám mây và khối lượng công việc được đóng gói (containerized), không có gì ngạc nhiên khi các nhà phát triển Go chủ yếu triển khai lên môi trường Linux (96%).

<img src="survey2024h2/os_deploy.svg" alt="Biểu đồ hệ điều hành người trả lời triển khai khi phát triển phần mềm Go" class="chart" />

Chúng tôi đã thêm một số câu hỏi để hiểu kiến trúc mà người trả lời đang triển khai khi triển khai lên Linux, Windows hay WebAssembly. Kiến trúc x86-64 / AMD64 là lựa chọn phổ biến nhất cho những người triển khai cả lên Linux (92%) và Windows (97%). ARM64 đứng thứ hai với 49% cho Linux và 21% cho Windows.

<img src="survey2024h2/arch_linux.svg" alt="Mức sử dụng kiến trúc Linux"
class="chart" /> <img src="survey2024h2/arch_win.svg" alt="Mức sử dụng kiến trúc Windows"
class="chart" />

Không nhiều người trả lời triển khai lên Web Assembly (chỉ khoảng 4% tổng số người trả lời), nhưng 73% trong số họ cho biết họ triển khai lên JS và 48% lên WASI Preview 1.

<img src="survey2024h2/arch_wa.svg" alt="Mức sử dụng kiến trúc Web assembly"
class="chart" />

#### Nhận thức và sở thích trình soạn thảo {#editor-awareness-and-preferences}

Chúng tôi đã giới thiệu một câu hỏi mới trong cuộc khảo sát này để đánh giá nhận thức và mức độ sử dụng các trình soạn thảo phổ biến cho Go. Khi diễn giải các kết quả này, hãy nhớ rằng 34% người trả lời đến với cuộc khảo sát từ VS Code và 9% đến từ GoLand, vì vậy họ có nhiều khả năng sử dụng những trình soạn thảo đó thường xuyên hơn.

VS Code là trình soạn thảo được sử dụng rộng rãi nhất, với 66% người trả lời sử dụng thường xuyên, và GoLand đứng thứ hai với 35%. Hầu hết tất cả người trả lời đều đã nghe về cả VS Code và GoLand, nhưng người trả lời có nhiều khả năng đã ít nhất thử VS Code hơn. Thú vị là, 33% người trả lời cho biết họ thường xuyên sử dụng 2 hoặc nhiều trình soạn thảo hơn. Họ có thể sử dụng các trình soạn thảo khác nhau cho các nhiệm vụ hoặc môi trường khác nhau, chẳng hạn như sử dụng Emacs hoặc Vim qua SSH, nơi IDE không có sẵn.

<img src="survey2024h2/editor_aware.svg" alt="Mức độ quen thuộc với từng trình soạn thảo" class="chart" />

Chúng tôi cũng đặt câu hỏi về sở thích trình soạn thảo, giống như chúng tôi đã hỏi trong các cuộc khảo sát trước. Vì các quần thể được lấy mẫu ngẫu nhiên của chúng tôi được tuyển dụng từ bên trong VS Code hoặc GoLand, họ có xu hướng mạnh mẽ ưa thích những trình soạn thảo đó. Để tránh làm lệch kết quả, chúng tôi chỉ hiển thị dữ liệu cho trình soạn thảo được ưa thích nhất ở đây từ nhóm tự chọn. 38% ưa thích VS Code và 35% ưa thích GoLand. Đây là sự khác biệt đáng chú ý so với cuộc khảo sát lần trước trong H1, khi 43% ưa thích VS Code và 33% ưa thích GoLand. Một giải thích có thể là cách người trả lời được tuyển dụng năm nay. Năm nay, thông báo VS Code bắt đầu mời các nhà phát triển tham gia khảo sát trước khi bài đăng trên blog Go được đăng, vì vậy một tỷ lệ lớn hơn của người trả lời đến từ lời nhắc VS Code năm nay, những người có thể đã đến từ bài đăng trên blog. Vì chúng tôi chỉ hiển thị những người trả lời tự chọn trong biểu đồ này, dữ liệu từ người trả lời từ lời nhắc VS Code không được đại diện ở đây. Một yếu tố đóng góp khác có thể là sự tăng nhẹ trong những người ưa thích "Khác" (4%). Các câu trả lời ghi vào cho thấy có sự quan tâm ngày càng tăng đối với các trình soạn thảo như [Zed](https://zed.dev/), chiếm 43% trong các câu trả lời ghi vào.

<img src="survey2024h2/editor.svg" alt="Các trình soạn thảo mã mà người trả lời ưa thích nhất để sử dụng với Go" class="chart" />

#### Công cụ phân tích mã {#code-analysis-tools}

Công cụ phân tích mã phổ biến nhất là `gopls`, được sử dụng có ý thức bởi 65% người trả lời. Vì `gopls` được sử dụng ngầm theo mặc định trong VS Code, đây có thể là một con số thấp hơn thực tế. Tiếp theo, `golangci-lint` được sử dụng bởi 57% người trả lời, và `staticcheck` được sử dụng bởi 34% người trả lời. Một tỷ lệ nhỏ hơn nhiều đã sử dụng các công cụ tùy chỉnh hoặc khác, điều này cho thấy hầu hết người trả lời ưa thích các công cụ phổ biến đã được thiết lập hơn là các giải pháp tùy chỉnh. Chỉ 10% người trả lời cho biết họ không sử dụng bất kỳ công cụ phân tích mã nào.

<img src="survey2024h2/code_analysis.svg" alt="Công cụ phân tích mã người trả lời sử dụng với Go" class="chart" />

#### Go trên đám mây {#cloud}

Go là ngôn ngữ phổ biến cho phát triển dựa trên đám mây hiện đại, vì vậy chúng tôi thường thêm các câu hỏi khảo sát để giúp chúng tôi hiểu những nền tảng và dịch vụ đám mây nào mà các nhà phát triển Go đang sử dụng. Trong chu kỳ này, chúng tôi muốn tìm hiểu về sở thích và trải nghiệm của các nhà phát triển Go trên các nhà cung cấp đám mây, với trọng tâm đặc biệt vào các nhà cung cấp đám mây lớn nhất: Amazon Web Services (AWS), Microsoft Azure, và Google Cloud. Chúng tôi cũng bao gồm một tùy chọn bổ sung cho "Bare Metal Servers" cho những người triển khai lên máy chủ mà không có ảo hóa.

Tương tự như các năm trước, gần một nửa số người trả lời (50%) triển khai chương trình Go lên Amazon Web Services. AWS theo sau là các máy chủ tự sở hữu hoặc do công ty sở hữu (37%), và Google Cloud (30%). Những người trả lời làm việc tại các tổ chức lớn có khả năng triển khai lên các máy chủ tự sở hữu hoặc do công ty sở hữu cao hơn một chút (48%) so với những người làm việc tại các tổ chức vừa và nhỏ (34%). Họ cũng có khả năng triển khai lên Microsoft Azure cao hơn một chút (25%) so với các tổ chức vừa và nhỏ (12%).

<img src="survey2024h2/cloud_platform.svg" alt="Các nhà cung cấp đám mây nơi người trả lời triển khai phần mềm Go" class="chart" />

Các dịch vụ đám mây được sử dụng phổ biến nhất là AWS Elastic Kubernetes Service (41%), AWS EC2 (39%), và Google Cloud GKE (29%). Mặc dù chúng tôi đã thấy mức sử dụng Kubernetes tăng lên theo thời gian, đây là lần đầu tiên chúng tôi thấy EKS trở nên phổ biến hơn EC2. Nhìn chung, các dịch vụ Kubernetes là những dịch vụ phổ biến nhất cho AWS, Google Cloud và Azure, tiếp theo là VM và sau đó là Serverless. Thế mạnh của Go trong containerization và phát triển microservices tự nhiên phù hợp với sự phổ biến ngày càng tăng của Kubernetes, vì nó cung cấp một nền tảng hiệu quả và có thể mở rộng để triển khai và quản lý các loại ứng dụng này.

<img src="survey2024h2/cloud_service.svg" alt="Các nền tảng đám mây nơi người trả lời triển khai phần mềm Go" class="chart" />

Chúng tôi đã hỏi một câu hỏi tiếp theo với những người trả lời đã triển khai mã Go lên ba nhà cung cấp đám mây hàng đầu, Amazon Web Services, Google Cloud và Microsoft Azure về những gì họ thích nhất khi triển khai mã Go lên mỗi đám mây. Câu trả lời phổ biến nhất trên các nhà cung cấp khác nhau thực sự là hiệu suất và tính năng ngôn ngữ của Go hơn là điều gì đó về nhà cung cấp đám mây.

Các lý do phổ biến khác là:

* Sự quen thuộc với nhà cung cấp đám mây nhất định so với các đám mây khác
* Dễ dàng triển khai ứng dụng Go trên nhà cung cấp đám mây nhất định
* API/SDK của nhà cung cấp đám mây cho Go dễ sử dụng
* API/SDK được ghi chép đầy đủ

Ngoài sự quen thuộc, những điều yêu thích hàng đầu làm nổi bật tầm quan trọng của việc có hỗ trợ hạng nhất cho Go để theo kịp kỳ vọng của nhà phát triển.

Cũng khá phổ biến khi người trả lời cho biết họ không có điều yêu thích về nhà cung cấp đám mây của họ. Từ phiên bản trước của cuộc khảo sát có các câu trả lời ghi vào, điều này thường có nghĩa là họ không tương tác trực tiếp với Đám mây. Đặc biệt, những người trả lời sử dụng Microsoft Azure có nhiều khả năng nói rằng "Không có gì" là điều yêu thích của họ (51%) so với AWS (27%) hoặc Google Cloud (30%).

<img src="survey2024h2/cloud_fave_all.svg" alt= "Những gì người trả lời thích nhất về mỗi trong số 3 đám mây hàng đầu" class="chart" />

### Hỗ trợ AI {#ai-assistance}

Nhóm Go giả thuyết rằng hỗ trợ AI có tiềm năng giảm bớt gánh nặng cho các nhà phát triển khỏi các nhiệm vụ tẻ nhạt và lặp đi lặp lại, cho phép họ tập trung vào các khía cạnh sáng tạo và thỏa mãn hơn trong công việc của họ. Để có cái nhìn sâu hơn về các lĩnh vực mà hỗ trợ AI có thể mang lại lợi ích nhất, chúng tôi đã thêm một phần trong khảo sát để xác định những phiền phức phổ biến của nhà phát triển.

Phần lớn người trả lời (70%) đang sử dụng trợ lý AI khi phát triển với Go. Cách sử dụng phổ biến nhất của trợ lý AI là hoàn thành mã dựa trên LLM (35%). Các câu trả lời phổ biến khác là viết bài kiểm thử (29%), tạo mã Go từ mô tả ngôn ngữ tự nhiên (27%) và động não ý tưởng (25%). Cũng có một thiểu số đáng kể (30%) người trả lời không đã sử dụng bất kỳ LLM nào để hỗ trợ trong tháng qua.

<img src="survey2024h2/ai_assist_tasks.svg" alt= "Các nhiệm vụ phổ biến nhất được sử dụng với hỗ trợ AI" class="chart" />

Một số kết quả này nổi bật khi so sánh với các phát hiện từ cuộc khảo sát 2023 H2, nơi chúng tôi hỏi người trả lời về 5 trường hợp sử dụng hàng đầu mà họ muốn thấy AI/ML hỗ trợ các nhà phát triển Go. Mặc dù một số câu trả lời mới được giới thiệu trong cuộc khảo sát hiện tại, chúng tôi vẫn có thể so sánh đại khái giữa những gì người trả lời nói họ muốn AI hỗ trợ và mức sử dụng thực tế của họ. Trong cuộc khảo sát trước đó, viết bài kiểm thử là trường hợp sử dụng mong muốn nhất (49%). Trong cuộc khảo sát 2024 H2 mới nhất của chúng tôi, khoảng 29% người trả lời đã sử dụng trợ lý AI cho việc này trong tháng qua. Điều này gợi ý rằng các giải pháp hiện tại không đáp ứng nhu cầu của nhà phát triển trong việc viết bài kiểm thử. Tương tự, vào năm 2023, 47% người trả lời nói họ muốn có gợi ý về các thực hành tốt nhất trong khi lập trình, trong khi chỉ 14% một năm sau đó cho biết họ đang sử dụng hỗ trợ AI cho trường hợp sử dụng này. 46% nói họ muốn được giúp đỡ phát hiện các lỗi phổ biến trong khi lập trình, và chỉ 13% cho biết họ đang sử dụng hỗ trợ AI cho điều này. Điều này có thể chỉ ra rằng các trợ lý AI hiện tại không được trang bị tốt cho các loại nhiệm vụ này, hoặc chúng không được tích hợp tốt vào quy trình làm việc hoặc hệ thống công cụ của nhà phát triển.

Cũng đáng ngạc nhiên khi thấy mức sử dụng AI cao như vậy để tạo mã Go từ ngôn ngữ tự nhiên và động não, vì cuộc khảo sát trước đó không chỉ ra đây là những trường hợp sử dụng được mong muốn nhiều. Có thể có một số giải thích cho những khác biệt này. Trong khi những người trả lời trước đó có thể không muốn một cách rõ ràng AI để tạo mã hoặc động não ban đầu, họ có thể đang hướng tới những cách sử dụng này vì chúng phù hợp với thế mạnh hiện tại của AI tổng quát, xử lý ngôn ngữ tự nhiên và tạo văn bản sáng tạo. Chúng ta cũng nên nhớ rằng người ta [không nhất thiết là những người dự đoán tốt nhất về hành vi của chính họ](https://www.nngroup.com/articles/first-rule-of-usability-dont-listen-to-users/).

<img src="survey2024h2/ai_assist_tasks_yoy.svg" alt= "Các nhiệm vụ sử dụng hỗ trợ AI trong năm 2024 so với những gì mong muốn trong năm 2023" class="chart" />

Chúng tôi cũng thấy một số khác biệt đáng chú ý trong cách các nhóm khác nhau trả lời câu hỏi này. Những người trả lời ở các tổ chức vừa và nhỏ có khả năng sử dụng LLM cao hơn một chút (75%) so với những người ở các tổ chức lớn (66%). Có thể có một số lý do tại sao, ví dụ, các tổ chức lớn hơn có thể có các yêu cầu bảo mật và tuân thủ nghiêm ngặt hơn và lo ngại về bảo mật của các trợ lý lập trình LLM, khả năng rò rỉ dữ liệu, hoặc tuân thủ các quy định cụ thể của ngành. Họ cũng có thể đã đầu tư vào các công cụ nhà phát triển và thực hành khác đã cung cấp lợi ích tương tự cho năng suất nhà phát triển.

<img src="survey2024h2/ai_assist_tasks_org.svg" alt= "Các nhiệm vụ phổ biến nhất được sử dụng với hỗ trợ AI theo quy mô tổ chức" class="chart" />

Các nhà phát triển Go có ít hơn 2 năm kinh nghiệm có nhiều khả năng sử dụng trợ lý AI (75%) so với các nhà phát triển Go có 5+ năm kinh nghiệm (67%). Các nhà phát triển Go ít kinh nghiệm hơn cũng có nhiều khả năng sử dụng chúng cho nhiều nhiệm vụ hơn, trung bình là 3,50. Mặc dù tất cả các cấp độ kinh nghiệm đều có xu hướng sử dụng hoàn thành mã dựa trên LLM, các nhà phát triển Go ít kinh nghiệm hơn có nhiều khả năng sử dụng Go cho các nhiệm vụ liên quan đến học tập và gỡ lỗi, chẳng hạn như giải thích những gì một đoạn mã Go làm, giải quyết lỗi biên dịch và gỡ lỗi các thất bại trong mã Go của họ. Điều này gợi ý rằng các trợ lý AI hiện đang cung cấp tiện ích lớn nhất cho những người ít quen thuộc với Go hơn. Chúng tôi không biết các trợ lý AI ảnh hưởng đến việc học hoặc bắt đầu dự án Go mới như thế nào, điều này chúng tôi muốn điều tra trong tương lai. Tuy nhiên, tất cả các cấp độ kinh nghiệm đều có tỷ lệ hài lòng tương tự với trợ lý AI của họ, khoảng 73%, vì vậy các nhà phát triển Go mới không hài lòng hơn với trợ lý AI, dù sử dụng chúng thường xuyên hơn.

<img src="survey2024h2/ai_assist_tasks_exp.svg" alt= "Các nhiệm vụ phổ biến nhất được sử dụng với hỗ trợ AI theo kinh nghiệm với Go" class="chart" />

Với những người trả lời đã báo cáo sử dụng hỗ trợ AI cho ít nhất một nhiệm vụ liên quan đến viết mã Go, chúng tôi đã hỏi một số câu hỏi tiếp theo để tìm hiểu thêm về cách họ sử dụng trợ lý AI. Các trợ lý AI được sử dụng phổ biến nhất là ChatGPT (68%) và GitHub Copilot (50%). Khi được hỏi trợ lý AI nào họ sử dụng *nhiều nhất* trong tháng qua, ChatGPT và Copilot xấp xỉ nhau ở mức 36% mỗi loại, vì vậy mặc dù nhiều người trả lời hơn đã sử dụng ChatGPT, nó không nhất thiết là trợ lý chính của họ. Người tham gia hài lòng tương tự với cả hai công cụ (73% hài lòng với ChatGPT, so với 78% với GitHub CoPilot). Tỷ lệ hài lòng cao nhất cho bất kỳ trợ lý AI nào là Anthropic Claude, ở mức 87%.

<img src="survey2024h2/ai_assistants_withother.svg" alt= "Các trợ lý AI phổ biến nhất được sử dụng" class="chart" /> <img src="survey2024h2/ai_primary.svg" alt=
"Các trợ lý AI chính phổ biến nhất được sử dụng" class="chart" />

### Thách thức cho các nhóm sử dụng Go {#team-challenges}

Trong phần này của cuộc khảo sát, chúng tôi muốn hiểu những thực hành tốt nhất hoặc công cụ nào nên được tích hợp tốt hơn vào quy trình làm việc của nhà phát triển. Cách tiếp cận của chúng tôi là xác định các vấn đề phổ biến cho các nhóm sử dụng Go. Sau đó chúng tôi hỏi người trả lời thách thức nào sẽ mang lại lợi ích nhất cho họ nếu chúng được giải quyết "một cách kỳ diệu". (Điều này là để người trả lời không tập trung vào các giải pháp cụ thể.) Các vấn đề phổ biến mà sẽ mang lại lợi ích nhất nếu được giải quyết sẽ được coi là ứng cử viên để cải thiện.

Các thách thức được báo cáo phổ biến nhất cho các nhóm là duy trì tiêu chuẩn mã hóa nhất quán trong codebase Go của chúng tôi (58%), xác định các vấn đề hiệu suất trong chương trình Go đang chạy (58%) và xác định các sự kém hiệu quả trong sử dụng tài nguyên trong chương trình Go đang chạy (57%).

<img src="survey2024h2/dev_challenges.svg" alt= "Các thách thức phổ biến nhất cho các nhóm" class="chart" />

21% người trả lời nói rằng nhóm của họ sẽ được hưởng lợi nhiều nhất từ việc duy trì tiêu chuẩn mã hóa nhất quán trong codebase Go của họ. Đây là câu trả lời phổ biến nhất, khiến nó trở thành ứng cử viên tốt để giải quyết. Trong một câu hỏi tiếp theo, chúng tôi đã biết thêm chi tiết về lý do tại sao điều này đặc biệt khó khăn.

<img src="survey2024h2/dev_challenges_most_benefit.svg" alt= "Lợi ích nhất khi được giải quyết" class="chart" />

Theo các câu trả lời ghi vào, nhiều nhóm gặp thách thức trong việc duy trì tiêu chuẩn mã hóa nhất quán vì các thành viên của họ có các cấp độ kinh nghiệm khác nhau với Go và đến từ các nền tảng lập trình khác nhau. Điều này dẫn đến sự không nhất quán trong phong cách mã và việc áp dụng các mẫu không thuần ngữ (non-idiomatic).

*"Có rất nhiều kỹ sư đa ngôn ngữ ở nơi tôi làm việc. Vì vậy mã Go được viết ra không nhất quán. Tôi tự coi mình là một Gopher và dành thời gian cố gắng thuyết phục đồng nghiệp của mình những gì là thuần ngữ trong Go" — Nhà phát triển Go với 2–4 năm kinh nghiệm.*

*"Hầu hết các thành viên trong nhóm đang học Go từ đầu. Đến từ các ngôn ngữ kiểu động, họ mất một thời gian để quen với ngôn ngữ mới. Họ có vẻ gặp khó khăn trong việc duy trì tính nhất quán của mã theo hướng dẫn Go." — Nhà phát triển Go với 2–4 năm kinh nghiệm.*

Điều này phản ánh một số phản hồi chúng tôi đã nghe trước đây về các đồng đội viết "Gava" hoặc "Guby" do kinh nghiệm ngôn ngữ trước đó của họ. Mặc dù phân tích tĩnh là một loại công cụ mà chúng tôi đã nghĩ đến để giải quyết vấn đề này khi chúng tôi đưa ra câu hỏi này, chúng tôi hiện đang khám phá các cách khác nhau để giải quyết vấn đề này.

### Single Instruction, Multiple Data (SIMD) {#simd}

SIMD, hay Single Instruction, Multiple Data, là một loại xử lý song song cho phép một lệnh CPU đơn lẻ hoạt động trên nhiều điểm dữ liệu đồng thời. Điều này tạo điều kiện cho các nhiệm vụ liên quan đến các tập dữ liệu lớn và các hoạt động lặp đi lặp lại, và thường được sử dụng để tối ưu hóa hiệu suất trong các lĩnh vực như phát triển trò chơi, xử lý dữ liệu và tính toán khoa học. Trong phần này của cuộc khảo sát, chúng tôi muốn đánh giá nhu cầu của người trả lời về hỗ trợ SIMD gốc trong Go.

Phần lớn người trả lời (89%) cho biết họ làm việc trong các dự án mà các tối ưu hóa hiệu suất là quan trọng ít nhất là một phần của thời gian. 40% cho biết họ làm việc trong các dự án như vậy ít nhất nửa thời gian. Điều này đúng ở các quy mô tổ chức và cấp độ kinh nghiệm khác nhau, cho thấy rằng hiệu suất là một vấn đề quan trọng đối với hầu hết các nhà phát triển.

<img src="survey2024h2/perf_freq.svg" alt= "Tần suất người trả lời làm việc trên phần mềm quan trọng về hiệu suất" class="chart" />

Khoảng một nửa số người trả lời (54%) cho biết họ ít nhất hơi quen với khái niệm SIMD. Làm việc với SIMD thường đòi hỏi sự hiểu biết sâu hơn về kiến trúc máy tính và các khái niệm lập trình cấp thấp, vì vậy không ngạc nhiên khi chúng tôi thấy rằng các nhà phát triển ít kinh nghiệm hơn có ít khả năng quen với SIMD hơn. Những người trả lời có nhiều kinh nghiệm hơn và làm việc trên các ứng dụng quan trọng về hiệu suất ít nhất nửa thời gian có nhiều khả năng quen với SIMD nhất.

<img src="survey2024h2/simd_fam.svg" alt= "Sự quen thuộc với SIMD" class="chart"
/>

Đối với những người ít nhất hơi quen với SIMD, chúng tôi đã hỏi một số câu hỏi tiếp theo để hiểu người trả lời bị ảnh hưởng như thế nào bởi sự vắng mặt của hỗ trợ SIMD gốc trong Go. Hơn một phần ba, khoảng 37%, cho biết họ đã bị ảnh hưởng. 17% người trả lời cho biết họ đã bị giới hạn về hiệu suất mà họ có thể đạt được trong các dự án của mình, 15% cho biết họ phải sử dụng ngôn ngữ khác thay vì Go để đạt được mục tiêu của họ, và 13% cho biết họ phải sử dụng các thư viện không phải Go khi họ muốn sử dụng các thư viện Go. Thú vị là, những người trả lời bị ảnh hưởng tiêu cực bởi sự vắng mặt của hỗ trợ SIMD gốc có khả năng sử dụng Go cho xử lý dữ liệu và AI/ML cao hơn một chút. Điều này gợi ý rằng thêm hỗ trợ SIMD có thể khiến Go trở thành lựa chọn tốt hơn cho các lĩnh vực này.

<img src="survey2024h2/simd_impact.svg" alt= "Tác động của việc thiếu hỗ trợ Go gốc cho SIMD" class="chart" /> <img src="survey2024h2/what_simd_impact.svg" alt=
"Những gì những người trả lời bị ảnh hưởng xây dựng với Go" class="chart" />


### Nhân khẩu học {#demographics}

Chúng tôi đặt các câu hỏi nhân khẩu học tương tự trong mỗi chu kỳ của cuộc khảo sát này để chúng tôi có thể hiểu mức độ so sánh có thể có của kết quả hàng năm. Ví dụ, nếu chúng tôi thấy những thay đổi trong người đã trả lời cuộc khảo sát về kinh nghiệm Go, thì rất có thể những khác biệt khác trong kết quả từ các chu kỳ trước là do sự thay đổi nhân khẩu học này. Chúng tôi cũng sử dụng các câu hỏi này để cung cấp so sánh giữa các nhóm, chẳng hạn như mức độ hài lòng theo thời gian sử dụng Go của người trả lời.

Chúng tôi không thấy bất kỳ thay đổi đáng kể nào về cấp độ kinh nghiệm trong số người trả lời trong chu kỳ này.

<img src="survey2024h2/go_exp.svg" alt= "Cấp độ kinh nghiệm của người trả lời"
class="chart" />

Có sự khác biệt về nhân khẩu học của người trả lời tùy thuộc vào việc họ đến từ Blog Go, extension VS Code, hay GoLand. Quần thể trả lời thông báo khảo sát trong VS Code có xu hướng ít kinh nghiệm hơn với Go; chúng tôi nghi ngờ đây là phản ánh sự phổ biến của VS Code với các nhà phát triển Go mới, những người có thể chưa sẵn sàng đầu tư vào giấy phép IDE trong khi họ vẫn đang học. Về số năm kinh nghiệm Go, những người trả lời được chọn ngẫu nhiên từ GoLand tương tự hơn với quần thể tự chọn của chúng tôi, những người tìm thấy cuộc khảo sát thông qua Blog Go. Việc thấy sự nhất quán giữa các mẫu cho phép chúng tôi khái quát hóa các phát hiện cho phần còn lại của cộng đồng một cách tự tin hơn.

<img src="survey2024h2/go_exp_src.svg" alt= "Kinh nghiệm với Go theo nguồn khảo sát" class="chart" />

Ngoài số năm kinh nghiệm với Go, chúng tôi cũng đo số năm kinh nghiệm lập trình chuyên nghiệp. Đối tượng của chúng tôi có xu hướng là một nhóm khá có kinh nghiệm, với 26% người trả lời có 16 năm hoặc hơn kinh nghiệm lập trình chuyên nghiệp.

<img src="survey2024h2/dev_exp.svg" alt= "Cấp độ kinh nghiệm nhà phát triển chuyên nghiệp tổng thể" class="chart" />

Nhóm tự chọn thậm chí có nhiều kinh nghiệm hơn các nhóm được chọn ngẫu nhiên, với 29% có 16 năm hoặc hơn kinh nghiệm chuyên nghiệp. Điều này gợi ý rằng nhóm tự chọn của chúng tôi thường có nhiều kinh nghiệm hơn các nhóm được chọn ngẫu nhiên và có thể giúp giải thích một số khác biệt mà chúng tôi thấy trong nhóm này.

<img src="survey2024h2/dev_exp_src.svg" alt= "Cấp độ kinh nghiệm nhà phát triển chuyên nghiệp theo nguồn khảo sát" class="chart" />

Chúng tôi thấy rằng 81% người trả lời được tuyển dụng toàn thời gian. Khi chúng tôi xem xét từng mẫu riêng lẻ, chúng tôi thấy một khác biệt nhỏ nhưng đáng kể trong số người trả lời từ VS Code, những người có khả năng là sinh viên cao hơn một chút. Điều này có nghĩa là VS Code là miễn phí.

<img src="survey2024h2/employment.svg" alt= "Tình trạng việc làm" class="chart" />
<img src="survey2024h2/employment_src.svg" alt= "Tình trạng việc làm theo nguồn khảo sát" class="chart" />

Tương tự như các năm trước, các trường hợp sử dụng phổ biến nhất cho Go là các dịch vụ API/RPC (75%) và công cụ dòng lệnh (62%). Các nhà phát triển Go có kinh nghiệm hơn báo cáo xây dựng nhiều loại ứng dụng hơn trong Go. Xu hướng này nhất quán trên mọi danh mục ứng dụng hoặc dịch vụ. Chúng tôi không tìm thấy bất kỳ khác biệt đáng chú ý nào về những gì người trả lời đang xây dựng dựa trên quy mô tổ chức của họ. Người trả lời từ các mẫu VS Code và GoLand ngẫu nhiên cũng không hiển thị sự khác biệt đáng kể.

<img src="survey2024h2/what.svg" alt= "Những gì người trả lời xây dựng với Go"
class="chart" />

### Thông tin tổ chức {#firmographics}

Chúng tôi đã lắng nghe từ người trả lời ở nhiều tổ chức khác nhau. Khoảng 29% làm việc tại các tổ chức lớn với 1.001 hoặc nhiều hơn nhân viên, 25% đến từ các tổ chức vừa với 101–1.000 nhân viên, và 43% làm việc tại các tổ chức nhỏ hơn với ít hơn 100 nhân viên. Như các năm trước, ngành phổ biến nhất mà mọi người làm việc là công nghệ (43%) trong khi phổ biến thứ hai là dịch vụ tài chính (13%).

<img src="survey2024h2/org_size.svg" alt= "Quy mô tổ chức nơi người trả lời làm việc" class="chart" /> <img src="survey2024h2/industry.svg" alt= "Các ngành người trả lời làm việc" class="chart" />

Như trong các cuộc khảo sát trước, vị trí phổ biến nhất cho người trả lời khảo sát là Hoa Kỳ (19%). Năm nay chúng tôi thấy một sự thay đổi đáng kể trong tỷ lệ người trả lời đến từ Ukraine, từ 1% lên 6%, khiến nó trở thành vị trí phổ biến thứ ba cho người trả lời khảo sát. Vì chúng tôi chỉ thấy sự khác biệt này trong số người trả lời tự chọn, chứ không phải trong các nhóm được lấy mẫu ngẫu nhiên, điều này gợi ý rằng có điều gì đó đã ảnh hưởng đến người đã khám phá cuộc khảo sát, thay vì sự tăng trưởng rộng rãi trong việc áp dụng Go ở tất cả các nhà phát triển ở Ukraine. Có lẽ có sự tăng cường khả năng hiển thị hoặc nhận thức về cuộc khảo sát hoặc Blog Go trong số các nhà phát triển ở Ukraine.

<img src="survey2024h2/location.svg" alt= "Vị trí của người trả lời" class="chart" />


## Phương pháp {#methodology}

Chúng tôi thông báo cuộc khảo sát chủ yếu thông qua Blog Go, nơi nó thường được chia sẻ trên nhiều kênh xã hội như Reddit, hoặc Hacker News. Chúng tôi cũng tuyển dụng người trả lời bằng cách sử dụng plugin VS Code Go để chọn ngẫu nhiên người dùng và hiển thị lời nhắc hỏi liệu họ có muốn tham gia vào cuộc khảo sát không. Với một chút giúp đỡ từ các người bạn của chúng tôi tại JetBrains, chúng tôi cũng có một mẫu ngẫu nhiên bổ sung từ việc nhắc một tập hợp con ngẫu nhiên của người dùng GoLand tham gia khảo sát. Điều này cho chúng tôi hai nguồn mà chúng tôi sử dụng để so sánh những người trả lời tự chọn từ các kênh truyền thống của chúng tôi và giúp xác định các tác động tiềm năng của [thiên kiến tự chọn](https://en.wikipedia.org/wiki/Self-selection_bias).

57% người trả lời khảo sát "tự chọn" để tham gia cuộc khảo sát, có nghĩa là họ tìm thấy nó trên blog Go hoặc các kênh Go xã hội khác. Những người không theo dõi các kênh này ít có khả năng tìm hiểu về cuộc khảo sát từ chúng, và trong một số trường hợp, họ trả lời khác với những người theo dõi chặt chẽ chúng. Ví dụ, họ có thể là người mới trong cộng đồng Go và chưa biết về blog Go. Khoảng 43% người trả lời được lấy mẫu ngẫu nhiên, có nghĩa là họ trả lời cuộc khảo sát sau khi thấy lời nhắc trong VS Code (25%) hoặc GoLand (11%). Trong khoảng thời gian từ ngày 9–23 tháng 9 năm 2024, có khoảng 10% cơ hội người dùng plugin VS Code sẽ thấy lời nhắc này. Lời nhắc trong GoLand cũng hoạt động tương tự trong khoảng ngày 9–20 tháng 9. Bằng cách xem xét cách các nhóm được lấy mẫu ngẫu nhiên khác với các câu trả lời tự chọn, cũng như với nhau, chúng tôi có thể khái quát hóa các phát hiện cho cộng đồng lớn hơn của các nhà phát triển Go một cách tự tin hơn.

<img src="survey2024h2/source.svg" alt="Biểu đồ các nguồn khác nhau của người trả lời khảo sát" class="chart" />

#### Cách đọc các kết quả này {#how-to-read-these-results}

Trong suốt báo cáo này, chúng tôi sử dụng biểu đồ câu trả lời khảo sát để cung cấp bằng chứng hỗ trợ cho các phát hiện của chúng tôi. Tất cả các biểu đồ này sử dụng định dạng tương tự. Tiêu đề là câu hỏi chính xác mà người trả lời khảo sát đã thấy. Trừ khi có ghi chú khác, các câu hỏi là nhiều lựa chọn và người tham gia chỉ có thể chọn một câu trả lời duy nhất; phụ đề của mỗi biểu đồ sẽ cho người đọc biết nếu câu hỏi cho phép nhiều câu trả lời hoặc là hộp văn bản mở thay vì câu hỏi nhiều lựa chọn. Đối với biểu đồ của câu trả lời văn bản mở, một thành viên nhóm Go đã đọc và phân loại thủ công tất cả các câu trả lời. Nhiều câu hỏi mở đã gợi ra nhiều câu trả lời khác nhau; để giữ kích thước biểu đồ hợp lý, chúng tôi đã nén chúng thành tối đa 10-12 chủ đề hàng đầu, với các chủ đề bổ sung tất cả được nhóm dưới "Khác". Nhãn phần trăm được hiển thị trong biểu đồ được làm tròn đến số nguyên gần nhất (ví dụ, 1,4% và 0,8% sẽ cả hai đều được hiển thị là 1%), nhưng độ dài của mỗi thanh và thứ tự hàng dựa trên các giá trị chưa làm tròn.

Để giúp người đọc hiểu trọng lượng bằng chứng cơ bản mỗi phát hiện, chúng tôi đã thêm các thanh lỗi hiển thị [khoảng tin cậy](https://en.wikipedia.org/wiki/Confidence_interval) 95% cho các câu trả lời; thanh hẹp hơn cho thấy độ tin cậy tăng. Đôi khi hai hoặc nhiều câu trả lời có các thanh lỗi chồng chéo, điều đó có nghĩa là thứ tự tương đối của các câu trả lời đó không có ý nghĩa thống kê (tức là các câu trả lời thực sự bằng nhau). Góc dưới bên phải của mỗi biểu đồ hiển thị số người có câu trả lời được bao gồm trong biểu đồ, ở dạng "n = [số người trả lời]". Trong các trường hợp chúng tôi tìm thấy sự khác biệt thú vị trong câu trả lời giữa các nhóm, (ví dụ: số năm kinh nghiệm, quy mô tổ chức, hoặc nguồn mẫu) chúng tôi đã hiển thị bảng phân tích được mã hóa màu của các khác biệt.

### Kết luận {#closing}

Cảm ơn bạn đã xem lại Go Developer Survey nửa năm của chúng tôi! Và xin cảm ơn rất nhiều tất cả những ai đã chia sẻ suy nghĩ của họ về Go và tất cả những người đã đóng góp để thực hiện cuộc khảo sát này. Điều đó có ý nghĩa rất lớn với chúng tôi và thực sự giúp chúng tôi cải thiện Go.

--- Alice (thay mặt nhóm Go tại Google)
