---
title: "Triển khai backend"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 4.3.3 </b> "
---

## 4.3.3 Triển khai backend

Backend chạy trên AWS là **ECS Fargate**, image lấy từ **ECR**. 

### Sau terraform apply — đẩy image và chạy lại ECS

Sau **terraform apply**, ECR, ECS, ALB và task definition đã có; task definition trỏ tới image tag trong **ecs_backend_image_tag** (**terraform.tfvars**). Mỗi lần đổi code backend, làm **hai bước**:

1. **Docker push** — Build image NestJS từ Dockerfile trong repo, **tag đúng** giống **ecs_backend_image_tag**, đăng nhập ECR rồi **docker push** lên **ecr_repository_url** (output Terraform).
2. **Chạy lại ECS** — **aws ecs update-service … --force-new-deployment** để task mới kéo image vừa push; đợi service **ổn định** và task healthy qua ALB.

**Bước 1 — Đăng nhập ECR, build, push** (ví dụ đứng ở thư mục cha chứa **SpendWiseApp**; nếu đã **cd** vào SpendWiseApp thì chỉnh đường dẫn **TF_DIR** và **docker build** cho khớp):

```bash
export TF_DIR=SpendWiseApp/infrastructure/environments/dev
export AWS_REGION="$(terraform -chdir=$TF_DIR output -raw aws_region)"
export ECR_REPO="$(terraform -chdir=$TF_DIR output -raw ecr_repository_url)"
export ECR_REGISTRY="${ECR_REPO%%/*}"
export IMAGE_TAG=latest   # phải khớp ecs_backend_image_tag trong Terraform

aws ecr get-login-password --region "$AWS_REGION" \
  | docker login --username AWS --password-stdin "$ECR_REGISTRY"

docker build -f SpendWiseApp/backend/Dockerfile -t spendwise-backend:"$IMAGE_TAG" SpendWiseApp/backend
docker tag spendwise-backend:"$IMAGE_TAG" "${ECR_REPO}:${IMAGE_TAG}"
docker push "${ECR_REPO}:${IMAGE_TAG}"
```

**Bước 2 — Triển khai lại ECS** (chạy sau khi **docker push** thành công):

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

### Migration CSDL (Prisma)

Sau khi **ECS task** (service backend) đã hoạt động ổn định, chạy đoạn **bash** dưới để thực hiện **migration** (Prisma). Shell đặt tại **root repo SpendWiseApp** (có **infrastructure/environments/dev**); nếu đang ở thư mục cha thì **cd SpendWiseApp** trước.

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

Kết quả **describe-tasks** là **exitCode** của container: **0** nghĩa là migrate **thành công**; khác 0 là lỗi — xem log CloudWatch (**ecs_log_group** trong terraform output). **prisma migrate deploy** chỉ áp các file đã có trong **prisma/migrations**.

Tham chiếu thiết kế: [4.2.3 Backend và nền tảng xử lý](../4.2-aws-infrastructure-security/4.2.3-backend-platform/).
