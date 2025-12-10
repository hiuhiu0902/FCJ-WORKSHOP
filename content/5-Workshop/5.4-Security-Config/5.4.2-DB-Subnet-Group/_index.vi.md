---
title : "Tạo DB Subnet Group"
date : "2023-10-25T00:00:00+07:00"
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

Trong sơ đồ kiến trúc, Database được triển khai theo mô hình **Multi-AZ** (Primary ở một Subnet và Standby ở Subnet khác) để đảm bảo tính sẵn sàng cao (High Availability).

Để RDS biết được nó được phép nằm ở những Subnet nào, chúng ta cần tạo **DB Subnet Group**.

### Các bước thực hiện

1. Truy cập **Amazon RDS Console** -> **Subnet groups**.
2. Nhấn **Create DB subnet group**.

### Cấu hình chi tiết

* **Name**: `workshop-db-subnet-group`
* **Description**: `Subnet group for Multi-AZ Architecture`
* **VPC**: Chọn VPC của dự án.

### Chọn Subnets (Quan trọng)

Theo sơ đồ, Database nằm ở lớp mạng sâu nhất (Deepest Private Subnets). Bạn cần chọn đúng 2 Subnet Private dành riêng cho DB:

1. **Availability Zones**: Chọn 2 AZs (ví dụ: `ap-southeast-1a` và `ap-southeast-1b`).
2. **Subnets**:
    * Chọn Subnet ID của **Private Subnet 3** (Zone A).
    * Chọn Subnet ID của **Private Subnet 4** (Zone B).

*(Lưu ý: Không chọn nhầm Subnet của Frontend hay Backend)*.

3. Nhấn **Create**.

Sau bước này, khi tạo RDS, bạn chỉ cần chọn `workshop-db-subnet-group`, AWS sẽ tự động phân phối Database Primary và Standby vào đúng vị trí theo sơ đồ.