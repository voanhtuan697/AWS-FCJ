---
title: "Deploy database"
date: 2026-03-10
weight: 4
chapter: false
pre: " <b> 4.3.4 </b> "
---

## 4.3.4 Deploy database

When create_rds = true, SpendWiseApp/infrastructure/environments/dev/main.tf enables **module "db_password_secret"** (counted) and **module "rds"**. If create_rds = false (default in variables for cost), RDS and the password secret are skipped and the backend runs without a live DATABASE_URL from Terraform.

### Code — root wiring for DB secret + RDS (environments/dev/main.tf)

count ties creation to var.create_rds. The password module feeds local.db_password_effective, which RDS and the ECS/Cognito locals consume.

```terraform
# SpendWiseApp/infrastructure/environments/dev/main.tf (excerpt)
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

### Code — Secrets Manager + random password (modules/db_password_secret/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/db_password_secret/main.tf (excerpt)
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

### Code — RDS instance (modules/rds/main.tf)

```terraform
# SpendWiseApp/infrastructure/modules/rds/main.tf (excerpt)
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

### modules/db_password_secret — what it creates and why

| Resource | Role |
|----------|------|
| **random_password** (optional) | If db_password is empty in tfvars, Terraform generates a strong password so nothing sensitive is committed to Git. |
| **aws_secretsmanager_secret + version** | Stores the **RDS master password** as a secret string; recovery window is configurable (db_password_secret_recovery_window_days — 0 for immediate delete in dev). |

The plaintext value is also passed into the RDS module and composed into **DATABASE_URL** for ECS and the Cognito Lambda via locals in main.tf.

### modules/rds — what it creates and why

| Resource | Role |
|----------|------|
| **aws_db_subnet_group** | Pins RDS to **private_data_subnet_ids** so the instance only exists on the data tier. |
| **aws_db_instance (PostgreSQL 16)** | Application database (db_name, db_username); **publicly_accessible = false**; **vpc_security_group_ids** = RDS SG (Postgres from ECS / bastion only). Dev-oriented flags in module: **multi_az = false**, short backup retention, skip_final_snapshot — adjust for production. |

### Optional bastion (related to DB access)

module "bastion" is a separate counted module in the same root module; it is documented with [4.3.5](../4.3.5-platform-edge-ops/) but exists so operators can use **SSM port forwarding** to reach RDS without opening Postgres to the world.

Design reference: [4.2.4 Database](../4.2-aws-infrastructure-security/4.2.4-database/).
