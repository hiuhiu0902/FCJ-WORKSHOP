---
title : "Initialize Bastion Host"
date : "2023-10-25T00:00:00+07:00"
weight : 1
chapter : false
pre : " <b> 5.5.1 </b> "
---

The Bastion Host (or Jump Server) is the only server in the system allowed to open SSH port (22) to the Internet.

### Implementation Steps

1. Access **EC2 Console** -> **Instances** -> **Launch instances**.
2. **Name:** `Bastion-Host`.
3. **OS Image:** Amazon Linux 2023 AMI (or Ubuntu).
4. **Instance type:** `t2.micro` or `t3.micro`.
5. **Key pair:** Select `workshop-key`.
6. **Network settings (Important):**
    * **VPC:** Select `3-tier-vpc`.
    * **Subnet:** Select `Public-Subnet-1a` (Must be Public).
    * **Auto-assign public IP:** **Enable**.
    * **Security Group:** Select `Bastion-SG` (Created in 5.4.1).
7. Click **Launch instance**.

### Verify Connection
Use Putty (Windows) or Terminal (Mac/Linux) to SSH into the Bastion Host using the provided Public IP.

![Bastion SSH Success](/images/5-Workshop/5.5-Compute-Setup/bastion-ssh-success.png)