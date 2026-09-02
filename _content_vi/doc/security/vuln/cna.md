---
title: Chính sách Go CNA
layout: article
---

[Quay lại Quản lý lỗ hổng bảo mật Go](/security/vuln)

## Tổng quan

Go CNA là một
[Cơ quan Đánh số CVE](https://www.cve.org/ProgramOrganization/CNAs), phát hành
[ID CVE](https://www.cve.org/ResourcesSupport/Glossary?activeTerm=glossaryCVEID) và xuất bản
[Bản ghi CVE](https://www.cve.org/ResourcesSupport/Glossary?activeTerm=glossaryRecord)
cho các lỗ hổng bảo mật công khai trong hệ sinh thái Go. Đây là CNA con của Google CNA.

## Phạm vi

Go CNA bao gồm các lỗ hổng bảo mật trong dự án Go ([thư viện
chuẩn](/pkg) và
[các sub-repository](https://pkg.go.dev/golang.org/x)) và các lỗ hổng bảo mật công khai
trong các module Go có thể import mà chưa được bao gồm bởi một CNA khác.

Phạm vi này nhằm mục đích loại trừ rõ ràng các lỗ hổng bảo mật trong các ứng dụng hoặc
gói được viết bằng Go mà không thể import (ví dụ, bất cứ thứ gì trong
package `main`). Xem [go.dev/security/vuln/database#excluded-reports](/security/vuln/database#excluded-reports) để biết thêm thông tin về các báo cáo bị loại trừ.

Để báo cáo các lỗ hổng bảo mật mới tiềm ẩn trong dự án Go, hãy tham khảo
[go.dev/security/policy](/security/policy).

## Yêu cầu ID CVE cho một lỗ hổng bảo mật công khai

**QUAN TRỌNG**: Biểu mẫu được liên kết bên dưới tạo ra một vấn đề công khai trên trình theo dõi vấn đề, và do đó
*không được* sử dụng để báo cáo các lỗ hổng bảo mật chưa được công bố trong Go (xem
[chính sách bảo mật](/security/policy) của chúng tôi để biết hướng dẫn báo cáo
các vấn đề chưa được công bố).

Để yêu cầu ID CVE cho một lỗ hổng bảo mật PUBLIC hiện có trong hệ sinh thái Go,
[gửi yêu cầu qua biểu mẫu này](/s/vulndb-report-new).

Một lỗ hổng bảo mật được coi là công khai nếu nó đã được công bố công khai, hoặc nó tồn tại trong
một gói bạn duy trì, và bạn đã sẵn sàng công bố nó công khai.
