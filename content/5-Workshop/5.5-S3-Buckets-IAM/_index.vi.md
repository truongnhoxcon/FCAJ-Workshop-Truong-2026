---
title: "S3 Buckets & IAM Roles"
date: 2024-06-29
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

Chúng ta sẽ tạo 2 S3 Buckets phục vụ lưu trữ mã nguồn tĩnh (frontend) và tệp tin người dùng (files), cùng với 2 vai trò IAM Roles cung cấp quyền hạn tối thiểu cho ECS Tasks.

---

### PHẦN 1 — Khởi tạo S3 Buckets

Truy cập dịch vụ **S3** ➔ **Create bucket**. Tạo lần lượt 2 buckets ở vùng **us-east-1** (Thay thế '<account-id>' bằng 12 chữ số Account ID của bạn).

#### 1. Bucket lưu trữ mã nguồn tĩnh Frontend: `realtime-collab-frontend-<account-id>`
- **Block public access**: Block all public access **➔ Tích chọn** (Bảo mật tuyệt đối, CloudFront sử dụng OAC để đọc).
- **Bucket Versioning**: Disabled (Không cần versioning).
- **Encryption**: SSE-S3 (AES-256)

![Khởi tạo S3 Frontend - Tên và Vùng](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-frontend-1.png)
![Khởi tạo S3 Frontend - Block Public Access và Versioning](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-frontend-2.png)
![Khởi tạo S3 Frontend - Encryption](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-frontend-3.png)

- **Bucket Policy** (Tab Permissions ➔ Bucket policy - Chính sách này sẽ cho phép CloudFront OAC được đọc file từ S3):
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Sid": "AllowCloudFrontServicePrincipalReadOnly",
        "Effect": "Allow",
        "Principal": {
          "Service": "cloudfront.amazonaws.com"
        },
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::realtime-collab-frontend-<account-id>/*",
        "Condition": {
          "ArnEquals": {
            "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<cloudfront-distribution-id>"
          }
        }
      }
    ]
  }
  ```

![Cấu hình Bucket Policy cho Frontend](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-frontend-policy.png)

#### 2. Bucket lưu trữ tệp tin người dùng: `realtime-collab-files-<account-id>`
- **Block public access**: Block all public access **➔ Tích chọn**
- **Bucket Versioning**: Enabled
- **Encryption**: SSE-S3 (AES-256)

Cấu hình bổ sung cho bucket lưu trữ tệp tin:
- **Bucket Policy** (Tab Permissions ➔ Bucket policy):
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [{
      "Sid": "DenyNonSSL",
      "Effect": "Deny",
      "Principal": "*",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::realtime-collab-files-<account-id>",
        "arn:aws:s3:::realtime-collab-files-<account-id>/*"
      ],
      "Condition": { "Bool": { "aws:SecureTransport": "false" } }
    }]
  }
  ```

![Cấu hình Bucket Policy cho Files](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-files-policy.png)

- **Lifecycle rules** (Tab Management ➔ Create lifecycle rule):
![Cấu hình Bucket Policy cho Files](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-files-cors.png)

  - **Rule 1**: Tên `archive-workspace-files` | Prefix: `workspace-` | Action: Chuyển sang lưu trữ Glacier sau 90 ngày.

  ![Cấu hình Lifecycle Rules 1 cho Files](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-files-lifecycle-rule-1.1.png)

  ![Cấu hình Lifecycle Rules 1 cho Files](/images/5-Workshop/5.5-S3-Buckets-IAM/s3-files-lifecycle-rule-1.2.png)

  - **Rule 2**: Tên `abort-incomplete-multipart` | Prefix: Để trống | Action: Hủy các tệp tải lên dở dang sau 7 ngày.

![Cấu hình Lifecycle Rules 2 cho Files](/images/5-Workshop/5.5-S3-Buckets-IAM/abort-incomplete-multipart-1.png)

![Cấu hình Lifecycle Rules 2 cho Files](/images/5-Workshop/5.5-S3-Buckets-IAM/abort-incomplete-multipart-2.png)

- **Cross-origin resource sharing (CORS)** (Tab Permissions ➔ CORS):
  ```json
  [{
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "POST", "DELETE"],
    "AllowedOrigins": ["*"],
    "MaxAgeSeconds": 3600
  }]
  ```

  ![Cấu hình CORS cho Files](/images/5-Workshop/5.5-S3-Buckets-IAM/file-CORS-1.png)

  ![Cấu hình CORS cho Files](/images/5-Workshop/5.5-S3-Buckets-IAM/file-CORS-2.png)

---

### PHẦN 2 — Cấu hình IAM Roles cho ECS

Để vận hành các container trên ECS Fargate một cách bảo mật nhất, chúng ta cần cấu hình hai vai trò (IAM Roles) khác nhau:
1. **Task Execution Role**: Dùng cho bản thân ECS Agent của AWS để tải image từ ECR, cấu hình log CloudWatch và lấy các khoá secrets từ Secrets Manager trong quá trình khởi tạo (bootstrap) container.
2. **Task Role**: Dùng cho mã nguồn ứng dụng chạy bên trong container để tương tác trực tiếp với các tài nguyên AWS (như đọc/ghi file lên S3, gửi API đến các dịch vụ bên ngoài) tại thời điểm chạy (runtime).

Dưới đây là các bước thiết lập chi tiết trên AWS Console:

---

#### Bước 1: Khởi tạo ECS Task Execution Role (`ecsTaskExecutionRole-realtime-collab-dev`)

1. Truy cập dịch vụ **IAM** từ thanh tìm kiếm AWS Console.
2. Tại menu bên trái, chọn **Roles** ➔ Bấm nút **Create role** ở góc phải.
3. Trong giao diện tạo Role:
   - **Trusted entity type**: Chọn **AWS service**.
   - **Service or use case**: Chọn **Elastic Container Service** ở danh sách thả xuống.
   - **Use case**: Tích chọn **Elastic Container Service Task** (Lưu ý chọn đúng dòng này để cho phép ECS Task đảm nhận vai trò).

   ![Khởi tạo ECS Task Execution Role](/images/5-Workshop/5.5-S3-Buckets-IAM/create-role-1.png)

   - Bấm **Next**.
4. Ở màn hình **Add permissions**:
   - Không tích chọn bất kỳ chính sách có sẵn nào (chúng ta sẽ cấu hình Inline Policies riêng để đảm bảo quyền hạn tối thiểu).
   - Bấm **Next**.
5. Ở màn hình **Name, review, and create**:
   - **Role name**: Nhập chính xác `ecsTaskExecutionRole-realtime-collab-dev`.
   - **Description**: Nhập `ECS Task Execution Role for realtime-collab project`.
   - Bấm **Create role** ở cuối trang.

![Khởi tạo ECS Task Execution Role](/images/5-Workshop/5.5-S3-Buckets-IAM/create-role-2.png)

6. Sau khi tạo xong, tìm kiếm tên role `ecsTaskExecutionRole-realtime-collab-dev` tại thanh tìm kiếm danh sách Roles và nhấp vào nó để cấu hình tiếp.
7. Thêm lần lượt **3 Inline Policies** sau đây bằng cách bấm vào dropdown **Add permissions** ở góc phải ➔ Chọn **Create inline policy**:

![Create inline policy](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-exec-role-create.png)

##### ➕ Inline Policy 1: Cấp quyền tải Docker Image từ ECR (`ecr-pull`)
* Bấm vào tab **JSON** ở công cụ tạo policy và dán mã nguồn sau:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ECRAuthToken",
      "Effect": "Allow",
      "Action": ["ecr:GetAuthorizationToken"],
      "Resource": ["*"]
    },
    {
      "Sid": "ECRImagePull",
      "Effect": "Allow",
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ],
      "Resource": ["*"]
    }
  ]
}
```
![Tạo Inline Policy ecr-pull](/images/5-Workshop/5.5-S3-Buckets-IAM/ecr-pull-policy.png)

* Bấm **Next**, đặt tên Policy là `ecr-pull` và chọn **Create policy**.

![Tạo Inline Policy ecr-pull](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-exec-policy-ecr.png)

##### ➕ Inline Policy 2: Cấp quyền đẩy log lên CloudWatch (`cloudwatch-logs-ecs`)
* Bấm **Add permissions** ➔ **Create inline policy**, chọn tab **JSON** và dán mã:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "CloudWatchLogsECS",
    "Effect": "Allow",
    "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
    "Resource": ["arn:aws:logs:*:*:log-group:/ecs/*"]
  }]
}
```
![Tạo Inline Policy secrets-execution](/images/5-Workshop/5.5-S3-Buckets-IAM/cloudwatch-logs-ecs.png)

* Bấm **Next**, đặt tên Policy là `cloudwatch-logs-ecs` và chọn **Create policy**.

##### ➕ Inline Policy 3: Cấp quyền đọc khoá secrets từ Secrets Manager (`secrets-execution`)
* Bấm **Add permissions** ➔ **Create inline policy**, chọn tab **JSON** và dán mã (Thay thế `<account-id>` bằng 12 số AWS Account ID của bạn):
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "SecretsManagerGetExecutionSecrets",
    "Effect": "Allow",
    "Action": ["secretsmanager:GetSecretValue"],
    "Resource": [
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/db-password*",
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/jwt-secret*",
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/twilio/api-credentials*",
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/redis-auth-token*"
    ]
  }]
}
```
![Tạo Inline Policy secrets-execution](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-exec-policy-secrets.png)

* Bấm **Next**, đặt tên Policy là `secrets-execution` và chọn **Create policy**.



---

#### Bước 2: Khởi tạo ECS Task Role (`ecsTaskRole-realtime-collab-dev`)

1. Quay lại danh sách **Roles** của dịch vụ IAM và bấm **Create role**.
2. Chọn cấu hình tương tự như Bước 1:
   - **Trusted entity type**: Chọn **AWS service**.
   - **Service or use case**: Chọn **Elastic Container Service** ➔ **Elastic Container Service Task**.
   - Bấm **Next**.

   ![Khởi tạo ECS Task Role](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-task-role-create.png)
   
3. Ở màn hình **Add permissions**: Bỏ qua việc chọn chính sách và bấm **Next**.
4. Ở màn hình **Name, review, and create**:
   - **Role name**: Nhập chính xác `ecsTaskRole-realtime-collab-dev`.
   - **Description**: Nhập `ECS Task Role for realtime-collab project runtime`.
   - Bấm **Create role**.

![Khởi tạo ECS Task Role](/images/5-Workshop/5.5-S3-Buckets-IAM/ecsTaskRole-realtime-collab-dev.png)

5. Nhấp chọn vào role `ecsTaskRole-realtime-collab-dev` vừa tạo để cấu hình thêm.
6. Thêm lần lượt **2 Inline Policies** sau đây thông qua nút **Add permissions** ➔ **Create inline policy**:

##### ➕ Inline Policy 1: Cấp quyền thao tác tập tin trên S3 Files Bucket (`s3-file-storage`)
* Chọn tab **JSON** và dán mã cấu hình (Thay thế `<account-id>` bằng Account ID của bạn):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "S3ObjectOperations",
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject", "s3:DeleteObject"],
      "Resource": "arn:aws:s3:::realtime-collab-files-<account-id>/*"
    },
    {
      "Sid": "S3ListBucket",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::realtime-collab-files-<account-id>"
    }
  ]
}
```
![Tạo Inline Policy s3-file-storage](/images/5-Workshop/5.5-S3-Buckets-IAM/iam-task-policy-s3.png)

* Bấm **Next**, đặt tên Policy là `s3-file-storage` và chọn **Create policy**.

##### ➕ Inline Policy 2: Cấp quyền runtime để lấy secrets phụ phục vụ app (`secrets-task`)
* Bấm **Add permissions** ➔ **Create inline policy**, chọn tab **JSON** và dán mã (Thay thế `<account-id>`):
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Sid": "SecretsManagerGetTwilio",
    "Effect": "Allow",
    "Action": ["secretsmanager:GetSecretValue"],
    "Resource": [
      "arn:aws:secretsmanager:us-east-1:<account-id>:secret:realtime-collab-dev/twilio/api-credentials*"
    ]
  }]
}
```
![Tạo Inline Policy secrets-task](/images/5-Workshop/5.5-S3-Buckets-IAM/secrets-task.png)

* Bấm **Next**, đặt tên Policy là `secrets-task` và chọn **Create policy**.

Sau khi hoàn thành hai vai trò trên, chúng ta đã sẵn sàng để cấu hình hạ tầng mạng VPC và phân quyền hoạt động an toàn cho các tác vụ ECS Fargate.
