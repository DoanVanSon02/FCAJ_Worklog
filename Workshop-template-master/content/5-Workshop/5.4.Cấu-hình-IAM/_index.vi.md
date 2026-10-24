---
title: "Cấu hình IAM"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Tổng quan

Sau khi hoàn thành EC2, EBS và Amazon S3, hệ thống đã có đầy đủ các tài nguyên cần thiết để phục vụ Web Server và lưu trữ dữ liệu. Tuy nhiên, tại thời điểm này EC2 vẫn chưa được cấp quyền truy cập vào Amazon S3.

Trong workshop này, tôi **không cấu hình Access Key và Secret Access Key trực tiếp trên EC2**. Thay vào đó, quyền truy cập được cung cấp thông qua **IAM Role** được gắn trực tiếp vào EC2 Instance.

Cách tiếp cận này giúp ứng dụng chạy trên EC2 nhận **temporary credentials** từ AWS thay vì phải lưu trữ thông tin xác thực dài hạn trên máy chủ.

Luồng phân quyền được sử dụng như sau:

```text
BackupLab-EC2-S3-Policy
          │
          ▼
BackupLab-EC2-S3-Role
          │
          ▼
      Amazon EC2
          │
          ▼
      Amazon S3
   application-data/*
```

Trong đó:

* **BackupLab-EC2-S3-Policy** xác định các quyền EC2 được phép thực hiện đối với Amazon S3.
* **BackupLab-EC2-S3-Role** là IAM Role được gắn vào EC2.
* **Amazon EC2** sử dụng IAM Role để nhận temporary credentials.
* Quyền truy cập được giới hạn vào dữ liệu trong prefix `application-data/` của S3 Bucket.

Mục tiêu của phần này là cho phép EC2 thực hiện các thao tác cần thiết:

```text
EC2 có thể:

✓ Liệt kê dữ liệu cần thiết trên S3
✓ Đọc object
✓ Upload object
```

Đồng thời, EC2 không được cấp những quyền không cần thiết:

```text
EC2 không thể:

✗ Xóa object
```

Workshop cũng không sử dụng:

```text
✗ Access Key hard-code
✗ AmazonS3FullAccess
✗ AdministratorAccess
```

Cách cấu hình này giúp áp dụng nguyên tắc **Least Privilege**, nghĩa là chỉ cấp cho EC2 đúng những quyền cần thiết để thực hiện chức năng của bài lab.

Trong phần 5.4, tôi sẽ lần lượt xác định các quyền EC2 cần sử dụng, tạo Custom IAM Policy, tạo IAM Role, gắn Role vào EC2 và thực hiện các bài kiểm thử để xác nhận quyền hoạt động đúng như thiết kế.

Sau khi hoàn thành phần này, luồng truy cập mong muốn là:

```text
AWS IAM
   │
   ▼
BackupLab-EC2-S3-Policy
   │
   ▼
BackupLab-EC2-S3-Role
   │
   ▼
Amazon EC2
   │
   ├── ListBucket
   ├── GetObject
   └── PutObject
   │
   ▼
Amazon S3
application-data/
```

Quyền `DeleteObject` không được cấp. Do đó, khi EC2 thử xóa object trên S3, kết quả mong đợi là:

```text
AccessDenied
```

Đây là bằng chứng thực tế cho việc áp dụng nguyên tắc **Least Privilege** trong hệ thống.

#### Nội dung

1. [Xác định quyền truy cập cần thiết](5.4.1.xác-định-quyền-truy-cập-cần-thiết/)
2. [Tạo và gắn IAM Role cho EC2](5.4.2.tạo-và-gắn-iam-role-cho-ec2/)
3. [Kiểm tra quyền truy cập Amazon S3](5.4.3.kiểm-tra-quyền-truy-cập-amazon-s3/)
4. [Kiểm tra kết quả](5.4.4.kiểm-tra-kết-quả/)