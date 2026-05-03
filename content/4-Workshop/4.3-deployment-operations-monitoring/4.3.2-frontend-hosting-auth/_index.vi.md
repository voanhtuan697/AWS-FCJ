---
title: "Triển khai hosting frontend và xác thực người dùng"
date: 2026-03-10
weight: 2
chapter: false
pre: " <b> 4.3.2 </b> "
---

## 4.3.2 Triển khai hosting frontend và xác thực người dùng

SpendWiseApp/infrastructure/environments/dev/main.tf triển khai **module "amplify"** và **module "cognito"**, cộng **aws_route53_zone.amplify** tùy chọn khi cần zone Route 53 public riêng cho DNS custom domain của Amplify.

### Mã — Amplify + Cognito (environments/dev/main.tf)

build_spec nhúng chỉ rõ monorepo **appRoot: frontend**, artifact .next. NEXT_PUBLIC_API_URL trỏ DNS của ALB (http/https tùy ACM). Lambda post-confirmation và VPC của Cognito chỉ bật khi create_rds = true.

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (trích)
module "amplify" {
  source = "../../modules/amplify"

  project_name   = var.project_name
  environment    = var.environment
  repository_url = var.amplify_repository_url
  access_token     = var.amplify_access_token
  branch_name      = var.amplify_branch_name

  build_spec = <<-EOF
version: 1
applications:
  - appRoot: frontend
    frontend:
      phases:
        preBuild:
          commands:
            - npm ci --legacy-peer-deps
        build:
          commands:
            - npm run build
      artifacts:
        baseDirectory: .next
        files:
          - "**/*"
      cache:
        paths:
          - node_modules/**/*
EOF

  app_environment_variables = {
    AMPLIFY_MONOREPO_APP_ROOT = "frontend"
    NEXT_PUBLIC_API_URL       = local.frontend_api_url
  }
}

module "cognito" {
  source = "../../modules/cognito"

  project_name = var.project_name
  environment  = var.environment

  enable_post_confirmation_trigger = var.create_rds
  post_confirmation_database_url   = local.cognito_post_confirmation_database_url
  post_confirmation_subnet_ids     = module.vpc.private_data_subnet_ids
  post_confirmation_security_group_ids = [module.security_groups.ecs_tasks_security_group_id]
}
```

### Mã — hosted zone tùy chọn cho DNS Amplify

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (trích)
resource "aws_route53_zone" "amplify" {
  count = var.enable_amplify_hosted_zone ? 1 : 0
  name  = var.amplify_hosted_zone_name
  # ...
}
```

### modules/amplify — tài nguyên và vai trò

| Tài nguyên | Để làm gì |
|------------|-----------|
| **aws_amplify_app** | Kết nối repo Git (URL + PAT trong biến) và cấu hình cách Amplify build **Next.js**. |
| **aws_amplify_branch** | Theo dõi nhánh (vd. main); có thể bật build khi push tùy enable_auto_build. |
| **Build spec (nhúng trong main.tf)** | Monorepo: appRoot: frontend để npm ci / npm run build chạy trong ./frontend; artifact từ .next. |
| **Biến môi trường trên app** | NEXT_PUBLIC_API_URL lấy từ local trỏ tới **ALB** (http hay https tùy alb_acm_certificate_arn). Trình duyệt cần khi build để gọi /auth, /users, … |

amplify_access_token và thông tin nhạy cảm đặt trong terraform.tfvars hoặc CI — không commit token thật.

### modules/cognito — tài nguyên và vai trò

| Tài nguyên | Để làm gì |
|------------|-----------|
| **aws_cognito_user_pool** | Người dùng theo email, chính sách mật khẩu, tùy chọn trigger **Lambda post-confirmation**. |
| **aws_cognito_user_pool_client (web)** | Client SPA công khai: generate_secret = false, luồng SRP / password / refresh cho frontend. |
| **Lambda post-confirmation** (khi create_rds = true ở dev) | Sau khi xác nhận email, **ghi/cập nhật user vào PostgreSQL** đồng bộ với Cognito. Chạy trong **subnet private data**, dùng **cùng SG với ECS task** để vào RDS (post_confirmation_security_group_ids). Khi create_rds = false thì tắt trigger để có auth mà chưa tốn RDS. |

Module còn có **IAM role**, gắn **VPC access** cho Lambda, **archive_file** đóng gói, **null_resource** chạy npm install dependency Lambda lúc apply.

### Tùy chọn: Route 53 zone cho custom domain Amplify

resource "aws_route53_zone" "amplify" (theo enable_amplify_hosted_zone) tạo **zone con public** (vd. app.dev.example.com). Cần **ủy quyền NS** từ DNS cha tới name server của zone (output amplify_route53_name_servers). Khác với DNS API trên ALB.

Tham chiếu thiết kế: [4.2.2 Frontend Hosting và xác thực người dùng](../4.2-aws-infrastructure-security/4.2.2-client-facing/).
