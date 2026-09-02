---
title: Mở rộng quy mô gopls cho hệ sinh thái Go ngày càng lớn
date: 2023-09-08
by:
- Robert Findley
- Alan Donovan
summary: Khi hệ sinh thái Go ngày càng lớn hơn, gopls phải trở nên nhỏ hơn
---

<style type="text/css" scoped>
  .chart {
    width: 100%;
  }
  @media (prefers-color-scheme: dark) {
    .chart {
      border-radius: 8px;
    }
  }
</style>

Đầu mùa hè này, nhóm Go đã phát hành phiên bản [v0.12](/s/gopls-v0.12)
của [gopls](https://pkg.go.dev/golang.org/x/tools/gopls),
[language server](https://microsoft.github.io/language-server-protocol/) cho Go, với tính năng viết lại lõi cho phép
nó mở rộng đến các codebase lớn hơn.
Đây là kết quả của một nỗ lực kéo dài một năm,
và chúng tôi rất hào hứng chia sẻ tiến độ của mình, cũng như nói một chút về
kiến trúc mới và ý nghĩa của nó với tương lai của gopls.

Kể từ bản phát hành v0.12, chúng tôi đã tinh chỉnh thiết kế mới,
tập trung vào việc làm cho các truy vấn tương tác (như tự động hoàn thành hoặc tìm
tham chiếu) nhanh như với v0.11,
dù giữ ít trạng thái hơn nhiều trong bộ nhớ.
Nếu bạn chưa thử, chúng tôi hy vọng bạn sẽ thử:

```
$ go install golang.org/x/tools/gopls@latest
```

Chúng tôi rất muốn nghe về kinh nghiệm của bạn qua [khảo sát ngắn này](https://google.qualtrics.com/jfe/form/SV_4SnGxpcSKN33WZw?s=blog).

## Giảm mức sử dụng bộ nhớ và thời gian khởi động {#results}

Trước khi đi vào chi tiết, hãy xem kết quả!
Biểu đồ bên dưới cho thấy sự thay đổi về thời gian khởi động và mức sử dụng bộ nhớ cho 28
kho lưu trữ Go phổ biến nhất trên GitHub.
Các phép đo này được thực hiện sau khi mở một file Go được chọn ngẫu nhiên
và chờ gopls tải đầy đủ trạng thái của nó,
và vì chúng tôi giả định rằng quá trình lập chỉ mục ban đầu được phân bổ trên nhiều phiên chỉnh sửa,
chúng tôi thực hiện các phép đo này lần _thứ hai_ chúng tôi mở file.

<div class="image">
<img src="gopls-scalability/performance-improvements.svg" alt="Tiết kiệm tương đối
trong bộ nhớ và thời gian khởi động" class="chart"/>
</div>

Trên các repo này, tiết kiệm trung bình khoảng 75%,
nhưng giảm bộ nhớ là phi tuyến:
khi dự án ngày càng lớn hơn, mức giảm tương đối trong mức sử dụng bộ nhớ cũng tăng.
Chúng tôi sẽ giải thích điều này chi tiết hơn bên dưới.

## Gopls và hệ sinh thái Go đang phát triển {#background}

Gopls cung cấp cho các trình soạn thảo không phụ thuộc ngôn ngữ các tính năng giống IDE như tự động hoàn thành,
định dạng, tham chiếu chéo và tái cấu trúc.
Từ khi bắt đầu vào năm 2018, gopls đã hợp nhất nhiều công cụ dòng lệnh khác nhau
như [guru](https://pkg.go.dev/golang.org/x/tools/cmd/guru),
[gorename](https://pkg.go.dev/golang.org/x/tools/cmd/gorename),
và [goimports](https://pkg.go.dev/golang.org/x/tools/cmd/goimports) và
đã trở thành [backend mặc định cho extension VS Code Go](/blog/gopls-vscode-go)
cũng như nhiều trình soạn thảo và plugin LSP khác.
Có thể bạn đã dùng gopls qua trình soạn thảo của mình mà không biết - đó là mục tiêu!

Năm năm trước, gopls cung cấp hiệu năng được cải thiện chỉ bằng cách duy trì một phiên có trạng thái.
Trong khi các công cụ dòng lệnh cũ hơn phải bắt đầu từ đầu mỗi khi thực thi,
gopls có thể lưu kết quả trung gian để giảm đáng kể độ trễ.
Nhưng tất cả trạng thái đó đi kèm với chi phí, và theo thời gian chúng tôi ngày càng [nghe từ người dùng](https://github.com/golang/go/issues?q=is%3Aissue+is%3Aclosed+in%3Atitle+gopls+memory)
rằng mức sử dụng bộ nhớ cao của gopls hầu như không thể chịu đựng được.

Trong khi đó, hệ sinh thái Go đang phát triển, với nhiều code được viết trong
các kho lưu trữ lớn hơn.
[Go workspaces](/blog/get-familiar-with-workspaces) cho phép
các nhà phát triển làm việc trên nhiều module cùng một lúc,
và [phát triển container hóa](https://code.visualstudio.com/docs/devcontainers/containers)
đặt các language server vào môi trường ngày càng bị hạn chế tài nguyên.
Codebase ngày càng lớn hơn, và môi trường phát triển ngày càng nhỏ hơn.
Chúng tôi cần thay đổi cách gopls mở rộng để theo kịp.

## Xem lại nguồn gốc trình biên dịch của gopls {#origins}

Theo nhiều cách, gopls giống trình biên dịch:
nó phải đọc, phân tích, kiểm tra kiểu và phân tích các file nguồn Go,
mà nó sử dụng nhiều [khối xây dựng](https://github.com/golang/example/tree/master/gotypes#introduction) trình biên dịch
được cung cấp bởi [thư viện chuẩn Go](https://pkg.go.dev/go) và module [golang.org/x/tools](https://pkg.go.dev/golang.org/x/tools).
Các khối xây dựng này sử dụng kỹ thuật "lập trình ký hiệu":
trong trình biên dịch đang chạy có một đối tượng hoặc "ký hiệu" duy nhất đại diện cho
mỗi hàm như `fmt.Println`.
Bất kỳ tham chiếu nào đến một hàm được biểu diễn như một con trỏ đến ký hiệu của nó.
Để kiểm tra xem hai tham chiếu có nói về cùng một ký hiệu không,
bạn không cần suy nghĩ về tên.
Bạn chỉ cần so sánh con trỏ. Con trỏ nhỏ hơn nhiều so với chuỗi,
và so sánh con trỏ rất rẻ, vì vậy các ký hiệu là cách hiệu quả để
biểu diễn một cấu trúc phức tạp như một chương trình.

Để phản hồi nhanh các yêu cầu, gopls v0.11 giữ tất cả các ký hiệu này trong bộ nhớ,
như thể gopls đang **biên dịch toàn bộ chương trình cùng một lúc**.
Kết quả là dấu ấn bộ nhớ tỷ lệ và lớn hơn nhiều
so với mã nguồn đang được chỉnh sửa (ví dụ,
các cây cú pháp được gõ thường lớn hơn 30 lần so với văn bản nguồn!).
<!-- deps(gopls) = 18.5MB source, 542MB RAM -->

## Biên dịch riêng biệt {#separate-compilation}

Các nhà thiết kế của trình biên dịch đầu tiên vào những năm 1950 nhanh chóng phát hiện ra
giới hạn của biên dịch nguyên khối.
Giải pháp của họ là chia chương trình thành các đơn vị và biên dịch từng đơn vị riêng biệt.
Biên dịch riêng biệt làm cho có thể xây dựng một chương trình không vừa trong bộ nhớ,
bằng cách thực hiện nó theo từng mảnh nhỏ.
Trong Go, các đơn vị là gói. Biên dịch các gói khác nhau không thể
hoàn toàn riêng biệt:
khi biên dịch gói P, trình biên dịch vẫn cần thông tin về những gì
được cung cấp bởi các gói mà P import.
Để sắp xếp điều này, hệ thống build Go biên dịch tất cả các gói được P import trước P,
và trình biên dịch Go viết một bản tóm tắt gọn của API được xuất của mỗi gói.
Các bản tóm tắt của các gói được P import được cung cấp làm đầu vào cho việc biên dịch P.

Gopls v0.12 mang biên dịch riêng biệt đến gopls,
tái sử dụng cùng định dạng tóm tắt gói được trình biên dịch sử dụng.
Ý tưởng đơn giản, nhưng có sự tinh tế trong chi tiết.
Chúng tôi đã viết lại mỗi thuật toán trước đây kiểm tra cấu trúc dữ liệu đại diện cho toàn bộ chương trình,
để nó giờ hoạt động trên một gói tại một thời điểm và lưu kết quả từng gói vào file,
giống như trình biên dịch phát ra object code.
Ví dụ, tìm tất cả các tham chiếu đến một hàm từng dễ như
tìm kiếm cấu trúc dữ liệu chương trình cho tất cả các lần xuất hiện của một giá trị con trỏ cụ thể.
Giờ, khi gopls xử lý từng gói, nó phải xây dựng và lưu một chỉ mục
liên kết mỗi vị trí định danh trong mã nguồn với tên
của ký hiệu mà nó tham chiếu đến.
Tại thời điểm truy vấn, gopls tải và tìm kiếm các chỉ mục này.
Các truy vấn toàn cục khác, chẳng hạn như "tìm các triển khai",
sử dụng các kỹ thuật tương tự.

Giống như lệnh `go build`, gopls giờ sử dụng một [bộ đệm dựa trên file](https://cs.opensource.google/go/x/tools/+/master:gopls/internal/lsp/filecache/filecache.go;l=5;drc=6f567c8090cb88f13a71b19595bf88c6b27dbeed)
để ghi lại các bản tóm tắt thông tin được tính toán từ mỗi gói,
bao gồm kiểu của mỗi khai báo, chỉ mục các tham chiếu chéo,
và tập phương thức của mỗi kiểu.
Vì bộ đệm được lưu giữ qua các tiến trình,
bạn sẽ nhận thấy rằng lần thứ hai bạn khởi động gopls trong workspace của mình,
nó sẵn sàng phục vụ nhanh hơn nhiều,
và nếu bạn chạy hai phiên bản gopls, chúng hoạt động cùng nhau hiệp lực.

<div class="image">
<img src="gopls-scalability/separate-compilation.png" alt="biên dịch riêng biệt" class="chart"/>
</div>

Kết quả của thay đổi này là mức sử dụng bộ nhớ của gopls tỷ lệ với
số lượng gói đang mở và các import trực tiếp của chúng.
Đây là lý do tại sao chúng tôi quan sát mở rộng phi tuyến trong biểu đồ trên:
khi các kho lưu trữ ngày càng lớn hơn, phần của dự án được quan sát bởi bất kỳ
gói đang mở nào ngày càng nhỏ hơn.

## Vô hiệu hóa chi tiết {#invalidation}

Khi bạn thực hiện thay đổi trong một gói, chỉ cần biên dịch lại
các gói import gói đó,
trực tiếp hoặc gián tiếp.
Ý tưởng này là cơ sở của tất cả các hệ thống build tăng dần từ Make vào những năm 1970,
và gopls đã sử dụng nó từ khi bắt đầu.
Thực tế, mỗi lần gõ phím trong trình soạn thảo được bật LSP bắt đầu một build tăng dần!
Tuy nhiên, trong một dự án lớn, các dependency gián tiếp tích lũy,
làm cho các rebuild tăng dần này quá chậm.
Hóa ra rất nhiều công việc này không thực sự cần thiết,
vì hầu hết các thay đổi, chẳng hạn như thêm câu lệnh trong một hàm hiện có,
không ảnh hưởng đến các bản tóm tắt import.

Nếu bạn thực hiện thay đổi nhỏ trong một file, chúng ta phải biên dịch lại gói của nó,
nhưng nếu thay đổi không ảnh hưởng đến bản tóm tắt import, chúng ta không phải biên dịch bất kỳ gói nào khác.
Hiệu ứng của thay đổi được "cắt tỉa". Thay đổi ảnh hưởng đến bản tóm tắt import
yêu cầu biên dịch lại các gói import trực tiếp gói đó,
nhưng hầu hết các thay đổi như vậy sẽ không ảnh hưởng đến bản tóm tắt import của _các_ gói _đó_,
trong trường hợp đó hiệu ứng vẫn được cắt tỉa và tránh biên dịch lại các importer gián tiếp.
Nhờ cắt tỉa này, hiếm khi thay đổi trong gói cấp thấp
yêu cầu biên dịch lại _tất cả_ các gói phụ thuộc gián tiếp vào gói đó.
Rebuild tăng dần được cắt tỉa làm cho lượng công việc tỷ lệ với
phạm vi của mỗi thay đổi.
Đây không phải là ý tưởng mới: nó được giới thiệu bởi [Vesta](https://www.hpl.hp.com/techreports/Compaq-DEC/SRC-RR-177.pdf)
và cũng được sử dụng trong [`go build`](/doc/go1.10#build).

Bản phát hành v0.12 giới thiệu một kỹ thuật cắt tỉa tương tự cho gopls,
đi thêm một bước để triển khai một heuristic cắt tỉa nhanh hơn dựa trên phân tích cú pháp.
Bằng cách giữ một đồ thị đơn giản hóa các tham chiếu ký hiệu trong bộ nhớ,
gopls có thể nhanh chóng xác định liệu thay đổi trong gói `c` có thể
ảnh hưởng đến gói `a` thông qua một chuỗi tham chiếu hay không.

<div class="image">
<img src="gopls-scalability/precise-pruning.png" alt="vô hiệu hóa chi tiết" class="chart"/>
</div>

Trong ví dụ trên, không có chuỗi tham chiếu từ `a` đến `c`,
vì vậy a không bị ảnh hưởng bởi các thay đổi trong c mặc dù nó phụ thuộc gián tiếp vào nó.

## Các khả năng mới {#new-possibilities}

Mặc dù chúng tôi hài lòng với những cải tiến hiệu năng đã đạt được,
chúng tôi cũng hào hứng về một số tính năng gopls khả thi giờ khi
gopls không còn bị ràng buộc bởi bộ nhớ.

Đầu tiên là phân tích tĩnh mạnh mẽ. Trước đây,
driver phân tích tĩnh của chúng tôi phải hoạt động trên biểu diễn trong bộ nhớ của gopls về các gói,
vì vậy nó không thể phân tích các dependency:
làm vậy sẽ kéo vào quá nhiều code bổ sung.
Với yêu cầu đó được loại bỏ, chúng tôi có thể bao gồm driver phân tích mới
trong gopls v0.12 phân tích tất cả các dependency,
dẫn đến độ chính xác cao hơn.
Ví dụ, gopls giờ báo cáo chẩn đoán cho các lỗi định dạng `Printf`
ngay cả trong các wrapper do người dùng định nghĩa xung quanh `fmt.Printf`.
Đáng chú ý, `go vet` đã cung cấp mức độ chính xác này trong nhiều năm,
nhưng gopls không thể làm điều này theo thời gian thực sau mỗi lần chỉnh sửa. Bây giờ nó có thể.

Thứ hai là [cấu hình workspace đơn giản hơn](/issue/57979)
và [xử lý cải thiện cho các build tag](/issue/29202).
Hai tính năng này đều có nghĩa là gopls "làm điều đúng" khi bạn
mở bất kỳ file Go nào trên máy của mình,
nhưng cả hai đều không khả thi nếu không có công việc tối ưu hóa vì (ví dụ)
mỗi cấu hình build nhân dấu ấn bộ nhớ!

## Hãy thử! {#try}

Ngoài các cải tiến khả năng mở rộng và hiệu năng,
chúng tôi cũng đã sửa [nhiều](https://github.com/golang/go/milestone/282?closed=1)
[lỗi được báo cáo](https://github.com/golang/go/milestone/318?closed=1) và
nhiều lỗi chưa được báo cáo mà chúng tôi phát hiện trong khi cải thiện độ bao phủ kiểm thử trong quá trình chuyển đổi.

Để cài đặt gopls mới nhất:

```
$ go install golang.org/x/tools/gopls@latest
```

Hãy thử nó và điền vào [khảo sát](https://google.qualtrics.com/jfe/form/SV_4SnGxpcSKN33WZw?s=blog) - và nếu bạn gặp lỗi,
[báo cáo nó](/issue/new) và chúng tôi sẽ sửa nó.
