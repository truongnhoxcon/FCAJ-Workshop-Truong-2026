---
title: "CloudFront Distribution"
date: 2024-06-29
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

We will configure Amazon CloudFront as our global secure HTTPS Ingress, handling SSL termination, WebSocket transport, integrating S3 static storage, and routing requests to the ALB.

---

#### 1. CloudFront Cache & Request Policies

When routing requests through CloudFront, we must configure how CloudFront handles cache storage and HTTP request headers forwarded to our API/WebSocket backend:
- **No Cache**: API and WebSockets deliver real-time content and must never be cached.
- **Forward Headers**: WebSockets require specific handshake headers (Upgrade, Connection) to establish persistent connection tunnels.

---


#### 2. Create CloudFront Distribution

Go to **CloudFront** ➔ **Distributions** ➔ click **Create distribution**.

---

##### 📝 PART 2.1: Complete the Default Distribution Wizard (S3 Focus)

The new CloudFront wizard simplifies creation using a 5-step process targeting the S3 Frontend bucket. Complete this initial workflow:

##### Step 1: Choose a plan
- Select the **Free ($0/month)** plan ➔ Click **Next**.

![Create Distribution - Choose Plan](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-origin-1.png)

##### Step 2: Get started
- **Distribution name**: Keep the default or set it to `s3-origin`.
- **Distribution type**: Select **Single website or app** (optimized for standard single-region SPAs).
- **Domain (Route 53 managed domain)**: Leave blank (Optional) ➔ Click **Next**.

![Create Distribution - Get Started](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-origin-2.png)

##### Step 3: Specify origin
- **Origin type**: Select **Amazon S3**.
- **S3 origin**: Choose or enter your S3 frontend bucket (e.g., `realtime-collab-frontend-<account-id>.s3.us-east-1.amazonaws.com`).
- **Origin path**: Leave blank.
- Click **Next**.

![Create Distribution - Specify Origin](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-origin-3.png)

##### Step 4: Enable security
- Check **Use monitor mode** (this runs WAF in monitor/COUNT mode instead of BLOCK mode, which avoids blocking file uploads/avatar changes larger than 8KB) ➔ Click **Next**.
![Create Distribution - Enable security](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-security.png)

##### Step 5: Review and create
- Review all the configuration details ➔ Click **Create distribution**.
![Create Distribution - Done](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-done.png)

##### Step 6: Copy policy
1. Navigate to the newly created Distribution ➔ Select the **Origins** tab.
2. Select the S3 Origin (e.g., `realtime-collab-frontend-...`) ➔ click **Edit**.
3. Scroll down to the **Bucket policy** section; the yellow banner and the **Copy policy** button will be displayed for you to copy.

![Copy policy from CloudFront](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-policy-1.png)
![Copy policy from CloudFront](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/copy-policy.png)

4. Configure Bucket Policy on S3:
- Go to the **Amazon S3** console and click on your **`realtime-collab-frontend-<account-id>`** bucket.
- Select the **Permissions** tab from the top menu bar.
- Scroll down to the **Bucket policy** section and click the **Edit** button.
![Paste S3 Bucket Policy](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-policy-2.png)
- Paste the copied JSON policy from CloudFront into the policy editor.
- Click **Save changes** at the bottom of the page.
![Origins Tab List Screen](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-alb-1.png)

---

##### ⚙️ PART 2.2: Advanced Configurations (Add ALB Backend & Routing)

Once the distribution is created, click on it from the list to add the ALB dynamic backend origin and configure custom routing behaviors:

##### 1. Add ALB Backend Origin
1. Go to the **Origins** tab ➔ Click **Create origin**.

![Configure ALB Backend Origin](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-alb-2.png)

2. Configure as follows:
   - **Origin domain**: Select or paste your ALB DNS Name (e.g., `realtime-collab-dev-alb-XXXXXXXX.us-east-1.elb.amazonaws.com`).
   - **Name**: Enter `alb-origin` (or keep default).
   - **Protocol**: Select **HTTP only** (CloudFront routes traffic to the ALB via HTTP port 80).
   - **HTTP port**: Keep `80`.

![Configure ALB Backend Origin](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-behavior-1.png)

3. Click **Create origin**.

###### 2. Configure Cache Behaviors (Routing Rules)
Go to the **Behaviors** tab. There is a default `Default (*)` behavior pointing to your S3 origin. Create two additional behaviors for APIs and WebSockets:

![Configure Behavior for Path /api/*](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-behavior-2.png)

* **Create Behavior 1 (For `/api/*` REST API)**:
  - Click **Create behavior**.
  - **Path pattern**: Enter `/api/*`
  - **Origin**: Select alb-origin.
  - **Viewer protocol policy**: Select **Redirect HTTP to HTTPS**.
  - **Allowed HTTP methods**: Select **GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE**.
  - **Cache policy**: Select the AWS-managed **CachingDisabled** policy.
  - **Origin request policy**: Select the AWS-managed **AllViewer** policy.
  - Click **Create behavior**.

  ![Error Pages Tab Screen](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-error-1.png)

* **Create Behavior 2 (For `/ws/*` WebSockets)**:
  - Click **Create behavior**.
  - **Path pattern**: Enter `/ws/*`
  - **Origin**: Select alb-origin.
  - **Viewer protocol policy**: Select **Redirect HTTP to HTTPS**.
  - **Allowed HTTP methods**: Select **GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE**.
  - **Cache policy**: Select **CachingDisabled**.
  - **Origin request policy**: Select **AllViewer**.
  - Click **Create behavior**.

  ![Configure Custom Error Response 403](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-error-2.png)

##### 3. Configure Custom Error Responses (For React SPA Client Routing)
Dynamic client-side routes (like /servers, /login) do not exist physically in S3. CloudFront must rewrite 403 or 404 errors to index.html to allow the SPA's router to load:

1. Select the **Error pages** tab ➔ Click **Create custom error response**.

![Configure Custom Error Response 404](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-error-3.png)

2. Set up 403 error page:
   - **HTTP error code**: 403: Forbidden
   - **Customize error response**: Select Yes
   - **Response page path**: `/index.html`
   - **HTTP response code**: `200: OK`

   ![Custom Error Responses List Screen](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-error-4.png)

3. Click **Create custom error response** again for 404 error page:
   - **HTTP error code**: 404: Not Found
   - **Customize error response**: Select Yes
   - **Response page path**: `/index.html`
   - **HTTP response code**: `200: OK`

   ![General Settings Edit Button](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-general-1.png)

##### 4. Configure Root Object & IPv6
1. Go to the **General** tab ➔ under the **Settings** section, click **Edit**.

![Configure Default Root Object index.html](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-general-2.png)

2. Configure as follows:
   - **Default root object**: Enter **index.html** (Crucial to load the landing page from S3).

   ![Default root object](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-status.png)

3. Click **Save changes**.


Deployment will take **5 - 10 minutes**. Once the status changes to active, copy the **Distribution domain name** (e.g., `xxxxxxxxxxxx.cloudfront.net`). This is the final global HTTPS Endpoint to access your application.

![CloudFront Distribution Completed Domain Name](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-domain.png)

* Access and experience the system at: **`https://d2gqfyhv3cphx1.cloudfront.net`**
