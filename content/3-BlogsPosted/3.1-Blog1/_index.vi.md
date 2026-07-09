---
title : "Blog 1"
date : 2024-01-01
weight : 3
chapter : false
pre : "<b>3.1</b>"
---

# Tự động hóa Deployment lên Amazon ECS Express Mode bằng GitHub Actions

Xin chào mọi người,

Đây là bài viết tổng hợp và chia sẻ lại giải pháp kỹ thuật về việc tối ưu hóa quy trình CI/CD cho các ứng dụng container hóa sử dụng **GitHub Actions** và **Amazon ECS Express Mode**. Giải pháp này tập trung vào hai yếu tố chính: đơn giản hóa quá trình vận hành và nâng cao tính bảo mật.

---

## 1. Các ưu điểm cốt lõi của giải pháp

- **Bảo mật thông qua OpenID Connect (OIDC):** Thay vì sử dụng các thông tin xác thực tĩnh như `AWS_ACCESS_KEY_ID` và `AWS_SECRET_ACCESS_KEY` lưu trong GitHub Secrets, giải pháp này thiết lập mối quan hệ tin cậy giữa GitHub và AWS qua OIDC. GitHub Actions sẽ tự động giả định (assume) một IAM Role ngắn hạn để thực thi tác vụ, tuân thủ nghiêm ngặt nguyên tắc phân quyền tối thiểu (Least Privilege).
- **Triển khai tối ưu với ECS Express Mode:** Tính năng Express Mode của **Amazon ECS** giúp tinh giản việc quản lý hạ tầng và cân bằng tải tự động. Khi kết hợp với Action chính thức *"Deploy Express Service"* từ AWS, quá trình cập nhật dịch vụ (rolling update) được thực hiện một cách nhanh chóng và ổn định.

---

## 2. Quy trình hoạt động

Khi có sự kiện push code hoặc merge Pull Request vào nhánh chính (ví dụ: `main`):

1. **GitHub Actions** kích hoạt quy trình runner.
2. Runner tiến hành build `Dockerfile` thành container image và đẩy (push) lên **Amazon Elastic Container Registry (ECR)**.
3. Action *"Deploy Express Service"* cập nhật cấu hình tác vụ và chuyển dịch lưu lượng sang phiên bản image mới trên **Amazon ECS Express Mode**.

---

## 3. Các bước triển khai chính

* **Bước 1:** Cấu hình OIDC Identity Provider trên **AWS IAM**, liên kết với URL của nhà cung cấp cấu hình từ GitHub (`token.actions.githubusercontent.com`).
* **Bước 2:** Tạo **IAM Role** chuyên dụng với các chính sách (policy) tối giản, chỉ cho phép ghi dữ liệu vào **Amazon ECR** và cập nhật dịch vụ trên **Amazon ECS Express Mode**.
* **Bước 3:** Sử dụng **GitHub Repository Variables** cho các thông tin cấu hình không nhạy cảm (như `AWS_REGION`, `ECR_REPOSITORY`, `ECS_SERVICE_NAME`) để dễ dàng quản lý và bảo trì workflow.
* **Bước 4:** Thiết lập file cấu hình quy trình hoàn chỉnh tại đường dẫn `.github/workflows/deploy.yml`.

---

## 4. Kết luận

Việc kết hợp **GitHub Actions** và **Amazon ECS Express Mode** thông qua cơ chế xác thực OIDC mang lại một giải pháp CI/CD toàn diện, vừa tối ưu hóa tốc độ triển khai tự động, vừa đáp ứng các tiêu chuẩn bảo mật nghiêm ngặt của doanh nghiệp. 

Phương án tiếp cận này giúp loại bỏ hoàn toàn rủi ro từ các thông tin xác thực tĩnh, giảm thiểu bề mặt tấn công bảo mật và đơn giản hóa quy trình quản lý hạ tầng container. Đây là một mô hình tham chiếu hiệu quả khi xây dựng chuỗi cung ứng phần mềm an toàn trên nền tảng AWS.

> **Bài viết gốc:** [AWS Containers Blog - Automated deployments with GitHub Actions for Amazon ECS Express Mode](https://aws.amazon.com/vi/blogs/containers/automated-deployments-with-github-actions-for-amazon-ecs-express-mode/)

![ConnectPrivate](/images/arcblog1.png)