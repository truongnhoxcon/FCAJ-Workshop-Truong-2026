---
title: "AWS Cognito User Pool"
date: 2024-06-29
weight: 4
chapter: false
pre: " <b> 5.2.4. </b> "
---

We will provision an **Amazon Cognito User Pool** to manage user registration, login, email verification, and secure access tokens without managing server-side database passwords.

---

#### Step 1: Create Cognito User Pool

1. Navigate to **Amazon Cognito** console ➔ Click **Create user pool**.
2. **Configure sign-in experience**:
   - Cognito user pool sign-in options: Check **Email**.
   - Click **Next**.
3. **Configure security requirements**:
   - Password policy: **Cognito defaults** (Min length: 8, requires numbers/symbols).
   - Multi-factor authentication (MFA): Select **No MFA** (for development).
   - User account recovery: Check **Enable self-service account recovery** ➔ **Email only**.
   - Click **Next**.
4. **Configure sign-up experience**:
   - Self-service sign-up: Check **Enable self-service sign-up**.
   - Attribute verification: Check **Allow Cognito to automatically send messages to verify and confirm**.
   - Attributes to verify: Select **Send email message, verify email address**.
   - Required attributes: Ensure **email** is selected.
   - Click **Next**.
5. **Configure message delivery**:
   - Email provider: Select **Send email with Cognito** (daily limit of 50 emails, ideal for development).
   - Click **Next**.
6. **Integrate app**:
   - User pool name: `realtime-collab-user-pool`
   - App client: Select **Public client**.
   - App client name: `realtime-collab-app-client`
   - Client secret: Select **Don't generate a client secret** (required for client-side single page applications).
   - Click **Next**.
7. **Review and create**: Review configuration settings and click **Create user pool**.

---

#### Step 2: Extract Credentials

1. Click on the newly created User Pool (`realtime-collab-user-pool`).
2. Copy the **User pool ID** (e.g., `us-east-1_abcdef123`).
3. Navigate to **App integration** tab ➔ Scroll down to the **App client list** ➔ Copy the **Client ID** (e.g., `3n70abcde...`).

We will use these credentials in both the backend and frontend configurations.
