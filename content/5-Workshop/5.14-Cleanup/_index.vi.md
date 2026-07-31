---
title: "Dọn dẹp tài nguyên"
date: 2026-07-31
weight: 14
chapter: false
pre: " <b> 5.14. </b> "
---

Sau khi hoàn thành workshop, các tài nguyên AWS được tạo cho CloudHop RAG nên được xóa nếu không còn sử dụng. Việc dọn dẹp giúp tránh duy trì các tài nguyên compute, lưu trữ, network và ứng dụng không còn cần thiết trong tài khoản AWS.

Các tài nguyên nên được xóa theo thứ tự phù hợp để tránh để lại những thành phần phụ thuộc không còn được sử dụng.

### Dọn dẹp

Dọn dẹp tài nguyên trên **AWS Console** để tránh phát sinh chi phí.

#### Danh sách cần xoá (trên Console):
1. **EC2 Instance**: Vào **EC2 Console** -> **Terminate instance**.
2. **S3 Buckets**: Vào **S3 Console** -> Xoá `aws-rag-bucket-vanh1234` và `rag-vectors-vanh1234`.
3. **API Gateway**: Vào **API Gateway Console** -> Xoá API.
4. **Amplify App**: Vào **Amplify Console** -> Xoá App.
5. **IAM Roles**: Vào **IAM Console** -> Xoá Role `rag-ec2-runtime-role`.

**Quy tắc**: "Clean up as you go" - xoá ngay khi kết thúc demo để tránh billing.