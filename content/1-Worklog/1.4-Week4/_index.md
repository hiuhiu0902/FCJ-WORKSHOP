---
title: "Week 4 Worklog"
date: "2025-09-29T09:00:00+07:00"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:
* Apply all knowledge from Month 1.
* Manually build a functional environment from scratch.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **Planning:**<br>- Sketch network diagram for 2-tier app (Public/Private). | 29/09/2025 | 29/09/2025 | |
| 2 | **Network Build:**<br>- Build VPC, 1 Public Subnet, 1 Private Subnet.<br>- Configure Route Tables. | 30/09/2025 | 30/09/2025 | |
| 3 | **Compute Build:**<br>- Launch EC2 in Public (Web Server).<br>- Launch EC2 in Private (Backend). | 01/10/2025 | 01/10/2025 | |
| 4 | **Access Testing:**<br>- SSH into Public EC2.<br>- Attempt SSH from Public to Private (Jumpbox). | 02/10/2025 | 02/10/2025 | |
| 5 | **Cleanup:**<br>- Terminate instances, delete NATs/Gateways. | 03/10/2025 | 03/10/2025 | |

### 🧠 Extra Knowledge: The "Bastion Host" Pattern
While trying to access the Private EC2, I couldn't connect directly because it has no Public IP. I learned about the **Bastion Host (Jump Box)** pattern.
* It acts as a secure gateway. I SSH into the Bastion (Public Subnet) first, and from there, I SSH into the Private instance.
* *Security Tip:* I should restrict the Security Group of the Bastion to allow SSH only from my specific home IP address, not `0.0.0.0/0`.

### Achievements:
* Built a complete networking environment manually without using the VPC Wizard.
* Successfully demonstrated the "Bastion Host" concept by accessing a private instance securely.