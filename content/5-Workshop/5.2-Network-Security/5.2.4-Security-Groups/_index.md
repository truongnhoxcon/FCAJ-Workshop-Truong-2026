---
title: "Security Groups"
date: 2024-06-29
weight: 4
chapter: false
pre: " <b> 5.2.4. </b> "
---

We will configure Security Groups to act as virtual firewalls to control traffic for each layer of services.

#### 1. Initialize 4 Security Groups

Navigate to the **VPC** service ➔ **Security Groups** and click **Create security group**. Create 4 empty security groups inside the realtime-collab-dev VPC:

| Security Group Name | Description |
|---|---|
| `alb-sg` | ALB: Accepts HTTP port 80 from the internet |
| `unified-backend-sg` | ECS Unified Backend: Accepts port 3000 from ALB |
| `rds-sg` | RDS PostgreSQL: Accepts port 5432 from ECS Backend |
| `redis-sg` | ElastiCache Redis: Accepts port 6379 from ECS Backend |

---

#### 2. Configure Inbound and Outbound Rules

For each security group, edit the **Inbound rules** and **Outbound rules** by selecting the group and clicking **Edit inbound/outbound rules**:

##### 2.1 Rules for alb-sg
- **Inbound Rules**:
  - Type: HTTP | Port: 80 | Source: `0.0.0.0/0` | Description: `HTTP from internet`
- **Outbound Rules**:
  - Type: All traffic | Destination: `0.0.0.0/0`

![Configure inbound rule for alb-sg](/images/5-Workshop/5.2-Network-Security/5.2.3-Security-Groups/alb-sg.png)

##### 2.2 Rules for unified-backend-sg
- **Inbound Rules**:
  - Type: Custom TCP | Port: `3000` | Source: select alb-sg | Description: `REST API and WS from ALB`
- **Outbound Rules**:
  - Type: All traffic | Destination: `0.0.0.0/0`

![Configure inbound rule for unified-backend-sg](/images/5-Workshop/5.2-Network-Security/5.2.3-Security-Groups/unified-backend-sg.png)

##### 2.3 Rules for rds-sg
- **Inbound Rules**:
  - Type: PostgreSQL | Port: `5432` | Source: select backend-sg

![Configure inbound rule for rds-sg](/images/5-Workshop/5.2-Network-Security/5.2.3-Security-Groups/rds-sg.png)

##### 2.4 Rules for redis-sg
- **Inbound Rules**:
  - Type: Custom TCP | Port: `6379` | Source: select backend-sg

![Configure inbound rule for redis-sg](/images/5-Workshop/5.2-Network-Security/5.2.3-Security-Groups/redis-sg.png)

