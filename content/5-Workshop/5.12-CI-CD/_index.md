---
title: "CI/CD Setup"
date: 2024-06-29
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

In this step, we will set up an automated **CI/CD** (Continuous Integration and Continuous Deployment) pipeline using **GitHub Actions**. The objective is to automate the following workflow: build a new Docker image from the latest source code ➔ push it to Amazon ECR ➔ deploy the updated version to the Amazon ECS Fargate cluster whenever changes are pushed to the `main` branch.

---

#### 1. Create IAM User and Configure Permissions on AWS
GitHub Actions requires programmatic Access Keys to authenticate and securely interact with your AWS resources.

1. Navigate to the **IAM Console** ➔ **Users** ➔ Click **Create user**.
2. Settings:
   - **User name**: `github-actions-deployer` ➔ Click **Next**.
3. Under **Set permissions**:
   - Select **Attach policies directly**.
   - Search for and select the managed policy: **`AmazonEC2ContainerRegistryPowerUser`** (allows logging in and pushing Docker images to ECR).
   - Click **Create policy** to create a custom policy for ECS deployment permissions. Paste the following JSON configuration:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "ECSActions",
                 "Effect": "Allow",
                 "Action": [
                     "ecs:DescribeServices",
                     "ecs:UpdateService",
                     "ecs:RegisterTaskDefinition",
                     "ecs:DescribeTaskDefinition"
                 ],
                 "Resource": "*"
             },
             {
                 "Sid": "PassRole",
                 "Effect": "Allow",
                 "Action": [
                     "iam:PassRole"
                 ],
                 "Resource": [
                     "arn:aws:iam::<account-id>:role/ecsTaskExecutionRole-realtime-collab-dev",
                     "arn:aws:iam::<account-id>:role/ecsTaskRole-realtime-collab-dev"
                 ]
             },
             {
                 "Sid": "S3Actions",
                 "Effect": "Allow",
                 "Action": [
                     "s3:ListAllMyBuckets",
                     "s3:ListBucket",
                     "s3:PutObject",
                     "s3:GetObject",
                     "s3:DeleteObject"
                 ],
                 "Resource": "*"
             },
             {
                 "Sid": "CloudFrontActions",
                 "Effect": "Allow",
                 "Action": [
                     "cloudfront:ListDistributions",
                     "cloudfront:CreateInvalidation"
                 ],
                 "Resource": "*"
             }
         ]
     }
     ```
     *(Replace `<account-id>` with your 12-digit AWS Account ID)*
   - Name the policy `ECSDeployPolicy` and click **Create**. Return to the user creation screen, refresh, find and check `ECSDeployPolicy` ➔ Click **Next** ➔ **Create user**.
4. Go to the newly created `github-actions-deployer` User ➔ Select the **Security credentials** tab.
5. Scroll down to the **Access keys** section ➔ Click **Create access key**.
6. Select **Command Line Interface (CLI)** ➔ Click through to complete the setup.
7. **Securely store** the two provided values:
   - **Access Key ID**
   - **Secret Access Key**

---

#### 2. Configure Secrets in GitHub Repository
To keep your sensitive AWS credentials secure, we will store them as GitHub Secrets.

1. Open your project repository page on GitHub.
2. Navigate to **Settings** ➔ **Secrets and variables** (on the left menu) ➔ Select **Actions**.
3. Click the **New repository secret** button to add the following secrets:
   - **Secret 1**:
     - Name: `AWS_ACCESS_KEY_ID`
     - Secret: *(Paste the Access Key ID of the IAM User created in Step 1)*
   - **Secret 2**:
     - Name: `AWS_SECRET_ACCESS_KEY`
     - Secret: *(Paste the Secret Access Key of the IAM User created in Step 1)*

---

#### 3. Create GitHub Actions Workflow File
We will create a YAML configuration file to declare the automated steps in GitHub Actions.

1. In your local project source code directory, create the following directory structure and file:
   - `.github/workflows/deploy.yml`
2. Open `deploy.yml` and paste the following configuration:

```yaml
name: Deploy Backend to Amazon ECS

on:
  push:
    branches:
      - main  # Automatically triggers when code is pushed to the main branch

env:
  AWS_REGION: us-east-1                    # Replace with your AWS Region
  ECR_REPOSITORY: realtime-collab-api      # ECR Repository name created in Step 8
  ECS_CLUSTER: realtime-collab-cluster     # ECS Cluster name
  ECS_SERVICE: realtime-collab-service     # ECS Service name created in Step 11

jobs:
  deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest

    steps:
      # Step 1: Pull the latest source code from GitHub
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: Authenticate with AWS credentials using secrets
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      # Step 3: Log in to Amazon ECR
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      # Step 4: Build the Docker image, tag it, and push to ECR
      - name: Build, tag, and push image to Amazon ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:${{ github.sha }} -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:${{ github.sha }}
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

      # Step 5: Instruct Amazon ECS to trigger a new deployment (pulls latest tag)
      - name: Force New ECS Deployment
        run: |
          aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment
```

---

#### 4. Push Code & Verify the CI/CD Pipeline
1. Save all configuration files and push the code changes to the `main` branch:
   ```bash
   git add .
   git commit -m "Set up automated GitHub Actions CI/CD for Amazon ECS"
   git push origin main
   ```
2. Go to your repository on GitHub ➔ Select the **Actions** tab.
3. You will see a workflow run named *"Deploy Backend to Amazon ECS"* executing automatically.
4. Click on the workflow to inspect the build details and logs. Once all steps turn green and succeed:
   - AWS ECS will trigger a rolling update, launching new tasks with the updated code and stopping the old tasks safely with zero downtime.
