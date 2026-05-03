---
title: "Terraform deployment (by layer)"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

## 4.3 Terraform deployment (by layer)

This chapter is a **hands-on AWS workshop**: you work from a **Git clone** of [SpendWise on GitHub](https://github.com/benguinsan/spendwise), edit configuration locally, and apply Terraform under **infrastructure/** in that repository. Design rationale stays in [4.2 AWS infrastructure and security foundation](../4.2-aws-infrastructure-security/). Here we focus on **order of work** and **what to open in the repo** — not pasting long Terraform into the browser.

### Workshop flow

1. **Clone the repository.** Official URL: **https://github.com/benguinsan/spendwise**. The repo root contains **frontend/**, **backend/**, and **infrastructure/**.

```bash
git clone https://github.com/benguinsan/spendwise.git
cd spendwise
```

2. **Install tooling on your laptop:** **Terraform**, **AWS CLI**, and a current **Git** client. Configure **AWS credentials** for the account you use in the lab **aws configure** or environment variables.

3. **Open the stack in an editor.** All Terraform for the sample environment lives under **infrastructure/** from the repo root. The dev root module is **environments/dev/main.tf**; reusable pieces are under **modules/**.

4. **Prepare variables.** From **infrastructure/environments/dev**, copy **terraform.tfvars.example** to **terraform.tfvars** and fill values. Treat tokens and secrets as confidential.

5. **Deploy in layers** following the sections below. Run **terraform init**, **plan**, and **apply** from **environments/dev** when your instructor says the dependencies for that layer are ready. In practice one **apply** often covers the whole stack; the numbered topics explain **which modules matter** for each concern.

**How to use these pages:** Treat them as a **checklist and map**. Open the cited paths in your clone instead of copying multi-page code blocks from the workshop site.

### Why Terraform fits this stack

- **Infrastructure as code:** VPCs, security groups, RDS, ECS, and related resources stay in versioned files.
- **Modules:** **modules/vpc**, **modules/ecs**, **modules/rds**, **modules/amplify**, **modules/cognito**, and others mirror the layered view in section 4.2.
- **Ordering:** Terraform resolves dependencies so **network → compute → data** stay consistent.

### Repository layout (infrastructure)

![Folder tree of SpendWiseApp/infrastructure: docs, environments, modules (alb, amplify, bastion, cloudfront_alb, cognito, db_password_secret, ecr, ecs, monitoring, rds, security_groups, vpc, waf_alb), scripts](/images/4-Workshop/4.3-spendwise-infrastructure-folder-tree.png)

### Chapters

1. [4.3.1 Run infrastructure with Terraform](4.3.1-run-infrastructure-terraform/)
2. [4.3.2 Frontend hosting](4.3.2-frontend-hosting/)
3. [4.3.3 Deploy backend](4.3.3-backend/)
