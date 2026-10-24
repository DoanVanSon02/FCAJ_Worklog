---
title: "Tạo CloudWatch Alarm cho EC2 và S3"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.6.2. </b> "
---

## Tạo CloudWatch Alarm cho EC2 và S3

Sau khi xác nhận Amazon CloudWatch đã nhận được metric từ AWS Backup, tôi tạo **CloudWatch Alarm** để kiểm thử cơ chế giám sát tự động.

Trong workshop, tôi tạo Alarm riêng cho:

```text
Amazon EC2
Amazon S3
```

dựa trên metric:

```text
NumberOfBackupJobsCompleted
```

Mục tiêu của Alarm trong workshop là:

> Xác nhận rằng CloudWatch có thể đánh giá AWS Backup Metric và tự động thay đổi trạng thái Alarm khi điều kiện đã cấu hình được đáp ứng.

---

### Tạo Alarm cho Amazon EC2

![Tạo Alarm cho Amazon EC2](/images/5/5.6.2/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon EC2</b>
</p>

![Tạo Alarm cho Amazon EC2](/images/5/5.6.2/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon EC2</b>
</p>

Tôi lựa chọn metric với các thông tin:

```text
Namespace:
AWS/Backup

Metric:
NumberOfBackupJobsCompleted

BackupVaultName:
BackupLab-Vault

ResourceType:
EC2
```
![Tạo Alarm cho Amazon EC2](/images/5/5.6.2/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon EC2</b>
</p>

![Tạo Alarm cho Amazon EC2](/images/5/5.6.2/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon EC2</b>
</p>

Sau đó chọn:

```text
Select metric
```

---

### Cấu hình Metric

Tại phần cấu hình Metric, tôi sử dụng:

```text
Statistic:
Sum

Period:
5 minutes
```

Tôi sử dụng `Sum` vì metric biểu diễn số lượng Backup Job xảy ra trong khoảng thời gian được theo dõi.

Period:

```text
5 minutes
```

được sử dụng để phù hợp với khoảng thời gian AWS Backup cập nhật metric sang Amazon CloudWatch.

---

### Cấu hình điều kiện Alarm

Tại phần điều kiện, tôi sử dụng:

```text
Threshold type:
Static
```

Điều kiện:

```text
Whenever:
NumberOfBackupJobsCompleted
is Greater/Equal
```

Threshold:

```text
1
```

Tức là:

```text
NumberOfBackupJobsCompleted >= 1
```

Alarm sẽ được kích hoạt khi CloudWatch ghi nhận ít nhất một Backup Job hoàn thành trong khoảng thời gian đánh giá.

Datapoint được cấu hình:

```text
1 out of 1
```

Điều này có nghĩa chỉ cần một datapoint đáp ứng điều kiện thì Alarm có thể chuyển sang trạng thái:

```text
ALARM
```

---

### Cấu hình Missing Data

Đối với Missing Data, tôi lựa chọn:

```text
Treat missing data as not breaching
```

Lý do là việc không có datapoint mới không đồng nghĩa với việc Backup Job gặp lỗi.

Có thể hiểu:

```text
Không có Backup Job chạy
          ≠
Backup Job thất bại
```

Do đó, khi không có dữ liệu mới, Alarm không nên tự động xem đây là một tình trạng vi phạm điều kiện.

![Tạo Alarm cho Amazon EC2](/images/5/5.6.2/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon EC2</b>
</p>

![Tạo Alarm cho Amazon EC2](/images/5/5.6.2/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon EC2</b>
</p>

---

### Notification

CloudWatch Alarm có thể kết hợp với Amazon SNS để gửi các loại thông báo như:

```text
Email
SMS
hoặc các workflow cảnh báo khác
```

Tuy nhiên, phạm vi của workshop tập trung vào việc chứng minh cơ chế:

```text
Metric
   ↓
Alarm Evaluation
   ↓
Alarm State
```

nên tôi không cấu hình SNS Notification.

Trong hệ thống thực tế, CloudWatch Alarm có thể được kết hợp với Amazon SNS hoặc các dịch vụ khác để gửi cảnh báo cho đội vận hành khi Backup Job gặp vấn đề.

Không cấu hình thì chúng ta xóa đi rồi cuộn xuống nhấn **Next**
![Tạo Alarm cho Amazon EC2](/images/5/5.6.2/0008.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon EC2</b>
</p>
---

### Đặt tên Alarm cho EC2

Alarm cho EC2 được đặt tên:

```text
BackupLab-BackupCompleted-Test-Alarm
```

Description:

```text
Test alarm when an EC2 AWS Backup job
completes successfully.
```

Sau đó tôi chọn:

```text
Create alarm
```

![Cấu hình CloudWatch Alarm theo dõi Backup Job của Amazon EC2](/images/5/5.6.2/0009.png?featherlight=false&width=70pc)

<p align="center">
  <b>Cấu hình CloudWatch Alarm theo dõi Backup Job của Amazon EC2</b>
</p>

![Tạo Alarm cho Amazon EC2](/images/5/5.6.2/0010.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon EC2</b>
</p>

---

### Tạo Alarm cho Amazon S3

![Tạo Alarm cho Amazon S3](/images/5/5.6.2/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon S3</b>
</p>

![Tạo Alarm cho Amazon S3](/images/5/5.6.2/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon S3</b>
</p>

Để giám sát cả hai Resource Type của workshop, tôi tiếp tục tạo một Alarm tương tự cho Amazon S3.

Metric được lựa chọn:

```text
Namespace:
AWS/Backup

Metric:
NumberOfBackupJobsCompleted

BackupVaultName:
BackupLab-Vault

ResourceType:
S3
```
![Tạo Alarm cho Amazon S3](/images/5/5.6.2/0011.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon S3</b>
</p>

Cấu hình:

```text
Statistic:
Sum

Period:
5 minutes

Threshold:
>= 1

Datapoints:
1 out of 1
```
![Tạo Alarm cho Amazon S3](/images/5/5.6.2/0012.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon S3</b>
</p>

![Tạo Alarm cho Amazon S3](/images/5/5.6.2/0013.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon S3</b>
</p>

Alarm được đặt tên:

```text
BackupLab-S3-BackupCompleted-Test-Alarm
```

Description:

```text
Test alarm when an S3 AWS Backup job
completes successfully.
```
![Tạo Alarm cho Amazon S3](/images/5/5.6.2/0014.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon S3</b>
</p>

![Tạo Alarm cho Amazon S3](/images/5/5.6.2/0015.png?featherlight=false&width=70pc)

<p align="center">
  <b>Tạo Alarm cho Amazon S3</b>
</p>

Việc tạo Alarm riêng cho EC2 và S3 giúp theo dõi hai Resource Type độc lập.

Có thể mô tả như sau:

```text
EC2 Backup
     ↓
EC2 Metric
     ↓
EC2 Alarm


S3 Backup
     ↓
S3 Metric
     ↓
S3 Alarm
```

Nếu chỉ sử dụng một metric tổng hợp theo Backup Vault, việc xác định Resource Type nào tạo ra datapoint sẽ khó rõ ràng hơn.

---

### Trạng thái của CloudWatch Alarm

Sau khi tạo, Alarm có thể bắt đầu ở trạng thái:

```text
INSUFFICIENT_DATA
```

Trạng thái này xuất hiện khi CloudWatch chưa có đủ datapoint để đánh giá điều kiện.

Khi datapoint phù hợp xuất hiện, Alarm có thể chuyển sang:

```text
ALARM
```

nếu:

```text
NumberOfBackupJobsCompleted >= 1
```

Nếu trong khoảng thời gian đánh giá không có datapoint vi phạm điều kiện, Alarm có thể ở trạng thái:

```text
OK
```

Có thể hiểu ba trạng thái chính như sau:

```text
INSUFFICIENT_DATA
→ Chưa đủ dữ liệu để đánh giá

OK
→ Điều kiện Alarm không bị vi phạm

ALARM
→ Điều kiện Alarm đã được đáp ứng
```

![Trạng thái CloudWatch Alarm sau khi đánh giá AWS Backup Metric](/images/5/5.6.2/0016.png?featherlight=false&width=70pc)

<p align="center">
  <b>Trạng thái CloudWatch Alarm sau khi đánh giá AWS Backup Metric</b>
</p>

---

### Vì sao Alarm của bài lab sử dụng `Completed` thay vì `Failed`?

Trong workshop, metric:

```text
NumberOfBackupJobsCompleted
```

với điều kiện:

```text
NumberOfBackupJobsCompleted >= 1
```

được sử dụng để **kiểm thử cơ chế CloudWatch Alarm**.

Nó giúp chứng minh luồng:

```text
AWS Backup
      ↓
Phát sinh Metric
      ↓
CloudWatch nhận Metric
      ↓
CloudWatch đánh giá Threshold
      ↓
Alarm thay đổi trạng thái
```

Tuy nhiên, trong môi trường thực tế, cảnh báo:

```text
Backup hoàn thành
```

thường không quan trọng bằng cảnh báo:

```text
Backup thất bại
```

Một cấu hình phù hợp hơn cho môi trường thực tế có thể sử dụng:

```text
Metric:
NumberOfBackupJobsFailed

Condition:
>= 1
```

Luồng cảnh báo khi đó:

```text
Backup Job FAILED
       ↓
NumberOfBackupJobsFailed
       ↓
CloudWatch Alarm
       ↓
SNS / Email / Incident
       ↓
Administrator
```

Do phạm vi workshop không yêu cầu cố tình làm hỏng một Backup Job đang hoạt động ổn định, tôi sử dụng `NumberOfBackupJobsCompleted` để kiểm thử chức năng của CloudWatch Alarm.

`NumberOfBackupJobsFailed` được ghi nhận như một hướng triển khai phù hợp hơn khi hệ thống được áp dụng trong môi trường thực tế.