---
title: "Đánh giá hệ thống"
date: 2026-07-31
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

Sau khi xác nhận toàn bộ hệ thống đã hoạt động đúng về mặt chức năng, CloudHop RAG được đánh giá trên ba khía cạnh chính: **chất lượng retrieval, chất lượng câu trả lời và hiệu năng runtime**.

Toàn bộ kết quả cuối cùng sử dụng bộ artifact **HotpotQA Distractor v002** đã được hiệu chỉnh. Benchmark retrieval gồm **500 câu hỏi validation**, **4.937 parent document** và **8.279 BGE-M3 child vector**.

Chất lượng retrieval được đánh giá ở mức **supporting title**, tức là kiểm tra hệ thống có tìm đúng các tài liệu chứa bằng chứng cần thiết hay không. Các số liệu này không phải là metric Supporting-Fact EM/F1 ở mức câu của HotpotQA chính thức. Đối với câu trả lời cuối, hệ thống sử dụng Exact Match (EM) và token-level F1.

## 1. Chất lượng Retrieval

Retrieval pipeline được đánh giá ở hai giai đoạn.

**Candidate Pool** cho biết quá trình retrieval có tìm được các supporting document cần thiết trước bước chọn bằng chứng cuối hay không. **Selected Top-10** phản ánh tập tài liệu cuối cùng được giữ lại để sử dụng cho bước sinh câu trả lời.

| Metric | Candidate Pool | Selected Top-10 |
| --- | ---: | ---: |
| Mean supporting-title recall | **0.9920** | **0.9740** |
| All supporting titles found | **0.9840** | **0.9480** |
| Recall@5 | 0.5420 | **0.9310** |
| Recall@10 | 0.6270 | **0.9740** |
| Precision@10 | 0.1254 | **0.1948** |
| MRR | 0.7807 | **0.9446** |
| nDCG@10 | 0.5816 | **0.9162** |

Ở giai đoạn candidate, pipeline có độ bao phủ bằng chứng rất cao. Hệ thống tìm đủ toàn bộ supporting title cho **492 trên 500 câu hỏi**, tương ứng tỷ lệ **0.9840**.

Sau bước chọn bằng chứng, toàn bộ supporting title vẫn còn trong Top-10 đối với **474 trên 500 câu hỏi**, trong khi mean supporting-title recall vẫn đạt **0.9740**.

Điểm đáng chú ý là bước chọn cuối giúp đưa các bằng chứng liên quan lên những vị trí cao hơn rõ rệt. Recall@5 tăng từ **0.5420 lên 0.9310**, MRR từ **0.7807 lên 0.9446** và nDCG@10 từ **0.5816 lên 0.9162**. Điều này cho thấy pipeline không chỉ tìm được bằng chứng mà còn sắp xếp chúng tốt hơn trong phần context cuối. Tuy nhiên, khi rút candidate pool lớn xuống còn mười tài liệu, một lượng nhỏ supporting evidence vẫn bị mất.

## 2. Chất lượng câu trả lời End-to-End

Một tập cố định gồm **20 câu hỏi** được sử dụng để đánh giá toàn bộ pipeline từ retrieval đến answer generation. Cả 20 request đều hoàn thành thành công.

| Metric | Kết quả |
| --- | ---: |
| Answer EM | **0.7500** |
| Answer F1 | **0.7750** |
| Câu trả lời exact match | **15 / 20** |
| Mean supporting-title recall | **0.9500** |
| All supporting titles found | **0.9000** |
| Recall@5 | **0.9250** |
| Recall@10 | **0.9500** |
| MRR | **0.9667** |
| nDCG@10 | **0.9243** |

Tập bằng chứng cuối chứa đủ toàn bộ supporting title cho **18 trên 20 câu hỏi**. Answer EM đạt **0.7500**, nghĩa là có 15 câu trả lời khớp hoàn toàn với đáp án tham chiếu, còn token-level F1 đạt **0.7750**.

Khoảng cách giữa độ bao phủ bằng chứng và độ chính xác của câu trả lời cũng là một điểm cần lưu ý. Retrieval tốt giúp tăng khả năng hệ thống có đủ thông tin cần thiết, nhưng câu trả lời cuối vẫn phụ thuộc vào việc generation stage hiểu và kết hợp các bằng chứng đó hiệu quả đến đâu.

## 3. Hiệu năng Runtime

Trên benchmark retrieval 500 câu hỏi, latency trung bình của toàn bộ retrieval pipeline là **25,91 giây mỗi câu**, với median là **25,72 giây**.

| Giai đoạn | Latency trung bình | Tỷ lệ trong tổng thời gian |
| --- | ---: | ---: |
| Query decomposition | 4,32 s | 16,7% |
| Retrieval + adaptive planning | 21,07 s | 81,3% |
| Cross-encoder reranking | 0,53 s | 2,0% |
| Toàn bộ retrieval pipeline | **25,91 s** | 100% |

Phần chiếm nhiều thời gian nhất là **retrieval và adaptive planning**, khoảng 81% tổng latency của retrieval pipeline. Query decomposition chiếm gần 17%, trong khi cross-encoder reranking chỉ chiếm khoảng 2%.

Với benchmark end-to-end 20 câu hỏi:

| Thành phần | Latency trung bình |
| --- | ---: |
| Retrieval pipeline | 26,86 s |
| Final generation | 12,43 s |
| End-to-end | **39,29 s** |

Kết quả này cho thấy cả retrieval và generation đều đóng góp đáng kể vào thời gian phản hồi, nhưng retrieval vẫn là thành phần chiếm nhiều thời gian hơn.

## 4. Đánh đổi giữa chất lượng và cấu hình triển khai

Các kết quả ở trên được đo bằng **cấu hình ưu tiên chất lượng**, sử dụng phạm vi retrieval rộng hơn, tối đa ba adaptive hop và cross-encoder reranking.

Trong khi đó, phiên bản triển khai trên AWS sử dụng một cấu hình nhẹ hơn để phù hợp với môi trường EC2 chạy CPU. Cấu hình này giảm số candidate từ BM25 và dense retrieval, giới hạn adaptive retrieval, bật Fast Mode và tắt cross-encoder reranker.

Vì vậy, hai cấu hình nên được hiểu là hai chế độ vận hành của cùng một hệ thống chứ không phải một phép ablation có kiểm soát. Cấu hình evaluation ưu tiên chất lượng retrieval, còn cấu hình triển khai cân bằng thêm thời gian phản hồi và lượng tài nguyên cần sử dụng.

Nhìn chung, kết quả cho thấy CloudHop RAG có thể tìm lại phần lớn supporting evidence cần thiết cho các câu hỏi multi-hop trong HotpotQA và đạt kết quả tốt ở bài toán trả lời ngắn. Thách thức kỹ thuật còn rõ nhất là latency, đặc biệt ở phần retrieval và adaptive planning.
