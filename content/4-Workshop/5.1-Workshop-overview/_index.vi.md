---
title: "Giới thiệu"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Giới thiệu 

- MiniMarket là một ứng dụng thương mại điện tử được xây dựng trên nền tảng .NET Core, áp dụng kiến trúc 3-lớp (3-Tier Architecture) hiện đại. Mục tiêu của Workshop này là chuyển đổi ứng dụng từ môi trường On-premise lên hạ tầng đám mây AWS (Cloud Native Migration) đảm bảo các tiêu chí của AWS Well-Architected Framework: Bảo mật, Tin cậy, Hiệu năng cao và Tối ưu chi phí

#### Tổng quan về workshop

Kiến trúc giải pháp:

- Compute: Sử dụng AWS Elastic Beanstalk (nền tảng Docker) để đơn giản hóa việc triển khai, quản lý hạ tầng và Auto Scaling
- Database: Amazon RDS for SQL Server được triển khai trong Private Subnet để đảm bảo an toàn dữ liệu
- Caching: Amazon ElastiCache (Redis) giúp lưu trữ Session người dùng và giảm tải truy vấn cho Database, tăng tốc độ phản hồi
- Network & Security (Mạng & Bảo mật):
  - VPC: Thiết kế theo mô hình Public/Private Subnet kết hợp NAT Gateway
  - Bảo mật lớp ứng dụng: Sử dụng AWS WAF kết hợp Amazon CloudFront để chống tấn công Web và phân phối nội dung toàn cầu
- Storage: Amazon S3 dùng để lưu trữ và phục vụ các static assets (hình ảnh sản phẩm) với độ bền cao
- DevOps: Quy trình CI/CD tự động hóa hoàn toàn với AWS CodePipeline và CodeBuild
- Monitoring: Amazon CloudWatch để theo dõi sức khỏe hệ thống (CPU, Network) và gửi cảnh báo


![overview](/images/5-Workshop/5.1-Workshop-overview/project_architecture_3.1.png)
