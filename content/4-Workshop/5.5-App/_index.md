---
title: "Application Deployment"
date: 2025-09-10
weight: 5
chapter: false
pre: " <b> 5.5 </b> "
---

#### Overview

After having network and data infrastructure, the next step is to bring .NET Core application source code to Cloud. Instead of managing each EC2 virtual server manually, we will use the **Platform-as-a-Service (PaaS)** platform which is **AWS Elastic Beanstalk**

Goals of this module:
*   **Containerization:** Package MiniMarket application into **Docker Container** to ensure uniform running environment (Dev = Prod)
*   **Deployment:** Deploy Container to Elastic Beanstalk. The system will automatically provision EC2, configure Load Balancer and Auto Scaling Group
*   **Connectivity:** Configure so application connects securely to RDS and Redis via Environment Variables

![App Deployment Architecture](/images/5-Workshop/5.5-App/beanstalk-diagram.png)

#### Content

1. [Package application with Docker](5.5.1-dockerize/)
2. [Initialize Elastic Beanstalk Environment](5.5.2-beanstalk-setup/)
3. [Configure Database & Redis connection](5.5.3-app-config/)