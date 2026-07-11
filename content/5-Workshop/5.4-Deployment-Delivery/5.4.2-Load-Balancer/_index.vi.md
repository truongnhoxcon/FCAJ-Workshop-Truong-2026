---
title: "Application Load Balancer"
date: 2024-06-29
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

Chúng ta sẽ thiết lập bộ cân bằng tải Application Load Balancer (ALB) ở phân vùng Public để nhận các yêu cầu từ người dùng và định tuyến chính xác đến dịch vụ Backend tương ứng.

---

#### 1. Khởi tạo 2 Target Groups

Trước khi tạo Load Balancer, chúng ta cần chuẩn bị các Target Groups để gom nhóm IP của các ECS Tasks:

Truy cập dịch vụ **EC2** ➔ **Target Groups** (ở menu bên trái mục Load Balancing) ➔ **Create target group**. Tạo lần lượt 2 Target Groups:

##### Target Group 1: `realtime-collab-dev-uni-api-tg` (Dành cho Unified Backend REST API)
- **Target type**: IP addresses
- **Target group name**: `realtime-collab-dev-uni-api-tg`
- **Protocol**: HTTP | **Port**: `3000`

![Khởi tạo Target Group API - Cấu hình Port](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-api-1.png)

- **VPC**: Chọn realtime-collab-dev
![Khởi tạo Target Group API - Health Check Path](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-api-2.png)

- **Health checks**:
  - Health check path: `/health`
![Khởi tạo Target Group API - Health Check Path](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-api-2.1.png)
  - Advanced settings: Interval: `15s` | Timeout: `5s` | Healthy: `2` | Unhealthy: `2` | Success: `200`
![Khởi tạo Target Group API - Health Check Advanced](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-api-3.png)


##### Target Group 2: `realtime-collab-dev-uni-ws-tg` (Dành cho Unified Backend WebSocket)
- **Target type**: IP addresses
- **Target group name**: `realtime-collab-dev-uni-ws-tg`
- **Protocol**: HTTP | **Port**: `3000`

![Khởi tạo Target Group WS - Cấu hình Port](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-ws-1.png)

- **VPC**: Chọn realtime-collab-dev
![Khởi tạo Target Group WS - Health Check Path](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-ws-2.png)

- **Health checks**:
  - Health check path: `/health`
![Khởi tạo Target Group WS - Health Check Advanced](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-ws-3.png)
  - Advanced settings: Interval: `15s` | Timeout: `5s` | Healthy: `2` | Unhealthy: `2` | Success: `200`
![Khởi tạo Target Group WS - Stickiness Enable](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/tg-ws-4.png)


---

#### 2. Khởi tạo Application Load Balancer (ALB)

Truy cập **EC2** ➔ **Load Balancers** ➔ **Create Load Balancer** ➔ chọn **Application Load Balancer**:

- **Basic configuration**:
  - **Load balancer name**: `realtime-collab-dev-alb`
  - **Scheme**: Internet-facing
  - **IP address type**: IPv4

![Khởi tạo ALB - Tên và Scheme](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-1.png)

- **Network mapping**:
  - **VPC**: Chọn realtime-collab-dev
  - **Mappings**: Chọn 2 Availability Zones tương ứng:
    - AZ 1 (us-east-1a) ➔ Subnet: Chọn public-subnet-1 (10.0.1.0/24)
    - AZ 2 (us-east-1b) ➔ Subnet: Chọn public-subnet-2 (10.0.2.0/24)

![Khởi tạo ALB - Network Mappings Subnets](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-2.png)

- **Security groups**: Chọn alb-sg (Xóa bỏ group default nếu có).
![Khởi tạo ALB - Security Groups](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-3.png)

- **Listeners and routing**:
  - **Protocol**: HTTP | **Port**: `80`
![Khởi tạo ALB - Security Groups](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-3.1.png)
  - **Default action**: Chọn **Return fixed response** | Response code: `404` | Content-type: text/plain | Response body: `Not Found` (Vì Frontend sẽ được phân phối trực tiếp từ CloudFront + S3, ALB chỉ chịu trách nhiệm định tuyến dynamic API/WS).
![Khởi tạo ALB - Listeners default action 404](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-create-4.png)

Bấm **Create load balancer**.

---

#### 2.2 Cấu hình thuộc tính Idle Timeout (Bắt buộc cho WebSockets)

Do giao diện AWS Console mới đã đưa mục **Attributes** ra ngoài màn hình khởi tạo, bạn cần thiết lập thuộc tính này sau khi tạo thành công Load Balancer:

1. Tại danh sách **Load balancers**, nhấp chọn vào Load Balancer `realtime-collab-dev-alb` vừa tạo.
2. Di chuyển xuống phần chi tiết bên dưới ➔ Chọn tab **Attributes**.
3. Bấm nút **Edit** ở phía bên phải.
![Cấu hình Idle Timeout cho ALB - Attributes tab](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-idle-1.png)
1. Tìm đến dòng **Connection idle timeout**, thay đổi giá trị từ **`60`** thành **`3600`** giây (Bắt buộc để giữ kết nối WebSocket không bị ngắt giữa chừng).
2. Bấm **Save changes** để lưu lại.
![Cấu hình Idle Timeout cho ALB - Thay đổi giá trị 3600](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-idle-2.png)

1. Sao chép lại **DNS name** của Load Balancer (dạng `realtime-collab-dev-alb-XXXXXXXX.us-east-1.elb.amazonaws.com`) để chuẩn bị cho các bước tiếp theo.

---

#### 3. Thêm các luật định tuyến nâng cao (Listener Rules)

Chúng ta cần định tuyến: các request REST API đi vào `/api/*` tới Target Group API, và các request WebSocket đi vào `/ws/*` tới Target Group WS:

1. Vào trang chi tiết Load Balancer vừa tạo ➔ Chọn tab **Listeners and rules**.
2. Bấm vào listener **HTTP:80** để quản lý các rules ➔ Bấm **Add rule**.

![Thêm Listener Rules cho HTTP:80](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-1.png)

3. **Thêm Rule 1 — REST API**:
   - **Condition**: Chọn Path ➔ Nhập `/api/*`
   ![Định tuyến Rule 1 REST API /api/*](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-2.png)
   - **Routing action**: Chọn Forward to Target Group: realtime-collab-dev-uni-api-tg
   ![Định tuyến Rule 1 REST API Forward](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-3.png)
    - **Priority**: `1` ➔ Nhấn Next
   ![Định tuyến Rule 1 REST API Forward](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/rule-1.png)

4. **Thêm Rule 2 — WebSocket**:
   - **Condition**: Chọn Path ➔ Nhập `/ws/*`
![Định tuyến Rule 2 WebSocket /ws/*](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-4.png)
   - **Routing action**: Chọn Forward to Target Group: realtime-collab-dev-uni-ws-tg
![Định tuyến Rule 2 WebSocket Forward](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-5.png)
   - **Priority**: `2` ➔ Nhấn Next
![Danh sách Listener Rules đã cấu hình thành công](/images/5-Workshop/5.4-Deployment-Delivery/5.4.2-Load-Balancer/alb-rules-6.png)

