---
title: "Nền tảng, biên và vận hành"
date: 2026-03-10
weight: 5
chapter: false
pre: " <b> 4.3.5 </b> "
---

## 4.3.5 Nền tảng, biên và vận hành

Các phần này nằm ở **biên** (cách client vào API), **quan sát** (log/metric), hoặc **vận hành khẩn cấp**. Trong SpendWiseApp/infrastructure/environments/dev/main.tf tương ứng **module "monitoring"** (khai báo sớm trong file nhưng gắn chặt ECS/ALB), **module "alb"**, **module "waf_alb"** tùy chọn, **aws_route53_zone.amplify** tùy chọn, và **module "bastion"** tùy chọn.

### Mã — monitoring, ALB, WAF, bastion (environments/dev/main.tf)

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (trích)
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

### Mã — log group CloudWatch cho ECS (modules/monitoring/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/monitoring/main.tf
resource "aws_cloudwatch_log_group" "ecs_backend" {
  name              = "/ecs/${local.name}/backend"
  retention_in_days = var.log_retention_days
}
```

### Mã — listener ALB (modules/alb/main.tf)

HTTP forward tới target group hoặc redirect 443 khi bật TLS; listener HTTPS chỉ tạo khi có ARN ACM.

```terraform
# SpendWiseApp/infrastructure/modules/alb/main.tf (trích)
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

### Mã — gắn WAF vào ALB (modules/waf_alb/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/waf_alb/main.tf (trích)
resource "aws_wafv2_web_acl_association" "alb" {
  resource_arn = var.alb_arn
  web_acl_arn  = aws_wafv2_web_acl.this.arn
}
```

### modules/monitoring — tài nguyên và vai trò

| Tài nguyên | Để làm gì |
|------------|-----------|
| **aws_cloudwatch_log_group** (/ecs/…/backend) | Nơi gom **luồng log awslogs của ECS**; thời gian lưu cấu hình được. Task definition ECS tham chiếu tên group này. |

File module có ghi chú có thể mở rộng **cảnh báo metric** (ALB 5xx, CPU, dung lượng RDS) sau.

### modules/alb — tài nguyên và vai trò

| Tài nguyên | Để làm gì |
|------------|-----------|
| **aws_lb (application, internet-facing)** | Trải trên **subnet public**; gắn **SG ALB** (80/443 từ Internet). |
| **aws_lb_target_group (chế độ IP)** | Trỏ tới **ENI task Fargate** trên port container; **health check HTTP** (đường dẫn từ alb_health_check_path). |
| **Listener HTTP** | Forward tới target group, hoặc **301 sang HTTPS** khi có alb_acm_certificate_arn. |
| **Listener HTTPS** | Listener thứ hai tùy chọn với **chứng chỉ ACM** cùng **region với ALB** (chính sách TLS 1.3 trong module). |

Output phục vụ **Amplify** (NEXT_PUBLIC_API_URL) và **ECS** (ARN target group, mảnh ARN cho autoscaling theo metric ALB).

### modules/waf_alb (tùy chọn, enable_waf)

| Tài nguyên | Để làm gì |
|------------|-----------|
| **aws_wafv2_web_acl (REGIONAL)** | Bộ **AWS Managed Rules Common Rule Set** phía trước ALB qua **aws_wafv2_web_acl_association**. Metric CloudWatch cho WAF có thể tắt để tiết kiệm. |

### modules/bastion (tùy chọn, create_bastion)

| Tài nguyên | Để làm gì |
|------------|-----------|
| **aws_instance (Amazon Linux 2023)** | EC2 nhỏ trong **subnet public**; bắt **IMDSv2**; **instance profile SSM** để kết nối Session Manager thay vì mở SSH toàn cầu. |
| **Security group** | RDS mở ingress động từ bastion khi bật đồng thời bastion + RDS. |

### Route 53: zone Amplify vs modules/route53_api

- **aws_route53_zone.amplify** trong root **dev** chỉ phục vụ **custom domain Amplify** (xem [4.3.2](../4.3.2-frontend-hosting-auth/)).
- **modules/route53_api** trong repo có thể quản **hostname API + xác thực DNS ACM** cho ALB; **chưa** gắn sẵn vào dev/main.tf mặc định — thêm khi muốn DNS API do Route 53 quản hoàn toàn trong Terraform.

**NAT Gateway** thuộc lớp mạng: [4.3.1](../4.3.1-vpc-network/).

Tham chiếu thiết kế: [4.2 Nền tảng hạ tầng AWS và bảo mật](../4.2-aws-infrastructure-security/).
