---
title: "Bản đề xuất"
date: "2025-09-09T19:53:52+07:00"
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
{{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây chỉ nhằm mục đích tham khảo, vui lòng **không sao chép nguyên văn** cho bài báo cáo của bạn kể cả warning này.
{{% /notice %}}

Tại phần này, bạn cần tóm tắt các nội dung trong workshop mà bạn **dự tính** sẽ làm.

# IoT Weather Platform for Lab Research  
## Giải pháp AWS Serverless hợp nhất cho giám sát thời tiết thời gian thực  

### 1. Tóm tắt điều hành  
Nền tảng Bán Thẻ Game là một dự án thương mại điện tử được xây dựng trong khuôn khổ kỳ thực tập OJT tại AWS. Mục tiêu là xây dựng một website hoàn chỉnh cho phép người dùng đăng ký, mua và nhận mã thẻ game một cách tự động.

Dự án sẽ áp dụng kiến trúc 3-tầng (3-Tier) doanh nghiệp cổ điển. Kiến trúc này bao gồm Application Load Balancer (ALB), các máy chủ Amazon EC2 (chạy Java API) và cơ sở dữ liệu Amazon RDS. Toàn bộ hệ thống sẽ được triển khai bên trong một AWS VPC (Virtual Private Cloud) tùy chỉnh với các lớp Mạng (Subnet) riêng biệt, mô phỏng một hệ thống doanh nghiệp có tính bảo mật và khả năng mở rộng cao. Đây là cơ hội để áp dụng trực tiếp kiến thức đã học trong tháng đầu tiên của kỳ OJT.

### 2. Tuyên bố vấn đề  
*Vấn đề hiện tại*  

Sinh viên OJT: Cần một dự án đủ thực tế để áp dụng sâu kiến thức của tháng học đầu tiên, đặc biệt là thiết kế VPC, Subnets, Route Tables, Security Groups, EC2, và RDS. Các kiến trúc serverless (chỉ dùng Lambda) không thể hiện rõ được kỹ năng thiết kế và quản lý hạ tầng mạng này.

Hệ thống (Bảo mật): Việc lưu trữ kho mã thẻ game (chưa sử dụng) và thông tin giao dịch đòi hỏi một hệ thống có tính bảo mật cực cao, không thể để cơ sở dữ liệu bị truy cập trực tiếp từ Internet. 

Người dùng (Game thủ): Cần một nền tảng đáng tin cậy, nhanh chóng và an toàn để mua mã thẻ game và nhận được sản phẩm (mã code) ngay lập tức sau khi thanh toán đồng thời có khuyến mãi lớn.

*Giải pháp*  
Tầng Web/Trình bày (Web Tier): Sử dụng AWS Amplify để host ứng dụng front-end (ví dụ: React, Next.js). Đây là điểm tiếp xúc duy nhất với người dùng.

Tầng Ứng dụng (Application Tier):

Application Load Balancer (ALB) được đặt trong Public Subnet để nhận yêu cầu (request) từ người dùng.

Amazon EC2 (chạy ứng dụng Java Spring Boot API) được đặt trong Private Subnet. EC2 sẽ được quản lý bởi Auto Scaling Group để đảm bảo tính sẵn sàng cao (High Availability).

Tầng Dữ liệu (Database Tier):

Amazon RDS (ví dụ: MySQL) được đặt trong một Private Subnet riêng biệt (Database Subnet), chỉ cho phép truy cập từ Tầng Ứng dụng (EC2).

Bảo mật & Quản lý: Amazon Cognito sẽ xử lý việc xác thực người dùng.
*Lợi ích và hoàn vốn đầu tư (ROI)*  
Giá trị học tập (OJT): Đây là dự án mẫu mực để thực hành thiết kế hạ tầng AWS theo tiêu chuẩn doanh nghiệp. Sinh viên sẽ phải tự tay cấu hình toàn bộ mạng (VPC), các luồng (routing) và quy tắc bảo mật (Security Group) cho cả 3 tầng.

Tính bảo mật cao: Dữ liệu nhạy cảm (CSDL, mã thẻ) được bảo vệ hoàn toàn trong các Private Subnets, không thể bị tấn công trực tiếp từ Internet.

Tính sẵn sàng cao: Việc sử dụng ALB và Auto Scaling Group (với ít nhất 2 EC2 instances) đảm bảo hệ thống vẫn hoạt động ngay cả khi một máy chủ gặp sự cố.

### 3. Kiến trúc giải pháp  

### 4. Triển khai kỹ thuật  
Chắc chắn rồi. Đây là toàn bộ proposal được xây dựng lại từ đầu theo kiến trúc 3-tầng (3-Tier) truyền thống, sử dụng Java API chạy trên EC2, đúng theo yêu cầu của bạn.

title: "Đề xuất Dự án OJT" date: "2025-11-08T19:53:52+07:00" weight: 2 chapter: false pre: " <b> 1. </b> "
{{% notice warning %}} ⚠️ Lưu ý: Thông tin dưới đây chỉ mang tính chất tham khảo. Vui lòng không sao chép nguyên văn vào báo cáo của bạn, bao gồm cả cảnh báo này. {{% /notice %}}

Trong phần này, bạn cần tóm tắt nội dung của dự án mà bạn lên kế hoạch thực hiện trong kỳ OJT.

Nền tảng Bán Thẻ Game Trực Tuyến
Giải pháp Thương mại điện tử 3-Tầng với Java API và AWS
1. Tóm tắt dự án
Nền tảng Bán Thẻ Game là một dự án thương mại điện tử được xây dựng trong khuôn khổ kỳ thực tập OJT tại AWS. Mục tiêu là xây dựng một website hoàn chỉnh cho phép người dùng đăng ký, mua và nhận mã thẻ game một cách tự động.

Dự án sẽ áp dụng kiến trúc 3-tầng (3-Tier) doanh nghiệp cổ điển. Kiến trúc này bao gồm Application Load Balancer (ALB), các máy chủ Amazon EC2 (chạy Java API) và cơ sở dữ liệu Amazon RDS. Toàn bộ hệ thống sẽ được triển khai bên trong một AWS VPC (Virtual Private Cloud) tùy chỉnh với các lớp Mạng (Subnet) riêng biệt, mô phỏng một hệ thống doanh nghiệp có tính bảo mật và khả năng mở rộng cao. Đây là cơ hội để áp dụng trực tiếp kiến thức đã học trong tháng đầu tiên của kỳ OJT.

2. Bài toán đặt ra
Vấn đề là gì?
Người dùng (Game thủ): Cần một nền tảng đáng tin cậy, nhanh chóng và an toàn để mua mã thẻ game và nhận được sản phẩm (mã code) ngay lập tức sau khi thanh toán.

Sinh viên OJT: Cần một dự án đủ thực tế để áp dụng sâu kiến thức của tháng học đầu tiên, đặc biệt là thiết kế VPC, Subnets, Route Tables, Security Groups, EC2, và RDS. Các kiến trúc serverless (chỉ dùng Lambda) không thể hiện rõ được kỹ năng thiết kế và quản lý hạ tầng mạng này.

Hệ thống (Bảo mật): Việc lưu trữ kho mã thẻ game (chưa sử dụng) và thông tin giao dịch đòi hỏi một hệ thống có tính bảo mật cực cao, không thể để cơ sở dữ liệu bị truy cập trực tiếp từ Internet.

Giải pháp
Chúng tôi đề xuất xây dựng hệ thống theo mô hình 3-tầng (3-Tier) tiêu chuẩn:

Tầng Web/Trình bày (Web Tier): Sử dụng AWS Amplify để host ứng dụng front-end (ví dụ: React, Next.js). Đây là điểm tiếp xúc duy nhất với người dùng.

Tầng Ứng dụng (Application Tier):

Application Load Balancer (ALB) được đặt trong Public Subnet để nhận yêu cầu (request) từ người dùng.

Amazon EC2 (chạy ứng dụng Java Spring Boot API) được đặt trong Private Subnet. EC2 sẽ được quản lý bởi Auto Scaling Group để đảm bảo tính sẵn sàng cao (High Availability).

Tầng Dữ liệu (Database Tier):

Amazon RDS (ví dụ: PostgreSQL) được đặt trong một Private Subnet riêng biệt (Database Subnet), chỉ cho phép truy cập từ Tầng Ứng dụng (EC2).

Bảo mật & Quản lý: Amazon Cognito sẽ xử lý việc xác thực người dùng.

Lợi ích và Giá trị
Giá trị học tập (OJT): Đây là dự án mẫu mực để thực hành thiết kế hạ tầng AWS theo tiêu chuẩn doanh nghiệp. Sinh viên sẽ phải tự tay cấu hình toàn bộ mạng (VPC), các luồng (routing) và quy tắc bảo mật (Security Group) cho cả 3 tầng.

Tính bảo mật cao: Dữ liệu nhạy cảm (CSDL, mã thẻ) được bảo vệ hoàn toàn trong các Private Subnets, không thể bị tấn công trực tiếp từ Internet.

Tính sẵn sàng cao: Việc sử dụng ALB và Auto Scaling Group (với ít nhất 2 EC2 instances) đảm bảo hệ thống vẫn hoạt động ngay cả khi một máy chủ gặp sự cố.

3. Kiến trúc giải pháp
Hệ thống được thiết kế hoàn toàn bên trong một VPC tùy chỉnh với 3 tầng bảo mật riêng biệt:

Web Tier (Amplify): Host front-end.

Application Tier (ALB + EC2): ALB (Public Subnet) nhận traffic và chuyển đến các EC2 (Private Subnet).

Database Tier (RDS): RDS (Private DB Subnet) chỉ nhận kết nối từ EC2.

Để gọi API thanh toán (ví dụ: VNPay) từ EC2 trong Private Subnet, chúng ta sẽ sử dụng một NAT Gateway đặt trong Public Subnet.

Các Dịch vụ AWS sử dụng
AWS VPC: Nền tảng mạng ảo, bao gồm 6 Subnets (2 Public, 2 Private App, 2 Private DB) trên 2 Availability Zones.

Application Load Balancer (ALB): Phân phối tải, là điểm cuối (endpoint) cho front-end.

Amazon EC2: Chạy ứng dụng Java Spring Boot API (file .jar).

Auto Scaling Group (ASG): Tự động quản lý số lượng máy chủ EC2.

Amazon RDS: Lưu trữ dữ liệu (PostgreSQL/MySQL).

NAT Gateway: Cho phép EC2 trong Private Subnet gọi ra Internet (để tích hợp cổng thanh toán).

AWS Amplify: Host front-end.

Amazon Cognito: Quản lý người dùng.

Amazon S3: Lưu trữ hình ảnh sản phẩm.

AWS Certificate Manager (ACM): Cung cấp chứng chỉ SSL/TLS miễn phí cho ALB.

Thiết kế thành phần
Front-end: React/Next.js (host trên Amplify), gọi đến endpoint của ALB.

Backend API: Java Spring Boot, chạy trên port 8080, được triển khai trên nhiều EC2 instances.

Cơ sở dữ liệu: Amazon RDS (PostgreSQL) với thiết kế schema cho Users, Products, CardCodes (kho mã thẻ), Orders.

Bảo mật (Quan trọng): Luồng truy cập được kiểm soát chặt chẽ bằng Security Groups:

ALB-SG: Cho phép truy cập port 443 (HTTPS) từ Internet (0.0.0.0/0).

EC2-SG: Chỉ cho phép truy cập port 8080 từ ALB-SG.

RDS-SG: Chỉ cho phép truy cập port 5432 từ EC2-SG.

4. Kế hoạch thực thi
Dự án được thực hiện trong 2 tháng (sau 1 tháng học lý thuyết).

Giai đoạn 1: Xây dựng Hạ tầng (Tháng 2 - Tuần 1-2)

Thiết kế và triển khai VPC (Subnets, Route Tables, Internet Gateway, NAT Gateway).

Triển khai RDS vào Private DB Subnet.

Cấu hình ALB, ASG và EC2 (với Java JDK) trong các subnet tương ứng.

Thiết kế Schema cho CSDL.

Giai đoạn 2: Xây dựng Backend (Tháng 2 - Tuần 3-4)

Xây dựng các API chính bằng Java Spring Boot (Sản phẩm, Đơn hàng, Kho mã thẻ).

Đóng gói (file .jar) và triển khai ứng dụng lên EC2.

Kiểm thử kết nối: ALB -> EC2 -> RDS.

Xây dựng API xác thực kết hợp với Cognito.

Giai đoạn 3: Xây dựng Frontend & Tích hợp (Tháng 3 - Tuần 1-3)

Xây dựng giao diện trên Amplify (React/Next.js).

Kết nối front-end với endpoint của ALB.

Tích hợp luồng đăng nhập/đăng ký với Cognito.

Tích hợp cổng thanh toán (Java API gọi ra ngoài qua NAT Gateway).

Giai đoạn 4: Kiểm thử & Hoàn thiện (Tháng 3 - Tuần 4)

Kiểm thử End-to-End toàn bộ luồng mua hàng.

Sửa lỗi, tối ưu và chuẩn bị tài liệu báo cáo OJT.
### 5. Lộ trình & Mốc triển khai  
Tháng 1 (OJT - Learning): Hoàn thành học lý thuyết về VPC, EC2, RDS, IAM.

Tháng 2 (OJT - Building 1):

    Tuần 1-2: Hạ tầng mạng (VPC, Subnets, Gateways) và CSDL (RDS) phải hoạt động.

    Tuần 3-4: Triển khai thành công Java API lên EC2 và truy cập được qua ALB.

Tháng 3 (OJT - Building 2):

    Tuần 1-2: Front-end (Amplify) và Cognito hoạt động.

    Tuần 3: Tích hợp thành công cổng thanh toán (Sandbox).

    Tuần 4: Hoàn thành sản phẩm (MVP), viết tài liệu và demo.

### 6. Ước tính ngân sách  
Kiến trúc này KHÔNG HOÀN TOÀN MIỄN PHÍ. Cần lưu ý về chi phí.

Dịch vụ có Free Tier:

Amazon EC2: 750 giờ/tháng (cho t2.micro hoặc t3.micro). Đủ cho 1 máy chủ chạy 24/7.

Amazon RDS: 750 giờ/tháng (cho db.t2.micro). Đủ cho 1 CSDL chạy 24/7.

Amazon Amplify, Cognito, S3: Đều có gói Free Tier rất lớn, đủ cho dự án OJT.

Dịch vụ TÍNH PHÍ (Không có Free Tier):

Application Load Balancer (ALB): Ước tính $18-20 USD/tháng.

NAT Gateway: Ước tính $32-35 USD/tháng. (Bắt buộc phải có để gọi API thanh toán).

Tổng chi phí dự kiến: Khoảng $50-60 USD/tháng. Lưu ý: Cần sử dụng AWS Budgets để theo dõi chi phí và tắt các tài nguyên (ALB, NAT GW) khi không phát triển để tiết kiệm.

### 7. Đánh giá rủi ro  
Rủi ro 1: Chi phí vượt mức (Cao): Sinh viên quên tắt ALB và NAT Gateway sau giờ làm việc.

Giảm thiểu: Thiết lập AWS Budgets. Viết kịch bản (script) để bật/tắt tài nguyên theo giờ.

Rủi ro 2: Cấu hình Mạng (Cao): Thiết kế VPC, Subnets, Route Tables, và Security Groups rất phức tạp và dễ lỗi.

Giảm thiểu: Vẽ sơ đồ kiến trúc thật kỹ. Kiểm tra kết nối (connectivity) từng bước (ví dụ: EC2 có ping được RDS không, EC2 có ra được Internet qua NAT không).

Rủi ro 3: Triển khai API (Trung bình): Quản lý việc triển khai file .jar lên nhiều máy chủ EC2 phức tạp hơn serverless.

Giảm thiểu: Bắt đầu với 1 EC2. Sử dụng User Data script để tự động cài đặt Java và chạy API khi EC2 khởi động.
### 8. Kết quả kỳ vọng  
Kết quả kỹ thuật: Một trang web bán thẻ game MVP hoạt động, được triển khai trên một kiến trúc 3-tầng bảo mật, mạnh mẽ và có tính sẵn sàng cao.

Kết quả học tập (OJT): Sinh viên nắm vững và có thể tự tay triển khai một kiến trúc AWS tiêu chuẩn doanh nghiệp từ A-Z, đặc biệt là các kỹ năng về Networking (VPC) và Hạ tầng (EC2, ALB).

Giá trị lâu dài: Kiến trúc này là nền tảng cho hầu hết các ứng dụng doanh nghiệp, mang lại giá trị lớn cho CV và kỹ năng thực tiễn.