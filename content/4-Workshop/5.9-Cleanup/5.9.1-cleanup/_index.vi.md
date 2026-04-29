---
title: "Dọn dẹp tài nguyên"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.9.1 </b> "
---

Để tránh phát sinh chi phí không mong muốn sau khi hoàn thành Workshop, hãy xóa tài nguyên theo đúng thứ tự sau:

1.  **NAT Gateway:** Delete NAT Gateway > Đợi Deleted > **Release Elastic IP** (Quan trọng nhất vì tốn tiền nhất)

![DEL1](/images/5-Workshop/5.9-Cleanup/DEL1.png)

![DEL2](/images/5-Workshop/5.9-Cleanup/DEL2.png)

2.  **Elastic Beanstalk:** Terminate Environment

![DEL3](/images/5-Workshop/5.9-Cleanup/DEL3.png)

3.  **ElastiCache:** Delete Redis Cluster (Bỏ chọn Create Backup)

![DEL4](/images/5-Workshop/5.9-Cleanup/DEL4.png)

4.  **RDS:** Stop (hoặc Delete nếu không dùng nữa - nhớ bỏ chọn Final Snapshot).

![DEL5](/images/5-Workshop/5.9-Cleanup/DEL5.png)

5.  **WAF:** Manage resources > Disassociate >  Delete protection pack (web ACL)

![DEL7](/images/5-Workshop/5.9-Cleanup/DEL7.png)

![DEL6](/images/5-Workshop/5.9-Cleanup/DEL6.png)

6.  **S3:** Empty và Delete Bucket (Có thể không xóa nếu còn sử dụng vì chi phí không tốn quá nhiều)

7.  **CloudFront** Disable và Delete