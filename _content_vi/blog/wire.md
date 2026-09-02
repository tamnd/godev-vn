---
title: Dependency Injection tại thời điểm biên dịch với Wire của Go Cloud
date: 2018-10-09
by:
- Robert van Gent
summary: Cách sử dụng Wire, một công cụ dependency injection cho Go.
---

## Tổng quan

Nhóm Go gần đây đã [thông báo](/blog/go-cloud) dự án mã nguồn mở [Go Cloud](https://github.com/google/go-cloud),
với các Cloud API di động và công cụ cho phát triển [open cloud](https://cloud.google.com/open-cloud/).
Bài viết này đi vào chi tiết hơn về Wire,
một công cụ dependency injection được sử dụng trong Go Cloud.

## Wire giải quyết vấn đề gì?

[Dependency injection](https://en.wikipedia.org/wiki/Dependency_injection)
là một kỹ thuật tiêu chuẩn để tạo ra mã linh hoạt và ít ràng buộc,
bằng cách cung cấp tường minh cho các thành phần tất cả các dependency chúng cần để hoạt động.
Trong Go, điều này thường có dạng truyền dependency vào các hàm khởi tạo:

	// NewUserStore returns a UserStore that uses cfg and db as dependencies.
	func NewUserStore(cfg *Config, db *mysql.DB) (*UserStore, error) {...}

Kỹ thuật này hoạt động tốt ở quy mô nhỏ,
nhưng các ứng dụng lớn hơn có thể có một đồ thị dependency phức tạp,
dẫn đến một khối mã khởi tạo lớn phụ thuộc vào thứ tự nhưng
không thú vị theo nghĩa khác.
Thường rất khó để tách mã này gọn gàng,
đặc biệt vì một số dependency được sử dụng nhiều lần.
Thay thế một triển khai của một dịch vụ bằng một triển khai khác có thể phiền phức vì
điều đó đòi hỏi sửa đổi đồ thị dependency bằng cách thêm một tập hợp hoàn toàn mới các
dependency (và các dependency của chúng...),
và xóa các dependency cũ không còn dùng.
Trong thực tế, việc thay đổi mã khởi tạo trong các ứng dụng có
đồ thị dependency lớn rất tẻ nhạt và chậm chạp.

Các công cụ dependency injection như Wire nhằm mục đích đơn giản hóa việc quản lý mã khởi tạo.
Bạn mô tả các dịch vụ của mình và các dependency của chúng,
dưới dạng mã hoặc cấu hình, sau đó Wire xử lý đồ thị kết quả
để xác định thứ tự và cách truyền cho mỗi dịch vụ những gì nó cần.
Thực hiện thay đổi đối với các dependency của ứng dụng bằng cách thay đổi chữ ký hàm
hoặc thêm hay xóa một trình khởi tạo,
và sau đó để Wire thực hiện công việc tẻ nhạt là tạo mã khởi tạo
cho toàn bộ đồ thị dependency.

## Tại sao đây là một phần của Go Cloud?

Mục tiêu của Go Cloud là giúp dễ dàng hơn khi viết ứng dụng Cloud di động
bằng cách cung cấp các API Go theo phong cách Go cho các dịch vụ Cloud hữu ích.
Ví dụ, [blob.Bucket](https://godoc.org/github.com/google/go-cloud/blob)
cung cấp một API lưu trữ với các triển khai cho Amazon S3 và Google Cloud Storage (GCS);
các ứng dụng được viết bằng `blob.Bucket` có thể hoán đổi triển khai mà không cần
thay đổi logic ứng dụng.
Tuy nhiên, mã khởi tạo vốn mang tính đặc thù của nhà cung cấp,
và mỗi nhà cung cấp có một tập hợp dependency khác nhau.

Ví dụ, [xây dựng `blob.Bucket` GCS](https://godoc.org/github.com/google/go-cloud/blob/gcsblob#OpenBucket)
yêu cầu một `gcp.HTTPClient`,
cuối cùng đòi hỏi `google.Credentials`,
trong khi [xây dựng cho S3](https://godoc.org/github.com/google/go-cloud/blob/s3blob)
yêu cầu một `aws.Config`,
cuối cùng đòi hỏi AWS credentials.
Do đó, việc cập nhật ứng dụng để sử dụng một triển khai `blob.Bucket` khác
đòi hỏi chính xác loại cập nhật tẻ nhạt đối với đồ thị dependency mà chúng tôi đã mô tả ở trên.
Trường hợp sử dụng chủ yếu cho Wire là giúp dễ dàng hoán đổi các triển khai
của các Go Cloud portable API,
nhưng nó cũng là một công cụ mục đích chung cho dependency injection.

## Điều này đã được thực hiện chưa?

Có một số framework dependency injection ngoài kia.
Cho Go, [dig của Uber](https://github.com/uber-go/dig) và [inject của Facebook](https://github.com/facebookgo/inject)
đều sử dụng reflection để thực hiện dependency injection tại thời điểm chạy.
Wire được lấy cảm hứng chủ yếu từ [Dagger 2](https://google.github.io/dagger/) của Java,
và sử dụng tạo mã thay vì reflection hay [service locator](https://en.wikipedia.org/wiki/Service_locator_pattern).

Chúng tôi nghĩ cách tiếp cận này có một số ưu điểm:

  - Dependency injection tại thời điểm chạy có thể khó theo dõi và gỡ lỗi khi
    đồ thị dependency trở nên phức tạp.
    Sử dụng tạo mã có nghĩa là mã khởi tạo được thực thi
    tại thời điểm chạy là mã Go thông thường, theo phong cách Go, dễ hiểu và gỡ lỗi.
    Không có gì bị che khuất bởi một framework can thiệp thực hiện "phép thuật".
    Cụ thể, các vấn đề như quên một dependency trở thành lỗi biên dịch,
    không phải lỗi khi chạy.
  - Không giống như [service locator](https://en.wikipedia.org/wiki/Service_locator_pattern),
    không cần phải nghĩ ra các tên hoặc khóa tùy ý để đăng ký dịch vụ.
    Wire sử dụng các kiểu Go để kết nối các thành phần với các dependency của chúng.
  - Dễ dàng hơn để tránh phình to dependency. Mã được tạo bởi Wire sẽ chỉ
    import các dependency bạn cần,
    vì vậy binary của bạn sẽ không có các import không dùng.
    Các dependency injector tại thời điểm chạy không thể xác định các dependency không dùng cho đến khi chạy.
  - Đồ thị dependency của Wire có thể biết được tĩnh, điều này tạo ra cơ hội cho hệ thống công cụ và trực quan hóa.

## Nó hoạt động như thế nào?

Wire có hai khái niệm cơ bản: provider và injector.

_Provider_ là các hàm Go thông thường "cung cấp" các giá trị được cung cấp bởi các dependency của chúng,
các dependency này được mô tả đơn giản là các tham số của hàm.
Đây là một số mã ví dụ định nghĩa ba provider:

	// NewUserStore is the same function we saw above; it is a provider for UserStore,
	// with dependencies on *Config and *mysql.DB.
	func NewUserStore(cfg *Config, db *mysql.DB) (*UserStore, error) {...}

	// NewDefaultConfig is a provider for *Config, with no dependencies.
	func NewDefaultConfig() *Config {...}

	// NewDB is a provider for *mysql.DB based on some connection info.
	func NewDB(info *ConnectionInfo) (*mysql.DB, error) {...}

Các provider thường được sử dụng cùng nhau có thể được nhóm vào `ProviderSet`.
Ví dụ, thường dùng `*Config` mặc định khi tạo `*UserStore`,
vì vậy chúng ta có thể nhóm `NewUserStore` và `NewDefaultConfig` vào một `ProviderSet`:

	var UserStoreSet = wire.ProviderSet(NewUserStore, NewDefaultConfig)

_Injector_ là các hàm được tạo ra gọi các provider theo thứ tự dependency.
Bạn viết chữ ký của injector, bao gồm bất kỳ đầu vào cần thiết nào làm đối số,
và chèn một lời gọi đến `wire.Build` với danh sách các provider hoặc provider
set cần thiết để xây dựng kết quả cuối cùng:

	func initUserStore() (*UserStore, error) {
		// We're going to get an error, because NewDB requires a *ConnectionInfo
		// and we didn't provide one.
		wire.Build(UserStoreSet, NewDB)
		return nil, nil  // These return values are ignored.
	}

Bây giờ chúng ta chạy go generate để thực thi wire:

	$ go generate
	wire.go:2:10: inject initUserStore: no provider found for ConnectionInfo (required by provider of *mysql.DB)
	wire: generate failed

Ối! Chúng ta chưa thêm `ConnectionInfo` hoặc cho Wire biết cách xây dựng nó.
Wire hữu ích chỉ cho chúng ta số dòng và các kiểu liên quan.
Chúng ta có thể thêm một provider cho nó vào `wire.Build`,
hoặc thêm nó như một đối số:

	func initUserStore(info ConnectionInfo) (*UserStore, error) {
		wire.Build(UserStoreSet, NewDB)
		return nil, nil  // These return values are ignored.
	}

Bây giờ `go generate` sẽ tạo một tệp mới với mã được tạo ra:

	// File: wire_gen.go
	// Code generated by Wire. DO NOT EDIT.
	//go:generate wire
	//+build !wireinject

	func initUserStore(info ConnectionInfo) (*UserStore, error) {
		defaultConfig := NewDefaultConfig()
		db, err := NewDB(info)
		if err != nil {
			return nil, err
		}
		userStore, err := NewUserStore(defaultConfig, db)
		if err != nil {
			return nil, err
		}
		return userStore, nil
	}

Bất kỳ khai báo nào không phải injector đều được sao chép vào tệp được tạo.
Không có dependency vào Wire tại thời điểm chạy:
tất cả mã được viết chỉ là mã Go thông thường.

Như bạn có thể thấy, đầu ra rất gần với những gì một nhà phát triển tự viết.
Đây là một ví dụ đơn giản chỉ với ba thành phần,
vì vậy viết trình khởi tạo bằng tay sẽ không quá khó,
nhưng Wire tiết kiệm rất nhiều công sức thủ công cho các thành phần và ứng dụng có
đồ thị dependency phức tạp hơn.

## Làm thế nào để tham gia và tìm hiểu thêm?

[Wire README](https://github.com/google/wire/blob/master/README.md)
đi vào chi tiết hơn về cách sử dụng Wire và các tính năng nâng cao hơn của nó.
Cũng có một [hướng dẫn](https://github.com/google/wire/tree/master/_tutorial)
hướng dẫn sử dụng Wire trong một ứng dụng đơn giản.

Chúng tôi trân trọng mọi ý kiến đóng góp của bạn về trải nghiệm với Wire!
Việc phát triển [Wire](https://github.com/google/wire) được thực hiện trên GitHub,
vì vậy bạn có thể [gửi một issue](https://github.com/google/wire/issues/new/choose)
để cho chúng tôi biết điều gì có thể tốt hơn.
Để cập nhật và thảo luận về dự án,
hãy tham gia [danh sách thư Go Cloud](https://groups.google.com/forum/#!forum/go-cloud).

Cảm ơn bạn đã dành thời gian tìm hiểu về Wire của Go Cloud.
Chúng tôi rất hào hứng được làm việc với bạn để làm cho Go trở thành ngôn ngữ được lựa chọn cho các nhà phát triển
xây dựng ứng dụng cloud di động.
