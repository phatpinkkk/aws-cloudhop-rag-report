---
title: "Dọn dẹp tài nguyên"
date: 2026-07-31
weight: 14
chapter: false
pre: " <b> 5.14. </b> "
---

Sau khi hoàn thành workshop, các tài nguyên AWS đã tạo cho CloudHop RAG nên được xóa khi không còn cần sử dụng. Việc dọn dẹp giúp tránh để các tài nguyên compute, storage, networking và application tiếp tục tồn tại trong tài khoản AWS dù dự án đã kết thúc.

Nếu hệ thống vẫn cần được giữ lại để demo trong thời gian ngắn, có thể chỉ **stop EC2 instance** thay vì xóa toàn bộ môi trường triển khai.

---

## 1. Thứ tự dọn dẹp

Nên xóa tài nguyên theo một thứ tự hợp lý để tránh bỏ sót các thành phần phụ thuộc lẫn nhau.

| Thứ tự | Tài nguyên | Thao tác |
| ---: | --- | --- |
| 1 | **AWS Amplify app** | Xóa frontend đã triển khai |
| 2 | **Amazon API Gateway API** | Xóa HTTP API và các route liên quan |
| 3 | **Amazon EC2 instance** | Terminate backend instance |
| 4 | **Elastic IP** | Release địa chỉ sau khi EC2 instance đã được terminate |
| 5 | **Amazon S3 Vectors index** | Xóa vector index đã triển khai |
| 6 | **Amazon S3 Vectors bucket** | Xóa vector bucket sau khi toàn bộ index bên trong đã được xóa |
| 7 | **Amazon S3 artifact bucket** | Xóa toàn bộ object trong bucket, sau đó xóa bucket |
| 8 | **IAM role** | Xóa `rag-ec2-runtime-role` sau khi EC2 instance không còn sử dụng role này |

Elastic IP cần được release riêng sau khi EC2 instance bị terminate. Nếu không kiểm tra bước này, một public IPv4 address không còn sử dụng vẫn có thể tiếp tục được giữ trong tài khoản.

---

## 2. Xóa các tài nguyên lưu trữ

Với Amazon S3 Vectors, cần xóa **vector index trước**, sau đó mới có thể xóa vector bucket chứa index đó.

Đối với S3 artifact bucket thông thường, trước tiên cần xóa toàn bộ object của dự án rồi mới xóa bucket.

Nếu bucket đã bật versioning, cần lưu ý rằng các phiên bản object cũ và delete marker cũng có thể vẫn còn tồn tại. Những dữ liệu này phải được xóa hết trước khi AWS cho phép xóa bucket.

Các identifier của lần triển khai hiện tại gồm:

| Tài nguyên | Giá trị của dự án |
| --- | --- |
| Artifact bucket | `aws-rag-bucket-vanh1234` |
| Vector bucket | `rag-vectors-vanh1234` |
| Vector index | `hotpotqa-val500-bge-m3-v002` |
| EC2 role | `rag-ec2-runtime-role` |

Nếu thực hiện lại workshop với tên tài nguyên khác, cần xóa đúng những tài nguyên đã được tạo trong lần triển khai đó.

---

## 3. Tạm dừng hệ thống thay vì xóa hoàn toàn

Nếu CloudHop RAG vẫn cần được sử dụng để demo, có thể tạm dừng deployment thay vì xóa toàn bộ.

Thao tác quan trọng nhất là **stop EC2 instance**, vì backend không cần tiếp tục chạy khi không có người sử dụng ứng dụng.

Các tài nguyên lưu trữ và cấu hình còn lại có thể được giữ tạm thời. Nhờ đó, khi cần sử dụng lại hệ thống, không phải xây dựng lại toàn bộ retrieval artifact từ đầu.

Khi khởi động EC2 instance trở lại:

1. Chờ các instance status check hoàn tất.
2. Xác nhận `aws-rag-api` đang chạy.
3. Gọi `/warmup`.
4. Kiểm tra `/health`.
5. Thử lại ứng dụng thông qua API Gateway hoặc Amplify frontend.

Nếu chắc chắn dự án sẽ không còn được sử dụng, xóa toàn bộ tài nguyên vẫn là lựa chọn phù hợp hơn.

---

## 4. Kiểm tra sau khi dọn dẹp

Sau khi xóa các tài nguyên chính, nên kiểm tra lại AWS Console để chắc chắn không còn tài nguyên của CloudHop RAG bị bỏ sót.

| Khu vực trên AWS | Kết quả mong đợi |
| --- | --- |
| EC2 Instances | Không còn CloudHop RAG instance ở trạng thái running hoặc stopped |
| Elastic IP addresses | Không còn Elastic IP của dự án được giữ lại |
| EBS Volumes | Không còn volume của dự án ở trạng thái không sử dụng |
| Amazon S3 | Artifact bucket của dự án đã được xóa |
| Amazon S3 Vectors | Vector index và vector bucket của dự án đã được xóa |
| API Gateway | CloudHop RAG API đã được xóa |
| AWS Amplify | Frontend application đã được xóa |
| IAM Roles | `rag-ec2-runtime-role` đã được xóa |

Nên kiểm tra riêng **Elastic IP** và **EBS volume**, vì đây là những tài nguyên có thể vẫn còn tồn tại ngay cả sau khi EC2 instance đã bị xóa.

---

## 5. Những gì nên giữ lại

Xóa môi trường triển khai trên AWS không có nghĩa là xóa toàn bộ dự án.

Những tài liệu và artifact sau nên được giữ lại:

- source repository của dự án;
- notebook dùng để xây dựng artifact offline;
- một bản local của bộ artifact v002 đã được kiểm tra nếu có khả năng cần sử dụng lại;
- báo cáo thực tập và tài liệu workshop.

Với các thành phần này, hệ thống có thể được triển khai lại sau này bằng cách thực hiện lại các bước từ Phần 5.4 đến 5.9.

---

Sau khi toàn bộ tài nguyên đã được xóa hoặc chủ động đưa về trạng thái tạm dừng, workshop CloudHop RAG được xem là hoàn tất và tài khoản AWS không còn phải duy trì một môi trường triển khai đang hoạt động.
