---
title: "Set up Firewall (WAF)"
date: 2025-09-10
weight: 2
chapter: false
pre: " <b> 5.7.2 </b> "
---

1.  Access **WAF & Shield** > **Protection packs (webs ACLs)** > **Create protection pack (web ACL)**

![WAF1](/images/5-Workshop/5.7-Security/WAF1.png)

2.  **App category:** E-commerce & transaction platforms
3.  **App focus:** Both API and web

![WAF2](/images/5-Workshop/5.7-Security/WAF2.png)

3.  **Add resources** > **Add CloudFront or Amplify resources** > **Select CloudFront distribution created in previous section**

![WAF3](/images/5-Workshop/5.7-Security/WAF3.png)

![WAF4](/images/5-Workshop/5.7-Security/WAF4.png)

4.  **Choose initial protections** > **Build your own pack from all of the protections AWS WAF offers** > **AWS-managed rule group**:

    ![WAF5](/images/5-Workshop/5.7-Security/WAF5.png)

    ![WAF6](/images/5-Workshop/5.7-Security/WAF6.png)

    *   Add **Core rule set** (Block bot, bad IP)
    *   Add **SQL database** (Block SQL Injection)

![WAF7](/images/5-Workshop/5.7-Security/WAF7.png)

**Testing:** Access URL: **https://[domain]/?id=1 OR 1=1**. If receiving error **403 Forbidden**, WAF is active.