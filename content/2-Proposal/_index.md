---
title: "BĐP - Online Game Card Sales Platform"
date: "2025-09-10T10:00:00+07:00"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

{{% notice info %}}
Technical Proposal: A High-Availability and Secure E-commerce Platform (Game Card Sales) Solution.
{{% /notice %}}

### 1. Executive Summary

The Game Card Sales Platform is an e-commerce project aiming to build a complete website that allows users to register, log in, purchase, and instantly receive game card codes automatically.

This solution is designed with a multi-tier architecture, prioritizing security and performance. By leveraging top-tier AWS services, we propose deploying the **initial architecture across two Availability Zones** following High Availability (HA) best practices, while also outlining a roadmap to upgrade to three AZs for maximum reliability in the future.

### 2. The Problem

#### System-side Problem (Security):
The game card code inventory (unused) and transaction information are extremely sensitive data. Exposure or loss could lead to severe financial damage. Therefore, the database system must be protected at the highest level, inaccessible directly from the Internet.

#### User-side Problem (Experience & Trust):
Gamers are a user group that demands speed. They need a reliable platform that responds quickly, is secure for payments, and delivers the code immediately after a successful transaction. Any delay or interruption will damage the platform's reputation.

### 3. Solution Architecture

We propose a "Well-Architected" design deployed within a custom AWS VPC.

---

#### 3.1. Current Architecture (Phase 1): 2 AZs + 2 NAT Gateways (Recommended)

To ensure High Availability from the start, the initial architecture will be deployed across **2 Availability Zones** with a fully fault-tolerant network infrastructure.
<img src="/images/2-Proposal/architect2.jpg" alt="Sơ đồ kiến trúc 3 AZ 3 NAT" width="40%" height="800px">

* **Edge Layer:** (Unchanged)
    * Users access via **Amazon Route 53** -> **Amazon CloudFront (CDN)**.
    * **AWS WAF** and **AWS Shield** are integrated with CloudFront to block attacks.

* **Load Balancing Layer:**
    * Valid requests are forwarded to the **Application Load Balancer (ALB)**.
    * The ALB is placed in 2 Public Subnets, with each Public Subnet in a different AZ.

* **Application Tier:**
    * Web/API servers (EC2) are placed in 2 Private Subnets (one per AZ).
    * Servers are managed by an **Auto Scaling Group (ASG)**, ensuring at least 2 instances are running across 2 different AZs.
    * Sensitive information (API keys, DB passwords) is retrieved from **AWS Secrets Manager**.

* **Data Tier:**
    * **Amazon RDS (Database):** The Primary database is placed in a Private DB Subnet in AZ 1.
    * **RDS Replica:** A synchronous replica is placed in a Private DB Subnet in AZ 2 for Multi-AZ Failover.
    * **Amazon ElastiCache (Caching):** A Redis cluster (Multi-AZ) is deployed to reduce load on the database.
    * **Amazon S3:** Stores images and logs.

* **Networking & Monitoring Layer:**
    * **NAT Gateway:** **02 NAT Gateways** will be created. Each NAT Gateway is placed in a Public Subnet in each AZ (one in AZ 1, one in AZ 2).
    * The Route Tables for the Private Subnets in AZ 1 will route internet-bound traffic through the NAT Gateway in AZ 1. Likewise, Private Subnets in AZ 2 will use the NAT Gateway in AZ 2.
    * *Benefit:* This design eliminates a single point of failure (SPOF) and avoids cross-AZ data transfer costs.
    * **Bastion Host:** Placed in a Public Subnet (e.g., in AZ 1) for admins to SSH into private instances.
    * **Amazon CloudWatch & VPC Flow Logs:** Monitor and log the entire system.

---

#### 3.2. Upgrade Roadmap (Phase 2): 3 AZs + 3 NAT Gateways

After the platform is stable and generating revenue, we will upgrade to a 3-AZ architecture to achieve maximum reliability and fault tolerance.

 > <img src="/images/2-Proposal/architect.png" alt="Sơ đồ kiến trúc 3 AZ 3 NAT" width="60%">

* **Key Changes:**
    1.  Expand the VPC to support **3 Availability Zones (AZs)**.
    2.  Deploy **03 NAT Gateways**, placing one in a Public Subnet in each AZ (AZ 1, AZ 2, and AZ 3).
    3.  Update Route Tables for Private Subnets: Private Subnets in AZ 1 will go through the NAT Gateway in AZ 1, AZ 2 through NAT in AZ 2, and AZ 3 through NAT in AZ 3.
    4.  Expand the **Application Load Balancer** and **Auto Scaling Group (ASG)** to distribute the application load across all 3 AZs.
    5.  Expand **ElastiCache** (if needed) and consider adding an **RDS Read Replica** in the third AZ to improve read performance.

* **Benefits after Upgrade:**
    * **Enhanced Fault Tolerance:** This is the biggest benefit. If one AZ fails, the system still has two active AZs to handle 100% of the traffic (instead of just one in Phase 1). This helps the system handle the load better and reduces the risk of disruption during an outage.
    * **Better Load Distribution:** Traffic is spread across 3 zones instead of 2, improving overall performance and reducing the load on individual instances.

### 4. Expected Outcomes

* **Technical:** A stable, fault-tolerant (from Phase 1) E-commerce (MVP) platform with a clear upgrade path.
* **User Experience:** Fast page loads, smooth purchasing process, and high security.
* **Security:** Sensitive data is protected within multiple private network layers.

---

### 5. Budget Estimation (Monthly Costs)

A detailed cost analysis for both architectures.

#### 5.1. Current Architecture Cost (2 AZs + 2 NAT Gateways)

* **Application Load Balancer (ALB):** ~$20.00 /month (fixed cost).
* **NAT Gateway (2 Gateways):**
    * Hourly cost: 2 * $0.045/hr * 730 hrs/month = ~$65.70
    * Data processing (Est. 100GB): 100 * $0.045/GB = $4.50
    * *Total NAT Gateway:* **~$70.20 /month**
* **Amazon ElastiCache:** Min. 1 node `cache.t4g.micro` (Multi-AZ) = **~$17.00 /month**
* **AWS WAF:** Basic cost (1 Web ACL + 10 Rules) = **~$10.00 /month**
* **Services (Assuming first-year Free Tier):**
    * **EC2 (App Server):** 2 x `t3.micro` (1 free, 1 paid) = ~$7.60 /month
    * **RDS (Database):** 2 x `db.t3.micro` (Multi-AZ, not eligible for Free Tier) = ~$24.00 /month
    * **CloudFront, S3, Secrets Manager:** Very low cost, under **~$1.00 /month** at initial usage.

> **Total Estimated Cost (Phase 1):**
> ~$20 (ALB) + ~$70.20 (NAT) + ~$17 (Cache) + ~$10 (WAF) + ~$7.60 (EC2) + ~$24 (RDS) + ~$1 (Other)
> **= Approximately $145 - $150 USD /month**

#### 5.2. Upgrade Architecture Cost (3 AZs + 3 NAT Gateways)

* **Application Load Balancer (ALB):** ~$20.00 /month (Unchanged).
* **NAT Gateway (3 Gateways):**
    * Hourly cost: 3 * $0.045/hr * 730 hrs/month = ~$98.55
    * Data processing (Est. 100GB): 100 * $0.045/GB = $4.50
    * *Total NAT Gateway:* **~$103.05 /month**
* **Amazon ElastiCache:** ~$17.00 /month (Unchanged).
* **AWS WAF:** ~$10.00 /month (Unchanged).
* **Services (Post-Free Tier or scaled):**
    * **EC2 (App Server):** 3 x `t3.micro` = 3 * ~$7.60 = ~$22.80 /month
    * **RDS (Database):** Multi-AZ (Primary + Replica) = ~$24.00 /month (Unchanged, unless Read Replica is added)
    * **CloudFront, S3, etc.:** ~$1.00 /month

> **Total Estimated Cost (Phase 2):**
> ~$20 (ALB) + ~$103.05 (NAT) + ~$17 (Cache) + ~$10 (WAF) + ~$22.80 (EC2) + ~$24 (RDS) + ~$1 (Other)
> **= Approximately $195 - $200 USD /month**

---

### 6. Risk Assessment

* **Cost Risk (High):**
    * The cost of NAT Gateways (now 2), ALB, and RDS Multi-AZ is significant. The Phase 1 (2 NAT) architecture is more expensive than a 1 NAT alternative but provides much higher reliability.
    * **Mitigation:** Start with Phase 1. Set up **AWS Budgets** to alert when costs exceed a threshold. Shut down/delete resources (especially NAT GWs) during non-development hours.

* **Network Configuration Risk (High):**
    * The network architecture is complex (VPC, Subnets, Route Tables, WAF, Security Groups). Misconfiguration can lead to system failure or security vulnerabilities.
    * **Mitigation:** Design the diagram carefully. Test connectivity step-by-step (EC2 -> RDS, EC2 -> NAT -> Internet). Adhere to the "principle of least privilege" for Security Groups.

* **Overload Risk during Failure (Medium - Phase 1):**
    * The Phase 1 (2 AZ) architecture is fault-tolerant. However, if one AZ fails, 100% of the traffic will be directed to the single remaining AZ.
    * **Mitigation:** Configure the Auto Scaling Group (ASG) to "scale up" (add instances) quickly in the remaining AZ. Monitor performance closely. Plan the upgrade to Phase 2 (3 AZs) as the system grows.

---

### 7. Implementation Roadmap (3 Months)

This is a proposed roadmap to build capacity and deploy the project within 3 months.

#### Month 1: Foundation & Core Concepts

* **Goal:** Master fundamental AWS services.
* **Activities:**
    * **Week 1:** Learn AWS Basics & IAM (Users, Groups, Roles, Policies).
    * **Week 2:** Deep dive into Networking (VPC, Subnets (Public/Private), Route Tables, Internet Gateway, Security Groups).
    * **Week 3:** Learn basic Compute & Storage (EC2, EBS, S3).
    * **Week 4:** *Practice:* Manually build a simple VPC, create 1 Public and 1 Private Subnet, launch an EC2 instance in the Public Subnet, and SSH into it.

#### Month 2: Advanced Architecture & Design

* **Goal:** Understand HA services and finalize the design.
* **Activities:**
    * **Week 1:** Learn High Availability: ALB (Application Load Balancer) and Auto Scaling Groups (ASG).
    * **Week 2:** Learn Data Layer: RDS (Multi-AZ, Read Replicas) and ElastiCache.
    * **Week 3:** Learn Connectivity & Security: NAT Gateway, WAF, AWS Secrets Manager.
    * **Week 4:** *Design:* Finalize the detailed architecture diagram (Phase 1), define specific IP ranges for each Subnet, and write out Security Group rules.

#### Month 3: Implementation & Testing

* **Goal:** Deploy Phase 1 and conduct testing.
* **Activities:**
    * **Week 1 (Infrastructure Deployment):** Based on the design from Month 2, build the entire infrastructure (VPC, Subnets, 2 NAT Gateways, ALB, RDS Multi-AZ, ElastiCache, Bastion Host).
    * **Week 2 (Application Deployment):** Configure the ASG, deploy the application code to the EC2 instances. Store the DB password in Secrets Manager and configure the app to retrieve it.
    * **Week 3 (Connectivity & Security):** Configure Route 53 to point to CloudFront, and CloudFront to point to the ALB. Enable WAF with basic rules.
    * **Week 4 (Testing - Basic "Chaos Engineering"):**
        * Test HA: Manually "Terminate" one EC2 instance to see if the ASG automatically launches a new one.
        * Test DB Failover: Manually "Reboot (with failover)" the Primary RDS instance to see if the system automatically fails over to the replica.
        * Test NAT: Terminate the EC2 instance in AZ 1; ensure the instance in AZ 2 can still access the internet via NAT Gateway 2.