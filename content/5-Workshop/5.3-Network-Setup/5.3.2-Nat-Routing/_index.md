---
title : "NAT Gateway & Routing Config"
date :  "2025-12-10T10:00:00+07:00"
weight : 2
chapter : false
pre : " <b> 5.3.2. </b> "
---

#### 1. Create Internet Gateway (IGW)
To allow the VPC to communicate with the public Internet, we need an Internet Gateway.

1.  In the **VPC Console**, select **Internet gateways** from the left menu.
2.  Click **Create internet gateway**.
3.  **Name tag:** `3-tier-igw`.
4.  Click **Create internet gateway**.

![create igw](/images/5-Workshop/5.3-Network-Setup/create-igw.png)

5.  After creation, select **Actions** -> **Attach to VPC**.
6.  Select `3-tier-vpc` and click **Attach internet gateway**.

![attach igw](/images/5-Workshop/5.3-Network-Setup/attach-igw.png)

#### 2. Create NAT Gateways (High Availability)
To allow servers in Private Subnets (like the App Server installing Java) to connect to the Internet (for downloading libraries) without allowing inbound connections, we use a NAT Gateway. We will create 2 NAT Gateways for 2 Zones to ensure HA.

1.  Go to **NAT gateways** -> **Create NAT gateway**.

**Create NAT Gateway 1 (Zone A):**
* **Name:** `nat-gw-1a`
* **Subnet:** Select `Public-Subnet-1a`.
* **Elastic IP allocation ID:** Click **Allocate Elastic IP** to generate a new static IP.
* Click **Create NAT gateway**.

![create nat 1](/images/5-Workshop/5.3-Network-Setup/create-nat-1.png)

**Create NAT Gateway 2 (Zone B):**
* Repeat the steps above.
* **Name:** `nat-gw-1b`
* **Subnet:** Select `Public-Subnet-1b`.
* **Elastic IP allocation ID:** Click **Allocate Elastic IP**.
* Click **Create NAT gateway**.

{{% notice note %}}
Creating a NAT Gateway may take a few minutes to reach the **Available** state. Please wait a moment before proceeding to the next step.
{{% /notice %}}

#### 3. Configure Route Tables
Now we will direct the network traffic flow.

Go to **Route tables** -> **Create route table**.

##### A. Create Public Route Table
For public subnets.

1.  **Name:** `Public-RT`.
2.  **VPC:** `3-tier-vpc`.
3.  Click **Create**.
4.  In the **Routes** tab, select **Edit routes** -> **Add route**:
    * **Destination:** `0.0.0.0/0` (The entire Internet).
    * **Target:** Select `Internet Gateway` -> `3-tier-igw`.
    * Click **Save changes**.
5.  In the **Subnet associations** tab, select **Edit subnet associations**:
    * Select `Public-Subnet-1a` and `Public-Subnet-1b`.
    * Click **Save associations**.

![public route table](/images/5-Workshop/5.3-Network-Setup/public-rt.png)

##### B. Create Private Route Table for Zone A
For private subnets in Zone A, routing through NAT A.

1.  Create a new Route Table named: `Private-RT-1a`.
2.  **VPC:** `3-tier-vpc`.
3.  **Edit routes** -> **Add route**:
    * **Destination:** `0.0.0.0/0`.
    * **Target:** Select `NAT Gateway` -> `nat-gw-1a`.
4.  **Edit subnet associations**:
    * Select the 3 private subnets in Zone A: `Web-Private-Subnet-1a`, `App-Private-Subnet-1a`, `DB-Private-Subnet-1a`.
    * Click **Save**.

![private rt 1a](/images/5-Workshop/5.3-Network-Setup/private-rt-a.png)

##### C. Create Private Route Table for Zone B
For private subnets in Zone B, routing through NAT B.

1.  Create a new Route Table named: `Private-RT-1b`.
2.  **VPC:** `3-tier-vpc`.
3.  **Edit routes** -> **Add route**:
    * **Destination:** `0.0.0.0/0`.
    * **Target:** Select `NAT Gateway` -> `nat-gw-1b`.
4.  **Edit subnet associations**:
    * Select the 3 private subnets in Zone B: `Web-Private-Subnet-1b`, `App-Private-Subnet-1b`, `DB-Private-Subnet-1b`.
    * Click **Save**.

{{% notice success %}}
Congratulations! You have completed setting up the network infrastructure with High Availability. All servers in Private Subnets can now securely download security updates.
{{% /notice %}}