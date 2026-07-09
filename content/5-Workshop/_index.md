---
title: "Workshop"
date: 2024-06-29
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# AntiGroup Infrastructure Manual Deployment Guide

#### Overview

In this workshop, you will manually deploy the entire infrastructure and services for **AntiGroup - Real-time Streaming Collaboration Platform** using the AWS Management Console.

The goal of this lab is to help you understand how to configure each infrastructure component corresponding to the project's Terraform source code, establish network connections, configure security, and manage data flows securely.

#### Order of Execution (Mandatory)

The deployment must strictly follow the sequence below to ensure dependent resources are created in the correct order:

1. [Overview](5.1-Workshop-overview/)
2. [Secrets Manager](5.2-Secrets-Manager/)
3. [VPC & Networking](5.3-VPC-Network/)
4. [Security Groups](5.4-Security-Groups/)
5. [S3 Buckets & IAM Roles](5.5-S3-Buckets-IAM/)
6. [RDS PostgreSQL](5.6-RDS-Database/)
7. [ElastiCache Redis](5.7-ElastiCache-Redis/)
8. [ECR & Push Images](5.8-ECR-Push/)
9. [ALB & Target Groups](5.9-Load-Balancer/)
10. [CloudFront Distribution](5.10-CloudFront/)
11. [ECS Cluster & Services](5.11-ECS-Fargate/)
12. [Resource Cleanup](5.12-Cleanup/)