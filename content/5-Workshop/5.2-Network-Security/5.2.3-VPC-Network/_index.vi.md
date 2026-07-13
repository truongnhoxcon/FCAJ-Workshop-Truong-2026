---
title: "VPC và Mạng"
date: 2024-06-29
weight: 3
chapter: false
pre: " <b> 5.2.3. </b> "
---

Chúng ta sẽ thiết kế một hạ tầng mạng ảo an toàn Multi-AZ có chứa phân vùng mạng công cộng (Public Subnets) cho Load Balancer và phân vùng mạng riêng tư (Private Subnets) cô lập các dịch vụ.

#### 1. Tạo VPC

Truy cập dịch vụ **VPC** trên AWS Management Console, chọn **Create VPC** và chọn **VPC and more** (sử dụng Wizard tự động). Cấu hình chi tiết như sau:

- **Name tag auto-generation**: `realtime-collab-dev`
- **IPv4 CIDR block**: `10.0.0.0/16`
- **Tenancy**: Default
  ![Cấu hình VPC Wizard phần 1](/images/5-Workshop/5.2-Network-Security/5.2.2-VPC-Network/vpc-wizard-1.png)
- **Number of Availability Zones (AZs)**: 2 (`us-east-1a`, `us-east-1b`)
- **Number of Public Subnets**: 2
  - Public subnet 1 CIDR block: `10.0.1.0/24` (AZ 1)
  - Public subnet 2 CIDR block: `10.0.2.0/24` (AZ 2)
- **Number of Private Subnets**: 2
  - Private subnet 1 CIDR block: `10.0.10.0/24` (AZ 1)
  - Private subnet 2 CIDR block: `10.0.11.0/24` (AZ 2)
  
![Cấu hình VPC Wizard phần 2](/images/5-Workshop/5.2-Network-Security/5.2.2-VPC-Network/vpc-wizard-2.png)

- **NAT Gateways**: Regional
- **VPC Endpoints**: Chọn S3 Gateway
- **DNS Options**:
  - Enable DNS hostnames ➔ **Tích chọn**
  - Enable DNS resolution ➔ **Tích chọn**

![Cấu hình VPC Wizard phần 3](/images/5-Workshop/5.2-Network-Security/5.2.2-VPC-Network/vpc-wizard-3.png)

Sau đó, nhấn **Create VPC**. Wizard sẽ tự động tạo Internet Gateway, Route Tables (Public & Private), NAT Gateway, và Elastic IP tương ứng.

---

#### 2. Kích hoạt VPC Flow Logs

Nhằm ghi lại toàn bộ lịch sử lưu lượng truy cập IP trong VPC để phục vụ việc giám sát và bảo mật:

1. Truy cập trang chi tiết VPC vừa tạo.
2. Chọn tab **Flow logs** và nhấn **Create flow log**.

![Chọn tab Flow logs và bấm Create flow log](/images/5-Workshop/5.2-Network-Security/5.2.2-VPC-Network/flow-logs-1.png)

3. Cấu hình các thông số:
   - **Filter**: All
   - **Maximum aggregation interval**: 1 minute
   - **Destination**: Send to CloudWatch Logs
   - **Destination log group**: Chọn hoặc tạo mới Log Group tên `/aws/vpc/flow-logs/realtime-collab-dev`
   - **Service access**: Chọn **Create and use a new service role** để AWS tự động tạo IAM Role cấp quyền ghi logs cho VPC.

![Cấu hình chi tiết VPC Flow logs](/images/5-Workshop/5.2-Network-Security/5.2.2-VPC-Network/flow-logs-2.png)

4. Nhấn **Create flow log**.

