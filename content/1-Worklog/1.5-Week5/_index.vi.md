---
title: "Nhật ký tuần 5"
date: 2026-07-06
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5

* Cải thiện chất lượng retrieval so với các baseline TF-IDF và MiniLM ban đầu.
* Áp dụng các phương pháp lexical và semantic retrieval mạnh hơn.
* Kết hợp BM25 và BGE-M3 thành một hybrid retrieval pipeline.
* Thiết kế cách biểu diễn tài liệu theo mô hình parent-child để vừa hỗ trợ retrieval hiệu quả, vừa giữ đủ context cho các bước phía sau.
* Bổ sung query decomposition, adaptive multi-hop retrieval và cross-encoder reranking.

### Công việc thực hiện trong tuần

| Ngày | Công việc | Tài liệu tham khảo |
| --- | --- | --- |
| 06/07/2026 | - Xem lại những hạn chế của các baseline retrieval.<br>- Thiết kế advanced retrieval benchmark với giao diện retriever thống nhất.<br>- Tìm hiểu BM25, dense retrieval mạnh hơn, hybrid retrieval, reranking và các chiến lược multi-hop retrieval. | |
| 07/07/2026 | - Xây dựng BM25 lexical retrieval.<br>- Đưa **BAAI/bge-m3** vào làm dense embedding model chính.<br>- Phân tích vai trò bổ trợ giữa keyword matching và semantic similarity. | |
| 08/07/2026 | - Thiết kế cách biểu diễn tài liệu theo mô hình parent-child.<br>- Chia tài liệu nguồn thành các child chunk nhỏ để hỗ trợ dense retrieval.<br>- Giữ lại parent document để cung cấp context đầy đủ hơn.<br>- Xây dựng ánh xạ child-to-parent. | |
| 09/07/2026 | - Kết hợp BM25 và BGE-M3 thành hybrid candidate pipeline.<br>- Xây dựng query decomposition cho các câu hỏi phức tạp.<br>- Bổ sung adaptive retrieval để hệ thống có thể thực hiện thêm retrieval hop khi bằng chứng hiện tại chưa đủ. | |
| 10/07/2026 | - Bổ sung reranking bằng `cross-encoder/ms-marco-MiniLM-L-6-v2`.<br>- Hoàn thiện retrieval flow: decomposition → BM25 + BGE-M3 → adaptive retrieval → candidate aggregation → reranking → selected evidence.<br>- Kiểm tra bằng chứng truy xuất và điều chỉnh các tham số cấu hình. | |

### Kết quả đạt được

* Xây dựng BM25 lexical retrieval.
* Chọn BGE-M3 làm dense embedding model chính của hệ thống.
* Hoàn thiện hybrid retrieval pipeline kết hợp lexical và semantic retrieval.
* Xây dựng parent-child document indexing.
* Bổ sung query decomposition và adaptive multi-hop retrieval.
* Tích hợp cross-encoder reranking để cải thiện thứ tự của các bằng chứng đã truy xuất.
* Hoàn thiện kiến trúc retrieval chính được sử dụng trong CloudHop RAG.