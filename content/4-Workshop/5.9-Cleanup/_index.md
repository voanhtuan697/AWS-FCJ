---
title: "Resource Cleanup"
date: 2025-09-10
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Overview

Congratulations on successfully deploying MiniMarket on AWS!

However, our work does not end there. The final step and also the most important step to protect your "wallet" is **Resource Cleanup**

The services we deployed such as **NAT Gateway, Elastic Load Balancer, RDS, ElastiCache** are all billed by the hour, whether you use them or not. If you forget to delete, the month-end bill can be very high

We will go through the system **Decommissioning** process in the correct order to ensure no resources are left behind causing hidden costs

#### Content

1. [Safe resource deletion process](5.9.1-cleanup/)