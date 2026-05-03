---
title : "Deploy frontend and backend to AWS"
date: 2026-03-10
weight : 4
chapter : false
pre : " <b> 4.4. </b> "
---

## 4.4 Deploy frontend and backend to AWS

This section describes the **operational flow** after Terraform has created Amplify, ECR, ECS, and (if enabled) RDS — how you **ship frontend and backend code** to the live environment. Architecture and Terraform variables are covered in [4.3 Terraform deployment (by layer)](../4.3-deployment-operations-monitoring/).

### Frontend (Amplify hosting)

- You need a **Git repository URL** (commonly GitHub) and an **access token** (personal access token or equivalent) so Amplify can **clone the repo** and run builds. Pass these into Terraform as **amplify_repository_url** and **amplify_access_token** (via terraform.tfvars or CI secrets — **never** commit real tokens).
- The branch Amplify builds (e.g. **main**) should match **amplify_branch_name**. After you push code, Amplify rebuilds the Next.js app (monorepo **frontend** directory) and serves it on Amplify’s CDN-style hosting.
- The API base URL the UI calls (**NEXT_PUBLIC_API_URL**) is wired to the ALB in Terraform; ensure the backend and DNS/TLS are ready before expecting the UI to call the API successfully.

**Example CLI** (requires **AWS CLI**, **Terraform**, and **Docker**; adjust **TF_DIR** and **docker** paths to match where you cloned the app — e.g. if your shell is **inside** SpendWiseApp, use **TF_DIR=infrastructure/environments/dev** and **-f backend/Dockerfile** with build context **backend**).

```bash
# Example: current directory is the parent of the SpendWiseApp folder (e.g. awslab workspace)
export TF_DIR=SpendWiseApp/infrastructure/environments/dev

terraform -chdir=$TF_DIR output amplify_app_id
terraform -chdir=$TF_DIR output amplify_branch_name
terraform -chdir=$TF_DIR output amplify_default_domain
```

After **git push** to the configured branch, Amplify usually starts a build automatically. To **trigger a release build** manually:

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

Track the job in the Amplify console or with **aws amplify get-job**.

### Backend (Docker → ECR → ECS)

- **Build** the backend (NestJS) image from the repo Dockerfile and **tag** it to match what Terraform uses (e.g. **latest** or a commit-based tag).
- **Authenticate** to ECR (aws ecr get-login-password), **docker push** the image to the repository Terraform created (see **ecr_repository_url** in the dev outputs).
- **Roll out** the new image on ECS — for example **aws ecs update-service --cluster … --service … --force-new-deployment** (or bump the task definition revision if your pipeline owns that). Wait until new tasks are **running** and healthy behind the ALB target group.

**Example CLI** (same **TF_DIR** convention as above):

```bash
export TF_DIR=SpendWiseApp/infrastructure/environments/dev
export AWS_REGION="$(terraform -chdir=$TF_DIR output -raw aws_region)"
export ECR_REPO="$(terraform -chdir=$TF_DIR output -raw ecr_repository_url)"
export ECR_REGISTRY="${ECR_REPO%%/*}"
export IMAGE_TAG=latest   # must match ecs_backend_image_tag in Terraform

aws ecr get-login-password --region "$AWS_REGION" \
  | docker login --username AWS --password-stdin "$ECR_REGISTRY"

docker build -f SpendWiseApp/backend/Dockerfile -t spendwise-backend:"$IMAGE_TAG" SpendWiseApp/backend
docker tag spendwise-backend:"$IMAGE_TAG" "${ECR_REPO}:${IMAGE_TAG}"
docker push "${ECR_REPO}:${IMAGE_TAG}"
```

Roll the ECS service and wait until it is stable:

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

**Database migrations** are handled by **scripts / code in the backend repo** (SpendWiseApp/backend) after deployment — this workshop page does not duplicate migration CLI steps; see your backend source and release pipeline.

### Workshop cross-links

- [4.3.2 Deploy frontend hosting and user authentication](../4.3-deployment-operations-monitoring/4.3.2-frontend-hosting-auth/) — Amplify + Cognito in Terraform.
- [4.3.3 Deploy backend](../4.3-deployment-operations-monitoring/4.3.3-backend/) — ECR, ECS, backend environment variables.
- [4.3.4 Deploy database](../4.3-deployment-operations-monitoring/4.3.4-database/) — RDS and the DB password secret.
