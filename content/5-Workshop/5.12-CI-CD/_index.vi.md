---
title: "Thiết lập CI/CD"
date: 2024-06-29
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

Trong bước này, chúng ta sẽ thiết lập một đường ống **CI/CD tự động** (Tích hợp liên tục và Triển khai liên tục) sử dụng **GitHub Actions**. Mục tiêu là tự động hóa quy trình: đóng gói Docker image từ mã nguồn mới nhất ➔ đẩy lên Amazon ECR ➔ cập nhật phiên bản mới lên cụm Amazon ECS Fargate mỗi khi có thay đổi code được đẩy lên nhánh `main`.

---

#### 1. Tạo IAM User và Phân quyền trên AWS
GitHub Actions cần các khóa truy cập (Access Keys) để xác thực và giao tiếp an toàn với các tài nguyên AWS của bạn.

1. Truy cập **IAM Console** ➔ **Users** ➔ Bấm **Create user**.
2. Thiết lập:
   - **User name**: `github-actions-deployer` ➔ Bấm **Next**.
3. Tại phần **Set permissions**:
   - Chọn **Attach policies directly**.
   - Tìm kiếm và tích chọn chính sách có sẵn: **`AmazonEC2ContainerRegistryPowerUser`** (Quyền đăng nhập và đẩy ảnh lên ECR).
   - Bấm **Create policy** để tạo một chính sách tùy chỉnh cho quyền truy cập ECS, dán đoạn JSON cấu hình dưới đây:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "ECSActions",
                 "Effect": "Allow",
                 "Action": [
                     "ecs:DescribeServices",
                     "ecs:UpdateService",
                     "ecs:RegisterTaskDefinition",
                     "ecs:DescribeTaskDefinition"
                 ],
                 "Resource": "*"
             },
             {
                 "Sid": "PassRole",
                 "Effect": "Allow",
                 "Action": [
                     "iam:PassRole"
                 ],
                 "Resource": [
                     "arn:aws:iam::<account-id>:role/ecsTaskExecutionRole-realtime-collab-dev",
                     "arn:aws:iam::<account-id>:role/ecsTaskRole-realtime-collab-dev"
                 ]
             },
             {
                 "Sid": "S3Actions",
                 "Effect": "Allow",
                 "Action": [
                     "s3:ListAllMyBuckets",
                     "s3:ListBucket",
                     "s3:PutObject",
                     "s3:GetObject",
                     "s3:DeleteObject"
                 ],
                 "Resource": "*"
             },
             {
                 "Sid": "CloudFrontActions",
                 "Effect": "Allow",
                 "Action": [
                     "cloudfront:ListDistributions",
                     "cloudfront:CreateInvalidation"
                 ],
                 "Resource": "*"
             }
         ]
     }
     ```
     *(Thay thế `<account-id>` bằng 12 chữ số AWS Account ID của bạn)*
   - Đặt tên chính sách là `ECSDeployPolicy` và nhấn **Create**. Quay lại màn hình tạo User, nhấn nút Refresh, tìm chính sách `ECSDeployPolicy` vừa tạo và tích chọn ➔ Bấm **Next** ➔ **Create user**.
4. Truy cập User `github-actions-deployer` vừa tạo ➔ Chọn tab **Security credentials**.
5. Cuộn xuống mục **Access keys** ➔ Chọn **Create access key**.
6. Chọn loại **Command Line Interface (CLI)** ➔ Nhấn tiếp tục cho đến khi hoàn thành.
7. **Lưu trữ an toàn** hai giá trị được cung cấp:
   - **Access Key ID**
   - **Secret Access Key**

---

#### 2. Cấu hình Secrets trên GitHub Repository
Để bảo mật thông tin nhạy cảm của AWS, chúng ta sẽ lưu chúng dưới dạng GitHub Secrets.

1. Mở trang Repository dự án của bạn trên GitHub.
2. Điều hướng tới **Settings** ➔ **Secrets and variables** (ở menu bên trái) ➔ Chọn **Actions**.
3. Bấm nút **New repository secret** để thêm các giá trị:
   - **Secret 1**:
     - Name: `AWS_ACCESS_KEY_ID`
     - Secret: *(Dán Access Key ID của IAM User vừa tạo)*
   - **Secret 2**:
     - Name: `AWS_SECRET_ACCESS_KEY`
     - Secret: *(Dán Secret Access Key của IAM User vừa tạo)*

---

#### 3. Tạo Tệp cấu hình GitHub Actions Workflow
Chúng ta sẽ tạo một tệp tin cấu hình YAML để khai báo các bước tự động trong GitHub Actions.

1. Tại thư mục mã nguồn dự án local, tạo cấu trúc thư mục và tệp tin sau:
   - `.github/workflows/deploy.yml`
2. Mở tệp `deploy.yml` và dán cấu hình sau:

```yaml
name: Deploy Backend to Amazon ECS

on:
  push:
    branches:
      - main  # Tự động chạy khi có lệnh push lên nhánh main

env:
  AWS_REGION: us-east-1                    # Thay thế bằng Region của bạn
  ECR_REPOSITORY: realtime-collab-api      # Tên ECR Repository bạn tạo ở Bước 8
  ECS_CLUSTER: realtime-collab-cluster     # Tên ECS Cluster của bạn
  ECS_SERVICE: realtime-collab-service     # Tên ECS Service bạn tạo ở Bước 11

jobs:
  deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest

    steps:
      # Bước 1: Kéo mã nguồn mới nhất từ GitHub về máy ảo chạy Action
      - name: Checkout code
        uses: actions/checkout@v4

      # Bước 2: Xác thực với tài khoản AWS thông qua Access Keys
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      # Bước 3: Đăng nhập vào hệ thống Amazon ECR
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      # Bước 4: Biên dịch Docker image, gắn thẻ phiên bản và đẩy lên ECR
      - name: Build, tag, and push image to Amazon ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:${{ github.sha }} -t $ECR_REGISTRY/$ECR_REPOSITORY:latest .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:${{ github.sha }}
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

      # Bước 5: Yêu cầu Amazon ECS khởi chạy deployment mới (Kéo ảnh latest mới nhất về)
      - name: Force New ECS Deployment
        run: |
          aws ecs update-service --cluster ${{ env.ECS_CLUSTER }} --service ${{ env.ECS_SERVICE }} --force-new-deployment
```

---

#### 4. Đẩy mã nguồn & Kiểm thử đường ống CI/CD
1. Lưu toàn bộ các tệp tin cấu hình và chạy lệnh đẩy code lên nhánh `main`:
   ```bash
   git add .
   git commit -m "Set up automated GitHub Actions CI/CD for Amazon ECS"
   git push origin main
   ```
2. Mở Repository trên GitHub ➔ Chọn tab **Actions**.
3. Bạn sẽ thấy một tiến trình có tên *"Deploy Backend to Amazon ECS"* đang tự động chạy.
4. Nhấp chọn để theo dõi nhật ký biên dịch chi tiết. Khi tất cả các bước hiển thị màu xanh lá cây thành công:
   - AWS ECS sẽ tự động kích hoạt quá trình thay thế tác vụ cũ (Rolling Update), khởi tạo Task mới với mã nguồn vừa push, sau đó dừng Task cũ lại một cách an toàn mà không làm gián đoạn người dùng (Zero Downtime).
