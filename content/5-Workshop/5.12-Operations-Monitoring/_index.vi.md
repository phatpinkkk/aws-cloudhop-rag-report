---
title: "Hướng dẫn vận hành"
date: 2024-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

### Hướng dẫn vận hành

Hướng dẫn vận hành hệ thống hàng ngày.

#### 1. Console Steps: Khởi động hệ thống
Sau khi dừng EC2, khi cần dùng lại:
1. **Start EC2**: Bật instance trong **EC2 Console**.
2. **SSM Session**: Kết nối qua **Session Manager** (không cần SSH key).

#### 2. CLI Steps: Start Backend
Sau khi đã vào terminal của EC2:
```bash
sudo systemctl restart aws-rag-api
# Warmup (cần thiết để load model)
curl -X POST http://127.0.0.1:8000/warmup
```

#### 3. CLI Steps: Kiểm tra
```bash
curl http://127.0.0.1:8000/health
```

#### 4. Xử lý lỗi thường gặp

| Lỗi | Nguyên nhân | Cách khắc phục |
| :--- | :--- | :--- |
| **Mixed Content** | Frontend HTTPS gọi backend HTTP | Kiểm tra `VITE_API_BASE_URL` trong Amplify Console |
| **CORS Error** | API Gateway chặn request | Kiểm tra lại cấu hình CORS tab trong API Gateway Console |
| **503 Service** | Backend chết | `sudo systemctl status aws-rag-api` để kiểm tra log |
| **SSH Timeout** | Security Group chặn | Dùng Session Manager trong Console thay vì SSH |