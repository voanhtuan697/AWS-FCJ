---
title: "Monitoring & Operations"
date: 2025-09-10
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

#### Overview

A Production system cannot be considered complete without **Monitoring** and **Alerting** capabilities. You cannot sit watching the screen 24/7 to check if the Server is still alive

In this module, we will set up monitoring and alerting for MiniMarket using AWS operations management services:

- **Amazon CloudWatch:** Collect metrics (Metrics) from EC2, RDS, ELB
- **Amazon SNS (Simple Notification Service):** Notification service. We will use it to send Emails to administrators when the system encounters issues

We will set up a **CloudWatch Alarm** to monitor Web Server CPU. If CPU exceeds **70%** (sign of overload or attack), the system will automatically trigger SNS to send emergency alert Emails

![Monitoring Architecture](/images/5-Workshop/5.8-Monitoring/monitoring-diagram.png)

#### Content

1. [Set up CloudWatch Alarms & SNS](5.8.1-cloudwatch/)