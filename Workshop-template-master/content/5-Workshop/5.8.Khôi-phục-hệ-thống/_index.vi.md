---
title: "Khôi phục và kiểm tra hệ thống"
date: 2026-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

### Tổng quan

Sau khi hoàn thành quá trình mô phỏng sự cố ở phần 5.7, trạng thái hiện tại của hệ thống đã khác với trạng thái được lưu trong Recovery Point.

Có thể so sánh như sau:

```text
RECOVERY POINT
────────────────────────────
Website       = Hoạt động bình thường
version.txt   = Version 1
customer.txt  = Tồn tại
app-data.txt  = Dữ liệu bình thường
s3-version    = Tồn tại


CURRENT SYSTEM AFTER INCIDENT
────────────────────────────
Website       = INCIDENT TEST
version.txt   = Version 2
customer.txt  = Bị xóa
app-data.txt  = Corrupted
s3-version    = Bị xóa
```

Trong phần này, tôi sử dụng Recovery Point đã được tạo **trước thời điểm xảy ra sự cố** để khôi phục EC2/EBS và dữ liệu Amazon S3.

Mục tiêu cuối cùng là kiểm tra xem hệ thống có thể quay trở lại trạng thái ban đầu hay không.

Luồng khôi phục của workshop:

```text
Recovery Point
      ↓
AWS Backup
      ↓
Restore Job
   ┌───────┴────────┐
   ↓                ↓
EC2 + EBS          S3
   ↓                ↓
Validation       Validation
   └───────┬────────┘
           ↓
      Recovery Pass
```

Đối với Amazon EC2, AWS Backup không ghi đè trực tiếp lên EC2 đang gặp sự cố.

Thay vào đó, Recovery Point được sử dụng để tạo một EC2 Instance mới cùng các EBS Volume tương ứng.

Do đó, sau quá trình Restore có thể tồn tại song song:

```text
EC2 GỐC
→ Trạng thái sau Incident

EC2 RESTORED
→ Trạng thái từ Recovery Point
```

Cách này giúp tôi dễ dàng so sánh trực tiếp giữa hệ thống bị sự cố và hệ thống đã được khôi phục.

Đối với Amazon S3, object:

```text
application-data/s3-version.txt
```

đã bị xóa trong phần 5.7.

Trong phần Restore, tôi sử dụng Recovery Point của Amazon S3 để thực hiện:

```text
Item-level Restore
```

thay vì khôi phục toàn bộ Bucket.

Luồng khôi phục S3:

```text
S3 Source Bucket
       │
       │ Backup
       ▼
   AWS Backup
       │
       ▼
Recovery Point
       │
       │ Restore
       ▼
S3 Source Bucket
```

Sau khi các Restore Job hoàn thành, tôi không chỉ kiểm tra trạng thái:

```text
COMPLETED
```

trong AWS Backup mà còn kiểm tra trực tiếp dữ liệu.

Các dữ liệu cần được xác nhận gồm:

```text
version.txt
→ Version 1

customer.txt
→ Xuất hiện trở lại

app-data.txt
→ Important website data

Website
→ AWS Backup & Recovery Lab

s3-version.txt
→ Xuất hiện trở lại
```

Điều này giúp phân biệt rõ:

```text
Restore Job hoàn thành
```

với:

```text
Dữ liệu thực sự được khôi phục đúng
```

Kết quả cuối cùng được kiểm tra theo ba trạng thái:

```text
Before Incident
       ↓
After Incident
       ↓
After Restore
```

Mục tiêu là chứng minh dữ liệu sau Restore quay trở lại đúng trạng thái đã được lưu trong Recovery Point trước sự cố.

Luồng end-to-end của workshop lúc này trở thành:

```text
BUILD
  ↓
BACKUP
  ↓
MONITOR
  ↓
INCIDENT
  ↓
RESTORE
  ↓
VALIDATE
```

Nếu toàn bộ dữ liệu và dịch vụ quay trở lại trạng thái mong muốn thì quá trình Backup & Recovery được xem là:

```text
PASS
```

#### Nội dung

1. [Khôi phục Amazon EC2 và EBS](5.8.1.khôi-phục-amazon-ec2-và-ebs/)
2. [Khôi phục dữ liệu Amazon S3](5.8.2.Khôi-phục-dữ-liệu-amazon-s3/)
3. [Kiểm tra dữ liệu sau khi khôi phục](5.8.3.kiểm-tra-dữ-liệu-sau-khi-khôi-phục/)