---
title: "Week 5 Worklog"
date: "2025-10-06T09:00:00+07:00"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Week 5 Objectives:
* Master Scaling and Load Balancing.
* Understand "Self-Healing" applications.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **ELB:**<br>- ALB concepts, Target Groups, Health Checks. | 06/10/2025 | 06/10/2025 | |
| 2 | **ASG:**<br>- Launch Templates.<br>- Scaling Policies (CPU vs Schedule). | 07/10/2025 | 07/10/2025 | |
| 3 | **Practice:**<br>- Put 2 EC2s behind an ALB.<br>- Verify traffic distribution. | 08/10/2025 | 08/10/2025 | |
| 4 | **Practice:**<br>- Create ASG.<br>- Stress test CPU to trigger scale-out. | 09/10/2025 | 09/10/2025 | |
| 5 | **Review:**<br>- "Stateless" app concepts. | 10/10/2025 | 10/10/2025 | |

### 🧠 Extra Knowledge: Connection Draining
I learned about a setting called **"Deregistration Delay"** (Connection Draining) on the Target Group. When an EC2 instance is scaling in (shutting down), the ALB stops sending *new* requests to it but keeps the connection open for a few minutes (default 300s) to allow *ongoing* requests to finish. This prevents users from seeing error pages during a deployment or scale-down event.

### Achievements:
* Configured a Load Balancer to distribute traffic across 2 Availability Zones.
* Created an Auto Scaling Group that automatically adds servers when CPU utilization spikes > 50%.