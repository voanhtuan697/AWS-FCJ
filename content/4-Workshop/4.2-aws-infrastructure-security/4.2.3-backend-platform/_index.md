---
title : "Backend and runtime platform"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 4.2.3 </b> "
---

## 4.2.3 Backend and runtime platform

This section covers **API ingress** and the **NestJS backend runtime** in **SpendWiseApp/infrastructure**: root Terraform lives in **environments/dev/main.tf** (and **environments/prod/main.tf** follows the same module layout); each block is under **modules/**.

![SpendWiseApp backend VPC: CloudFront → optional WAF → Internet Gateway → ALB; ECS Fargate across two AZs; RDS PostgreSQL (primary and backup); optional bastion and NAT; ECR/ECS via PrivateLink and S3 via gateway VPC endpoint; CloudWatch logs and alarms](/images/4-Workshop/4.2.3-spendwise-backend-vpc.png)

**Network foundation:** The **vpc** and **security_groups** modules provide subnets (public / private app / private data) and security groups for the ALB, ECS, RDS, and bastion — details in [4.2.1](../4.2.1-vpc-network/).

### Application Load Balancer

**Role:** The **ALB** sits on **public** subnets with HTTP/HTTPS listeners (HTTPS when **alb_acm_certificate_arn** is set) and a target group pointing at the backend container port. It is the sole Internet-facing entry to the container service.

**Why we chose it:** Layer-7 load balancing, health checks, and clean integration with ECS auto scaling on **ALBRequestCountPerTarget** — a natural fit for an HTTP NestJS API behind a reverse proxy.

### Amazon CloudFront (API — optional)

**Role:** When **enable_api_cloudfront** is true, the **cloudfront_api** module places CloudFront in front of the ALB; the API is reachable at an HTTPS URL such as **https://dxxx.cloudfront.net** using CloudFront’s default certificate, so browsers can call the API over HTTPS without requiring ACM/custom domains on the ALB — reducing mixed-content issues when the Amplify SPA is served over HTTPS.

**Why we chose it:** Fast HTTPS for the API URL in lab/dev setups; **frontend_api_url** in **main.tf** picks this URL or the ALB-only URL (**frontend_api_url_alb_only**) so **NEXT_PUBLIC_API_URL** matches how clients call the API.

### AWS WAF (optional)

**Role:** When **enable_waf** is true, **waf_alb** attaches a WAFv2 Web ACL to the **ALB ARN**, filtering requests per the module’s rule configuration.

**Why we chose it:** Adds protection in front of the application; optional CloudWatch metrics via **waf_enable_cloudwatch_metrics**.

### Amazon Elastic Container Registry

**Role:** **ECR** stores the NestJS backend image; ECS tasks pull from within the VPC.

**Why we chose it:** A private registry in the account with IAM control and internal pulls — no reliance on a public registry for production images.

### Amazon ECS on Fargate

**Role:** The **ecs** module runs a **Fargate** cluster on **private_app** subnets, **depends_on** the ALB so listeners exist first; registers tasks with the ALB target group; **locals.ecs_backend_environment** includes **DATABASE_URL** when RDS is enabled, variables from **terraform.tfvars**, **COGNITO_***, and **CORS_ORIGIN** aligned with the Amplify domain. Auto scaling tracks requests per target.

**Why we chose it:** No EC2 fleet to manage for workers; fits stateless APIs and scales with ALB load.

### Amazon RDS and AWS Secrets Manager

**Role:** **RDS** provisions PostgreSQL in **private_data** subnets when **create_rds** is true. **db_password_secret** stores the database password in Secrets Manager when RDS is used (recovery window from variables). The Cognito **PostConfirmation** Lambda writes users to the database when RDS is enabled — see [4.2.2](../4.2.2-client-facing/) for the broader SPA/auth flow.

**Why we chose it:** Managed databases and secrets separated from plain **terraform.tfvars** for production-oriented setups.

### Amazon CloudWatch

**Role:** **monitoring** creates the log group; its name is passed into **ecs** so containers write stdout/stderr there.

**Why we chose it:** Centralized logging and a path to metrics and alarms for backend operations and troubleshooting.

### Bastion (optional)

**Role:** When **create_bastion** is true, **bastion** provisions an EC2 instance in a public subnet, reachable via **SSM** — commonly used for port forwarding to RDS during migrations or debugging.

**Why we chose it:** Keeps Postgres off the public Internet while allowing controlled access.

Operational commands and outputs: [4.3.3 Deploy backend](../../4.3-deployment-operations-monitoring/4.3.3-backend/).
