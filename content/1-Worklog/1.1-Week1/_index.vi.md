---
title: "Báo cáo công việc Tuần 1"
date: "2025-09-09T19:53:52+07:00"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Mục tiêu Tuần 1:

* Kết nối và làm quen với các thành viên trong chương trình First Cloud Journey.
* Hiểu về Cơ sở hạ tầng toàn cầu của AWS (Regions - Vùng, Availability Zones - Khu vực sẵn sàng).
* Nắm vững các khái niệm về Quản lý danh tính và quyền truy cập (IAM): Users, Groups, Roles, và Policies.
* Bảo mật tài khoản AWS Root (Tài khoản gốc) và thiết lập quyền truy cập quản trị phù hợp.

### Các nhiệm vụ thực hiện trong tuần:

| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| :--- | :--- | :--- | :--- | :--- |
| 1 | - Làm quen với các thành viên FCJ <br> - Đọc và ghi chú các quy định của doanh nghiệp <br> - Tổng quan về các khái niệm Cloud & Hạ tầng toàn cầu AWS | 06/09/2025 | 06/09/2025 | |
| 2 | - Tạo tài khoản AWS Free Tier <br> - **Ưu tiên bảo mật (Security First):** <br>&emsp; + Bảo mật Root User với MFA <br>&emsp; + Tạo Ngân sách tài khoản (Cảnh báo chi phí/Billing Alarm) <br>&emsp; + Tìm hiểu Mô hình Trách nhiệm Chia sẻ (Shared Responsibility Model) | 07/09/2025 | 07/09/2025 | <https://aws.amazon.com/getting-started/> <br><br> https://www.youtube.com/@AWSStudyGroup |
| 3 | - **Tìm hiểu sâu về IAM (Định danh):** <br>&emsp; + Phân biệt IAM Users và Root User <br>&emsp; + Hiểu về IAM Groups <br> - **Thực hành:** <br>&emsp; + Tạo nhóm "Admin" với quyền AdministratorAccess <br>&emsp; + Tạo IAM User dùng hàng ngày và thêm vào nhóm Admin <br>&emsp; + Thiết lập Chính sách mật khẩu (Password Policy) tùy chỉnh | 08/09/2025 | 08/09/2025 | <https://docs.aws.amazon.com/iam/> |
| 4 | - **Tìm hiểu sâu về IAM (Quyền truy cập):** <br>&emsp; + Hiểu về IAM Policies (Cấu trúc JSON) <br>&emsp; + Managed Policies vs. Inline Policies <br>&emsp; + Hiểu về IAM Roles (Service Roles vs. Cross-account) <br> - **Thực hành:** <br>&emsp; + Viết một Policy tùy chỉnh (ví dụ: chỉ cho phép đọc S3) <br>&emsp; + Tạo Role cho một dịch vụ AWS (chuẩn bị lý thuyết) | 09/09/2025 | 09/09/2025 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - **AWS CLI & Access Keys:** <br>&emsp; + Cài đặt AWS CLI <br>&emsp; + Tạo Access Keys cho IAM User <br>&emsp; + Cấu hình CLI (`aws configure`) <br> - **Ôn tập & Dọn dẹp:** <br>&emsp; + Kiểm tra MFA đã hoạt động trên Root và IAM Users chưa <br>&emsp; + Test quyền truy cập CLI (ví dụ: `aws sts get-caller-identity`) | 10/09/2025 | 10/09/2025 | <https://aws.amazon.com/cli/> |

### Thành tựu Tuần 1:

* Đã kết nối và làm quen thành công với các thành viên của First Cloud Journey.
* Tạo thành công tài khoản AWS Free Tier với đầy đủ bảo mật và cài đặt Cảnh báo chi phí (Billing Alarms).
* Bảo mật tài khoản Root User bằng Xác thực đa yếu tố (MFA).
* Nắm vững các kiến thức nền tảng về IAM:
    * Tạo được IAM User quản trị (ngưng sử dụng Root cho các tác vụ hàng ngày).
    * Tạo IAM Groups để quản lý quyền hạn hiệu quả.
    * Hiểu cấu trúc của JSON Policies và Nguyên tắc đặc quyền tối thiểu (Principle of Least Privilege).
* Cài đặt và cấu hình thành công AWS CLI.
* Kết nối thành công vào tài khoản AWS thông qua terminal và xác minh danh tính bằng lệnh `sts get-caller-identity`.