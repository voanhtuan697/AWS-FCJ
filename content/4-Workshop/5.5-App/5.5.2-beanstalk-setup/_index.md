---
title: "Initialize Elastic Beanstalk"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.5.2 </b> "
---

We will create an environment to run the application

1.  Access **Elastic Beanstalk** > **Create application**

![EB1](/images/5-Workshop/5.5-App/EB1.png)

- **App Name:** MiniMarket-App
- **Platform:** Docker (Amazon Linux 2023)
- **Application code:** Select **Sample application** (To test infrastructure first)

![EB2](/images/5-Workshop/5.5-App/EB2.png)

![EB3](/images/5-Workshop/5.5-App/EB6.png)

2.  **Network Configuration (Networking) - Extremely Important:**

- **VPC:** Select the VPC you created for MiniMarket
- **Instance settings:**
  - Public IP address: **Uncheck**
  - Subnets: Select 2 **Private Subnets**
  - EC2 security groups: Select **sg-web-app**

![EB4](/images/5-Workshop/5.5-App/EB3.png)

![EB5](/images/5-Workshop/5.5-App/EBSG.png)

- **Capacity:**
  - Environment type: Select Load balanced

![EB6](/images/5-Workshop/5.5-App/EB4.png)

- **Load balancer network settings:**
  - Visibility: **Public**
  - Subnets: Select 2 **Public Subnets**

![EB7](/images/5-Workshop/5.5-App/EB5.png)

3.  Click **Create**. The system will take about 5-7 minutes to initialize