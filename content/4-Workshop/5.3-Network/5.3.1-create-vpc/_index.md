---
title: "Create VPC & Subnets"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.3.1 </b> "
---

1. Open [Amazon VPC console](https://ap-southeast-1.console.aws.amazon.com/vpcconsole/home?region=ap-southeast-1#Home:) (Note: Choose region suitable for needs, here the group uses Region ap-southeast-1)
2. Select **Create VPC**

![VPC Tutorial 1](/images/5-Workshop/5.3-Network/vpc-tutorial-1.png)

3. Configuration:
- Name: **MiniMarket-VPC** 
- IPv4 CIDR: **10.0.0.0/16**

![VPC Tutorial 2](/images/5-Workshop/5.3-Network/vpc-tutorial-2.png)

4. Create Subnets (Split 2 AZs to ensure High Availability):
- Public Subnets (2): **10.0.1.0/24 & 10.0.2.0/24** (Used for Load Balancer & NAT)
- Private Subnets (2): **10.0.3.0/24 & 10.0.4.0/24** (Used for App, DB, Redis)

![VPC Tutorial 3](/images/5-Workshop/5.3-Network/vpc-tutorial-3.png)

5. Click **Create VPC** and wait for state to change to **Available** is successful

![VPC Tutorial 4](/images/5-Workshop/5.3-Network/vpc-tutorial-4.png)