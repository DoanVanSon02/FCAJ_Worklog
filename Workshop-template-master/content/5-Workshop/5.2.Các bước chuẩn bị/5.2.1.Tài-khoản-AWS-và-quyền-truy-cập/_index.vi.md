---
title: "Tài khoản AWS và quyền truy cập"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.2.1. </b> "
---

### Tài khoản AWS và quyền truy cập

Workshop yêu cầu một **AWS Account** có khả năng tạo và quản lý các tài nguyên thuộc những dịch vụ sau:

* **Amazon EC2 và Amazon EBS**
* **Amazon S3**
* **AWS Identity and Access Management (IAM)**
* **AWS Backup**
* **Amazon CloudWatch**

Tài khoản thực hiện workshop cần có đủ quyền để tạo và quản lý các tài nguyên cần thiết trong quá trình triển khai, bao gồm:

* EC2 Instance
* EBS Volume
* S3 Bucket
* IAM Role và IAM Policy
* Backup Vault
* Backup Plan
* CloudWatch Alarm

Trong quá trình xây dựng hệ thống, tôi **không sử dụng Access Key được lưu trực tiếp trên EC2**. Thay vào đó, quyền truy cập giữa các dịch vụ được cấp thông qua **IAM Role**.

Cách triển khai này giúp hạn chế việc lưu trữ thông tin xác thực trực tiếp trên máy chủ và phù hợp hơn với nguyên tắc bảo mật khi làm việc với các dịch vụ AWS.

Đối với Web Server EC2, quyền truy cập Amazon S3 sẽ được cung cấp thông qua IAM Role:

```text
BackupLab-EC2-S3-Role