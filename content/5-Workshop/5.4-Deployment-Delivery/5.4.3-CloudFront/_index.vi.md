---
title: "CloudFront Distribution"
date: 2024-06-29
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

Chúng ta sẽ cấu hình Amazon CloudFront làm điểm truy cập HTTPS bảo mật toàn cầu (Ingress) cho dự án, xử lý mã hóa TLS, truyền WebSocket, tích hợp lưu trữ tĩnh S3 và định tuyến lưu lượng động tới ALB.

---

#### 1. Các chính sách Cache và Request của CloudFront

Khi định tuyến các yêu cầu (requests) qua CloudFront, chúng ta cần cấu hình cách CloudFront xử lý bộ nhớ đệm (Cache) và các tiêu đề (Headers) gửi tới API/WebSocket:
- **Tắt Cache**: API và WebSocket là dữ liệu thời gian thực động, không được phép lưu cache.
- **Forward Headers**: WebSocket yêu cầu các headers đặc biệt (Upgrade, Connection) để thiết lập bắt tay.


---

#### 2. Khởi tạo CloudFront Distribution

Chọn **CloudFront** ➔ **Distributions** ➔ Bấm **Create distribution**.

---

##### 📝 PHẦN 2.1: Hoàn thành Trình khởi tạo Distribution mặc định (Focus S3)

Màn hình tạo mới hiện tại của CloudFront được thiết kế tối giản qua 5 bước để hướng dẫn khởi tạo nhanh với S3 Bucket Frontend. Hãy hoàn thành các bước này:

##### Bước 1: Choose a plan
- Tích chọn gói **Free ($0/month)** ➔ Bấm **Next**.

![Khởi tạo Distribution - Chọn gói cước](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-origin-1.png)


##### Bước 2: Get started
- **Distribution name**: Giữ mặc định hoặc nhập `s3-origin`.
- **Distribution type**: Tích chọn **Single website or app** (Tối ưu cho ứng dụng React SPA đơn vùng).
- **Domain (Route 53 managed domain)**: Để trống (Không bắt buộc) ➔ Bấm **Next**.

![Khởi tạo Distribution - Get started](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-origin-2.png)

##### Bước 3: Specify origin
- **Origin type**: Chọn **Amazon S3**.
- **S3 origin**: Chọn S3 bucket frontend của bạn (Dạng: `realtime-collab-frontend-<account-id>.s3.us-east-1.amazonaws.com`).
- **Origin path**: Để trống.
- Bấm **Next**.
![Khởi tạo Distribution - Specify origin](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-origin-3.png)


##### Bước 4: Enable security
- Tích chọn **Use monitor mode** ➔ Bấm **Next**.
![Khởi tạo Distribution - Enable security](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-security.png)

##### Bước 5: Review and create
- Kiểm tra lại toàn bộ thông tin ➔ Bấm **Create distribution**.
![Khởi tạo Distribution - Done](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-done.png)

##### Bước 6: Copy policy
1. Truy cập vào Distribution vừa tạo ➔ Chọn tab **Origins**.
2. Tích chọn S3 Origin (ví dụ: `realtime-collab-frontend-...`) ➔ Chọn **Edit**.
![Copy policy từ CloudFront](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-policy-1.png)
3. Cuộn xuống mục **Bucket policy**, thông báo màu vàng kèm nút **Copy policy** sẽ xuất hiện lại để bạn sao chép.
![Copy policy từ CloudFront](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/copy-policy.png)

4. Cấu hình Bucket Policy trên S3:
- Truy cập dịch vụ **Amazon S3** và nhấn chọn vào bucket **`realtime-collab-frontend-<account-id>`**.
- Chọn tab **Permissions** ở thanh menu ngang phía trên.
- Cuộn xuống mục **Bucket policy** và nhấn nút **Edit** ở bên phải.
![Dán Bucket Policy trong S3 Permissions](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-policy-2.png)
- Dán toàn bộ chính sách JSON đã copy ở CloudFront vào khung soạn thảo mã.
- Nhấn **Save changes** ở cuối trang để lưu lại.
![Origins tab list](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-alb-1.png)

---

##### ⚙️ PHẦN 2.2: Cấu hình nâng cao (Thêm ALB Backend & Định tuyến)

Sau khi khởi tạo xong Distribution, hãy nhấp chọn vào Distribution vừa tạo để cấu hình thêm nguồn dữ liệu động ALB và các quy tắc định tuyến:

##### 1. Cấu hình ALB Backend Origin
1. Chọn tab **Origins** ➔ Bấm nút **Create origin**.

![Khởi tạo ALB Backend Origin](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-alb-2.png)

2. Cấu hình:
   - **Origin domain**: Chọn hoặc dán DNS Name của ALB (Ví dụ: `realtime-collab-dev-alb-XXXXXXXX.us-east-1.elb.amazonaws.com`).
   - **Name**: Nhập `alb-origin` (hoặc để mặc định).
   - **Protocol**: Chọn **HTTP only** (Vì CloudFront giao tiếp với ALB qua port HTTP 80).
   - **HTTP port**: Giữ nguyên `80`.

![Khởi tạo ALB Backend Origin](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-behavior-1.png)

3. Bấm **Create origin**.

##### 2. Cấu hình các Behavior (Quy tắc định tuyến đường dẫn)
Di chuyển sang tab **Behaviors**. Mặc định sẽ có một behavior `Default (*)` trỏ về `s3-origin`. Ta cần tạo thêm 2 Behaviors cho API và WebSockets:

![Cấu hình Behavior cho Path /api/*](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-behavior-2.png)

* **Tạo Behavior 1 (Dành cho `/api/*` REST API)**:
  - Bấm **Create behavior**.
  - **Path pattern**: Nhập `/api/*`
  - **Origin**: Chọn alb-origin.
  - **Viewer protocol policy**: Chọn **Redirect HTTP to HTTPS**.
  - **Allowed HTTP methods**: Chọn **GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE**.
  - **Cache policy**: Chọn chính sách mặc định **CachingDisabled** (để tắt cache).
  - **Origin request policy**: Chọn chính sách mặc định **AllViewer** (để forward toàn bộ headers).
  - Nhấn **Create behavior**.

  ![Error Pages tab](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-error-1.png)

* **Tạo Behavior 2 (Dành cho `/ws/*` WebSockets)**:
  - Bấm **Create behavior**.
  - **Path pattern**: Nhập `/ws/*`
  - **Origin**: Chọn alb-origin.
  - **Viewer protocol policy**: Chọn **Redirect HTTP to HTTPS**.
  - **Allowed HTTP methods**: Chọn **GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE**.
  - **Cache policy**: Chọn **CachingDisabled**.
  - **Origin request policy**: Chọn **AllViewer**.
  - Nhấn **Create behavior**.

  ![Cấu hình Custom Error Response 403](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-error-2.png)


##### 3. Cấu hình Custom Error Responses (Bắt buộc cho React SPA Client Routing)
Khi người dùng truy cập trực tiếp các đường dẫn dynamic trên trình duyệt (như /servers, /login), S3 sẽ trả về lỗi 403 hoặc 404 vì các thư mục này không tồn tại vật lý. Ta cần rewrite lỗi này về index.html của React app:

1. Di chuyển sang tab **Error pages** ➔ Chọn **Create custom error response**.

![Cấu hình Custom Error Response 404](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-error-3.png)

2. Thiết lập lỗi 403:
   - **HTTP error code**: 403: Forbidden
   - **Customize error response**: Chọn Yes
   - **Response page path**: `/index.html`
   - **HTTP response code**: `200: OK`

   ![Danh sách Custom Error Responses](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-error-4.png)

3. Tiếp tục bấm **Create custom error response** để thiết lập lỗi 404:
   - **HTTP error code**: 404: Not Found
   - **Customize error response**: Chọn Yes
   - **Response page path**: `/index.html`
   - **HTTP response code**: `200: OK`

   ![General Settings Edit click](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-general-1.png)

##### 4. Cấu hình Root Object & IPv6
1. Di chuyển sang tab **General** ➔ Tại mục **Settings**, bấm nút **Edit**.

![Cấu hình Default Root Object index.html](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-general-2.png)

2. Cấu hình:
   - **Default root object**: Nhập **`index.html`** (Rất quan trọng để tải trang chủ từ S3).

   ![Default root object](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-status.png)

3. Bấm **Save changes** để lưu lại.


Quá trình deploy các thay đổi sẽ diễn ra từ **5 - 10 phút**. Khi Status chuyển sang trạng thái hoạt động bình thường, hãy sao chép lại địa chỉ **Distribution domain name** (dạng `xxxxxxxxxxxx.cloudfront.net`). Đây chính là Endpoint HTTPS duy nhất để truy cập vào ứng dụng.

![Địa chỉ Domain name CloudFront hoàn thành](/images/5-Workshop/5.4-Deployment-Delivery/5.4.3-CloudFront/cf-domain.png)

* Truy cập và trải nghiệm trực tiếp hệ thống tại địa chỉ: **`https://d2gqfyhv3cphx1.cloudfront.net`**
