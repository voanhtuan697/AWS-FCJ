---
title : "Backend and runtime platform"
date: 2026-03-10
weight : 3
chapter : false
pre : " <b> 4.2.3 </b> "
---

## 4.2.3 Backend and runtime platform

This section describes **API ingress**, the **container runtime**, and **baseline observability** for the SpendWiseApp backend (**NestJS** behind the load balancer). **Database** details and secrets are in [4.2.4 Database](4.2.4-database/):

![SpendWiseApp backend VPC: WAF to Internet Gateway to ALB; ECS Fargate across two AZs; RDS PostgreSQL; optional bastion and NAT; ECR and ECS via PrivateLink and S3 gateway endpoint; CloudWatch logs and alarms; Amplify and Cognito above the VPC (overall architecture diagram)](/images/4-Workshop/4.2.3-backend-platform-vpc.png)

### WAF on the ALB

**Role:** **WAFv2** Web ACL attached directly to the **ALB ARN** (module **waf_alb**, enabled via **enable_waf**).

**Why we chose it:** Filters abusive requests at the **API edge** before ECS — can be toggled per environment (for example off in dev to save cost).

### ECR

**Role:** **Elastic Container Registry** stores backend images for ECS tasks to pull.

**Why we chose it:** Managed registry; pulls over **VPC endpoints** instead of self-hosting a registry.

### ECS Fargate

**Role:** **ECS service** runs tasks in **private app subnets** using the **image registered (pushed) to ECR**.

**Why we chose it:** No EC2 cluster to operate; scales with load; fits a **stateless** API behind the ALB.

### CloudWatch (monitoring)

**Role:** **CloudWatch Logs** for ECS container stdout/stderr; metrics and alarms baseline.

**Why we chose it:** Centralized logs and minimal alerting for operations — no separate logging stack to run.
