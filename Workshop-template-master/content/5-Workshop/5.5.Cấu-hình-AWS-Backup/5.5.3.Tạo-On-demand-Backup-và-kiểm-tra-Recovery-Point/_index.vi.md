---
title: "Tạo On-demand Backup và kiểm tra Recovery Point"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---

## Tạo On-demand Backup và kiểm tra Recovery Point

Sau khi Backup Plan được hoàn thành, hệ thống đã có khả năng chạy Backup tự động theo lịch.

Tuy nhiên, nếu chờ đến thời điểm:

```text
00:30
```

mới kiểm thử thì quá trình thực hiện workshop sẽ mất nhiều thời gian.

Vì vậy, tôi sử dụng **On-demand Backup** để tạo Recovery Point ngay lập tức.

On-demand Backup không thay thế Backup Plan.

Hai cơ chế tồn tại song song:

```text
Backup Plan
→ Backup định kỳ

On-demand Backup
→ Backup ngay khi cần
```

On-demand Backup được sử dụng trong bài lab để kiểm tra hệ thống trước khi bước sang tình huống mô phỏng sự cố.

---

### Kiểm tra dữ liệu trước khi Backup

Trước khi chạy Backup, tôi kiểm tra lại trạng thái dữ liệu hiện tại.

Trên EBS:

```bash
cat /data/version.txt
cat /data/customer.txt
cat /data/website-data/app-data.txt
```

Kết quả:

```text
AWS Backup Recovery Lab - Version 1
Customer data before backup
Important website data
```

Trên Amazon S3:

```text
application-data/
├── s3-version.txt
└── ec2-version.txt
```

File chính được sử dụng cho bài kiểm thử là:

```text
s3-version.txt
```

với nội dung:

```text
AWS Backup Recovery Lab - S3 Version 1
```

Đây chính là trạng thái **Before Incident** mà tôi muốn lưu lại trong Recovery Point.

![Trạng thái dữ liệu trước khi thực hiện Backup](/images/5/5.5.3/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Trạng thái dữ liệu trước khi thực hiện Backup</b>
</p>

---

### Tạo On-demand Backup cho EC2

Tôi truy cập:

```text
AWS Backup
→ Protected resources / Create on-demand backup
```
![Tạo On-demand Backup cho EC2](/images/5/5.5.3/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo On-demand Backup cho EC2</b>
</p>


Tại phần **Resource type**, tôi chọn:

```text
EC2
```

Resource:

```text
backup-lab-web-server
```

Backup window:

```text
Create backup now
```

Backup Vault:

```text
BackupLab-Vault
```

Retention:

```text
7 days
```

IAM Role:

```text
AWSBackupDefaultServiceRole
```

Sau khi kiểm tra cấu hình, tôi chọn:

```text
Create on-demand backup
```
![Tạo On-demand Backup cho EC2](/images/5/5.5.3/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo On-demand Backup cho EC2</b>
</p>

![Tạo On-demand Backup cho EC2](/images/5/5.5.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo On-demand Backup cho EC2</b>
</p>
---

### Theo dõi EC2 Backup Job

Sau khi tạo On-demand Backup, tôi truy cập:

```text
AWS Backup
→ Jobs
→ Backup jobs
```

Backup Job có thể lần lượt chuyển qua các trạng thái:

```text
CREATED
   ↓
RUNNING
   ↓
COMPLETED
```

Tôi chỉ tiếp tục bài lab khi:

```text
Status = COMPLETED
```

Trạng thái `COMPLETED` cho biết AWS Backup đã tạo thành công Recovery Point của EC2.

![EC2 Backup Job hoàn thành với trạng thái COMPLETED](/images/5/5.5.3/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 Backup Job hoàn thành với trạng thái COMPLETED</b>
</p>

---

### Tạo On-demand Backup cho Amazon S3

Sau khi EC2 Backup Job hoàn thành, tôi tiếp tục tạo một On-demand Backup riêng cho Amazon S3.

Resource type:

```text
S3
```

Resource:

```text
aws-backup-lab-s3-162918102026
```

Cấu hình:

```text
Create backup now

Backup Vault:
BackupLab-Vault

Retention:
7 days

IAM Role:
AWSBackupDefaultServiceRole
```

Sau khi kiểm tra cấu hình, tôi chọn:

```text
Create on-demand backup
```
![Tạo On-demand Backup cho s3](/images/5/5.5.3/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo On-demand Backup cho s3</b>
</p>

![Tạo On-demand Backup cho S3](/images/5/5.5.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo On-demand Backup cho S3</b>
</p>
---

### Theo dõi S3 Backup Job

S3 Backup Job có thể mất nhiều thời gian hơn EC2, đặc biệt ở lần Backup đầu tiên.

Trong thời gian Backup đang chạy, tôi không:

```text
Xóa object

Thay đổi dữ liệu dùng để kiểm thử

Tạo thêm một Backup Job trùng lặp
```

Tôi tiếp tục theo dõi tại:

```text
AWS Backup
→ Jobs
→ Backup jobs
```

cho đến khi:

```text
S3
Status = COMPLETED
```

Chỉ khi Backup Job của S3 hoàn thành, tôi mới tiếp tục sang bước kiểm tra Recovery Point.

![Tạo On-demand Backup cho S3](/images/5/5.5.3/0008.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo On-demand Backup cho S3</b>
</p>

---

### Kiểm tra Protected Resources

Sau khi các Backup Job đầu tiên hoàn thành, tôi truy cập:

```text
AWS Backup
→ Protected resources
```

Hệ thống cần hiển thị các tài nguyên đã được bảo vệ:

```text
Amazon EC2

Amazon S3
```

Điều này cho thấy AWS Backup đã tạo Recovery Point tương ứng với các tài nguyên của workshop.

![EC2 và S3 xuất hiện trong danh sách Protected resources](/images/5/5.5.3/0009.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 và S3 xuất hiện trong danh sách Protected resources</b>
</p>

---

### Kiểm tra Recovery Point trong Backup Vault

Tôi tiếp tục truy cập:

```text
AWS Backup
→ Backup vaults
→ BackupLab-Vault
```

Trước khi thực hiện Backup:

```text
Recovery Points = 0
```

Sau khi Backup thành công, `BackupLab-Vault` cần chứa Recovery Point của:

```text
Amazon EC2

Amazon S3
```

Recovery Point có thể được hiểu là:

> **Trạng thái đã được sao lưu của tài nguyên tại một thời điểm xác định.**

AWS Backup lưu và quản lý các Recovery Point này trong Backup Vault.

Các Recovery Point được tạo ở bước này sẽ là nguồn dữ liệu được sử dụng khi thực hiện Restore ở phần sau.

![Recovery Point của EC2 và S3 được lưu trong BackupLab-Vault](/images/5/5.5.3/0010.png?featherlight=false&width=70pc)

<p align="center">
  <b>Recovery Point của EC2 và S3 được lưu trong BackupLab-Vault</b>
</p>
