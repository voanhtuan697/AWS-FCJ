---
title: "Deploy backend"
date: 2026-03-10
weight: 3
chapter: false
pre: " <b> 4.3.3 </b> "
---

## 4.3.3 Deploy backend

The backend runtime on AWS is **ECS Fargate** pulling an image from **ECR**. In SpendWiseApp/infrastructure/environments/dev/main.tf this maps to **module "ecr"** and **module "ecs"**. **module "alb"** must exist before the service can register healthy targets — it is declared earlier in the same file and is described under [4.3.5 Platform, edge & operations](../4.3.5-platform-edge-ops/) (ALB + listener + target group).

### Code — ECR repository (modules/ecr/main.tf)

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

### Code — locals for ECS env + ECS module (environments/dev/main.tf)

ecs_backend_environment injects DATABASE_URL when RDS is on, merges extra vars from terraform.tfvars, and appends Cognito identifiers from the Cognito module output.

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (excerpt)
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

### Code — Fargate service + autoscaling (modules/ecs/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/ecs/main.tf (excerpt)
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

### modules/ecr — what it creates and why

| Resource | Role |
|----------|------|
| **aws_ecr_repository (…-backend)** | Private container registry for the NestJS image. **image_scanning_configuration** enables scan-on-push for supply-chain visibility. force_delete = true helps tear down dev stacks cleanly. |

After Terraform creates the repo, your pipeline or laptop runs docker build / docker push to tag (e.g. latest or a commit SHA) referenced by ecs_backend_image_tag.

### modules/ecs — what it creates and why

| Resource | Role |
|----------|------|
| **aws_ecs_cluster** | Logical cluster name; **Container Insights** enabled for metrics. |
| **aws_ecs_task_definition (Fargate)** | Declares CPU/memory, **execution role** (pull image, write logs), **task role** (runtime AWS API access), container port, and **awslogs** driver pointing at the log group from module.monitoring. |
| **aws_ecs_service** | Runs tasks in **private_app subnets**, attaches the **ECS tasks security group**, optional assign_public_ip (default false — use NAT/endpoints for egress). Registers with the **ALB target group** for HTTP traffic. |
| **Application Auto Scaling** | **Target tracking** on **ALBRequestCountPerTarget** so desired task count follows load (ecs_alb_request_count_target_value in variables). |

Environment variables injected in main.tf locals include **DATABASE_URL** when RDS exists, plus **COGNITO_*** from module.cognito so the API can validate JWTs without hard-coding pool IDs in Terraform files.

**Note:** module "ecs" has depends_on = [module.alb] so listeners exist before service creation.

Design reference: [4.2.3 Backend and runtime platform](../4.2-aws-infrastructure-security/4.2.3-backend-platform/).
