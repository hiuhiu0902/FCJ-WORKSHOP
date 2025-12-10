---
title: "Initialize Database (RDS)"
date: 2023-10-25
weight: 6
---

# Initialize Amazon RDS

In this section, we will initialize a database using the **Amazon RDS** service. This database will be placed in a **Private Subnet** and will only allow access from the App Server.

## Architecture

* **Service:** Amazon RDS (Relational Database Service).
* **Engine:** MySQL or PostgreSQL (depending on your script).
* **Network:** Placed in the `DB Subnet Group` (created in step 5.4.2).
* **Security:** Uses `DB-Security-Group` (created in step 5.4.1) to allow traffic on port 3306/5432 from the App Server.

---

## Implementation Steps

### 1. Engine Configuration
* Access the **AWS Console** -> Search for **RDS**.
* Click **Create database**.
* **Choose a database creation method:** Standard create.
* **Engine options:** Select `MySQL` (or PostgreSQL depending on your project).
* **Templates:** Select `Free tier` (or Dev/Test) to save costs for this lab.

### 2. Settings Configuration
* **DB instance identifier:** `workshop-db`
* **Master username:** `admin`
* **Master password:** Enter a strong password (**Note:** Remember this password to configure it in the App Server later).

### 3. Instance & Storage Configuration
* **DB instance class:** `db.t3.micro` (For Free tier).
* **Storage:** Keep defaults (usually gp2 or gp3, 20GB).

### 4. Connectivity Configuration (Important)
This step connects the resources created in sections 5.3 and 5.4:

* **VPC:** Select the created VPC (e.g., `workshop-vpc`).
* **DB Subnet Group:** Select `workshop-db-subnet-group` (Created in step 5.4.2).
* **Public access:** Select **No** (Since the DB resides in a Private Subnet).
* **VPC security group:**
    * Select **Choose existing**.
    * Remove the `default` security group.
    * Select `workshop-db-sg` (Created in step 5.4.1).
* **Availability Zone:** Select `No preference` or specify a specific Zone.

### 5. Authentication
* Select **Password authentication**.

### 6. Create Database
* Scroll to the bottom and click **Create database**.
* The initialization process may take 5-10 minutes.

---

## Verify Results

Once the status changes to **Available**:
1. Click on the DB identifier (`workshop-db`).
2. In the **Connectivity & security** tab, copy the **Endpoint** value (e.g., `workshop-db.xxxx.us-east-1.rds.amazonaws.com`).
3. This is the host address you will use to configure the App Server in the previous step.