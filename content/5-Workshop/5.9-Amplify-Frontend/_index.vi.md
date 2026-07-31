---
title: "Triển khai Frontend với AWS Amplify"
date: 2026-07-31
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

Thành phần trực tiếp tương tác với người dùng của CloudHop RAG là một ứng dụng web **React/Vite** được triển khai bằng **AWS Amplify**. Giao diện cho phép người dùng nhập câu hỏi, nhận câu trả lời được sinh ra và xem các nguồn hỗ trợ do RAG backend trả về.

Frontend giao tiếp với backend thông qua HTTPS endpoint được tạo trên Amazon API Gateway. Cách triển khai này tách giao diện web khỏi EC2 service nhưng vẫn cung cấp một điểm truy cập thống nhất cho người dùng.

### Triển khai Frontend Amplify

Triển khai giao diện React lên AWS Amplify.

#### 1. Console Steps: Cấu hình Build Settings
Trong **Amplify Console**, kết nối GitHub repository và cấu hình:
- **Build settings**:
  ```yaml
  version: 1
  frontend:
    phases:
      preBuild:
        commands:
          - npm install
      build:
        commands:
          - npm run build
    artifacts:
      baseDirectory: dist
      files:
        - '**/*'
    cache:
      paths:
        - node_modules/**/*
  ```

#### 2. Console Steps: Biến môi trường
Đây là bước quan trọng nhất để frontend kết nối đúng API:
1. Vào **App settings** -> **Environment variables**.
2. Thêm biến:
   - Key: `VITE_API_BASE_URL`
   - Value: `https://<api-gateway-url>` (của bước 5.6)

> **Lưu ý**: Tuyệt đối không dùng URL HTTP của EC2 vì sẽ gây lỗi **Mixed Content** (trình duyệt chặn gọi HTTP từ HTTPS).

#### 3. Console Steps: Triển khai
Nhấn **Redeploy this version** để Amplify tự động build và cập nhật URL frontend.