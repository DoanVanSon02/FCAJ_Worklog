---
title: "Mô phỏng sự cố trên EC2 và EBS"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.7.2. </b> "
---

### Mô phỏng sự cố trên EC2 và EBS

Sau khi xác nhận Recovery Point đã tồn tại, tôi bắt đầu mô phỏng sự cố trên EC2 và EBS.

Tôi kết nối lại với EC2:

```text
backup-lab-web-server
```

thông qua:

```text
In SSH client
```

Dữ liệu trên EBS Data Volume hiện được lưu tại:

```text
/data
```

Trước khi thay đổi, tôi kiểm tra lại trạng thái ban đầu:

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

Đây chính là trạng thái dữ liệu đã được lưu trong Recovery Point.

---

#### Thay đổi `version.txt`

Đầu tiên, tôi mô phỏng trường hợp dữ liệu bị ghi đè hoặc chỉnh sửa sau thời điểm Backup.

File được sử dụng:

```text
/data/version.txt
```

Trước sự cố, nội dung của file là:

```text
AWS Backup Recovery Lab - Version 1
```

Tôi thay đổi nội dung thành Version 2:

```bash
echo "AWS Backup Recovery Lab - Version 2 - DATA CHANGED" | sudo tee /data/version.txt
```

Sau đó kiểm tra lại:

```bash
cat /data/version.txt
```

Kết quả:

```text
AWS Backup Recovery Lab - Version 2 - DATA CHANGED
```

Trạng thái dữ liệu lúc này:

```text
Before Backup:
Version 1

After Incident:
Version 2
```

Sau khi thực hiện Restore ở phần tiếp theo, tôi kỳ vọng dữ liệu quay trở lại:

```text
After Restore:
Version 1
```

---

#### Xóa `customer.txt`

Tiếp theo, tôi mô phỏng trường hợp một file dữ liệu quan trọng bị xóa sau thời điểm Backup.

File ban đầu:

```text
/data/customer.txt
```

Tôi thực hiện:

```bash
sudo rm /data/customer.txt
```

Sau đó kiểm tra thư mục:

```bash
ls -l /data
```

Lúc này:

```text
customer.txt
```

không còn xuất hiện trong danh sách.

Tôi có thể kiểm tra trực tiếp:

```bash
cat /data/customer.txt
```

Hệ thống sẽ báo file không tồn tại.

Tình huống này mô phỏng trường hợp một file dữ liệu quan trọng bị người dùng hoặc ứng dụng xóa nhầm sau khi bản Backup đã được tạo.

Trạng thái:

```text
Before Incident:
customer.txt = tồn tại

After Incident:
customer.txt = bị xóa
```

Sau quá trình Restore, mục tiêu là file:

```text
/data/customer.txt
```

xuất hiện trở lại với nội dung:

```text
Customer data before backup
```

---

#### Làm sai nội dung `app-data.txt`

Tôi tiếp tục mô phỏng trường hợp file vẫn tồn tại nhưng nội dung bên trong không còn chính xác.

File:

```text
/data/website-data/app-data.txt
```

ban đầu chứa:

```text
Important website data
```

Tôi ghi đè nội dung bằng:

```bash
echo "CORRUPTED APPLICATION DATA" | sudo tee /data/website-data/app-data.txt
```

Sau đó kiểm tra:

```bash
cat /data/website-data/app-data.txt
```

Kết quả:

```text
CORRUPTED APPLICATION DATA
```

Trường hợp này khác với việc xóa file.

File vẫn tồn tại nhưng dữ liệu bên trong đã bị thay đổi.

Sau ba bài kiểm thử, trạng thái dữ liệu trên EBS là:

```text
version.txt
→ dữ liệu bị thay đổi

customer.txt
→ bị xóa

app-data.txt
→ nội dung bị sai
```

Việc tạo ba dạng sự cố khác nhau giúp quá trình Restore sau này kiểm tra được nhiều tình huống hơn thay vì chỉ một loại lỗi duy nhất.

![Mô phỏng sự cố trên EBS](/FCAJ_Worklog/images/5/5.7.2/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Mô phỏng sự cố trên EBS</b>
</p>
---

#### Thay đổi nội dung website

Ngoài dữ liệu trên EBS, tôi thay đổi nội dung trang Web để dễ dàng quan sát sự khác biệt giữa EC2 hiện tại và EC2 được Restore sau này.

File website:

```text
/var/www/html/index.html
```

Tôi chỉnh sửa bằng:

```bash
sudo nano /var/www/html/index.html
```

Sau đó thay nội dung thành:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Incident Test</title>
</head>
<body>
    <h1>INCIDENT TEST</h1>

    <h2>Website data was changed after backup.</h2>

    <p>This page represents the system state after an incident.</p>
</body>
</html>
```

Sau khi lưu, tôi truy cập lại:

```text
http://<PUBLIC-IP>
```

Website lúc này hiển thị:

```text
INCIDENT TEST

Website data was changed after backup.
```

Việc thay đổi Website giúp tôi dễ dàng phân biệt hai trạng thái:

```text
EC2 hiện tại
→ INCIDENT TEST

EC2 được Restore sau này
→ AWS Backup & Recovery Lab
```
![Mô phỏng sự cố trên EBS](/FCAJ_Worklog/images/5/5.7.2/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Mô phỏng sự cố trên EBS</b>
</p>

Nếu quá trình Restore thành công, EC2 được khôi phục phải hiển thị lại nội dung Website trước sự cố.

---
