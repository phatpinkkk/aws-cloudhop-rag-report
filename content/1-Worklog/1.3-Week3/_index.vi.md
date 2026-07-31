---
title: "Nhật ký tuần 3"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3

* Hiểu vòng đời cơ bản của một hệ thống machine learning và cách ứng dụng AI có thể được triển khai trên cloud.
* Tìm hiểu text embedding, vector similarity và semantic retrieval.
* Hiểu Retrieval-Augmented Generation và vai trò của retrieval trong việc giúp câu trả lời của LLM có căn cứ.
* Phân biệt bài toán hỏi đáp single-hop và multi-hop.
* Xác định hướng phát triển cho CloudHop RAG và lựa chọn benchmark phù hợp.

### Công việc thực hiện trong tuần

| Ngày | Công việc | Tài liệu tham khảo |
| --- | --- | --- |
| 22/06/2026 | - Ôn lại vòng đời machine learning:<br>&emsp; + Chuẩn bị dữ liệu<br>&emsp; + Model inference<br>&emsp; + Evaluation<br>&emsp; + Deployment<br>- Tìm hiểu cách các workload AI/ML có thể kết hợp với hạ tầng AWS. | <https://cloudjourney.awsstudygroup.com/> |
| 23/06/2026 | - Tìm hiểu text embedding và cách biểu diễn văn bản dưới dạng vector.<br>- Tìm hiểu cosine similarity và semantic search.<br>- So sánh truy xuất dựa trên từ khóa với truy xuất theo ngữ nghĩa. | |
| 24/06/2026 | - Tìm hiểu Retrieval-Augmented Generation.<br>- Phân tích pipeline RAG cơ bản:<br>&emsp; + Truy xuất bằng chứng bên ngoài<br>&emsp; + Xây dựng ngữ cảnh<br>&emsp; + Sinh câu trả lời<br>- Hiểu vai trò của bằng chứng truy xuất trong việc giúp câu trả lời có căn cứ hơn. | |
| 25/06/2026 | - Tìm hiểu bài toán single-hop và multi-hop question answering.<br>- Phân tích vì sao một số câu hỏi cần kết hợp bằng chứng từ nhiều tài liệu.<br>- Xác định luồng ban đầu cho CloudHop RAG: question → retrieval → evidence selection → generation → answer. | |
| 26/06/2026 | - Tìm hiểu benchmark HotpotQA.<br>- Phân tích bridge question, comparison question, candidate context, answer và supporting fact.<br>- Chọn HotpotQA Distractor làm benchmark có phạm vi kiểm soát chính của dự án. | <https://hotpotqa.github.io/> |

### Kết quả đạt được

* Hiểu các giai đoạn chính trong vòng đời của một hệ thống machine learning.
* Hiểu cách text embedding biểu diễn thông tin ngữ nghĩa.
* Phân biệt được lexical retrieval và semantic retrieval.
* Nắm được nguyên lý hoạt động của Retrieval-Augmented Generation.
* Hiểu vì sao bài toán multi-hop cần tổng hợp các bằng chứng bổ trợ từ nhiều tài liệu.
* Xác định CloudHop RAG là một hệ thống RAG đa bước.
* Lựa chọn HotpotQA làm benchmark chính để đánh giá retrieval và chất lượng câu trả lời.