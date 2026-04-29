---
title: "Cấu hình Route Table"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.3.3 </b> "
---

#### Tạo Route Table
Ấn vào phần **Route tables** trong VPC dashboard

![RT1](/images/5-Workshop/5.3-Network/RT1.png)

Tạo 2 Route Table, **Public** với **Private**

![RT2](/images/5-Workshop/5.3-Network/RT2.png)

![RT3](/images/5-Workshop/5.3-Network/RT3.png)

1. **Public Route Table:**
   - Đối với **Public Route Table**, trong phần **Routes** ấn vào **Edit routes**
   - Trỏ 0.0.0.0/0 về **Internet Gateway** 

![RT4](/images/5-Workshop/5.3-Network/RT4.png)

![RT5](/images/5-Workshop/5.3-Network/RT5.png)

   - Và trong phần **Subnet associations**, gán cho cả hai **Public Subnets**

![RT6](/images/5-Workshop/5.3-Network/RT6.png)

![RT7](/images/5-Workshop/5.3-Network/RT7.png)

2. **Private Route Table:**
   - Đối với **Public Route Table**, chúng ta sẽ trỏ 0.0.0.0/0 về **NAT Gateway** 

![RT8](/images/5-Workshop/5.3-Network/RT8.png)

   - Và trong phần **Subnet associations**, gán cho cả hai **Private Subnets**

![RT9](/images/5-Workshop/5.3-Network/RT9.png)

{{% notice tip %}}
Việc tách biệt Route Table đảm bảo rằng các Database trong Private Subnet không bao giờ bị lộ trực tiếp ra Internet.
{{% /notice %}}