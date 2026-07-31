---
title: "Xây dựng Offline Artifact"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

Trước khi RAG backend có thể xử lý câu hỏi, dữ liệu nguồn cần được chuyển thành các artifact sẵn sàng cho retrieval. Quá trình này được thực hiện offline để các bước xử lý tài liệu, chia đoạn, tạo embedding và xây dựng index không phải lặp lại mỗi khi người dùng gửi request.

CloudHop RAG sử dụng **HotpotQA Distractor** làm bộ dữ liệu benchmark cho quá trình này. HotpotQA được thiết kế cho bài toán hỏi đáp đa bước, trong đó thông tin cần thiết cho một câu trả lời có thể nằm trên nhiều tài liệu hỗ trợ khác nhau. Các trường câu hỏi, câu trả lời, context và supporting facts đã được gán nhãn tạo điều kiện thuận lợi cho việc xây dựng và đánh giá retrieval pipeline của dự án.

Bộ artifact cuối cùng được xây dựng từ **500 câu hỏi thuộc validation split**. Context của các câu hỏi được chuẩn hóa về định dạng corpus của dự án trước khi tạo parent document, child chunk, BM25 index và BGE-M3 embedding.

### Offline Artifact Build

Bước này chuẩn bị dữ liệu và tạo các file chỉ mục (index) cần thiết.

#### 1. CLI Steps: Môi trường chạy notebook
Bạn cần chạy notebook `build_s3_offline_artifacts.ipynb` trong thư mục `backend/notebooks/`. Đảm bảo máy tính của bạn đã cài đặt các thư viện cần thiết:

```bash
pip install jupyter pandas numpy scikit-learn
```

#### 2. Notebook UI Steps: Cấu hình dataset
Mở Jupyter Notebook, tìm đoạn cấu hình dữ liệu và điều chỉnh kích thước dataset phù hợp (ví dụ: 500 mẫu cho bản demo):

```python
# Cấu hình trong notebook
DATASET_SIZE = 500
```

#### 3. Notebook UI Steps: Thực thi
Chạy toàn bộ các cells trong notebook. Sau khi chạy xong, các file cần thiết (`corpus.jsonl`, `index_manifest.json`,...) sẽ xuất hiện trong thư mục `output/`.