---
title: Hướng dẫn di chuyển encoding/json/v2
layout: article
---

Được giới thiệu trong Go 1.27, gói [encoding/json/v2](/pkg/encoding/json/v2) là một bản sửa đổi lớn của gói [encoding/json](/pkg/encoding/json) ban đầu.
Hướng dẫn này mô tả lý do bạn có thể muốn chuyển từ gói v1 sang gói v2, cũng như cách thực hiện việc đó một cách an toàn.

# Tại sao di chuyển?

Trước hết: bạn không bắt buộc phải làm vậy!
Gói `encoding/json` sẽ không bao giờ biến mất.
Gói này được bảo đảm bởi cam kết tương thích Go 1, vì vậy các gói sử dụng `encoding/json` sẽ tiếp tục hoạt động vô thời hạn.

Ngoài ra, `encoding/json` và `encoding/json/v2` tương thích với nhau.
Ví dụ, nếu một kiểu triển khai hành vi marshaling tùy chỉnh bằng [`encoding/json/v2.MarshalerTo`](/pkg/encoding/json/v2#MarshalerTo), các bên gọi marshaling kiểu này thông qua [`encoding/json.Marshal`](/pkg/encoding/json#Marshal) vẫn sẽ đi qua marshaler tùy chỉnh đó.
Tương tự, các thẻ struct `json` mới được giới thiệu trong Go 1.27 cùng với `encoding/json/v2` cũng được `encoding/json` hỗ trợ.

Mặc dù bạn không bắt buộc phải di chuyển, có một số lý do tốt để thực hiện việc đó:

Trước tiên, API mới dễ sử dụng hơn.
Ví dụ, dễ dàng marshal tới một [`io.Writer`](/pkg/io#Writer) bằng [`encoding/json/v2.MarshalWrite`](/pkg/encoding/json/v2#MarshalWrite), thay vì cần một [`encoding/json.Encoder`](/pkg/encoding/json#Encoder).
[`encoding/json/v2.Marshalers`](/pkg/encoding/json/v2#Marshalers) cho phép ghi đè hành vi marshal của các kiểu cụ thể, ngay cả khi bạn không kiểm soát các kiểu đó.
[`encoding/json/v2.MatchCaseInsenstiveNames`](/pkg/encoding/json/v2#MatchCaseInsenstiveNames) cho phép kiểm soát độ nhạy chữ hoa chữ thường khi so khớp tên thành viên của đối tượng JSON với các trường struct Go.

Mặc dù những cải tiến này rất hữu ích, lý do tốt nhất để di chuyển là gói v2 chọn các giá trị mặc định nghiêm ngặt hơn và có khả năng tương tác tốt hơn so với v1. [tài liệu `encoding/json`](/pkg/encoding/json#hdr-Migrating_to_v2) chứa toàn bộ tập hợp khác biệt, nhưng một số điểm nổi bật bao gồm:

* Trong v1, các byte UTF-8 không hợp lệ bên trong một chuỗi được âm thầm thay thế bằng ký tự thay thế Unicode. Ngược lại, trong v2, sự xuất hiện của UTF-8 không hợp lệ sẽ dẫn đến lỗi.
* Trong v1, một đối tượng JSON có các tên trùng lặp được cho phép. Ngược lại, trong v2, một đối tượng JSON có các tên trùng lặp sẽ dẫn đến lỗi.
* Trong v1, một slice Go nil hoặc map Go nil được marshal thành JSON null. Ngược lại, v2 marshal slice Go nil hoặc map Go nil thành một mảng JSON hoặc đối tượng JSON rỗng tương ứng.
* Trong v1, lỗi không bao giờ được báo cáo trong thời gian chạy đối với các kiểu struct Go có một dạng lỗi cấu trúc nào đó (ví dụ: thẻ trường không hợp lệ). Ngược lại, v2 báo cáo lỗi thời gian chạy đối với các kiểu Go không hợp lệ liên quan đến tuần tự hóa JSON.

Những thay đổi này được thiết kế để làm cho `encoding/json/v2` có khả năng tương tác tốt hơn với hệ sinh thái JSON rộng hơn, ít gây bất ngờ hơn và ít xảy ra lỗi hơn.
Nhưng chúng không tương thích ngược; một số ứng dụng có thể phụ thuộc vào hành vi của v1.
Do đó, việc di chuyển sang v2 phải bao gồm kiểm thử cẩn thận để đảm bảo tính tương thích.

# API so với thay đổi về hành vi

Đối với việc marshal và unmarshal đơn giản, các API Go phần lớn tương thích ở cấp độ ngôn ngữ, do đó việc di chuyển API là rất đơn giản.
Ví dụ, `b, err := json.Marshal(v)` sẽ tiếp tục biên dịch chỉ bằng cách chuyển `import "encoding/json"` sang `import "encoding/json/v2"`.

Phần khó khăn khi di chuyển từ v1 sang v2 không phải là sự khác biệt về API, mà là các thay đổi về hành vi trong việc marshal và unmarshal.
Hãy xem xét chương trình này:

```
package main

import (
	"encoding/json"
	"fmt"
)

type Pet struct {
	Name      string
	Nicknames []string
}

func main() {
	pets := []Pet{
		{Name: "Oliver", Nicknames: []string{"Ollie", "Olliepop"}},
		{Name: "Remi"},
	}
	b, err := json.Marshal(pets)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(b))
}
```

Khi [chạy](/play/p/Us887UVmEwm), chương trình này xuất ra:

```
[{"Name":"Oliver","Nicknames":["Ollie","Olliepop"]},{"Name":"Remi","Nicknames":null}]
```

Nếu chúng ta di chuyển chương trình này một cách đơn giản sang `encoding/json/v2` bằng cách thay đổi import, chương trình vẫn biên dịch được.
Khi [chạy](/play/p/E-SprmrVHZF), chương trình xuất ra:

```
[{"Name":"Oliver","Nicknames":["Ollie","Olliepop"]},{"Name":"Remi","Nicknames":[]}]
```

Lưu ý rằng trường "Nicknames" của Remi đã thay đổi từ `null` thành `[]`.
Nếu đây là một chương trình mới, việc sử dụng mảng rỗng có khả năng là một cải tiến tốt, nhưng trong một ứng dụng hiện có, các trình tiêu thụ ở phía sau của đầu ra này có thể đang phụ thuộc vào sự hiện diện của `null`, vì vậy thay đổi này có thể làm hỏng chúng.

# Các tùy chọn

Tất cả những nơi mà hành vi của v2 khác với v1 đều được bao phủ bởi [`Options`](/pkg/encoding/json/v2#Options) cho phép chỉ định hành vi của v1 bằng API v2.
Để sử dụng API v2 nhưng chỉ định tất cả hành vi của v1, hãy sử dụng [`DefaultOptionsV1`](/pkg/encoding/json#DefaultOptionsV1):

```
package main

import (
	jsonv1 "encoding/json"
	"encoding/json/v2"
	"fmt"
)

type Pet struct {
	Name      string
	Nicknames []string
}

func main() {
	pets := []Pet{
		{Name: "Oliver", Nicknames: []string{"Ollie", "Olliepop"}},
		{Name: "Remi"},
	}
	b, err := json.Marshal(pets, jsonv1.DefaultOptionsV1())
	if err != nil {
		panic(err)
	}
	fmt.Println(string(b))
}
```

Khi [chạy](/play/p/8wg09vDeNN6), chương trình này một lần nữa xuất ra `null`:

```
[{"Name":"Oliver","Nicknames":["Ollie","Olliepop"]},{"Name":"Remi","Nicknames":null}]
```

Tài liệu [DefaultOptionsV1](/pkg/encoding/json#DefaultOptionsV1) liệt kê toàn bộ tập tùy chọn được sử dụng cho khả năng tương thích với v1.
Các tùy chọn sau ghi đè các tùy chọn trước đó, vì vậy bạn có thể sử dụng danh sách này để bật từng hành vi của v2 một cách lần lượt.

# Migration

Depending on the type of application and its risk tolerance, there are several different ways to approach a v2 migration:

* High risk tolerance: [all-at-once](#all-at-once)
* Low risk tolerance or troubleshooting: [option-by-option](#option-by-option)
* Production server applications: [jsonsplit](#jsonsplit)

## All-at-once {#all-at-once}

If the application is simple or has high risk tolerance (there is little consequence to the migration causing problems), then there may be no need for an elaborate migration process.
Simply update callsites to use `encoding/json/v2`, make sure the tests pass, and check it in.

If you do run into compatibility issues, the source of the difference may be clear from the change or error message, in which case you can set the [appropriate compatibility option](/pkg/encoding/json#DefaultOptionsV1).
If the source of the problem is not clear, you may want to use one of the approaches below to help troubleshoot.

This can also be a quick way to find and fix obvious issues (such as those identified by unit tests) before moving on to a more nuanced approach to track down the remainder.

## Option-by-option {#option-by-option}

If the application is complex or has a low risk tolerance, then you may need to take a slower, more careful approach.

As mentioned above, calling `Marshal` or `Unmarshal` with [DefaultOptionsV1](/pkg/encoding/json#DefaultOptionsV1) makes the call behave identically to `encoding/json`.
As a first step, migrate all calls to `encoding/json/v2` with `DefaultOptionsV1`.
This is a trivial and safe change; in fact, this is exactly how [`encoding/json` implements `Marshal` and `Unmarshal`](https://cs.opensource.google/go/go/+/refs/tags/go1.27rc2:src/encoding/json/v2_encode.go;l=184-186)!

Additional passed options override earlier options, so you can disable individual [v1 compatibility options](/pkg/encoding/json#DefaultOptionsV1).
For example, `json.Marshal(v, jsonv1.DefaultOptionsV1(), json.FormatNilSliceAsNull(false))` will behave like v1 except that `nil` slices format as empty arrays.

This provides a path to perform a slower migration rather than changing all behavior at once.
You could enable one option at a time to be very sure what is changing, or group similar options.
Alternatively, when troubleshooting these options provide a way to bisect down to the exact breaking behavior change, similar to the option detection we'll see in `jsonsplit` below.

## jsonsplit {#jsonsplit}

[`github.com/go-json-experiment/jsonsplit`](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit) is a JSON wrapper package that aids migration by reporting differences between v1 and v2 in a production setting.

`jsonsplit` provides `Marshal` and `Unmarshal` wrapper functions which behave the same as `encoding/json` by default, but can be configured at runtime to use v1, v2, or both.

When configured to use both ([`CallBothButReturnV1`](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit#CallMode)), `jsonsplit.Marshal` will marshal the input twice: once with v1 and v2.
It will report any differences, but still return the v1 value to the caller.
This allows a production service to report differences without changing its behavior.
With the optional [`AutoDetectOptions`](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit#Codec), `jsonsplit` will even automatically determine which specific options cause the difference.

Note that this functionality comes at a cost.
Marshaling with both v1 and v2 to detect differences will approximately double the cost of marshaling, and `AutoDetectOptions` performs even more marshals to narrow down the relevant options.
To mitigate these costs, `jsonsplit` allows checking only a random subset of calls via [`SetMarshalCallRatio`](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit#Codec.SetMarshalCallRatio).

Here, we have applied `jsonsplit` to the original example:

```
package main

import (
	"fmt"

	"github.com/go-json-experiment/jsonsplit"
)

func init() {
	// Call both v1 and v2 so we can detect differences, but continue using
	// v1 output.
	jsonsplit.GlobalCodec.SetMarshalCallMode(jsonsplit.CallBothButReturnV1)

	// Specify that when a difference is detected, to auto-detect which
	// options are causing the difference.
	jsonsplit.GlobalCodec.AutoDetectOptions = true

	// Log every time we detect a difference between v1 and v2.
	jsonsplit.GlobalCodec.ReportDifference = func(d jsonsplit.Difference) {
		fmt.Printf("detected jsonv1-to-jsonv2 difference: %v\n", d)
	}
}

type Pet struct {
	Name      string
	Nicknames []string
}

func main() {
	pets := []Pet{
		{Name: "Oliver", Nicknames: []string{"Ollie", "Olliepop"}},
		{Name: "Remi"},
	}
	b, err := jsonsplit.Marshal(pets)
	if err != nil {
		panic(err)
	}
	fmt.Println(string(b))
}
```

When [run](/play/p/wagp5v8V1w-), this reports the difference and even that `FormatNilSliceAsNull` is responsible for the difference:

```
detected jsonv1-to-jsonv2 difference: {"Caller":"main.main+5","Func":"Marshal","GoType":"[]main.Pet","JSONValueV1":[{"Name":"Oliver","Nicknames":["Ollie","Olliepop"]},{"Name":"Remi","Nicknames":null}],"JSONValueV2":[{"Name":"Oliver","Nicknames":["Ollie","Olliepop"]},{"Name":"Remi","Nicknames":[]}],"Options":["jsonv2.FormatNilSliceAsNull"]}
[{"Name":"Oliver","Nicknames":["Ollie","Olliepop"]},{"Name":"Remi","Nicknames":null}]
```

We can migrate our production service smoothly using a procedure like the following:

1. Switch callsites to `jsonsplit`, set [`CallBothButReturnV1`](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit#CallMode), [`AutoDetectOptions`](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit#CallMode) (optional), and wire up your preferred monitoring approach to [`ReportDifference`](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit#CallMode) (such as logging or published metrics).

2. Monitor your production environment for reported differences.

3. Encode differences.

Where `jsonsplit` reports differences, adjust the options or types to ensure identical output.

For example, in the example above, pass the `json.FormatNilSliceAsNull(true)` option.
In other cases, v2 may report a problem that is straightforward to fix.
For example, applying the "string" JSON struct field tag to an invalid type (such as a struct) is ignored in v1, but reports an error in v2.
While [`ReportErrorsWithLegacySemantics`](/pkg/encoding/json#ReportErrorsWithLegacySemantics) would suppress the error, it makes more sense to drop the "string" tag.
It isn't doing anything anyway.

Note that a difference in output does not necessarily mean that downstream behavior is broken, but that there is an opportunity for breakage.
We adjust options now so we can complete the vast majority of the migration without stopping to evaluate subtle output changes, but after switching to v2, you should revisit these locations to determine if you can migrate to the new behavior.

4. Switch to v2.

Once your production environment stops reporting new differences, you can migrate to v2 behavior by setting [`OnlyCallV2`](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit#CallMode) or [`CallBothButReturnV2`](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit#CallMode) to keep checking for differences.

5. Clean up.

Once the switch to v2 is safely deployed, clean up the migration, by switching from `jsonsplit` to `encoding/json/v2` itself.
At this point, you can evaluate the cases from (3) that kept some v1 behavior to determine if it is safe to switch them to the v2 behavior.

See the [`jsonsplit` documentation](https://pkg.go.dev/github.com/go-json-experiment/jsonsplit#hdr-Example_usage_and_migration) for more details about `jsonsplit` migrations.
