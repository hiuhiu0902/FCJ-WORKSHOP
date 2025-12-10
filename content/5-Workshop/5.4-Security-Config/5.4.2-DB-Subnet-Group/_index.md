---
title : "Create DB Subnet Group"
date : "2023-10-25T00:00:00+07:00"
weight : 2
chapter : false
pre : " <b> 5.4.2 </b> "
---

In our architecture, the Database is deployed using a **Multi-AZ** model (Primary in one Subnet and Standby in another) to ensure **High Availability**.

To define which subnets the RDS instance is allowed to reside in, we need to create a **DB Subnet Group**.

### Execution Steps

1. Access **Amazon RDS Console** -> **Subnet groups**.
2. Click **Create DB subnet group**.

### Configuration Details

* **Name**: `workshop-db-subnet-group`
* **Description**: `Subnet group for Multi-AZ Architecture`
* **VPC**: Select the project VPC.

### Select Subnets (Important)

According to the diagram, the Database is located in the deepest network layer (Deepest Private Subnets). You must select the 2 Private Subnets specifically reserved for the DB:

1. **Availability Zones**: Select 2 AZs (e.g., `ap-southeast-1a` and `ap-southeast-1b`).
2. **Subnets**:
    * Select the Subnet ID for **Private Subnet 3** (Zone A).
    * Select the Subnet ID for **Private Subnet 4** (Zone B).

*(Note: Do not accidentally select the Frontend or Backend subnets)*.

3. Click **Create**.

After this step, when creating the RDS instance, you simply select `workshop-db-subnet-group`, and AWS will automatically distribute the Primary and Standby databases into the correct locations according to the architecture.