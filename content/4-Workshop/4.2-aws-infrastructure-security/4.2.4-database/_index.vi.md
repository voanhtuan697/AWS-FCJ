---
title : "Cơ sở dữ liệu"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 4.2.4 </b> "
---

## 4.2.4 Cơ sở dữ liệu

**Tầng dữ liệu** đặt **RDS PostgreSQL** trong **subnet private data**, tách khỏi subnet ứng dụng — traffic DB không ra Internet công cộng; mật khẩu qua **Secrets Manager**; **bastion** tùy chọn cho thao tác vận hành.

Có thể bố trí **RDS trên nhiều AZ** (ví dụ primary và bản **standby / backup** giữa các **private subnet** như sơ đồ) để tăng **độ sẵn sàng (HA)** và khả năng chịu lỗi. Trong **giai đoạn phát triển** thường **chỉ dùng một instance** để **tối ưu chi phí**; môi trường production có thể bật **Multi-AZ** / standby theo yêu cầu vận hành.

![RDS PostgreSQL và RDS backup trong hai private subnet (minh họa HA giữa các AZ)](/images/4-Workshop/4.2.4-database-private-rds.png)

### RDS PostgreSQL

**Vai trò:** CSDL quan hệ chính cho backend **NestJS** / **Prisma**; instance nằm trong **private data subnet**, chỉ lắng nghe Postgres nội bộ.

**Lý do chọn:** PostgreSQL phù hợp mô hình quan hệ + Prisma; RDS giảm vận hành patch/backup cơ bản so với tự cài Postgres trên EC2.

### RDS Security Group (nhóm bảo mật tầng dữ liệu)

**Vai trò:** Security Group gắn **RDS** (ví dụ tên **…-rds-sg** trong module **security_groups**): ingress **TCP 5432** chỉ từ **Security Group của task ECS**; có thể thêm luồng từ **bastion** khi bật truy cập DB qua bastion.

**Lý do chọn:** Không mở Postgres ra 0.0.0.0/0; chỉ cho phép lớp ứng dụng (và tùy chọn jump host) nói chuyện với DB — giảm bề mặt tấn công.

### Secrets Manager (mật khẩu DB)

**Vai trò:** Lưu **mật khẩu RDS** dạng secret, inject vào ứng dụng/Lambda qua IAM thay vì hard-code.

**Lý do chọn:** Không để mật khẩu trong repo hay biến môi trường thô; xoay vòng và kiểm soát truy cập theo chính sách AWS.

### Bastion (tùy chọn)

**Vai trò:** Máy **EC2** trong subnet public (hoặc tương đương) để admin kết nối DB qua SSM/SSH tùy cấu hình.

**Lý do chọn:** Khi cần truy vấn/troubleshoot DB trực tiếp mà không bật endpoint công khai cho RDS; có thể tắt trên dev để tiết kiệm.

### Bảng dữ liệu (PostgreSQL)

Các bảng trong hệ thống:

| Bảng (PostgreSQL) | Vai trò chính |
| :--- | :--- |
| **users** | Người dùng ứng dụng; liên kết **Cognito** (cognito_sub), email duy nhất. |
| **wallets** | Ví theo người dùng; số dư và **tiền tệ** (mặc định VND). |
| **categories** | Danh mục thu/chi/chuyển theo user; dùng cho giao dịch và ngân sách. |
| **transactions** | Giao dịch (thu/chi/chuyển), số tiền, ví nguồn/đích, danh mục, **date**; **note** tùy chọn. |
| **budgets** | Ngân sách theo **tháng/năm** và user (theo danh mục hoặc tổng). |
| **tags** | Nhãn do user định nghĩa; gắn nhiều-nhiều với giao dịch. |
| **transaction_tags** | Bảng nối **giao dịch ↔ tag**. |
| **goals** | Mục tiêu tiết kiệm (tên, mô tả tùy chọn, **target** / **current**, deadline; cờ **notification_sent**). |
| **recurring_transactions** | Giao dịch **định kỳ** (**interval**: ngày/tuần/tháng/năm), ví, danh mục tùy chọn, **next_date**, **is_active**. |
| **notifications** | Thông báo (**type**: budget alert / đạt goal / reminder), **message**, **is_read**. |
