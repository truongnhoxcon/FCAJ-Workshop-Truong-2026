---
title: "RDS PostgreSQL"
date: 2024-06-29
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

Chúng ta sẽ thiết lập cơ sở dữ liệu quan hệ Amazon RDS PostgreSQL chạy ở chế độ riêng tư (Private) và bắt buộc sử dụng mã hoá truyền tải SSL.

---

#### 1. Tạo DB Subnet Group

Trước khi tạo cơ sở dữ liệu, chúng ta cần khai báo nhóm subnet để RDS chạy trong phân vùng Private:

1. Truy cập dịch vụ **RDS** ➔ **Subnet groups** ➔ **Create DB subnet group**.
2. Nhập thông tin cấu hình:
   - **Name**: `realtime-collab-dev-db-subnet-group`
   - **Description**: `Private subnet group for RDS`
   - **VPC**: Chọn realtime-collab-dev
   - **Add subnets**: Chọn các Availability Zones us-east-1a và us-east-1b.
   - **Subnets**: Tích chọn 2 private subnets tương ứng với dải IP 10.0.10.0/24 và 10.0.11.0/24.
3. Nhấn **Create**.

![Khởi tạo DB Subnet Group](/images/5-Workshop/5.6-RDS-Database/db-subnet-group-create.png)

---

#### 2. Tạo Parameter Group (Bắt buộc mã hoá SSL)

1. Truy cập **RDS** ➔ **Parameter groups** ➔ **Create parameter group**.
2. Thiết lập:
   - **Parameter group family**: Chọn postgres15
   - **Type**: DB Parameter Group
   - **Group name**: `realtime-collab-dev-pg15-ssl`
   - **Description**: `PostgreSQL 15 enforce SSL`
3. Nhấn **Create**.

![Khởi tạo Parameter Group](/images/5-Workshop/5.6-RDS-Database/db-subnet-group-details.png)


4. Chọn Parameter Group vừa tạo ➔ bấm **Edit** ➔ Tìm kiếm tham số `rds.force_ssl` và chuyển giá trị thành `1` (Apply method: `immediate`). Nhấn **Save changes**.

![Eidt Parameter Group](/images/5-Workshop/5.6-RDS-Database/db-parameter-group-create.png)

![Bắt buộc sử dụng mã hoá SSL](/images/5-Workshop/5.6-RDS-Database/db-parameter-group-edit.png)

---

#### 3. Khởi tạo Database Instance

Chọn **RDS** ➔ **Databases** ➔ **Create database**. Sử dụng cấu hình:

- **Choose a database creation method**: Full configuration
- **Engine options**: PostgreSQL
- **Templates**: Dev/Test

![Khởi tạo Database - Engine Options](/images/5-Workshop/5.6-RDS-Database/db-create-engine.png)

- **Availability and durability**: **Multi-AZ DB instance**

![Khởi tạo Database - Settings & Credentials](/images/5-Workshop/5.6-RDS-Database/2az.png)


- **Settings**:
  - **DB instance identifier**: `realtime-collab-dev-db`
  - **Master username**: `postgres`
  - **Master password**: Nhập đúng mật khẩu đã lưu trong Secrets Manager `db-password` ở Bước 1.

![Khởi tạo Database - Master Password](/images/5-Workshop/5.6-RDS-Database/db-create-settings-2.png)
- **Instance configuration**: db.t3.medium
- **Storage**:
  - Storage type: `gp3` | Allocated storage: `100 GB`
  - **Enable storage autoscaling**: **Tích chọn** | Maximum storage threshold: `500 GB`

![Khởi tạo Database - Instance Type](/images/5-Workshop/5.6-RDS-Database/db-create-instance.png)
![Khởi tạo Database - Storage Autoscaling](/images/5-Workshop/5.6-RDS-Database/db-create-storage.png)
- **Connectivity**:
  - **VPC**: realtime-collab-dev
  - **DB Subnet group**: realtime-collab-dev-db-subnet-group
  - **Public access**: Chọn No
  - **VPC security group (existing)**: Chọn rds-sg (Xóa bỏ group default nếu có).
- **Database authentication**: Password authentication

![Khởi tạo Database - Connectivity Part 1](/images/5-Workshop/5.6-RDS-Database/db-create-network-1.png)
![Khởi tạo Database - Connectivity Part 2](/images/5-Workshop/5.6-RDS-Database/db-create-network-2.png)
- **Additional configuration**:
  - **Initial database name**: `realtime_collab`
  - **DB parameter group**: Chọn `realtime-collab-dev-pg15-ssl`
![Khởi tạo Database - Additional Configuration Part 1](/images/5-Workshop/5.6-RDS-Database/db-create-additional-1.png)
  - **Backup**:
    - Backup retention period: 7 days
    - Backup window: Chọn 03:00 - 04:00 UTC
    - **Copy tags to snapshots**: **Tích chọn**

![Khởi tạo Database - Additional Configuration Part 2](/images/5-Workshop/5.6-RDS-Database/db-create-additional-2.png)
  - **Encryption**: **Tích chọn** Enable encryption
  - **Monitoring**:
    - **Enable Enhanced Monitoring**: **Tích chọn** (Granularity: 60 seconds, tạo Monitoring role mới).
  - **Performance Insights**: **Tích chọn** Enable Performance Insights(Retention: 7 days).
  - **Log exports**: **Tích chọn** PostgreSQL log
  - **Maintenance**: **Tích chọn** Auto minor version upgrade
  - **Deletion protection**: Không tích chọn (để dễ dàng xóa khi dọn dẹp).

![Khởi tạo Database - Additional Configuration Part 3](/images/5-Workshop/5.6-RDS-Database/db-create-additional-3.png)

Nhấn **Create database**. Quá trình khởi tạo sẽ mất khoảng **10 - 15 phút**. Sau khi Database hiển thị trạng thái Available, hãy sao chép lại **Endpoint** của database để sử dụng làm biến môi trường DB_HOST ở các bước sau.
