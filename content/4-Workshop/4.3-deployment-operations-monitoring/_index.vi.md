---
title: "Triển khai Terraform (theo lớp)"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 4.3. </b> "
---

## 4.3 Triển khai Terraform (theo lớp)

Chương này là **workshop thực hành trên AWS**: làm việc từ **bản clone Git** của dự án [SpendWise trên GitHub](https://github.com/benguinsan/spendwise), chỉnh cấu hình trên máy, rồi chạy Terraform trong thư mục **infrastructure/** của repo đó. Lý do chọn từng dịch vụ nằm ở [4.2 Nền tảng hạ tầng AWS và bảo mật](../4.2-aws-infrastructure-security/). Ở đây tập trung **thứ tự làm việc** và **mở đúng file trong repo** — không nhét cả khối Terraform dài vào trình duyệt để copy.

### Luồng workshop

1. **Clone repository.** URL chính thức: **https://github.com/benguinsan/spendwise**. Thư mục gốc có **frontend/**, **backend/**, **infrastructure/**.

```bash
git clone https://github.com/benguinsan/spendwise.git
cd spendwise
```

2. **Cài công cụ trên máy:** **Terraform**, **AWS CLI**, **Git**. Cấu hình **quyền AWS** cho tài khoản lab **aws configure** hoặc biến môi trường.

3. **Mở code trong editor.** Toàn bộ Terraform cho môi trường mẫu nằm trong **infrastructure/** kể từ gốc repo. Root module dev là **environments/dev/main.tf**; phần tái sử dụng nằm trong **modules/**.

4. **Chuẩn bị biến.** Trong **infrastructure/environments/dev**, sao chép **terraform.tfvars.example** thành **terraform.tfvars** và điền giá trị. Token và secret không commit.

5. **Triển khai theo lớp** theo các mục dưới. Chạy **terraform init**, **plan**, **apply** trong **environments/dev**. Thực tế thường **một lần apply** cho cả stack; các mục numbered giải thích **module nào đang đóng vai trò gì**.

**Cách dùng trang này:** Coi như **checklist và sơ đồ đường đi**. Mở đúng đường dẫn trong bản clone — **không** copy từng trang Terraform từ web workshop.

### Vì sao dùng Terraform

- **Hạ tầng dưới dạng mã:** VPC, security group, RDS, ECS… nằm trong file có phiên bản.
- **Module:** **modules/vpc**, **modules/ecs**, **modules/rds**, **modules/amplify**, **modules/cognito**, … khớp phân lớp ở mục 4.2.
- **Thứ tự:** Terraform xử lý phụ thuộc để **mạng → compute → dữ liệu** nhất quán.

### Cấu trúc thư mục (infrastructure)

![Cây thư mục SpendWiseApp/infrastructure: docs, environments, modules (alb, amplify, bastion, cloudfront_alb, cognito, db_password_secret, ecr, ecs, monitoring, rds, security_groups, vpc, waf_alb), scripts](/images/4-workshop/4.3-spendwise-infrastructure-folder-tree.png)

### Nội dung

1. [4.3.1 Chạy hạ tầng với Terraform](4.3.1-run-infrastructure-terraform/)
2. [4.3.2 Hosting frontend](4.3.2-frontend-hosting/)
3. [4.3.3 Triển khai backend](4.3.3-backend/)
