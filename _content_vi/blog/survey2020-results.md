---
title: Kết quả Khảo sát Developer Go 2020
date: 2021-03-09
by:
- Alice Merrick
tags:
- survey
- community
summary: Phân tích kết quả từ Khảo sát Developer Go 2020.
---

## Cảm ơn vì lượng phản hồi tuyệt vời! {#thanks}

Năm 2020, chúng tôi lại có lượt tham gia tuyệt vời với 9.648 phản hồi, gần [bằng năm 2019](/blog/survey2019-results). Cảm ơn bạn đã dành thời gian cung cấp cho cộng đồng những thông tin về trải nghiệm sử dụng Go của bạn!

## Thiết kế khảo sát theo module mới {#new}

Bạn có thể nhận thấy rằng một số câu hỏi có cỡ mẫu nhỏ hơn ("n=") so với các câu hỏi khác. Điều đó là vì một số câu hỏi được hiển thị cho tất cả mọi người trong khi một số khác chỉ được hiển thị cho một tập hợp con ngẫu nhiên của người trả lời.

## Điểm nổi bật {#highlights}

- Việc sử dụng Go đang mở rộng trong môi trường làm việc và doanh nghiệp, với 76% người trả lời [sử dụng Go tại nơi làm việc](#TOC_4.1) và 66% cho biết [Go rất quan trọng đối với sự thành công của công ty họ](#TOC_6.1).
- [Sự hài lòng tổng thể](#TOC_6.) cao với 92% người trả lời hài lòng khi sử dụng Go.
- Đa số [người trả lời cảm thấy năng suất](#TOC_6.2) trong Go dưới 3 tháng, với 81% cảm thấy rất hoặc cực kỳ có năng suất với Go.
- Người trả lời cho biết [nâng cấp nhanh lên phiên bản Go mới nhất](#TOC_7.), với 76% trong 5 tháng đầu tiên.
- [Người dùng pkg.go.dev thành công hơn (91%)](#TOC_12.) trong việc tìm kiếm các package Go so với người không dùng (82%).
- [Việc áp dụng module](#TOC_8.) trong Go gần như phổ quát với 77% hài lòng, nhưng người trả lời cũng nhấn mạnh nhu cầu cải thiện tài liệu.
- Go tiếp tục được sử dụng nhiều cho [API, CLI, Web, DevOps và Xử lý dữ liệu](#TOC_7.).
- [Các nhóm thiểu đại diện](#TOC_12.1) có xu hướng cảm thấy ít được chào đón hơn trong cộng đồng.


## Chúng tôi đã nghe từ ai? {#who}

Các câu hỏi nhân khẩu học giúp chúng tôi phân biệt những sự khác biệt theo năm nào có thể do thay đổi trong đối tượng trả lời khảo sát so với những thay đổi trong tình cảm hay hành vi. Vì nhân khẩu học của chúng tôi tương tự năm ngoái, chúng tôi có thể khá tự tin rằng các thay đổi theo năm khác không chủ yếu là do sự thay đổi nhân khẩu học.

Ví dụ, phân bố quy mô tổ chức, kinh nghiệm developer và ngành công nghiệp vẫn gần như nhau từ năm 2019 đến 2020.

<img src="survey2020/orgsize.svg" alt="Biểu đồ cột về quy mô tổ chức từ năm 2019 đến 2020, trong đó đa số có ít hơn 1000 nhân viên" width="700"/>

<img src="survey2020/devex_yoy.svg" alt="Biểu đồ cột về số năm kinh nghiệm chuyên nghiệp từ năm 2019 đến 2020, với đa số có từ 3 đến 10 năm kinh nghiệm" width="700"/>
<img src="survey2020/industry_yoy.svg" alt="Biểu đồ cột về ngành công nghiệp tổ chức từ năm 2019 đến 2020, với đa số trong ngành Công nghệ" width="700"/>

Gần một nửa (48%) người trả lời đã sử dụng Go dưới hai năm. Năm 2020, chúng tôi có ít phản hồi hơn từ những người đã sử dụng Go dưới một năm.

<img src="survey2020/goex_yoy.svg" alt="Biểu đồ cột về số năm kinh nghiệm sử dụng Go" width="700"/>


<p id="TOC_4.1">Đa số cho biết họ sử dụng Go tại nơi làm việc (76%) và ngoài giờ làm việc (62%).
Tỷ lệ người trả lời sử dụng Go tại nơi làm việc đang có xu hướng tăng dần qua các năm.</p>

<img src="survey2020/where_yoy.svg" alt="Biểu đồ cột về nơi Go đang được sử dụng tại nơi làm việc hoặc ngoài giờ làm việc" width="700"/>

Năm nay chúng tôi đã giới thiệu một câu hỏi mới về trách nhiệm công việc chính. Chúng tôi thấy rằng trách nhiệm chính của 70% người trả lời là phát triển phần mềm và ứng dụng, nhưng một thiểu số đáng kể (10%) đang thiết kế hệ thống và kiến trúc IT.

<img src="survey2020/job_responsibility.svg" alt="Trách nhiệm công việc chính" width="700"/>

Giống như các năm trước, chúng tôi thấy rằng hầu hết người trả lời không thường xuyên đóng góp
cho các dự án mã nguồn mở Go, với 75% cho biết họ làm như vậy "không thường xuyên" hoặc "không bao giờ".

<img src="survey2020/foss_yoy.svg" alt="Tần suất người trả lời đóng góp cho các dự án mã nguồn mở viết bằng Go từ năm 2017 đến 2020, trong đó kết quả vẫn gần như giống nhau mỗi năm và chỉ có 7% đóng góp hàng ngày" width="700"/>

## Công cụ và thực hành developer {#devtools}

Giống như các năm trước, đại đa số người trả lời khảo sát cho biết họ làm việc
với Go trên các hệ thống Linux (63%) và macOS (55%). Tỷ lệ người trả lời chủ yếu phát triển trên Linux có vẻ đang giảm nhẹ theo thời gian.

<img src="survey2020/os_yoy.svg" alt="Hệ điều hành chính từ năm 2017 đến 2020" width="700"/>

Lần đầu tiên, sở thích trình soạn thảo có vẻ đã ổn định: VS Code vẫn là trình soạn thảo được ưa thích nhất (41%), với GoLand đứng thứ hai mạnh mẽ (35%). Hai trình soạn thảo này chiếm 76% số câu trả lời, và các sở thích khác không tiếp tục giảm như ở những năm trước.

<img src="survey2020/editor_pref_yoy.svg" alt="Sở thích trình soạn thảo từ năm 2017 đến 2020" width="700"/>

Năm nay chúng tôi đề nghị người trả lời ưu tiên các cải tiến cho trình soạn thảo của họ theo số lượng "GopherCoin" giả định họ sẽ chi ra nếu họ có 100 đồng (một loại tiền tệ hư cấu). Hoàn thành mã nhận được số GopherCoin trung bình cao nhất mỗi người trả lời. Một nửa số người trả lời trao cho 4 tính năng hàng đầu (hoàn thành mã, điều hướng mã, hiệu suất trình soạn thảo và tái cấu trúc) 10 hoặc nhiều đồng hơn.

<img src="survey2020/editor_improvements_means.svg" alt="Biểu đồ cột về số GopherCoin trung bình được chi mỗi người trả lời" width="700"/>

Đa số người trả lời (63%) dành 10 đến 30% thời gian của họ để tái cấu trúc, gợi ý rằng đây là một nhiệm vụ phổ biến và chúng tôi muốn khám phá các cách để cải thiện nó. Điều này cũng giải thích tại sao hỗ trợ tái cấu trúc là một trong những cải tiến trình soạn thảo được đầu tư nhiều nhất.

<img src="survey2020/refactor_time.svg" alt="Biểu đồ cột về thời gian dành cho việc tái cấu trúc" width="700"/>

Năm ngoái chúng tôi đã hỏi về các kỹ thuật developer cụ thể và phát hiện ra rằng gần 90% người trả lời đang sử dụng text logging để gỡ lỗi, vì vậy năm nay chúng tôi đã thêm câu hỏi tiếp theo để tìm hiểu lý do tại sao. Kết quả cho thấy rằng 43% sử dụng nó vì nó cho phép họ sử dụng cùng chiến lược gỡ lỗi trên các ngôn ngữ khác nhau, và 42% thích sử dụng text logging hơn các kỹ thuật gỡ lỗi khác.
Tuy nhiên, 27% không biết cách bắt đầu với các công cụ gỡ lỗi của Go và 24% chưa bao giờ thử sử dụng các công cụ gỡ lỗi của Go, vì vậy có cơ hội để cải thiện hệ thống công cụ gỡ lỗi về khả năng khám phá, khả năng sử dụng và tài liệu.
Ngoài ra, vì một phần tư người trả lời chưa bao giờ thử sử dụng các công cụ gỡ lỗi, các điểm đau có thể bị báo cáo thiếu.

<img src="survey2020/why_printf.svg" alt="" width="700"/>


## Cảm nhận đối với Go {#sentiments}

Lần đầu tiên, năm nay chúng tôi hỏi về sự hài lòng tổng thể. 92% người trả lời cho biết họ rất hài lòng hoặc hơi hài lòng khi sử dụng Go trong năm vừa qua.

<img src="survey2020/csat.svg" alt="Biểu đồ cột về sự hài lòng tổng thể trên thang điểm 5 từ rất không hài lòng đến rất hài lòng" width="700"/>

Đây là năm thứ 3 chúng tôi hỏi câu hỏi "Bạn có giới thiệu không..." [Net Promoter Score](https://en.wikipedia.org/wiki/Net_Promoter) (NPS). Năm nay kết quả NPS của chúng tôi là 61 (68% "người quảng bá" trừ đi 6% "người phản đối"), về mặt thống kê không thay đổi so với năm 2019 và 2018.

<img src="survey2020/nps.svg" alt="Biểu đồ cột xếp chồng của người quảng bá, người thụ động và người phản đối" width="700"/>

<p id="TOC_6.1">Tương tự như các năm trước, 91% người trả lời cho biết họ muốn sử dụng Go cho dự án mới tiếp theo. 89% cho biết Go đang hoạt động tốt cho nhóm của họ. Năm nay chúng tôi thấy sự tăng trưởng trong số người trả lời đồng ý rằng Go rất quan trọng đối với sự thành công của công ty họ, từ 59% năm 2019 lên 66% năm 2020. Người trả lời làm việc tại các tổ chức có 5.000 nhân viên trở lên ít có khả năng đồng ý hơn (63%), trong khi những người ở các tổ chức nhỏ hơn có nhiều khả năng đồng ý hơn (73%).</p>

<img src="survey2020/attitudes_yoy.svg" alt="Biểu đồ cột về sự đồng ý với các câu Tôi muốn sử dụng Go cho dự án tiếp theo của mình, Go đang hoạt động tốt cho nhóm tôi, 89%, và Go rất quan trọng đối với sự thành công của công ty tôi" width="700"/>

Giống như năm ngoái, chúng tôi đề nghị người trả lời đánh giá các lĩnh vực cụ thể của phát triển Go theo sự hài lòng và tầm quan trọng. Sự hài lòng khi sử dụng dịch vụ đám mây, gỡ lỗi và sử dụng module (các lĩnh vực năm ngoái được nhấn mạnh là cơ hội cải thiện) tăng lên trong khi hầu hết các điểm tầm quan trọng vẫn gần như nhau. Chúng tôi cũng đã giới thiệu một vài chủ đề mới: framework API và Web. Chúng tôi thấy rằng sự hài lòng với web framework thấp hơn các lĩnh vực khác (64%). Nó không phải là quá quan trọng đối với hầu hết người dùng hiện tại (chỉ có 28% người trả lời cho biết nó rất hoặc cực kỳ quan trọng), nhưng nó có thể là một tính năng quan trọng còn thiếu đối với những developer Go tiềm năng.

<img src="survey2020/feature_sat_yoy.svg" alt="Biểu đồ cột về sự hài lòng với các khía cạnh của Go từ năm 2019 đến 2020, hiển thị sự hài lòng cao nhất với tốc độ build, độ tin cậy và sử dụng concurrency và thấp nhất với web framework" width="700"/>

<p id="TOC_6.2">81% người trả lời cho biết họ cảm thấy rất hoặc cực kỳ có năng suất khi sử dụng Go. Người trả lời ở các tổ chức lớn hơn có nhiều khả năng cảm thấy cực kỳ có năng suất hơn so với những người ở các tổ chức nhỏ hơn.</p>

<img src="survey2020/prod.svg" alt="Biểu đồ cột xếp chồng về năng suất được nhận thức trên thang điểm 5 từ không chút nào đến cực kỳ có năng suất" width="700"/>

Chúng tôi đã nghe nhiều ý kiến cho rằng rất dễ dàng để đạt được năng suất cao nhanh chóng với Go. Chúng tôi đã hỏi những người trả lời cảm thấy ít nhất hơi có năng suất mất bao lâu để họ trở nên có năng suất. 93% cho biết mất chưa đến một năm, với đa số cảm thấy có năng suất trong vòng 3 tháng.

<img src="survey2020/prod_time.svg" alt="Biểu đồ cột về thời gian trước khi cảm thấy có năng suất" width="700"/>

Mặc dù gần như giống năm ngoái, tỷ lệ người trả lời đồng ý với câu "Tôi cảm thấy được chào đón trong cộng đồng Go" có vẻ đang giảm dần theo thời gian, hoặc ít nhất là không duy trì được cùng xu hướng tăng như các lĩnh vực khác.

Chúng tôi cũng thấy sự gia tăng đáng kể theo năm
trong tỷ lệ người trả lời cảm thấy ban lãnh đạo dự án Go
hiểu nhu cầu của họ (63%).

Tất cả những kết quả này cho thấy mô hình đồng ý cao hơn tương quan với
kinh nghiệm Go ngày càng nhiều, bắt đầu từ khoảng hai năm.
Nói cách khác, người trả lời đã sử dụng Go càng lâu,
thì họ càng có khả năng đồng ý với mỗi câu trong số những câu đó.

<img src="survey2020/attitudes_community_yoy.svg" alt="Biểu đồ cột hiển thị sự đồng ý với các câu Tôi cảm thấy được chào đón trong cộng đồng Go, Tôi tự tin vào ban lãnh đạo Go, Tôi cảm thấy được chào đón để đóng góp, Ban lãnh đạo dự án Go hiểu nhu cầu của tôi, và Quy trình đóng góp cho dự án Go rõ ràng đối với tôi" width="700"/>

Chúng tôi đã đặt câu hỏi văn bản mở về những gì chúng tôi có thể làm để cộng đồng Go trở nên thân thiện hơn và những khuyến nghị phổ biến nhất (21%) liên quan đến các hình thức khác nhau của hoặc cải tiến/bổ sung cho tài nguyên học tập và tài liệu.

<img src="survey2020/more_welcoming.svg" alt="Biểu đồ cột về các khuyến nghị để cải thiện sự thân thiện của cộng đồng Go" width="700"/>


## Làm việc với Go {#uses}

Xây dựng dịch vụ API/RPC (74%) và CLI (65%) vẫn là những cách sử dụng Go phổ biến nhất. Chúng tôi không thấy bất kỳ thay đổi đáng kể nào so với năm ngoái, khi chúng tôi giới thiệu ngẫu nhiên hóa vào thứ tự các tùy chọn. (Trước năm 2019, các tùy chọn ở đầu danh sách được chọn với tỷ lệ không tương xứng.)
Chúng tôi cũng đã phân tích điều này theo quy mô tổ chức và thấy rằng người trả lời sử dụng Go tương tự nhau ở các doanh nghiệp lớn hoặc tổ chức nhỏ hơn, mặc dù các tổ chức lớn ít có khả năng sử dụng Go cho các dịch vụ web trả về HTML hơn một chút.

<img src="survey2020/app_yoy.svg" alt="Biểu đồ cột về các trường hợp sử dụng Go từ năm 2019 đến 2020 bao gồm dịch vụ API hoặc RPC, CLI, framework, dịch vụ web, tự động hóa, agent và daemon, xử lý dữ liệu, GUI, trò chơi và ứng dụng di động" width="700"/>

Năm nay chúng tôi hiểu rõ hơn về loại phần mềm nào mà người trả lời viết bằng Go ở nhà so với ở nơi làm việc. Mặc dù dịch vụ web trả về HTML là trường hợp sử dụng phổ biến thứ 4, nhưng điều này là do việc sử dụng không liên quan đến công việc. Nhiều người trả lời sử dụng Go cho tự động hóa/script, agent và daemon, và xử lý dữ liệu cho công việc hơn là dịch vụ web trả về HTML.
Một tỷ lệ lớn hơn của các trường hợp sử dụng ít phổ biến nhất (ứng dụng desktop/GUI, trò chơi và ứng dụng di động) đang được viết bên ngoài công việc.

<img src="survey2020/app_context.svg" alt="Biểu đồ cột xếp chồng về tỷ lệ trường hợp sử dụng là tại nơi làm việc, ngoài giờ làm việc hay cả hai" width="700"/>

Một câu hỏi mới khác hỏi người trả lời hài lòng như thế nào với mỗi trường hợp sử dụng.
CLI có sự hài lòng cao nhất, với 85% người trả lời cho biết họ rất hài lòng, hài lòng vừa phải hoặc hơi hài lòng khi sử dụng Go cho CLI.
Các trường hợp sử dụng phổ biến của Go có xu hướng có điểm hài lòng cao hơn, nhưng sự hài lòng và mức độ phổ biến không hoàn toàn tương ứng.
Ví dụ, agent và daemon có tỷ lệ hài lòng cao thứ 2 nhưng đứng thứ 6 về mức độ sử dụng.

<img src="survey2020/app_sat_bin.svg" alt="Biểu đồ cột về sự hài lòng với mỗi trường hợp sử dụng" width="700"/>

Các câu hỏi tiếp theo khám phá các trường hợp sử dụng khác nhau, ví dụ, các nền tảng mà người trả lời hướng đến với CLI.
Không có gì ngạc nhiên khi thấy Linux (93%) và macOS (59%) được đại diện nhiều, vì developer sử dụng Linux và macOS nhiều và Linux được sử dụng nhiều trong đám mây, nhưng thậm chí Windows cũng là mục tiêu của gần một phần ba developer CLI.

<img src="survey2020/cli_platforms.svg" alt="Biểu đồ cột về các nền tảng đang được nhắm đến cho CLI" width="700"/>

Một cái nhìn cụ thể hơn về Go để xử lý dữ liệu cho thấy Kafka là engine duy nhất được áp dụng rộng rãi, nhưng đa số người trả lời cho biết họ sử dụng Go với engine xử lý dữ liệu tùy chỉnh.

<img src="survey2020/dpe.svg" alt="Biểu đồ cột về các engine xử lý dữ liệu được sử dụng bởi những người dùng Go để xử lý dữ liệu" width="700"/>

Chúng tôi cũng hỏi về các lĩnh vực rộng hơn mà người trả lời làm việc với Go.
Lĩnh vực phổ biến nhất cho đến nay là phát triển web (68%),
nhưng các lĩnh vực phổ biến khác bao gồm cơ sở dữ liệu (46%), DevOps (42%),
lập trình mạng (41%) và lập trình hệ thống (40%).

<img src="survey2020/domain_yoy.svg" alt="Biểu đồ cột về loại công việc nơi Go đang được sử dụng" width="700"/>


<p id="TOC_7.1">Tương tự như năm ngoái, chúng tôi thấy rằng 76% người trả lời đánh giá bản phát hành Go hiện tại để sử dụng trong production, nhưng năm nay chúng tôi tinh chỉnh thang thời gian và thấy rằng 60% bắt đầu đánh giá phiên bản mới trước hoặc trong vòng 2 tháng kể từ khi phát hành.
Điều này làm nổi bật tầm quan trọng của các nhà cung cấp platform-as-a-service trong việc nhanh chóng hỗ trợ các bản phát hành ổn định mới của Go.</p>


<img src="survey2020/update_time.svg" alt="Biểu đồ cột về mức độ sớm người trả lời bắt đầu đánh giá bản phát hành Go mới" width="700"/>

## Module {#modules}

Năm nay chúng tôi thấy việc áp dụng module Go gần như phổ quát, và sự gia tăng đáng kể trong tỷ lệ người trả lời chỉ sử dụng module để quản lý package. 96% người trả lời cho biết họ đang sử dụng module để quản lý package, tăng từ 89% năm ngoái. 87% người trả lời cho biết họ _chỉ_ sử dụng module để quản lý package, tăng từ 71% năm ngoái.
Trong khi đó, việc sử dụng các công cụ quản lý package khác đã giảm.

<img src="survey2020/modules_adoption_yoy.svg" alt="Biểu đồ cột về các phương pháp được sử dụng để quản lý package Go" width="700"/>

Sự hài lòng với module cũng tăng so với năm ngoái. 77% người trả lời cho biết họ rất hài lòng, hài lòng vừa phải hoặc hơi hài lòng với module, so với 68% năm 2019.

<img src="survey2020/modules_sat_yoy.svg" alt="Biểu đồ cột xếp chồng về sự hài lòng khi sử dụng module trên thang điểm 7 từ rất không hài lòng đến rất hài lòng" width="700"/>

## Tài liệu chính thức
Hầu hết người trả lời cho biết họ gặp khó khăn với tài liệu chính thức. 62% người trả lời gặp khó khăn khi tìm đủ thông tin để triển khai đầy đủ một tính năng trong ứng dụng của họ và hơn một phần ba đã gặp khó khăn khi bắt đầu với điều gì đó họ chưa làm trước đây.

<img src="survey2020/doc_struggles.svg" alt="Biểu đồ cột về những khó khăn khi sử dụng tài liệu Go chính thức" width="700"/>

Các lĩnh vực vấn đề nhất của tài liệu chính thức là về sử dụng module và phát triển CLI, với 20% người trả lời thấy tài liệu về module hơi hoặc hoàn toàn không hữu ích, và 16% đối với tài liệu về phát triển CLI.

<img src="survey2020/doc_helpfulness.svg" alt="Biểu đồ cột xếp chồng về tính hữu ích của các lĩnh vực tài liệu cụ thể bao gồm sử dụng module, phát triển công cụ CLI, xử lý lỗi, phát triển dịch vụ web, truy cập dữ liệu, concurrency và file input/output, được đánh giá trên thang điểm 5 từ hoàn toàn không đến rất hữu ích" width="700"/>

## Go trong đám mây {#cloud}

Go được thiết kế với tính toán phân tán hiện đại trong đầu,
và chúng tôi muốn tiếp tục cải thiện trải nghiệm developer khi xây dựng
dịch vụ đám mây với Go.

- Ba nhà cung cấp đám mây toàn cầu lớn nhất (Amazon Web Services,
Google Cloud Platform và Microsoft Azure) tiếp tục tăng
về việc sử dụng trong số người trả lời khảo sát,
trong khi hầu hết các nhà cung cấp khác được sử dụng bởi tỷ lệ nhỏ hơn của người trả lời mỗi năm.
Azure đặc biệt có mức tăng đáng kể từ 7% lên 12%.
- Triển khai on-prem lên các máy chủ tự sở hữu hoặc thuộc sở hữu công ty tiếp tục
giảm như là các mục tiêu triển khai phổ biến nhất.

<img src="survey2020/cloud_yoy.svg" alt="Biểu đồ cột về các nhà cung cấp đám mây được sử dụng để triển khai các chương trình Go, trong đó AWS là phổ biến nhất ở mức 44%" width="700"/>

Người trả lời triển khai lên AWS và Azure thấy sự gia tăng trong việc triển khai lên nền tảng Kubernetes được quản lý, hiện ở mức 40% và 54%, tương ứng. Azure chứng kiến sự sụt giảm đáng kể trong tỷ lệ người dùng triển khai chương trình Go lên VMs và một số tăng trưởng trong việc sử dụng container từ 18% lên 25%.
Trong khi đó, GCP (vốn đã có tỷ lệ cao người trả lời báo cáo sử dụng Kubernetes được quản lý) thấy sự tăng trưởng trong việc triển khai lên serverless Cloud Run từ 10% lên 17%.

<img src="survey2020/cloud_services_yoy.svg" alt="Biểu đồ cột về tỷ lệ dịch vụ đang được sử dụng với mỗi nhà cung cấp" width="700"/>

Nhìn chung, đa số người trả lời đều hài lòng với việc sử dụng Go trên cả ba
nhà cung cấp đám mây lớn, và các con số về mặt thống kê không thay đổi so với năm ngoái.
Người trả lời báo cáo mức độ hài lòng tương tự khi phát triển Go cho
AWS (82% hài lòng) và GCP (80%).
Azure nhận được điểm hài lòng thấp hơn (58% hài lòng),
và các câu trả lời văn bản tự do thường đề cập đến nhu cầu cải tiến SDK Go của Azure và hỗ trợ Go cho Azure Functions.

<img src="survey2020/cloud_csat.svg" alt="Biểu đồ cột xếp chồng về sự hài lòng khi sử dụng Go với AWS, GCP và Azure" width="700"/>

## Các điểm đau {#pain}

Những lý do hàng đầu khiến người trả lời không thể sử dụng Go nhiều hơn vẫn là làm việc
trên một dự án bằng ngôn ngữ khác (54%),
làm việc trong một nhóm thích sử dụng ngôn ngữ khác (34%),
và sự thiếu hụt một tính năng quan trọng trong chính Go (26%).

Năm nay chúng tôi đã giới thiệu một tùy chọn mới, "Tôi đã sử dụng Go ở khắp nơi tôi muốn", để người trả lời có thể từ chối lựa chọn những thứ không ngăn họ sử dụng Go. Điều này làm giảm đáng kể tỷ lệ lựa chọn tất cả các tùy chọn khác, nhưng không thay đổi thứ tự tương đối của chúng.
Chúng tôi cũng đã giới thiệu một tùy chọn cho "Go thiếu các framework quan trọng".

Nếu chúng tôi chỉ nhìn vào những người trả lời đã chọn lý do để không sử dụng Go, chúng tôi có thể hiểu rõ hơn về xu hướng theo từng năm. Làm việc trên một dự án hiện có bằng ngôn ngữ khác và sở thích của dự án/nhóm/quản lý đối với ngôn ngữ khác đang giảm dần theo thời gian.

<img src="survey2020/goblockers_yoy_sans_na.svg" alt="Biểu đồ cột về lý do ngăn người trả lời sử dụng Go nhiều hơn" width="700"/>

Trong số 26% người trả lời cho biết Go thiếu các tính năng ngôn ngữ mà họ cần,
88% chọn generics là tính năng quan trọng còn thiếu.
Các tính năng quan trọng còn thiếu khác là xử lý lỗi tốt hơn (58%), null safety (44%), tính năng lập trình hàm (42%) và
hệ thống kiểu mạnh hơn/mở rộng hơn (41%).

Để rõ ràng, những con số này là từ tập hợp con người trả lời cho biết họ
có thể sử dụng Go nhiều hơn nếu nó không thiếu một hoặc nhiều tính năng quan trọng mà họ cần,
không phải từ toàn bộ đối tượng người trả lời khảo sát. Để đặt điều đó trong bối cảnh, 18% người trả lời bị ngăn không thể sử dụng Go vì thiếu generics.

<img src="survey2020/missing_features.svg" alt="Biểu đồ cột về các tính năng quan trọng còn thiếu" width="700"/>

Thách thức hàng đầu mà người trả lời báo cáo khi sử dụng Go vẫn là sự thiếu hụt generics trong Go (18%), trong khi quản lý module/package và các vấn đề về đường cong học tập/thực hành tốt nhất/tài liệu đều ở mức 13%.

<img src="survey2020/biggest_challenge.svg" alt="Biểu đồ cột về những thách thức lớn nhất mà người trả lời đối mặt khi sử dụng Go" width="700"/>

## Cộng đồng Go {#community}

Năm nay chúng tôi đã hỏi người trả lời về 5 tài nguyên hàng đầu của họ để trả lời các câu hỏi liên quan đến Go. Năm ngoái chúng tôi chỉ hỏi về 3 tài nguyên hàng đầu, vì vậy kết quả không thể so sánh trực tiếp, tuy nhiên StackOverflow vẫn là tài nguyên phổ biến nhất ở mức 65%.
Đọc mã nguồn (57%) vẫn là một tài nguyên phổ biến khác trong khi sự phụ thuộc vào godoc.org (39%) đã giảm đáng kể. Trang web khám phá package pkg.go.dev là mới trong danh sách năm nay và là tài nguyên hàng đầu cho 32% người trả lời. Người trả lời sử dụng pkg.go.dev có nhiều khả năng đồng ý rằng họ có thể nhanh chóng tìm thấy các package/thư viện Go họ cần hơn: 91% cho người dùng pkg.go.dev so với 82% cho tất cả những người khác.

<img src="survey2020/resources.svg" alt="Biểu đồ cột về 5 tài nguyên hàng đầu mà người trả lời sử dụng để trả lời các câu hỏi liên quan đến Go" width="700"/>

Qua các năm, tỷ lệ người trả lời
không tham dự bất kỳ sự kiện nào liên quan đến Go đã có xu hướng tăng. Do Covid-19, năm nay chúng tôi đã sửa đổi câu hỏi về các sự kiện Go, và thấy rằng hơn một phần tư người trả lời đã dành nhiều thời gian hơn trong các kênh Go trực tuyến so với các năm trước, và 14% đã tham dự một buổi gặp mặt Go ảo, gấp đôi so với năm ngoái. 64% những người tham dự sự kiện ảo cho biết đây là sự kiện ảo đầu tiên của họ.

<img src="survey2020/events.svg" alt="Biểu đồ cột về sự tham gia của người trả lời trong các kênh và sự kiện trực tuyến" width="700"/>

<p id="TOC_12.1">Chúng tôi thấy 12% người trả lời xác định với một nhóm truyền thống thiểu đại diện (ví dụ:
sắc tộc, bản sắc giới tính, v.v.), giống như năm 2019, và 2% tự xác định là phụ nữ, ít hơn năm 2019 (3%).
Người trả lời xác định với các nhóm thiểu đại diện cho thấy tỷ lệ không đồng ý cao hơn với câu
"Tôi cảm thấy được chào đón trong cộng đồng Go" (10% so với 4%) hơn những người không xác định với nhóm thiểu đại diện. Những câu hỏi này cho phép chúng tôi đo lường sự đa dạng trong cộng đồng và làm nổi bật các cơ hội tiếp cận và phát triển.</p>

<img src="survey2020/underrep.svg" alt="Biểu đồ cột về các nhóm thiểu đại diện" width="700"/>
<img src="survey2020/underrep_groups_women.svg" alt="Biểu đồ cột về những người tự xác định là phụ nữ" width="700"/>
<img src="survey2020/welcome_underrep.svg" alt="Biểu đồ cột về sự chào đón của các nhóm thiểu đại diện" width="700"/>

Chúng tôi đã thêm một câu hỏi bổ sung trong năm nay về việc sử dụng công nghệ hỗ trợ, và thấy rằng 8% người trả lời đang sử dụng một số loại công nghệ hỗ trợ. Công nghệ hỗ trợ được sử dụng phổ biến nhất là cài đặt độ tương phản hoặc màu sắc (2%). Đây là một lời nhắc nhở tuyệt vời rằng chúng tôi có những người dùng với nhu cầu tiếp cận và giúp thúc đẩy một số quyết định thiết kế của chúng tôi trên các trang web được quản lý bởi nhóm Go.

<img src="survey2020/at.svg" alt="Biểu đồ cột về việc sử dụng công nghệ hỗ trợ" width="700"/>

Nhóm Go coi trọng sự đa dạng và hòa nhập, không chỉ đơn giản là vì đó là điều đúng đắn cần làm, mà còn vì những tiếng nói đa dạng có thể soi sáng những điểm mù của chúng tôi và cuối cùng mang lại lợi ích cho tất cả người dùng. Cách chúng tôi hỏi về thông tin nhạy cảm, bao gồm giới tính và các nhóm truyền thống thiểu đại diện, đã thay đổi theo các quy định về quyền riêng tư dữ liệu và chúng tôi hy vọng sẽ làm cho những câu hỏi này, đặc biệt là về sự đa dạng giới tính, trở nên bao gồm hơn trong tương lai.


## Kết luận {#conclusion}

Cảm ơn bạn đã tham gia cùng chúng tôi xem kết quả khảo sát developer năm 2020 của chúng tôi!
Hiểu được kinh nghiệm và thách thức của developer giúp chúng tôi đo lường tiến độ và định hướng tương lai của Go.
Xin cảm ơn một lần nữa đến tất cả những người đã đóng góp cho khảo sát này, chúng tôi không thể làm được điều đó nếu không có bạn. Chúng tôi hy vọng sẽ gặp lại bạn năm sau!
