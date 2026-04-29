---
title: "Thiết lập CodePipeline"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.6.2 </b> "
---
1.  Truy cập **CodePipeline** > **Create pipeline**
   
    ![pipeline1](/images/5-Workshop/5.6-ClCD/pipeline1.png)

2. **Category:** Chọn Buid custom pipeline

    ![pipeline2](/images/5-Workshop/5.6-ClCD/pipeline2.png)

3.  **Settings:** Chọn **New service role**
   
    ![pipeline3](/images/5-Workshop/5.6-ClCD/pipeline3.png)

4.  **Source Stage:** Chọn **GitHub (via GitHub App)** > Connect to GitHub > Chọn Repo và nhánh **mà bạn deploy lên cloud**

    ![pipline4](/images/5-Workshop/5.6-ClCD/pipeline4.png)

5.  **Build Stage:** Chọn **AWS CodeBuild** > Chọn project **MiniMarket-Build** vừa tạo

    ![pipline5](/images/5-Workshop/5.6-ClCD/pipeline5.png)

6.  **Test Stage:** Bấm **Skip test stage**
7.  **Deploy Stage:**
    *   Provider: **AWS Elastic Beanstalk**
    *   Application name: **MiniMarket-App**
    *   Environment name: Chọn môi trường đang chạy

    ![pipeline6](/images/5-Workshop/5.6-ClCD/pipeline6.png)

8.  Bấm **Create pipeline**



{{% notice warning %}}
Nếu bước Deploy bị lỗi Permission, hãy vào IAM Role của CodePipeline và cấp quyền **AdministratorAccess-AWSElasticBeanstalk**
{{% /notice %}}
