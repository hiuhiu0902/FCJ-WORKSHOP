---
title : "Cài đặt máy chủ (Compute)"
date : "2023-10-25T00:00:00+07:00"
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

Trong phần này, chúng ta sẽ khởi tạo các máy chủ EC2 cần thiết. Vì chúng ta tuân thủ kiến trúc bảo mật 3 lớp, chúng ta không thể SSH trực tiếp vào App Server.

Quy trình kết nối sẽ như sau:
**Laptop (Putty) -> Internet -> Bastion Host (Public) -> App Server (Private)**

Chúng ta sẽ thực hiện 2 bước chính:
1.  **Tạo Bastion Host:** Máy chủ nằm ở Public Subnet, đóng vai trò "cầu nối".
2.  **Tạo & Cấu hình App Server:** Máy chủ nằm ở Private Subnet. Chúng ta sẽ cài đặt môi trường (Java/NodeJS, Database Client), deploy code và kiểm tra kết nối tới RDS.

Sau khi cấu hình xong App Server, chúng ta sẽ tạo một **AMI (Amazon Machine Image)** từ máy chủ này để sử dụng cho Auto Scaling Group ở bài sau.