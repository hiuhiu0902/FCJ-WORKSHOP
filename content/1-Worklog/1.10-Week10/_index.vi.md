---
title: "Nhật ký Tuần 10"
date: "2025-11-10T09:00:00+07:00"
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu Tuần 10:
* Triển khai logic nghiệp vụ cốt lõi (Xử lý đơn hàng).
* Cấu hình Auto Scaling Group (ASG) cho ứng dụng Spring Boot.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **Launch Template:**<br>- Script cài Java 17 (Corretto) & chạy file Jar. | 10/11/2025 | 10/11/2025 | |
| 2 | **Tích hợp Secrets:**<br>- Cấu hình app lấy mật khẩu DB từ AWS. | 11/11/2025 | 11/11/2025 | |
| 3 | **Triển khai ASG:**<br>- Tạo ASG trải dài trên 2 AZs. | 12/11/2025 | 12/11/2025 | |
| 4 | **Kết nối:**<br>- Gắn ASG vào ALB Target Group. | 13/11/2025 | 13/11/2025 | |
| 5 | **Kiểm tra:**<br>- Test luồng mua hàng. | 14/11/2025 | 14/11/2025 | |

### 🧠 Kiến thức mở rộng: Tính toàn vẹn giao dịch (`@Transactional`)
Trong thương mại điện tử, **Race Conditions** (Điều kiện đua) là rủi ro lớn (ví dụ: 2 người cùng mua 1 chiếc thẻ cuối cùng trong cùng 1 mili-giây).
Tôi đã giải quyết vấn đề này bằng annotation `@Transactional` trong Spring Boot kết hợp với **Pessimistic Locking** (hàm `findAndLockCards` trong repository). Điều này đảm bảo khi người dùng bắt đầu thanh toán, các mã thẻ cụ thể sẽ bị khóa trong database cho đến khi giao dịch thành công hoặc bị hủy.

### 💻 Backend Code: Logic Xử Lý Đơn Hàng
Dưới đây là phương thức `createOrder` trong `OrderService.java`. Nó thể hiện cách tôi kiểm tra tồn kho, khóa thẻ và tạo đơn hàng trong một giao dịch nguyên tử (atomic transaction).

**File:** `OrderService.java`
```java
@Transactional // Đảm bảo Atomicity: Tất cả thành công hoặc tất cả thất bại
public Order createOrder(CreateOrderRequest request) {
    User user = authenticationService.getCurrentUser();
    Order order = new Order();
    order.setUser(user);
    order.setPayment(request.getPaymentMethod());
    order.setCreatedAt(LocalDateTime.now());
    order.setStatus(OrderStatus.PENDING);

    List<OrderItem> items = new ArrayList<>();
    Long total = 0L;

    for (OrderItemRequest item : request.getOrderItemRequests()) {
        ProductVariant variant = productVariantsRepository.findById(item.getVariantId())
                .orElseThrow(() -> new BadRequestException("Variant not found"));

        // Quan trọng: Khóa row trong DB để tránh bán trùng (Race condition)
        List<Storage> storagesToSell = stockRepository.findAndLockCards(
                CardStatus.UNUSED,
                variant.getVariantId(),
                PageRequest.of(0, item.getQuantity())
        );

        if (storagesToSell.size() < item.getQuantity()) {
            throw new BadRequestException("Not enough stock for variant: " + variant.getProduct().getName());
        }

        OrderItem orderItem = new OrderItem();
        orderItem.setOrder(order);
        orderItem.setProduct(variant.getProduct());
        orderItem.setQuantity(item.getQuantity());
        orderItem.setPrice(variant.getPrice());

        items.add(orderItem);
        total += variant.getPrice() * item.getQuantity();

        // Đánh dấu thẻ là PENDING ngay lập tức
        for (Storage storage : storagesToSell) {
            storage.setStatus(CardStatus.PENDING_PAYMENT);
            storage.setOrderItem(orderItem);
            stockRepository.save(storage);
        }
    }

    order.setOrderItems(items);
    order.setTotalAmount(total);
    return orderRepository.save(order);
}