---
title: "ECR & Đẩy Docker Images"
date: 2024-06-29
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

Chúng ta sẽ tạo kho lưu trữ hình ảnh Docker private trên Amazon ECR và tiến hành build, push container image của dịch vụ backend unified lên AWS.

---

#### 1. Khởi tạo ECR Repository

Truy cập dịch vụ **Elastic Container Registry (ECR)** ➔ **Repositories** ➔ **Create repository**. Tạo 1 kho lưu trữ:

- **Repository name**: `unified-backend`
- **Tag mutability**: Mutable

![Khởi tạo ECR Repository Phần 1](/images/5-Workshop/5.4-Deployment-Delivery/5.4.1-ECR-Push/ecr-create-1.png)

- **Scan on push**: **Kích hoạt (Enable)**

![Khởi tạo ECR Repository Phần 2](/images/5-Workshop/5.4-Deployment-Delivery/5.4.1-ECR-Push/ecr-create-2.png)

---

#### 2. Cấu hình Lifecycle Policy cho Repo

Nhằm tự động dọn dẹp các image cũ để tối ưu hóa chi phí lưu trữ, truy cập vào repo vừa tạo ➔ chọn **Lifecycle policy** ở thanh menu bên trái ➔ **Create rule**. Thiết lập 2 rules sau:

![Cấu hình Lifecycle Rule 1](/images/5-Workshop/5.4-Deployment-Delivery/5.4.1-ECR-Push/ecr-created.png)

##### Rule 1: Hủy các image không gắn tag (Untagged) sau 1 ngày
- **Rule priority**: `1`
- **Description**: `Expire untagged images after 1 day`
- **Image status**: Untagged
- **Match criteria**: Days since image created ➔ Days before action: 1

![Cấu hình Lifecycle Rule 1](/images/5-Workshop/5.4-Deployment-Delivery/5.4.1-ECR-Push/ecr-lifecycle-1.png)

##### Rule 2: Chỉ giữ lại 30 image có tag mới nhất
- **Rule priority**: `2`
- **Description**: `Keep only 30 most recent tagged images`
- **Image status**: Tagged (prefix matching) ➔ Prefix list: nhập `v, latest, sha-`
![Cấu hình Lifecycle Rule 2 - Phần 1](/images/5-Workshop/5.4-Deployment-Delivery/5.4.1-ECR-Push/ecr-lifecycle-2.png)

- **Match criteria**: Image count ➔ Count number: 30
- **Action**: Expire
![Cấu hình Lifecycle Rule 2 - Phần 2](/images/5-Workshop/5.4-Deployment-Delivery/5.4.1-ECR-Push/ecr-lifecycle-3.png)

---

#### 3. Build & Push Docker Images lên ECR

Mở terminal trên máy phát triển (yêu cầu máy đã cài đặt Docker và AWS CLI đã cấu hình quyền xác thực), di chuyển tới thư mục gốc của dự án AntiCollab và chạy lệnh sau:

```bash
bash deploy-ecr.sh us-east-1
```

Script này sẽ tự động:
1. Đăng nhập Docker Client vào Amazon ECR Registry.
2. Build Docker image từ mã nguồn của Backend Unified.
3. Đẩy (Push) image này lên ECR tương ứng với tag latest.

Khi hoàn thành, sẽ được kết quả:


![Kết quả chạy deploy-ecr.sh trên terminal](/images/5-Workshop/5.4-Deployment-Delivery/5.4.1-ECR-Push/ecr-deploy-terminal.png)
Hãy sao chép địa chỉ Image URI này để điền vào phần Task Definitions ở bước chạy ECS.
