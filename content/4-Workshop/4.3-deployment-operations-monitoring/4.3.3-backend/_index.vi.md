---
title: "Triển khai backend"
date: 2026-03-10
weight: 3
chapter: false
pre: " <b> 4.3.3 </b> "
---

## 4.3.3 Triển khai backend

Backend chạy trên AWS là **ECS Fargate**, image lấy từ **ECR**. Trong SpendWiseApp/infrastructure/environments/dev/main.tf tương ứng **module "ecr"** và **module "ecs"**. **module "alb"** phải tồn tại trước khi service đăng ký target khỏe — khai báo trước ecs trong cùng file; chi tiết ALB/listener/target group xem [4.3.5 Nền tảng, biên và vận hành](../4.3.5-platform-edge-ops/).

### Mã — kho ECR (modules/ecr/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/ecr/main.tf
resource "aws_ecr_repository" "backend" {
  name                 = "${local.name}-backend"
  image_tag_mutability = "MUTABLE"
  force_delete         = true

  image_scanning_configuration {
    scan_on_push = true
  }
}
```

### Mã — locals cho biến môi trường ECS + module ECS (environments/dev/main.tf)

ecs_backend_environment chèn DATABASE_URL khi RDS bật, nối thêm biến từ terraform.tfvars, và thêm các ID Cognito từ output module Cognito.

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (trích)
locals {
  ecs_backend_environment = concat(
    var.create_rds && module.rds.db_instance_endpoint != null ? [
      {
        name  = "DATABASE_URL"
        value = "postgresql://${var.db_username}:${local.db_password_effective}@${module.rds.db_instance_endpoint}/${var.db_name}?sslmode=no-verify&schema=public"
      }
    ] : [],
    var.ecs_backend_environment,
    [
      { name = "COGNITO_REGION", value = var.aws_region },
      { name = "COGNITO_USER_POOL_ID", value = module.cognito.user_pool_id },
      { name = "COGNITO_CLIENT_ID", value = module.cognito.user_pool_client_id },
    ]
  )
}

module "ecr" {
  source       = "../../modules/ecr"
  project_name = var.project_name
  environment  = var.environment
}

module "ecs" {
  source = "../../modules/ecs"

  depends_on = [module.alb]

  private_subnet_ids               = module.vpc.private_app_subnet_ids
  ecs_tasks_security_group_id      = module.security_groups.ecs_tasks_security_group_id
  target_group_arn                 = module.alb.target_group_arn
  container_image                  = "${module.ecr.repository_url}:${var.ecs_backend_image_tag}"
  container_port                   = var.app_container_port
  cloudwatch_log_group_name        = module.monitoring.ecs_log_group_name
  container_environment            = local.ecs_backend_environment
  desired_count                    = var.ecs_desired_count
  task_cpu                         = var.ecs_task_cpu
  task_memory                      = var.ecs_task_memory
  assign_public_ip                 = var.ecs_assign_public_ip
  autoscaling_min_capacity         = var.ecs_autoscaling_min_capacity
  autoscaling_max_capacity         = var.ecs_autoscaling_max_capacity
  alb_request_count_resource_label = local.ecs_alb_request_count_resource_label
  alb_request_count_target_value   = var.ecs_alb_request_count_target_value
}
```

### Mã — dịch vụ Fargate + autoscaling (modules/ecs/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/ecs/main.tf (trích)
resource "aws_ecs_service" "backend" {
  name            = "${local.name}-backend"
  cluster         = aws_ecs_cluster.this.id
  task_definition = aws_ecs_task_definition.backend.arn
  desired_count   = var.desired_count
  launch_type     = "FARGATE"

  network_configuration {
    subnets          = var.private_subnet_ids
    security_groups  = [var.ecs_tasks_security_group_id]
    assign_public_ip = var.assign_public_ip
  }

  load_balancer {
    target_group_arn = var.target_group_arn
    container_name   = "backend"
    container_port   = var.container_port
  }
}

resource "aws_appautoscaling_policy" "backend_alb_requests" {
  name               = "${local.name}-ecs-alb-requests"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.backend.resource_id
  scalable_dimension = aws_appautoscaling_target.backend.scalable_dimension
  service_namespace  = aws_appautoscaling_target.backend.service_namespace

  target_tracking_scaling_policy_configuration {
    predefined_metric_specification {
      predefined_metric_type = "ALBRequestCountPerTarget"
      resource_label         = var.alb_request_count_resource_label
    }
    target_value       = var.alb_request_count_target_value
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}
```

### modules/ecr — tài nguyên và vai trò

| Tài nguyên | Để làm gì |
|------------|-----------|
| **aws_ecr_repository (…-backend)** | Registry container riêng cho image NestJS. **image_scanning_configuration** bật quét khi push. force_delete = true giúp xóa stack dev gọn. |

Sau khi Terraform tạo repo, pipeline hoặc máy local docker build / docker push với tag (vd. latest hoặc SHA commit) khớp ecs_backend_image_tag.

### modules/ecs — tài nguyên và vai trò

| Tài nguyên | Để làm gì |
|------------|-----------|
| **aws_ecs_cluster** | Tên cluster; bật **Container Insights** cho metric. |
| **aws_ecs_task_definition (Fargate)** | CPU/bộ nhớ, **execution role** (kéo image, ghi log), **task role** (quyền runtime gọi API AWS), port container, driver **awslogs** trỏ tới log group từ module.monitoring. |
| **aws_ecs_service** | Chạy task trong **subnet private_app**, gắn **SG ECS**, tùy assign_public_ip (mặc định false — egress qua NAT/endpoint). Đăng ký **target group của ALB** cho HTTP. |
| **Application Auto Scaling** | **Target tracking** theo **ALBRequestCountPerTarget** để số task theo tải (ecs_alb_request_count_target_value). |

Biến môi trường ghép trong locals của main.tf gồm **DATABASE_URL** khi có RDS, và **COGNITO_*** từ module.cognito để API xác thực JWT không hard-code pool trong file Terraform.

**Lưu ý:** module "ecs" có depends_on = [module.alb] để listener sẵn sàng trước khi tạo service.

Tham chiếu thiết kế: [4.2.3 Backend và nền tảng xử lý](../4.2-aws-infrastructure-security/4.2.3-backend-platform/).
