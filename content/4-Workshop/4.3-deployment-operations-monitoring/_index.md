---
title: "Terraform deployment (by layer)"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

## 4.3 Terraform deployment (by layer)

This part describes **deploying SpendWiseApp infrastructure with Terraform** along the same axis as [4.2 AWS infrastructure and security foundation](../4.2-aws-infrastructure-security/) — from VPC and networking through frontend hosting and authentication, backend runtime, database, and finally **platform / edge / operations** components that sit between or beside a strict “frontend-only” or “backend-only” split.

### Terraform — role and why we use it

- **Infrastructure as code:** Desired state for VPCs, security groups, RDS, ECS, and related resources lives in versioned configuration. Teams can review diffs, reproduce environments, and align dev, staging, and production without manual drift.
- **Modular layout:** Terraform modules in the SpendWiseApp repo (for example vpc, ecs, rds, amplify, cognito) mirror the **layered architecture** in section 4.2, which keeps ownership clear and limits copy-paste across environments.
- **Dependencies and ordering:** Terraform resolves create/destroy order and passes outputs between modules (for example subnets into ECS, secrets into tasks). That fits stacks where **network → compute → data** must stay consistent.

Design choices (what each service does and how it is secured) stay in **section 4.2**. The subpages below focus on **how deployment is grouped for Terraform apply** in practice.

### Chapters

1. [4.3.1 Deploy VPC and networking](4.3.1-vpc-network/)
2. [4.3.2 Deploy frontend hosting and user authentication](4.3.2-frontend-hosting-auth/)
3. [4.3.3 Deploy backend](4.3.3-backend/)
4. [4.3.4 Deploy database](4.3.4-database/)
5. [4.3.5 Platform, edge & operations](4.3.5-platform-edge-ops/) — ALB, WAF, Route 53, Bastion, monitoring (cross-cutting; not owned entirely by “FE” or “BE” alone).
