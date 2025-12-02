---
title: "Nhật ký Tuần 11"
date: "2025-11-17T09:00:00+07:00"
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu Tuần 11:
* Triển khai Bảo mật ứng dụng (Spring Security).
* Tối ưu hóa phân phối với CloudFront và WAF.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **Route 53:**<br>- Bản ghi Alias trỏ về ALB. | 17/11/2025 | 17/11/2025 | |
| 2 | **CloudFront:**<br>- Thiết lập Distribution (CDN). | 18/11/2025 | 18/11/2025 | |
| 3 | **WAF:**<br>- Gắn vào CloudFront để chặn tấn công. | 19/11/2025 | 19/11/2025 | |
| 4 | **Spring Security:**<br>- Cấu hình JWT Filter và AuthenticationManager. | 20/11/2025 | 20/11/2025 | |
| 5 | **Cấu hình cuối:**<br>- Chuyển hướng HTTPS. | 21/11/2025 | 21/11/2025 | |

### 🧠 Kiến thức mở rộng: Stateful vs Stateless Auth
Khác với xác thực dựa trên Session truyền thống (Stateful), tôi đã triển khai **Xác thực phi trạng thái (Stateless)** sử dụng JWT.
* **Cơ chế:** Khi người dùng đăng nhập qua `AuthenticationService`, server xác thực và cấp một Token có chữ ký số.
* **Lợi ích:** Server không cần lưu dữ liệu session trong RAM. Điều này cho phép Auto Scaling Group mở rộng ứng dụng theo chiều ngang thoải mái mà không lo về vấn đề "Sticky Sessions".

### 💻 Backend Code: Dịch vụ Xác thực An toàn
Dưới đây là logic `login` trong `AuthenticationService.java`. Nó ủy quyền việc kiểm tra mật khẩu cho `AuthenticationManager` của Spring Security để đảm bảo an toàn.

**File:** `AuthenticationService.java`
```java
public AccountResponse login(LoginRequest loginRequest) {
    try {
        // 1. Ủy quyền xác thực cho Spring Security Manager
        // Bước này sẽ kiểm tra username/password với DB (đã mã hóa BCrypt)
        authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(
                loginRequest.getUsername(),
                loginRequest.getPassword()
        ));
    } catch (BadRequestException e) {
        throw new BadRequestException("Invalid username or password");
    }

    // 2. Nếu hợp lệ, lấy thông tin user
    User user = authenticationRepository.findUserByUsername(loginRequest.getUsername());
    Member profile = memberRepository.findMemberByUser(user);
    
    // 3. Sinh JWT Token
    String token = tokenService.generateToken(user);
    
    AccountResponse response = new AccountResponse();
    response.setUsername(user.getUsername());
    response.setRole(user.getRole());
    response.setAddress(profile.getAddress());
    response.setToken(token); // Trả Token về cho Client
    return response;
}