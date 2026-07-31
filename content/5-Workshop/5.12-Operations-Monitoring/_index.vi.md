---
title: "Vận hành và xử lý sự cố"
date: 2026-07-31
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

Sau khi triển khai hoàn tất, phần lớn công việc vận hành hằng ngày của CloudHop RAG tập trung vào backend trên EC2. Ứng dụng chạy dưới dạng `systemd` service có tên `aws-rag-api`, còn **AWS Systems Manager Session Manager** được dùng để truy cập instance khi cần bảo trì hoặc kiểm tra sự cố.

Backend cung cấp hai endpoint `/health` và `/warmup` để kiểm tra trạng thái ứng dụng và khởi tạo retrieval pipeline. Khi service không khởi động được hoặc request trả về kết quả bất thường, có thể kiểm tra log trực tiếp qua systemd journal để xác định nguyên nhân.

---

## 1. Khởi động hệ thống

Nếu EC2 instance đang ở trạng thái stopped, mở **EC2 Console**, khởi động lại instance và chờ đến khi các status check hoàn tất.

Sau đó, nếu cần thao tác trực tiếp trên máy chủ, kết nối vào instance bằng **AWS Systems Manager Session Manager**. Quy trình vận hành thông thường của dự án không sử dụng SSH.

---

## 2. Khởi động hoặc restart Backend

Sau khi kết nối vào EC2 instance, restart FastAPI service:

`sudo systemctl restart aws-rag-api`

Kiểm tra trạng thái service:

`sudo systemctl status aws-rag-api`

Trước khi gửi request thực tế từ người dùng, nên warm up retrieval pipeline:

`curl -X POST http://127.0.0.1:8000/warmup`

Bước warm-up khởi tạo trước các thành phần retrieval chính, nhờ đó request đầu tiên không phải đồng thời thực hiện toàn bộ công việc startup.

---

## 3. Kiểm tra Backend

Kiểm tra health endpoint:

`curl http://127.0.0.1:8000/health`

Nếu service không khởi động đúng hoặc một request bị lỗi, xem 100 dòng log gần nhất:

`sudo journalctl -u aws-rag-api -n 100 --no-pager`

Để theo dõi log theo thời gian thực:

`sudo journalctl -u aws-rag-api -f`

Các bước kiểm tra này giúp phân biệt lỗi nằm ở chính backend hay ở những lớp phía ngoài như API Gateway hoặc frontend.

---

## 4. Xử lý một số sự cố thường gặp

| Vấn đề | Nguyên nhân có thể xảy ra | Cách xử lý |
| --- | --- | --- |
| **Mixed Content** | Amplify frontend đang gọi trực tiếp địa chỉ HTTP của EC2 | Kiểm tra `VITE_API_BASE_URL` và bảo đảm biến này sử dụng HTTPS URL của API Gateway. Nếu vừa thay đổi giá trị, cần redeploy frontend |
| **CORS Error** | Origin của frontend chưa được API Gateway hoặc backend cho phép | Kiểm tra lại cấu hình CORS trên API Gateway và CORS setting của backend |
| **503 / Service Unavailable** | Backend không hoạt động, API Gateway không kết nối được đến EC2 hoặc request mất quá nhiều thời gian | Kiểm tra `aws-rag-api`, thử gọi trực tiếp EC2 endpoint và warm up backend trước khi thử lại |
| **Backend không khởi động** | Lỗi ứng dụng hoặc runtime configuration | Kiểm tra bằng `systemctl status`, sau đó xem log chi tiết bằng `journalctl` |
| **Request đầu tiên phản hồi chậm** | Các thành phần retrieval chưa được khởi tạo | Gọi `/warmup` trước khi gửi các query thông thường |
| **Không truy cập được EC2 bằng Session Manager** | Có vấn đề với trạng thái instance, IAM role hoặc kết nối Systems Manager | Xác nhận EC2 đang chạy và policy `AmazonSSMManagedInstanceCore` đã được gắn vào IAM role của instance |

---

## 5. Quy trình vận hành thông thường

Khi cần đưa hệ thống trở lại hoạt động, có thể thực hiện theo trình tự:

1. Khởi động EC2 instance.
2. Kết nối bằng Session Manager nếu cần bảo trì hoặc kiểm tra trực tiếp.
3. Xác nhận `aws-rag-api` đang chạy.
4. Gọi `/warmup`.
5. Kiểm tra `/health`.
6. Sử dụng frontend đã triển khai hoặc kiểm tra request thông qua API Gateway.

Với cách tổ chức này, công việc vận hành thường ngày chủ yếu tập trung vào EC2 backend, trong khi các thành phần còn lại như Amazon S3, Amazon S3 Vectors, API Gateway và Amplify tiếp tục hoạt động dưới dạng các dịch vụ được AWS quản lý.
