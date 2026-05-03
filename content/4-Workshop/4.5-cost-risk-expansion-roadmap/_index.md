---
title : "Cost, risk, and expansion roadmap"
date: 2026-03-10
weight : 5
chapter : false
pre : " <b> 4.5. </b> "
---

## 4.5 Cost, risk, and expansion roadmap

### Cost optimization
- Track cost drivers: ALB, ECS Fargate, VPC endpoints/NAT, RDS, Amplify, Cognito, CloudWatch.
- Keep dev lean:
  - Disable custom domain when not required.
  - Keep one small Single-AZ RDS instance.
  - Disable bastion when there is no DB maintenance session.
  - Tune log retention and monitor NAT usage.

### Risks and mitigation
- Cost spikes under traffic growth or misconfiguration.
- Secret exposure and environment configuration drift.
- Migration/backup issues and deployment interruption.
- Mitigation: release checklist, proactive monitoring, least-privilege access, and secure secret handling.

### Expansion roadmap
- Standardize end-to-end CI/CD.
- Add automated tests and smoke tests.
- Improve security controls (WAF, secret rotation, policy hardening).
- Evolve architecture when production readiness is required.
