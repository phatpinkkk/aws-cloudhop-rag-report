---
title: "Bảo mật và cân nhắc chi phí"
date: 2026-07-31
weight: 13
chapter: false
pre: " <b> 5.13. </b> "
---

CloudHop RAG được triển khai như một môi trường dự án thực tế, trong đó các yếu tố bảo mật và chi phí vẫn được xem xét trong quá trình thiết kế. Quyền truy cập AWS của EC2 backend được cấp thông qua IAM role thay vì đưa AWS credential trực tiếp vào ứng dụng, đồng thời Systems Manager Session Manager được sử dụng để quản lý instance.

API Gateway cung cấp lớp API HTTPS cho frontend trên Amplify, còn CORS giới hạn request từ trình duyệt về đúng frontend origin đã cấu hình. Về chi phí, thành phần duy trì thường xuyên nhất là EC2 instance, cùng với chi phí lưu trữ, vector search, API request và frontend hosting.