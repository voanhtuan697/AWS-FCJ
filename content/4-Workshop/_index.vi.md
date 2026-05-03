---
title: "Workshop"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 4. </b> "
---

# Workshop: Kiến trúc SpendWiseApp trên AWS

Phần workshop này **đi sâu vào kiến trúc hạ tầng AWS** của **SpendWiseApp**: các dịch vụ, phân lớp mạng và hướng bảo mật. **Triển khai Terraform theo lớp** (VPC, frontend/xác thực, backend) nằm ở [4.3 Triển khai Terraform (theo lớp)](4.3-deployment-operations-monitoring/); phần còn lại của stack (RDS, ALB, v.v.) vẫn trong repo **infrastructure** và [4.2](4.2-aws-infrastructure-security/) khi cần nắm thiết kế.

Nội dung được gom thành các nhóm chính để thuận tiện theo dõi:
- Tổng quan và kiến trúc hệ thống.
- Nền tảng hạ tầng AWS và bảo mật.
- Triển khai Terraform (theo lớp), gồm hosting frontend và vận hành backend (ECR/ECS).
- Chi phí, rủi ro và lộ trình mở rộng.

#### Nội dung

1. [4.1 Tổng quan và kiến trúc hệ thống](4.1-system-overview-architecture/)
2. [4.2 Nền tảng hạ tầng AWS và bảo mật](4.2-aws-infrastructure-security/)
3. [4.3 Triển khai Terraform (theo lớp)](4.3-deployment-operations-monitoring/)
4. [4.5 Chi phí, rủi ro và định hướng mở rộng](4.5-cost-risk-expansion-roadmap/)
5. [4.6 Dọn dẹp tài nguyên](4.6-legacy-cleanup/)