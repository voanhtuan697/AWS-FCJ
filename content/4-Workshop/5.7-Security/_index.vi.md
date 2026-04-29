---
title: "Tối ưu hóa & Bảo mật"
date: 2025-09-10
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Dọn dẹp tài nguyên

#### Tổng quan

Một hệ thống Production không chỉ cần "chạy được" mà còn phải "chạy nhanh" và "an toàn". Trong phần này, chúng ta sẽ tinh chỉnh kiến trúc MiniMarket

Các hạng mục thực hiện:
*   **Offloading Static Assets:** Chuyển toàn bộ hình ảnh sản phẩm từ Web Server sang **Amazon S3** và phân phối qua **Amazon CloudFront** (CDN) để tăng tốc độ tải trang toàn cầu và giảm tải cho server
*   **Security Hardening:** Triển khai **AWS WAF (Web Application Firewall)** chặn trước CloudFront để bảo vệ ứng dụng khỏi các cuộc tấn công phổ biến như SQL Injection và XSS

![Security Architecture](/images/5-Workshop/5.7-Security/security-diagram.png)

#### Nội dung

1. [Cấu hình S3 và CloudFront](5.7.1-s3-cloudfront/)
2. [Thiết lập AWS WAF](5.7.2-waf/)