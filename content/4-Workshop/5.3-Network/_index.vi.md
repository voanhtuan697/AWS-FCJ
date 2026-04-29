---
title: "Thiết lập Hạ tầng Mạng"
date: 2025-09-10
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Tổng quan

Trong phần này, chúng ta sẽ xây dựng nền móng mạng lưới cho ứng dụng MiniMarket. Một kiến trúc mạng an toàn là yếu tố tiên quyết để bảo vệ ứng dụng và dữ liệu

Chúng ta sẽ thiết kế một **VPC** gồm có:
- **Public Subnet:** Dành cho các thành phần giao tiếp trực tiếp với Internet (Load Balancer, NAT Gateway).
- **Private Subnet** Dành cho các thành phần cần bảo mật (App Server, Database, Redis)

Ngoài ra, chúng ta sẽ cấu hình **NAT Gateway** để cho phép các máy chủ ở bên trong Private Subnet có thể tải được các bản cập nhật và Docker Image từ Internet mà không bị lộ địa chỉ IP ra ngoài

![VPC Architecture](/images/5-Workshop/5.3-Network/vpc-diagram.png)

#### Nội dung

- [Khởi tạo VPC & Subnet](5.3.1-create-vpc/)
- [Cấu hình Internet & NAT Gateway](5.3.2-gateways/)
- [Cấu hình Route Table](5.3.3-routing/)
