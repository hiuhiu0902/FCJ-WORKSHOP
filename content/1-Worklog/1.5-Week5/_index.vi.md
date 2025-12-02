---
title: "Nhật ký Tuần 5"
date: "2025-10-06T09:00:00+07:00"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu Tuần 5:
* Nắm vững Scaling và Load Balancing.
* Hiểu về ứng dụng "Tự phục hồi" (Self-Healing).

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **ELB:**<br>- Khái niệm ALB, Target Groups, Health Checks. | 06/10/2025 | 06/10/2025 | |
| 2 | **ASG:**<br>- Launch Templates.<br>- Scaling Policies (theo CPU vs Lịch trình). | 07/10/2025 | 07/10/2025 | |
| 3 | **Thực hành:**<br>- Đặt 2 EC2 sau ALB.<br>- Kiểm tra phân phối traffic. | 08/10/2025 | 08/10/2025 | |
| 4 | **Thực hành:**<br>- Tạo ASG.<br>- Stress test CPU để kích hoạt scale-out. | 09/10/2025 | 09/10/2025 | |
| 5 | **Ôn tập:**<br>- Khái niệm ứng dụng "Stateless". | 10/10/2025 | 10/10/2025 | |

### 🧠 Kiến thức mở rộng: Connection Draining
Tôi đã học được một cài đặt tên là **"Deregistration Delay"** (Connection Draining) trên Target Group. Khi một EC2 instance bị hủy (scale-in), ALB sẽ ngừng gửi request *mới* vào nó nhưng vẫn giữ kết nối trong vài phút (mặc định 300s) để các request *đang xử lý dở* có thể hoàn tất. Điều này giúp người dùng không bị lỗi trang web giữa chừng khi hệ thống đang thu nhỏ quy mô.

### Thành tựu đạt được:
* Đã cấu hình Load Balancer phân phối tải trên 2 Availability Zones.
* Tạo Auto Scaling Group tự động thêm server khi CPU vượt quá 50%.