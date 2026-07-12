---
title: "AWS Cognito User Pool"
date: 2024-06-29
weight: 4
chapter: false
pre: " <b> 5.2.4. </b> "
---

We will provision an **Amazon Cognito User Pool** to manage user registration, login, email verification, and secure access tokens without managing server-side database passwords.

---

#### Step 1: Create Cognito User Pool (User Directory)

AWS Cognito has introduced a streamlined wizard for configuring application resources. Follow these steps to set up the user directory:

1. Navigate to the **Amazon Cognito** console ➔ Click **Create user pool**.
2. Under **Define your application**:
   - **Application type**: Select **Single-page application (SPA)** (since AntiCollab frontend is built on React/Vite).
   - **Name your application**: Enter a descriptive name, e.g., `realtime-collab-app`.
3. Under **Configure options**:
   - **Options for sign-in identifiers**: Check **Email**.
   - **Self-registration**: Check **Enable self-registration** (allows users to sign up via public APIs).
   - **Required attributes for sign-up**: Click the dropdown and select **email**.
4. Under **Add a return URL - optional**:
   - For local testing, you can input `http://localhost`. For production, you will add your CloudFront domain. (You can also leave this blank as it is optional).
5. Click **Create user directory** at the bottom right of the page.

---

#### Step 2: Extract Credentials

1. Once the user directory is created, click on it in the Cognito console.
2. In the pool details page:
   - Copy the **User pool ID** (displayed at the top, e.g., `us-east-1_abcdef123`).
3. Navigate to the **App integration** tab ➔ Scroll down to the **App client list** ➔ Copy the **Client ID** (e.g., `3n70abcde...`).

We will use these credentials in both the backend and frontend configurations.
