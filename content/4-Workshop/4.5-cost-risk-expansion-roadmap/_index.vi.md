---
title : "Chi phí, rủi ro và định hướng mở rộng"
date : 2024-01-01
weight : 5
chapter : false
pre : " <b> 4.5. </b> "
---

## 4.5 Chi phí, rủi ro và định hướng mở rộng

### Tối ưu chi phí
- Theo dõi chi phí theo nhóm dịch vụ chính: ALB, ECS Fargate, VPC endpoint/NAT, RDS, Amplify, Cognito, CloudWatch.
- Ưu tiên cấu hình tiết kiệm cho dev:
  - Tắt custom domain khi chưa cần.
  - Duy trì 1 RDS nhỏ (Single-AZ).
  - Tắt bastion khi không thao tác DB.
  - Tối ưu log retention và theo dõi chi phí NAT.

### Rủi ro và giảm thiểu
- Rủi ro tăng chi phí khi traffic tăng hoặc cấu hình sai.
- Rủi ro lộ secret và sai lệch cấu hình môi trường.
- Rủi ro migration/backup dữ liệu và gián đoạn deploy.
- Hướng giảm thiểu: checklist release, giám sát chủ động, least privilege và quản lý secret an toàn.

### Lộ trình mở rộng
- Chuẩn hóa CI/CD đầy đủ.
- Bổ sung test tự động và smoke test.
- Tăng cường lớp bảo mật (WAF, secret rotation, policy hardening).
- Nâng cấp kiến trúc khi cần production-ready.
