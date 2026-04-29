---
title: "Workshop"
date: 2026-03-10
weight: 4
chapter: false
pre: " <b> 4. </b> "
---



# Triển khai ứng dụng Cloud-Native MiniMarket trên AWS

#### Tổng quan

Workshop này cung cấp hướng dẫn toàn diện về việc chuyển đổi (Re-platforming) ứng dụng thương mại điện tử **MiniMarket** (phát triển trên nền tảng .NET Core) từ môi trường cục bộ lên hạ tầng đám mây AWS theo kiến trúc **Cloud Native**.

Chúng ta sẽ không chỉ đơn thuần là thuê một máy chủ ảo (EC2) để chạy code. Thay vào đó, chúng ta sẽ xây dựng một hệ thống phân tán, có khả năng mở rộng cao (Scalable), bảo mật (Secure) và vận hành tự động (Automated) dựa trên các dịch vụ được quản lý (Managed Services).

Chúng ta sẽ thiết lập một kiến trúc Đa tầng (Multi-tier) bao gồm các thành phần cốt lõi:

- **Compute**: Sử dụng AWS Elastic Beanstalk kết hợp với Docker để đơn giản hóa việc triển khai và quản lý ứng dụng, hỗ trợ Auto Scaling tự động dựa trên lưu lượng truy cập.
- **Data & Caching**: Chuyển đổi từ SQL Server cục bộ sang Amazon RDS (đặt trong Private Subnet) để đảm bảo an toàn dữ liệu. Đồng thời, triển khai Amazon ElastiCache (Redis) để quản lý Session người dùng, đảm bảo hiệu năng cao cho ứng dụng.
- **Networking & Security**: Sử dụng **VPC** cùng với **Public/Private Subnet** và **NAT Gateway** cho kết nối ra ngoài an toàn, và bảo vệ ứng dụng trước các cuộc tấn công bằng **AWS WAF** kết hợp **CloudFront**.
- **DevOps**: Xây dựng quy trình **CI/CD** sử dụng **AWS CodePipeline** và **CodeBuild**, cho phép tự động hóa quy trình từ lúc commit code lên GitHub đến khi ứng dụng chạy trên môi trường Production
#### Nội dung

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Chuẩn bị](5.2-Prerequiste/)
3. [Thiết lập hạ tầng mạng (VPC, NAT, Security Groups)](5.3-Network/)
4. [Triển khai tầng dữ liệu (RDS & Redis)](5.4-Data/)
5. [Triển khai ứng dụng với Elastic Beanstalk & Docker](5.5-App/)
6. [Tự động hóa với CI/CD Pipeline](5.6-CICD/)
7. [Tối ưu hóa và Bảo mật(S3, CloudFront, WAF)](5.7-Security/)
8. [Giám sát (CloudWatch)](5.8-Monitoring/)
9. [Dọn dẹp tài nguyên](5.9-Cleanup/)