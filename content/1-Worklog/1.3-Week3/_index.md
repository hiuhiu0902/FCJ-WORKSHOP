---
title: "Week 3 Worklog"
date: "2025-09-22T09:00:00+07:00"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:
* Understand core Compute (EC2) and Storage (EBS, S3) services.
* Learn how to manage data persistence.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **EC2 Deep Dive:**<br>- Instance Types (T3, M5, etc.).<br>- AMI selection.<br>- Key Pairs & User Data scripts. | 22/09/2025 | 22/09/2025 | [AWS EC2 Docs](https://docs.aws.amazon.com/ec2/) |
| 2 | **EBS (Block Storage):**<br>- Volume types (gp3, io2).<br>- Attaching/Detaching volumes.<br>- Snapshots & Lifecycle Manager. | 23/09/2025 | 23/09/2025 | |
| 3 | **S3 (Object Storage):**<br>- Buckets & Objects.<br>- Storage Classes (Standard, IA, Glacier).<br>- Versioning & Bucket Policies. | 24/09/2025 | 24/09/2025 | |
| 4 | **Practice Lab:**<br>- Launch EC2 with User Data to install Apache.<br>- Create S3 bucket and host static `index.html`. | 25/09/2025 | 25/09/2025 | |
| 5 | **Review:**<br>- Differentiate when to use EBS vs S3. | 26/09/2025 | 26/09/2025 | |

### 🧠 Extra Knowledge: Ephemeral Store vs. EBS
I discovered that some EC2 instance types come with "Instance Store" (Ephemeral storage). This storage is physically attached to the host server and is very fast. **However**, if I stop or terminate the instance, **all data on the Instance Store is lost**. This is why for the database in my future project, I must use **EBS (Elastic Block Store)** because it persists independently of the instance's lifecycle.

### Achievements:
* Launched web servers using User Data bootstrapping (automating Apache install).
* Managed persistent storage with EBS volumes and practiced restoring from Snapshots.
* Hosted static assets on S3 and understood Object vs Block storage differences.