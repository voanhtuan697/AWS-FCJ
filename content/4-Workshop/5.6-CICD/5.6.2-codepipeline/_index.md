---
title: "Set up CodePipeline"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.6.2 </b> "
---
1.  Access **CodePipeline** > **Create pipeline**
   
    ![pipeline1](/images/5-Workshop/5.6-ClCD/pipeline1.png)

2. **Category:** Select Build custom pipeline

    ![pipeline2](/images/5-Workshop/5.6-ClCD/pipeline2.png)

3.  **Settings:** Select **New service role**
   
    ![pipeline3](/images/5-Workshop/5.6-ClCD/pipeline3.png)

4.  **Source Stage:** Select **GitHub (via GitHub App)** > Connect to GitHub > Select Repo and branch **that you deploy to cloud**

    ![pipline4](/images/5-Workshop/5.6-ClCD/pipeline4.png)

5.  **Build Stage:** Select **AWS CodeBuild** > Select project **MiniMarket-Build** just created

    ![pipline5](/images/5-Workshop/5.6-ClCD/pipeline5.png)

6.  **Test Stage:** Click **Skip test stage**
7.  **Deploy Stage:**
    *   Provider: **AWS Elastic Beanstalk**
    *   Application name: **MiniMarket-App**
    *   Environment name: Select running environment

    ![pipeline6](/images/5-Workshop/5.6-ClCD/pipeline6.png)

8.  Click **Create pipeline**



{{% notice warning %}}
If Deploy step has Permission error, go to IAM Role of CodePipeline and grant permission **AdministratorAccess-AWSElasticBeanstalk**
{{% /notice %}}