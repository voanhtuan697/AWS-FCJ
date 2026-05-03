---
title: "Proposal"
date: 2026-03-10
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Proposal for Deploying SpendWiseApp on AWS

## 1. Project Overview

SpendWiseApp is a full-stack personal expense management web application that helps users track, analyze, and control personal finance in a practical and visual way.

The application is deployed on AWS Cloud to improve flexibility, reliability, and data safety.

## 2. Problems and Solution

### Current Problems

For a personal finance application, user data sensitivity and system stability are critical. If deployed on a single server or fragmented self-managed infrastructure, common issues include:
- Financial data loss risk due to manual backup and inconsistent recovery procedures.
- Usage spikes (especially at month start/end) causing slow responses, congestion, and outages.
- Direct server patching/upgrades leading to configuration drift and operational mistakes.
- Lack of centralized observability, increasing incident detection and recovery time.
- Weak security posture when access control and infrastructure policy are not standardized.

### Solution

To address these issues, SpendWiseApp adopts an AWS cloud architecture:
- Modern frontend: Next.js hosted and deployed on AWS Amplify.
- Backend service: NestJS deployed on AWS ECS Fargate.
- Network foundation: VPC-based architecture with public/private segmentation and ALB traffic distribution.

## 3. Solution Architecture

The architecture follows a practical 3-tier cloud model.
![SpendWise AWS architecture](/images/AWS_SpendWise.drawio.png)

**AWS Services**

| Service | Role in SpendWise |
| :--- | :--- |
| **Amazon VPC** | Private network with public/private subnets across AZs for access/data isolation. |
| **NAT Gateway** | Enables outbound internet access from private subnets when needed. |
| **Security Group** | Firewall policy for ALB, ECS, RDS, bastion, and VPC endpoints. |
| **Application Load Balancer** | Internet entry point; load balancing and routing to ECS backend. |
| **Amazon ECS (Fargate)** | Runs NestJS backend containers without EC2 management. |
| **Amazon ECR** | Container image registry for backend deployments. |
| **VPC Endpoint (Interface)** | Private connectivity to ECR API, ECR DKR, CloudWatch Logs, and Cognito IDP. |
| **VPC Endpoint (Gateway — S3)** | Private S3 access path (e.g., image layer pull flows). |
| **Amazon Cognito** | User pool/app client for sign-up, login, and account confirmation. |
| **AWS Lambda** (Cognito trigger) | PostConfirmation processing when RDS integration is enabled. |
| **Amazon RDS (PostgreSQL)** | Relational database in private network for finance data. |
| **AWS Amplify** | Frontend build and hosting pipeline from Git source. |
| **Amazon CloudFront** | CDN in front of the ALB; accelerates delivery to users and terminates HTTPS before traffic reaches the load balancer. |
| **AWS WAF** | Web protection against common attacks (SQLi/XSS/bot traffic) at edge/API entry. |
| **AWS Secrets Manager (SM)** | Secure storage for secret keys, tokens, and DB credentials. |
| **Amazon EC2 (Bastion)** | Controlled jump host for DB operations via SSM/port forwarding. |
| **Amazon CloudWatch** | Centralized logs, metrics, alarms, and operational monitoring. |

## 4. Timeline (8 weeks)

- **Weeks 1–2 — Foundation & network**
  - **Week 1:** Finalize functional and non-functional requirements; initialize Terraform workflow, remote state, and environment layout.
  - **Week 2:** Stand up VPC, public/private segmentation, security groups, and baseline routing; validate network design against the target architecture.

- **Weeks 3–4 — Application runtime on AWS**
  - **Week 3:** Provision ALB, ECR, and ECS (Fargate); containerize and ship the NestJS backend; wire health checks and basic service rollout.
  - **Week 4:** Integrate Cognito authentication flows; connect the Amplify-hosted frontend to the live API; smoke-test auth and core API paths end to end.

- **Weeks 5–6 — Data & security hardening**
  - **Week 5:** Enable RDS (PostgreSQL), Secrets Manager for credentials, and run Prisma migrations; confirm ECS-to-RDS connectivity over private subnets only.
  - **Week 6:** Validate private networking and least-privilege paths (including VPC endpoints where applicable); configure HTTPS on the ALB (ACM) and optional edge controls (for example WAF / CloudFront) as needed.

- **Weeks 7–8 — Stabilization & handover**
  - **Week 7:** Run baseline performance and smoke tests; implement CloudWatch logging/metrics and lightweight alarms aligned with ECS/ALB/RDS.
  - **Week 8:** Complete project documentation.

## 5. Budget

Reference monthly estimate (USD), based on current AWS pricing and SpendWise service configuration.

| AWS Service | Component / Usage | Cost (USD/month) |
|---|---|---:|
| Elastic Load Balancing | Application Load Balancer (ALB + LCU) | $18 - $35 |
| Amazon ECS | Fargate (vCPU & Memory) | $9 - $25 |
| Amazon VPC | VPC Endpoints (Interface + Gateway) | $20 - $70 |
| Amazon VPC | NAT Gateway (optional) | $0 or $33 - $60 |
| Amazon RDS | PostgreSQL (small Single-AZ) | $12 - $35 |
| AWS Amplify | Frontend hosting + build | $5 - $20 |
| Amazon Cognito | MAU & auth flow | $0 - $10 |
| Amazon ECR | Image storage | $1 - $8 |
| Amazon CloudWatch | Logs / Metrics / Alarms | $3 - $15 |
| Amazon CloudFront | Distribution + data transfer (ALB origin) | $5 - $40 |
| AWS WAF | Web ACL + Rules + Requests (optional) | $0 or $8 - $25 |
| AWS Secrets Manager | Secret storage + API calls | $1 - $6 |
| Amazon EC2 | Bastion (optional) | $0 or $5 - $12 |
| Terraform (IaC) | Infrastructure provisioning through IaC definitions | No direct AWS charge |
| **TOTAL AWS COST** |  | **$74 - $361** |

Cost control recommendations:
- Temporarily disable bastion when DB interactive access is not required.
- Keep only one small Single-AZ RDS instance during development.
- Reduce CloudWatch retention for low-value logs.
- Watch NAT Gateway costs closely and prefer endpoint-based private access where applicable.

## 6. Risks

- **Cost spike risk** on NAT/ALB under traffic growth or misconfiguration.  
  *Mitigation*: budget alarms, weekly cost review, right-sizing.

- **Config drift risk** across frontend/backend/Cognito secrets and environment variables.  
  *Mitigation*: standardized env templates and release checklist.

- **Database operation risk** (migration, connection pool, backup quality).  
  *Mitigation*: controlled migration process, restore drills, DB monitoring.

- **Deployment interruption risk** if image/deploy automation is incomplete.  
  *Mitigation*: standardized build/push/deploy pipeline and smoke tests.

- **Security risk** from broad inbound rules or exposed credentials.  
  *Mitigation*: least-privilege policy, secure secret storage, periodic IAM/SG review.

