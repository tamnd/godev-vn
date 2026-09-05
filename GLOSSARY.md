# Translation Glossary

Tệp này xác định các bản dịch tiếng Việt ưu tiên và các quyết định văn phong cho
`_content_vi/`.

Cập nhật bảng thuật ngữ này trong quá trình dịch:

- Thêm thuật ngữ khi một cụm từ lặp lại nhiều lần hoặc mang tính chuyên ngành.
- Ưu tiên một bản dịch thống nhất cho mỗi khái niệm, trừ khi ngữ cảnh đòi hỏi khác đi.
- Giữ nguyên tên sản phẩm, tên kho lưu trữ, tên tệp và cú pháp truy vấn.
- Giữ nguyên mã nguồn, URL và các đoạn lệnh.
- Giữ nguyên tên khóa trong front matter, ví dụ `title:`, `redirect:`, `layout:`.
  Trình kết xuất đọc những tên đó, chúng không phải là văn bản. Chỉ dịch phần
  giá trị khi giá trị đó là câu chữ, và `godev audit` refuse ở L09 nếu danh sách
  khóa khác với bản tiếng Anh.
- Khi chủ ý giữ nguyên một thuật ngữ không dịch, hãy ghi rõ quyết định đó tại đây.
- Một thuật ngữ nằm trong tên riêng thì giữ nguyên, kể cả khi bảng dưới đây có
  bản dịch cho nó. Tên bài nói, tên bài viết, tên nhóm làm việc, tên vai trò và
  tên mã trạng thái HTTP đều là tên riêng: "Go, Open Source, Community", "Go
  Community Outreach Working Group", "Vulnerability Management for Go", "Go
  project stewards", "307 Temporary Redirect".
- `godev audit` vẫn báo những trường hợp đó ở mức notice, vì nó không phân biệt
  được tên riêng với câu văn. Đó là notice chứ không phải refusal, và cách xử lý
  đúng là bỏ qua, không phải dịch tên riêng ra tiếng Việt.

## Quy ước

- Văn phong: rõ ràng, kỹ thuật, trung tính.
- Tên sản phẩm: `Go`
- Tên dự án: `dự án Go`
- Lịch sử quản lý mã nguồn: `lịch sử quản lý mã nguồn`
- Tác giả: `tác giả`
- Commit: `commit`
- Thay đổi: `thay đổi`
- Tìm kiếm: `tìm kiếm`
- Chuyển hướng: `chuyển hướng`
- Máy chủ: `máy chủ`
- Nguồn tham chiếu chính: `nguồn thông tin chính xác nhất`

## Thuật ngữ đã chốt

| Thuật ngữ gốc | Bản dịch ưu tiên | Ghi chú |
| --- | --- | --- |
| Authors of Go | Tác giả của Go | Page title in `_content_vi/AUTHORS.md`. |
| Go project | dự án Go | Keep `Go` untranslated. |
| AUTHORS | AUTHORS | Filename, do not translate. |
| CONTRIBUTORS | CONTRIBUTORS | Filename, do not translate. |
| author | tác giả | Use for people credited for commits or changes. |
| contributor | contributor | Keep unchanged, do not translate. |
| commit | commit | Keep the Git term unchanged. |
| repository | kho lưu trữ | Translate it. This row used to say keep it unchanged and the corpus disagreed all along: 420 occurrences of `kho lưu trữ` across 85 files against 89 of the English across 27, and doc/contribute.html and talks/2012/go1.slide each use both in the same page. The English is the minority and this row was the thing making it look like the rule. |
| sub-repository | sub-repository | Keep unchanged. It is what the Go project calls the golang.org/x repositories, it is a name and not a description, and there are 9 on the site. A page writing `sub-repository của kho lưu trữ Go` is doing the right thing with both halves. |
| source control history | lịch sử quản lý mã nguồn | Prefer this over shorter paraphrases. |
| authoritative source | nguồn thông tin chính xác nhất | Matches current translation. |
| Go's Gerrit instance | máy chủ Gerrit của Go | Keep `Gerrit` as a product name. |
| search for changes | tìm kiếm các thay đổi | General action phrase. |
| redirect | chuyển hướng | Translate it in prose. The note here used to say "for front matter semantics", which read as an instruction to translate the front matter key `redirect:` as well, and that is what kept happening: `about.md` is nothing but `redirect: https://pkg.go.dev/about` between two rule lines, and every attempt at it came back with the key in Vietnamese and was refused by L09. A front matter key is read by the renderer and is not text. |
| ecosystem | hệ sinh thái | Prefer this in Go platform context. |
| community | cộng đồng | General community term. |
| open-source | mã nguồn mở | Use in general prose. |
| anniversary | kỷ niệm | Use for anniversary or milestone context. |
| birthday | sinh nhật | Use for celebratory wording. |
| cloud software | phần mềm đám mây | Prefer this in infra context. |
| garbage collector | bộ gom rác | Preferred runtime term. |
| tooling | hệ thống công cụ | Use for developer-tooling context. |
| release | bản phát hành | Use for product or language releases. Not inside the three rows below. |
| release notes | ghi chú bản phát hành | Translate it. The site menu says it this way, so a page that leaves it in English disagrees with its own navigation. |
| release candidate | release candidate | Keep unchanged. This is what Vietnamese Go writing calls it, and "bản ứng viên phát hành" is nobody's phrase. |
| pre-release | pre-release | Keep unchanged. Semver vocabulary, and it appears next to `v1.2.3-beta.1` where translating it would be confusing. |
| point release | point release | Keep unchanged. Go release engineering calls a 1.x.y fix release this, there is no Vietnamese for it, and blog/go1.1.md and blog/contributors-summit.md both already write it in English. |
| vulnerability | lỗ hổng bảo mật | Security context. |
| dependency | dependency | Keep this term in technical prose when clearer. |
| generics | generics | Keep the feature name unchanged for now. |
| production-ready | sẵn sàng cho môi trường production | Preferred phrase in release notes. |
| editor-neutral | trung lập với trình soạn thảo | For tooling compatibility context. |
| workspace | workspace | Keep unchanged in Go feature naming. |
| soft memory limit | giới hạn bộ nhớ mềm | Runtime and GC context. |
| supply chain security | bảo mật chuỗi cung ứng | Security context. |
| false positive | dương tính giả | Security and tooling context. |
| structured logging | structured logging | Keep unchanged for now. |
| profile-guided optimization | tối ưu hóa dựa trên hồ sơ thực thi | Compiler and performance context. |
| interface | interface | Keep unchanged when it is the Go type. The corpus writes `kiểu interface` 97 times against 4 for `kiểu giao diện`, and `một interface` 97 times against 15, so this is the settled convention and not a preference. `giao diện` is right for the other sense and stays: `giao diện người dùng` in doc/diagnostics.html and the browser described as an interface in blog/contributors-summit-2019.md are both correct. |
| map | map | Keep unchanged when it is the Go type. `bản đồ` is a geographic map and is wrong for it. The four `bản đồ` on disk are a stack map, a process memory map and a mark and allocate diagram, all of which are real maps. blog/swisstable.md is the file to look at: its body already says `kiểu map`, and a title reading `Bản đồ Go` disagrees with the page under it. |
| type parameter | tham số kiểu | Translate it. 359 against 74 across the corpus, including ref/spec.html, so the English is the minority and not the rule. |
| Contributors Summit | Contributors Summit | Keep unchanged. It is the name of an event, blog/contributors-summit.md already writes it and `Go Contributor Summit` in English, and `Hội nghị thượng đỉnh` is what a diplomatic summit is called. |

## Quyết định đang áp dụng

- Dùng tiếng Việt có dấu nhất quán trong toàn bộ `_content_vi/`.
- Ưu tiên cách diễn đạt tự nhiên trong tiếng Việt thay vì bám sát từng từ tiếng Anh.
