---
title: "CI/CD Automation"
date: 2025-09-10
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

#### Overview

In the Cloud Native environment, manual deployment (Manual Deployment) is risky and time-consuming. This module will guide you to build a fully automated **CI/CD (Continuous Integration / Continuous Deployment)** process

The process operates as follows:

1.  **Source:** Developer pushes code (Push) to **GitHub**
2.  **Build:** **AWS CodePipeline** detects changes and activates **AWS CodeBuild**. CodeBuild will package the Docker Image and push to the **Amazon ECR** repository
3.  **Deploy:** Pipeline automatically commands **Elastic Beanstalk** to update the latest version from ECR without service interruption

![CI/CD Pipeline](/images/5-Workshop/5.6-ClCD/pipeline-diagram.png)

#### Content

1. [Create Build Project with AWS CodeBuild](5.6.1-codebuild/)
2. [Set up AWS CodePipeline](5.6.2-codepipeline/)