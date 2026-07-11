---
title: "Application Load Balancer"
date: 2024-06-29
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

We will provision an internet-facing Application Load Balancer (ALB) in the Public Subnets to receive client traffic and route it to the appropriate ECS Backend services.

---

#### 1. Create 2 Target Groups

Before creating the Load Balancer, we need to set up the Target Groups to register the IPs of our ECS tasks:

Navigate to **EC2** ➔ **Target Groups** (under the Load Balancing menu) ➔ **Create target group**. Create 2 Target Groups:

##### Target Group 1: `realtime-collab-dev-uni-api-tg` (For Unified Backend REST API)
- **Target type**: IP addresses
- **Target group name**: `realtime-collab-dev-uni-api-tg`
- **Protocol**: HTTP | **Port**: `3000`

![Create Target Group API - Configure Port](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-api-1.png)

- **VPC**: Select realtime-collab-dev
![Create Target Group API - Health Check Path](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-api-2.png)

- **Health checks**:
  - Health check path: `/health`
![Create Target Group API - Health Check Path](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-api-2.1.png)
  - Advanced settings: Interval: `15s` | Timeout: `5s` | Healthy: `2` | Unhealthy: `2` | Success: `200`
![Create Target Group API - Health Check Advanced](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-api-3.png)


##### Target Group 2: `realtime-collab-dev-uni-ws-tg` (For Unified Backend WebSocket)
- **Target type**: IP addresses
- **Target group name**: `realtime-collab-dev-uni-ws-tg`
- **Protocol**: HTTP | Port: `3000`

![Create Target Group WS - Configure Port](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-ws-1.png)

- **VPC**: Select realtime-collab-dev
![Create Target Group WS - Health Check Path](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-ws-2.png)

- **Health checks**:
  - Health check path: `/health`
![Create Target Group WS - Health Check Advanced](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-ws-3.png)
  - Advanced settings: Interval: `15s` | Timeout: `5s` | Healthy: `2` | Unhealthy: `2` | Success: `200`
![Create Target Group WS - Stickiness Enable](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-ws-4.png)


---

#### 2. Create the Application Load Balancer (ALB)

Navigate to **EC2** ➔ **Load Balancers** ➔ **Create Load Balancer** ➔ select **Application Load Balancer**:

- **Basic configuration**:
  - **Load balancer name**: `realtime-collab-dev-alb`
  - **Scheme**: Internet-facing
  - **IP address type**: IPv4

![Create ALB - Name and Scheme](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-1.png)

- **Network mapping**:
  - **VPC**: Select realtime-collab-dev
  - **Mappings**: Select 2 availability zones:
    - AZ 1 (us-east-1a) ➔ Subnet: public-subnet-1 (10.0.1.0/24)
    - AZ 2 (us-east-1b) ➔ Subnet: public-subnet-2 (10.0.2.0/24)

![Create ALB - Network Mappings Subnets](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-2.png)

- **Security groups**: Select alb-sg (Remove the default security group).
![Create ALB - Security Groups](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-3.png)

- **Listeners and routing**:
  - **Protocol**: HTTP | Port: `80`
![Create ALB - Security Groups](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-3.1.png)
  - **Default action**: Select **Return fixed response** | Response code: `404` | Content-type: text/plain | Response body: `Not Found` (Since the frontend will be served directly from CloudFront + S3, the ALB is only responsible for dynamic API/WS routing).
![Create ALB - Listeners default action 404](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-4.png)

Click **Create load balancer**.

---

#### 2.2 Configure Idle Timeout Attribute (Required for WebSockets)

Since the new AWS Console UI has removed **Attributes** configuration from the initial creation form, you must configure this attribute after the Load Balancer has been successfully created:

1. In the **Load balancers** list, check the box next to your newly created `realtime-collab-dev-alb` Load Balancer.
2. In the details panel below, navigate to the **Attributes** tab.
3. Click the **Edit** button on the right-hand side.
![Configure Idle Timeout for ALB - Attributes tab](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-idle-1.png)
4. Locate the **Connection idle timeout** setting and change its value from **60** to **3600** seconds (Required to keep WebSocket connections alive).
5. Click **Save changes**.
![Configure Idle Timeout for ALB - Change to 3600](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-idle-2.png)

6. Copy the **DNS name** of the ALB (e.g., `realtime-collab-dev-alb-XXXXXXXX.us-east-1.elb.amazonaws.com`) for the upcoming configuration steps.

---

#### 3. Add Advanced Routing Listener Rules

We need to segregate traffic: requests entering `/api/*` go to the Unified API Target Group, and WebSocket requests entering `/ws/*` go to the Unified WS Target Group:

1. Open the details page of the newly created Load Balancer ➔ Select **Listeners and rules** tab.
2. Click the listener **HTTP:80** to manage rules ➔ Click **Add rule**.

![Add Listener Rules for HTTP:80](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-1.png)

3. **Add Rule 1 — REST API**:
   - **Condition**: Select Path ➔ Enter `/api/*`
   ![Route Rule 1 REST API /api/*](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-2.png)
   - **Routing action**: Select Forward to Target Group: `realtime-collab-dev-uni-api-tg`
   ![Route Rule 1 REST API Forward](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-3.png)
   - **Priority**: `1` ➔ Press Next
   ![Route Rule 1 REST API Forward](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/rule-1.png)

4. **Add Rule 2 — WebSocket**:
   - **Condition**: Select Path ➔ Enter `/ws/*`
![Route Rule 2 WebSocket /ws/*](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-4.png)
   - **Routing action**: Select Forward to Target Group: `realtime-collab-dev-uni-ws-tg`
![Route Rule 2 WebSocket Forward](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-5.png)
   - **Priority**: `2` ➔ Press Next
![Listener Rules List configured successfully](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-6.png)
