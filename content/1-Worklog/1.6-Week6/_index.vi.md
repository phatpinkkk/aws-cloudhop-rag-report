---
title: "Nhật ký tuần 6"
date: 2026-07-13
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu tuần 6

* Chuyển các thử nghiệm retrieval từ code phụ thuộc nhiều vào notebook sang cấu trúc dự án có thể tái sử dụng và tái lập.
* Chuẩn hóa quá trình chuẩn bị dữ liệu, cấu hình, quản lý artifact và validation.
* Phát hiện và xử lý các vấn đề căn chỉnh giữa câu hỏi HotpotQA và supporting evidence.
* Xây dựng benchmark artifact hợp lệ cho giai đoạn đánh giá cuối.
* Chuẩn bị các retrieval artifact có thể lưu trữ lâu dài và đưa lên AWS.

### Công việc thực hiện trong tuần

| Ngày | Công việc | Tài liệu tham khảo |
| --- | --- | --- |
| 13/07/2026 | - Refactor các phần logic dùng chung thành những source module có thể tái sử dụng.<br>- Xây dựng schema chung, retriever interface, benchmark configuration và validation utility.<br>- Giảm sự phụ thuộc vào trạng thái riêng của notebook. | |
| 14/07/2026 | - Xây dựng cơ chế quản lý artifact cho processed document, BM25 index, embedding, mapping và manifest.<br>- Bổ sung deterministic configuration và artifact hashing.<br>- Tổ chức cấu hình dự án bằng các file cấu hình có thể tái sử dụng. | |
| 15/07/2026 | - Chạy quá trình chuẩn bị dữ liệu ở quy mô lớn hơn.<br>- Phát hiện những trường hợp supporting fact không xuất hiện trong candidate evidence.<br>- Kiểm tra lại mối liên hệ giữa question, context và supporting title.<br>- Tăng cường validation thay vì bỏ qua các mẫu dữ liệu không hợp lệ. | |
| 16/07/2026 | - Kiểm tra artifact đánh giá 500 câu hỏi đầu tiên.<br>- Phát hiện artifact v001 không phù hợp để dùng cho retrieval evaluation cuối cùng vì corpus không phải lúc nào cũng chứa đầy đủ gold supporting title cần thiết.<br>- Quyết định xây dựng lại benchmark thay vì sử dụng các kết quả đánh giá không hợp lệ. | |
| 17/07/2026 | - Xây dựng lại benchmark bằng HotpotQA Distractor.<br>- Bổ sung bước kiểm tra gold-title coverage rõ ràng.<br>- Sinh parent document, child document, child-to-parent mapping, BM25 artifact, BGE-M3 vector, manifest và các artifact dùng để import vào S3 Vectors.<br>- Kiểm tra và xác nhận benchmark v002 sau khi sửa. | |

### Kết quả đạt được

* Refactor dự án thành các module có thể tái sử dụng cho data preparation, retrieval, artifact management và evaluation.
* Chuẩn hóa cấu hình và cách lưu trữ artifact lâu dài.
* Phát hiện và xử lý một vấn đề quan trọng liên quan đến sự căn chỉnh của supporting evidence.
* Không sử dụng benchmark v001 sau khi xác định kết quả từ artifact này có thể gây hiểu sai về chất lượng retrieval.
* Xây dựng artifact **HotpotQA Distractor v002** đã được hiệu chỉnh.
* Xác nhận corpus cuối gồm **500 câu hỏi validation, 4.937 parent document và 8.279 BGE-M3 child vector**.
* Kiểm tra và xác nhận benchmark mới **không thiếu gold supporting title**.
* Chuẩn bị đầy đủ artifact để đưa lên Amazon S3 và Amazon S3 Vectors ở giai đoạn triển khai.