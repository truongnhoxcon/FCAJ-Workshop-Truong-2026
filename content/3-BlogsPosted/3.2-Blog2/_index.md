---
title : "Blog 2"
date : 2024-01-01
weight : 4
chapter : false
pre : "<b>3.2</b>"
---

# Deploying Internal DNS for Internet-facing Load Balancers on AWS via Event-driven Architecture

Hello everyone,

During my research and hands-on experience with networking concepts on AWS, **Elastic Load Balancing (ELB)** stands out as a foundational service. However, when provisioning internet-facing load balancers, there is a specific Domain Name System (DNS) resolution constraint that requires careful consideration.

---

## 1. Problem Statement

When deploying a **Network Load Balancer (NLB)** or an **Application Load Balancer (ALB)** in *Internet-facing* mode, AWS automatically provisions a randomized, public DNS host name (e.g., `nlb-123...elb.us-east-1.amazonaws.com`).

By design, this canonical name **only resolves to Public IP addresses**. AWS does not natively offer out-of-the-box DNS records that map to the underlying Private IPs of that specific load balancer. This limitation introduces architectural blockers in several enterprise patterns:
- When implementing third-party firewall gateways inline between the ALB/NLB and the Internet Gateway, requiring internal traffic inspection mapped to local DNS records.
- When routing ingress traffic directly from on-premises environments into the Load Balancer via **AWS Direct Connect** circuits.

---

## 2. Solution Overview

To address this challenge, we can implement an automated framework to dynamically provision and govern **Private Hosted Zones** inside **Amazon Route 53**. The core purpose of this system is to maintain and continuously update local DNS records pointing to the Private IPs of Internet-facing Load Balancers.

This solution is purpose-built for enterprise Multi-account environments governed by **AWS Organizations**, orchestrating the collective capabilities of **AWS CloudTrail**, **Amazon EventBridge**, **AWS Lambda**, and **Amazon DynamoDB**.

---

## 3. Deep-Dive Deployment Architecture

The structural architecture is segregated into two primary layers deployed across distinct AWS accounts:

### A. Shared Services Account (Centralized Management)
- **Amazon Route 53:** Hosts 2 Private Hosted Zones per target AWS Region:
    - ALB target suffix: `internal.region.elb.amazonaws.com`
    - NLB target suffix: `internal.elb.region.amazonaws.com`
- **AWS Lambda:** Houses 2 functional compute workers:
    - `r53-scavenger`: A time-triggered cron job tasked with sweeping existing ALB/NLB Private IPs and seeding them into Route 53 during initial discovery.
    - `r53-updater`: An event-reactive function that captures real-time lifecycle changes to guarantee continuous DNS record accuracy.
- **Amazon DynamoDB:** Serves as the state store cache for active load balancers and Elastic Network Interfaces (ENIs), drastically minimizing cross-account API throttling and optimizing overall lookups.
- **Amazon EventBridge Custom Bus:** Acts as the central ingestion hub receiving lifecycle event payloads routed from standard workload accounts, immediately invoking the `r53-updater` Lambda function.

### B. Workload Accounts (Application Layers)
- Houses a cross-account **IAM Role** granting scoped read access for the central `r53-scavenger` Lambda to inventory infrastructure states.
- Configures an **EventBridge Rule** to intercept raw API auditing logs emitted from CloudTrail and forward them directly to the central Custom Bus in the Shared Services account.

---

## 4. Automated Event-driven Workflow

Once deployed, the synchronization workflow triggers completely out-of-band via an event-driven sequence:

```text
[User Action / System Scaling]
               │
               ▼
   [Captured by AWS CloudTrail] (APIs: CreateLoadBalancer, CreateNetworkInterface, etc.)
               │
               ▼
[EventBridge Rule at Workload Account] ──(Forwarded)──> [Custom Bus at Shared Services]
                                                                     │
                                                                     ▼
                                                          [Lambda r53-updater]
                                                                     │
                                                     ┌───────────────┴───────────────┐
                                                     ▼                               ▼
                                         [Update DynamoDB Cache]         [Sync Route 53 DNS Record]

---
```
> **Original Post:** [AWS Networking & Content Delivery Blog - Deploying internal DNS zones for internet-facing load balancers](https://aws.amazon.com/blogs/networking-and-content-delivery/deploying-internal-dns-zones-for-internet-facing-load-balancers/)

![ConnectPrivate](/images/arcblog2.png)