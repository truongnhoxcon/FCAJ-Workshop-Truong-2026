---
title : "Giới thiệu"
date : 2024-06-29
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Kiến trúc hệ thống AntiCollab

Nền tảng **AntiCollab** được thiết kế dựa trên kiến trúc Cloud Native và Microservices chạy Serverless Container trên AWS. Toàn bộ hạ tầng mạng được thiết lập bảo mật nhiều tầng trên Multi-AZ.

#### Sơ đồ kiến trúc hạ tầng
Kiến trúc chi tiết của nền tảng được mô tả qua sơ đồ dưới đây:

![AntiCollab AWS Architecture](/images/2-Proposal/Diagram.png)

#### Tổng quan các bước thực hiện
Trong workshop này, chúng ta sẽ lần lượt đi qua các cấu hình chi tiết:
1. **AWS Cognito**: Khởi tạo danh bạ người dùng (User Pool) phục vụ đăng ký, đăng nhập và xác thực.
2. **AWS Secrets Manager**: Lưu trữ an toàn các thông tin nhạy cảm (RDS password, JWT, Twilio credentials, Redis AUTH Token và Cognito credentials).
3. **VPC và Mạng**: Thiết lập mạng ảo Multi-AZ, NAT Gateway và S3 Gateway Endpoint.
4. **Security Groups**: Cấu hình phân lớp tường lửa ảo bảo mật cổng kết nối cho các dịch vụ.
5. **S3 Buckets & IAM Roles**: Tạo các kho lưu trữ và các vai trò IAM (Task Role, Task Execution Role).
6. **RDS Database**: Khởi tạo cơ sở dữ liệu quan hệ PostgreSQL ở chế độ Multi-AZ.
7. **ElastiCache Redis**: Khởi tạo cụm bộ nhớ đệm cache và pub/sub ở chế độ Multi-AZ.
8. **ECR & Push Images**: Biên dịch Docker image và tải lên kho lưu trữ ECR.
9. **Application Load Balancer**: Thiết lập Target Groups và định tuyến định danh cho REST API & WebSockets.
10. **CloudFront Distribution**: Cấu hình CDN phân phối nội dung tĩnh, định tuyến API/WS về ALB, bảo mật qua AWS WAF (chế độ Monitor).
11. **ECS Fargate**: Cấu hình Task Definitions và chạy cụm container với 2 Tasks hoạt động song song.
12. **Dọn dẹp tài nguyên**: Trình tự xóa bỏ các tài nguyên tránh phát sinh chi phí.

*Luồng hoạt động của hệ thống (Architectural Flow)*
Luồng dữ liệu và tương tác giữa các dịch vụ tương ứng với các ký hiệu số trên sơ đồ kiến trúc:
- **1. Authenticate (Xác thực - Số 1)**: Client (Frontend) gửi thông tin đăng nhập trực tiếp tới **AWS Cognito User Pool**, Cognito xác thực và trả về JSON Web Token (JWT).
- **2. Access Application (Truy cập ứng dụng - Số 2)**: User truy cập trang web thông qua địa chỉ HTTPS/WSS của **CloudFront**, luồng đi qua tường lửa **AWS WAF** (được cấu hình ở chế độ Monitor mode).
- **3. Serve Static Content (Phân phối tài nguyên tĩnh - Số 3)**: CloudFront gửi yêu cầu qua cơ chế OAC tới **S3 Frontend** để tải về mã nguồn React app.
- **4. Dynamic Routing (Định tuyến động - Số 4)**: Các yêu cầu API (`/api/*`) hoặc WebSocket (`/ws/*`) được CloudFront chuyển tiếp về **Application Load Balancer (ALB)**. Client gửi kèm token JWT của Cognito trong Header `Authorization`.
- **5. Backend Processing (Xử lý ứng dụng - Số 5)**: ALB định tuyến yêu cầu tới **ECS Tasks** trong Private Subnet. Tại đây, Express và SocketAuth Middleware giải mã và kiểm tra tính hợp lệ của token JWT với AWS Cognito public keys. (Khi khởi chạy, ECS Task dùng Task Execution Role để kéo ảnh từ **Amazon ECR** và Task Role để lấy cấu hình từ **AWS Secrets Manager**).
- **6. Database & Cache Sync (Đồng bộ cơ sở dữ liệu & Cache - Số 6)**: Task kết nối với **RDS PostgreSQL (Port 5432 SSL)** để truy vấn/lưu dữ liệu và kết nối với **ElastiCache Redis (Port 6379 TLS)** để pub/sub đồng bộ sự kiện Socket.io thời gian thực.
- **7. File Upload (Tải lên tập tin - Số 7)**: ECS Task tạo Pre-signed URL gửi cho Client, Client tải tệp tin đính kèm trực tiếp lên **S3 Files** thông qua **VPC S3 Gateway Endpoint** nội bộ để đảm bảo bảo mật và tốc độ cao.
- **8. WebRTC Signaling (Tín hiệu cuộc gọi thoại - Số 8)**: ECS Task liên kết với **Twilio API** để lấy danh sách máy chủ TURN (ICE Servers) thông qua NAT Gateway và Internet Gateway (IGW) giúp người dùng thiết lập kết nối cuộc gọi voice/video trực tiếp (WebRTC).


