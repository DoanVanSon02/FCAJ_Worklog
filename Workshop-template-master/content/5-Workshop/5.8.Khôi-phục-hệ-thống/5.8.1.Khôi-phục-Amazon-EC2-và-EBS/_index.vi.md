---
title: "Khôi phục Amazon EC2 và EBS"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.8.1. </b> "
---

## Khôi phục Amazon EC2 và EBS

Sau khi mô phỏng sự cố, tôi sử dụng Recovery Point của Amazon EC2 đã được tạo **trước thời điểm xảy ra sự cố** để thực hiện quá trình Restore.

Mục tiêu của bước này là tạo lại một EC2 Instance cùng Root EBS và Data EBS từ trạng thái đã được lưu trong Recovery Point.

---

### Lựa chọn Recovery Point

Tôi truy cập:

```text
AWS Backup
→ Protected resources
```

Sau đó chọn tài nguyên Amazon EC2 tương ứng với:

```text
backup-lab-web-server
```

Trong danh sách Recovery Point, tôi lựa chọn bản Backup được tạo **trước khi mô phỏng sự cố**.

Sau đó chọn:

```text
Restore
```

Điều quan trọng là không lựa chọn một Recovery Point được tạo sau khi dữ liệu đã bị thay đổi.

Recovery Point cần sử dụng phải chứa trạng thái:

```text
Website
→ Hoạt động bình thường

version.txt
→ Version 1

customer.txt
→ Tồn tại

app-data.txt
→ Important website data
```
![Khôi phục EC2](/images/5/5.8.1/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>[Khôi phục EC2</b>
</p>
---

### Cách AWS Backup khôi phục EC2

AWS Backup không ghi đè trực tiếp lên EC2 đang gặp sự cố.

Khi Restore một EC2 Recovery Point, AWS Backup tạo một tập tài nguyên mới gồm:

```text
EC2 Instance mới
+
Root EBS Volume
+
Data EBS Volume
```

Do đó, EC2 gốc vẫn được giữ nguyên ở trạng thái sau sự cố.


Cách khôi phục này phù hợp với workshop vì tôi có thể giữ lại EC2 bị sự cố để so sánh trực tiếp với EC2 được Restore.

---

### Cấu hình Restore

Tại màn hình Restore, tôi kiểm tra các thông tin:

```text
Instance type

VPC

Subnet

Security Group

Instance profile

Restore role
```

Tôi giữ cấu hình mạng tương tự Web Server ban đầu để EC2 được Restore có thể hoạt động trong cùng môi trường.

Cấu hình sử dụng:

```text
Instance type:
t3.micro

VPC:
VPC đang sử dụng cho workshop

Subnet:
Subnet tương ứng

Security Group:
backup-lab-web-sg
```

AWS Backup có thể sử dụng thông tin cấu hình của EC2 được bảo vệ để đưa ra các giá trị mặc định cho:

```text
Instance Type
VPC
Subnet
Security Group
```

Tôi kiểm tra lại các thông tin này trước khi bắt đầu Restore để bảo đảm EC2 mới được tạo trong đúng môi trường của workshop.

![Cấu hình Restore Amazon EC2 từ Recovery Point](/images/5/5.8.1/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Cấu hình Restore Amazon EC2 từ Recovery Point</b>
</p>

---

### Instance Profile của EC2 Restore

EC2 ban đầu đang sử dụng IAM Role:

```text
BackupLab-EC2-S3-Role
```

Role này được sử dụng để:

```text
EC2
 ↓
BackupLab-EC2-S3-Role
 ↓
Amazon S3
```

Trong quá trình Restore, có thể sử dụng lại Instance Profile của EC2 gốc hoặc tạo EC2 được Restore mà chưa gắn Instance Profile.

Chọn:

```text
Restore with original IAM role
```

---

### Restore Role

Tại mục:

```text
Restore role
```

tôi lựa chọn:

```text
Default
```

Đây là IAM Role được AWS Backup sử dụng để thực hiện các thao tác Restore.

![Cấu hình Restore Amazon EC2 từ Recovery Point](/images/5/5.8.1/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Cấu hình Restore Amazon EC2 từ Recovery Point</b>
</p>

---

### Bắt đầu Restore

Sau khi kiểm tra toàn bộ cấu hình, tôi chọn:

```text
Restore backup
```

Sau đó truy cập:

```text
AWS Backup
→ Jobs
→ Restore jobs
```

để theo dõi quá trình Restore.

Restore Job có thể lần lượt chuyển qua các trạng thái:

```text
PENDING
   ↓
RUNNING
   ↓
COMPLETED
```

Tôi chỉ tiếp tục kiểm tra EC2 sau khi:

```text
Status = COMPLETED
```

Điều này cho biết AWS Backup đã hoàn thành quá trình tạo lại tài nguyên từ Recovery Point.

![EC2 Restore Job hoàn thành với trạng thái COMPLETED](/images/5/5.8.1/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 Restore Job hoàn thành với trạng thái COMPLETED</b>
</p>

---

### Xác định EC2 được Restore

Sau khi Restore Job hoàn thành, tôi truy cập:

```text
Amazon EC2
→ Instances
```

AWS Backup đã tạo một EC2 Instance mới.

Để tránh nhầm lẫn với EC2 đang gặp sự cố, tôi đặt tên Instance mới là:

```text
backup-lab-web-server-restored
```

Tại thời điểm này, hệ thống có hai Instance:

```text
backup-lab-web-server
→ EC2 gốc
→ Trạng thái sau Incident


backup-lab-web-server-restored
→ EC2 mới
→ Được tạo từ Recovery Point
```

Tôi kiểm tra EC2 mới và xác nhận:

```text
Instance state:
Running

Status checks:
3/3 checks passed
```

![EC2 gốc và EC2 được khôi phục tồn tại đồng thời](/images/5/5.8.1/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 gốc và EC2 được khôi phục tồn tại đồng thời</b>
</p>

---
