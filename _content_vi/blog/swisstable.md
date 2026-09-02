---
title: Map Go nhanh hơn với Swiss Tables
date: 2025-02-26
by:
- Michael Pratt
summary: Go 1.24 cải thiện hiệu suất map với một triển khai map hoàn toàn mới
---

Bảng băm (hash table) là một cấu trúc dữ liệu trung tâm trong khoa học máy tính, và nó cung cấp triển khai cho kiểu map trong nhiều ngôn ngữ, bao gồm Go.

Khái niệm về bảng băm được [mô tả lần đầu](https://spectrum.ieee.org/hans-peter-luhn-and-the-birth-of-the-hashing-algorithm) bởi Hans Peter Luhn vào năm 1953 trong một bản ghi nhớ nội bộ của IBM đề xuất tăng tốc tìm kiếm bằng cách đặt các phần tử vào các "bucket" và sử dụng danh sách liên kết cho tràn bucket khi các bucket đã chứa một phần tử.
Ngày nay chúng ta gọi đây là [bảng băm sử dụng chaining](https://en.wikipedia.org/wiki/Hash_table#Separate_chaining).

Năm 1954, Gene M. Amdahl, Elaine M. McGraw và Arthur L. Samuel lần đầu tiên sử dụng sơ đồ "open addressing" khi lập trình IBM 701.
Khi một bucket đã chứa một phần tử, phần tử mới được đặt vào bucket trống tiếp theo.
Ý tưởng này được hình thức hóa và công bố vào năm 1957 bởi W. Wesley Peterson trong ["Addressing for Random-Access Storage"](https://ieeexplore.ieee.org/document/5392733).
Ngày nay chúng ta gọi đây là [bảng băm sử dụng open addressing với linear probing](https://en.wikipedia.org/wiki/Hash_table#Open_addressing).

Với các cấu trúc dữ liệu đã tồn tại lâu đời như vậy, dễ nghĩ rằng chúng đã "hoàn chỉnh"; rằng chúng ta đã biết tất cả những gì cần biết về chúng và không thể cải thiện thêm nữa.
Điều đó không đúng!
Nghiên cứu khoa học máy tính tiếp tục đạt được những tiến bộ trong các thuật toán cơ bản, cả về độ phức tạp thuật toán lẫn tận dụng phần cứng CPU hiện đại.
Ví dụ, Go 1.19 [đã chuyển gói `sort`](/doc/go1.19#sortpkgsort) từ quicksort truyền thống sang [pattern-defeating quicksort](https://arxiv.org/pdf/2106.05123.pdf), một thuật toán sắp xếp mới lạ từ Orson R. L. Peters, được mô tả lần đầu vào năm 2015.

Giống như các thuật toán sắp xếp, các cấu trúc dữ liệu bảng băm tiếp tục thấy những cải tiến.
Vào năm 2017, Sam Benzaquen, Alkis Evlogimenos, Matt Kulukundis và Roman Perepelitsa tại Google đã trình bày [một thiết kế bảng băm C++ mới](https://www.youtube.com/watch?v=ncHmEUmJZf4), được gọi là "Swiss Tables".
Vào năm 2018, triển khai của họ được [công bố mã nguồn mở trong thư viện Abseil C++](https://abseil.io/blog/20180927-swisstables).

Go 1.24 bao gồm một triển khai hoàn toàn mới của kiểu map tích hợp sẵn, dựa trên thiết kế Swiss Table.
Trong bài đăng blog này, chúng ta sẽ xem xét Swiss Tables cải thiện như thế nào so với các bảng băm truyền thống, và một số thách thức đặc biệt trong việc đưa thiết kế Swiss Table vào map của Go.

## Bảng băm open-addressed

Swiss Tables là một dạng bảng băm open-addressed, vì vậy hãy nhanh chóng tổng quan về cách hoạt động của một bảng băm open-addressed cơ bản.

Trong một bảng băm open-addressed, tất cả các phần tử được lưu trữ trong một mảng lưu trữ duy nhất.
Chúng ta sẽ gọi mỗi vị trí trong mảng là một *slot*.
Slot mà một khóa thuộc về chủ yếu được xác định bởi *hàm băm* (hash function), `hash(key)`.
Hàm băm ánh xạ mỗi khóa thành một số nguyên, trong đó cùng một khóa luôn ánh xạ thành cùng một số nguyên, và các khóa khác nhau lý tưởng theo một phân phối ngẫu nhiên đều của các số nguyên.
Đặc điểm xác định của các bảng băm open-addressed là chúng giải quyết xung đột bằng cách lưu trữ khóa ở nơi khác trong mảng lưu trữ.
Vì vậy, nếu slot đã đầy (một *xung đột*), thì một *chuỗi probe* được sử dụng để xem xét các slot khác cho đến khi tìm thấy slot trống.
Hãy xem một bảng băm mẫu để xem cách này hoạt động.

### Ví dụ

Dưới đây bạn có thể thấy một mảng lưu trữ 16 slot cho một bảng băm, và khóa (nếu có) được lưu trữ trong mỗi slot.
Các giá trị không được hiển thị, vì chúng không liên quan đến ví dụ này.

<style>
/*
go.dev .Article max-width is 55em. Only enable horizontal scrolling if the
screen is narrow enough to require scrolling (narrower than article width)
because otherwise some platforms (e.g., Chrome on macOS) display a scrollbar
even when the screen is wide enough.
*/
@media screen and (max-width: 55em) {
    .swisstable-table-container {
        /* Scroll horizontally on overflow (likely on mobile) */
        overflow: scroll;
    }
}

.swisstable-table {
    /* Combine table inner borders (1px total rather than 2px, one for cell above and one for cell below. */
    border-collapse: collapse;
    /* All column widths equal. */
    table-layout: fixed;
    /* Center table within container div */
    margin: 0 auto;
}

.swisstable-table-cell {
    /* Black border between cells. */
    border: 1px solid;
    /* Add visual spacing around contents. */
    padding: 0.5em 1em 0.5em 1em;
    /* Center within cell. */
    text-align: center;
}
</style>

<div class="swisstable-table-container">
    <table class="swisstable-table">
        <thead>
            <tr>
                <th class="swisstable-table-cell">Slot</th>
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
                <td class="swisstable-table-cell">Key</td>
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

Để chèn một khóa mới, chúng ta sử dụng hàm băm để chọn một slot.
Vì chỉ có 16 slot, chúng ta cần giới hạn trong khoảng này, vì vậy chúng ta sẽ sử dụng `hash(key) % 16` làm slot mục tiêu.
Giả sử chúng ta muốn chèn khóa `98` và `hash(98) % 16 = 7`.
Slot 7 trống, vì vậy chúng ta chỉ cần chèn 98 vào đó.
Mặt khác, giả sử chúng ta muốn chèn khóa `25` và `hash(25) % 16 = 3`.
Slot 3 là xung đột vì nó đã chứa khóa 56.
Vì vậy chúng ta không thể chèn ở đây.

Chúng ta sử dụng một chuỗi probe để tìm slot khác.
Có nhiều chuỗi probe đã được biết đến.
Chuỗi probe gốc và đơn giản nhất là *linear probing*, đơn giản thử các slot kế tiếp theo thứ tự.

Vì vậy, trong ví dụ `hash(25) % 16 = 3`, vì slot 3 đang được sử dụng, chúng ta sẽ xem xét slot 4 tiếp theo, cũng đang được sử dụng.
Slot 5 cũng vậy.
Cuối cùng, chúng ta sẽ đến slot 6 trống, nơi chúng ta sẽ lưu khóa 25.

Tra cứu theo cùng cách tiếp cận.
Tra cứu khóa 25 sẽ bắt đầu từ slot 3, kiểm tra xem nó có chứa khóa 25 không (nó không chứa), và sau đó tiếp tục linear probing cho đến khi tìm thấy khóa 25 trong slot 6.

Ví dụ này sử dụng một mảng lưu trữ với 16 slot.
Điều gì xảy ra nếu chúng ta chèn hơn 16 phần tử?
Nếu bảng băm hết chỗ, nó sẽ tăng trưởng, thường bằng cách nhân đôi kích thước của mảng lưu trữ.
Tất cả các mục hiện có được chèn lại vào mảng lưu trữ mới.

Các bảng băm open-addressed thực sự không chờ cho đến khi mảng lưu trữ hoàn toàn đầy để tăng trưởng vì khi mảng ngày càng đầy hơn, độ dài trung bình của mỗi chuỗi probe tăng lên.
Trong ví dụ trên sử dụng khóa 25, chúng ta phải thử 4 slot khác nhau để tìm một slot trống.
Nếu mảng chỉ có một slot trống, độ dài probe tệ nhất sẽ là O(n).
Tức là, bạn có thể cần quét toàn bộ mảng.
Tỷ lệ slot đã được sử dụng được gọi là *load factor*, và hầu hết các bảng băm xác định một *load factor tối đa* (thường là 70-90%) tại điểm đó chúng sẽ tăng trưởng để tránh các chuỗi probe cực kỳ dài của các bảng băm gần đầy.

## Swiss Table

[Thiết kế](https://abseil.io/about/design/swisstables) Swiss Table cũng là một dạng bảng băm open-addressed.
Hãy xem nó cải thiện như thế nào so với một bảng băm open-addressed truyền thống.
Chúng ta vẫn có một mảng lưu trữ duy nhất, nhưng chúng ta sẽ chia mảng thành các *nhóm* logic gồm 8 slot mỗi nhóm.
(Các kích thước nhóm lớn hơn cũng có thể. Thêm về điều đó bên dưới.)

Ngoài ra, mỗi nhóm có một *control word* 64-bit cho metadata.
Mỗi byte trong 8 byte của control word tương ứng với một trong các slot trong nhóm.
Giá trị của mỗi byte biểu thị slot đó trống, đã xóa hoặc đang được sử dụng.
Nếu đang được sử dụng, byte chứa 7 bit thấp hơn của hash cho khóa của slot đó (được gọi là `h2`).

<!-- Group table followed by control word table. Both are in the same container so they scroll together on mobile. -->
<div class="swisstable-table-container">
    <table class="swisstable-table">
        <thead>
            <tr>
                <th class="swisstable-table-cell"></th>
                <th class="swisstable-table-cell" colspan="8">Group 0</th>
                <th class="swisstable-table-cell" colspan="8">Group 1</th>
            </tr>
            <tr>
                <th class="swisstable-table-cell">Slot</th>
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
                <td class="swisstable-table-cell">Key</td>
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
    <br/> <!-- Visual space between the tables -->
    <table class="swisstable-table">
        <thead>
            <tr>
                <th class="swisstable-table-cell"></th>
                <th class="swisstable-table-cell" colspan="8">64-bit control word 0</th>
                <th class="swisstable-table-cell" colspan="8">64-bit control word 1</th>
            </tr>
            <tr>
                <th class="swisstable-table-cell">Slot</th>
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

Quá trình chèn diễn ra như sau:

1. Tính `hash(key)` và chia hash thành hai phần: 57 bit cao hơn (được gọi là `h1`) và 7 bit thấp hơn (được gọi là `h2`).
2. Các bit cao (`h1`) được sử dụng để chọn nhóm đầu tiên cần xem xét: `h1 % 2` trong trường hợp này, vì chỉ có 2 nhóm.
3. Trong một nhóm, tất cả các slot đều đủ điều kiện để chứa khóa. Chúng ta phải xác định trước tiên liệu có slot nào đã chứa khóa này không, trong trường hợp đó đây là cập nhật chứ không phải là chèn mới.
4. Nếu không có slot nào chứa khóa, thì chúng ta tìm slot trống để đặt khóa này.
5. Nếu không có slot nào trống, thì chúng ta tiếp tục chuỗi probe bằng cách tìm kiếm nhóm tiếp theo.

Tra cứu theo cùng quá trình cơ bản.
Nếu chúng ta tìm thấy slot trống trong bước 4, thì chúng ta biết một lần chèn sẽ đã sử dụng slot này và có thể dừng tìm kiếm.

Bước 3 là nơi phép thuật Swiss Table xảy ra.
Chúng ta cần kiểm tra liệu slot nào trong nhóm có chứa khóa mong muốn không.
Một cách đơn giản, chúng ta có thể thực hiện quét tuyến tính và so sánh tất cả 8 khóa.
Tuy nhiên, control word cho phép chúng ta làm điều này hiệu quả hơn.
Mỗi byte chứa 7 bit thấp hơn của hash (`h2`) cho slot đó.
Nếu chúng ta xác định byte nào của control word chứa `h2` chúng ta đang tìm kiếm, chúng ta sẽ có một tập các ứng cử viên phù hợp.

Nói cách khác, chúng ta muốn thực hiện so sánh bằng nhau từng byte trong control word.
Ví dụ, nếu chúng ta đang tìm kiếm khóa 32, trong đó `h2 = 89`, thao tác chúng ta muốn trông như thế này.

<!-- Visualization of SIMD comparison -->
<div class="swisstable-table-container">
    <table class="swisstable-table">
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

Đây là một thao tác được hỗ trợ bởi phần cứng [SIMD](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data), trong đó một lệnh duy nhất thực hiện các thao tác song song trên các giá trị độc lập trong một giá trị lớn hơn (*vector*). Trong trường hợp này, chúng ta [có thể triển khai thao tác này](https://cs.opensource.google/go/go/+/master:src/internal/runtime/maps/group.go;drc=a08984bc8f2acacebeeadf7445ecfb67b7e7d7b1;l=155?ss=go) sử dụng một tập hợp các thao tác số học và bitwise tiêu chuẩn khi phần cứng SIMD đặc biệt không có sẵn.

Kết quả là một tập các slot ứng cử viên.
Các slot mà `h2` không khớp không có khóa phù hợp, vì vậy chúng có thể được bỏ qua.
Các slot mà `h2` khớp là các kết quả tiềm năng, nhưng chúng ta vẫn phải kiểm tra toàn bộ khóa, vì có khả năng xung đột (xác suất xung đột 1/128 với hash 7-bit, vẫn khá thấp).

Thao tác này rất mạnh mẽ, vì chúng ta đã thực hiện hiệu quả 8 bước của chuỗi probe cùng một lúc, song song.
Điều này tăng tốc tra cứu và chèn bằng cách giảm số lần so sánh trung bình chúng ta cần thực hiện.
Cải tiến đối với hành vi probing này đã cho phép cả các triển khai Abseil và Go tăng load factor tối đa của các map Swiss Table so với các map trước đó, giúp giảm dung lượng bộ nhớ trung bình.

## Thách thức của Go

Kiểu map tích hợp sẵn của Go có một số thuộc tính bất thường đặt ra những thách thức bổ sung khi áp dụng một thiết kế map mới.
Hai điều đặc biệt khó xử lý.

### Tăng trưởng dần dần

Khi bảng băm đạt đến load factor tối đa, nó cần tăng kích thước mảng lưu trữ.
Thông thường, điều này có nghĩa là lần chèn tiếp theo nhân đôi kích thước mảng và sao chép tất cả các mục sang mảng mới.
Hãy tưởng tượng chèn vào một map với 1GB dữ liệu.
Hầu hết các lần chèn rất nhanh, nhưng lần chèn cần tăng trưởng map từ 1GB lên 2GB sẽ cần sao chép 1GB dữ liệu, sẽ mất nhiều thời gian.

Go thường được sử dụng cho các máy chủ nhạy cảm với độ trễ, vì vậy chúng ta không muốn các thao tác trên các kiểu tích hợp sẵn có thể ảnh hưởng tùy ý lớn đến độ trễ đuôi.
Thay vào đó, các map Go tăng trưởng dần dần, sao cho mỗi lần chèn có giới hạn trên về lượng công việc tăng trưởng nó phải thực hiện.
Điều này giới hạn tác động độ trễ của một lần chèn map duy nhất.

Thật không may, thiết kế Swiss Table Abseil (C++) giả định tăng trưởng tất cả một lúc, và chuỗi probe phụ thuộc vào tổng số nhóm, khiến việc chia nhỏ trở nên khó khăn.

Map tích hợp sẵn của Go giải quyết điều này bằng một lớp gián tiếp khác bằng cách chia mỗi map thành nhiều Swiss Table.
Thay vì một Swiss Table duy nhất triển khai toàn bộ map, mỗi map bao gồm một hoặc nhiều bảng độc lập bao phủ một tập hợp con của không gian khóa.
Một bảng riêng lẻ lưu trữ tối đa 1024 mục.
Một số bit trên biến của hash được sử dụng để chọn bảng nào một khóa thuộc về.
Đây là một dạng của [*extendible hashing*](https://en.wikipedia.org/wiki/Extendible_hashing), trong đó số bit được sử dụng tăng lên khi cần thiết để phân biệt tổng số bảng.

Trong quá trình chèn, nếu một bảng riêng lẻ cần tăng trưởng, nó sẽ làm như vậy tất cả một lúc, nhưng các bảng khác không bị ảnh hưởng.
Do đó giới hạn trên cho một lần chèn duy nhất là độ trễ tăng trưởng một bảng 1024 mục thành hai bảng 1024 mục, sao chép 1024 mục.

### Thay đổi trong khi duyệt

Nhiều thiết kế bảng băm, bao gồm Swiss Tables của Abseil, cấm thay đổi map trong khi duyệt.
Đặc tả ngôn ngữ Go [cho phép rõ ràng](/ref/spec#For_statements:~:text=The%20iteration%20order,iterations%20is%200.) thay đổi trong khi duyệt, với các ngữ nghĩa sau:

* Nếu một mục bị xóa trước khi đến lượt nó, nó sẽ không được tạo ra.
* Nếu một mục được cập nhật trước khi đến lượt nó, giá trị đã cập nhật sẽ được tạo ra.
* Nếu một mục mới được thêm vào, nó có thể hoặc không được tạo ra.

Một cách tiếp cận điển hình để duyệt bảng băm là đơn giản đi qua mảng lưu trữ và tạo ra các giá trị theo thứ tự chúng được sắp xếp trong bộ nhớ.
Cách tiếp cận này vi phạm các ngữ nghĩa trên, đặc biệt là vì các lần chèn có thể tăng trưởng map, điều này sẽ xáo trộn bố cục bộ nhớ.

Chúng ta có thể tránh tác động của việc xáo trộn trong quá trình tăng trưởng bằng cách để iterator giữ tham chiếu đến bảng nó hiện đang duyệt.
Nếu bảng đó tăng trưởng trong quá trình duyệt, chúng ta tiếp tục sử dụng phiên bản cũ của bảng và do đó tiếp tục cung cấp các khóa theo thứ tự của bố cục bộ nhớ cũ.

Điều này có hoạt động với các ngữ nghĩa trên không?
Các mục mới được thêm vào sau khi tăng trưởng sẽ bị bỏ qua hoàn toàn, vì chúng chỉ được thêm vào bảng đã tăng trưởng, không phải bảng cũ.
Điều đó không sao, vì ngữ nghĩa cho phép các mục mới không được tạo ra.
Các cập nhật và xóa là vấn đề, tuy nhiên: sử dụng bảng cũ có thể tạo ra các mục lỗi thời hoặc đã xóa.

Trường hợp ngoại lệ này được giải quyết bằng cách chỉ sử dụng bảng cũ để xác định thứ tự duyệt.
Trước khi thực sự trả về mục, chúng ta tham khảo bảng đã tăng trưởng để xác định liệu mục còn tồn tại không, và để lấy giá trị mới nhất.

Điều này bao gồm tất cả các ngữ nghĩa cốt lõi, mặc dù còn có thêm nhiều trường hợp ngoại lệ nhỏ không được đề cập ở đây.
Cuối cùng, tính dễ chấp nhận của map Go với duyệt dẫn đến duyệt trở thành phần phức tạp nhất của triển khai map Go.

## Công việc tương lai

Trong [các microbenchmark](/issue/54766#issuecomment-2542444404), các thao tác map nhanh hơn tới 60% so với Go 1.23.
Cải thiện hiệu suất chính xác thay đổi khá nhiều do sự đa dạng rộng của các thao tác và cách sử dụng map, và một số trường hợp ngoại lệ thực sự bị tụt hậu so với Go 1.23.
Nhìn chung, trong các benchmark ứng dụng đầy đủ, chúng tôi tìm thấy cải thiện thời gian CPU trung bình hình học khoảng 1,5%.

Có thêm các cải tiến map mà chúng tôi muốn điều tra cho các bản phát hành Go trong tương lai.
Ví dụ, chúng tôi có thể [tăng tính cục bộ](/issue/70835) của các thao tác trên các map không nằm trong bộ đệm CPU.

Chúng tôi cũng có thể cải thiện thêm các so sánh control word.
Như đã mô tả ở trên, chúng tôi có một triển khai portable sử dụng các thao tác số học và bitwise tiêu chuẩn.
Tuy nhiên, một số kiến trúc có các lệnh SIMD thực hiện loại so sánh này trực tiếp.
Go 1.24 đã sử dụng các lệnh SIMD 8-byte cho amd64, nhưng chúng tôi có thể mở rộng hỗ trợ sang các kiến trúc khác.
Quan trọng hơn, trong khi các lệnh tiêu chuẩn hoạt động trên các word tối đa 8 byte, các lệnh SIMD gần như luôn hỗ trợ ít nhất các word 16 byte.
Điều này có nghĩa là chúng tôi có thể tăng kích thước nhóm lên 16 slot, và thực hiện 16 phép so sánh hash song song thay vì 8.
Điều này sẽ giảm thêm số lần probe trung bình cần thiết cho tra cứu.

## Lời cảm ơn

Một triển khai map Go dựa trên Swiss Table đã được chờ đợi từ lâu và liên quan đến nhiều người đóng góp.
Tôi muốn cảm ơn YunHao Zhang ([@zhangyunhao116](https://github.com/zhangyunhao116)), PJ Malloy ([@thepudds](https://github.com/thepudds)), và [@andy-wm-arthur](https://github.com/andy-wm-arthur) vì đã xây dựng các phiên bản ban đầu của triển khai Swiss Table Go.
Peter Mattis ([@petermattis](https://github.com/petermattis)) đã kết hợp những ý tưởng này với các giải pháp cho những thách thức Go ở trên để xây dựng [`github.com/cockroachdb/swiss`](https://pkg.go.dev/github.com/cockroachdb/swiss), một triển khai Swiss Table tuân thủ đặc tả Go.
Triển khai map tích hợp sẵn Go 1.24 dựa nhiều vào công việc của Peter.
Cảm ơn tất cả mọi người trong cộng đồng đã đóng góp!
