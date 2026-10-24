---
title: "Quy ước đặt tên và Tag tài nguyên"
date: 2026-01-01
weight: 6
chapter: false
pre: " <b> 5.2.6. </b> "
---

### Quy ước đặt tên và Tag tài nguyên

Để các tài nguyên trong workshop dễ nhận biết, quản lý và dọn dẹp sau khi hoàn thành, tôi sử dụng một quy ước thống nhất về **tên tài nguyên** và **Tag**.

Việc đặt tên và gắn Tag ngay từ đầu giúp hạn chế nhầm lẫn giữa các tài nguyên, đặc biệt khi trong cùng AWS Account có nhiều EC2 Instance, EBS Volume, IAM Role, Backup Plan hoặc các tài nguyên khác.

---

#### Quy ước đặt tên tài nguyên

Các tài nguyên chính trong workshop được đặt tên như sau:

| Tài nguyên | Tên |
| --- | --- |
| EC2 Instance | `backup-lab-web-server` |
| Security Group | `backup-lab-web-sg` |
| EBS Data Volume | `backup-lab-data-volume` |
| IAM Policy | `BackupLab-EC2-S3-Policy` |
| IAM Role | `BackupLab-EC2-S3-Role` |
| Backup Vault | `BackupLab-Vault` |
| Backup Plan | `BackupLab-Daily-Plan` |
| Backup Rule | `Daily-Backup-Rule` |
| Resource Assignment | `BackupLab-Resources` |
| CloudWatch Alarm | `BackupLab-BackupCompleted-Test-Alarm` |

Các tên trên được sử dụng thống nhất trong toàn bộ workshop để dễ dàng xác định chức năng của từng tài nguyên.

Ví dụ:

```text
backup-lab-web-server
```

được sử dụng cho EC2 Instance chạy Web Server.

Trong khi:

```text
BackupLab-Vault
```

được sử dụng làm Backup Vault để quản lý các Recovery Point được tạo bởi AWS Backup.

---

#### Quy ước tên Amazon S3 Bucket

Đối với Amazon S3, tên bucket phải duy nhất nên tôi sử dụng cấu trúc:

```text
aws-backup-lab-s3-<unique-suffix>
```

Trong đó:

```text
<unique-suffix>
```

là phần giá trị bổ sung để giúp tên bucket không bị trùng với các bucket đã tồn tại.

Ví dụ:

```text
aws-backup-lab-s3-abc123
```

Tên bucket thực tế có thể thay đổi tùy vào tên còn khả dụng tại thời điểm tạo tài nguyên.

---

#### Quy ước Tag

Bên cạnh việc đặt tên tài nguyên, tôi sử dụng **Tag** để xác định các tài nguyên thuộc cùng workshop.

Các tài nguyên có hỗ trợ Tag sẽ sử dụng:

```text
Project = AWS-Backup-Recovery-Lab
Environment = Lab
```

Trong đó:

* **Project** được sử dụng để xác định tài nguyên thuộc dự án Backup & Recovery.
* **Environment** được sử dụng để xác định đây là môi trường Lab.

Ví dụ, EC2 Instance có thể được gắn các Tag như sau:

```text
Name        = backup-lab-web-server
Project     = AWS-Backup-Recovery-Lab
Environment = Lab
```

Tag `Name` giúp nhận biết trực tiếp tên của EC2 Instance trên AWS Management Console.

Tag `Project` và `Environment` giúp phân biệt các tài nguyên thuộc workshop với những tài nguyên khác trong cùng AWS Account.

---

#### Lợi ích của việc sử dụng quy ước đặt tên và Tag

Việc sử dụng thống nhất tên tài nguyên và Tag giúp quá trình thực hiện workshop dễ quản lý hơn.

Khi cần kiểm tra một tài nguyên, tôi có thể xác định nhanh tài nguyên đó thuộc thành phần nào của hệ thống.

Ví dụ:

```text
backup-lab-web-server
```

cho biết đây là EC2 Instance của Web Server.

```text
backup-lab-data-volume
```

cho biết đây là EBS Data Volume được sử dụng để lưu dữ liệu.

```text
BackupLab-Vault
```

cho biết đây là Backup Vault được sử dụng trong hệ thống Backup & Recovery.

Ngoài ra, Tag còn hỗ trợ việc xác định các tài nguyên thuộc cùng project khi cần kiểm tra hoặc dọn dẹp sau khi hoàn thành workshop.

Nhờ đó, tôi có thể dễ dàng xác định các tài nguyên có:

```text
Project = AWS-Backup-Recovery-Lab
```

và tiến hành kiểm tra trước khi thực hiện bước **Dọn dẹp tài nguyên** ở cuối workshop.