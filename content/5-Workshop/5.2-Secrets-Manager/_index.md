---
title: "AWS Secrets Manager"
date: 2024-06-29
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

To secure passwords and API Keys from our source code, we will start by creating centralized secret storage.

#### Create 4 Secrets on AWS Secrets Manager

Navigate to **AWS Secrets Manager** and click **Store a new secret**. Set all of these secrets with a **Recovery window: 7 days** to minimize testing costs.

---

##### 1. `realtime-collab-dev/db-password` (Database Password)
- **Type**: Other type of secret ➔ tab `Plaintext`
- **Value**: Enter a random 32-character string
- **Secret name**: `realtime-collab-dev/db-password`
- **Description**: RDS PostgreSQL master password

![Select type and value for db-password](/images/5-Workshop/5.2-Secrets-Manager/db-password-1.png)
![Configure name and description for db-password](/images/5-Workshop/5.2-Secrets-Manager/db-password-2.png)

##### 2. `realtime-collab-dev/jwt-secret` (JWT Secret Token)
- **Type**: Other type of secret ➔ tab `Key/value`
- **Key/Value**:
  - Key: `secret`
  - Value: [A random string of at least 64 characters]
- **Secret name**: `realtime-collab-dev/jwt-secret`

![Select type and value for jwt-secret](/images/5-Workshop/5.2-Secrets-Manager/jwt-secret-1.png)
![Configure name for jwt-secret](/images/5-Workshop/5.2-Secrets-Manager/jwt-secret-2.png)

##### 3. `realtime-collab-dev/redis-auth-token` (Redis Authentication Token)
- **Type**: Other type of secret ➔ tab `Plaintext`
- **Value**: [A random 32-character string]
- **Secret name**: `realtime-collab-dev/redis-auth-token`

![Select type and value for redis-auth-token](/images/5-Workshop/5.2-Secrets-Manager/redis-token-1.png)
![Configure name for redis-auth-token](/images/5-Workshop/5.2-Secrets-Manager/redis-token-2.png)

##### 4. `realtime-collab-dev/twilio/api-credentials` (Twilio TURN Config)
- **Type**: Other type of secret ➔ tab `Key/value`
- **Key/Value**:
  - Key `accountSid` ➔ Value: [Twilio Account SID]
  - Key `authToken` ➔ Value: [Your Twilio Auth Token]
  - Key `apiKeySid` ➔ Value: [Your Twilio API Key SID]
  - Key `apiKeySecret` ➔ Value: [Your Twilio API Key Secret]
- **Secret name**: `realtime-collab-dev/twilio/api-credentials`

![Select type and values for twilio credentials](/images/5-Workshop/5.2-Secrets-Manager/twilio-credentials-1.png)
![Configure name for twilio credentials](/images/5-Workshop/5.2-Secrets-Manager/twilio-credentials-2.png)

---

After creating the secrets, make sure to record the **ARN** of each secret. You will need them for configuring IAM Roles and ECS Task Definitions.
