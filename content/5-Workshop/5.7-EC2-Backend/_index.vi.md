---
title: "Triển khai Backend trên Amazon EC2"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

FastAPI backend là lớp xử lý chính của CloudHop RAG. Backend nhận câu hỏi từ lớp API, tải các retrieval artifact đã chuẩn bị, thực hiện BM25 và dense retrieval, điều phối multi-hop pipeline, xây dựng context cuối cùng và gửi request sinh câu trả lời đến Groq.

Backend được triển khai trên **Amazon EC2**, cung cấp một môi trường vận hành ổn định để Python RAG pipeline và các dependency của hệ thống có thể được duy trì giữa các request.

### Triển khai Backend trên EC2

Cài đặt FastAPI backend trên EC2 sử dụng `systemd` để chạy dưới dạng service.

*(Lưu ý: Các bước này thực hiện trên Terminal sau khi đã SSH hoặc dùng Session Manager vào EC2)*

#### 1. CLI Steps: Chuẩn bị môi trường
**Clone dự án và cài đặt dependencies:**

```bash
# Clone dự án
git clone https://github.com/vietanh1802/aws-rag-project.git
cd ~/aws-rag-project/backend

# Tạo môi trường ảo
python3 -m venv .venv
source .venv/bin/activate

# Cài đặt thư viện
pip install -r requirements.txt
```

#### 2. CLI Steps: Cấu hình môi trường (`.env.prod`)
Tạo file `~/aws-rag-project/backend/.env.prod` bằng lệnh `nano ~/aws-rag-project/backend/.env.prod` và dán nội dung sau (thay `GROQ_API_KEY` của bạn):

```env
USE_TF=0
USE_FLAX=0
AWS_REGION=ap-southeast-1
GROQ_API_KEY=gsk_xxxxxxxxxxxxxxxxxxxxxxxxxxxx
RAG_INDEX_ID=hotpotqa-val500-bge-m3-v001
S3_ARTIFACT_BUCKET=aws-rag-bucket-vanh1234
S3_PROCESSED_ID=hotpotqa-val500-v001
S3_VECTOR_BUCKET=rag-vectors-vanh1234
S3_VECTOR_INDEX=hotpotqa-val500-bge-m3-v001
RAG_ARTIFACT_LAYOUT=s3vectors
RAG_AUTO_DOWNLOAD_ARTIFACTS=true
RAG_DEVICE=cpu
RAG_FAST_MODE=true
RAG_USE_RERANKER=false
BM25_TOP_K=15
VECTOR_TOP_K=15
HOP_CANDIDATE_CAP=15
MAX_ADAPTIVE_HOPS=1
HOP_EVIDENCE_TOP_N=3
RERANK_TOP_N=20
```

#### 3. CLI Steps: Tạo Systemd Service
Tạo file service để tự động chạy backend khi khởi động máy:

```bash
sudo nano /etc/systemd/system/aws-rag-api.service
```

Dán nội dung sau:

```ini
[Unit]
Description=AWS RAG API Service
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/aws-rag-project/backend
EnvironmentFile=/home/ubuntu/aws-rag-project/backend/.env.prod
ExecStart=/home/ubuntu/aws-rag-project/backend/.venv/bin/uvicorn app.main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```

#### 4. CLI Steps: Kích hoạt Service

```bash
sudo systemctl daemon-reload
sudo systemctl enable aws-rag-api
sudo systemctl restart aws-rag-api
```

#### 5. CLI Steps: Kiểm tra trạng thái

```bash
# Kiểm tra log
sudo journalctl -u aws-rag-api -f

# Kiểm tra health check
curl http://127.0.0.1:8000/health