---
title: "Resource Cleanup"
date: 2025-09-10
weight: 1
chapter: false
pre: " <b> 5.9.1 </b> "
---

To avoid unexpected costs after completing the Workshop, delete resources in the following correct order:

1.  **NAT Gateway:** Delete NAT Gateway > Wait for Deleted > **Release Elastic IP** (Most important as it costs the most)

![DEL1](/images/5-Workshop/5.9-Cleanup/DEL1.png)

![DEL2](/images/5-Workshop/5.9-Cleanup/DEL2.png)

2.  **Elastic Beanstalk:** Terminate Environment

![DEL3](/images/5-Workshop/5.9-Cleanup/DEL3.png)

3.  **ElastiCache:** Delete Redis Cluster (Uncheck Create Backup)

![DEL4](/images/5-Workshop/5.9-Cleanup/DEL4.png)

4.  **RDS:** Stop (or Delete if no longer in use - remember to uncheck Final Snapshot).

![DEL5](/images/5-Workshop/5.9-Cleanup/DEL5.png)

5.  **WAF:** Manage resources > Disassociate >  Delete protection pack (web ACL)

![DEL7](/images/5-Workshop/5.9-Cleanup/DEL7.png)

![DEL6](/images/5-Workshop/5.9-Cleanup/DEL6.png)

6.  **S3:** Empty and Delete Bucket (Can skip deleting if still in use as cost is not too high)

7.  **CloudFront** Disable and Delete