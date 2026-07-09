---
title: "VPC & Networking"
date: 2024-06-29
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

We will construct a secure Multi-AZ virtual network containing Public Subnets for the Load Balancer and Private Subnets to isolate our application services.

#### 1. Create VPC

Navigate to the **VPC** service on the AWS Management Console, click **Create VPC**, and select **VPC and more** (automated Wizard). Configure the details as follows:

- **Name tag auto-generation**: `realtime-collab-dev`
- **IPv4 CIDR block**: `10.0.0.0/16`
- **Tenancy**: Default
  ![VPC Wizard configuration part 1](/images/5-Workshop/5.3-VPC-Network/vpc-wizard-1.png)
- **Number of Availability Zones (AZs)**: 2 (`us-east-1a`, `us-east-1b`)
- **Number of Public Subnets**: 2
  - Public subnet 1 CIDR block: `10.0.1.0/24` (AZ 1)
  - Public subnet 2 CIDR block: `10.0.2.0/24` (AZ 2)
- **Number of Private Subnets**: 2
  - Private subnet 1 CIDR block: `10.0.10.0/24` (AZ 1)
  - Private subnet 2 CIDR block: `10.0.11.0/24` (AZ 2)
  
![VPC Wizard configuration part 2](/images/5-Workshop/5.3-VPC-Network/vpc-wizard-2.png)

- **NAT Gateways**: Regional
- **VPC Endpoints**: Check S3 Gateway
- **DNS Options**:
  - Enable DNS hostnames ➔ **Check**
  - Enable DNS resolution ➔ **Check**

![VPC Wizard configuration part 3](/images/5-Workshop/5.3-VPC-Network/vpc-wizard-3.png)

Click **Create VPC**. The Wizard will automatically create the Internet Gateway, Route Tables (Public & Private), NAT Gateway, and Elastic IP.

---

#### 2. Enable VPC Flow Logs

To record all IP traffic history within the VPC for monitoring and security auditing:

1. Go to the details page of the newly created VPC.
2. Select the **Flow logs** tab and click **Create flow log**.

![Select Flow logs tab and click Create flow log](/images/5-Workshop/5.3-VPC-Network/flow-logs-1.png)

3. Configure the following parameters:
   - **Filter**: All
   - **Maximum aggregation interval**: 1 minute
   - **Destination**: Send to CloudWatch Logs
   - **Destination log group**: Select or create a log group named `/aws/vpc/flow-logs/realtime-collab-dev`
   - **Service access**: Select **Create and use a new service role** so AWS automatically generates the IAM Role with log writing permissions.

![VPC Flow logs configuration details](/images/5-Workshop/5.3-VPC-Network/flow-logs-2.png)

4. Click **Create flow log**.
