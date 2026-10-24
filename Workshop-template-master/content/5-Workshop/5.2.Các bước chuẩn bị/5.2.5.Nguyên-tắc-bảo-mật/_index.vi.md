---
title: "Nguyên tắc bảo mật"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5.2.5. </b> "
---

### Nguyên tắc bảo mật áp dụng

Ngay từ bước chuẩn bị, tôi xác định một số nguyên tắc bảo mật cơ bản cho hệ thống nhằm hạn chế rủi ro trong quá trình triển khai và sử dụng các dịch vụ AWS.

Các nguyên tắc chính được áp dụng trong workshop gồm:

* Không hard-code Access Key trên EC2.
* Sử dụng IAM Role để cấp quyền giữa các dịch vụ.
* Áp dụng nguyên tắc Least Privilege.
* Hạn chế sử dụng quyền quản trị toàn bộ dịch vụ.
* Giữ Amazon S3 ở trạng thái private.
* Chỉ mở các cổng mạng cần thiết trên EC2.

---

#### Không hard-code Access Key

Trong workshop, EC2 truy cập Amazon S3 thông qua **IAM Role** thay vì lưu trực tiếp thông tin xác thực trên máy chủ.

Cụ thể, EC2 sử dụng:

```text
BackupLab-EC2-S3-Role
```

để nhận các quyền cần thiết khi truy cập Amazon S3.

Tôi không lưu trực tiếp các thông tin như:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

trên EC2.

Cách làm này giúp giảm nguy cơ làm lộ thông tin xác thực trong quá trình sử dụng máy chủ, đồng thời phù hợp hơn với cơ chế phân quyền của AWS.

---

#### Áp dụng nguyên tắc Least Privilege

**Least Privilege** là nguyên tắc chỉ cấp cho người dùng hoặc dịch vụ những quyền thực sự cần thiết để hoàn thành nhiệm vụ.

Trong workshop, EC2 không được cấp toàn bộ quyền quản trị Amazon S3.

Thay vào đó, tôi sử dụng custom IAM Policy:

```text
BackupLab-EC2-S3-Policy
```

Policy này chỉ cấp các quyền cần thiết đối với S3 Bucket được sử dụng trong bài lab.

Một số quyền được sử dụng gồm:

```text
s3:ListBucket
s3:GetObject
s3:GetObjectVersion
s3:PutObject
```

Các quyền trên cho phép EC2:

* Liệt kê các object trong bucket.
* Đọc object từ Amazon S3.
* Đọc các phiên bản object.
* Tải object từ EC2 lên Amazon S3.

Workshop không cấp các quyền rộng như:

```text
AdministratorAccess
```

hoặc:

```text
AmazonS3FullAccess
```

cho EC2 chỉ để hoàn thành bài lab.

Việc giới hạn quyền giúp giảm phạm vi ảnh hưởng nếu tài nguyên hoặc ứng dụng gặp sự cố bảo mật.

---

#### Sử dụng IAM Role giữa các dịch vụ

Các dịch vụ trong workshop sử dụng **IAM Role** để thực hiện các thao tác cần thiết.

Đối với EC2:

```text
BackupLab-EC2-S3-Role
```

được sử dụng để truy cập Amazon S3.

Đối với AWS Backup:

```text
AWSBackupDefaultServiceRole
```

được sử dụng để thực hiện các thao tác Backup và Restore đối với những tài nguyên được bảo vệ.

Việc tách riêng các IAM Role giúp mỗi dịch vụ chỉ sử dụng đúng quyền phục vụ cho chức năng của mình.

---

#### Hạn chế public resource

Amazon S3 Bucket trong workshop được giữ ở trạng thái **private**.

Cấu hình:

```text
Block Public Access = Enabled
```

được bật để hạn chế khả năng bucket hoặc object vô tình bị công khai ra Internet.

Trong kiến trúc này, S3 không được sử dụng như một public website.

EC2 truy cập dữ liệu trong S3 thông qua IAM Role, vì vậy không cần mở quyền truy cập public cho bucket.

---

#### Giới hạn cổng truy cập trên EC2

Security Group của EC2 chỉ mở những cổng cần thiết cho workshop.

Các cổng được sử dụng gồm:

```text
HTTP 80
SSH 22
```

Trong đó:

* **HTTP 80** được sử dụng để người dùng truy cập Web Server.
* **SSH 22** chỉ được sử dụng khi cần truy cập và quản trị EC2.

Đối với SSH, nguồn truy cập nên được giới hạn thay vì mở cho toàn bộ Internet nếu không cần thiết.

Việc chỉ mở những cổng cần thiết giúp giảm bề mặt tấn công của Web Server.

---

#### Tổng kết nguyên tắc bảo mật

Các nguyên tắc bảo mật được áp dụng trong workshop có thể tóm tắt như sau:

| Nguyên tắc | Cách áp dụng trong workshop |
| --- | --- |
| **Không hard-code Access Key** | EC2 sử dụng IAM Role để truy cập S3 |
| **Least Privilege** | Chỉ cấp các quyền S3 cần thiết |
| **Tách quyền giữa các dịch vụ** | EC2 và AWS Backup sử dụng IAM Role riêng |
| **Hạn chế public resource** | S3 bật Block Public Access |
| **Giới hạn cổng mạng** | EC2 chỉ mở HTTP 80 và SSH 22 khi cần thiết |

Việc áp dụng các nguyên tắc trên giúp hệ thống đáp ứng các yêu cầu bảo mật cơ bản, đồng thời hạn chế việc cấp quyền quá rộng hoặc làm lộ thông tin xác thực trong quá trình triển khai workshop.