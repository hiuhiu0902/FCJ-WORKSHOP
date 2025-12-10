---
title : "Cấu hình Security Group (N-Tier)"
date : "2023-10-25T00:00:00+07:00"
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

Dựa trên sơ đồ kiến trúc, chúng ta cần thiết lập hệ thống tường lửa nhiều lớp (Security Group Chaining) để đảm bảo traffic đi đúng luồng từ ngoài vào trong: **Internet -> Public ALB -> Frontend -> Internal ALB -> Backend -> Database**.

Chúng ta sẽ tạo lần lượt các Security Group (SG) sau trong VPC:

### 1. SG cho Bastion Host (Quản trị)
Dùng để SSH vào các server bên trong để sửa lỗi.
* **Name:** `Bastion-SG`
* **Inbound:**
    * Type: SSH (22) | Source: `My IP` (IP máy tính của bạn).

### 2. SG cho Public Load Balancer (External ALB)
Đây là ALB số (5) trong sơ đồ, nhận traffic từ CloudFront/User.
* **Name:** `Public-ALB-SG`
* **Inbound:**
    * Type: HTTP (80) | Source: `0.0.0.0/0` (Internet).
    * Type: HTTPS (443) | Source: `0.0.0.0/0`.

### 3. SG cho Frontend Instances
Đây là cụm Auto Scaling Group (Frontend) trong sơ đồ. Chỉ nhận traffic từ Public ALB.
* **Name:** `Frontend-SG`
* **Inbound:**
    * Type: HTTP (80) | Source: `Public-ALB-SG` (Chỉ cho phép từ ALB ngoài).
    * Type: SSH (22) | Source: `Bastion-SG` (Quản trị).

### 4. SG cho Internal Load Balancer
Đây là ALB nằm giữa Frontend và Backend. Chỉ nhận traffic từ Frontend.
* **Name:** `Internal-ALB-SG`
* **Inbound:**
    * Type: HTTP (80) | Source: `Frontend-SG` (Chỉ Frontend mới gọi được vào đây).

### 5. SG cho Backend Instances (API)
Đây là cụm Auto Scaling Group (Backend) xử lý logic nghiệp vụ.
* **Name:** `Backend-SG`
* **Inbound:**
    * Type: Custom TCP (8080/AppPort) | Source: `Internal-ALB-SG` (Chỉ nhận từ Internal ALB).
    * Type: SSH (22) | Source: `Bastion-SG`.

### 6. SG cho Database (RDS)
Lớp cuối cùng chứa dữ liệu (Primary & Standby).
* **Name:** `Database-SG`
* **Inbound:**
    * Type: MySQL/Aurora (3306) | Source: `Backend-SG` (Tuyệt đối chỉ cho Backend truy cập).


{{% notice note %}}
**Lưu ý:** Trong sơ đồ có WAF và CloudFront, ở bước Security Group này chúng ta mở port 80/443 cho Public ALB. Khi triển khai WAF thực tế, bạn có thể giới hạn `Public-ALB-SG` chỉ nhận traffic từ dải IP của CloudFront (Managed Prefix List) để bảo mật tuyệt đối.
{{% /notice %}}