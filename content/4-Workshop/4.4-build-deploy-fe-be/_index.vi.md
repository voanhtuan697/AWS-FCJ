---
title : "Triển khai frontend và backend trên AWS"
date: 2026-03-10
weight : 4
chapter : false
pre : " <b> 4.4. </b> "
---

## 4.4 Triển khai frontend và backend trên AWS

Mục này mô tả **luồng thực tế** sau khi Terraform đã tạo Amplify, ECR, ECS và (nếu bật) RDS — tức cách **đẩy code frontend/backend** lên môi trường đang chạy. Kiến trúc và biến Terraform tương ứng nằm ở [4.3 Triển khai Terraform (theo lớp)](../4.3-deployment-operations-monitoring/).

### Frontend (Amplify hosting)

- Cần **URL repository Git** (thường là GitHub) và **access token** (Personal Access Token hoặc token tương đương) để Amplify **clone repo** và chạy build — hai giá trị này truyền vào Terraform qua biến kiểu **amplify_repository_url** và **amplify_access_token** (đặt trong terraform.tfvars hoặc biến môi trường CI, **không** commit token lên git).
- Nhánh build (ví dụ **main**) khớp **amplify_branch_name**; sau khi push code, Amplify build lại ứng dụng Next.js (monorepo thư mục **frontend**) và phục vụ hosting CDN của Amplify.
- URL API mà frontend gọi (**NEXT_PUBLIC_API_URL**) đã được Terraform gắn với ALB; đảm bảo backend và DNS/TLS đã sẵn sàng trước khi kỳ vọng giao diện gọi API thành công.

**Lệnh CLI minh họa** (cần **AWS CLI**, **Terraform**, **Docker**; chỉnh **TF_DIR** và đường dẫn **docker build** theo chỗ bạn clone repo — ví dụ nếu shell đang ở **trong** SpendWiseApp thì dùng **TF_DIR=infrastructure/environments/dev** và **-f backend/Dockerfile** với context **backend**).

```bash
# Ví dụ: đứng ở thư mục cha chứa thư mục SpendWiseApp (vd. workspace awslab)
export TF_DIR=SpendWiseApp/infrastructure/environments/dev

# Xem app Amplify, nhánh, domain mặc định (sau terraform apply)
terraform -chdir=$TF_DIR output amplify_app_id
terraform -chdir=$TF_DIR output amplify_branch_name
terraform -chdir=$TF_DIR output amplify_default_domain
```

Sau khi **git push** lên nhánh đã cấu hình, Amplify thường tự chạy build (tùy auto build). Nếu cần **ép một bản build mới** từ CLI:

```bash
export AWS_REGION="$(terraform -chdir=$TF_DIR output -raw aws_region)"
export AMPLIFY_APP_ID="$(terraform -chdir=$TF_DIR output -raw amplify_app_id)"
export AMPLIFY_BRANCH="$(terraform -chdir=$TF_DIR output -raw amplify_branch_name)"

aws amplify start-job \
  --region "$AWS_REGION" \
  --app-id "$AMPLIFY_APP_ID" \
  --branch-name "$AMPLIFY_BRANCH" \
  --job-type RELEASE
```

Theo dõi job trong console Amplify hoặc: **aws amplify get-job --app-id … --branch-name … --job-id …**.

### Backend (Docker → ECR → ECS)

- **Build** image backend (NestJS) phù hợp Dockerfile trong repo, **tag** trùng với giá trị dùng trong Terraform (ví dụ **latest** hoặc tag theo commit).
- **Đăng nhập** ECR (aws ecr get-login-password), **docker push** image lên repository mà Terraform đã tạo (output **ecr_repository_url** trong môi trường dev).
- **Triển khai phiên bản mới** trên ECS: cập nhật service để kéo image mới — ví dụ **aws ecs update-service --cluster … --service … --force-new-deployment** (hoặc đổi revision task definition nếu pipeline của bạn quản lý theo bản ghi). Đợi task mới **running** và healthy qua target group của ALB.

**Lệnh CLI minh họa** (cùng quy ước **TF_DIR** như trên):

```bash
export TF_DIR=SpendWiseApp/infrastructure/environments/dev
export AWS_REGION="$(terraform -chdir=$TF_DIR output -raw aws_region)"
export ECR_REPO="$(terraform -chdir=$TF_DIR output -raw ecr_repository_url)"
# Registry ECR là phần trước dấu / của URL repo (vd. 123456789012.dkr.ecr.ap-southeast-1.amazonaws.com)
export ECR_REGISTRY="${ECR_REPO%%/*}"
export IMAGE_TAG=latest   # phải khớp ecs_backend_image_tag trong Terraform

aws ecr get-login-password --region "$AWS_REGION" \
  | docker login --username AWS --password-stdin "$ECR_REGISTRY"

docker build -f SpendWiseApp/backend/Dockerfile -t spendwise-backend:"$IMAGE_TAG" SpendWiseApp/backend
docker tag spendwise-backend:"$IMAGE_TAG" "${ECR_REPO}:${IMAGE_TAG}"
docker push "${ECR_REPO}:${IMAGE_TAG}"
```

Cập nhật dịch vụ ECS để kéo image mới và đợi ổn định:

```bash
export ECS_CLUSTER="$(terraform -chdir=$TF_DIR output -raw ecs_cluster_name)"
export ECS_SERVICE="$(terraform -chdir=$TF_DIR output -raw ecs_service_name)"

aws ecs update-service \
  --region "$AWS_REGION" \
  --cluster "$ECS_CLUSTER" \
  --service "$ECS_SERVICE" \
  --force-new-deployment

aws ecs wait services-stable \
  --region "$AWS_REGION" \
  --cluster "$ECS_CLUSTER" \
  --services "$ECS_SERVICE"
```

**Migration CSDL:** đã được xử lý bằng **script / mã trong repo backend** (SpendWiseApp/backend) sau khi triển khai — workshop không lặp hướng dẫn CLI migration tại đây; xem mã nguồn và pipeline backend của bạn.

### Tham chiếu trong workshop

- [4.3.2 Triển khai hosting frontend và xác thực](../4.3-deployment-operations-monitoring/4.3.2-frontend-hosting-auth/) — Amplify + Cognito trong Terraform.
- [4.3.3 Triển khai backend](../4.3-deployment-operations-monitoring/4.3.3-backend/) — ECR, ECS, biến môi trường backend.
- [4.3.4 Triển khai database](../4.3-deployment-operations-monitoring/4.3.4-database/) — RDS và secret mật khẩu.
