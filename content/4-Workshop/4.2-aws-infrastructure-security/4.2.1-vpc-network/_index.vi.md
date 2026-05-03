---
title : "VPC và mạng"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 4.2.1 </b> "
---

## 4.2.1 VPC và mạng

Thiết lập nền tảng mạng AWS phù hợp mô hình **3-tier** cho **SpendWiseApp**, tách lớp public/private và tăng tính sẵn sàng trên **hai Availability Zone**. 

| Thành phần | Vai trò |
| :--- | :--- |
| **VPC** | VPC riêng (bật DNS hostnames / DNS support); **6 subnet** — mỗi AZ một subnet **public**, một **private app** (tầng ứng dụng / ECS), một **private data** (tầng dữ liệu / RDS); phân bổ **CIDR con** rõ theo từng tầng. |
| **Internet Gateway** | Điểm nối giữa VPC và **Internet công cộng**; subnet **public** dùng route tới IGW để truy cập Internet (ví dụ ALB nhận traffic từ người dùng). |
| **NAT Gateway** | Cho phép tài nguyên trong subnet **private** gửi traffic **ra Internet (outbound)** mà không gán public IP trực tiếp cho từng EC2/task; inbound từ Internet **không** đến thẳng vào private như với máy public. |
| **VPC endpoint** | Đường đi **riêng trong mạng AWS** tới dịch vụ AWS (kiểu Interface hoặc Gateway); traffic **không phải đi ra Internet công cộng** để tới một số API AWS — giảm phơi bày và thường kết hợp với việc hạn chế hoặc tắt NAT cho các luồng đó. |

### VPC và subnets

VPC (Virtual Private Cloud) là mạng riêng ảo của bạn trên AWS. Hầu hết các tài nguyên sẽ nằm trong VPC. Sau khi khởi tạo VPC thành công ta sẽ triển khai các subnets:

| Nhóm subnet | Số subnet | Mục đích |
| :--- | :---: | :--- |
| **Public** | 2 | Một subnet public trên mỗi AZ dùng cho tài nguyên tiếp nhận hoặc thoát Internet — ví dụ **Application Load Balancer**, và đặt **NAT Gateway** khi bật outbound cho biên private. |
| **Private (ứng dụng)** | 2 | Trên mỗi AZ một subnet **private app**: đặt task **ECS Fargate** chạy backend; không quảng bá địa chỉ public trực tiếp cho người dùng cuối — luồng ingress người dùng đi qua ALB ở lớp public. |
| **Private (dữ liệu)** | 2 | Trên mỗi AZ một subnet **private data**: chứa **RDS PostgreSQL**; chỉ cho phép kết nối nội bộ (ví dụ Postgres) theo Security Group, không lộ máy chủ DB ra Internet công cộng. |

### Security Group (ứng dụng và dữ liệu)

**Security Group** là tường lửa **stateful** gắn với **ENI** trong VPC: chỉ định nghĩa luật **cho phép** (mặc định còn lại là từ chối), thường tham chiếu **CIDR** hoặc **Security Group** khác.

Các Security Group trong hệ thống:

| Security Group | Vai trò |
| :--- | :--- |
| ALB Security Group | Gắn **ALB**. Inbound: **HTTP (80)** và **HTTPS (443)** từ **Internet** (mọi nguồn — **0.0.0.0/0**). |
| ECS Security Group | Gắn **task ECS Fargate**. Chỉ cho phép traffic đến **từ ALB** (theo ALB Security Group). |
| Bastion Security Group | Gắn **máy bastion** (khi bật). |
| RDS Security Group | Gắn **RDS PostgreSQL**. Inbound: **5432** từ ECS Security Group của task; có thể thêm từ bastion SG khi bật truy cập DB qua bastion. |

### VPC endpoint

**VPC endpoint** cho phép lưu lượng tới API của một số dịch vụ AWS **đi trong mạng nội bộ của AWS**, không phải **ra Internet công cộng** rồi quay lại — giảm phơi bày và thường giúp giảm phụ thuộc NAT cho các luồng đó. Hai dạng liên quan tới triển khai này:

- **Gateway endpoint** — dùng cho **Amazon S3**. Gắn vào **bảng định tuyến (route table)**: tiền tố đích được đẩy thẳng tới S3, **không** tạo thêm ENI trong subnet.
- **Interface endpoint** — tạo **ENI** trong subnet và cung cấp **DNS riêng** cho API regional. Ở đây dùng cho **ECR** (API và DKR), **CloudWatch Logs**, **Cognito IDP**, v.v. Các Interface endpoint dùng chung một **Security Group** tên **vpc_endpoints**, cho **HTTPS (TCP 443)** từ CIDR VPC. Gateway **S3** gắn các route table phù hợp public/private (nhánh phụ thuộc **enable_nat_gateway**).

| Resource | Loại | Gắn subnet / route table | Ý nghĩa vận hành |
| :--- | :--- | :--- | :--- |
| aws_security_group.vpc_endpoints | — | Áp cho VPC | Ingress **TCP 443** từ vpc_cidr tới các endpoint Interface. |
| aws_vpc_endpoint.ecr_api | Interface | private_app | **ECR API** — xác thực và token khi pull image. |
| aws_vpc_endpoint.ecr_dkr | Interface | private_app | **ECR DKR** — API registry Docker (manifest/metadata). |
| aws_vpc_endpoint.s3 | Gateway | Danh sách route table (public + private; nhánh phụ thuộc enable_nat_gateway) | **S3** — gateway, phục vụ blob/layer qua prefix list. |
| aws_vpc_endpoint.logs | Interface | private_app | **CloudWatch Logs** — đẩy log container (driver **awslogs**) không bắt buộc đi NAT. |
| aws_vpc_endpoint.cognito_idp | Interface | Chỉ các subnet **private_app** nằm trong AZ mà dịch vụ Cognito IDP endpoint hỗ trợ | **Cognito IDP** — gọi API nhận diện trong biên VPC. |

