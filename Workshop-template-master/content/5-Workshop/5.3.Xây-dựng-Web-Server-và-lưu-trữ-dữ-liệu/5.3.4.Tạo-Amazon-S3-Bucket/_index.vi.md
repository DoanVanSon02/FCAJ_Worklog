---
title: "Tạo Amazon S3 Bucket"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.3.4. </b> "
---

## Tạo Amazon S3 Bucket

Ngoài dữ liệu trên EC2/EBS, workshop sử dụng **Amazon S3** để lưu trữ dữ liệu dạng object.

S3 đóng vai trò là nguồn dữ liệu thứ hai được bảo vệ bởi AWS Backup. Điều này giúp bài lab kiểm thử Backup & Recovery trên cả:

```text
Compute + Block Storage
EC2 / EBS

và

Object Storage
Amazon S3
```

Tôi truy cập:

```text
AWS Management Console
→ Amazon S3
→ Create bucket
```

---

### Cấu hình Bucket

Tại phần **Bucket type**, tôi chọn:

```text
General purpose
```

Tên Bucket sử dụng quy ước:

```text
aws-backup-lab-s3-<unique-suffix>
```

Do tên S3 Bucket phải duy nhất trên phạm vi toàn AWS, hậu tố thực tế có thể khác tùy theo tên còn khả dụng tại thời điểm tạo.

![Tạo S3 Bucket](/images/5/5.3.4/001.png?featherlight=false&width=70pc)

Region được sử dụng:

```text
US East (N. Virginia)
us-east-1
```

---

### Cấu hình Object Ownership

Tại phần **Object Ownership**, tôi giữ cấu hình:

```text
ACLs disabled
Bucket owner enforced
```

ACL không cần thiết trong kiến trúc của workshop vì quyền truy cập được quản lý thông qua IAM.

Việc tắt ACL giúp mô hình phân quyền đơn giản hơn và dễ kiểm soát hơn.

---

### Cấu hình Block Public Access

Tại phần **Block Public Access settings for this bucket**, tôi giữ:

```text
Block all public access
= Enabled
```

S3 Bucket trong workshop chỉ được EC2 và AWS Backup truy cập thông qua quyền IAM phù hợp.

Người dùng Internet không cần truy cập trực tiếp vào S3 Bucket.

Vì vậy, bucket không được cấu hình public.

Đây cũng là một biện pháp bảo mật quan trọng trong kiến trúc của workshop.

![](/images/5/5.3.4/002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.3.10: S3 Bucket được cấu hình chặn truy cập công khai</b>
</p>

---

### Bật Versioning

![Versioning](/images/5/5.3.4/003.png?featherlight=false&width=70pc)


Chọn:

```text
Enable
```

Kết quả:

```text
Bucket Versioning:
Enabled
```

Versioning giúp lưu nhiều phiên bản của cùng một object và là cấu hình quan trọng đối với việc bảo vệ dữ liệu S3 trong bài lab.

<p align="center">
  <b>Hình 5.3.11: Bật Versioning cho S3 Bucket</b>
</p>

---

### Tạo dữ liệu thử nghiệm trên S3

Trong Bucket, tôi tạo prefix:

```text
application-data/
```

Sau đó tạo file:

```text
s3-version.txt
```

với nội dung:

```text
AWS Backup Recovery Lab - S3 Version 1
```

File được upload vào:

```text
application-data/s3-version.txt
```

Cấu trúc sau khi upload:

```text
S3 Bucket
│
└── application-data/
    │
    └── s3-version.txt
```

---

### Vì sao đặt dữ liệu trong `application-data/`?

Thay vì cấp quyền trên toàn bộ object trong bucket, ở phần IAM tiếp theo tôi sẽ giới hạn quyền EC2 vào prefix:

```text
application-data/*
```

Việc giới hạn quyền theo prefix giúp minh họa rõ nguyên tắc **Least Privilege**.

EC2 chỉ được cấp quyền trên vùng dữ liệu cần thiết cho workshop thay vì toàn bộ S3 Bucket.

---

### Kiểm tra Object

Sau khi upload, tôi mở:

```text
application-data/
```

và xác nhận file:

```text
s3-version.txt
```

đã tồn tại.

Nội dung của file:

```text
AWS Backup Recovery Lab - S3 Version 1
```

![Kiểm tra object](/images/5/5.3.4/004.png?featherlight=false&width=70pc)


<p align="center">
  <b>Hình 5.3.12: Dữ liệu thử nghiệm được lưu trên Amazon S3</b>
</p>

---

### Kết quả

Sau khi hoàn thành phần này, S3 Bucket cần có trạng thái:

```text
Bucket = Created

Region = us-east-1

Block Public Access = Enabled

ACL = Disabled

Versioning = Enabled

application-data/
└── s3-version.txt
```

Amazon S3 hiện đã có dữ liệu thử nghiệm ban đầu.