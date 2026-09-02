---
title: Kiểm thử Thời gian (và các tính bất đồng bộ khác)
date: 2025-08-26
by:
- Damien Neil
tags:
- concurrency
- testing
- synctest
summary: Thảo luận về kiểm thử mã bất đồng bộ
  và khám phá gói `testing/synctest`.
  Dựa trên bài nói chuyện tại GopherCon Europe 2025 cùng tên.
---

Trong Go 1.24, chúng tôi đã giới thiệu gói [`testing/synctest`](/pkg/testing/synctest)
dưới dạng gói thử nghiệm.
Gói này có thể đơn giản hóa đáng kể việc viết bài kiểm thử cho mã đồng thời,
bất đồng bộ.
Trong Go 1.25, gói `testing/synctest` đã tốt nghiệp từ thử nghiệm
sang phiên bản chính thức.

Nội dung dưới đây là phiên bản blog của bài nói chuyện của tôi về
gói [`testing/synctest`](/pkg/testing/synctest)
tại GopherCon Europe 2025 ở Berlin.

## Hàm bất đồng bộ là gì?

Một hàm đồng bộ khá đơn giản.
Bạn gọi nó, nó làm gì đó, và nó trả về.

Một hàm bất đồng bộ thì khác.
Bạn gọi nó, nó trả về, và sau đó nó làm gì đó.

Để có một ví dụ cụ thể, dù hơi nhân tạo,
hàm `Cleanup` sau đây là đồng bộ.
Bạn gọi nó, nó xóa thư mục cache, và nó trả về.

```
func (c *Cache) Cleanup() {
    os.RemoveAll(c.cacheDir)
}
```

`CleanupInBackground` là một hàm bất đồng bộ.
Bạn gọi nó, nó trả về, và thư mục cache được xóa... sớm hay muộn.

```
func (c *Cache) CleanupInBackground() {
    go os.RemoveAll(c.cacheDir)
}
```

Đôi khi một hàm bất đồng bộ làm gì đó trong tương lai.
Ví dụ, hàm `WithDeadline` của gói `context`
trả về một context sẽ bị hủy trong tương lai.

```
package context

// WithDeadline returns a derived context
// with a deadline no later than d.
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)
```

Khi tôi nói về kiểm thử mã đồng thời,
tôi có nghĩa là kiểm thử các loại thao tác bất đồng bộ này,
cả những thao tác dùng thời gian thực và những thao tác không dùng.

## Bài kiểm thử

Một bài kiểm thử xác minh rằng một hệ thống hoạt động như chúng ta mong đợi.
Có rất nhiều thuật ngữ mô tả các loại
bài kiểm thử như unit test, integration test, v.v., nhưng
với mục đích ở đây mọi loại bài kiểm thử đều thu gọn thành ba bước:

1. Thiết lập một số điều kiện ban đầu.
2. Yêu cầu hệ thống được kiểm thử làm gì đó.
3. Xác minh kết quả.

Kiểm thử một hàm đồng bộ rất đơn giản:

- Bạn gọi hàm;
- hàm làm gì đó và trả về;
- bạn xác minh kết quả.

Tuy nhiên, kiểm thử một hàm bất đồng bộ thì phức tạp:

- Bạn gọi hàm;
- nó trả về;
- bạn chờ nó hoàn thành những gì nó làm;
- bạn xác minh kết quả.

Nếu bạn không chờ đúng lượng thời gian,
bạn có thể thấy mình đang xác minh kết quả của một thao tác chưa xảy ra
hoặc mới chỉ xảy ra một phần.
Điều này không bao giờ kết thúc tốt.

Kiểm thử một hàm bất đồng bộ đặc biệt phức tạp
khi bạn muốn khẳng định rằng điều gì đó *chưa* xảy ra.
Bạn có thể xác minh rằng điều đó chưa xảy ra,
nhưng làm sao bạn biết chắc chắn rằng nó sẽ không xảy ra sau?

## Một ví dụ

Để làm mọi thứ cụ thể hơn một chút,
hãy làm việc với một ví dụ thực tế.
Hãy xem xét lại hàm `WithDeadline` của gói `context`.

```
package context

// WithDeadline returns a derived context
// with a deadline no later than d.
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc)
```

Có hai bài kiểm thử hiển nhiên cần viết cho `WithDeadline`.

1. Context *không* bị hủy *trước* deadline.
2. Context *bị* hủy *sau* deadline.

Hãy viết một bài kiểm thử.

Để giữ cho lượng code ít áp đảo hơn một chút,
chúng ta sẽ chỉ kiểm thử trường hợp thứ hai:
Sau khi deadline hết hạn, context bị hủy.

```
func TestWithDeadlineAfterDeadline(t *testing.T) {
    deadline := time.Now().Add(1 * time.Second)
    ctx, _ := context.WithDeadline(t.Context(), deadline)

    time.Sleep(time.Until(deadline))

    if err := ctx.Err(); err != context.DeadlineExceeded {
        t.Fatalf("context not canceled after deadline")
    }
}
```

Bài kiểm thử này đơn giản:

1. Dùng `context.WithDeadline` để tạo một context với deadline một giây trong tương lai.
2. Chờ đến deadline.
3. Xác minh rằng context bị hủy.

Thật không may, bài kiểm thử này rõ ràng có vấn đề.
Nó ngủ đến chính xác thời điểm deadline hết hạn.
Rất có thể context chưa bị hủy vào thời điểm chúng ta kiểm tra nó.
Tốt nhất, bài kiểm thử này sẽ rất không ổn định.

Hãy sửa nó.

```
time.Sleep(time.Until(deadline) + 100*time.Millisecond)
```

Chúng ta có thể ngủ đến 100ms sau deadline.
Một trăm millisecond là một khoảng thời gian dài trong thế giới máy tính.
Điều này sẽ ổn thôi.

Thật không may, chúng ta vẫn có hai vấn đề.

Đầu tiên, bài kiểm thử này mất 1,1 giây để thực thi.
Đó là chậm.
Đây là một bài kiểm thử đơn giản.
Nó nên thực thi trong millisecond là cùng.

Thứ hai, bài kiểm thử này không ổn định.
Một trăm millisecond là một khoảng thời gian dài trong thế giới máy tính,
nhưng trên một hệ thống CI (continuous integration) bị quá tải
không hiếm khi thấy các khoảng dừng dài hơn nhiều so với đó.
Bài kiểm thử này có thể sẽ vượt qua nhất quán trên máy trạm của lập trình viên,
nhưng tôi sẽ không ngạc nhiên khi thấy những thất bại lẻ tẻ trong hệ thống CI.

## Chậm hay không ổn định: Chọn cả hai

Các bài kiểm thử dùng thời gian thực luôn chậm hoặc không ổn định.
Thường thì cả hai.
Nếu bài kiểm thử chờ lâu hơn cần thiết, nó chậm.
Nếu nó không chờ đủ lâu, nó không ổn định.
Bạn có thể làm bài kiểm thử chậm hơn và ít không ổn định hơn,
hoặc ít chậm hơn và không ổn định hơn,
nhưng bạn không thể làm nó vừa nhanh vừa đáng tin cậy.

Chúng tôi có rất nhiều bài kiểm thử trong gói `net/http` sử dụng cách tiếp cận này.
Tất cả đều chậm và/hoặc không ổn định, đó là điều đã đưa tôi đến
con đường dẫn chúng ta đến đây ngày hôm nay.

## Viết hàm đồng bộ?

Cách đơn giản nhất để kiểm thử một hàm bất đồng bộ là không làm vậy.
Hàm đồng bộ dễ kiểm thử hơn.
Nếu bạn có thể chuyển đổi một hàm bất đồng bộ thành đồng bộ,
nó sẽ dễ kiểm thử hơn.

Ví dụ, nếu chúng ta xem xét các hàm dọn dẹp cache của chúng ta từ trước đó,
`Cleanup` đồng bộ rõ ràng tốt hơn
`CleanupInBackground` bất đồng bộ.
Hàm đồng bộ dễ kiểm thử hơn,
và bên gọi có thể dễ dàng khởi động một goroutine mới để chạy nó ở nền khi cần.
Như một quy tắc chung,
bạn có thể đẩy tính đồng thời lên càng cao trong call stack,
thì càng tốt.

```
// CleanupInBackground is hard to test.
cache.CleanupInBackground()

// Cleanup is easy to test,
// and easy to run in the background when needed.
go cache.Cleanup()
```


Thật không may, loại chuyển đổi này không phải lúc nào cũng có thể thực hiện.
Ví dụ, `context.WithDeadline` là một API vốn dĩ bất đồng bộ.

## Công cụ hóa mã để có thể kiểm thử?

Một cách tiếp cận tốt hơn là làm cho mã của chúng ta có thể kiểm thử hơn.

Đây là một ví dụ về cách nó có thể trông như thế nào cho bài kiểm thử `WithDeadline` của chúng ta:

```
func TestWithDeadlineAfterDeadline(t *testing.T) {
    clock := fakeClock()
    timeout := 1 * time.Second
    deadline := clock.Now().Add(timeout)

    ctx, _ := context.WithDeadlineClock(
        t.Context(), deadline, clock)

    clock.Advance(timeout)
    context.WaitUntilIdle(ctx)
    if err := ctx.Err(); err != context.DeadlineExceeded {
        t.Fatalf("context not canceled after deadline")
    }
}
```

Thay vì dùng thời gian thực, chúng ta dùng một triển khai thời gian giả.
Dùng thời gian giả tránh các bài kiểm thử chậm không cần thiết,
vì chúng ta không bao giờ phải chờ đợi trong khi không làm gì.
Nó cũng giúp tránh sự không ổn định của bài kiểm thử,
vì thời gian hiện tại chỉ thay đổi khi bài kiểm thử điều chỉnh nó.

Có nhiều gói thời gian giả khác nhau,
hoặc bạn có thể tự viết.

Để dùng thời gian giả, chúng ta cần sửa đổi API để chấp nhận đồng hồ giả.
Tôi đã thêm hàm `context.WithDeadlineClock` ở đây,
nhận thêm một tham số đồng hồ:

```
ctx, _ := context.WithDeadlineClock(
    t.Context(), deadline, clock)
```

Khi chúng ta tiến đồng hồ giả, chúng ta gặp vấn đề.
Tiến thời gian là một thao tác bất đồng bộ.
Các goroutine đang ngủ có thể thức dậy,
các timer có thể gửi trên channel của chúng,
và các hàm timer có thể chạy.
Chúng ta cần chờ công việc đó hoàn thành trước khi có thể kiểm thử
hành vi mong đợi của hệ thống.

Tôi đã thêm hàm `context.WaitUntilIdle` ở đây,
chờ cho đến khi mọi công việc nền liên quan đến một context hoàn thành:

```
clock.Advance(timeout)
context.WaitUntilIdle(ctx)
```

Đây là một ví dụ đơn giản, nhưng nó minh họa
hai nguyên tắc cơ bản của việc viết mã đồng thời có thể kiểm thử:

1. Dùng thời gian giả (nếu bạn dùng thời gian).
2. Có một cách để chờ trạng thái ổn định (quiescence),
   đây là cách nói trang trọng của
   "mọi hoạt động nền đã dừng lại và hệ thống ổn định".

Câu hỏi thú vị, tất nhiên, là làm thế nào để thực hiện điều này.
Tôi đã lướt qua các chi tiết trong ví dụ này vì
có một số nhược điểm lớn đối với cách tiếp cận này.

Nó khó.
Dùng đồng hồ giả không khó,
nhưng xác định khi nào công việc đồng thời nền đã hoàn thành
và an toàn để kiểm tra trạng thái của hệ thống thì khó.

Mã của bạn trở nên kém đặc trưng hơn.
Bạn không thể dùng các hàm gói time chuẩn.
Bạn cần rất cẩn thận để theo dõi mọi thứ đang xảy ra
ở nền.

Bạn cần công cụ hóa không chỉ mã của bạn,
mà còn bất kỳ gói nào khác bạn dùng.
Nếu bạn gọi bất kỳ mã đồng thời bên thứ ba nào,
bạn có thể không có may mắn.

Tệ nhất là, có thể gần như không thể
áp dụng cách tiếp cận này vào một codebase hiện có.

Tôi đã cố gắng áp dụng cách tiếp cận này cho triển khai HTTP của Go,
và trong khi tôi đã có một số thành công ở một số nơi,
HTTP/2 server đơn giản là đánh bại tôi.
Đặc biệt, việc thêm công cụ để phát hiện trạng thái ổn định
mà không cần viết lại rộng rãi đã tỏ ra không khả thi,
hoặc ít nhất là vượt quá kỹ năng của tôi.

## Các hack runtime kinh khủng?

Chúng ta làm gì nếu không thể làm cho mã của mình có thể kiểm thử?

Nếu thay vì công cụ hóa mã của mình,
chúng ta có một cách để quan sát hành vi của hệ thống chưa được công cụ hóa thì sao?

Một chương trình Go bao gồm một tập hợp các goroutine.
Các goroutine đó có các trạng thái.
Chúng ta chỉ cần chờ cho đến khi tất cả các goroutine dừng chạy.

Thật không may, Go runtime không cung cấp bất kỳ cách nào để biết
những goroutine đó đang làm gì. Hay có không?

Gói `runtime` chứa một hàm cho chúng ta stack trace
cho mọi goroutine đang chạy, cũng như trạng thái của chúng.
Đây là văn bản dành cho con người đọc,
nhưng chúng ta có thể phân tích đầu ra đó.
Liệu chúng ta có thể dùng điều này để phát hiện trạng thái ổn định không?

Bây giờ, tất nhiên đây là một ý tưởng tồi.
Không có gì đảm bảo rằng định dạng của các stack trace này sẽ ổn định theo thời gian.
Bạn không nên làm điều này.

Tôi đã làm.
Và nó hoạt động.
Thật ra, nó hoạt động đáng ngạc nhiên.

Với một triển khai đơn giản của đồng hồ giả,
một lượng nhỏ công cụ để theo dõi những goroutine nào là một phần của bài kiểm thử,
và một số lạm dụng kinh khủng của `runtime.Stack`,
cuối cùng tôi đã có cách để viết các bài kiểm thử nhanh, đáng tin cậy cho gói `http`.

Triển khai cơ bản của các bài kiểm thử này thật khủng khiếp,
nhưng nó cho thấy rằng có một khái niệm hữu ích ở đây.

## Một cách tốt hơn

Go có tính đồng thời tích hợp sẵn,
nhưng kiểm thử các chương trình dùng tính đồng thời đó là khó.

Chúng ta đối mặt với một lựa chọn đáng tiếc:
Chúng ta có thể viết mã đơn giản, đặc trưng, nhưng sẽ không thể kiểm thử nhanh và đáng tin cậy;
hoặc chúng ta có thể viết mã có thể kiểm thử, nhưng nó sẽ phức tạp và không đặc trưng.

Vì vậy chúng tôi đã tự hỏi mình có thể làm gì để cải thiện điều này.

Như chúng ta đã thấy trước đó, hai tính năng cơ bản cần thiết để viết mã đồng thời có thể kiểm thử là
thời gian giả và một cách để chờ trạng thái ổn định.

Chúng ta cần một cách tốt hơn để chờ trạng thái ổn định.
Chúng ta nên có thể hỏi runtime khi nào các goroutine nền đã hoàn thành công việc của chúng.
Chúng ta cũng muốn có thể giới hạn phạm vi của truy vấn này đối với một bài kiểm thử duy nhất,
để các bài kiểm thử không liên quan không can thiệp với nhau.

Chúng ta cũng cần hỗ trợ tốt hơn để kiểm thử các chương trình dùng thời gian giả.

Không khó để tạo ra một triển khai thời gian giả,
nhưng mã dùng một triển khai như thế này không phải là đặc trưng.

Mã đặc trưng sẽ dùng `time.Timer`,
nhưng không thể tạo ra một `Timer` giả.
Chúng tôi đã hỏi chính mình liệu chúng tôi có nên cung cấp cách để các bài kiểm thử
tạo `Timer` giả không, nơi bài kiểm thử kiểm soát khi nào timer kích hoạt.

Một triển khai kiểm thử của thời gian cần định nghĩa một phiên bản hoàn toàn mới của gói `time`,
và truyền nó cho mọi hàm hoạt động trên thời gian.
Chúng tôi đã xem xét liệu chúng tôi có nên định nghĩa một interface thời gian chung không,
theo cách tương tự như `net.Conn` là interface chung mô tả kết nối mạng.

Tuy nhiên, điều chúng tôi nhận ra là không giống như kết nối mạng,
chỉ có một triển khai có thể của thời gian giả.
Mạng giả có thể muốn giới thiệu độ trễ hoặc lỗi.
Thời gian, ngược lại, chỉ làm một điều: Nó tiến về phía trước.
Các bài kiểm thử cần kiểm soát tốc độ thời gian tiến,
nhưng một timer được lên lịch kích hoạt mười giây trong tương lai
luôn nên kích hoạt mười (có thể giả) giây trong tương lai.

Ngoài ra, chúng tôi không muốn làm xáo trộn toàn bộ hệ sinh thái Go.
Hầu hết các chương trình ngày nay dùng các hàm trong gói time.
Chúng tôi muốn giữ cho những chương trình đó không chỉ hoạt động,
mà còn đặc trưng.

Điều này dẫn đến kết luận rằng những gì chúng ta cần là một cách để bài kiểm thử
yêu cầu gói time dùng đồng hồ giả,
gần giống như cách Go playground dùng đồng hồ giả.
Khác với playground,
chúng ta cần giới hạn phạm vi của thay đổi đó đối với một bài kiểm thử duy nhất.
(Có thể không rõ ràng rằng Go playground dùng đồng hồ giả,
vì chúng tôi biến các độ trễ giả thành độ trễ thực ở front end,
nhưng nó có.)

## Thử nghiệm `synctest`

Và vì vậy trong Go 1.24 chúng tôi đã giới thiệu [`testing/synctest`](/pkg/testing/synctest),
một gói thử nghiệm mới để đơn giản hóa việc kiểm thử các chương trình đồng thời.
Trong những tháng sau khi phát hành Go 1.24,
chúng tôi đã thu thập phản hồi từ những người dùng sớm.
(Cảm ơn tất cả mọi người đã thử!)
Chúng tôi đã thực hiện một số thay đổi để giải quyết các vấn đề và hạn chế.
Và bây giờ, trong Go 1.25, chúng tôi đã phát hành gói `testing/synctest`
như một phần của thư viện chuẩn.

Nó cho phép bạn chạy một hàm trong cái mà chúng tôi gọi là "bubble" (bong bóng).
Trong bubble, gói time dùng đồng hồ giả,
và gói `synctest` cung cấp một hàm để chờ bubble đạt trạng thái ổn định.

## Gói `synctest`

Gói `synctest` chứa chỉ hai hàm.

```
package synctest

// Test executes f in a new bubble.
// Goroutines in the bubble use a fake clock.
func Test(t *testing.T, f func(*testing.T))

// Wait waits for background activity in the bubble to complete.
func Wait()
```

[`Test`](/pkg/testing/synctest#Test) thực thi một hàm trong một bubble mới.

[`Wait`](/pkg/testing/synctest#Wait) chặn cho đến khi mọi goroutine trong bubble bị chặn
chờ một goroutine khác trong bubble.
Chúng ta gọi trạng thái đó là "bị chặn vĩnh cửu".

## Kiểm thử với synctest

Hãy xem một ví dụ về synctest trong hành động.

```
func TestWithDeadlineAfterDeadline(t *testing.T) {
    synctest.Test(t, func(t *testing.T) {
        deadline := time.Now().Add(1 * time.Second)
        ctx, _ := context.WithDeadline(t.Context(), deadline)

        time.Sleep(time.Until(deadline))
        synctest.Wait()
        if err := ctx.Err(); err != context.DeadlineExceeded {
            t.Fatalf("context not canceled after deadline")
        }
    })
}
```

Điều này có vẻ quen thuộc một chút.
Đây là bài kiểm thử ngây thơ cho `context.WithDeadline` mà chúng ta đã xem trước đó.
Những thay đổi duy nhất là chúng ta đã bọc bài kiểm thử trong
một lời gọi `synctest.Test` để thực thi nó trong một bubble
và chúng ta đã thêm một lời gọi `synctest.Wait`.

Bài kiểm thử này nhanh và đáng tin cậy.
Nó chạy gần như ngay lập tức.
Nó kiểm thử chính xác hành vi mong đợi của hệ thống được kiểm thử.
Nó cũng không yêu cầu sửa đổi gói `context`.

Dùng gói `synctest`,
chúng ta có thể viết mã đơn giản, đặc trưng
và kiểm thử nó một cách đáng tin cậy.

Đây là một ví dụ rất đơn giản, tất nhiên,
nhưng đây là bài kiểm thử thực sự của mã production thực sự.
Nếu `synctest` tồn tại khi gói `context` được viết,
chúng tôi đã có thể viết bài kiểm thử cho nó dễ dàng hơn nhiều.

## Thời gian

Thời gian trong bubble hoạt động gần giống thời gian giả trong Go playground.
Thời gian bắt đầu lúc nửa đêm, ngày 1 tháng 1 năm 2000 UTC.
Nếu bạn cần chạy một bài kiểm thử ở một thời điểm cụ thể vì một lý do nào đó,
bạn chỉ cần ngủ đến đó.

```
func TestAtSpecificTime(t *testing.T) {
   synctest.Test(t, func(t *testing.T) {
       // 2000-01-01 00:00:00 +0000 UTC
       t.Log(time.Now().In(time.UTC))

       // This does not take 25 years.
       time.Sleep(time.Until(
           time.Date(2025, 1, 1, 0, 0, 0, 0, time.UTC)))

       // 2025-01-01 00:00:00 +0000 UTC
       t.Log(time.Now().In(time.UTC))
   })
}
```

Thời gian chỉ trôi khi mọi goroutine trong bubble đã bị chặn.
Bạn có thể nghĩ bubble như một máy tính vô hạn nhanh:
Bất kỳ lượng tính toán nào cũng không mất thời gian.

Bài kiểm thử sau đây sẽ luôn in rằng không giây
thời gian giả nào đã trôi qua kể từ đầu bài kiểm thử,
bất kể bao nhiêu thời gian thực đã trôi qua.

```
func TestExpensiveWork(t *testing.T) {
   synctest.Test(t, func(t *testing.T) {
       start := time.Now()
       for range 1e7 {
           // do expensive work
       }
       t.Log(time.Since(start)) // 0s
   })
}
```

Trong bài kiểm thử tiếp theo, lời gọi `time.Sleep` sẽ trả về ngay lập tức,
thay vì chờ mười giây thực.
Bài kiểm thử sẽ luôn in rằng đúng mười giây giả
đã trôi qua kể từ đầu bài kiểm thử.

```
func TestSleep(t *testing.T) {
   synctest.Test(t, func(t *testing.T) {
       start := time.Now()
       time.Sleep(10 * time.Second)
       t.Log(time.Since(start)) // 10s
   })
}
```

## Chờ trạng thái ổn định

Hàm [`synctest.Wait`](/pkg/testing/synctest#Wait)
cho phép chúng ta chờ hoạt động nền hoàn thành.

```
func TestWait(t *testing.T) {
   synctest.Test(t, func(t *testing.T) {
       done := false
       go func() {
           done = true
       }()

       // Wait for the above goroutine to finish.
       synctest.Wait()

       t.Log(done) // true
   })
}
```

Nếu chúng ta không có lời gọi `Wait` trong bài kiểm thử trên,
chúng ta sẽ có race condition:
Một goroutine sửa đổi biến `done`
trong khi goroutine khác đọc nó mà không đồng bộ hóa.
Lời gọi `Wait` cung cấp sự đồng bộ hóa đó.

Bạn có thể quen thuộc với cờ kiểm thử `-race`,
kích hoạt bộ phát hiện data race.
Bộ phát hiện race biết về sự đồng bộ hóa được cung cấp bởi `Wait`,
và không phàn nàn về bài kiểm thử này.
Nếu chúng ta quên lời gọi `Wait`, bộ phát hiện race sẽ đúng đắn phàn nàn.

Hàm `synctest.Wait` cung cấp sự đồng bộ hóa,
nhưng sự trôi qua của thời gian thì không.

Trong ví dụ tiếp theo, một goroutine ghi vào biến `done`
trong khi goroutine khác ngủ một nanosecond trước khi đọc nó.
Rõ ràng là khi chạy với đồng hồ thực bên ngoài bubble synctest,
mã này chứa race condition.
Bên trong bubble synctest,
dù đồng hồ giả đảm bảo goroutine hoàn thành trước khi `time.Sleep` trả về,
bộ phát hiện race vẫn sẽ báo cáo data race,
giống như khi mã này chạy bên ngoài bubble synctest.

```
func TestTimeDataRace(t *testing.T) {
   synctest.Test(t, func(t *testing.T) {
       done := false
       go func() {
           done = true // write
       }()

       time.Sleep(1 * time.Nanosecond)

       t.Log(done)     // read (unsynchronized)
   })
}
```


Thêm lời gọi `Wait` cung cấp đồng bộ hóa rõ ràng và sửa data race:

```
time.Sleep(1 * time.Nanosecond)
synctest.Wait() // synchronize
t.Log(done)     // read
```

## Ví dụ: `io.Copy`

Tận dụng sự đồng bộ hóa được cung cấp bởi `synctest.Wait` cho phép chúng ta
viết các bài kiểm thử đơn giản hơn với ít đồng bộ hóa rõ ràng hơn.

Ví dụ, hãy xem xét bài kiểm thử này của [`io.Copy`](/pkg/io#Copy).

```
func TestIOCopy(t *testing.T) {
   synctest.Test(t, func(t *testing.T) {
       srcReader, srcWriter := io.Pipe()
       defer srcWriter.Close()

       var dst bytes.Buffer
       go io.Copy(&dst, srcReader)

       data := "1234"
       srcWriter.Write([]byte("1234"))
       synctest.Wait()

       if got, want := dst.String(), data; got != want {
           t.Errorf("Copy wrote %q, want %q", got, want)
       }
   })
}
```

Hàm `io.Copy` sao chép dữ liệu từ `io.Reader` sang `io.Writer`.
Bạn có thể không ngay lập tức nghĩ đến `io.Copy` như một hàm đồng thời,
vì nó chặn cho đến khi việc sao chép hoàn thành.
Tuy nhiên, cung cấp dữ liệu cho reader của `io.Copy` là một thao tác bất đồng bộ:

- `Copy` gọi phương thức `Read` của reader;
- `Read` trả về một số dữ liệu;
- và dữ liệu được ghi vào writer sau đó.

Trong bài kiểm thử này, chúng ta đang xác minh rằng `io.Copy` ghi dữ liệu mới vào writer
mà không cần chờ điền đầy buffer.

Xem qua bài kiểm thử từng bước,
trước tiên chúng ta tạo một `io.Pipe` để làm nguồn `io.Copy` đọc từ:

```
srcReader, srcWriter := io.Pipe()
defer srcWriter.Close()
```

Chúng ta gọi `io.Copy` trong một goroutine mới,
sao chép từ đầu đọc của pipe vào một `bytes.Buffer`:

```
var dst bytes.Buffer
go io.Copy(&dst, srcReader)
```

Chúng ta ghi vào đầu kia của pipe,
và chờ `io.Copy` xử lý dữ liệu:

```
data := "1234"
srcWriter.Write([]byte("1234"))
synctest.Wait()
```

Cuối cùng, chúng ta xác minh rằng buffer đích chứa dữ liệu mong muốn:

```
if got, want := dst.String(), data; got != want {
    t.Errorf("Copy wrote %q, want %q", got, want)
}
```

Chúng ta không cần thêm mutex hoặc đồng bộ hóa khác xung quanh buffer đích,
vì `synctest.Wait` đảm bảo rằng nó không bao giờ được truy cập đồng thời.

Bài kiểm thử này minh họa một vài điểm quan trọng.

Ngay cả các hàm đồng bộ như `io.Copy`,
không thực hiện công việc nền bổ sung sau khi chúng trả về,
cũng có thể thể hiện các hành vi bất đồng bộ.

Dùng `synctest.Wait`, chúng ta có thể kiểm thử những hành vi đó.

Cũng lưu ý rằng bài kiểm thử này không làm việc với thời gian.
Nhiều hệ thống bất đồng bộ liên quan đến thời gian, nhưng không phải tất cả.

## Thoát bubble

Hàm `synctest.Test` chờ tất cả các goroutine trong bubble thoát
trước khi trả về.
Thời gian dừng tiến sau khi goroutine gốc (goroutine được khởi động bởi `Test`) trả về.

Trong ví dụ tiếp theo, `Test` chờ goroutine nền chạy và thoát
trước khi nó trả về:

```
func TestWaitForGoroutine(t *testing.T) {
    synctest.Test(t, func(t *testing.T) {
        go func() {
            // This runs before synctest.Test returns.
        }()
    })
}
```

Trong ví dụ này, chúng ta lên lịch một `time.AfterFunc` cho một thời điểm trong tương lai.
Goroutine gốc của bubble trả về trước khi đến thời điểm đó,
vì vậy `AfterFunc` không bao giờ chạy:

```
func TestDoNotWaitForTimer(t *testing.T) {
    synctest.Test(t, func(t *testing.T) {
        time.AfterFunc(1 * time.Nanosecond, func() {
            // This never runs.
        })
    })
}
```

Trong ví dụ tiếp theo, chúng ta khởi động một goroutine đang ngủ.
Goroutine gốc trả về và thời gian dừng tiến.
Bubble bây giờ bị deadlock,
vì `Test` đang chờ tất cả các goroutine trong bubble hoàn thành
nhưng goroutine đang ngủ đang chờ thời gian tiến.

```
func TestDeadlock(t *testing.T) {
    synctest.Test(t, func(t *testing.T) {
        go func() {
            // This sleep never returns and the test deadlocks.
            time.Sleep(1 * time.Nanosecond)
        }()
    })
}
```

## Deadlock

Gói `synctest` panic khi một bubble bị deadlock
do mọi goroutine trong bubble bị chặn vĩnh cửu
bởi một goroutine khác trong bubble.

```
--- FAIL: Test (0.00s)
--- FAIL: TestDeadlock (0.00s)
panic: deadlock: main bubble goroutine has exited but blocked goroutines remain [recovered, repanicked]

goroutine 7 [running]:
(stacks elided for clarity)

goroutine 10 [sleep (durable), synctest bubble 1]:
time.Sleep(0x1)
	/Users/dneil/src/go/src/runtime/time.go:361 +0x130
_.TestDeadlock.func1.1()
	/tmp/s/main_test.go:13 +0x20
created by _.TestDeadlock.func1 in goroutine 9
	/tmp/s/main_test.go:11 +0x24
FAIL	_	0.173s
FAIL
```

Runtime sẽ in stack trace cho mọi goroutine trong bubble bị deadlock.

Khi in trạng thái của một goroutine trong bubble,
runtime chỉ ra khi goroutine bị chặn vĩnh cửu.
Bạn có thể thấy rằng goroutine đang ngủ trong bài kiểm thử này bị chặn vĩnh cửu.

## Chặn vĩnh cửu

"Bị chặn vĩnh cửu" là một khái niệm cốt lõi trong synctest.

Một goroutine bị chặn vĩnh cửu khi nó không chỉ bị chặn,
mà còn khi nó chỉ có thể được bỏ chặn bởi một goroutine khác trong cùng bubble.

Khi mọi goroutine trong một bubble bị chặn vĩnh cửu:

1. `synctest.Wait` trả về.
2. Nếu không có lời gọi `synctest.Wait` đang xử lý,
   thời gian giả tiến ngay lập tức đến điểm tiếp theo sẽ đánh thức một goroutine.
3. Nếu không có goroutine nào có thể được đánh thức bằng cách tiến thời gian,
   bubble bị deadlock và bài kiểm thử thất bại.

Điều quan trọng là chúng ta phải phân biệt giữa một goroutine chỉ bị chặn
và một goroutine bị *chặn vĩnh cửu*.
Chúng ta không muốn khai báo deadlock khi một goroutine tạm thời bị chặn
bởi một sự kiện phát sinh bên ngoài bubble của nó.

Hãy xem xét một số cách mà một goroutine có thể bị chặn không vĩnh cửu.

### Không chặn vĩnh cửu: I/O (tệp, pipe, kết nối mạng, v.v.)

Giới hạn quan trọng nhất là I/O không chặn vĩnh cửu,
bao gồm cả I/O mạng.
Một goroutine đọc từ kết nối mạng có thể bị chặn,
nhưng nó sẽ được bỏ chặn bởi dữ liệu đến trên kết nối đó.

Điều này rõ ràng đúng với kết nối đến một dịch vụ mạng nào đó,
nhưng nó cũng đúng với kết nối loopback,
ngay cả khi cả reader và writer đều ở trong cùng bubble.

Khi chúng ta ghi dữ liệu vào một socket mạng,
ngay cả socket loopback,
dữ liệu được truyền đến kernel để phân phối.
Có một khoảng thời gian giữa việc lời gọi hệ thống ghi trả về
và kernel thông báo cho phía kia của kết nối rằng dữ liệu có sẵn.
Go runtime không thể phân biệt giữa một goroutine bị chặn chờ
dữ liệu đã có trong buffer của kernel
và một goroutine bị chặn chờ dữ liệu sẽ không đến.

Điều này có nghĩa là các bài kiểm thử của các chương trình mạng dùng synctest
thường không thể dùng kết nối mạng thực.
Thay vào đó, chúng nên dùng một mạng giả trong bộ nhớ.

Tôi sẽ không đi qua quá trình tạo mạng giả ở đây,
nhưng tài liệu gói `synctest` chứa
[một ví dụ hoàn chỉnh đã được làm việc](/pkg/testing/synctest#hdr-Example__HTTP_100_Continue)
về kiểm thử HTTP client và server giao tiếp qua mạng giả.

### Không chặn vĩnh cửu: syscall, cgo call, bất cứ thứ gì không phải Go

Syscall và cgo call không chặn vĩnh cửu.
Chúng ta chỉ có thể suy luận về trạng thái của các goroutine thực thi mã Go.

### Không chặn vĩnh cửu: Mutex

Có lẽ gây ngạc nhiên, mutex không chặn vĩnh cửu.
Đây là quyết định xuất phát từ thực tế:
Mutex thường được dùng để bảo vệ trạng thái toàn cục,
vì vậy một goroutine trong bubble thường cần thu thập một mutex được giữ bên ngoài bubble của nó.
Mutex rất nhạy cảm về hiệu năng,
vì vậy thêm công cụ bổ sung vào chúng
có nguy cơ làm chậm các chương trình không kiểm thử.

Chúng ta có thể kiểm thử các chương trình dùng mutex với synctest,
nhưng đồng hồ giả sẽ không tiến trong khi một goroutine đang bị chặn khi thu thập mutex.
Điều này chưa gây ra vấn đề trong bất kỳ trường hợp nào chúng tôi đã gặp,
nhưng đây là điều cần lưu ý.

### Chặn vĩnh cửu: `time.Sleep`

Vậy điều gì là chặn vĩnh cửu?

`time.Sleep` rõ ràng là vĩnh cửu,
vì thời gian chỉ có thể tiến khi mọi goroutine trong bubble bị chặn vĩnh cửu.

### Chặn vĩnh cửu: gửi hoặc nhận trên channel được tạo trong cùng bubble

Các thao tác trên channel được tạo trong cùng bubble là vĩnh cửu.

Chúng ta phân biệt giữa channel thuộc bubble (được tạo trong bubble)
và channel không thuộc bubble (được tạo bên ngoài bất kỳ bubble nào).
Điều này có nghĩa là một hàm dùng channel toàn cục để đồng bộ hóa,
ví dụ để kiểm soát quyền truy cập vào tài nguyên được cache toàn cục,
có thể được gọi an toàn từ trong bubble.

Cố gắng thao tác trên channel thuộc bubble từ bên ngoài bubble của nó là lỗi.

### Chặn vĩnh cửu: `sync.WaitGroup` thuộc cùng bubble

Chúng ta cũng liên kết `sync.WaitGroup` với các bubble.

`WaitGroup` không có constructor,
vì vậy chúng ta tạo ra sự liên kết với bubble ngầm định ở lời gọi đầu tiên đến `Go` hoặc `Add`.

Cũng như với channel,
chờ đợi trên `WaitGroup` thuộc cùng bubble là chặn vĩnh cửu,
và chờ trên một cái từ bên ngoài bubble thì không.
Gọi `Go` hoặc `Add` trên `WaitGroup` thuộc một bubble khác là lỗi.

### Chặn vĩnh cửu: `sync.Cond.Wait`

Chờ trên `sync.Cond` luôn là chặn vĩnh cửu.
Đánh thức một goroutine đang chờ trên `Cond` trong một bubble khác là lỗi.

### Chặn vĩnh cửu: `select{}`

Cuối cùng, một select rỗng là chặn vĩnh cửu.
(Một select có các nhánh là chặn vĩnh cửu nếu tất cả các thao tác trong nó đều như vậy.)

Đó là danh sách đầy đủ các thao tác chặn vĩnh cửu.
Nó không dài lắm,
nhưng đủ để xử lý hầu hết các chương trình thực tế.

Quy tắc là một goroutine bị chặn vĩnh cửu khi nó bị chặn,
và chúng ta có thể đảm bảo rằng nó chỉ có thể được bỏ chặn
bởi một goroutine khác trong bubble của nó.

Trong các trường hợp có thể cố gắng đánh thức một goroutine trong bubble từ bên ngoài bubble của nó,
chúng ta panic.
Ví dụ, thao tác trên channel thuộc bubble từ bên ngoài bubble của nó là lỗi.

## Thay đổi từ 1.24 đến 1.25

Chúng tôi đã phát hành phiên bản thử nghiệm của gói `synctest` trong Go 1.24.
Để đảm bảo những người dùng sớm biết về trạng thái thử nghiệm của gói,
bạn cần đặt cờ GOEXPERIMENT để làm cho gói hiển thị.

Phản hồi chúng tôi nhận được từ những người dùng sớm đó là vô cùng quý giá,
vừa để chứng minh rằng gói hữu ích
vừa để phát hiện các khu vực mà API cần sửa.

Đây là một số thay đổi được thực hiện giữa phiên bản thử nghiệm
và phiên bản được phát hành trong Go 1.25.

### Thay thế Run bằng Test

Phiên bản gốc của API tạo bubble bằng hàm `Run`:

```
// Run executes f in a new bubble.
func Run(f func())
```

Rõ ràng rằng chúng ta cần một cách để tạo `*testing.T`
trong phạm vi bubble.
Ví dụ, `t.Cleanup` nên chạy các hàm dọn dẹp trong cùng bubble
chúng được đăng ký, không phải sau khi bubble thoát.
Chúng tôi đã đổi tên `Run` thành `Test` và làm cho nó tạo ra `T` trong phạm vi thời gian sống
của bubble mới.

### Thời gian dừng khi goroutine gốc của bubble trả về

Ban đầu chúng tôi vẫn tiếp tục tiến thời gian trong bubble miễn là
bubble chứa bất kỳ goroutine nào đang chờ các sự kiện tương lai.
Điều này hóa ra rất khó hiểu khi một goroutine tồn tại lâu không bao giờ trả về,
chẳng hạn như một goroutine đọc mãi mãi từ `time.Ticker`.
Chúng tôi bây giờ dừng tiến thời gian khi goroutine gốc của bubble trả về.
Nếu bubble bị chặn chờ thời gian tiến,
điều này dẫn đến deadlock và panic có thể được phân tích.

### Đã loại bỏ các trường hợp "vĩnh cửu" không thực sự vĩnh cửu

Chúng tôi đã làm sạch định nghĩa về "chặn vĩnh cửu".
Triển khai gốc có các trường hợp mà một goroutine bị chặn vĩnh cửu có thể
bị bỏ chặn từ bên ngoài bubble.
Ví dụ, channel ghi lại liệu chúng có được tạo trong bubble hay không,
nhưng không ghi lại trong bubble nào chúng được tạo,
vì vậy một bubble có thể bỏ chặn channel trong một bubble khác.
Triển khai hiện tại không chứa các trường hợp nào chúng tôi biết
mà một goroutine bị chặn vĩnh cửu có thể bị bỏ chặn từ bên ngoài bubble của nó.

### Stack trace tốt hơn

Chúng tôi đã cải thiện thông tin được in trong stack trace.
Khi một bubble bị deadlock, theo mặc định chúng tôi bây giờ chỉ in stack cho các goroutine trong bubble đó.
Stack trace cũng chỉ ra rõ ràng những goroutine nào trong bubble bị chặn vĩnh cửu.

### Ngẫu nhiên hóa các sự kiện xảy ra cùng lúc

Chúng tôi đã cải thiện việc ngẫu nhiên hóa các sự kiện xảy ra cùng lúc.
Ban đầu, các timer được lên lịch kích hoạt vào cùng thời điểm
luôn làm vậy theo thứ tự chúng được tạo.
Thứ tự này bây giờ được ngẫu nhiên hóa.

## Công việc trong tương lai

Chúng tôi khá hài lòng với gói synctest hiện tại.

Ngoài các bản sửa lỗi không thể tránh khỏi,
chúng tôi hiện không dự kiến bất kỳ thay đổi lớn nào trong tương lai.
Tất nhiên, với việc áp dụng rộng rãi hơn, luôn có thể chúng tôi sẽ phát hiện ra điều gì đó
cần làm.

Một lĩnh vực công việc có thể là cải thiện việc phát hiện các goroutine bị chặn vĩnh cửu.
Sẽ thật tốt nếu chúng ta có thể làm cho các thao tác mutex chặn vĩnh cửu,
với hạn chế rằng một mutex được thu thập trong bubble phải được giải phóng
trong cùng bubble.

Kiểm thử mã mạng với synctest yêu cầu mạng giả.
Hàm `net.Pipe` có thể tạo ra `net.Conn` giả,
nhưng hiện tại không có hàm thư viện chuẩn nào tạo ra
`net.Listener` hoặc `net.PacketConn` giả.
Ngoài ra, `net.Conn` trả về bởi `net.Pipe` là đồng bộ, mỗi lần ghi chặn
cho đến khi một lần đọc tiêu thụ dữ liệu, điều này không đại diện cho hành vi mạng thực.
Có lẽ chúng ta nên thêm các triển khai giả tốt của các interface mạng phổ biến
vào thư viện chuẩn.

## Kết luận

Đó là gói `synctest`.

Tôi không thể nói rằng nó làm cho việc kiểm thử mã đồng thời đơn giản,
vì tính đồng thời không bao giờ đơn giản.
Điều nó làm là cho phép bạn viết mã đồng thời đơn giản nhất có thể,
dùng Go đặc trưng,
và gói time tiêu chuẩn,
và sau đó viết các bài kiểm thử nhanh, đáng tin cậy cho nó.

Tôi hy vọng bạn thấy nó hữu ích.
