---
title: "Triển khai Backend trên Amazon EC2"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

FastAPI backend là thành phần trực tiếp thực thi phần lớn logic của CloudHop RAG. Backend nhận câu hỏi từ lớp API, tải các retrieval artifact đã chuẩn bị, thực hiện BM25 và dense retrieval, điều phối pipeline multi-hop, xây dựng context cuối cùng và gửi phần bằng chứng đã chọn đến Groq để sinh câu trả lời.

Backend được triển khai trên **Amazon EC2** để môi trường Python, BM25 index và embedding model có thể được giữ sẵn giữa các request thay vì phải khởi tạo lại từ đầu. Phần này hướng dẫn triển khai backend và kết nối nó với các tài nguyên Amazon S3 và Amazon S3 Vectors đã chuẩn bị ở Phần 5.5 và 5.6.

---

## 1. Tạo IAM Role cho EC2 Runtime

Trước khi tạo EC2 instance, cần tạo một IAM role với tên:

`rag-ec2-runtime-role`

Role này được gắn trực tiếp vào EC2 instance để backend có thể truy cập các dịch vụ AWS cần thiết mà không phải lưu AWS access key trong ứng dụng.

Runtime role cần quyền để:

- đọc các artifact của CloudHop RAG từ **Amazon S3**;
- truy vấn **Amazon S3 Vectors** index mà hệ thống sử dụng;
- sử dụng **AWS Systems Manager Session Manager** để quản trị instance.

Managed policy **`AmazonSSMManagedInstanceCore`** được gắn vào role để EC2 có thể được truy cập thông qua Session Manager.

Trong quá trình xử lý câu hỏi thông thường, ứng dụng chỉ cần đọc artifact và truy vấn vector index, không cần quyền chỉnh sửa các dữ liệu này.

---

## 2. Khởi tạo EC2 Instance

Tạo một Ubuntu EC2 instance tại Region **`ap-southeast-1`** và gắn IAM role `rag-ec2-runtime-role`.

Cấu hình chính của dự án:

| Thiết lập | Cấu hình của dự án |
| --- | --- |
| Hệ điều hành | Ubuntu Server LTS |
| Region | `ap-southeast-1` |
| IAM role | `rag-ec2-runtime-role` |
| Backend port | `8000` |
| Quản trị instance | AWS Systems Manager Session Manager |
| Thiết bị runtime | CPU |

Instance cần có đủ bộ nhớ để chạy môi trường Python, BGE-M3 query encoder, BM25 index và các artifact được tải từ S3.

Một **Elastic IP** được gắn với instance để địa chỉ backend không thay đổi sau khi EC2 được stop rồi start lại. Địa chỉ này sẽ được Amazon API Gateway sử dụng ở bước tiếp theo.

![EC2 instance chạy backend](/images/5-Workshop/5.7-EC2-Backend/ec2-instance.png)

Trong phạm vi workshop, port `8000` được mở để API Gateway có thể kết nối đến backend. Đây là lựa chọn thực tế cho môi trường demo của dự án, chưa phải một thiết kế mạng production hoàn chỉnh. Các giới hạn bảo mật còn lại sẽ được phân tích thêm ở Phần 5.13.

---

## 3. Kết nối bằng Systems Manager

Việc quản trị instance được thực hiện bằng **AWS Systems Manager Session Manager** thay vì SSH thông thường.

Từ AWS Console:

**EC2 → Instances → chọn backend instance → Connect → Session Manager**

Cách này giúp không phải quản lý SSH key trong quy trình của dự án và cũng không cần mở port `22`.

![Kết nối EC2 bằng Session Manager](/images/5-Workshop/5.7-EC2-Backend/session-manager.png)

Sau khi kết nối, chuyển sang Ubuntu user nếu cần và tiếp tục cài đặt trực tiếp từ terminal của instance.

---

## 4. Cài đặt Backend

Clone mã nguồn và tạo môi trường Python trên EC2.

```bash
cd ~
git clone https://github.com/vietanh1802/aws-rag-project.git
cd ~/aws-rag-project/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Backend không xây dựng lại corpus hay tạo lại toàn bộ vector index. Những artifact này đã được chuẩn bị ở giai đoạn offline và lưu trên AWS.

Khi chạy, ứng dụng chỉ tải các artifact cần thiết từ Amazon S3 và truy vấn Amazon S3 Vectors trong quá trình xử lý câu hỏi.

---

## 5. Cấu hình môi trường Production

Các thông số runtime được lưu trong:

`/home/ubuntu/aws-rag-project/backend/.env.prod`

File này xác định những tài nguyên AWS mà backend sử dụng, đồng thời chọn cấu hình retrieval nhẹ hơn dành cho phiên bản đã triển khai.

```env
AWS_REGION=ap-southeast-1
GROQ_API_KEY=<your-groq-key>

S3_ARTIFACT_BUCKET=aws-rag-bucket-vanh1234
S3_VECTOR_BUCKET=rag-vectors-vanh1234
RAG_INDEX_ID=hotpotqa-val500-bge-m3-v002
S3_PROCESSED_ID=hotpotqa-val500-v002
S3_VECTOR_INDEX=hotpotqa-val500-bge-m3-v002

RAG_DEVICE=cpu
RAG_FAST_MODE=true
RAG_USE_RERANKER=false
BM25_TOP_K=15
VECTOR_TOP_K=15
HOP_CANDIDATE_CAP=15
MAX_ADAPTIVE_HOPS=1
HOP_EVIDENCE_TOP_N=3
```

Khác biệt chính so với cấu hình dùng để đánh giá chất lượng là phiên bản triển khai giảm độ sâu retrieval và tắt cross-encoder reranker để phù hợp hơn với CPU và giảm latency.

Groq API key được lưu trong `.env.prod`, còn file này được loại khỏi Git. AWS credential không được lưu tại đây vì AWS SDK sẽ tự nhận temporary credential thông qua IAM role gắn với EC2 instance.

![Cấu hình môi trường production](/images/5-Workshop/5.7-EC2-Backend/env-prod.png)

Khi chụp màn hình cấu hình để đưa vào báo cáo, cần xóa hoặc che Groq API key.

---

## 6. Chạy FastAPI bằng systemd

Backend được quản lý bằng `systemd` để service có thể tự khởi động cùng EC2 instance và tự restart nếu process gặp lỗi.

Service chạy Uvicorn trên port `8000` và sử dụng `.env.prod` làm runtime configuration.

Sau khi tạo service file, chạy:

```bash
sudo systemctl daemon-reload
sudo systemctl enable aws-rag-api
sudo systemctl restart aws-rag-api
sudo systemctl status aws-rag-api
```

Có thể theo dõi log của ứng dụng bằng:

`sudo journalctl -u aws-rag-api -f`

Backend cũng cung cấp endpoint `/warmup`. Endpoint này được gọi sau khi service khởi động để các thành phần retrieval được khởi tạo trước, tránh để request đầu tiên của người dùng phải chờ toàn bộ quá trình startup.

---

## 7. Kiểm tra Backend

Trước tiên, kiểm tra service trực tiếp trên EC2 instance:

`curl http://127.0.0.1:8000/health`

Sau đó khởi tạo retrieval pipeline:

`curl -X POST http://127.0.0.1:8000/warmup`

Cuối cùng, gửi một câu hỏi thử nghiệm đến query endpoint:

```bash
curl -X POST http://127.0.0.1:8000/query -H "Content-Type: application/json" -d '{"question":"Were Scott Derrickson and Ed Wood of the same nationality?"}'
```

Nếu request trả về kết quả thành công, có thể xác nhận backend đã:

- tải được processed document và BM25 artifact từ Amazon S3;
- truy vấn được Amazon S3 Vectors;
- chạy được RAG pipeline;
- gọi Groq để sinh câu trả lời.

Ở thời điểm này, backend đã hoạt động trên EC2 qua HTTP. Phần tiếp theo sẽ đặt **Amazon API Gateway** phía trước backend để cung cấp HTTPS endpoint cho frontend.

---

## 8. Kết quả

Sau khi hoàn thành phần này, CloudHop RAG backend đã chạy ổn định dưới dạng một FastAPI service trên Amazon EC2.

Môi trường triển khai hiện có:

- một EC2 instance với Elastic IP cố định;
- quyền truy cập Amazon S3 và Amazon S3 Vectors thông qua IAM role;
- Session Manager để quản trị instance;
- runtime configuration trong `.env.prod`;
- `systemd` service giúp backend chạy liên tục;
- các endpoint `/health`, `/warmup` và `/query` hoạt động bình thường.

Phần 5.8 sẽ đưa backend ra ngoài thông qua Amazon API Gateway để frontend trên AWS Amplify có thể gọi API bằng HTTPS.
