---
title : "Introduction"
date : 2024-06-29
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### AntiGroup Routing Architecture

The **AntiGroup** platform is designed based on a Cloud Native and Microservices architecture running serverless containers on AWS Fargate. The network infrastructure is secured at multiple levels across Multi-AZ.

#### Infrastructure Architecture Diagram
The detailed architecture is represented by the following diagram:

![AntiGroup AWS Architecture](/images/2-Proposal/Diagram.png)

#### Workshop Roadmap
In this workshop, we will go through the following configuration steps:
1. **AWS Secrets Manager**: Securely store sensitive environment variables.
2. **VPC & Networking**: Configure Multi-AZ Virtual Private Cloud, NAT Gateway, and S3 Gateway Endpoint.
3. **Security Groups**: Set up layered port security.
4. **S3 Buckets & IAM Roles**: Provision S3 storage and execution roles.
5. **RDS Database**: Provision Multi-AZ RDS PostgreSQL database instance.
6. **ElastiCache Redis**: Configure Multi-AZ ElastiCache Redis replication group.
7. **ECR & Push Images**: Build and push Unified Backend container images to ECR.
8. **Application Load Balancer**: Create Target Groups and configure ALB listener rules.
9. **CloudFront Distribution**: Set up global HTTPS Ingress and integrate AWS WAF.
10. **ECS Fargate**: Register Task Definitions and deploy ECS Services with 2 tasks.
11. **Resource Cleanup**: Step-by-step deletion guide to avoid unwanted charges.