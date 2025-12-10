---
title: "Khởi tạo Database (RDS)"
date: 2023-10-25
weight: 6
---

# Khởi tạo Amazon RDS

Trong phần này, chúng ta sẽ khởi tạo cơ sở dữ liệu (Database) sử dụng dịch vụ **Amazon RDS**. Database này sẽ được đặt trong mạng con riêng tư (Private Subnet) và chỉ cho phép App Server truy cập.

## Kiến trúc

* **Service:** Amazon RDS (Relational Database Service).
* **Engine:** MySQL hoặc PostgreSQL (tùy theo script của bạn).
* **Network:** Đặt trong `DB Subnet Group` (đã tạo ở bước 5.4.2).
* **Security:** Sử dụng `DB-Security-Group` (đã tạo ở bước 5.4.1) để cho phép port 3306/5432 từ App Server.

---

## Các bước thực hiện

### 1. Cấu hình Engine
* Truy cập **AWS Console** -> Tìm kiếm **RDS**.
* Chọn **Create database**.
* **Choose a database creation method:** Standard create.
* **Engine options:** Chọn `MySQL` (hoặc PostgreSQL tùy project).
* **Templates:** Chọn `Free tier` (hoặc Dev/Test) để tiết kiệm chi phí cho bài lab.

### 2. Cấu hình Settings
* **DB instance identifier:** `workshop-db`
* **Master username:** `admin`
* **Master password:** Nhập mật khẩu bảo mật (Lưu ý ghi nhớ để cấu hình vào App Server sau này).

### 3. Cấu hình Instance & Storage
* **DB instance class:** `db.t3.micro` (Dành cho Free tier).
* **Storage:** Giữ mặc định (thường là gp2 hoặc gp3, 20GB).

### 4. Cấu hình Connectivity (Quan trọng)
Đây là bước kết nối với các tài nguyên đã tạo ở phần 5.3 và 5.4:

* **VPC:** Chọn VPC đã tạo (ví dụ: `workshop-vpc`).
* **DB Subnet Group:** Chọn `workshop-db-subnet-group` (Đã tạo ở bài 5.4.2).
* **Public access:** Chọn **No** (Vì DB nằm trong Private Subnet).
* **VPC security group:**
    * Chọn **Choose existing**.
    * Xóa `default`.
    * Chọn `workshop-db-sg` (Đã tạo ở bài 5.4.1).
* **Availability Zone:** Chọn `No preference` hoặc chỉ định Zone cụ thể.

### 5. Xác thực (Authentication)
* Chọn **Password authentication**.

### 6. Tạo Database
* Cuộn xuống cuối cùng và nhấn **Create database**.
* Quá trình khởi tạo có thể mất từ 5-10 phút.

---

## Kiểm tra kết quả

Sau khi trạng thái chuyển sang **Available**:
1.  Bấm vào tên DB (`workshop-db`).
2.  Tại tab **Connectivity & security**, copy giá trị **Endpoint** (Ví dụ: `workshop-db.xxxx.us-east-1.rds.amazonaws.com`).
3.  Đây là địa chỉ host để bạn cấu hình vào App Server ở bài trước.