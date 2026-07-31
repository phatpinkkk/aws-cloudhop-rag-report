---
title: "Dọn dẹp"
date: 2024-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

### Dọn dẹp

Dọn dẹp tài nguyên trên **AWS Console** để tránh phát sinh chi phí.

#### Danh sách cần xoá (trên Console):
1. **EC2 Instance**: Vào **EC2 Console** -> **Terminate instance**.
2. **S3 Buckets**: Vào **S3 Console** -> Xoá `aws-rag-bucket-vanh1234` và `rag-vectors-vanh1234`.
3. **API Gateway**: Vào **API Gateway Console** -> Xoá API.
4. **Amplify App**: Vào **Amplify Console** -> Xoá App.
5. **IAM Roles**: Vào **IAM Console** -> Xoá Role `rag-ec2-runtime-role`.

**Quy tắc**: "Clean up as you go" - xoá ngay khi kết thúc demo để tránh billing.