---
title: Giới thiệu API của pkg.go.dev
date: 2026-05-21
by:
- Ethan Lee
- Hana Kim
- Jonathan Amsterdam
summary: Giới thiệu API lập trình mới cho pkg.go.dev, cho phép các nhà phát triển lấy dữ liệu về gói và mô-đun trực tiếp.
---

Since its inception, [pkg.go.dev](https://pkg.go.dev) has established itself as
the Go community's primary resource for package documentation and discovery.
While we initially prioritized creating a comprehensive and highly accessible
web interface for users, the need for programmatic access has become
increasingly clear. Developers building tools, IDE integrations, and automated
workflows have historically relied on fragile workarounds like web scraping to
access this data. To better address these evolving requirements, we are now
expanding our platform to provide robust, direct access to the information our
community needs.

Today, we are excited to introduce the official
[pkg.go.dev API](https://pkg.go.dev/api) — a service interface for querying
metadata about published Go modules. This launch is a direct response to
years of community feedback. Furthermore, the need for a formalized
interface has become even more acute with the rise of AI-assisted coding.
Tools can now access the specific, high-fidelity context needed to reason
about the Go ecosystem with greater precision.

## The service interface

Built for stability and efficient caching, the API uses a stateless, GET-only
architecture. Primary endpoints are currently hosted under the `/v1beta` path.
Following a period of community feedback and confirmed stability, we intend to
transition toward a formal `v1` release.

For a complete interactive reference of all endpoints, query parameters, and
response shapes, see the [pkg.go.dev/api specification](https://pkg.go.dev/api).
The machine-readable API contract is also published directly as an [OpenAPI
specification](https://pkg.go.dev/v1beta/openapi.yaml).

### Core endpoints

| Endpoint | Description |
| :--- | :--- |
| `/v1beta/package/{path}` | Information about the package at `{path}`. |
| `/v1beta/module/{path}` | Information about the module at `{path}`. |
| `/v1beta/versions/{path}` | Versions of the module at `{path}`. |
| `/v1beta/packages/{path}` | Information about packages of the module at `{path}`. |
| `/v1beta/search?q={query}` | Search results for a given query. |
| `/v1beta/symbols/{path}` | List of symbols declared by the package at `{path}`. |
| `/v1beta/imported-by/{path}` | Paths of packages importing the package at `{path}`. |
| `/v1beta/vulns/{path}` | Vulnerabilities of the module or package at `{path}`. |

One design principle for this API is "precision over convenience." For context,
when `go mod tidy` encounters an import of a package that isn't provided by an
existing dependency of the main module, it applies the "longest module path"
rule to determine which module is needed. (The fact that two or more modules
could provide the package is what makes it possible to later carve out a
submodule without breaking existing programs.) The
[pkg.go.dev](https://pkg.go.dev) web interface follows a similar convention
when choosing which package to display for a given package path.
By contrast, the [pkg.go.dev](https://pkg.go.dev) API requires the module to be
specified unambiguously. If a package path is ambiguous because it exists in
multiple modules, the API returns a list of candidates and reports an error
asking the client to be more specific.

For example, a package imported as `example.com/a/b/c` could be provided by
module `example.com/a` or by `example.com/a/b`. While the
[pkg.go.dev](https://pkg.go.dev) web interface will automatically resolve the
"longest module path" (`example.com/a/b`), a client querying the API must
specify the module explicitly to avoid an ambiguous resolution error.

### Chỉ định phiên bản

Đối với các endpoint truy xuất thông tin về gói, module hoặc symbol, bạn có thể chỉ định phiên bản mong muốn bằng tham số truy vấn tùy chọn `version`. API trả về thông tin về phiên bản mới nhất của module hoặc gói theo mặc định. Tham số này hỗ trợ:

* **Phiên bản ngữ nghĩa:** Truy xuất dữ liệu cho một thẻ bản phát hành cụ thể (ví dụ: `?version=v1.2.3` hoặc `?version=v0.6.0`).
* **Tên nhánh:** Tham chiếu các nhánh phát triển mặc định—cụ thể là `master` hoặc `main` (ví dụ: `?version=master`). API sẽ tự động phân giải nhánh thành pseudo-version tương ứng. Lưu ý rằng các tên nhánh tùy chỉnh hoặc tùy ý không được hỗ trợ.

Nếu tham số `version` bị bỏ qua, API mặc định phân giải yêu cầu dựa trên phiên bản được gắn thẻ mới nhất của gói hoặc module.

### Ví dụ: yêu cầu API thô

Để truy xuất trực tiếp metadata có cấu trúc cho một gói cụ thể (sử dụng `jq` để định dạng):

```console
$ curl https://pkg.go.dev/v1beta/package/github.com/google/go-cmp/cmp | jq .
{
  "modulePath": "github.com/google/go-cmp",
  "version": "v0.7.0",
  "isLatest": true,
  "isStandardLibrary": false,
  "goos": "all",
  "goarch": "all",
  "path": "github.com/google/go-cmp/cmp",
  "name": "cmp",
  "synopsis": "Package cmp determines equality of values.",
  "isRedistributable": true
}
```

Để truy vấn phiên bản nhánh cụ thể (như `master`) và xem nó được tự động phân giải thành pseudo-version tương ứng:

```console
$ curl -s "https://pkg.go.dev/v1beta/package/github.com/google/go-cmp/cmp?version=master" | jq '{path, version}'
{
  "path": "github.com/google/go-cmp/cmp",
  "version": "v0.7.1-0.20260310220054-34c9473539b8"
}
```

## Bản triển khai tham chiếu pkgsite-cli

Để minh họa cách tương tác với API của chúng tôi, chúng tôi cung cấp một triển khai client tham chiếu:
[pkgsite-cli](https://github.com/golang/pkgsite/tree/master/cmd/internal/pkgsite-cli).
Triển khai này đóng vai trò là ví dụ thực tế cho các nhà phát triển muốn xây dựng tích hợp của riêng họ, cho thấy cách xử lý dữ liệu trực tiếp từ terminal. Xin lưu ý rằng khi API tiếp tục phát triển, giao diện và hành vi của lệnh này có thể thay đổi.

Để bắt đầu, hãy cài đặt lệnh:

```bash
$ go install golang.org/x/pkgsite/cmd/internal/pkgsite-cli@latest
```

Để tìm kiếm các gói:

```
$ pkgsite-cli search "uuid"
github.com/google/uuid
  Module:   github.com/google/uuid@v1.6.0
  Synopsis: Package uuid generates and inspects UUIDs.
... more
```

Để kiểm tra một gói cụ thể:

```
$ pkgsite-cli package github.com/google/go-cmp/cmp
github.com/google/go-cmp/cmp
  Name:      cmp
  Module:    github.com/google/go-cmp
  Version:   v0.7.0 (latest)
  Synopsis:  Package cmp determines equality of values.
```

Để xem những gói nào import một gói cụ thể:

```
$ pkgsite-cli package --imported-by github.com/google/go-cmp/cmp
github.com/google/go-cmp/cmp
  Name:     cmp
  Module:   github.com/google/go-cmp
  Version:  v0.7.0 (latest)
  Synopsis: Package cmp determines equality of values.

Imported by:
  cloud.google.com/go/internal/testutil
  cuelang.org/go/internal/cuetxtar
  chainguard.dev/apko/pkg/build/types
  ... more
```

Để liệt kê các symbol được khai báo bởi một gói:

```
$ pkgsite-cli package --symbols github.com/google/go-cmp/cmp
github.com/google/go-cmp/cmp
  Name:     cmp
  Module:   github.com/google/go-cmp
  Version:  v0.7.0 (latest)
  Synopsis: Package cmp determines equality of values.

Symbols:
  type Indirect struct{}
  type MapIndex struct{}
  type Option interface{}
  ... more
```

Để liệt kê các phiên bản của một module:

```
$ pkgsite-cli module -versions github.com/google/go-cmp
github.com/google/go-cmp
  Version:          v0.7.0 (latest)
  Repository:       https://github.com/google/go-cmp
  Has go.mod:       yes
  Redistributable:  yes

Versions:
  v0.7.0
  v0.6.0
  v0.5.9
  ... more
```

Để liệt kê cả phiên bản và gói của một module:

```
$ pkgsite-cli module -packages -versions github.com/google/go-cmp
github.com/google/go-cmp
  Version:          v0.7.0 (latest)
  Repository:       https://github.com/google/go-cmp
  Has go.mod:       yes
  Redistributable:  yes

Versions:
  v0.7.0
  v0.6.0
  v0.5.9
  ... more

Packages:
  github.com/google/go-cmp/cmp             Package cmp determines equality of values.
  github.com/google/go-cmp/cmp/cmpopts     Package cmpopts provides common options for the cmp package.
  ... more
```

Lệnh này xử lý phân trang và định dạng, cho phép bạn tập trung vào dữ liệu cần thiết cho các script hoặc việc điều tra thủ công. Để tìm hiểu thêm, vui lòng truy cập [tài liệu của pkgsite-cli](https://pkg.go.dev/golang.org/x/pkgsite/cmd/internal/pkgsite-cli).

## Tính ổn định và tương lai

Phần này kết thúc chuyến tham quan ngắn về API [pkg.go.dev](https://pkg.go.dev). Mặc dù chúng tôi dự định mở rộng các khả năng của giao diện này theo thời gian, chúng tôi cam kết duy trì khả năng tương thích ngược để các tích hợp hiện có tiếp tục hoạt động liền mạch. (Lưu ý rằng giao diện dòng lệnh của ứng dụng khách tham chiếu `pkgsite-cli` hiện chưa ổn định.) Chúng tôi hoan nghênh phản hồi của bạn thông qua [trình theo dõi vấn đề](https://github.com/golang/go/issues), và mong được thấy những công cụ và quy trình làm việc mới mà cộng đồng sẽ xây dựng.
