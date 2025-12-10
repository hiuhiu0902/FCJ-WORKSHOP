---
title: "Workshop"
date: "2025-09-09T19:53:52+07:00"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying Highly Available N-Tier Architecture

#### Overview

**N-Tier Architecture** is a common pattern for building enterprise applications that are scalable, secure, and highly available on AWS.

In this lab, you will learn how to deploy a complete web system consisting of a separate Frontend and Backend, utilizing core services such as VPC, EC2, RDS, and Application Load Balancers.

You will build a system with 3 distinct tiers, where each tier performs a specific role and is protected by strict network security layers:
+ **Presentation Tier (Public)** - Contains the Public Load Balancer and Frontend Auto Scaling Group. This tier receives traffic from Internet users via HTTP/HTTPS.
+ **Logic Tier (Private)** - Contains the Internal Load Balancer and Backend Auto Scaling Group. This tier processes business logic and only accepts connections from the Presentation Tier.
+ **Data Tier (Private)** - Contains the Amazon RDS Multi-AZ database. This tier stores persistent data, is the most heavily secured, and cannot be accessed directly from the Internet.

#### Content

1. [Introduction & Architecture](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Network Setup (VPC)](5.3-Network-Setup/)
4. [Security Configuration](5.4-Security-Config/)
5. [Compute Setup](5.5-Compute-Setup/)
6. [Database Setup (RDS)](5.6-Database-Setup/)
7. [Load Balancer Configuration](5.7-LoadBalancer-Setup/)
