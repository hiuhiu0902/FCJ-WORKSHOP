---
title : "Tạo VPC và Subnets"
date :  "2025-12-10T10:00:00+07:00"
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

#### 1. Khởi tạo VPC
Bước đầu tiên, chúng ta sẽ tạo một mạng riêng ảo (VPC) để chứa toàn bộ tài nguyên của dự án.

1.  Mở [Amazon VPC console](https://console.aws.amazon.com/vpc/).
2.  Ở thanh điều hướng bên trái, chọn **Your VPCs**.
3.  Click **Create VPC**.

![create vpc button](/images/5-Workshop/5.3-Network-Setup/5.3.1/create-vpc-button.png)

4.  Trong màn hình **Create VPC**, điền các thông số:
    * **Resources to create:** Chọn **VPC only** (Chúng ta sẽ tạo subnet thủ công để hiểu rõ cấu trúc).
    * **Name tag:** `3-tier-vpc`
    * **IPv4 CIDR block:** `10.75.0.0/16` (Cung cấp 65,536 địa chỉ IP).
    * **Tenancy:** Default.
    * Click **Create VPC**.

![vpc config](/images/5-Workshop/5.3-Network-Setup/5.3.1/vpc-config.png)

5.  Sau khi tạo xong, chọn **Actions** -> **Edit VPC settings**.
6.  Tích chọn **Enable DNS hostnames** (Để các server có thể phân giải tên miền AWS).
7.  Click **Save changes**.

---

#### 2. Tạo Subnets (Chia mạng con)
Chúng ta sẽ tạo tổng cộng **8 Subnets** chia đều cho 2 Availability Zones (AZ) tại Singapore (`ap-southeast-1`).

Vào menu **Subnets** -> **Create subnet**. Chọn VPC `3-tier-vpc` vừa tạo.

##### A. Tạo Subnets cho Zone A (ap-southeast-1a)
Tại phần **Subnet settings**, lần lượt nhấn "Add new subnet" để tạo 4 subnets sau:

| Subnet name | Availability Zone | IPv4 CIDR block | Mục đích |
| :--- | :--- | :--- | :--- |
| `Public-Subnet-1a` | `ap-southeast-1a` | `10.75.1.0/24` | Chứa NAT Gateway, Bastion |
| `Web-Private-Subnet-1a` | `ap-southeast-1a` | `10.75.3.0/24` | Dự phòng cho Web Tier |
| `App-Private-Subnet-1a` | `ap-southeast-1a` | `10.75.5.0/24` | Chứa App Server (Java) |
| `DB-Private-Subnet-1a` | `ap-southeast-1a` | `10.75.7.0/24` | Chứa RDS Primary |

![subnet zone a](/images/5-Workshop/5.3-Network-Setup/subnet-creation-a.png)

##### B. Tạo Subnets cho Zone B (ap-southeast-1b)
Tiếp tục tạo thêm 4 subnets cho Zone B để đảm bảo tính sẵn sàng cao (HA):

| Subnet name | Availability Zone | IPv4 CIDR block | Mục đích |
| :--- | :--- | :--- | :--- |
| `Public-Subnet-1b` | `ap-southeast-1b` | `10.75.2.0/24` | Chứa NAT Gateway 2 |
| `Web-Private-Subnet-1b` | `ap-southeast-1b` | `10.75.4.0/24` | Dự phòng HA |
| `App-Private-Subnet-1b` | `ap-southeast-1b` | `10.75.6.0/24` | Chứa App Server HA |
| `DB-Private-Subnet-1b` | `ap-southeast-1b` | `10.75.8.0/24` | Chứa RDS Standby |

![subnet zone b](/images/5-Workshop/5.3-Network-Setup/subnet-creation-b.png)

8.  Click **Create subnet**.

{{% notice success %}}
Kiểm tra lại: Bạn sẽ thấy danh sách 8 subnets với trạng thái **Available**. Hãy chắc chắn cột **Available IPv4 CIDR** khớp với bảng trên.
{{% /notice %}}