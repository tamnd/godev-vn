---
title: Map Go nhanh hơn với Swiss Tables
date: 2025-02-26
by:
- Michael Pratt
summary: Go 1.24 cải thiện hiệu năng của map với một cách triển khai map hoàn toàn mới
---

Bảng băm là một cấu trúc dữ liệu trung tâm trong khoa học máy tính, và nó cung cấp cách triển khai cho kiểu `map` trong nhiều ngôn ngữ, bao gồm Go.

Khái niệm về bảng băm được [mô tả lần đầu](https://spectrum.ieee.org/hans-peter-luhn-and-the-birth-of-the-hashing-algorithm) bởi Hans Peter Luhn vào năm 1953 trong một bản ghi nhớ nội bộ của IBM, trong đó đề xuất tăng tốc việc tìm kiếm bằng cách đặt các mục vào các "bucket" và sử dụng danh sách liên kết để xử lý phần tràn khi các bucket đã chứa một mục.
Ngày nay, chúng ta sẽ gọi đây là [bảng băm sử dụng chaining](https://en.wikipedia.org/wiki/Hash_table#Separate_chaining).

Năm 1954, Gene M. Amdahl, Elaine M. McGraw và Arthur L. Samuel lần đầu tiên sử dụng cơ chế "open addressing" khi lập trình cho IBM 701.
Khi một bucket đã chứa một mục, mục mới được đặt vào bucket trống tiếp theo.
Ý tưởng này được W. Wesley Peterson chính thức hóa và công bố vào năm 1957 trong ["Addressing for Random-Access Storage"](https://ieeexplore.ieee.org/document/5392733).
Ngày nay, chúng ta sẽ gọi đây là [bảng băm sử dụng open addressing với linear probing](https://en.wikipedia.org/wiki/Hash_table#Open_addressing).

Với những cấu trúc dữ liệu đã tồn tại lâu như vậy, thật dễ nghĩ rằng chúng đã "hoàn thiện"; rằng chúng ta đã biết mọi thứ cần biết về chúng và chúng không thể được cải thiện thêm nữa.
Điều đó không đúng!
Nghiên cứu khoa học máy tính vẫn tiếp tục tạo ra những tiến bộ trong các thuật toán nền tảng, cả về độ phức tạp của thuật toán lẫn việc tận dụng phần cứng CPU hiện đại.
Ví dụ, Go 1.19 đã [chuyển gói `sort`](/doc/go1.19#sortpkgsort) từ quicksort truyền thống sang [pattern-defeating quicksort](https://arxiv.org/pdf/2106.05123.pdf), một thuật toán sắp xếp mới của Orson R. L. Peters, được mô tả lần đầu vào năm 2015.

Giống như các thuật toán sắp xếp, cấu trúc dữ liệu bảng băm tiếp tục được cải tiến.
Năm 2017, Sam Benzaquen, Alkis Evlogimenos, Matt Kulukundis và Roman Perepelitsa tại Google đã giới thiệu [một thiết kế bảng băm C++ mới](https://www.youtube.com/watch?v=ncHmEUmJZf4), được gọi là "Swiss Tables".
Năm 2018, triển khai của họ đã được [mã nguồn mở trong thư viện Abseil C++](https://abseil.io/blog/20180927-swisstables).

Go 1.24 bao gồm một triển khai hoàn toàn mới của kiểu `map` tích hợp sẵn, dựa trên thiết kế Swiss Table.
Trong bài đăng blog này, chúng ta sẽ xem cách Swiss Tables cải thiện các bảng băm truyền thống, cũng như một số thách thức riêng biệt khi đưa thiết kế Swiss Table vào các `map` của Go.

## Bảng băm địa chỉ mở

Swiss Tables là một dạng của bảng băm địa chỉ mở, vì vậy hãy cùng xem nhanh cách hoạt động của một bảng băm địa chỉ mở cơ bản.

Trong một bảng băm địa chỉ mở, tất cả mục được lưu trữ trong một mảng nền duy nhất.
Chúng ta sẽ gọi mỗi vị trí trong mảng là một *ô*.
Ô mà một khóa thuộc về chủ yếu được xác định bởi *hàm băm*, `hash(key)`.
Hàm băm ánh xạ mỗi khóa thành một số nguyên, trong đó cùng một khóa luôn ánh xạ tới cùng một số nguyên, còn các khóa khác nhau lý tưởng nhất sẽ tuân theo phân phối ngẫu nhiên đồng đều của các số nguyên.
Đặc điểm xác định của bảng băm địa chỉ mở là chúng giải quyết xung đột bằng cách lưu trữ khóa ở nơi khác trong mảng nền.
Vì vậy, nếu ô đã đầy (một *xung đột*), thì một *chuỗi dò* được sử dụng để xem xét các ô khác cho đến khi tìm thấy một ô trống.
Hãy xem một bảng băm mẫu để hiểu cách hoạt động này.

### Ví dụ

Bên dưới, bạn có thể thấy một mảng nền 16 ô cho một bảng băm, cùng với khóa (nếu có) được lưu trong mỗi ô.
Các giá trị không được hiển thị vì chúng không liên quan đến ví dụ này.

<style>
/*
Giữ bảng nằm trong cột nội dung. overflow-x: auto thêm thanh cuộn ngang chỉ
khi bảng rộng hơn cột (ví dụ trên các khung nhìn hẹp), vì vậy các cột bên phải
không bao giờ bị cắt.
*/
.swisstable-table-container {
    overflow-x: auto;
}

.swisstable-table {
    /* Kết hợp đường viền bên trong của bảng (tổng cộng 1px thay vì 2px, một cho ô phía trên và một cho ô phía dưới). */
    border-collapse: collapse;
    /* Bố cục cố định: các cột dữ liệu chia đều phần chiều rộng còn lại sau
       cột đầu tiên rộng hơn (cột nhãn) bên dưới. */
    table-layout: fixed;
    /* Khớp với chiều rộng của cột văn bản xung quanh. */
    width: 100%;
    /*
    Giữ các cột dễ sử dụng trên điện thoại (cuộn thay vì ép chúng lại).
    Được tính là nhãn 3em + 16 cột dữ liệu khoảng 2.5em mỗi cột. Không phải
    max-content: với bố cục cố định, chiều rộng được suy ra từ hàng đầu tiên,
    vì vậy các hàng tiêu đề colspan của các bảng nhóm/từ điều khiển sẽ làm
    các cột dữ liệu bị thu hẹp.
    */
    min-width: 44em;
}

/*
Bộ chọn phần tử con tăng độ ưu tiên cao hơn quy tắc toàn trang
"div.markdown th, div.markdown td", quy tắc này nếu không sẽ ghi đè phần đệm
bên dưới bằng phần đệm ngang rộng 2em và kéo giãn các ô.
*/
.swisstable-table .swisstable-table-cell {
    /* Đường viền đen giữa các ô. */
    border: 1px solid;
    /* Thêm khoảng cách trực quan xung quanh nội dung. */
    padding: 0.5em 0.6em 0.5em 0.6em;
    /* Căn giữa trong ô. */
    text-align: center;
}

/* Cho cột nhãn ("Slot", "Key", "h2") thêm nhiều không gian hơn một chút so với các cột dữ liệu. */
.swisstable-table .swisstable-table-cell:first-child {
    width: 3em;
}

/*
Bảng so sánh SIMD có các nhãn hàng dài ("Comparison", "Control word"), vì vậy
các quy tắc bên dưới ghi đè chiều rộng cơ sở của .swisstable-table ở trên
(cùng độ ưu tiên, quy tắc xuất hiện sau trong mã nguồn sẽ thắng).

Cùng chiều rộng cột dữ liệu như các bảng khác: nhãn 8em + 8 cột dữ liệu khoảng 2.5em mỗi cột.
*/
.swisstable-table-simd-comparison {
    min-width: 28em;
}

/* Mở rộng cột đầu tiên ít nhất bằng nhãn lớn nhất để nó không bị cắt. */
.swisstable-table-simd-comparison .swisstable-table-cell:first-child {
    width: 8em;
}
</style>

<div class="swisstable-table-container">
    <table class="swisstable-table">
        <thead>
            <tr>
                <th class="swisstable-table-cell">Ô</th>
                <th class="swisstable-table-cell">0</th>
                <th class="swisstable-table-cell">1</th>
                <th class="swisstable-table-cell">2</th>
                <th class="swisstable-table-cell">3</th>
                <th class="swisstable-table-cell">4</th>
                <th class="swisstable-table-cell">5</th>
                <th class="swisstable-table-cell">6</th>
                <th class="swisstable-table-cell">7</th>
                <th class="swisstable-table-cell">8</th>
                <th class="swisstable-table-cell">9</th>
                <th class="swisstable-table-cell">10</th>
                <th class="swisstable-table-cell">11</th>
                <th class="swisstable-table-cell">12</th>
                <th class="swisstable-table-cell">13</th>
                <th class="swisstable-table-cell">14</th>
                <th class="swisstable-table-cell">15</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td class="swisstable-table-cell">Khóa</td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell">56</td>
                <td class="swisstable-table-cell">32</td>
                <td class="swisstable-table-cell">21</td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell">78</td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
            </tr>
        </tbody>
    </table>
</div>

Để chèn một khóa mới, chúng ta sử dụng hàm băm để chọn một ô.
Vì chỉ có 16 ô, chúng ta cần giới hạn trong phạm vi này, nên sẽ sử dụng `hash(key) % 16` làm ô đích.
Giả sử chúng ta muốn chèn khóa `98` và `hash(98) % 16 = 7`.
Ô 7 đang trống, vì vậy chúng ta chỉ cần chèn 98 vào đó.
Mặt khác, giả sử chúng ta muốn chèn khóa `25` và `hash(25) % 16 = 3`.
Ô 3 là một xung đột vì nó đã chứa khóa 56.
Do đó chúng ta không thể chèn vào đây.

Chúng ta sử dụng một chuỗi dò để tìm một vị trí khác.
Có nhiều chuỗi dò nổi tiếng.
Chuỗi dò ban đầu và đơn giản nhất là *dò tuyến tính*, chỉ đơn giản thử các vị trí liên tiếp theo thứ tự.

Vì vậy, trong ví dụ `hash(25) % 16 = 3`, do vị trí 3 đang được sử dụng, chúng ta sẽ xét vị trí 4 tiếp theo, vị trí này cũng đang được sử dụng.
Vị trí 5 cũng vậy.
Cuối cùng, chúng ta sẽ đến vị trí trống 6, nơi chúng ta sẽ lưu khóa 25.

Việc tra cứu cũng tuân theo cùng cách tiếp cận.
Việc tra cứu khóa 25 sẽ bắt đầu tại vị trí 3, kiểm tra xem nó có chứa khóa 25 hay không (không có), sau đó tiếp tục dò tuyến tính cho đến khi tìm thấy khóa 25 tại vị trí 6.

Ví dụ này sử dụng một mảng dự phòng có 16 vị trí.
Điều gì xảy ra nếu chúng ta chèn nhiều hơn 16 phần tử?
Nếu bảng băm hết không gian, nó sẽ tăng kích thước, thường bằng cách nhân đôi kích thước của mảng dự phòng.
Tất cả mục hiện có sẽ được chèn lại vào mảng dự phòng mới.

Các bảng băm định địa chỉ mở thực tế không đợi đến khi mảng dự phòng hoàn toàn đầy mới tăng kích thước, vì khi mảng càng đầy, độ dài trung bình của mỗi chuỗi dò càng tăng.
Trong ví dụ trên sử dụng khóa 25, chúng ta phải dò 4 vị trí khác nhau để tìm một vị trí trống.
Nếu mảng chỉ còn một vị trí trống, độ dài dò trong trường hợp xấu nhất sẽ là O(n).
Nghĩa là bạn có thể cần quét toàn bộ mảng.
Tỷ lệ các vị trí đã sử dụng được gọi là *hệ số tải*, và hầu hết bảng băm định nghĩa một *hệ số tải tối đa* (thường là 70-90%) tại đó chúng sẽ tăng kích thước để tránh các chuỗi dò cực kỳ dài của những bảng băm gần đầy.

## Swiss Table

Thiết kế [Swiss Table](https://abseil.io/about/design/swisstables) cũng là một dạng bảng băm định địa chỉ mở.
Hãy xem cách nó cải thiện so với một bảng băm định địa chỉ mở truyền thống.
Chúng ta vẫn có một mảng dự phòng duy nhất để lưu trữ, nhưng sẽ chia mảng này thành các *nhóm* logic, mỗi nhóm gồm 8 vị trí.
(Các kích thước nhóm lớn hơn cũng có thể thực hiện được. Xem thêm ở bên dưới.)

Ngoài ra, mỗi nhóm có một *từ điều khiển* 64-bit để chứa siêu dữ liệu.
Mỗi trong số 8 byte trong từ điều khiển tương ứng với một vị trí trong nhóm.
Giá trị của mỗi byte cho biết vị trí đó đang trống, đã bị xóa hay đang được sử dụng.
Nếu đang được sử dụng, byte chứa 7 bit thấp hơn của giá trị băm cho khóa của vị trí đó (gọi là `h2`).

<!-- Bảng nhóm theo sau bởi bảng từ điều khiển. Cả hai nằm trong cùng một vùng chứa để cuộn cùng nhau trên thiết bị di động. -->
<div class="swisstable-table-container">
    <table class="swisstable-table">
        <thead>
            <tr>
                <th class="swisstable-table-cell"></th>
                <th class="swisstable-table-cell" colspan="8">Nhóm 0</th>
                <th class="swisstable-table-cell" colspan="8">Nhóm 1</th>
            </tr>
            <tr>
                <th class="swisstable-table-cell">Vị trí</th>
                <th class="swisstable-table-cell">0</th>
                <th class="swisstable-table-cell">1</th>
                <th class="swisstable-table-cell">2</th>
                <th class="swisstable-table-cell">3</th>
                <th class="swisstable-table-cell">4</th>
                <th class="swisstable-table-cell">5</th>
                <th class="swisstable-table-cell">6</th>
                <th class="swisstable-table-cell">7</th>
                <th class="swisstable-table-cell">0</th>
                <th class="swisstable-table-cell">1</th>
                <th class="swisstable-table-cell">2</th>
                <th class="swisstable-table-cell">3</th>
                <th class="swisstable-table-cell">4</th>
                <th class="swisstable-table-cell">5</th>
                <th class="swisstable-table-cell">6</th>
                <th class="swisstable-table-cell">7</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td class="swisstable-table-cell">Khóa</td>
                <td class="swisstable-table-cell">56</td>
                <td class="swisstable-table-cell">32</td>
                <td class="swisstable-table-cell">21</td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell">78</td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
            </tr>
        </tbody>
    </table>
    <br/> <!-- Khoảng trống hiển thị giữa các bảng -->
    <table class="swisstable-table">
        <thead>
            <tr>
                <th class="swisstable-table-cell"></th>
                <th class="swisstable-table-cell" colspan="8">từ điều khiển 64-bit 0</th>
                <th class="swisstable-table-cell" colspan="8">từ điều khiển 64-bit 1</th>
            </tr>
            <tr>
                <th class="swisstable-table-cell">Vị trí</th>
                <th class="swisstable-table-cell">0</th>
                <th class="swisstable-table-cell">1</th>
                <th class="swisstable-table-cell">2</th>
                <th class="swisstable-table-cell">3</th>
                <th class="swisstable-table-cell">4</th>
                <th class="swisstable-table-cell">5</th>
                <th class="swisstable-table-cell">6</th>
                <th class="swisstable-table-cell">7</th>
                <th class="swisstable-table-cell">0</th>
                <th class="swisstable-table-cell">1</th>
                <th class="swisstable-table-cell">2</th>
                <th class="swisstable-table-cell">3</th>
                <th class="swisstable-table-cell">4</th>
                <th class="swisstable-table-cell">5</th>
                <th class="swisstable-table-cell">6</th>
                <th class="swisstable-table-cell">7</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <td class="swisstable-table-cell">h2</td>
                <td class="swisstable-table-cell">23</td>
                <td class="swisstable-table-cell">89</td>
                <td class="swisstable-table-cell">50</td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell">47</td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
                <td class="swisstable-table-cell"></td>
            </tr>
        </tbody>
    </table>
</div>

Việc chèn hoạt động như sau:

1. Tính toán `hash(key)` và chia hash thành hai phần: 57 bit cao (được gọi là `h1`) và 7 bit thấp (được gọi là `h2`).
2. Các bit cao (`h1`) được dùng để chọn nhóm đầu tiên cần xem xét: `h1 % 2` trong trường hợp này, vì chỉ có 2 nhóm.
3. Trong một nhóm, tất cả vị trí đều có khả năng chứa khóa như nhau. Trước tiên, chúng ta phải xác định xem có vị trí nào đã chứa khóa này hay không; nếu có, đây là một thao tác cập nhật thay vì một lần chèn mới.
4. Nếu không có vị trí nào chứa khóa, chúng ta tìm một vị trí trống để đặt khóa này.
5. Nếu không có vị trí nào trống, chúng ta tiếp tục chuỗi thăm dò bằng cách tìm kiếm nhóm tiếp theo.

Tra cứu tuân theo cùng một quy trình cơ bản.
Nếu tìm thấy một vị trí trống ở bước 4, chúng ta biết rằng một lần chèn sẽ sử dụng vị trí này và có thể dừng việc tìm kiếm.

Bước 3 là nơi phép màu của Swiss Table xảy ra.
Chúng ta cần kiểm tra xem có ô nào trong một nhóm chứa khóa mong muốn hay không.
Một cách đơn giản, chúng ta có thể chỉ cần quét tuyến tính và so sánh cả 8 khóa.
Tuy nhiên, từ điều khiển cho phép chúng ta thực hiện việc này hiệu quả hơn.
Mỗi byte chứa 7 bit thấp hơn của giá trị băm (`h2`) cho ô đó.
Nếu xác định được những byte nào của từ điều khiển chứa `h2` mà chúng ta đang tìm kiếm, chúng ta sẽ có một tập các kết quả khớp ứng viên.

Nói cách khác, chúng ta muốn thực hiện phép so sánh bằng nhau theo từng byte bên trong từ điều khiển.
Ví dụ, nếu chúng ta đang tìm khóa 32, với `h2 = 89`, phép toán mong muốn sẽ trông như sau.

<!-- Visualization of SIMD comparison -->
<div class="swisstable-table-container">
    <table class="swisstable-table swisstable-table-simd-comparison">
        <tbody>
            <tr>
                <td class="swisstable-table-cell"><strong>Test word</strong></td>
                <td class="swisstable-table-cell">89</td>
                <td class="swisstable-table-cell">89</td>
                <td class="swisstable-table-cell">89</td>
                <td class="swisstable-table-cell">89</td>
                <td class="swisstable-table-cell">89</td>
                <td class="swisstable-table-cell">89</td>
                <td class="swisstable-table-cell">89</td>
                <td class="swisstable-table-cell">89</td>
            </tr>
            <tr>
                <td class="swisstable-table-cell"><strong>Comparison</strong></td>
                <td class="swisstable-table-cell">==</td>
                <td class="swisstable-table-cell">==</td>
                <td class="swisstable-table-cell">==</td>
                <td class="swisstable-table-cell">==</td>
                <td class="swisstable-table-cell">==</td>
                <td class="swisstable-table-cell">==</td>
                <td class="swisstable-table-cell">==</td>
                <td class="swisstable-table-cell">==</td>
            </tr>
            <tr>
                <td class="swisstable-table-cell"><strong>Control word</strong></td>
                <td class="swisstable-table-cell">23</td>
                <td class="swisstable-table-cell">89</td>
                <td class="swisstable-table-cell">50</td>
                <td class="swisstable-table-cell">-</td>
                <td class="swisstable-table-cell">-</td>
                <td class="swisstable-table-cell">-</td>
                <td class="swisstable-table-cell">-</td>
                <td class="swisstable-table-cell">-</td>
            </tr>
            <tr>
                <td class="swisstable-table-cell"><strong>Result</strong></td>
                <td class="swisstable-table-cell">0</td>
                <td class="swisstable-table-cell">1</td>
                <td class="swisstable-table-cell">0</td>
                <td class="swisstable-table-cell">0</td>
                <td class="swisstable-table-cell">0</td>
                <td class="swisstable-table-cell">0</td>
                <td class="swisstable-table-cell">0</td>
                <td class="swisstable-table-cell">0</td>
            </tr>
        </tbody>
    </table>
</div>

Đây là một phép toán được phần cứng [SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data) hỗ trợ, trong đó một lệnh duy nhất thực hiện các phép toán song song trên những giá trị độc lập bên trong một giá trị lớn hơn (*vector*). Trong trường hợp này, chúng ta [có thể triển khai phép toán này](https://cs.opensource.google/go/go/+/master:src/internal/runtime/maps/group.go;drc=a08984bc8f2acacebeeadf7445ecfb67b7e7d7b1;l=155?ss=go) bằng cách sử dụng một tập các phép toán số học và bit tiêu chuẩn khi không có phần cứng SIMD chuyên dụng.

Kết quả là một tập các ô ứng viên.
Các ô có `h2` không khớp không chứa khóa khớp, vì vậy có thể bỏ qua chúng.
Các ô có `h2` khớp là những kết quả khớp tiềm năng, nhưng chúng ta vẫn phải kiểm tra toàn bộ khóa, vì có khả năng xảy ra xung đột (xác suất xung đột là 1/128 với giá trị băm 7 bit, vì vậy vẫn khá thấp).

Phép toán này rất mạnh, vì chúng ta đã thực hiện hiệu quả 8 bước của một chuỗi dò tìm cùng lúc, song song.
Điều này tăng tốc quá trình tra cứu và chèn bằng cách giảm số lần so sánh trung bình mà chúng ta cần thực hiện.
Cải tiến này đối với hành vi dò tìm cho phép cả các triển khai Abseil và Go tăng hệ số tải tối đa của các map Swiss Table so với các map trước đây, từ đó làm giảm mức sử dụng bộ nhớ trung bình.

## Các thách thức của Go

Kiểu map tích hợp sẵn của Go có một số thuộc tính đặc biệt gây thêm thách thức khi áp dụng một thiết kế map mới.  
Có hai vấn đề đặc biệt khó xử lý.

### Tăng trưởng từng phần

Khi một bảng băm đạt đến hệ số tải tối đa, nó cần mở rộng mảng lưu trữ.  
Thông thường, điều này có nghĩa là lần chèn tiếp theo sẽ nhân đôi kích thước của mảng và sao chép tất cả mục vào mảng mới.  
Hãy tưởng tượng việc chèn vào một map có 1GB mục.  
Hầu hết các lần chèn đều rất nhanh, nhưng lần chèn cần mở rộng map từ 1GB lên 2GB sẽ phải sao chép 1GB mục, mất rất nhiều thời gian.

Go thường được dùng cho các máy chủ nhạy với độ trễ, vì vậy chúng tôi không muốn các thao tác trên kiểu tích hợp sẵn có thể gây ảnh hưởng lớn không giới hạn đến độ trễ đuôi.  
Thay vào đó, map của Go tăng trưởng từng phần, để mỗi lần chèn có giới hạn trên về lượng công việc mở rộng mà nó phải thực hiện.  
Điều này giới hạn ảnh hưởng của một lần chèn map đến độ trễ.

Đáng tiếc là thiết kế Swiss Table của Abseil (C++) giả định việc mở rộng diễn ra toàn bộ cùng lúc, và chuỗi thăm dò phụ thuộc vào tổng số nhóm, khiến việc chia nhỏ quá trình này trở nên khó khăn.

Map tích hợp sẵn của Go giải quyết vấn đề này bằng một lớp gián tiếp khác: chia mỗi map thành nhiều Swiss Table.  
Thay vì một Swiss Table duy nhất triển khai toàn bộ map, mỗi map bao gồm một hoặc nhiều bảng độc lập bao phủ một tập con của không gian khóa.  
Một bảng riêng lẻ lưu tối đa 1024 mục.  
Một số lượng bit cao thay đổi trong giá trị băm được dùng để chọn bảng mà một khóa thuộc về.  
Đây là một dạng [*băm mở rộng*](https://en.wikipedia.org/wiki/Extendible_hashing), trong đó số lượng bit được sử dụng tăng lên khi cần để phân biệt tổng số bảng.

Trong quá trình chèn, nếu một bảng riêng lẻ cần mở rộng, nó sẽ thực hiện toàn bộ việc mở rộng cùng lúc, nhưng các bảng khác không bị ảnh hưởng.  
Do đó, giới hạn trên cho một lần chèn là độ trễ của việc mở rộng một bảng 1024 mục thành hai bảng 1024 mục, sao chép 1024 mục.

### Sửa đổi trong quá trình lặp

Nhiều thiết kế bảng băm, bao gồm Swiss Table của Abseil, cấm sửa đổi map trong quá trình lặp.  
Đặc tả ngôn ngữ Go [cho phép rõ ràng]( /ref/spec#For_statements:~:text=The%20iteration%20order,iterations%20is%200.) việc sửa đổi trong quá trình lặp, với các ngữ nghĩa sau:

* Nếu một mục bị xóa trước khi được duyệt đến, nó sẽ không được tạo ra.
* Nếu một mục được cập nhật trước khi được duyệt đến, giá trị đã cập nhật sẽ được tạo ra.
* Nếu một mục mới được thêm vào, nó có thể được tạo ra hoặc không.

Cách tiếp cận thông thường để lặp qua bảng băm là chỉ cần duyệt qua mảng lưu trữ và tạo ra các giá trị theo thứ tự chúng được sắp xếp trong bộ nhớ.  
Cách tiếp cận này xung đột với các ngữ nghĩa trên, đặc biệt là vì việc chèn có thể làm map mở rộng, từ đó xáo trộn bố cục bộ nhớ.

Chúng ta có thể tránh ảnh hưởng của việc xáo trộn trong quá trình mở rộng bằng cách để bộ lặp giữ tham chiếu đến bảng mà nó hiện đang lặp qua.  
Nếu bảng đó mở rộng trong quá trình lặp, chúng ta tiếp tục sử dụng phiên bản cũ của bảng và do đó tiếp tục trả về các khóa theo thứ tự của bố cục bộ nhớ cũ.

Điều này có hoạt động với các ngữ nghĩa trên không?  
Các mục mới được thêm sau khi mở rộng sẽ hoàn toàn bị bỏ sót, vì chúng chỉ được thêm vào bảng đã mở rộng, không phải bảng cũ.  
Điều đó không sao, vì các ngữ nghĩa cho phép các mục mới không được tạo ra.  
Tuy nhiên, việc cập nhật và xóa lại là vấn đề: sử dụng bảng cũ có thể tạo ra các mục đã cũ hoặc đã bị xóa.

Trường hợp biên này được xử lý bằng cách chỉ sử dụng bảng cũ để xác định thứ tự lặp.  
Trước khi thực sự trả về mục, chúng ta kiểm tra bảng đã mở rộng để xác định mục đó còn tồn tại hay không, đồng thời lấy giá trị mới nhất.

Điều này bao quát tất cả các ngữ nghĩa cốt lõi, mặc dù vẫn còn nhiều trường hợp biên nhỏ hơn không được đề cập ở đây.  
Cuối cùng, tính linh hoạt của map Go đối với kết quả lặp khiến việc lặp trở thành phần phức tạp nhất trong quá trình triển khai map của Go.

## Công việc trong tương lai

Trong [microbenchmarks](/issue/54766#issuecomment-2542444404), các thao tác trên map nhanh hơn tới 60% so với trong Go 1.23.
Mức cải thiện hiệu năng chính xác thay đổi khá nhiều do sự đa dạng lớn của các thao tác và cách sử dụng map, và một số trường hợp biên thực sự giảm hiệu năng so với Go 1.23.
Nhìn chung, trong các benchmark ứng dụng đầy đủ, chúng tôi nhận thấy thời gian CPU trung bình hình học được cải thiện khoảng 1,5%.

Có thêm nhiều cải tiến cho map mà chúng tôi muốn nghiên cứu trong các bản phát hành Go tương lai.
Ví dụ, chúng tôi có thể [tăng tính cục bộ của](/issue/70835) các thao tác trên những map không nằm trong bộ nhớ đệm CPU.

Chúng tôi cũng có thể tiếp tục cải thiện việc so sánh các từ điều khiển.
Như đã mô tả ở trên, chúng tôi có một triển khai di động sử dụng các phép toán số học và bitwise tiêu chuẩn.
Tuy nhiên, một số kiến trúc có các lệnh SIMD thực hiện trực tiếp kiểu so sánh này.
Go 1.24 đã sử dụng các lệnh SIMD 8 byte cho amd64, nhưng chúng tôi có thể mở rộng hỗ trợ sang các kiến trúc khác.
Quan trọng hơn, trong khi các lệnh tiêu chuẩn hoạt động trên các từ có kích thước tối đa 8 byte, các lệnh SIMD gần như luôn hỗ trợ ít nhất các từ 16 byte.
Điều này có nghĩa là chúng tôi có thể tăng kích thước nhóm lên 16 ô, và thực hiện 16 phép so sánh hash song song thay vì 8.
Điều này sẽ tiếp tục giảm số lần dò trung bình cần thiết cho các thao tác tra cứu.

## Lời cảm ơn

Một triển khai map Go dựa trên Swiss Table đã được mong đợi từ lâu và có sự tham gia của nhiều contributor.
Tôi muốn cảm ơn YunHao Zhang ([@zhangyunhao116](https://github.com/zhangyunhao116)), PJ Malloy ([@thepudds](https://github.com/thepudds)), và [@andy-wm-arthur](https://github.com/andy-wm-arthur) vì đã xây dựng các phiên bản ban đầu của một triển khai Go Swiss Table.
Peter Mattis ([@petermattis](https://github.com/petermattis)) đã kết hợp những ý tưởng này với các giải pháp cho những thách thức của Go ở trên để xây dựng [`github.com/cockroachdb/swiss`](https://pkg.go.dev/github.com/cockroachdb/swiss), một triển khai Swiss Table tuân thủ đặc tả Go.
Triển khai map tích hợp sẵn trong Go 1.24 dựa rất nhiều trên công việc của Peter.
Cảm ơn mọi người trong cộng đồng đã đóng góp!
