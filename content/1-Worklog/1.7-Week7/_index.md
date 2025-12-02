---
title: "Week 7 Worklog"
date: "2025-10-20T09:00:00+07:00"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:
* Secure private network outbound traffic.
* Manage secrets and WAF.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **NAT Gateway:**<br>- Allow Private Subnet internet access. | 20/10/2025 | 20/10/2025 | |
| 2 | **Secrets Manager:**<br>- Storing DB passwords.<br>- Rotation. | 21/10/2025 | 21/10/2025 | |
| 3 | **WAF:**<br>- Basic SQL injection protection. | 22/10/2025 | 22/10/2025 | |
| 4 | **Practice:**<br>- Create NAT Gateway.<br>- Verify Private EC2 internet access. | 23/10/2025 | 23/10/2025 | |
| 5 | **Review:**<br>- Cost analysis of NAT. | 24/10/2025 | 24/10/2025 | |

### 🧠 Extra Knowledge: The Cost of NAT Gateways
While implementing NAT Gateway, I learned it's one of the "hidden costs" in AWS. It charges not just for hourly usage (~$0.045/hr) but also for **Data Processing** ($0.045/GB). For the Game Card project, downloading heavy updates on private servers could be expensive. In the future, I might consider **VPC Endpoints** for accessing AWS services (like S3/DynamoDB) to avoid NAT processing fees.

### Achievements:
* Enabled secure outbound internet using NAT Gateway (private instances can update OS but cannot be reached from outside).
* Secured DB credentials with AWS Secrets Manager instead of hardcoding.