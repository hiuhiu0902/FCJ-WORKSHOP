---
title : "App Server & AMI Setup"
date : "2023-10-25T00:00:00+07:00"
weight : 2
chapter : false
pre : " <b> 5.5.2 </b> "
---

We will create an EC2 Instance to serve as a "Template". After completing the installation and configuration, we will create an AMI from it.

### 1. Initialization & Installation
1. Access **EC2 Console** -> **Launch instances**.
2. **Name:** `App-Server-Template`.
3. **OS Image:** Amazon Linux 2023.
4. **Instance type:** `t3.micro`.
5. **Key pair:** Select `workshop-key`.
6. **Network settings (Important):**
    * **VPC:** `3-tier-vpc`.
    * **Subnet:** `App-Private-Subnet-1a` (Must be Private - No Public IP).
    * **Auto-assign public IP:** Disable.
    * **Security Group:** Select `Backend-SG` (Allows port 8080 from ALB and 22 from Bastion).
7. **Advanced details -> IAM instance profile:** Select `SSM-Role` (Optional, for Systems Manager access).
8. Click **Launch instance**.

### 2. Environment Verification (Results)
After SSHing into the Private instance (via Bastion) and installing the Java/MySQL Client, run the commands to verify the Java version and test the connection to RDS.

Expected Result:

![App Server Setup Success](/images/5-Workshop/5.5-Compute-Setup/app-server-success.png)
*(Illustration: Terminal showing `java -version` and the `MySQL >` command prompt upon successful connection to RDS)*

### 3. Create Golden AMI
After ensuring the App Server is running stably, create an Image (`v1-backend-ami`) to use for the next step.