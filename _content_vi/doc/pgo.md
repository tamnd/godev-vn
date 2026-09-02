---
title: Tối ưu hóa dựa trên hồ sơ thực thi
layout: article
---

Bắt đầu từ Go 1.20, trình biên dịch Go hỗ trợ tối ưu hóa dựa trên hồ sơ thực thi (PGO) để tối ưu hóa hơn nữa quá trình xây dựng.

Mục lục:

 [Tổng quan](#overview)\
 [Thu thập hồ sơ](#collecting-profiles)\
 [Xây dựng với PGO](#building)\
 [Ghi chú](#notes)\
 [Câu hỏi thường gặp](#faq)\
 [Phụ lục: các nguồn hồ sơ thay thế](#alternative-sources)

# Tổng quan {#overview}

Tối ưu hóa dựa trên hồ sơ thực thi (PGO), còn được gọi là tối ưu hóa theo phản hồi trực tiếp (FDO), là một kỹ thuật tối ưu hóa trình biên dịch đưa thông tin (một hồ sơ) từ các lần chạy đại diện của ứng dụng trở lại trình biên dịch cho lần xây dựng tiếp theo của ứng dụng, sử dụng thông tin đó để đưa ra quyết định tối ưu hóa sáng suốt hơn.
Ví dụ, trình biên dịch có thể quyết định nội tuyến tích cực hơn các hàm mà hồ sơ cho thấy được gọi thường xuyên.

Trong Go, trình biên dịch sử dụng hồ sơ CPU pprof làm hồ sơ đầu vào, chẳng hạn từ [runtime/pprof](https://pkg.go.dev/runtime/pprof) hoặc [net/http/pprof](https://pkg.go.dev/net/http/pprof).

Tính đến Go 1.22, các benchmark cho một tập hợp chương trình Go đại diện cho thấy việc xây dựng với PGO cải thiện hiệu suất khoảng 2-14%.
Chúng tôi dự kiến lợi ích hiệu suất sẽ tăng dần theo thời gian khi các tối ưu hóa bổ sung tận dụng PGO trong các phiên bản Go tương lai.


# Thu thập hồ sơ {#collecting-profiles}

Trình biên dịch Go yêu cầu một hồ sơ CPU pprof làm đầu vào cho PGO.
Các hồ sơ được tạo bởi Go runtime (chẳng hạn từ [runtime/pprof](https://pkg.go.dev/runtime/pprof) và [net/http/pprof](https://pkg.go.dev/net/http/pprof)) có thể được dùng trực tiếp làm đầu vào cho trình biên dịch.
Cũng có thể sử dụng hoặc chuyển đổi hồ sơ từ các hệ thống đo hiệu năng khác. Xem [phụ lục](#alternative-sources) để biết thêm thông tin.

Để đạt kết quả tốt nhất, điều quan trọng là các hồ sơ phải _đại diện_ cho hành vi thực tế của ứng dụng trong môi trường production.
Sử dụng hồ sơ không đại diện nhiều khả năng sẽ cho ra binary với ít hoặc không có cải thiện trong production.
Do đó, việc thu thập hồ sơ trực tiếp từ môi trường production được khuyến nghị, và là phương pháp chính mà PGO của Go được thiết kế cho.

Quy trình làm việc thông thường như sau:

1. Xây dựng và phát hành binary ban đầu (không dùng PGO).
2. Thu thập hồ sơ từ production.
3. Khi đến lúc phát hành binary cập nhật, xây dựng từ source mới nhất và cung cấp hồ sơ production.
4. GOTO 2

PGO của Go thường mạnh mẽ trước sự chênh lệch giữa phiên bản đã được đo hồ sơ của ứng dụng và phiên bản đang xây dựng với hồ sơ, cũng như khi xây dựng với các hồ sơ thu thập từ binary đã được tối ưu hóa.
Đây là điều làm cho vòng đời lặp này khả thi.
Xem phần [AutoFDO](#autofdo) để biết thêm chi tiết về quy trình này.

Nếu việc thu thập từ môi trường production khó hoặc không thể thực hiện được (ví dụ: một công cụ dòng lệnh được phân phối cho người dùng cuối), cũng có thể thu thập từ một benchmark đại diện.
Lưu ý rằng việc xây dựng benchmark đại diện thường khá khó khăn (cũng như việc giữ cho chúng đại diện khi ứng dụng phát triển).
Đặc biệt, _microbenchmark thường là các ứng viên tệ cho việc đo hồ sơ PGO_, vì chúng chỉ kiểm tra một phần nhỏ của ứng dụng, mang lại lợi ích nhỏ khi áp dụng cho toàn bộ chương trình.

# Xây dựng với PGO {#building}

Cách xây dựng chuẩn là lưu một hồ sơ CPU pprof với tên tệp `default.pgo` trong thư mục gói main của binary được đo hồ sơ.
Theo mặc định, `go build` sẽ tự động phát hiện các tệp `default.pgo` và bật PGO.

Khuyến nghị commit hồ sơ trực tiếp trong kho mã nguồn vì hồ sơ là đầu vào của quá trình xây dựng quan trọng cho các bản build có khả năng tái tạo (và hiệu suất cao!).
Lưu cùng với mã nguồn đơn giản hóa trải nghiệm xây dựng vì không cần thêm bước nào để lấy hồ sơ ngoài việc fetch mã nguồn.

Đối với các tình huống phức tạp hơn, cờ `go build -pgo` kiểm soát việc chọn hồ sơ PGO.
Cờ này mặc định là `-pgo=auto` cho hành vi `default.pgo` được mô tả ở trên.
Đặt cờ thành `-pgo=off` sẽ tắt hoàn toàn các tối ưu hóa PGO.

Nếu bạn không thể dùng `default.pgo` (ví dụ: hồ sơ khác nhau cho các tình huống khác nhau của một binary, không thể lưu hồ sơ cùng với mã nguồn, v.v.), bạn có thể truyền trực tiếp đường dẫn đến hồ sơ cần dùng (ví dụ: `go build -pgo=/tmp/foo.pprof`).

_Lưu ý: Đường dẫn truyền cho `-pgo` áp dụng cho tất cả các gói main.
Ví dụ: `go build -pgo=/tmp/foo.pprof ./cmd/foo ./cmd/bar` áp dụng `foo.pprof` cho cả hai binary `foo` và `bar`, điều này thường không phải là điều bạn muốn.
Thông thường các binary khác nhau nên có hồ sơ khác nhau, được truyền qua các lệnh `go build` riêng biệt._

_Lưu ý: Trước Go 1.21, mặc định là `-pgo=off`. PGO phải được bật rõ ràng._

# Ghi chú {#notes}

## Thu thập hồ sơ đại diện từ production

Môi trường production của bạn là nguồn hồ sơ đại diện tốt nhất cho ứng dụng của bạn, như đã mô tả trong phần [Thu thập hồ sơ](#collecting-profiles).

Cách đơn giản nhất để bắt đầu là thêm [net/http/pprof](https://pkg.go.dev/net/http/pprof) vào ứng dụng của bạn rồi fetch `/debug/pprof/profile?seconds=30` từ một instance tùy ý của service.
Đây là cách tuyệt vời để bắt đầu, nhưng có những cách mà hồ sơ này có thể không đại diện:

* Instance này có thể không làm gì tại thời điểm được đo hồ sơ, dù nó thường bận.

* Các mẫu lưu lượng có thể thay đổi trong ngày, khiến hành vi thay đổi theo ngày.

* Các instance có thể thực hiện các thao tác chạy lâu (ví dụ: 5 phút làm thao tác A, sau đó 5 phút làm thao tác B, v.v.).
  Một hồ sơ 30 giây có thể chỉ bao phủ một loại thao tác duy nhất.

* Các instance có thể không nhận phân phối công bằng của các yêu cầu (một số instance nhận nhiều loại yêu cầu hơn các instance khác).

Một chiến lược mạnh mẽ hơn là thu thập nhiều hồ sơ ở các thời điểm khác nhau từ các instance khác nhau để hạn chế tác động của sự khác biệt giữa các hồ sơ instance riêng lẻ.
Nhiều hồ sơ có thể được [hợp nhất](#merging-profiles) thành một hồ sơ duy nhất để dùng với PGO.

Nhiều tổ chức chạy các dịch vụ "đo hồ sơ liên tục" thực hiện loại đo hồ sơ lấy mẫu trên toàn đội tàu này một cách tự động, sau đó có thể được dùng làm nguồn hồ sơ cho PGO.

## Hợp nhất hồ sơ {#merging-profiles}

Công cụ pprof có thể hợp nhất nhiều hồ sơ như sau:

```
$ go tool pprof -proto a.pprof b.pprof > merged.pprof
```

Việc hợp nhất này thực chất là tổng đơn giản của các mẫu trong đầu vào, bất kể thời gian thực của hồ sơ.
Do đó, khi đo hồ sơ một lát thời gian nhỏ của ứng dụng (ví dụ: một máy chủ chạy vô thời hạn), bạn nên đảm bảo rằng tất cả các hồ sơ có cùng thời gian thực (tức là tất cả hồ sơ được thu thập trong 30 giây).
Ngược lại, các hồ sơ có thời gian thực dài hơn sẽ được biểu diễn quá mức trong hồ sơ đã hợp nhất.

## AutoFDO {#autofdo}

PGO của Go được thiết kế để hỗ trợ quy trình làm việc theo phong cách "[AutoFDO](https://research.google/pubs/pub45290/)".

Hãy xem xét kỹ hơn quy trình được mô tả trong phần [Thu thập hồ sơ](#collecting-profiles):

1. Xây dựng và phát hành binary ban đầu (không dùng PGO).
2. Thu thập hồ sơ từ production.
3. Khi đến lúc phát hành binary cập nhật, xây dựng từ source mới nhất và cung cấp hồ sơ production.
4. GOTO 2

Nghe có vẻ đơn giản, nhưng có một số thuộc tính quan trọng cần lưu ý ở đây:

* Việc phát triển luôn diễn ra liên tục, vì vậy mã nguồn của phiên bản đã được đo hồ sơ của binary (bước 2) có thể hơi khác so với mã nguồn mới nhất đang được xây dựng (bước 3).
  PGO của Go được thiết kế để mạnh mẽ trước điều này, mà chúng tôi gọi là _tính ổn định của mã nguồn_.

* Đây là một vòng khép kín.
  Nghĩa là, sau lần lặp đầu tiên, phiên bản đã được đo hồ sơ của binary đã là phiên bản được tối ưu hóa PGO với hồ sơ từ lần lặp trước đó.
  PGO của Go cũng được thiết kế để mạnh mẽ trước điều này, mà chúng tôi gọi là _tính ổn định lặp_.

_Tính ổn định của mã nguồn_ đạt được bằng cách sử dụng heuristic để khớp các mẫu từ hồ sơ với mã nguồn đang biên dịch.
Do đó, nhiều thay đổi đối với mã nguồn, chẳng hạn như thêm hàm mới, không ảnh hưởng đến việc khớp mã hiện có.
Khi trình biên dịch không thể khớp mã đã thay đổi, một số tối ưu hóa bị mất, nhưng lưu ý rằng đây là _sự giảm cấp dần_.
Một hàm không khớp có thể bỏ lỡ cơ hội tối ưu hóa, nhưng lợi ích PGO tổng thể thường được phân bổ trên nhiều hàm. Xem phần [tính ổn định của mã nguồn](#source-stability) để biết thêm chi tiết về việc khớp và giảm cấp.

_Tính ổn định lặp_ là việc ngăn chặn các chu kỳ hiệu suất biến đổi trong các bản build PGO liên tiếp (ví dụ: build #1 nhanh, build #2 chậm, build #3 nhanh, v.v.).
Chúng tôi dùng hồ sơ CPU để xác định các hàm nóng cần tối ưu hóa.
Về lý thuyết, một hàm nóng có thể được tăng tốc nhiều đến mức nó không còn xuất hiện nóng trong hồ sơ tiếp theo và không được tối ưu hóa, khiến nó chậm lại.
Trình biên dịch Go áp dụng cách tiếp cận bảo thủ đối với các tối ưu hóa PGO, mà chúng tôi tin là ngăn chặn sự biến đổi đáng kể.
Nếu bạn quan sát thấy loại không ổn định này, hãy báo cáo sự cố tại [go.dev/issue/new](/issue/new).

Cùng nhau, tính ổn định của mã nguồn và lặp loại bỏ yêu cầu xây dựng hai giai đoạn, trong đó bản build đầu tiên không được tối ưu hóa được đo hồ sơ như một canary, sau đó được xây dựng lại với PGO cho production (trừ khi cần đỉnh hiệu suất tuyệt đối).

## Tính ổn định của mã nguồn và tái cấu trúc {#source-stability}

Như đã mô tả ở trên, PGO của Go nỗ lực tốt nhất để tiếp tục khớp các mẫu từ hồ sơ cũ hơn với mã nguồn hiện tại.
Cụ thể, Go sử dụng độ lệch dòng trong các hàm (ví dụ: lời gọi trên dòng thứ 5 của hàm foo).

Nhiều thay đổi phổ biến sẽ không phá vỡ việc khớp, bao gồm:

* Thay đổi trong tệp bên ngoài hàm nóng (thêm/thay đổi mã trên hoặc dưới hàm).

* Di chuyển hàm sang tệp khác trong cùng gói (trình biên dịch hoàn toàn bỏ qua tên tệp nguồn).

Một số thay đổi có thể phá vỡ việc khớp:

* Thay đổi trong hàm nóng (có thể ảnh hưởng đến độ lệch dòng).

* Đổi tên hàm (và/hoặc kiểu cho các phương thức) (thay đổi tên symbol).

* Di chuyển hàm sang gói khác (thay đổi tên symbol).

Nếu hồ sơ tương đối gần đây, các khác biệt có thể chỉ ảnh hưởng đến một số ít hàm nóng, hạn chế tác động của các tối ưu hóa bị bỏ lỡ trong các hàm không khớp được.
Tuy nhiên, sự giảm cấp sẽ dần tích lũy theo thời gian vì mã hiếm khi được tái cấu trúc _trở lại_ dạng cũ, vì vậy điều quan trọng là phải thu thập hồ sơ mới thường xuyên để hạn chế sự chênh lệch từ production.

Một tình huống mà việc khớp hồ sơ có thể giảm cấp đáng kể là tái cấu trúc quy mô lớn đổi tên nhiều hàm hoặc di chuyển chúng giữa các gói.
Trong trường hợp này, bạn có thể chịu một đợt giảm hiệu suất ngắn hạn cho đến khi hồ sơ mới phản ánh cấu trúc mới.

Đối với các việc đổi tên đơn thuần, một hồ sơ hiện có về lý thuyết có thể được viết lại để thay đổi tên symbol cũ sang tên mới.
[github.com/google/pprof/profile](https://pkg.go.dev/github.com/google/pprof/profile) chứa các nguyên hàm cần thiết để viết lại hồ sơ pprof theo cách này, nhưng tính đến thời điểm viết này không có công cụ nào sẵn sàng để dùng ngay cho mục đích này.

## Hiệu suất của mã mới

Khi thêm mã mới hoặc bật các đường dẫn mã mới với việc đổi cờ, mã đó sẽ không có trong hồ sơ ở lần build đầu tiên, và do đó sẽ không nhận được tối ưu hóa PGO cho đến khi một hồ sơ mới phản ánh mã mới được thu thập.
Hãy lưu ý khi đánh giá việc triển khai mã mới rằng bản phát hành ban đầu sẽ không đại diện cho hiệu suất ổn định của nó.

# Câu hỏi thường gặp {#faq}

## Có thể tối ưu hóa các gói thư viện chuẩn Go với PGO không?

Có.
PGO trong Go áp dụng cho toàn bộ chương trình.
Tất cả các gói được xây dựng lại để xem xét các tối ưu hóa dựa trên hồ sơ tiềm năng, bao gồm cả các gói thư viện chuẩn.

## Có thể tối ưu hóa các gói trong các module phụ thuộc với PGO không?

Có.
PGO trong Go áp dụng cho toàn bộ chương trình.
Tất cả các gói được xây dựng lại để xem xét các tối ưu hóa dựa trên hồ sơ tiềm năng, bao gồm cả các gói trong các dependency.
Điều này có nghĩa là cách ứng dụng của bạn sử dụng một dependency ảnh hưởng đến các tối ưu hóa được áp dụng cho dependency đó.

## PGO với hồ sơ không đại diện có làm chương trình của tôi chậm hơn so với không dùng PGO không?

Không nên như vậy.
Trong khi một hồ sơ không đại diện cho hành vi production sẽ dẫn đến tối ưu hóa ở các phần lạnh của ứng dụng, nó không nên làm chậm các phần nóng của ứng dụng.
Nếu bạn gặp trường hợp PGO dẫn đến hiệu suất kém hơn so với tắt PGO, hãy báo cáo sự cố tại [go.dev/issue/new](/issue/new).

## Tôi có thể dùng cùng một hồ sơ cho các bản build GOOS/GOARCH khác nhau không?

Có.
Định dạng của các hồ sơ tương đương nhau trên các cấu hình OS và kiến trúc, vì vậy chúng có thể được dùng trên các cấu hình khác nhau.
Ví dụ, hồ sơ được thu thập từ binary linux/arm64 có thể được dùng trong bản build windows/amd64.

Tuy nhiên, các lưu ý về tính ổn định của mã nguồn được thảo luận [ở trên](#autofdo) cũng áp dụng ở đây.
Bất kỳ mã nào khác nhau giữa các cấu hình này sẽ không được tối ưu hóa.
Đối với hầu hết các ứng dụng, phần lớn mã là không phụ thuộc nền tảng, vì vậy sự giảm cấp dạng này bị hạn chế.

Ví dụ cụ thể, phần nội bộ của việc xử lý tệp trong gói `os` khác nhau giữa Linux và Windows.
Nếu các hàm này nóng trong hồ sơ Linux, các hàm tương đương của Windows sẽ không nhận được tối ưu hóa PGO vì chúng không khớp với hồ sơ.

Bạn có thể hợp nhất các hồ sơ của các bản build GOOS/GOARCH khác nhau. Xem câu hỏi tiếp theo để biết các đánh đổi khi làm vậy.

## Làm thế nào để xử lý một binary duy nhất được dùng cho nhiều loại khối lượng công việc khác nhau?

Không có lựa chọn rõ ràng nào ở đây.
Một binary duy nhất được dùng cho các loại khối lượng công việc khác nhau (ví dụ: một cơ sở dữ liệu được dùng theo cách đọc nhiều trong một service, và ghi nhiều trong service khác) có thể có các thành phần nóng khác nhau, được hưởng lợi từ các tối ưu hóa khác nhau.

Có ba tùy chọn:

1. Xây dựng các phiên bản khác nhau của binary cho mỗi khối lượng công việc: dùng hồ sơ từ mỗi khối lượng công việc để xây dựng nhiều bản build dành riêng cho từng khối lượng công việc.
   Điều này sẽ cung cấp hiệu suất tốt nhất cho mỗi khối lượng công việc, nhưng có thể tăng thêm độ phức tạp vận hành liên quan đến việc xử lý nhiều binary và nguồn hồ sơ.

2. Xây dựng một binary duy nhất chỉ dùng hồ sơ từ khối lượng công việc "quan trọng nhất": chọn khối lượng công việc "quan trọng nhất" (lớn nhất, nhạy cảm nhất về hiệu suất), và xây dựng chỉ dùng hồ sơ từ khối lượng công việc đó.
   Điều này cung cấp hiệu suất tốt nhất cho khối lượng công việc được chọn, và có thể vẫn cải thiện hiệu suất khiêm tốn cho các khối lượng công việc khác từ việc tối ưu hóa mã dùng chung trên các khối lượng công việc.

3. Hợp nhất hồ sơ trên các khối lượng công việc: lấy hồ sơ từ mỗi khối lượng công việc (có trọng số theo tổng dấu ấn) và hợp nhất chúng thành một hồ sơ "toàn đội tàu" duy nhất được dùng để xây dựng.
   Điều này có thể cung cấp cải thiện hiệu suất khiêm tốn cho tất cả các khối lượng công việc.

## PGO ảnh hưởng đến thời gian xây dựng như thế nào?

Bật bản build PGO có thể gây ra sự tăng đáng kể trong thời gian xây dựng gói.
Thành phần đáng chú ý nhất là các hồ sơ PGO áp dụng cho tất cả các gói trong một binary, có nghĩa là lần đầu tiên sử dụng hồ sơ yêu cầu xây dựng lại mọi gói trong đồ thị dependency.
Những bản build này được cache như bất kỳ bản build nào khác, vì vậy các bản build tăng dần tiếp theo sử dụng cùng hồ sơ không yêu cầu xây dựng lại hoàn toàn.

Nếu bạn gặp tình trạng tăng thời gian xây dựng cực đoan, hãy báo cáo sự cố tại [go.dev/issue/new](/issue/new).

## PGO ảnh hưởng đến kích thước binary như thế nào?

PGO có thể dẫn đến binary lớn hơn một chút do nội tuyến hàm bổ sung.

# Phụ lục: các nguồn hồ sơ thay thế {#alternative-sources}

Các hồ sơ CPU được tạo bởi Go runtime (thông qua [runtime/pprof](https://pkg.go.dev/runtime/pprof), v.v.) đã ở định dạng đúng để dùng trực tiếp làm đầu vào PGO.
Tuy nhiên, các tổ chức có thể có công cụ ưu tiên thay thế (ví dụ: Linux perf), hoặc các hệ thống đo hồ sơ liên tục trên toàn đội tàu hiện có mà họ muốn dùng với PGO của Go.

Các hồ sơ từ nguồn thay thế có thể được dùng với PGO của Go nếu được chuyển đổi sang [định dạng pprof](https://github.com/google/pprof/tree/main/proto), miễn là chúng đáp ứng các yêu cầu chung sau:

* Một trong các chỉ số mẫu phải có kiểu/đơn vị là "samples"/"count" hoặc "cpu"/"nanoseconds".

* Các mẫu phải đại diện cho các mẫu thời gian CPU tại vị trí lấy mẫu.

* Hồ sơ phải được symbol hóa ([Function.name](https://github.com/google/pprof/blob/76d1ae5aea2b3f738f2058d17533b747a1a5cd01/proto/profile.proto#L208) phải được đặt).

* Các mẫu phải chứa các khung stack cho các hàm được nội tuyến.
  Nếu các hàm nội tuyến bị bỏ qua, Go sẽ không thể duy trì tính ổn định lặp.

* [Function.start_line](https://github.com/google/pprof/blob/76d1ae5aea2b3f738f2058d17533b747a1a5cd01/proto/profile.proto#L215) phải được đặt.
  Đây là số dòng bắt đầu của hàm.
  Tức là dòng chứa từ khóa `func`.
  Trình biên dịch Go sử dụng trường này để tính toán độ lệch dòng của các mẫu (`Location.Line.line - Function.start_line`).
  **Lưu ý rằng nhiều trình chuyển đổi pprof hiện có bỏ qua trường này.**

_Lưu ý: Trước Go 1.21, siêu dữ liệu DWARF bỏ qua các dòng bắt đầu hàm (`DW_AT_decl_line`), điều này có thể khiến các công cụ khó xác định dòng bắt đầu._

Xem trang [PGO Tools](/wiki/PGO-Tools) trên Go Wiki để biết thêm thông tin về tính tương thích PGO của các công cụ bên thứ ba cụ thể.
