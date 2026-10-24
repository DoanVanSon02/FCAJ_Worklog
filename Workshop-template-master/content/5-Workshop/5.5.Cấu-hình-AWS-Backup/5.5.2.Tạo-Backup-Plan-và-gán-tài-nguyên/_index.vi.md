---
title: "Tạo Backup Plan và gán tài nguyên"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

## Tạo Backup Plan và gán tài nguyên

Sau khi Backup Vault được tạo, tôi tiếp tục xây dựng **Backup Plan** để xác định lịch sao lưu, thời gian lưu Recovery Point và các tài nguyên cần được bảo vệ.

Backup Vault trả lời câu hỏi:

```text
Bản backup được quản lý ở đâu?
```

Trong khi Backup Plan trả lời:

```text
Backup được thực hiện theo lịch nào
và được giữ trong bao lâu?
```

---

### Tạo Backup Plan

Tôi truy cập:

```text
AWS Backup
→ Backup plans
→ Create backup plan
```
![Tạo Backup Plan](/images/5/5.5.2/0001.png?featherlight=false&width=70pc)
<p align="center">
  <b>Tạo Backup Plan</b>
</p>

Sau đó chọn:

```text
Build a new plan
```

Tên Backup Plan:

```text
BackupLab-Daily-Plan
```

Tên Backup Rule:

```text
Daily-Backup-Rule
```

Tôi cấu hình Backup Rule như sau:

| Cấu hình | Giá trị sử dụng |
| --- | --- |
| Backup Vault | `BackupLab-Vault` |
| Backup frequency | Daily |
| Start time | 00:30 |
| Time zone | Asia/Saigon (UTC+07) |
| Start within | 8 hours |
| Complete within | 7 days |
| Continuous Backup / PITR | Disabled |
| Cold storage | Disabled |
| Total retention | 7 days |
| Backup indexes | Disabled |
| Scan mode | None |
| Destination copy | Không cấu hình |

**Start within** xác định khoảng thời gian Backup Job được phép bắt đầu.

**Complete within** xác định khoảng thời gian tối đa để Backup Job hoàn thành sau khi đã bắt đầu.

![Tạo Backup Plan](/images/5/5.5.2/0002.png?featherlight=false&width=70pc)
<p align="center">
  <b>Tạo Backup Plan</b>
</p>

![Tạo Backup Plan](/images/5/5.5.2/0003.png?featherlight=false&width=70pc)
<p align="center">
  <b>Tạo Backup Plan</b>
</p>

![Tạo Backup Plan](/images/5/5.5.2/0004.png?featherlight=false&width=70pc)
<p align="center">
  <b>Tạo Backup Plan</b>
</p>

![Tạo Backup Plan](/images/5/5.5.2/0005.png?featherlight=false&width=70pc)
<p align="center">
  <b>Tạo Backup Plan</b>
</p>
---

### Tại sao sử dụng Daily Backup?

Hệ thống của workshop chỉ có một Web Server và lượng dữ liệu nhỏ.

Tôi sử dụng:

```text
Frequency = Daily
```

thay vì Hourly vì mục tiêu của workshop là minh họa một chính sách Backup định kỳ cơ bản.

Trong hệ thống thực tế, tần suất Backup cần được lựa chọn dựa trên:

```text
RPO
+
Mức độ thay đổi dữ liệu
+
Chi phí lưu trữ
```

Tuy nhiên, với phạm vi của workshop, Daily Backup đã đủ để chứng minh quá trình sao lưu tự động.

---

### Tại sao giữ Recovery Point 7 ngày?

Tôi cấu hình:

```text
Retention = 7 days
```

vì bài lab chỉ cần giữ Recovery Point trong thời gian ngắn để thực hiện quy trình:

```text
Backup
   ↓
Mô phỏng sự cố
   ↓
Restore
   ↓
Validation
```

Việc giữ Backup trong thời gian dài hơn không mang lại thêm giá trị cho bài thử nhưng có thể làm tăng chi phí lưu trữ.

---

### Tại sao không sử dụng Cold Storage?

Cold Storage phù hợp hơn với các bản Backup cần lưu trữ trong thời gian dài.

Workshop chỉ sử dụng:

```text
Retention = 7 days
```

nên tôi giữ:

```text
Cold storage = Disabled
```

Việc chuyển các bản Backup ngắn hạn sang tầng lưu trữ dài hạn không phù hợp với mục tiêu và vòng đời dữ liệu của bài lab.

---

### Point-in-time recovery

Tôi giữ:

```text
Point-in-time recovery
= Disabled
```
Mục tiêu của workshop là thực hiện **scheduled recovery** từ một Recovery Point cụ thể.

---

### Cấu hình S3 Backup

Trong phần Advanced settings của S3, tôi không sử dụng Backup ACL.

S3 Bucket đã được cấu hình từ trước:

```text
ACLs disabled
Bucket owner enforced
```

Do đó việc đưa ACL vào Backup không cần thiết.

Tôi sử dụng:

```text
Back up ACLs = Disabled
Back up object tags = Disabled
```

Object Tag cũng không được sử dụng trong tình huống kiểm thử nên không cần được bảo vệ trong phạm vi workshop.

---

### Recovery Point Tag

Tôi thêm Tag:

```text
Project = AWS-Backup-Recovery-Lab
```

cho Recovery Point.

Tag này giúp nhận biết các bản Backup thuộc workshop và thuận tiện hơn trong quá trình quản lý tài nguyên.

Sau khi hoàn thành cấu hình, tôi chọn:

```text
Create plan
```

---

### Gán tài nguyên vào Backup Plan

Backup Plan mới chỉ xác định:

```text
Backup khi nào?

Backup giữ bao lâu?

Backup lưu ở Vault nào?
```

nhưng chưa xác định:

```text
Backup tài nguyên nào?
```

Do đó tôi tiếp tục chọn:

```text
Assign resources
```

Tên Resource Assignment:

```text
BackupLab-Resources
```

---

### Cấu hình IAM Role cho AWS Backup

Tại phần IAM Role, tôi chọn:

```text
Create default role
```

AWS Backup sẽ tạo hoặc sử dụng:

```text
AWSBackupDefaultServiceRole
```

Role này cho phép AWS Backup thực hiện các thao tác cần thiết khi Backup và Restore tài nguyên.

![Cấu hình IAM Role cho AWS Backup](/images/5/5.5.2/0006.png?featherlight=false&width=70pc)
<p align="center">
  <b>Cấu hình IAM Role cho AWS Backup</b>
</p>

Role này khác hoàn toàn với:

```text
BackupLab-EC2-S3-Role
```

đã được tạo ở phần 5.4.

---

### Phân biệt hai IAM Role

IAM Role dành cho EC2:

```text
BackupLab-EC2-S3-Role
        ↓
EC2 sử dụng
        ↓
Truy cập S3
```

Trong khi IAM Role dành cho AWS Backup:

```text
AWSBackupDefaultServiceRole
        ↓
AWS Backup sử dụng
        ↓
Backup / Restore
```

Tôi không sử dụng `BackupLab-EC2-S3-Role` làm Service Role cho AWS Backup vì hai Role phục vụ hai mục đích khác nhau.

---

### Kiểm tra quyền Backup S3

`AWSBackupDefaultServiceRole` sử dụng các policy cần thiết để AWS Backup thực hiện Backup và Restore.

Các policy chung gồm:

```text
AWSBackupServiceRolePolicyForBackup

AWSBackupServiceRolePolicyForRestores
```

Đối với Amazon S3, AWS Backup còn sử dụng các policy liên quan tới Backup và Restore S3:

```text
AWSBackupServiceRolePolicyForS3Backup

AWSBackupServiceRolePolicyForS3Restore
```

Vì workshop có cả Backup và Restore dữ liệu S3, tôi kiểm tra Role để bảo đảm các quyền cần thiết đã được cấu hình trước khi tiếp tục.

![AWSBackupDefaultServiceRole được sử dụng bởi AWS Backup](/images/5/5.5.2/0009.png?featherlight=false&width=70pc)

<p align="center">
  <b>AWSBackupDefaultServiceRole được sử dụng bởi AWS Backup</b>
</p>

---

### Chọn Resource cần Backup

Tại **Resource selection**, tôi chọn:

```text
Include specific resource types
```

thay vì:

```text
Include all resource types
```

Lý do là workshop chỉ cần bảo vệ đúng các tài nguyên đã được thiết kế.

Tôi chọn:

```text
Amazon EC2
→ backup-lab-web-server

Amazon S3
→ aws-backup-lab-s3-162918102026
```
![Gán EC2 và S3 vào BackupLab-Daily-Plan](/images/5/5.5.2/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Gán EC2 và S3 vào BackupLab-Daily-Plan</b>
</p>
---

### Tại sao không chọn EBS riêng?

EC2 trong workshop có:

```text
Root EBS
+
Data EBS /data
```

Tuy nhiên, tôi **không gán EBS Volume thành một resource độc lập** trong Backup Plan.

Trong bài lab:

```text
EC2
├── Root EBS
└── Data EBS
        ↓
   Backup EC2
```

đã đáp ứng mục tiêu bảo vệ toàn bộ Web Server.

Nếu tôi tiếp tục chọn Data EBS thành một tài nguyên riêng:

```text
EC2 Backup
+
EBS Backup riêng
```

thì sẽ tạo thêm một luồng Backup độc lập không cần thiết đối với phạm vi của workshop.

---

### Exclude Resources và Refine Selection

Tôi để trống:

```text
Exclude specific resource IDs
```

vì chỉ những resource cụ thể đã được lựa chọn mới được đưa vào Backup Plan.

Tôi cũng không sử dụng:

```text
Refine selection using tags
```

Tag-based selection phù hợp hơn khi hệ thống có nhiều resource và cần tự động bảo vệ các tài nguyên dựa trên Tag.

Trong workshop chỉ có một EC2 và một S3 Bucket nên việc lựa chọn trực tiếp giúp cấu hình dễ kiểm tra và dễ quản lý hơn.

Sau khi hoàn tất, tôi chọn:

```text
Assign resources
```

![Gán EC2 và S3 vào BackupLab-Daily-Plan](/images/5/5.5.2/0008.png?featherlight=false&width=70pc)

<p align="center">
  <b>Gán EC2 và S3 vào BackupLab-Daily-Plan</b>
</p>

Sau bước này, `BackupLab-Daily-Plan` đã có đầy đủ Backup Rule và các tài nguyên cần được bảo vệ.