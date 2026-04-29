---
title: "Triển khai Tầng Dữ liệu"
date: 2025-09-10
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Tổng quan

Dữ liệu là tài sản quan trọng nhất của mọi hệ thống. Vậy nên chúng ta sẽ thiết lập tầng dữ liệu (Data Layer) cho MiniMarket với tiêu chí: **Bảo mật tối đa** và **Hiệu năng cao**

Chúng ta sẽ triển khai hai dịch vụ cốt lõi:
1.  **Amazon RDS (Relational Database Service):** Sử dụng SQL Server để lưu trữ dữ liệu nghiệp vụ (Sản phẩm, Đơn hàng, Người dùng). Database sẽ được đặt trong **Private Subnet** để ngăn chặn truy cập trực tiếp từ Internet
2.  **Amazon ElastiCache (Redis):** Sử dụng Redis làm bộ nhớ đệm (In-memory Cache) để lưu trữ Session đăng nhập và giảm tải truy vấn cho Database chính

![Data Layer Architecture](/images/5-Workshop/5.4-Data/data-diagram.png)

#### Nội dung

1. [Thiết lập Security Groups cho DB & Cache](5.4.1-security-groups/)
2. [Khởi tạo Amazon RDS (SQL Server)](5.4.2-rds/)
3. [Khởi tạo Amazon ElastiCache (Redis)](5.4.3-elasticache/)

