---
title: "Triển khai Frontend với AWS Amplify"
date: 2026-07-31
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

Thành phần cuối cùng mà người dùng trực tiếp tương tác trong CloudHop RAG là một ứng dụng web **React/Vite** được triển khai bằng **AWS Amplify Hosting**. Giao diện cho phép người dùng nhập câu hỏi, nhận câu trả lời do hệ thống sinh ra và xem các nguồn bằng chứng mà RAG backend trả về.

Frontend kết nối với backend thông qua **HTTPS endpoint của Amazon API Gateway** đã tạo ở Phần 5.8. Sau bước này, toàn bộ luồng xử lý từ trình duyệt đến backend đã được kết nối hoàn chỉnh.

---

## 1. Kết nối Repository Frontend

Trong AWS Management Console, mở:

**AWS Amplify → Create new app → Deploy from Git**

Kết nối repository của dự án và chọn branch dùng để triển khai.

Vì repository chứa cả backend và frontend, Amplify cần được cấu hình để build từ đúng thư mục frontend.

| Thiết lập | Giá trị |
| --- | --- |
| Ứng dụng | React/Vite frontend |
| App root | `frontend` |
| Build command | `npm run build` |
| Output directory | `dist` |

Amplify sẽ cài đặt các dependency của frontend, chạy Vite production build và publish các static file được tạo ra.

---

## 2. Cấu hình API Endpoint

Frontend cần biết địa chỉ API công khai để gửi câu hỏi của người dùng đến backend.

Trong **Amplify → App settings → Environment variables**, thêm biến:

| Biến | Giá trị |
| --- | --- |
| `VITE_API_BASE_URL` | `https://<api-id>.execute-api.ap-southeast-1.amazonaws.com` |

Giá trị này phải là **HTTPS URL của API Gateway**, không phải địa chỉ HTTP trực tiếp của EC2.

Khi đó, đường đi của request là:

**Trình duyệt → AWS Amplify → Amazon API Gateway → Amazon EC2**

Sử dụng API Gateway giúp frontend không cần truy cập trực tiếp vào EC2 backend, đồng thời giữ toàn bộ đường truyền phía trình duyệt trên HTTPS.

Vì Vite đưa environment variable vào ứng dụng ngay trong quá trình build, frontend cần được deploy lại mỗi khi `VITE_API_BASE_URL` được thêm mới hoặc thay đổi.

---

## 3. Triển khai bằng AWS Amplify

Sau khi repository và environment variable đã được cấu hình, bắt đầu deployment trên Amplify.

Amplify sẽ tự động build frontend và publish ứng dụng lên một địa chỉ HTTPS có tên miền `amplifyapp.com`. Sau đó, mỗi lần có thay đổi được push lên branch đã kết nối, Amplify cũng có thể tự động kích hoạt một deployment mới.

Khi quá trình triển khai hoàn tất, mở URL do Amplify cung cấp trên trình duyệt để kiểm tra ứng dụng.

---

## 4. Kiểm tra toàn bộ ứng dụng

Thử gửi một câu hỏi multi-hop, ví dụ:

*"Were Scott Derrickson and Ed Wood of the same nationality?"*

Nếu hệ thống hoạt động đúng, request sẽ đi qua toàn bộ kiến trúc đã triển khai:

**Trình duyệt → Amplify → API Gateway → EC2 FastAPI → Amazon S3 + Amazon S3 Vectors → Groq → Câu trả lời**

Giao diện cần hiển thị được câu trả lời do hệ thống sinh ra cùng với các nguồn bằng chứng mà backend trả về.

![Ứng dụng CloudHop RAG sau khi triển khai](/images/5-Workshop/5.9-Amplify-Frontend/deployed-app.png)

Đây là bước kiểm tra trực quan đầu tiên cho thấy các thành phần chính của hệ thống AWS đã kết nối thành công với nhau và hoạt động theo đúng luồng end-to-end.

---

## 5. Kết quả

Sau khi hoàn thành phần này, CloudHop RAG đã có một giao diện web HTTPS hoàn chỉnh để người dùng truy cập.

Ở thời điểm này:

- AWS Amplify đang host React/Vite frontend;
- frontend gửi request đến Amazon API Gateway;
- API Gateway chuyển tiếp request đến FastAPI backend trên EC2;
- backend truy xuất bằng chứng từ Amazon S3 và Amazon S3 Vectors;
- Groq sinh câu trả lời cuối cùng;
- frontend hiển thị câu trả lời cùng các nguồn hỗ trợ.

Như vậy, các bước triển khai chính đã hoàn tất. Phần 5.10 sẽ kiểm tra toàn bộ hệ thống theo cách có hệ thống hơn, còn Phần 5.11 sẽ tập trung đánh giá chất lượng retrieval, chất lượng câu trả lời và latency.
