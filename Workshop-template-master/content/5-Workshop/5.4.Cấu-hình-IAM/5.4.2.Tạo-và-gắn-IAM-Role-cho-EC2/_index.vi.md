---
title: "Tạo và gắn IAM Role cho EC2"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

## Tạo và gắn IAM Role cho EC2

Phần này bao gồm ba bước liên tiếp:

```text
Tạo Custom IAM Policy
          ↓
Tạo IAM Role cho EC2
          ↓
Gắn IAM Role vào EC2
```

---

### Tạo Custom IAM Policy

Tôi truy cập:

```text
AWS Management Console
→ IAM
→ Policies
→ Create policy
→ Select a service → S3
```
![Tạo policy](/images/5/5.4.2/0001.png?featherlight=false&width=70pc)

Sau đó chọn:

```text
JSON
```

để tự xây dựng Policy.

Policy được cấu hình như sau:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowBucketInformation",
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation"
      ],
      "Resource": "arn:aws:s3:::aws-backup-lab-s3-162918102026"
    },
    {
      "Sid": "AllowListApplicationData",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::aws-backup-lab-s3-162918102026",
      "Condition": {
        "StringLike": {
          "s3:prefix": [
            "application-data",
            "application-data/*"
          ]
        }
      }
    },
    {
      "Sid": "AllowApplicationDataObjects",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::aws-backup-lab-s3-162918102026/application-data/*"
    }
  ]
}
```
![Tạo policy](/images/5/5.4.2/0013.png?featherlight=false&width=70pc)

### Giải thích các quyền trong Policy

Policy được chia thành ba nhóm quyền khác nhau.

#### Quyền lấy thông tin Bucket

```text
s3:GetBucketLocation
```

cho phép AWS CLI xác định Region của Bucket.

Quyền này được áp dụng ở cấp Bucket:

```text
arn:aws:s3:::arn:aws:s3:::aws-backup-lab-s3-162918102026
```

#### Quyền liệt kê object

```text
s3:ListBucket
```

cho phép EC2 liệt kê nội dung trong Bucket.

Tuy nhiên, quyền này không được cấp không giới hạn.

Tôi sử dụng điều kiện:

```json
"Condition": {
  "StringLike": {
    "s3:prefix": [
      "application-data",
      "application-data/*"
    ]
  }
}
```

để giới hạn phạm vi liên quan tới:

```text
application-data/
```

Điều này giúp tránh việc cấp quyền liệt kê dữ liệu không cần thiết bên ngoài khu vực ứng dụng sử dụng.

#### Quyền thao tác với object

Hai quyền:

```text
s3:GetObject
s3:PutObject
```

chỉ được áp dụng cho:

```text
arn:aws:s3:::arn:aws:s3:::aws-backup-lab-s3-162918102026/application-data/*
```

Do đó EC2 không tự động có quyền đọc hoặc ghi mọi object trong Bucket.

---

### Đặt tên IAM Policy

Tôi sử dụng tên:

```text
BackupLab-EC2-S3-Policy
```

Description:

```text
Allow the Backup Lab EC2 instance to access
application-data in the lab S3 bucket.
```

Sau khi kiểm tra Policy, tôi chọn:

```text
Create policy
```

<p align="center">
  <b>Hình 5.4.1: Tạo Custom IAM Policy cho phép EC2 truy cập Amazon S3</b>
</p>

![Tạo policy](/images/5/5.4.2/0003.png?featherlight=false&width=70pc)

Kiểm tra lại sau khi tạo:

![Tạo policy](/images/5/5.4.2/0004.png?featherlight=false&width=70pc)

---

### Tạo IAM Role cho EC2

Sau khi Custom Policy được tạo, tôi tiếp tục tạo IAM Role để EC2 có thể sử dụng Policy này.

Tôi truy cập:

```text
IAM
→ Roles
→ Create role
```

Tại:

```text
Trusted entity type
```

chọn:

```text
AWS service
```

Service:

```text
EC2
```

Sau đó chọn:

```text
Next
```
![Tạo IAM Role cho EC2](/images/5/5.4.2/0005.png?featherlight=false&width=70pc)

---

### Tại sao Trusted Entity là EC2?

IAM Role cần xác định dịch vụ nào được phép sử dụng Role.

Trong trường hợp này:

```text
Amazon EC2
     │
     │ Assume Role
     ▼
BackupLab-EC2-S3-Role
```

Do đó Trusted Entity phải là Amazon EC2.

Trust Policy tương ứng có dạng:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Tôi không cần nhập thủ công Trust Policy này khi tạo Role bằng:

```text
AWS service → EC2
```

vì AWS tự động tạo Trust Policy tương ứng.

---

### Gắn Policy vào Role

Tại bước **Add permissions**, tôi tìm:

```text
BackupLab-EC2-S3-Policy
```

và chọn Policy này.

Tôi không sử dụng:

```text
AmazonS3FullAccess
```

hoặc:

```text
AdministratorAccess
```

vì hai Policy trên cung cấp phạm vi quyền lớn hơn yêu cầu của bài lab.

![Gắn Policy vào Role](/images/5/5.4.2/0006.png?featherlight=false&width=70pc)

Tên Role được sử dụng:

```text
BackupLab-EC2-S3-Role
```

Description:

```text
IAM role for the Backup Lab EC2 instance
to access application data in Amazon S3.
```

Sau đó chọn:

```text
Create role
```
![Gắn policy vào Role](/images/5/5.4.2/0007.png?featherlight=false&width=70pc)

![Gắn policy vào Role](/images/5/5.4.2/0008.png?featherlight=false&width=70pc)

Kiểm tra sau khi đã tạo xong:

![Gắn policy vào Role](/images/5/5.4.2/0009.png?featherlight=false&width=70pc)
---

### Gắn IAM Role vào EC2

IAM Role đã được tạo nhưng EC2 chưa tự động sử dụng Role này.

Tôi truy cập:

```text
EC2
→ Instances
```

chọn:

```text
backup-lab-web-server
```

sau đó:

```text
Actions
→ Security
→ Modify IAM role
```
![Gắn policy vào Role](/images/5/5.4.2/0010.png?featherlight=false&width=70pc)

Tại mục:

```text
IAM role
```

chọn:

```text
BackupLab-EC2-S3-Role
```

sau đó chọn:

```text
Update IAM role
```
![Gắn policy vào Role](/images/5/5.4.2/0011.png?featherlight=false&width=70pc)

Luồng phân quyền sau bước này trở thành:

```text
BackupLab-EC2-S3-Policy
          ↓
BackupLab-EC2-S3-Role
          ↓
backup-lab-web-server
          ↓
      Amazon S3
```

Sau khi hoàn thành bước này, EC2 đã được gắn IAM Role và sẵn sàng để kiểm tra quyền truy cập Amazon S3 ở phần tiếp theo.