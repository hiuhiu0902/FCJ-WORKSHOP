---
title: "Workshop"
date: "2025-09-09T19:53:52+07:00"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying Highly Available N-Tier Architecture

#### Overview

**N-Tier Architecture** (Kiến trúc đa tầng) là mô hình phổ biến để xây dựng các ứng dụng doanh nghiệp có khả năng mở rộng, bảo mật và độ sẵn sàng cao trên AWS.

Trong bài thực hành (Lab) này, bạn sẽ học cách triển khai một hệ thống web hoàn chỉnh bao gồm Frontend và Backend tách biệt, sử dụng các dịch vụ cốt lõi như VPC, EC2, RDS, và Application Load Balancer.

Bạn sẽ xây dựng hệ thống với 3 tầng riêng biệt (Tiers), mỗi tầng đóng vai trò cụ thể và được bảo vệ bởi các lớp bảo mật mạng nghiêm ngặt:
+ **Presentation Tier (Public)** - Chứa Public Load Balancer và Frontend Auto Scaling Group. Đây là điểm tiếp nhận traffic từ người dùng Internet thông qua giao thức HTTP/HTTPS.
+ **Logic Tier (Private)** - Chứa Internal Load Balancer và Backend Auto Scaling Group. Tầng này xử lý logic nghiệp vụ và chỉ chấp nhận kết nối từ tầng Presentation.
+ **Data Tier (Private)** - Chứa cơ sở dữ liệu Amazon RDS Multi-AZ. Tầng này lưu trữ dữ liệu bền vững và được bảo vệ kỹ nhất, không thể truy cập trực tiếp từ Internet.

#### Content

1. [Giới thiệu & Kiến trúc](5.1-Workshop-overview/)
2. [Các bước chuẩn bị](5.2-Prerequiste/)
3. [Thiết lập hạ tầng mạng (VPC)](5.3-Network-Setup/)
4. [Cấu hình bảo mật (Security)](5.4-Security-Config/)
5. [Cài đặt máy chủ (Compute)](5.5-Compute-Setup/)
6. [Khởi tạo Database (RDS)](5.6-Database-Setup/)
7. [Cấu hình Load Balancer](5.7-LoadBalancer-Setup/)
