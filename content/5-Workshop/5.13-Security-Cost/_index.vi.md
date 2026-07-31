---
title: "Bảo mật và cân nhắc chi phí"
date: 2026-07-31
weight: 13
chapter: false
pre: " <b> 5.13. </b> "
---

CloudHop RAG được triển khai trong phạm vi một dự án thực hành, không phải một dịch vụ production ở quy mô lớn. Dù vậy, các vấn đề về bảo mật và chi phí vẫn được cân nhắc ngay từ khi thiết kế kiến trúc.

Về bảo mật, hệ thống ưu tiên tránh sử dụng AWS credential dài hạn, giữ dữ liệu lưu trữ ở trạng thái private, giới hạn quyền của backend và tận dụng các cơ chế truy cập do AWS quản lý. Về chi phí, phần tốn chi phí định kỳ nhiều nhất là EC2 backend, trong khi phần lớn các dịch vụ còn lại được tính phí chủ yếu theo dung lượng lưu trữ hoặc số lượng request.

---

## 1. Các cân nhắc về bảo mật

### Truy cập AWS bằng IAM

EC2 backend sử dụng IAM role `rag-ec2-runtime-role` thay vì lưu AWS access key trực tiếp trong ứng dụng.

Role này cung cấp các quyền cần thiết để:

- đọc retrieval artifact từ Amazon S3;
- truy vấn Amazon S3 Vectors;
- truy cập instance thông qua AWS Systems Manager Session Manager.

Trong quá trình vận hành thông thường, backend không cần quyền chỉnh sửa các retrieval artifact đã lưu.

### Giữ dữ liệu lưu trữ ở trạng thái private

Dữ liệu của dự án trên Amazon S3 không được mở public. Backend truy cập bucket thông qua IAM permission thay vì sử dụng public URL.

Amazon S3 Vectors cũng được truy cập thông qua IAM role của EC2 instance. Frontend không kết nối trực tiếp với bất kỳ dịch vụ lưu trữ nào trong hai dịch vụ này.

### Quyền truy cập quản trị

Việc quản trị EC2 hằng ngày được thực hiện bằng **AWS Systems Manager Session Manager** thay vì SSH. Cách này giúp không phải quản lý SSH key trong quy trình của dự án và cũng không cần mở port `22`.

### HTTPS và CORS

Ứng dụng hướng tới người dùng được phục vụ qua HTTPS bằng AWS Amplify, còn Amazon API Gateway cung cấp HTTPS endpoint cho các request đến backend.

CORS được cấu hình để chỉ cho phép frontend origin dự kiến, thay vì mở quyền truy cập từ mọi origin trên trình duyệt.

### API key của dịch vụ bên ngoài

Groq API key được lưu trong file `.env.prod` trên EC2 và file này được loại khỏi Git.

Cách làm này phù hợp với phạm vi hiện tại của dự án. Tuy nhiên, nếu triển khai ở môi trường production, API key nên được chuyển sang một dịch vụ quản lý secret chuyên dụng thay vì lưu trực tiếp trong file cấu hình.

---

## 2. Những giới hạn hiện tại

Kiến trúc được giữ ở mức đơn giản để phù hợp với phạm vi dự án, vì vậy vẫn còn một số hạn chế.

| Hạn chế | Thiết kế hiện tại | Hướng cải thiện khi triển khai production |
| --- | --- | --- |
| Backend còn lộ ra mạng công khai | EC2 port `8000` có thể truy cập để API Gateway kết nối | Đưa backend vào private network và sử dụng private integration |
| Chưa có xác thực người dùng | Không có lớp xác thực end-user | Bổ sung authentication hoặc authorization |
| Khả năng xử lý của backend | Chỉ có một EC2 instance | Sử dụng nhiều instance hoặc một lớp compute có khả năng scale |
| Tính sẵn sàng | Chỉ có một backend instance | Triển khai nhiều instance trên nhiều Availability Zone |
| Groq API key | Lưu trong `.env.prod` | Chuyển secret sang dịch vụ quản lý secret |
| Monitoring | Chủ yếu kiểm tra qua `systemd` và `journalctl` | Bổ sung monitoring và alerting tập trung |

Những hạn chế này vẫn chấp nhận được với một hệ thống demo trong phạm vi thực tập, nhưng cần được xử lý nếu muốn đưa hệ thống vào sử dụng như một dịch vụ production thực sự.

---

## 3. Các nguồn chi phí chính

Mỗi dịch vụ AWS có cách tính phí khác nhau.

| Dịch vụ | Nguồn chi phí chính |
| --- | --- |
| **Amazon EC2** | Thời gian instance chạy |
| **EBS** | Dung lượng lưu trữ gắn với EC2 instance |
| **Public IPv4 / Elastic IP** | Mức sử dụng public IPv4 |
| **Amazon S3** | Dung lượng artifact và số lượng request |
| **Amazon S3 Vectors** | Dung lượng vector và các retrieval request |
| **Amazon API Gateway** | Số lượng API request |
| **AWS Amplify Hosting** | Build, hosting và data transfer |

**Groq API** bên ngoài AWS cũng có chi phí theo mức sử dụng, nhưng khoản này không nằm trong hóa đơn AWS.

Trong dự án này, EC2 là tài nguyên cần được quản lý chủ động nhất vì chi phí compute vẫn tiếp tục phát sinh trong thời gian instance đang chạy, kể cả khi không có người sử dụng ứng dụng.

---

## 4. Kiểm soát chi phí

Một số cách đơn giản giúp giữ chi phí triển khai ở mức hợp lý:

1. **Stop EC2 instance khi không sử dụng ứng dụng.**
2. **Xóa các phiên bản artifact và vector index cũ** khi không còn cần thiết.
3. **Giữ các tài nguyên trong cùng một AWS Region** để tránh phát sinh data transfer giữa các Region.
4. **Không triển khai thêm hạ tầng không cần thiết** đối với một môi trường demo nhỏ.
5. **Xóa toàn bộ tài nguyên của dự án sau khi hoàn thành workshop.**

Dự án cũng tách phần tạo embedding cho toàn bộ corpus ra khỏi backend online. BGE-M3 embedding chỉ được tạo một lần ở giai đoạn offline, sau đó được tái sử dụng trong ứng dụng đã triển khai. Cách này giúp tránh đưa một bước tính toán nặng vào môi trường EC2 đang phục vụ request.

---

## 5. Tổng kết

Kiến trúc hiện tại sử dụng IAM role thay cho AWS credential được nhúng trực tiếp trong ứng dụng, giữ các retrieval store ở trạng thái private, dùng Session Manager để quản trị EC2 và cung cấp giao diện người dùng thông qua các dịch vụ HTTPS được AWS quản lý.

Đồng thời, hệ thống vẫn được giữ ở quy mô gọn nhẹ để phù hợp với mục tiêu của dự án. Những điểm còn hạn chế rõ nhất là EC2 backend vẫn có endpoint công khai, chưa có xác thực người dùng, chỉ sử dụng một backend instance và Groq API key vẫn được lưu trong file cấu hình.

Về chi phí, EC2 là tài nguyên định kỳ cần chú ý nhiều nhất. Với phạm vi dự án này, hai biện pháp hiệu quả nhất là stop compute khi không sử dụng và xóa những tài nguyên triển khai không còn cần thiết.

Phần 5.14 sẽ thực hiện dọn dẹp các tài nguyên AWS đã tạo trong workshop để tránh phát sinh chi phí sau khi dự án kết thúc.
