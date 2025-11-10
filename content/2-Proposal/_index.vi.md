---
title: "BĐP - Nền tảng Bán Thẻ Game Trực Tuyến"
date: "2025-09-10T10:00:00+07:00"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

{{% notice info %}}
Tài liệu đề xuất kỹ thuật: Giải pháp Nền tảng Thương mại điện tử (Bán thẻ game) có tính sẵn sàng cao và bảo mật.
{{% /notice %}}

### 1. Tóm tắt điều hành

Nền tảng Bán Thẻ Game là một dự án thương mại điện tử với mục tiêu xây dựng một website hoàn chỉnh, cho phép người dùng đăng ký, đăng nhập, mua và nhận mã thẻ game ngay lập tức một cách tự động.

Giải pháp này được thiết kế với kiến trúc đa tầng (multi-tier), ưu tiên hàng đầu cho bảo mật và hiệu suất. Bằng cách sử dụng các dịch vụ hàng đầu của AWS, chúng tôi đề xuất triển khai **kiến trúc ban đầu trên hai Vùng Sẵn sàng (Availability Zones)** để tối ưu chi phí, đồng thời vạch ra lộ trình nâng cấp lên ba AZ để đảm bảo độ tin cậy tối đa trong tương lai.

### 2. Bài toán đặt ra

#### Vấn đề về phía Hệ thống (Bảo mật):
Kho mã thẻ game (chưa sử dụng) và thông tin giao dịch là dữ liệu cực kỳ nhạy cảm. Việc để lộ hoặc mất mát có thể gây thiệt hại tài chính nghiêm trọng. Do đó, hệ thống CSDL phải được bảo vệ ở mức tối đa, không thể bị truy cập trực tiếp từ Internet.

#### Vấn đề về phía Người dùng (Trải nghiệm & Tin cậy):
Game thủ là nhóm người dùng đòi hỏi tốc độ. Họ cần một nền tảng đáng tin cậy, phản hồi nhanh, an toàn khi thanh toán, và phải nhận được mã code ngay lập tức sau khi giao dịch thành công. Bất kỳ sự chậm trễ hay gián đoạn nào cũng sẽ làm giảm uy tín của nền tảng.

### 3. Kiến trúc giải pháp

Chúng tôi đề xuất một kiến trúc "Well-Architected" được triển khai trong một AWS VPC tùy chỉnh.

---

#### 3.1. Kiến trúc Hiện tại (Giai đoạn 1): 2 AZ + 1 NAT Gateway

Để cân bằng giữa chi phí và tính sẵn sàng, kiến trúc ban đầu sẽ được triển khai trên **2 Vùng Sẵn sàng (AZ)**.

![Kiến trúc Giai đoạn 1: 2 AZ + 1 NAT Gateway](/images/proposal/architecture-2az-1nat.png)

* **Lớp Biên (Edge Layer):** (Không đổi)
    * Người dùng truy cập qua **Amazon Route 53** -> **Amazon CloudFront (CDN)**.
    * **AWS WAF** và **AWS Shield** được tích hợp với CloudFront để chặn tấn công.

* **Lớp Cân bằng tải (Load Balancing Layer):**
    * Yêu cầu hợp lệ được chuyển đến **Application Load Balancer (ALB)**.
    * ALB được đặt trong 2 Public Subnet, mỗi Public Subnet thuộc 1 AZ.

* **Lớp Ứng dụng (Application Tier):**
    * Các máy chủ Web/API (EC2) được đặt trong 2 Private Subnet (mỗi AZ 1 Private Subnet).
    * Các máy chủ được quản lý bởi **Auto Scaling Group (ASG)**, đảm bảo luôn có ít nhất 2 máy chủ chạy ở 2 AZ khác nhau.
    * Truy xuất thông tin nhạy cảm (API key, mật khẩu CSDL) từ **AWS Secrets Manager**.

* **Lớp Dữ liệu (Data Tier):**
    * **Amazon RDS (Database):** CSDL chính (Primary) được đặt trong Private DB Subnet ở AZ 1.
    * **RDS Replica (Bản sao):** Một bản sao đồng bộ (Replica) được đặt ở Private DB Subnet tại AZ 2 để dự phòng (Multi-AZ Failover).
    * **Amazon ElastiCache (Caching):** Một cụm cache (Redis) được triển khai (có thể là Multi-AZ) để giảm tải cho CSDL.
    * **Amazon S3:** Lưu trữ hình ảnh, logs.

* **Lớp Mạng & Giám sát (Networking & Monitoring):**
    * **NAT Gateway:** Chỉ **01 NAT Gateway** duy nhất được tạo ra và đặt trong Public Subnet ở AZ 1.
    * Tất cả các Route Table của cả 2 Private Subnet (ở AZ 1 và AZ 2) đều sẽ trỏ ra Internet thông qua 01 NAT Gateway này.
    * **Bastion Host:** Đặt trong Public Subnet (có thể ở AZ 1) để admin SSH vào máy chủ.
    * **Amazon CloudWatch & VPC Flow Logs:** Giám sát và ghi log toàn bộ hệ thống.

---

#### 3.2. Lộ trình Nâng cấp (Giai đoạn 2): 3 AZ + 3 NAT Gateway

Sau khi nền tảng hoạt động ổn định và có doanh thu, chúng ta có thể nâng cấp lên kiến trúc 3 AZ để đạt độ tin cậy tối đa.

<<<![Kiến trúc Giai đoạn 2: 3 AZ + 3 NAT Gateway](/images/2-Proposal/architect.jpeg)>>>

* **Thay đổi chính:**
    1.  Mở rộng VPC để hỗ trợ **3 Vùng Sẵn sàng (AZ)**.
    2.  Triển khai **03 NAT Gateway**, mỗi NAT Gateway đặt tại một Public Subnet ở mỗi AZ.
    3.  Cập nhật Route Table cho các Private Subnet: Private Subnet trong AZ 1 sẽ đi qua NAT Gateway ở AZ 1, Private Subnet ở AZ 2 đi qua NAT ở AZ 2, v.v.

* **Lợi ích sau khi nâng cấp:**
    * **Loại bỏ Điểm lỗi Đơn (SPOF):** Ở kiến trúc 1, nếu NAT Gateway (hoặc AZ chứa nó) gặp sự cố, toàn bộ hệ thống sẽ mất kết nối Internet (không thể gọi API thanh toán). Kiến trúc 3 NAT Gateway đảm bảo sự cố ở 1 AZ không ảnh hưởng đến 2 AZ còn lại.
    * **Tăng Tính sẵn sàng (High Availability):** Đạt được mức độ chịu lỗi cao nhất mà AWS khuyến nghị cho các hệ thống quan trọng.

### 4. Kết quả kỳ vọng

* **Về kỹ thuật:** Một nền tảng E-commerce (MVP) hoạt động ổn định, có khả năng chống chịu lỗi (fault tolerance) tốt (cấp độ 2 AZ) và có lộ trình nâng cấp rõ ràng lên 3 AZ.
* **Về trải nghiệm người dùng:** Tốc độ tải trang nhanh, quy trình mua hàng mượt mà và an toàn.
* **Về bảo mật:** Dữ liệu nhạy cảm được bảo vệ an toàn trong nhiều lớp mạng riêng biệt.

---

### 5. Ước tính ngân sách (Chi phí hàng tháng)

Phân tích chi tiết chi phí cho cả hai kiến trúc.

#### 5.1. Chi phí Kiến trúc Hiện tại (2 AZ + 1 NAT Gateway)

* **Application Load Balancer (ALB):** ~$20.00 /tháng (chi phí cố định).
* **NAT Gateway (1 Gateway):**
    * Chi phí giờ: 1 * $0.045/giờ * 730 giờ/tháng = ~$32.85
    * Chi phí xử lý dữ liệu (Giả định 100GB): 100 * $0.045/GB = $4.50
    * *Tổng NAT Gateway:* **~$37.35 /tháng**
* **Amazon ElastiCache:** Tối thiểu 1 node `cache.t4g.micro` (Multi-AZ) = **~$17.00 /tháng**
* **AWS WAF:** Chi phí cơ bản (1 Web ACL + 10 Quy tắc) = **~$10.00 /tháng**
* **Dịch vụ trong Free Tier (Giả định năm đầu):**
    * **EC2 (App Server):** Free Tier cung cấp 750 giờ `t2.micro` (hoặc `t3.micro`). Chúng ta cần 2 máy chủ (cho 2 AZ), vậy 1 máy chủ sẽ miễn phí, 1 máy chủ sẽ tính phí.
        * 1 x `t3.micro` (miễn phí) = $0.00
        * 1 x `t3.micro` (~$0.0104/giờ) = ~$7.60 /tháng
    * **RDS (Database):** Free Tier cung cấp 750 giờ `db.t2.micro` (Single-AZ). Kiến trúc của chúng ta là Multi-AZ (Primary + Replica).
        * 1 x `db.t3.micro` (Primary) = ~$12.00 /tháng
        * 1 x `db.t3.micro` (Replica) = ~$12.00 /tháng
        * (Ghi chú: Sẽ không dùng được Free Tier cho RDS Multi-AZ)
    * **CloudFront, S3, Secrets Manager:** Chi phí rất thấp, dưới **~$1.00 /tháng** ở mức sử dụng ban đầu.

> **Tổng chi phí ước tính (Giai đoạn 1):**
> ~$20 (ALB) + ~$37.35 (NAT) + ~$17 (Cache) + ~$10 (WAF) + ~$7.60 (EC2) + ~$24 (RDS) + ~$1 (Khác)
> **= Khoảng $115 - $120 USD /tháng**

#### 5.2. Chi phí Kiến trúc Nâng cấp (3 AZ + 3 NAT Gateway)

* **Application Load Balancer (ALB):** ~$20.00 /tháng (Không đổi).
* **NAT Gateway (3 Gateway):**
    * Chi phí giờ: 3 * $0.045/giờ * 730 giờ/tháng = ~$98.55
    * Chi phí xử lý dữ liệu (Giả định 100GB, chia 3): 100 * $0.045/GB = $4.50
    * *Tổng NAT Gateway:* **~$103.05 /tháng**
* **Amazon ElastiCache:** ~$17.00 /tháng (Không đổi).
* **AWS WAF:** ~$10.00 /tháng (Không đổi).
* **Dịch vụ (Đã hết Free Tier hoặc mở rộng):**
    * **EC2 (App Server):** 3 máy chủ `t3.micro` = 3 * ~$7.60 = ~$22.80 /tháng
    * **RDS (Database):** Multi-AZ (Primary + Replica) = ~$24.00 /tháng (Không đổi)
    * **CloudFront, S3, v.v:** ~$1.00 /tháng

> **Tổng chi phí ước tính (Giai đoạn 2):**
> ~$20 (ALB) + ~$103.05 (NAT) + ~$17 (Cache) + ~$10 (WAF) + ~$22.80 (EC2) + ~$24 (RDS) + ~$1 (Khác)
> **= Khoảng $195 - $200 USD /tháng**

---

### 6. Đánh giá rủi ro

* **Rủi ro chi phí (Cao):**
    * Chi phí cho NAT Gateway, ALB, và RDS Multi-AZ là đáng kể. Kiến trúc Giai đoạn 2 có chi phí cao hơn gần gấp đôi.
    * **Giảm thiểu:** Bắt đầu với Giai đoạn 1. Thiết lập **AWS Budgets** để cảnh báo khi chi phí vượt ngưỡng. Tắt/xóa tài nguyên (đặc biệt là NAT GW) khi không phát triển.

* **Rủi ro cấu hình mạng (Cao):**
    * Kiến trúc mạng phức tạp (VPC, Subnets, Route Tables, WAF). Cấu hình sai có thể khiến hệ thống không hoạt động hoặc bị hở lỗ hổng bảo mật.
    * **Giảm thiểu:** Thiết kế sơ đồ kỹ. Kiểm tra kết nối từng bước (EC2 -> RDS, EC2 -> NAT -> Internet).

* **Rủi ro Điểm lỗi Đơn (Medium - Giai đoạn 1):**
    * Kiến trúc Giai đoạn 1 sử dụng 1 NAT Gateway duy nhất. Nếu AZ chứa NAT Gateway này gặp sự cố, các máy chủ trong Private Subnet sẽ không thể gọi ra ngoài (ví dụ: API thanh toán).
    * **Giảm thiểu:** Đây là rủi ro chấp nhận được để tiết kiệm chi phí ban đầu. Lên kế hoạch nâng cấp lên Giai đoạn 2 (3 AZ + 3 NAT) khi hệ thống yêu cầu độ tin cậy cao hơn.