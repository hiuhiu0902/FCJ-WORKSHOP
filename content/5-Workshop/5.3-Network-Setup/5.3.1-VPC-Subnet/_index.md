---
title : "Create VPC & Subnets"
date :  "2025-12-10T10:00:00+07:00"
weight : 1
chapter : false
pre : " <b> 5.3.1. </b> "
---

#### 1. Initialize VPC
The first step is to create a Virtual Private Cloud (VPC) to house all project resources.

1.  Open [Amazon VPC console](https://console.aws.amazon.com/vpc/).
2.  In the left navigation pane, choose **Your VPCs**.
3.  Click **Create VPC**.

![create vpc button](/images/5-Workshop/5.3-Network-Setup/create-vpc-button.png)

4.  On the **Create VPC** screen, configure the following:
    * **Resources to create:** Select **VPC only** (We will create subnets manually to understand the structure).
    * **Name tag:** `3-tier-vpc`
    * **IPv4 CIDR block:** `10.75.0.0/16` (Provides 65,536 IP addresses).
    * **Tenancy:** Default.
    * Click **Create VPC**.

![vpc config](/images/5-Workshop/5.3-Network-Setup/vpc-config.png)

5.  After creation, select **Actions** -> **Edit VPC settings**.
6.  Check **Enable DNS hostnames** (Allows AWS resources to have DNS names).
7.  Click **Save changes**.

---

#### 2. Create Subnets
We will create a total of **8 Subnets** distributed across 2 Availability Zones (AZ) in Singapore (`ap-southeast-1`).

Go to **Subnets** -> **Create subnet**. Select the `3-tier-vpc` you just created.

##### A. Create Subnets for Zone A (ap-southeast-1a)
In the **Subnet settings**, click "Add new subnet" to add the following 4 subnets:

| Subnet name | Availability Zone | IPv4 CIDR block | Purpose |
| :--- | :--- | :--- | :--- |
| `Public-Subnet-1a` | `ap-southeast-1a` | `10.75.1.0/24` | NAT Gateway, Bastion |
| `Web-Private-Subnet-1a` | `ap-southeast-1a` | `10.75.3.0/24` | Web Tier (Reserved) |
| `App-Private-Subnet-1a` | `ap-southeast-1a` | `10.75.5.0/24` | App Server (Java) |
| `DB-Private-Subnet-1a` | `ap-southeast-1a` | `10.75.7.0/24` | RDS Primary |

![subnet zone a](/images/5-Workshop/5.3-Network-Setup/subnet-creation-a.png)

##### B. Create Subnets for Zone B (ap-southeast-1b)
Continue adding 4 more subnets for Zone B to ensure High Availability (HA):

| Subnet name | Availability Zone | IPv4 CIDR block | Purpose |
| :--- | :--- | :--- | :--- |
| `Public-Subnet-1b` | `ap-southeast-1b` | `10.75.2.0/24` | NAT Gateway 2 |
| `Web-Private-Subnet-1b` | `ap-southeast-1b` | `10.75.4.0/24` | HA Web Tier |
| `App-Private-Subnet-1b` | `ap-southeast-1b` | `10.75.6.0/24` | HA App Server |
| `DB-Private-Subnet-1b` | `ap-southeast-1b` | `10.75.8.0/24` | RDS Standby |

![subnet zone b](/images/5-Workshop/5.3-Network-Setup/subnet-creation-b.png)

8.  Click **Create subnet**.

{{% notice success %}}
Double-check: You should see a list of 8 subnets with the status **Available**. Ensure the **Available IPv4 CIDR** matches the table above.
{{% /notice %}}