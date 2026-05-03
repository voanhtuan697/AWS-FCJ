---
title : "Clean up"
date : 2024-01-01
weight : 99
chapter : false
pre : " <b> 4.6. </b> "
draft : false
---

## 4.6 Clean up

When the workshop stack is no longer needed, remove AWS resources provisioned by Terraform with **terraform destroy**. That command plans and tears down everything still tracked in the **state file** for the environment you run it in - it is **destructive** (for example **RDS** data and **Secrets Manager** secrets managed by the stack).

### Where to run it

From the repo **SpendWiseApp**, use the same environment directory you used for **apply** (see [4.3.1 Run infrastructure with Terraform](../4.3-deployment-operations-monitoring/4.3.1-run-infrastructure-terraform/)):

```bash
cd infrastructure/environments/dev   # or environments/prod
terraform destroy
```

Terraform prints the planned destroys; type **yes** when prompted unless you intentionally pass **-auto-approve** (only for automation - avoid in shared or production accounts by mistake).

### Before you destroy

- Confirm **AWS credentials** and **region** point at the intended account (profile / **AWS_PROFILE**, **AWS_REGION**).
- If you use a **remote backend**, ensure you are destroying the **correct workspace/state** (dev vs prod).
- Expect **downtime** for the app: ALB, ECS, Amplify-linked resources, Cognito pool in that stack, etc., disappear in dependency order - you do not need to destroy manually per service if everything is under this root module.

### After destroy

- **State** should show no managed resources (or only data sources) for that root; if destroy fails partway, fix the reported error and run **terraform destroy** again until it completes.
- **Leftovers** (manual resources, objects outside Terraform, or retained S3/ECR policies) are not removed by Terraform - clean those in the console or CLI if your account policy requires it.

For a short reminder in the apply/plan flow, the same command appears in [4.3.1](../4.3-deployment-operations-monitoring/4.3.1-run-infrastructure-terraform/).
