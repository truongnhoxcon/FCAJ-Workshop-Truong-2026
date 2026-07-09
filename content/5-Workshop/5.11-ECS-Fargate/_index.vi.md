---
title: "ECS Cluster & Services"
date: 2024-06-29
weight: 11
chapter: false
pre: " <b> 5.11. </b> "
---

Chúng ta sẽ thiết lập cụm máy chủ Amazon ECS Fargate chạy các container serverless và cấu hình chi tiết Task Definition và Service liên kết với bộ cân bằng tải ALB.

---

#### Trước hết: Tạo Log Group cho Unified Backend
Vào **CloudWatch** ➔ **Log groups** ➔ **Create log group** tên `/ecs/unified-backend` với **Retention**: `30 days`.

![Khởi tạo ECS Cluster](/images/5-Workshop/5.11-ECS-Fargate/ecs-cluster-create.png)

---

#### 1. Khởi tạo ECS Cluster

Truy cập dịch vụ **Elastic Container Service (ECS)** ➔ **Clusters** ➔ **Create cluster**:

- **Cluster name**: `realtime-collab-dev`
- **Infrastructure**: **Tích chọn** Fargate only

![Khởi tạo ECS Cluster Phần 1](/images/5-Workshop/5.11-ECS-Fargate/ecs-task-json-1.png)

- **Monitoring**: **Tích chọn** Container Insights

![Khởi tạo ECS Cluster Phần 2](/images/5-Workshop/5.11-ECS-Fargate/ecs-task-json-2.png)


---

#### 2. Tạo Task Definition bằng JSON

Chúng ta sẽ tạo mô tả tác vụ bằng cách dán mã nguồn JSON trực tiếp:

Truy cập **ECS** ➔ **Task definitions** ➔ **Create new task definition** ➔ chọn **Create new revision with JSON**.

![Khởi tạo Task Definition](/images/5-Workshop/5.11-ECS-Fargate/ecs-task-created.png)

##### 2.1 Task Definition — Unified Backend (`realtime-collab-unified-backend`)

Dán JSON dưới đây và thay thế các giá trị:
- REPLACE_ACCOUNT_ID: Nhập 12 số AWS Account ID.
- REPLACE_RDS_ENDPOINT: Endpoint của RDS PostgreSQL đã lưu ở Bước 7.
- REPLACE_REDIS_ENDPOINT: Primary Endpoint của Redis ở Bước 8.
- REPLACE_DB_PASSWORD_ARN: ARN của db-password
- REPLACE_JWT_SECRET_ARN: ARN của jwt-secret
- REPLACE_TWILIO_SECRET_ARN: ARN của twilio/api-credentials
- REPLACE_REDIS_SECRET_ARN: ARN của redis-auth-token

```json
{
  "family": "realtime-collab-unified-backend",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::REPLACE_ACCOUNT_ID:role/ecsTaskExecutionRole-realtime-collab-dev",
  "taskRoleArn": "arn:aws:iam::REPLACE_ACCOUNT_ID:role/ecsTaskRole-realtime-collab-dev",
  "containerDefinitions": [{
    "name": "unified-backend",
    "image": "REPLACE_ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/unified-backend:latest",
    "portMappings": [{ "containerPort": 3000, "protocol": "tcp" }],
    "environment": [
      { "name": "PORT",         "value": "3000" },
      { "name": "NODE_ENV",     "value": "production" },
      { "name": "DB_HOST",      "value": "REPLACE_RDS_ENDPOINT" },
      { "name": "REDIS_HOST",   "value": "REPLACE_REDIS_ENDPOINT" },
      { "name": "AWS_REGION",   "value": "us-east-1" },
      { "name": "DB_SSL",       "value": "true" },
      { "name": "REDIS_TLS",    "value": "true" },
      { "name": "S3_BUCKET_NAME","value": "realtime-collab-files-REPLACE_ACCOUNT_ID" }
    ],
    "secrets": [
      { "name": "DB_PASSWORD",         "valueFrom": "REPLACE_DB_PASSWORD_ARN" },
      { "name": "JWT_SECRET",          "valueFrom": "REPLACE_JWT_SECRET_ARN:secret::" },
      { "name": "TWILIO_ACCOUNT_SID",  "valueFrom": "REPLACE_TWILIO_SECRET_ARN:accountSid::" },
      { "name": "TWILIO_AUTH_TOKEN",   "valueFrom": "REPLACE_TWILIO_SECRET_ARN:authToken::" },
      { "name": "REDIS_PASSWORD",     "valueFrom": "REPLACE_REDIS_SECRET_ARN" }
    ],
    "healthCheck": {
      "command": ["CMD-SHELL", "wget -qO- http://localhost:3000/health || exit 1"],
      "interval": 30, "timeout": 5, "retries": 3, "startPeriod": 60
    },
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group":         "/ecs/unified-backend",
        "awslogs-region":        "us-east-1",
        "awslogs-stream-prefix": "ecs"
      }
    },
    "essential": true
  }]
}
```

![ECS Service](/images/5-Workshop/5.11-ECS-Fargate/ecs-service-status.png)

---

#### 3. Khởi tạo ECS Service (Sử dụng AWS CLI)

##### Bước 1: Thu thập thông số cần thiết
Hãy truy cập AWS Console để tìm và ghi lại các thông tin sau:

1. **Target Group ARNs**:
   * Truy cập dịch vụ **EC2** ➔ **Target groups** ở menu bên trái.
   * Chọn realtime-collab-dev-uni-api-tg ➔ Sao chép giá trị **ARN** ở phần Details.
   * Chọn realtime-collab-dev-uni-ws-tg ➔ Sao chép giá trị **ARN**.
2. **Private Subnet IDs**:
   * Truy cập dịch vụ **VPC** ➔ **Subnets** ➔ Tìm kiếm và sao chép ID của private-subnet-1 (Dạng `subnet-xxxxxxxxxxxx`) và private-subnet-2.
3. **Security Group ID**:
   * Truy cập dịch vụ **VPC** ➔ **Security groups** ➔ Tìm kiếm và sao chép ID của nhóm bảo mật realtime-collab-dev-unified-backend-sg (Dạng `sg-xxxxxxxxxxxx`).

---

##### Bước 2: Thực thi lệnh khởi tạo Service trên Terminal
Mở terminal trên máy tính của bạn (đảm bảo máy đã được cài đặt AWS CLI và đã cấu hình tài khoản AWS hợp lệ) và chạy lệnh sau (Hãy thay thế các chuỗi `REPLACE_...` bằng các thông số thực tế vừa thu thập ở Bước 1):

```bash
aws ecs create-service \
  --cluster realtime-collab-dev \
  --service-name unified-backend \
  --task-definition realtime-collab-unified-backend \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration 'awsvpcConfiguration={subnets=[REPLACE_PRIVATE_SUBNET_1_ID,REPLACE_PRIVATE_SUBNET_2_ID],securityGroups=[REPLACE_UNIFIED_BACKEND_SG_ID],assignPublicIp=DISABLED}' \
  --load-balancers '[
    {
      "targetGroupArn": "REPLACE_API_TARGET_GROUP_ARN",
      "containerName": "unified-backend",
      "containerPort": 3000
    },
    {
      "targetGroupArn": "REPLACE_WS_TARGET_GROUP_ARN",
      "containerName": "unified-backend",
      "containerPort": 3000
    }
  ]'
```

Sau khi chạy lệnh thành công, AWS sẽ trả về thông tin cấu hình Service dạng JSON.

---

##### Bước 3: Kiểm tra trạng thái triển khai
1. Truy cập dịch vụ **ECS** ➔ **Clusters** ➔ Chọn Cluster `realtime-collab-dev`.
2. Tại tab **Services**, bạn sẽ thấy Service **unified-backend** đã được tạo.
3. Nhấp chọn vào tên Service ➔ Chọn tab **Tasks** để giám sát quá trình tải docker image và khởi chạy các containers. Khi các tasks hiển thị trạng thái **Running** và Health status là **Healthy**, hệ thống đã chính thức hoạt động!

![ECS Service trạng thái hoạt động Healthy](/images/5-Workshop/5.11-ECS-Fargate/tasks-status.png)



