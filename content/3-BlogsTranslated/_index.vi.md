---
title: "Translated Blogs"
date: "2025-09-09T19:53:52+07:00"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section will list and introduce the blogs you have translated.

### [Blog 1 - Karrot đã xây dựng nền tảng tính năng (feature platform) trên AWS như thế nào, Phần 1: Động lực và phục vụ tính năng (feature serving)](3.1-Blog1/)
Bài blog này phân tích lý do Karrot, cộng đồng địa phương hàng đầu Hàn Quốc, cần xây dựng một nền tảng tính năng (feature platform) mới. Bài viết đi sâu vào các vấn đề của kiến trúc cũ (phụ thuộc vào máy chủ, khó mở rộng) và trình bày kiến trúc giải pháp mới. Phần này tập trung vào thành phần **Feature Serving**, giải thích cách họ sử dụng multi-level cache (Local Cache, Amazon ElastiCache, Amazon DynamoDB) để đạt được độ trễ thấp cho hệ thống đề xuất ML của mình.

### [Blog 2 - Karrot đã xây dựng nền tảng tính năng (feature platform) trên AWS như thế nào, Phần 2: Nhập tính năng (Feature ingestion)](3.2-Blog2/)
Phần tiếp theo này đi sâu vào thành phần **Feature Ingestion** của nền tảng. Bài viết giải thích chi tiết hai quy trình: **Stream Ingestion** (thu thập dữ liệu thời gian thực) sử dụng Amazon MSK và Amazon EKS, và **Batch Ingestion** (xử lý dữ liệu theo lô) sử dụng AWS Batch và AWS Fargate. Bài blog cũng chia sẻ các kết quả đạt được, bao gồm khả năng xử lý lưu lượng truy cập lớn và tối ưu hóa chi phí nhờ các dịch vụ AWS.

### [Blog 3 - Triển khai LLMs trên Amazon EKS bằng cách sử dụng vLLM Deep Learning Containers](3.3-Blog3/)
Bài blog này cung cấp một hướng dẫn kỹ thuật chi tiết về cách triển khai các Large Language Models (LLMs) hiệu suất cao trên Amazon EKS. Giải pháp sử dụng vLLM Deep Learning Containers (DLCs) để tối ưu hóa, kết hợp với các instance P4d (GPU A100), Elastic Fabric Adapter (EFA) cho mạng tốc độ cao, và Amazon FSx for Lustre để lưu trữ model weights. Bài viết hướng dẫn từng bước thiết lập môi trường, từ tạo cluster EKS, cấu hình node group, đến triển khai và kiểm tra API.