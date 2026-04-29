---
title: "Create Build Project"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.6.1 </b> "
---

1.  Access **CodeBuild** > **Create project**

    ![cicd1](/images/5-Workshop/5.6-ClCD/cicd1.png)

2.  **Project name:** MiniMarket-Build
    
    ![cicd2](/images/5-Workshop/5.6-ClCD/cicd2.png)

3.  **Source:** Select GitHub (Connect to Repo containing code)
  
    ![cicd3](/images/5-Workshop/5.6-ClCD/cicd3.png)

4.  **Environment:**
     - Environment image: Managed Image
     - Operating system: Amazon Linux
     - Runtime: Standard 
     - Image: 5.0
     - Service role: New service role

    ![cicd4](/images/5-Workshop/5.6-ClCD/cicd4.png)

    ![cicd5](/images/5-Workshop/5.6-ClCD/cicd5.png)

    - **Privileged:** Enable (Required to run Docker build commands)

    ![cicd6](/images/5-Workshop/5.6-ClCD/cicd6.png)

5.  **Buildspec:** Use a buildspec file

    ![cicd7](/images/5-Workshop/5.6-ClCD/cicd7.png)

6.  Click **Create build project**

{{% notice tip %}}
 After creation is complete, go to **IAM Role** of the newly created CodeBuild, grant additional permission **AmazonEC2ContainerRegistryPowerUser** so it can push images to ECR
{{% /notice %}}