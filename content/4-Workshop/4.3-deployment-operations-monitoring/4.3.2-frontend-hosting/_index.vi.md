---
title: "Hosting frontend"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 4.3.2 </b> "
aliases:
  - /4-workshop/4.3-deployment-operations-monitoring/4.3.2-frontend-hosting-auth/
---

## 4.3.2 Hosting frontend

### AWS Amplify Console sau khi terraform apply

Sau khi **terraform apply** thành công, mở **AWS Amplify** đúng **Region** đã triển khai (ví dụ **US East (N. Virginia)**, mã vùng **us-east-1**). Trang **All apps** liệt kê ứng dụng Terraform vừa tạo; giao diện có thể như sau:

![Trang All apps — Amplify sau khi apply thành công](/images/4-workshop/amplify-console-after-terraform-apply.png)

Sau khi bấm vào ứng dụng, màn hình **Overview** hiển thị nhánh **main**; truy cập nhánh này để thực hiện deploy.

![Amplify — Overview, nhánh main](/images/4-workshop/amplify-console-overview-main-branch.png)

Thiết kế liên quan (Amplify, API URL): [4.2.2 Frontend Hosting và xác thực người dùng](../../4.2-aws-infrastructure-security/4.2.2-client-facing/).

---
