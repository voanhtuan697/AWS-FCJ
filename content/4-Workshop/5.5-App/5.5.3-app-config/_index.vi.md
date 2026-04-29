---
title: "Cấu hình Biến môi trường"
date: 2025-09-10
weight: 3
chapter: false
pre: " <b> 5.5.3 </b> "
---

Để ứng dụng kết nối được với Database và Redis, chúng ta không hardcode trong code mà dùng Biến môi trường

1.  Vào Beanstalk Environment > **Configuration** > **Updates, monitoring, and logging** > **Edit**
2.  Kéo xuống mục **Environment properties**
3.  Thêm các biến sau:

    - **Name:** ConnectionStrings\_\_DefaultConnection

      - **Value:** Server=sql-shop-db....rds.amazonaws.com;Database=MiniMarketDB;User Id=admin;Password=PASSWORD BẠN ĐẶT;TrustServerCertificate=True;

    - **Name:** ConnectionStrings\_\_RedisConnection

      - **Value:** webapp.redis.cache...:6379

    - **Name:** VnPay\_\_IPNUrl

      - **Value:** https://[cloudfrontdomain].cloudfront.net/Payment/VnPayIPN

    - **Name:** VnPay\_\_ReturnUrl
      - **Value:** https://[cloudfrontdomain].cloudfront.net/Payment/VnPayReturn

![config1](/images/5-Workshop/5.5-App/config1.png)

4.  Bấm **Apply**. Server sẽ khởi động lại để nhận cấu hình mới
