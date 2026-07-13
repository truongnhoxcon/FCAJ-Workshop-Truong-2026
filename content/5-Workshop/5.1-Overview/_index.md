---
title : "Introduction"
date : 2024-06-29
weight : 1 
chapter : false
pre : " <b> 5.1. </b> "
---

#### AntiCollab Routing Architecture

The **AntiCollab** platform is designed based on a Cloud Native and Microservices architecture running serverless containers on AWS Fargate. The network infrastructure is secured at multiple levels across Multi-AZ.

#### Network Architecture Diagram
The detailed architecture of the platform is described in the diagram below:

![AntiCollab AWS Architecture](/images/2-Proposal/Diagram.png)

#### Workshop Roadmap
In this workshop, we will go through the following configuration steps:
1. **AWS Cognito**: Create a user directory (User Pool) for user registration, login, and secure token validation.
2. **AWS Secrets Manager**: Securely store sensitive configurations (RDS password, JWT, Twilio credentials, Redis AUTH Token, and Cognito credentials).
3. **VPC & Networking**: Configure Multi-AZ Virtual Private Cloud, NAT Gateway, and S3 Gateway Endpoint.
4. **Security Groups**: Set up layered port security as virtual firewalls.
5. **S3 Buckets & IAM Roles**: Provision S3 storage buckets and configure execution roles (Task Role, Task Execution Role).
6. **RDS Database**: Provision Multi-AZ RDS PostgreSQL database instance.
7. **ElastiCache Redis**: Configure Multi-AZ ElastiCache Redis replication group for caching and pub/sub.
8. **ECR & Push Images**: Package Docker images and push them to ECR repositories.
9. **Application Load Balancer**: Create Target Groups and configure listener routing rules.
10. **CloudFront Distribution**: Set up CDN distribution for static hosting, route API/WS to ALB, and activate WAF in Monitor mode.
11. **ECS Fargate**: Register Task Definitions and deploy container services running 2 tasks.
12. **Resource Cleanup**: Step-by-step deletion guide to avoid unwanted charges.

*Architectural Flow (Routing Steps)*
The data flow and service interactions correspond to the numbered badges in the architecture diagram:
- **1. Authenticate (No. 1)**: Client (Frontend) sends login credentials directly to the **AWS Cognito User Pool**, which authenticates and returns a JSON Web Token (JWT).
- **2. Access Application (No. 2)**: User accesses the application via the secure HTTPS/WSS entry point on **CloudFront**, passing through the **AWS WAF** firewall (set to monitor/COUNT mode).
- **3. Serve Static Content (No. 3)**: CloudFront routes static asset requests via Origin Access Control (OAC) to the **S3 Frontend** bucket to serve the React application.
- **4. Dynamic Routing (No. 4)**: API calls (`/api/*`) or WebSocket handshakes (`/ws/*`) are forwarded by CloudFront to the **Application Load Balancer (ALB)**. Client includes the Cognito JWT in the `Authorization` header.
- **5. Backend Processing (No. 5)**: ALB routes the traffic to the active **ECS Tasks** in Private Subnets. Express routes and SocketAuth middleware validate and verify the JWT with AWS Cognito public keys. (Upon task startup, the container uses Task Execution Role to pull the Docker image from **Amazon ECR**, and uses Task Role to fetch credentials from **AWS Secrets Manager**).
- **6. Database & Cache Sync (No. 6)**: The task connects to **RDS PostgreSQL (Port 5432 SSL)** for persistent storage and **ElastiCache Redis (Port 6379 TLS)** as a Pub/Sub adapter to synchronize WebSocket events across containers.
- **7. File Upload (No. 7)**: ECS Task generates a Pre-signed URL for S3 upload. The Client uploads files directly to **S3 Files** bucket through the internal **VPC S3 Gateway Endpoint** to optimize security and speed.
- **8. WebRTC Signaling (No. 8)**: The task integrates with **Twilio API** to retrieve STUN/TURN configurations (ICE Servers) through NAT Gateway and Internet Gateway (IGW) for peer-to-peer audio/video streaming (WebRTC).