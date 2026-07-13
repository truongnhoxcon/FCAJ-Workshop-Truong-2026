---
title: "Workshop"
date: 2024-06-29
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# AntiCollab Infrastructure Manual Deployment Guide

#### Overview

In this workshop, you will manually deploy the entire infrastructure and services for **AntiCollab - Real-time Streaming Collaboration Platform** using the AWS Management Console.

The goal of this lab is to help you understand how to configure each infrastructure component corresponding to the project's Terraform source code, establish network connections, configure security, and manage data flows securely.

#### Order of Execution (Mandatory)

The deployment must strictly follow the sequence below to ensure dependent resources are created in the correct order:

1. [Overview](5.1-Overview/)
2. [Network & Security](5.2-Network-Security/)
   - [AWS Cognito User Pool](5.2-Network-Security/5.2.1-Cognito-Auth/)
   - [Secrets Manager](5.2-Network-Security/5.2.2-Secrets-Manager/)
   - [VPC & Networking](5.2-Network-Security/5.2.3-VPC-Network/)
   - [Security Groups](5.2-Network-Security/5.2.4-Security-Groups/)
3. [Data & Storage](5.3-Data-Storage/)
   - [S3 Buckets & IAM Roles](5.3-Data-Storage/5.3.1-S3-Buckets-IAM/)
   - [RDS PostgreSQL](5.3-Data-Storage/5.3.2-RDS-Database/)
   - [ElastiCache Redis](5.3-Data-Storage/5.3.3-ElastiCache-Redis/)
4. [App Deployment & Delivery](5.4-Deployment-Delivery/)
   - [ECR & Push Images](5.4-Deployment-Delivery/5.4.1-ECR-Push/)
   - [ALB & Target Groups](5.4-Deployment-Delivery/5.4.2-Load-Balancer/)
   - [CloudFront Distribution](5.4-Deployment-Delivery/5.4.3-CloudFront/)
   - [ECS Cluster & Services](5.4-Deployment-Delivery/5.4.4-ECS-Fargate/)
5. [Resource Cleanup](5.5-Cleanup/)