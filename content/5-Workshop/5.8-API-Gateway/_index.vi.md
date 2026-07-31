---
title: "Amazon API Gateway"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

FastAPI backend trên EC2 hoạt động qua HTTP, trong khi frontend được AWS Amplify phân phối qua HTTPS. Trình duyệt không thể gọi trực tiếp HTTP backend từ một trang HTTPS, vì vậy **Amazon API Gateway** được sử dụng làm điểm truy cập HTTPS công khai cho ứng dụng.

API Gateway nhận request từ trình duyệt và chuyển tiếp đến FastAPI service trên EC2. Dịch vụ này đồng thời cung cấp cấu hình CORS cần thiết để frontend trên Amplify có thể gọi backend thành công.

### API Gateway & CORS

API Gateway đóng vai trò là lớp HTTPS an toàn, kết nối frontend (Amplify) với backend (EC2).

#### 1. Console Steps: Tạo API Gateway
1. Truy cập **API Gateway Console** -> **Create API** -> **HTTP API**.
2. **Add integration**: Chọn **HTTP** và nhập URL của EC2 backend (ví dụ: `http://54.251.81.140:8000`).
3. Định nghĩa các routes (đường dẫn):
   - `GET /health`
   - `POST /query`
   - `POST /warmup`
4. Deploy API và ghi lại **Invoke URL** (ví dụ: `https://b6asncvgs6.execute-api.ap-southeast-1.amazonaws.com`).

#### 2. Console Steps: Cấu hình CORS
Để frontend gọi được API, bạn phải cấu hình CORS:
1. Trong API Gateway Console, chọn **CORS** tab.
2. **Allow-Origin**: `https://<your-amplify-app-url>`
3. **Allow-Methods**: `GET`, `POST`, `OPTIONS`
4. **Allow-Headers**: `content-type`
5. Nhấn **Save** và **Deploy** lại API stage.

#### 3. CLI Steps: Kiểm tra API Gateway
Test qua PowerShell:

```powershell
# Test health
curl.exe "https://<api-gateway-url>/health"

# Test query (sử dụng file query.json)
'{"question":"Were Scott Derrickson and Ed Wood of the same nationality?"}' | Set-Content -Encoding utf8 query.json

curl.exe -s -X POST "https://<api-gateway-url>/query" `
  -H "Content-Type: application/json" `
  --data-binary "@query.json"