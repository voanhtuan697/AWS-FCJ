---
title: "Khởi tạo Elastic Beanstalk"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.5.2 </b> "
---

Chúng ta sẽ tạo môi trường chạy ứng dụng

1.  Truy cập **Elastic Beanstalk** > **Create application**

![EB1](/images/5-Workshop/5.5-App/EB1.png)

- **App Name:** MiniMarket-App
- **Platform:** Docker (Amazon Linux 2023)
- **Application code:** Chọn **Sample application** (Để test hạ tầng trước)

![EB2](/images/5-Workshop/5.5-App/EB2.png)

![EB3](/images/5-Workshop/5.5-App/EB6.png)

2.  **Cấu hình Mạng (Networking) - Cực kỳ quan trọng:**

- **VPC:** Chọn VPC mà bạn đã tạo cho MiniMarket
- **Instance settings:**
  - Public IP address: **Bỏ tích**
  - Subnets: Tích chọn 2 **Private Subnet**
  - EC2 security groups: Chọn **sg-web-app**

![EB4](/images/5-Workshop/5.5-App/EB3.png)

![EB5](/images/5-Workshop/5.5-App/EBSG.png)

- **Capacity:**
  - Environment type: Chọn Load balanced

![EB6](/images/5-Workshop/5.5-App/EB4.png)

- **Load balancer network settings:**
  - Visibility: **Public**
  - Subnets: Tích chọn 2 **Public Subnet**

![EB7](/images/5-Workshop/5.5-App/EB5.png)

3.  Bấm **Create**. Hệ thống sẽ mất khoảng 5-7 phút để khởi tạo
