---
title: Cơ sở dữ liệu lỗ hổng bảo mật Go
layout: article
---

[Quay lại Quản lý lỗ hổng bảo mật Go](/security/vuln)

## Tổng quan

Cơ sở dữ liệu lỗ hổng bảo mật Go ([https://vuln.go.dev](https://vuln.go.dev))
cung cấp thông tin lỗ hổng bảo mật Go theo
[schema Open Source Vulnerability (OSV)](https://ossf.github.io/osv-schema/).

Bạn cũng có thể duyệt các lỗ hổng bảo mật trong cơ sở dữ liệu tại [pkg.go.dev/vuln](https://pkg.go.dev/vuln).

**Không** dựa vào nội dung của kho lưu trữ Git x/vulndb. Các tệp YAML trong
kho lưu trữ đó được duy trì bằng định dạng nội bộ có thể thay đổi
mà không có cảnh báo.

## Đóng góp

Chúng tôi rất mong tất cả các nhà duy trì gói Go [đóng góp](/s/vulndb-report-new)
thông tin về các lỗ hổng bảo mật công khai trong các dự án của riêng họ,
và [cập nhật](/s/vulndb-report-feedback) thông tin hiện có về các lỗ hổng bảo mật
trong các gói Go của họ.

Chúng tôi hướng đến việc làm cho việc báo cáo trở thành một quy trình ít rào cản,
vì vậy hãy thoải mái [gửi cho chúng tôi các đề xuất của bạn](/s/vuln-feedback).

Vui lòng **không** sử dụng các biểu mẫu trên để báo cáo lỗ hổng bảo mật trong thư viện chuẩn
hoặc các sub-repository của Go.
Thay vào đó, hãy tuân theo quy trình tại [go.dev/security/policy](/security/policy)
cho các lỗ hổng bảo mật về dự án Go.

## API

Cơ sở dữ liệu lỗ hổng bảo mật Go chính thức, [https://vuln.go.dev](https://vuln.go.dev),
là một máy chủ HTTP có thể phản hồi các yêu cầu GET cho các endpoint được chỉ định bên dưới.

Các endpoint không có tham số truy vấn, và không yêu cầu header cụ thể nào.
Vì điều này, ngay cả một trang web phục vụ từ hệ thống tệp cố định (bao gồm URL `file://`)
cũng có thể triển khai API này.

Mỗi endpoint trả về phản hồi được mã hóa JSON, ở dạng không nén
(nếu được yêu cầu dưới dạng `.json`) hoặc dạng nén gzip (nếu được yêu cầu dưới dạng `.json.gz`).

Các endpoint là:

- `/index/db.json[.gz]`

  Trả về metadata về cơ sở dữ liệu:

  ```json
  {
    // Thời điểm mới nhất cơ sở dữ liệu được coi là
    // đã được sửa đổi, dưới dạng dấu thời gian UTC được định dạng RFC3339
    // kết thúc bằng "Z".
    "modified": string
  }
  ```

  Lưu ý rằng thời gian sửa đổi *không nên* được so sánh với thời gian thực,
  ví dụ: cho mục đích vô hiệu hóa cache, vì có thể có độ trễ khi thực hiện
  các sửa đổi cơ sở dữ liệu.

  Xem [/index/db.json](https://vuln.go.dev/index/db.json) để có ví dụ trực tiếp.

- `/index/modules.json[.gz]`

  Trả về danh sách chứa metadata về mỗi module trong cơ sở dữ liệu:

  ```json
  [ {
    // Đường dẫn module.
    "path": string,
    // Các lỗ hổng bảo mật ảnh hưởng đến module này.
    "vulns":
      [ {
        // ID lỗ hổng bảo mật.
        "id": string,
        // Thời điểm mới nhất lỗ hổng bảo mật được coi là
        // đã được sửa đổi, dưới dạng dấu thời gian UTC được định dạng RFC3339
        // kết thúc bằng "Z".
        "modified": string,
        // (Tùy chọn) Phiên bản module (ở định dạng SemVer 2.0.0)
        // chứa bản vá mới nhất cho lỗ hổng bảo mật.
        // Nếu không biết hoặc không có, nên bỏ qua.
        "fixed": string,
      } ]
  } ]
  ```

  Xem [/index/modules.json](https://vuln.go.dev/index/modules.json) để có ví dụ trực tiếp.

- `/index/vulns.json[.gz]`

  Trả về danh sách chứa metadata về mỗi lỗ hổng bảo mật trong cơ sở dữ liệu:

  ```json
   [ {
       // ID lỗ hổng bảo mật.
       "id": string,
       // Thời điểm mới nhất lỗ hổng bảo mật được coi là
       // đã được sửa đổi, dưới dạng dấu thời gian UTC được định dạng RFC3339
       // kết thúc bằng "Z".
       "modified": string,
       // Danh sách ID của cùng một lỗ hổng bảo mật trong các cơ sở dữ liệu khác.
       "aliases": [ string ]
   } ]
  ```

  Xem [/index/vulns.json](https://vuln.go.dev/index/vulns.json) để có ví dụ trực tiếp.

- `/ID/$id.json[.gz]`

  Trả về báo cáo riêng lẻ cho lỗ hổng bảo mật với ID `$id`,
  ở định dạng OSV (được mô tả bên dưới trong [Schema](#schema)).

  Xem [/ID/GO-2022-0191.json](https://vuln.go.dev/ID/GO-2022-0191.json)
  để có ví dụ trực tiếp.

### Tải xuống hàng loạt

Để dễ dàng tải xuống toàn bộ cơ sở dữ liệu lỗ hổng bảo mật Go,
một tệp zip chứa tất cả các tệp index và OSV có sẵn tại
[vuln.go.dev/vulndb.zip](https://vuln.go.dev/vulndb.zip).

### Sử dụng trong `govulncheck`

Theo mặc định, `govulncheck` sử dụng cơ sở dữ liệu lỗ hổng bảo mật Go chính thức tại [vuln.go.dev](https://vuln.go.dev).

Lệnh có thể được cấu hình để liên hệ với một cơ sở dữ liệu lỗ hổng bảo mật khác bằng cờ `-db`, chấp nhận URL cơ sở dữ liệu lỗ hổng bảo mật với giao thức `http://`, `https://`, hoặc `file://`.

Để hoạt động chính xác với `govulncheck`, cơ sở dữ liệu lỗ hổng bảo mật được chỉ định phải triển khai API được mô tả ở trên. Lệnh `govulncheck` sử dụng các endpoint được nén ".json.gz" khi đọc từ nguồn http(s), và các endpoint ".json" khi đọc từ nguồn tệp.

### API kế thừa

Cơ sở dữ liệu chính thức chứa một số endpoint bổ sung là một phần của API kế thừa.
Chúng tôi có kế hoạch sớm xóa hỗ trợ cho các endpoint này. Nếu bạn đang dựa vào API kế thừa
và cần thêm thời gian để di chuyển, [hãy cho chúng tôi biết](/s/govulncheck-feedback).

## Schema

Các báo cáo sử dụng
[schema Open Source Vulnerability (OSV)](https://ossf.github.io/osv-schema/).
Cơ sở dữ liệu lỗ hổng bảo mật Go gán các ý nghĩa sau cho các trường:

### id

Trường id là một định danh duy nhất cho mục lỗ hổng bảo mật. Đây là một chuỗi
có định dạng GO-\<YEAR>-\<ENTRYID>.

### affected

Trường [affected](https://ossf.github.io/osv-schema/#affected-fields) là một
mảng JSON chứa các đối tượng mô tả các phiên bản module chứa
lỗ hổng bảo mật.

#### affected[].package

Trường
[affected[].package](https://ossf.github.io/osv-schema/#affectedpackage-field)
là một đối tượng JSON xác định _module_ bị ảnh hưởng. Đối tượng có hai
trường bắt buộc:

- **ecosystem**: sẽ luôn là "Go"
- **name**: đây là đường dẫn module Go
  - Các gói có thể import trong thư viện chuẩn sẽ có tên _stdlib_.
  - Lệnh go sẽ có tên _toolchain_.

#### affected[].ecosystem_specific

Trường
[affected[].ecosystem_specific](https://ossf.github.io/osv-schema/#affectedecosystem_specific-field)
là một đối tượng JSON với thông tin bổ sung về lỗ hổng bảo mật,
được sử dụng bởi các công cụ phát hiện lỗ hổng bảo mật của Go.

Hiện tại, ecosystem specific sẽ luôn là một đối tượng với một trường duy nhất,
`imports`.

##### affected[].ecosystem_specific.imports

Trường `affected[].ecosystem_specific.imports` là một mảng JSON chứa
các gói và ký hiệu bị ảnh hưởng bởi lỗ hổng bảo mật. Mỗi đối tượng trong
mảng sẽ có hai trường sau:

- **path:** một chuỗi với đường dẫn import của gói chứa lỗ hổng bảo mật
- **symbols:** một mảng chuỗi với các tên của các ký hiệu (hàm hoặc phương thức) chứa lỗ hổng bảo mật
- **goos**: một mảng chuỗi với hệ điều hành thực thi nơi các ký hiệu xuất hiện, nếu biết
- **goarch**: một mảng chuỗi với kiến trúc nơi các ký hiệu xuất hiện, nếu biết

### database_specific

Trường `database_specific` chứa các trường tùy chỉnh dành riêng cho cơ sở dữ liệu lỗ hổng bảo mật Go.

#### database_specific.url

Trường `database_specific.url` là một chuỗi đại diện cho
URL đầy đủ của báo cáo lỗ hổng bảo mật Go, ví dụ: "https://pkg.go.dev/vuln/GO-2023-1621".

#### database_specific.review_status

Trường `database_specific.review_status` là một chuỗi đại diện cho trạng thái xem xét
của báo cáo lỗ hổng bảo mật. Nếu không có, báo cáo nên được
coi là `REVIEWED`. Các giá trị có thể là:

- `UNREVIEWED`: Báo cáo được tạo tự động dựa trên một nguồn khác, chẳng hạn như
CVE hoặc GHSA. Dữ liệu của nó có thể bị hạn chế và chưa được nhóm Go xác minh.
- `REVIEWED`: Báo cáo bắt nguồn từ nhóm Go, hoặc được tạo dựa trên nguồn bên ngoài.
Một thành viên của nhóm Go đã xem xét báo cáo và khi thích hợp, đã thêm dữ liệu bổ sung.

Để biết thông tin về các trường khác trong schema, hãy tham khảo [đặc tả OSV](https://ossf.github.io/osv-schema).

## Lưu ý về Phiên bản

Công cụ của chúng tôi cố gắng tự động ánh xạ các module và phiên bản trong
các khuyến nghị nguồn thành các module và phiên bản Go chính thức, phù hợp với
[số phiên bản module Go](/doc/modules/version-numbers) tiêu chuẩn. Các công cụ như
`govulncheck` được thiết kế để dựa vào các phiên bản tiêu chuẩn này để xác định
liệu một dự án Go có bị ảnh hưởng bởi lỗ hổng bảo mật trong một dependency hay không.

Trong một số trường hợp, chẳng hạn như khi một dự án Go sử dụng sơ đồ phiên bản riêng của nó,
việc ánh xạ sang các phiên bản Go tiêu chuẩn có thể thất bại. Khi điều này xảy ra, báo cáo cơ sở dữ liệu lỗ hổng bảo mật Go có thể liệt kê thận trọng tất cả các phiên bản Go là bị ảnh hưởng. Điều này đảm bảo rằng các công cụ như `govulncheck` không bỏ qua báo cáo lỗ hổng bảo mật do phạm vi phiên bản không được nhận ra (dương tính âm giả).
Tuy nhiên, việc liệt kê thận trọng tất cả các phiên bản là bị ảnh hưởng có thể khiến các công cụ
báo cáo không chính xác một phiên bản đã vá của module là chứa lỗ hổng bảo mật
(dương tính giả).

Nếu bạn tin rằng `govulncheck` đang báo cáo không chính xác (hoặc không báo cáo) một
lỗ hổng bảo mật, hãy
[đề xuất chỉnh sửa](https://github.com/golang/vulndb/issues/new?assignees=&labels=Needs+Triage%2CSuggested+Edit&template=suggest_edit.yaml&title=x%2Fvulndb%3A+suggestion+regarding+GO-2024-2965&report=GO-XXXX-YYYY)
cho báo cáo lỗ hổng bảo mật và chúng tôi sẽ xem xét nó.

## Ví dụ

Tất cả các lỗ hổng bảo mật trong cơ sở dữ liệu lỗ hổng bảo mật Go đều sử dụng schema OSV
được mô tả ở trên.

Xem các liên kết bên dưới để có ví dụ về các lỗ hổng bảo mật Go khác nhau:

- **Lỗ hổng bảo mật trong thư viện chuẩn Go** (GO-2022-0191):
  [JSON](https://vuln.go.dev/ID/GO-2022-0191.json),
  [HTML](https://pkg.go.dev/vuln/GO-2022-0191)
- **Lỗ hổng bảo mật trong toolchain Go** (GO-2022-0189):
  [JSON](https://vuln.go.dev/ID/GO-2022-0189.json),
  [HTML](https://pkg.go.dev/vuln/GO-2022-0189)
- **Lỗ hổng bảo mật trong module Go** (GO-2020-0015):
  [JSON](https://vuln.go.dev/ID/GO-2020-0015.json),
  [HTML](https://pkg.go.dev/vuln/GO-2020-0015)

## Báo cáo bị loại trừ

Các báo cáo trong cơ sở dữ liệu lỗ hổng bảo mật Go được thu thập từ các nguồn khác nhau
và được chọn lọc bởi nhóm Bảo mật Go. Chúng tôi có thể gặp phải một khuyến nghị lỗ hổng bảo mật
(ví dụ: CVE hoặc GHSA) và chọn loại trừ nó vì nhiều lý do.
Trong những trường hợp này, một báo cáo tối giản sẽ được tạo trong kho lưu trữ x/vulndb, tại
[x/vulndb/data/excluded](https://github.com/golang/vulndb/tree/master/data/excluded).

Các báo cáo có thể bị loại trừ vì những lý do sau:

- `NOT_GO_CODE`: Lỗ hổng bảo mật không nằm trong gói Go,
  nhưng nó được đánh dấu là khuyến nghị bảo mật cho hệ sinh thái Go bởi một nguồn khác.
  Lỗ hổng bảo mật này không thể ảnh hưởng đến bất kỳ
  gói Go nào. (Ví dụ: lỗ hổng bảo mật trong thư viện C++.)
- `NOT_IMPORTABLE`: Lỗ hổng bảo mật xảy ra trong package `main`, một package
  `internal/` chỉ được import bởi package `main`, hoặc một số vị trí khác không thể
  bao giờ được import bởi một module khác.
- `EFFECTIVELY_PRIVATE`: Mặc dù lỗ hổng bảo mật xảy ra trong một gói Go có thể
  được import bởi một module khác, nhưng gói này không được thiết kế để sử dụng bên ngoài
  và không có khả năng bao giờ được import bên ngoài module nơi nó được
  định nghĩa.
- `DEPENDENT_VULNERABILITY`: Lỗ hổng bảo mật này là một phần của một
  lỗ hổng bảo mật khác trong cơ sở dữ liệu. Ví dụ: nếu gói A chứa một
  lỗ hổng bảo mật, gói B phụ thuộc vào gói A, và có các ID CVE riêng biệt
  cho gói A và B, chúng tôi có thể đánh dấu báo cáo cho B là một lỗ hổng bảo mật
  phụ thuộc hoàn toàn được thay thế bởi báo cáo cho A.
- `NOT_A_VULNERABILITY`: Mặc dù đã được gán một ID CVE hoặc GHSA, nhưng không có
  lỗ hổng bảo mật đã biết nào liên quan đến nó.
- `WITHDRAWN`: Lỗ hổng bảo mật đã được rút lại bởi nguồn của nó.

Hiện tại, các báo cáo bị loại trừ không được phục vụ qua
API [vuln.go.dev](https://vuln.go.dev). Tuy nhiên, nếu bạn có
một trường hợp sử dụng cụ thể và sẽ hữu ích khi có quyền truy cập vào thông tin này
thông qua API,
[hãy cho chúng tôi biết](/s/govulncheck-feedback).
