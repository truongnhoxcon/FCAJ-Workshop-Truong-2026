---
title : "Blog 1"
date : 2024-01-01
weight : 3
chapter : false
pre : "<b>3.1</b>"
---

# Automated Deployments to Amazon ECS Express Mode via GitHub Actions

Hello everyone,

This post summarizes and shares a technical solution for optimizing the CI/CD pipeline for containerized applications using **GitHub Actions** and **Amazon ECS Express Mode**. This approach focuses on two primary objectives: streamlining operations and hardening security patterns.

---

## 1. Core Advantages of the Solution

- **Security via OpenID Connect (OIDC):** Instead of relying on static, long-lived credentials like `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` stored in GitHub Secrets, this solution establishes a trust relationship between GitHub and AWS through OIDC. GitHub Actions dynamically assumes a short-lived IAM Role to execute tasks, strictly adhering to the principle of Least Privilege.
- **Optimized Deployment with ECS Express Mode:** The Express Mode feature in **Amazon ECS** simplifies infrastructure management and automates load balancing. When paired with the official AWS *"Deploy Express Service"* Action, service rolling updates are executed rapidly and reliably.

---

## 2. Workflow Overview

Whenever a code push or Pull Request merge occurs on the default branch (e.g., `main`):

1. **GitHub Actions** triggers the runner pipeline.
2. The runner builds the `Dockerfile` into a container image and pushes it to **Amazon Elastic Container Registry (ECR)**.
3. The *"Deploy Express Service"* Action updates the task configuration and shifts traffic to the newly built image version running on **Amazon ECS Express Mode**.

---

## 3. Key Implementation Steps

* **Step 1:** Configure the OIDC Identity Provider within **AWS IAM**, mapping it to the GitHub token issuer URL (`token.actions.githubusercontent.com`).
* **Step 2:** Create a dedicated **IAM Role** with minimal policies, granting scoped permissions exclusively for pushing images to **Amazon ECR** and updating services on **Amazon ECS Express Mode**.
* **Step 3:** Leverage **GitHub Repository Variables** for non-sensitive configuration parameters (such as `AWS_REGION`, `ECR_REPOSITORY`, `ECS_SERVICE_NAME`) to simplify workflow maintenance.
* **Step 4:** Define the complete workflow automation file under the path `.github/workflows/deploy.yml`.

---

## 4. Conclusion

Integrating **GitHub Actions** with **Amazon ECS Express Mode** via OIDC authentication delivers a comprehensive CI/CD solution that maximizes automated deployment velocity while satisfying rigorous enterprise security compliance. 

This architectural paradigm entirely mitigates the risks associated with static credentials, shrinks the attack surface, and demystifies container infrastructure operations. It serves as an effective reference model for engineering secure software supply chains on top of the AWS ecosystem.

> **Original Post:** [AWS Containers Blog - Automated deployments with GitHub Actions for Amazon ECS Express Mode](https://aws.amazon.com/vi/blogs/containers/automated-deployments-with-github-actions-for-amazon-ecs-express-mode/)

![ConnectPrivate](/images/arcblog1.png)