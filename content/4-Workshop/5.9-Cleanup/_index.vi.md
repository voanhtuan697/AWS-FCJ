---
title: "Dọn dẹp tài nguyên"
date: 2025-09-10
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Tổng quan

Chúc mừng bạn đã thành công triển khai MiniMarket trên AWS!

Tuy nhiên, công việc của chúng ta chưa kết thúc ở đó. Bước cuối cùng và cũng là bước quan trọng nhất để bảo vệ "ví tiền" của bạn là **Dọn dẹp tài nguyên**

Các dịch vụ chúng ta đã triển khai như **NAT Gateway, Elastic Load Balancer, RDS, ElastiCache** đều tính phí theo giờ, dù bạn có sử dụng hay không. Nếu quên xóa, hóa đơn cuối tháng có thể rất cao

Chúng ta sẽ đi qua quy trình **Decommissioning** hệ thống theo đúng trình tự để đảm bảo không còn tài nguyên nào bị sót lại gây phát sinh chi phí ngầm

#### Nội dung

1. [Quy trình xóa tài nguyên an toàn](5.9.1-cleanup/)
