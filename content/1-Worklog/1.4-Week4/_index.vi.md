---
title: "Nhật ký Tuần 4"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu Tuần 4

- Tìm hiểu và tối ưu lưu trữ trên **Amazon S3**.
- Nâng cao kỹ năng quản lý, bảo mật và tối ưu hiệu suất S3.
- Tìm hiểu **CloudFront**, **Global Accelerator** và các phương pháp giảm độ trễ phân phối nội dung.
- Nghiên cứu **FSx**, **Storage Gateway** và tích hợp hệ thống qua **SQS**, **SNS**, **Step Functions**.


**Thời gian:** 30/03/2026 - 03/04/2026

---

### Tổng quan Nhiệm vụ Tuần

| Ngày | Hoạt động                                                                                                                                                                                                                                                | Ngày bắt đầu | Ngày kết thúc | Tài liệu tham khảo                            |
| ---- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ------------- | --------------------------------------------- |
| 1    | - Tìm hiểu **Amazon S3 cơ bản** <br> + Tạo bucket và tải dữ liệu mẫu <br> + So sánh các loại **storage tiers**: Standard, Infrequent Access, Glacier <br> +	Phân tích chi phí theo từng lớp lưu trữ                                      | 30/03/2026   | 30/03/2026    | <https://000024.awsstudygroup.com/>           |
| 2    | - Nâng cao quản lý **Amazon S3** <br> + Cấu hình **Cross-Region Replication (CRR)** giữa hai vùng <br> + Thiết lập **Lifecycle Policy** tự động chuyển dữ liệu sang Glacier <br> + Bật **Versioning** và kiểm tra khả năng khôi phục dữ liệu | 31/03/2026   | 31/03/2026    | <https://000024.awsstudygroup.com/>           |
| 3    | -**bảo mật dữ liệu S3** <br> + Áp dụng **IAM policy** và **bucket policy** kiểm soát truy cập <br> + Cấu hình **Server-Side Encryption (SSE)** và **AWS KMS** <br> + Kiểm tra cài đặt **Block Public Access**     | 01/04/2026   | 01/04/2026    | <https://000057.awsstudygroup.com/>           |
| 4    | - Khám phá **CloudFront & Global Accelerator** <br> + Thiết lập **CloudFront Distribution** phân phối nội dung từ S3 <br> + Tích hợp **Global Accelerator** để giảm độ trễ toàn cầu <br> +	Đánh giá hiệu suất qua công cụ đo tốc độ truy cập  | 02/04/2026   | 02/04/2026    | <https://000057.awsstudygroup.com/>           |
| 5    | - Tìm hiểu **AWS Storage Extras** <br> + Tìm hiểu **AWS Storage Gateway** và **hybrid storage** <br> + Thử nghiệm **FSx for Windows** và **FSx for Lustre** <br> + So sánh hiệu năng giữa các giải pháp lưu trữ                          | 03/04/2026   | 03/04/2026    | <https://www.youtube.com/watch?v=_yunukwcAwc> |

---

### Thành tựu Tuần 4

- Hiểu rõ **Amazon S3**: các storage class, chi phí, hiệu suất và độ bền theo từng lớp.
- Thực hành quản lý **S3 nâng cao**: Cross-Region Replication, Lifecycle Policy, Versioning.
- Củng cố bảo mật S3: IAM, Bucket Policy, mã hóa SSE/KMS, Block Public Access.
- Thiết lập **CloudFront** và **Global Accelerator**, kiểm tra độ trễ và trải nghiệm người dùng.
- Tìm hiểu **Storage Gateway (hybrid)**, **FSx for Windows** và **FSx for Lustre** cho workload hiệu năng cao.
- Hiểu cơ chế **Integration & Messaging**: SQS, SNS, Step Functions trên AWS.

