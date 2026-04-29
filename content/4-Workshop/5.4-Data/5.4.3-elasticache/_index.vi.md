---
title: "Khởi tạo ElastiCache Redis"
date: 2025-09-10
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

1.  Truy cập **ElastiCache** > **Subnet groups** > **Create subnet group**
    *   Name: **redis-private-group**
    *   Subnets: Chọn 2 **Private Subnet**

![Elasti1](/images/5-Workshop/5.4-Data/Elasti1.png)

![Elasti2](/images/5-Workshop/5.4-Data/Elasti2.png)

2.  Vào **Redis OSS caches** > **Create cache**

![Elasti2.9](/images/5-Workshop/5.4-Data/Elasti2.9.png)

3.  Tại màn hình **Cluster settings**:
    *   **Engine:** Chọn **Redis OSS**
    *   **Deployment option:** Chọn **Node-based cluster**
    *   **Creation method:** Chọn **Cluster cache** (Configure and create a new cluster)
    *   **Cluster mode:** Chọn **Disabled** (Chế độ đơn giản, 1 Shard)

![Elasti3](/images/5-Workshop/5.4-Data/Elasti3.png)

4.  Tại màn hình **Location**:
    *   **Location:** AWS Cloud
    *   **Multi-AZ:** **Bỏ tích** (Enable) *Lưu ý: Tắt tính năng này để tiết kiệm chi phí cho môi trường Lab*
    *   **Auto-failover:** **Bỏ tích** (Enable)

5.  Tại màn hình **Cache settings**:
    *   **Engine version:** Để mặc định (VD: 7.1)
    *   **Port:** **6379**
    *   **Node type:** Chọn dòng **t3** > Chọn **cache.t3.micro**
    *   **Number of replicas:** Nhập **0** (Chúng ta chỉ cần 1 node chính, không cần node dự phòng)

![Elasti4](/images/5-Workshop/5.4-Data/Elasti4.png)

6.  Tại màn hình **Connectivity**:
    *   **Network type:** IPv4
    *   **Subnet groups:** Chọn **Choose existing subnet group** > Chọn **redis-private-group** vừa tạo

![Elasti5](/images/5-Workshop/5.4-Data/Elasti5.png)

7.  Tại màn hình **Advanced settings** (Quan trọng):
    *   **Encryption at rest:** Enable (Mặc định)
    *   **Encryption in transit:** **Bỏ tích (Disable)**
        *   *Lý do:* Tắt mã hóa đường truyền giúp đơn giản hóa việc kết nối từ code .NET trong môi trường nội bộ VPC mà không cần cấu hình chứng chỉ SSL phức tạp
    *   **Selected security groups:** Chọn **Manage** > chọn **sg-redis-cache** (Bỏ chọn default)

![Elasti6](/images/5-Workshop/5.4-Data/Elasti6.png)

8.  Kéo xuống cuối cùng và bấm **Create**

#### 3. Lấy thông tin kết nối
Quá trình khởi tạo sẽ mất khoảng 5-10 phút
1.  Khi trạng thái chuyển sang **Available** (Màu xanh)
2.  Bấm vào tên Cluster (**webapp** hoặc tên bạn đặt)
3.  Tại tab **Overview**, tìm mục **Primary endpoint**
4.  Copy chuỗi kết nối này (Ví dụ: **webapp.xxxx.cache.amazonaws.com**)

![Elasti7](/images/5-Workshop/5.4-Data/Elasti7.png)

{{% notice tip %}}
Endpoint này sẽ được dùng để cấu hình biến môi trường **ConnectionStrings__RedisConnection** cho Elastic Beanstalk ở các bước sau.
{{% /notice %}}