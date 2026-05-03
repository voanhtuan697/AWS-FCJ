---
title: "Triển khai database"
date: 2026-03-10
weight: 4
chapter: false
pre: " <b> 4.3.4 </b> "
---

## 4.3.4 Triển khai database

Khi create_rds = true, SpendWiseApp/infrastructure/environments/dev/main.tf bật **module "db_password_secret"** (dùng count) và **module "rds"**. Khi create_rds = false (mặc định trong biến để tiết kiệm), RDS và secret mật khẩu không tạo; backend không nhận DATABASE_URL từ Terraform.

### Mã — root: secret mật khẩu + RDS (environments/dev/main.tf)

count gắn với var.create_rds. Module secret cấp local.db_password_effective cho RDS và cho locals ECS/Cognito.

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (trích)
locals {
  db_password_effective = var.create_rds ? module.db_password_secret[0].password : ""
}

module "db_password_secret" {
  count  = var.create_rds ? 1 : 0
  source = "../../modules/db_password_secret"

  project_name            = var.project_name
  environment             = var.environment
  plaintext_password      = var.db_password
  recovery_window_in_days = var.db_password_secret_recovery_window_days
}

module "rds" {
  source = "../../modules/rds"

  project_name            = var.project_name
  environment             = var.environment
  create_rds              = var.create_rds
  private_data_subnet_ids = module.vpc.private_data_subnet_ids
  rds_security_group_id   = module.security_groups.rds_security_group_id
  db_name                 = var.db_name
  db_username             = var.db_username
  db_password             = local.db_password_effective
}
```

### Mã — Secrets Manager + mật khẩu ngẫu nhiên (modules/db_password_secret/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/db_password_secret/main.tf (trích)
resource "random_password" "db" {
  count   = trimspace(var.plaintext_password) == "" ? 1 : 0
  length  = 24
  special = true
  override_special = "!#$%&*()-_=+[]{}<>:?"
}

resource "aws_secretsmanager_secret" "db_password" {
  name                    = "${local.name}/db/password"
  recovery_window_in_days = var.recovery_window_in_days
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = local.db_password
}
```

### Mã — instance RDS (modules/rds/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/rds/main.tf (trích)
resource "aws_db_instance" "this" {
  count = var.create_rds ? 1 : 0

  identifier             = "${local.name}-postgres"
  engine                 = "postgres"
  engine_version         = "16"
  db_subnet_group_name   = aws_db_subnet_group.this[0].name
  vpc_security_group_ids = [var.rds_security_group_id]
  publicly_accessible    = false
  multi_az               = false
  # ...
}
```

### modules/db_password_secret — tài nguyên và vai trò

| Tài nguyên | Để làm gì |
|------------|-----------|
| **random_password** (tùy) | Nếu db_password để trống trong tfvars, Terraform tự sinh mật khẩu mạnh, tránh commit secret lên Git. |
| **aws_secretsmanager_secret + version** | Lưu **mật khẩu RDS** dạng secret string; cửa sổ khôi phục cấu hình qua db_password_secret_recovery_window_days (0 = xóa ngay, phù hợp dev). |

Giá trị plaintext cũng được đưa vào module RDS và ghép **DATABASE_URL** cho ECS và Lambda Cognito qua locals trong main.tf.

### modules/rds — tài nguyên và vai trò

| Tài nguyên | Để làm gì |
|------------|-----------|
| **aws_db_subnet_group** | Gắn RDS vào **private_data_subnet_ids** — chỉ tầng dữ liệu. |
| **aws_db_instance (PostgreSQL 16)** | CSDL ứng dụng (db_name, db_username); **publicly_accessible = false**; **vpc_security_group_ids** = SG RDS (chỉ Postgres từ ECS / bastion). Trong module đang ưu tiên dev: **multi_az = false**, backup ngắn, skip_final_snapshot — production cần chỉnh lại. |

### Bastion tùy chọn (liên quan truy cập DB)

module "bastion" là module count riêng trong cùng root; mô tả chi tiết ở [4.3.5](../4.3.5-platform-edge-ops/), mục đích là **SSM port forwarding** tới RDS mà không mở Postgres ra Internet.

Tham chiếu thiết kế: [4.2.4 Cơ sở dữ liệu](../4.2-aws-infrastructure-security/4.2.4-database/).
