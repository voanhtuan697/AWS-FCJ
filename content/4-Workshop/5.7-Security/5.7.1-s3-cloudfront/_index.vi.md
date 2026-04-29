---
title: "Cấu hình S3 & CloudFront"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.7.1 </b> "
---
**1. Tạo S3 Bucket:**
*   Tạo Bucket (VD: **minimarket-assets-prod**)
*   Block Public Access: **On**


![s31](/images/5-Workshop/5.7-Security/s31.png)

![s32](/images/5-Workshop/5.7-Security/s32.png)

*   Upload thủ công thư mục **images** từ code lên Bucket này

![s33](/images/5-Workshop/5.7-Security/s33.png)

**2. Tạo CloudFront Distribution:**
*   **Origin type:** Chọn Elastic Load Balancer
*   **Origin Domain:** Chọn Load Balancer của Beanstalk

![LB1](/images/5-Workshop/5.7-Security/LB1.png)
![LB2](/images/5-Workshop/5.7-Security/LB2.png)

*   **Settings:** Chọn Customize origin settings
    *   **Protocol:** HTTP Only

![LB3](/images/5-Workshop/5.7-Security/LB3.png)

*   **Cache settings:** Chọn Customize cache settings
    *   **Viewer Protocol Policy:** Redirect HTTP to HTTPS

![LB4](/images/5-Workshop/5.7-Security/LB4.png)

**3. Thêm Origin S3 (Để lấy ảnh):**
*   Vào **Distribution** mới tạo

![LB5](/images/5-Workshop/5.7-Security/LB5.png)

*   Vào tab **Origins** > Create Origin

![LB6](/images/5-Workshop/5.7-Security/LB6.png)

*   **Origin domain** chọn cái S3 đã được tạo lúc nãy (**minimarket-assets-prod**)

![LB7](/images/5-Workshop/5.7-Security/LB7.png)

*   **Origin Access:** Chọn **Origin access control (OAC)** > Create new OAC

![LB8](/images/5-Workshop/5.7-Security/LB8.png)

![LB9](/images/5-Workshop/5.7-Security/LB9.png)

*   **Bucket Policy:** Copy policy do CloudFront cung cấp và dán vào S3 Bucket policy

![LB10](/images/5-Workshop/5.7-Security/LB10.png)

![LB11](/images/5-Workshop/5.7-Security/LB11.png)

![LB12](/images/5-Workshop/5.7-Security/LB12.png)

![LB13](/images/5-Workshop/5.7-Security/LB13.png)


**4. Cấu hình Behavior:**
*   Quay lại **CloudFront** vào tab **Behaviors**

![LB14](/images/5-Workshop/5.7-Security/LB14.png)

*   Tạo Behavior với Path pattern: **/images/**
*   Trỏ về Origin S3

![LB15](/images/5-Workshop/5.7-Security/LB15.png)

*   Cache Policy: **CachingOptimized**

![LB16](/images/5-Workshop/5.7-Security/LB16.png)
