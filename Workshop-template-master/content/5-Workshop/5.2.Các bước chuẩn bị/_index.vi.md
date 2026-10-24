---
title: "Các bước chuẩn bị"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Tổng quan

Trước khi triển khai hệ thống **Backup & Recovery trên AWS**, cần chuẩn bị đầy đủ môi trường thực hiện, xác định các dịch vụ AWS sẽ sử dụng, Region triển khai, quyền truy cập cũng như các quy ước quản lý tài nguyên.

Việc chuẩn bị thống nhất ngay từ đầu giúp quá trình triển khai dễ theo dõi hơn, hạn chế nhầm lẫn giữa các tài nguyên và thuận tiện cho việc kiểm thử, khôi phục cũng như dọn dẹp tài nguyên sau khi hoàn thành workshop.

Phần chuẩn bị của workshop bao gồm các nội dung chính sau:

* **Tài khoản AWS và quyền truy cập:**  
  Kiểm tra tài khoản AWS và bảo đảm người thực hiện có đủ quyền để tạo và quản lý các tài nguyên EC2, EBS, S3, IAM, AWS Backup và CloudWatch.

* **Lựa chọn Region:**  
  Xác định Region được sử dụng xuyên suốt workshop để các tài nguyên được triển khai tập trung và dễ quản lý.

* **Công cụ sử dụng:**  
  Chuẩn bị các công cụ cần thiết như AWS Management Console, EC2 Instance Connect, Linux Shell, AWS CLI, trình duyệt Web và draw.io.

* **Lựa chọn các dịch vụ AWS:**  
  Xác định vai trò của Amazon EC2, Amazon EBS, Amazon S3, AWS IAM, AWS Backup và Amazon CloudWatch trong kiến trúc hệ thống.

* **Nguyên tắc bảo mật:**  
  Xác định các nguyên tắc như không hard-code Access Key, áp dụng Least Privilege, sử dụng IAM Role và hạn chế các tài nguyên public không cần thiết.

* **Quy ước đặt tên và Tag tài nguyên:**  
  Thống nhất cách đặt tên EC2 Instance, EBS Volume, IAM Role, IAM Policy, Backup Vault, Backup Plan và các tài nguyên khác để thuận tiện trong quá trình quản lý.

* **Chi phí dự kiến:**  
  Xác định các tài nguyên có khả năng phát sinh chi phí và giới hạn quy mô triển khai ở mức cần thiết cho bài lab.

Sau khi hoàn thành các bước chuẩn bị trên, môi trường AWS sẽ sẵn sàng để triển khai **Web Server, hệ thống lưu trữ dữ liệu và các thành phần Backup & Recovery** trong những phần tiếp theo.


#### Nội dung

1. [Tài khoản AWS và quyền truy cập](5.2.1.tài-khoản-aws-và-quyền-truy-cập/)
2. [Lựa chọn Region](5.2.2.lựa-chọn-region/)
3. [Công cụ sử dụng](5.2.3.công-cụ-sử-dụng/)
4. [Lựa chọn các dịch vụ AWS](5.2.4.lựa-chọn-các-dịch-vụ-aws/)
5. [Nguyên tắc bảo mật áp dụng](5.2.5.nguyên-tắc-bảo-mật/)
6. [Quy ước đặt tên và Tag tài nguyên](5.2.6.quy-ước-đặt-tên-và-tag-tài-nguyên/)
7. [Chi phí dự kiến](5.2.7.chi-phí-dự-kiến/)