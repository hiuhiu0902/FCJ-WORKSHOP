---
title: "Week 6 Worklog"
date: "2025-10-13T09:00:00+07:00"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:
* Learn managed database services (RDS).
* Understand Multi-AZ vs Read Replicas.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **RDS Fundamentals:**<br>- DB Engines.<br>- Multi-AZ Deployments. | 13/10/2025 | 13/10/2025 | |
| 2 | **RDS Performance:**<br>- Read Replicas.<br>- RDS Security Groups. | 14/10/2025 | 14/10/2025 | |
| 3 | **ElastiCache:**<br>- Redis basics.<br>- Caching strategies. | 15/10/2025 | 15/10/2025 | |
| 4 | **Practice:**<br>- Launch RDS (MySQL).<br>- Connect from EC2. | 16/10/2025 | 16/10/2025 | |
| 5 | **Review:**<br>- Backup strategies. | 17/10/2025 | 17/10/2025 | |

### 🧠 Extra Knowledge: Multi-AZ vs. Read Replica
It's crucial not to confuse these two:
* **Multi-AZ** is for *High Availability*. Data replication is **Synchronous** (Real-time). You generally cannot read from the standby instance unless failover occurs.
* **Read Replica** is for *Performance*. Data replication is **Asynchronous** (slight delay). You can route heavy query traffic here to relieve the main database.

### Achievements:
* Deployed managed Relational Database (MySQL) without managing OS patches.
* Successfully connected a web server to the database using Security Group chaining.