---
title: "Workshop"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

# Workshop: SpendWiseApp Architecture on AWS

This workshop goes deep into **AWS infrastructure architecture** for **SpendWiseApp**: which services are used, how the network is layered, and the security approach. **Terraform deployment by layer** (VPC, frontend or auth, backend) is covered in [4.3 Terraform deployment (by layer)](4.3-deployment-operations-monitoring/); other stack pieces such as RDS and the load balancer remain in the same **infrastructure** repo and in [4.2](4.2-aws-infrastructure-security/) where relevant.

Content is grouped into the following main sections for easier navigation:
- System overview and architecture.
- AWS infrastructure and security foundation.
- Terraform deployment (by layer), including frontend hosting and backend (ECR/ECS) operations.
- Cost, risk, and expansion roadmap.

#### Content

1. [4.1 System overview and architecture](4.1-system-overview-architecture/)
2. [4.2 AWS infrastructure and security foundation](4.2-aws-infrastructure-security/)
3. [4.3 Terraform deployment (by layer)](4.3-deployment-operations-monitoring/)
4. [4.5 Cost, risk, and expansion roadmap](4.5-cost-risk-expansion-roadmap/)
5. [4.6 Clean up](4.6-legacy-cleanup/)
