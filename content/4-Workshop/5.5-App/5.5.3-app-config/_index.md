---
title: "Configure Environment Variables"
date: 2025-09-10
weight: 3
chapter: false
pre: " <b> 5.5.3 </b> "
---

To enable the application to connect to Database and Redis, we do not hardcode in the code but use Environment Variables

1.  Go to Beanstalk Environment > **Configuration** > **Updates, monitoring, and logging** > **Edit**
2.  Scroll down to **Environment properties** section
3.  Add the following variables:

    - **Name:** ConnectionStrings\_\_DefaultConnection

      - **Value:** Server=sql-shop-db....rds.amazonaws.com;Database=MiniMarketDB;User Id=admin;Password=PASSWORD YOU SET;TrustServerCertificate=True;

    - **Name:** ConnectionStrings\_\_RedisConnection

      - **Value:** webapp.redis.cache...:6379

    - **Name:** VnPay\_\_IPNUrl

      - **Value:** https://[cloudfrontdomain].cloudfront.net/Payment/VnPayIPN

    - **Name:** VnPay\_\_ReturnUrl
      - **Value:** https://[cloudfrontdomain].cloudfront.net/Payment/VnPayReturn

![config1](/images/5-Workshop/5.5-App/config1.png)

4.  Click **Apply**. Server will restart to apply new configuration