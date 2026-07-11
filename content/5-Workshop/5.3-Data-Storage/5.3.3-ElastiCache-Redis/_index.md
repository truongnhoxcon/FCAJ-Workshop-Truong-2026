---
title: "ElastiCache Redis"
date: 2024-06-29
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

We will create an Amazon ElastiCache (Redis OSS) cluster to synchronize real-time socket events (Pub/Sub) and cache user online/offline presence states.

---

#### 1. Create Cache Subnet Group

1. Navigate to the **ElastiCache** service on the AWS Console ➔ select **Subnet groups** in the left menu ➔ click **Create subnet group**.
2. Configure as follows:
   - **Name**: `realtime-collab-dev-redis-subnet-group`
   - **Description**: `Private subnet group for Redis`
   - **VPC**: Select realtime-collab-dev
   - **Subnets**: Select the 2 private subnets matching the IP ranges 10.0.10.0/24 and 10.0.11.0/24.
3. Click **Create**.

![Create Redis Cache Subnet Group](/images/5-Workshop/5.3-Data-Storage/5.3.3-ElastiCache-Redis/redis-subnet-group-create.png)

---

#### 2. Create Redis Cache Cluster (New Console UI)

Navigate to **ElastiCache** ➔ **Caches** (in the left menu) ➔ click **Create cache**.

---

##### 📝 STEP 1: Configure Settings

On the **Step 1: Settings** screen, configure the following options:

##### 1. Configuration
- **Engine**: Select **Redis OSS** (Note: Do not select *Valkey* to ensure compatibility with our project's Socket.io Redis Adapter).
- **Deployment option**: Select **Node-based cluster** (Runs a standard node structure to control development costs, do not choose *Serverless*).
- **Creation method**: Select **Cluster cache** (To display all detailed custom configuration options).
- **Cluster mode**: Select **Disabled** (Runs a single node for testing and development).

![Configure Redis Settings Part 1](/images/5-Workshop/5.3-Data-Storage/5.3.3-ElastiCache-Redis/redis-create-settings-1.png)

##### 2. Cluster info
- **Name**: Enter `realtime-collab-dev-redis`
- **Description**: Enter `Redis 7 for realtime-collab`
- **Location**: Choose **AWS Cloud**
- **Multi-AZ**: Check **Enabled**.

![Configure Redis Settings Part 2](/images/5-Workshop/5.3-Data-Storage/5.3.3-ElastiCache-Redis/redis-create-settings-2.png)

##### 3. Cache settings
- **Engine version**: Select **7.0** (or the latest available 7.x version).
- **Port**: Keep the default `6379`.
- **Parameter groups**: Select default.redis7.
- **Node type**: Click the selection box and select **cache.t3.micro**.
- **Number of replicas**: Enter **`1`**.

![Configure Redis Settings Part 3](/images/5-Workshop/5.3-Data-Storage/5.3.3-ElastiCache-Redis/redis-create-settings-3.png)

##### 4. Connectivity
- **Network type**: Select IPv4.
- **Subnet groups**: Choose **Choose existing subnet group** ➔ Select the `realtime-collab-dev-redis-subnet-group` created in Step 1.
- **Availability Zone placements**: Leave as No preference.

![Configure Redis Connectivity](/images/5-Workshop/5.3-Data-Storage/5.3.3-ElastiCache-Redis/redis-create-advanced-1.png)

Click **Next** to proceed to Step 2.

---

##### ⚙️ STEP 2: Configure Advanced Settings

On the **Step 2: Advanced settings** screen, configure the security parameters:

##### 1. Security
- **Encryption at rest**: Select **Enable** ➔ Choose **Default key**.
- **Encryption in transit**: Select **Enable** (Crucial for secure payload transmission).
- **Access control**: Choose **Redis AUTH default user**.
- **Auth token**: Enter the authentication token (Redis password) retrieved from AWS Secrets Manager `redis-auth-token` in Step 1.

![Configure Redis Advanced Security & Auth Token](/images/5-Workshop/5.3-Data-Storage/5.3.3-ElastiCache-Redis/redis-create-advanced-2.png)

- **Selected security groups**: Click **Manage** ➔ Select **redis-sg** created in Step 4 (ensure default is unchecked) ➔ click **Save**.

![Configure Redis Security Groups](/images/5-Workshop/5.3-Data-Storage/5.3.3-ElastiCache-Redis/redis-create-advanced-3.png)

##### 2. Backup & Maintenance
- **Enable automatic backups**: Uncheck.
- **Auto minor version upgrade**: Select **Enable**.

![Redis Cluster List Status](/images/5-Workshop/5.3-Data-Storage/5.3.3-ElastiCache-Redis/redis-create-status.png)

Click **Next** to review all details on the **Step 3: Review and create** screen.

Click **Create** to launch the cache cluster.

---

#### 3. Retrieve Connection Endpoint

The Redis cluster takes **5 - 10 minutes** to provision. Once the status shows Available:
1. Click on the cluster name `realtime-collab-dev-redis`.
2. Copy the **Primary Endpoint** (e.g., `realtime-collab-dev-redis.xxxxxx.ng.0001.use1.cache.amazonaws.com`).
