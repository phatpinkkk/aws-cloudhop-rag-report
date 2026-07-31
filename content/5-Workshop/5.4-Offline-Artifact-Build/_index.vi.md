---
title: "Xây dựng Artifact Offline"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Trước khi CloudHop RAG có thể tiếp nhận câu hỏi và thực hiện retrieval, dữ liệu HotpotQA cần được xử lý thành một bộ artifact hoàn chỉnh. Công việc này được thực hiện offline để những bước tốn nhiều thời gian như chuẩn bị tài liệu, xây dựng BM25 index và tạo embedding không phải lặp lại mỗi khi người dùng gửi request.

Bộ artifact cuối cùng được xây dựng từ **500 câu hỏi thuộc tập validation của HotpotQA Distractor**. Dữ liệu được chuyển thành parent document và child chunk, sau đó lập chỉ mục cho cả lexical retrieval và dense retrieval, kiểm tra tính hợp lệ và đóng gói để sử dụng trong quá trình triển khai AWS ở các phần tiếp theo.

---

## 1. Môi trường xây dựng

Quá trình xây dựng artifact offline được thực hiện trên **Google Colab**. GPU chủ yếu được sử dụng ở bước tạo embedding bằng BGE-M3, đây là phần tốn nhiều tài nguyên tính toán nhất trong giai đoạn tiền xử lý.

Công việc này được tách khỏi EC2 backend ngay từ đầu. EC2 chỉ cần tải và sử dụng các artifact đã chuẩn bị sẵn, thay vì phải tự chia tài liệu, xây dựng index hoặc tạo lại embedding khi hệ thống đang phục vụ người dùng.

Notebook chính dùng để xây dựng artifact là:

`backend/notebooks/build_s3_offline_artifacts.ipynb`

---

## 2. Dataset và bộ artifact cuối

CloudHop RAG sử dụng **HotpotQA Distractor** vì mỗi câu hỏi đã đi kèm một tập tài liệu ứng viên giới hạn, trong đó có cả bằng chứng hỗ trợ được gán nhãn và các tài liệu gây nhiễu. Cách tổ chức này phù hợp để xây dựng và đánh giá pipeline retrieval đa bước trong một phạm vi rõ ràng.

Phiên bản artifact cuối cùng là **v002**.

| Hạng mục | Kết quả cuối |
| --- | ---: |
| Số câu hỏi đánh giá | 500 |
| Parent document | 4.937 |
| Số passage đã xử lý | 4.963 |
| Child vector | 8.279 |
| Embedding model | `BAAI/bge-m3` |
| Kích thước embedding | 1.024 |
| Supporting title bị thiếu | 0 |
| Title collision | 0 |

Trong quá trình phát triển, một phiên bản artifact trước đó được phát hiện không chứa đầy đủ các supporting document cần thiết cho một số câu hỏi. Vì vậy, nhóm không sử dụng kết quả từ phiên bản này mà xây dựng lại bộ dữ liệu bằng HotpotQA Distractor. Phiên bản **v002** chỉ được đưa vào đánh giá và triển khai sau khi đã vượt qua các bước kiểm tra dữ liệu.

---

## 3. Parent document và child chunk

Retrieval pipeline sử dụng cấu trúc tài liệu **parent-child**.

Mỗi bài viết nguồn được giữ lại dưới dạng **parent document**, trong khi các **child chunk** nhỏ hơn được tạo ra để phục vụ retrieval. Các chunk nhỏ giúp việc so khớp câu hỏi với nội dung chính xác hơn, còn parent document giữ lại phần ngữ cảnh rộng hơn để sử dụng ở các bước xử lý và sinh câu trả lời sau đó.

Cấu hình chunking được sử dụng trong bộ artifact cuối:

- kích thước child chunk: **500 ký tự**
- phần chồng lấp giữa hai chunk: **100 ký tự**

Mối quan hệ giữa các thành phần có thể hình dung như sau:

**Bài viết nguồn → parent document → các child chunk**

Mỗi child chunk đều giữ tham chiếu đến parent document tương ứng. Nhờ đó, khi một chunk được retrieval tìm thấy, hệ thống có thể ánh xạ trở lại tài liệu đầy đủ hơn trước khi xây dựng context cho LLM.

---

## 4. Xây dựng BM25 và BGE-M3 embedding

Cùng một tập child chunk được chuẩn bị cho hai phương pháp retrieval bổ trợ cho nhau.

### BM25

BM25 index được xây dựng trên các child chunk để phục vụ lexical retrieval. Phương pháp này đặc biệt hữu ích khi câu hỏi chứa tên riêng, thực thể, con số hoặc những thuật ngữ có tính phân biệt cao.

Các BM25 artifact được lưu lại sau khi xây dựng và sẽ được tải lên **Amazon S3** ở bước triển khai. Khi FastAPI backend khởi động trên EC2, các artifact này được tải về và sử dụng cho lexical retrieval.

### BGE-M3 Embedding

Các child chunk đồng thời được mã hóa bằng **`BAAI/bge-m3`** để tạo dense vector có **1.024 chiều**, phục vụ semantic retrieval.

Các vector này được chuẩn bị theo định dạng phù hợp để đưa vào **Amazon S3 Vectors**, nơi chúng sẽ được lưu trữ và truy vấn khi hệ thống xử lý câu hỏi online.

![Quá trình tạo embedding cho các child document](/images/5-Workshop/5.4-Offline-Artifact-Build/embedding-progress.png)

Việc sử dụng cả BM25 và dense embedding giúp backend có hai cách khác nhau để tìm bằng chứng trên cùng một corpus. Kết quả của hai phương pháp sẽ được kết hợp trong RAG pipeline khi hệ thống xử lý câu hỏi thực tế.

---

## 5. Kiểm tra artifact

Trước khi đưa bất kỳ dữ liệu nào lên AWS, toàn bộ bộ artifact được kiểm tra để bảo đảm benchmark và các thành phần dùng cho deployment nhất quán với nhau.

Các bước kiểm tra cuối bao gồm:

- có đủ 500 câu hỏi dùng cho evaluation;
- toàn bộ supporting title cần thiết đều xuất hiện trong corpus;
- ánh xạ giữa parent document và child document là hợp lệ;
- embedding có đúng 1.024 chiều;
- số lượng vector đúng với số lượng mong đợi;
- không xuất hiện title collision;
- tất cả các file artifact cần thiết đều tồn tại.

Phiên bản **v002** vượt qua toàn bộ các bước kiểm tra trên và có **0 supporting title bị thiếu**.

![Kết quả kiểm tra artifact cuối](/images/5-Workshop/5.4-Offline-Artifact-Build/validation-checklist.png)

Bước kiểm tra này đặc biệt quan trọng đối với retrieval evaluation. Nếu bằng chứng cần thiết không có trong corpus ngay từ đầu, một retriever dù tốt đến đâu cũng không thể tìm được nó, và các metric sau đó sẽ không phản ánh đúng chất lượng của hệ thống.

---

## 6. Bộ artifact hoàn chỉnh

Sau khi hoàn tất quá trình xây dựng và kiểm tra, dự án có một bộ artifact được quản lý theo phiên bản và sẵn sàng đưa lên AWS.

| Artifact | Mục đích | Nơi sử dụng tiếp theo |
| --- | --- | --- |
| Corpus và evaluation files | Dữ liệu nguồn và thông tin tham chiếu cho benchmark | Amazon S3 |
| Parent document | Cung cấp ngữ cảnh rộng hơn cho bằng chứng đã retrieval | Amazon S3 |
| Child document | Đơn vị retrieval và thông tin ánh xạ về parent | Amazon S3 |
| BM25 artifact | Lexical retrieval | Amazon S3 |
| Index manifest | Ghi lại phiên bản và cấu hình của artifact | Amazon S3 |
| BGE-M3 vector | Dense semantic retrieval | Amazon S3 Vectors |

Các identifier cuối cùng được sử dụng trong quá trình triển khai là:

| Hạng mục | Giá trị |
| --- | --- |
| Processed ID | `hotpotqa-val500-v002` |
| Vector index | `hotpotqa-val500-bge-m3-v002` |
| Embedding model | `BAAI/bge-m3` |

Đến thời điểm này, bộ retrieval artifact đã hoàn chỉnh nhưng vẫn chưa được đưa vào môi trường AWS. **Phần 5.5** sẽ tải các file cần lưu trữ lâu dài lên Amazon S3, còn **Phần 5.6** sẽ tạo và nạp dense vector vào Amazon S3 Vectors.
