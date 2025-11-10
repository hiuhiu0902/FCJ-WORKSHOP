---
title: "BDP - Online Game Card Sales Platform"
date: "2025-09-10T10:00:00+07:00"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

{{% notice info %}}
Technical Proposal Document: A Highly Available and Secure E-commerce Platform (Game Card Sales).
{{% /notice %}}

### 1. Executive Summary

The Online Game Card Sales Platform is an e-commerce project aiming to build a complete website allowing users to register, log in, purchase, and instantly receive game card codes automatically.

This solution is designed with a multi-tier architecture, prioritizing security and performance. By utilizing top-tier AWS services, we propose an **initial architecture deployed on two Availability Zones** to optimize costs, while outlining a future upgrade roadmap to three AZs to ensure maximum reliability.

### 2. Problem Statement

#### System-Side Problem (Security):
The inventory of game card codes (unused) and transaction information is extremely sensitive data. Exposure or loss could lead to severe financial damage. Therefore, the database system must be protected at the highest level, inaccessible directly from the Internet.

#### User-Side Problem (Experience & Trust):
Gamers are a user group that demands speed. They need a reliable platform that responds quickly, is secure for payments, and delivers the code immediately after a successful transaction. Any delay or interruption will erode the platform's credibility.

### 3. Solution Architecture

We propose a "Well-Architected" framework deployed within a custom AWS VPC.

---

#### 3.1. Current Architecture (Phase 1): 2 AZ + 1 NAT Gateway

To balance cost and availability, the initial architecture will be deployed across **2 Availability Zones (AZs)**.

 > <img src="/images/2-Proposal/architect.png" alt="" width="80%">
* **Edge Layer:** (Unchanged)
    * Users access via **Amazon Route 53** -> **Amazon CloudFront (CDN)**.
    * **AWS WAF** and **AWS Shield** are integrated with CloudFront to block attacks.

* **Load Balancing Layer:**
    * Valid requests are forwarded to an **Application Load Balancer (ALB)**.
    * The ALB is placed in 2 Public Subnets, one in each AZ.

* **Application Tier:**
    * Web/API servers (EC2) are placed in 2 Private Subnets (one per AZ).
    * Servers are managed by an **Auto Scaling Group (ASG)**, ensuring at least 2 instances are running in 2 different AZs.
    * Sensitive information (API keys, DB passwords) is retrieved from **AWS Secrets Manager**.

* **Data Tier:**
    * **Amazon RDS (Database):** The Primary database is in a Private DB Subnet in AZ 1.
    * **RDS Replica:** A synchronous Replica is placed in a Private DB Subnet in AZ 2 for Multi-AZ Failover.
    * **Amazon ElastiCache (Caching):** A cache cluster (Redis) is deployed (potentially Multi-AZ) to reduce database load.
    * **Amazon S3:** Stores images, logs.

* **Networking & Monitoring Layer:**
    * **NAT Gateway:** Only **01 single NAT Gateway** is created and placed in the Public Subnet of AZ 1.
    * All Route Tables for *both* Private Subnets (in AZ 1 and AZ 2) will route Internet-bound traffic through this single NAT Gateway.
    * **Bastion Host:** Placed in a Public Subnet (e.g., in AZ 1) for admin SSH access.
    * **Amazon CloudWatch & VPC Flow Logs:** Monitor and log the entire system.

---

#### 3.2. Upgrade Roadmap (Phase 2): 3 AZ + 3 NAT Gateway

After the platform is stable and generating revenue, we can upgrade to a 3 AZ architecture for maximum reliability.

 > <img src="/images/2-Proposal/architect.png" alt="" width="80%">

* **Key Changes:**
    1.  Expand the VPC to support **3 Availability Zones (AZs)**.
    2.  Deploy **03 NAT Gateways**, one in a Public Subnet in each AZ.
    3.  Update Route Tables: The Private Subnet in AZ 1 will use the NAT Gateway in AZ 1, the Private Subnet in AZ 2 will use the NAT in AZ 2, etc.

* **Benefits After Upgrading:**
    * **Eliminates Single Point of Failure (SPOF):** In the Phase 1 architecture, if the single NAT Gateway (or its AZ) fails, the entire system loses Internet access (cannot call payment APIs). The 3 NAT Gateway architecture ensures a failure in one AZ does not affect the other two.
    * **Increases High Availability:** Achieves the highest level of fault tolerance recommended by AWS for critical systems.

### 4. Expected Outcomes

* **Technically:** A stable, operational E-commerce (MVP) platform that is fault-tolerant (2 AZ level) with a clear upgrade path to 3 AZs.
* **User Experience:** Fast page load times, with a smooth and secure purchasing process.
* **Security:** Sensitive data is protected within multiple private network layers.

---

### 5. Budget Estimation (Monthly Cost)

A detailed cost analysis for both architectures.

#### 5.1. Cost - Current Architecture (2 AZ + 1 NAT Gateway)

* **Application Load Balancer (ALB):** ~$20.00 /month (fixed cost).
* **NAT Gateway (1 Gateway):**
    * Hourly cost: 1 * $0.045/hr * 730 hrs/mo = ~$32.85
    * Data processing (Assuming 100GB): 100 * $0.045/GB = $4.50
    * *Total NAT Gateway:* **~$37.35 /month**
* **Amazon ElastiCache:** Min. 1 node `cache.t4g.micro` (Multi-AZ) = **~$17.00 /month**
* **AWS WAF:** Basic cost (1 Web ACL + 10 Rules) = **~$10.00 /month**
* **Services in Free Tier (Assuming first year):**
    * **EC2 (App Server):** Free Tier provides 750 hrs `t2.micro` (or `t3.micro`). We need 2 servers (for 2 AZs), so 1 is free, 1 is paid.
        * 1 x `t3.micro` (free) = $0.00
        * 1 x `t3.micro` (~$0.0104/hr) = ~$7.60 /month
    * **RDS (Database):** Free Tier provides 750 hrs `db.t2.micro` (Single-AZ). Our architecture is Multi-AZ (Primary + Replica).
        * 1 x `db.t3.micro` (Primary) = ~$12.00 /month
        * 1 x `db.t3.micro` (Replica) = ~$12.00 /month
        * (Note: Free Tier cannot be used for RDS Multi-AZ)
    * **CloudFront, S3, Secrets Manager:** Very low cost, under **~$1.00 /month** at initial usage.

> **Total Estimated Cost (Phase 1):**
> ~$20 (ALB) + ~$37.35 (NAT) + ~$17 (Cache) + ~$10 (WAF) + ~$7.60 (EC2) + ~$24 (RDS) + ~$1 (Other)
> **= Approx. $115 - $120 USD /month**

#### 5.2. Cost - Upgrade Architecture (3 AZ + 3 NAT Gateway)

* **Application Load Balancer (ALB):** ~$20.00 /month (Unchanged).
* **NAT Gateway (3 Gateways):**
    * Hourly cost: 3 * $0.045/hr * 730 hrs/mo = ~$98.55
    * Data processing (Assuming 100GB, split): 100 * $0.045/GB = $4.50
    * *Total NAT Gateway:* **~$103.05 /month**
* **Amazon ElastiCache:** ~$17.00 /month (Unchanged).
* **AWS WAF:** ~$10.00 /month (Unchanged).
* **Services (Post-Free Tier or expanded):**
    * **EC2 (App Server):** 3 `t3.micro` servers = 3 * ~$7.60 = ~$22.80 /month
    * **RDS (Database):** Multi-AZ (Primary + Replica) = ~$24.00 /month (Unchanged)
    * **CloudFront, S3, etc:** ~$1.00 /month

> **Total Estimated Cost (Phase 2):**
> ~$20 (ALB) + ~$103.05 (NAT) + ~$17 (Cache) + ~$10 (WAF) + ~$22.80 (EC2) + ~$24 (RDS) + ~$1 (Other)
> **= Approx. $195 - $200 USD /month**

---

### 6. Risk Assessment

* **Cost Risk (High):**
    * Costs for NAT Gateway, ALB, and RDS Multi-AZ are significant. The Phase 2 architecture nearly doubles the cost.
    * **Mitigation:** Start with Phase 1. Set up **AWS Budgets** to alert on cost thresholds. Terminate/delete resources (especially NAT GWs) when not in development.

* **Network Configuration Risk (High):**
    * This is a complex network architecture (VPC, Subnets, Route Tables, WAF). A single misconfiguration can lead to system failure or security vulnerabilities.
    * **Mitigation:** Design the diagram carefully. Test connectivity step-by-step (EC2 -> RDS, EC2 -> NAT -> Internet).

* **Single Point of Failure Risk (Medium - Phase 1):**
    * The Phase 1 architecture uses a single NAT Gateway. If the AZ containing this NAT Gateway fails, private instances will lose outbound Internet access (e.g., for payment APIs).
    * **Mitigation:** This is an acceptable risk for initial cost savings. Plan to upgrade to Phase 2 (3 AZ + 3 NAT) when the system requires higher reliability.