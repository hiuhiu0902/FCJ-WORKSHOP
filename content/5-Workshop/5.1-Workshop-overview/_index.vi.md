---
title : "Giới thiệu Architecture"
date : "2025-09-10T10:00:00+07:00"
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Tổng quan
Trong bài thực hành này, chúng ta sẽ triển khai kiến trúc **N-Tier Microservices** (Frontend và Backend tách biệt) với độ sẵn sàng cao (High Availability). Hệ thống bao gồm:
1.  **Public Tier:** Chứa Public Load Balancer đón traffic từ Internet.
2.  **Frontend Tier (Web):** Chứa các Web Server giao tiếp với người dùng.
3.  **Backend Tier (App):** Chứa các App Server xử lý logic nghiệp vụ, nhận request qua **Internal Load Balancer**.
4.  **Database Tier:** Chứa RDS Multi-AZ nằm trong vùng mạng bí mật nhất.

#### Sơ đồ kiến trúc
Dưới đây là sơ đồ kiến trúc chi tiết chúng ta sẽ xây dựng:

![Sơ đồ kiến trúc chi tiết](/images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png)

#### Các bước thực hiện
1.  **Network:** Thiết kế VPC với 4 lớp mạng (Public, Web, App, DB).
2.  **Database:** Khởi tạo RDS Multi-AZ (Primary & Standby).
3.  **Backend Compute:** Cấu hình Internal ALB và Auto Scaling cho Backend.
4.  **Frontend Compute:** Cấu hình Public ALB và Auto Scaling cho Frontend.