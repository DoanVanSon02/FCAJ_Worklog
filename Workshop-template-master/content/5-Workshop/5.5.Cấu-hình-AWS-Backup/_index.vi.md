---
title: "Cấu hình AWS Backup"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Tổng quan

Sau khi hoàn thành Web Server, EBS, Amazon S3 và IAM, hệ thống đã có đầy đủ các tài nguyên cần được bảo vệ.

Trong phần này, tôi sử dụng **AWS Backup** để xây dựng cơ chế sao lưu tập trung cho các tài nguyên chính của workshop gồm:

```text
Amazon EC2
├── Root EBS
└── Data EBS

Amazon S3
└── application-data/
```

Thay vì tạo snapshot hoặc các bản backup riêng lẻ theo cách thủ công, AWS Backup cho phép quản lý tập trung toàn bộ quá trình sao lưu và khôi phục.

Luồng hoạt động chính được sử dụng trong workshop:

```text
Backup Vault
     ↓
Backup Plan
     ↓
Resource Assignment
     ↓
Backup Job
     ↓
Recovery Point
     ↓
Restore
```

Trong đó:

* **Backup Vault** được sử dụng để quản lý các Recovery Point.
* **Backup Plan** xác định lịch Backup và thời gian lưu giữ dữ liệu.
* **Resource Assignment** xác định các tài nguyên cần được bảo vệ.
* **Backup Job** đại diện cho quá trình AWS Backup thực hiện sao lưu.
* **Recovery Point** đại diện cho trạng thái đã được sao lưu của tài nguyên tại một thời điểm cụ thể.
* **Restore** sử dụng Recovery Point để khôi phục tài nguyên sau khi xảy ra sự cố.

Trong workshop này, tôi tạo một Backup Vault riêng:

```text
BackupLab-Vault
```

để quản lý các Recovery Point của bài lab.

Sau đó, tôi xây dựng Backup Plan:

```text
BackupLab-Daily-Plan
```

với Backup Rule:

```text
Daily-Backup-Rule
```

Backup Plan được cấu hình để thực hiện sao lưu định kỳ và giữ Recovery Point trong:

```text
7 ngày
```

Các tài nguyên được đưa vào Backup Plan gồm:

```text
Amazon EC2
→ backup-lab-web-server

Amazon S3
→ aws-backup-lab-s3-<unique-suffix>
```

AWS Backup sử dụng Service Role:

```text
AWSBackupDefaultServiceRole
```

để thực hiện các thao tác Backup và Restore đối với các tài nguyên được bảo vệ.

Role này khác với:

```text
BackupLab-EC2-S3-Role
```

được sử dụng ở phần IAM trước đó để cho phép EC2 truy cập Amazon S3.

Sau khi Backup Plan được cấu hình, tôi sử dụng **On-demand Backup** để tạo Recovery Point ngay lập tức thay vì chờ đến thời điểm Backup định kỳ.

Hai cơ chế được sử dụng song song:

```text
Backup Plan
→ Backup tự động theo lịch

On-demand Backup
→ Backup ngay khi cần kiểm thử
```

Trước khi thực hiện Backup, dữ liệu trên EBS và S3 được giữ ở trạng thái ban đầu:

```text
Version 1
```

Sau khi Backup Job hoàn thành thành công, Recovery Point của EC2 và Amazon S3 sẽ được lưu trong:

```text
BackupLab-Vault
```

Quy trình kiểm thử trong workshop được thực hiện theo thứ tự:

```text
Version 1
    ↓
BACKUP
    ↓
Recovery Point Version 1
    ↓
INCIDENT
    ↓
Version 2 / Deleted / Corrupted
    ↓
RESTORE
    ↓
Version 1
```

Việc giữ đúng trình tự trên giúp bảo đảm Recovery Point được tạo trước khi sự cố xảy ra và có thể được sử dụng để chứng minh quá trình Backup & Recovery hoạt động chính xác.

#### Nội dung

1. [Chuẩn bị AWS Backup và tạo Backup Vault](5.5.1.chuẩn-bị-aws-backup-và-tạo-backup-vault/)
2. [Tạo Backup Plan và gán tài nguyên](5.5.2.tạo-backup-plan-và-gán-tài-nguyên/)
3. [Tạo On-demand Backup và kiểm tra Recovery Point](5.5.3.tạo-on-demand-backup-và-kiểm-tra-recovery-point/)