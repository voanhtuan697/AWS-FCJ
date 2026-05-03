---
title: "Deploy backend"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 4.3.3 </b> "
---

## 4.3.3 Deploy backend

The backend runtime on AWS is **ECS Fargate** pulling an image from **ECR**. In SpendWiseApp/infrastructure/environments/dev/main.tf this maps to **module "ecr"** and **module "ecs"**. **module "alb"** must exist before the service can register healthy targets; it is declared **before** **module "ecs"** in the same **main.tf**. For listener and target group details, open **modules/alb** in your clone.

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

### After terraform apply — push image and roll ECS

After **terraform apply**, ECR, ECS, the ALB, and the task definition already exist; the task definition points at the image tag in **ecs_backend_image_tag** (**terraform.tfvars**). For each backend code change, run **two steps**:

1. **Docker push** — Build the NestJS image from the repo Dockerfile, **tag** it to match **ecs_backend_image_tag**, log in to ECR, and **docker push** to **ecr_repository_url** (Terraform output).
2. **Roll ECS** — **aws ecs update-service … --force-new-deployment** so new tasks pull the pushed image; wait until the service is **stable** and tasks are healthy behind the ALB.

**Step 1 — ECR login, build, push** (example: shell in the parent directory of **SpendWiseApp**; if your cwd is inside SpendWiseApp, adjust **TF_DIR** and **docker build** paths):

```bash
export TF_DIR=SpendWiseApp/infrastructure/environments/dev
export AWS_REGION="$(terraform -chdir=$TF_DIR output -raw aws_region)"
export ECR_REPO="$(terraform -chdir=$TF_DIR output -raw ecr_repository_url)"
export ECR_REGISTRY="${ECR_REPO%%/*}"
export IMAGE_TAG=latest   # must match ecs_backend_image_tag in Terraform

aws ecr get-login-password --region "$AWS_REGION" \
  | docker login --username AWS --password-stdin "$ECR_REGISTRY"

docker build -f SpendWiseApp/backend/Dockerfile -t spendwise-backend:"$IMAGE_TAG" SpendWiseApp/backend
docker tag spendwise-backend:"$IMAGE_TAG" "${ECR_REPO}:${IMAGE_TAG}"
docker push "${ECR_REPO}:${IMAGE_TAG}"
```

**Step 2 — ECS new deployment** (after a successful **docker push**):

```bash
export ECS_CLUSTER="$(terraform -chdir=$TF_DIR output -raw ecs_cluster_name)"
export ECS_SERVICE="$(terraform -chdir=$TF_DIR output -raw ecs_service_name)"

aws ecs update-service \
  --region "$AWS_REGION" \
  --cluster "$ECS_CLUSTER" \
  --service "$ECS_SERVICE" \
  --force-new-deployment

aws ecs wait services-stable \
  --region "$AWS_REGION" \
  --cluster "$ECS_CLUSTER" \
  --services "$ECS_SERVICE"
```

### Database migrations (Prisma)

After the backend **ECS service** is **running stably**, run the **bash** block below to apply **migrations** (Prisma). Use the **SpendWiseApp repo root** (with **infrastructure/environments/dev**); if your cwd is the parent directory, **cd SpendWiseApp** first.

```bash
export TF_DIR="infrastructure/environments/dev"
export TFVARS="$TF_DIR/terraform.tfvars"

REGION=$(
  grep -E '^[[:space:]]*aws_region[[:space:]]*=' "$TFVARS" \
    | head -1 \
    | sed -E 's/.*"([^"]+)".*/\1/'
)
ASSIGN_PUBLIC=DISABLED
grep -E \
  '^[[:space:]]*ecs_assign_public_ip[[:space:]]*=[[:space:]]*true' \
  "$TFVARS" >/dev/null 2>&1 \
  && ASSIGN_PUBLIC=ENABLED

export REGION ASSIGN_PUBLIC
export CLUSTER=$(
  terraform -chdir="$TF_DIR" output -raw ecs_cluster_name
)
export TASK_FAMILY=$(
  terraform -chdir="$TF_DIR" output -raw ecs_task_definition_family
)
export SUBNETS=$(
  terraform -chdir="$TF_DIR" output -raw ecs_private_app_subnet_ids_csv
)
export ECS_SG=$(
  terraform -chdir="$TF_DIR" output -raw ecs_tasks_security_group_id
)

NETCFG='awsvpcConfiguration={subnets=['
NETCFG+="${SUBNETS}"
NETCFG+=']'
NETCFG+=',securityGroups=['
NETCFG+="${ECS_SG}"
NETCFG+=']'
NETCFG+=',assignPublicIp='
NETCFG+="${ASSIGN_PUBLIC}"
NETCFG+='}'

OVERRIDES='{"containerOverrides":['
OVERRIDES+='{"name":"backend","command":["npx","prisma","migrate","deploy"]}'
OVERRIDES+=']}'

export TASK_ARN=$(
  aws ecs run-task \
    --region "$REGION" \
    --cluster "$CLUSTER" \
    --launch-type FARGATE \
    --task-definition "$TASK_FAMILY" \
    --network-configuration "$NETCFG" \
    --overrides "$OVERRIDES" \
    --query 'tasks[0].taskArn' \
    --output text
)

aws ecs wait tasks-stopped \
  --region "$REGION" \
  --cluster "$CLUSTER" \
  --tasks "$TASK_ARN"

aws ecs describe-tasks \
  --region "$REGION" \
  --cluster "$CLUSTER" \
  --tasks "$TASK_ARN" \
  --query 'tasks[0].containers[0].exitCode' \
  --output text
```

**describe-tasks** prints **exitCode**: **0** means success; non-zero means failure — check ECS logs (CloudWatch **ecs_log_group** output). **prisma migrate deploy** applies only files already in **prisma/migrations**.

Design reference: [4.2.3 Backend and runtime platform](../4.2-aws-infrastructure-security/4.2.3-backend-platform/).
