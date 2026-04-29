---
title: "Set up Security Groups"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.4.1 </b> "
---

1.  Access **EC2** > **Security Groups** > **Create security group**

![Security1](/images/5-Workshop/5.4-Data/security1.png)

![Security2](/images/5-Workshop/5.4-Data/security2.png)

#### Group 1: Web Server (sg-web-app)
*   **Description:** Allow HTTP from Internet
*   **Inbound Rules:**
    *   Type: HTTP (80) | Source: 0.0.0.0/0 (Or only from Load Balancer if wanting stricter security)

![Security3](/images/5-Workshop/5.4-Data/security3.png)

#### Group 2: Database (sg-db-sql)
*   **Description:** Allow access only from Web Server
*   **Inbound Rules:**
    *   Type: MSSQL (1433) | Source: Custom > Select ID of **sg-web-app**

![Security4](/images/5-Workshop/5.4-Data/security4.png)

#### Group 3: Redis Cache (sg-redis-cache)
*   **Description:** Allow access only from Web Server
*   **Inbound Rules:**
    *   Type: Custom TCP (6379) | Source: Custom > Select ID of **sg-web-app**

![Security5](/images/5-Workshop/5.4-Data/security5.png)