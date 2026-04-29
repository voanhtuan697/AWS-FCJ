---
title: "Nhật ký Tuần 3"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu Tuần 3

- Nắm bắt các khái niệm cốt lõi về **High Availability** và **Scalability** trong môi trường cloud.
- Tìm hiểu các dịch vụ database được quản lý: **RDS**, **Aurora**, **ElastiCache**.
- Cấu hình và quản lý **Amazon Route 53** cho DNS và điều hướng lưu lượng.
- Nghiên cứu các mô hình kiến trúc điển hình trên AWS và thực hành thiết kế hệ thống.


**Thời gian:** 23/10/2026 - 27/10/2026

---

### Tổng quan Nhiệm vụ Tuần

| Ngày | Hoạt động                                                                                                                                                                                                                                                         | Ngày bắt đầu | Ngày kết thúc | Tài liệu tham khảo                     |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ------------- | -------------------------------------- |
| 1    | - Nghiên cứu **High Availability & Scalability** <br> + Tìm hiểu mô hình **Multi-AZ** <br> + Cấu hình thử nghiệm **Auto Scaling Group** <br> + Nắm vai trò **Elastic Load Balancer (ELB)** trong việc phân phối lưu lượng                      | 23/03/2026   | 23/03/2026    | <https://000010.awsstudygroup.com/>    |
| 2    | - Tìm hiểu **Amazon RDS** và **Aurora** <br> + Khởi tạo instance RDS ở chế độ **Multi-AZ** <br> + Thử nghiệm **read replica** và cơ chế **failover** <br> + Tìm hiểu **Aurora cluster** và khả năng tự phục hồi                                                          | 24/03/2026   | 24/03/2026    | <https://000010.awsstudygroup.com/>    |
| 3    | - Làm việc với **Amazon ElastiCache** <br> + So sánh hai engine **Redis** và **Memcached** <br> + Thực hành tạo cache cluster <br> + Giám sát hiệu suất qua **CloudWatch**                                                                                        | 25/03/2026   | 25/03/2026    | <https://000019.awsstudygroup.com/>    |
| 4    | - Cấu hình **Amazon Route 53** <br> + Tạo **hosted zone** và bản ghi DNS <br> + Thiết lập **failover routing** <br> + Kiểm tra **latency-based routing** giữa các region                                                                                  | 26/03/2026   | 26/03/2026    | <https://aws.amazon.com/route53/>      |
| 5  | - Nghiên cứu **Classic Solutions Architecture** <br> + Phân tích mô hình **multi-tier** và **serverless** <br> + Nghiên cứu các pattern thiết kế có tính mở rộng cao <br> + Thiết kế mô hình thử nghiệm kết hợp **EC2**, **RDS**, và **ELB** | 27/03/2026   | 27/03/2026    | <https://aws.amazon.com/architecture/> |

---

### Thành tựu Tuần 3

- Nắm vững chiến lược **High Availability**: Multi-AZ, failover tự động, Auto Scaling Group, ELB.
- Mở rộng kiến thức database AWS: quản lý RDS, hiểu lợi thế **Aurora**, dùng **ElastiCache** tăng tốc.
- Làm chủ DNS và định tuyến qua **Route 53**: hosted zone, routing policy, health check, failover routing.
- Tăng cường tư duy thiết kế hệ thống: kết hợp **Compute**, **Database** và **Networking** để xây dựng kiến trúc linh hoạt, tối ưu chi phí.