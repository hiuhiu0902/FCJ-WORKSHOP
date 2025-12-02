---
title: "Week 10 Worklog"
date: "2025-11-10T09:00:00+07:00"
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Week 10 Objectives:
* Deploy the core business logic (Order Processing).
* Configure Auto Scaling Group (ASG) for the Spring Boot application.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **Launch Template:**<br>- Script to install Java 17 (Corretto) & Run Jar file. | 10/11/2025 | 10/11/2025 | |
| 2 | **Secrets Integration:**<br>- App config to fetch DB password. | 11/11/2025 | 11/11/2025 | |
| 3 | **ASG Deployment:**<br>- Create ASG spanning 2 AZs. | 12/11/2025 | 12/11/2025 | |
| 4 | **Connection:**<br>- Attach ASG to ALB Target Group. | 13/11/2025 | 13/11/2025 | |
| 5 | **Verification:**<br>- Test Purchase Flow. | 14/11/2025 | 14/11/2025 | |

### 🧠 Extra Knowledge: Transactional Integrity (`@Transactional`)
In e-commerce, **Race Conditions** are a major risk (e.g., two users buying the last card at the exact same millisecond).
I addressed this by using the `@Transactional` annotation in Spring Boot combined with **Pessimistic Locking** (`findAndLockCards` in repository). This ensures that once a user starts the checkout process, the specific card codes are locked in the database until the transaction commits or rolls back.

### 💻 Backend Code: Order Processing Logic
Below is the `createOrder` method in `OrderService.java`. It demonstrates how I verify stock, lock the items, and create the order in a single atomic transaction.

**File:** `OrderService.java`
```java
@Transactional // Ensures Atomicity: All succeed or all fail
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

        // Critical: Lock stock to prevent race conditions
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

        // Mark cards as PENDING immediately
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