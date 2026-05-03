---
title: "Deploy frontend hosting and user authentication"
date: 2026-03-10
weight: 2
chapter: false
pre: " <b> 4.3.2 </b> "
---

## 4.3.2 Deploy frontend hosting and user authentication

SpendWiseApp/infrastructure/environments/dev/main.tf provisions **module "amplify"** and **module "cognito"**, plus an optional **aws_route53_zone.amplify** when you want a dedicated public zone for Amplify custom-domain DNS.

### Code — Amplify + Cognito (environments/dev/main.tf)

The inline build_spec pins the monorepo app root to frontend and publishes .next artifacts. NEXT_PUBLIC_API_URL is wired to the ALB DNS name (scheme follows ACM). Cognito’s post-confirmation Lambda and VPC config only activate when create_rds is true.

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (excerpt)
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

### Code — optional hosted zone for Amplify DNS

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (excerpt)
resource "aws_route53_zone" "amplify" {
  count = var.enable_amplify_hosted_zone ? 1 : 0
  name  = var.amplify_hosted_zone_name
  # ...
}
```

### modules/amplify — what it creates and why

| Resource | Role |
|----------|------|
| **aws_amplify_app** | Connects to your Git repository (URL + PAT in variables) and defines how Amplify builds the **Next.js** app. |
| **aws_amplify_branch** | Tracks a branch (e.g. main); enables CI-style builds on push depending on enable_auto_build. |
| **Build spec (inline in main.tf)** | Monorepo layout: appRoot: frontend so npm ci / npm run build run under ./frontend; artifacts from .next. |
| **Environment variables on the app** | NEXT_PUBLIC_API_URL is set from a local value that points at the **ALB** (http vs https depends on alb_acm_certificate_arn). The browser needs this at build time for calls to /auth, /users, etc. |

Secrets (amplify_access_token) belong in terraform.tfvars or CI secrets — never commit real tokens.

### modules/cognito — what it creates and why

| Resource | Role |
|----------|------|
| **aws_cognito_user_pool** | Email-based users, password policy, optional **post-confirmation Lambda** trigger. |
| **aws_cognito_user_pool_client (web)** | Public SPA client: generate_secret = false, SRP / password / refresh flows for the frontend. |
| **Post-confirmation Lambda** (when create_rds = true in dev) | After email confirmation, **upserts the user into PostgreSQL** so app data stays aligned with Cognito. Runs in **private data subnets** with the **same SG as ECS tasks** for RDS access (post_confirmation_security_group_ids). When create_rds = false, the trigger is disabled so you can stand up auth without paying for RDS. |

Supporting pieces in that module include **IAM roles**, **VPC access attachment** for the Lambda, **archive_file** packaging, and a **null_resource** that runs npm install for Lambda dependencies at apply time.

### Optional: Route 53 zone for Amplify custom domain

resource "aws_route53_zone" "amplify" (counted on enable_amplify_hosted_zone) creates a **child public zone** (e.g. app.dev.example.com). You **delegate NS** from the parent registrar to the zone’s name servers (see amplify_route53_name_servers output). This is separate from API DNS on the ALB.

Design reference: [4.2.2 Frontend hosting and user authentication](../4.2-aws-infrastructure-security/4.2.2-client-facing/).
