---
title: "ElastiCache Redis"
date: 2024-06-29
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

Chúng ta sẽ tạo cụm Amazon ElastiCache (Redis OSS) để đồng bộ hóa các sự kiện thời gian thực (Pub/Sub) và lưu trạng thái online/offline của người dùng.

---

#### 1. Tạo Cache Subnet Group

1. Truy cập dịch vụ **ElastiCache** trên AWS Console ➔ **Subnet groups** ở menu bên trái ➔ Bấm **Create subnet group**.
2. Thiết lập:
   - **Name**: `realtime-collab-dev-redis-subnet-group`
   - **Description**: `Private subnet group for Redis`
   - **VPC**: Chọn realtime-collab-dev
   - **Subnets**: Tích chọn 2 private subnets tương ứng với dải IP 10.0.10.0/24 và 10.0.11.0/24.
3. Nhấn **Create**.

![Khởi tạo Redis Cache Subnet Group](/images/5-Workshop/5.7-ElastiCache-Redis/redis-subnet-group-create.png)

---

#### 2. Khởi tạo Redis Cache Cluster (Giao diện mới)

Chọn **ElastiCache** ➔ **Caches** (ở menu bên trái) ➔ Bấm **Create cache**.

---

##### 📝 BƯỚC 1: Cấu hình Settings

Tại màn hình **Step 1: Settings**, thiết lập các mục sau:

##### 1. Configuration (Cấu hình Engine)
- **Engine**: Tích chọn **Redis OSS** (Lưu ý: Không chọn *Valkey* để tương thích tốt nhất với Socket.io Redis Adapter của dự án).
- **Deployment option**: Tích chọn **Node-based cluster** (Sử dụng cluster dựa trên node để kiểm soát chi phí tối ưu, không chọn *Serverless*).
- **Creation method**: Tích chọn **Cluster cache** (Để hiển thị đầy đủ các tuỳ chọn cấu hình).
- **Cluster mode**: Tích chọn **Disabled** (Chỉ chạy node đơn cho môi trường phát triển thử nghiệm).

![Khởi tạo Redis - Cấu hình Settings Phần 1](/images/5-Workshop/5.7-ElastiCache-Redis/redis-create-settings-1.png)

##### 2. Cluster info
- **Name**: Nhập `realtime-collab-dev-redis`
- **Description**: Nhập `Redis 7 for realtime-collab`
- **Location**: Chọn **AWS Cloud**
- **Multi-AZ**: **Tích chọn (Enabled)**.

![Khởi tạo Redis - Cấu hình Settings Phần 2](/images/5-Workshop/5.7-ElastiCache-Redis/redis-create-settings-2.png)

##### 3. Cache settings
- **Engine version**: Chọn **7.0** (hoặc phiên bản 7.x khả dụng).
- **Port**: Giữ mặc định `6379`.
- **Parameter groups**: Chọn default.redis7.
- **Node type**: Click chọn và chọn dòng **cache.t3.micro**.
- **Number of replicas**: Nhập **`1`**.

![Khởi tạo Redis - Cache settings](/images/5-Workshop/5.7-ElastiCache-Redis/redis-create-settings-3.png)

##### 4. Connectivity (Kết nối mạng)
- **Network type**: Chọn IPv4.
- **Subnet groups**: Tích chọn **Choose existing subnet group** ➔ Chọn group `realtime-collab-dev-redis-subnet-group` đã tạo ở mục 1.
- **Availability Zone placements**: Giữ nguyên No preference.

![Khởi tạo Redis - Kết nối Connectivity](/images/5-Workshop/5.7-ElastiCache-Redis/redis-create-advanced-1.png)

Bấm **Next** để chuyển sang Bước 2.

---

##### ⚙️ BƯỚC 2: Cấu hình Advanced Settings

Tại màn hình **Step 2: Advanced settings**, cấu hình phần bảo mật:

##### 1. Security (Bảo mật & Mã hoá)
- **Encryption at rest**: Tích chọn **Enable** ➔ Chọn **Default key**.
- **Encryption in transit**: Tích chọn **Enable** (Rất quan trọng để bảo mật luồng thông tin truyền tải).
- **Access control**: Chọn **Redis AUTH default user**.
- **Auth token**: Nhập mã token xác thực (mật khẩu Redis) đã lưu trong Secrets Manager `redis-auth-token` ở Bước 1.

![Khởi tạo Redis - Advanced Settings](/images/5-Workshop/5.7-ElastiCache-Redis/redis-create-advanced-2.png)

- **Selected security groups**: Bấm nút **Manage** ➔ Tích chọn nhóm bảo mật **redis-sg** đã tạo ở Bước 4 (bỏ tích group default nếu có) ➔ Bấm **Save**.

![Khởi tạo Redis - Advanced Settings](/images/5-Workshop/5.7-ElastiCache-Redis/redis-create-advanced-3.png)

##### 2. Backup & Maintenance
- **Enable automatic backups**: Bỏ tích chọn.
- **Auto minor version upgrade**: Tích chọn **Enable**.

![Danh sách cụm Redis ở trạng thái khởi tạo](/images/5-Workshop/5.7-ElastiCache-Redis/redis-create-status.png)

Bấm **Next** để kiểm tra lại toàn bộ thông tin tại màn hình **Step 3: Review and create**.

Bấm **Create** để tiến hành khởi tạo.


---

#### 3. Thu thập thông tin kết nối

Quá trình tạo cụm Redis mất khoảng **5 - 10 phút**. Khi trạng thái hiển thị là Available:
1. Click vào tên cluster `realtime-collab-dev-redis` để xem chi tiết.
2. Sao chép địa chỉ **Primary Endpoint** (Dạng `realtime-collab-dev-redis.xxxxxx.ng.0001.use1.cache.amazonaws.com`).

