---
title: "Nhật ký thực tập"
date: 2026-06-08
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

Kỳ thực tập tại **AWS First Cloud AI Journey** cho tôi cơ hội đi từ việc làm quen với những dịch vụ AWS nền tảng đến tham gia phát triển một dự án AI hoàn chỉnh cùng nhóm. Trong những tuần đầu, tôi chủ yếu tập trung xây dựng kiến thức thực tế về AWS, bao gồm Amazon EC2, Amazon S3, IAM, networking và các công cụ cơ bản để quản lý tài nguyên trên cloud.

Khi đã có nền tảng cần thiết, nhóm bắt đầu phát triển **AWS CloudHop RAG**, một hệ thống Retrieval-Augmented Generation đa bước dành cho bài toán hỏi đáp trên HotpotQA. Phần công việc tôi tham gia nhiều nhất tập trung vào retrieval và evaluation. Tôi phụ trách chuẩn bị và kiểm tra dữ liệu benchmark, xây dựng và cải thiện retrieval pipeline, thử nghiệm các phương pháp hybrid retrieval và multi-hop retrieval, đồng thời phân tích chất lượng bằng chứng truy xuất, độ chính xác của câu trả lời và latency của hệ thống.

Quá trình làm dự án cũng giúp tôi hiểu rõ hơn cách các thành phần riêng lẻ kết nối với nhau trong một hệ thống thực tế. Trong khi các thành viên trong nhóm phụ trách những phần khác nhau của quá trình triển khai trên AWS và xây dựng ứng dụng, tôi tập trung kiểm tra retrieval pipeline và đánh giá xem hệ thống cuối cùng có tìm đúng bằng chứng và trả lời đúng câu hỏi hay không. Nhờ đó, tôi có thể thấy rõ mối liên hệ giữa các thử nghiệm retrieval mình thực hiện với backend đã triển khai, các dịch vụ AWS và ứng dụng mà người dùng trực tiếp tương tác.

Phần nhật ký này tóm tắt những công việc chính tôi đã thực hiện cùng nhóm trong 8 tuần, từ **08/06/2026 đến 31/07/2026**.

**Tuần 1:** [Làm quen với FCAJ và kiến thức nền tảng về AWS](1.1-week1/)

**Tuần 2:** [Amazon S3, IAM và networking trên AWS](1.2-week2/)

**Tuần 3:** [Machine learning, embedding và nền tảng RAG](1.3-week3/)

**Tuần 4:** [Chuẩn bị HotpotQA và xây dựng retrieval baseline](1.4-week4/)

**Tuần 5:** [Hybrid retrieval và multi-hop retrieval nâng cao](1.5-week5/)

**Tuần 6:** [Hoàn thiện pipeline có khả năng tái lập và kiểm tra benchmark](1.6-week6/)

**Tuần 7:** [Đánh giá retrieval và hoàn thiện kiến trúc AWS](1.7-week7/)

**Tuần 8:** [Hỗ trợ tích hợp cuối, đánh giá và hoàn thiện dự án](1.8-week8/)