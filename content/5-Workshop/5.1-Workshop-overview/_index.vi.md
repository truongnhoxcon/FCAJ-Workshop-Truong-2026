---
title : "Giới thiệu"
date : 2024-06-29
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Kiến trúc hệ thống AntiGroup

Nền tảng **AntiGroup** được thiết kế dựa trên kiến trúc Cloud Native và Microservices chạy Serverless Container trên AWS. Toàn bộ hạ tầng mạng được thiết lập bảo mật nhiều tầng trên Multi-AZ.

#### Sơ đồ kiến trúc hạ tầng
Kiến trúc chi tiết của nền tảng được mô tả qua sơ đồ dưới đây:

![AntiGroup AWS Architecture](/images/2-Proposal/Diagram.png)

#### Tổng quan các bước thực hiện
Trong workshop này, chúng ta sẽ lần lượt đi qua các cấu hình chi tiết:
1. **AWS Secrets Manager**: Lưu trữ an toàn các thông tin nhạy cảm.
2. **VPC và Mạng**: Thiết lập mạng ảo Multi-AZ, NAT Gateway và S3 Gateway Endpoint.
3. **Security Groups**: Phân lớp các quy tắc bảo mật cổng kết nối.
4. **S3 Buckets & IAM Roles**: Tạo các kho lưu trữ và vai trò phân quyền.
5. **RDS Database**: Khởi tạo cơ sở dữ liệu quan hệ PostgreSQL ở chế độ Multi-AZ.
6. **ElastiCache Redis**: Khởi tạo cụm bộ nhớ đệm cache và pub/sub ở chế độ Multi-AZ.
7. **ECR & Push Images**: Build và push các container images của Unified Backend lên ECR.
8. **Application Load Balancer**: Tạo Target Groups và cấu hình định tuyến cho API & WebSocket.
9. **CloudFront Distribution**: Cấu hình HTTPS Ingress toàn cầu và tích hợp AWS WAF bảo vệ.
10. **ECS Fargate**: Cấu hình Task Definitions và chạy Services với 2 Tasks song song.
11. **Dọn dẹp tài nguyên**: Trình tự xóa bỏ các tài nguyên tránh phát sinh chi phí.


