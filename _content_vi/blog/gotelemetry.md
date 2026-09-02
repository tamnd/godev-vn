---
title: Telemetry trong Go 1.23 và xa hơn
date: 2024-09-03
by:
- Robert Findley
tags:
- telemetry
- tools
summary: Go 1.23 bao gồm telemetry theo kiểu opt-in cho bộ công cụ Go.
---

<style type="text/css" scoped>
  #blog #content img#prompt {
    max-width: 500px;
  }
  .centered {
    display: flex;
    flex-direction: column;
    align-items: center;
  }
  .chart {
    width: 100%;
  }
  @media (prefers-color-scheme: dark) {
    .chart {
      border-radius: 8px;
    }
  }
  figure.captioned {
    display: table;
  }
  figure.captioned figcaption {
    display: table-caption;
    caption-side: bottom;
    font-style: italic;
    font-size: small;
    text-align: center;
  }
</style>

Go 1.23 cung cấp một cách mới để bạn giúp cải thiện bộ công cụ Go. Bằng cách
bật [tải lên telemetry](/doc/go1.23#telemetry), bạn có thể chọn chia sẻ
dữ liệu về các chương trình trong bộ công cụ và cách sử dụng của chúng với nhóm Go. Dữ liệu này sẽ
giúp các cộng tác viên Go sửa lỗi, tránh hồi quy và đưa ra quyết định tốt hơn.

Theo mặc định, dữ liệu telemetry của Go chỉ được lưu trữ trên máy tính cục bộ của bạn. Nếu bạn
bật tải lên, một tập con [hạn chế](/doc/telemetry#proposals) dữ liệu của bạn
được công bố hàng tuần lên [telemetry.go.dev](https://telemetry.go.dev).

Bắt đầu từ Go 1.23, bạn có thể bật tải lên dữ liệu telemetry cục bộ của mình
bằng lệnh sau:

```
go telemetry on
```

Để tắt ngay cả việc thu thập dữ liệu telemetry cục bộ, chạy lệnh sau:

```
go telemetry off
```

[Tài liệu telemetry](/doc/telemetry) có mô tả chi tiết hơn về
cách triển khai.

## Lịch sử ngắn gọn về telemetry trong Go

Mặc dù telemetry phần mềm không phải là ý tưởng mới, nhóm Go đã trải qua nhiều
lần lặp để tìm kiếm một triển khai telemetry đáp ứng các yêu cầu của Go
về hiệu năng, tính di động và tính minh bạch.

[Thiết kế ban đầu](https://research.swtch.com/telemetry-design) nhằm mục đích không gây
phiền nhiễu, cởi mở và bảo vệ quyền riêng tư đến mức có thể được chấp nhận khi
bật mặc định, nhưng nhiều người dùng đã nêu lo ngại trong một cuộc
[thảo luận công khai](/issue/58409) kéo dài, và thiết kế cuối cùng đã được
[thay đổi](https://research.swtch.com/telemetry-opt-in#campaign) để yêu cầu
sự đồng ý rõ ràng của người dùng khi tải lên từ xa.

Thiết kế mới được [chấp nhận](/issue/58894) vào tháng 4 năm 2023 và được triển khai trong
mùa hè đó.

### Telemetry trong gopls

Phiên bản đầu tiên của telemetry Go được giao trong
[v0.14](https://github.com/golang/tools/releases/tag/gopls%2Fv0.14.0)
của language server Go
[`gopls`](https://go.googlesource.com/tools/+/refs/heads/master/gopls/), vào
tháng 10 năm 2023. Sau khi ra mắt, khoảng 100 người dùng đã bật tải lên,
có thể được thúc đẩy bởi ghi chú phát hành hoặc thảo luận trong kênh
[Gophers Slack](https://gophers.slack.com/messages/gopls/), và dữ liệu
bắt đầu dần dần đến. Không lâu sau, telemetry đã tìm thấy lỗi đầu tiên trong
gopls:

<div class="image">
<div class="centered">
<figure class="captioned">
<img src="gotelemetry/neat.png" alt="Telemetry tìm thấy lỗi đầu tiên" />
<figcaption>
Một stack trace mà Dan nhận thấy trong dữ liệu telemetry đã tải lên của anh ấy đã dẫn đến việc
báo cáo và sửa một lỗi. Đáng chú ý là chúng tôi không biết ai đã báo cáo stack đó.
</figcaption>
</figure>
</div>
</div>


### Nhắc nhở từ IDE

Mặc dù rất vui khi thấy telemetry hoạt động trong thực tế, và chúng tôi đánh giá cao
sự ủng hộ của những người chấp nhận sớm đó, 100 người tham gia không đủ để đo lường
những loại điều chúng tôi muốn đo lường.

Như Russ Cox đã [chỉ ra](https://research.swtch.com/telemetry-opt-in#campaign)
trong các bài đăng blog gốc của anh ấy, một nhược điểm của cách tiếp cận tắt theo mặc định đối với
telemetry là nhu cầu liên tục khuyến khích sự tham gia. Cần có hoạt động tiếp cận
để duy trì một mẫu người dùng đủ lớn để phân tích dữ liệu định lượng có ý nghĩa,
và đại diện cho dân số người dùng. Mặc dù các bài đăng blog và
ghi chú phát hành có thể tăng cường sự tham gia (và chúng tôi sẽ đánh giá cao nếu bạn
bật telemetry sau khi đọc bài này!), chúng dẫn đến một mẫu bị lệch. Ví dụ,
chúng tôi hầu như không nhận được dữ liệu cho `GOOS=windows` từ những người chấp nhận sớm
telemetry trong gopls.

Để tiếp cận nhiều người dùng hơn, chúng tôi đã giới thiệu một [lời nhắc](/doc/telemetry#ide) trong
[plugin VS Code Go](https://marketplace.visualstudio.com/items?itemName=golang.go)
hỏi người dùng có muốn bật telemetry không:

<div class="image">
<div class="centered">
<figure class="captioned">
<img id="prompt" src="gotelemetry/prompt.png" alt="Lời nhắc VS Code" />
<figcaption>
Lời nhắc telemetry, như được hiển thị bởi VS Code.
</figcaption>
</figure>
</div>
</div>

Tính đến thời điểm bài đăng blog này, lời nhắc đã được triển khai cho 5% người dùng VS Code Go, và
mẫu telemetry đã tăng lên khoảng 1800 người tham gia mỗi tuần:

<div class="image">
<div class="centered">
<figure class="captioned">
<img src="gotelemetry/uploads.png" alt="Tải lên hàng tuần so với Tỷ lệ nhắc nhở" class="chart"/>
<figcaption>Nhắc nhở giúp tiếp cận nhiều người dùng hơn.</figcaption>
</figure>
</div>
</div>

(Sự tăng đột biến ban đầu có thể là do nhắc nhở *tất cả* người dùng của extension
[VS Code Go nightly](https://marketplace.visualstudio.com/items?itemName=golang.go-nightly)).

Tuy nhiên, nó đã tạo ra một độ lệch đáng chú ý về phía người dùng VS Code, so với
[kết quả khảo sát Go gần đây nhất](survey2024-h1-results.md):

<div class="image">
<div class="centered">
<figure class="captioned">
<img src="gotelemetry/vscode_skew.png" alt="Lệch về phía người dùng VS Code" class="chart"/>
<figcaption>Chúng tôi nghi ngờ rằng VS Code đang được đại diện quá mức trong dữ liệu telemetry.</figcaption>
</figure>
</div>
</div>

Chúng tôi đang lên kế hoạch giải quyết độ lệch này bằng cách [nhắc nhở tất cả các trình soạn thảo có khả năng LSP sử dụng
gopls](/issue/67821), sử dụng một tính năng của chính language server protocol.

### Chiến thắng của Telemetry

Vì thận trọng, chúng tôi chỉ đề xuất thu thập một vài chỉ số cơ bản cho
lần ra mắt đầu tiên của telemetry trong gopls. Một trong số đó là
[stack counter](/doc/telemetry#stack-counters) [`gopls/bug`](/issue/62249),
ghi lại các điều kiện bất ngờ hoặc "không thể" gặp phải bởi gopls. Thực tế,
đây là một loại khẳng định, nhưng thay vì dừng chương trình, nó
ghi lại trong telemetry rằng nó đã được đến trong một số lần thực thi, cùng với
stack.

Trong quá trình làm việc về [khả năng mở rộng của gopls](gopls-scalability.md), chúng tôi đã thêm nhiều
khẳng định kiểu này, nhưng chúng tôi hiếm khi thấy chúng thất bại trong các bài kiểm thử hoặc trong
cách sử dụng gopls của chính mình. Chúng tôi kỳ vọng rằng hầu hết các khẳng định này là
không thể đạt được.

Khi chúng tôi bắt đầu nhắc nhở người dùng ngẫu nhiên trong VS Code để bật telemetry, chúng tôi thấy
rằng nhiều điều kiện này *đã* được đạt đến trong thực tế, và bối cảnh của
stack trace thường đủ để chúng tôi tái tạo và sửa các lỗi tồn tại lâu dài.
Chúng tôi bắt đầu thu thập các vấn đề này dưới nhãn
[`gopls/telemetry-wins`](https://github.com/golang/go/issues?q=is%3Aissue+label%3Agopls%2Ftelemetry-wins),
để theo dõi các "chiến thắng" được tạo điều kiện bởi telemetry.

Tôi đã bắt đầu nghĩ về "chiến thắng của telemetry" với một ý nghĩa thứ hai: khi so sánh
phát triển gopls có và không có telemetry, *telemetry thắng*.

<div class="image">
<div class="centered">
<figure class="captioned">
<img src="gotelemetry/telemetry_wins.png" alt="Telemetry thắng."/>
<figcaption>Cảm ơn Paul vì những gợi ý.</figcaption>
</figure>
</div>
</div>

Khía cạnh đáng ngạc nhiên nhất của các lỗi đến từ telemetry là có bao nhiêu trong số chúng là
*thực sự có*. Đúng là một số trong số chúng không thể nhìn thấy đối với người dùng, nhưng một số lượng tốt
trong số chúng là các hành vi không đúng thực sự của gopls - những thứ như thiếu tham chiếu chéo,
hoặc tự động hoàn thành không chính xác một cách tinh tế trong một số điều kiện hiếm gặp. Chúng
chính xác là loại thứ mà người dùng có thể cảm thấy khó chịu nhẹ nhưng
có thể sẽ không báo cáo là vấn đề. Có lẽ người dùng sẽ cho rằng
hành vi đó là có chủ ý. Nếu họ có báo cáo vấn đề, họ có thể không chắc
cách tái tạo lỗi, hoặc chúng tôi sẽ cần một cuộc trao đổi dài trên trình theo dõi vấn đề
để nắm bắt một stack trace. Không có telemetry, *không có cách hợp lý nào*
mà hầu hết các lỗi này sẽ được phát hiện, chứ đừng nói đến việc sửa.

Và tất cả điều này chỉ từ một vài bộ đếm. Chúng tôi chỉ đã trang bị stack trace
cho các lỗi tiềm ẩn _mà chúng tôi đã biết về_. Còn những vấn đề chúng tôi không
dự đoán thì sao?

### Báo cáo sự cố tự động

Go 1.23 bao gồm một API mới
[`runtime.SetCrashOutput`](/doc/go1.23#runtimedebugpkgruntimedebug) có thể
được sử dụng để triển khai báo cáo sự cố tự động thông qua một tiến trình watchdog.
Bắt đầu với
[v0.15.0](https://github.com/golang/tools/releases/tag/gopls%2Fv0.15.0), gopls
báo cáo một stack counter `crash/crash` khi nó gặp sự cố, *miễn là gopls
được xây dựng với Go 1.23*.

Khi chúng tôi phát hành gopls@v0.15.0, chỉ một số ít người dùng trong mẫu của chúng tôi đã xây dựng
gopls bằng bản build phát triển chưa phát hành của Go 1.23, nhưng bộ đếm
`crash/crash` mới vẫn tìm thấy
[hai lỗi](https://github.com/golang/tools/releases/tag/gopls%2Fv0.15.2).

## Telemetry trong bộ công cụ Go và xa hơn

Với việc telemetry đã chứng minh hữu ích như thế nào chỉ với một lượng nhỏ
đo đạc và một phần của mẫu mục tiêu, tương lai trông sáng sủa.

Go 1.23 ghi lại telemetry trong bộ công cụ Go, bao gồm lệnh `go`
và các công cụ khác như trình biên dịch, trình liên kết và `go vet`. Chúng tôi đã thêm
telemetry vào `vulncheck` và plugin VS Code Go, và
[chúng tôi đề xuất](/issue/68384) thêm nó vào `delve` nữa.

Loạt blog telemetry gốc phác thảo
[nhiều ý tưởng](https://research.swtch.com/telemetry-uses) về cách telemetry có thể
được sử dụng để cải thiện Go. Chúng tôi mong muốn được khám phá những ý tưởng đó và nhiều hơn nữa.

Trong gopls, chúng tôi dự định sử dụng telemetry để cải thiện độ tin cậy và định hướng
việc ra quyết định và ưu tiên hóa. Với báo cáo sự cố tự động được bật
bởi Go 1.23, chúng tôi kỳ vọng sẽ phát hiện nhiều sự cố hơn trong quá trình kiểm thử trước khi phát hành.
Trong tương lai, chúng tôi sẽ thêm nhiều bộ đếm hơn để đo lường trải nghiệm người dùng - độ trễ của các hoạt động
chính, tần suất sử dụng các tính năng khác nhau - để chúng tôi có thể tập trung nỗ lực
vào nơi chúng sẽ mang lại lợi ích nhiều nhất cho các nhà phát triển Go.

Go tròn 15 tuổi vào tháng 11 này, và cả ngôn ngữ lẫn hệ sinh thái của nó tiếp tục
phát triển. Telemetry sẽ đóng một vai trò quan trọng trong việc giúp các cộng tác viên Go di chuyển
nhanh hơn và an toàn hơn, đúng hướng.
