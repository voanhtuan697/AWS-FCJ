---
title: "Nhật ký Tuần 7"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Mục tiêu Tuần 7

- Hiểu và áp dụng các phương pháp bảo mật dữ liệu và hệ thống trong AWS cho **ứng dụng quản lý chi tiêu**. 
- Quản lý, cấu hình và tối ưu **Amazon VPC** cho hệ thống **backend–database–cache**. 
- Triển khai các giải pháp **Backup & Disaster Recovery (DR)** đảm bảo an toàn dữ liệu tài chính người dùng.


**Thời gian:** 20/04/2026 - 24/04/2026

---

### Tổng quan Nhiệm vụ Tuần

| Ngày | Hoạt động                                                                                                                                                                                                                                                                                     | Ngày bắt đầu | Ngày kết thúc | Tài liệu tham khảo                 |
| ---- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ------------- | ---------------------------------- |
| 1    | - Bắt đầu viết **Proposal** của đồ án PEM <br> + Hoàn thiện **Executive Summary**, **Problem Statement** (bài toán quản lý chi tiêu cá nhân) <br> + Mô tả kiến trúc tổng quan hệ thống (Frontend – Backend – Database – AWS Services) <br> + Nghiên cứu các dịch vụ AWS sẽ sử dụng: Cognito, ECS, RDS, S3, ElastiCache,...                                          | 20/04/2026   | 20/04/2026    | -                   |
| 2    | - Xây dựng kiến trúc backend theo mô hình **Domain-Driven Design** cho PEM <br> + Thiết kế entity: users, accounts, transactions, categories, budgets  <br> + Triển khai Repository & Unit of Work Pattern <br> + Chạy migration và seed dữ liệu PostgreSQL  <br> + Xây dựng service xử lý nghiệp vụ: <br> &emsp; · Quản lý giao dịch (income/expense/transfer)  <br> &emsp; · Quản lý tài khoản tài chính <br> &emsp; · Tính toán dashboard (thu/chi, số dư)                                  | 21/04/2026   | 21/04/2026    | -                   |
| 3    | - Khởi tạo frontend **WebApp (Next.js)** <br> + Xây dựng layout cơ bản: Header, Navigation, Dashboard  <br> + Xây dựng các trang chính:  <br> &emsp; · Danh sách giao dịch <br> &emsp; · Quản lý tài khoản <br> &emsp; · Dashboard thống kê <br> + Viết phần Architecture & Technical Implementation cho Proposal                                                         | 22/04/2026   | 22/04/2026    | -                   |
| 4    | - Thiết kế **AWS Architecture Diagram** cho hệ thống PEM <br> + Xác định các thành phần:  <br> &emsp; · ECS Fargate (Backend NestJS)  <br> &emsp; · RDS PostgreSQL  <br> &emsp; · S3 (lưu hóa đơn, file export)  <br> &emsp; · CloudFront + Amplify (Frontend) <br> &emsp; · Cognito (Authentication)  <br> &emsp; · ElastiCache Redis (Caching) <br> &emsp; · ALB, VPC, NAT Gateway <br> + Làm rõ luồng dữ liệu: User → Frontend → Backend → Database/Cache/S3 | 23/04/2026   | 23/04/2026    | -                   |
| 5    | - Tìm hiểu về **Disaster Recovery & Backup** <br> + Sử dụng AWS Backup để sao lưu RDS  <br> + Cấu hình Multi-AZ cho RDS PostgreSQL  <br> + Thực hành Cross-Region Failover <br> + Đảm bảo an toàn dữ liệu tài chính người dùng (transactions, budgets)                                                    | 24/04/2026   | 24/04/2026    | <https://aws.amazon.com/backup/>   |

---

### Thành tựu Tuần 7

- Bảo mật hệ thống:

  - Áp dụng mã hóa dữ liệu với KMS cho RDS và S3 
  - Sử dụng Cognito để quản lý xác thực JWT 
  - Cấu hình WAF & Shield chống tấn công 

- Làm chủ **mạng riêng ảo Amazon VPC**:

  - Thiết kế VPC với: 
    - Public subnet: ALB 
    - Private subnet: ECS, RDS, Redis 
  - Cấu hình Security Groups & NACLs 
  - Hiểu cơ chế NAT Gateway cho outbound traffic 

- Disaster Recovery:

  - Backup tự động với AWS Backup 
  - Multi-AZ giúp failover database 
  - Xây dựng chiến lược DR cho hệ thống tài chính 