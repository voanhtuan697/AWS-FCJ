---
title: "Thiết lập Security Groups"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

1.  Truy cập **EC2** > **Security Groups** > **Create security group**

![Security1](/images/5-Workshop/5.4-Data/security1.png)

![Security2](/images/5-Workshop/5.4-Data/security2.png)

#### Nhóm 1: Web Server (sg-web-app)
*   **Description:** Cho phép HTTP từ Internet
*   **Inbound Rules:**
    *   Type: HTTP (80) | Source: 0.0.0.0/0 (Hoặc chỉ từ Load Balancer nếu muốn chặt chẽ hơn)

![Security3](/images/5-Workshop/5.4-Data/security3.png)

#### Nhóm 2: Database (sg-db-sql)
*   **Description:** Chỉ cho phép truy cập từ Web Server
*   **Inbound Rules:**
    *   Type: MSSQL (1433) | Source: Custom > Chọn ID của **sg-web-app**

![Security4](/images/5-Workshop/5.4-Data/security4.png)

#### Nhóm 3: Redis Cache (sg-redis-cache)
*   **Description:** Chỉ cho phép truy cập từ Web Server
*   **Inbound Rules:**
    *   Type: Custom TCP (6379) | Source: Custom > Chọn ID của **sg-web-app**

![Security5](/images/5-Workshop/5.4-Data/security5.png)