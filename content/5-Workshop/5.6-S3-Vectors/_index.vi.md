---
title: "Amazon S3 Vectors"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

CloudHop RAG kết hợp lexical retrieval bằng BM25 với dense semantic retrieval. Phần dense retrieval được triển khai bằng **Amazon S3 Vectors**, nơi lưu các BGE-M3 embedding đã được tạo ở Phần 5.4 và cho phép backend thực hiện vector search khi hệ thống đang chạy.

Mỗi child chunk được biểu diễn bằng một **BGE-M3 vector 1.024 chiều**. Phần này hướng dẫn tạo vector bucket và vector index, nạp các vector đã chuẩn bị, sau đó kiểm tra lại semantic retrieval trước khi triển khai backend trên EC2.

---

## 1. Thiết kế lưu trữ vector

Trong dự án, Amazon S3 và Amazon S3 Vectors đảm nhiệm hai vai trò khác nhau.

| Dịch vụ | Dữ liệu lưu trữ | Vai trò |
| --- | --- | --- |
| **Amazon S3** | Document, BM25 artifact, manifest | Lưu trữ artifact lâu dài |
| **Amazon S3 Vectors** | BGE-M3 embedding | Thực hiện dense similarity retrieval |

Các tài nguyên vector được tạo trong cùng Region với toàn bộ hệ thống:

`ap-southeast-1`

Cấu hình vector cuối cùng của dự án là:

| Thiết lập | Giá trị |
| --- | --- |
| Vector bucket | `rag-vectors-vanh1234` |
| Index | `hotpotqa-val500-bge-m3-v002` |
| Embedding model | `BAAI/bge-m3` |
| Dimension | `1024` |
| Distance metric | `cosine` |
| Data type | `float32` |

Các giá trị này phải khớp với cấu hình ở bước build offline. Query embedding do backend tạo ra và các vector đã lưu phải nằm trong cùng một vector space thì similarity search mới hoạt động đúng.

---

## 2. Tạo Vector Bucket

Trong AWS Management Console, mở **Amazon S3 → Vector buckets** và tạo một vector bucket mới tại Region `ap-southeast-1`.

Dự án sử dụng:

`rag-vectors-vanh1234`

Khi thực hiện lại workshop, có thể chọn một vector bucket name khác nếu tên này đã được sử dụng.

![Tạo vector bucket](/images/5-Workshop/5.6-S3-Vectors/create-vector-bucket.png)

Vector bucket đóng vai trò là nơi chứa một hoặc nhiều vector index. Các thông số như embedding dimension và distance metric sẽ được cấu hình ở cấp index.

---

## 3. Tạo Vector Index

Bên trong vector bucket, tạo một index có cùng identifier với bộ artifact v002 cuối:

`hotpotqa-val500-bge-m3-v002`

Cấu hình:

- **Data type:** `float32`
- **Dimension:** `1024`
- **Distance metric:** `cosine`

![Tạo vector index](/images/5-Workshop/5.6-S3-Vectors/create-index.png)

Tên index được gắn version để luôn khớp với bộ artifact v002 tương ứng đang lưu trên Amazon S3.

Thông số dimension cũng phải đúng với BGE-M3. Nếu index được tạo với dimension khác, các vector đã chuẩn bị sẽ không thể được nạp vào đúng cách.

---

## 4. Nạp các Vector đã chuẩn bị

Ở Phần 5.4, dense vector đã được tạo và chia thành các batch phục vụ quá trình import. Ở bước này, các batch đó được nạp vào S3 Vectors index.

Dự án sử dụng script ingestion được tạo cùng bộ artifact offline:

```bash
python ingest_s3vectors.py --region ap-southeast-1
```

Bộ artifact cuối cùng có **8.279 child vector**, vì vậy số vector được nạp thành công cần khớp với số lượng đã xác nhận ở Phần 5.4.

![Nạp vector vào Amazon S3 Vectors](/images/5-Workshop/5.6-S3-Vectors/ingest-output.png)

Mỗi vector sử dụng `child_id` tương ứng làm key. Khi vector search trả về kết quả, backend có thể dùng key này để tìm lại child document trong Amazon S3, sau đó ánh xạ tiếp về parent document để lấy phần ngữ cảnh rộng hơn.

---

## 5. Kiểm tra Vector Index

Sau khi ingestion hoàn tất, cần kiểm tra lại xem index đã tồn tại và có dữ liệu hay chưa.

Có thể kiểm tra nhanh bằng AWS CLI:

```bash
aws s3vectors list-vectors --vector-bucket-name rag-vectors-vanh1234 --index-name hotpotqa-val500-bge-m3-v002 --max-results 5 --region ap-southeast-1
```

Các key trả về cần tương ứng với child document identifier trong bộ artifact v002.

Tuy nhiên, kiểm tra hữu ích hơn là chạy một truy vấn semantic retrieval thực tế. Dự án có bước kiểm tra retrieval trong đó câu hỏi được mã hóa bằng BGE-M3, gửi đến S3 Vectors index, rồi các vector key trả về được ánh xạ lại về document tương ứng.

![Dense retrieval hoạt động trên vector index](/images/5-Workshop/5.6-S3-Vectors/retrieval-check.png)

Ví dụ, với câu hỏi multi-hop:

*"Were Scott Derrickson and Ed Wood of the same nationality?"*

kết quả retrieval cần trả về các tài liệu liên quan đến những thực thể xuất hiện trong câu hỏi. Nếu bước này hoạt động đúng, có thể xác nhận embedding model, vector đã lưu, cấu hình index và document mapping đang nhất quán với nhau.

---

## 6. Backend sử dụng S3 Vectors như thế nào

Khi hệ thống chạy, FastAPI backend thực hiện dense retrieval theo luồng:

**Câu hỏi → BGE-M3 query embedding → Amazon S3 Vectors → matching child ID → child document → parent document**

Vector index chỉ lưu embedding và key tương ứng, còn nội dung văn bản vẫn nằm trong các processed artifact trên Amazon S3 thông thường.

Cách tách này giúp tránh việc lưu lặp toàn bộ document text trong vector store. Amazon S3 Vectors chỉ đảm nhiệm similarity search, còn backend dùng child ID trả về để lấy nội dung tương ứng và mở rộng về parent context khi cần.

Kết quả dense retrieval sau đó được kết hợp với kết quả BM25 để tạo hybrid retrieval pipeline như đã mô tả ở Phần 5.3.

---

## 7. Quyền truy cập của Backend

Trong quá trình vận hành bình thường, EC2 backend chỉ cần quyền truy vấn vector index. Backend không cần tạo index mới hoặc chỉnh sửa vector đã lưu.

Vì vậy, IAM role được cấu hình ở Phần 5.7 chỉ cần cấp cho backend quyền truy cập cần thiết để query các tài nguyên S3 Vectors. Việc tạo index và ingestion vector được giữ riêng như một phần của quá trình deployment.

Cách tổ chức này giúp tách rõ logic phục vụ request khỏi công việc quản lý index.

---

## 8. Kết quả

Sau khi hoàn thành phần này, hai nguồn retrieval chính đã sẵn sàng:

- **Amazon S3** lưu document, BM25 artifact và manifest.
- **Amazon S3 Vectors** lưu **8.279 BGE-M3 child vector** dùng cho semantic retrieval.

Dense vector index cũng đã được kiểm tra bằng một truy vấn retrieval thực tế trước khi backend được đưa vào hệ thống.

Phần 5.7 sẽ triển khai FastAPI backend trên Amazon EC2 và kết nối backend với cả hai nguồn retrieval này.
