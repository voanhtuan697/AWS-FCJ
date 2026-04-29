---
title: "Khởi tạo Amazon RDS"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

1.  Truy cập **RDS Console** > **Subnet groups** > **Create DB subnet group**
    *   Name: **db-private-group**.
    *   Subnets: Chọn 2 AZ và chọn đúng 2 **Private Subnet**

![db1](/images/5-Workshop/5.4-Data/db1.png)
![db3](/images/5-Workshop/5.4-Data/db3.png)

2.  Vào **Databases** > **Create database**

![db4](/images/5-Workshop/5.4-Data/db4.png)

3.  **Engine options:** Microsoft SQL Server (Express Edition)


![db5](/images/5-Workshop/5.4-Data/db5.png)

4.  **Templates:** Free tier.
5.  **Settings:** Đặt mật khẩu Master Password (ghi nhớ để dùng sau này)

![db6](/images/5-Workshop/5.4-Data/db6.png)

6.  **Connectivity:**
    *   VPC: **VPC mà bạn đã tạo cho Web**
    *   Subnet group: **db-private-group**
    *   Public access: **No**
    *   VPC security group: Chọn **Security group mà bạn tạo cho database**

![db7](/images/5-Workshop/5.4-Data/db7.png)

7.  Bấm **Create database**