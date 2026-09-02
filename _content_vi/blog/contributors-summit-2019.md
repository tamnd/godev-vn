---
title: Contributors Summit 2019
date: 2019-08-15
by:
- Carmen Andoh and contributors
tags:
- community
summary: Báo cáo từ Go Contributor Summit tại GopherCon 2019.
template: true
---

## Giới thiệu

Trong ba năm liên tiếp, đội ngũ Go và các contributor đã cùng tập hợp
vào ngày trước GopherCon để thảo luận và lập kế hoạch cho tương lai của dự án Go.
Sự kiện bao gồm việc tự tổ chức thành các nhóm thảo luận nhỏ,
một cuộc thảo luận theo kiểu hội trường về quy trình đề xuất vào buổi sáng,
và các cuộc thảo luận bàn tròn theo nhóm vào buổi chiều
dựa trên những chủ đề do các contributor của chúng tôi lựa chọn.
Chúng tôi đã đề nghị năm contributor viết về trải nghiệm của họ
trong các cuộc thảo luận khác nhau tại hội nghị thượng đỉnh năm nay.

{{image "contributors-summit-2019/group.jpg" 800}}

_(Ảnh của Steve Francia.)_

## Trình biên dịch và Runtime (báo cáo của Lynn Boger)

Go Contributor Summit là một cơ hội tuyệt vời
để gặp gỡ và thảo luận về các chủ đề cũng như ý tưởng với những người khác cũng đóng góp cho Go.

Ngày bắt đầu bằng khoảng thời gian để gặp gỡ mọi người trong phòng.
Có sự kết hợp tốt giữa đội ngũ Go cốt lõi
và những người khác đang tích cực đóng góp cho Go.
Từ đó chúng tôi quyết định những chủ đề nào được quan tâm
và cách chia nhóm lớn thành các nhóm nhỏ hơn.
Lĩnh vực tôi quan tâm là trình biên dịch, vì vậy tôi tham gia nhóm đó
và ở cùng họ trong phần lớn thời gian.

Trong cuộc họp đầu tiên, một danh sách dài các chủ đề đã được đưa ra
và kết quả là nhóm trình biên dịch quyết định tiếp tục gặp nhau trong suốt ngày hôm đó.
Tôi có một vài chủ đề quan tâm mà tôi chia sẻ và nhiều chủ đề do những người khác đề xuất
cũng khiến tôi quan tâm.
Không phải mọi mục trong danh sách đều được thảo luận chi tiết;
sau đây là danh sách của tôi về những chủ đề nhận được nhiều sự quan tâm và thảo luận nhất,
tiếp theo là một số nhận xét ngắn được đưa ra về các chủ đề khác.

**Kích thước binary**.
Có mối lo ngại được nêu ra về kích thước binary,
đặc biệt là việc nó tiếp tục tăng qua mỗi bản phát hành.
Một số lý do khả dĩ đã được xác định như việc tăng cường inlining và các tối ưu hóa khác.
Nhiều khả năng có một nhóm người dùng muốn binary nhỏ,
và một nhóm khác muốn hiệu năng tốt nhất có thể, còn một số người có thể không quan tâm.
Điều này dẫn đến chủ đề TinyGo, và có nhận xét rằng TinyGo không phải là một triển khai đầy đủ của Go
và điều quan trọng là phải giữ cho TinyGo không đi chệch khỏi Go và làm phân tách cộng đồng người dùng.
Cần có thêm nghiên cứu để hiểu nhu cầu của người dùng và các lý do chính xác
góp phần vào kích thước hiện tại.
Nếu có cơ hội giảm kích thước mà không ảnh hưởng đến hiệu năng,
những thay đổi đó có thể được thực hiện, nhưng nếu hiệu năng bị ảnh hưởng
một số người dùng sẽ ưu tiên hiệu năng tốt hơn.

**Vector assembly**.
Cách tận dụng vector assembly trong Go đã được thảo luận trong một khoảng thời gian
và đã là một chủ đề được quan tâm trong quá khứ.
Tôi đã chia vấn đề này thành ba khả năng riêng biệt, vì tất cả đều liên quan đến việc sử dụng các chỉ thị vector,
nhưng cách sử dụng chúng khác nhau, bắt đầu với chủ đề vector assembly.
Đây là một trường hợp khác về sự đánh đổi trong trình biên dịch.

Đối với hầu hết các mục tiêu, có những hàm quan trọng trong các gói chuẩn
như crypto, hash, math và các gói khác, nơi việc sử dụng assembly là cần thiết
để đạt được hiệu năng tốt nhất có thể; tuy nhiên việc có các hàm lớn
được viết bằng assembly khiến chúng khó hỗ trợ và bảo trì
và có thể yêu cầu các triển khai khác nhau cho từng nền tảng mục tiêu.
Một giải pháp là sử dụng macro assembly hoặc các
kỹ thuật sinh mã cấp cao khác để làm cho vector assembly dễ đọc và dễ hiểu hơn.

Một khía cạnh khác của câu hỏi này là liệu trình biên dịch Go có thể
trực tiếp sinh các chỉ thị vector SIMD khi biên dịch một tệp nguồn Go hay không,
bằng cách cải tiến trình biên dịch Go để biến đổi các chuỗi mã nhằm “simdize”
mã và tận dụng các chỉ thị vector.
Việc triển khai SIMD trong trình biên dịch Go sẽ làm tăng độ phức tạp và thời gian biên dịch,
và có thể không phải lúc nào cũng tạo ra mã có hiệu năng tốt hơn.
Cách biến đổi mã trong một số trường hợp có thể phụ thuộc
vào nền tảng mục tiêu, vì vậy điều đó sẽ không lý tưởng.

Một cách khác để tận dụng các chỉ thị vector trong Go là cung cấp một cách
giúp việc sử dụng các chỉ thị vector từ bên trong mã nguồn Go trở nên dễ dàng hơn.
Các chủ đề được thảo luận bao gồm intrinsic, hoặc các triển khai tồn tại trong những trình biên dịch khác như Rust.
Trong gcc, một số nền tảng cung cấp inline asm, và Go có thể cũng cung cấp khả năng này,
nhưng tôi biết từ kinh nghiệm rằng việc trộn inline asm với mã Go làm tăng độ phức tạp
cho trình biên dịch về mặt theo dõi việc sử dụng thanh ghi và gỡ lỗi.
Nó cho phép người dùng thực hiện những việc mà trình biên dịch có thể không dự đoán hoặc không mong muốn,
và nó cũng thêm một mức độ phức tạp khác.
Nó có thể được chèn vào những vị trí không lý tưởng.

Tóm lại, điều quan trọng là cung cấp một cách để tận dụng
các chỉ thị vector hiện có, đồng thời làm cho việc viết chúng dễ dàng và an toàn hơn.
Khi có thể, các hàm nên sử dụng càng nhiều mã Go càng tốt,
và có thể tìm ra cách sử dụng assembly cấp cao.
Đã có một số thảo luận về việc thiết kế một gói vector thử nghiệm
để thử triển khai một số ý tưởng này.

**Quy ước gọi hàm mới**.
Một số người quan tâm đến chủ đề
[những thay đổi ABI để cung cấp quy ước gọi hàm dựa trên thanh ghi](/issue/18597).
Trạng thái hiện tại đã được báo cáo cùng với các chi tiết.
Đã có thảo luận về những việc còn cần thực hiện trước khi có thể sử dụng nó.
Đặc tả ABI cần được viết trước và chưa rõ khi nào việc đó sẽ được thực hiện.
Tôi biết điều này sẽ mang lại lợi ích cho một số nền tảng mục tiêu nhiều hơn các nền tảng khác
và quy ước gọi hàm dựa trên thanh ghi được sử dụng trong hầu hết trình biên dịch cho các nền tảng khác.

**Các tối ưu hóa chung**.
Một số tối ưu hóa có lợi hơn cho một số nền tảng khác ngoài x86 đã được thảo luận.
Đặc biệt, các tối ưu hóa vòng lặp như đưa bất biến ra ngoài vòng lặp và giảm cường độ có thể được thực hiện
và mang lại nhiều lợi ích hơn trên một số nền tảng.
Các giải pháp tiềm năng đã được thảo luận, và việc triển khai có lẽ sẽ
phụ thuộc vào những mục tiêu coi các cải tiến đó là quan trọng.

**Tối ưu hóa theo hướng phản hồi**.
Điều này đã được thảo luận và tranh luận như một cải tiến khả thi trong tương lai.
Theo kinh nghiệm của tôi, rất khó tìm được các chương trình có ý nghĩa để sử dụng cho việc thu thập dữ liệu hiệu năng mà sau đó có thể dùng để tối ưu hóa mã.
Điều này làm tăng thời gian biên dịch và chiếm nhiều không gian để lưu dữ liệu, dữ liệu này có thể chỉ có ý nghĩa đối với một tập hợp nhỏ các chương trình.

**Các submission đang chờ**.
Một vài thành viên trong nhóm đã đề cập đến những thay đổi mà họ đang thực hiện và dự định sẽ sớm gửi, bao gồm các cải tiến cho makeslice và viết lại rulegen.

**Các mối quan tâm về thời gian biên dịch**.
Thời gian biên dịch đã được thảo luận ngắn gọn. Có lưu ý rằng việc đo thời gian từng phase đã được thêm vào đầu ra của GOSSAFUNC.

**Trao đổi với contributor về compiler**.
Có người hỏi liệu có cần một mailing list cho compiler Go hay không.
Có đề xuất rằng chúng ta sử dụng golang-dev cho mục đích đó,
thêm compiler vào dòng chủ đề để nhận diện.
Nếu lưu lượng trên golang-dev quá lớn, thì một mailing list dành riêng cho compiler
có thể được xem xét vào một thời điểm nào đó trong tương lai.

**Cộng đồng**.
Tôi thấy ngày này rất hữu ích về mặt kết nối với những người
đã tích cực tham gia trong cộng đồng và có các lĩnh vực quan tâm tương tự.
Tôi đã có thể gặp nhiều người mà trước đây tôi chỉ biết qua user name
xuất hiện trong các issue hoặc mailing list hoặc CL.
Tôi đã có thể thảo luận một số chủ đề và các issue hiện có
và nhận phản hồi tương tác trực tiếp thay vì chờ phản hồi trực tuyến.
Tôi được khuyến khích viết issue về những vấn đề mà tôi đã thấy.
Những kết nối này không chỉ diễn ra trong ngày hôm đó mà còn trong lúc
gặp gỡ những người khác trong suốt hội nghị,
sau khi đã được giới thiệu vào ngày đầu tiên này, dẫn đến nhiều cuộc thảo luận thú vị.
Hy vọng những kết nối này sẽ dẫn đến việc giao tiếp hiệu quả hơn
và cải thiện việc xử lý issue cũng như thay đổi mã trong tương lai.

## Công cụ (báo cáo của Paul Jolly)

Phiên thảo luận nhóm nhỏ về công cụ trong hội nghị contributor summit có hình thức mở rộng,
với hai phiên bổ sung trong các ngày chính của hội nghị được tổ chức bởi nhóm
[golang-tools](/wiki/golang-tools).
Bản tóm tắt này được chia thành hai phần: phiên công cụ tại hội thảo contributor,
và báo cáo kết hợp từ các phiên golang-tools trong những ngày chính của hội nghị.

**Contributor summit**.
Phiên công cụ bắt đầu bằng phần giới thiệu từ khoảng 25 người tham dự,
sau đó là phần động não về các chủ đề, bao gồm:
gopls, ARM 32-bit, eval, signal, analysis, go/packages api, refactoring, pprof,
trải nghiệm module, phân tích mono repo, go mobile, dependencies, tích hợp trình soạn thảo,
các quyết định tối ưu compiler, debugging, trực quan hóa, tài liệu.
Rất nhiều người với rất nhiều mối quan tâm về rất nhiều công cụ!

Phiên tập trung vào hai lĩnh vực (tất cả thời gian cho phép): gopls và trực quan hóa.
[Gopls](/wiki/gopls) (phát âm: “go please”) là một triển khai của máy chủ
[Language Server Protocol (LSP)](https://langserver.org) cho Go.
Rebecca Stambler, tác giả chính của gopls, cùng phần còn lại của nhóm Go tools quan tâm
đến việc lắng nghe trải nghiệm của mọi người với gopls: tính ổn định, các tính năng còn thiếu, việc tích hợp trong trình soạn thảo có hoạt động hay không, v.v.?
Cảm nhận chung là gopls đang ở trạng thái rất tốt và hoạt động cực kỳ hiệu quả đối với phần lớn trường hợp sử dụng.
Độ bao phủ của integration test cần được cải thiện, nhưng đây là một vấn đề khó để đạt được mức “đúng” trên tất cả trình soạn thảo.
Chúng tôi đã thảo luận về một phương thức tốt hơn để người dùng báo cáo các lỗi gopls mà họ gặp phải thông qua trình soạn thảo của mình,
telemetry/chẩn đoán, các chỉ số hiệu năng của gopls, tất cả các chủ đề này đã được đề cập chi tiết hơn
trong các phiên golang-tools tiếp theo vào những ngày chính của hội nghị (xem bên dưới).
Một lĩnh vực thảo luận quan trọng là cách mở rộng gopls, ví dụ dưới dạng
các kiểm tra bổ sung kiểu vet của go/analysis, kiểm tra lint, refactoring, v.v.
Hiện tại chưa có giải pháp tốt, nhưng vấn đề này đang được tích cực nghiên cứu.
Cuộc trò chuyện chuyển sang chủ đề rất rộng về trực quan hóa, với phần giới thiệu dựa trên bản demo từ Anthony Starks (người cũng đã có một bài nói chuyện xuất sắc về
[Go for information displays](https://www.youtube.com/watch?v=NyDNJnioWhI) tại GopherCon 2018).

**Các ngày hội nghị**.
Các phiên golang-tools trong những ngày chính của hội nghị là sự tiếp nối của các
[monthly calls](/wiki/golang-tools) đã diễn ra kể từ khi nhóm được thành lập tại GopherCon 2018.
Ghi chú đầy đủ có sẵn cho các phiên
[day 1](https://docs.google.com/document/d/1-RVyttQ0ncjCpR_sRwizf-Ubedkr0Emwmk2LhnsUOmE/edit) và
[day 2](https://docs.google.com/document/d/1ZI_WqpLCB8DO6teJ3aBuXTeYD2iZZZlkDptmcY6Ja60/edit#heading=h.x9lkytc2gxmg).
Các phiên này một lần nữa thu hút đông người tham dự với 25-30 người trong mỗi phiên.
Nhóm Go tools có mặt với lực lượng đông đảo (một dấu hiệu tốt cho thấy sự hỗ trợ đang được dành cho lĩnh vực này), cũng như nhóm nền tảng Uber.
Khác với contributor summit, mục tiêu của các phiên này là rời đi với các mục hành động cụ thể.

**Gopls**.
Mức độ “sẵn sàng” của Gopls là trọng tâm chính trong cả hai phiên thảo luận.
Câu trả lời này về cơ bản quy về việc xác định khi nào hợp lý để nói với các trình tích hợp trình soạn thảo rằng “chúng ta đã có bản đầu tiên tốt của gopls” và sau đó lập danh sách các tích hợp plugin/trình soạn thảo “được phê chuẩn” đã biết là hoạt động với gopls.
Trọng tâm của việc “chứng nhận” các tích hợp plugin/trình soạn thảo này là một quy trình được xác định rõ ràng để người dùng có thể báo cáo các vấn đề họ gặp phải với gopls.
Hiệu năng và bộ nhớ không phải là yếu tố cản trở đối với “bản phát hành” ban đầu này.
Cuộc thảo luận về cách mở rộng gopls, bắt đầu từ hội nghị contributor vào ngày trước đó, tiếp tục một cách nghiêm túc.
Mặc dù có nhiều lợi ích và điểm hấp dẫn rõ ràng khi mở rộng gopls
(các kiểm tra go/analysis tùy chỉnh, hỗ trợ linter, tái cấu trúc, tạo mã…),
vẫn chưa có câu trả lời rõ ràng về cách triển khai điều này theo cách có thể mở rộng.
Những người tham dự đồng ý rằng đây không nên được xem là yếu tố cản trở đối với “bản phát hành” ban đầu, nhưng cần tiếp tục được nghiên cứu và phát triển.
Theo tinh thần của gopls và các tích hợp trình soạn thảo,
Heschi Kreinick từ nhóm Go tools đã nêu ra chủ đề về hỗ trợ gỡ lỗi.
Delve đã trở thành trình gỡ lỗi mặc định trên thực tế cho Go và đang ở trạng thái tốt;
giờ đây cần xác lập trạng thái của việc tích hợp trình gỡ lỗi với trình soạn thảo,
theo một quy trình tương tự như gopls và các tích hợp “được phê chuẩn”.

**Go Discovery Site**.
Phiên golang-tools thứ hai bắt đầu bằng phần giới thiệu xuất sắc về Go Discovery Site do Julie Qiu từ nhóm Go tools trình bày, cùng với một bản demo nhanh.
Julie đã nói về các kế hoạch cho Discovery Site: mở mã nguồn dự án,
những tín hiệu nào được sử dụng trong xếp hạng tìm kiếm, cách [godoc.org](http://godoc.org/) cuối cùng sẽ được thay thế,
cách các submodule nên hoạt động, cách người dùng có thể khám phá các phiên bản chính mới.

**Build Tags**.
Sau đó cuộc trò chuyện chuyển sang hỗ trợ build tag trong gopls.
Đây là một lĩnh vực rõ ràng cần được hiểu rõ hơn
(các trường hợp sử dụng hiện đang được thu thập trong [issue 33389](/issue/33389)).
Dựa trên cuộc trò chuyện này, phiên thảo luận kết thúc với
Alexander Zolotov từ nhóm JetBrains GoLand đề xuất rằng các nhóm gopls và
GoLand nên chia sẻ kinh nghiệm trong lĩnh vực này cũng như nhiều lĩnh vực khác, vì GoLand
đã tích lũy được rất nhiều kinh nghiệm.

**Join Us!**
Chúng ta có thể dễ dàng đã thảo luận về các chủ đề liên quan đến hệ thống công cụ trong nhiều ngày!
Tin tốt là các cuộc gọi golang-tools sẽ tiếp tục trong tương lai gần.
Bất kỳ ai quan tâm đến hệ thống công cụ của Go đều được khuyến khích tham gia: [the wiki](/wiki/golang-tools) có thêm chi tiết.

## Sử dụng trong doanh nghiệp (báo cáo của Daniel Theophanes)

Chủ động tìm hiểu nhu cầu của những nhà phát triển ít lên tiếng hơn sẽ là thách thức lớn nhất, đồng thời cũng là thành công lớn nhất, đối với ngôn ngữ Go. Có một bộ phận lớn lập trình viên không tích cực tham gia vào cộng đồng Go.
Một số người là cộng sự kinh doanh, nhân viên tiếp thị hoặc đảm bảo chất lượng, những người cũng tham gia phát triển.
Một số người sẽ đảm nhận vai trò quản lý và đưa ra quyết định về tuyển dụng hoặc công nghệ.
Những người khác chỉ làm công việc của mình rồi trở về với gia đình.
Và cuối cùng, nhiều khi những nhà phát triển này làm việc trong các doanh nghiệp có hợp đồng bảo vệ IP nghiêm ngặt.
Mặc dù phần lớn những nhà phát triển này sẽ không trực tiếp tham gia vào mã nguồn mở hoặc các đề xuất của cộng đồng Go, khả năng sử dụng Go của họ phụ thuộc vào cả hai điều này.

Cộng đồng Go và các đề xuất của Go cần hiểu nhu cầu của những nhà phát triển ít lên tiếng hơn này.
Các đề xuất của Go có thể tác động lớn đến những gì được chấp nhận và sử dụng.
Ví dụ, thư mục vendor và sau đó là proxy Go modules có vai trò cực kỳ quan trọng đối với các doanh nghiệp kiểm soát chặt chẽ mã nguồn và thường có ít cuộc trao đổi trực tiếp hơn với cộng đồng Go.
Việc có những cơ chế này cho phép các tổ chức đó có thể sử dụng Go.
Do đó, chúng ta không chỉ phải chú ý đến những người dùng Go hiện tại,
mà còn phải chú ý đến các nhà phát triển và tổ chức đã cân nhắc sử dụng Go,
nhưng đã chọn không sử dụng nó.
Chúng ta cần hiểu những lý do này.

Tương tự, nếu cộng đồng Go chú ý đến các môi trường “doanh nghiệp”, điều đó sẽ mở ra cơ hội cho nhiều tổ chức khác có thể sử dụng Go.
Bằng cách đảm bảo xác thực active directory hoạt động, những người dùng vốn bị buộc phải sử dụng một hệ sinh thái khác có thể tiếp tục cân nhắc Go.
Bằng cách đảm bảo WSDL hoạt động ngay, một nhóm người dùng có thể chọn Go làm công cụ.
Không ai đề xuất việc thay đổi một cách mù quáng để làm hài lòng những người dùng không sử dụng Go.
Thay vào đó, chúng ta nên nhận thức được tiềm năng chưa được khai thác và những trở ngại chưa được nhận ra trong ngôn ngữ Go và hệ sinh thái.

Mặc dù đã thảo luận về một số khả năng khác nhau để chủ động thu thập thông tin này từ bên ngoài, đây là một vấn đề mà về cơ bản chúng tôi cần sự giúp đỡ của bạn.
Nếu bạn đang ở trong một tổ chức không sử dụng Go dù đã từng cân nhắc,
hãy cho chúng tôi biết tại sao Go không được chọn.
Nếu bạn đang ở trong một tổ chức nơi Go chỉ được sử dụng cho một phần nhỏ các tác vụ lập trình,
nhưng không được sử dụng cho những tác vụ khác, tại sao nó không được sử dụng rộng rãi hơn? Có những trở ngại cụ thể nào đối với việc áp dụng không?

## Giáo dục (báo cáo của Andy Walker)

Một trong những buổi thảo luận bàn tròn mà tôi tham gia tại Contributors Summit năm nay có chủ đề về giáo dục Go, cụ thể là những loại tài nguyên nào chúng ta cung cấp cho lập trình viên Go mới, và cách chúng ta có thể cải thiện chúng. Tham dự có một số nhà tổ chức, kỹ sư và nhà giáo dục rất tâm huyết, mỗi người đều có góc nhìn riêng về chủ đề này, thông qua các công cụ họ đã thiết kế, tài liệu họ đã viết hoặc các hội thảo họ đã tổ chức cho những nhà phát triển với đủ mọi nền tảng.

Ngay từ đầu, cuộc trò chuyện chuyển sang việc liệu Go có phải là một ngôn ngữ lập trình đầu tiên tốt hay không. Tôi không chắc, và đã lên tiếng phản đối ý kiến đó. Tôi lập luận rằng Go không phải là một ngôn ngữ đầu tiên tốt vì nó không được tạo ra với mục đích đó. Như Rob Pike [đã viết vào năm 2012](/talks/2012/splash.article), “ngôn ngữ này được thiết kế bởi và dành cho những người viết—cũng như đọc, gỡ lỗi và bảo trì—các hệ thống phần mềm lớn”. Với tôi, định hướng cốt lõi này rất rõ ràng: Go là một phản hồi có chủ đích đối với những khiếm khuyết được nhận thấy trong các quy trình mà những kỹ sư giàu kinh nghiệm sử dụng, chứ không phải một nỗ lực tạo ra ngôn ngữ lập trình lý tưởng, và do đó giả định rằng người dùng đã có một mức độ quen thuộc cơ bản nhất định với các khái niệm lập trình.

Điều này thể hiện rõ trong tài liệu chính thức tại [golang.org/doc](/doc/). Tài liệu đi thẳng vào cách cài đặt ngôn ngữ trước khi chuyển người dùng đến [tour](/tour/), vốn hướng đến các lập trình viên đã quen thuộc với một ngôn ngữ dạng C. Từ đó, họ được đưa đến [How to Write Go Code](/doc/code.html), nơi cung cấp phần giới thiệu rất cơ bản về workspace Go cổ điển không dùng module, trước khi chuyển ngay sang việc viết thư viện và kiểm thử. Cuối cùng, chúng ta có [Effective Go](/doc/effective_go.html), cùng một loạt tài liệu tham khảo bao gồm [spec](/ref/spec), được bổ sung bằng một số ví dụ. Đây đều là những tài nguyên khá tốt nếu bạn đã quen thuộc với một ngôn ngữ dạng C, nhưng chúng vẫn còn nhiều điểm cần cải thiện, và không có gì dành cho người mới hoàn toàn hoặc thậm chí người chuyển trực tiếp từ một ngôn ngữ như Python.

Là một điểm khởi đầu dễ tiếp cận và có tính tương tác, tour là mục tiêu đầu tiên tự nhiên để giúp ngôn ngữ trở nên thân thiện hơn với người mới, và tôi nghĩ có thể đạt được nhiều tiến bộ chỉ bằng cách tập trung vào đó. Trước tiên, nó nên là liên kết đầu tiên trong tài liệu, nếu không thì cũng nên là liên kết đầu tiên trên thanh điều hướng ở đầu golang.org, được đặt ở vị trí nổi bật. Chúng ta nên khuyến khích người dùng tò mò bắt đầu ngay và thử nghiệm với ngôn ngữ. Chúng ta cũng nên cân nhắc đưa vào các phần giới thiệu tùy chọn dành cho những người đến từ các ngôn ngữ phổ biến khác, cùng với những khác biệt mà họ có thể gặp phải trong Go, kèm theo các bài tập tương tác. Điều này sẽ giúp ích rất nhiều cho các lập trình viên Go mới trong việc ánh xạ những khái niệm họ đã quen thuộc sang Go.

Đối với các lập trình viên có kinh nghiệm, nên có phần trình bày sâu hơn dưới dạng tùy chọn cho hầu hết các phần trong tour, cho phép họ đào sâu vào tài liệu chi tiết hơn hoặc các bài tập tương tác liệt kê những quyết định thiết kế và nguyên tắc kiến trúc tốt trong Go. Họ nên tìm được câu trả lời cho những câu hỏi như:

- Tại sao có nhiều kiểu số nguyên đến vậy khi tôi được khuyến khích dùng `int` trong phần lớn thời gian?
- Có bao giờ có lý do hợp lý để chọn value receiver không?
- Tại sao có `int` thông thường nhưng lại không có `float` thông thường?
- Kênh chỉ gửi và chỉ nhận là gì, và khi nào tôi sẽ dùng chúng?
- Làm thế nào để kết hợp hiệu quả các primitive đồng thời, và khi nào tôi _không_ muốn dùng channel?
- `uint` hữu ích cho việc gì? Tôi có nên dùng nó để giới hạn người dùng của mình chỉ nhập các giá trị dương không? Tại sao không?

Tour nên là nơi mà họ có thể quay lại sau khi hoàn thành lượt tìm hiểu đầu tiên để đào sâu hơn vào một số lựa chọn thú vị hơn trong thiết kế ngôn ngữ.

Nhưng chúng ta có thể làm nhiều hơn. Nhiều người tìm đến lập trình như một cách để thiết kế ứng dụng hoặc giải quyết một nhu cầu cụ thể, và họ thường muốn nhắm đến giao diện mà họ quen thuộc nhất: trình duyệt. Go hiện vẫn chưa có câu chuyện tốt về front-end. JavaScript vẫn là ngôn ngữ duy nhất thực sự cung cấp cả môi trường front-end lẫn back-end, nhưng WASM đang nhanh chóng trở thành một nền tảng cấp một, và có rất nhiều hướng chúng ta có thể phát triển với nó. Chúng ta có thể cung cấp thứ gì đó giống như [vecty](https://github.com/gopherjs/vecty) trong [The Go Play Space](https://goplay.space/), hoặc có thể là [Gio](https://gioui.org/), nhắm đến WASM, để mọi người có thể bắt đầu lập trình trong trình duyệt ngay lập tức, khơi gợi trí tưởng tượng của họ, đồng thời cung cấp cho họ con đường chuyển đổi từ playground của chúng ta sang terminal và GitHub.

Vậy Go có phải là một ngôn ngữ đầu tiên tốt không?
Tôi thực sự không biết, nhưng chắc chắn có một số lượng đáng kể người bước vào nghề lập trình với Go là điểm khởi đầu, và tôi rất quan tâm đến việc trò chuyện với họ, tìm hiểu hành trình và quy trình của họ, cũng như định hình tương lai của giáo dục Go với những đóng góp từ họ.

## Nền tảng học tập (báo cáo của Ronna Steinberg)

Chúng tôi đã thảo luận về hình dáng của một nền tảng học Go nên như thế nào
và cách chúng ta có thể kết hợp các tài nguyên toàn cầu để dạy ngôn ngữ này một cách hiệu quả.
Nhìn chung, chúng tôi đồng ý rằng việc dạy và học sẽ dễ dàng hơn với khả năng trực quan hóa
và REPL mang lại sự hài lòng cao.
Chúng tôi cũng tổng quan một số giải pháp hiện có để trực quan hóa với Go:
templates, Go WASM, GopherJS cũng như việc tạo SVG và GIF.

Vấn đề lỗi của trình biên dịch không có ý nghĩa với lập trình viên mới cũng được nêu ra
và chúng tôi đã xem xét các ý tưởng về cách xử lý vấn đề này, có thể là một ngân hàng lỗi và cách chúng có thể hữu ích.
Một ý tưởng là một trình bao bọc cho trình biên dịch, giúp giải thích các lỗi cho bạn, kèm theo ví dụ và giải pháp.

Một nhóm mới đã họp lại cho vòng thảo luận thứ hai sau đó và chúng tôi tập trung nhiều hơn vào
nền tảng học Go nên có UX như thế nào,
và liệu chúng ta có thể, cũng như bằng cách nào, lấy các tài liệu hiện có (bài nói chuyện, bài viết blog, podcast, v.v.)
từ cộng đồng và tổ chức chúng thành một chương trình mà mọi người có thể học theo.
Một nền tảng như vậy có nên liên kết đến các tài nguyên bên ngoài đó không?
Nhúng chúng?
Trích dẫn chúng?
Chúng tôi đồng ý rằng một giải pháp kiểu cổng thông tin (với các liên kết bên ngoài đến tài nguyên)
khiến việc điều hướng trở nên khó khăn và làm giảm trải nghiệm học tập,
điều này dẫn chúng tôi đến kết luận rằng đóng góp như vậy không thể là thụ động,
và các contributor có khả năng sẽ phải chủ động tham gia để đưa tài liệu của họ lên nền tảng.
Sau đó, có rất nhiều sự hào hứng xung quanh ý tưởng thêm cơ chế bỏ phiếu vào nền tảng,
về cơ bản biến người học thành contributor,
đồng thời khuyến khích các contributor đưa tài liệu của họ lên nền tảng.

(Nếu bạn quan tâm đến việc hỗ trợ các nỗ lực giáo dục cho Go,
vui lòng gửi email cho Carmen Andoh candoh@google.com.)

## Cảm ơn!

Cảm ơn tất cả những người tham dự vì những cuộc thảo luận tuyệt vời trong ngày contributor,
và đặc biệt cảm ơn Lynn, Paul, Daniel, Andy và Ronna
đã dành thời gian viết các báo cáo này.
