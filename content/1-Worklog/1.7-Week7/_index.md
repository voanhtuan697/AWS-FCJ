---
title: "Week 7 Worklog"
weight: 7
chapter: false
pre: " <b>1.7.</b>"
---

### Week 7 Goals

- Understand and apply data and system security methods in AWS for the **expense management application**.
- Manage, configure, and optimize the **Amazon VPC** for the **backend–database–cache** system.
- Deploy **Backup & Disaster Recovery (DR)** solutions to ensure the safety of users' financial data.

**Time Period:** 10/11/2025 - 16/11/2025

---

### Weekly Task Overview

| Day | Activity                                                                                                                                                                                                                                                           | Start Date   | End Date     | Reference                               |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- | ------------ | ---------------------------------------- |
| 1 | - Start writing the **Proposal** for the SpendWiseApp project <br> + Complete the **Executive Summary**, **Problem Statement** (personal expense management problem) <br> + Describe the overall system architecture (Frontend – Backend – Database – AWS Services) <br> + Research the AWS services to be used: Cognito, ECS, RDS, S3, ElastiCache,... | 20/04/2026 | 20/04/2026 | - |
| 2 | - Build the backend architecture using the **Domain-Driven Design** model for SpendWiseApp <br> + Design entities: users, accounts, transactions, categories, budgets <br> + Implement the Repository & Unit of Work Pattern <br> + Run migration and seed PostgreSQL data <br> + Build business process services: <br> &emsp; · Transaction management (income/expense/transfer) <br> &emsp; • Manage financial accounts <br> &emsp; • Calculate dashboard (income/expenses, balance) | 21/04/2026 | 21/04/2026 | - |
| 3 | - Initialize frontend **WebApp (Next.js)** <br> + Build basic layout: Header, Navigation, Dashboard <br> + Build main pages: <br> &emsp; • Transaction list <br> &emsp; • Account management <br> &emsp; • Statistics dashboard <br> + Write Architecture & Technical Implementation for Proposal | 22/04/2026 | 22/04/2026 | - |
| 4 | - Designing an **AWS Architecture Diagram** for a SpendWiseApp system <br> + Identifying components: <br> &emsp; · ECS Fargate (NestJS Backend) <br> &emsp; · RDS PostgreSQL <br> &emsp; · S3 (invoice storage, export files) <br> &emsp; · CloudFront + Amplify (Frontend) <br> &emsp; · Cognito (Authentication) <br> &emsp; · ElastiCache Redis (Caching) <br> &emsp; · ALB, VPC, NAT Gateway <br> + Clarifying data flow: User → Frontend → Backend → Database/Cache/S3 | 23/04/2026 | 23/04/2026 | - |
| 5 | - Learn about **Disaster Recovery & Backup** <br> + Using AWS Backup to back up RDS <br> + Configuring Multi-AZ for PostgreSQL RDS <br> + Practicing Cross-Region Failover <br> + Ensuring the security of user financial data (transactions, budgets) | 24/04/2026 | 24/04/2026 | <https://aws.amazon.com/backup/> |

---

### Week 7 Achievements

- System Security:

  - Applying KMS data encryption for RDS and S3
  - Using Cognito to manage JWT authentication
  - Configuring WAF & Shield against attacks

- Mastering **Amazon Virtual Private Network (VPC)**:

  - Designing VPCs with:
  - Public subnet: ALB
  - Private subnet: ECS, RDS, Redis
  - Configuring Security Groups & NACLs
  - Understanding NAT Gateway mechanisms for outbound traffic

- Disaster Recovery:

  - Automatic backup with AWS Backup
  - Multi-AZ for database failover
  - Building a Disaster Recovery (DR) strategy for financial systems






