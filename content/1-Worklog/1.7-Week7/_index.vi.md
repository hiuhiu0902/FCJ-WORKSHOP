---
title: "Nhật ký Tuần 7"
date: "2025-10-20T09:00:00+07:00"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu Tuần 7:
* Bảo mật traffic chiều đi từ mạng riêng (Private network).
* Quản lý secrets và tường lửa WAF.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **NAT Gateway:**<br>- Cho phép Private Subnet truy cập internet. | 20/10/2025 | 20/10/2025 | |
| 2 | **Secrets Manager:**<br>- Lưu mật khẩu DB.<br>- Xoay vòng mật khẩu (Rotation). | 21/10/2025 | 21/10/2025 | |
| 3 | **WAF:**<br>- Bảo vệ cơ bản chống SQL injection. | 22/10/2025 | 22/10/2025 | |
| 4 | **Thực hành:**<br>- Tạo NAT Gateway.<br>- Kiểm tra Private EC2 vào mạng. | 23/10/2025 | 23/10/2025 | |
| 5 | **Ôn tập:**<br>- Phân tích chi phí NAT. | 24/10/2025 | 24/10/2025 | |

### 🧠 Kiến thức mở rộng: Chi phí của NAT Gateway
Khi triển khai NAT Gateway, tôi nhận ra đây là một trong những "chi phí ngầm" lớn trên AWS. Nó không chỉ tính tiền theo giờ (~$0.045/h) mà còn tính tiền **Xử lý dữ liệu** ($0.045/GB). Với dự án Game Card, nếu server tải nhiều bản update nặng, chi phí sẽ rất cao. Trong tương lai, tôi sẽ cân nhắc dùng **VPC Endpoint** để truy cập các dịch vụ AWS (như S3) để né phí qua NAT.

### Thành tựu đạt được:
* Đã cấu hình internet chiều đi an toàn dùng NAT Gateway.
* Bảo mật thông tin DB bằng Secrets Manager thay vì lưu trong code.