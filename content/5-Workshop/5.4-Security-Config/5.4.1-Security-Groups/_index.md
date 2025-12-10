---
title : "Configure Security Groups (N-Tier)"
date : "2023-10-25T00:00:00+07:00"
weight : 1
chapter : false
pre : " <b> 5.4.1 </b> "
---

Based on the architecture diagram, we need to establish a multi-layer firewall system (**Security Group Chaining**) to ensure traffic flows correctly from the outside in: **Internet -> Public ALB -> Frontend -> Internal ALB -> Backend -> Database**.

We will create the following Security Groups (SG) in the VPC:

### 1. SG for Bastion Host (Management)
Used to SSH into internal servers for troubleshooting.
* **Name:** `Bastion-SG`
* **Inbound:**
    * Type: SSH (22) | Source: `My IP` (Your current IP address).

### 2. SG for Public Load Balancer (External ALB)
This corresponds to component (5) in the diagram, receiving traffic from CloudFront/Users.
* **Name:** `Public-ALB-SG`
* **Inbound:**
    * Type: HTTP (80) | Source: `0.0.0.0/0` (Internet).
    * Type: HTTPS (443) | Source: `0.0.0.0/0`.

### 3. SG for Frontend Instances
This is the Auto Scaling Group (Frontend) in the diagram. It only accepts traffic from the Public ALB.
* **Name:** `Frontend-SG`
* **Inbound:**
    * Type: HTTP (80) | Source: `Public-ALB-SG` (Only allow from External ALB).
    * Type: SSH (22) | Source: `Bastion-SG` (Management).

### 4. SG for Internal Load Balancer
This ALB sits between the Frontend and Backend. It only accepts traffic from the Frontend.
* **Name:** `Internal-ALB-SG`
* **Inbound:**
    * Type: HTTP (80) | Source: `Frontend-SG` (Only Frontend can call this).

### 5. SG for Backend Instances (API)
This is the Auto Scaling Group (Backend) that processes business logic.
* **Name:** `Backend-SG`
* **Inbound:**
    * Type: Custom TCP (8080/AppPort) | Source: `Internal-ALB-SG` (Only accept from Internal ALB).
    * Type: SSH (22) | Source: `Bastion-SG`.

### 6. SG for Database (RDS)
The final layer containing data (Primary & Standby).
* **Name:** `Database-SG`
* **Inbound:**
    * Type: MySQL/Aurora (3306) | Source: `Backend-SG` (Strictly only allow Backend access).


{{% notice note %}}
**Note:** The diagram includes WAF and CloudFront. In this Security Group step, we are opening port 80/443 for the Public ALB to the world. When actually deploying WAF, you can restrict the `Public-ALB-SG` to only accept traffic from CloudFront's IP range (Managed Prefix List) for absolute security.
{{% /notice %}}