---
title : "Nền tảng hạ tầng AWS và bảo mật"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

## 4.2 Nền tảng hạ tầng AWS và bảo mật

**Nền tảng hạ tầng và bảo mật.** Kiến trúc hạ tầng AWS Cloud của **SpendWiseApp** được đặt trong **mô hình ba tầng (3-tier)** — phù hợp ứng dụng web/app và có thể điều chỉnh chi phí theo quy mô:

- **Tầng giao diện (Presentation Tier)** — trải nghiệm người dùng và hosting frontend (ví dụ Amplify).
- **Tầng ứng dụng (Application Tier)** — API và xử lý nghiệp vụ (ví dụ ALB, ECS Fargate, ECR).
- **Tầng dữ liệu (Data Tier)** — CSDL và lưu trữ bền (ví dụ RDS PostgreSQL, Secrets Manager cho mật khẩu DB).

Chi tiết triển khai được chia **theo lớp kỹ thuật** (VPC → Frontend Hosting và xác thực người dùng → backend runtime → CSDL), bám các module thực tế:

1. [4.2.1 VPC và mạng](4.2.1-vpc-network/) — `vpc`, VPC endpoints, `security_groups`.
2. [4.2.2 Frontend Hosting và xác thực người dùng](4.2.2-client-facing/) — `amplify`, `cognito`.
3. [4.2.3 Backend và nền tảng xử lý](4.2.3-backend-platform/) — `alb`, `waf_alb`, `route53_api`, `ecr`, `ecs`, `monitoring`.
4. [4.2.4 Cơ sở dữ liệu](4.2.4-database/) — `rds`, `db_password_secret`, `bastion`.