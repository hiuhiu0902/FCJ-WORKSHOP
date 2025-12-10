---
title : "Configure Load Balancers"
date : "2023-10-25T00:00:00+07:00"
weight : 7
chapter : false
pre : " <b> 5.7 </b> "
---

The N-Tier architecture requires 2 separate Application Load Balancers (ALBs).

### Part A: Create Target Groups
1. Go to **Target Groups** -> **Create target group**.
2. **Target Group 1 (For Backend):**
    * Name: `Backend-TG`
    * Port: `8080`.
    * VPC: `3-tier-vpc`.
3. **Target Group 2 (For Frontend):**
    * Name: `Frontend-TG`
    * Port: `80`.
    * VPC: `3-tier-vpc`.

### Part B: Create Internal ALB (For Backend)
This ALB sits between Frontend and Backend.

1. **Load Balancers** -> **Create load balancer** -> **Application Load Balancer**.
2. **Scheme:** **Internal** (Critical).
3. **Network mapping:** Select 2 **Private Subnets** (Web-Private-Subnet).
4. **Security groups:** Select `Internal-ALB-SG`.
5. **Listeners:** HTTP/80 -> Forward to `Backend-TG`.
6. Copy the **DNS Name** after creation.

### Part C: Create Public ALB (For Frontend)
This ALB accepts traffic from the Internet.

1. **Create load balancer** -> **Application Load Balancer**.
2. **Scheme:** **Internet-facing** (Critical).
3. **Network mapping:** Select 2 **Public Subnets**.
4. **Security groups:** Select `Public-ALB-SG`.
5. **Listeners:** HTTP/80 -> Forward to `Frontend-TG`.

The load balancing infrastructure is now ready for Auto Scaling attachment.