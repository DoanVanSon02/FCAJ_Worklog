---
title: "Cài đặt Apache Web Server"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

### Cài đặt Apache Web Server

Sau khi EC2 hoạt động ổn định, tôi tiến hành cài đặt **Apache HTTP Server** để biến EC2 thành một Web Server thực tế.

Việc sử dụng Web Server giúp workshop có một hệ thống cụ thể để kiểm tra trạng thái trước và sau khi thực hiện Backup/Restore, thay vì chỉ sao lưu một EC2 trống.

---

#### Kết nối đến EC2

Tại AWS Management Console, tôi truy cập:

```text
EC2
→ Instances
→ backup-lab-web-server
→ Connect
```

Sau đó lựa chọn:

```text
In SSH Client
Copy đường dẫn ở "4. Connect to your instance using its Public DNS"
```

mở Terminal kết nối SSH với EC2 và thao tác.

![Kết nối EC2 bằng SSH](/FCAJ_Worklog/images/5/5.3.2/001.png?featherlight=false&width=70pc)

---

#### Cập nhật hệ thống

Trước khi cài đặt Apache, tôi cập nhật các package của hệ thống bằng lệnh:

```bash
sudo yum update -y
```

Sau khi quá trình cập nhật hoàn tất, EC2 đã sẵn sàng để cài đặt Web Server.

---

#### Cài đặt Apache HTTP Server

Tôi cài đặt Apache bằng lệnh:

```bash
sudo yum install httpd -y
```

Sau khi cài đặt hoàn tất, khởi động dịch vụ Apache:

```bash
sudo systemctl start httpd
```

Để Apache tự động khởi động khi EC2 được reboot, tôi thực hiện:

```bash
sudo systemctl enable httpd
```

---

#### Kiểm tra trạng thái Apache

Để kiểm tra Apache đã hoạt động hay chưa, tôi sử dụng:

```bash
sudo systemctl status httpd
```

Kết quả mong đợi:

```text
Active: active (running)
```

Trạng thái `active (running)` cho biết Apache HTTP Server đã được khởi động thành công và đang hoạt động trên EC2.

![Apache HTTP Server hoạt động trên EC2](/FCAJ_Worklog/images/5/5.3.2/002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.3.5: Apache HTTP Server hoạt động trên EC2</b>
</p>

---

#### Tạo nội dung Website

Sau khi Apache hoạt động, tôi tạo một trang Web thử nghiệm tại:

```text
/var/www/html/index.html
```

Mở file bằng lệnh:

```bash
sudo nano /var/www/html/index.html
```

Sau đó thêm nội dung:

```html
<!DOCTYPE html>
<html>
<head>
    <title>AWS Backup & Recovery Lab</title>
</head>

<body>
    <h1>AWS Backup & Recovery Lab</h1>

    <h2>Web Server is running successfully.</h2>

    <p>Server: Amazon EC2</p>
    <p>Operating System: Amazon Linux 2023</p>
    <p>Web Server: Apache HTTP Server</p>

    <hr>

    <p>This server will be protected using AWS Backup.</p>
</body>
</html>
```

Sau khi nhập nội dung, tôi lưu file `index.html`.

Website này đóng vai trò là nội dung ban đầu của Web Server và sẽ giúp kiểm tra trạng thái hệ thống trong quá trình Backup và Restore ở các phần tiếp theo.

---

#### Kiểm tra Website từ trình duyệt

Sau khi tạo trang Web, tôi lấy **Public IPv4 Address** của EC2 tại:

```text
EC2
→ Instances
→ backup-lab-web-server
```

Sau đó truy cập địa chỉ sau bằng trình duyệt:

```text
http://<PUBLIC-IP>
```

Trong đó:

```text
<PUBLIC-IP>
```

được thay bằng Public IPv4 thực tế của EC2.

Ví dụ:

```text
http://54.x.x.x
```

Nếu Apache và Security Group được cấu hình đúng, trình duyệt sẽ hiển thị nội dung:

```text
AWS Backup & Recovery Lab

Web Server is running successfully.

Server: Amazon EC2
Operating System: Amazon Linux 2023
Web Server: Apache HTTP Server

This server will be protected using AWS Backup.
```

![Website hoạt động thành công trên Amazon EC2](/FCAJ_Worklog/images/5/5.3.2/003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.3.6: Website hoạt động thành công trên Amazon EC2</b>
</p>

Ảnh trên thể hiện trạng thái ban đầu của Web Server **trước khi xảy ra sự cố**.

Ở phần Restore sau này, trạng thái của hệ thống sau khi khôi phục sẽ được kiểm tra và so sánh với trạng thái ban đầu này.

---

#### Kết quả

Sau khi hoàn thành phần này, hệ thống cần đạt được các trạng thái sau:

```text
Apache = active (running)

HTTP port 80 = accessible

Website = displayed successfully
```

EC2 hiện đã hoạt động như một Web Server và sẵn sàng để tiếp tục cấu hình **EBS Data Volume** nhằm lưu trữ dữ liệu ứng dụng.