---
title: "Optimization & Security"
date: 2025-09-10
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Resource Cleanup

#### Overview

A Production system needs not only to "run" but also to "run fast" and be "secure". In this part, we will refine the MiniMarket architecture

Implementation items:
*   **Offloading Static Assets:** Transfer all product images from Web Server to **Amazon S3** and distribute via **Amazon CloudFront** (CDN) to accelerate global page load speed and offload the server
*   **Security Hardening:** Deploy **AWS WAF (Web Application Firewall)** in front of CloudFront to protect the application from common attacks such as SQL Injection and XSS

![Security Architecture](/images/5-Workshop/5.7-Security/security-diagram.png)

#### Content

1. [Configure S3 and CloudFront](5.7.1-s3-cloudfront/)
2. [Set up AWS WAF](5.7.2-waf/)