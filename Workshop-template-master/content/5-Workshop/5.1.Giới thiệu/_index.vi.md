---
title : "Giới thiệu"
date :  "`r Sys.Date()`" 
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---

### Giới thiệu về AWS Backup và Recovery

**AWS Backup** là dịch vụ sao lưu tập trung của Amazon Web Services, cho phép quản lý và tự động hóa quá trình sao lưu dữ liệu của nhiều tài nguyên AWS từ một giao diện thống nhất. Thông qua AWS Backup, người quản trị có thể xây dựng kế hoạch sao lưu, xác định thời gian lưu trữ, quản lý các bản sao lưu và thực hiện khôi phục khi xảy ra sự cố.

Trong AWS Backup, các bản sao lưu được quản lý dưới dạng **Recovery Point** và được lưu trong **Backup Vault**. Recovery Point đại diện cho trạng thái của tài nguyên tại một thời điểm nhất định và có thể được sử dụng để khôi phục tài nguyên khi dữ liệu bị thay đổi, xóa hoặc hệ thống gặp sự cố.

Trong workshop này, AWS Backup được sử dụng để bảo vệ hai nhóm tài nguyên chính:

* **Amazon EC2 và các EBS Volume gắn với EC2**: bảo vệ Web Server và dữ liệu lưu trữ trên máy chủ.
* **Amazon S3**: bảo vệ dữ liệu dạng object được sử dụng bởi ứng dụng.

Ngoài AWS Backup, workshop còn sử dụng **AWS Identity and Access Management (IAM)** để quản lý quyền truy cập giữa các dịch vụ và **Amazon CloudWatch** để theo dõi các chỉ số liên quan đến quá trình sao lưu.

---

### Tổng quan về workshop

Trong workshop này, một Web Server đơn giản sẽ được triển khai trên **Amazon EC2** và sử dụng **Apache HTTP Server** để cung cấp nội dung Web cho người dùng.

Một **Amazon EBS Data Volume** được gắn vào EC2 để lưu trữ dữ liệu cần bảo vệ. Bên cạnh đó, **Amazon S3** được sử dụng làm nơi lưu trữ dữ liệu dạng object của ứng dụng.

Để EC2 có thể truy cập S3 mà không cần lưu Access Key trực tiếp trên máy chủ, workshop sử dụng:

```text
BackupLab-EC2-S3-Role
```

kết hợp với:

```text
BackupLab-EC2-S3-Policy
```

IAM Role này được gắn vào EC2 và chỉ cung cấp các quyền S3 cần thiết.

AWS Backup sử dụng một IAM Role riêng:

```text
AWSBackupDefaultServiceRole
```

Role này cho phép AWS Backup thực hiện các thao tác cần thiết trong quá trình **Backup và Restore** tài nguyên.

Sau khi hệ thống được xây dựng, **AWS Backup** sẽ được cấu hình để sao lưu EC2/EBS và S3. Các bản backup được quản lý dưới dạng **Recovery Point** trong **Backup Vault**.

**Amazon CloudWatch** được sử dụng để theo dõi các metric của AWS Backup, qua đó giúp quan sát trạng thái của các Backup Job và cấu hình Alarm phục vụ giám sát.

Sau khi hoàn thành quá trình sao lưu, workshop sẽ mô phỏng một sự cố bằng cách thay đổi hoặc xóa dữ liệu. Các Recovery Point được tạo trước sự cố sau đó sẽ được sử dụng để khôi phục hệ thống.

---

### Kiến trúc tổng quan

![Kiến trúc tổng quan hệ thống Backup & Recovery trên AWS](/FCAJ_Worklog/images/5/5.1.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.1: Kiến trúc tổng quan hệ thống Backup & Recovery trên AWS</b>
</p>

Trong kiến trúc trên, người dùng truy cập Web Server trên **Amazon EC2** thông qua giao thức HTTP. EC2 sử dụng **Amazon EBS** để lưu trữ dữ liệu và sử dụng IAM Role để truy cập **Amazon S3**.

**AWS Backup** thực hiện sao lưu EC2 cùng các EBS Volume được bảo vệ và dữ liệu trên Amazon S3. Các Recovery Point được lưu trong **Backup Vault**.

Khi xảy ra sự cố, AWS Backup sử dụng Recovery Point để khôi phục EC2/EBS hoặc dữ liệu S3. **Amazon CloudWatch** nhận các metric từ AWS Backup để phục vụ việc giám sát quá trình sao lưu.