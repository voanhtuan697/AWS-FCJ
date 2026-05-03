---
title : "Backend và nền tảng xử lý"
date: 2026-03-10
weight : 3
chapter : false
pre : " <b> 4.2.3 </b> "
---

## 4.2.3 Backend và nền tảng xử lý

Phần này mô tả **ingress API**, **runtime container** và **quan sát cơ bản** cho backend **SpendWiseApp** (NestJS sau load balancer). Chi tiết **CSDL** và secret xem [4.2.4 Cơ sở dữ liệu](4.2.4-database/):

![VPC backend SpendWiseApp: WAF → Internet Gateway → ALB; ECS Fargate trên hai AZ; RDS PostgreSQL; bastion và NAT tùy chọn; ECR/ECS qua PrivateLink và S3 qua gateway endpoint; CloudWatch log và alarm; Amplify và Cognito phía trên VPC (sơ đồ kiến trúc tổng)](/images/4-Workshop/4.2.3-backend-platform-vpc.png)

### WAF trên ALB

**Vai trò:** **WAFv2** Web ACL gắn trực tiếp **ARN của ALB** (module **waf_alb**, bật qua biến enable_waf).

**Lý do chọn:** Lọc request độc hại ở **biên API** trước ECS — bật/tắt theo môi trường (ví dụ tiết kiệm dev).

### ECR

**Vai trò:** **Elastic Container Registry** lưu trữ image backend để task ECS pull.

**Lý do chọn:** Registry được quản lý, kết hợp pull qua **VPC endpoint** thay vì tự host registry.

### ECS Fargate

**Vai trò:** **ECS service** chạy task trong **subnet app private** theo image đã **đăng ký (push)** lên ECR.

**Lý do chọn:** Không vận hành cluster EC2; scale theo tải; phù hợp API **stateless** sau ALB.

### CloudWatch (monitoring)

**Vai trò:** **CloudWatch Logs** cho stdout/stderr container ECS; khung metric/alarm.

**Lý do chọn:** Log tập trung và cảnh báo tối thiểu cho vận hành, không tự dựng stack log riêng.
