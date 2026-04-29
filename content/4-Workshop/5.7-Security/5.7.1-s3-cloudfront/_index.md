---
title: "Configure S3 & CloudFront"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.7.1 </b> "
---
**1. Create S3 Bucket:**
*   Create Bucket (Ex: **minimarket-assets-prod**)
*   Block Public Access: **On**


![s31](/images/5-Workshop/5.7-Security/s31.png)

![s32](/images/5-Workshop/5.7-Security/s32.png)

*   Manually upload **images** folder from code to this Bucket

![s33](/images/5-Workshop/5.7-Security/s33.png)

**2. Create CloudFront Distribution:**
*   **Origin type:** Select Elastic Load Balancer
*   **Origin Domain:** Select Load Balancer of Beanstalk

![LB1](/images/5-Workshop/5.7-Security/LB1.png)
![LB2](/images/5-Workshop/5.7-Security/LB2.png)

*   **Settings:** Select Customize origin settings
    *   **Protocol:** HTTP Only

![LB3](/images/5-Workshop/5.7-Security/LB3.png)

*   **Cache settings:** Select Customize cache settings
    *   **Viewer Protocol Policy:** Redirect HTTP to HTTPS

![LB4](/images/5-Workshop/5.7-Security/LB4.png)

**3. Add S3 Origin (To retrieve images):**
*   Go to newly created **Distribution**

![LB5](/images/5-Workshop/5.7-Security/LB5.png)

*   Go to **Origins** tab > Create Origin

![LB6](/images/5-Workshop/5.7-Security/LB6.png)

*   **Origin domain** select the S3 created earlier (**minimarket-assets-prod**)

![LB7](/images/5-Workshop/5.7-Security/LB7.png)

*   **Origin Access:** Select **Origin access control (OAC)** > Create new OAC

![LB8](/images/5-Workshop/5.7-Security/LB8.png)

![LB9](/images/5-Workshop/5.7-Security/LB9.png)

*   **Bucket Policy:** Copy policy provided by CloudFront and paste into S3 Bucket policy

![LB10](/images/5-Workshop/5.7-Security/LB10.png)

![LB11](/images/5-Workshop/5.7-Security/LB11.png)

![LB12](/images/5-Workshop/5.7-Security/LB12.png)

![LB13](/images/5-Workshop/5.7-Security/LB13.png)


**4. Configure Behavior:**
*   Return to **CloudFront** go to **Behaviors** tab

![LB14](/images/5-Workshop/5.7-Security/LB14.png)

*   Create Behavior with Path pattern: **/images/**
*   Point to Origin S3

![LB15](/images/5-Workshop/5.7-Security/LB15.png)

*   Cache Policy: **CachingOptimized**

![LB16](/images/5-Workshop/5.7-Security/LB16.png)