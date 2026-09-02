---
title: Xây dựng ứng dụng LLM trong Go
date: 2024-09-12
by:
- Eli Bendersky
tags:
- llm
- ai
- network
summary: Ứng dụng LLM trong Go sử dụng Gemini, langchaingo và Genkit
---

Khi khả năng của các LLM (Large Language Model) và các công cụ liền kề như
mô hình embedding phát triển đáng kể trong năm qua, ngày càng nhiều nhà phát triển
đang xem xét tích hợp LLM vào ứng dụng của họ.

Vì LLM thường đòi hỏi phần cứng chuyên dụng và tài nguyên tính toán đáng kể,
chúng thường được đóng gói dưới dạng dịch vụ mạng cung cấp API để
truy cập. Đây là cách các API cho các LLM hàng đầu như OpenAI hay Google Gemini hoạt động;
thậm chí các công cụ run-your-own-LLM như [Ollama](https://ollama.com/) bọc
LLM trong REST API để sử dụng cục bộ. Hơn nữa, các nhà phát triển tận dụng
LLM trong ứng dụng của họ thường cần các công cụ bổ sung như
Vector Database, cũng thường được triển khai dưới dạng dịch vụ mạng.

Nói cách khác, các ứng dụng LLM rất giống các ứng dụng
cloud-native hiện đại khác: chúng yêu cầu hỗ trợ xuất sắc cho giao thức REST và RPC,
concurrency và hiệu suất. Đây chính xác là những lĩnh vực
mà Go xuất sắc, làm cho nó trở thành ngôn ngữ tuyệt vời để viết ứng dụng LLM.

Bài blog này trình bày một ví dụ về việc sử dụng Go cho ứng dụng LLM đơn giản.
Nó bắt đầu bằng cách mô tả vấn đề mà ứng dụng demo đang giải quyết, và tiến hành trình bày
nhiều biến thể của ứng dụng đều hoàn thành cùng một nhiệm vụ, nhưng sử dụng các package khác nhau để triển khai. Tất cả
mã cho các demo của bài viết này
[có sẵn trực tuyến](https://github.com/golang/example/tree/master/ragserver).

## Máy chủ RAG cho Q&A

Một kỹ thuật ứng dụng LLM phổ biến là RAG,
[Retrieval Augmented Generation](https://en.wikipedia.org/wiki/Retrieval-augmented_generation).
RAG là một trong những cách có thể mở rộng nhất để tùy chỉnh cơ sở kiến thức của LLM
cho các tương tác chuyên biệt theo lĩnh vực.

Chúng ta sẽ xây dựng một *máy chủ RAG* bằng Go. Đây là HTTP server cung cấp
hai hoạt động cho người dùng:

* Thêm tài liệu vào cơ sở kiến thức
* Hỏi LLM một câu hỏi về cơ sở kiến thức này

Trong kịch bản thực tế điển hình, người dùng sẽ thêm một kho tài liệu vào
máy chủ, và tiến hành đặt câu hỏi. Ví dụ, một công ty có thể điền đầy
cơ sở kiến thức của máy chủ RAG với tài liệu nội bộ và sử dụng nó để
cung cấp khả năng Q&A được hỗ trợ bởi LLM cho người dùng nội bộ.

Đây là sơ đồ hiển thị các tương tác của máy chủ với thế giới bên ngoài:

<div class="image"><div class="centered">
<figure>
<img src="llmpowered/rag-server-diagram.png" alt="RAG server diagram"/>
</figure>
</div></div>

Ngoài việc người dùng gửi yêu cầu HTTP (hai hoạt động đã mô tả
ở trên), máy chủ tương tác với:

* Mô hình embedding để tính [vector embedding](https://en.wikipedia.org/wiki/Sentence_embedding)
  cho các tài liệu được gửi và cho câu hỏi của người dùng.
* Vector Database để lưu trữ và truy xuất embedding một cách hiệu quả.
* Một LLM để đặt câu hỏi dựa trên ngữ cảnh thu thập từ cơ sở kiến thức.

Cụ thể, máy chủ cung cấp hai HTTP endpoint cho người dùng:

`/add/: POST {"documents": [{"text": "..."}, {"text": "..."}, ...]}`: gửi
một chuỗi tài liệu văn bản đến máy chủ, để thêm vào cơ sở kiến thức của nó.
Với yêu cầu này, máy chủ:

1. Tính vector embedding cho mỗi tài liệu bằng mô hình embedding.
2. Lưu tài liệu cùng với vector embedding của chúng trong vector DB.

`/query/: POST {"content": "..."}`: gửi câu hỏi đến máy chủ. Với
yêu cầu này, máy chủ:

1. Tính vector embedding của câu hỏi bằng mô hình embedding.
2. Sử dụng tìm kiếm tương đồng của vector DB để tìm các tài liệu liên quan nhất
   đến câu hỏi trong cơ sở kiến thức.
3. Sử dụng kỹ thuật prompt engineering đơn giản để diễn đạt lại câu hỏi với các tài liệu liên quan nhất
   tìm được ở bước (2) làm ngữ cảnh, và gửi đến LLM,
   trả về câu trả lời của nó cho người dùng.

Các dịch vụ được demo sử dụng là:

* [Google Gemini API](https://ai.google.dev/) cho LLM và mô hình embedding.
* [Weaviate](https://weaviate.io/) cho vector DB được host cục bộ; Weaviate
  là cơ sở dữ liệu vector mã nguồn mở
  [được triển khai bằng Go](https://github.com/weaviate/weaviate).

Việc thay thế bằng các dịch vụ tương đương khác sẽ rất đơn giản. Thực tế,
đây chính là nội dung của biến thể thứ hai và thứ ba của máy chủ!
Chúng ta sẽ bắt đầu với biến thể đầu tiên sử dụng trực tiếp các công cụ này.

## Sử dụng trực tiếp Gemini API và Weaviate

Cả Gemini API và Weaviate đều có Go SDK (thư viện client) tiện lợi,
và biến thể máy chủ đầu tiên của chúng ta sử dụng chúng trực tiếp. Mã đầy đủ của
biến thể này [ở thư mục này](https://github.com/golang/example/tree/master/ragserver/ragserver).

Chúng ta sẽ không tái tạo toàn bộ mã trong bài blog này, nhưng đây là một số lưu ý
cần ghi nhớ khi đọc nó:

**Cấu trúc**: cấu trúc mã sẽ quen thuộc với bất kỳ ai đã viết
HTTP server bằng Go. Thư viện client cho Gemini và Weaviate được khởi tạo
và các client được lưu trong một giá trị trạng thái được truyền đến các HTTP handler.

**Đăng ký route**: các route HTTP cho máy chủ của chúng ta rất đơn giản để thiết lập
bằng cách sử dụng [các cải tiến routing](/blog/routing-enhancements) được giới thiệu trong
Go 1.22:

```Go
mux := http.NewServeMux()
mux.HandleFunc("POST /add/", server.addDocumentsHandler)
mux.HandleFunc("POST /query/", server.queryHandler)
```

**Concurrency**: các HTTP handler của máy chủ chúng ta tiếp cận
các dịch vụ khác qua mạng và chờ phản hồi. Điều này không phải là vấn đề
với Go, vì mỗi HTTP handler chạy đồng thời trong goroutine riêng của nó. Máy chủ
RAG này có thể xử lý số lượng lớn yêu cầu đồng thời, và mã của
mỗi handler là tuyến tính và đồng bộ.

**Batch API**: vì yêu cầu `/add/` có thể cung cấp số lượng lớn tài liệu
để thêm vào cơ sở kiến thức, máy chủ tận dụng *batch API* cho cả
embedding (`embModel.BatchEmbedContents`) và Weaviate DB
(`rs.wvClient.Batch`) để có hiệu quả.

## Sử dụng LangChain cho Go

Biến thể máy chủ RAG thứ hai của chúng ta sử dụng LangChainGo để hoàn thành cùng nhiệm vụ.

[LangChain](https://www.langchain.com/) là framework Python phổ biến để
xây dựng ứng dụng LLM.
[LangChainGo](https://github.com/tmc/langchaingo) là tương đương Go của nó. Framework
có một số công cụ để xây dựng ứng dụng từ các thành phần module, và
hỗ trợ nhiều nhà cung cấp LLM và cơ sở dữ liệu vector trong một API chung. Điều này cho phép
nhà phát triển viết mã có thể hoạt động với bất kỳ nhà cung cấp nào và thay đổi nhà cung cấp
rất dễ dàng.

Mã đầy đủ cho biến thể này [ở thư mục này](https://github.com/golang/example/tree/master/ragserver/ragserver-langchaingo).
Bạn sẽ nhận thấy hai điều khi đọc mã:

Thứ nhất, nó ngắn hơn đôi chút so với biến thể trước. LangChainGo đảm nhiệm
việc bọc các API đầy đủ của cơ sở dữ liệu vector trong các interface chung, và ít
mã hơn cần thiết để khởi tạo và xử lý với Weaviate.

Thứ hai, API LangChainGo làm cho việc chuyển đổi nhà cung cấp khá dễ dàng. Giả sử
chúng ta muốn thay thế Weaviate bằng một vector DB khác; trong biến thể trước, chúng ta sẽ
phải viết lại tất cả mã giao tiếp với vector DB để sử dụng API mới. Với
framework như LangChainGo, chúng ta không cần làm vậy nữa. Miễn là LangChainGo
hỗ trợ vector DB mới mà chúng ta quan tâm, chúng ta có thể thay thế
chỉ một vài dòng mã trong máy chủ, vì tất cả các DB đều triển khai một
[interface chung](https://pkg.go.dev/github.com/tmc/langchaingo@v0.1.12/vectorstores#VectorStore):

```Go
type VectorStore interface {
	AddDocuments(ctx context.Context, docs []schema.Document, options ...Option) ([]string, error)
	SimilaritySearch(ctx context.Context, query string, numDocuments int, options ...Option) ([]schema.Document, error)
}
```

## Sử dụng Genkit cho Go

Đầu năm nay, Google đã giới thiệu [Genkit cho Go](https://developers.googleblog.com/en/introducing-genkit-for-go-build-scalable-ai-powered-apps-in-go/),
một framework mã nguồn mở mới để xây dựng ứng dụng LLM. Genkit chia sẻ
một số đặc điểm với LangChain, nhưng khác biệt ở một số khía cạnh khác.

Giống LangChain, nó cung cấp các interface chung có thể được triển khai bởi
các nhà cung cấp khác nhau (như plugin), và do đó làm cho việc chuyển đổi từ cái này sang cái kia
đơn giản hơn. Tuy nhiên, nó không cố gắng quy định cách các thành phần LLM khác nhau
tương tác; thay vào đó, nó tập trung vào các tính năng production như quản lý prompt và
kỹ thuật, và triển khai với hệ thống công cụ lập trình viên tích hợp.

Biến thể máy chủ RAG thứ ba của chúng ta sử dụng Genkit cho Go để hoàn thành cùng nhiệm vụ.
Mã đầy đủ của nó [ở thư mục này](https://github.com/golang/example/tree/master/ragserver/ragserver-genkit).

Biến thể này khá tương tự với LangChainGo, với các interface chung cho
LLM, embedder và vector DB được sử dụng thay vì API nhà cung cấp trực tiếp, làm
cho việc chuyển đổi từ cái này sang cái khác dễ dàng hơn. Ngoài ra, việc triển khai ứng dụng LLM
lên production dễ dàng hơn nhiều với Genkit; chúng ta không triển khai điều này
trong biến thể của mình, nhưng hãy đọc [tài liệu](https://firebase.google.com/docs/genkit-go/get-started-go)
nếu bạn quan tâm.

## Tóm tắt: Go cho ứng dụng LLM

Các mẫu trong bài viết này chỉ cung cấp một ví dụ về những gì có thể để xây dựng
ứng dụng LLM bằng Go. Nó chứng minh sự đơn giản của việc xây dựng
máy chủ RAG mạnh mẽ với tương đối ít mã; điều quan trọng nhất là các mẫu
mang mức độ sẵn sàng cho môi trường production đáng kể vì một số tính năng Go cơ bản.

Làm việc với các dịch vụ LLM thường có nghĩa là gửi yêu cầu REST hoặc RPC đến một
dịch vụ mạng, chờ phản hồi, gửi yêu cầu mới đến các dịch vụ khác dựa trên
điều đó và vân vân. Go xuất sắc ở tất cả những điều này, cung cấp các công cụ tuyệt vời để quản lý
concurrency và sự phức tạp của việc phối hợp các dịch vụ mạng.

Ngoài ra, hiệu suất và độ tin cậy tuyệt vời của Go như một ngôn ngữ Cloud-native
làm cho nó trở thành lựa chọn tự nhiên để triển khai các khối xây dựng cơ bản hơn
của hệ sinh thái LLM. Để xem một số ví dụ, hãy xem các dự án như
[Ollama](https://ollama.com/), [LocalAI](https://localai.io/),
[Weaviate](https://weaviate.io/) hay [Milvus](https://zilliz.com/what-is-milvus).
