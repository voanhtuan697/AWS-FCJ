---
title: "Proposal"
date: 2026-03-10
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Personal Expense Manager (PEM)

### 1. Project Overview & Objectives

The **Personal Expense Manager (PEM)** is a full-stack web application designed to solve a real-world problem: helping users track, analyze, and control their personal finances in a visual and efficient manner. According to surveys, over 60% of Vietnamese people do not have a habit of systematically tracking their expenses, leading to poor financial control and ineffective savings.

The project is built entirely on the **AWS platform**, demonstrating the ability to integrate cloud services into real-world products and showcasing the knowledge accumulated during 3 weeks of AWS Services training. Core objectives of the project:

- Help users record income and expenses quickly and easily.
- Categorize transactions by category for spending structure analysis.
- Provide visual statistical reports to help users understand their financial situation.
- Alert users when spending is at risk of exceeding set budgets.
- Support management of multiple financial accounts (cash, bank, credit card...).

### 2. Technology Architecture & AWS Infrastructure

#### Tech Stack Details:

| Layer / Component      | Technology & Service                         |
| :--------------------- | :------------------------------------------- |
| **Frontend Framework** | Next.js 14 (App Router), TypeScript          |
| **Frontend UI**        | Tailwind CSS, Shadcn/ui component library    |
| **Frontend Charts**    | Chart.js with react-chartjs-2 wrapper        |
| **Frontend State**     | TanStack Query (React Query) v5              |
| **Backend Framework**  | NestJS (Node.js), TypeScript                 |
| **Backend ORM**        | TypeORM with migration and entity            |
| **Backend Validation** | class-validator, class-transformer           |
| **Backend Docs**       | Swagger/OpenAPI (@nestjs/swagger)            |
| **Database**           | PostgreSQL 15 on AWS RDS (db.t3.micro)       |
| **Caching**            | Redis 7 on AWS ElastiCache (cache.t3.micro)  |
| **Authentication**     | AWS Cognito User Pool + JWT                  |
| **File Storage**       | AWS S3 (Standard storage class)              |
| **Email Service**      | AWS SES (Simple Email Service)               |
| **Serverless**         | AWS Lambda (Node.js 20.x runtime)            |
| **Scheduling**         | Amazon EventBridge (CloudWatch Events)       |
| **Frontend Hosting**   | AWS Amplify + CloudFront CDN                 |
| **Backend Hosting**    | AWS ECS Fargate (serverless container)       |
| **Container Registry** | Amazon ECR (Elastic Container Registry)      |
| **Load Balancer**      | AWS Application Load Balancer (ALB)          |
| **Networking**         | AWS VPC, Private/Public Subnets, NAT Gateway |
| **CI/CD**              | AWS CodePipeline + CodeBuild + CodeDeploy    |
| **Monitoring**         | Amazon CloudWatch (logs, metrics, alarms)    |
| **Testing**            | Jest, Supertest, Testing Library             |
| **Version Control**    | Git + GitHub                                 |

#### AWS Architecture Diagram:

![PEM AWS Infrastructure Diagram](/images/2-Proposal/AWS_infrastructure_diagram.jpg)

**System Architecture:** Next.js (Amplify/CloudFront) → ALB → ECS Fargate (NestJS) → RDS PostgreSQL / ElastiCache Redis / S3, with Lambda + EventBridge for background jobs, Cognito for authentication, CodePipeline CI/CD, CloudWatch monitoring.

### 3. PostgreSQL Database Design

The PostgreSQL database is designed following normalization principles up to **3NF (Third Normal Form)**, ensuring data consistency and good query performance.

| Table Name       | Main Columns                                                                                                           | Purpose                                              |
| :--------------- | :--------------------------------------------------------------------------------------------------------------------- | :--------------------------------------------------- |
| **users**        | id (UUID PK), cognito_sub, email, full_name, avatar_url, currency, timezone, created_at                                | Store user info, linked with Cognito                 |
| **accounts**     | id (UUID PK), user_id (FK), name, type (ENUM), balance, currency, color, icon, is_active                               | Manage user's financial accounts                     |
| **categories**   | id (UUID PK), user_id (FK), parent_id (FK self-ref), name, type (ENUM), icon, color, is_default                        | Transaction categorization, supports hierarchy       |
| **transactions** | id (UUID PK), user_id (FK), account_id (FK), category_id (FK), amount, type (ENUM), description, date, receipt_url     | Record all financial transactions                    |
| **budgets**      | id (UUID PK), user_id (FK), category_id (FK), amount_limit, spent_amount, period_type (ENUM), start_date, end_date     | Set and track budget limits                          |
| **budget_alerts**| id (UUID PK), budget_id (FK), alert_type (ENUM: 80%, 100%), sent_at, email_sent_to                                     | History of budget alert emails sent                  |
| **report_exports** | id (UUID PK), user_id (FK), type (ENUM: PDF/EXCEL), s3_key, s3_url, expires_at, created_at                          | Store exported report download links                 |

**Key indexes for query optimization:**
- `idx_transactions_user_date` on transactions(user_id, date DESC)
- `idx_transactions_category` on transactions(category_id, date)
- `idx_budgets_active` on budgets(user_id, is_active, start_date, end_date)

### 4. Core Features

#### 4.1. User Authentication (AWS Cognito)

- **Registration:** Users sign up with name, email, and password → Cognito sends 6-digit OTP via email → User verifies OTP → Account activated → Auto-login to dashboard.
- **Login:** Frontend calls Cognito SDK (initiateAuth API) → Returns AccessToken (1h), IdToken (1h), RefreshToken (30d) → Backend verifies JWT via Cognito JWKS endpoint.
- **Forgot Password:** User enters email → Cognito sends verification code → User enters code + new password → Password updated.

#### 4.2. Overview Dashboard

The Dashboard provides a comprehensive overview of finances in the current period (default: current month):

- **Summary Cards:** Total Income (green), Total Expenses (red), Net Balance, Total Assets.
- **Bar Chart:** Compares total income and expenses monthly for the past 6 months. Data cached on Redis with 10-minute TTL.
- **Pie Chart:** Shows spending distribution by category with legend.
- **Recent Transactions:** Displays 5 most recent transactions with "View All" link.

#### 4.3. Financial Account Management

- **Account List:** Displays all financial accounts as cards (icon, name, type, current balance, action buttons).
- **Add New Account:** Name, Type (Cash/Bank Account/Credit Card/E-Wallet/Investment), Initial Balance, Currency, Color, Icon.
- **Total Assets Card:** Sum of positive balances – Credit card debts.

#### 4.4. Transaction Management

Transactions are divided into 3 types: **Income**, **Expense**, and **Transfer** (between internal accounts).

- **Transaction List:** All transactions sorted by date descending, grouped by day.
- **Add Transaction:** Type tabs, Amount input, Category grid, Account dropdown, DateTime picker, Notes, Receipt upload (direct to S3 via presigned URL).
- **Advanced Filters:** Date range, Transaction type, Categories, Accounts, Amount range, Full-text keyword search.

#### 4.5. Category Management

- **Default Categories (Expense):** 🍽️ Food, 🚗 Transport, 🛒 Shopping, 🏠 Housing, 💊 Health, 🎓 Education, 🎬 Entertainment, 💝 Family & Friends, 💳 Finance.
- **Default Categories (Income):** 💰 Salary, 📈 Investment, 🏢 Business, 🎁 Others.
- **Custom Categories:** Users can create their own categories with name, type, parent category, icon (200+ emojis), and color.

#### 4.6. Budget Management

- **Create Budget:** Select expense category, set limit amount, choose period (Monthly/Weekly/Custom), toggle Active status.
- **Track Progress:** Progress bar with color coding (<70% green, 70-90% yellow, 90-100% orange, >100% red).
- **AWS Lambda + SES Alert System:** Lambda function triggered by EventBridge every hour → Queries active budgets at ≥80% usage → Sends HTML email via SES → Logs to `budget_alerts` table to prevent duplicates. Also checks for 100% (over-budget) alerts.

#### 4.7. Statistical Reports & Export

- **Comprehensive Report:** Line chart of daily income/expense trends, summary table, breakdown by category, comparison with previous period.
- **PDF Export:** Generated by `pdfkit` on NestJS, includes user header, SVG charts, transaction list.
- **Excel Export:** Generated by `exceljs` with multiple sheets (Summary, Full Transactions, Category Statistics).
- Files uploaded to S3 with presigned URL (1-hour validity), stored in `report_exports` table.

#### 4.8. Profile Settings

- Personal info, avatar upload (to S3), default currency, timezone, password change, budget alert email preferences.

### 5. AWS Production Deployment

#### 5.1. Frontend – AWS Amplify + CloudFront

- Next.js 14 deployed on AWS Amplify, auto-detect and build on push to `main` branch.
- CloudFront CDN for global content distribution from edge locations.
- Environment variables configured in Amplify Console (Cognito Pool ID, API endpoint).

#### 5.2. Backend – AWS ECS Fargate

- Multi-stage Dockerfile for optimized image size.
- Docker image pushed to ECR with git SHA tag.
- ECS Task Definition: 0.5 vCPU, 1GB RAM, env vars from Secrets Manager.
- Auto-scaling based on CPU utilization (scale-out >70%, scale-in <30%).
- Rolling deployment for zero-downtime updates.

#### 5.3. Database – Amazon RDS PostgreSQL

- db.t3.micro for dev, db.t3.small for production.
- 20GB SSD with Storage Autoscaling.
- Multi-AZ Deployment with automatic failover (<60s).
- Daily automated backup at 3 AM, 7-day retention with PITR support.
- Located in private subnet, storage encryption with AWS KMS.

#### 5.4. Caching – Amazon ElastiCache Redis

- Redis 7 single node (cache.t3.micro) in private subnet.
- Cache key strategy: `user:{userId}:dashboard:stats:{period}` with 10-min TTL.
- Auto-invalidation on transaction changes.

### 6. CI/CD Pipeline with AWS CodePipeline

The pipeline consists of 4 stages:

1. **Source:** Connected to GitHub, auto-triggers on push to `main`.
2. **Build (CodeBuild):** Run unit tests (`jest --coverage`), build Docker image, tag & push to ECR.
3. **Deploy to Staging:** Auto-deploy to staging ECS, run smoke tests.
4. **Deploy to Production:** Manual approval required, then deploy to production ECS.

### 7. Monitoring with Amazon CloudWatch

#### CloudWatch Dashboard Metrics:
- **ECS:** CPUUtilization, MemoryUtilization, running task count.
- **ALB:** RequestCount, TargetResponseTime, HTTPCode_Target_5XX_Count.
- **RDS:** DatabaseConnections, CPUUtilization, FreeStorageSpace.
- **Lambda:** Invocations, Duration, Errors.
- **ElastiCache:** CacheHits, CacheMisses, CurrConnections.

#### CloudWatch Alarms:
- ECS CPU > 80% for 5 minutes → SNS email notification.
- RDS Connections > 80 → Connection pool warning.
- ALB HTTP 5xx > 10 errors/min → Critical backend error alert.
- Lambda Error Rate > 5% → Budget checker issue alert.

### 8. Performance Evaluation

| Metric                             | Measured Result | Target           |
| :--------------------------------- | :-------------- | :--------------- |
| API Response Time (p50)            | 45ms            | < 200ms ✅        |
| API Response Time (p95)            | 120ms           | < 500ms ✅        |
| API Response Time (p99)            | 280ms           | < 1000ms ✅       |
| Dashboard load time (with cache)   | 180ms           | < 300ms ✅        |
| Dashboard load time (cold)         | 650ms           | < 1000ms ✅       |
| Unit Test Coverage                 | 78.3%           | > 75% ✅          |
| Integration Tests Pass Rate        | 100%            | 100% ✅           |
| RDS Max Connections (concurrent)   | 32/100          | < 80 ✅           |
| ECS CPU Utilization (normal load)  | 18%             | < 70% ✅          |
| ElastiCache Hit Rate               | 87.4%           | > 80% ✅          |
| S3 Upload Success Rate             | 99.98%          | > 99.9% ✅        |
| Lambda Execution Duration (avg)    | 1.2s            | < 3s ✅           |
