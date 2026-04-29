---
title: "Nhật ký Tuần 5"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu Tuần 5

- Triển khai và quản lý container trên AWS với ECS, EKS và Docker.
- Nắm bắt các khái niệm và lợi ích của kiến trúc **Serverless**.
- Thiết kế ứng dụng **Serverless** tích hợp Lambda, API Gateway, S3 và DynamoDB.
- Tìm hiểu các dịch vụ **database AWS**: RDS, Aurora, DynamoDB, Redshift, ElastiCache.


**Thời gian:** 06/04/2026 - 10/04/2026

---

### Tổng quan Nhiệm vụ Tuần

| Ngày | Hoạt động                                                                                                                                                                                                                                               | Ngày bắt đầu | Ngày kết thúc | Tài liệu tham khảo                   |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ------------- | ------------------------------------ |
| 1    | - Tìm hiểu **Containers on AWS** <br> + Phân biệt giữa **ECS** và **EKS** <br> + Triển khai container mẫu bằng **ECS (Fargate)** <br> + Làm quen với **container orchestration**                                                           | 06/04/2026   | 06/04/2026    | <https://aws.amazon.com/containers/> |
| 2    | - Thực hành với **EKS (Elastic Kubernetes Service)** <br> + Tạo cụm EKS và triển khai ứng dụng mẫu <br> + Đẩy Docker image từ local lên ECR <br> + Theo dõi cụm EKS qua **AWS Console**                        | 07/04/2026   | 07/04/2026    | <https://aws.amazon.com/eks/>        |
| 3    | - Khám phá kiến trúc **Serverless Overview** <br> + Hiểu khái niệm **event-driven computing** <br> + Tạo và kiểm thử **AWS Lambda** <br> + Tích hợp **Lambda** với **API Gateway**                                                       | 08/04/2026   | 08/04/2026    | <https://aws.amazon.com/lambda/>     |
| 4    | - Thiết kế **Serverless Architecture** <br> + Kết nối **Lambda** với **DynamoDB** và **S3** <br> + Xây dựng pipeline dữ liệu đơn giản bằng **Step Functions** <br> + Kiểm thử quy trình serverless end-to-end                                           | 09/04/2026   | 09/04/2026    | <https://000025.awsstudygroup.com/>  |
| 5    | - Tìm hiểu về **Databases in AWS** <br> +So sánh **relational** và **NoSQL database** <br> +	Tạo database thử nghiệm trong **RDS** và **DynamoDB** <br> + Làm quen với **Redshift** choanalytics và **ElastiCache** để tăng tốc truy xuất | 10/04/2026   | 10/04/2026    | <https://000025.awsstudygroup.com/>  |


---

### Thành tựu Tuần 5

- Triển khai container bằng **ECS (Fargate)** và **EKS**; quản lý image với ECR.
- Xây dựng hàm **AWS Lambda** theo hướng **event-driven**; tích hợp với API Gateway tạo RESTful endpoint.
- Thiết kế ứng dụng **Serverless** hoàn chỉnh: Lambda + DynamoDB + S3 + Step Functions.
- Làm việc với RDS (relational), DynamoDB (NoSQL), Aurora, Redshift (analytics) và ElastiCache.
- Kết hợp Containers, Serverless và Databases trong cùng môi trường triển khai theo tư duy DevOps.

