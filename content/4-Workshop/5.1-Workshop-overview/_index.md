---
title: "Introduction"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

#### Introduction

- MiniMarket is an e-commerce application built on the .NET Core platform, applying modern 3-Tier Architecture. The goal of this Workshop is to re-platform the application from an On-premise environment to AWS cloud infrastructure (Cloud Native Migration) ensuring AWS Well-Architected Framework criteria: Security, Reliability, Performance Efficiency, and Cost Optimization

#### Workshop Overview

Solution Architecture:

- Compute: Use AWS Elastic Beanstalk (Docker platform) to simplify deployment, infrastructure management, and Auto Scaling
- Database: Amazon RDS for SQL Server deployed in Private Subnet to ensure data security
- Caching: Amazon ElastiCache (Redis) helps store User Sessions and offload Database queries, increasing response speed
- Network & Security:
  - VPC: Designed with Public/Private Subnet model combined with NAT Gateway
  - Application Layer Security: Use AWS WAF combined with Amazon CloudFront to protect against Web attacks and distribute content globally
- Storage: Amazon S3 used to store and serve static assets (product images) with high durability
- DevOps: Fully automated CI/CD process with AWS CodePipeline and CodeBuild
- Monitoring: Amazon CloudWatch to monitor system health (CPU, Network) and send alerts


![overview](/images/5-Workshop/5.1-Workshop-overview/project_architecture_3.1.png)