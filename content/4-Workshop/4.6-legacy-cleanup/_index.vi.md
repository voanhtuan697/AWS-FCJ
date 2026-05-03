---
title : "Dọn dẹp tài nguyên"
date : 2024-01-01
weight : 99
chapter : false
pre : " <b> 4.6. </b> "
draft : false
---

## 4.6 Dọn dẹp tài nguyên

Khi không còn cần stack workshop trên AWS, gỡ tài nguyên do Terraform tạo bằng **terraform destroy**. Lệnh này lập kế hoạch rồi hủy mọi thứ còn được **state** quản lý cho **môi trường** bạn chạy - **có hủy dữ liệu** (ví dụ **RDS** và secret **Secrets Manager** do stack quản lý).

### Chạy ở đâu

Trong repo **SpendWiseApp**, dùng **cùng thư mục environment** đã dùng khi **apply** (xem [4.3.1 Chạy hạ tầng với Terraform](../4.3-deployment-operations-monitoring/4.3.1-run-infrastructure-terraform/)):

```bash
cd infrastructure/environments/dev   # hoặc environments/prod
terraform destroy
```

Terraform in kế hoạch hủy; gõ **yes** khi được hỏi, trừ khi bạn **cố ý** dùng **-auto-approve** (chỉ nên dùng trong tự động hóa - tránh nhầm tài khoản dùng chung hoặc production).

### Trước khi destroy

- Kiểm tra **credential AWS** và **region** đúng tài khoản mục tiêu (profile / **AWS_PROFILE**, **AWS_REGION**).
- Nếu dùng **remote backend**, chắc chắn đang destroy đúng **workspace/state** (dev so với prod).
- Ứng dụng sẽ **ngừng hoạt động**: ALB, ECS, tài nguyên gắn Amplify, User Pool Cognito trong stack, v.v. bị gỡ theo thứ tự phụ thuộc - không cần hủy từng dịch vụ tay nếu toàn bộ nằm trong root module này.

### Sau khi destroy

- **State** nên không còn resource được quản lý (hoặc chỉ còn data source) cho root đó; nếu destroy dở, xử lý lỗi Terraform báo rồi chạy lại **terraform destroy** cho đến khi xong.
- **Tài nguyên tạo ngoài Terraform** hoặc object còn sót (S3/ECR thủ công, policy đặc biệt) Terraform **không** xóa - dọn ở console hoặc CLI nếu chính sách tài khoản yêu cầu.

Nhắc ngắn trong luồng apply/plan: cùng lệnh nằm tại [4.3.1](../4.3-deployment-operations-monitoring/4.3.1-run-infrastructure-terraform/).
