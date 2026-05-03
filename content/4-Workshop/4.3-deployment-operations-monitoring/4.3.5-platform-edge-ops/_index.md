---
title: "Platform, edge & operations"
date: 2026-03-10
weight: 5
chapter: false
pre: " <b> 4.3.5 </b> "
---

## 4.3.5 Platform, edge & operations

These pieces sit at the **edge** (how clients reach the API), **observability**, or **break-glass operations**. In SpendWiseApp/infrastructure/environments/dev/main.tf they appear as **module "monitoring"** (early in the file but conceptually tied to ECS/ALB), **module "alb"**, optional **module "waf_alb"**, optional **aws_route53_zone.amplify**, and optional **module "bastion"**.

### Code — monitoring, ALB, WAF, bastion (environments/dev/main.tf)

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (excerpt)
module "monitoring" {
  source       = "../../modules/monitoring"
  project_name = var.project_name
  environment  = var.environment
}

module "alb" {
  source = "../../modules/alb"

  project_name          = var.project_name
  environment           = var.environment
  vpc_id                = module.vpc.vpc_id
  public_subnet_ids     = module.vpc.public_subnet_ids
  alb_security_group_id = module.security_groups.alb_security_group_id
  container_port        = var.app_container_port
  health_check_path     = var.alb_health_check_path
  enable_https_listener = trimspace(var.alb_acm_certificate_arn) != ""
  acm_certificate_arn   = var.alb_acm_certificate_arn
}

module "waf_alb" {
  count  = var.enable_waf ? 1 : 0
  source = "../../modules/waf_alb"

  project_name              = var.project_name
  environment               = var.environment
  alb_arn                   = module.alb.alb_arn
  enable_cloudwatch_metrics = var.waf_enable_cloudwatch_metrics
}

module "bastion" {
  count  = var.create_bastion ? 1 : 0
  source = "../../modules/bastion"

  project_name                = var.project_name
  environment                 = var.environment
  subnet_id                   = module.vpc.public_subnet_ids[0]
  security_group_id           = module.security_groups.bastion_security_group_id
  instance_type               = var.bastion_instance_type
  associate_public_ip_address = var.bastion_associate_public_ip
}
```

### Code — CloudWatch log group for ECS (modules/monitoring/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/monitoring/main.tf
resource "aws_cloudwatch_log_group" "ecs_backend" {
  name              = "/ecs/${local.name}/backend"
  retention_in_days = var.log_retention_days
}
```

### Code — ALB listeners (modules/alb/main.tf)

HTTP either forwards to the target group or redirects to 443 when TLS is enabled; HTTPS listener is created only when an ACM ARN is provided.

```terraform
# SpendWiseApp/infrastructure/modules/alb/main.tf (excerpt)
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.this.arn
  port              = 80
  protocol          = "HTTP"

  default_action {
    type = local.enable_https ? "redirect" : "forward"

    dynamic "redirect" {
      for_each = local.enable_https ? [1] : []
      content {
        port        = "443"
        protocol    = "HTTPS"
        status_code = "HTTP_301"
      }
    }

    target_group_arn = local.enable_https ? null : aws_lb_target_group.ecs.arn
  }
}

resource "aws_lb_listener" "https" {
  count             = local.enable_https ? 1 : 0
  load_balancer_arn = aws_lb.this.arn
  port              = 443
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS13-1-2-2021-06"
  certificate_arn   = var.acm_certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.ecs.arn
  }
}
```

### Code — WAF association (modules/waf_alb/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/waf_alb/main.tf (excerpt)
resource "aws_wafv2_web_acl_association" "alb" {
  resource_arn = var.alb_arn
  web_acl_arn  = aws_wafv2_web_acl.this.arn
}
```

### modules/monitoring — what it creates and why

| Resource | Role |
|----------|------|
| **aws_cloudwatch_log_group** (/ecs/…/backend) | Central sink for **ECS task awslogs streams**; retention is configurable. The ECS task definition references this name. |

The module file comments note room to add **metric alarms** (ALB 5xx, CPU, RDS storage) later.

### modules/alb — what it creates and why

| Resource | Role |
|----------|------|
| **aws_lb (application, internet-facing)** | Spans **public subnets**; attached to the **ALB security group** (80/443 from the world). |
| **aws_lb_target_group (IP mode)** | Targets **Fargate task ENIs** on the app container port; **HTTP health checks** (path from alb_health_check_path). |
| **aws_lb_listener HTTP** | Forwards to the target group, or **301 redirect to HTTPS** when alb_acm_certificate_arn is set. |
| **aws_lb_listener HTTPS** | Optional second listener with **ACM certificate** in the **same region as the ALB** (TLS 1.3 policy in module). |

Outputs feed **Amplify** (NEXT_PUBLIC_API_URL) and **ECS** (target group ARN, ARN fragments for autoscaling metrics).

### modules/waf_alb (optional, enable_waf)

| Resource | Role |
|----------|------|
| **aws_wafv2_web_acl (REGIONAL)** | **AWS Managed Rules Common Rule Set** in front of the ALB via **aws_wafv2_web_acl_association**. CloudWatch metrics for WAF can be toggled to control cost. |

### modules/bastion (optional, create_bastion)

| Resource | Role |
|----------|------|
| **aws_instance (Amazon Linux 2023)** | Small EC2 in a **public subnet**; **IMDSv2 required**; **SSM instance profile** so you connect with Session Manager instead of opening SSH globally. |
| **Security group** | Used by RDS dynamic ingress when bastion + RDS are both enabled. |

### Route 53: Amplify zone vs modules/route53_api

- **aws_route53_zone.amplify** in the dev root is only for **Amplify custom-domain** workflows (see [4.3.2](../4.3.2-frontend-hosting-auth/)).
- **modules/route53_api** in the repo can manage **API hostname + ACM DNS validation** for the ALB; it is **not** wired into the default dev/main.tf shown here — add it when you want full Route 53–managed API DNS in Terraform.

**NAT Gateway** stays with the network layer: [4.3.1](../4.3.1-vpc-network/).

Design reference: [4.2 AWS infrastructure and security foundation](../4.2-aws-infrastructure-security/).
