---
title: "Nhật ký tuần 4"
date: 2026-06-29
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4

* Phân tích chi tiết cấu trúc dữ liệu của HotpotQA.
* Chuẩn bị các cấu trúc dữ liệu có thể tái sử dụng cho những thử nghiệm retrieval.
* Xây dựng các baseline ban đầu cho lexical retrieval và dense retrieval.
* Đánh giá chất lượng retrieval độc lập với bước sinh câu trả lời.
* Xác định những hạn chế của single-pass retrieval đối với câu hỏi multi-hop.

### Công việc thực hiện trong tuần

| Ngày | Công việc | Tài liệu tham khảo |
| --- | --- | --- |
| 29/06/2026 | - Phân tích cấu trúc dataset HotpotQA.<br>- Kiểm tra question ID, question, answer, supporting fact, context và question type.<br>- Xác định những trường dữ liệu cần thiết cho retrieval evaluation. | <https://hotpotqa.github.io/> |
| 30/06/2026 | - Thiết kế quy trình chuẩn bị dữ liệu ban đầu.<br>- Chuyển context của HotpotQA thành các đơn vị bằng chứng có thể truy xuất.<br>- Căn chỉnh metadata của câu hỏi với candidate evidence và supporting fact.<br>- Lưu các artifact phục vụ kiểm tra và tái sử dụng dữ liệu. | |
| 01/07/2026 | - Xây dựng lexical retrieval baseline bằng TF-IDF.<br>- Thử retrieval với nhiều giá trị top-k.<br>- Xác định cách đánh giá retrieval dựa trên khả năng tìm lại supporting evidence được gán nhãn. | |
| 02/07/2026 | - Xây dựng dense retrieval baseline bằng Sentence Transformers MiniLM.<br>- Tạo vector representation cho các candidate retrieval.<br>- So sánh hành vi semantic retrieval với baseline TF-IDF. | |
| 03/07/2026 | - Đánh giá TF-IDF và MiniLM với nhiều cấu hình top-k.<br>- Kiểm tra các trường hợp retrieval thành công và thất bại.<br>- Phân tích nguyên nhân single-pass retrieval có thể bỏ sót bằng chứng cần thiết cho câu hỏi multi-hop.<br>- Chưa đưa answer generation vào benchmark để có thể đánh giá riêng chất lượng retrieval. | |

### Kết quả đạt được

* Hoàn thành quy trình kiểm tra và chuẩn bị HotpotQA ban đầu.
* Xây dựng được cấu trúc dữ liệu có thể tái sử dụng cho question, context và supporting evidence.
* Xây dựng TF-IDF lexical retrieval baseline.
* Xây dựng MiniLM dense retrieval baseline.
* Đánh giá retrieval trên nhiều giá trị top-k.
* Thiết lập một baseline chỉ đánh giá retrieval trước khi đưa answer generation vào pipeline.
* Nhận thấy cần có những phương pháp hybrid retrieval và multi-hop retrieval mạnh hơn để xử lý tốt các câu hỏi phức tạp.