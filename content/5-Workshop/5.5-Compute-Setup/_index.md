---
title : "Compute Setup"
date : "2023-10-25T00:00:00+07:00"
weight : 5
chapter : false
pre : " <b> 5.5 </b> "
---

In this section, we will initialize the necessary EC2 instances. Since we are adhering to a secure 3-tier architecture, we cannot SSH directly into the App Server.

The connection flow will be:
**Laptop (Putty) -> Internet -> Bastion Host (Public) -> App Server (Private)**

We will perform 2 main steps:
1.  **Create Bastion Host:** A server located in the Public Subnet, acting as a "bridge".
2.  **Create & Configure App Server:** A server located in the Private Subnet. We will install the environment (Java/NodeJS, Database Client), deploy code, and verify connectivity to RDS.

After configuring the App Server, we will create an **AMI (Amazon Machine Image)** from this server to use for the Auto Scaling Group in the next section.