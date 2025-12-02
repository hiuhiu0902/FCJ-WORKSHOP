---
title: "Week 2 Worklog"
date: "2025-09-15T09:00:00+07:00"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

### Week 2 Objectives:
* Master AWS Networking fundamentals.
* Understand how to isolate resources using VPCs and Subnets.
* Control traffic flow with Route Tables and Security Groups.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **VPC & Subnets:**<br>- Learn CIDR notation.<br>- Understand Default vs. Custom VPC.<br>- Public vs. Private Subnets difference. | 15/09/2025 | 15/09/2025 | [AWS VPC Docs](https://docs.aws.amazon.com/vpc/) |
| 2 | **Connectivity:**<br>- Internet Gateway (IGW) setup.<br>- Route Tables (Main vs. Custom).<br>- Associating Subnets with Route Tables. | 16/09/2025 | 16/09/2025 | |
| 3 | **Security:**<br>- **Security Groups** (Stateful) vs. **NACLs** (Stateless).<br>- Create a Security Group allowing SSH (22) and HTTP (80) only from my IP. | 17/09/2025 | 17/09/2025 | |
| 4 | **Practice Lab:**<br>- Create a custom VPC.<br>- Create 1 Public Subnet and attach an IGW.<br>- Attempt to ping an instance. | 18/09/2025 | 18/09/2025 | |
| 5 | **Review:**<br>- Validate understanding of "Public" (has IGW route) vs "Private" (no IGW route). | 19/09/2025 | 19/09/2025 | |

### 🧠 Extra Knowledge: The "Stateful" Nature of Security Groups
While configuring the Security Group today, I learned an important distinction:
* **Security Groups are Stateful:** If I allow an *inbound* request on port 80, the response (outbound) is automatically allowed, regardless of outbound rules.
* **NACLs are Stateless:** I must explicitly allow traffic in *both* directions (Inbound and Outbound), otherwise the response packet gets dropped. This explains why my ping failed when I used NACLs but forgot the outbound rule!

### Achievements:
* Successfully created a custom VPC with logically divided subnets (CIDR /16 and /24).
* Configured a functioning Internet Gateway and Route Table.
* Verified network security using restricted Security Groups.