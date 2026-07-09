---
title: "Worklog Tuần 4"
date: 2026-05-11
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

* Hiêu về IAM, Cognito, SSO, AWS Organizations.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                                                                                                                                                               | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                                                                      |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ------------------------------------------------------------------------------------------------------------------- |
| 2   | - Học lý thuyết về IAM, Cognito, SSO, Organizations. <br> - Tìm hiểu dịch vụ AWS KMS và AWS Security Hub. <br> - Kích hoạt AWS Security Hub, đánh giá điểm số bảo mật.                                                                                                                                                                  | 11/05/2026   | 11/05/2026      | <https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i> <br> <https://000018.awsstudygroup.com/> |
| 3   | - Quản lý Tags chuyên sâu trên Console và CLI, sử dụng Resource Group. <br> - Khởi tạo VPC, Security Group, EC2 và thiết lập Webhook gửi thông báo về Slack. <br> - Viết IAM Role cho Lambda. Tạo các hàm Lambda để tự động tắt (Stop) và bật (Start) EC2.                                                                              | 12/05/2026   | 12/05/2026      | <https://000027.awsstudygroup.com/> <br> <https://000022.awsstudygroup.com/>                                        |
| 4   | - Tạo IAM User, thiết lập Policy/Role và thử tính năng Switch Roles. <br> - Truy cập console đa Region, kiểm tra sự chặt chẽ của Policy khi thiếu Tag hợp lệ.                                                                                                                                                                           | 13/05/2026   | 13/05/2026      | <https://000028.awsstudygroup.com/>                                                                                 |
| 5   | - Tạo Restriction Policy, tài khoản IAM Limited User và kiểm thử giới hạn. <br> - Tạo Policy, Group, S3 Bucket và mã hóa với AWS KMS. Thiết lập AWS CloudTrail và dùng Amazon Athena truy vấn nhật ký. <br> - Thực hiện kiểm tra và chia sẻ dữ liệu đã được mã hóa trên không gian lưu trữ S3.                                          | 14/05/2026   | 14/05/2026      | <https://000030.awsstudygroup.com/> <br> <https://000033.awsstudygroup.com/>                                        |
| 6   | <br> - Khởi tạo IAM Group, IAM Users, kiểm tra quyền hạn. Tạo Admin IAM Role và chuyển đổi vai trò (Switch role), giới hạn Switch role theo IP và thời gian, dọn dẹp tài nguyên. <br> - Khởi tạo EC2, S3 bucket, tạo người dùng IAM mới và Access Key. Sử dụng Access Key và giải pháp bảo mật hơn bằng cách dùng IAM Role gán cho EC2. | 15/05/2026   | 15/05/2026      | <https://000044.awsstudygroup.com/> <br> <https://000048.awsstudygroup.com/>                                        |

### Kết quả đạt được tuần 4:

* Thiết lập hàng rào bảo mật chặt chẽ bằng IAM, biết cách giới hạn quyền và kiểm soát việc tạo tài nguyên bằng Tagging.
* Tiết kiệm chi phí đám mây thông qua việc tự động hóa lịch bật/tắt máy chủ bằng AWS Lambda kết hợp cảnh báo qua Slack.
* Cấu hình thành công các giới hạn bảo mật nâng cao, chỉ cho phép người dùng chuyển đổi vai trò khi truy cập từ một IP cụ thể hoặc trong một khoảng thời gian được định trước.
* Hiểu được sự khác biệt giữa việc cấp quyền trực tiếp bằng Access Key so với việc sử dụng IAM Role an toàn hơn khi cho phép EC2 tương tác với các dịch vụ khác.
