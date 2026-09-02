---
title: Kết quả Khảo sát Developer Go 2021
date: 2022-04-19
by:
- Alice Merrick
tags:
- survey
- community
summary: Phân tích kết quả từ Khảo sát Developer Go 2021.
---


## Cảm ơn cộng đồng vì lượng phản hồi tuyệt vời!

Năm 2021, chúng tôi đã chạy Khảo sát Developer Go từ ngày 26 tháng 10 đến ngày 16 tháng 11 và nhận được 11.840 phản hồi, lượt tham gia lớn nhất từ trước đến nay trong 6 năm chúng tôi tổ chức khảo sát! Cảm ơn bạn đã dành thời gian cung cấp cho cộng đồng những thông tin về trải nghiệm sử dụng Go của bạn.

## Điểm nổi bật {#highlights}

- Hầu hết các phản hồi đều nhất quán với các năm trước. Ví dụ, [mức độ hài lòng với Go vẫn rất cao ở mức 92%](#satisfaction) và 75% người trả lời sử dụng Go tại nơi làm việc.
- Năm nay chúng tôi đã [lấy mẫu ngẫu nhiên](#changes) một số người tham gia bằng plugin VS Code Go, điều này dẫn đến một số thay đổi trong đối tượng trả lời khảo sát.
- Thiếu thư viện quan trọng, tính năng ngôn ngữ và cơ sở hạ tầng là những [rào cản phổ biến nhất khi sử dụng Go](#adoption). (Lưu ý: khảo sát này được thực hiện trước khi phát hành Go 1.18 với generics, tính năng bị báo cáo thiếu nhiều nhất)
- Người trả lời muốn [ưu tiên cải thiện](#prioritization) việc gỡ lỗi và quản lý dependency.
- [Những thách thức lớn nhất khi sử dụng module](#modules) liên quan đến versioning, sử dụng private repo và quy trình làm việc đa module. (Lưu ý: khảo sát này được thực hiện trước Go 1.18 đã giới thiệu workspace giải quyết nhiều mối quan ngại trong số này).
- 81% người trả lời [tự tin vào định hướng lâu dài của dự án Go](#satisfaction).

## Chúng tôi đã nghe từ ai? {#demographics}

Nhân khẩu học của chúng tôi khá ổn định qua các năm ([Xem kết quả 2020](/blog/survey2020-results)). Nhất quán với các năm trước, Go chủ yếu được sử dụng trong ngành công nghệ. 70% người trả lời là developer phần mềm, một số trong IT hoặc DevOps và 76% người trả lời cho biết họ lập trình bằng Go trong công việc.
<img src="survey2021/industry_yoy.svg" alt="Biểu đồ cột về ngành công nghiệp nơi người trả lời làm việc" width="700"/>
<img src="survey2021/where_yoy.svg" alt="Biểu đồ cột cho thấy Go được sử dụng nhiều hơn tại nơi làm việc so với ngoài giờ làm việc" width="700"/>
<img src="survey2021/app_yoy.svg" alt="Biểu đồ cột về các mục đích sử dụng Go, trong đó dịch vụ API/RPC và ứng dụng CLI là phổ biến nhất" width="700"/>

Một số thống kê nhân khẩu học mới từ năm 2021:
* Hầu hết người trả lời mô tả tổ chức của họ là doanh nghiệp hoặc doanh nghiệp vừa và nhỏ, với khoảng một phần tư mô tả tổ chức của họ là startup. Công ty tư vấn và tổ chức công cộng ít phổ biến hơn nhiều.
* Đại đa số người trả lời làm việc trong các nhóm ít hơn mười người.
* Hơn một nửa (55%) người trả lời sử dụng Go tại nơi làm việc hàng ngày. Người trả lời sử dụng Go ít thường xuyên hơn ngoài giờ làm việc.

<img src="survey2021/orgtype.svg" alt="Biểu đồ cột về loại tổ chức, trong đó doanh nghiệp là phản hồi phổ biến nhất" width="700"/>

<img src="survey2021/teamsize.svg" alt="Biểu đồ cột về quy mô nhóm, trong đó 2 đến 5 người là quy mô phổ biến nhất" width="700"/>

<img src="survey2021/gofreq_comparison.svg" alt="Tần suất sử dụng Go tại nơi làm việc so với ngoài giờ làm việc, trong đó sử dụng Go tại nơi làm việc thường xuyên nhất là hàng ngày và ngoài giờ làm việc ít phổ biến hơn và thường nhất là hàng tuần" width="700"/>

### Bản sắc giới tính {#gender}
Chúng tôi hỏi về bản sắc giới tính trong khảo sát vì nó cho chúng tôi biết ai đang được đại diện trong kết quả và thêm một chiều hướng khác để đo lường tính bao trùm của cộng đồng. Nhóm Go coi trọng sự đa dạng và hòa nhập, không chỉ vì đó là điều đúng đắn cần làm, mà còn vì những tiếng nói đa dạng giúp chúng tôi đưa ra quyết định tốt hơn. Năm nay chúng tôi đã đổi cách đặt câu hỏi về bản sắc giới tính để bao gồm hơn các bản sắc giới tính khác. Tỷ lệ tương tự tự xác định là phụ nữ như các năm trước (2%). Điều này cũng đúng trong [nhóm được lấy mẫu ngẫu nhiên](#changes), cho thấy đây không chỉ là do việc lấy mẫu.
<img src="survey2021/gender.svg" alt="Biểu đồ cột hiển thị bản sắc giới tính của người trả lời, trong đó 92% người trả lời tự xác định là nam" width="700"/>

### Công nghệ hỗ trợ

Năm nay chúng tôi một lần nữa thấy rằng khoảng 8% người trả lời đang sử dụng một số hình thức công nghệ hỗ trợ. Hầu hết các thách thức liên quan đến nhu cầu về chủ đề có độ tương phản cao hơn và kích thước phông chữ lớn hơn trên các trang web liên quan đến Go hoặc trong trình soạn thảo mã của họ; chúng tôi đang lên kế hoạch hành động dựa trên phản hồi về trang web trong năm nay. Những nhu cầu về khả năng tiếp cận này là điều mà tất cả chúng ta nên ghi nhớ khi đóng góp cho hệ sinh thái Go.

## Cái nhìn chi tiết hơn về những thách thức đối với việc áp dụng Go {#adoption}
Năm nay chúng tôi đã sửa đổi các câu hỏi để nhắm vào các trường hợp thực tế mà Go không được áp dụng và lý do tại sao. Trước tiên, chúng tôi hỏi liệu người trả lời có hay không đánh giá việc sử dụng ngôn ngữ khác so với Go trong năm vừa qua. 43% người trả lời cho biết họ đã đánh giá việc chuyển sang Go, từ Go, hoặc áp dụng Go khi không có ngôn ngữ được thiết lập trước đó. 80% trong số các đánh giá này chủ yếu là vì lý do kinh doanh.

<img src="survey2021/evaluated.svg" alt="Biểu đồ hiển thị tỷ lệ người trả lời đánh giá Go so với ngôn ngữ khác trong năm vừa qua" width="700"/>

Chúng tôi mong đợi các trường hợp sử dụng phổ biến nhất cho Go sẽ là các mục đích sử dụng dự định phổ biến nhất cho những người đánh giá Go. Dịch vụ API/RPC là trường hợp sử dụng phổ biến nhất cho đến nay, nhưng đáng ngạc nhiên là xử lý dữ liệu là trường hợp sử dụng dự định phổ biến thứ hai.

<img src="survey2021/intended_app.svg" alt="Biểu đồ hiển thị loại ứng dụng họ cân nhắc sử dụng Go" width="700"/>

Trong số những người trả lời đã đánh giá Go, 75% cuối cùng đã sử dụng Go. (Tất nhiên, vì hầu hết tất cả người trả lời khảo sát đều báo cáo sử dụng Go, chúng tôi có thể không nghe từ các developer đã đánh giá Go và quyết định không sử dụng nó.)

<img src="survey2021/adopted.svg" alt="Biểu đồ hiển thị tỷ lệ đã sử dụng Go so với những người ở lại với ngôn ngữ hiện tại hoặc chọn ngôn ngữ khác" width="700"/>

Đối với những người đã đánh giá Go nhưng không sử dụng nó, chúng tôi đã hỏi những thách thức nào đã ngăn họ sử dụng Go và thách thức nào là rào cản chính.
<img src="survey2021/blockers.svg" alt="Biểu đồ hiển thị các rào cản khi sử dụng Go" width="700"/>

Bức tranh chúng tôi nhận được từ những kết quả này ủng hộ các phát hiện trước đó rằng các tính năng còn thiếu và thiếu hỗ trợ hệ sinh thái/thư viện là những rào cản kỹ thuật đáng kể nhất đối với việc áp dụng Go.

Chúng tôi đã hỏi thêm chi tiết về các tính năng hoặc thư viện mà người trả lời đang thiếu và thấy rằng generics là tính năng quan trọng còn thiếu phổ biến nhất, chúng tôi mong đợi đây sẽ là một rào cản ít đáng kể hơn sau khi giới thiệu generics trong Go 1.18. Các tính năng còn thiếu phổ biến tiếp theo liên quan đến hệ thống kiểu của Go. Chúng tôi muốn xem cách giới thiệu generics có thể ảnh hưởng hoặc giải quyết các nhu cầu cơ bản xung quanh hệ thống kiểu của Go trước khi thực hiện các thay đổi bổ sung. Hiện tại, chúng tôi sẽ thu thập thêm thông tin về bối cảnh của những nhu cầu này và có thể trong tương lai khám phá các cách khác nhau để đáp ứng những nhu cầu đó thông qua hệ thống công cụ, thư viện hoặc thay đổi hệ thống kiểu.

Đối với các thư viện còn thiếu, không có sự đồng thuận rõ ràng về việc bổ sung nào sẽ giúp tỷ lệ lớn nhất những người muốn áp dụng Go có thể làm điều đó. Điều đó sẽ cần khám phá thêm.

Vậy người trả lời đã sử dụng gì khi họ không chọn Go?

<img src="survey2021/lang_instead.svg" alt="Biểu đồ về ngôn ngữ nào mà người trả lời đã sử dụng thay vì Go" width="700"/>

Rust, Python và Java là những lựa chọn phổ biến nhất. [Rust và Go có các tập tính năng bổ sung cho nhau](https://thenewstack.io/rust-vs-go-why-theyre-better-together/), vì vậy Rust có thể là lựa chọn tốt khi Go không đáp ứng nhu cầu tính năng cho một dự án. Lý do chính để sử dụng Python là thiếu thư viện và hỗ trợ cơ sở hạ tầng hiện có, vì vậy hệ sinh thái package lớn của Python có thể làm cho việc chuyển sang Go trở nên khó khăn. Tương tự, lý do phổ biến nhất để sử dụng Java thay thế là vì các tính năng còn thiếu của Go, điều có thể được giảm nhẹ bởi sự ra đời của generics trong bản phát hành 1.18.

## Sự hài lòng và ưu tiên với Go {#satisfaction}
Hãy xem xét những lĩnh vực Go đang làm tốt và nơi có thể cải thiện.

Nhất quán với năm ngoái, 92% người trả lời cho biết họ rất hài lòng hoặc hơi hài lòng khi sử dụng Go trong năm vừa qua.

<img src="survey2021/csat.svg" alt="Sự hài lòng tổng thể trên thang điểm 5 từ rất không hài lòng đến rất hài lòng" width="700"/>

Xu hướng theo năm về thái độ cộng đồng đã có sự biến động nhỏ. Những người sử dụng Go dưới 3 tháng có xu hướng ít có khả năng đồng ý với những câu này hơn. Người trả lời ngày càng thấy Go quan trọng đối với sự thành công của công ty họ.

<img src="survey2021/attitudes_yoy.svg" alt="Thái độ xung quanh việc sử dụng Go tại nơi làm việc" width="700"/>
<img src="survey2021/attitudes_community_yoy.svg" alt="Thái độ cộng đồng xung quanh sự chào đón và sự tự tin vào định hướng của dự án Go" width="700"/>

### Ưu tiên {#prioritization}
Trong vài năm gần đây, chúng tôi đã đề nghị người trả lời đánh giá các lĩnh vực cụ thể về mức độ hài lòng của họ và tầm quan trọng của các lĩnh vực đó đối với họ; chúng tôi sử dụng thông tin này để xác định các lĩnh vực quan trọng đối với người trả lời nhưng chưa được hài lòng. Tuy nhiên, hầu hết các lĩnh vực này chỉ cho thấy sự khác biệt nhỏ về cả tầm quan trọng và sự hài lòng.

<img src="survey2021/imp_vs_sat2.svg" alt="Biểu đồ phân tán về tầm quan trọng so với sự hài lòng cho thấy hầu hết các lĩnh vực có sự hài lòng cao và kích thước file nhị phân ít quan trọng hơn các lĩnh vực khác" width="700"/>

Năm nay chúng tôi đã giới thiệu một câu hỏi mới để khám phá các cách thay thế để ưu tiên công việc trên các lĩnh vực cụ thể. "Giả sử bạn có 10 GopherCoin để chi cho việc cải thiện các khía cạnh sau khi làm việc với Go. Bạn sẽ phân bổ các đồng xu như thế nào?" Hai lĩnh vực nổi bật là nhận được nhiều GopherCoin hơn đáng kể là quản lý dependency (sử dụng module) và chẩn đoán lỗi, những lĩnh vực mà chúng tôi sẽ dành nguồn lực trong năm 2022.

<img src="survey2021/improvements_sums.svg" alt="Tổng số đồng xu được chi cho mỗi lĩnh vực cải thiện" width="700"/>

### Những thách thức khi làm việc với module {#modules}
Thách thức liên quan đến module phổ biến nhất là làm việc trên nhiều module (19% người trả lời), tiếp theo là các ý kiến về versioning (bao gồm sự e ngại về việc cam kết với v1 API ổn định). Liên quan đến versioning, 9% câu trả lời thảo luận về quản lý phiên bản hoặc cập nhật dependency. Năm vị trí hàng đầu được hoàn thành bởi những thách thức xung quanh private repo (bao gồm xác thực với GitLab nói riêng) và ghi nhớ các lệnh `go mod` khác nhau cùng với việc hiểu các thông báo lỗi của chúng.

## Học Go {#learning}

Năm nay chúng tôi đã áp dụng một cấu trúc mới để khám phá năng suất tương đối trong các cấp độ kinh nghiệm khác nhau với Go. Đại đa số người trả lời (88%) đồng ý rằng họ thường xuyên đạt được mức năng suất cao và 85% đồng ý rằng họ thường có thể đạt được trạng thái flow khi viết bằng Go. Tỷ lệ đồng ý tăng theo kinh nghiệm với Go.

<img src="survey2021/productivity.svg" alt="Biểu đồ hiển thị tỷ lệ người trả lời đồng ý rằng họ cảm thấy có năng suất khi sử dụng Go và có thể đạt trạng thái flow khi viết Go" width="700"/>

### Chúng ta nên đầu tư vào tài liệu thực hành tốt nhất ở lĩnh vực nào?

Một nửa người trả lời muốn có hướng dẫn thêm về các thực hành tốt nhất về tối ưu hóa hiệu suất và cấu trúc thư mục dự án. Không ngạc nhiên, Gopher mới (sử dụng Go dưới 1 năm) cần hướng dẫn nhiều hơn so với Gopher có kinh nghiệm hơn, mặc dù các lĩnh vực hàng đầu nhất quán giữa cả hai nhóm. Đáng chú ý là Gopher mới yêu cầu hướng dẫn thêm về concurrency hơn so với Gopher có kinh nghiệm hơn.

<img src="survey2021/best_practices.svg" alt="Biểu đồ hiển thị các lĩnh vực mà người trả lời muốn có thêm hướng dẫn về thực hành tốt nhất" width="700"/>

### Developer học ngôn ngữ mới như thế nào?
Khoảng một nửa người trả lời học ngôn ngữ mới tại nơi làm việc, nhưng gần như nhiều người (45%) học ngoài trường học hoặc công việc. Người trả lời thường xuyên nhất (90%) báo cáo học một mình. Trong số những người cho biết họ học tại nơi làm việc, nơi có thể có cơ hội học theo nhóm, 84% học một mình thay vì theo nhóm.

<img src="survey2021/learn_where.svg" alt="Biểu đồ cho thấy một nửa người trả lời học ngôn ngữ mới tại nơi làm việc trong khi 45% học ngôn ngữ mới ngoài trường học hoặc công việc" width="700"/>
<img src="survey2021/learn_with.svg" alt="Biểu đồ cho thấy 90% người trả lời học ngôn ngữ lập trình mới cuối cùng của họ một mình" width="700"/>

Nhiều tài nguyên hàng đầu nhấn mạnh tầm quan trọng của tài liệu tốt, nhưng hướng dẫn trực tiếp nổi bật là một tài nguyên đặc biệt hữu ích để học ngôn ngữ.

<img src="survey2021/learning_resources.svg" alt="Biểu đồ cho thấy tài nguyên nào hữu ích nhất để học ngôn ngữ lập trình mới, trong đó đọc tài liệu tham khảo và hướng dẫn viết là hữu ích nhất" width="700"/>

## Công cụ và thực hành developer {#devtools}
Giống như các năm trước, đại đa số người trả lời khảo sát cho biết họ làm việc
với Go trên các hệ thống Linux (63%) và macOS (55%). Tỷ lệ người trả lời chủ yếu phát triển trên Linux có vẻ đang giảm nhẹ theo thời gian.

<img src="survey2021/os_yoy.svg" alt="Hệ điều hành chính từ năm 2019 đến 2021" width="700"/>

### Các nền tảng mục tiêu
Hơn 90% người trả lời hướng đến Linux! Mặc dù nhiều người trả lời phát triển trên macOS hơn Windows, nhưng họ thường xuyên triển khai lên Windows hơn macOS.

<img src="survey2021/os_deploy.svg" alt="Biểu đồ hiển thị các nền tảng mà người trả lời triển khai mã Go của họ" width="700"/>

### Fuzzing
Hầu hết người trả lời không quen với fuzzing hoặc vẫn tự coi mình là người mới về fuzzing. Dựa trên phát hiện này, chúng tôi dự định 1) đảm bảo tài liệu fuzzing của Go giải thích các khái niệm fuzzing ngoài các chi tiết cụ thể về fuzzing trong Go, và 2) thiết kế đầu ra và thông báo lỗi có thể hành động, để giúp các developer mới học fuzzing áp dụng nó thành công.

<img src="survey2021/fuzz.svg" alt="Biểu đồ hiển thị tỷ lệ người trả lời đã sử dụng fuzzing" width="700"/>

## Điện toán đám mây {#cloud}

Go được thiết kế với tính toán phân tán hiện đại trong đầu, và chúng tôi muốn tiếp tục cải thiện trải nghiệm developer khi xây dựng dịch vụ đám mây với Go. Tỷ lệ người trả lời triển khai chương trình Go lên ba nhà cung cấp đám mây toàn cầu lớn nhất (Amazon Web Services, Google Cloud Platform và Microsoft Azure) gần như giữ nguyên trong năm nay và các triển khai on-prem lên các máy chủ tự sở hữu hoặc thuộc sở hữu công ty tiếp tục giảm.

<img src="survey2021/cloud_yoy.svg" alt="Biểu đồ cột về các nhà cung cấp đám mây được sử dụng để triển khai các chương trình Go, trong đó AWS là phổ biến nhất ở mức 44%" width="700"/>

Người trả lời triển khai lên AWS thấy sự gia tăng trong việc triển khai lên nền tảng Kubernetes được quản lý, hiện ở mức 35% trong số những người triển khai lên bất kỳ trong ba nhà cung cấp đám mây lớn nhất. Tất cả các nhà cung cấp đám mây này đều chứng kiến sự giảm trong tỷ lệ người dùng triển khai chương trình Go lên VMs.


<img src="survey2021/cloud_services_yoy.svg" alt="Biểu đồ cột về tỷ lệ dịch vụ đang được sử dụng với mỗi nhà cung cấp" width="700"/>

## Những thay đổi trong năm nay {#changes}

Năm ngoái chúng tôi đã giới thiệu [thiết kế khảo sát theo module](/blog/survey2020-results) để chúng tôi có thể hỏi nhiều câu hỏi hơn mà không kéo dài khảo sát. Chúng tôi tiếp tục thiết kế theo module trong năm nay, mặc dù một số câu hỏi đã bị ngừng và các câu hỏi khác được thêm vào hoặc sửa đổi. Không có người trả lời nào nhìn thấy tất cả các câu hỏi trong khảo sát. Ngoài ra, một số câu hỏi có thể có cỡ mẫu nhỏ hơn nhiều vì chúng được hỏi có chọn lọc dựa trên câu hỏi trước đó.

Thay đổi đáng kể nhất đối với khảo sát trong năm nay là cách chúng tôi tuyển dụng người tham gia. Trong những năm trước, chúng tôi đã công bố khảo sát qua Blog Go, nơi nó được chia sẻ trên các kênh mạng xã hội khác nhau như Twitter, Reddit hoặc Hacker News. Năm nay, ngoài các kênh truyền thống, chúng tôi đã sử dụng plugin VS Code Go để lựa chọn ngẫu nhiên người dùng để hiển thị lời nhắc hỏi xem họ có muốn tham gia khảo sát không. Điều này tạo ra một mẫu ngẫu nhiên mà chúng tôi đã sử dụng để so sánh với những người trả lời tự chọn từ các kênh truyền thống và giúp xác định các tác động tiềm năng của [sự thiên lệch tự chọn](https://en.wikipedia.org/wiki/Self-selection_bias).

<img src="survey2021/rsamp.svg" alt="Tỷ lệ người trả lời từ mỗi nguồn" width="700"/>

Gần một phần ba người trả lời của chúng tôi được tuyển dụng theo cách này nên các câu trả lời của họ có tiềm năng ảnh hưởng đáng kể đến các câu trả lời chúng tôi thấy trong năm nay. Một số khác biệt chính mà chúng tôi thấy giữa hai nhóm này là:

### Nhiều Gopher mới hơn
Mẫu được chọn ngẫu nhiên có tỷ lệ Gopher mới cao hơn (những người sử dụng Go dưới một năm). Có thể là Gopher mới ít kết nối với hệ sinh thái Go hoặc các kênh mạng xã hội hơn, vì vậy họ có nhiều khả năng thấy khảo sát được quảng cáo trong IDE của họ hơn là tìm thấy nó thông qua các phương tiện khác. Bất kể lý do là gì, thật tuyệt vời khi nghe từ một phần rộng hơn của cộng đồng Go.

<img src="survey2021/goex_s.svg" alt="So sánh tỷ lệ người trả lời có từng cấp độ kinh nghiệm cho các nhóm được chọn ngẫu nhiên so với tự chọn" width="700"/>

### Nhiều người dùng VS Code hơn
Không có gì ngạc nhiên khi 91% người trả lời đến với khảo sát từ plugin VS Code thích sử dụng VS Code khi sử dụng Go. Do đó, chúng tôi thấy sở thích trình soạn thảo cao hơn nhiều cho VS Code trong năm nay. Khi chúng tôi loại trừ mẫu ngẫu nhiên, kết quả không khác biệt đáng kể về mặt thống kê so với năm ngoái, vì vậy chúng tôi biết đây là kết quả của sự thay đổi trong mẫu của chúng tôi chứ không phải sở thích tổng thể. Tương tự, người dùng VS Code cũng có nhiều khả năng phát triển trên Windows hơn các người trả lời khác, vì vậy chúng tôi thấy sự tăng nhẹ trong sở thích cho Windows trong năm nay. Chúng tôi cũng thấy sự thay đổi nhỏ trong việc sử dụng một số kỹ thuật developer phổ biến với việc sử dụng trình soạn thảo VS Code.

<img src="survey2021/editor_s.svg" alt="Biểu đồ cột được nhóm về trình soạn thảo nào mà người trả lời thích từ mỗi nhóm mẫu" width="700"/>

<img src="survey2021/os_s.svg" alt="Biểu đồ cột được nhóm về hệ điều hành chính mà người trả lời sử dụng để phát triển Go" width="700"/>

<img src="survey2021/devtech_s.svg" alt="Biểu đồ cột được nhóm hiển thị kỹ thuật nào mà người trả lời sử dụng khi viết Go" width="700"/>

### Tài nguyên khác nhau
Mẫu được chọn ngẫu nhiên ít có khả năng đánh giá các kênh mạng xã hội như Blog Go là trong số các tài nguyên hàng đầu của họ để trả lời các câu hỏi liên quan đến Go, vì vậy họ có thể ít có khả năng thấy khảo sát được quảng cáo trên các kênh đó hơn.

 <img src="survey2021/resources_s.svg" alt="Biểu đồ cột được nhóm hiển thị các tài nguyên hàng đầu mà người trả lời sử dụng khi viết Go" width="700"/>

## Kết luận {#conclusion}

Cảm ơn bạn đã tham gia cùng chúng tôi xem kết quả khảo sát developer năm 2021! Để tóm tắt, một số kết luận chính:

* Hầu hết các chỉ số theo năm của chúng tôi vẫn ổn định với hầu hết các thay đổi do sự thay đổi trong mẫu của chúng tôi.
* Sự hài lòng với Go vẫn cao!
* Ba phần tư người trả lời sử dụng Go tại nơi làm việc và nhiều người sử dụng Go hàng ngày, vì vậy giúp bạn hoàn thành công việc là ưu tiên hàng đầu.
* Chúng tôi sẽ ưu tiên cải thiện việc gỡ lỗi và quy trình làm việc quản lý dependency.
* Chúng tôi sẽ tiếp tục nỗ lực để làm cho Go trở thành một cộng đồng bao trùm cho tất cả các loại Gopher.

Hiểu được kinh nghiệm và thách thức của developer giúp chúng tôi đo lường tiến độ và định hướng tương lai của Go. Xin cảm ơn một lần nữa đến tất cả những người đã đóng góp cho khảo sát này, chúng tôi không thể làm được điều đó nếu không có bạn. Chúng tôi hy vọng sẽ gặp lại bạn năm sau!
