---
title: "Week 8 Worklog"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Goals

- Enhance knowledge of **Solutions Architecture** on AWS.
- Optimize and refine the SpendWiseApp system architecture.
- Study **AWS Best Practices** and **Well-Architected Framework**.

**Time Period:** 27/04/2026 - 01/05/2026

---

### Weekly Task Overview

| Day | Activity                                                                                                                                                                                                                                             | Start Date  | End Date    | Reference                                    |
| --- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ----------- | --------------------------------------------- |
| 1 | - Integrate **S3 & Redis** into the system <br> + Upload invoice images (receipts) to S3 using presigned URLs <br> + Cache dashboard data using Redis <br> + Optimize API performance (reduce DB queries) | 27/04/2026 | 27/04/2026 | <https://aws.amazon.com/s3/> |
| 2 | - Outline the complete system architecture <br> + Refer to e-commerce and financial app architectures <br> + Define the stack: <br> &emsp; · Frontend: Next.js (Amplify) <br> &emsp; · Backend: NestJS (ECS Fargate) <br> &emsp; · DB: RDS PostgreSQL <br> &emsp; · Cache: Redis <br> &emsp; · Auth: Cognito | 28/04/2026 | 28/04/2026 | <https://aws.amazon.com/architecture/> |
| 3 | - Data & Service Optimization <br> + Database Normalization using **3NF** <br> + Query Transaction & Dashboard Optimization <br> + Budget Logic **tracking & alert** | 29/04/2026 | 29/04/2026 | - |
| 4 | - Finalize Proposal <br> + Solution Architecture <br> + Implementation Plan <br> + Cost Estimation <br> + Detailed description of modules: <br> &emsp; · Transactions <br> &emsp; · Budget <br> &emsp; · Reports <br> &emsp; · Authentication | 30/04/2026 | 30/04/2026 | - |
| 5 | - Review and optimize architecture <br> + Ensure **High Availability** <br> + Optimize costs **(EC2 vs Fargate vs Serverless)** <br> + Check compatibility between AWS services | 01/04/2026 | 01/04/2026 | <https://aws.amazon.com/whitepapers/> |

---

### Week 8 Achievements

- Advanced Solutions Architecture:

  - Designing a production-ready architecture for a financial application
  - Understanding the model:
    - Microservices
    - Event-driven (Lambda + EventBridge)
    - Serverless

- Integrating advanced AWS services:

  - Using Lambda to handle budget alerts
  - EventBridge for scheduling
  - SES for sending email alerts

- AWS Well-Architected Framework:

  - Applying the 5 pillars:
  - Security (Cognito, IAM, encryption)
  - Reliability (Multi-AZ, backup)
  - Performance (Redis cache)
  - Cost Optimization (resource optimization)
  - Operational Excellence (CI/CD, monitoring)

- Week 8 Summary:

  - Completing the production-ready SpendWiseApp system architecture
  - Combining AWS theory and practice
  - Preparing for the next phase:
    - Automation (IaC)
    - DevOps
    - Real-world deployment

