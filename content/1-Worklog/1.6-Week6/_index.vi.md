---
title: "Nhật ký Tuần 6"
date: "2025-10-13T09:00:00+07:00"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Mục tiêu Tuần 6:
* Học về dịch vụ cơ sở dữ liệu quản lý (RDS).
* Hiểu Multi-AZ và Read Replicas.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **Cơ bản RDS:**<br>- DB Engines.<br>- Triển khai Multi-AZ. | 13/10/2025 | 13/10/2025 | |
| 2 | **Hiệu năng RDS:**<br>- Read Replicas.<br>- RDS Security Groups. | 14/10/2025 | 14/10/2025 | |
| 3 | **ElastiCache:**<br>- Cơ bản về Redis.<br>- Chiến lược Caching. | 15/10/2025 | 15/10/2025 | |
| 4 | **Thực hành:**<br>- Khởi chạy RDS (MySQL).<br>- Kết nối từ EC2. | 16/10/2025 | 16/10/2025 | |
| 5 | **Ôn tập:**<br>- Chiến lược sao lưu (Backup). | 17/10/2025 | 17/10/2025 | |

### 🧠 Kiến thức mở rộng: Multi-AZ và Read Replica
Rất quan trọng để không nhầm lẫn hai khái niệm này:
* **Multi-AZ** dùng cho *Tính sẵn sàng cao (HA)*. Dữ liệu được sao chép **Đồng bộ (Synchronous)**. Thường ta không thể đọc dữ liệu từ máy phụ (Standby) trừ khi máy chính bị sập.
* **Read Replica** dùng cho *Hiệu năng*. Dữ liệu được sao chép **Bất đồng bộ (Asynchronous)** (có độ trễ nhỏ). Ta có thể chia tải các câu lệnh truy vấn nặng sang đây để giảm tải cho DB chính.

### Thành tựu đạt được:
* Đã triển khai Database MySQL được AWS quản lý (không cần lo việc vá lỗi HĐH).
* Kết nối thành công từ Web Server vào Database dùng chuỗi Security Group.