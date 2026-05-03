---
title: "Run infrastructure with Terraform"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.3.1 </b> "
aliases:
  - /4-workshop/4.3-deployment-operations-monitoring/4.3.1-vpc-network/
---

## 4.3.1 Run infrastructure with Terraform

This page walks through **Terraform operations** to deploy the SpendWise stack on AWS: clone the repo, create **terraform.tfvars**, then **init / plan / apply**.

Follow [section 4.3](../) for the workshop flow. Commands below assume your shell is in the **dev root module**:

```bash
git clone https://github.com/benguinsan/spendwise.git
cd spendwise/infrastructure/environments/dev
```

### Prerequisites

1. **Terraform** and **AWS CLI** credentials (profile or environment variables) with permission to create the resources your stack needs (VPC, EC2, RDS, IAM, …).
2. In **spendwise/infrastructure/environments/dev**, create **terraform.tfvars** from [terraform.tfvars.example](https://github.com/benguinsan/spendwise/blob/main/infrastructure/environments/dev/terraform.tfvars.example). Fill in real values; see **variables.tf**. **Do not commit terraform.tfvars** if it contains secrets.

   Example **terraform.tfvars** layout (illustrative; **amplify_access_token** must be a PAT you create on GitHub):

```hcl
# infrastructure/environments/dev/terraform.tfvars — example only

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
amplify_access_token   = "<your GitHub PAT — do not share>"
amplify_branch_name    = "main"
```

3. Configure **backend.tf** for remote state if you use it; **local** state is fine for experiments.
4. Read **main.tf** and **variables.tf** under **environments/dev** to see which modules are invoked before you apply.

### Run Terraform

From **environments/dev**:

```bash
terraform init
terraform validate
terraform plan
terraform apply
```

After a successful apply:

```bash
terraform output
```

Example output; **DNS names, IDs, and ARNs will differ** per AWS account and deployment — this shows the kinds of values returned (ALB, Amplify, CloudFront, Cognito, ECS, RDS, …):

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

When you need to remove the stack from AWS (destructive — mind your data):

```bash
terraform destroy
```

### Notes

- One **apply** usually creates **everything** declared in **main.tf** (networking, ALB, ECS, RDS, Amplify, Cognito, …) according to Terraform dependencies — workshop sections [4.3.2](../4.3.2-frontend-hosting/) and [4.3.3](../4.3.3-backend/) explain **what each module group does**; you do not have to apply partial stacks separately.
- Open **modules/** in the repo to inspect resources; compare with the big-picture architecture in [section 4.2](../../4.2-aws-infrastructure-security/).

---
