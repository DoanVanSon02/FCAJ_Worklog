---
title: "Xây dựng Web Server và lưu trữ dữ liệu"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---


### Tổng quan

Sau khi hoàn thành các bước chuẩn bị, tôi bắt đầu xây dựng hệ thống cần được bảo vệ trong workshop.

Trong phần này, một **Web Server** được triển khai trên **Amazon EC2** và sử dụng **Apache HTTP Server** để cung cấp nội dung Web. EC2 đóng vai trò là máy chủ chính của hệ thống và sẽ được sử dụng làm tài nguyên cần bảo vệ bằng AWS Backup ở các phần tiếp theo.

Bên cạnh Root Volume của EC2, tôi tạo thêm một **Amazon EBS Data Volume** riêng để lưu trữ dữ liệu ứng dụng. Data Volume được mount tại:

```text
/data
```

Việc tách dữ liệu ứng dụng khỏi Root Volume giúp quá trình kiểm thử Backup và Restore rõ ràng hơn, vì có thể trực tiếp kiểm tra trạng thái của các file dữ liệu trước và sau khi khôi phục.

Ngoài dữ liệu dạng block trên Amazon EBS, workshop còn sử dụng **Amazon S3** để lưu trữ dữ liệu dạng object.

Dữ liệu trên S3 được tổ chức trong prefix:

```text
application-data/
```

S3 Bucket được giữ ở trạng thái private, bật **Block Public Access** và **Versioning** để phục vụ cho quá trình Backup & Recovery ở các phần tiếp theo.


Trạng thái mong muốn sau khi hoàn thành phần 5.3 gồm:

```text
EC2 Web Server = Running

Apache = Active

Website = Truy cập được qua HTTP

EBS Data Volume = Mount tại /data

Dữ liệu thử nghiệm trên EBS = Đã tạo

S3 Bucket = Đã tạo

Block Public Access = Enabled

S3 Versioning = Enabled

Dữ liệu thử nghiệm trên S3 = Đã tạo
```

Ở thời điểm này, **EC2 chưa được cấp quyền truy cập Amazon S3**.

Tôi không sử dụng Access Key trực tiếp trên EC2. Quyền truy cập giữa EC2 và S3 sẽ được cấu hình thông qua **IAM Role** ở phần:

```text
5.4. Cấu hình IAM
```

### Nội dung

1. [Khởi tạo Amazon EC2](5.3.1.khởi-tạo-amazon-ec2/)
2. [Cài đặt Apache Web Server](5.3.2.cài-đặt-apache-web-server/)
3. [Tạo EBS Data Volume](5.3.3.tạo-ebs-data-volume/)
4. [Tạo Amazon S3 Bucket](5.3.4.tạo-amazon-s3-bucket/)
