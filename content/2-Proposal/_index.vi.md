---
title: "Bản đề xuất"
date: 2026-03-10
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Đề xuất triển khai SpendWiseApp trên AWS

## 1. Tổng quan dự án

Ứng dụng Quản lý Chi tiêu Cá nhân Spendwise là một web application hoàn chỉnh được xây dựng nhằm giải quyết bài toán thực tế: giúp người dùng theo dõi, phân tích và kiểm soát tình hình tài chính cá nhân một cách trực quan và hiệu quả. 

Ứng dụng được triển khai hoàn toàn trên môi trường AWS Cloud nhằm đảm bảo tính linh hoạt và an toàn dữ liệu.

## 2. Vấn đề và giải pháp

### Vấn đề hiện tại

Với một ứng dụng quản lý chi tiêu, dữ liệu người dùng và độ ổn định của hệ thống là yếu tố cần được chú trọng. Nếu triển khai theo mô hình máy chủ đơn lẻ hoặc hạ tầng tự quản rời rạc, hệ thống sẽ dễ gặp các tình huống:
- Rủi ro mất mát dữ liệu tài chính do backup thủ công, quy trình khôi phục không nhất quán.
- Theo hành vi người dùng lượng truy cập thường tăng đột biến vào đầu/cuối tháng khiến hệ thống phản hồi chậm, dễ nghẽn và gián đoạn.
- Việc nâng cấp phiên bản hoặc sửa lỗi phải thao tác trực tiếp trên máy chủ, dễ phát sinh sai sót cấu hình.
- Khó giám sát tập trung khi có sự cố, dẫn đến thời gian phát hiện và xử lý lỗi kéo dài.
- Khó bảo đảm mức an toàn dữ liệu khi cơ chế phân quyền truy cập và kiểm soát hạ tầng chưa được chuẩn hóa.

### Giải pháp

Để cải thiện phần nào vấn đề Spendwise lựa chọn tiếp cận theo kiến trúc Cloud thông qua AWS Cloud:
- Frontend hiện đại: Sử dụng framework NextJS được lưu trữ và triển khai thông qua dịch vụ AWS Amplify
- Backend: Sử dụng framework NestJS và triển khai thông qua AWS ECS Fargate
- Hệ thống mạng: Triển khai trong VPC, tách lớp truy cập public/private và điều phối lưu lượng qua Application Load Balancer.


## 3. Kiến trúc giải pháp

Kiến trúc đề xuất bám sát mô hình 3-tier ứng dụng trên môi trường cloud:
![Sơ đồ kiến trúc AWS SpendWise](/images/AWS_SpendWise.drawio.png)

**Các dịch vụ AWS**

| Dịch vụ | Vai trò trong SpendWise |
| :--- | :--- |
| **Amazon VPC** | Mạng riêng; chia subnet public/private trên nhiều AZ để tách lớp truy cập và dữ liệu. |
| **NAT Gateway** | Cho subnet private ra internet khi cần (ví dụ outbound không đi qua VPC endpoint). |
| **Security Group** | Tường lửa cho ALB, ECS, RDS, bastion và các VPC endpoint. |
| **Application Load Balancer** | Điểm vào từ internet; cân bằng tải và chuyển traffic tới backend trên ECS. |
| **Amazon ECS (Fargate)** | Chạy container backend NestJS; không quản lý EC2. |
| **Amazon ECR** | Lưu image backend để ECS kéo và triển khai. |
| **VPC Endpoint (Interface)** | Kết nối nội bộ tới ECR API, ECR DKR, CloudWatch Logs, Cognito IDP — task private subnet vận hành mà không cần NAT cho các luồng này. |
| **VPC Endpoint (Gateway — S3)** | Truy cập S3 nội bộ (ví dụ blob layer khi kéo image) qua gateway endpoint. |
| **Amazon Cognito** | User pool / app client; đăng ký, đăng nhập, xác nhận; có thể gắn Lambda PostConfirmation khi bật RDS. |
| **AWS Lambda** (Cognito, khi bật RDS) | PostConfirmation: ghi/cập nhật người dùng ứng dụng sau khi xác nhận đăng ký. |
| **Amazon RDS (PostgreSQL)** | CSDL quan hệ cho dữ liệu chi tiêu; đặt trong mạng private. |
| **AWS Amplify** | Build và host frontend Next.js từ Git; inject biến môi trường (ví dụ URL API) lúc build. |
| **Amazon CloudFront** | CDN phía trước ALB; tăng tốc phân phối. |
| **AWS WAF** | Lớp bảo vệ web trước các request độc hại phổ biến (SQLi/XSS/bot), giảm rủi ro tấn công vào ALB/API. |
| **AWS Secrets Manager (SM)** | Lưu trữ và quản lý an toàn secret key, token, mật khẩu DB. |
| **Amazon EC2 (Bastion)** | Máy nhảy vào VPC để vận hành DB (SSM / port forwarding)|
| **Amazon CloudWatch** | Log container ECS và nền tảng giám sát (module monitoring) cho vận hành, cảnh báo. |

## 4. Timeline (8 tuần)

- **Tuần 1–2 — Nền tảng & mạng**
  - **Tuần 1:** Chốt yêu cầu chức năng và NFR; khởi tạo Terraform, remote state và bố cục môi trường.
  - **Tuần 2:** Dựng VPC, phân tách public/private, security group và routing cơ bản; đối chiếu thiết kế mạng với kiến trúc mục tiêu.

- **Tuần 3–4 — Runtime ứng dụng trên AWS**
  - **Tuần 3:** Cấp ALB, ECR và ECS (Fargate); đóng gói backend NestJS; health check và rollout dịch vụ ban đầu.
  - **Tuần 4:** Tích hợp Cognito; kết nối frontend Amplify với API thực tế; smoke test luồng xác thực và API lõi end-to-end.

- **Tuần 5–6 — Dữ liệu & cứng hóa bảo mật**
  - **Tuần 5:** Bật RDS (PostgreSQL), Secrets Manager cho credential; chạy migration Prisma; xác nhận ECS ↔ RDS chỉ qua subnet private.
  - **Tuần 6:** Rà soát private networking và đường đi least-privilege (kể cả VPC endpoint khi áp dụng); HTTPS cho ALB (ACM) và tùy chọn lớp biên (ví dụ WAF / CloudFront).

- **Tuần 7–8 — Ổn định & bàn giao**
  - **Tuần 7:** Kiểm thử hiệu năng và smoke test; CloudWatch log/metric và cảnh báo nhẹ gắn ECS/ALB/RDS.
  - **Tuần 8:** Hoàn thiện tài liệu.
  
## 5. Ngân sách

Ước tính tham chiếu theo chi phí **1 tháng** (USD), dựa trên pricing hiện tại từ AWS Pricing và cấu hình dịch vụ trong SpendWise:

| Dịch vụ AWS | Thành phần / Sử dụng | Chi phí (USD/tháng) |
|---|---|---:|
| Elastic Load Balancing | Application Load Balancer (ALB + LCU) | $18 - $35 |
| Amazon ECS | Fargate (vCPU & Memory) | $9 - $25 |
| Amazon VPC | VPC Endpoints (Interface + Gateway) | $20 - $70 |
| Amazon VPC | NAT Gateway (nếu bật) | $0 hoặc $33 - $60 |
| Amazon RDS | PostgreSQL (Single-AZ nhỏ) | $12 - $35 |
| AWS Amplify | Frontend hosting + build | $5 - $20 |
| Amazon Cognito | MAU & auth flow | $0 - $10 |
| Amazon ECR | Image storage | $1 - $8 |
| Amazon CloudWatch | Logs / Metrics / Alarms | $3 - $15 |
| Amazon CloudFront | Distribution + data transfer (origin ALB) | $5 - $40 |
| AWS WAF | Web ACL + Rules + Requests (nếu bật) | $0 hoặc $8 - $25 |
| AWS Secrets Manager | Secrets lưu trữ + API calls | $1 - $6 |
| Amazon EC2 | Bastion (nếu bật) | $0 hoặc $5 - $12 |
| Terraform (IaC) | Xây dựng hạ tầng thông qua IaC | Không phí AWS trực tiếp |
| **TỔNG CHI PHÍ AWS** |  | **$74 - $361** |

Đề xuất kiểm soát chi phí:
- Giai đoạn đầu khi phát triển có thể tạm thời tắt Bastion nếu chưa có nhu cầu tương tác vào DB vào RDS.
- Chỉ duy trì 1 RDS duy nhất (Single-AZ, cấu hình nhỏ) để tránh phát sinh chi phí do nhân bản DB không cần thiết.
- Tối ưu retention log CloudWatch, tránh lưu log quá dài.
- Theo dõi sát chi phí NAT Gateway; cân nhắc endpoint/private pull strategy khi cần.

## 6. Rủi ro

- **Rủi ro chi phí NAT/ALB tăng nhanh** khi traffic tăng hoặc cấu hình chưa tối ưu.  
  *Giảm thiểu*: đặt budget alarm, review cost theo tuần, right-size tài nguyên.

- **Sai lệch cấu hình secret/env giữa frontend-backend-cognito** gây lỗi auth hoặc API.  
  *Giảm thiểu*: chuẩn hóa biến môi trường theo môi trường, checklist release bắt buộc.

- **Rủi ro vận hành DB** khi bật RDS (migration, connection pool, backup).  
  *Giảm thiểu*: migration có kiểm soát, test restore định kỳ, giám sát connection.

- **Gián đoạn triển khai do quy trình image/deploy chưa tự động hoàn toàn**.  
  *Giảm thiểu*: chuẩn hóa pipeline build/push/deploy, thêm smoke test sau release.

- **Rủi ro bảo mật hạ tầng** nếu mở inbound rộng hoặc để lộ credential trong tfvars.  
  *Giảm thiểu*: nguyên tắc least privilege, quản lý secret an toàn, review SG/IAM định kỳ.

