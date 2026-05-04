---
title: "Frontend hosting"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 4.3.2 </b> "
aliases:
  - /4-workshop/4.3-deployment-operations-monitoring/4.3.2-frontend-hosting-auth/
---

## 4.3.2 Frontend hosting

### AWS Amplify Console sau khi terraform apply

Sau khi **terraform apply** thành công, mở **AWS Amplify** đúng **Region** đã triển khai (ví dụ **US East (N. Virginia)**, mã vùng **us-east-1**). Trang **All apps** liệt kê ứng dụng Terraform vừa tạo; giao diện có thể như sau:

![Trang All apps — Amplify sau khi apply thành công](/images/4-Workshop/amplify-console-after-terraform-apply.png)

Sau khi bấm vào ứng dụng, màn hình **Overview** hiển thị nhánh **main**; truy cập nhánh này để thực hiện deploy.

![Amplify — Overview, nhánh main](/images/4-Workshop/amplify-console-overview-main-branch.png)

### Đặc tả build: amplify.yml

Amplify Hosting không tự suy ra bố cục monorepo. **Đặc tả build** ở **thư mục gốc repository** — file **amplify.yml** trong [SpendWiseApp](https://github.com/benguinsan/spendwise) — cho Amplify biết thư mục ứng dụng Next.js, cách cài dependency, lệnh build và thư mục artifact để publish.

```yaml
# SpendWiseApp/amplify.yml (repository root)
version: 1
applications:
  - appRoot: frontend
    frontend:
      phases:
        preBuild:
          commands:
            - npm install --legacy-peer-deps
        build:
          commands:
            - npm run build
      artifacts:
        baseDirectory: .next
        files:
          - '**/*'
      cache:
        paths:
          - node_modules/**/*
          - .next/cache/**/*
```

| Block | Vai trò |
| :--- | :--- |
| **appRoot: frontend** | Ứng dụng Next.js nằm trong **frontend/**; Amplify chạy các lệnh build từ thư mục đó. |
| **preBuild → npm install --legacy-peer-deps** | Cài dependency trước khi biên dịch; **--legacy-peer-deps** tránh lỗi phân giải peer dependency quá chặt, thường gặp trong workspace Next.js hỗn hợp. |
| **build → npm run build** | Sinh bản build production của Next.js (gồm cây **.next** mà Amplify phục vụ theo kiểu hosting này). |
| **artifacts.baseDirectory: .next** | Khai báo thư mục gốc artifact cho bản build đã sinh (khớp cách ứng dụng này được build cho Amplify). |
| **cache.paths** | Tăng tốc build lặp lại nhờ tái sử dụng **node_modules** và **.next/cache** giữa các lần build. |

**Biến môi trường lúc build:** Giá trị như **NEXT_PUBLIC_API_URL** do Terraform đưa vào môi trường app Amplify, nhưng được **đóng gói vào bundle phía client tại thời điểm build**. Sau khi đổi biến URL API trong Terraform hoặc trên console Amplify, hãy kích hoạt **triển khai mới** (build lại) để trình duyệt tải bundle khớp URL backend. Hành vi build được định nghĩa trong **amplify.yml**; giá trị biến đến từ Terraform / cấu hình app Amplify.

### Vì sao dùng CloudFront cho API (mixed content)

Frontend trên Amplify được phục vụ cho trình duyệt qua **HTTPS**. Trình duyệt áp dụng quy tắc **mixed content**: trang tải bằng HTTPS không được tải subresource hay gọi API qua **HTTP** thuần theo kiểu được coi là *active* content (ví dụ **fetch** / XHR tới API **http://**). Nếu SPA gọi API NestJS qua **HTTP** với hostname ALB thuần, trình duyệt sẽ chặn hoặc cảnh báo, ứng dụng có thể hỏng hoặc hành vi khác nhau giữa các trình duyệt.

**Các lựa chọn trong stack này:**

1. **HTTPS trên ALB với hostname do bạn quản lý** — Dùng **alb_acm_certificate_arn** (ACM **cùng region** với ALB), xác thực chứng chỉ trong DNS, trỏ hostname về ALB, đặt **alb_public_api_base_url**. Đây là hướng lâu dài đúng khi bạn có domain; với bài lab ngắn sẽ nhiều thành phần hơn.

2. **CloudFront phía trước ALB (enable_api_cloudfront = true)** — Terraform tạo phân phối **Amazon CloudFront** với **chứng chỉ mặc định của CloudFront**, nên API truy cập được qua URL **https://dxxxx.cloudfront.net** mà **không cần mua domain** hay gắn ACM lên ALB. Trình duyệt thấy **HTTPS → HTTPS**, nên **hết mixed content**. Tùy cách các module nối origin, ALB có thể vẫn chỉ HTTP phía sau CloudFront trong khi URL API **hướng tới người dùng** luôn là HTTPS.

**Vì sao CloudFront hợp workshop và môi trường dev:** mang lại **URL HTTPS ổn định** nhanh, **khớp với Amplify** khi frontend chỉ được phục vụ qua HTTPS, và tránh trì hoãn xác thực DNS của ACM khi bạn chỉ cần lab chạy được. Đánh đổi là thêm một chặng mạng và cần nắm cấu hình origin CloudFront–ALB; với production khi đã có domain riêng, thường ưu tiên ACM + ALB hoặc thiết lập CloudFront tùy chỉnh — xem [4.2.2](../../4.2-aws-infrastructure-security/4.2.2-client-facing/) và [4.2.3](../../4.2-aws-infrastructure-security/4.2.3-backend-platform/).

Thiết kế liên quan (Amplify, API URL): [4.2.2 Frontend hosting và xác thực người dùng](../../4.2-aws-infrastructure-security/4.2.2-client-facing/).

---
