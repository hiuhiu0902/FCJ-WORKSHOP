---
title : "Cấu hình Load Balancers"
date : "2023-10-25T00:00:00+07:00"
weight : 7
chapter : false
pre : " <b> 5.7 </b> "
---

Hệ thống N-Tier yêu cầu 2 Application Load Balancer (ALB) riêng biệt để định tuyến traffic.

### Phần A: Tạo Target Groups
Trước khi tạo ALB, ta cần tạo "nhóm đích" để ALB biết sẽ chuyển traffic đi đâu.

1. Vào **EC2 Console** -> **Target Groups** -> **Create target group**.
2. **Target Group 1 (Cho Backend):**
    * Target type: `Instances`
    * Name: `Backend-TG`
    * Protocol: HTTP | Port: `8080` (Port của App Java/Nodejs).
    * VPC: `3-tier-vpc`.
    * Health check path: `/api/health` (hoặc `/`).
3. **Target Group 2 (Cho Frontend):**
    * Target type: `Instances`
    * Name: `Frontend-TG`
    * Protocol: HTTP | Port: `80` (Port của Nginx/Apache).
    * VPC: `3-tier-vpc`.

### Phần B: Tạo Internal ALB (Cho Backend)
ALB này nằm giữa Frontend và Backend.

1. Vào **Load Balancers** -> **Create load balancer** -> **Application Load Balancer**.
2. **Basic configuration:**
    * Name: `Internal-App-ALB`
    * Scheme: **Internal** (Quan trọng: Chỉ nội bộ VPC mới gọi được).
    * IP address type: IPv4.
3. **Network mapping:**
    * VPC: `3-tier-vpc`.
    * Subnets: Chọn 2 **Private Subnet** (Web-Private-Subnet-1a/1b) - *Nơi đặt Frontend để gọi vào ALB này*.
4. **Security groups:** Chọn `Internal-ALB-SG` (Đã tạo ở 5.4.1).
5. **Listeners:** Protocol HTTP/80 -> Forward to `Backend-TG`.
6. **Create load balancer**.
7. *Lưu lại DNS Name (Ví dụ: internal-app-alb-xxx.ap-southeast-1.elb.amazonaws.com).*

### Phần C: Tạo Public ALB (Cho Frontend)
ALB này đón khách từ Internet.

1. **Create load balancer** -> **Application Load Balancer**.
2. **Basic configuration:**
    * Name: `Public-Web-ALB`
    * Scheme: **Internet-facing** (Quan trọng: Để truy cập từ ngoài).
3. **Network mapping:**
    * VPC: `3-tier-vpc`.
    * Subnets: Chọn 2 **Public Subnet** (Public-Subnet-1a/1b).
4. **Security groups:** Chọn `Public-ALB-SG` (Đã tạo ở 5.4.1).
5. **Listeners:** Protocol HTTP/80 -> Forward to `Frontend-TG`.
6. **Create load balancer**.

Sau bước này, hạ tầng cân bằng tải đã sẵn sàng để gắn các máy chủ Auto Scaling vào.