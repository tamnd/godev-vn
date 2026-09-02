---
title: Go trên ARM và xa hơn nữa
date: 2020-12-17
by:
- Russ Cox
summary: Hỗ trợ của Go cho ARM64 và các kiến trúc khác
---


Ngành công nghiệp đang xôn xao về các bộ xử lý không phải x86 gần đây,
vì vậy chúng tôi nghĩ sẽ đáng để có một bài đăng ngắn về sự hỗ trợ của Go cho chúng.

Chúng tôi luôn coi tính di động là điều quan trọng đối với Go,
không bó buộc quá nhiều vào bất kỳ hệ điều hành hay kiến trúc cụ thể nào.
[Bản phát hành mã nguồn mở đầu tiên của Go](https://opensource.googleblog.com/2009/11/hey-ho-lets-go.html)
đã bao gồm hỗ trợ cho hai hệ điều hành (Linux và Mac OS X) và ba
kiến trúc (x86 64-bit, x86 32-bit và ARM 32-bit).

Qua nhiều năm, chúng tôi đã bổ sung hỗ trợ cho nhiều kết hợp hệ điều hành và kiến trúc hơn:

- Go 1 (tháng 3 năm 2012) hỗ trợ các hệ thống gốc cũng như FreeBSD,
  NetBSD và OpenBSD trên x86 64-bit và 32-bit,
  cùng Plan 9 trên x86 32-bit.
- Go 1.3 (tháng 6 năm 2014) thêm hỗ trợ cho Solaris trên x86 64-bit.
- Go 1.4 (tháng 12 năm 2014) thêm hỗ trợ cho Android trên ARM 32-bit và Plan 9 trên x86 64-bit.
- Go 1.5 (tháng 8 năm 2015) thêm hỗ trợ cho Linux trên ARM 64-bit và PowerPC 64-bit,
  cũng như iOS trên ARM 32-bit và 64-bit.
- Go 1.6 (tháng 2 năm 2016) thêm hỗ trợ cho Linux trên MIPS 64-bit,
  cũng như Android trên x86 32-bit.
  Nó cũng thêm một bản tải về nhị phân chính thức cho Linux trên ARM 32-bit,
  chủ yếu cho các hệ thống Raspberry Pi.
- Go 1.7 (tháng 8 năm 2016) thêm hỗ trợ cho Linux trên z Systems (S390x) và Plan 9 trên ARM 32-bit.
- Go 1.8 (tháng 2 năm 2017) thêm hỗ trợ cho Linux trên MIPS 32-bit,
  và nó đã thêm các bản tải về nhị phân chính thức cho Linux trên PowerPC 64-bit và z Systems.
- Go 1.9 (tháng 8 năm 2017) thêm các bản tải về nhị phân chính thức cho Linux trên ARM 64-bit.
- Go 1.12 (tháng 2 năm 2018) thêm hỗ trợ cho Windows 10 IoT Core trên ARM 32-bit,
  chẳng hạn như Raspberry Pi 3.
  Nó cũng thêm hỗ trợ cho AIX trên PowerPC 64-bit.
- Go 1.14 (tháng 2 năm 2019) thêm hỗ trợ cho Linux trên RISC-V 64-bit.

Mặc dù cổng x86-64 nhận được sự chú ý nhiều nhất trong những ngày đầu của Go,
ngày nay tất cả các kiến trúc mục tiêu của chúng tôi đều được hỗ trợ tốt bởi [back end compiler dựa trên SSA](https://www.youtube.com/watch?v=uTMvKVma5ms)
và tạo ra code xuất sắc.
Chúng tôi đã được giúp đỡ dọc đường bởi nhiều người đóng góp,
bao gồm các kỹ sư từ Amazon, ARM, Atos,
IBM, Intel và MIPS.

Go hỗ trợ cross-compiling cho tất cả các hệ thống này một cách sẵn sàng với nỗ lực tối thiểu.
Ví dụ, để xây dựng một ứng dụng cho Windows dựa trên x86 32-bit từ hệ thống Linux 64-bit:

	GOARCH=386 GOOS=windows go build myapp  # writes myapp.exe

Trong năm qua, một số nhà cung cấp lớn đã thông báo về phần cứng ARM64 mới
cho máy chủ, laptop và máy tính dành cho lập trình viên.
Go đã ở vị thế thuận lợi cho điều này. Nhiều năm nay,
Go đã cung cấp năng lực cho Docker, Kubernetes và phần còn lại của hệ sinh thái Go
trên các máy chủ Linux ARM64,
cũng như các ứng dụng di động trên thiết bị Android và iOS ARM64.

Kể từ thông báo của Apple về việc chuyển Mac sang Apple Silicon trong mùa hè này,
Apple và Google đã hợp tác để đảm bảo rằng Go và hệ sinh thái Go rộng lớn hơn
hoạt động tốt trên chúng,
cả khi chạy các nhị phân Go x86 dưới Rosetta 2 và khi chạy các nhị phân Go ARM64 gốc.
Đầu tuần này, chúng tôi đã phát hành bản beta Go 1.16 đầu tiên,
bao gồm hỗ trợ gốc cho Mac sử dụng chip M1.
Bạn có thể tải và thử bản beta Go 1.16 cho Mac M1 và tất cả các hệ thống khác của bạn
trên [trang tải về Go](/dl/#go1.16beta1).
(Tất nhiên, đây là bản beta và, như tất cả các bản beta,
chắc chắn có những lỗi mà chúng tôi chưa biết.
Nếu bạn gặp bất kỳ sự cố nào, hãy báo cáo tại [golang.org/issue/new](/issue/new).)

Sẽ luôn tốt khi sử dụng cùng kiến trúc CPU cho phát triển cục bộ như trong môi trường production,
để loại bỏ một biến thể giữa hai môi trường.
Nếu bạn triển khai đến các máy chủ production ARM64,
Go giúp dễ dàng phát triển trên các hệ thống Linux và Mac ARM64 cũng vậy.
Nhưng tất nhiên, vẫn dễ dàng như trước để làm việc trên một hệ thống và cross-compile
để triển khai sang hệ thống khác,
dù bạn đang làm việc trên hệ thống x86 và triển khai sang ARM,
làm việc trên Windows và triển khai sang Linux,
hoặc một số kết hợp khác.

Mục tiêu tiếp theo mà chúng tôi muốn thêm hỗ trợ là các hệ thống ARM64 Windows 10.
Nếu bạn có chuyên môn và muốn giúp đỡ,
chúng tôi đang phối hợp công việc trên [golang.org/issue/36439](/issue/36439).
