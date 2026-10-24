---
title: "Kiểm tra dữ liệu sau khi khôi phục"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.8.3. </b> "
---

## Kiểm tra dữ liệu sau khi khôi phục

Restore Job hiển thị:

```text
COMPLETED
```

chưa phải là bằng chứng duy nhất để kết luận hệ thống đã được phục hồi chính xác.

Tôi cần kiểm tra trực tiếp dữ liệu sau Restore.

Điều này giúp phân biệt:

```text
Restore Job hoàn thành
```

với:

```text
Dữ liệu thực sự được khôi phục đúng
```

Vì vậy, tôi thực hiện Validation cho từng thành phần của hệ thống.

---

### Kiểm tra `version.txt`

Trên EC2 đã được Restore:

```text
backup-lab-web-server-restored
```

tôi chạy:

```bash
cat /data/version.txt
```

Trước sự cố, dữ liệu là:

```text
AWS Backup Recovery Lab - Version 1
```

Sau khi mô phỏng sự cố:

```text
AWS Backup Recovery Lab - Version 2 - DATA CHANGED
```

Sau Restore, kết quả mong đợi là:

```text
AWS Backup Recovery Lab - Version 1
```

Luồng kiểm thử:

```text
BEFORE BACKUP
Version 1
     ↓
INCIDENT
Version 2
     ↓
RESTORE
Version 1 ✓
```

Kết quả này chứng minh dữ liệu bị thay đổi sau thời điểm Backup không xuất hiện trong Recovery Point được sử dụng để Restore.

---

### Kiểm tra `customer.txt`

Tiếp theo, tôi kiểm tra thư mục:

```bash
ls -l /data
```

và xác nhận file:

```text
customer.txt
```

đã xuất hiện trở lại.

Sau đó kiểm tra nội dung:

```bash
cat /data/customer.txt
```

Kết quả mong đợi:

```text
Customer data before backup
```

Luồng kiểm thử:

```text
BEFORE INCIDENT
customer.txt tồn tại
        ↓
INCIDENT
customer.txt bị xóa
        ↓
RESTORE
customer.txt xuất hiện lại ✓
```

Điều này chứng minh file đã bị xóa sau thời điểm Backup có thể được phục hồi từ Recovery Point.

---

### Kiểm tra `app-data.txt`

Tôi tiếp tục kiểm tra:

```bash
cat /data/website-data/app-data.txt
```

Kết quả mong đợi:

```text
Important website data
```

thay vì nội dung đã bị thay đổi trong phần mô phỏng sự cố:

```text
CORRUPTED APPLICATION DATA
```

Luồng kiểm thử:

```text
BEFORE
Important website data
        ↓
INCIDENT
CORRUPTED APPLICATION DATA
        ↓
RESTORE
Important website data ✓
```

Qua ba file trên, tôi đã kiểm thử được ba dạng sự cố khác nhau:

```text
Dữ liệu bị thay đổi  → Restore ✓

File bị xóa          → Restore ✓

Nội dung bị sai      → Restore ✓
```
![Dữ liệu trên EBS được khôi phục về trạng thái trước sự cố](/FCAJ_Worklog/images/5/5.8.3/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Dữ liệu trên EBS được khôi phục về trạng thái trước sự cố</b>
</p>

Ảnh Terminal nên thể hiện các kết quả:

```text
AWS Backup Recovery Lab - Version 1

Customer data before backup

Important website data
```

Đây là một trong những bằng chứng quan trọng nhất của toàn bộ workshop vì nó chứng minh trực tiếp dữ liệu trên EBS đã được khôi phục về trạng thái trước sự cố.

---

### Kiểm tra Apache

Sau khi xác nhận dữ liệu trên EBS, tôi kiểm tra trạng thái Apache Web Server:

```bash
sudo systemctl status httpd
```

Kết quả mong đợi:

```text
Active: active (running)
```

Nếu Apache chưa chạy, tôi có thể khởi động lại bằng:

```bash
sudo systemctl start httpd
```

Sau đó kiểm tra lại:

```bash
sudo systemctl status httpd
```

Mục tiêu là bảo đảm Web Server trên EC2 được Restore có thể hoạt động bình thường.

![Dữ liệu trên EBS được khôi phục về trạng thái trước sự cố](/FCAJ_Worklog/images/5/5.8.3/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Dữ liệu trên EBS được khôi phục về trạng thái trước sự cố</b>
</p>
---

### Kiểm tra Website

Tôi lấy:

```text
Public IPv4
```

của EC2:

```text
backup-lab-web-server-restored
```

Sau đó truy cập:

```text
http://<RESTORED-PUBLIC-IP>
```

Website được Restore cần hiển thị lại nội dung ban đầu:

```text
AWS Backup & Recovery Lab

Web Server is running successfully.
```

Trong khi EC2 gốc vẫn giữ trạng thái sau sự cố:

```text
INCIDENT TEST
```

Do đó, tôi có thể mở hai tab trình duyệt để so sánh trực tiếp:

```text
EC2 GỐC
────────────────
INCIDENT TEST


EC2 RESTORED
────────────────
AWS Backup & Recovery Lab
Web Server is running successfully.
```

![So sánh website EC2 gặp sự cố và EC2 được khôi phục](/FCAJ_Worklog/images/5/5.8.3/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>So sánh website EC2 gặp sự cố và EC2 được khôi phục</b>
</p>

![So sánh website EC2 gặp sự cố và EC2 được khôi phục](/FCAJ_Worklog/images/5/5.8.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>So sánh website EC2 gặp sự cố và EC2 được khôi phục</b>
</p>

Hình này giúp chứng minh trực quan rằng EC2 được Restore đã quay trở lại trạng thái Website trước khi xảy ra sự cố.

---

### Kiểm tra Amazon S3

Sau khi kiểm tra EC2/EBS, tôi tiếp tục xác nhận dữ liệu Amazon S3.

Tôi truy cập:

```text
Amazon S3
→ Bucket
→ application-data/
```

và xác nhận object:

```text
s3-version.txt
```

đã xuất hiện trở lại.

Sau đó tôi mở hoặc Download object để kiểm tra nội dung.

Kết quả mong đợi:

```text
AWS Backup Recovery Lab - S3 Version 1
```

Luồng kiểm thử:

```text
BEFORE INCIDENT
s3-version.txt tồn tại
        ↓
INCIDENT
s3-version.txt bị xóa
        ↓
AWS BACKUP RESTORE
        ↓
s3-version.txt xuất hiện lại ✓
```

![s3-version.txt xuất hiện trở lại sau khi Restore](/FCAJ_Worklog/images/5/5.8.3/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>s3-version.txt xuất hiện trở lại sau khi Restore</b>
</p>

![ Nội dung s3-version.txt được khôi phục về Version 1](/FCAJ_Worklog/images/5/5.8.3/0006.png?featherlight=false&width=70pc)

<p align="center">
  <b>Nội dung s3-version.txt được khôi phục về Version 1</b>
</p>

---

### Kiểm tra Restore Jobs

Cuối cùng, tôi quay lại:

```text
AWS Backup
→ Jobs
→ Restore jobs
```

và xác nhận:

```text
EC2 Restore Job
Status = COMPLETED

S3 Restore Job
Status = COMPLETED
```

![Các Restore Job của EC2 và S3 hoàn thành thành công](/FCAJ_Worklog/images/5/5.8.3/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Các Restore Job của EC2 và S3 hoàn thành thành công</b>
</p>

Ảnh này cung cấp bằng chứng từ phía AWS Backup, trong khi các hình Terminal, Website và Amazon S3 cung cấp bằng chứng từ chính dữ liệu sau Restore.

Sau khi hoàn thành quá trình kiểm tra, kết quả cần đạt là:

```text
✓ version.txt quay trở lại Version 1

✓ customer.txt xuất hiện trở lại

✓ app-data.txt quay trở lại nội dung ban đầu

✓ Apache hoạt động bình thường

✓ Website Restore hiển thị nội dung trước sự cố

✓ s3-version.txt xuất hiện trở lại

✓ Nội dung s3-version.txt = S3 Version 1

✓ EC2 Restore Job = COMPLETED

✓ S3 Restore Job = COMPLETED
```