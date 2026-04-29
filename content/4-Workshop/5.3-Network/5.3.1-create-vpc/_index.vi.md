---
title: "Khởi tạo VPC & Subnets"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

1. Mở [Amazon VPC console](https://ap-southeast-1.console.aws.amazon.com/vpcconsole/home?region=ap-southeast-1#Home:) (Lưu ý: Hãy chọn region phù hợp với nhu cầu, ở đây nhóm sử dụng Region ap-southeast-1)
2. Chọn **Create VPC**

![VPC Tutorial 1](/images/5-Workshop/5.3-Network/vpc-tutorial-1.png)

3. Cấu hình:
- Tên: **MiniMarket-VPC** 
- IPv4 CIDR: **10.0.0.0/16**

![VPC Tutorial 2](/images/5-Workshop/5.3-Network/vpc-tutorial-2.png)

4. Tạo Subnets (Chia 2 AZ để đảm bảo High Availability):
- Public Subnets (2): **10.0.1.0/24 & 10.0.2.0/24** (Dùng cho Load Balancer & NAT)
- Private Subnets (2): **10.0.3.0/24 & 10.0.4.0/24** (Dùng cho App, DB, Redis)

![VPC Tutorial 3](/images/5-Workshop/5.3-Network/vpc-tutorial-3.png)

5. Nhấn **Create VPC** và đợi cho state chuyển sang **Available** là thành công

![VPC Tutorial 4](/images/5-Workshop/5.3-Network/vpc-tutorial-4.png)