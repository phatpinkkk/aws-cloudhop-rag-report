---
title: "Tổng quan kiến trúc"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

### Tổng quan kiến trúc

Dự án này triển khai hệ thống RAG (Retrieval-Augmented Generation) từ đầu đến cuối trên AWS. Kiến trúc theo cách tiếp cận serverless và container hóa để đảm bảo khả năng mở rộng và dễ dàng triển khai.

#### Sơ đồ kiến trúc

![Architecture Diagram](/images/5-Workshop/5.1-Architecture-Overview/architecture.png)

#### Quy trình hoạt động
1. **User Request**: Người dùng gửi truy vấn thông qua ứng dụng frontend.
2. **Frontend**: Ứng dụng React được host trên AWS Amplify.
3. **API Gateway**: Đóng vai trò là điểm vào HTTPS, xử lý CORS và điều hướng request đến backend.
4. **Backend (EC2)**: Dịch vụ FastAPI chạy trên EC2 xử lý truy vấn.
5. **Retrieval**: Backend thực hiện tìm kiếm (dense retrieval) sử dụng S3 Vectors và chỉ mục BM25 lưu trữ trong S3.
6. **Generation**: Ngữ cảnh (context) được trích xuất sẽ được gửi đến Groq API để tạo câu trả lời cuối cùng.