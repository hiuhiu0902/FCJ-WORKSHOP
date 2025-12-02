---
title: "Week 8 Worklog"
date: "2025-10-27T09:00:00+07:00"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:
* Finalize architecture for final project.
* Document IP ranges and firewall rules.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **Diagramming:**<br>- Draw full HA Architecture (Multi-AZ). | 27/10/2025 | 27/10/2025 | [Draw.io](https://draw.io) |
| 2 | **IP Planning:**<br>- Define CIDR blocks (10.0.0.0/16). | 28/10/2025 | 28/10/2025 | |
| 3 | **Security Planning:**<br>- Write out SG Rules. | 29/10/2025 | 29/10/2025 | |
| 4 | **Review:**<br>- Peer review design. | 30/10/2025 | 30/10/2025 | |
| 5 | **Final Prep:**<br>- Deployment checklist. | 31/10/2025 | 31/10/2025 | |

### 🧠 Extra Knowledge: "Blast Radius" Reduction
In my design, I spread resources across 2 AZs (Availability Zones). This is to minimize the **"Blast Radius"**. If a disaster (fire, flood) hits one physical data center (AZ 1), my application in AZ 2 continues to run. This is a core concept of Reliability in the Well-Architected Framework.

### Achievements:
* Completed detailed architectural diagram for the Game Card Platform.
* Defined strict security group rules (Principle of Least Privilege).