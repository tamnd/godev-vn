---
title: Comments
---

<!--
This is just a placeholder page for enabling a test.
In the deployed site it is overwritten with the content of go.googlesource.com/wiki.
-->

Mỗi package đều nên có một comment mô tả package. Comment này phải đặt ngay trước câu lệnh `package` trong một trong các tệp của package (chỉ cần xuất hiện ở một tệp duy nhất). Comment nên bắt đầu bằng một câu duy nhất có dạng "Package _packagename_" và tóm tắt ngắn gọn chức năng của package. Câu giới thiệu này sẽ được hiển thị trong danh sách tất cả các package của godoc.

Các câu và đoạn văn tiếp theo có thể bổ sung thêm chi tiết. Các câu cần được viết đúng ngữ pháp và dấu câu.

```go
// Package superman implements methods for saving the world.
//
// Experience has shown that a small number of procedures can prove
// helpful when attempting to save the world.
package superman
```

Hầu hết mọi kiểu, hằng, biến và hàm ở cấp độ package đều nên có comment. Comment cho `bar` nên có dạng "_bar_ floats on high o'er vales and hills.". Chữ cái đầu của _bar_ không được viết hoa trừ khi trong mã nguồn nó được viết hoa.

```go
// enterOrbit causes Superman to fly into low Earth orbit, a position
// that presents several possibilities for planet salvation.
func enterOrbit() os.Error {
  ...
}
```

Bất kỳ đoạn văn bản nào được thụt đầu dòng bên trong comment, godoc sẽ hiển thị dưới dạng khối được định dạng sẵn. Điều này hữu ích cho việc đưa ví dụ mã nguồn vào comment.

```go
// fight can be used on any enemy and returns whether Superman won.
//
// Examples:
//
//  fight("a random potato")
//  fight(LexLuthor{})
//
func fight(enemy interface{}) bool {
	// This is testing proper escaping in the wiki.
	for i := 0; i < 10; i++ {
		println("fight!")
	}
}
```


