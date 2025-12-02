---
title: "AWS Cloud Mastery Series #3"
date: "2025-11-29T08:30:00+07:00"
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

# Summary Report: “AWS Well-Architected – Security Pillar Workshop”

### Event Overview
**Date:** November 29, 2025
**Location:** AWS Vietnam Office
**Scale:** Over 350 cloud engineers, architects, and students.

### Event Objectives
- Comprehensive coverage of the security lifecycle on AWS.
- Bridging the gap between foundational architecture and real-world production security incidents.
- Building a sustainable cloud security community in Vietnam.

### Speaker
- **Hoang Van Kha** (and the FCAJ organizing team)

### Key Highlights

#### 1. Security Foundations
- **Core Philosophies:** Shared Responsibility Model, Zero Trust, and Defense in Depth.
- **Design Principle:** Implementing **"Least-privilege by design"** from day one, not as an afterthought.

#### 2. Identity & Access Management (IAM)
- **Identity-first Architecture:** shifting focus from network perimeter to identity perimeter.
- **Modern Controls:** Using **IAM Identity Center** for SSO, enforcing permission boundaries, and MFA.
- **Continuous Analysis:** Moving beyond static permissions to continuous access analysis.

#### 3. Detection & Monitoring
- **Org-level Visibility:** Implementing CloudTrail at the Organization level for comprehensive audit trails.
- **Full-layer Logging:** GuardDuty, Security Hub, and centralized logging.
- **Detection-as-Code:** Treating detection rules as software artifacts to automate threat identification.

#### 4. Infrastructure & Application Protection
- **Network Defense:** VPC segmentation strategies, Security Groups vs NACLs, and perimeter defense with AWS WAF, Shield, and Network Firewall.
- **AppSec:** Secure workload configuration, integrating security controls into CI/CD pipelines, and runtime protection.

#### 5. Data Protection
- **Encryption:** Enforcement at rest and in transit across all services.
- **Secrets Management:** Utilizing AWS KMS, Secrets Manager, and Parameter Store.
- **Guardrails:** Establishing strict data access guardrails to prevent unauthorized exposure.

#### 6. Incident Response (IR)
- **IR Lifecycle:** Aligning with AWS standards (Preparation -> Detection -> Recovery).
- **Forensics:** Techniques for isolation, snapshotting, and **forensic evidence collection**.
- **Automation:** Using Lambda and Step Functions for automated remediation to reduce mean time to recovery (MTTR).

### Key Takeaways for "First Cloud Journey"

#### The "Identity-First" Shift
- Security is no longer just about firewalls (Infrastructure Protection); it's about Identity. We must master IAM Roles and Identity Center as the primary defense layer.

#### Automation is Mandatory
- "Manual security is failed security." The shift towards **Detection-as-Code** and automated remediation (using Lambda) is essential for scaling security in modern cloud environments.

#### Community & Continuous Learning
- Security is a vast field. The roadmap ahead requires deep dives into **DevSecOps**, **Zero Trust**, and **Security Automation**.

### Event Experience

Attending the workshop at the AWS Vietnam Office was an energizing experience, witnessing over **100 participants** ranging from students to seasoned architects.

**What stood out:**
- **The Atmosphere:** The blend of learning and real-world experience created a high-quality technical discussion environment. It wasn't just theory; we discussed real production incidents and automation patterns.
- **Practicality:** The session successfully bridged the gap between abstract concepts (like Zero Trust) and concrete implementations (like using permission boundaries and KMS).
- **Community:** Seeing the strong engagement from the FCAJ team and the participants highlighted the growing strength of the cloud security community in Vietnam.

> **Next Steps:** I look forward to the upcoming phases focusing on **DevSecOps** and **Security Automation** to further deepen my technical expertise.