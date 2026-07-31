---
title: "Kiến trúc hệ thống"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

CloudHop RAG được triển khai dưới dạng một ứng dụng RAG đa bước chạy trên web tại **Asia Pacific (Singapore) Region (`ap-southeast-1`)**. Kiến trúc được chia thành các phần riêng cho frontend, lớp API, backend và hệ thống lưu trữ phục vụ retrieval. Mỗi thành phần đảm nhiệm một vai trò cụ thể, nhưng vẫn phối hợp với nhau trong cùng một luồng xử lý từ lúc người dùng gửi câu hỏi đến khi nhận được câu trả lời.

Frontend được triển khai trên **AWS Amplify**. Từ giao diện web, request của người dùng được gửi qua **Amazon API Gateway** đến FastAPI backend chạy trên **Amazon EC2**. Backend thực hiện lexical retrieval bằng các BM25 artifact lưu trên **Amazon S3**, đồng thời thực hiện dense retrieval thông qua **Amazon S3 Vectors**. Các bằng chứng thu được sau đó được tổng hợp và chọn lọc trước khi phần ngữ cảnh phù hợp được gửi đến **Groq API** để sinh câu trả lời.

Phần này chỉ tập trung vào cách hệ thống được tổ chức ở mức tổng thể. Các phần tiếp theo sẽ lần lượt trình bày cách từng thành phần được chuẩn bị, triển khai và kiểm thử.

---

## 1. Tổng quan kiến trúc

<img src="/images/2-Proposal/AWS-RAG.drawio.png" alt="Sơ đồ kiến trúc" width="700">

Ở mức tổng quát, một request đi qua hệ thống theo luồng:

**Người dùng → AWS Amplify → Amazon API Gateway → Amazon EC2 → Amazon S3 / Amazon S3 Vectors → Groq API → Câu trả lời**

Kiến trúc gồm bốn phần chính:

1. **Frontend** – ứng dụng React/Vite được triển khai trên AWS Amplify.
2. **Lớp API** – Amazon API Gateway cung cấp HTTPS endpoint cho trình duyệt và chuyển tiếp request đến backend.
3. **Lớp ứng dụng** – FastAPI chạy trên Amazon EC2, chịu trách nhiệm điều phối retrieval, xử lý bằng chứng và các lần gọi LLM.
4. **Lớp lưu trữ phục vụ retrieval** – Amazon S3 lưu corpus đã xử lý và các BM25 artifact, trong khi Amazon S3 Vectors lưu dense vector phục vụ semantic retrieval.

**Groq API** nằm ngoài môi trường AWS và cung cấp LLM cho các bước như query decomposition, multi-hop planning và sinh câu trả lời cuối cùng.

---

## 2. Xử lý offline và online

Một quyết định quan trọng trong thiết kế CloudHop RAG là tách **quá trình chuẩn bị artifact offline** khỏi **quá trình xử lý câu hỏi online**.

| | Pipeline offline | Pipeline online |
| --- | --- | --- |
| Thời điểm chạy | Khi cần chuẩn bị một phiên bản corpus hoặc index mới | Mỗi khi người dùng gửi request |
| Công việc chính | Chuẩn bị tài liệu, xây dựng BM25 artifact, tạo embedding, kiểm tra và tải artifact lên AWS | Truy xuất bằng chứng, xử lý câu hỏi, gọi LLM và trả về câu trả lời |
| Môi trường | Notebook hoặc script chuẩn bị dữ liệu | FastAPI backend trên EC2 |

Phần offline thực hiện trước những công việc tốn nhiều thời gian và tài nguyên. Khi hệ thống đi vào hoạt động, backend chỉ cần sử dụng lại các artifact đã có thay vì xây dựng chúng lại cho từng request.

Cách tổ chức này giúp backend tập trung vào retrieval và answer generation. Đồng thời, một phiên bản artifact mới có thể được xây dựng và kiểm tra độc lập trước khi đưa vào hệ thống đang triển khai.

---

## 3. Luồng chuẩn bị artifact offline

![Pipeline chuẩn bị artifact offline](/images/5-Workshop/5.3-Architecture/offline-pipeline.png)

Trước khi triển khai ứng dụng, corpus HotpotQA được xử lý thành các artifact sẵn sàng cho retrieval.

Dự án sử dụng cấu trúc tài liệu **parent-child**. Parent document giữ lại phần nội dung rộng hơn của bài viết, còn child chunk là những đoạn nhỏ hơn được dùng trực tiếp trong quá trình retrieval.

Các child chunk được lập chỉ mục theo hai cách bổ trợ cho nhau:

- **BM25** hỗ trợ lexical retrieval, đặc biệt hiệu quả khi câu hỏi chứa tên riêng, thực thể hoặc thuật ngữ cần khớp chính xác.
- **BGE-M3 embedding** hỗ trợ semantic retrieval trong những trường hợp câu hỏi và bằng chứng liên quan sử dụng cách diễn đạt khác nhau.

Các tài liệu đã xử lý, BM25 artifact và metadata của index được lưu trên **Amazon S3**. Dense vector được lưu trên **Amazon S3 Vectors**.

Nhờ đó, backend có thể sử dụng đồng thời hai hướng retrieval khác nhau mà không phải thực hiện lại quá trình chuẩn bị dữ liệu trong lúc ứng dụng đang phục vụ người dùng. Toàn bộ quy trình xây dựng và kiểm tra artifact được trình bày chi tiết trong **Phần 5.4**.

---

## 4. Luồng xử lý câu hỏi online

![Luồng xử lý câu hỏi online](/images/5-Workshop/5.3-Architecture/online-query-flow.png)

Khi người dùng gửi một câu hỏi, backend sử dụng các artifact đã được chuẩn bị để tìm bằng chứng và sinh câu trả lời.

Luồng xử lý chính là:

**Câu hỏi → query decomposition tùy chọn → BM25 + dense retrieval → gộp candidate → mở rộng về parent document → adaptive multi-hop retrieval → reranking tùy chọn → xây dựng context → sinh câu trả lời**

Các bước chính gồm:

1. **Query decomposition** *(tùy chọn)* – một câu hỏi phức tạp có thể được chia thành các câu hỏi nhỏ hơn để hỗ trợ retrieval.
2. **Hybrid retrieval** – BM25 và S3 Vectors cùng tìm kiếm bằng chứng theo hai hướng lexical và semantic.
3. **Gộp candidate** – kết quả từ hai phương pháp retrieval được kết hợp thành một candidate set chung.
4. **Mở rộng về parent document** – các child chunk được truy xuất sẽ được ánh xạ ngược về parent document để hệ thống có thêm ngữ cảnh khi xử lý bằng chứng.
5. **Adaptive multi-hop retrieval** – nếu bằng chứng hiện tại chưa đủ, hệ thống có thể thực hiện thêm một bước retrieval dựa trên thông tin đã tìm được trước đó.
6. **Reranking** *(tùy chọn)* – các candidate có thể được chấm điểm lại trước khi chọn context cuối cùng.
7. **Sinh câu trả lời** – bằng chứng đã chọn được gửi đến Groq để tạo câu trả lời ngắn và tập trung vào thông tin cần thiết.
8. **Trả kết quả** – câu trả lời được gửi lại cho người dùng kèm theo các nguồn hỗ trợ.

Cấu hình khi triển khai thực tế được rút gọn hơn so với cấu hình dùng để đánh giá chất lượng, vì hệ thống còn phải cân nhắc tài nguyên tính toán và thời gian phản hồi. Sự đánh đổi giữa chất lượng và latency sẽ được phân tích riêng trong **Phần 5.11**.

---

## 5. Vai trò của các dịch vụ AWS

Mỗi dịch vụ AWS trong hệ thống đảm nhiệm một vai trò riêng.

| Dịch vụ | Vai trò trong CloudHop RAG |
| --- | --- |
| **AWS Amplify Hosting** | Triển khai và phục vụ React/Vite frontend |
| **Amazon API Gateway** | Cung cấp HTTPS API cho frontend |
| **Amazon EC2** | Chạy FastAPI backend và điều phối pipeline RAG |
| **Amazon S3** | Lưu processed document, BM25 artifact và manifest |
| **Amazon S3 Vectors** | Lưu dense vector và thực hiện semantic retrieval |
| **AWS IAM** | Kiểm soát quyền truy cập của backend đến các tài nguyên AWS |
| **AWS Systems Manager** | Cung cấp Session Manager để quản trị EC2 |

**Groq** là dịch vụ LLM bên ngoài AWS được pipeline RAG sử dụng.

Điểm cần phân biệt rõ là **Amazon S3 và Amazon S3 Vectors chịu trách nhiệm lưu trữ dữ liệu phục vụ retrieval, còn Amazon EC2 là nơi thực thi logic RAG**. BM25 không chạy bên trong Amazon S3. Các file index của BM25 chỉ được lưu ở đó và được backend tải về để sử dụng. Ngược lại, dense vector search được thực hiện trực tiếp thông qua Amazon S3 Vectors.

Các phần tiếp theo của workshop bám theo đúng cấu trúc kiến trúc này:

- **5.4** chuẩn bị các artifact offline.
- **5.5** tải các artifact cần lưu lâu dài lên Amazon S3.
- **5.6** tạo và nạp dữ liệu vào Amazon S3 Vectors.
- **5.7** triển khai FastAPI backend trên Amazon EC2.
- **5.8** đưa backend ra ngoài thông qua Amazon API Gateway.
- **5.9** triển khai frontend bằng AWS Amplify.

Sau khi đã nắm được kiến trúc tổng thể, bước tiếp theo là xây dựng và kiểm tra các retrieval artifact trước khi tải chúng lên AWS.