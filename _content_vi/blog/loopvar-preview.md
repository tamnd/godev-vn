---
title: Sửa vòng lặp For trong Go 1.22
date: 2023-09-19
by:
- David Chase
- Russ Cox
summary: Go 1.21 đã phát hành bản xem trước của một thay đổi trong Go 1.22 nhằm giúp vòng lặp for ít gây ra lỗi hơn.
template: true
---

Go 1.21 bao gồm bản xem trước của một thay đổi đối với phạm vi vòng lặp `for`
mà chúng tôi dự định đưa vào Go 1.22,
loại bỏ một trong những lỗi phổ biến nhất trong Go.

## Vấn đề

Nếu bạn đã từng viết bất kỳ lượng code Go nào, bạn có thể đã mắc lỗi
giữ lại tham chiếu đến biến vòng lặp sau khi vòng lặp đó kết thúc,
lúc đó biến đã nhận giá trị mới mà bạn không mong muốn.
Ví dụ, hãy xem xét chương trình sau:

{{raw `
	func main() {
		done := make(chan bool)

		values := []string{"a", "b", "c"}
		for _, v := range values {
			go func() {
				fmt.Println(v)
				done <- true
			}()
		}

		// wait for all goroutines to complete before exiting
		for _ = range values {
			<-done
		}
	}
`}}

Ba goroutine được tạo ra đều in cùng một biến `v`,
do đó chúng thường in ra "c", "c", "c", thay vì in "a", "b", và "c" theo một thứ tự nào đó.

[Mục FAQ của Go "Điều gì xảy ra với các closure chạy như goroutine?"](/doc/faq#closures_and_goroutines)
đưa ra ví dụ này và nhận xét
"Một số nhầm lẫn có thể nảy sinh khi sử dụng closure với concurrency."

Mặc dù concurrency thường liên quan, nhưng không nhất thiết phải vậy.
Ví dụ này có cùng vấn đề nhưng không có goroutine nào:

{{raw `
	func main() {
		var prints []func()
		for i := 1; i <= 3; i++ {
			prints = append(prints, func() { fmt.Println(i) })
		}
		for _, print := range prints {
			print()
		}
	}
`}}

Loại lỗi này đã gây ra vấn đề trên môi trường production tại nhiều công ty,
bao gồm một
[sự cố được ghi nhận công khai tại Lets Encrypt](https://bugzilla.mozilla.org/show_bug.cgi?id=1619047).
Trong trường hợp đó, việc vô tình bắt giữ biến vòng lặp được trải rộng qua
nhiều hàm và khó nhận ra hơn nhiều:

	// authz2ModelMapToPB converts a mapping of domain name to authz2Models into a
	// protobuf authorizations map
	func authz2ModelMapToPB(m map[string]authz2Model) (*sapb.Authorizations, error) {
		resp := &sapb.Authorizations{}
		for k, v := range m {
			// Make a copy of k because it will be reassigned with each loop.
			kCopy := k
			authzPB, err := modelToAuthzPB(&v)
			if err != nil {
				return nil, err
			}
			resp.Authz = append(resp.Authz, &sapb.Authorizations_MapElement{
				Domain: &kCopy,
				Authz: authzPB,
			})
		}
		return resp, nil
	}

Tác giả của đoạn code này rõ ràng đã hiểu vấn đề tổng quát, vì họ đã tạo bản sao của `k`,
nhưng hóa ra `modelToAuthzPB` đã dùng con trỏ tới các trường trong `v` khi tạo kết quả,
vì vậy vòng lặp cũng cần tạo bản sao của `v`.

Các công cụ đã được viết để nhận diện những lỗi này, nhưng rất khó phân tích
liệu các tham chiếu đến một biến có vượt quá vòng lặp của nó hay không.
Các công cụ này phải chọn giữa âm tính giả và dương tính giả.
Bộ phân tích `loopclosure` được sử dụng bởi `go vet` và `gopls` chọn âm tính giả,
chỉ báo cáo khi chắc chắn có vấn đề nhưng bỏ sót những trường hợp khác.
Các công cụ kiểm tra khác chọn dương tính giả, cáo buộc code đúng là sai.
Chúng tôi đã chạy phân tích các commit thêm dòng `x := x` trong code Go mã nguồn mở,
kỳ vọng tìm thấy các bản vá lỗi.
Thay vào đó, chúng tôi tìm thấy nhiều dòng không cần thiết được thêm vào,
cho thấy rằng các công cụ kiểm tra phổ biến có tỷ lệ dương tính giả đáng kể,
nhưng các lập trình viên vẫn thêm dòng để giữ cho công cụ kiểm tra hài lòng.

Một cặp ví dụ chúng tôi tìm thấy đặc biệt rõ ràng:

Diff này có trong một chương trình:

	     for _, informer := range c.informerMap {
	+        informer := informer
	         go informer.Run(stopCh)
	     }

Và diff này có trong một chương trình khác:

	     for _, a := range alarms {
	+        a := a
	         go a.Monitor(b)
	     }

Một trong hai diff này là bản vá lỗi; cái còn lại là thay đổi không cần thiết.
Bạn không thể biết cái nào là cái nào trừ khi bạn biết thêm về các kiểu
và hàm liên quan.

## Giải pháp

Với Go 1.22, chúng tôi dự định thay đổi vòng lặp `for` để các biến này có
phạm vi theo từng lần lặp thay vì phạm vi toàn vòng lặp.
Thay đổi này sẽ sửa các ví dụ trên, để chúng không còn là chương trình Go có lỗi nữa;
nó sẽ chấm dứt các vấn đề production gây ra bởi những lỗi như vậy;
và nó sẽ loại bỏ sự cần thiết của các công cụ thiếu chính xác nhắc nhở người dùng
thực hiện những thay đổi không cần thiết đối với code của họ.

Để đảm bảo tương thích ngược với code hiện có, ngữ nghĩa mới
sẽ chỉ áp dụng cho các package chứa trong các module khai báo `go 1.22` hoặc
cao hơn trong các tệp `go.mod` của chúng.
Quyết định theo module này cung cấp cho lập trình viên quyền kiểm soát việc cập nhật dần dần
sang ngữ nghĩa mới trong toàn bộ codebase.
Cũng có thể sử dụng dòng `//go:build` để kiểm soát quyết định trên cơ sở
từng tệp.

Code cũ sẽ tiếp tục có ý nghĩa chính xác như ngày hôm nay:
sửa chữa chỉ áp dụng cho code mới hoặc được cập nhật.
Điều này sẽ cho phép các lập trình viên kiểm soát khi nào ngữ nghĩa thay đổi
trong một package cụ thể.
Là hệ quả của [công việc tương thích tiến](toolchain),
Go 1.21 sẽ không cố gắng biên dịch code khai báo `go 1.22` hoặc cao hơn.
Chúng tôi đã bao gồm một trường hợp đặc biệt với hiệu quả tương tự trong
các bản phát hành điểm Go 1.20.8 và Go 1.19.13,
vì vậy khi Go 1.22 được phát hành,
code được viết phụ thuộc vào ngữ nghĩa mới sẽ không bao giờ được biên dịch với
ngữ nghĩa cũ, trừ khi mọi người đang dùng các phiên bản Go rất cũ, [không còn được hỗ trợ](/doc/devel/release#policy).


## Xem trước giải pháp

Go 1.21 bao gồm bản xem trước của thay đổi phạm vi.
Nếu bạn biên dịch code với `GOEXPERIMENT=loopvar` được đặt trong môi trường của bạn,
thì ngữ nghĩa mới được áp dụng cho tất cả các vòng lặp
(bỏ qua các dòng `go` trong `go.mod`).
Ví dụ, để kiểm tra xem các test của bạn có còn pass với ngữ nghĩa vòng lặp mới
được áp dụng cho package và tất cả các dependency của bạn:

	GOEXPERIMENT=loopvar go test

Chúng tôi đã vá bộ công cụ Go nội bộ tại Google để bắt buộc chế độ này trong tất cả các build
từ đầu tháng 5 năm 2023, và trong bốn tháng qua
chúng tôi chưa nhận được báo cáo nào về bất kỳ vấn đề nào trong code production.

Bạn cũng có thể thử các chương trình thử nghiệm để hiểu rõ hơn về ngữ nghĩa
trên Go playground bằng cách thêm comment `// GOEXPERIMENT=loopvar`
ở đầu chương trình, như trong [chương trình này](/play/p/YchKkkA1ETH).
(Comment này chỉ áp dụng trong Go playground.)

## Sửa các test có lỗi

Mặc dù chúng tôi không có vấn đề gì trên production,
để chuẩn bị cho việc chuyển đổi đó, chúng tôi đã phải sửa nhiều test có lỗi không
kiểm tra những gì chúng nghĩ đang kiểm tra, như thế này:

	func TestAllEvenBuggy(t *testing.T) {
		testCases := []int{1, 2, 4, 6}
		for _, v := range testCases {
			t.Run("sub", func(t *testing.T) {
				t.Parallel()
				if v&1 != 0 {
					t.Fatal("odd v", v)
				}
			})
		}
	}

Trong Go 1.21, test này pass vì `t.Parallel` chặn từng subtest
cho đến khi toàn bộ vòng lặp kết thúc và sau đó chạy tất cả các subtest
song song. Khi vòng lặp kết thúc, `v` luôn là 6,
vì vậy các subtest đều kiểm tra xem 6 có là số chẵn không,
nên test pass.
Tất nhiên, test này thực sự nên fail, vì 1 không phải là số chẵn.
Việc sửa vòng lặp for làm lộ ra loại test có lỗi này.

Để giúp chuẩn bị cho loại phát hiện này, chúng tôi đã cải thiện độ chính xác
của bộ phân tích `loopclosure` trong Go 1.21 để nó có thể nhận diện và
báo cáo vấn đề này.
Bạn có thể xem báo cáo [trong chương trình này](/play/p/WkJkgXRXg0m) trên Go playground.
Nếu `go vet` đang báo cáo loại vấn đề này trong các test của bạn,
việc sửa chúng sẽ chuẩn bị bạn tốt hơn cho Go 1.22.

Nếu bạn gặp phải các vấn đề khác,
[FAQ](/wiki/LoopvarExperiment#my-test-fails-with-the-change-how-can-i-debug-it)
có các liên kết đến các ví dụ và chi tiết về việc sử dụng một công cụ chúng tôi đã viết để nhận diện
vòng lặp cụ thể nào gây ra test fail khi ngữ nghĩa mới được áp dụng.

## Thêm thông tin

Để biết thêm thông tin về thay đổi, hãy xem
[tài liệu thiết kế](https://go.googlesource.com/proposal/+/master/design/60078-loopvar.md)
và
[FAQ](/wiki/LoopvarExperiment).
