---
title: "Data Layer Deployment"
date: 2025-09-10
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Overview

Data is the most important asset of every system. Therefore we will set up the data layer (Data Layer) for MiniMarket with criteria: **Maximum Security** and **High Performance**

We will deploy two core services:
1.  **Amazon RDS (Relational Database Service):** Use SQL Server to store business data (Products, Orders, Users). Database will be placed in **Private Subnet** to prevent direct access from the Internet
2.  **Amazon ElastiCache (Redis):** Use Redis as cache memory (In-memory Cache) to store Login Sessions and offload queries for the main Database

![Data Layer Architecture](/images/5-Workshop/5.4-Data/data-diagram.png)

#### Content

1. [Set up Security Groups for DB & Cache](5.4.1-security-groups/)
2. [Initialize Amazon RDS (SQL Server)](5.4.2-rds/)
3. [Initialize Amazon ElastiCache (Redis)](5.4.3-elasticache/)