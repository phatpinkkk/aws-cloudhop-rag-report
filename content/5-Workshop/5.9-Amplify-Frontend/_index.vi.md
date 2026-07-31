---
title: "Triển khai Frontend Amplify"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

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