---
title: "Triển khai VPC và mạng"
date: 2026-03-10
weight: 1
chapter: false
pre: " <b> 4.3.1 </b> "
---

## 4.3.1 Triển khai VPC và mạng

Stack **dev** nối mạng trong SpendWiseApp/infrastructure/environments/dev/main.tf qua **module "vpc"** và **module "security_groups"**. Hai module này quyết định mọi tài nguyên khác gắn vào đâu và luồng traffic được phép như thế nào.

### Mã — root module (environments/dev/main.tf)

Lấy **hai AZ** từ data source rồi truyền vào module VPC. Module security group nhận vpc_id, **port container** (để quy tắc ALB → task), và có mở Postgres từ bastion hay không khi **đồng thời** bật bastion và RDS.

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (trích)
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

### Mã — bố cục VPC (modules/vpc/main.tf)

Subnet **public / private app / private data** mỗi loại **hai AZ**. NAT tùy chọn; khi tắt, subnet private dùng route table cô lập, không default route ra Internet.

```terraform
# SpendWiseApp/infrastructure/modules/vpc/main.tf (trích)
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

### Mã — security group (modules/security_groups/main.tf)

ALB mở HTTP/HTTPS từ Internet. ECS chỉ nhận **đúng port container từ SG của ALB**. RDS nhận Postgres từ SG ECS và tùy chọn từ SG bastion.

```terraform
# SpendWiseApp/infrastructure/modules/security_groups/main.tf (trích)
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
  # ingress động từ bastion khi enable_bastion_rds_access = true
}
```

### modules/vpc — tài nguyên và vai trò

| Tài nguyên | Để làm gì |
|------------|-----------|
| **VPC** (aws_vpc) | Mạng IPv4 riêng cho ứng dụng; bật DNS hostnames/support để ECS/RDS phân giải tên nội bộ. |
| **Internet gateway** | Cho **subnet public** đường ra/vào Internet (ALB, đặt NAT). |
| **Subnet public (2 AZ)** | Chứa thành phần hướng Internet (ALB, bastion có public IP nếu bật). Một subnet cũng là chỗ đặt **NAT Gateway**. |
| **Subnet private app (2 AZ)** | Chạy **ECS Fargate** (mặc định không gán public IP) — tầng ứng dụng. |
| **Subnet private data (2 AZ)** | Đặt **RDS** và **Lambda** trong VPC (ví dụ trigger sau xác nhận Cognito) — tầng dữ liệu, không expose trực tiếp ra Internet. |
| **EIP + NAT Gateway** (khi enable_nat_gateway = true) | Cho subnet private **đi ra ngoài** có kiểm soát (pull image, gọi API AWS…). Biến enable_nat_gateway trong environments/dev/variables.tf ghi chú: NAT phục vụ ECS/ECR và outbound. |
| **Route table** | Public → IGW; private → NAT nếu có NAT; khi tắt NAT, private dùng RT cô lập (không default route Internet — tiết kiệm/thử nghiệm nhưng hạn chế egress). |

availability_zones được cắt **2 AZ** trong locals của main.tf cho bố cục HA tối thiểu.

### modules/security_groups — tài nguyên và vai trò

Security group là **firewall L4 có trạng thái** gắn ENI. Module này khớp mô hình 3 lớp:

| Security group | Ý ingress / egress |
|------------------|---------------------|
| **ALB** | Cho **80/443 từ Internet**; egress mở để ALB tới IP task trên port ứng dụng. |
| **ECS tasks** | Chỉ cho **port container từ SG của ALB**; egress mở cho ứng dụng (DB, Cognito, API AWS…). |
| **RDS** | **PostgreSQL 5432 từ SG ECS**; tùy chọn **5432 từ SG bastion** khi bật bastion + RDS. |
| **Bastion** | Mặc định chỉ egress; không mở SSH 0.0.0.0/0 trong module — vận hành qua **SSM Session Manager** (xem IAM bastion trong repo). |

### Cách apply (dev)

Từ thư mục SpendWiseApp/infrastructure/environments/dev/ (sau khi cấu hình terraform.tfvars cho region, token Amplify, v.v.):

```bash
terraform init
terraform plan
terraform apply
```

Một lần apply tạo cả stack, nhưng các tài nguyên trên là phần **mạng + biên L4**. Lý do thiết kế: [4.2.1 VPC và mạng](../4.2-aws-infrastructure-security/4.2.1-vpc-network/).
