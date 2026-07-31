---
title: "Amazon API Gateway"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

FastAPI backend ở Phần 5.7 hiện đã có thể truy cập qua HTTP trên EC2 instance. Tuy nhiên, frontend lại được AWS Amplify phục vụ qua HTTPS. Vì vậy, **Amazon API Gateway** được đặt ở giữa để làm điểm truy cập HTTPS công khai cho trình duyệt trước khi request được chuyển tiếp đến backend.

API Gateway đảm nhiệm việc chuyển request đến FastAPI service, đồng thời cung cấp cấu hình CORS cần thiết để frontend có thể gọi API từ trình duyệt.

---

## 1. Tạo HTTP API

Trong AWS Management Console, mở:

**Amazon API Gateway → Create API → HTTP API**

Tạo một HTTP API cho CloudHop RAG backend và sử dụng Elastic IP của EC2 instance ở Phần 5.7 làm integration target.

Dự án sử dụng ba endpoint chính:

| Route | Backend endpoint | Mục đích |
| --- | --- | --- |
| `GET /health` | `http://<elastic-ip>:8000/health` | Kiểm tra backend có đang hoạt động hay không |
| `POST /warmup` | `http://<elastic-ip>:8000/warmup` | Khởi tạo retrieval pipeline |
| `POST /query` | `http://<elastic-ip>:8000/query` | Gửi câu hỏi và nhận câu trả lời |

Việc sử dụng Elastic IP giúp integration target giữ nguyên ngay cả khi EC2 instance được stop rồi start lại.

---

## 2. Cấu hình Route

Tạo ba route ở trên và kết nối từng route với FastAPI endpoint tương ứng.

Route trên API Gateway và đường dẫn phía backend nên được giữ giống nhau. Ví dụ:

**`POST /query` → `http://<elastic-ip>:8000/query`**

Cách tổ chức này giúp việc kiểm tra lỗi đơn giản hơn. Nếu gọi trực tiếp EC2 endpoint vẫn thành công nhưng request qua API Gateway thất bại, có thể khoanh vùng vấn đề ở phần cấu hình gateway thay vì phải kiểm tra lại toàn bộ RAG backend.

---

## 3. Cấu hình CORS

Frontend trên Amplify và API Gateway nằm ở hai origin khác nhau, vì vậy trình duyệt cần **Cross-Origin Resource Sharing (CORS)** để cho phép frontend gửi request đến API.

Với HTTP API trên API Gateway, cấu hình Amplify origin đang được sử dụng cùng các method cần thiết cho ứng dụng.

| Thiết lập | Giá trị |
| --- | --- |
| Allowed origin | `https://<your-amplify-app>.amplifyapp.com` |
| Allowed methods | `GET`, `POST`, `OPTIONS` |
| Allowed headers | `content-type` |

FastAPI backend cũng cho phép frontend origin tương ứng thông qua runtime configuration.

Thay vì dùng wildcard, dự án chỉ cho phép đúng Amplify origin cần thiết. Cách này giúp giới hạn browser access vào frontend dự kiến của hệ thống.

---

## 4. Sử dụng HTTPS Invoke URL

Sau khi API được tạo, API Gateway cung cấp một HTTPS Invoke URL có dạng:

`https://<api-id>.execute-api.ap-southeast-1.amazonaws.com`

Địa chỉ này sẽ trở thành API base URL mà frontend sử dụng ở Phần 5.9.

Từ thời điểm này, đường đi của request trên trình duyệt là:

**AWS Amplify → HTTPS → Amazon API Gateway → HTTP → Amazon EC2**

Frontend không cần gọi trực tiếp địa chỉ EC2.

---

## 5. Kiểm tra API

Nên kiểm tra `/health` trước vì endpoint này không cần chạy toàn bộ RAG pipeline:

`curl.exe "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/health"`

Nếu nhận được response thành công, có thể xác nhận API Gateway đã kết nối được với EC2 backend.

Sau đó, kiểm tra route `/query` bằng cùng câu hỏi HotpotQA đã dùng ở các phần trước:

```powershell
curl.exe -X POST "https://<api-id>.execute-api.ap-southeast-1.amazonaws.com/query" -H "Content-Type: application/json" --data "{\"question\":\"Were Scott Derrickson and Ed Wood of the same nationality?\"}"
```

Nếu request thành công, response sẽ gồm câu trả lời được sinh ra và các nguồn bằng chứng hỗ trợ.

Việc kiểm tra backend trực tiếp ở Phần 5.7 trước, rồi kiểm tra lại thông qua API Gateway ở phần này, giúp xác định lỗi theo từng lớp thay vì phải debug toàn bộ hệ thống cùng lúc.

---

## 6. Kết quả

Sau bước này, Amazon API Gateway đã trở thành HTTPS endpoint mà frontend CloudHop RAG sử dụng.

Ở thời điểm hiện tại:

- FastAPI backend đang chạy trên Amazon EC2;
- API Gateway chuyển tiếp các request `/health`, `/warmup` và `/query` đến backend;
- CORS cho phép request từ Amplify frontend;
- query endpoint đã có thể được gọi qua HTTPS.

Phần 5.9 sẽ triển khai React frontend bằng AWS Amplify và cấu hình frontend sử dụng Invoke URL của API Gateway.
