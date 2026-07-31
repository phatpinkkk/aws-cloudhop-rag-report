---
title: "Lưu trữ với Amazon S3"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Các artifact đã chuẩn bị ở Phần 5.4 cần được lưu trữ độc lập với EC2 instance đang chạy backend. Vì vậy, CloudHop RAG sử dụng **Amazon S3** để lưu các tài liệu đã xử lý, BM25 artifact và metadata của index mà online retrieval pipeline cần sử dụng.

Phần này hướng dẫn tạo S3 bucket cho dự án, tải toàn bộ cây artifact lên AWS và kiểm tra lại dữ liệu trước khi triển khai backend.

---

## 1. Vai trò của Amazon S3

Amazon S3 được dùng để lưu các artifact bền vững không phải dạng vector của hệ thống RAG.

| Dữ liệu lưu trên Amazon S3 | Mục đích |
| --- | --- |
| Parent document và child document | Phục vụ retrieval và xây dựng context |
| BM25 artifact | Lexical retrieval |
| Index manifest | Lưu phiên bản artifact và cấu hình |
| Corpus và evaluation files | Hỗ trợ tái lập và đánh giá về sau |

Dense embedding được xử lý riêng bằng **Amazon S3 Vectors** ở Phần 5.6.

Việc lưu các artifact này bên ngoài EC2 giúp dữ liệu retrieval không phụ thuộc vào vòng đời của instance. Backend có thể được restart hoặc thay thế mà không làm mất dữ liệu đã chuẩn bị.

---

## 2. Tạo Artifact Bucket

Dự án được triển khai tại **Asia Pacific (Singapore) Region (`ap-southeast-1`)**, vì vậy S3 bucket cũng được tạo trong cùng Region để giữ cấu hình nhất quán với các tài nguyên còn lại.

Trong AWS Management Console, mở **Amazon S3 → Create bucket** và cấu hình:

| Thiết lập | Giá trị |
| --- | --- |
| Bucket name | Một tên duy nhất trên toàn cầu |
| AWS Region | `ap-southeast-1` |
| Object Ownership | ACLs disabled |
| Block Public Access | Giữ toàn bộ public access ở trạng thái blocked |
| Default encryption | SSE-S3 |

Trong lần triển khai của dự án, bucket được sử dụng là:

`aws-rag-bucket-vanh1234`

Khi thực hiện lại workshop, cần chọn một bucket name khác và bảo đảm tên đó chưa được sử dụng trên AWS.

![Tạo artifact bucket](/images/5-Workshop/5.5-S3-Storage/create-bucket.png)

Bucket được giữ ở trạng thái private vì frontend không truy cập trực tiếp vào các file này. Backend sẽ được cấp quyền đọc thông qua IAM role ở bước triển khai EC2.

---

## 3. Cấu trúc Artifact trên S3

Các file được tạo ở Phần 5.4 sẽ được tải lên dưới prefix `rag/`.

Cấu trúc chính gồm:

**rag/**  
→ **corpora/** – dữ liệu nguồn và các file phục vụ evaluation  
→ **processed/** – parent document, child document và mapping  
→ **indexes/** – BM25 artifact, manifest và các file dùng để import vector

Với bộ artifact v002 cuối cùng, các identifier chính là:

| Hạng mục | Giá trị |
| --- | --- |
| Processed ID | `hotpotqa-val500-v002` |
| Index ID | `hotpotqa-val500-bge-m3-v002` |
| Artifact prefix | `rag` |

Việc giữ version trong tên artifact và index giúp có thể tải lên một phiên bản mới mà không ghi đè bộ dữ liệu cũ. Điều này cũng giúp việc chuyển đổi hoặc rollback giữa các phiên bản rõ ràng hơn.

---

## 4. Tải Artifact lên S3

Thư mục được tạo ở Phần 5.4 đã có cấu trúc tương ứng với layout mong muốn trên S3, vì vậy có thể tải toàn bộ thư mục lên bằng AWS CLI.

Lệnh upload điển hình:

```bash
aws s3 sync "s3_manual_upload/hotpotqa-val500-bge-m3-v002/rag" "s3://aws-rag-bucket-vanh1234/rag" --region ap-southeast-1
```

Nếu sử dụng bucket name khác, thay `aws-rag-bucket-vanh1234` bằng tên bucket của bạn.

![Tải cây artifact lên Amazon S3](/images/5-Workshop/5.5-S3-Storage/upload-output.png)

Sau khi hoàn tất, S3 cần chứa đầy đủ corpus, processed parent/child document, BM25 artifact, manifest và các S3 Vectors import file đã chuẩn bị ở bước trước.

---

## 5. Kiểm tra dữ liệu sau khi upload

Sau khi upload xong, nên kiểm tra lại trực tiếp trong S3 Console hoặc bằng AWS CLI.

```bash
aws s3 ls s3://aws-rag-bucket-vanh1234/rag/ --recursive --region ap-southeast-1
```

Tối thiểu, cần thấy các nhóm artifact sau:

- `rag/corpora/...`
- `rag/processed/hotpotqa-val500-v002/...`
- `rag/indexes/hotpotqa-val500-bge-m3-v002/bm25/...`
- `rag/indexes/hotpotqa-val500-bge-m3-v002/manifests/...`
- `rag/indexes/hotpotqa-val500-bge-m3-v002/s3vectors-import/...`

![Các RAG artifact sau khi được tải lên Amazon S3](/images/5-Workshop/5.5-S3-Storage/s3-console-tree.png)

Đối với runtime, các file quan trọng nhất là processed parent/child document, BM25 artifact và index manifest. Các vector-import file vẫn được lưu trên S3 để sử dụng khi nạp dữ liệu vào Amazon S3 Vectors ở phần tiếp theo.

---

## 6. Quyền truy cập của Backend

S3 bucket vẫn được giữ private. FastAPI backend không lưu AWS access key trực tiếp trong source code hay file cấu hình ứng dụng. Thay vào đó, EC2 instance sẽ nhận quyền thông qua IAM role được cấu hình ở Phần 5.7.

Backend cần quyền đọc đối với artifact prefix của dự án để có thể:

- tải index manifest;
- tải processed parent document và child document;
- tải BM25 index khi service khởi động.

Trong quá trình xử lý câu hỏi thông thường, backend chỉ đọc các artifact này và không cần chỉnh sửa chúng.

Chi tiết về IAM role và permission sẽ được trình bày cùng phần triển khai EC2 và phần bảo mật, thay vì lặp lại tại đây.

---

## 7. Kết quả

Sau khi hoàn thành phần này, toàn bộ RAG artifact không phải dạng vector đã được lưu trong một Amazon S3 bucket private tại Region `ap-southeast-1` và sẵn sàng để backend sử dụng.

Vai trò lưu trữ lúc này được tách rõ:

**Amazon S3** lưu tài liệu, BM25 artifact và metadata.  
**Amazon S3 Vectors** lưu và tìm kiếm dense vector BGE-M3.

Phần 5.6 sẽ tiếp tục tạo vector bucket và index, sau đó nạp các vector đã chuẩn bị từ Phần 5.4 vào Amazon S3 Vectors.
