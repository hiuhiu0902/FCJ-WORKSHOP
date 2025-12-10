---
title : "Architecture Overview"
date : "2025-09-10T10:00:00+07:00"
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Overview
In this workshop, we will deploy an **N-Tier Microservices** architecture (with separate Frontend and Backend) designed for **High Availability**. The system includes:

1.  **Public Tier:** Contains the Public Load Balancer that receives traffic from the Internet.
2.  **Frontend Tier (Web):** Contains Web Servers that interact with users.
3.  **Backend Tier (App):** Contains App Servers that process business logic, receiving requests via an **Internal Load Balancer**.
4.  **Database Tier:** Contains RDS Multi-AZ located in the most secure private network zone.

#### Architecture Diagram
Below is the detailed architecture diagram we will build:

![Detailed Architecture Diagram](/images/5-Workshop/5.1-Workshop-overview/architecture-diagram.png)

#### Implementation Steps
1.  **Network:** Design a VPC with 4 network layers (Public, Web, App, DB).
2.  **Database:** Initialize RDS Multi-AZ (Primary & Standby).
3.  **Backend Compute:** Configure Internal ALB and Auto Scaling for the Backend.
4.  **Frontend Compute:** Configure Public ALB and Auto Scaling for the Frontend.