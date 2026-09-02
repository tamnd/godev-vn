---
title: Extension VS Code Go gia nhập dự án Go
date: 2020-06-09
by:
- The Go team
summary: Thông báo về việc VS Code Go chuyển sang dự án Go.
---


Khi dự án Go ra đời, "một mục tiêu bao trùm là Go làm được nhiều hơn để hỗ trợ lập trình viên bằng cách kích hoạt công cụ, tự động hóa các tác vụ thường ngày như định dạng code, và xóa bỏ các rào cản khi làm việc với codebase lớn" ([FAQ Go](/doc/faq#What_is_the_purpose_of_the_project)). Ngày nay, hơn một thập kỷ sau, chúng tôi vẫn được dẫn dắt bởi cùng mục tiêu đó, đặc biệt khi nó liên quan đến công cụ quan trọng nhất của lập trình viên: trình soạn thảo của họ.

Trong suốt thập kỷ qua, các nhà phát triển Go đã phụ thuộc vào nhiều trình soạn thảo và hàng chục công cụ, plugin được tạo ra độc lập. Phần lớn thành công ban đầu của Go có thể được quy cho các công cụ phát triển tuyệt vời được tạo ra bởi cộng đồng Go. [Extension VS Code cho Go](https://github.com/microsoft/vscode-go), được xây dựng từ nhiều công cụ đó, hiện được sử dụng bởi 41 phần trăm các nhà phát triển Go ([khảo sát nhà phát triển Go](/blog/survey2019-results)).

Khi extension VS Code Go ngày càng phổ biến hơn và [hệ sinh thái mở rộng](https://www.youtube.com/watch?v=EFJfdWzBHwE), nó đòi hỏi [bảo trì và hỗ trợ nhiều hơn](https://twitter.com/ramyanexus/status/1154470078978486272). Trong vài năm qua, nhóm Go đã hợp tác với nhóm VS Code để hỗ trợ những người duy trì extension Go. Nhóm Go cũng bắt đầu một sáng kiến mới để cải thiện các công cụ hỗ trợ tất cả các extension trình soạn thảo Go, với trọng tâm là hỗ trợ [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) với [`gopls`](/s/gopls) và [Debug Adapter Protocol với Delve](https://github.com/go-delve/delve/issues/1515).

Qua sự hợp tác giữa nhóm VS Code và nhóm Go, chúng tôi nhận ra rằng nhóm Go ở vị trí độc đáo để phát triển trải nghiệm phát triển Go song song với ngôn ngữ Go.

Do đó, chúng tôi vui mừng thông báo giai đoạn tiếp theo trong quan hệ đối tác của nhóm Go với nhóm VS Code: **Extension VS Code cho Go chính thức gia nhập dự án Go**. Điều này đi kèm hai thay đổi quan trọng:

1. Nhà xuất bản của plugin đang chuyển từ "Microsoft" sang "Go Team at Google".
2. Kho lưu trữ của dự án đang chuyển để gia nhập phần còn lại của dự án Go tại [https://github.com/golang/vscode-go](https://github.com/golang/vscode-go).

Chúng tôi không thể diễn đạt đủ lòng biết ơn đối với những người đã giúp xây dựng và duy trì extension đáng yêu này. Chúng tôi biết rằng những ý tưởng và tính năng sáng tạo đến từ bạn, người dùng của chúng tôi. Mục tiêu chính của nhóm Go với tư cách là chủ sở hữu extension là giảm gánh nặng công việc bảo trì cho cộng đồng Go. Chúng tôi sẽ đảm bảo các build luôn xanh, các vấn đề được phân loại, và tài liệu được cập nhật. Các thành viên nhóm Go sẽ cập nhật cho người đóng góp về các thay đổi ngôn ngữ liên quan, và chúng tôi sẽ làm mịn các cạnh sắc giữa các dependency khác nhau của extension.

Tiếp tục chia sẻ suy nghĩ của bạn với chúng tôi bằng cách tạo [vấn đề](https://github.com/golang/vscode-go/issues) và thực hiện [đóng góp](https://github.com/golang/vscode-go/blob/master/docs/contributing.md) vào dự án. Quy trình đóng góp bây giờ sẽ giống như [phần còn lại của dự án Go](/doc/contribute.html). Các thành viên nhóm Go sẽ cung cấp hỗ trợ chung trên kênh #vscode tại [Gophers Slack](https://invite.slack.golangbridge.org/), và chúng tôi cũng đã tạo kênh #vscode-dev để thảo luận về vấn đề và phát triển ý tưởng với người đóng góp.

Chúng tôi hào hứng về bước tiến mới này, và hy vọng bạn cũng vậy. Bằng cách duy trì một extension trình soạn thảo Go lớn, cũng như hệ thống công cụ và ngôn ngữ Go, nhóm Go sẽ có thể cung cấp cho tất cả người dùng Go, bất kể trình soạn thảo của họ là gì, một trải nghiệm phát triển gắn kết và tinh tế hơn.

Như mọi khi, mục tiêu của chúng tôi vẫn như cũ: Mỗi người dùng nên có trải nghiệm xuất sắc khi viết code Go.

*Xem bài đăng kèm theo từ [nhóm Visual Studio Code](https://aka.ms/go-blog-vscode-202006).*
