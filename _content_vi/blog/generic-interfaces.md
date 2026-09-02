---
title: Interface generic
date: 2025-07-07
by:
- Axel Wagner
tags:
- type parameters
- generics
- interfaces
summary: Thêm tham số kiểu vào các kiểu interface mang lại sức mạnh đáng ngạc nhiên
template: true
---

Có một ý tưởng không hiển nhiên cho đến khi bạn nghe về nó lần đầu: vì interface tự thân cũng là kiểu, chúng cũng có thể có tham số kiểu.
Ý tưởng này tỏ ra mạnh mẽ một cách đáng ngạc nhiên khi dùng để biểu đạt các ràng buộc trên các hàm và kiểu generic.
Trong bài viết này, chúng ta sẽ minh họa điều đó bằng cách thảo luận về việc sử dụng interface với tham số kiểu trong một vài kịch bản phổ biến.

## Một tập hợp cây đơn giản

Làm ví dụ khởi đầu, giả sử chúng ta cần một phiên bản generic của một [cây tìm kiếm nhị phân](https://en.wikipedia.org/wiki/Binary_search_tree).
Các phần tử được lưu trong một cây như vậy cần có thứ tự, vì vậy tham số kiểu của chúng ta cần một ràng buộc xác định thứ tự cần sử dụng.
Một lựa chọn đơn giản là sử dụng ràng buộc [cmp.Ordered](/pkg/cmp#Ordered), được giới thiệu trong Go 1.21.
Nó giới hạn một tham số kiểu ở các kiểu có thứ tự (chuỗi và số) và cho phép các phương thức của kiểu sử dụng các toán tử sắp xếp tích hợp.

{{raw `
    // Giá trị zero của Tree là một cây rỗng sẵn sàng để sử dụng.
    type Tree[E cmp.Ordered] struct {
        root *node[E]
    }

    func (t *Tree[E]) Insert(element E) {
        t.root = t.root.insert(element)
    }

    type node[E cmp.Ordered] struct {
        value E
        left  *node[E]
        right *node[E]
    }

    func (n *node[E]) insert(element E) *node[E] {
        if n == nil {
            return &node[E]{value: element}
        }
        switch {
        case element < n.value:
            n.left = n.left.insert(element)
        case element > n.value:
            n.right = n.right.insert(element)
        }
        return n
    }
`}}

([playground](/play/p/H7-n33X7P2h))

Tuy nhiên, cách tiếp cận này có nhược điểm là nó chỉ hoạt động với các kiểu cơ bản mà `<code>&lt;</code>` được định nghĩa;
bạn không thể chèn các kiểu struct, như [time.Time](/pkg/time#Time).

Chúng ta có thể khắc phục điều đó bằng cách yêu cầu người dùng cung cấp một hàm so sánh:

{{raw `
    // FuncTree phải được tạo bằng NewFuncTree.
    type FuncTree[E any] struct {
        root *funcNode[E]
        cmp  func(E, E) int
    }

    func NewFuncTree[E any](cmp func(E, E) int) *FuncTree[E] {
        return &FuncTree[E]{cmp: cmp}
    }

    func (t *FuncTree[E]) Insert(element E) {
        t.root = t.root.insert(t.cmp, element)
    }

    type funcNode[E any] struct {
        value E
        left  *funcNode[E]
        right *funcNode[E]
    }

    func (n *funcNode[E]) insert(cmp func(E, E) int, element E) *funcNode[E] {
        if n == nil {
            return &funcNode[E]{value: element}
        }
        sign := cmp(element, n.value)
        switch {
        case sign < 0:
            n.left = n.left.insert(cmp, element)
        case sign > 0:
            n.right = n.right.insert(cmp, element)
        }
        return n
    }
`}}

([playground](/play/p/tiEjuxCHtFF))

Cách này hoạt động, nhưng cũng đi kèm với các nhược điểm.
Chúng ta không còn có thể sử dụng giá trị zero của kiểu container nữa, vì nó cần có một hàm so sánh được khởi tạo rõ ràng.
Và việc sử dụng một trường hàm khiến trình biên dịch khó inline các lời gọi so sánh hơn, điều này có thể gây ra chi phí đáng kể khi chạy.

Sử dụng một phương thức trên kiểu phần tử có thể giải quyết các vấn đề này, vì các phương thức được liên kết trực tiếp với một kiểu.
Một phương thức không cần phải được truyền vào một cách rõ ràng và trình biên dịch có thể thấy đích của lời gọi, đồng thời có thể inline nó.
Nhưng làm thế nào chúng ta có thể biểu đạt ràng buộc để yêu cầu các kiểu phần tử cung cấp phương thức cần thiết?

## Sử dụng receiver trong các ràng buộc

Cách tiếp cận đầu tiên mà chúng ta có thể thử là định nghĩa một interface thông thường có phương thức `Compare`:

{{raw `
    type Comparer interface {
        Compare(Comparer) int
    }
`}}

Tuy nhiên, chúng ta nhanh chóng nhận ra rằng cách này không hoạt động tốt.
Để triển khai interface này, tham số của phương thức cũng phải chính là `Comparer`.
Điều đó không chỉ có nghĩa là phần triển khai của phương thức này phải ép kiểu tham số về chính kiểu của nó, mà còn yêu cầu mọi kiểu phải tham chiếu rõ ràng đến gói của chúng ta bằng tên kiểu `Comparer` (nếu không, chữ ký phương thức sẽ không giống hệt nhau).
Điều đó không thực sự trực giao.

Một cách tiếp cận tốt hơn là làm cho chính interface `Comparer` trở thành generic:

{{raw `
    type Comparer[T any] interface {
        Compare(T) int
    }
`}}

`Comparer` này giờ mô tả cả một họ interface, mỗi interface tương ứng với một kiểu mà `Comparer` có thể được khởi tạo.
Một kiểu triển khai `Comparer[T]` khai báo rằng "Tôi có thể so sánh bản thân mình với một `T`".
Ví dụ, `time.Time` tự nhiên triển khai `Comparer[time.Time]` vì [nó có phương thức `Compare` tương ứng](/pkg/time#Time.Compare):

{{raw `
    // Triển khai Comparer[Time]
    func (t Time) Compare(u Time) int
`}}

Cách này tốt hơn, nhưng vẫn chưa đủ.
Điều chúng ta thực sự muốn là một ràng buộc cho biết tham số kiểu có thể được so sánh với *chính nó*: chúng ta muốn ràng buộc đó có tính tự tham chiếu.
Điểm tinh tế ở đây là khía cạnh tự tham chiếu không cần phải là một phần của chính định nghĩa interface; cụ thể, ràng buộc cho `T` trong kiểu `Comparer` chỉ là `any`.
Thay vào đó, nó là hệ quả của cách chúng ta sử dụng `Comparer` làm ràng buộc cho tham số kiểu của `MethodTree`:

{{raw `
    // Giá trị zero của MethodTree là một cây rỗng sẵn sàng sử dụng.
    type MethodTree[E Comparer[E]] struct {
        root *methodNode[E]
    }

    func (t *MethodTree[E]) Insert(element E) {
        t.root = t.root.insert(element)
    }

    type methodNode[E Comparer[E]] struct {
        value E
        left  *methodNode[E]
        right *methodNode[E]
    }

    func (n *methodNode[E]) insert(element E) *methodNode[E] {
        if n == nil {
            return &methodNode[E]{value: element}
        }
        sign := element.Compare(n.value)
        switch {
        case sign < 0:
            n.left = n.left.insert(element)
        case sign > 0:
            n.right = n.right.insert(element)
        }
        return n
    }
`}}

([playground](/play/p/LuhzYej_2SP))

Vì `time.Time` triển khai `Comparer[time.Time]`, nó giờ là một đối số kiểu hợp lệ cho bộ chứa này, và chúng ta vẫn có thể sử dụng giá trị zero làm một bộ chứa rỗng:

{{raw `
    var t MethodTree[time.Time]
    t.Insert(time.Now())
`}}

Để có đầy đủ tính linh hoạt, một thư viện có thể cung cấp cả ba phiên bản API.
Nếu muốn giảm sự lặp lại, tất cả các phiên bản có thể sử dụng một phần triển khai dùng chung.
Chúng ta có thể dùng phiên bản hàm cho việc đó, vì nó tổng quát nhất:

{{raw `
    type node[E any] struct {
        value E
        left  *node[E]
        right *node[E]
    }

    func (n *node[E]) insert(cmp func(E, E) int, element E) *node[E] {
        if n == nil {
            return &node[E]{value: element}
        }
        sign := cmp(element, n.value)
        switch {
        case sign < 0:
            n.left = n.left.insert(cmp, element)
        case sign > 0:
            n.right = n.right.insert(cmp, element)
        }
        return n
    }

    // Insert chèn phần tử vào cây, nếu E triển khai cmp.Ordered.
    func (t *Tree[E]) Insert(element E) {
        t.root = t.root.insert(cmp.Compare[E], element)
    }

    // Insert chèn phần tử vào cây, sử dụng hàm so sánh được cung cấp.
    func (t *FuncTree[E]) Insert(element E) {
        t.root = t.root.insert(t.cmp, element)
    }

    // Insert chèn phần tử vào cây, nếu E triển khai Comparer[E].
    func (t *MethodTree[E]) Insert(element E) {
        t.root = t.root.insert(E.Compare, element)
    }
`}}

([playground](/play/p/jzmoaH5eaIv))

Một nhận xét quan trọng ở đây là phần triển khai dùng chung (biến thể dựa trên hàm) không bị ràng buộc theo bất kỳ cách nào.
Nó phải duy trì tính linh hoạt tối đa để đóng vai trò là phần lõi dùng chung.
Chúng ta cũng không lưu hàm so sánh trong một trường của struct.
Thay vào đó, chúng ta truyền nó dưới dạng tham số vì đối số hàm dễ để trình biên dịch phân tích hơn so với các trường của struct.

Tất nhiên, vẫn có một lượng mã mẫu cần viết.
Tất cả các phần triển khai được export cần lặp lại toàn bộ API với các mẫu gọi hơi khác nhau.
Nhưng phần này khá đơn giản để viết và đọc.

## Kết hợp các phương thức và tập kiểu

Chúng ta có thể sử dụng cấu trúc dữ liệu cây mới để triển khai một tập có thứ tự, cung cấp khả năng tra cứu phần tử trong thời gian logarit.
Bây giờ hãy tưởng tượng chúng ta cần làm cho việc tra cứu chạy trong thời gian hằng số; chúng ta có thể thử thực hiện điều này bằng cách duy trì một Go map thông thường bên cạnh cây:

{{raw `
    type OrderedSet[E Comparer[E]] struct {
        tree     MethodTree[E] // để lặp theo thứ tự hiệu quả
        elements map[E]bool    // để tra cứu (gần) trong thời gian hằng số
    }

    func (s *OrderedSet[E]) Has(e E) bool {
        return s.elements[e]
    }

    func (s *OrderedSet[E]) Insert(e E) {
        if s.elements == nil {
            s.elements = make(map[E]bool)
        }
        if s.elements[e] {
            return
        }
        s.elements[e] = true
        s.tree.Insert(e)
    }

    func (s *OrderedSet[E]) All() iter.Seq[E] {
        return func(yield func(E) bool) {
            s.tree.root.all(yield)
        }
    }

    func (n *node[E]) all(yield func(E) bool) bool {
        return n == nil || (n.left.all(yield) && yield(n.value) && n.right.all(yield))
    }
`}}

([sân chơi](/play/p/TANUnnSnDqf))

Tuy nhiên, việc biên dịch mã này sẽ tạo ra một lỗi:

> invalid map key type E (missing comparable constraint)

Thông báo lỗi cho biết chúng ta cần ràng buộc thêm tham số kiểu để có thể sử dụng nó làm khóa của map.
Ràng buộc `comparable` là một ràng buộc đặc biệt được khai báo sẵn, được thỏa mãn bởi tất cả các kiểu có định nghĩa các toán tử bằng `==` và `!=`.
Trong Go, đó cũng là tập các kiểu có thể được sử dụng làm khóa cho kiểu `map` dựng sẵn.

Chúng ta có ba tùy chọn để thêm ràng buộc này vào tham số kiểu, mỗi tùy chọn có những đánh đổi khác nhau:

1.  Chúng ta có thể [nhúng](/ref/spec#Embedded_interfaces) `comparable` vào định nghĩa `Comparer` ban đầu ([sân chơi](/play/p/g8NLjZCq97q)):

    {{raw `
        type Comparer[E any] interface {
            comparable
            Compare(E) int
        }
    `}}

    Nhược điểm của cách này là nó cũng sẽ khiến các kiểu `Tree` của chúng ta chỉ có thể sử dụng với các kiểu là `comparable`.
    Nói chung, chúng ta không muốn hạn chế các kiểu generic một cách không cần thiết.
2.  Chúng ta có thể thêm một định nghĩa ràng buộc mới ([sân chơi](/play/p/Z2eg4X8xK5Z)).

    {{raw `
        type Comparer[E any] interface {
            Compare(E) int
        }

        type ComparableComparer[E any] interface {
            comparable
            Comparer[E]
        }
    `}}

    Cách này gọn gàng, nhưng nó đưa một định danh mới (`ComparableComparer`) vào API của chúng ta, và việc đặt tên rất khó.
3.  Chúng ta có thể thêm ràng buộc trực tiếp vào kiểu có nhiều ràng buộc hơn ([sân chơi](/play/p/ZfggVma_jNc)):

    {{raw `
        type OrderedSet[E interface {
            comparable
            Comparer[E]
        }] struct {
            tree     Tree[E]
            elements map[E]struct{}
        }
    `}}

    Cách này có thể trở nên hơi khó đọc, đặc biệt nếu cần thực hiện thường xuyên.
    Nó cũng khiến việc tái sử dụng ràng buộc ở những nơi khác trở nên khó khăn hơn.

Việc sử dụng tùy chọn nào trong số này là một lựa chọn về phong cách và cuối cùng phụ thuộc vào sở thích cá nhân.

## (Không) ràng buộc các generic interface

Tại thời điểm này, đáng để thảo luận về các ràng buộc trên generic interface.
Bạn có thể muốn định nghĩa một interface cho một kiểu container generic.
Ví dụ, giả sử bạn có một thuật toán yêu cầu một cấu trúc dữ liệu tập hợp.
Có nhiều loại triển khai tập hợp khác nhau với các đánh đổi khác nhau.
Việc định nghĩa một interface cho các thao tác trên tập hợp mà bạn yêu cầu có thể làm tăng tính linh hoạt cho gói của bạn, để người dùng quyết định những đánh đổi nào phù hợp với ứng dụng cụ thể:

{{raw `
    type Set[E any] interface {
        Insert(E)
        Delete(E)
        Has(E) bool
        All() iter.Seq[E]
    }
`}}

Một câu hỏi tự nhiên ở đây là ràng buộc trên interface này nên là gì.
Nếu có thể, tham số kiểu trên các generic interface nên sử dụng `any` làm ràng buộc, cho phép các kiểu tùy ý.

Từ những thảo luận ở trên, các lý do sẽ trở nên rõ ràng:
Các triển khai cụ thể khác nhau có thể yêu cầu các ràng buộc khác nhau.
Tất cả các kiểu `Tree` mà chúng ta đã xem xét ở trên, cũng như kiểu `OrderedSet`, đều có thể triển khai `Set` cho các kiểu phần tử của chúng, mặc dù các kiểu này có các ràng buộc khác nhau.

Mục đích của việc định nghĩa một interface là để giao việc triển khai cho người dùng.
Vì không thể dự đoán người dùng có thể muốn áp đặt những loại ràng buộc nào lên triển khai của họ, hãy cố gắng để mọi ràng buộc (mạnh hơn `any`) cho các triển khai cụ thể, không phải các interface.

## Bộ nhận con trỏ

Hãy thử sử dụng interface `Set` trong một ví dụ.
Xét một hàm loại bỏ các phần tử trùng lặp trong một chuỗi:

{{raw `
    // Unique loại bỏ các phần tử trùng lặp khỏi chuỗi đầu vào, chỉ trả về
    // lần xuất hiện đầu tiên của mỗi phần tử.
    func Unique[E comparable](input iter.Seq[E]) iter.Seq[E] {
        return func(yield func(E) bool) {
            seen := make(map[E]bool)
            for v := range input {
                if seen[v] {
                    continue
                }
                if !yield(v) {
                    return
                }
                seen[v] = true
            }
        }
    }
`}}

([playground](/play/p/hsYoFjkU9kA))

Điều này sử dụng một `map[E]bool` làm một tập hợp đơn giản của các phần tử `E`.
Do đó, nó chỉ hoạt động với các kiểu là `comparable` và vì vậy định nghĩa các toán tử so sánh bằng tích hợp sẵn.
Nếu chúng ta muốn tổng quát hóa điều này cho các kiểu tùy ý, chúng ta cần thay thế nó bằng một tập hợp generic:

{{raw `
    // Unique loại bỏ các phần tử trùng lặp khỏi chuỗi đầu vào, chỉ trả về
    // lần xuất hiện đầu tiên của mỗi phần tử.
    func Unique[E any](input iter.Seq[E]) iter.Seq[E] {
        return func(yield func(E) bool) {
            var seen Set[E]
            for v := range input {
                if seen.Has(v) {
                    continue
                }
                if !yield(v) {
                    return
                }
                seen.Insert(v)
            }
        }
    }
`}}

([playground](/play/p/FZYPNf56nnY))

Tuy nhiên, điều này không hoạt động.
`Set[E]` là một kiểu interface, và biến `seen` sẽ được khởi tạo thành `nil`.
Chúng ta cần sử dụng một triển khai cụ thể của interface `Set[E]`.
Nhưng như chúng ta đã thấy trong bài viết này, không có một triển khai tổng quát nào của tập hợp hoạt động với kiểu phần tử `any`.

Chúng ta phải yêu cầu người dùng cung cấp một triển khai cụ thể mà chúng ta có thể sử dụng, dưới dạng một tham số kiểu bổ sung:

{{raw `
    // Unique loại bỏ các phần tử trùng lặp khỏi chuỗi đầu vào, chỉ trả về
    // lần xuất hiện đầu tiên của mỗi phần tử.
    func Unique[E any, S Set[E]](input iter.Seq[E]) iter.Seq[E] {
        return func(yield func(E) bool) {
            var seen S
            for v := range input {
                if seen.Has(v) {
                    continue
                }
                if !yield(v) {
                    return
                }
                seen.Insert(v)
            }
        }
    }
`}}

([playground](/play/p/kjkGy5cNz8T))

Tuy nhiên, khi khởi tạo điều này với triển khai tập hợp của chúng ta, chúng ta gặp một vấn đề khác:

{{raw `
    // OrderedSet[E] không thỏa mãn Set[E] (phương thức All có bộ nhận con trỏ)
    Unique[E, OrderedSet[E]](slices.Values(s))
    // panic: địa chỉ bộ nhớ không hợp lệ hoặc tham chiếu con trỏ nil
    Unique[E, *OrderedSet[E]](slices.Values(s))
`}}

Vấn đề đầu tiên đã rõ từ thông báo lỗi: Ràng buộc kiểu của chúng ta nói rằng đối số kiểu cho `S` cần triển khai interface `Set[E]`.
Và vì các phương thức trên `OrderedSet` sử dụng bộ nhận con trỏ, đối số kiểu cũng phải là kiểu con trỏ.

Khi cố gắng làm điều đó, chúng ta gặp vấn đề thứ hai.
Điều này bắt nguồn từ việc chúng ta khai báo một biến trong phần triển khai:

{{raw `
    var seen S
`}}

Nếu `S` là `*OrderedSet[E]`, biến sẽ được khởi tạo với `nil`, như trước.
Việc gọi `seen.Insert` sẽ gây panic.

Nếu chúng ta chỉ có kiểu con trỏ, chúng ta không thể lấy được một biến hợp lệ của kiểu giá trị.
Và nếu chúng ta chỉ có kiểu giá trị, chúng ta không thể gọi các phương thức con trỏ trên nó.
Hệ quả là chúng ta cần cả kiểu giá trị *và* kiểu con trỏ.
Vì vậy, chúng ta phải giới thiệu một tham số kiểu bổ sung `PS` với một ràng buộc mới `PtrToSet`:

{{raw `
    // PtrToSet được triển khai bởi một kiểu con trỏ triển khai interface Set[E].
    type PtrToSet[S, E any] interface {
        *S
        Set[E]
    }

    // Unique loại bỏ các phần tử trùng lặp khỏi chuỗi đầu vào, chỉ trả về
    // lần xuất hiện đầu tiên của mỗi phần tử.
    func Unique[E, S any, PS PtrToSet[S, E]](input iter.Seq[E]) iter.Seq[E] {
        return func(yield func(E) bool) {
            // Chúng ta chuyển đổi sang PS, vì chỉ PS mới bị ràng buộc có các phương thức.
            // Việc chuyển đổi được cho phép vì tập hợp kiểu của PS chỉ chứa *S.
            seen := PS(new(S))
            for v := range input {
                if seen.Has(v) {
                    continue
                }
                if !yield(v) {
                    return
                }
                seen.Insert(v)
            }
        }
    }
`}}

([playground](/play/p/Kp1jJRVjmYa))

Mẹo ở đây là sự kết nối của hai tham số kiểu trong chữ ký hàm thông qua tham số kiểu bổ sung trên interface `PtrToSet`.
Bản thân `S` không bị ràng buộc, nhưng `PS` phải có kiểu `*S` và phải có các phương thức mà chúng ta cần.
Vì vậy, về cơ bản, chúng ta đang hạn chế `S` phải có một số phương thức, nhưng các phương thức đó cần sử dụng receiver con trỏ.

Mặc dù việc định nghĩa một hàm với loại ràng buộc này yêu cầu thêm một tham số kiểu, điều quan trọng là việc này không làm phức tạp mã sử dụng nó:
miễn là tham số kiểu bổ sung này nằm ở cuối danh sách tham số kiểu, nó [có thể được suy luận](/blog/type-inference):

{{raw `
    // Đối số kiểu thứ ba được suy luận là *OrderedSet[int]
    Unique[int, OrderedSet[int]](slices.Values(s))
`}}

Đây là một mẫu tổng quát và đáng ghi nhớ: khi bạn gặp mẫu này trong công việc của người khác, hoặc khi bạn muốn sử dụng nó trong mã của chính mình.

{{raw `
    func SomeFunction[T any, PT interface{ *T; SomeMethods }]()
`}}

Nếu bạn có hai tham số kiểu, trong đó một tham số bị ràng buộc là con trỏ tới tham số còn lại, ràng buộc đó đảm bảo rằng các phương thức liên quan sử dụng receiver con trỏ.

## Có nên ràng buộc receiver con trỏ không?

Ở thời điểm này, có thể bạn cảm thấy khá choáng ngợp.
Điều này khá phức tạp và có vẻ không hợp lý khi mong đợi mọi lập trình viên Go hiểu được điều gì đang xảy ra trong chữ ký hàm này.
Chúng ta cũng phải đưa thêm nhiều tên hơn vào API của mình.
Khi mọi người cảnh báo không nên thêm generics vào Go ngay từ đầu, đây là một trong những điều họ lo ngại.

Vì vậy, nếu bạn thấy mình bị cuốn vào những vấn đề này, hãy thử lùi lại một bước.
Chúng ta thường có thể tránh sự phức tạp này bằng cách suy nghĩ về vấn đề theo một cách khác.
Trong ví dụ này, chúng ta đã xây dựng một hàm nhận một `iter.Seq[E]` và trả về một `iter.Seq[E]` chứa các phần tử duy nhất.
Nhưng để loại bỏ các phần tử trùng lặp, chúng ta cần thu thập các phần tử duy nhất vào một set.
Và vì việc này yêu cầu cấp phát không gian cho toàn bộ kết quả, chúng ta thực sự không được hưởng lợi nhiều từ việc biểu diễn kết quả dưới dạng một luồng.

Nếu xem xét lại vấn đề này, chúng ta có thể tránh hoàn toàn tham số kiểu bổ sung bằng cách sử dụng `Set[E]` như một giá trị interface thông thường:

{{raw `
    // InsertAll thêm tất cả các phần tử duy nhất từ seq vào set.
    func InsertAll[E any](set Set[E], seq iter.Seq[E]) {
        for v := range seq {
            set.Insert(v)
        }
    }
`}}

([playground](/play/p/woZcHodAgaa))

Bằng cách sử dụng `Set` như một kiểu interface thuần túy, rõ ràng là bên gọi phải cung cấp một giá trị hợp lệ của triển khai cụ thể của họ.
Đây là một mẫu rất phổ biến.
Và nếu họ cần một `iter.Seq[E]`, họ chỉ cần gọi `All()` trên `set` để lấy được một giá trị.

Điều này làm mọi thứ phức tạp hơn một chút đối với bên gọi, nhưng nó có một ưu điểm khác so với ràng buộc receiver con trỏ:
hãy nhớ rằng chúng ta bắt đầu với `map[E]bool` như một kiểu set đơn giản.
Việc triển khai interface `Set[E]` dựa trên kiểu đó rất dễ:

{{raw `
    type HashSet[E comparable] map[E]bool

    func (s HashSet[E]) Insert(v E)       { s[v] = true }
    func (s HashSet[E]) Delete(v E)       { delete(s, v) }
    func (s HashSet[E]) Has(v E) bool     { return s[v] }
    func (s HashSet[E]) All() iter.Seq[E] { return maps.Keys(s) }
`}}

([playground](/play/p/KPPpWa7M93d))

Triển khai này không sử dụng receiver con trỏ.
Vì vậy, mặc dù nó hoàn toàn hợp lệ, nó sẽ không thể được sử dụng với ràng buộc phức tạp về receiver con trỏ.
Nhưng nó hoạt động tốt với phiên bản `InsertAll` của chúng ta.
Cũng như nhiều ràng buộc khác, việc buộc các phương thức sử dụng receiver con trỏ thực tế có thể là quá hạn chế đối với nhiều trường hợp sử dụng thực tế.

## Kết luận

Tôi hy vọng điều này minh họa một số mẫu thiết kế và sự đánh đổi mà tham số kiểu trên interface cho phép.
Đây là một công cụ mạnh mẽ, nhưng nó cũng đi kèm với chi phí.
Các điểm chính cần ghi nhớ là:

1. Sử dụng interface generic để biểu đạt các ràng buộc trên receiver bằng cách sử dụng chúng theo kiểu tự tham chiếu.
2. Sử dụng chúng để tạo các mối quan hệ bị ràng buộc giữa những tham số kiểu khác nhau.
3. Sử dụng chúng để trừu tượng hóa các triển khai khác nhau với các loại ràng buộc khác nhau.
4. Khi bạn thấy mình ở tình huống cần ràng buộc với receiver là con trỏ, hãy cân nhắc liệu bạn có thể tái cấu trúc mã của mình để tránh sự phức tạp bổ sung hay không. Xem ["Bạn có nên ràng buộc với receiver là con trỏ không?"](#should-you-constrain-to-pointer-receivers).

Như mọi khi, đừng thiết kế quá mức cần thiết: một giải pháp kém linh hoạt hơn nhưng đơn giản hơn và dễ đọc hơn cuối cùng có thể là lựa chọn sáng suốt hơn.
