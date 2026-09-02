---
title: Gói unique mới
date: 2024-08-27
by:
- Michael Knyszek
tags:
- interning
- unique
summary: Gói mới để interning trong Go 1.23.
---

Thư viện chuẩn của Go 1.23 hiện bao gồm [gói `unique` mới](https://pkg.go.dev/unique).
Mục đích của gói này là cho phép chuẩn hóa các giá trị có thể so sánh.
Nói cách khác, gói này cho phép bạn loại bỏ các giá trị trùng lặp sao cho chúng trỏ đến
một bản sao duy nhất, chuẩn tắc, trong khi hiệu quả quản lý các bản sao chuẩn tắc đó phía sau.
Bạn có thể đã quen với khái niệm này, được gọi là
["interning"](https://en.wikipedia.org/wiki/Interning_(computer_science)).
Hãy cùng tìm hiểu cách nó hoạt động và tại sao nó hữu ích.

## Một cài đặt đơn giản của interning

Ở mức độ cao, interning rất đơn giản.
Hãy xem đoạn code mẫu dưới đây, dùng một map thông thường để loại bỏ các chuỗi trùng lặp.

```
var internPool map[string]string

// Intern trả về một chuỗi bằng với s nhưng có thể chia sẻ bộ nhớ với
// một chuỗi đã được truyền vào Intern trước đó.
func Intern(s string) string {
	pooled, ok := internPool[s]
	if !ok {
		// Clone chuỗi phòng trường hợp nó là một phần của một chuỗi lớn hơn nhiều.
		// Điều này hiếm khi xảy ra nếu interning được sử dụng đúng cách.
		pooled = strings.Clone(s)
		internPool[pooled] = pooled
	}
	return pooled
}
```

Cách này hữu ích khi bạn đang xây dựng nhiều chuỗi có khả năng trùng lặp, chẳng hạn khi phân tích cú pháp một định dạng văn bản.

Cài đặt này rất đơn giản và hoạt động tốt trong một số trường hợp, nhưng nó có một vài vấn đề:

* Nó không bao giờ xóa chuỗi khỏi pool.
* Nó không thể được sử dụng an toàn bởi nhiều goroutine đồng thời.
* Nó chỉ hoạt động với chuỗi, mặc dù ý tưởng mang tính tổng quát hơn.

Cũng có một điểm bỏ lỡ trong cài đặt này, và nó khá tinh tế.
Bên trong, [các chuỗi là các cấu trúc bất biến gồm một con trỏ và một độ dài](/blog/slices).
Khi so sánh hai chuỗi, nếu các con trỏ không bằng nhau, chúng ta phải so sánh nội dung của chúng để xác định sự bằng nhau.
Nhưng nếu chúng ta biết rằng hai chuỗi đã được chuẩn hóa, thì *đủ* chỉ cần kiểm tra con trỏ của chúng.

## Gói `unique`

Gói `unique` mới giới thiệu một hàm tương tự `Intern` được gọi là
[`Make`](https://pkg.go.dev/unique#Make).

Nó hoạt động gần giống `Intern`.
Bên trong cũng có một map toàn cục ([một map đồng thời generic nhanh](https://pkg.go.dev/internal/concurrent@go1.23.0)) và `Make` tra cứu giá trị được cung cấp trong map đó.
Nhưng nó cũng khác với `Intern` ở hai điểm quan trọng.
Thứ nhất, nó chấp nhận giá trị của bất kỳ kiểu có thể so sánh nào.
Và thứ hai, nó trả về một giá trị bao bọc,
[`Handle[T]`](https://pkg.go.dev/unique#Handle), từ đó có thể lấy ra giá trị chuẩn tắc.

`Handle[T]` này là chìa khóa của thiết kế.
`Handle[T]` có thuộc tính là hai giá trị `Handle[T]` bằng nhau khi và chỉ khi các giá trị dùng để tạo ra chúng bằng nhau.
Hơn nữa, việc so sánh hai giá trị `Handle[T]` rẻ: nó quy về phép so sánh con trỏ.
So với việc so sánh hai chuỗi dài, điều đó rẻ hơn hàng bậc độ lớn!

Cho đến nay, đây là điều bạn không thể làm trong code Go thông thường.

Nhưng `Handle[T]` cũng có mục đích thứ hai: miễn là `Handle[T]` tồn tại cho một giá trị, map sẽ giữ lại bản sao chuẩn tắc của giá trị đó.
Khi tất cả các giá trị `Handle[T]` ánh xạ đến một giá trị cụ thể biến mất, gói đánh dấu mục nhập map nội bộ đó là có thể xóa, để được thu hồi trong tương lai gần.
Điều này thiết lập một chính sách rõ ràng về khi nào cần xóa các mục khỏi map: khi các mục chuẩn tắc không còn được sử dụng nữa, bộ gom rác có thể tự do dọn sạch chúng.

Nếu bạn đã từng dùng Lisp, tất cả điều này có thể nghe quen thuộc.
[Ký hiệu](https://en.wikipedia.org/wiki/Symbol_(programming)) trong Lisp là các chuỗi được intern, nhưng không phải là chuỗi thực sự, và tất cả giá trị chuỗi của các ký hiệu được đảm bảo nằm trong cùng một pool.
Mối quan hệ giữa ký hiệu và chuỗi song song với mối quan hệ giữa `Handle[string]` và `string`.

## Một ví dụ thực tế

Vậy, làm thế nào để sử dụng `unique.Make`?
Hãy nhìn vào gói `net/netip` trong thư viện chuẩn, nơi intern các giá trị kiểu `addrDetail`, là một phần của cấu trúc
[`netip.Addr`](https://pkg.go.dev/net/netip#Addr).

Dưới đây là phiên bản rút gọn của code thực tế từ `net/netip` sử dụng `unique`.

```
// Addr biểu diễn một địa chỉ IPv4 hoặc IPv6 (có hoặc không có vùng phạm vi địa chỉ),
// tương tự net.IP hoặc net.IPAddr.
type Addr struct {
	// Các trường unexported không liên quan khác...

	// Chi tiết về địa chỉ, gom lại và chuẩn hóa.
	z unique.Handle[addrDetail]
}

// addrDetail cho biết địa chỉ là IPv4 hay IPv6, và nếu là IPv6,
// chỉ định tên vùng cho địa chỉ.
type addrDetail struct {
	isV6   bool   // IPv4 là false, IPv6 là true.
	zoneV6 string // Có thể != "" nếu IsV6 là true.
}

var z6noz = unique.Make(addrDetail{isV6: true})

// WithZone trả về một IP giống ip nhưng với vùng được cung cấp.
// Nếu zone trống, vùng bị xóa. Nếu ip là địa chỉ IPv4,
// WithZone là no-op và trả về ip không thay đổi.
func (ip Addr) WithZone(zone string) Addr {
	if !ip.Is6() {
		return ip
	}
	if zone == "" {
		ip.z = z6noz
		return ip
	}
	ip.z = unique.Make(addrDetail{isV6: true, zoneV6: zone})
	return ip
}
```

Vì nhiều địa chỉ IP có thể dùng cùng một vùng và vùng này là một phần danh tính của chúng, việc chuẩn hóa chúng rất có ý nghĩa.
Việc loại bỏ các vùng trùng lặp giảm mức sử dụng bộ nhớ trung bình của mỗi `netip.Addr`, trong khi thực tế là chúng được chuẩn hóa có nghĩa là các giá trị `netip.Addr` được so sánh hiệu quả hơn, vì việc so sánh tên vùng trở thành một phép so sánh con trỏ đơn giản.

## Ghi chú về interning chuỗi

Mặc dù gói `unique` hữu ích, `Make` không hoàn toàn giống `Intern` cho chuỗi, vì `Handle[T]` cần thiết để giữ chuỗi không bị xóa khỏi map nội bộ.
Điều này có nghĩa là bạn cần sửa đổi code của mình để giữ cả handle lẫn chuỗi.

Nhưng chuỗi đặc biệt ở chỗ, mặc dù chúng hoạt động như các giá trị, thực tế chúng chứa con trỏ bên trong, như đã đề cập trước đó.
Điều này có nghĩa là chúng ta có thể tiềm năng chuẩn hóa chỉ bộ nhớ lưu trữ bên dưới của chuỗi, ẩn chi tiết của `Handle[T]` bên trong chuỗi chính nó.
Vì vậy, vẫn còn một chỗ trong tương lai cho cái tôi sẽ gọi là _interning chuỗi trong suốt_, trong đó chuỗi có thể được intern mà không cần kiểu `Handle[T]`, tương tự hàm `Intern` nhưng với ngữ nghĩa gần hơn với `Make`.

Trong thời gian đó, `unique.Make("my string").Value()` là một cách giải quyết có thể.
Dù việc không giữ handle sẽ cho phép chuỗi bị xóa khỏi map nội bộ của `unique`, các mục trong map không bị xóa ngay lập tức.
Trong thực tế, các mục sẽ không bị xóa cho đến ít nhất lần thu gom rác tiếp theo hoàn thành, vì vậy cách giải quyết này vẫn cho phép một mức độ loại bỏ trùng lặp nhất định trong các khoảng thời gian giữa các lần thu gom.

## Một chút lịch sử và nhìn về tương lai

Thực ra, gói `net/netip` đã intern các chuỗi vùng kể từ khi nó được giới thiệu lần đầu.
Gói interning mà nó sử dụng là một bản sao nội bộ của gói
[go4.org/intern](https://pkg.go.dev/go4.org/intern).
Giống như gói `unique`, nó có kiểu `Value` (trông rất giống `Handle[T]`, trước khi có generics), và có thuộc tính đáng chú ý là các mục trong map nội bộ bị xóa khi handle của chúng không còn được tham chiếu.

Nhưng để đạt được hành vi này, nó phải làm một số điều không an toàn.
Cụ thể, nó đưa ra một số giả định về hành vi của bộ gom rác để cài đặt [_con trỏ yếu_](https://en.wikipedia.org/wiki/Weak_reference) bên ngoài runtime.
Con trỏ yếu là con trỏ không ngăn bộ gom rác thu hồi một biến; khi điều này xảy ra, con trỏ tự động trở thành nil.
Thực ra, con trỏ yếu _cũng_ là trừu tượng cốt lõi cơ bản của gói `unique`.

Đúng vậy: trong quá trình cài đặt gói `unique`, chúng tôi đã thêm hỗ trợ con trỏ yếu thích hợp vào bộ gom rác.
Và sau khi lội qua mê cung các quyết định thiết kế đáng tiếc đi kèm với con trỏ yếu (như, con trỏ yếu có nên theo dõi [sự phục sinh đối tượng](https://en.wikipedia.org/wiki/Object_resurrection) không? Không!), chúng tôi ngạc nhiên khi thấy tất cả mọi thứ trở nên đơn giản và rõ ràng như thế nào.
Ngạc nhiên đến mức con trỏ yếu hiện là một [đề xuất công khai](/issue/67552).

Công việc này cũng dẫn chúng tôi xem xét lại finalizer, dẫn đến một đề xuất khác cho [phương thức thay thế finalizer](/issue/67535) dễ sử dụng hơn và hiệu quả hơn.
Với [một hàm băm cho các giá trị có thể so sánh](/issue/54670) đang trên đường tới, tương lai của việc [xây dựng bộ nhớ cache hiệu quả](/issue/67552#issuecomment-2200755798) trong Go rất sáng sủa!
