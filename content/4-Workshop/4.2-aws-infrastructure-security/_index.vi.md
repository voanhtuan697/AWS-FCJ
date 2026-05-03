---
title : "Nền tảng hạ tầng AWS và bảo mật"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 4.2. </b> "
---

## 4.2 Nền tảng hạ tầng AWS và bảo mật

**Nền tảng hạ tầng và bảo mật.** **SpendWiseApp** trên AWS Cloud theo **mô hình ba tầng** phù hợp tải web/app và **mở rộng quy mô có kiểm soát chi phí**:

- **Tầng trình bày (Presentation tier)** — trải nghiệm người dùng và hosting frontend (ví dụ Amplify).
- **Tầng ứng dụng (Application tier)** — API và logic nghiệp vụ (ví dụ ALB, ECS Fargate, ECR).
- **Tầng dữ liệu (Data tier)** — lưu trữ bền và cơ sở dữ liệu (ví dụ RDS PostgreSQL, Secrets Manager cho thông tin đăng nhập DB).

Chi tiết triển khai được chia **theo lớp kỹ thuật** (VPC → Frontend Hosting và xác thực người dùng → backend runtime → cơ sở dữ liệu), **khớp với các module** đó:

1. [4.2.1 VPC và mạng](4.2.1-vpc-network/).
2. [4.2.2 Frontend Hosting và xác thực người dùng](4.2.2-client-facing/).
3. [4.2.3 Backend và nền tảng xử lý](4.2.3-backend-platform/).
4. [4.2.4 Cơ sở dữ liệu](4.2.4-database/).
