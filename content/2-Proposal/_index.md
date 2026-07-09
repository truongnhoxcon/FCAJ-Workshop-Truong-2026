---
title: "Project Proposal"
date: 2024-06-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

#### AntiGroup Real-time Chat & Streaming Platform
*Technical Proposal & Cloud Infrastructure Design on Amazon Web Services*

---

### 1. Executive Summary
The **AntiGroup** project aims to build a Discord-like real-time community collaboration platform, allowing users to exchange text messages, check online presence, and initiate voice/video group calls directly. The platform is tailored for students and researchers to collaborate within laboratory environments.

To guarantee high availability, low messaging latency (under 200ms), and elastic scalability without excessive upfront hardware costs, the solution is designed to run natively on **Amazon Web Services (AWS)**.

---

### 2. Solution Goals
This technical proposal focuses on resolving the following infrastructure challenges:
- **Low Latency Messaging**: Utilizing WebSockets backed by Redis Pub/Sub adapter to sync socket events across containers, ensuring text messages and presence states are updated immediately.
- **Optimized Voice/Video Calling**: Setting up WebRTC for direct Peer-to-Peer (P2P) media delivery, and configuring a TURN relay server (using Twilio) when direct connection fails due to symmetric NAT/firewalls.
- **Rapid Infrastructure Prototyping**: Utilizing Terraform configs to quickly model and test basic network infrastructure compatibility before manual Console deployment.
- **Unified App Architecture**: Merges REST API and WebSocket events into a single unified backend service to reduce compute resource overhead.
- **Redis Pub/Sub Sync**: ElastiCache Redis is utilized as a pub/sub adapter to sync socket events across containers, ensuring real-time consistency.
- **S3 Static Frontend & Pre-signed Uploads**: Frontend SPA assets are hosted on S3 and distributed via CloudFront CDN. Dynamic file uploads use S3 Pre-signed URLs to upload files directly, bypassing backend compute entirely.
- **Serverless Containers**: AWS ECS Fargate runs backend containers and scales them automatically using Service Auto Scaling policy without needing server provisioning.
- **Cost-effective Security**: Terminates SSL/TLS globally through CloudFront CDN at minimal costs in stage/dev environments without buying separate certificates.

---

### 3. Solution Architecture
The AWS infrastructure design is detailed below:

![AntiGroup AWS Architecture](/images/2-Proposal/Diagram.png)

*AWS Services Used & Data Flow*
- **Amazon VPC & Subnets (Multi-AZ)**: Secure Multi-AZ network topology separated into Public Subnets (hosting ALB, NAT Gateway) and Private Subnets (hosting ECS Fargate Tasks, RDS PostgreSQL, and ElastiCache Redis).
- **Amazon CloudFront & AWS WAF**: Operates as a secure global HTTPS/WSS Ingress point protected by AWS WAF to filter malicious traffic. Distributes static frontend from S3 via OAC and routes dynamic API/WebSocket traffic to the ALB.
- **Application Load Balancer (ALB)**: Receives traffic from CloudFront and routes /api/* to the REST API Target Group (Round-robin) and /ws/* to the WebSocket Target Group (Sticky sessions) of the ECS Service.
- **Amazon ECS (Fargate)**: Runs serverless docker containers for the Unified Backend (Node.js Express & Socket.io on port 3000) running across 2 Availability Zones (Task 1 and Task 2) under a single ECS Service for high availability and automatic failover.
- **Amazon RDS (PostgreSQL)**: The primary relational database running in Multi-AZ configuration (synchronous replication between Master DB in AZ1 and Standby DB in AZ2) storing users, messages, servers, and channels.
- **Amazon ElastiCache (Redis)**: Highly available Redis cache running in Multi-AZ (asynchronous replication from Primary node in AZ1 to Replica node in AZ2) used as a Redis Pub/Sub adapter to sync socket events across containers and cache user presence states.
- **Amazon S3 & VPC Gateway Endpoint**: Includes S3 Frontend for static web hosting and S3 Files for attachments. ECS Tasks connect to S3 Files via internal VPC S3 Gateway Endpoint to optimize network performance and security.
- **AWS Secrets Manager & CloudWatch**: Securely stores credentials (RDS password, JWT, Twilio API keys, Redis AUTH Token) injected into tasks at startup. Outbound application logs are streamed to CloudWatch Logs.
- **NAT Gateway & Internet Gateway (IGW)**: Provides one-way outbound internet routing for private subnets to pull images from ECR, retrieve secrets, send logs, and fetch WebRTC ICE/TURN tokens from Twilio APIs.
- **AWS IAM**: Enforces least privilege access using ecsTaskRole (access to S3, Secrets Manager) and ecsTaskExecutionRole (ECR image pulls, CloudWatch logs) to maintain security boundaries.

*Component Design*
- **Client (Frontend)**: ReactJS application hosted statically on S3 using useWebSocket.js for Socket.io message exchange and presence updates, and useWebRTC.js for WebRTC calls.
- **Unified Backend Service**: Merges REST API (Node.js/Express) and WebSockets/Signaling (Node.js/Socket.io) into a single container. Accesses PostgreSQL via connection pool, handles Redis Pub/Sub events, processes CRUD requests, generates S3 pre-signed upload URLs, and issues JWT tokens.

---

### 4. Technical Implementation
*Implementation Phases*
The project was executed by the team in 4 distinct phases:
1. **Phase 1: Requirements Analysis & App Development (Weeks 7 - 9)**: Team members analyzed the real-time chat and video calling specifications; developed the ReactJS frontend replicating a Discord-like UI and the Node.js backend (Express and Socket.io); and integrated local PostgreSQL and Redis databases.
2. **Phase 2: Architecture Design & Local Verification (Week 10)**: Designed the AWS Architecture Diagram on draw.io, performed end-to-end local integration testing using Docker Compose, and developed Terraform scripts to model and verify basic cloud networks.
3. **Phase 3: Console Infrastructure Deployment (Week 11)**: Manually configured and provisioned the production cloud infrastructure (VPC Multi-AZ, Secrets Manager, Security Groups, S3, RDS PostgreSQL Multi-AZ, ElastiCache Redis Multi-AZ, ALB, CloudFront + WAF, and ECS Cluster/Service Fargate) on the AWS Console; built container images and pushed to ECR.
4. **Phase 4: System Integration Testing & Handover (Week 12)**: Executed system integration tests on AWS; validated WebSocket messaging, WebRTC group calling (via Twilio), and S3 file attachments; audited error logs via CloudWatch Logs; completed self-evaluation reports, and cleaned up AWS resources.

*Technical Requirements*
- **Environment**: Docker Desktop locally. AWS CLI v2 configured. Terraform >= 1.6 for IaC deployment.
- **Frontend**: NodeJS >= 18 for ReactJS building, Socket.io-client for websockets, Lucide React for UI icons.
- **Backend**: NodeJS >= 18, PostgreSQL relational DB, Redis for cache & pub/sub, Twilio API keys for TURN server.

---

### 5. Roadmap & Milestones
The team's project roadmap is divided into the following milestones:
- **Milestone 1 (Weeks 7 - 9)**:
  - Design relational database schemas and finalize code for Unified Backend and React Frontend.
  - Run local end-to-end developer testing and resolve layout/logic issues.
- **Milestone 2 (Week 10)**:
  - Sketch the detailed AWS network architecture diagram on draw.io.
  - Complete local application stack integration testing via Docker Compose.
  - Write Terraform scripts to test and validate network and routing structures.
- **Milestone 3 (Week 11)**:
  - Provision VPC, Secrets Manager, Security Groups, S3, and IAM Roles on the AWS Console.
  - Deploy highly available Multi-AZ database engines (RDS PostgreSQL and ElastiCache Redis).
  - Wrap backend services into Docker images, push to ECR, and configure target groups, ALB, CloudFront + WAF, and ECS Fargate.
- **Milestone 4 (Week 12)**:
  - Perform live environment integration testing for WebSockets and WebRTC group calling.
  - Complete the internship technical report, self-evaluation, and delete sandbox resources.

---

### 6. Budget Estimation
Estimated infrastructure costs in us-east-1:

*Monthly Estimated Infrastructure Costs*
- **Amazon ECS Fargate**: 2 tasks running continuously (0.5 vCPU, 1 GB RAM) **$18.00/month**.
- **Amazon RDS (PostgreSQL)**: Instance class db.t3.medium, Multi-AZ, 20 GB GP3 storage **$30.00/month**.
- **Amazon ElastiCache (Redis)**: Instance class cache.t3.micro Multi-AZ (1 replica) for pub/sub sync **$24.00/month**.
- **Application Load Balancer (ALB)**: 1 ALB for backend traffic routing **$22.26/month**.
- **Amazon CloudFront**: Caches static assets & terminates SSL (Free Tier includes 1 TB outbound bandwidth) **$2.00/month** (when exceeding free tier).
- **Amazon S3**: File storage & Frontend static hosting (< 50 GB) **$1.20/month**.
- **AWS Secrets Manager**: Stores 4 secret keys **$1.60/month** ($0.40/secret).
- **AWS CloudWatch**: Collects log streams and metric histories **$3.00/month**.
- **Network / Bandwidth**: Data transfer to Internet **$2.00/month**.

**Total Monthly Infrastructure Cost**: **$104.06/month**

---

### 7. Risk Assessment
*Risk Matrix*
- **Risk 1: WebRTC direct P2P call fails due to symmetric NAT/firewalls.**
  - *Impact*: High.
  - *Mitigation*: Fall back to Twilio TURN relay server, tunneling media streams over TCP/UDP.
- **Risk 2: Chat message out-of-order or offline states out-of-sync during backend scaling.**
  - *Impact*: Medium.
  - *Mitigation*: Redis Pub/Sub adapter coordinates message/presence events across instances in real time using presence.handler.js and chat.handler.js.
- **Risk 3: AWS cost overrun exceeding development budget.**
  - *Impact*: Medium.
  - *Mitigation*: Set AWS Budget Alerts at $50 and $80 thresholds. Script auto-stop scripts for ECS Tasks during off-hours.
- **Risk 4: API Keys or DB passwords leaked to public repositories.**
  - *Impact*: Critical.
  - *Mitigation*: Inject secret environment variables at runtime from AWS Secrets Manager; never store passwords in source code.

---

### 8. Expected Outcomes
*Technical Enhancements*
- E2E real-time community chat platform with text message delivery latencies under 200ms.
- Stable WebRTC audio/video group calls.
- Fast prototyping and baseline network design saved as Terraform scripts for easy reference.

*Long-term Values*
- A solid reference architecture for serverless WebSockets, WebRTC signaling, and S3-based static web hosting on AWS.
- Standardized manual and helper-script-based deployment process using ECR pushes and container deployment.