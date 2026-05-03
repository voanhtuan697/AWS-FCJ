---
title: "Chạy hạ tầng với Terraform"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.3.1 </b> "
aliases:
  - /4-workshop/4.3-deployment-operations-monitoring/4.3.1-vpc-network/
---

## 4.3.1 Chạy hạ tầng với Terraform

Trang này hướng dẫn **thao tác Terraform** để triển khai stack SpendWise lên AWS: clone repo, tạo **terraform.tfvars**, rồi **init / plan / apply**. 

Làm theo [mục 4.3](../) về luồng workshop. Mọi lệnh dưới đây giả định bạn đang ở **root module dev**:

```bash
git clone https://github.com/benguinsan/spendwise.git
cd spendwise/infrastructure/environments/dev
```

### Chuẩn bị

1. **Terraform** và **AWS CLI** đã cấu hình credential (profile hoặc biến môi trường) có quyền tạo tài nguyên cần cho stack (VPC, EC2, RDS, IAM, …).
2. Trong **spendwise/infrastructure/environments/dev**, tạo **terraform.tfvars** từ [terraform.tfvars.example](https://github.com/benguinsan/spendwise/blob/main/infrastructure/environments/dev/terraform.tfvars.example). Điền giá trị thật; xem **variables.tf**. **Không commit terraform.tfvars** nếu có secret.

   **Ví dụ** cấu trúc **terraform.tfvars** (minh họa; biến **amplify_access_token** là PAT do bạn tự tạo trên GitHub):

```hcl
# infrastructure/environments/dev/terraform.tfvars — ví dụ minh họa

aws_region         = "us-east-1"
project_name       = "spendwise"
environment        = "dev"
enable_nat_gateway = false

app_container_port      = 3000
alb_health_check_path   = "/"
alb_acm_certificate_arn = ""

enable_api_cloudfront = true

create_rds   = true
db_username  = "appuser"
db_name      = "spendwise"
rds_multi_az = false
db_password  = ""
db_password_secret_recovery_window_days = 0
enable_waf                             = false
waf_enable_cloudwatch_metrics            = false
ecs_backend_image_tag = "v3"

ecs_backend_environment = [
  { name = "PORT", value = "3000" },
  { name = "DEBUG_LOG_DATABASE_URL", value = "1" },
  { name = "MIGRATE_FAIL_SLEEP_SEC", value = "300" }
]

create_bastion              = true
bastion_instance_type       = "t3.nano"
bastion_associate_public_ip = true

amplify_repository_url = "https://github.com/benguinsan/spendwise"
amplify_access_token   = "<điền PAT GitHub của bạn — không chia sẻ>"
amplify_branch_name    = "main"
```

3. Cấu hình **backend** trong **backend.tf** nếu dùng remote state; có thể dùng state **local** khi thử.
4. Đọc **main.tf** và **variables.tf** trong **environments/dev** để hiểu module nào được gọi trước khi apply.

### Chạy Terraform

Trong thư mục **environments/dev**:

```bash
terraform init
terraform validate
terraform plan
terraform apply
```

Sau khi apply thành công:

```bash
terraform output
```

Ví dụ kết quả; **tên DNS, ID, ARN sẽ khác** theo tài khoản AWS và lần triển khai — dùng để hình dung các output thường gặp (ALB, Amplify, CloudFront, Cognito, ECS, RDS, …):

```text
Outputs:

alb_dns_name = "spendwise-dev-alb-451994859.us-east-1.elb.amazonaws.com"
amplify_app_id = "d291555glmfdju"
amplify_branch_name = "main"
amplify_default_domain = "d291555glmfdju.amplifyapp.com"
amplify_route53_name_servers = []
api_cloudfront_domain_name = "d268q4rmn4bwbh.cloudfront.net"
api_cloudfront_url = "https://d268q4rmn4bwbh.cloudfront.net"
aws_region = "us-east-1"
bastion_instance_id = "i-02570dd1595695547"
cognito_client_id = "7hqi2rc2gsf2n18d1nnmj901sm"
cognito_user_pool_id = "us-east-1_Vi9hq65tq"
db_password_secret_arn = "arn:aws:secretsmanager:us-east-1:123456789012:secret:spendwise-dev/db/password-yxQzCU"
ecr_repository_url = "123456789012.dkr.ecr.us-east-1.amazonaws.com/spendwise-dev-backend"
ecs_cluster_name = "spendwise-dev-cluster"
ecs_fargate_assign_public_ip = "DISABLED"
ecs_log_group = "/ecs/spendwise-dev/backend"
ecs_private_app_subnet_ids_csv = "subnet-0c5e7023772375ff2,subnet-0569a2e34885aa89c"
ecs_service_name = "spendwise-dev-backend"
ecs_task_definition_family = "spendwise-dev-backend"
ecs_tasks_security_group_id = "sg-0c5e475c0ff2f228b"
rds_endpoint = "spendwise-dev-postgres.cizq0eiqsxvu.us-east-1.rds.amazonaws.com:5432"
```

Khi cần gỡ stack khỏi AWS (cẩn trọng dữ liệu):

```bash
terraform destroy
```

### Gợi ý

- Một lần **apply** thường tạo **toàn bộ** tài nguyên khai báo trong **main.tf** (mạng, ALB, ECS, RDS, Amplify, Cognito, …) theo phụ thuộc Terraform — các mục [4.3.2](../4.3.2-frontend-hosting/) và [4.3.3](../4.3.3-backend/) trong workshop giúp đọc **ý nghĩa** từng nhóm module, không bắt buộc apply từng phần riêng.
- Mở **modules/** trong repo để tra cứu resource; đối chiếu kiến trúc tổng quan tại [mục 4.2](../../4.2-aws-infrastructure-security/).

---
