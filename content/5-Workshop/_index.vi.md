---
title: "Workshop"
date: 2024-06-29
weight: 5
chapter: false
pre: " <b> 5. </b> "
---


# Hướng dẫn Triển khai hệ thống AntiGroup lên AWS Management Console

#### Tổng quan

Trong phần này, chúng ta sẽ thực hiện triển khai thủ công toàn bộ hạ tầng và các dịch vụ của nền tảng **AntiGroup - Real-time Streaming Collaboration Platform** thông qua AWS Management Console. 

Mục tiêu của bài lab là giúp hiểu rõ cách cấu hình từng thành phần hạ tầng tương đương với mã nguồn Terraform của dự án, thiết lập kết nối mạng, cài đặt bảo mật và quản lý luồng dữ liệu một cách an toàn.

#### Thứ tự thực hiện (Bắt buộc)

Việc triển khai cần tuân thủ nghiêm ngặt theo trình tự dưới đây để đảm bảo các tài nguyên phụ thuộc được tạo đúng thứ tự:

1. [Giới thiệu](5.1-Workshop-overview/)
2. [Secrets Manager](5.2-Secrets-Manager/)
3. [VPC và Mạng](5.3-VPC-Network/)
4. [Security Groups](5.4-Security-Groups/)
5. [S3 Buckets & IAM Roles](5.5-S3-Buckets-IAM/)
6. [RDS PostgreSQL](5.6-RDS-Database/)
7. [ElastiCache Redis](5.7-ElastiCache-Redis/)
8. [ECR & Push Images](5.8-ECR-Push/)
9. [ALB & Target Groups](5.9-Load-Balancer/)
10. [CloudFront Distribution](5.10-CloudFront/)
11. [ECS Cluster & Services](5.11-ECS-Fargate/)
12. [Dọn dẹp tài nguyên](5.12-Cleanup/)