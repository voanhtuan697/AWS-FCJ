---
title : "Frontend Hosting and user authentication"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 4.2.2 </b> "
---

## 4.2.2 Frontend Hosting and user authentication

This section covers **AWS Amplify** for frontend hosting and **Amazon Cognito** for user authentication:

![Users to Amplify (front-end hosting); GitHub repo to Amplify; Amplify to Amazon Cognito](/images/4-Workshop/4.2.2-amplify-github-cognito.png)

### Amplify

**Role:** Build and deliver **Next.js** from a Git repository; configure build-time environment variables (for example **NEXT_PUBLIC_API_URL**) so the browser calls the correct backend API. That URL is set in Terraform from the ALB: when **alb_acm_certificate_arn** is set, the frontend build gets **https://** and the ALB DNS name; otherwise **http://** (same hostname shape as today’s **dev** root module). When **enable_api_cloudfront** is true, the build can use the CloudFront URL instead. A **stable API hostname** such as **api.example.com** requires DNS records at your provider pointing at the ALB or CloudFront target; this repository does not define those records.

**Why we chose it:** Managed CI/CD and HTTPS reduce the burden of operating your own static web/CDN stack; fits a monorepo when the frontend app root is explicit. For **Next.js**, a pure static **S3 + CloudFront** pattern is often **more cumbersome** because the framework may use **SSR** (server-side rendering).

### Cognito

**Role:** **User Pool** and **App client** for sign-up, sign-in, and issuing tokens; optional **Lambda triggers** (for example **PostConfirmation** syncing users into RDS).

**Why we chose it:** Fully managed authentication, standard **JWTs**, and integration with the frontend and Lambda — no need to build and run a dedicated auth server. Strong efficiency and flexible **scaling** as usage grows.
