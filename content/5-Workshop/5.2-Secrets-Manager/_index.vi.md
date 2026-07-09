---
title: "AWS Secrets Manager"
date: 2024-06-29
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

Để bảo mật hoàn toàn mật khẩu và API Keys khỏi mã nguồn, chúng ta sẽ bắt đầu bằng cách tạo các kho lưu trữ bí mật tập trung.

#### Tạo 4 Secrets trên AWS Secrets Manager

Truy cập dịch vụ **AWS Secrets Manager** và chọn **Store a new secret**. Thiết lập tất cả các secret này với chế độ **Recovery window: 7 days** để tiết kiệm tài nguyên thử nghiệm.

---

##### 1. `realtime-collab-dev/db-password` (Mật khẩu cơ sở dữ liệu)
- **Type**: Other type of secret ➔ tab `Plaintext`
- **Value**: Nhập chuỗi ngẫu nhiên 32 ký tự
- **Secret name**: `realtime-collab-dev/db-password`
- **Description**: RDS PostgreSQL master password

![Chọn kiểu và nhập mật khẩu db-password](/images/5-Workshop/5.2-Secrets-Manager/db-password-1.png)
![Cấu hình tên và mô tả db-password](/images/5-Workshop/5.2-Secrets-Manager/db-password-2.png)

##### 2. `realtime-collab-dev/jwt-secret` (Khóa bảo mật JWT)
- **Type**: Other type of secret ➔ tab `Key/value`
- **Key/Value**:
  - Key: `secret`
  - Value: [Chuỗi ngẫu nhiên dài ít nhất 64 ký tự]
- **Secret name**: `realtime-collab-dev/jwt-secret`

![Chọn kiểu và nhập khóa jwt-secret](/images/5-Workshop/5.2-Secrets-Manager/jwt-secret-1.png)
![Cấu hình tên jwt-secret](/images/5-Workshop/5.2-Secrets-Manager/jwt-secret-2.png)

##### 3. `realtime-collab-dev/redis-auth-token` (Mã xác thực Redis)
- **Type**: Other type of secret ➔ tab `Plaintext`
- **Value**: [Chuỗi ngẫu nhiên 32 ký tự]
- **Secret name**: `realtime-collab-dev/redis-auth-token`

![Chọn kiểu và nhập token redis-auth-token](/images/5-Workshop/5.2-Secrets-Manager/redis-token-1.png)
![Cấu hình tên redis-auth-token](/images/5-Workshop/5.2-Secrets-Manager/redis-token-2.png)

##### 4. `realtime-collab-dev/twilio/api-credentials` (Cấu hình Twilio TURN)
- **Type**: Other type of secret ➔ tab `Key/value`
- **Key/Value**:
  - Key `accountSid` ➔ Value: [Twilio Account SID]
  - Key `authToken` ➔ Value: [Twilio Auth Token]
  - Key `apiKeySid` ➔ Value: [Twilio API Key SID]
  - Key `apiKeySecret` ➔ Value: [Twilio API Key Secret]
- **Secret name**: `realtime-collab-dev/twilio/api-credentials`

![Chọn kiểu và nhập thông tin twilio](/images/5-Workshop/5.2-Secrets-Manager/twilio-credentials-1.png)
![Cấu hình tên twilio](/images/5-Workshop/5.2-Secrets-Manager/twilio-credentials-2.png)

---

Sau khi tạo xong, hãy lưu lại **ARN** của từng secret. Sẽ cần ở các bước cấu hình IAM Roles và ECS Task Definitions tiếp theo.

