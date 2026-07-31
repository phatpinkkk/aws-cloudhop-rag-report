---
title: "Nhật ký tuần 2"
date: 2026-06-15
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2

* Hiểu cách Amazon S3 cung cấp khả năng lưu trữ object lâu dài trên cloud.
* Nắm được cách AWS quản lý danh tính và quyền truy cập thông qua IAM.
* Tìm hiểu những khái niệm networking cơ bản của Amazon VPC.
* Thực hành kiểm soát quyền truy cập bằng role, policy và security group.
* Hiểu cách compute, storage, identity và networking phối hợp với nhau trong một ứng dụng AWS.

### Công việc thực hiện trong tuần

| Ngày | Công việc | Tài liệu tham khảo |
| --- | --- | --- |
| 15/06/2026 | - Tìm hiểu các khái niệm của Amazon S3:<br>&emsp; + Bucket và object<br>&emsp; + Prefix<br>&emsp; + Storage classes<br>&emsp; + Versioning<br>- **Thực hành:** tạo bucket và thử upload/download object. | <https://cloudjourney.awsstudygroup.com/> |
| 16/06/2026 | - Tìm hiểu quyền truy cập và cơ chế bảo vệ dữ liệu trên S3.<br>- Phân biệt identity-based policy và resource-based policy.<br>- Tìm hiểu cấu hình public access và các nguyên tắc bảo mật S3 cơ bản. | |
| 17/06/2026 | - Tìm hiểu các thành phần chính của IAM:<br>&emsp; + Users<br>&emsp; + Groups<br>&emsp; + Roles<br>&emsp; + Policies<br>- Tìm hiểu nguyên tắc least privilege.<br>- Hiểu lý do workload trên AWS nên sử dụng IAM role thay vì hard-code thông tin xác thực. | |
| 18/06/2026 | - Tìm hiểu nền tảng Amazon VPC:<br>&emsp; + VPC và subnet<br>&emsp; + Route table<br>&emsp; + Internet Gateway<br>&emsp; + Security group<br>&emsp; + Public và private networking | |
| 19/06/2026 | - **Thực hành:** kết hợp EC2, IAM và S3 trong một workload đơn giản.<br>- Cấu hình quyền để compute resource có thể truy cập storage resource phù hợp.<br>- Ôn lại cách network và identity control phối hợp để bảo vệ workload trên AWS. | |

### Kết quả đạt được

* Hiểu những khái niệm quan trọng của Amazon S3 và thực hành lưu trữ, truy xuất object.
* Nắm được cách policy và quyền truy cập kiểm soát dữ liệu trên S3.
* Hiểu IAM user, role, policy và nguyên tắc least privilege.
* Nắm được kiến thức cơ bản về VPC, subnet, routing và security group.
* Hiểu cách EC2 workload truy cập các dịch vụ AWS an toàn thông qua IAM permission.
* Xây dựng được nền tảng về storage, identity và networking sau này được áp dụng trực tiếp trong kiến trúc CloudHop RAG.