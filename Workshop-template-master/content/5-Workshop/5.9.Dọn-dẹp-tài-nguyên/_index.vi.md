---
title: "Dọn dẹp tài nguyên"
date: 2026-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

Sau khi hoàn thành quá trình Backup, Monitoring, Restore và Validation, tôi tiến hành xóa các tài nguyên không còn sử dụng để tránh tiếp tục phát sinh chi phí.

Việc Cleanup cần được thực hiện theo đúng thứ tự vì một số tài nguyên phụ thuộc lẫn nhau.

---

### Xóa Resource Assignment và Backup Plan

Trước tiên, tôi truy cập:

```text
AWS Backup
→ Backup plans
→ BackupLab-Daily-Plan
```

Sau đó xóa Resource Assignment:

```text
BackupLab-Resources
```

Tiếp theo, tôi xóa Backup Plan:

```text
BackupLab-Daily-Plan
```

Việc xóa Backup Plan giúp dừng các lần Backup tự động trong tương lai nhưng không làm mất các Recovery Point đã được tạo trước đó.

Luồng Cleanup:

```text
BackupLab-Resources
        ↓
Delete Resource Assignment
        ↓
BackupLab-Daily-Plan
        ↓
Delete Backup Plan
```
![Xóa tài nguyên](/images/5/5.9/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>

![Xóa tài nguyên](/images/5/5.9/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>
---

### Xóa Recovery Point và Backup Vault

Tiếp theo, tôi truy cập:

```text
AWS Backup
→ Backup vaults
→ BackupLab-Vault
```

Tôi xóa toàn bộ Recovery Point của:

```text
Amazon EC2

Amazon S3
```

Sau khi xác nhận:

```text
Recovery Points = 0
```
![Xóa tài nguyên](/images/5/5.9/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>

tôi tiếp tục xóa:

```text
BackupLab-Vault
```
![Xóa tài nguyên](/images/5/5.9/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>

Thứ tự cần thực hiện:

```text
Recovery Points
        ↓
Delete
        ↓
Recovery Points = 0
        ↓
Delete BackupLab-Vault
```

Việc xóa Recovery Point trước giúp tránh trường hợp Backup Vault không thể bị xóa do vẫn còn Recovery Point bên trong.

---

### Terminate Amazon EC2

Sau khi hoàn thành Cleanup của AWS Backup, tôi chuyển sang Amazon EC2.

Tôi truy cập:

```text
Amazon EC2
→ Instances
```

và terminate cả hai Instance:

```text
backup-lab-web-server-restored

backup-lab-web-server
```

Trong đó:

```text
backup-lab-web-server
→ EC2 gốc

backup-lab-web-server-restored
→ EC2 được tạo từ quá trình Restore
```

Tôi sử dụng:

```text
Terminate
```

thay vì chỉ:

```text
Stop
```

vì workshop đã hoàn thành và các Instance không còn cần thiết.

Nếu chỉ Stop EC2, một số tài nguyên liên quan như EBS vẫn có thể tiếp tục phát sinh chi phí.

![Xóa tài nguyên](/images/5/5.9/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>
---

### Xóa EBS Volume còn lại

Sau khi các EC2 Instance bị terminate, tôi truy cập:

```text
EC2
→ Elastic Block Store
→ Volumes
```

và kiểm tra các EBS Volume còn ở trạng thái:

```text
Available
```

Trạng thái `Available` cho biết Volume hiện không được gắn với EC2 Instance nào.

Nếu xác nhận Volume thuộc workshop, ví dụ:

```text
backup-lab-data-volume
```

tôi thực hiện xóa Volume đó.

Mục tiêu là bảo đảm không còn EBS Volume của bài lab tiếp tục tồn tại và phát sinh chi phí lưu trữ.

![Xóa tài nguyên](/images/5/5.9/0006.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>
---

### Xóa Amazon S3 Bucket

Tiếp theo, tôi thực hiện Cleanup Amazon S3 Bucket.

Do Bucket đã bật:

```text
Versioning = Enabled
```

nên việc chỉ xóa các object hiện tại chưa đủ để làm Bucket hoàn toàn rỗng.

Tôi truy cập S3 Bucket và bật:

```text
Show versions
```

Sau đó xóa toàn bộ:

```text
Current versions

Previous versions

Delete markers
```
![Xóa tài nguyên](/images/5/5.9/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>

Có thể hình dung:

```text
S3 Bucket
   ↓
Show versions
   ↓
Current versions
   +
Previous versions
   +
Delete markers
   ↓
Delete
   ↓
Bucket Empty
```

Sau khi xác nhận Bucket hoàn toàn rỗng, tôi mới thực hiện xóa S3 Bucket.

![Xóa tài nguyên](/images/5/5.9/0008.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>
---

### Xóa CloudWatch Alarm

Tiếp theo, tôi truy cập:

```text
Amazon CloudWatch
→ Alarms
```

và xóa các Alarm được tạo riêng cho workshop:

```text
BackupLab-BackupCompleted-Test-Alarm

BackupLab-S3-BackupCompleted-Test-Alarm
```

Sau khi xóa, CloudWatch không còn các Alarm dùng để giám sát Backup Job của bài lab.

![Xóa tài nguyên](/images/5/5.9/0009.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>
---

### Xóa Custom IAM Role và Policy

Sau đó, tôi Cleanup các IAM Resource do mình tự tạo trong workshop.

IAM Role cần xóa:

```text
BackupLab-EC2-S3-Role
```
![Xóa tài nguyên](/images/5/5.9/0010.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>

Custom IAM Policy cần xóa:

```text
BackupLab-EC2-S3-Policy
```
![Xóa tài nguyên](/images/5/5.9/0011.png?featherlight=false&width=70pc)

<p align="center">
  <b>Xóa tài nguyên</b>
</p>

Thứ tự có thể thực hiện:

```text
BackupLab-EC2-S3-Role
        ↓
Delete Role
        ↓
BackupLab-EC2-S3-Policy
        ↓
Delete Policy
```

Tôi chỉ xóa các IAM Resource được tạo riêng cho workshop.

---

### Không xóa AWS Service Role nếu không chắc chắn

Tôi không xóa các AWS Service Role hoặc Service-Linked Role nếu không chắc chắn chúng chỉ được sử dụng cho workshop.

Ví dụ:

```text
AWSBackupDefaultServiceRole

AWSServiceRoleForBackup
```

Các Role này không phát sinh chi phí chỉ vì chúng tồn tại và có thể tiếp tục được AWS sử dụng cho các bài lab hoặc workload khác.

Do đó, trong quá trình Cleanup, tôi tập trung vào các Custom IAM Resource mà mình đã chủ động tạo.

---

### Kiểm tra trạng thái sau Cleanup

Sau khi hoàn thành, tôi kiểm tra lại các dịch vụ để bảo đảm không còn tài nguyên của bài lab đang hoạt động.

| Dịch vụ | Trạng thái mong đợi |
| --- | --- |
| EC2 | Không còn Instance của Lab |
| EBS | Không còn Volume của Lab |
| S3 | Bucket của Lab đã được xóa |
| AWS Backup | Không còn Backup Plan |
| AWS Backup | Không còn Recovery Point |
| AWS Backup | Backup Vault đã được xóa |
| CloudWatch | Alarm của Lab đã được xóa |
| IAM | Custom Role và Policy đã được xóa |

---

### Thứ tự Cleanup tổng quát

Toàn bộ quá trình Cleanup được thực hiện theo thứ tự:

```text
Resource Assignment
        ↓
Backup Plan
        ↓
Recovery Points
        ↓
Backup Vault
        ↓
EC2 Instances
        ↓
EBS Volumes
        ↓
S3 Versions / Delete Markers
        ↓
S3 Bucket
        ↓
CloudWatch Alarms
        ↓
Custom IAM Role / Policy

Thứ tự này giúp hạn chế lỗi do phụ thuộc giữa các tài nguyên và giúp việc Cleanup dễ kiểm soát hơn.
```

---
