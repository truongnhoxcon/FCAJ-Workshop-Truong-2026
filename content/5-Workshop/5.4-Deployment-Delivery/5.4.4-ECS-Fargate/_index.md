---
title: "ECS Cluster & Services"
date: 2024-06-29
weight: 4
chapter: false
pre: " <b> 5.4.4. </b> "
---

We will provision the serverless ECS Cluster on AWS Fargate, register our Task Definition, and start our Unified Service behind the ALB.

---

#### Prerequisite: Create Unified Backend Log Group
Go to **CloudWatch** ➔ **Log groups** ➔ **Create log group** named `/ecs/unified-backend` with **Retention**: `30 days`.

![Create ECS Cluster](/images/5-Workshop/5.4-Deployment-Delivery/5.4.4-ECS-Fargate/ecs-cluster-create.png)

---

#### 1. Create ECS Cluster

Navigate to **Elastic Container Service (ECS)** ➔ **Clusters** ➔ click **Create cluster**:

- **Cluster name**: `realtime-collab-dev`
- **Infrastructure**: Check Fargate only

![Create ECS Cluster - Part 1](/images/5-Workshop/5.4-Deployment-Delivery/5.4.4-ECS-Fargate/ecs-task-json-1.png)

- **Monitoring**: Check Container Insights

![Create ECS Cluster - Part 2](/images/5-Workshop/5.4-Deployment-Delivery/5.4.4-ECS-Fargate/ecs-task-json-2.png)

---

#### 2. Create Task Definition using JSON

Navigate to **ECS** ➔ **Task definitions** ➔ **Create new task definition** ➔ select **Create new revision with JSON**.

![Create Task Definition](/images/5-Workshop/5.4-Deployment-Delivery/5.4.4-ECS-Fargate/ecs-task-created.png)

##### 2.1 Task Definition — Unified Backend (`realtime-collab-unified-backend`)

Paste the JSON below and replace the placeholders:
- REPLACE_ACCOUNT_ID: Your 12-digit AWS Account ID.
- REPLACE_RDS_ENDPOINT: The RDS PostgreSQL Endpoint from Step 7.
- REPLACE_REDIS_ENDPOINT: The ElastiCache Redis Primary Endpoint from Step 8.
- REPLACE_DB_PASSWORD_ARN: The ARN of db-password.
- REPLACE_JWT_SECRET_ARN: The ARN of jwt-secret.
- REPLACE_TWILIO_SECRET_ARN: The ARN of twilio/api-credentials.
- REPLACE_REDIS_SECRET_ARN: The ARN of redis-auth-token.
- REPLACE_COGNITO_SECRET_ARN: The ARN of realtime-collab-dev/cognito.

```json
{
  "family": "realtime-collab-unified-backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::REPLACE_ACCOUNT_ID:role/ecsTaskExecutionRole-realtime-collab-dev",
  "taskRoleArn": "arn:aws:iam::REPLACE_ACCOUNT_ID:role/ecsTaskRole-realtime-collab-dev",
  "containerDefinitions": [{
    "name": "unified-backend",
    "image": "REPLACE_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/unified-backend:latest",
    "portMappings": [{ "containerPort": 3000, "protocol": "tcp" }],
    "environment": [
      { "name": "PORT",         "value": "3000" },
      { "name": "NODE_ENV",     "value": "production" },
      { "name": "DB_HOST",      "value": "REPLACE_RDS_ENDPOINT" },
      { "name": "REDIS_HOST",   "value": "REPLACE_REDIS_ENDPOINT" },
      { "name": "AWS_REGION",   "value": "us-east-1" },
      { "name": "DB_SSL",       "value": "true" },
      { "name": "REDIS_TLS",    "value": "true" },
      { "name": "S3_BUCKET_NAME","value": "realtime-collab-files-REPLACE_ACCOUNT_ID" }
    ],
    "secrets": [
      { "name": "DB_PASSWORD",         "valueFrom": "REPLACE_DB_PASSWORD_ARN" },
      { "name": "JWT_SECRET",          "valueFrom": "REPLACE_JWT_SECRET_ARN:secret::" },
      { "name": "TWILIO_ACCOUNT_SID",  "valueFrom": "REPLACE_TWILIO_SECRET_ARN:accountSid::" },
      { "name": "TWILIO_AUTH_TOKEN",   "valueFrom": "REPLACE_TWILIO_SECRET_ARN:authToken::" },
      { "name": "REDIS_PASSWORD",      "valueFrom": "REPLACE_REDIS_SECRET_ARN" },
      { "name": "COGNITO_USER_POOL_ID","valueFrom": "REPLACE_COGNITO_SECRET_ARN:userPoolId::" },
      { "name": "COGNITO_CLIENT_ID",   "valueFrom": "REPLACE_COGNITO_SECRET_ARN:clientId::" }
    ],
    "healthCheck": {
      "command": ["CMD-SHELL", "wget -qO- http://localhost:3000/health || exit 1"],
      "interval": 30, "timeout": 5, "retries": 3, "startPeriod": 60
    },
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group":         "/ecs/unified-backend",
        "awslogs-region":        "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    },
    "essential": true
  }]
}
```

![ECS Service](/images/5-Workshop/5.4-Deployment-Delivery/5.4.4-ECS-Fargate/ecs-service-status.png)

---

#### 3. Create ECS Service (Using AWS CLI)

##### Step 1: Collect Required Parameters
Access your AWS Console to gather the following IDs and ARNs:

1. **Target Group ARNs**:
   * Navigate to the **EC2** service ➔ select **Target groups** from the left-hand menu.
   * Click on realtime-collab-dev-uni-api-tg ➔ Copy the **ARN** from the Details tab.
   * Click on realtime-collab-dev-uni-ws-tg ➔ Copy the **ARN**.
2. **Private Subnet IDs**:
   * Navigate to the **VPC** service ➔ select **Subnets** ➔ Copy the Subnet IDs for both private-subnet-1 (looks like `subnet-xxxxxxxxxxxx`) and private-subnet-2.
3. **Security Group ID**:
   * Navigate to the **VPC** service ➔ select **Security groups** ➔ Copy the Group ID for realtime-collab-dev-unified-backend-sg (looks like `sg-xxxxxxxxxxxx`).

---

##### Step 2: Execute Service Creation Command on Terminal
Open your terminal (ensure you have the AWS CLI installed and configured with credentials) and run the following command (replace `REPLACE_...` placeholders with the values you gathered in Step 1):

```bash
aws ecs create-service \
  --cluster realtime-collab-dev \
  --service-name unified-backend \
  --task-definition realtime-collab-unified-backend \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration 'awsvpcConfiguration={subnets=[REPLACE_PRIVATE_SUBNET_1_ID,REPLACE_PRIVATE_SUBNET_2_ID],securityGroups=[REPLACE_UNIFIED_BACKEND_SG_ID],assignPublicIp=DISABLED}' \
  --load-balancers '[
    {
      "targetGroupArn": "REPLACE_API_TARGET_GROUP_ARN",
      "containerName": "unified-backend",
      "containerPort": 3000
    },
    {
      "targetGroupArn": "REPLACE_WS_TARGET_GROUP_ARN",
      "containerName": "unified-backend",
      "containerPort": 3000
    }
  ]'
```

After a successful invocation, the CLI will output the newly created Service details in JSON format.

---

##### Step 3: Verify Service Deployment Status
1. Navigate to **ECS** ➔ **Clusters** ➔ Click on the `realtime-collab-dev` Cluster.
2. In the **Services** tab, you should see the **unified-backend** service successfully created.
3. Click on the Service name ➔ Select the **Tasks** tab to monitor docker image download and container provisioning. Once the tasks display status **Running** and the Health status shows **Healthy**, the system is fully operational!

![ECS Service running healthy status](/images/5-Workshop/5.4-Deployment-Delivery/5.4.4-ECS-Fargate/tasks-status.png)

