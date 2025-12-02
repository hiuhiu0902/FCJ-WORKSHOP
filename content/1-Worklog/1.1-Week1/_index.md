---
title: "Week 1 Worklog"
date: "2025-09-09T19:53:52+07:00"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

### Week 1 Objectives:

* Connect and get acquainted with members of First Cloud Journey.
* Understand AWS Global Infrastructure (Regions, Availability Zones).
* Master Identity and Access Management (IAM) concepts: Users, Groups, Roles, and Policies.
* Secure the AWS Root account and set up proper administrative access.

### Tasks to be carried out this week:

| Day | Task | Start Date | Completion Date | Reference Material |
| :--- | :--- | :--- | :--- | :--- |
| 1 | - Get acquainted with FCJ members <br> - Read and take note of the rules of the business <br> - Overview of AWS Cloud Concepts & Global Infrastructure | 09/06/2025 | 09/06/2025 | |
| 2 | - Create AWS Free Tier account <br> - **Security First:** <br>&emsp; + Secure Root User with MFA <br>&emsp; + Create an Account Budget (Billing Alarm) <br>&emsp; + Understand Shared Responsibility Model | 09/07/2025 | 09/07/2025 | <https://aws.amazon.com/getting-started/> <br><br> https://www.youtube.com/@AWSStudyGroup |
| 3 | - **Deep Dive IAM (Identity):** <br>&emsp; + Understand IAM Users vs. Root User <br>&emsp; + Understand IAM Groups <br> - **Practice:** <br>&emsp; + Create an "Admin" Group with AdministratorAccess <br>&emsp; + Create a daily IAM User and add to Admin Group <br>&emsp; + Set up a custom Password Policy | 09/08/2025 | 09/08/2025 | <https://docs.aws.amazon.com/iam/> |
| 4 | - **Deep Dive IAM (Access):** <br>&emsp; + Understand IAM Policies (JSON structure) <br>&emsp; + Managed Policies vs. Inline Policies <br>&emsp; + Understand IAM Roles (Service Roles vs. Cross-account) <br> - **Practice:** <br>&emsp; + Write a custom Policy (e.g., allow S3 read-only) <br>&emsp; + Create a Role for an AWS Service (theory preparation) | 09/09/2025 | 09/09/2025 | <https://cloudjourney.awsstudygroup.com/> |
| 5 | - **AWS CLI & Access Keys:** <br>&emsp; + Install AWS CLI <br>&emsp; + Generate Access Keys for your IAM User <br>&emsp; + Configure CLI (`aws configure`) <br> - **Review & Cleanup:** <br>&emsp; + Verify MFA is active on Root and IAM Users <br>&emsp; + Test CLI access (e.g., `aws sts get-caller-identity`) | 09/10/2025 | 09/10/2025 | <https://aws.amazon.com/cli/> |

### Week 1 Achievements:

* Successfully got acquainted with members of the First Cloud Journey.
* Created a fully secured AWS Free Tier account with Billing Alarms set.
* Secured the Root User with Multi-Factor Authentication (MFA).
* Gained mastery of IAM fundamentals:
    * Created an administrative IAM User (stopped using Root for daily tasks).
    * Created IAM Groups to manage permissions efficiently.
    * Understood the structure of JSON Policies and the Principle of Least Privilege.
* Installed and configured the AWS CLI.
* Successfully connected to the AWS account via terminal and verified identity using `sts get-caller-identity`.