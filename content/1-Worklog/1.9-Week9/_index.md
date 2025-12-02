---
title: "Week 9 Worklog"
date: "2025-11-03T09:00:00+07:00"
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Week 9 Objectives:
* Build core infrastructure based on the Game Card Platform design.
* Setup **Spring Boot** Project structure and Database connectivity.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **VPC Build:**<br>- VPC, 2 Public Subnets, 2 Private Subnets. | 03/11/2025 | 03/11/2025 | |
| 2 | **Gateways:**<br>- Deploy IGW and 2 NAT Gateways (High Availability). | 04/11/2025 | 04/11/2025 | |
| 3 | **Database:**<br>- Launch RDS MySQL (Multi-AZ) in Private Subnets. | 05/11/2025 | 05/11/2025 | |
| 4 | **Backend Setup:**<br>- Init Spring Boot project.<br>- Configure JPA & Hibernate. | 06/11/2025 | 06/11/2025 | |
| 5 | **Verification:**<br>- Check connectivity between EC2 (Spring Boot) and RDS. | 07/11/2025 | 07/11/2025 | |

### 🧠 Extra Knowledge: JPA Specifications
In the backend logic, instead of writing raw SQL queries which are hard to maintain, I utilized **Spring Data JPA Specifications**. This allows me to build dynamic queries (e.g., filtering products by name, branch, AND price range simultaneously) in a type-safe and object-oriented way.

### 💻 Backend Code: Dynamic Product Search
Here is how I implemented the advanced search logic in `ProductService.java` using `Specification` and `CriteriaBuilder`.

**File:** `ProductService.java`
```java
public Page<ProductResponse> searchProductsPublic(String keyword, String branchName, Long minPrice, Long maxPrice, Pageable pageable) {
    Specification<Product> spec = (root, query, cb) -> {
        List<Predicate> predicates = new ArrayList<>();

        // Search by name (Case insensitive)
        if (StringUtils.hasText(keyword)) {
            predicates.add(cb.like(cb.lower(root.get("name")), "%" + keyword.toLowerCase() + "%"));
        }

        // Filter by Branch
        if (StringUtils.hasText(branchName)) {
            predicates.add(cb.equal(root.get("branch").get("name"), branchName));
        }

        // Filter by Price range (Join with Variants table)
        if (minPrice != null || maxPrice != null) {
            var variantJoin = root.join("variant");
            if (minPrice != null) {
                predicates.add(cb.greaterThanOrEqualTo(variantJoin.get("price"), minPrice));
            }
            if (maxPrice != null) {
                predicates.add(cb.lessThanOrEqualTo(variantJoin.get("price"), maxPrice));
            }
            query.distinct(true); // Avoid duplicates
        }

        return cb.and(predicates.toArray(new Predicate[0]));
    };

    return productRepository.findAll(spec, pageable).map(this::convertToProductResponse);
}