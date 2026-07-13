---
title: "Workshop"
date: 2024-06-29
weight: 5
chapter: false
pre: " <b> 5. </b> "
---


# Hướng dẫn Triển khai hệ thống AntiCollab lên AWS Management Console

#### Tổng quan

Trong phần này, chúng ta sẽ thực hiện triển khai thủ công toàn bộ hạ tầng và các dịch vụ của nền tảng **AntiCollab - Real-time Streaming Collaboration Platform** thông qua AWS Management Console. 

Mục tiêu của bài lab là giúp hiểu rõ cách cấu hình từng thành phần hạ tầng tương đương với mã nguồn Terraform của dự án, thiết lập kết nối mạng, cài đặt bảo mật và quản lý luồng dữ liệu một cách an toàn.

#### Thứ tự thực hiện (Bắt buộc)

Việc triển khai cần tuân thủ nghiêm ngặt theo trình tự dưới đây để đảm bảo các tài nguyên phụ thuộc được tạo đúng thứ tự:

1. [Giới thiệu](5.1-Overview/)
2. [Mạng & Bảo mật](5.2-Network-Security/)
   - [AWS Cognito User Pool](5.2-Network-Security/5.2.1-Cognito-Auth/)
   - [Secrets Manager](5.2-Network-Security/5.2.2-Secrets-Manager/)
   - [VPC và Mạng](5.2-Network-Security/5.2.3-VPC-Network/)
   - [Security Groups](5.2-Network-Security/5.2.4-Security-Groups/)
3. [Dữ liệu & Lưu trữ](5.3-Data-Storage/)
   - [S3 Buckets & IAM Roles](5.3-Data-Storage/5.3.1-S3-Buckets-IAM/)
   - [RDS PostgreSQL](5.3-Data-Storage/5.3.2-RDS-Database/)
   - [ElastiCache Redis](5.3-Data-Storage/5.3.3-ElastiCache-Redis/)
4. [Triển khai & Phân phối](5.4-Deployment-Delivery/)
   - [ECR & Push Images](5.4-Deployment-Delivery/5.4.1-ECR-Push/)
   - [ALB & Target Groups](5.4-Deployment-Delivery/5.4.2-Load-Balancer/)
   - [CloudFront Distribution](5.4-Deployment-Delivery/5.4.3-CloudFront/)
   - [ECS Cluster & Services](5.4-Deployment-Delivery/5.4.4-ECS-Fargate/)
5. [Dọn dẹp tài nguyên](5.5-Cleanup/)