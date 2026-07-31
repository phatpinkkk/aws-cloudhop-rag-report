---
title: "Thiết lập S3 Vectors"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

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