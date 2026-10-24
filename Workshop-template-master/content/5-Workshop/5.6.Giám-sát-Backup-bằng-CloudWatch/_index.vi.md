---
title: "Giám sát AWS Backup bằng Amazon CloudWatch"
date: 2026-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

### Tổng quan

Sau khi các Backup Job của Amazon EC2 và Amazon S3 được thực hiện thành công, tôi tiếp tục bổ sung cơ chế giám sát cho hệ thống bằng **Amazon CloudWatch**.

AWS Backup chịu trách nhiệm thực hiện quá trình Backup và Restore, trong khi Amazon CloudWatch được sử dụng để theo dõi các metric liên quan đến trạng thái của Backup Job và Recovery Point.

Hệ thống lúc này không chỉ có khả năng:

```text
Backup
   ↓
Recovery Point
   ↓
Restore
```

mà còn có thêm cơ chế giám sát:

```text
AWS Backup
     ↓
Backup Metrics
     ↓
Amazon CloudWatch
     ↓
CloudWatch Alarm
```

Việc bổ sung CloudWatch giúp theo dõi hoạt động Backup thay vì chỉ thực hiện sao lưu mà không có cơ chế quan sát trạng thái.

Trong môi trường thực tế, nếu một Backup Job không hoàn thành như mong đợi nhưng không có hệ thống giám sát, người quản trị có thể không phát hiện được sự cố kịp thời.

AWS Backup cung cấp các metric thông qua namespace:

```text
AWS/Backup
```

Các metric này có thể được phân biệt theo các dimension như:

```text
BackupVaultName
ResourceType
```

Trong workshop, tôi sử dụng:

```text
BackupVaultName = BackupLab-Vault
```

và theo dõi riêng hai loại tài nguyên:

```text
ResourceType = EC2

ResourceType = S3
```

Một số metric quan trọng được sử dụng gồm:

```text
NumberOfBackupJobsCreated
NumberOfBackupJobsRunning
NumberOfBackupJobsCompleted
NumberOfBackupJobsFailed
NumberOfRecoveryPointsCompleted
```

Trong bài lab, các Backup Job của EC2 và S3 đã hoàn thành thành công nên tôi sử dụng:

```text
NumberOfBackupJobsCompleted
```

để kiểm thử luồng:

```text
AWS Backup
     ↓
Backup Job
     ↓
Metric
     ↓
Amazon CloudWatch
     ↓
CloudWatch Alarm
```

CloudWatch Alarm được tạo riêng cho:

```text
Amazon EC2

Amazon S3
```

với điều kiện kiểm thử:

```text
NumberOfBackupJobsCompleted >= 1
```

Mục tiêu của Alarm trong workshop là xác nhận rằng CloudWatch có thể nhận metric từ AWS Backup, đánh giá điều kiện đã cấu hình và thay đổi trạng thái Alarm.

Luồng giám sát tổng quát của hệ thống sau khi cấu hình là:

```text
                   AWS Backup
                       │
                       │ AWS/Backup Metrics
                       ▼
                 Amazon CloudWatch
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
      EC2 Backup Metric    S3 Backup Metric
             │                   │
             ▼                   ▼
         EC2 Alarm           S3 Alarm
```

Trong môi trường thực tế, metric:

```text
NumberOfBackupJobsFailed
```

sẽ phù hợp hơn để cảnh báo khi Backup Job thất bại.

Tuy nhiên, trong phạm vi workshop, tôi không cố tình làm hỏng Backup Job đang hoạt động ổn định mà sử dụng metric `NumberOfBackupJobsCompleted` để chứng minh cơ chế **Metric → Alarm** hoạt động chính xác.

Sau khi hoàn thành phần này, hệ thống sẽ có đầy đủ các thành phần:

```text
Backup
   ↓
Recovery Point

Monitoring
   ↓
CloudWatch Alarm

Restore
   ↓
Được kiểm thử ở phần tiếp theo
```

#### Nội dung

1. [Theo dõi AWS Backup Metrics](5.6.1.theo-dõi-aws-backup-metrics/)
2. [Tạo CloudWatch Alarm cho EC2 và S3](5.6.2.tạo-cloudwatch-alarm-cho-ec2-và-s3/)