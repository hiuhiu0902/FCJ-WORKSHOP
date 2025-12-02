---
title: "Nhật ký Tuần 9"
date: "2025-11-03T09:00:00+07:00"
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu Tuần 9:
* Xây dựng hạ tầng cốt lõi theo thiết kế dự án Bán Thẻ Game.
* Thiết lập dự án **Spring Boot** và kết nối Cơ sở dữ liệu.

### Nhiệm vụ trong tuần:
| Ngày | Nhiệm vụ | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo |
| --- | --- | --- | --- | --- |
| 1 | **Xây dựng VPC:**<br>- VPC, 2 Public Subnet, 2 Private Subnet. | 03/11/2025 | 03/11/2025 | |
| 2 | **Cổng kết nối:**<br>- Triển khai IGW và 2 NAT Gateway (Sẵn sàng cao). | 04/11/2025 | 04/11/2025 | |
| 3 | **Cơ sở dữ liệu:**<br>- Khởi chạy RDS MySQL (Multi-AZ). | 05/11/2025 | 05/11/2025 | |
| 4 | **Backend Setup:**<br>- Khởi tạo Spring Boot project.<br>- Cấu hình JPA & Hibernate. | 06/11/2025 | 06/11/2025 | |
| 5 | **Kiểm tra:**<br>- Kiểm tra kết nối từ EC2 (Spring Boot) sang RDS. | 07/11/2025 | 07/11/2025 | |

### 🧠 Kiến thức mở rộng: JPA Specifications
Trong phần logic Backend, thay vì viết các câu lệnh SQL thô khó bảo trì, tôi đã sử dụng **Spring Data JPA Specifications**. Kỹ thuật này cho phép tôi xây dựng các truy vấn động (ví dụ: lọc sản phẩm theo tên, nhà mạng VÀ khoảng giá cùng lúc) một cách an toàn và hướng đối tượng.

### 💻 Backend Code: Tìm kiếm Sản phẩm Động
Dưới đây là cách tôi triển khai logic tìm kiếm nâng cao trong `ProductService.java` sử dụng `Specification` và `CriteriaBuilder`.

**File:** `ProductService.java`
```java
public Page<ProductResponse> searchProductsPublic(String keyword, String branchName, Long minPrice, Long maxPrice, Pageable pageable) {
    Specification<Product> spec = (root, query, cb) -> {
        List<Predicate> predicates = new ArrayList<>();

        // Tìm theo tên (Không phân biệt hoa thường)
        if (StringUtils.hasText(keyword)) {
            predicates.add(cb.like(cb.lower(root.get("name")), "%" + keyword.toLowerCase() + "%"));
        }

        // Lọc theo Nhà mạng (Branch)
        if (StringUtils.hasText(branchName)) {
            predicates.add(cb.equal(root.get("branch").get("name"), branchName));
        }

        // Lọc theo khoảng giá (Join bảng Variants)
        if (minPrice != null || maxPrice != null) {
            var variantJoin = root.join("variant");
            if (minPrice != null) {
                predicates.add(cb.greaterThanOrEqualTo(variantJoin.get("price"), minPrice));
            }
            if (maxPrice != null) {
                predicates.add(cb.lessThanOrEqualTo(variantJoin.get("price"), maxPrice));
            }
            query.distinct(true); // Tránh trùng lặp sản phẩm
        }

        return cb.and(predicates.toArray(new Predicate[0]));
    };

    return productRepository.findAll(spec, pageable).map(this::convertToProductResponse);
}