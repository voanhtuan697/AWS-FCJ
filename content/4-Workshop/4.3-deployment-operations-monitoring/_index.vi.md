---
title: "Triển khai Terraform (theo lớp)"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

## 4.3 Triển khai Terraform (theo lớp)

Phần này mô tả **triển khai hạ tầng SpendWiseApp bằng Terraform** theo cùng trục với [4.2 Nền tảng hạ tầng AWS và bảo mật](../4.2-aws-infrastructure-security/) — từ VPC và mạng, hosting frontend và xác thực, backend, database, đến các thành phần **nền tảng / biên / vận hành** không gói gọn hết trong ranh giới “chỉ FE” hay “chỉ BE”.

### Terraform — vai trò và lý do lựa chọn

- **Hạ tầng dưới dạng mã (IaC):** Trạng thái mong muốn của VPC, security group, RDS, ECS… được lưu trong cấu hình có phiên bản; có thể review diff, tái tạo môi trường và đồng bộ dev / staging / production, hạn chế lệch cấu hình thủ công.
- **Module hóa:** Các module Terraform trong kho SpendWiseApp (ví dụ vpc, ecs, rds, amplify, cognito) bám theo **phân lớp kiến trúc** ở mục 4.2, giúp phân ranh giới trách nhiệm và giảm sao chép giữa các môi trường.
- **Phụ thuộc và thứ tự:** Terraform xử lý thứ tự tạo / hủy tài nguyên và chuyển output giữa các module (ví dụ subnet cho ECS, bí mật cho task) — phù hợp chuỗi **mạng → compute → dữ liệu**.

Lý do chọn từng dịch vụ và cách bảo mật vẫn nằm ở **mục 4.2**. Các trang con dưới đây tập trung **cách gom lệnh triển khai Terraform** trong thực tế.

### Nội dung

1. [4.3.1 Triển khai VPC và mạng](4.3.1-vpc-network/)
2. [4.3.2 Triển khai hosting frontend và xác thực người dùng](4.3.2-frontend-hosting-auth/)
3. [4.3.3 Triển khai backend](4.3.3-backend/)
4. [4.3.4 Triển khai database](4.3.4-database/)
5. [4.3.5 Nền tảng, biên và vận hành](4.3.5-platform-edge-ops/) — ALB, WAF, Route 53, Bastion, giám sát (dùng chung nhiều lớp; không thuộc hẳn một phía FE hay BE).
