---
title: "Nhật ký Tuần 12"
date: "2025-11-24T09:00:00+07:00"
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Mục tiêu Tuần 12:
* Kiểm chứng khả năng chịu lỗi và đóng dự án.
* Thực hiện kiểm thử Chaos Engineering với ứng dụng Java.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **Test HA:**<br>- Xóa EC2 đang chạy file Jar, xem ASG tự tạo mới. | 24/11/2025 | 24/11/2025 | |
| 2 | **Test DB Failover:**<br>- Reboot RDS với chế độ failover. | 25/11/2025 | 25/11/2025 | |
| 3 | **Test Rollback Giao dịch:**<br>- Giả lập lỗi thanh toán khi đang tạo đơn. | 26/11/2025 | 26/11/2025 | |
| 4 | **Tài liệu:**<br>- Báo cáo Post-Mortem. | 27/11/2025 | 27/11/2025 | |
| 5 | **Đóng dự án:**<br>- **XÓA TOÀN BỘ TÀI NGUYÊN.** | 28/11/2025 | 28/11/2025 | |

### 🧠 Kiến thức mở rộng: Tính chất ACID trong Kiểm thử
Ngoài việc phá hoại hạ tầng (tắt server), tôi còn tập trung vào **Tính toàn vẹn dữ liệu**.
Tôi đã kiểm chứng rằng nếu `OrderService.handlePaymentFailure()` được gọi, hệ thống sẽ rollback trạng thái kho từ `PENDING` về `UNUSED` một cách chính xác. Điều này khẳng định tính **A**tomicity (Nguyên tử) và **C**onsistency (Nhất quán) của MySQL database được quản lý bởi Spring Transaction.

### 💻 Automation Code: Chaos Monkey Script (Giả lập sự cố)
Tôi sử dụng một script Python (chạy bên ngoài ứng dụng Java) để tự động terminate một EC2 instance ngẫu nhiên, nhằm kiểm tra xem ASG có "cứu" hệ thống không.

**File:** `chaos_test.py`
```python
import boto3
import random

def kill_random_instance():
    ec2 = boto3.client('ec2', region_name='ap-southeast-1')
    
    # 1. Lấy danh sách các instance đang chạy có tag Project=GameCard
    response = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Project', 'Values': ['GameCard']},
            {'Name': 'instance-state-name', 'Values': ['running']}
        ]
    )
    
    instances = []
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instances.append(instance['InstanceId'])

    if not instances:
        print("Không tìm thấy instance nào để 'giết'!")
        return

    # 2. Chọn ngẫu nhiên 1 nạn nhân
    victim_id = random.choice(instances)
    
    # 3. Terminate instance đó
    print(f"🔥 Đang terminate instance: {victim_id} để test Auto Scaling...")
    ec2.terminate_instances(InstanceIds=[victim_id])
    print("✅ Đã gửi lệnh terminate. Hãy kiểm tra Console xem instance mới có được tạo không!")

if __name__ == '__main__':
    kill_random_instance()