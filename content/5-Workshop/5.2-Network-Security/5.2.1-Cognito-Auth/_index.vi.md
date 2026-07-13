---
title: "AWS Cognito User Pool"
date: 2024-06-29
weight: 1
chapter: false
pre: " <b> 5.2.1. </b> "
---

Chúng ta sẽ khởi tạo một **Amazon Cognito User Pool** để quản lý việc đăng ký, đăng nhập, xác thực email của người dùng và phát hành mã xác thực bảo mật mà không cần trực tiếp lưu trữ hay mã hóa mật khẩu ở database phía Backend.

---

#### Bước 1: Khởi tạo Cognito User Pool (User Directory)

Giao diện mới của AWS Cognito cung cấp một trình hướng dẫn đơn giản hóa để cấu hình tài nguyên cho ứng dụng. Hãy thực hiện theo các bước sau:

1. Truy cập dịch vụ **Amazon Cognito** ➔ Chọn **Create user pool**.
![Khởi tạo Cognito User Pool](/images/5-Workshop/5.2-Network-Security/5.2.1-Cognito-Auth/create-user-pool.png)
2. Tại mục **Define your application (Định nghĩa ứng dụng)**:
   - **Application type**: Chọn **Single-page application (SPA)** (Vì frontend của AntiCollab được phát triển bằng React/Vite).
   - **Name your application**: Nhập tên ứng dụng, ví dụ: `realtime-collab-app`.
![Khởi tạo Cognito User Pool](/images/5-Workshop/5.2-Network-Security/5.2.1-Cognito-Auth/define-your-app.png)
3. Tại mục **Configure options (Cấu hình tùy chọn)**:
   - **Options for sign-in identifiers**: Tích chọn **Email**.
   - **Self-registration**: Tích chọn **Enable self-registration** (Cho phép người dùng tự đăng ký thông qua public APIs).
   - **Required attributes for sign-up**: Nhấp chọn menu thả xuống và chọn **email**.
![Khởi tạo Cognito User Pool](/images/5-Workshop/5.2-Network-Security/5.2.1-Cognito-Auth/configure-options.png)
4. Tại mục **Add a return URL - optional (Thêm URL phản hồi)**: Điền `http://localhost`.
5. Nhấn nút **Create user directory (Tạo danh bạ người dùng)** ở góc dưới cùng bên phải để hoàn tất khởi tạo.
![Khởi tạo Cognito User Pool](/images/5-Workshop/5.2-Network-Security/5.2.1-Cognito-Auth/add-a-return.png)

---

#### Bước 2: Trích xuất thông tin kết nối

1. Nhấp chọn vào User Pool vừa khởi tạo trong danh sách của Cognito.
2. Tại trang chi tiết của User Pool:
   - Sao chép lại **User pool ID** ở phần thông tin chung phía trên (dạng `us-east-1_abcdef123`).
![Trích xuất thông tin kết nối](/images/5-Workshop/5.2-Network-Security/5.2.1-Cognito-Auth/user-pool-id.png)
3. Di chuyển qua tab **App integration** ➔ cuộn xuống phần **App client list** ➔ Sao chép lại **Client ID** (dạng `3n70abcde...`).
![Trích xuất thông tin kết nối](/images/5-Workshop/5.2-Network-Security/5.2.1-Cognito-Auth/client-id.png)
Chúng ta sẽ sử dụng 2 giá trị này để cấu hình biến môi trường cho cả Backend và Frontend ở các bước tiếp theo.
