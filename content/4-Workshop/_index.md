---
title: "Workshop"
date: 2026-03-10
weight: 4
chapter: false
pre: " <b> 4. </b> "
---



# Deploying Cloud-Native MiniMarket Application on AWS

#### Overview

This workshop provides a comprehensive guide on re-platforming the **MiniMarket** e-commerce application (developed on the .NET Core platform) from a local environment to the AWS cloud infrastructure using a **Cloud Native** architecture.

We will not merely rent a virtual server (EC2) to run code. Instead, we will build a distributed system that is highly scalable, secure, and automated based on managed services.

We will establish a Multi-tier architecture consisting of core components:

- **Compute**: Use AWS Elastic Beanstalk combined with Docker to simplify application deployment and management, supporting automatic Auto Scaling based on traffic.
- **Data & Caching**: Migrate from local SQL Server to Amazon RDS (placed in Private Subnet) to ensure data security. Simultaneously, deploy Amazon ElastiCache (Redis) to manage User Sessions, ensuring high performance for the application.
- **Networking & Security**: Use **VPC** along with **Public/Private Subnet** and **NAT Gateway** for secure outbound connection, and protect the application against attacks using **AWS WAF** combined with **CloudFront**.
- **DevOps**: Build a **CI/CD** process using **AWS CodePipeline** and **CodeBuild**, allowing automation of the process from committing code to GitHub until the application runs in the Production environment
#### Content

1. [Workshop Overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [Network Infrastructure Setup (VPC, NAT, Security Groups)](5.3-Network/)
4. [Data Layer Deployment (RDS & Redis)](5.4-Data/)
5. [Application Deployment with Elastic Beanstalk & Docker](5.5-App/)
6. [Automation with CI/CD Pipeline](5.6-CICD/)
7. [Optimization and Security(S3, CloudFront, WAF)](5.7-Security/)
8. [Monitoring (CloudWatch)](5.8-Monitoring/)
9. [Resource Cleanup](5.9-Cleanup/)