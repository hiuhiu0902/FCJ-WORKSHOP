---
title: "Nhật ký Tuần 8"
date: "2025-10-27T09:00:00+07:00"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu Tuần 8:
* Chốt kiến trúc cho dự án cuối khóa.
* Tài liệu hóa dải IP và quy tắc tường lửa.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **Vẽ sơ đồ:**<br>- Vẽ kiến trúc HA đầy đủ (Multi-AZ). | 27/10/2025 | 27/10/2025 | [Draw.io](https://draw.io) |
| 2 | **Quy hoạch IP:**<br>- Định nghĩa CIDR blocks (10.0.0.0/16). | 28/10/2025 | 28/10/2025 | |
| 3 | **Quy hoạch Bảo mật:**<br>- Viết quy tắc Security Group. | 29/10/2025 | 29/10/2025 | |
| 4 | **Review:**<br>- Nhờ người khác review thiết kế. | 30/10/2025 | 30/10/2025 | |
| 5 | **Chuẩn bị:**<br>- Danh sách kiểm tra triển khai (Checklist). | 31/10/2025 | 31/10/2025 | |

### 🧠 Kiến thức mở rộng: Giảm thiểu "Phạm vi ảnh hưởng" (Blast Radius)
Trong thiết kế của mình, tôi phân tán tài nguyên ra 2 AZs. Điều này nhằm giảm thiểu **"Blast Radius"**. Nếu thảm họa (cháy, lũ lụt, mất điện) xảy ra tại một trung tâm dữ liệu (AZ 1), ứng dụng của tôi tại AZ 2 vẫn hoạt động bình thường. Đây là khái niệm cốt lõi của tính Tin cậy (Reliability) trong Well-Architected Framework.

### Thành tựu đạt được:
* Hoàn thành sơ đồ kiến trúc chi tiết cho dự án Game Card Platform.
* Định nghĩa các quy tắc security group chặt chẽ (Nguyên tắc đặc quyền tối thiểu).