---
title: "Kiểm thử và xác thực"
date: 2026-07-31
weight: 10
chapter: false
pre: " <b> 5.10. </b> "
---

Sau khi các thành phần AWS đã được kết nối với nhau, hệ thống cần được kiểm tra từ backend trên EC2 cho đến giao diện trên trình duyệt. Phần này tập trung vào **functional validation**, tức là xác nhận các thành phần có thể giao tiếp đúng với nhau và hoàn thành trọn vẹn một request end-to-end. Chất lượng retrieval và chất lượng câu trả lời sẽ được đánh giá riêng ở Phần 5.11.

## 1. Kiểm tra Backend và Retrieval

FastAPI backend chạy trên Amazon EC2 dưới dạng `systemd` service có tên `aws-rag-api`.

Có thể kiểm tra trạng thái service bằng:

`sudo systemctl status aws-rag-api`

Endpoint `/health` dùng để xác nhận ứng dụng đang hoạt động:

`curl http://127.0.0.1:8000/health`

Sau đó, retrieval pipeline được khởi tạo bằng:

`curl -X POST http://127.0.0.1:8000/warmup`

Nếu warm-up hoàn tất thành công, có thể xác nhận backend đã khởi tạo được các thành phần retrieval cần thiết cho pipeline đang triển khai.

## 2. Kiểm tra Amazon API Gateway

Sau khi backend đã hoạt động đúng khi gọi trực tiếp trên EC2, hệ thống tiếp tục được kiểm tra thông qua Amazon API Gateway.

API đã triển khai cung cấp ba endpoint:

- `GET /health`
- `POST /warmup`
- `POST /query`

Route `/health` được dùng để xác nhận request HTTPS có thể đi qua API Gateway và đến đúng EC2 backend. Cấu hình CORS cũng được kiểm tra để bảo đảm request từ frontend trên AWS Amplify được chấp nhận.

Tiếp theo, một câu hỏi thực tế được gửi qua `/query`. Nếu hệ thống trả về thành công, có thể xác nhận backend đã thực hiện được retrieval, sinh câu trả lời và gửi lại các nguồn bằng chứng thông qua public API.

## 3. Kiểm tra trên trình duyệt

Bước xác thực chức năng cuối cùng được thực hiện trực tiếp từ ứng dụng đã triển khai trên AWS Amplify.

Một request thành công trên trình duyệt sẽ đi qua toàn bộ hệ thống theo luồng:

**Trình duyệt → AWS Amplify → Amazon API Gateway → Amazon EC2 → Amazon S3 + Amazon S3 Vectors → Groq → Câu trả lời + nguồn hỗ trợ**

Kết quả này cho thấy frontend, lớp API, backend, hệ thống lưu trữ phục vụ retrieval và LLM bên ngoài đã được kết nối đúng với nhau.

## 4. Kết quả xác thực

| Hạng mục kiểm tra | Kết quả mong đợi | Trạng thái |
| --- | --- | --- |
| EC2 backend | FastAPI service đang hoạt động | Đạt |
| Health endpoint | Trả về response thành công | Đạt |
| Pipeline warm-up | Retrieval pipeline khởi tạo thành công | Đạt |
| Amazon S3 | Các retrieval artifact cần thiết có thể được truy cập | Đạt |
| Amazon S3 Vectors | Dense retrieval trả về kết quả | Đạt |
| API Gateway | Request HTTPS đến được backend | Đạt |
| CORS | Amplify origin được chấp nhận | Đạt |
| Query endpoint | Trả về câu trả lời và các nguồn hỗ trợ | Đạt |
| Amplify frontend | Trình duyệt hiển thị đầy đủ kết quả | Đạt |

Các bước trên xác nhận CloudHop RAG đang hoạt động như một ứng dụng thống nhất. Một câu hỏi của người dùng có thể đi qua toàn bộ hệ thống đã triển khai và trả về câu trả lời kèm bằng chứng hỗ trợ.
