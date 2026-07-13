---
title: "Security Groups"
date: 2024-06-29
weight: 4
chapter: false
pre: " <b> 5.2.4. </b> "
---

Chúng ta sẽ tạo các nhóm bảo mật (Security Groups) đóng vai trò tường lửa ảo kiểm soát lưu lượng ra vào cho từng lớp dịch vụ.

#### 1. Khởi tạo 4 Security Groups trống

Truy cập dịch vụ **VPC** ➔ **Security Groups** và chọn **Create security group**. Hãy tạo 4 nhóm bảo mật trống với VPC đã chọn là realtime-collab-dev:

| Tên nhóm bảo mật | Mô tả mục đích |
|---|---|
| `alb-sg` | ALB: Tiếp nhận HTTP port 80 từ internet |
| `unified-backend-sg` | ECS Unified Backend: Tiếp nhận port 3000 từ ALB |
| `rds-sg` | RDS PostgreSQL: Tiếp nhận port 5432 từ cụm ECS Backend |
| `redis-sg` | ElastiCache Redis: Tiếp nhận port 6379 từ cụm ECS Backend |

---

#### 2. Cấu hình chi tiết các Rules (Luật bảo mật)

Sau khi tạo xong các nhóm bảo mật "vỏ", tiến hành chỉnh sửa các **Inbound rules** và **Outbound rules** cho từng nhóm bằng cách tích chọn nhóm đó và bấm **Edit inbound/outbound rules**:

##### 2.1 Rules cho alb-sg
- **Inbound Rules**:
  - Type: HTTP | Port: 80 | Source: `0.0.0.0/0` | Description: `HTTP from internet`
- **Outbound Rules**:
  - Type: All traffic | Destination: `0.0.0.0/0`

![Cấu hình inbound rule cho alb-sg](/images/5-Workshop/5.2-Network-Security/5.2.3-Security-Groups/alb-sg.png)

##### 2.2 Rules cho unified-backend-sg
- **Inbound Rules**:
  - Type: Custom TCP | Port: `3000` | Source: chọn alb-sg | Description: `REST API and WS from ALB`
- **Outbound Rules**:
  - Type: All traffic | Destination: `0.0.0.0/0`

![Cấu hình inbound rule cho unified-backend-sg](/images/5-Workshop/5.2-Network-Security/5.2.3-Security-Groups/unified-backend-sg.png)

##### 2.3 Rules cho rds-sg
- **Inbound Rules**:
  - Type: PostgreSQL | Port: `5432` | Source: chọn backend-sg

![Cấu hình inbound rule cho rds-sg](/images/5-Workshop/5.2-Network-Security/5.2.3-Security-Groups/rds-sg.png)

##### 2.4 Rules cho redis-sg
- **Inbound Rules**:
  - Type: Custom TCP | Port: `6379` | Source: chọn backend-sg

![Cấu hình inbound rule cho redis-sg](/images/5-Workshop/5.2-Network-Security/5.2.3-Security-Groups/redis-sg.png)

