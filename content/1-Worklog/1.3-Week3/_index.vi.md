---
title: "Nhật ký Tuần 3"
date: "2025-09-22T09:00:00+07:00"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu Tuần 3:
* Hiểu các dịch vụ cốt lõi về Tính toán (EC2) và Lưu trữ (EBS, S3).
* Học cách quản lý tính bền vững của dữ liệu.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **Chuyên sâu EC2:**<br>- Các loại Instance (T3, M5...).<br>- Lựa chọn AMI.<br>- Key Pairs & kịch bản User Data. | 22/09/2025 | 22/09/2025 | [AWS EC2 Docs](https://docs.aws.amazon.com/ec2/) |
| 2 | **EBS (Block Storage):**<br>- Các loại Volume (gp3, io2).<br>- Gắn/Gỡ volume.<br>- Snapshot & Lifecycle Manager. | 23/09/2025 | 23/09/2025 | |
| 3 | **S3 (Object Storage):**<br>- Buckets & Objects.<br>- Các lớp lưu trữ (Standard, IA, Glacier).<br>- Versioning & Bucket Policies. | 24/09/2025 | 24/09/2025 | |
| 4 | **Thực hành Lab:**<br>- Khởi chạy EC2 dùng User Data cài Apache.<br>- Tạo S3 bucket host file `index.html`. | 25/09/2025 | 25/09/2025 | |
| 5 | **Ôn tập:**<br>- Phân biệt khi nào dùng EBS so với S3. | 26/09/2025 | 26/09/2025 | |

### 🧠 Kiến thức mở rộng: Instance Store vs EBS
Tôi phát hiện ra một số loại EC2 đi kèm với "Instance Store" (Lưu trữ tạm thời). Ổ cứng này gắn trực tiếp vật lý vào máy chủ nên tốc độ rất nhanh. **Tuy nhiên**, nếu tôi Stop hoặc Terminate máy ảo, **mọi dữ liệu trên Instance Store sẽ mất sạch**. Đó là lý do tại sao với Database cho dự án sắp tới, tôi bắt buộc phải dùng **EBS** vì dữ liệu trên EBS tồn tại độc lập với vòng đời của EC2.

### Thành tựu đạt được:
* Đã khởi chạy web server sử dụng kỹ thuật User Data bootstrapping (tự động cài Apache).
* Quản lý lưu trữ bền vững với EBS volume và thực hành khôi phục dữ liệu từ Snapshot.
* Hiểu rõ sự khác biệt giữa lưu trữ dạng Block (EBS) và dạng Object (S3).