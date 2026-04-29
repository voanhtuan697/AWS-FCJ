---
title: "Giám sát với CloudWatch"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.8.1 </b> "
---
1.  **Tạo SNS Topic:**
    *   Vào SNS > Topics > Create Topic 
        *   **Type:** Standard
        *   **Name:** DevOps-Alerts 

    ![SNS1](/images/5-Workshop/5.8-Monitoring/SNS1.png)

    ![SNS2](/images/5-Workshop/5.8-Monitoring/SNS2.png)

 
2.  **Tạo Subscription**
    *   Create Subscription > Protocol: Email > Nhập email của bạn (Nhớ Confirm mail)

    ![SNS3](/images/5-Workshop/5.8-Monitoring/SNS3.png)

    ![SNS4](/images/5-Workshop/5.8-Monitoring/SNS4.png)

    ![SNS5](/images/5-Workshop/5.8-Monitoring/SNS5.png)


3.  **Tạo Alarm CPU:**
    *   Vào CloudWatch > Alarms > Create alarm

    ![CW1](/images/5-Workshop/5.8-Monitoring/CW1.png)

    *   Select metric > EC2 > Per-Instance Metrics > Chọn InstanceID của Beanstalk > **CPUUtilization**

    ![CW2](/images/5-Workshop/5.8-Monitoring/CW2.png)

    ![CW3](/images/5-Workshop/5.8-Monitoring/CW3.png)

    ![CW4](/images/5-Workshop/5.8-Monitoring/CW4.png)

    *   Condition: **CPUUtilization:** Greater than **70%**

    ![CW5](/images/5-Workshop/5.8-Monitoring/CW5.png)

    *   Notification: Chọn Topic **DevOps-Alerts**

    ![CW6](/images/5-Workshop/5.8-Monitoring/CW6.png)

    *   Create Alarm.