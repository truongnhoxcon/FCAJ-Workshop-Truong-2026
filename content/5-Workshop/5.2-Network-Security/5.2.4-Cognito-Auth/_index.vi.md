---
title: "AWS Cognito User Pool"
date: 2024-06-29
weight: 4
chapter: false
pre: " <b> 5.2.4. </b> "
---

Chúng ta sẽ khởi tạo một **Amazon Cognito User Pool** để quản lý việc đăng ký, đăng nhập, xác thực email của người dùng và phát hành mã xác thực bảo mật mà không cần trực tiếp lưu trữ hay mã hóa mật khẩu ở database phía Backend.

---

#### Bước 1: Khởi tạo Cognito User Pool

1. Truy cập dịch vụ **Amazon Cognito** ➔ Chọn **Create user pool**.
2. **Configure sign-in experience (Cấu hình đăng nhập)**:
   - Cognito user pool sign-in options: Tích chọn **Email**.
   - Nhấn **Next**.
3. **Configure security requirements (Cấu hình bảo mật)**:
   - Password policy: Chọn **Cognito defaults** (Độ dài tối thiểu 8 ký tự, chứa số/chữ hoa/ký tự đặc biệt).
   - Multi-factor authentication (MFA): Chọn **No MFA** (Phục vụ cho môi trường học tập/thử nghiệm).
   - User account recovery: Tích chọn **Enable self-service account recovery** ➔ Chọn **Email only**.
   - Nhấn **Next**.
4. **Configure sign-up experience (Cấu hình đăng ký)**:
   - Self-service sign-up: Tích chọn **Enable self-service sign-up**.
   - Attribute verification: Tích chọn **Allow Cognito to automatically send messages to verify and confirm**.
   - Attributes to verify: Chọn **Send email message, verify email address**.
   - Required attributes: Đảm bảo đã tích chọn **email**.
   - Nhấn **Next**.
5. **Configure message delivery (Cấu hình gửi mail)**:
   - Email provider: Chọn **Send email with Cognito** (Giới hạn gửi 50 email mỗi ngày, phù hợp cho thử nghiệm).
   - Nhấn **Next**.
6. **Integrate app (Tích hợp ứng dụng)**:
   - User pool name: `realtime-collab-user-pool`
   - App client: Chọn **Public client**.
   - App client name: `realtime-collab-app-client`
   - Client secret: Chọn **Don't generate a client secret** (Bắt buộc đối với ứng dụng chạy hoàn toàn ở trình duyệt Client-side SPA như React).
   - Nhấn **Next**.
7. **Review and create**: Kiểm tra lại thông tin cấu hình và nhấn **Create user pool**.

---

#### Bước 2: Trích xuất thông tin kết nối

1. Nhấp chọn vào User Pool vừa khởi tạo (`realtime-collab-user-pool`).
2. Sao chép lại **User pool ID** ở phần thông tin chung (dạng `us-east-1_abcdef123`).
3. Chuyển qua tab **App integration** ➔ cuộn xuống phần **App client list** ➔ Sao chép lại **Client ID** (dạng `3n70abcde...`).

Chúng ta sẽ sử dụng 2 giá trị này để cấu hình biến môi trường cho cả Backend và Frontend ở các bước tiếp theo.
