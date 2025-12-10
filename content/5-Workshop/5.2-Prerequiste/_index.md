---
title : "Prerequisites"
date :  "2025-09-10T10:00:00+07:00"
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

#### 1. Select Region
In this workshop, we will use the **Singapore (ap-southeast-1)** Region.
To ensure High Availability, resources will be distributed across 2 Availability Zones (AZs):
* `ap-southeast-1a`
* `ap-southeast-1b`

Please ensure you have selected the correct Region in the top-right corner of the console.

![Select Singapore Region](/images/5-Workshop/5.2-Prerequiste/select-region.png)

#### 2. Create EC2 Key Pair
To SSH into EC2 instances (Web Server, App Server, Bastion Host), you need to generate a Key Pair.

1.  Go to **EC2 Console** -> Left menu select **Key Pairs**.
2.  Click **Create key pair**.
3.  Enter details:
    * **Name:** `workshop-key`
    * **Key pair type:** `RSA`
    * **Private key file format:**
        * Select `.ppk` if you use **Windows (Putty)**.
        * Select `.pem` if you use **MacOS/Linux** or Windows PowerShell.
4.  Click **Create key pair** and save the file to your computer (Do not lose this file).

![Create Key Pair](/images/5-Workshop/5.2-Prerequiste/create-keypair.png)

#### 3. IAM Permissions Configuration (Optional)

Create a Role to allow EC2 to access Systems Manager (for connection without opening port 22).
* Go to **IAM** -> **Roles** -> **Create role**.
* Service: **EC2**.
* Policy: `AmazonSSMManagedInstanceCore`.
* Role name: `SSM-Role`.

If you are using a restricted lab account (not a Root/Admin account), ensure your User has the following Policy attached to perform actions like creating VPC, EC2, RDS, and Auto Scaling:

*(Note: If you are using a personal account with AdministratorAccess, you can skip this step)*

<details>
<summary><b>Click to view JSON Policy details</b></summary>

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "cloudformation:*",
                "cloudwatch:*",
                "ec2:*",
                "elasticloadbalancing:*",
                "autoscaling:*",
                "rds:*",
                "s3:*",
                "iam:CreateServiceLinkedRole",
                "iam:ListRoles",
                "iam:PassRole",
                "secretsmanager:*"
            ],
            "Resource": "*"
        }
    ]
}