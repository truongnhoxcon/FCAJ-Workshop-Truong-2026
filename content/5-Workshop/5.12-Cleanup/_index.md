---
title: "Resource Cleanup"
date: 2024-06-29
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

To avoid incurring unwanted charges on your AWS account after completing the workshop, please clean up all provisioned resources in the following recommended order.

---

#### Cleanup Order (Recommended)

> [!CAUTION]
> The deletion of resources is irreversible. Ensure you have backed up any logs or data if necessary before proceeding.

##### 1. Clean Up Amazon ECS
1. Navigate to **ECS** ➔ **Clusters** ➔ `realtime-collab-dev` Cluster ➔ select the **Services** tab.
2. Select the Service `unified-backend` ➔ Click **Update service** ➔ set **Desired tasks** to `0` ➔ Save.
3. Wait for all tasks to stop (Running tasks = 0) ➔ Select the Service and click **Delete**.
4. Return to the Cluster page and click **Delete cluster**.

##### 2. Delete Application Load Balancer (ALB)
1. Navigate to **EC2** ➔ **Load Balancers** ➔ Select `realtime-collab-dev-alb` ➔ Click **Actions** ➔ **Delete load balancer**.
2. Go to **Target Groups** in the left menu ➔ Select the 2 target groups (`uni-api-tg`, `uni-ws-tg`) ➔ Click **Actions** ➔ **Delete**.

##### 3. Disable and Delete CloudFront Distribution
1. Navigate to **CloudFront** ➔ **Distributions** ➔ Select the project's distribution ➔ Click **Disable**.
2. Wait approximately **5 minutes** for the distribution to disable (Status changes to `Disabled`).
3. Select the distribution and click **Delete**.

##### 4. Delete Amazon S3 Buckets
1. Navigate to **S3** ➔ Bucket list.
2. For each of the 2 buckets (`realtime-collab-frontend-<account-id>` and `realtime-collab-files-<account-id>`), select the bucket ➔ click **Empty** (to permanently delete all files first).
3. Once emptied, select the bucket ➔ click **Delete**. Enter the bucket name to confirm deletion.

##### 5. Delete Amazon RDS PostgreSQL
1. Navigate to **RDS** ➔ **Databases** ➔ Select `realtime-collab-dev-db` ➔ Click **Actions** ➔ **Delete**.
2. Uncheck *Create final snapshot* and *Retain automated backups* (since this is a test environment). Confirm and click **Delete**.
3. Go to **Subnet groups** and **Parameter groups** to delete the custom subnet and parameter groups created in Step 7.

##### 6. Delete ElastiCache Redis
1. Navigate to **ElastiCache** ➔ **Redis OSS caches** ➔ Select the `realtime-collab-dev-redis` cluster ➔ Click **Delete**.
2. Choose not to save a backup snapshot ➔ Click **Delete**.
3. Go to **Subnet groups** and delete the custom Redis subnet group.

##### 7. Delete ECR Repository
1. Navigate to **ECR** ➔ **Repositories**.
2. Select the repository `unified-backend` ➔ Click **Delete**.

##### 8. Delete AWS Secrets Manager Secrets
1. Navigate to **Secrets Manager** ➔ select the created secrets.
2. Click **Actions** ➔ **Delete secret** ➔ select the minimum schedule window (e.g., `7 days`) to queue them for deletion.

##### 9. Delete Amazon VPC
1. Navigate to **VPC** ➔ **Your VPCs**.
2. Select `realtime-collab-dev` ➔ Click **Actions** ➔ **Delete VPC**.
3. AWS will show all associated resources (Subnets, Route Tables, Internet Gateway, NAT Gateway, Security Groups) that will be deleted automatically. Click **Delete VPC** to confirm.
