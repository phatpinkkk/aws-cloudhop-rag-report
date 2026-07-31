---
title: "Lưu trữ trên Amazon S3"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Các retrieval artifact được tạo ở bước trước cần được lưu trữ độc lập với EC2 instance đang vận hành backend. Vì vậy, CloudHop RAG sử dụng **Amazon S3** làm nơi lưu trữ lâu dài cho corpus đã xử lý, BM25 index, document mapping và các index manifest cần thiết cho RAG pipeline khi chạy online.

Việc lưu các artifact này trên S3 cho phép backend tải và sử dụng đúng phiên bản artifact khi khởi động mà không cần xây dựng lại corpus hoặc retrieval index trên EC2.

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