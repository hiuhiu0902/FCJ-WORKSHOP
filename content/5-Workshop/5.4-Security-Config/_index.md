---
title : "Security Configuration"
date : "2023-10-25T00:00:00+07:00"
weight : 4
chapter : false
pre : " <b> 5.4 </b> "
---

# Security Configuration

In an **N-Tier** architecture, security is critical. Before deploying compute resources (EC2, RDS), we must establish network barriers to strictly control ingress and egress traffic.

In this section, we will set up a two-layer defense system:

1.  **Security Groups (Virtual Firewalls):**
    * We will apply **Security Group Chaining**.
    * **Rule:** A tier only accepts traffic from the tier immediately preceding it (e.g., Database only accepts connections from the Backend App, never from the Internet or Frontend).
    * **Data Flow:** `Internet` -> `Public ALB` -> `Frontend` -> `Internal ALB` -> `Backend` -> `Database`.

2.  **DB Subnet Group:**
    * This is a mandatory configuration for initializing **Amazon RDS**.
    * It groups the **Private Subnets** (where the Database resides), defining the secure network scope for the database and enabling Multi-AZ capabilities.

### Content
1. [Configure Security Groups](5.4.1-Security-Groups/)
2. [Create DB Subnet Group](5.4.2-DB-Subnet-Group/)