---
title : "Backend và nền tảng xử lý"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 4.2.3 </b> "
---

## 4.2.3 Backend và nền tảng xử lý

Phần này trình bày **đường vào API** và **runtime backend NestJS** trong **SpendWiseApp/infrastructure**: Terraform gốc nằm ở **environments/dev/main.tf** (và **environments/prod/main.tf** cùng bố cục module); từng khối nằm dưới **modules/**.

![SpendWiseApp backend VPC: CloudFront → WAF (tùy chọn) → Internet Gateway → ALB; ECS Fargate trên hai AZ; RDS PostgreSQL (primary và backup); bastion và NAT tùy chọn; ECR/ECS qua PrivateLink và S3 qua gateway VPC endpoint; CloudWatch log và cảnh báo](/images/4-Workshop/4.2.3-spendwise-backend-vpc.png)

**Nền tảng mạng:** Module **vpc** và **security_groups** cung cấp subnet (public / private app / private data) và security group cho ALB, ECS, RDS, bastion — chi tiết trong [4.2.1](../4.2.1-vpc-network/).

### Application Load Balancer

**Vai trò:** **ALB** đặt trên subnet **public**, listener HTTP/HTTPS (HTTPS khi **alb_acm_certificate_arn** được cấu hình) và target group trỏ tới port container backend. Đây là điểm vào duy nhất từ Internet tới dịch vụ container.

**Lý do chọn:** Cân bằng tải lớp 7, health check và tích hợp gọn với auto scaling ECS theo **ALBRequestCountPerTarget** — rất hợp với HTTP API NestJS sau reverse proxy.

### Amazon CloudFront (API — tùy chọn)

**Vai trò:** Khi **enable_api_cloudfront** là true, module **cloudfront_api** đặt CloudFront phía trước ALB; API có thể gọi qua URL HTTPS dạng **https://dxxx.cloudfront.net** (chứng chỉ mặc định của CloudFront), trình duyệt gọi API qua HTTPS mà không bắt buộc ACM/custom domain trên ALB — giảm vấn đề mixed content khi SPA Amplify chạy HTTPS.

**Lý do chọn:** Có HTTPS nhanh cho URL API trong môi trường lab/dev; **frontend_api_url** trong **main.tf** chọn URL này hoặc URL chỉ ALB (**frontend_api_url_alb_only**) để **NEXT_PUBLIC_API_URL** khớp cách client gọi API.

### AWS WAF (tùy chọn)

**Vai trò:** Khi **enable_waf** là true, **waf_alb** gắn Web ACL WAFv2 lên **ARN ALB**, lọc request theo cấu hình rule của module.

**Lý do chọn:** Thêm lớp bảo vệ trước ứng dụng; có thể bật metric CloudWatch qua **waf_enable_cloudwatch_metrics**.

### Amazon Elastic Container Registry

**Vai trò:** **ECR** lưu image backend NestJS; task ECS kéo image từ trong VPC.

**Lý do chọn:** Registry riêng trong tài khoản, kiểm soát IAM và pull nội bộ — không phụ thuộc registry công khai cho image production.

### Amazon ECS trên Fargate

**Vai trò:** Module **ecs** chạy cluster **Fargate** trên subnet **private_app**, **depends_on** ALB để listener có trước; đăng ký task với target group ALB; **locals.ecs_backend_environment** gồm **DATABASE_URL** khi RDS bật, biến từ **terraform.tfvars**, **COGNITO_***, **CORS_ORIGIN** khớp domain Amplify. Auto scaling theo request trên từng target.

**Lý do chọn:** Không cần quản lý hạ tầng EC2 cho worker; phù hợp API stateless và scale theo tải ALB.

### Amazon RDS và AWS Secrets Manager

**Vai trò:** **RDS** tạo PostgreSQL trong subnet **private_data** khi **create_rds** là true. **db_password_secret** lưu mật khẩu DB trong Secrets Manager khi dùng RDS (recovery window theo biến). Lambda **PostConfirmation** của Cognito ghi user vào CSDL khi RDS bật — xem thêm luồng SPA/auth tại [4.2.2](../4.2.2-client-facing/).

**Lý do chọn:** Cơ sở dữ liệu và secret được quản lý, tách khỏi **terraform.tfvars** thuần theo hướng sẵn sàng cho production.

### Amazon CloudWatch

**Vai trò:** **monitoring** tạo log group; tên log group truyền vào **ecs** để container ghi stdout/stderr vào đó.

**Lý do chọn:** Log tập trung và lộ trình bổ sung metric/cảnh báo cho vận hành backend và xử lý sự cố.

### Bastion (tùy chọn)

**Vai trò:** Khi **create_bastion** là true, **bastion** tạo instance EC2 trong subnet public, truy cập qua **SSM** — thường dùng port forwarding tới RDS khi migration hoặc gỡ lỗi.

**Lý do chọn:** Postgres không hở ra Internet công cộng nhưng vẫn truy cập được qua kênh kiểm soát.

Lệnh vận hành và output: [4.3.3 Triển khai backend](../../4.3-deployment-operations-monitoring/4.3.3-backend/).
