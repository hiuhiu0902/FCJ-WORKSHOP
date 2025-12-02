---
title: "Nhật ký Tuần 2"
date: "2025-09-15T09:00:00+07:00"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu Tuần 2:
* Nắm vững các kiến thức cơ bản về Mạng trên AWS (Networking).
* Hiểu cách cô lập tài nguyên bằng VPC và Subnet.
* Kiểm soát luồng traffic bằng Route Table và Security Group.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **VPC & Subnets:**<br>- Học về CIDR block.<br>- Phân biệt Default VPC và Custom VPC.<br>- Sự khác nhau giữa Public và Private Subnet. | 15/09/2025 | 15/09/2025 | [AWS VPC Docs](https://docs.aws.amazon.com/vpc/) |
| 2 | **Kết nối:**<br>- Thiết lập Internet Gateway (IGW).<br>- Route Tables (Bảng định tuyến chính vs tùy chỉnh).<br>- Gán Subnet vào Route Table. | 16/09/2025 | 16/09/2025 | |
| 3 | **Bảo mật:**<br>- **Security Groups** (Stateful) vs **NACLs** (Stateless).<br>- Tạo Security Group chỉ cho phép SSH (22) và HTTP (80) từ IP của tôi. | 17/09/2025 | 17/09/2025 | |
| 4 | **Thực hành Lab:**<br>- Tạo một VPC tùy chỉnh.<br>- Tạo 1 Public Subnet và gắn IGW.<br>- Thử ping tới instance. | 18/09/2025 | 18/09/2025 | |
| 5 | **Ôn tập:**<br>- Kiểm chứng lại hiểu biết về "Public" (có đường ra IGW) và "Private" (không có đường ra IGW). | 19/09/2025 | 19/09/2025 | |

### Thành tựu đạt được:
* Đã tạo thành công VPC tùy chỉnh với các subn---
title: "Nhật ký Tuần 2"
date: "2025-09-15T09:00:00+07:00"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu Tuần 2:
* Nắm vững các kiến thức cơ bản về Mạng trên AWS (Networking).
* Hiểu cách cô lập tài nguyên bằng VPC và Subnet.
* Kiểm soát luồng traffic bằng Route Table và Security Group.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **VPC & Subnets:**<br>- Học về CIDR block.<br>- Phân biệt Default VPC và Custom VPC.<br>- Sự khác nhau giữa Public và Private Subnet. | 15/09/2025 | 15/09/2025 | [AWS VPC Docs](https://docs.aws.amazon.com/vpc/) |
| 2 | **Kết nối:**<br>- Thiết lập Internet Gateway (IGW).<br>- Route Tables (Bảng định tuyến chính vs tùy chỉnh).<br>- Gán Subnet vào Route Table. | 16/09/2025 | 16/09/2025 | |
| 3 | **Bảo mật:**<br>- **Security Groups** (Stateful) vs **NACLs** (Stateless).<br>- Tạo Security Group chỉ cho phép SSH (22) và HTTP (80) từ IP của tôi. | 17/09/2025 | 17/09/2025 | |
| 4 | **Thực hành Lab:**<br>- Tạo một VPC tùy chỉnh.<br>- Tạo 1 Public Subnet và gắn IGW.<br>- Thử ping tới instance. | 18/09/2025 | 18/09/2025 | |
| 5 | **Ôn tập:**<br>- Kiểm chứng lại hiểu biết về "Public" (có đường ra IGW) và "Private" (không có đường ra IGW). | 19/09/2025 | 19/09/2025 | |

### 🧠 Kiến thức mở rộng: Tính chất "Stateful" của Security Group
Trong quá trình cấu hình tường lửa hôm nay, tôi đã nhận ra một điểm quan trọng:
* **Security Group là Stateful:** Nếu tôi cho phép traffic đi vào (Inbound) ở cổng 80, thì traffic phản hồi đi ra (Outbound) sẽ tự động được cho phép.
* **NACL là Stateless:** Tôi bắt buộc phải mở rule ở **cả hai chiều** (Vào và Ra). Đây là lý do tại sao lúc đầu tôi dùng NACL chặn IP nhưng lại vô tình chặn luôn cả gói tin phản hồi của server!

### Thành tựu đạt được:
* Đã tạo thành công VPC tùy chỉnh với các subnet được phân chia logic (CIDR /16 và /24).
* Cấu hình Internet Gateway và Route Table hoạt động tốt.
* Xác minh bảo mật mạng bằng cách sử dụng Security Group hạn chế quyền truy cập.et được phân chia logic.
* Cấu hình Internet Gateway và Route Table hoạt động tốt.
* Xác minh bảo mật mạng bằng cách sử dụng Security Group hạn chế quyền truy cập.