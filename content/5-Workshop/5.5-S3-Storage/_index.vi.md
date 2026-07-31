---
title: "Thiết lập S3 Storage"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

### Thiết lập S3 Storage

S3 đóng vai trò là kho lưu trữ trung tâm cho dữ liệu corpus, artifacts đã xử lý và BM25 index.

#### 1. Console Steps: Tạo bucket
Truy cập **S3 Console** để tạo các bucket trên vùng `ap-southeast-1`:
- Bucket 1: `aws-rag-bucket-vanh1234` (lưu trữ corpus/artifacts)
- Bucket 2: `rag-vectors-vanh1234` (lưu trữ vector embeddings)

#### 2. CLI Steps: Chuẩn bị dữ liệu và Upload
Đảm bảo bạn chuẩn bị cấu trúc thư mục trên máy tính local:

```text
./rag/
  corpora/ ...
  processed/ ...
  indexes/ ...
```

Sử dụng AWS CLI để đồng bộ dữ liệu từ local lên S3:

```bash
# Upload corpus & indexes
aws s3 sync "./rag" "s3://aws-rag-bucket-vanh1234/rag" --region ap-southeast-1

# Kiểm tra dữ liệu đã upload
aws s3 ls s3://aws-rag-bucket-vanh1234/rag/ --recursive --region ap-southeast-1