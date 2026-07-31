---
title: "Nhật ký tuần 7"
date: 2026-07-20
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu tuần 7

* Đánh giá định lượng retrieval pipeline trên benchmark v002 đã được hiệu chỉnh.
* Đo chất lượng retrieval và runtime trước khi hoàn thiện triển khai production.
* Cải thiện độ ổn định của các benchmark chạy trong thời gian dài.
* Đánh giá toàn bộ pipeline từ retrieval đến answer generation.
* Hoàn thiện kiến trúc triển khai AWS và chuẩn bị các artifact phục vụ môi trường cloud.

### Công việc thực hiện trong tuần

| Ngày | Công việc | Tài liệu tham khảo |
| --- | --- | --- |
| 20/07/2026 | - Tái tạo vector index v002 từ các BGE-M3 vector đã chuẩn bị.<br>- Chạy smoke test trên một số câu hỏi HotpotQA đã biết trước bằng chứng đúng.<br>- Kiểm tra candidate retrieval, selected evidence, supporting-title coverage, reranking và các trường latency. | |
| 21/07/2026 | - Chạy retrieval benchmark trung gian.<br>- Đánh giá chất lượng candidate và selected evidence bằng Recall, MRR, nDCG và supporting-title coverage.<br>- Kiểm tra các trường hợp retrieval khó hoặc thất bại. | |
| 22/07/2026 | - Phân tích thời gian của decomposition, retrieval/adaptive planning, reranking và toàn bộ retrieval pipeline.<br>- Bổ sung retry, checkpoint, lưu attempt, bỏ qua các question ID đã hoàn thành và hỗ trợ resume cho các lượt đánh giá dài. | |
| 23/07/2026 | - Hoàn thành benchmark retrieval 500 câu hỏi trên artifact đã hiệu chỉnh.<br>- Phân tích chất lượng retrieval, phân bố latency và độ ổn định theo thời gian chạy.<br>- Chạy bộ đánh giá end-to-end cố định sử dụng bằng chứng truy xuất và Groq để sinh câu trả lời. | |
| 24/07/2026 | - Tổng hợp kết quả retrieval và end-to-end evaluation.<br>- Hoàn thiện kiến trúc AWS production: Amplify → API Gateway → EC2 FastAPI → S3 / S3 Vectors → Groq.<br>- Chuẩn bị corpus, BM25, manifest và vector artifact phục vụ triển khai trên AWS. | |

### Kết quả đạt được

* Hoàn thành benchmark retrieval đã hiệu chỉnh trên **500 câu hỏi, với 500/500 câu chạy thành công**.
* Candidate stage đạt supporting-title recall trung bình **0.9920** và tỷ lệ tìm đủ toàn bộ supporting title **0.9840**.
* Selected Top-10 đạt supporting-title recall **0.9740**, MRR **0.9446** và nDCG@10 **0.9162**.
* Đo được latency trung bình của retrieval pipeline là **25,91 giây** và median latency là **25,72 giây**.
* Bổ sung cơ chế resume và khôi phục cho các benchmark phải chạy trong thời gian dài.
* Hoàn thành bộ đánh giá end-to-end cố định gồm 20 câu hỏi, đạt **Answer EM = 0.7500**, **Answer F1 = 0.7750** và **15/20 câu trả lời đúng**.
* Hoàn thiện kiến trúc AWS và chuẩn bị các retrieval artifact đã được kiểm tra để phục vụ triển khai production.