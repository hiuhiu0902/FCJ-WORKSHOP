---
title : "Các bước chuẩn bị"
date :  "2025-09-10T10:00:00+07:00"
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. Chọn Region
Trong bài thực hành này, chúng ta sẽ sử dụng Region **Singapore (ap-southeast-1)**.
Để đảm bảo tính sẵn sàng cao (High Availability), các tài nguyên sẽ được phân bố trên 2 Availability Zones (AZs) là:
* `ap-southeast-1a`
* `ap-southeast-1b`

Hãy đảm bảo bạn đã chọn đúng Region ở góc trên bên phải màn hình console.

![Chọn Region Singapore](/images/5-Workshop/5.2-Prerequiste/select-region.png)

#### 2. Tạo EC2 Key Pair
Để kết nối SSH vào các EC2 instance (Web Server, App Server, Bastion Host), bạn cần khởi tạo một Key Pair.

1.  Truy cập **EC2 Console** -> Menu trái chọn **Key Pairs**.
2.  Nhấn **Create key pair**.
3.  Nhập thông tin:
    * **Name:** `workshop-key`
    * **Key pair type:** `RSA`
    * **Private key file format:**
        * Chọn `.ppk` nếu bạn dùng **Windows (Putty)**.
        * Chọn `.pem` nếu bạn dùng **MacOS/Linux** hoặc Windows PowerShell.
4.  Nhấn **Create key pair** và lưu file key về máy tính (Đừng làm mất file này).

![Tạo Key Pair](/images/5-Workshop/5.2-Prerequiste/create-keypair.png)

#### 3. Cấu hình IAM Permissions (Optional)
Tạo Role cho phép EC2 truy cập Systems Manager (để kết nối không cần mở port 22).
* Vào **IAM** -> **Roles** -> **Create role**.
* Service: **EC2**.
* Policy: `AmazonSSMManagedInstanceCore`.
* Role name: `SSM-Role`.
Nếu bạn đang sử dụng tài khoản thực hành được cấp phát hạn chế (không phải tài khoản Root/Admin), hãy đảm bảo User của bạn được gắn Policy dưới đây để có đủ quyền tạo VPC, EC2, RDS và Auto Scaling:

*(Lưu ý: Nếu bạn dùng tài khoản cá nhân có quyền AdministratorAccess, bạn có thể bỏ qua bước này)*

<details>
<summary><b>Click để xem chi tiết JSON Policy</b></summary>

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "cloudformation:*",
                "cloudwatch:*",
                "ec2:*",
                "elasticloadbalancing:*",
                "autoscaling:*",
                "rds:*",
                "s3:*",
                "iam:CreateServiceLinkedRole",
                "iam:ListRoles",
                "iam:PassRole",
                "secretsmanager:*"
            ],
            "Resource": "*"
        }
    ]
}