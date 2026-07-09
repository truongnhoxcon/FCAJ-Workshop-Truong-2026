---
title: "Đề xuất dự án"
date: 2024-06-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

#### Nền tảng Chat & Streaming thời gian thực AntiGroup
*Báo cáo đề xuất giải pháp kỹ thuật và hạ tầng cloud trên Amazon Web Services*

---

### 1. Giới thiệu tổng quan
Dự án **AntiGroup** hướng tới xây dựng một nền tảng giao tiếp cộng đồng thời gian thực (Real-time Collaboration Platform) tương tự Discord, cho phép người dùng trao đổi tin nhắn văn bản, cập nhật trạng thái hoạt động trực tuyến, và tham gia các cuộc gọi thoại/gọi video trực tiếp. Nền tảng này được thiết kế để phục vụ cho các nhóm sinh viên học tập, nghiên cứu và trao đổi học thuật trong môi trường phòng lab.

Để đảm bảo khả năng sẵn sàng cao (High Availability), độ trễ thấp dưới 200ms đối với tin nhắn, và khả năng co giãn linh hoạt theo lượng người dùng thực tế mà không đòi hỏi chi phí đầu tư hạ tầng máy chủ vật lý ban đầu lớn, hệ thống được thiết kế để triển khai toàn diện trên môi trường Cloud Native của **Amazon Web Services (AWS)**.

---

### 2. Mục tiêu giải pháp
Bản đề xuất giải pháp kỹ thuật này tập trung giải quyết các bài toán hạ tầng sau:
- **Thời gian thực với độ trễ tối thiểu**: Tận dụng giao thức WebSocket kết hợp với Redis Pub/Sub làm trung gian đồng bộ sự kiện giữa các container dịch vụ, đảm bảo tin nhắn văn bản và trạng thái người dùng (trực tuyến, bận, ngoại tuyến) được cập nhật ngay lập tức.
- **Tối ưu hóa gọi video nhóm**: Cấu hình WebRTC (Web Real-Time Communication) cho phép truyền phát âm thanh/hình ảnh trực tiếp peer-to-peer (P2P) giữa các trình duyệt. Đồng thời thiết lập cơ chế dự phòng STUN/TURN khi kết nối trực tiếp bị chặn bởi tường lửa hoặc NAT phức tạp.
- **Thử nghiệm nhanh hạ tầng**: Sử dụng mã nguồn Terraform để viết code IaC thử nghiệm nhanh tính tương thích của các mô hình kiến trúc hạ tầng mạng cơ bản, trước khi tiến hành triển khai chính thức.
- **Tiết kiệm chi phí vận hành**: Cơ chế tự động co giãn (Service Auto Scaling) của Fargate đảm bảo doanh nghiệp chỉ chi trả cho lượng tài nguyên CPU/RAM thực tế tiêu thụ tại mỗi thời điểm.
- **Bảo mật tối đa với chi phí tối thiểu**: Tận dụng CloudFront CDN để termination TLS/SSL toàn cục mà không cần mua các chứng chỉ SSL riêng đắt đỏ cho tên miền trong giai đoạn thử nghiệm/phát triển.

---

### 3. Kiến trúc giải pháp
Kiến trúc hệ thống Cloud-native trên AWS được thể hiện qua sơ đồ dưới đây:

![AntiGroup AWS Architecture](/images/2-Proposal/Diagram.png)

*Dịch vụ AWS sử dụng và Luồng hoạt động*
- **Amazon VPC & Subnets (Multi-AZ)**: Xây dựng hạ tầng mạng an toàn Multi-AZ chia thành Public Subnets (chứa ALB, NAT Gateway) và Private Subnets bảo vệ các ứng dụng (ECS Fargate Tasks, RDS PostgreSQL, ElastiCache Redis).
- **Amazon CloudFront & AWS WAF**: Làm cổng truy cập HTTPS/WSS bảo mật toàn cầu (Ingress), tích hợp tường lửa AWS WAF để lọc traffic xấu trước khi vào CloudFront. Phân phối web tĩnh trực tiếp từ S3 Frontend qua cơ chế OAC và chuyển tiếp API/WebSocket tới ALB.
- **Application Load Balancer (ALB)**: Nhận traffic từ CloudFront, thực hiện định tuyến: /api/* tới Target Group API (Round-robin) và /ws/* tới Target Group WS (Sticky session) của ECS Service.
- **Amazon ECS (Fargate)**: Chạy ứng dụng Serverless Container cho Unified Backend (tích hợp Node.js Express & Socket.io trên port 3000) phân bổ trên 2 Availability Zones (Task 1 và Task 2) để tự động dự phòng lỗi và cân bằng tải.
- **Amazon RDS (PostgreSQL)**: Cơ sở dữ liệu quan hệ chính hoạt động chế độ Multi-AZ (đồng bộ dữ liệu thời gian thực giữa Master DB ở AZ1 và Standby DB ở AZ2) để lưu trữ lâu dài thông tin người dùng, tin nhắn, server.
- **Amazon ElastiCache (Redis)**: Lưu trữ phân tán chế độ Multi-AZ (Primary node ở AZ1 đồng bộ bất đối xứng tới Replica node ở AZ2). Đóng vai trò làm Redis Pub/Sub đồng bộ sự kiện Socket.io giữa các ECS tasks và lưu cache trạng thái hoạt động tạm thời (ephemeral presence state) của người dùng.
- **Amazon S3 & VPC Gateway Endpoint**: Gồm bucket S3 Frontend lưu trữ mã nguồn tĩnh và bucket S3 Files lưu trữ tập tin đính kèm. ECS Tasks kết nối trực tiếp tới S3 Files qua VPC S3 Gateway Endpoint nội bộ để tối ưu băng thông và bảo mật tối đa.
- **AWS Secrets Manager & CloudWatch**: Quản lý tập trung các thông tin nhạy cảm (RDS password, JWT, Twilio credentials, Redis AUTH Token) được nạp trực tiếp vào ECS Tasks lúc khởi chạy (startup). Đồng thời các tasks ghi log trực tiếp về CloudWatch Logs.
- **NAT Gateway & Internet Gateway (IGW)**: Cung cấp đường truyền internet một chiều cho các ECS Tasks trong Private Subnets tải ảnh Docker từ ECR, đọc Secrets, gửi logs, và kết nối với Twilio API lấy cấu hình WebRTC TURN server.
- **AWS IAM**: Phân quyền chi tiết cho ECS Tasks qua ecsTaskRole (truy cập S3, Secrets Manager) và ecsTaskExecutionRole (kéo ảnh từ ECR, đẩy logs CloudWatch) nhằm đảm bảo nguyên tắc phân quyền tối thiểu (Least Privilege).

*Thiết kế thành phần*
- **Client (Frontend)**: Ứng dụng ReactJS viết trên Vite và TailwindCSS được lưu trữ tĩnh trên S3. Sử dụng useWebSocket.js để duy trì kết nối Socket.io trao đổi tin nhắn, cập nhật danh sách bạn bè và trạng thái trực tuyến. Sử dụng useWebRTC.js để khởi tạo kết nối gọi điện/video nhóm.
- **Unified Backend Service**: Tích hợp cả REST API Node.js/Express và Realtime Service/Signaling Server Node.js/Socket.io trong cùng một container. Giao tiếp với PostgreSQL thông qua Pool kết nối, đồng bộ hóa tin nhắn/trạng thái và điều phối gọi thoại qua Redis Pub/Sub, đồng thời xử lý các tác vụ CRUD, tạo Pre-signed URL cho S3 và tạo token JWT.

---

### 4. Triển khai kỹ thuật
*Các giai đoạn triển khai*
Dự án được cả nhóm thực hiện qua 4 giai đoạn cụ thể:
1. **Giai đoạn 1: Phân tích nghiệp vụ & Phát triển ứng dụng (Tuần 7 - 9)**: Các thành viên trong nhóm phân tích yêu cầu ứng dụng Chat & Video Streaming; lập trình Frontend (ReactJS) giao diện kiểu Discord và Unified Backend (Node.js/Express, Socket.io); kết nối cơ sở dữ liệu PostgreSQL và Redis cục bộ.
2. **Giai đoạn 2: Sơ đồ kiến trúc & Kiểm thử cục bộ (Tuần 10)**: Thiết kế chi tiết sơ đồ AWS Architecture trên draw.io; kiểm thử, ráp nối và chạy thử nghiệm cục bộ ứng dụng qua Docker Compose; sử dụng Terraform để thử nghiệm nhanh và xác thực tính thực tiễn các mô hình kiến trúc hạ tầng trên AWS.
3. **Giai đoạn 3: Triển khai hạ tầng dịch vụ trên AWS Console (Tuần 11)**: Thực hiện khởi tạo trực quan toàn bộ tài nguyên (VPC Multi-AZ, Secrets Manager, Security Groups, S3, RDS PostgreSQL Multi-AZ, ElastiCache Redis Multi-AZ, ALB, CloudFront + WAF, và ECS Cluster/Service Fargate) trên AWS Console; đóng gói Docker image và đẩy lên ECR.
4. **Giai đoạn 4: Kiểm thử hệ thống & Nghiệm thu (Tuần 12)**: Kiểm thử tích hợp hệ thống end-to-end trên AWS; kiểm thử chi tiết WebSocket chat, WebRTC calls (qua Twilio) và upload tệp; theo dõi log qua CloudWatch; tự đánh giá báo cáo thực tập và dọn dẹp các tài nguyên để tránh phát sinh chi phí.

*Yêu cầu kỹ thuật*
- **Môi trường chạy**: Docker Desktop trên máy phát triển cục bộ. AWS CLI v2 đã cấu hình quyền triển khai. Terraform phiên bản >= 1.6 để thi công hạ tầng.
- **Frontend**: NodeJS >= 18 để build ReactJS. Thư viện Socket.io-client phục vụ kết nối websocket, Lucide React cho các biểu tượng UI.
- **Backend**: NodeJS >= 18. PostgreSQL để lưu trữ dữ liệu bền vững. Redis để lưu trữ cache và pub/sub. Twilio API credentials được thiết lập sẵn cho dịch vụ TURN để phục vụ WebRTC cuộc gọi thoại.

---

### 5. Lộ trình & Mốc triển khai
Lộ trình thực hiện của nhóm được chia theo các mốc cụ thể:
- **Mốc 1 (Tuần 7 - 9)**:
  - Thiết kế cấu trúc cơ sở dữ liệu và hoàn thiện lập trình Unified Backend cùng React Frontend.
  - Chạy thử nghiệm và sửa các lỗi nghiệp vụ cơ bản dưới máy Local.
- **Mốc 2 (Tuần 10)**:
  - Thiết kế và vẽ sơ đồ kiến trúc hạ tầng AWS (draw.io) với đầy đủ chú thích luồng hoạt động.
  - Tích hợp, ráp nối và kiểm thử toàn diện ứng dụng cục bộ qua Docker Compose.
  - Viết cấu hình Terraform để kiểm nghiệm nhanh mô hình định tuyến mạng trên AWS.
- **Mốc 3 (Tuần 11)**:
  - Triển khai hạ tầng mạng, bảo mật và các kho lưu trữ (VPC, Secrets Manager, Security Groups, S3, IAM Roles).
  - Khởi tạo cụm RDS PostgreSQL và ElastiCache Redis chế độ Multi-AZ bảo mật cao.
  - Đóng gói container image Backend đẩy lên ECR; cấu hình Target Groups, ALB, CloudFront + WAF và triển khai ECS Service Fargate chạy 2 Tasks song song.
- **Mốc 4 (Tuần 12)**:
  - Kiểm thử tích hợp và vận hành đầu cuối toàn bộ hệ thống; kiểm chứng độ trễ WebSocket và media WebRTC.
  - Viết báo cáo tự đánh giá kết quả thực tập và tiến hành dọn dẹp hạ tầng.

---

### 6. Ước tính ngân sách
Dự án được ước tính chi phí hạ tầng hàng tháng tại khu vực us-east-1 như sau:

*Chi phí hạ tầng hàng tháng (Ước tính)*
- **Amazon ECS Fargate**: 2 Tasks hoạt động liên tục (0.5 vCPU, 1 GB RAM cho mỗi task) **$18.00/tháng**.
- **Amazon RDS (PostgreSQL)**: Lớp instance db.t3.medium, Multi-AZ, 20 GB dung lượng lưu trữ GP3 **$30.00/tháng**.
- **Amazon ElastiCache (Redis)**: Lớp instance cache.t3.micro Multi-AZ (1 replica) phục vụ Pub/Sub và lưu trữ trạng thái **$24.00/tháng**.
- **Application Load Balancer (ALB)**: 1 ALB chính định tuyến lưu lượng Backend **$22.26/tháng**.
- **Amazon CloudFront**: Phục vụ truyền tải tài nguyên tĩnh và bảo mật TLS (Free tier bao gồm 1 TB truyền dữ liệu đầu ra) **$2.00/tháng** (khi vượt quá Free Tier).
- **Amazon S3**: Lưu trữ tập tin & Frontend static files (dung lượng < 50 GB) **$1.20/tháng**.
- **AWS Secrets Manager**: Lưu trữ 4 secrets cần thiết cho hệ thống **$1.60/tháng** ($0.40/secret/tháng).
- **AWS CloudWatch**: Ghi nhận logs của ECS Task và lưu trữ chỉ số **$3.00/tháng**.
- **Mạng và Băng thông**: Truyền tải dữ liệu xuyên khu vực và ra Internet **$2.00/tháng**.

**Tổng cộng chi phí hạ tầng ước tính**: **$104.06/tháng**

---

### 7. Đánh giá rủi ro
*Ma trận rủi ro*
- **Rủi ro 1: WebRTC không thể thiết lập kết nối cuộc gọi voice/video do tường lửa/NAT đối xứng phức tạp.**
  - *Ảnh hưởng*: Cao.
  - *Chiến lược giảm thiểu*: Cấu hình dự phòng máy chủ TURN (Twilio Co-turn) để chuyển tiếp luồng media qua giao thức TCP/UDP nếu kết nối P2P trực tiếp bị chặn.
- **Rủi ro 2: Mất đồng bộ tin nhắn hoặc trạng thái online/offline của người dùng khi hệ thống tự động co giãn thêm container.**
  - *Ảnh hưởng*: Trung bình.
  - *Chiến lược giảm thiểu*: Sử dụng Redis Pub/Sub đồng bộ hóa thời gian thực các sự kiện nhắn tin và sự kiện hiện diện xuyên suốt các container của cụm dịch vụ thông qua presence.handler.js và chat.handler.js.
- **Rủi ro 3: Tràn chi phí hạ tầng AWS vượt quá ngân sách nghiên cứu.**
  - *Ảnh hưởng*: Trung bình.
  - *Chiến lược giảm thiểu*: Cài đặt AWS Budget Alerts cảnh báo khi chi phí chạm ngưỡng 50 USD và 80 USD. Cấu hình kịch bản tự động tắt các ECS Tasks ngoài giờ làm việc trong môi trường thử nghiệm.
- **Rủi ro 4: Rò rỉ thông tin cấu hình nhạy cảm (DB password, API keys) lên các kho lưu trữ mã nguồn mở.**
  - *Ảnh hưởng*: Rất cao.
  - *Chiến lược giảm thiểu*: Tuyệt đối không hardcode mật khẩu hay credentials vào mã nguồn; nạp toàn bộ biến cấu hình vào các ECS Container thông qua AWS Secrets Manager tại thời điểm chạy (runtime).

---

### 8. Kết quả kỳ vọng
*Cải tiến kỹ thuật*
- Triển khai thành công nền tảng trò chuyện và phát trực tuyến nhóm hoạt động trơn tru với độ trễ dưới 200ms đối với tin nhắn văn bản.
- Hỗ trợ gọi thoại và gọi video thời gian thực thông qua WebRTC ổn định cho các cuộc thảo luận nhóm phòng lab.
- Khảo sát nhanh và lưu vết cấu trúc mạng cơ bản dưới dạng mã nguồn Terraform, giúp dễ dàng tham chiếu khi cần thiết.

*Giá trị lâu dài*
- Xây dựng một kiến trúc tham chiếu mẫu (reference architecture) về hệ thống Chat & Streaming Cloud Native trên AWS cho các khóa sinh viên và dự án tiếp theo trong phòng lab.
- Quy trình triển khai và bàn giao ứng dụng được tối ưu hóa thông qua các script hỗ trợ đóng gói và đẩy image tự động (như deploy-ecr.sh), giúp tiết kiệm thời gian phát triển và bàn giao hạ tầng.