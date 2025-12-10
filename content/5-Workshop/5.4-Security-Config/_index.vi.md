---
title : "Cấu hình bảo mật (Security)"
date : "2023-10-25T00:00:00+07:00"
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

# Cấu hình bảo mật (Security Configuration)

Trong kiến trúc **N-Tier**, bảo mật là yếu tố sống còn. Trước khi triển khai các tài nguyên tính toán (EC2, RDS), chúng ta cần thiết lập các hàng rào mạng để kiểm soát chặt chẽ luồng dữ liệu ra vào (Ingress/Egress).

Tại phần này, chúng ta sẽ thiết lập hệ thống phòng thủ hai lớp:

1.  **Security Groups (Tường lửa ảo):**
    * Chúng ta sẽ không mở cổng tràn lan. Thay vào đó, chúng ta áp dụng kỹ thuật **Security Group Chaining** (Xâu chuỗi).
    * Quy tắc: Tầng sau chỉ chấp nhận traffic đến từ tầng liền trước nó (Ví dụ: Database chỉ nhận kết nối từ Backend App, tuyệt đối không nhận từ Internet hay Frontend).
    * **Luồng dữ liệu:** `Internet` -> `Public ALB` -> `Frontend` -> `Internal ALB` -> `Backend` -> `Database`.

2.  **DB Subnet Group:**
    * Đây là cấu hình bắt buộc để khởi tạo **Amazon RDS**.
    * Nó gom nhóm các **Private Subnets** (nơi chứa Database) lại với nhau, giúp RDS biết được phạm vi mạng nào an toàn để đặt dữ liệu và hỗ trợ cơ chế dự phòng đa vùng (Multi-AZ).

### Nội dung thực hành
1. [Cấu hình Security Groups](5.4.1-Security-Groups/)
2. [Tạo DB Subnet Group](5.4.2-DB-Subnet-Group/)