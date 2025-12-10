---
title : "Khởi tạo Bastion Host"
date : "2023-10-25T00:00:00+07:00"
weight : 1
chapter : false
pre : " <b> 5.5.1 </b> "
---

Bastion Host (hay Jump Server) là máy chủ duy nhất trong hệ thống được phép mở cổng SSH (22) ra Internet.

### Các bước thực hiện

1. Truy cập **EC2 Console** -> **Instances** -> **Launch instances**.
2. **Name:** `Bastion-Host`.
3. **OS Image:** Amazon Linux 2023 AMI (hoặc Ubuntu tùy quen thuộc).
4. **Instance type:** `t2.micro` hoặc `t3.micro` (Free tier).
5. **Key pair:** Chọn `workshop-key` (đã tạo ở bài 5.2).
6. **Network settings (Quan trọng):**
    * **VPC:** Chọn `3-tier-vpc`.
    * **Subnet:** Chọn `Public-Subnet-1a` (Bắt buộc phải là Public).
    * **Auto-assign public IP:** **Enable**.
    * **Security Group:** Chọn `Bastion-SG` (Đã tạo ở 5.4.1).
7. Nhấn **Launch instance**.

### Kiểm tra kết nối
Sử dụng Putty (Windows) hoặc Terminal (Mac/Linux) để SSH vào Bastion Host thông qua Public IP vừa được cấp.

![Bastion SSH Success](/images/5-Workshop/5.5-Compute-Setup/bastion-ssh-success.png)
*User mặc định của Amazon Linux là `ec2-user`, của Ubuntu là `ubuntu`.*