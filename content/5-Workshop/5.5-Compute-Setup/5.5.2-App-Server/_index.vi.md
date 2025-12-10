---
title : "Thiết lập App Server & AMI"
date : "2023-10-25T00:00:00+07:00"
weight : 2
chapter : false
pre : " <b> 5.5.2 </b> "
---

Chúng ta sẽ tạo một EC2 Instance đóng vai trò là "Template" (Mẫu). Sau khi cài đặt xong mọi thứ, ta sẽ tạo AMI từ nó.

### 1. Khởi tạo & Cài đặt
1. Access **EC2 Console** -> **Launch instances**.
2. **Name:** `App-Server-Template`.
3. **OS Image:** Amazon Linux 2023.
4. **Instance type:** `t3.micro`.
5. **Key pair:** Select `workshop-key`.
6. **Network settings (Important):**
    * **VPC:** `3-tier-vpc`.
    * **Subnet:** `App-Private-Subnet-1a` (Must be Private - No Public IP).
    * **Auto-assign public IP:** Disable.
    * **Security Group:** Select `Backend-SG` (Allows port 8080 from ALB and 22 from Bastion).
7. **Advanced details -> IAM instance profile:** Select `SSM-Role` (Optional, for Systems Manager access).
8. Click **Launch instance**.

### 2. Kiểm tra môi trường (Kết quả)
Sau khi SSH vào máy Private (thông qua Bastion) và cài đặt Java/MySQL Client, hãy chạy lệnh kiểm tra phiên bản Java và kết nối thử tới RDS.

Kết quả mong đợi:

![App Server Setup Success](/images/5-Workshop/5.5-Compute-Setup/app-server-success.png)
*(Hình ảnh minh họa: Terminal hiển thị `java -version` và dấu nhắc lệnh `MySQL >` khi kết nối thành công tới RDS)*

### 3. Tạo Golden AMI
Sau khi đảm bảo App Server chạy ổn định, hãy tạo Image (`v1-backend-ami`) để dùng cho bước tiếp theo.