---
title: "Worklog Tuần 3"
date: 2026-05-04
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Hiểu các dịch vụ lưu trữ trên AWS

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                                                                                        | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                                                             |
| --- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | -------------------------------------------------------------------------- |
| 2   | - Tìm hiểu về S3: <br> &emsp; + Access Point, Storage Class. <br> &emsp; + Static Website, CORS.                                                                                                                                                                 | 04/05/2026   | 04/05/2026      | <https://www.youtube.com/playlist?list=PLahN4TLWtox2a3vElknwzU_urND8hLn1i> |
| 3   | - Cài đặt VMWare Workstation. <br> - Thực hành trích xuất máy ảo từ On-premises, tải lên AWS và triển khai thành một máy chủ ảo EC2 thực tế từ AMI. <br> - Trích xuất máy ảo từ AWS EC2 mang về lại môi trường On-premises.                                      | 05/05/2026   | 05/05/2026      | <https://000014.awsstudygroup.com/>                                        |
| 4   | - Khởi tạo Storage Gateway, tạo File Shares trên AWS và Mount các File shares này trực tiếp lên máy tính ở môi trường On-premises.                                                                                                                               | 06/05/2026   | 06/05/2026      | <https://000024.awsstudygroup.com/>                                        |
| 5   | - Khởi tạo hệ thống tệp đa vùng sẵn sàng cao (Multi-AZ) sử dụng cả ổ cứng SSD và HDD. <br> - Tạo các File shares mới, tiến hành chạy kiểm tra hiệu năng và giám sát hiệu năng. <br> - Khử trùng lặp dữ liệu để tiết kiệm dung lượng và bật tính năng sao lưu ẩn. | 07/05/2026   | 07/05/2026      | <https://000025.awsstudygroup.com/>                                        |
| 6   | - Thực hành quản lý phiên truy cập của người dùng và các tệp đang mở. <br> - Thiết lập hạn mức lưu trữ cho từng người dùng, cấu hình mở rộng thông lượng và dung lượng.                                                                                          | 08/05/2026   | 08/05/2026      | <https://000025.awsstudygroup.com/>                                        |

### Kết quả đạt được tuần 3:
* Hiểu về dịch vụ S3
* Thực hiện thành công việc di chuyển hai chiều (Import/Export) một máy ảo thực tế giữa môi trường cục bộ (VMWare) và đám mây AWS.
* Cấu hình thành công hệ thống Backup tự động có kèm thông báo.
