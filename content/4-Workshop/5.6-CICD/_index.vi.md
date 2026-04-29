---
title: "Tự động hóa CI/CD"
date: 2025-09-10
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Tổng quan

Trong môi trường Cloud Native, việc triển khai thủ công (Manual Deployment) là rủi ro và tốn thời gian. Module này sẽ hướng dẫn bạn xây dựng một quy trình **CI/CD (Continuous Integration / Continuous Deployment)** hoàn toàn tự động

Quy trình hoạt động như sau:

1.  **Source:** Developer đẩy code (Push) lên **GitHub**
2.  **Build:** **AWS CodePipeline** phát hiện thay đổi và kích hoạt **AWS CodeBuild**. CodeBuild sẽ đóng gói Docker Image và đẩy lên kho chứa **Amazon ECR**
3.  **Deploy:** Pipeline tự động ra lệnh cho **Elastic Beanstalk** cập nhật phiên bản mới nhất từ ECR mà không gây gián đoạn dịch vụ

![CI/CD Pipeline](/images/5-Workshop/5.6-ClCD/pipeline-diagram.png)

#### Nội dung

1. [Tạo Build Project với AWS CodeBuild](5.6.1-codebuild/)
2. [Thiết lập AWS CodePipeline](5.6.2-codepipeline/)

