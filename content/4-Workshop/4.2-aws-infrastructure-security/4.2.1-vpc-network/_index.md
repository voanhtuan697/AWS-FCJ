---
title : "VPC and networking"
date: 2026-03-10
weight : 1
chapter : false
pre : " <b> 4.2.1 </b> "
---

## 4.2.1 VPC and networking

AWS networking for **SpendWiseApp** follows a **3-tier** layout with **public/private** separation and **two Availability Zones** for availability.

| Component | Role |
| :--- | :--- |
| **VPC** | Dedicated VPC (DNS hostnames / DNS support enabled); **6 subnets** — per AZ one **public**, one **private app** (application / ECS tier), one **private data** (data / RDS tier); **child CIDRs** mapped clearly per tier. |
| **Internet Gateway** | Attachment between the VPC and the **public Internet**; **public** subnets route to the IGW for Internet reachability (for example an ALB receiving user traffic). |
| **NAT Gateway** | Lets resources in **private** subnets initiate **outbound** Internet traffic without assigning a public IP to each EC2/task; **inbound** from the Internet does not land directly on private workloads like on a public instance. |
| **VPC endpoint** | **Private path inside the AWS network** to AWS services (Interface or Gateway types); traffic **does not go out over the public Internet** to reach those APIs — reduces exposure and often pairs with limiting or disabling NAT for those flows. |

### VPC and subnets

A **VPC (Virtual Private Cloud)** is your private network on AWS. Most resources run inside a VPC. Once the VPC is provisioned successfully, subnets are deployed as follows:

| Subnet group | Count | Purpose |
| :--- | :---: | :--- |
| **Public** | 2 | One public subnet per AZ for Internet-facing or egress plumbing — for example the **Application Load Balancer**, and the **NAT Gateway** when outbound access for private subnets is enabled. |
| **Private (application)** | 2 | One **private app** subnet per AZ for **ECS Fargate** tasks running the backend; workloads do not advertise public IPs directly to end users — user ingress goes through the ALB on the public tier. |
| **Private (data)** | 2 | One **private data** subnet per AZ for **RDS PostgreSQL**; connectivity stays internal (for example Postgres) via Security Groups without exposing the database to the public Internet. |

### Security Group (application and data)

A **Security Group** is a **stateful** firewall applied to **ENIs** in the VPC: rules are **allow-only** (everything else is denied by default), typically referencing a **CIDR** or **another Security Group**.

Security groups in this deployment:

| Security Group | Role |
| :--- | :--- |
| ALB Security Group | On the **ALB**. Inbound: **HTTP (80)** and **HTTPS (443)** from the **Internet** (any source — **0.0.0.0/0**). |
| ECS Security Group | On **ECS Fargate** tasks. Allows inbound traffic **only from the ALB** (via ALB Security Group). |
| Bastion Security Group | On the **bastion** host (when enabled). |
| RDS Security Group | On **RDS PostgreSQL**. Inbound: **5432** from ECS task Security Group; optional from bastion SG when bastion DB access is enabled. |

### VPC endpoint

A **VPC endpoint** allows traffic to AWS service APIs **inside the AWS network**, instead of **going out over the public Internet and back** — reducing exposure and often reducing reliance on NAT for those flows. Two kinds matter for this deployment:

- **Gateway endpoint** — for **Amazon S3**. Attached to **route tables**: matching prefixes are routed straight to S3 **without** adding an ENI in the subnet.
- **Interface endpoint** — creates **ENIs** in subnets and provides **private DNS** for regional APIs. Used here for **ECR** (API and DKR), **CloudWatch Logs**, **Cognito IDP**, and so on. Interface endpoints share one **Security Group** named **vpc_endpoints**, allowing **HTTPS (TCP 443)** from the VPC CIDR. The **S3** gateway is attached to the appropriate public/private **route tables** (branch depends on **enable_nat_gateway**).

| Resource | Type | Subnet / route table | Operational note |
| :--- | :--- | :--- | :--- |
| aws_security_group.vpc_endpoints | — | Applies to the VPC | Ingress **TCP 443** from vpc_cidr to Interface endpoints. |
| aws_vpc_endpoint.ecr_api | Interface | private_app | **ECR API** — authentication/token for image pulls. |
| aws_vpc_endpoint.ecr_dkr | Interface | private_app | **ECR DKR** — Docker registry API (manifest/metadata). |
| aws_vpc_endpoint.s3 | Gateway | Route table list (public + private; branch depends on enable_nat_gateway) | **S3** — gateway; blob/layer traffic via prefix list. |
| aws_vpc_endpoint.logs | Interface | private_app | **CloudWatch Logs** — shipping container logs (**awslogs**) without needing NAT for that path. |
| aws_vpc_endpoint.cognito_idp | Interface | **private_app** subnets only in AZs supported by the Cognito IDP endpoint service | **Cognito IDP** — identity API calls within the VPC boundary. |
