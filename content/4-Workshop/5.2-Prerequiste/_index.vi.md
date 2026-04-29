---
title: "Các bước chuẩn bị"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### IAM permissions

- Gắn quyền AdministratorAccess trong IAM permission policy vào tài khoản AWS để dễ dàng làm việc hơn
    - Lưu ý: Việc sử dụng quyền Administrator chỉ được khuyến nghị cho môi trường Workshop để đảm bảo quá trình triển khai không bị gián đoạn. Trong môi trường Production thực tế, cần tuân thủ nguyên tắc Least Privilege cho từng dịch vụ

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```

#### Source Code

- Repository GitHub chứa code .NET Core và Dockerfile hợp lệ

![repository](/images/5-Workshop/5.2-Prerequisite/source.png)

