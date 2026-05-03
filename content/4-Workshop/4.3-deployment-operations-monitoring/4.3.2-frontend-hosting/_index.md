---
title: "Frontend hosting"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 4.3.2 </b> "
aliases:
  - /4-workshop/4.3-deployment-operations-monitoring/4.3.2-frontend-hosting-auth/
---

## 4.3.2 Frontend hosting

### AWS Amplify Console after terraform apply

After a successful **terraform apply**, open **AWS Amplify** in the **Region** you deployed to (e.g. **US East (N. Virginia)**, region code **us-east-1**). The **All apps** page lists the app Terraform created; the UI may look like this:

![All apps — Amplify after a successful apply](/images/4-workshop/amplify-console-after-terraform-apply.png)

After you open the app, the **Overview** page shows the **main** branch; open that branch to deploy.

![Amplify — Overview, main branch](/images/4-workshop/amplify-console-overview-main-branch.png)

Related design (Amplify, API URL): [4.2.2 Frontend hosting and user authentication](../../4.2-aws-infrastructure-security/4.2.2-client-facing/).

---
