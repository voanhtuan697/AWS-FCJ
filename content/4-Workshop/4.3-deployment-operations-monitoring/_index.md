---
title: "Terraform deployment (by layer)"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

## 4.3 Terraform deployment (by layer)

This chapter is a **hands-on AWS workshop**: you work from a **Git clone** of [SpendWise on GitHub](https://github.com/benguinsan/spendwise), edit configuration locally, and run Terraform under **infrastructure/** in that repository. Design rationale is in [4.2 AWS infrastructure and security foundation](../4.2-aws-infrastructure-security/). Here the focus is **order of work** and **opening the right files in the repo** — not dumping long Terraform into the browser to copy.

### Workshop flow

1. **Clone the repository.** Official URL: **https://github.com/benguinsan/spendwise**. The repo root contains **frontend/**, **backend/**, and **infrastructure/**.

```bash
git clone https://github.com/benguinsan/spendwise.git
cd spendwise
```

2. **Install tooling on your laptop:** **Terraform**, **AWS CLI**, and a current **Git** client. Configure **AWS credentials** for the lab account with **aws configure** or environment variables.

3. **Open the stack in an editor.** All Terraform for the sample environment lives under **infrastructure/** from the repo root. The dev root module is **environments/dev/main.tf**; reusable pieces live under **modules/**.

4. **Prepare variables.** In **infrastructure/environments/dev**, copy **terraform.tfvars.example** to **terraform.tfvars** and fill in values. **Do not commit** tokens or secrets.

5. **Deploy in layers** using the sections below. Run **terraform init**, **plan**, and **apply** in **environments/dev**. In practice one **apply** often covers the whole stack; the numbered sections explain **which module plays which role**.

**How to use these pages:** Treat them as a **checklist and roadmap**. Open the matching paths in your clone — **do not** copy whole Terraform pages from the workshop site.

### Why use Terraform

- **Infrastructure as code:** VPCs, security groups, RDS, ECS, and related resources live in versioned files.
- **Modules:** **modules/vpc**, **modules/ecs**, **modules/rds**, **modules/amplify**, **modules/cognito**, and others line up with the layers in section 4.2.
- **Ordering:** Terraform resolves dependencies so **network → compute → data** stay consistent.

### Repository layout (infrastructure)

![Folder tree of SpendWiseApp/infrastructure: docs, environments, modules (alb, amplify, bastion, cloudfront_alb, cognito, db_password_secret, ecr, ecs, monitoring, rds, security_groups, vpc, waf_alb), scripts](/images/4-workshop/4.3-spendwise-infrastructure-folder-tree.png)

### Content

1. [4.3.1 Run infrastructure with Terraform](4.3.1-run-infrastructure-terraform/)
2. [4.3.2 Frontend hosting](4.3.2-frontend-hosting/)
3. [4.3.3 Deploy backend](4.3.3-backend/)
