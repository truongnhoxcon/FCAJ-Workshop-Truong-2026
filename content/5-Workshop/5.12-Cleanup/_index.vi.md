---
title: "Dọn dẹp tài nguyên"
date: 2024-06-29
weight: 12
chapter: false
pre: " <b> 5.12. </b> "
---

Để tránh phát sinh các chi phí không mong muốn trên tài khoản AWS của bạn sau khi kết thúc workshop, hãy tiến hành dọn dẹp các tài nguyên theo thứ tự ngược lại dưới đây.

---

#### Thứ tự dọn dẹp (Khuyến nghị)

> [!CAUTION]
> Quá trình xóa tài nguyên là không thể hoàn tác. Hãy chắc chắn rằng bạn đã lưu trữ tất cả các log hoặc dữ liệu cần thiết trước khi thực hiện.

##### 1. Dọn dẹp cụm máy chủ Amazon ECS
1. Truy cập **ECS** ➔ **Clusters** ➔ cụm `realtime-collab-dev` ➔ tab **Services**.
2. Chọn Service `unified-backend` ➔ Bấm **Update service** ➔ Đặt **Desired tasks** về `0` ➔ Lưu lại.
3. Chờ cho tới khi các tasks dừng hoàn toàn (Running tasks = 0) ➔ Chọn Service và bấm **Delete**.
4. Quay lại trang Cluster và bấm **Delete cluster** để xóa toàn bộ cụm.

##### 2. Xóa Application Load Balancer (ALB)
1. Truy cập **EC2** ➔ **Load Balancers** ➔ Chọn `realtime-collab-dev-alb` ➔ Bấm **Actions** ➔ **Delete load balancer**.
2. Di chuyển sang mục **Target Groups** ở menu bên trái ➔ Chọn 2 target groups (`uni-api-tg`, `uni-ws-tg`) ➔ Bấm **Actions** ➔ **Delete**.

##### 3. Vô hiệu hóa và Xóa CloudFront Distribution
1. Truy cập **CloudFront** ➔ **Distributions** ➔ Chọn distribution của dự án ➔ Bấm **Disable**.
2. Đợi khoảng **5 phút** để quá trình vô hiệu hóa hoàn thành (Status chuyển sang `Disabled`).
3. Chọn tiếp distribution đó và bấm **Delete**.

##### 4. Xóa Amazon S3 Buckets
1. Truy cập **S3** ➔ Danh sách buckets.
2. Đối với từng bucket trong 2 buckets (`realtime-collab-frontend-<account-id>` và `realtime-collab-files-<account-id>`) của workshop, chọn bucket đó ➔ bấm **Empty** (để xóa toàn bộ tệp tin bên trong trước).
3. Sau khi Empty thành công, chọn tiếp bucket đó và bấm **Delete**. Nhập tên bucket để xác nhận xóa.

##### 5. Xóa Amazon RDS PostgreSQL
1. Truy cập **RDS** ➔ **Databases** ➔ Chọn `realtime-collab-dev-db` ➔ Bấm **Actions** ➔ **Delete**.
2. Bỏ tích chọn *Create final snapshot* và *Retain automated backups* (do đây là môi trường thử nghiệm). Tích chọn xác nhận và chọn **Delete**.
3. Di chuyển sang mục **Subnet groups** và **Parameter groups** để xóa các nhóm subnet và parameter tùy chỉnh đã tạo ở Bước 7.

##### 6. Xóa ElastiCache Redis
1. Truy cập **ElastiCache** ➔ **Redis OSS caches** ➔ Chọn cụm `realtime-collab-dev-redis` ➔ Bấm **Delete**.
2. Chọn không lưu trữ snapshot ➔ Bấm **Delete**.
3. Di chuyển sang mục **Subnet groups** để xóa subnet group tương ứng.

##### 7. Xóa kho lưu trữ ECR Repository
1. Truy cập **ECR** ➔ **Repositories**.
2. Chọn repo `unified-backend` ➔ Bấm **Delete**.

##### 8. Xóa AWS Secrets Manager Secrets
1. Truy cập **Secrets Manager** ➔ Chọn các secrets đã tạo.
2. Bấm **Actions** ➔ **Delete secret** ➔ Chọn thời gian chờ xóa ngắn nhất (ví dụ: `7 days`) để đưa vào hàng chờ xóa.

##### 9. Xóa mạng Amazon VPC
1. Truy cập **VPC** ➔ **Your VPCs**.
2. Chọn `realtime-collab-dev` ➔ Bấm **Actions** ➔ **Delete VPC**.
3. AWS sẽ hiển thị danh sách tất cả các tài nguyên liên kết mạng (Subnets, Route Tables, Internet Gateway, NAT Gateway, Security Groups) sẽ được tự động xóa kèm theo. Bấm **Delete VPC** để xác nhận xóa sạch toàn bộ hạ tầng mạng.
