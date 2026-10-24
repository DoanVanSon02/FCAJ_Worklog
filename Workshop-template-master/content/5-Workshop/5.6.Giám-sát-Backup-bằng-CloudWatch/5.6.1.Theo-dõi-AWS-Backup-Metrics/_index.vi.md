---
title: "Theo dõi AWS Backup Metrics"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.6.1. </b> "
---

## Theo dõi AWS Backup Metrics

Sau khi các Backup Job của Amazon EC2 và Amazon S3 hoàn thành, tôi sử dụng **Amazon CloudWatch** để theo dõi các metric được AWS Backup gửi sang.

Tôi truy cập:

```text
AWS Management Console
→ CloudWatch
→ Metrics
→ All metrics / Classic metrics
```

Sau đó chọn namespace:

```text
AWS/Backup
```

Tại đây, CloudWatch hiển thị các metric do AWS Backup gửi sang.

![Các AWS Backup Metrics của EC2 và S3 trong Amazon CloudWatch](/images/5/5.6.1/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Các AWS Backup Metrics của EC2 và S3 trong Amazon CloudWatch</b>
</p>
---

### Lựa chọn Backup Vault và Resource Type

Các Backup Job ở phần trước đều sử dụng Backup Vault:

```text
BackupLab-Vault
```

Do đó, tôi tập trung vào những metric có dimension:

```text
BackupVaultName = BackupLab-Vault
```

Sau đó phân biệt các tài nguyên bằng:

```text
ResourceType = EC2

ResourceType = S3
```

Cách phân loại này giúp tôi theo dõi riêng trạng thái Backup của từng loại tài nguyên trong cùng một Backup Vault.

Luồng theo dõi có thể mô tả như sau:

```text
BackupLab-Vault
       │
       ├── ResourceType = EC2
       │
       └── ResourceType = S3
```

---

### Các metric được sử dụng

Một số metric quan trọng đối với workshop gồm:

| Metric | Ý nghĩa |
| --- | --- |
| `NumberOfBackupJobsCreated` | Số Backup Job được AWS Backup tạo |
| `NumberOfBackupJobsRunning` | Số Backup Job đang thực hiện |
| `NumberOfBackupJobsCompleted` | Số Backup Job đã hoàn thành |
| `NumberOfBackupJobsFailed` | Số Backup Job có trạng thái Failed |
| `NumberOfRecoveryPointsCompleted` | Số Recovery Point được tạo thành công |

Sau khi On-demand Backup của EC2 và S3 hoàn thành, tôi có thể theo dõi các metric tương ứng.

Đối với Amazon EC2:

```text
EC2
├── NumberOfBackupJobsCreated
├── NumberOfBackupJobsRunning
├── NumberOfBackupJobsCompleted
└── NumberOfRecoveryPointsCompleted
```

Đối với Amazon S3:

```text
S3
├── NumberOfBackupJobsCreated
├── NumberOfBackupJobsRunning
├── NumberOfBackupJobsCompleted
└── NumberOfRecoveryPointsCompleted
```

Việc các metric xuất hiện trong CloudWatch cho thấy hoạt động của AWS Backup đã được hệ thống Monitoring ghi nhận.

![Các AWS Backup Metrics của EC2 và S3 trong Amazon CloudWatch](/images/5/5.6.1/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Các AWS Backup Metrics của EC2 và S3 trong Amazon CloudWatch</b>
</p>

![Các AWS Backup Metrics của EC2 và S3 trong Amazon CloudWatch](/images/5/5.6.1/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Các AWS Backup Metrics của EC2 và S3 trong Amazon CloudWatch</b>
</p>

Ảnh nên thể hiện:

```text
Namespace:
AWS/Backup

BackupVaultName:
BackupLab-Vault

ResourceType:
EC2 / S3
```

và các metric như:

```text
NumberOfBackupJobsCompleted
NumberOfRecoveryPointsCompleted
```

---

### Ý nghĩa của `NumberOfBackupJobsCompleted`

Metric:

```text
NumberOfBackupJobsCompleted
```

cho biết số lượng Backup Job mà AWS Backup đã hoàn thành thành công.

Trong phần 5.5, cả EC2 và S3 đều có:

```text
Backup Job
Status = COMPLETED
```

Do đó CloudWatch có thể ghi nhận metric tương ứng.

Luồng kiểm tra:

```text
On-demand Backup
       ↓
Backup Job
       ↓
COMPLETED
       ↓
NumberOfBackupJobsCompleted
       ↓
Amazon CloudWatch
```

Metric này được sử dụng trong workshop để xác nhận rằng **CloudWatch thực sự nhận được dữ liệu từ AWS Backup**.

---

### Metric `NumberOfBackupJobsFailed`

Trong môi trường thực tế, một trong những metric quan trọng hơn là:

```text
NumberOfBackupJobsFailed
```

Metric này cho biết số lượng Backup Job có trạng thái:

```text
FAILED
```

Luồng cảnh báo có thể được triển khai như sau:

```text
Backup Job
     ↓
FAILED
     ↓
NumberOfBackupJobsFailed >= 1
     ↓
CloudWatch Alarm
     ↓
Cảnh báo người quản trị
```

Metric này có ý nghĩa quan trọng trong hệ thống thực tế vì nó giúp phát hiện khi một Backup Job không hoàn thành thành công.

Trong workshop hiện tại, các Backup Job đều đã hoàn thành với trạng thái:

```text
COMPLETED
```

do đó chưa phát sinh:

```text
NumberOfBackupJobsFailed > 0
```

Thay vào đó, trong phần tiếp theo tôi sử dụng:

```text
NumberOfBackupJobsCompleted
```

để kiểm thử cơ chế:

```text
AWS Backup Metric
       ↓
Amazon CloudWatch
       ↓
CloudWatch Alarm
```

Cách tiếp cận này giúp kiểm tra được chức năng Monitoring mà không làm thay đổi các cấu hình Backup đang hoạt động ổn định.