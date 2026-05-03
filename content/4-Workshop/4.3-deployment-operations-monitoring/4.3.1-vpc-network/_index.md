---
title: "Deploy VPC and networking"
date: 2026-03-10
weight: 1
chapter: false
pre: " <b> 4.3.1 </b> "
---

## 4.3.1 Deploy VPC and networking

The dev stack wires networking in SpendWiseApp/infrastructure/environments/dev/main.tf through **module "vpc"** and **module "security_groups"**. Together they define where every other resource attaches and how traffic may flow.

### Code — root module (environments/dev/main.tf)

Two AZ names are taken from the data source, then passed into the VPC module. Security groups receive the VPC id, the app container port (for ALB → task rules), and whether to open Postgres from the bastion when both bastion and RDS exist.

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (excerpt)
data "aws_availability_zones" "available" {
  state = "available"
}

locals {
  azs = slice(data.aws_availability_zones.available.names, 0, 2)
}

module "vpc" {
  source = "../../modules/vpc"

  project_name       = var.project_name
  environment        = var.environment
  availability_zones = local.azs
  enable_nat_gateway = var.enable_nat_gateway
}

module "security_groups" {
  source = "../../modules/security_groups"

  project_name              = var.project_name
  environment               = var.environment
  vpc_id                    = module.vpc.vpc_id
  app_container_port        = var.app_container_port
  enable_bastion_rds_access = var.create_bastion && var.create_rds
}
```

### Code — VPC layout (modules/vpc/main.tf)

Public / private app / private data subnets are created in pairs (one per AZ). NAT is optional; when off, private subnets use isolated route tables without a default route to the internet.

```terraform
# SpendWiseApp/infrastructure/modules/vpc/main.tf (excerpt)
resource "aws_subnet" "public" {
  count                   = 2
  vpc_id                  = aws_vpc.this.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 1)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
}

resource "aws_subnet" "private_app" {
  count             = 2
  vpc_id            = aws_vpc.this.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 11)
  availability_zone = var.availability_zones[count.index]
}

resource "aws_subnet" "private_data" {
  count             = 2
  vpc_id            = aws_vpc.this.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 21)
  availability_zone = var.availability_zones[count.index]
}
```

### Code — security groups (modules/security_groups/main.tf)

ALB accepts HTTP/HTTPS from the internet. ECS tasks accept **only** the container port from the ALB security group. RDS accepts PostgreSQL from ECS tasks, and optionally from the bastion SG.

```terraform
# SpendWiseApp/infrastructure/modules/security_groups/main.tf (excerpt)
resource "aws_security_group" "ecs_tasks" {
  name   = "${local.name}-ecs-sg"
  vpc_id = var.vpc_id

  ingress {
    description     = "From ALB"
    from_port       = var.app_container_port
    to_port         = var.app_container_port
    protocol        = "tcp"
    security_groups = [aws_security_group.alb.id]
  }
  # ...
}

resource "aws_security_group" "rds" {
  name   = "${local.name}-rds-sg"
  vpc_id = var.vpc_id

  ingress {
    description     = "Postgres from ECS"
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.ecs_tasks.id]
  }
  # dynamic ingress from bastion when enable_bastion_rds_access = true
}
```

### modules/vpc — what it creates and why

| Resource | Role |
|----------|------|
| **VPC** (aws_vpc) | Isolated IPv4 network for the app; DNS hostnames/support enabled so ECS and RDS can resolve internal names. |
| **Internet gateway** | Gives **public subnets** a path to/from the internet (ALB, NAT placement). |
| **Public subnets (×2 AZs)** | Host internet-facing components (ALB, optional bastion with public IP). One subnet also anchors the **NAT Gateway**. |
| **Private app subnets (×2)** | Where **ECS Fargate** tasks run without public IPs by default — application tier. |
| **Private data subnets (×2)** | Where **RDS** and VPC-attached **Lambda** (e.g. Cognito post-confirmation) live — data tier, no direct internet exposure. |
| **Elastic IP + NAT Gateway** (when enable_nat_gateway = true) | Lets private subnets initiate **outbound** traffic (e.g. pull images, call AWS APIs) via a single controlled egress. Variable enable_nat_gateway in environments/dev/variables.tf documents the intent: NAT for ECS/ECR pull patterns. |
| **Route tables** | Public RT → IGW; private RT → NAT when NAT is on; without NAT, private subnets use isolated RTs (no default route to internet — useful for cost/experiments, but limits outbound). |

availability_zones is sliced to **two AZs** in main.tf locals for a minimal HA layout.

### modules/security_groups — what it creates and why

Security groups are **stateful L4 firewalls** attached to ENIs. This module aligns rules with the three-tier model:

| Security group | Ingress / egress idea |
|------------------|------------------------|
| **ALB** | Allows **80/443 from the internet**; egress open so the ALB can reach task IPs on the app port. |
| **ECS tasks** | Allows **container port only from the ALB SG**; egress open for app outbound (DB, Cognito, AWS APIs, etc.). |
| **RDS** | **PostgreSQL (5432) from the ECS tasks SG**; optional **5432 from bastion SG** when bastion + RDS are both enabled. |
| **Bastion** | Egress only by default; no wide ingress in module — operators use **SSM Session Manager** (see bastion IAM in the repo) instead of SSH from 0.0.0.0/0. |

### How to apply (dev)

From SpendWiseApp/infrastructure/environments/dev/ (after configuring terraform.tfvars for region, Amplify token, etc.):

```bash
terraform init
terraform plan
terraform apply
```

A single apply creates the whole stack, but the resources above are the **network + perimeter** slice. Design rationale: [4.2.1 VPC and networking](../4.2-aws-infrastructure-security/4.2.1-vpc-network/).
