---
title: "Amazon S3 Vectors"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

BM25 hoạt động hiệu quả khi câu hỏi có nhiều từ hoặc cụm từ trùng với tài liệu hỗ trợ, nhưng các câu hỏi đa bước cũng có thể cần những bằng chứng được diễn đạt theo cách khác. Vì vậy, CloudHop RAG kết hợp lexical retrieval với dense semantic retrieval thông qua **Amazon S3 Vectors**.

Mỗi child chunk được tạo trong quá trình offline build được mã hóa bằng **BGE-M3** thành vector 1.024 chiều. Các vector này được lưu trong S3 Vectors index và được EC2 backend truy vấn tại runtime bằng embedding của câu hỏi hoặc retrieval query tương ứng.

### Thiết lập S3 Vectors

S3 Vectors lưu trữ các vector embeddings phục vụ cho tính năng dense retrieval.

#### 1. Console Steps: Cấu hình Index
Truy cập **S3 Vectors Console** (hoặc service tương ứng) để tạo Index mới với các thông số:
- **Dimension**: 1024
- **Distance metric**: cosine
- **Bucket**: `rag-vectors-vanh1234`
- **Index name**: `hotpotqa-val500-bge-m3-v001`

#### 2. CLI Steps: Ingest Vectors
Sử dụng script `ingest_s3vectors.py` (nằm trong project) để đẩy dữ liệu từ các file JSON đã tạo ở bước 5.2 lên S3 Vectors bucket:

```bash
# Di chuyển vào thư mục chứa script
cd path/to/your/scripts

# Chạy script ingest
python ingest_s3vectors.py --region ap-southeast-1
```

#### 3. CLI Steps: Kiểm tra
Sử dụng AWS CLI để liệt kê vector và đảm bảo dữ liệu đã có:

```bash
aws s3vectors list-vectors \
  --region ap-southeast-1 \
  --vector-bucket-name rag-vectors-vanh1234 \
  --index-name hotpotqa-val500-bge-m3-v001 \
  --max-items 5