---
title: "Network Infrastructure Setup"
date: 2025-09-10
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

#### Overview

In this section, we will build the network foundation for the MiniMarket application. A secure network architecture is a prerequisite to protect the application and data

We will design a **VPC** consisting of:
- **Public Subnet:** Dedicated to components communicating directly with the Internet (Load Balancer, NAT Gateway).
- **Private Subnet** Dedicated to components requiring security (App Server, Database, Redis)

Additionally, we will configure **NAT Gateway** to allow servers inside the Private Subnet to download updates and Docker Images from the Internet without exposing IP addresses externally

![VPC Architecture](/images/5-Workshop/5.3-Network/vpc-diagram.png)

#### Content

- [Create VPC & Subnet](5.3.1-create-vpc/)
- [Configure Internet & NAT Gateway](5.3.2-gateways/)
- [Configure Route Table](5.3.3-routing/)