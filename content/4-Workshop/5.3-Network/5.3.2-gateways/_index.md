---
title: "Configure Internet & NAT Gateway"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.3.2 </b> "
---

#### Create Internet Gateway
1. In the [VPC dashboard](https://ap-southeast-1.console.aws.amazon.com/vpcconsole/home?region=ap-southeast-1#Home:) click on **Internet gateways**

![IGW1](/images/5-Workshop/5.3-Network/IGW1.png)

2. Then click on **Create internet gateway**

![IGW2](/images/5-Workshop/5.3-Network/IGW2.png)

3. In the **Internet gateway** creation section, name it as desired then click on **Create internet gateway** and wait for it to be created

![IGW3](/images/5-Workshop/5.3-Network/IGW3.png)

4. After the **Internet gateway** is finished creating go to **Actions** and click on **Attach to VPC** to attach it to the VPC created in the previous section

![IGW4](/images/5-Workshop/5.3-Network/IGW4.png)

![IGW5](/images/5-Workshop/5.3-Network/IGW5.png)

#### Create NAT Gateway
1. Create **NAT Gateway** placed in **Public Subnet 1**
2. Assign **Elastic IP** to have a static address to the Internet

![NAT1](/images/5-Workshop/5.3-Network/NAT1.png)

![NAT2](/images/5-Workshop/5.3-Network/NAT2.png)