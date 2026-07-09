---
title: "Week 4 Worklog"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Week 4 Objectives:

* Understand IAM, Cognito, SSO, and AWS Organizations.

### Tasks to be carried out this week:
| Day | Task | Start Date | Completion Date | Resource Reference |
| --- | --- | --- | --- | --- |
| 2   | - Learn theory of IAM, Cognito, SSO, and Organizations. <br> - Learn about AWS KMS and AWS Security Hub. <br> - Enable AWS Security Hub, assess security score | 11/05/2026 | 11/05/2026 | <https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i> <br> <https://000018.awsstudygroup.com/> |
| 3   | - Manage tags in-depth on Console and CLI, using Resource Groups. <br> - Initialize VPC, Security Group, EC2 and set up Webhook to send notifications to Slack. <br> - Write IAM Role for Lambda. Create Lambda functions to automatically Stop and Start EC2 | 12/05/2026 | 12/05/2026 | <https://000027.awsstudygroup.com/> <br> <https://000022.awsstudygroup.com/> |
| 4   | - Create IAM User, set up Policy/Role and test the Switch Roles feature. <br> - Access multi-Region console, check Policy enforcement when valid Tags are missing | 13/05/2026 | 13/05/2026 | <https://000028.awsstudygroup.com/> |
| 5   | - Create Restriction Policy, IAM Limited User account and test restrictions. <br> - Create Policy, Group, S3 Bucket and encrypt with AWS KMS. Set up AWS CloudTrail and use Amazon Athena to query logs. <br> - Perform check and share encrypted data on S3 storage | 14/05/2026 | 14/05/2026 | <https://000030.awsstudygroup.com/> <br> <https://000033.awsstudygroup.com/> |
| 6   | - Initialize IAM Group, IAM Users, test permissions. Create Admin IAM Role and switch roles, limit Switch role by IP and time, clean up resources. <br> - Initialize EC2, S3 bucket, create new IAM user and Access Key. Use Access Key and the more secure solution of assigning IAM Role to EC2 | 15/05/2026 | 15/05/2026 | <https://000044.awsstudygroup.com/> <br> <https://000048.awsstudygroup.com/> |

### Week 4 Achievements:

* Established tight security boundaries using IAM, knowing how to limit permissions and control resource creation using Tagging.
* Saved cloud costs by automating server start/stop schedules using AWS Lambda combined with Slack alerts.
* Successfully configured advanced security constraints, only allowing users to switch roles when accessing from a specific IP or within a predetermined time window.
* Understood the difference between granting permissions directly using Access Keys versus using safer IAM Roles to allow EC2 to interact with other services.
