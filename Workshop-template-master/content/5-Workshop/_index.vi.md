---
title: "Workshop"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# XÂY DỰNG HỆ THỐNG BACKUP & RECOVERY CHO WEB SERVER TRÊN AWS

#### Tổng quan

**AWS Backup** là dịch vụ quản lý sao lưu tập trung của Amazon Web Services, giúp tự động hóa và quản lý quá trình sao lưu dữ liệu trên nhiều dịch vụ AWS. Dịch vụ cho phép tạo các kế hoạch sao lưu, lưu trữ Recovery Point trong Backup Vault và khôi phục tài nguyên khi xảy ra sự cố.

Trong bài lab này, chúng ta sẽ xây dựng một Web Server trên **Amazon EC2**, sử dụng **Amazon EBS** để lưu trữ dữ liệu và **Amazon S3** để lưu trữ dữ liệu dạng object. Sau đó, hệ thống sẽ được tích hợp với **AWS Backup** để thực hiện sao lưu và khôi phục dữ liệu.

Ngoài ra, **AWS Identity and Access Management (IAM)** được sử dụng để kiểm soát quyền truy cập giữa các dịch vụ, trong khi **Amazon CloudWatch** được sử dụng để theo dõi trạng thái các Backup Job và hỗ trợ giám sát hoạt động sao lưu.

Sau khi hoàn thành cấu hình Backup, một tình huống sự cố sẽ được mô phỏng bằng cách thay đổi hoặc xóa dữ liệu trên EC2, EBS và S3. Các Recovery Point đã tạo trước đó sẽ được sử dụng để khôi phục hệ thống và kiểm tra khả năng phục hồi dữ liệu.


#### Nội dung

1. [Giới thiệu](<5.1.giới-thiệu/>)
2. [Các bước chuẩn bị](<5.2.các-bước-chuẩn-bị/>)
3. [Xây dựng Web Server và lưu trữ dữ liệu](5.3.xây-dựng-web-server-và-lưu-trữ-dữ-liệu/)
4. [Cấu hình IAM](5.4.cấu-hình-iam/)
5. [Cấu hình AWS Backup](5.5.cấu-hình-aws-backup/)
6. [Giám sát Backup bằng CloudWatch](5.6.giám-sát-backup-bằng-cloudwatch/)
7. [Mô phỏng sự cố](5.7.mô-phỏng-sự-cố/)
8. [Khôi phục hệ thống](5.8.khôi-phục-hệ-thống/)
9. [Dọn dẹp tài nguyên](5.9.dọn-dẹp-tài-nguyên/)