---
title: "Initialize Amazon RDS"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.4.2 </b> "
---

1.  Access **RDS Console** > **Subnet groups** > **Create DB subnet group**
    *   Name: **db-private-group**
    *   Subnets: Select 2 AZs and select exactly 2 **Private Subnets**

![db1](/images/5-Workshop/5.4-Data/db1.png)
![db3](/images/5-Workshop/5.4-Data/db3.png)

2.  Go to **Databases** > **Create database**

![db4](/images/5-Workshop/5.4-Data/db4.png)

3.  **Engine options:** Microsoft SQL Server (Express Edition)


![db5](/images/5-Workshop/5.4-Data/db5.png)

4.  **Templates:** Free tier
5.  **Settings:** Set Master Password (remember for later use)

![db6](/images/5-Workshop/5.4-Data/db6.png)

6.  **Connectivity:**
    *   VPC: **VPC you created for Web**
    *   Subnet group: **db-private-group**
    *   Public access: **No**
    *   VPC security group: Select **Security group you created for database**

![db7](/images/5-Workshop/5.4-Data/db7.png)

7.  Click **Create database**