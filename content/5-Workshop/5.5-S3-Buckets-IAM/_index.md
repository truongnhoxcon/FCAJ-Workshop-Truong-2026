---
title: "S3 Buckets & IAM Roles"
date: 2024-06-29
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

We will create 2 S3 Buckets for static web hosting (frontend) and user files storage, and 2 IAM Roles to provide the least privileges required for ECS Tasks.

---

### PART 1 — S3 Buckets Setup

Navigate to **S3** ➔ **Create bucket**. Create the following 2 buckets in **us-east-1** (Replace '<account-id>' with your 12-digit AWS Account ID).

#### 1. Frontend Static Hosting Bucket: `realtime-collab-frontend-<account-id>`
- **Block public access**: Block all public access **➔ Enabled** (Secure storage, CloudFront uses OAC to read).
- **Bucket Versioning**: Disabled (No versioning needed).
- **Encryption**: SSE-S3 (AES-256)

![Create S3 Frontend - Name and Region](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-frontend-1.png)
![Create S3 Frontend - Block Public Access and Versioning](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-frontend-2.png)
![Create S3 Frontend - Encryption](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-frontend-3.png)

- **Bucket Policy** (Permissions tab ➔ Bucket policy - allows CloudFront OAC to read objects):
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "AllowCloudFrontServicePrincipalReadOnly",
        "Effect": "Allow",
        "Principal": {
          "Service": "cloudfront.amazonaws.com"
        },
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::realtime-collab-frontend-<account-id>/*",
        "Condition": {
          "ArnEquals": {
            "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<cloudfront-distribution-id>"
          }
        }
      }
    ]
  }
  ```

![Configure Bucket Policy for Frontend](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-frontend-policy.png)

#### 2. Application File Storage Bucket: `realtime-collab-files-<account-id>`
- **Block public access**: Block all public access **➔ Enabled**
- **Bucket Versioning**: Enabled
- **Encryption**: SSE-S3 (AES-256)

Additional configuration for the files bucket:
- **Bucket Policy** (Permissions tab ➔ Bucket policy):
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [{
      "Sid": "DenyNonSSL",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::realtime-collab-files-<account-id>",
        "arn:aws:s3:::realtime-collab-files-<account-id>/*"
      ],
      "Condition": { "Bool": { "aws:SecureTransport": "false" } }
    }]
  }
  ```

![Configure Bucket Policy for Files](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-files-policy.png)

- **Lifecycle rules** (Management tab ➔ Create lifecycle rule):
![Configure S3 Files CORS](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-files-cors.png)

  - **Rule 1**: Name `archive-workspace-files` | Prefix: `workspace-` | Action: Transition to Glacier storage after 90 days.

  ![Configure Lifecycle Rule 1 for Files - Part 1](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-files-lifecycle-rule-1.1.png)
  ![Configure Lifecycle Rule 1 for Files - Part 2](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-files-lifecycle-rule-1.2.png)

  - **Rule 2**: Name `abort-incomplete-multipart` | Prefix: Empty | Action: Abort incomplete multipart uploads after 7 days.

![Configure Lifecycle Rule 2 for Files - Part 1](/images/5-Workshop/5.5-S3-Buckets-IAM/abort-incomplete-multipart-1.png)
![Configure Lifecycle Rule 2 for Files - Part 2](/images/5-Workshop/5.5-S3-Buckets-IAM/abort-incomplete-multipart-2.png)

- **Cross-origin resource sharing (CORS)** (Permissions tab ➔ CORS):
  ```json
  [{
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
    "AllowedOrigins": ["*"],
    "MaxAgeSeconds": 3600
  }]
  ```

  ![Configure CORS for Files - Part 1](/images/5-Workshop/5.5-S3-Buckets-IAM/file-CORS-1.png)
  ![Configure CORS for Files - Part 2](/images/5-Workshop/5.5-S3-Buckets-IAM/file-CORS-2.png)

---

### PART 2 — IAM Roles configuration for ECS

To operate containers on ECS Fargate securely, we must configure two distinct IAM Roles:
1. **Task Execution Role**: Used by the AWS ECS Agent itself to pull Docker images from ECR, set up CloudWatch Log Streams, and retrieve database/API credentials from AWS Secrets Manager during container bootstrapping.
2. **Task Role**: Assumed directly by the running application containers (runtime) to interact with S3 Buckets or other AWS APIs.

Here are the step-by-step instructions to configure these roles via the AWS Console:

---

#### Step 1: Create ECS Task Execution Role (`ecsTaskExecutionRole-realtime-collab-dev`)

1. Open the AWS Console and search for the **IAM** service.
2. In the left navigation pane, select **Roles** ➔ click **Create role** on the top right.
3. In the creation wizard:
   - **Trusted entity type**: Select **AWS service**.
   - **Service or use case**: Choose **Elastic Container Service** from the dropdown.
   - **Use case**: Select **Elastic Container Service Task** (Ensures ECS Tasks can assume this role).
   - Click **Next**.

   ![Create ECS Task Execution Role - Part 1](/images/5-Workshop/5.5-S3-Buckets-IAM/create-role-1.png)
   
4. In the **Add permissions** screen:
   - Do not select any managed policies (we will add custom Inline Policies to adhere to the principle of least privilege).
   - Click **Next**.
5. In the **Name, review, and create** screen:
   - **Role name**: Enter exactly `ecsTaskExecutionRole-realtime-collab-dev`.
   - **Description**: Enter `ECS Task Execution Role for realtime-collab project`.
   - Click **Create role** at the bottom of the page.

![Create ECS Task Execution Role - Part 2](/images/5-Workshop/5.5-S3-Buckets-IAM/create-role-2.png)

6. Once created, search for `ecsTaskExecutionRole-realtime-collab-dev` in the Roles list and click on it.
7. Add the following **3 Inline Policies** by clicking the **Add permissions** dropdown ➔ Select **Create inline policy**:

![Create Inline Policy](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-exec-role-create.png)

##### ➕ Inline Policy 1: ECR Docker Image Pull permission (`ecr-pull`)
* Select the **JSON** editor tab and paste the following policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ECRAuthToken",
      "Effect": "Allow",
      "Action": ["ecr:GetAuthorizationToken"],
      "Resource": ["*"]
    },
    {
      "Sid": "ECRImagePull",
      "Effect": "Allow",
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ],
      "Resource": ["*"]
    }
  ]
}
```
![Configure Inline Policy ecr-pull](/images/5-Workshop/5.5-S3-Buckets-IAM/ecr-pull-policy.png)

* Click **Next**, name the policy `ecr-pull`, and click **Create policy**.

![Create Inline Policy ecr-pull](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-exec-policy-ecr.png)

##### ➕ Inline Policy 2: CloudWatch Log Streams permission (`cloudwatch-logs-ecs`)
* Click **Add permissions** ➔ **Create inline policy**, choose the **JSON** tab and paste:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "CloudWatchLogsECS",
    "Effect": "Allow",
    "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
    "Resource": ["arn:aws:logs:*:*:log-group:/ecs/*"]
  }]
}
```
![Configure Inline Policy cloudwatch-logs-ecs](/images/5-Workshop/5.5-S3-Buckets-IAM/cloudwatch-logs-ecs.png)

* Click **Next**, name the policy `cloudwatch-logs-ecs`, and click **Create policy**.

##### ➕ Inline Policy 3: AWS Secrets Manager Access permission (`secrets-execution`)
* Click **Add permissions** ➔ **Create inline policy**, choose the **JSON** tab and paste (Replace `<account-id>` with your 12-digit AWS Account ID):
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "SecretsManagerGetExecutionSecrets",
    "Effect": "Allow",
    "Action": ["secretsmanager:GetSecretValue"],
    "Resource": [
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/db-password*",
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/jwt-secret*",
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/twilio/api-credentials*",
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/redis-auth-token*"
    ]
  }]
}
```
![Create Inline Policy secrets-execution](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-exec-policy-secrets.png)

* Click **Next**, name the policy `secrets-execution`, and click **Create policy**.

---

#### Step 2: Create ECS Task Role (`ecsTaskRole-realtime-collab-dev`)

1. Go back to the **Roles** tab under the IAM console and click **Create role**.
2. Select the same trusted settings as in Step 1:
   - **Trusted entity type**: Select **AWS service**.
   - **Service or use case**: Choose **Elastic Container Service** ➔ **Elastic Container Service Task**.
   - Click **Next**.

   ![Create ECS Task Role - Part 1](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-task-role-create.png)

3. In the **Add permissions** screen: Click **Next** without selecting any policies.
4. In the **Name, review, and create** screen:
   - **Role name**: Enter exactly `ecsTaskRole-realtime-collab-dev`.
   - **Description**: Enter `ECS Task Role for realtime-collab project runtime`.
   - Click **Create role**.

![Create ECS Task Role - Part 2](/images/5-Workshop/5.5-S3-Buckets-IAM/ecsTaskRole-realtime-collab-dev.png)

5. Find and click on your newly created `ecsTaskRole-realtime-collab-dev` role.
6. Attach the following **2 Inline Policies** using **Add permissions** ➔ **Create inline policy**:

##### ➕ Inline Policy 1: S3 File Storage Operations permission (`s3-file-storage`)
* Select the **JSON** tab and paste (Replace `<account-id>` with your AWS Account ID):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ObjectOperations",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::realtime-collab-files-<account-id>/*"
    },
    {
      "Sid": "S3ListBucket",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::realtime-collab-files-<account-id>"
    }
  ]
}
```
![Create Inline Policy s3-file-storage](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-task-policy-s3.png)

* Click **Next**, name the policy `s3-file-storage`, and click **Create policy**.

##### ➕ Inline Policy 2: Twilio Credentials runtime access permission (`secrets-task`)
* Click **Add permissions** ➔ **Create inline policy**, choose the **JSON** tab and paste (Replace `<account-id>`):
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "SecretsManagerGetTwilio",
    "Effect": "Allow",
    "Action": ["secretsmanager:GetSecretValue"],
    "Resource": [
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/twilio/api-credentials*"
    ]
  }]
}
```
![Configure Inline Policy secrets-task](/images/5-Workshop/5.5-S3-Buckets-IAM/secrets-task.png)

* Click **Next**, name the policy `secrets-task`, and click **Create policy**.

