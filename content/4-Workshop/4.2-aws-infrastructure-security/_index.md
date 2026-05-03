---
title : "AWS infrastructure and security foundation"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

## 4.2 AWS infrastructure and security foundation

**Infrastructure and security.** **SpendWiseApp** on AWS Cloud follows a **three-tier pattern** suited to web workloads and **cost-conscious** scaling:

- **Presentation tier** — user experience and frontend hosting (for example Amplify).
- **Application tier** — APIs and business logic (for example ALB, ECS Fargate, ECR).
- **Data tier** — durable storage and databases (for example RDS PostgreSQL, Secrets Manager for DB credentials).

Implementation detail is split **by technical layer** (VPC → Frontend Hosting and user authentication → backend runtime → database), aligned with those modules:

1. [4.2.1 VPC and networking](4.2.1-vpc-network/) — `vpc`, VPC endpoints, `security_groups`.
2. [4.2.2 Frontend Hosting and user authentication](4.2.2-client-facing/) — `amplify`, `cognito`.
3. [4.2.3 Backend and runtime platform](4.2.3-backend-platform/) — `alb`, `waf_alb`, `route53_api`, `ecr`, `ecs`, `monitoring`.
4. [4.2.4 Database](4.2.4-database/) — `rds`, `db_password_secret`, `bastion`.