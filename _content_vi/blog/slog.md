---
title: Structured Logging với slog
date: 2023-08-22
by:
- Jonathan Amsterdam
summary: Thư viện chuẩn Go 1.21 bổ sung gói structured logging mới, log/slog.
---

Gói `log/slog` mới trong Go 1.21 mang structured logging vào thư viện chuẩn. Structured log sử dụng các cặp khóa-giá trị để có thể phân tích cú pháp, lọc, tìm kiếm và phân tích nhanh chóng, đáng tin cậy.
Đối với các máy chủ, logging là cách quan trọng để các lập trình viên quan sát hành vi chi tiết của hệ thống, và thường là nơi đầu tiên họ đến để gỡ lỗi. Do đó, log thường rất nhiều, và khả năng tìm kiếm, lọc nhanh là điều thiết yếu.

Thư viện chuẩn đã có gói logging `log` kể từ lần phát hành đầu tiên của Go hơn một thập kỷ trước.
Theo thời gian, chúng tôi nhận ra rằng structured logging là điều quan trọng với các lập trình viên Go. Nó luôn xếp hạng cao trong khảo sát hàng năm của chúng tôi, và nhiều gói trong hệ sinh thái Go đã cung cấp tính năng này. Một số gói khá phổ biến: một trong những gói structured logging đầu tiên cho Go, [logrus](https://pkg.go.dev/github.com/sirupsen/logrus), được dùng trong hơn 100.000 gói khác.

Với nhiều gói structured logging để lựa chọn, các chương trình lớn thường sẽ kết thúc với việc bao gồm hơn một gói thông qua các dependency. Chương trình chính có thể phải cấu hình từng gói logging này để đầu ra log nhất quán: tất cả đi đến cùng một nơi, theo cùng một định dạng. Bằng cách đưa structured logging vào thư viện chuẩn, chúng tôi có thể cung cấp một framework chung mà tất cả các gói structured logging khác có thể dùng chung.

## Khám phá `slog`

Đây là chương trình đơn giản nhất sử dụng `slog`:

```
package main

import "log/slog"

func main() {
	slog.Info("hello, world")
}
```

Tại thời điểm viết bài này, nó in ra:

    2023/08/04 16:09:19 INFO hello, world

Hàm `Info` in một thông báo ở mức log Info bằng logger mặc định, trong trường hợp này là logger mặc định từ gói `log`, cùng logger bạn nhận được khi viết `log.Printf`.
Điều đó giải thích vì sao đầu ra trông rất giống nhau: chỉ có "INFO" là mới.
Ngay từ đầu, `slog` và gói `log` gốc phối hợp với nhau để dễ dàng bắt đầu.

Ngoài `Info`, còn có các hàm cho ba mức khác là `Debug`, `Warn`, và `Error`, cùng với một hàm `Log` tổng quát hơn nhận mức log làm tham số. Trong `slog`, các mức chỉ là số nguyên, do đó bạn không bị giới hạn ở bốn mức có tên. Ví dụ, `Info` là 0 và `Warn` là 4, vì vậy nếu hệ thống logging của bạn có một mức ở giữa, bạn có thể dùng 2 cho mức đó.

Không giống với gói `log`, chúng ta có thể dễ dàng thêm các cặp khóa-giá trị vào đầu ra bằng cách viết chúng sau thông báo:

```
slog.Info("hello, world", "user", os.Getenv("USER"))
```

Đầu ra bây giờ trông như sau:

    2023/08/04 16:27:19 INFO hello, world user=jba

Như đã đề cập, các hàm cấp cao nhất của `slog` sử dụng logger mặc định.
Chúng ta có thể lấy logger này một cách tường minh và gọi các phương thức của nó:

```
logger := slog.Default()
logger.Info("hello, world", "user", os.Getenv("USER"))
```

Mỗi hàm cấp cao nhất tương ứng với một phương thức trên `slog.Logger`.
Đầu ra giống như trước.

Ban đầu, đầu ra của slog đi qua `log.Logger` mặc định, tạo ra đầu ra như chúng ta đã thấy ở trên.
Chúng ta có thể thay đổi đầu ra bằng cách thay đổi _handler_ được dùng bởi logger.
`slog` đi kèm với hai handler tích hợp sẵn.
Một `TextHandler` phát ra tất cả thông tin log theo dạng `key=value`.
Chương trình này tạo một logger mới sử dụng `TextHandler` và gọi phương thức `Info` tương tự:

```
logger := slog.New(slog.NewTextHandler(os.Stdout, nil))
logger.Info("hello, world", "user", os.Getenv("USER"))
```

Bây giờ đầu ra trông như sau:

    time=2023-08-04T16:56:03.786-04:00 level=INFO msg="hello, world" user=jba

Mọi thứ đã được chuyển thành cặp khóa-giá trị, với các chuỗi được trích dẫn khi cần để bảo toàn cấu trúc.

Để có đầu ra JSON, hãy cài đặt `JSONHandler` tích hợp sẵn:

```
logger := slog.New(slog.NewJSONHandler(os.Stdout, nil))
logger.Info("hello, world", "user", os.Getenv("USER"))
```

Bây giờ đầu ra của chúng ta là một chuỗi các đối tượng JSON, một đối tượng cho mỗi lần gọi logging:

    {"time":"2023-08-04T16:58:02.939245411-04:00","level":"INFO","msg":"hello, world","user":"jba"}

Bạn không bị giới hạn ở các handler tích hợp sẵn. Bất kỳ ai cũng có thể viết một handler bằng cách triển khai interface `slog.Handler`. Một handler có thể tạo đầu ra theo một định dạng cụ thể, hoặc có thể bọc một handler khác để thêm chức năng.
Một trong những [ví dụ](https://pkg.go.dev/log/slog@master#example-Handler-LevelHandler) trong tài liệu `slog` cho thấy cách viết một wrapping handler thay đổi mức tối thiểu mà các thông báo log sẽ được hiển thị.

Cú pháp khóa-giá trị xen kẽ cho các thuộc tính mà chúng ta đã dùng cho đến nay rất tiện lợi, nhưng đối với các câu lệnh log thực thi thường xuyên, có thể hiệu quả hơn khi sử dụng kiểu `Attr` và gọi phương thức `LogAttrs`.
Chúng phối hợp để giảm thiểu việc cấp phát bộ nhớ.
Có các hàm để tạo `Attr` từ chuỗi, số và các kiểu thông dụng khác. Lời gọi `LogAttrs` này tạo ra đầu ra giống như trên, nhưng nhanh hơn:

```
slog.LogAttrs(context.Background(), slog.LevelInfo, "hello, world",
    slog.String("user", os.Getenv("USER")))
```

`slog` còn nhiều điều hơn thế:

- Như lời gọi `LogAttrs` cho thấy, bạn có thể truyền `context.Context` vào một số hàm log để handler có thể trích xuất thông tin context như trace ID. (Hủy context không ngăn mục log được ghi.)

- Bạn có thể gọi `Logger.With` để thêm thuộc tính vào logger, chúng sẽ xuất hiện trong toàn bộ đầu ra của nó, thực chất là trích xuất phần chung của nhiều câu lệnh log. Điều này không chỉ tiện lợi mà còn giúp cải thiện hiệu năng, như thảo luận bên dưới.

- Các thuộc tính có thể được kết hợp thành các nhóm. Điều này có thể thêm cấu trúc hơn cho đầu ra log và giúp phân biệt các khóa mà nếu không sẽ bị trùng nhau.

- Bạn có thể kiểm soát cách một giá trị xuất hiện trong log bằng cách cung cấp kiểu của nó với phương thức `LogValue`. Điều đó có thể được dùng để [log các trường của một struct dưới dạng nhóm](https://pkg.go.dev/log/slog@master#example-LogValuer-Group) hoặc [che giấu dữ liệu nhạy cảm](https://pkg.go.dev/log/slog@master#example-LogValuer-Secret), cùng nhiều mục đích khác.

Nơi tốt nhất để tìm hiểu về tất cả `slog` là [tài liệu gói](https://pkg.go.dev/log/slog).


## Hiệu năng

Chúng tôi muốn `slog` phải nhanh.
Để đạt được những cải tiến hiệu năng ở quy mô lớn, chúng tôi đã thiết kế [interface `Handler`](https://pkg.go.dev/log/slog#Handler) để cung cấp các cơ hội tối ưu hóa. Phương thức `Enabled` được gọi ở đầu mỗi sự kiện log, cho handler cơ hội loại bỏ nhanh các sự kiện log không mong muốn. Các phương thức `WithAttrs` và `WithGroup` cho phép handler định dạng các thuộc tính được thêm bởi `Logger.With` một lần, thay vì ở mỗi lần gọi logging. Việc tiền định dạng này có thể cung cấp một cải thiện tốc độ đáng kể khi các thuộc tính lớn, như `http.Request`, được thêm vào `Logger` và sau đó được dùng trong nhiều lần gọi logging.

Để định hướng cho công việc tối ưu hóa hiệu năng, chúng tôi đã khảo sát các mẫu logging điển hình trong các dự án mã nguồn mở hiện có. Chúng tôi phát hiện rằng hơn 95% lời gọi đến các phương thức logging truyền năm thuộc tính trở xuống. Chúng tôi cũng phân loại các kiểu thuộc tính, phát hiện rằng một số kiểu thông dụng chiếm phần lớn.
Sau đó chúng tôi đã viết các benchmark nắm bắt các trường hợp phổ biến và dùng chúng làm hướng dẫn để xem thời gian đi đâu.
Những lợi ích lớn nhất đến từ việc chú ý cẩn thận đến việc cấp phát bộ nhớ.

## Quá trình thiết kế

Gói `slog` là một trong những bổ sung lớn nhất cho thư viện chuẩn kể từ khi Go 1 được phát hành vào năm 2012. Chúng tôi muốn dành thời gian thiết kế nó, và chúng tôi biết rằng phản hồi của cộng đồng sẽ là điều cần thiết.

Đến tháng 4 năm 2022, chúng tôi đã thu thập đủ dữ liệu để chứng minh tầm quan trọng của structured logging đối với cộng đồng Go. Nhóm Go quyết định khám phá việc thêm nó vào thư viện chuẩn.

Chúng tôi bắt đầu bằng cách xem xét cách các gói structured logging hiện có được thiết kế. Chúng tôi cũng tận dụng bộ sưu tập lớn mã Go mã nguồn mở được lưu trữ trên Go module proxy để tìm hiểu cách các gói này thực sự được sử dụng.
Thiết kế đầu tiên của chúng tôi được định hướng bởi nghiên cứu này cũng như tinh thần đơn giản của Go.
Chúng tôi muốn một API gọn nhẹ trên trang và dễ hiểu, mà không phải hy sinh hiệu năng.

Mục tiêu không bao giờ là thay thế các gói logging của bên thứ ba hiện có.
Chúng đều tốt trong những gì chúng làm, và việc thay thế mã hiện có hoạt động tốt thường không phải là cách sử dụng thời gian tốt của lập trình viên.
Chúng tôi chia API thành một frontend, `Logger`, gọi đến một backend interface, `Handler`.
Bằng cách đó, các gói logging hiện có có thể kết nối với một backend chung, để các gói sử dụng chúng có thể tương tác mà không cần viết lại.
Các handler đã được viết hoặc đang được phát triển cho nhiều gói logging phổ biến, bao gồm
[Zap](https://github.com/uber-go/zap/tree/master/exp/zapslog),
[logr](https://github.com/go-logr/logr/pull/196)
và [hclog](https://github.com/evanphx/go-hclog-slog).

Chúng tôi đã chia sẻ thiết kế ban đầu trong nhóm Go và các lập trình viên khác có kinh nghiệm logging phong phú. Chúng tôi đã thực hiện các thay đổi dựa trên phản hồi của họ, và đến tháng 8 năm 2022 chúng tôi cảm thấy mình có một thiết kế khả thi. Vào ngày 29 tháng 8, chúng tôi đã công bố [triển khai thử nghiệm](https://github.com/golang/exp/tree/master/slog) và bắt đầu một [cuộc thảo luận GitHub](https://github.com/golang/go/discussions/54763) để nghe cộng đồng nói gì.
Phản hồi rất nhiệt tình và phần lớn tích cực.
Nhờ những nhận xét sâu sắc từ các nhà thiết kế và người dùng của các gói structured logging khác, chúng tôi đã thực hiện một số thay đổi và thêm một số tính năng, như nhóm và interface `LogValuer`. Chúng tôi đã thay đổi ánh xạ từ mức log sang số nguyên hai lần.

Sau hai tháng và khoảng 300 nhận xét, chúng tôi cảm thấy đã sẵn sàng cho một [đề xuất](/issue/56345) thực sự và [tài liệu thiết kế](https://go.googlesource.com/proposal/+/03441cb358c7b27a8443bca839e5d7a314677ea6/design/56345-structured-logging.md) kèm theo.
Vấn đề đề xuất nhận được hơn 800 nhận xét và dẫn đến nhiều cải tiến cho API và triển khai. Dưới đây là hai ví dụ về thay đổi API, cả hai đều liên quan đến `context.Context`:

1. Ban đầu API hỗ trợ việc thêm logger vào context. Nhiều người cảm thấy đây là cách tiện lợi để dễ dàng truyền logger qua các tầng mã không quan tâm đến nó. Nhưng những người khác cảm thấy nó đang mang vào một dependency ngầm, làm cho mã khó hiểu hơn.
Cuối cùng, chúng tôi đã xóa tính năng này vì nó quá gây tranh cãi.

2. Chúng tôi cũng đã vật lộn với câu hỏi liên quan về việc truyền context vào các phương thức logging, thử nghiệm nhiều thiết kế. Ban đầu chúng tôi không muốn theo mẫu chuẩn là truyền context làm tham số đầu tiên vì chúng tôi không muốn mỗi lần gọi logging đều yêu cầu một context, nhưng cuối cùng đã tạo ra hai tập phương thức logging, một có context và một không có.

Một thay đổi chúng tôi đã không thực hiện liên quan đến cú pháp khóa-giá trị xen kẽ để biểu thị thuộc tính:

    slog.Info("message", "k1", v1, "k2", v2)

Nhiều người cảm thấy mạnh mẽ rằng đây là ý tưởng tồi. Họ thấy nó khó đọc và dễ bị sai khi bỏ sót một khóa hoặc giá trị. Họ thích các thuộc tính tường minh để biểu thị cấu trúc:

    slog.Info("message", slog.Int("k1", v1), slog.String("k2", v2))

Nhưng chúng tôi cảm thấy cú pháp nhẹ hơn rất quan trọng để giữ Go dễ dàng và thú vị khi sử dụng, đặc biệt là cho các lập trình viên Go mới. Chúng tôi cũng biết rằng một số gói logging Go, như `logr`, `go-kit/log` và `zap` (với `SugaredLogger`), đã thành công khi dùng các khóa và giá trị xen kẽ. Chúng tôi đã thêm một [kiểm tra vet](https://pkg.go.dev/golang.org/x/tools/go/analysis/passes/slog) để bắt các lỗi thường gặp, nhưng không thay đổi thiết kế.

Vào ngày 15 tháng 3 năm 2023, đề xuất được chấp nhận, nhưng vẫn còn một số vấn đề nhỏ chưa được giải quyết. Trong vài tuần tiếp theo, mười thay đổi bổ sung được đề xuất và giải quyết. Đến đầu tháng 7, việc triển khai gói `log/slog` đã hoàn chỉnh, cùng với gói `testing/slogtest` để xác minh các handler và kiểm tra vet cho việc sử dụng đúng các khóa và giá trị xen kẽ.

Và vào ngày 8 tháng 8, Go 1.21 được phát hành, cùng với `slog`.
Chúng tôi hy vọng bạn thấy nó hữu ích, và thú vị khi sử dụng như khi chúng tôi xây dựng nó.

Và lời cảm ơn lớn đến tất cả những người đã tham gia vào quá trình thảo luận và đề xuất. Đóng góp của bạn đã cải thiện `slog` rất nhiều.


## Tài nguyên

[Tài liệu](https://pkg.go.dev/log/slog) của gói `log/slog` giải thích cách sử dụng nó và cung cấp một số ví dụ.

[Trang wiki](/wiki/Resources-for-slog) có các tài nguyên bổ sung do cộng đồng Go cung cấp, bao gồm nhiều handler khác nhau.

Nếu bạn muốn viết một handler, hãy tham khảo [hướng dẫn viết handler](https://github.com/golang/example/blob/master/slog-handler-guide/README.md).
