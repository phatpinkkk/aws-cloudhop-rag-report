---
title: "Kiến trúc hệ thống"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

CloudHop RAG được triển khai dưới dạng một ứng dụng RAG trên nền tảng web tại **Asia Pacific (Singapore) Region (`ap-southeast-1`)**. Kiến trúc tách riêng frontend, lớp API, RAG backend và hệ thống lưu trữ phục vụ retrieval, giúp mỗi thành phần có vai trò rõ ràng nhưng vẫn phối hợp trong một luồng xử lý thống nhất.

Người dùng tương tác với frontend được triển khai trên **AWS Amplify**. Request được gửi qua **Amazon API Gateway** đến FastAPI backend chạy trên **Amazon EC2**. Backend thực hiện lexical retrieval bằng BM25 artifact lưu trên **Amazon S3**, đồng thời thực hiện dense retrieval thông qua **Amazon S3 Vectors**, sau đó xử lý các bằng chứng thu được trước khi gửi context cuối cùng đến Groq LLM API.

## Sơ đồ kiến trúc

![Kiến trúc AWS CloudHop RAG](/images/5-Workshop/5.3-Architecture/architecture.png)

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