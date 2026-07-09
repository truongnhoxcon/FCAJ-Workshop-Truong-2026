---
title: "ECR & Push Docker Images"
date: 2024-06-29
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

We will create a private Docker image repository on Amazon ECR, build the unified backend container, and push it to AWS.

---

#### 1. Create ECR Repository

Navigate to the **Elastic Container Registry (ECR)** service ➔ **Repositories** ➔ **Create repository**. Create 1 repository:

- **Repository name**: `unified-backend`
- **Tag mutability**: Mutable

![Create ECR Repository Part 1](/images/5-Workshop/5.8-ECR-Push/ecr-create-1.png)

- **Scan on push**: **Enable**

![Create ECR Repository Part 2](/images/5-Workshop/5.8-ECR-Push/ecr-create-2.png)

---

#### 2. Configure ECR Lifecycle Policy

To automatically clean up old images and optimize storage costs, select the repository ➔ click **Lifecycle policy** in the left sidebar ➔ **Create rule**. Set up the following 2 rules:

![Configure Lifecycle Rule 1](/images/5-Workshop/5.8-ECR-Push/ecr-created.png)

##### Rule 1: Expire untagged images after 1 day
- **Rule priority**: `1`
- **Description**: `Expire untagged images after 1 day`
- **Image status**: Untagged
- **Match criteria**: Days since image created ➔ Days before action: 1

![Configure Lifecycle Rule 1](/images/5-Workshop/5.8-ECR-Push/ecr-lifecycle-1.png)

##### Rule 2: Keep only 30 most recent tagged images
- **Rule priority**: `2`
- **Description**: `Keep only 30 most recent tagged images`
- **Image status**: Tagged ➔ Prefix list: enter `v, latest, sha-`
- **Match criteria**: Image count ➔ Count number: 30
- **Action**: Expire

![Configure Lifecycle Rule 2 - Part 1](/images/5-Workshop/5.8-ECR-Push/ecr-lifecycle-2.png)
![Configure Lifecycle Rule 2 - Part 2](/images/5-Workshop/5.8-ECR-Push/ecr-lifecycle-3.png)

---

#### 3. Build & Push Docker Image to ECR

Open a terminal on your development machine (ensure Docker is running and AWS CLI is authenticated), navigate to the root directory of the `Real-time-Streaming-Collaboration` repository, and execute the following command:

```bash
bash deploy-ecr.sh us-east-1
```

This script will automatically:
1. Log in your local Docker client to the Amazon ECR registry.
2. Build the Docker image from the backend source code.
3. Push this image to ECR with the latest tag.

Once completed, you will see:

![ECR Repository Docker Image List](/images/5-Workshop/5.8-ECR-Push/ecr-images.png)
Copy this URI for the ECS Task Definitions configuration step.
