---
title: "Offline Artifact Build"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

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