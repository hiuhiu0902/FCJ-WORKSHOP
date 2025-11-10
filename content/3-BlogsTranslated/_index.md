---
title: "Translated Blogs"
date: "2025-09-09T19:53:52+07:00"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

This section will list and introduce the blogs you have translated.

### [Blog 1 - How Karrot Built its Feature Platform on AWS, Part 1: Motivation and Feature Serving](3.1-Blog1/)
This blog analyzes why Karrot, South Korea's leading local community, needed to build a new feature platform. It details the problems with the old architecture (server dependency, difficult to scale) and presents the new solution. This part focuses on the **Feature Serving** component, explaining how they use a multi-level cache (Local Cache, Amazon ElastiCache, Amazon DynamoDB) to achieve low latency for their ML recommendation system.

### [Blog 2 - How Karrot Built its Feature Platform on AWS, Part 2: Feature ingestion](3.2-Blog2/)
This second part dives into the **Feature Ingestion** component of the platform. It details two processes: **Stream Ingestion** (real-time data collection) using Amazon MSK and Amazon EKS, and **Batch Ingestion** (processing data in batches) using AWS Batch and AWS Fargate. The blog also shares the results, including the ability to handle high traffic and optimize costs using AWS services.

### [Blog 3 - Deploy LLMs on Amazon EKS using vLLM Deep Learning Containers](3.3-Blog3/)
This blog provides a detailed technical guide on how to deploy high-performance Large Language Models (LLMs) on Amazon EKS. The solution uses vLLM Deep Learning Containers (DLCs) for optimization, combined with P4d instances (A100 GPUs), Elastic Fabric Adapter (EFA) for high-speed networking, and Amazon FSx for Lustre for model weight storage. The post provides a step-by-step guide to setting up the environment, from creating the EKS cluster to deploying and testing the API.