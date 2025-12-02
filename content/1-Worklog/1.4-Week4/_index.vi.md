---
title: "Nhật ký Tuần 4"
date: "2025-09-29T09:00:00+07:00"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu Tuần 4:
* Áp dụng toàn bộ kiến thức Tháng 1.
* Tự tay xây dựng môi trường hoạt động từ con số 0.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **Lập kế hoạch:**<br>- Vẽ sơ đồ mạng cho ứng dụng 2-tier (Public/Private). | 29/09/2025 | 29/09/2025 | |
| 2 | **Xây dựng Mạng:**<br>- Tạo VPC, 1 Public Subnet, 1 Private Subnet.<br>- Cấu hình Route Tables. | 30/09/2025 | 30/09/2025 | |
| 3 | **Xây dựng Compute:**<br>- Chạy EC2 ở Public (Web Server).<br>- Chạy EC2 ở Private (Backend). | 01/10/2025 | 01/10/2025 | |
| 4 | **Kiểm tra truy cập:**<br>- SSH vào Public EC2.<br>- Thử SSH từ Public sang Private (Jumpbox). | 02/10/2025 | 02/10/2025 | |
| 5 | **Dọn dẹp:**<br>- Terminate instances, xóa NATs/Gateways. | 03/10/2025 | 03/10/2025 | |

### 🧠 Kiến thức mở rộng: Mô hình Bastion Host
Khi cố gắng truy cập vào EC2 ở Private Subnet, tôi không thể kết nối vì nó không có Public IP. Tôi đã học được mô hình **Bastion Host (Máy trạm trung gian)**.
* Nó hoạt động như một cổng an ninh. Tôi SSH vào Bastion (ở Public Subnet) trước, từ đó mới SSH tiếp vào máy Private.
* *Mẹo bảo mật:* Tôi chỉ nên cho phép Bastion nhận kết nối SSH từ đúng IP nhà mạng của tôi, tuyệt đối không mở `0.0.0.0/0`.

### Thành tựu đạt được:
* Tự xây dựng hoàn chỉnh môi trường mạng bằng tay (không dùng VPC Wizard).
* Chứng minh khái niệm "Bastion Host" bằng cách truy cập instance trong mạng riêng an toàn.