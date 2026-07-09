---
title: "RDS PostgreSQL"
date: 2024-06-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

We will configure a persistent Amazon RDS PostgreSQL instance running privately and enforcing SSL encryption for secure data transport.

---

#### 1. Create DB Subnet Group

Before creating the database, define a subnet group to ensure the RDS database is located inside the private subnets:

1. Navigate to **RDS** ➔ **Subnet groups** ➔ **Create DB subnet group**.
2. Configure as follows:
   - **Name**: `realtime-collab-dev-db-subnet-group`
   - **Description**: `Private subnet group for RDS`
   - **VPC**: Select realtime-collab-dev
   - **Add subnets**: Select availability zones us-east-1a and us-east-1b.
   - **Subnets**: Check the private subnets with CIDR blocks 10.0.10.0/24 and 10.0.11.0/24.
3. Click **Create**.

![Create DB Subnet Group](/images/5-Workshop/5.6-RDS-Database/db-subnet-group-create.png)

---

#### 2. Create Parameter Group (Enforce SSL)

1. Navigate to **RDS** ➔ **Parameter groups** ➔ **Create parameter group**.
2. Select:
   - **Parameter group family**: Choose postgres15
   - **Type**: DB Parameter Group
   - **Group name**: `realtime-collab-dev-pg15-ssl`
   - **Description**: `PostgreSQL 15 enforce SSL`
3. Click **Create**.

![Create Parameter Group](/images/5-Workshop/5.6-RDS-Database/db-subnet-group-details.png)

4. Select the created group ➔ click **Edit** ➔ Search for parameter `rds.force_ssl` and set its value to `1` (Apply method: `immediate`). Click **Save changes**.

![Edit Parameter Group](/images/5-Workshop/5.6-RDS-Database/db-parameter-group-create.png)

![Enforce SSL encryption](/images/5-Workshop/5.6-RDS-Database/db-parameter-group-edit.png)

---

#### 3. Provision Database Instance

Go to **RDS** ➔ **Databases** ➔ **Create database**. Configure with the following options:

- **Choose a database creation method**: Full configuration
- **Engine options**: PostgreSQL
- **Templates**: Dev/Test

![Database creation Engine Options](/images/5-Workshop/5.6-RDS-Database/db-create-engine.png)

- **Availability and durability**: **Multi-AZ DB instance**

![Database Settings & Credentials](/images/5-Workshop/5.6-RDS-Database/2az.png)

- **Settings**:
  - **DB instance identifier**: `realtime-collab-dev-db`
  - **Master username**: `postgres`
  - **Master password**: Use the exact password generated in Secrets Manager `db-password` in Step 1.

![Database Master Password settings](/images/5-Workshop/5.6-RDS-Database/db-create-settings-2.png)
- **Instance configuration**: db.t3.medium
- **Storage**:
  - Storage type: `gp3` | Allocated storage: `100 GB`
  - **Enable storage autoscaling**: **Check** | Maximum storage threshold: `500 GB`

![Database Instance Type](/images/5-Workshop/5.6-RDS-Database/db-create-instance.png)
![Database Storage Autoscaling](/images/5-Workshop/5.6-RDS-Database/db-create-storage.png)
- **Connectivity**:
  - **VPC**: realtime-collab-dev
  - **DB Subnet group**: realtime-collab-dev-db-subnet-group
  - **Public access**: Select No
  - **VPC security group (existing)**: Choose rds-sg (Remove the default security group).
- **Database authentication**: Password authentication

![Database Connectivity Part 1](/images/5-Workshop/5.6-RDS-Database/db-create-network-1.png)
![Database Connectivity Part 2](/images/5-Workshop/5.6-RDS-Database/db-create-network-2.png)
- **Additional configuration**:
  - **Initial database name**: `realtime_collab`
  - **DB parameter group**: Select `realtime-collab-dev-pg15-ssl`
  
![Database Additional Configuration Part 1](/images/5-Workshop/5.6-RDS-Database/db-create-additional-1.png)
  - **Backup**:
    - Backup retention period: 7 days
    - Backup window: Select 03:00 - 04:00 UTC
    - **Copy tags to snapshots**: **Check**

![Database Additional Configuration Part 2](/images/5-Workshop/5.6-RDS-Database/db-create-additional-2.png)
  - **Encryption**: **Check** Enable encryption
  - **Monitoring**:
    - **Enable Enhanced Monitoring**: **Check** (Granularity: 60 seconds, create new Monitoring role).
  - **Performance Insights**: **Check** Enable Performance Insights (Retention: 7 days).
  - **Log exports**: **Check** PostgreSQL log
  - **Maintenance**: **Check** Auto minor version upgrade
  - **Deletion protection**: Uncheck (for easy cleanup during this workshop).

![Database Additional Configuration Part 3](/images/5-Workshop/5.6-RDS-Database/db-create-additional-3.png)

Click **Create database**. The initialization takes about **10 - 15 minutes**. Once the status changes to Available, copy the **Endpoint** to use as DB_HOST in subsequent steps.
