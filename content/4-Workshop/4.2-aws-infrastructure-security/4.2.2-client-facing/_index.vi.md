---
title : "Frontend Hosting và xác thực người dùng"
date: 2026-03-10
weight : 2
chapter : false
pre : " <b> 4.2.2 </b> "
---

## 4.2.2 Frontend Hosting và xác thực người dùng

Phần này giới thiệu **AWS Amplify** cho hosting frontend và **Amazon Cognito** cho xác thực người dùng:

![Luồng GitHub → Amplify hosting; Route 53 (domain) → Amplify; Amplify → Cognito (trích từ sơ đồ kiến trúc tổng)](/images/4-Workshop/4.2.2-client-facing-edge.png)

### Amplify

**Vai trò:** Build và phân phối **Next.js** từ repository Git; cấu hình biến môi trường build (ví dụ NEXT_PUBLIC_API_URL) để trình duyệt gọi đúng API backend.

**Lý do chọn:** CI/CD và HTTPS được quản lý sẵn, giảm gánh tự vận hành máy chủ web tĩnh/CDN; phù hợp monorepo khi chỉ rõ thư mục app frontend (app root). Với **Next.js**, việc lựa chọn mô hình **S3 + CloudFront** kiểu hosting tĩnh thuần thường **rườm rà hơn** vì framework có thể dùng **SSR** (render phía server).

### Cognito

**Vai trò:** **User Pool** và **App client** phục vụ đăng ký, đăng nhập và cấp token; có thể gắn **Lambda trigger** (ví dụ **PostConfirmation** đồng bộ user vào RDS).

**Lý do chọn:** Dịch vụ xác thực do AWS vận hành, JWT chuẩn, tích hợp frontend và Lambda — tránh phải tự dựng và bảo trì máy chủ xác thực riêng. Hiệu quả mạnh mẽ và linh hoạt trong việc Scale.
