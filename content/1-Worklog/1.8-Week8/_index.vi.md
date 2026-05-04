---
title: "Nhật ký Tuần 8"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu Tuần 8

- Nâng cao kiến thức về **Solutions Architecture** trên AWS.
- Tối ưu và hoàn thiện kiến trúc hệ thống SpendWiseApp.
- Nghiên cứu **AWS Best Practices** và **Well-Architected Framework**.



**Thời gian:** 27/04/2026 - 01/05/2026

---

### Tổng quan Nhiệm vụ Tuần

| Ngày | Hoạt động                                                                                                                                                                                                                                                                                | Ngày bắt đầu | Ngày kết thúc | Tài liệu tham khảo                           |
| ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ------------- | -------------------------------------------- |
| 1    | - Tích hợp **S3 & Redis** vào hệ thống  <br> + Upload ảnh hóa đơn (receipt) lên S3 bằng presigned URL <br> + Cache dữ liệu dashboard bằng Redis <br> + Tối ưu hiệu năng API (giảm truy vấn DB)                                                                     | 27/04/2026   | 27/04/2026    | <https://aws.amazon.com/s3/>               |
| 2    | - Phác thảo kiến trúc hệ thống hoàn chỉnh  <br> + Tham khảo kiến trúc e-commerce và financial apps  <br> + Xác định stack: <br> &emsp; · Frontend: Next.js (Amplify)  <br> &emsp; · Backend: NestJS (ECS Fargate) <br> &emsp; · DB: RDS PostgreSQL  <br> &emsp; · Cache: Redis <br> &emsp; · Auth: Cognito                           | 28/04/2026   | 28/04/2026    | <https://aws.amazon.com/architecture/>     |
| 3    | - Tối ưu dữ liệu & service  <br> + Chuẩn hóa database theo **3NF**  <br> + Tối ưu query transaction & dashboard <br> + Cải thiện logic budget **tracking & alert**                                             | 29/04/2026   | 29/04/2026    | -                   |
| 4    | - Hoàn thiện Proposal  <br> + Solution Architecture <br> + Implementation Plan  <br> + Cost Estimation <br> + Mô tả chi tiết các module: Transactions, budget, reports và authentication              | 30/04/2026   | 30/04/2026    | -                   |
| 5    | - Rà soát và tối ưu kiến trúc <br> + Đảm bảo **High Availability** <br> + Tối ưu chi phí **(EC2 vs Fargate vs Serverless)** <br> + Kiểm tra tính tương thích giữa các service AWS                       | 01/04/2026   | 01/04/2026    | <https://aws.amazon.com/whitepapers/>        |





### Thành tựu Tuần 8

- Advanced Solutions Architecture:

  - Thiết kế kiến trúc production-ready cho ứng dụng tài chính 
  - Hiểu mô hình: 
    - Microservices 
    - Event-driven (Lambda + EventBridge) 
    - Serverless 

- Tích hợp dịch vụ AWS nâng cao:

  - Sử dụng Lambda để xử lý budget alert 
  - EventBridge để scheduling 
  - SES để gửi email cảnh báo 

- AWS Well-Architected Framework:

  - Áp dụng 5 trụ cột: 
    - Security (Cognito, IAM, encryption) 
    - Reliability (Multi-AZ, backup) 
    - Performance (Redis cache) 
    - Cost Optimization (tối ưu resource) 
    - Operational Excellence (CI/CD, monitoring) 

- Tổng kết tuần 8:

  - Hoàn thiện kiến trúc hệ thống SpendWiseApp ở mức production-ready 
  - Kết hợp lý thuyết + thực hành AWS 
  - Sẵn sàng cho giai đoạn: 
    - Automation (IaC) 
    - DevOps 
    - Triển khai thực tế 
