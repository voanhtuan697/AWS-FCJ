---
title: "Initialize ElastiCache Redis"
date: 2025-09-10
weight: 3
chapter: false
pre: " <b> 5.4.3 </b> "
---

1.  Access **ElastiCache** > **Subnet groups** > **Create subnet group**
    *   Name: **redis-private-group**
    *   Subnets: Select 2 **Private Subnets**

![Elasti1](/images/5-Workshop/5.4-Data/Elasti1.png)

![Elasti2](/images/5-Workshop/5.4-Data/Elasti2.png)

2.  Go to **Redis OSS caches** > **Create cache**

![Elasti2.9](/images/5-Workshop/5.4-Data/Elasti2.9.png)

3.  At **Cluster settings** screen:
    *   **Engine:** Select **Redis OSS**
    *   **Deployment option:** Select **Node-based cluster**
    *   **Creation method:** Select **Cluster cache** (Configure and create a new cluster)
    *   **Cluster mode:** Select **Disabled** (Simple mode, 1 Shard)

![Elasti3](/images/5-Workshop/5.4-Data/Elasti3.png)

4.  At **Location** screen:
    *   **Location:** AWS Cloud
    *   **Multi-AZ:** **Uncheck** (Enable) *Note: Disable this feature to save costs for Lab environment*
    *   **Auto-failover:** **Uncheck** (Enable)

5.  At **Cache settings** screen:
    *   **Engine version:** Leave default (Ex: 7.1)
    *   **Port:** **6379**
    *   **Node type:** Select **t3** family > Select **cache.t3.micro**
    *   **Number of replicas:** Enter **0** (We only need 1 primary node, no replica node needed)

![Elasti4](/images/5-Workshop/5.4-Data/Elasti4.png)

6.  At **Connectivity** screen:
    *   **Network type:** IPv4
    *   **Subnet groups:** Select **Choose existing subnet group** > Select **redis-private-group** just created

![Elasti5](/images/5-Workshop/5.4-Data/Elasti5.png)

7.  At **Advanced settings** screen (Important):
    *   **Encryption at rest:** Enable (Default)
    *   **Encryption in transit:** **Uncheck (Disable)**
        *   *Reason:* Disabling encryption in transit simplifies connection from .NET code in internal VPC environment without configuring complex SSL certificates
    *   **Selected security groups:** Select **Manage** > select **sg-redis-cache** (Uncheck default)

![Elasti6](/images/5-Workshop/5.4-Data/Elasti6.png)

8.  Scroll to the bottom and click **Create**

#### 3. Get connection information
Initialization process will take about 5-10 minutes
1.  When status changes to **Available** (Green)
2.  Click on Cluster name (**webapp** or name you set)
3.  At **Overview** tab, find **Primary endpoint**
4.  Copy this connection string (Ex: **webapp.xxxx.cache.amazonaws.com**)

![Elasti7](/images/5-Workshop/5.4-Data/Elasti7.png)

{{% notice tip %}}
This Endpoint will be used to configure **ConnectionStrings__RedisConnection** environment variable for Elastic Beanstalk in the following steps.
{{% /notice %}}