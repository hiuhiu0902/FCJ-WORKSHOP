---
title : "Network Configuration (VPC)"
date :  "2025-12-10T10:00:00+07:00"
weight : 3
chapter : true
pre : " <b> 5.3. </b> "
---

### Network Overview
Welcome to the most critical part of the Workshop: **Building the Network Foundation**.

In this chapter, we will design a **High Availability (HA)** Virtual Private Cloud (VPC) in the **Singapore (ap-southeast-1)** Region. This network will host all your Web Servers, App Servers, and Databases.

#### Technical Objectives
By the end of this chapter, you will have:
* **1 VPC** with the CIDR block `10.75.0.0/16`.
* **8 Subnets** strategically distributed across 2 Availability Zones.
* **A Complete Routing System** with Internet Gateway and NAT Gateway.

#### Implementation Steps
This chapter is divided into 2 labs:

1.  [**Create VPC & Subnets:**](/en/5-workshop/5.3-network-setup/5.3.1-vpc-subnets/) creating the skeleton of the network.
2.  [**NAT & Routing Config:**](/en/5-workshop/5.3-network-setup/5.3.2-nat-routing/) connecting the veins to allow traffic flow.

![](/images/5-Workshop/5.3-Network-Setup/architecture-diagram.png)