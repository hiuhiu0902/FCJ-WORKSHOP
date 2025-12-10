---
title : "Cấu hình NAT & Route Tables"
date :  "2025-12-10T10:00:00+07:00"
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

#### 1. Tạo Internet Gateway (IGW)
Để VPC có thể giao tiếp với Internet công cộng, chúng ta cần một cổng Internet Gateway.

1.  Tại giao diện **VPC Console**, chọn **Internet gateways** ở menu trái.
2.  Click **Create internet gateway**.
3.  **Name tag:** `3-tier-igw`.
4.  Click **Create internet gateway**.

![create igw](/images/5-Workshop/5.3-Network-Setup/5.3.2/create-igw.png)

5.  Sau khi tạo, chọn **Actions** -> **Attach to VPC**.
6.  Chọn VPC `3-tier-vpc` và nhấn **Attach internet gateway**.

![attach igw](/images/5-Workshop/5.3-Network-Setup/5.3.2/attach-igw.png)

#### 2. Tạo NAT Gateways (High Availability)
Để các server trong Private Subnet (như App Server cài Java) có thể kết nối ra ngoài Internet (để tải thư viện) mà không cho phép Internet kết nối ngược vào, chúng ta dùng NAT Gateway. Chúng ta sẽ tạo 2 NAT Gateway cho 2 Zone để đảm bảo HA.

1.  Vào menu **NAT gateways** -> **Create NAT gateway**.

**Tạo NAT Gateway 1 (Zone A):**
* **Name:** `nat-gw-1a`
* **Subnet:** Chọn `Public-Subnet-1a`.
* **Elastic IP allocation ID:** Nhấn nút **Allocate Elastic IP** để tạo mới một IP tĩnh.
* Click **Create NAT gateway**.

![create nat 1](/images/5-Workshop/5.3-Network-Setup/5.3.2/create-nat-1.png)

**Tạo NAT Gateway 2 (Zone B):**
* Lặp lại các bước trên.
* **Name:** `nat-gw-1b`
* **Subnet:** Chọn `Public-Subnet-1b`.
* **Elastic IP allocation ID:** Nhấn **Allocate Elastic IP**.
* Click **Create NAT gateway**.

{{% notice note %}}
Quá trình tạo NAT Gateway có thể mất vài phút để chuyển sang trạng thái **Available**. Hãy chờ một chút trước khi sang bước tiếp theo.
{{% /notice %}}

#### 3. Cấu hình Route Tables (Bảng định tuyến)
Bây giờ chúng ta sẽ điều hướng luồng đi của mạng.

Vào menu **Route tables** -> **Create route table**.

##### A. Tạo Public Route Table
Dành cho các subnet công khai (Public Subnets).

1.  **Name:** `Public-RT`.
2.  **VPC:** `3-tier-vpc`.
3.  Click **Create**.
4.  Trong tab **Routes**, chọn **Edit routes** -> **Add route**:
    * **Destination:** `0.0.0.0/0` (Toàn bộ Internet).
    * **Target:** Chọn `Internet Gateway` -> `3-tier-igw`.
    * Click **Save changes**.
5.  Trong tab **Subnet associations**, chọn **Edit subnet associations**:
    * Chọn `Public-Subnet-1a` và `Public-Subnet-1b`.
    * Click **Save associations**.

![public route table](/images/5-Workshop/5.3-Network-Setup/5.3.2/public-rt.png)

##### B. Tạo Private Route Table cho Zone A
Dành cho các subnet ẩn (Private Subnets) ở Zone A, đi qua NAT A.

1.  Tạo Route Table mới tên: `Private-RT-1a`.
2.  **VPC:** `3-tier-vpc`.
3.  **Edit routes** -> **Add route**:
    * **Destination:** `0.0.0.0/0`.
    * **Target:** Chọn `NAT Gateway` -> `nat-gw-1a`.
4.  **Edit subnet associations**:
    * Chọn 3 subnets private thuộc Zone A: `Web-Private-Subnet-1a`, `App-Private-Subnet-1a`, `DB-Private-Subnet-1a`.
    * Click **Save**.

![private rt 1a](/images/5-Workshop/5.3-Network-Setup/5.3.2/private-rt-a.png)

##### C. Tạo Private Route Table cho Zone B
Dành cho các subnet ẩn ở Zone B, đi qua NAT B.

1.  Tạo Route Table mới tên: `Private-RT-1b`.
2.  **VPC:** `3-tier-vpc`.
3.  **Edit routes** -> **Add route**:
    * **Destination:** `0.0.0.0/0`.
    * **Target:** Chọn `NAT Gateway` -> `nat-gw-1b`.
4.  **Edit subnet associations**:
    * Chọn 3 subnets private thuộc Zone B: `Web-Private-Subnet-1b`, `App-Private-Subnet-1b`, `DB-Private-Subnet-1b`.
    * Click **Save**.

{{% notice success %}}
Chúc mừng! Bạn đã hoàn thành việc thiết lập hạ tầng mạng với khả năng dự phòng cao (High Availability). Mọi server nằm trong Private Subnet giờ đây đã có thể tải cập nhật bảo mật một cách an toàn.
{{% /notice %}}