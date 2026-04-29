---
title: "Configure Route Table"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.3.3 </b> "
---

#### Create Route Table
Click on **Route tables** section in VPC dashboard

![RT1](/images/5-Workshop/5.3-Network/RT1.png)

Create 2 Route Tables, **Public** with **Private**

![RT2](/images/5-Workshop/5.3-Network/RT2.png)

![RT3](/images/5-Workshop/5.3-Network/RT3.png)

1. **Public Route Table:**
   - For **Public Route Table**, in **Routes** section click **Edit routes**
   - Point 0.0.0.0/0 to **Internet Gateway** 

![RT4](/images/5-Workshop/5.3-Network/RT4.png)

![RT5](/images/5-Workshop/5.3-Network/RT5.png)

   - And in **Subnet associations** section, assign to both **Public Subnets**

![RT6](/images/5-Workshop/5.3-Network/RT6.png)

![RT7](/images/5-Workshop/5.3-Network/RT7.png)

2. **Private Route Table:**
   - For **Private Route Table**, we will point 0.0.0.0/0 to **NAT Gateway** 

![RT8](/images/5-Workshop/5.3-Network/RT8.png)

   - And in **Subnet associations** section, assign to both **Private Subnets**

![RT9](/images/5-Workshop/5.3-Network/RT9.png)

{{% notice tip %}}
Separating Route Tables ensures that Databases in Private Subnet are never exposed directly to the Internet.
{{% /notice %}}