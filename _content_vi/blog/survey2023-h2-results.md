---
title: Kết quả Khảo sát Nhà phát triển Go 2023 H2
date: 2023-12-05
by:
- Todd Kulesza
tags:
- survey
- community
- developer experience research
summary: Những gì chúng tôi học được từ khảo sát nhà phát triển 2023 H2
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

Vào tháng 8 năm 2023, nhóm Go tại Google đã thực hiện cuộc khảo sát định kỳ sáu tháng một lần đối với các nhà phát triển Go. Chúng tôi đã chiêu mộ người tham gia qua một bài đăng công khai trên blog Go và một lời nhắc ngẫu nhiên trong VS Code, thu được 4.005 câu trả lời. Chúng tôi tập trung câu hỏi khảo sát xung quanh một số chủ đề: cảm nhận chung và phản hồi về việc phát triển với Go, các ngăn xếp công nghệ được sử dụng cùng Go, cách nhà phát triển bắt đầu các dự án Go mới, kinh nghiệm gần đây với các thông báo lỗi toolchain, và hiểu sự quan tâm của nhà phát triển đối với ML/AI.

Cảm ơn tất cả những người đã tham gia cuộc khảo sát này! Báo cáo này chia sẻ những gì chúng tôi học được từ phản hồi của bạn.

## tl;dr

1. Các nhà phát triển Go cho biết họ **quan tâm hơn đến hệ thống công cụ AI/ML giúp cải thiện chất lượng, độ tin cậy, và hiệu suất của code họ viết**, thay vì viết code cho họ. Một "người đánh giá" chuyên gia luôn thức, không bao giờ bận rộn có thể là một trong những hình thức hỗ trợ nhà phát triển AI hữu ích hơn.
1. Những yêu cầu hàng đầu để cải thiện cảnh báo và lỗi toolchain là **làm cho thông báo dễ hiểu và có thể hành động hơn**; cảm nhận này được chia sẻ bởi các nhà phát triển ở tất cả các mức kinh nghiệm, nhưng đặc biệt mạnh mẽ trong số các nhà phát triển Go mới hơn.
1. Thử nghiệm của chúng tôi với các mẫu dự án (`gonew`) dường như giải quyết các vấn đề quan trọng cho các nhà phát triển Go (đặc biệt là các nhà phát triển mới với Go) và làm như vậy theo cách phù hợp với quy trình làm việc hiện tại của họ để bắt đầu một dự án mới. Dựa trên những phát hiện này, chúng tôi tin rằng **`gonew` có thể giảm đáng kể rào cản onboarding cho các nhà phát triển Go mới và dễ dàng áp dụng Go trong các tổ chức**.
1. Ba trong bốn người tham gia làm việc trên phần mềm Go cũng sử dụng dịch vụ đám mây; đây là bằng chứng rằng **các nhà phát triển coi Go là ngôn ngữ cho phát triển dựa trên đám mây hiện đại**.
1. **Cảm nhận của nhà phát triển đối với Go vẫn cực kỳ tích cực**, với 90% người tham gia khảo sát cho biết họ cảm thấy hài lòng khi làm việc với Go trong năm trước.

## Mục lục

- <a href="#sentiment">Cảm nhận của nhà phát triển</a>
- <a href="#devenv">Môi trường phát triển</a>
- <a href="#stacks">Ngăn xếp công nghệ</a>
- <a href="#gonew">Cách nhà phát triển bắt đầu dự án Go mới</a>
- <a href="#err_handling">Mục tiêu của nhà phát triển cho xử lý lỗi</a>
- <a href="#mlai">Hiểu các trường hợp sử dụng ML/AI</a>
- <a href="#err_msgs">Thông báo lỗi toolchain</a>
- <a href="#microservices">Microservices</a>
- <a href="#modules">Tác giả và bảo trì module</a>
- <a href="#demographics">Nhân khẩu học</a>
- <a href="#firmographics">Thống kê tổ chức</a>
- <a href="#methodology">Phương pháp luận</a>
- <a href="#closing">Kết luận</a>

## Cảm nhận của nhà phát triển {#sentiment}

Các nhà phát triển Go tiếp tục báo cáo mức độ hài lòng cao với hệ sinh thái Go. Đa số lớn người tham gia cho biết họ cảm thấy hài lòng khi làm việc với Go trong năm qua (90% hài lòng, 6% không hài lòng), và đa số (52%) còn đi xa hơn và cho biết họ "rất hài lòng", mức đánh giá cao nhất. Độc giả lâu dài có thể đã nhận thấy rằng con số này không thay đổi nhiều từ năm này sang năm khác. Điều này được kỳ vọng đối với một dự án lớn, ổn định như Go; chúng tôi xem chỉ số này là [chỉ số trễ](https://en.wikipedia.org/wiki/Economic_indicator#Lagging_indicators) có thể giúp xác nhận các vấn đề lan rộng trong hệ sinh thái Go, nhưng không phải là nơi chúng tôi kỳ vọng lần đầu tiên biết về các vấn đề tiềm ẩn.

Chúng tôi thường thấy rằng người làm việc với Go càng lâu thì càng có nhiều khả năng báo cáo hài lòng với Go. Xu hướng này tiếp tục vào năm 2023; trong số những người tham gia có dưới một năm kinh nghiệm Go, 82% báo cáo hài lòng với trải nghiệm phát triển Go, so với 94% nhà phát triển Go có từ năm năm kinh nghiệm trở lên. Có thể có sự kết hợp của nhiều yếu tố góp phần vào điều này, chẳng hạn như một số người tham gia phát triển sự trân trọng đối với các lựa chọn thiết kế của Go theo thời gian, hoặc quyết định rằng Go không phù hợp với công việc của họ và do đó không quay lại cuộc khảo sát này trong những năm tiếp theo (tức là, [survivorship bias](https://en.wikipedia.org/wiki/Survivorship_bias)). Tuy nhiên, dữ liệu này giúp chúng tôi lượng hóa trải nghiệm bắt đầu hiện tại cho các nhà phát triển Go, và có vẻ rõ ràng rằng chúng tôi có thể làm nhiều hơn để giúp các Gopher mới nổi tìm chỗ đứng và tận hưởng những thành công ban đầu khi phát triển với Go.

Điểm quan trọng cần rút ra là đa số lớn những người chọn làm việc với Go trong năm qua đều hài lòng với trải nghiệm của họ. Hơn nữa, số lượng người làm việc với Go tiếp tục tăng; chúng tôi thấy bằng chứng về điều này từ nghiên cứu bên ngoài như [Khảo sát Nhà phát triển của Stack Overflow](https://survey.stackoverflow.co/2023/#most-popular-technologies-language-prof) (đã tìm thấy 14% nhà phát triển chuyên nghiệp làm việc với Go trong năm qua, tăng khoảng 15% so với cùng kỳ năm trước), cũng như phân tích cho [go.dev](/) (cho thấy mức tăng 8% lượng khách truy cập so với cùng kỳ năm trước). Kết hợp sự tăng trưởng này với điểm hài lòng cao là bằng chứng rằng Go tiếp tục thu hút các nhà phát triển, và cho thấy rằng nhiều nhà phát triển chọn học ngôn ngữ này cảm thấy tốt về quyết định của họ lâu dài sau đó. Theo lời của họ:

> "Sau hơn 30 năm phát triển bằng C, C++, Java, và bây giờ là bảy năm lập trình bằng Go, đây vẫn là ngôn ngữ hiệu quả nhất. Nó không hoàn hảo (không có ngôn ngữ nào là hoàn hảo), nhưng nó có sự cân bằng tốt nhất giữa năng suất, độ phức tạp, và hiệu suất." <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 5 -- 9 năm kinh nghiệm</span>

> "Đây hiện là ngôn ngữ tốt nhất tôi biết, và tôi đã thử nhiều ngôn ngữ. Hệ thống công cụ thật tuyệt vời, thời gian biên dịch tốt, và tôi thực sự có thể làm việc hiệu quả. Tôi vui mừng khi có Go như một công cụ, và tôi không cần sử dụng TypeScript phía server. Cảm ơn." <span class="quote_source">--- Nhà phát triển Go mã nguồn mở có 3 -- 4 năm kinh nghiệm</span>

<img src="survey2023h2/csat.svg" alt="Biểu đồ mức độ hài lòng của nhà phát triển với Go" class="chart" />

## Môi trường phát triển {#devenv}

Như trong các năm trước, đa số người tham gia khảo sát cho chúng tôi biết họ làm việc với Go trên hệ thống Linux (63%) và macOS (58%). Các biến động nhỏ trong những con số này từ năm này sang năm khác rất có thể phụ thuộc vào người tìm và phản hồi cuộc khảo sát này (đặc biệt là trên blog Go), vì chúng tôi không thấy xu hướng nhất quán theo năm trong mẫu ngẫu nhiên từ VS Code.

Chúng tôi tiếp tục thấy rằng các thành viên mới hơn của cộng đồng Go có nhiều khả năng làm việc với Windows hơn so với các nhà phát triển Go có kinh nghiệm hơn. Chúng tôi diễn giải đây là tín hiệu rằng việc phát triển trên Windows quan trọng cho việc onboarding các nhà phát triển mới vào hệ sinh thái Go, và là một chủ đề nhóm chúng tôi hy vọng tập trung nhiều hơn vào năm 2024.

<img src="survey2023h2/os_dev.svg" alt="Biểu đồ hệ điều hành người tham gia sử dụng khi phát triển phần mềm Go" class="chart" /> <img
src="survey2023h2/os_dev_exp.svg" alt="Biểu đồ hệ điều hành người tham gia sử dụng khi phát triển phần mềm Go, phân chia theo thời gian kinh nghiệm"
class="chart" />

Người tham gia tiếp tục tập trung nhiều vào việc triển khai Linux. Với sự phổ biến của Go cho phát triển đám mây và khối lượng công việc containerized, điều này không có gì đáng ngạc nhiên nhưng vẫn là sự xác nhận quan trọng. Chúng tôi tìm thấy ít sự khác biệt có ý nghĩa dựa trên các yếu tố như quy mô tổ chức hoặc mức độ kinh nghiệm; thực tế là, trong khi các nhà phát triển Go mới có vẻ có nhiều khả năng *phát triển* trên Windows, 92% vẫn *triển khai* lên hệ thống Linux. Có lẽ phát hiện thú vị nhất từ sự phân tích này là các nhà phát triển Go có kinh nghiệm hơn cho biết họ triển khai cho nhiều loại hệ thống hơn (đặc biệt là WebAssembly và IoT), mặc dù không rõ đó là vì các triển khai như vậy thách thức hơn với các nhà phát triển Go mới hay là kết quả của các nhà phát triển Go có kinh nghiệm sử dụng Go trong nhiều ngữ cảnh hơn. Chúng tôi cũng quan sát thấy rằng cả IoT và WebAssembly đã tăng đều đặn trong những năm gần đây, mỗi loại tăng từ 3% vào năm 2021 lên 6% và 5% vào năm 2023.

<img src="survey2023h2/os_deploy.svg" alt="Biểu đồ các nền tảng người tham gia triển khai phần mềm Go" class="chart" />

Bối cảnh kiến trúc máy tính đã thay đổi trong vài năm qua, và chúng tôi thấy điều đó phản ánh trong các kiến trúc hiện tại mà các nhà phát triển Go cho biết họ làm việc cùng. Trong khi các hệ thống tương thích x86 vẫn chiếm đa số việc phát triển (89%), ARM64 hiện cũng được sử dụng bởi đa số người tham gia (56%). Việc áp dụng này có vẻ được thúc đẩy một phần bởi Apple Silicon; các nhà phát triển macOS hiện có nhiều khả năng cho biết họ phát triển cho ARM64 hơn so với kiến trúc x86 (76% so với 71%). Tuy nhiên, phần cứng Apple không phải là yếu tố duy nhất thúc đẩy việc áp dụng ARM64: trong số những người tham gia không phát triển trên macOS chút nào, 29% vẫn cho biết họ phát triển cho ARM64.

<img src="survey2023h2/arch.svg" alt="Biểu đồ các kiến trúc người tham gia sử dụng với Go" class="chart" />

Các trình soạn thảo code phổ biến nhất trong số người tham gia Khảo sát Nhà phát triển Go tiếp tục là [VS Code](https://code.visualstudio.com/) (44%) và [GoLand](https://www.jetbrains.com/go/) (31%). Cả hai tỷ lệ này đều giảm nhẹ so với 2023 H1 (46% và 33%), nhưng vẫn nằm trong biên độ sai số của cuộc khảo sát này. Trong danh mục "Khác", [Helix](https://helix-editor.com/) chiếm đa số câu trả lời. Tương tự như kết quả về hệ điều hành ở trên, chúng tôi không tin đây đại diện cho một sự thay đổi có ý nghĩa trong việc sử dụng trình soạn thảo code, mà là cho thấy một số biến động chúng tôi kỳ vọng thấy trong một cuộc khảo sát cộng đồng như thế này.

Chúng tôi cũng xem xét mức độ hài lòng của người tham gia với Go dựa trên trình soạn thảo họ thích sử dụng. Sau khi kiểm soát độ dài kinh nghiệm, chúng tôi không tìm thấy sự khác biệt: chúng tôi không tin rằng người ta thích làm việc với Go nhiều hơn hoặc ít hơn dựa trên trình soạn thảo code họ sử dụng. Điều đó không nhất thiết có nghĩa là tất cả các trình soạn thảo Go đều bằng nhau, nhưng có thể phản ánh rằng mọi người tìm thấy trình soạn thảo phù hợp nhất với nhu cầu của riêng họ. Điều này cho thấy hệ sinh thái Go có sự đa dạng lành mạnh của các trình soạn thảo khác nhau phù hợp với các trường hợp sử dụng và sở thích nhà phát triển khác nhau.

<img src="survey2023h2/editor_self_select.svg" alt="Biểu đồ các trình soạn thảo code người tham gia thích sử dụng với Go" class="chart" />

## Ngăn xếp công nghệ {#stacks}

Để hiểu rõ hơn về mạng lưới phần mềm và dịch vụ mà các nhà phát triển Go tương tác, chúng tôi đã đặt một số câu hỏi về ngăn xếp công nghệ. Chúng tôi đang chia sẻ những kết quả này với cộng đồng để cho thấy những công cụ và nền tảng nào đang được sử dụng phổ biến ngày nay, nhưng chúng tôi tin rằng mọi người nên xem xét nhu cầu và trường hợp sử dụng của riêng họ khi chọn ngăn xếp công nghệ. Nói rõ hơn: chúng tôi không có ý định để người đọc sử dụng dữ liệu này để chọn các thành phần của ngăn xếp công nghệ vì chúng phổ biến, cũng không tránh các thành phần vì chúng không được sử dụng phổ biến.

Đầu tiên, chúng tôi có thể tự tin nói rằng Go là ngôn ngữ phát triển dựa trên đám mây hiện đại. Thực tế, 75% người tham gia làm việc trên phần mềm Go tích hợp với dịch vụ đám mây. Đối với gần một nửa người tham gia, điều này bao gồm AWS (48%), và gần một phần ba sử dụng GCP (29%) cho phát triển và triển khai Go của họ. Đối với cả AWS và GCP, việc sử dụng được cân bằng đều giữa các doanh nghiệp lớn và tổ chức nhỏ hơn. Microsoft Azure là nhà cung cấp đám mây duy nhất có nhiều khả năng được sử dụng trong các tổ chức lớn (công ty có > 1.000 nhân viên) hơn so với các cửa hàng nhỏ hơn; các nhà cung cấp khác không cho thấy sự khác biệt có ý nghĩa trong việc sử dụng dựa trên quy mô của tổ chức.

<img src="survey2023h2/cloud.svg" alt="Biểu đồ các nền tảng đám mây người tham gia sử dụng với Go" class="chart" />

Cơ sở dữ liệu là các thành phần cực kỳ phổ biến của hệ thống phần mềm, và chúng tôi thấy rằng 91% người tham gia cho biết dịch vụ Go của họ sử dụng ít nhất một cơ sở dữ liệu. Thường xuyên nhất là PostgreSQL (59%), nhưng với số lượng người tham gia ở mức hai con số báo cáo việc sử dụng sáu cơ sở dữ liệu bổ sung, có thể nói an toàn rằng không chỉ có một vài DB tiêu chuẩn cho các nhà phát triển Go. Chúng tôi lại thấy sự khác biệt dựa trên quy mô tổ chức, với người tham gia từ các tổ chức nhỏ hơn có nhiều khả năng báo cáo sử dụng PostgreSQL và Redis, trong khi các nhà phát triển từ tổ chức lớn có phần nhiều hơn khả năng sử dụng cơ sở dữ liệu dành riêng cho nhà cung cấp đám mây của họ.

<img src="survey2023h2/db.svg" alt="Biểu đồ cơ sở dữ liệu người tham gia sử dụng với Go" class="chart" />

Một thành phần phổ biến khác mà người tham gia báo cáo sử dụng là cache hoặc kho lưu trữ khóa-giá trị; 68% người tham gia cho biết họ làm việc trên phần mềm Go tích hợp ít nhất một loại. Redis rõ ràng là phổ biến nhất (57%), theo sau ở khoảng cách xa là etcd (10%) và memcached (7%).

<img src="survey2023h2/cache.svg" alt="Biểu đồ cache người tham gia sử dụng với Go" class="chart" />

Tương tự như cơ sở dữ liệu, người tham gia khảo sát cho chúng tôi biết họ sử dụng nhiều hệ thống quan sát khác nhau. Prometheus và Grafana được đề cập phổ biến nhất (cả hai đều ở mức 43%), nhưng Open Telemetry, Datadog, và Sentry đều ở mức hai con số.

<img src="survey2023h2/metrics.svg" alt="Biểu đồ hệ thống metric người tham gia sử dụng với Go" class="chart" />

Nếu ai đó hỏi "Chúng ta đã JSON hóa mọi thứ chưa?"... có, chúng ta đã làm. Gần như mọi người tham gia (96%!) cho biết phần mềm Go của họ sử dụng định dạng dữ liệu JSON; đó gần như là toàn bộ khi nói đến dữ liệu tự báo cáo. YAML, CSV, và protocol buffer cũng được sử dụng bởi khoảng một nửa người tham gia, và tỷ lệ hai con số làm việc với TOML và XML.

<img src="survey2023h2/data.svg" alt="Biểu đồ định dạng dữ liệu người tham gia sử dụng với Go" class="chart" />

Đối với các dịch vụ xác thực và ủy quyền, chúng tôi thấy hầu hết người tham gia đang xây dựng trên nền tảng do các tiêu chuẩn như [JWT](https://jwt.io/introduction) và [OAuth2](https://oauth.net/2/) cung cấp. Đây cũng có vẻ là một lĩnh vực mà giải pháp của nhà cung cấp đám mây của tổ chức có khả năng được sử dụng tương đương như hầu hết các giải pháp thay thế sẵn dùng.

<img src="survey2023h2/auth.svg" alt="Biểu đồ hệ thống xác thực người tham gia sử dụng với Go" class="chart" />

Cuối cùng, chúng tôi có một số dịch vụ khác không thuộc về các danh mục trên một cách gọn gàng. Chúng tôi thấy rằng gần một nửa người tham gia làm việc với gRPC trong phần mềm Go của họ (47%). Đối với nhu cầu infrastructure-as-code, Terraform là công cụ được khoảng ¼ người tham gia lựa chọn. Các công nghệ khá phổ biến khác được sử dụng cùng Go bao gồm Apache Kafka, ElasticSearch, GraphQL, và RabbitMQ.

<img src="survey2023h2/other_tech.svg" alt="Biểu đồ các hệ thống xác thực người tham gia sử dụng với Go" class="chart" />

Chúng tôi cũng xem xét các công nghệ nào có xu hướng được sử dụng cùng nhau. Mặc dù không có gì rõ ràng tương tự như [LAMP stack](https://en.wikipedia.org/wiki/LAMP_(software_bundle)) nổi lên từ phân tích này, chúng tôi đã xác định một số mô hình thú vị:

- Tất cả hoặc không có gì: Mỗi danh mục (ngoại trừ định dạng dữ liệu) cho thấy mối tương quan mạnh mẽ, trong đó nếu người tham gia trả lời "Không có" cho một danh mục, họ có nhiều khả năng trả lời "Không có" cho tất cả các danh mục khác. Chúng tôi diễn giải điều này là bằng chứng rằng thiểu số các trường hợp sử dụng không yêu cầu bất kỳ thành phần ngăn xếp công nghệ nào trong số này, nhưng một khi trường hợp sử dụng yêu cầu bất kỳ thành phần nào, nó có nhiều khả năng yêu cầu (hoặc ít nhất được đơn giản hóa bởi) nhiều hơn chỉ một.
- Xu hướng thiên về công nghệ đa nền tảng: Các giải pháp dành riêng cho nhà cung cấp (tức là, dịch vụ duy nhất đối với một nền tảng đám mây) không được áp dụng phổ biến. Tuy nhiên, nếu người tham gia sử dụng một giải pháp dành riêng cho nhà cung cấp (ví dụ: cho metrics), họ có nhiều khả năng cũng nói rằng họ sử dụng các giải pháp dành riêng cho đám mây trong các lĩnh vực khác (ví dụ: cơ sở dữ liệu, xác thực, caching, v.v.).
- Đa đám mây: Ba nền tảng đám mây lớn nhất có nhiều khả năng tham gia vào các cài đặt đa đám mây. Ví dụ, nếu một tổ chức đang sử dụng bất kỳ nhà cung cấp đám mây nào không phải AWS, họ có thể cũng đang sử dụng AWS. Mô hình này rõ ràng nhất đối với Amazon Web Services, nhưng cũng rõ ràng (ở mức độ thấp hơn) đối với Google Cloud Platform và Microsoft Azure.

## Cách nhà phát triển bắt đầu dự án Go mới {#gonew}

Là một phần của [thử nghiệm với các mẫu dự án](/blog/gonew), chúng tôi muốn hiểu cách các nhà phát triển Go bắt đầu với các dự án mới ngày hôm nay. Người tham gia cho chúng tôi biết những thách thức lớn nhất của họ là chọn cách thích hợp để cấu trúc dự án (54%) và học cách viết Go idiomatic (47%). Như hai người tham gia đã diễn đạt:

> "Tìm kiếm một cấu trúc phù hợp và mức độ trừu tượng đúng đắn cho một dự án mới có thể khá tẻ nhạt; nhìn vào các dự án cộng đồng và doanh nghiệp có hồ sơ cao để tìm cảm hứng có thể khá bối rối vì mọi người cấu trúc dự án của họ theo cách khác nhau" <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 5 -- 9 năm kinh nghiệm Go</span>

> "Sẽ thật tuyệt nếu [Go có] toolchain để tạo cấu trúc cơ bản của [dự án] cho web hoặc CLI như \`go init \<project name\>\`" <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 3 -- 4 năm kinh nghiệm</span>

Các nhà phát triển Go mới hơn còn có nhiều khả năng gặp phải những thách thức này: tỷ lệ tăng lên 59% và 53% đối với những người tham gia có dưới hai năm kinh nghiệm với Go. Đây là cả hai lĩnh vực chúng tôi hy vọng cải thiện thông qua nguyên mẫu `gonew`: các mẫu có thể cung cấp cho các nhà phát triển Go mới các cấu trúc dự án và mô hình thiết kế được kiểm tra kỹ lưỡng, với các triển khai ban đầu được viết theo Go idiomatic. Những kết quả khảo sát này đã giúp nhóm chúng tôi giữ mục đích của `gonew` tập trung vào các nhiệm vụ mà cộng đồng Go đang gặp khó khăn nhất.

<img src="survey2023h2/new_challenge.svg" alt="Biểu đồ về những thách thức người tham gia gặp phải khi bắt đầu dự án Go mới" class="chart" />

Đa số người tham gia cho chúng tôi biết họ sử dụng các mẫu hoặc copy+paste code từ các dự án hiện có khi bắt đầu một dự án Go mới (58%). Trong số những người tham gia có dưới năm năm kinh nghiệm Go, tỷ lệ này tăng lên gần ⅔ (63%). Đây là sự xác nhận quan trọng rằng cách tiếp cận dựa trên mẫu trong `gonew` dường như đáp ứng các nhà phát triển ở nơi họ đã ở, căn chỉnh cách tiếp cận phổ biến không chính thức với hệ thống công cụ kiểu lệnh `go`. Điều này được hỗ trợ thêm bởi các yêu cầu tính năng phổ biến cho các mẫu dự án: đa số người tham gia yêu cầu 1) cấu trúc thư mục được cấu hình sẵn để tổ chức dự án và 2) code mẫu cho các nhiệm vụ phổ biến trong miền dự án. Những kết quả này phù hợp tốt với những thách thức mà nhà phát triển cho biết họ gặp phải trong phần trước. Các câu trả lời cho câu hỏi này cũng giúp tách biệt sự khác nhau giữa cấu trúc dự án và mô hình thiết kế, với gần gấp đôi số người tham gia nói rằng họ muốn các mẫu dự án Go cung cấp cái trước hơn cái sau.

<img src="survey2023h2/new_approach.svg" alt="Biểu đồ về các cách tiếp cận người tham gia sử dụng khi bắt đầu dự án Go mới" class="chart" />

<img src="survey2023h2/templates.svg" alt="Biểu đồ về chức năng người tham gia yêu cầu khi bắt đầu dự án Go mới" class="chart" />

Đa số người tham gia cho chúng tôi biết khả năng thực hiện các thay đổi cho một mẫu *và* có những thay đổi đó lan truyền đến các dự án dựa trên mẫu đó có ít nhất tầm quan trọng vừa phải. Giai thoại, chúng tôi chưa nói chuyện với bất kỳ nhà phát triển nào *hiện tại* có chức năng này với các cách tiếp cận mẫu tự tạo, nhưng nó cho thấy đây là một lĩnh vực thú vị cho sự phát triển trong tương lai.

<img src="survey2023h2/template_updates.svg" alt="Biểu đồ về sự quan tâm của người tham gia đến các mẫu có thể cập nhật" class="chart" />

## Mục tiêu của nhà phát triển cho xử lý lỗi {#err_handling}

Một chủ đề thường xuyên được thảo luận trong số các nhà phát triển Go là các cải tiến tiềm năng cho xử lý lỗi. Như một người tham gia tóm tắt:

> "Xử lý lỗi thêm quá nhiều boilerplate (Tôi biết, bạn có thể đã nghe điều này trước đây)" <span class="quote_source">--- Nhà phát triển Go mã nguồn mở có 1 -- 2 năm kinh nghiệm</span>

Nhưng, chúng tôi cũng nghe từ nhiều nhà phát triển rằng họ đánh giá cao cách tiếp cận của Go đối với xử lý lỗi:

> "Xử lý lỗi Go thật đơn giản và hiệu quả. Khi tôi có các backend bằng Java và C# và đang khám phá Rust và Zig, tôi luôn vui mừng khi quay lại viết code Go. Và một trong những lý do là, tin hay không, xử lý lỗi. Nó thực sự đơn giản, rõ ràng và hiệu quả. Xin hãy giữ nguyên như vậy." <span class="quote_source">--- Nhà phát triển Go mã nguồn mở có 5 -- 9 năm kinh nghiệm</span>

Thay vì hỏi về các sửa đổi cụ thể cho xử lý lỗi trong Go, chúng tôi muốn hiểu rõ hơn các mục tiêu cấp cao hơn của nhà phát triển và liệu cách tiếp cận hiện tại của Go có được chứng minh là hữu ích và có thể sử dụng được không. Chúng tôi thấy rằng đa số người tham gia đánh giá cao cách tiếp cận của Go đối với xử lý lỗi (55%) và cho biết nó giúp họ biết khi nào cần kiểm tra lỗi (50%). Cả hai kết quả này đều mạnh hơn đối với những người tham gia có nhiều kinh nghiệm Go hơn, cho thấy rằng các nhà phát triển ngày càng trân trọng cách tiếp cận của Go đối với xử lý lỗi theo thời gian, hoặc đây là một yếu tố khiến các nhà phát triển cuối cùng rời bỏ hệ sinh thái Go (hoặc ít nhất là ngừng trả lời các cuộc khảo sát liên quan đến Go). Nhiều người tham gia khảo sát cũng cảm thấy rằng Go yêu cầu nhiều code boilerplate tẻ nhạt để kiểm tra lỗi (43%); điều này vẫn đúng bất kể người tham gia có bao nhiêu kinh nghiệm Go trước đó. Thú vị là, khi người tham gia nói rằng họ đánh giá cao cách xử lý lỗi của Go, họ ít có khả năng nói rằng nó cũng dẫn đến nhiều code boilerplate, nhóm chúng tôi có giả thuyết rằng các nhà phát triển Go vừa đánh giá cao cách tiếp cận của ngôn ngữ đối với xử lý lỗi vừa cảm thấy nó quá dài dòng, nhưng chỉ 14% người tham gia đồng ý với cả hai phát biểu.

Các vấn đề cụ thể mà người tham gia đề cập bao gồm những thách thức trong việc biết loại lỗi nào cần kiểm tra (28%), muốn dễ dàng hiển thị stack trace cùng với thông báo lỗi (28%), và sự dễ dàng mà lỗi có thể bị bỏ qua hoàn toàn (19%). Khoảng ⅓ người tham gia cũng quan tâm đến việc áp dụng các khái niệm từ các ngôn ngữ khác, chẳng hạn như toán tử `?` của Rust (31%).

Nhóm Go không có kế hoạch thêm exceptions vào ngôn ngữ, nhưng vì đây theo giai thoại là một yêu cầu phổ biến, chúng tôi đã đưa nó vào như một lựa chọn phản hồi. Chỉ 1 trong 10 người tham gia cho biết họ muốn có thể sử dụng exceptions trong Go, và điều này có tương quan nghịch với kinh nghiệm, các nhà phát triển Go kỳ cựu hơn ít có khả năng quan tâm đến exceptions hơn so với những người tham gia mới hơn với cộng đồng Go.

<img src="survey2023h2/error_handling.svg" alt="Biểu đồ suy nghĩ của người tham gia về cách tiếp cận xử lý lỗi của Go" class="chart" />

## Hiểu các trường hợp sử dụng ML/AI {#mlai}

Nhóm Go đang xem xét cách bối cảnh các công nghệ ML/AI mới đang phát triển có thể ảnh hưởng đến phát triển phần mềm theo hai hướng riêng biệt: 1) hệ thống công cụ ML/AI có thể giúp các kỹ sư viết phần mềm tốt hơn như thế nào, và 2) Go có thể giúp các kỹ sư mang hỗ trợ ML/AI vào ứng dụng và dịch vụ của họ như thế nào? Dưới đây, chúng tôi đi sâu vào từng lĩnh vực này.

### Giúp các kỹ sư viết phần mềm tốt hơn

Không thể phủ nhận chúng ta đang ở [một chu kỳ hype xung quanh các khả năng cho AI/ML](https://www.gartner.com/en/articles/what-s-new-in-artificial-intelligence-from-the-2023-gartner-hype-cycle). Chúng tôi muốn lùi lại để tập trung vào những thách thức rộng lớn hơn mà nhà phát triển gặp phải và nơi họ nghĩ AI có thể hữu ích trong công việc thường ngày. Các câu trả lời khá đáng ngạc nhiên, đặc biệt là với sự tập trung hiện tại của ngành vào trợ lý code.

Đầu tiên, chúng tôi thấy một vài trường hợp sử dụng AI mà khoảng một nửa người tham gia cho là hữu ích: tạo test (49%), gợi ý phương pháp hay nhất tại chỗ (47%), và phát hiện sớm các lỗi có thể xảy ra trong quá trình phát triển (46%). Chủ đề thống nhất của những trường hợp sử dụng hàng đầu này là mỗi trường hợp có thể giúp cải thiện chất lượng và độ tin cậy của code mà kỹ sư đang viết. Một trường hợp sử dụng thứ tư (giúp viết tài liệu) thu hút sự quan tâm từ khoảng ⅓ người tham gia. Các trường hợp còn lại bao gồm một đuôi dài các ý tưởng có tiềm năng phong phú, nhưng những trường hợp này có ít sự quan tâm chung hơn so với bốn trường hợp hàng đầu.

Khi chúng tôi xem xét thời gian kinh nghiệm của nhà phát triển với Go, chúng tôi thấy rằng người tham gia mới quan tâm đến việc giúp giải quyết lỗi trình biên dịch và giải thích những gì một đoạn code Go làm nhiều hơn so với các nhà phát triển Go kỳ cựu. Đây có thể là những lĩnh vực mà AI có thể giúp cải thiện trải nghiệm bắt đầu cho các Gopher mới; ví dụ, một trợ lý AI có thể giúp giải thích bằng ngôn ngữ tự nhiên những gì một khối code không có tài liệu làm, hoặc gợi ý các giải pháp phổ biến cho các thông báo lỗi cụ thể. Ngược lại, chúng tôi không thấy sự khác biệt giữa các mức kinh nghiệm đối với các chủ đề như "phát hiện lỗi phổ biến", cả nhà phát triển Go mới và kỳ cựu đều cho biết họ sẽ đánh giá cao hệ thống công cụ giúp với điều này.

Có thể nhìn vào dữ liệu này và thấy ba xu hướng rộng:

1. Người tham gia bày tỏ sự quan tâm trong việc nhận phản hồi từ "người đánh giá chuyên gia" trong thời gian thực, không chỉ trong thời gian đánh giá.
1. Nói chung, người tham gia có vẻ quan tâm nhất đến hệ thống công cụ giúp họ tránh khỏi các nhiệm vụ ít thú vị hơn (ví dụ: viết test hoặc ghi chép code).
1. Viết hoặc dịch code hàng loạt có sự quan tâm khá thấp, đặc biệt là đối với các nhà phát triển có hơn một hoặc hai năm kinh nghiệm.

Nhìn chung, có vẻ như ngày hôm nay, các nhà phát triển ít hào hứng hơn với viễn cảnh các máy móc thực hiện các phần thú vị (ví dụ: sáng tạo, thú vị, thách thức phù hợp) của phát triển phần mềm, nhưng thấy giá trị trong một bộ "mắt" khác xem xét code của họ và có thể xử lý các nhiệm vụ nhàm chán hoặc lặp đi lặp lại cho họ. Như một người tham gia đã diễn đạt:

> "Tôi đặc biệt quan tâm đến việc sử dụng AI/ML để cải thiện năng suất của mình với Go. Có một hệ thống được đào tạo theo các phương pháp hay nhất Go, có thể phát hiện các anti-pattern, lỗi, tạo test, với tỷ lệ ảo giác thấp, sẽ rất tuyệt vời." <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 5 -- 9 năm kinh nghiệm</span>

Tuy nhiên, cuộc khảo sát này chỉ là một điểm dữ liệu trong một lĩnh vực nghiên cứu đang phát triển nhanh chóng, vì vậy tốt nhất là nên giữ những kết quả này trong bối cảnh.

<img src="survey2023h2/ml_use_cases.svg" alt="Biểu đồ về sự quan tâm của người tham gia đến hỗ trợ AI/ML cho các nhiệm vụ phát triển" class="chart" />

### Mang tính năng AI vào ứng dụng và dịch vụ

Ngoài việc xem xét cách các nhà phát triển Go có thể hưởng lợi từ hệ thống công cụ dựa trên AI/ML, chúng tôi đã khám phá kế hoạch của họ để xây dựng các ứng dụng và dịch vụ hỗ trợ AI (hoặc cơ sở hạ tầng hỗ trợ) với Go. Chúng tôi thấy rằng chúng ta vẫn đang ở giai đoạn đầu của [đường cong áp dụng](https://en.wikipedia.org/wiki/Technology_adoption_life_cycle): hầu hết người tham gia chưa thử sử dụng Go trong những lĩnh vực này, mặc dù mỗi chủ đề đều được khoảng một nửa người tham gia quan tâm ở một mức độ nào đó. Ví dụ, đa số người tham gia báo cáo quan tâm đến việc tích hợp các dịch vụ Go họ làm việc với LLMs (49%), nhưng chỉ 13% đã làm như vậy hoặc đang đánh giá trường hợp sử dụng này. Tại thời điểm của cuộc khảo sát này, các phản hồi nhẹ nhàng gợi ý rằng các nhà phát triển có thể quan tâm nhất đến việc sử dụng Go để gọi trực tiếp LLMs, xây dựng các pipeline dữ liệu cần thiết để cung cấp cho hệ thống ML/AI, và tạo các endpoint API mà các dịch vụ khác có thể gọi để tương tác với các mô hình ML/AI.

> "Tôi muốn tích hợp phần ETL [extract, transform, and load] bằng Go, để giữ một codebase nhất quán, mạnh mẽ, đáng tin cậy." <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 3 -- 4 năm kinh nghiệm</span>

<img src="survey2023h2/ml_adoption.svg" alt="Biểu đồ về việc sử dụng hiện tại (và sự quan tâm) của người tham gia đối với Go cho hệ thống AI/ML" class="chart" />

## Thông báo lỗi toolchain {#err_msgs}

Nhiều nhà phát triển có thể liên tưởng đến trải nghiệm bực bội khi nhìn thấy một thông báo lỗi, nghĩ rằng họ biết ý nghĩa của nó và cách giải quyết, nhưng sau nhiều giờ debug vô ích mới nhận ra nó có nghĩa hoàn toàn khác. Một người tham gia đã giải thích sự thất vọng của họ như sau:

> "Thường thì những phàn nàn được in ra không liên quan gì đến vấn đề, nhưng có thể mất một giờ trước khi tôi phát hiện ra điều đó. Các thông báo lỗi ngắn gọn đến mức đáng lo ngại, và dường như không cố gắng đoán xem người dùng đang cố gắng làm gì hoặc [giải thích những gì họ] đang làm sai." <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 10+ năm kinh nghiệm</span>

Chúng tôi tin rằng cảnh báo và lỗi do hệ thống công cụ nhà phát triển phát ra phải ngắn gọn, dễ hiểu, và có thể hành động: người đọc chúng phải có thể hiểu chính xác điều gì đã xảy ra và họ có thể làm gì để giải quyết vấn đề. Đây là một tiêu chuẩn cao nhất định phải phấn đấu đạt được, và với cuộc khảo sát này, chúng tôi đã thực hiện một số đo lường để hiểu cách các nhà phát triển nhận thức các cảnh báo và thông báo lỗi hiện tại của Go.

Khi nghĩ về thông báo lỗi Go gần đây nhất họ đã xử lý, người tham gia cho chúng tôi biết còn nhiều chỗ cần cải thiện. Chỉ có đa số nhỏ hiểu được vấn đề là gì chỉ từ thông báo lỗi (54%), và thậm chí ít hơn biết phải làm gì tiếp theo để giải quyết vấn đề (41%). Có vẻ như một lượng thông tin bổ sung tương đối nhỏ có thể tăng đáng kể các tỷ lệ này, vì ¼ người tham gia cho biết họ hầu như biết cách sửa vấn đề, nhưng cần xem ví dụ trước. Hơn nữa, với 11% người tham gia cho biết họ không thể hiểu thông báo lỗi, bây giờ chúng tôi có cơ sở về khả năng hiểu hiện tại của các thông báo lỗi của toolchain Go.

Các cải tiến đối với các thông báo lỗi toolchain của Go sẽ đặc biệt có lợi cho các Gopher ít kinh nghiệm hơn. Người tham gia có đến hai năm kinh nghiệm ít có khả năng hơn so với các Gopher kỳ cựu để nói rằng họ hiểu vấn đề (47% so với 61%) hoặc biết cách sửa nó (29% so với 52%), và có gấp đôi khả năng cần phải tìm kiếm trực tuyến để sửa vấn đề (21% so với 9%) hoặc thậm chí hiểu ý nghĩa của lỗi (15% so với 7%).

Chúng tôi hy vọng tập trung vào việc cải thiện các thông báo lỗi toolchain trong năm 2024. Những kết quả khảo sát này cho thấy đây là lĩnh vực gây thất vọng cho các nhà phát triển ở tất cả các mức kinh nghiệm, và sẽ đặc biệt giúp các nhà phát triển mới hơn bắt đầu với Go.

<img src="survey2023h2/err_exp.svg" alt="Biểu đồ về kinh nghiệm xử lý lỗi" class="chart" />

<img src="survey2023h2/err_exp_exp.svg" alt="Biểu đồ về kinh nghiệm xử lý lỗi, phân chia theo thời gian kinh nghiệm Go" class="chart" />

Để hiểu *cách* các thông báo này có thể được cải thiện, chúng tôi đã hỏi người tham gia khảo sát một câu hỏi mở: "Nếu bạn có thể ước và cải thiện một điều về thông báo lỗi trong toolchain Go, bạn sẽ thay đổi điều gì?". Các câu trả lời phần lớn phù hợp với giả thuyết của chúng tôi rằng thông báo lỗi tốt phải vừa dễ hiểu vừa có thể hành động. Phản hồi phổ biến nhất là một số dạng "Giúp tôi hiểu điều gì đã dẫn đến lỗi này" (36%), 21% người tham gia đã yêu cầu rõ ràng hướng dẫn để sửa vấn đề, và 14% người tham gia đã đề cập đến các ngôn ngữ như Rust hoặc Elm là những ví dụ điển hình đang cố gắng làm cả hai điều này. Theo lời của một người tham gia:

> "Đối với lỗi biên dịch, đầu ra theo phong cách Elm hoặc Rust chỉ ra chính xác vấn đề trong mã nguồn. Các lỗi nên bao gồm gợi ý để sửa chúng khi có thể... Tôi nghĩ một chính sách chung là 'tối ưu hóa đầu ra lỗi để đọc bởi con người' với 'cung cấp gợi ý khi có thể' sẽ rất được chào đón ở đây." <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 5 -- 9 năm kinh nghiệm</span>

Có thể hiểu được, có ranh giới khái niệm mờ nhạt giữa thông báo lỗi toolchain và thông báo lỗi runtime. Ví dụ, một trong những yêu cầu hàng đầu là cải thiện stack trace hoặc các cách tiếp cận khác để hỗ trợ debug crash runtime (22%). Tương tự, một chủ đề đáng ngạc nhiên trong 4% phản hồi là về những thách thức khi nhận trợ giúp từ chính lệnh `go`. Đây là những ví dụ tuyệt vời về cộng đồng Go giúp chúng tôi xác định các điểm đau liên quan không có trên radar của chúng tôi. Chúng tôi bắt đầu cuộc điều tra này tập trung vào việc cải thiện lỗi compile-time, nhưng một trong những lĩnh vực cốt lõi mà các nhà phát triển Go muốn thấy cải thiện thực sự liên quan đến lỗi run-time, trong khi một lĩnh vực khác là về hệ thống trợ giúp của lệnh `go`.

> "Khi một lỗi được ném ra, call stack có thể rất lớn và bao gồm một loạt các tệp tôi không quan tâm. Tôi chỉ muốn biết vấn đề ở đâu trong CODE CỦA TÔI, không phải thư viện tôi đang sử dụng, hoặc cách panic được xử lý." <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 1 -- 2 năm kinh nghiệm</span>

> "Lấy trợ giúp qua \`go help run\` đổ ra một bức tường văn bản, với các liên kết đến các bài đọc thêm để tìm các cờ dòng lệnh khả dụng. Hoặc thực tế là nó hiểu \`go run --help\` nhưng thay vì hiển thị trợ giúp, nó nói 'hãy chạy go help run thay thế'. Chỉ cho tôi xem danh sách các cờ trong \`go run --help\`." <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 3 -- 4 năm kinh nghiệm</span>

<img src="survey2023h2/text_err_wish.svg" alt="Biểu đồ về các cải tiến tiềm năng cho các thông báo lỗi của Go" class="chart" />

## Microservices {#microservices}

Chúng tôi thường nghe rằng các nhà phát triển thấy Go phù hợp tốt cho microservices, nhưng chúng tôi chưa bao giờ cố gắng định lượng có bao nhiêu nhà phát triển Go đã áp dụng loại kiến trúc dịch vụ này, hiểu cách các dịch vụ đó giao tiếp với nhau, hoặc những thách thức mà nhà phát triển gặp phải khi làm việc trên chúng. Năm nay chúng tôi đã thêm một vài câu hỏi để hiểu rõ hơn lĩnh vực này.

Đa số người tham gia cho biết họ chủ yếu làm việc trên microservices (43%), với một phần tư khác cho biết họ làm việc trên cả microservices và monolith. Chỉ khoảng ⅕ người tham gia chủ yếu làm việc trên các ứng dụng Go monolithic. Đây là một trong số ít lĩnh vực chúng tôi thấy sự khác biệt dựa trên quy mô tổ chức của người tham gia làm việc, các tổ chức lớn dường như có nhiều khả năng áp dụng kiến trúc microservice hơn so với các công ty nhỏ hơn. Người tham gia từ tổ chức lớn (>1.000 nhân viên) có nhiều khả năng nhất cho biết họ làm việc trên microservices (55%), với chỉ 11% những người tham gia này chủ yếu làm việc trên monolith.

<img src="survey2023h2/service_arch.svg" alt="Biểu đồ về kiến trúc dịch vụ chính của người tham gia" class="chart" />

Chúng tôi thấy một số sự phân kỳ trong số lượng microservices tạo nên các nền tảng Go. Một nhóm bao gồm một số ít (2 đến 5) dịch vụ (40%), trong khi nhóm kia bao gồm các tập hợp lớn hơn, với tối thiểu 10 dịch vụ thành phần (37%). Số lượng microservices liên quan có vẻ không tương quan với quy mô tổ chức.

<img src="survey2023h2/service_num.svg" alt="Biểu đồ về số lượng microservices trong hệ thống của người tham gia" class="chart" />

Đa số lớn người tham gia sử dụng một số dạng yêu cầu phản hồi trực tiếp (ví dụ: RPC, HTTP, v.v.) để giao tiếp microservice (72%). Một tỷ lệ nhỏ hơn sử dụng hàng đợi tin nhắn (14%) hoặc cách tiếp cận pub/sub (9%); một lần nữa, chúng tôi không thấy sự khác biệt nào ở đây dựa trên quy mô tổ chức.

<img src="survey2023h2/service_comm.svg" alt="Biểu đồ về cách các microservices giao tiếp với nhau" class="chart" />

Đa số người tham gia xây dựng microservices với nhiều ngôn ngữ, với chỉ khoảng ¼ sử dụng độc quyền Go. Python là ngôn ngữ đồng hành phổ biến nhất (33%), cùng với Node.js (28%) và Java (26%). Chúng tôi lại thấy sự khác biệt dựa trên quy mô tổ chức, với các tổ chức lớn hơn có nhiều khả năng tích hợp microservices Python (43%) và Java (36%), trong khi các tổ chức nhỏ hơn có phần nhiều hơn khả năng chỉ sử dụng Go (30%). Các ngôn ngữ khác có vẻ được sử dụng đều nhau dựa trên quy mô tổ chức.

<img src="survey2023h2/service_lang.svg" alt="Biểu đồ về các ngôn ngữ khác mà microservices Go tương tác với" class="chart" />

Nhìn chung, người tham gia cho chúng tôi biết testing và debugging là thách thức lớn nhất của họ khi viết các ứng dụng dựa trên microservice, tiếp theo là độ phức tạp vận hành. Nhiều thách thức khác chiếm đuôi dài trên biểu đồ này, mặc dù "khả năng di chuyển" nổi bật như là không phải là vấn đề đối với hầu hết người tham gia. Chúng tôi diễn giải điều này có nghĩa là các dịch vụ như vậy không có ý định di chuyển được (ngoài containerization cơ bản); ví dụ, nếu microservices của một tổ chức ban đầu được cung cấp bởi cơ sở dữ liệu PostgreSQL, các nhà phát triển không lo ngại về việc có thể chuyển sang cơ sở dữ liệu Oracle trong tương lai gần.

<img src="survey2023h2/service_challenge.svg" alt="Biểu đồ về những thách thức người tham gia gặp phải khi viết ứng dụng dựa trên microservice" class="chart" />

## Tác giả và bảo trì module {#modules}

Go có một hệ sinh thái module do cộng đồng thúc đẩy phong phú, và chúng tôi muốn hiểu những động lực và thách thức mà các nhà phát triển duy trì các module này phải đối mặt. Chúng tôi thấy rằng khoảng ⅕ người tham gia duy trì (hoặc đã từng duy trì) một module Go mã nguồn mở. Đây là một tỷ lệ cao đáng ngạc nhiên, và có thể bị sai lệch do cách chúng tôi chia sẻ cuộc khảo sát này: những người duy trì module có thể có nhiều khả năng theo dõi chặt chẽ blog Go (nơi cuộc khảo sát này được công bố) hơn so với các nhà phát triển Go khác.

<img src="survey2023h2/mod_maintainer.svg" alt="Biểu đồ về có bao nhiêu người tham gia đã từng là người duy trì cho một module Go công khai" class="chart" />

Những người duy trì module có vẻ chủ yếu tự thúc đẩy, họ báo cáo làm việc trên các module mà họ cần cho các dự án cá nhân (58%) hoặc công việc (56%), rằng họ làm như vậy vì họ thích làm việc trên các module này (63%) và là một phần của cộng đồng Go công khai (44%), và rằng họ học được các kỹ năng hữu ích từ việc duy trì module của họ (44%). Các động lực bên ngoài hơn, chẳng hạn như nhận được sự công nhận (15%), thăng tiến sự nghiệp (36%), hoặc tiền mặt (20%) nằm ở cuối danh sách.

<img src="survey2023h2/mod_motivation.svg" alt="Biểu đồ về động lực của những người duy trì module công khai" class="chart" />

Do các hình thức [động lực nội tại](https://en.wikipedia.org/wiki/Motivation#Intrinsic_and_extrinsic) được xác định ở trên, sau đây là một thách thức quan trọng cho những người duy trì module là tìm thời gian để dành cho module của họ (41%). Mặc dù điều này có thể không có vẻ như là một phát hiện có thể hành động (chúng tôi không thể cho các nhà phát triển Go thêm một hoặc hai giờ mỗi ngày, đúng không?), nhưng đây là một lăng kính hữu ích để xem xét hệ thống công cụ và phát triển module, những nhiệm vụ này rất có thể xảy ra khi nhà phát triển đã bị áp lực về thời gian, và có thể đã vài tuần hoặc vài tháng kể từ khi họ có cơ hội làm việc trên nó, vì vậy mọi thứ không còn mới mẻ trong tâm trí họ. Do đó, các khía cạnh như thông báo lỗi dễ hiểu và có thể hành động có thể đặc biệt hữu ích: thay vì yêu cầu ai đó một lần nữa tìm kiếm cú pháp lệnh `go` cụ thể, có lẽ đầu ra lỗi có thể cung cấp giải pháp họ cần ngay trong terminal của họ.

<img src="survey2023h2/mod_challenge.svg" alt="Biểu đồ về những thách thức người tham gia gặp phải khi duy trì module Go công khai" class="chart" />

## Nhân khẩu học {#demographics}

Hầu hết người tham gia khảo sát báo cáo sử dụng Go cho công việc chính của họ (78%), và đa số (59%) cho biết họ sử dụng nó cho các dự án cá nhân hoặc mã nguồn mở. Thực tế là phổ biến khi người tham gia sử dụng Go cho cả công việc *và* dự án cá nhân/OSS, với 43% người tham gia cho biết họ sử dụng Go trong mỗi tình huống này.

<img src="survey2023h2/where.svg" alt="Biểu đồ về các tình huống người tham gia gần đây đã sử dụng Go" class="chart" />

Đa số người tham gia đã làm việc với Go dưới năm năm (68%). Như chúng tôi đã thấy trong [các năm trước](/blog/survey2023-q1-results#novice-respondents-are-more-likely-to-prefer-windows-than-more-experienced-respondents), những người tìm thấy cuộc khảo sát này qua VS Code có xu hướng ít kinh nghiệm hơn so với những người tìm thấy cuộc khảo sát qua các kênh khác.

Khi chúng tôi phân tích nơi mọi người sử dụng Go theo mức độ kinh nghiệm của họ, hai phát hiện nổi bật. Đầu tiên, đa số người tham gia từ tất cả các mức kinh nghiệm cho biết họ đang sử dụng Go chuyên nghiệp; thực tế là, đối với những người có hơn hai năm kinh nghiệm, phần lớn sử dụng Go trong công việc (85% -- 91%). Xu hướng tương tự tồn tại cho việc phát triển mã nguồn mở. Phát hiện thứ hai là các nhà phát triển có ít kinh nghiệm Go hơn có nhiều khả năng sử dụng Go để mở rộng kỹ năng của họ (38%) hoặc để đánh giá nó để sử dụng trong công việc (13%) so với các nhà phát triển Go có kinh nghiệm hơn. Chúng tôi diễn giải điều này có nghĩa là nhiều Gopher ban đầu coi Go là một phần của "upskilling" hoặc mở rộng hiểu biết về phát triển phần mềm, nhưng trong vòng một hoặc hai năm, họ nhìn Go là công cụ để làm hơn là học.

<img src="survey2023h2/go_exp.svg" alt="Biểu đồ về thời gian người tham gia đã làm việc với Go" class="chart" />

<img src="survey2023h2/where_exp.svg" alt="Biểu đồ về các tình huống người tham gia gần đây đã sử dụng Go, phân chia theo mức độ kinh nghiệm Go của họ" class="chart" />

Các trường hợp sử dụng phổ biến nhất cho Go tiếp tục là dịch vụ API/RPC (74%) và công cụ dòng lệnh (62%). Mọi người cho chúng tôi biết Go là lựa chọn tuyệt vời cho các loại phần mềm này vì nhiều lý do, bao gồm máy chủ HTTP tích hợp và các primitives đồng thời, dễ cross-compilation, và triển khai binary đơn.

Đối tượng dự định của nhiều hệ thống công cụ này là trong môi trường kinh doanh (62%), với 17% người tham gia báo cáo rằng họ phát triển chủ yếu cho các ứng dụng hướng người tiêu dùng hơn. Điều này không có gì đáng ngạc nhiên khi biết rằng Go ít được sử dụng cho các ứng dụng hướng người tiêu dùng như desktop, mobile, hoặc gaming, so với việc sử dụng rất nhiều cho các dịch vụ backend, hệ thống công cụ CLI, và phát triển đám mây, nhưng đây là sự xác nhận hữu ích về mức độ Go được sử dụng nhiều trong các môi trường B2B.

<img src="survey2023h2/what.svg" alt="Biểu đồ về các loại thứ người tham gia đang xây dựng với Go" class="chart" />

<img src="survey2023h2/enduser.svg" alt="Biểu đồ về đối tượng sử dụng phần mềm người tham gia xây dựng" class="chart" />

Người tham gia có khoảng ngang nhau khi nói rằng đây là lần đầu tiên họ trả lời Khảo sát Nhà phát triển Go so với nói rằng họ đã thực hiện cuộc khảo sát này trước đây. Có sự khác biệt có ý nghĩa giữa những người biết về cuộc khảo sát này qua blog Go, trong đó 61% báo cáo đã thực hiện cuộc khảo sát này trước đây, so với những người biết về cuộc khảo sát qua thông báo trong VS Code, trong đó chỉ 31% cho biết họ đã thực hiện cuộc khảo sát này trước đây. Chúng tôi không kỳ vọng mọi người hoàn toàn nhớ mọi cuộc khảo sát họ đã trả lời trên internet, nhưng điều này cho chúng tôi một số sự tin tưởng rằng chúng tôi đang nghe từ sự kết hợp cân bằng giữa người tham gia mới và lặp lại trong mỗi cuộc khảo sát. Hơn nữa, điều này cho chúng tôi biết rằng sự kết hợp của các bài đăng truyền thông xã hội và lấy mẫu ngẫu nhiên trong trình soạn thảo đều cần thiết để nghe từ tập hợp đa dạng các nhà phát triển Go.

<img src="survey2023h2/return_respondent.svg" alt="Biểu đồ về có bao nhiêu người tham gia cho biết họ đã thực hiện cuộc khảo sát này trước đây" class="chart" />

## Thống kê tổ chức {#firmographics}

Người tham gia cuộc khảo sát này báo cáo làm việc tại nhiều tổ chức khác nhau, từ các doanh nghiệp hơn nghìn người (27%), đến các doanh nghiệp vừa (25%) và các tổ chức nhỏ hơn với < 100 nhân viên (44%). Khoảng một nửa người tham gia làm việc trong ngành công nghệ (50%), tăng lớn so với ngành phổ biến tiếp theo, dịch vụ tài chính, ở mức 13%.

Điều này thống kê không thay đổi so với vài Khảo sát Nhà phát triển Go trước đây, chúng tôi tiếp tục nghe từ những người ở các quốc gia khác nhau và trong các tổ chức có quy mô và ngành khác nhau ở các tỷ lệ nhất quán từ năm này sang năm khác.

<img src="survey2023h2/org_size.svg" alt="Biểu đồ về các quy mô tổ chức khác nhau nơi người tham gia sử dụng Go" class="chart" />

<img src="survey2023h2/industry.svg" alt="Biểu đồ về các ngành khác nhau nơi người tham gia sử dụng Go" class="chart" />

<img src="survey2023h2/location.svg" alt="Biểu đồ về các quốc gia hoặc khu vực nơi người tham gia sống" class="chart" />

## Phương pháp luận {#methodology}

Hầu hết người tham gia khảo sát "tự chọn" để tham gia cuộc khảo sát này, có nghĩa là họ tìm thấy nó trên blog Go hoặc các kênh Go xã hội khác. Một vấn đề tiềm năng với cách tiếp cận này là những người không theo dõi các kênh này ít có khả năng biết về cuộc khảo sát hơn và có thể phản hồi khác với những người có theo dõi chặt chẽ. Khoảng 40% người tham gia được lấy mẫu ngẫu nhiên, có nghĩa là họ đã phản hồi cuộc khảo sát sau khi thấy lời nhắc trong VS Code (mọi người sử dụng plugin VS Code Go trong khoảng giữa tháng 7 đến giữa tháng 8 năm 2023 có 10% cơ hội nhận được lời nhắc ngẫu nhiên này). Nhóm được lấy mẫu ngẫu nhiên này giúp chúng tôi khái quát hóa những phát hiện này cho cộng đồng lớn hơn của các nhà phát triển Go.

### Cách đọc kết quả này

Xuyên suốt báo cáo này, chúng tôi sử dụng các biểu đồ phản hồi khảo sát để cung cấp bằng chứng hỗ trợ cho các phát hiện của mình. Tất cả các biểu đồ này sử dụng định dạng tương tự. Tiêu đề là câu hỏi chính xác mà người tham gia khảo sát đã thấy. Trừ khi có ghi chú khác, các câu hỏi là câu hỏi nhiều lựa chọn và người tham gia chỉ có thể chọn một câu trả lời duy nhất; phụ đề của mỗi biểu đồ sẽ cho người đọc biết liệu câu hỏi có cho phép nhiều lựa chọn hoặc là hộp văn bản mở thay vì câu hỏi nhiều lựa chọn hay không. Đối với các biểu đồ câu trả lời văn bản mở, một thành viên nhóm Go đã đọc và phân loại thủ công các câu trả lời. Nhiều câu hỏi mở thu hút nhiều loại phản hồi; để giữ kích thước biểu đồ hợp lý, chúng tôi rút gọn xuống tối đa 10 chủ đề hàng đầu, với các chủ đề bổ sung được nhóm dưới "Khác". Nhãn phần trăm được hiển thị trong biểu đồ được làm tròn đến số nguyên gần nhất (ví dụ: 1,4% và 0,8% đều sẽ được hiển thị là 1%), nhưng độ dài của mỗi thanh và thứ tự hàng dựa trên các giá trị chưa làm tròn.

Để giúp người đọc hiểu được trọng lượng bằng chứng đằng sau mỗi phát hiện, chúng tôi bao gồm các thanh lỗi hiển thị khoảng [tin cậy](https://en.wikipedia.org/wiki/Confidence_interval) 95% cho các phản hồi; thanh hẹp hơn cho thấy độ tin cậy tăng. Đôi khi hai hoặc nhiều phản hồi có các thanh lỗi chồng lên nhau, có nghĩa là thứ tự tương đối của các phản hồi đó không có ý nghĩa thống kê (tức là, các phản hồi thực tế bằng nhau). Góc dưới bên phải của mỗi biểu đồ hiển thị số người có phản hồi được đưa vào biểu đồ, dưới dạng "n = [số người tham gia]".

Chúng tôi bao gồm các trích dẫn chọn lọc từ người tham gia để giúp làm rõ nhiều phát hiện của chúng tôi. Các trích dẫn này bao gồm thời gian người tham gia đã sử dụng Go. Nếu người tham gia cho biết họ sử dụng Go trong công việc, chúng tôi gọi họ là "nhà phát triển Go chuyên nghiệp"; nếu họ không sử dụng Go trong công việc nhưng có sử dụng Go cho phát triển mã nguồn mở, chúng tôi gọi họ là "nhà phát triển Go mã nguồn mở".

## Kết luận {#closing}

Câu hỏi cuối cùng trong cuộc khảo sát của chúng tôi luôn hỏi người tham gia liệu có điều gì khác họ muốn chia sẻ với chúng tôi về Go không. Phản hồi phổ biến nhất mà mọi người cung cấp là "cảm ơn!", và năm nay cũng không khác (33%). Về các cải tiến ngôn ngữ được yêu cầu, chúng tôi thấy sự ràng buộc thống kê ba chiều giữa cải thiện khả năng biểu đạt (12%), cải thiện xử lý lỗi (12%), và cải thiện tính an toàn kiểu dữ liệu hoặc độ tin cậy (9%). Người tham gia có nhiều ý tưởng để cải thiện khả năng biểu đạt, với xu hướng chung của phản hồi này là "Đây là điều cụ thể tôi viết thường xuyên, và tôi muốn nó dễ dàng hơn để diễn đạt trong Go". Các vấn đề với xử lý lỗi tiếp tục là những phàn nàn về sự dài dòng của code này ngày hôm nay, trong khi phản hồi về tính an toàn kiểu dữ liệu thường nhất đề cập đến [sum types](https://en.wikipedia.org/wiki/Tagged_union). Loại phản hồi cấp cao này cực kỳ hữu ích khi nhóm Go cố gắng lên kế hoạch các lĩnh vực tập trung cho năm sắp tới, vì nó cho chúng tôi biết các hướng chung mà cộng đồng hy vọng định hướng hệ sinh thái.

> "Tôi biết về thái độ của Go đối với sự đơn giản và tôi trân trọng nó. Tôi chỉ muốn [có] thêm một chút tính năng. Đối với tôi, sẽ tốt hơn nếu có xử lý lỗi (không phải exceptions), và có thể một số tiện nghi phổ biến như map/reduce/filter và toán tử ternary. Bất cứ điều gì không quá mơ hồ sẽ giúp tôi tiết kiệm một số câu lệnh 'if'." <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 1 -- 2 năm kinh nghiệm</span>

> "Xin hãy giữ Go phù hợp với các giá trị lâu dài mà Go đã thiết lập từ lâu, sự ổn định ngôn ngữ và thư viện. [...] Đó là một môi trường tôi có thể tin tưởng để không làm hỏng code của tôi sau 2 hoặc 3 năm. Vì điều đó, cảm ơn rất nhiều." <span class="quote_source">--- Nhà phát triển Go chuyên nghiệp có 10+ năm kinh nghiệm</span>

<img src="survey2023h2/text_anything_else.svg" alt="Biểu đồ về các chủ đề khác mà người tham gia đã chia sẻ với chúng tôi" class="chart" />

Đó là tất cả cho phiên bản định kỳ sáu tháng một lần này của Khảo sát Nhà phát triển Go. Cảm ơn tất cả những người đã chia sẻ phản hồi của họ về Go, chúng tôi vô cùng biết ơn vì bạn đã dành thời gian giúp định hình tương lai của Go, và chúng tôi hy vọng bạn thấy một số phản hồi của chính mình được phản ánh trong báo cáo này. 🩵

--- Todd (thay mặt nhóm Go tại Google)
