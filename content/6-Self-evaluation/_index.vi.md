---
title: "Tự đánh giá"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 6. </b> "
---

Trong kỳ thực tập tại **AWS First Cloud AI Journey**, điều tôi thấy có giá trị nhất là được tham gia vào một dự án AI tương đối hoàn chỉnh, thay vì chỉ dừng lại ở những bài thực hành hoặc thử nghiệm riêng lẻ. Với **AWS CloudHop RAG**, phần công việc tôi phụ trách nhiều nhất là retrieval và evaluation, từ chuẩn bị benchmark, xây dựng lexical retrieval và dense retrieval, phát triển multi-hop retrieval, xử lý lỗi cho đến phân tích chất lượng và hiệu năng của hệ thống. Qua quá trình này, tôi tự tin hơn khi gặp một vấn đề kỹ thuật mới, biết cách kiểm tra lại kết quả thay vì vội tin vào một con số, đồng thời hiểu rõ hơn một pipeline AI cần kết nối với các thành phần khác như thế nào để trở thành một ứng dụng cloud hoàn chỉnh.

Nhìn chung, tôi đánh giá mình đã hoàn thành khá tốt những phần việc được giao, đặc biệt là các nhiệm vụ liên quan trực tiếp đến retrieval và đánh giá hệ thống. Điểm mạnh của tôi là khả năng tự học nhanh, kiên trì tìm nguyên nhân khi kết quả có vấn đề và khá cẩn thận khi làm việc với dữ liệu cũng như các chỉ số đánh giá. Tuy nhiên, tôi không cho rằng quá trình làm việc của mình đã tối ưu. Có những thử nghiệm mất nhiều thời gian hơn cần thiết, và đôi lúc tôi dành quá lâu để tự tìm hiểu một vấn đề trước khi trao đổi với các thành viên khác. Tôi cũng chưa có nhiều thời gian trực tiếp thao tác với một số phần của quá trình triển khai AWS như mình mong muốn.

### Tự đánh giá

| STT | Tiêu chí | Tự đánh giá | Tốt | Khá | Trung bình |
| --- | --- | --- | :---: | :---: | :---: |
| 1 | **Kiến thức chuyên môn và kỹ năng kỹ thuật** | Tôi hiểu sâu hơn về RAG, lexical retrieval, semantic retrieval, embedding, multi-hop retrieval, evaluation và cách những thành phần này phối hợp trong một ứng dụng triển khai trên AWS. | ✅ | ☐ | ☐ |
| 2 | **Khả năng học hỏi** | Tôi có thể tiếp cận tương đối nhanh các phương pháp retrieval, kỹ thuật đánh giá và kiến thức AWS mới, sau đó áp dụng trực tiếp vào dự án. | ✅ | ☐ | ☐ |
| 3 | **Tính chủ động** | Tôi chủ động tìm hiểu thêm các hướng retrieval và evaluation ngoài baseline ban đầu, đồng thời tự kiểm tra những kết quả bất thường thay vì chỉ tiếp tục chạy thử nghiệm. | ✅ | ☐ | ☐ |
| 4 | **Kỷ luật và quản lý thời gian** | Tôi hoàn thành các đầu việc chính, nhưng một số thử nghiệm và benchmark vẫn có thể được lên kế hoạch, sắp xếp ưu tiên và giới hạn phạm vi tốt hơn. | ☐ | ✅ | ☐ |
| 5 | **Giao tiếp** | Tôi thường xuyên chia sẻ kết quả và phát hiện kỹ thuật với nhóm, nhưng cần cải thiện việc báo sớm những vướng mắc và tiến độ trung gian thay vì đợi đến khi đã tự tìm hiểu khá lâu. | ☐ | ✅ | ☐ |
| 6 | **Làm việc nhóm** | Tôi phối hợp tốt với các thành viên phụ trách retrieval, backend, frontend và triển khai AWS, đồng thời tham gia kiểm tra hệ thống sau khi các thành phần được tích hợp. | ✅ | ☐ | ☐ |
| 7 | **Giải quyết vấn đề** | Khi retrieval hoặc evaluation cho kết quả bất thường, tôi thường lần theo từng bước của pipeline, kiểm tra dữ liệu, candidate, selected evidence và thời gian xử lý thay vì chỉ nhìn vào metric cuối cùng. | ✅ | ☐ | ☐ |
| 8 | **Mức độ đóng góp cho dự án** | Phần đóng góp chính của tôi nằm ở chuẩn bị benchmark, phát triển retrieval, evaluation, debugging, phân tích kết quả và hỗ trợ kiểm tra hệ thống cuối cùng. | ✅ | ☐ | ☐ |
| 9 | **Đánh giá chung** | Tôi hoàn thành tốt phần trách nhiệm chính và đóng góp ổn định cho dự án, đồng thời nhận ra mình vẫn cần cải thiện thêm về kinh nghiệm triển khai AWS, giao tiếp trong quá trình xử lý vấn đề và hiệu quả khi tổ chức thử nghiệm. | ✅ | ☐ | ☐ |

### Đóng góp cá nhân

Trong CloudHop RAG, phần tôi phụ trách chính là **retrieval và evaluation**. Tôi tham gia chuẩn bị và kiểm tra benchmark HotpotQA, xây dựng và so sánh các phương pháp retrieval, cải thiện pipeline multi-hop và phân tích cả chất lượng truy xuất lẫn thời gian xử lý.

Ở giai đoạn cuối, tôi cũng tham gia kiểm tra lại hệ thống sau khi pipeline được tích hợp vào ứng dụng AWS, đặc biệt là xem các bằng chứng được truy xuất và câu trả lời được sinh ra có còn hợp lý hay không. Dù không trực tiếp phụ trách toàn bộ quá trình triển khai, công việc này buộc tôi phải hiểu cách phần retrieval của mình tương tác với backend, API, vector storage và các thành phần khác trong hệ thống.

Một điều tôi học được khá rõ từ quá trình evaluation là không nên nhìn một metric riêng lẻ rồi kết luận ngay hệ thống tốt hay chưa. Khi có kết quả bất thường, tôi phải quay lại kiểm tra dữ liệu đầu vào, candidate pool, bằng chứng sau reranking và cả câu trả lời cuối cùng. Cách làm này giúp tôi cẩn thận hơn khi đọc và giải thích kết quả thử nghiệm.

### Khó khăn và cách xử lý

Một trong những khó khăn lớn nhất tôi gặp phải là cân bằng giữa **chất lượng retrieval và giới hạn thực tế của môi trường triển khai**.

Cấu hình dùng để đánh giá chất lượng có thể sử dụng nhiều candidate hơn, nhiều retrieval hop hơn, thêm reranking và nhiều lần gọi mô hình hơn. Những thành phần này giúp hệ thống tìm và sắp xếp bằng chứng tốt hơn, nhưng đổi lại latency và nhu cầu tài nguyên cũng tăng đáng kể. Trong khi đó, môi trường AWS mà nhóm sử dụng cần đáp ứng những giới hạn thực tế hơn về CPU, thời gian phản hồi và chi phí.

Thay vì cố ép một cấu hình duy nhất phải phù hợp cho cả hai mục đích, nhóm giữ một cấu hình mạnh hơn để đánh giá có kiểm soát, còn ứng dụng triển khai sử dụng cấu hình gọn hơn với ít candidate, ít bước retrieval và ít xử lý tốn tài nguyên hơn.

Đây là một bài học quan trọng đối với tôi vì nó giúp tôi nhận ra rằng trong kỹ thuật hiếm khi tồn tại một cấu hình đơn giản là "tốt nhất" trong mọi trường hợp. Khi thử nghiệm, ưu tiên có thể là chất lượng retrieval; nhưng khi đưa hệ thống vào sử dụng thực tế, còn phải tính đến latency, chi phí, tài nguyên sẵn có và độ ổn định. Việc tìm được điểm cân bằng phù hợp giữa những yếu tố này cũng là một phần quan trọng của công việc kỹ sư.

### Những điểm cần cải thiện

Điều đầu tiên tôi muốn cải thiện là **kinh nghiệm triển khai AWS và làm việc trực tiếp với hạ tầng cloud**. Đến cuối kỳ thực tập, tôi đã hiểu kiến trúc tổng thể của hệ thống rõ hơn nhiều so với lúc bắt đầu, nhưng công việc của tôi vẫn tập trung chủ yếu vào retrieval và evaluation. Trong các dự án sau, tôi muốn trực tiếp phụ trách nhiều hơn những phần như triển khai backend, networking, phân quyền, tích hợp API và xử lý sự cố trong môi trường production.

Tôi cũng muốn cải thiện cách **quản lý thời gian dành cho thử nghiệm**. Khi phát triển retrieval, có rất nhiều phương pháp và cấu hình có thể thử. Một số thử nghiệm mang lại thông tin hữu ích, nhưng cũng có những thử nghiệm không ảnh hưởng nhiều đến hướng cuối cùng của dự án. Sau trải nghiệm này, tôi muốn tập thói quen đặt ra giả thuyết rõ hơn trước khi chạy thử nghiệm và ưu tiên những thử nghiệm có khả năng trả lời một câu hỏi kỹ thuật thực sự quan trọng.

Một điểm khác là **giao tiếp trong quá trình giải quyết vấn đề**. Tôi thường có xu hướng tự tìm hiểu khá sâu trước khi đưa vấn đề ra trao đổi. Điều này giúp tôi rèn khả năng làm việc độc lập, nhưng nếu vấn đề có ảnh hưởng đến phần việc của những thành viên khác thì việc chia sẻ sớm hơn sẽ giúp cả nhóm phối hợp hiệu quả hơn và tránh mất thời gian trùng lặp.

### Làm việc nhóm

Quá trình làm việc với nhóm nhìn chung diễn ra khá thuận lợi vì sau một thời gian, mỗi thành viên dần tập trung vào những phần phù hợp với thế mạnh của mình. Tôi dành nhiều thời gian hơn cho retrieval, chuẩn bị benchmark và evaluation, trong khi các thành viên khác tham gia sâu hơn vào backend, frontend hoặc những phần cụ thể của quá trình triển khai AWS.

Dù có sự phân chia như vậy, các phần việc vẫn liên quan chặt chẽ với nhau. Tôi thường xuyên chia sẻ kết quả retrieval và evaluation với nhóm, đồng thời hỗ trợ kiểm tra xem ứng dụng cuối cùng có trả về đúng bằng chứng và câu trả lời như mong đợi hay không.

Nhìn lại, tôi nghĩ mình có thể chủ động tham gia trực tiếp nhiều hơn vào những phần nằm ngoài trách nhiệm chính. Việc chia nhỏ công việc giúp nhóm tiến nhanh hơn, nhưng tôi cũng nhận ra rằng chuyên môn hóa không có nghĩa là chỉ hiểu phần của mình. Với một hệ thống AI hoàn chỉnh, mỗi thành viên vẫn cần có đủ cái nhìn tổng thể để hiểu những thay đổi ở một thành phần có thể ảnh hưởng đến các phần còn lại như thế nào.

### Điều rút ra sau kỳ thực tập

Bài học lớn nhất tôi nhận được từ kỳ thực tập là **làm tốt phần AI mới chỉ là một phần của công việc AI Engineer**.

Retrieval và evaluation là những phần tôi cảm thấy tự tin nhất, nhưng để CloudHop RAG thực sự hoạt động còn cần dữ liệu được chuẩn bị đúng, backend ổn định, API kết nối được, hạ tầng AWS được cấu hình phù hợp, quyền truy cập được quản lý an toàn và các thành phần phải phối hợp với nhau trong giới hạn latency và tài nguyên thực tế. Một phương pháp cho kết quả tốt trong thử nghiệm chưa chắc đã là giải pháp tốt khi đưa vào vận hành.

Kỳ thực tập giúp tôi tự tin hơn khi xử lý các vấn đề kỹ thuật mà trước đó mình chưa quen, nhưng đồng thời cũng cho tôi thấy khá rõ những điểm mình còn thiếu. Trong thời gian tới, tôi muốn tiếp tục phát triển nền tảng machine learning và kỹ năng evaluation, đồng thời nâng cao khả năng triển khai để có thể tham gia trọn vẹn hơn vào quá trình đưa một ý tưởng AI từ giai đoạn thử nghiệm thành một ứng dụng hoàn chỉnh, ổn định và thực sự sử dụng được.