---
title : "Blog 2"
date : 2024-01-01
weight : 4
chapter : false
pre : "<b>3.2</b>"
---

# Triển khai Internal DNS cho Internet-facing Load Balancer trên AWS bằng Kiến trúc Event-driven

Chào các bạn, 

Trong quá trình tìm hiểu và làm việc với hệ thống mạng trên AWS, **Elastic Load Balancing (ELB)** là một dịch vụ nền tảng thường xuyên được sử dụng. Tuy nhiên, khi cấu hình các bộ cân bằng tải này, có một thách thức về mặt phân giải tên miền (DNS) mà chúng ta cần đặc biệt lưu ý.

---

## 1. Phân tích vấn đề

Khi triển khai **Network Load Balancer (NLB)** hoặc **Application Load Balancer (ALB)** ở chế độ *Internet-facing* (hướng ra internet), AWS sẽ cung cấp một tên miền DNS ngẫu nhiên (ví dụ: `nlb-123...elb.us-east-1.amazonaws.com`).

Đặc điểm của tên miền này là **chỉ phân giải ra địa chỉ Public IP**. AWS không cung cấp sẵn bất kỳ bản ghi nào để phân giải về địa chỉ Private IP của bộ cân bằng tải đó. Hạn chế này gây ra rất nhiều khó khăn trong các kịch bản thực tế:
- Khi sử dụng các cổng tường lửa (firewall gateway) của bên thứ ba nằm giữa ALB/NLB và Internet Gateway, yêu cầu phải kiểm tra lưu lượng đầu vào dựa trên DNS nội bộ.
- Khi cần định tuyến lưu lượng từ mạng nội bộ (On-premises) lên thẳng Load Balancer thông qua các kết nối **AWS Direct Connect**.

---

## 2. Tổng quan giải pháp

Để giải quyết bài toán trên, chúng ta có thể xây dựng một giải pháp tự động tạo và quản lý các **Private Hosted Zones** trên **Amazon Route 53**. Nhiệm vụ của hệ thống này là lưu trữ và liên tục cập nhật các bản ghi DNS nội bộ cho các Internet-facing Load Balancers.

Giải pháp này được thiết kế tối ưu cho môi trường đa tài khoản (Multi-account) sử dụng **AWS Organizations**, kết hợp sức mạnh của **AWS CloudTrail**, **Amazon EventBridge**, **AWS Lambda** và **Amazon DynamoDB**.

---

## 3. Chi tiết kiến trúc triển khai

Hệ thống được chia thành hai thành phần chính, đặt tại các tài khoản AWS khác nhau:

### A. Tại Tài khoản Dịch vụ dùng chung (Shared Services Account)
- **Amazon Route 53:** Khởi tạo 2 Private Hosted Zones cho mỗi Region:
    - ALB sử dụng hậu tố: `internal.region.elb.amazonaws.com`
    - NLB sử dụng hậu tố: `internal.elb.region.amazonaws.com`
- **AWS Lambda:** Triển khai 2 hàm xử lý:
    - `r53-scavenger`: Chạy theo lịch trình (cron job) để thu thập các Private IP hiện có của ALB/NLB và nạp vào Route 53 trong lần chạy đầu tiên.
    - `r53-updater`: Hàm bắt các sự kiện vòng đời (tạo, cập nhật, mở rộng, xóa) để duy trì tính chính xác tức thì của bản ghi DNS.
- **Amazon DynamoDB:** Bảng cơ sở dữ liệu dùng để lưu trữ trạng thái của các bộ cân bằng tải và giao diện mạng (ENI), giúp tối ưu hiệu suất và giảm thiểu các lệnh gọi API tra cứu chéo tài khoản.
- **Amazon EventBridge Custom Bus:** Đóng vai trò trung tâm tiếp nhận các sự kiện từ các tài khoản thành viên (Workload Accounts) và kích hoạt hàm Lambda `r53-updater`.

### B. Tại các Tài khoản Thực thi (Workload Accounts)
- Cấu hình một **IAM Role** cấp quyền cho hàm Lambda `r53-scavenger` ở tài khoản trung tâm có thể truy cập và thu thập dữ liệu cấu hình.
- Thiết lập **EventBridge Rule** để bắt các luồng sự kiện từ CloudTrail và định tuyến chúng về Custom Bus ở tài khoản dùng chung.

---

## 4. Luồng hoạt động tự động

Khi hệ thống đi vào hoạt động, quá trình cập nhật IP sẽ diễn ra hoàn toàn tự động theo mô hình hướng sự kiện (Event-driven):

```text
[Hành động người dùng / Hệ thống]
               │
               ▼
   [AWS CloudTrail ghi nhận] (API: CreateLoadBalancer, CreateNetworkInterface,...)
               │
               ▼
[EventBridge Rule tại Workload Account] ──(Chuyển tiếp)──> [Custom Bus tại Shared Services]
                                                                        │
                                                                        ▼
                                                             [Lambda r53-updater]
                                                                        │
                                                       ┌────────────────┴────────────────┐
                                                       ▼                                 ▼
                                            [Cập nhật bảng DynamoDB]         [Đồng bộ bản ghi Route 53]

```
> **Bài viết gốc:** [AWS Networking & Content Delivery Blog - Deploying internal DNS zones for internet-facing load balancers](https://aws.amazon.com/blogs/networking-and-content-delivery/deploying-internal-dns-zones-for-internet-facing-load-balancers/)

![ConnectPrivate](/images/arcblog2.png)