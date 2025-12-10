---
title : "Cấu hình Mạng (VPC)"
date :  "2025-12-10T10:00:00+07:00"
weight : 3
chapter : true
pre : " <b> 5.3. </b> "
---

### Tổng quan phần Mạng
Chào mừng bạn đến với phần quan trọng nhất của Workshop: **Xây dựng nền móng hạ tầng mạng**.

Trong chương này, chúng ta sẽ thiết kế một Virtual Private Cloud (VPC) đạt chuẩn **High Availability (HA)** tại Region **Singapore (ap-southeast-1)**. Hệ thống mạng này sẽ là nơi chứa toàn bộ Web Server, App Server và Database của bạn.

#### Mục tiêu kỹ thuật
Sau khi hoàn thành chương này, bạn sẽ có:
* **1 VPC** với dải mạng `10.75.0.0/16`.
* **8 Subnets** được phân chia khoa học trên 2 Availability Zones.
* **Hệ thống Routing** hoàn chỉnh với Internet Gateway và NAT Gateway.

#### Các bước thực hiện
Chương này được chia làm 2 bài lab nhỏ:

1.  [**Tạo VPC & Subnets:**](/vi/5-workshop/5.3-network-setup/5.3.1-vpc-subnets/) Tạo khung xương cho hệ thống mạng.
2.  [**Cấu hình NAT & Routing:**](/vi/5-workshop/5.3-network-setup/5.3.2-nat-routing/) Kết nối mạch máu để lưu thông dữ liệu.

![Sơ đồ VPC](/images/5-Workshop/5.3-Network-Setup/vpc-diagram.png)