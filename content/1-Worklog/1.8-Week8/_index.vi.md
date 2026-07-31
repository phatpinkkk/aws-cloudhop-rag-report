---
title: "Nhật ký tuần 8"
date: 2026-07-27
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8

* Phối hợp cùng nhóm hoàn thiện những bước tích hợp và triển khai cuối của AWS CloudHop RAG.
* Kiểm tra hành vi retrieval và end-to-end của hệ thống sau khi triển khai.
* Tổng hợp kết quả đánh giá cuối cho retrieval và answer generation.
* Phân tích chất lượng retrieval, độ chính xác câu trả lời và hiệu năng khi vận hành.
* Hoàn thiện tài liệu kỹ thuật và báo cáo dự án.

### Công việc thực hiện trong tuần

| Ngày | Công việc | Tài liệu tham khảo |
| --- | --- | --- |
| 27/07/2026 | - Cùng nhóm rà soát kiến trúc AWS cuối cùng và trạng thái triển khai.<br>- Kiểm tra các retrieval artifact và cấu hình evaluation được hệ thống sử dụng.<br>- Chuẩn bị input, metric và cấu trúc kết quả phục vụ bước kiểm tra cuối. | |
| 28/07/2026 | - Xác nhận các artifact HotpotQA Distractor v002 đã hiệu chỉnh được retrieval pipeline sử dụng đúng cách.<br>- Chạy retrieval smoke test và kiểm tra bằng chứng trả về cho một số câu hỏi HotpotQA.<br>- Kiểm tra supporting-title coverage và retrieval output trước khi tổng hợp kết quả. | |
| 29/07/2026 | - Phối hợp cùng nhóm kiểm tra chức năng của ứng dụng đã triển khai.<br>- Thử toàn bộ luồng hỏi đáp thông qua hệ thống thực tế.<br>- Kiểm tra câu trả lời và các nguồn bằng chứng được trả về để đánh giá tính đúng đắn và nhất quán. | |
| 30/07/2026 | - Tổng hợp kết quả retrieval evaluation từ benchmark đã hiệu chỉnh.<br>- Phân tích Recall, MRR, nDCG, supporting-title coverage và retrieval latency.<br>- Rà soát kết quả end-to-end cố định và đối chiếu câu trả lời sinh ra với gold answer của HotpotQA. | |
| 31/07/2026 | - Tổng hợp các kết quả cuối về retrieval và answer quality.<br>- Phân tích mối liên hệ giữa chất lượng bằng chứng truy xuất và độ chính xác của câu trả lời cuối.<br>- Cập nhật tài liệu về evaluation, architecture, deployment và testing.<br>- Hoàn thiện Worklog, Workshop và các phần còn lại của báo cáo thực tập. | |

### Kết quả đạt được

* Phối hợp cùng nhóm kiểm tra và hoàn thiện quá trình triển khai AWS CloudHop RAG.
* Xác nhận ứng dụng đã triển khai có thể tiếp nhận câu hỏi và trả về câu trả lời kèm các nguồn bằng chứng hỗ trợ.
* Tổng hợp kết quả retrieval trên benchmark HotpotQA đã hiệu chỉnh và kết quả end-to-end evaluation cuối cùng.
* Phân tích chất lượng retrieval bằng supporting-title recall, Recall@k, MRR và nDCG.
* Phân tích latency của retrieval và generation để hiểu rõ hơn hành vi runtime của hệ thống.
* Xác nhận kết quả end-to-end 20 câu hỏi cuối cùng đạt **Answer EM = 0.7500**, **Answer F1 = 0.7750**, với **15/20 câu trả lời được đánh giá là đúng**.
* Hoàn thiện phần lớn tài liệu evaluation và đóng góp vào quá trình hoàn thiện Workshop cũng như báo cáo thực tập.