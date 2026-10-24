---
title: "Khởi tạo Amazon EC2"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

### Khởi tạo Amazon EC2

Amazon EC2 được sử dụng làm Web Server chính của hệ thống. Instance này sẽ chạy Apache, chứa website thử nghiệm và được sử dụng làm tài nguyên cần bảo vệ bằng AWS Backup ở các phần tiếp theo.

Tôi truy cập:

```text
AWS Management Console
→ EC2
→ Instances
→ Launch instances
```

---

#### Đặt tên Instance

Tại phần **Name and tags**, tôi sử dụng:

```text
Name = backup-lab-web-server
```

---

#### Chọn Amazon Machine Image

Tại phần **Application and OS Images**, tôi lựa chọn:

```text
Amazon Linux 2023
Architecture: 64-bit (x86)
```

Amazon Linux 2023 được lựa chọn vì đây là hệ điều hành Linux được AWS cung cấp và tối ưu để chạy trên EC2.

Hệ điều hành này hỗ trợ trình quản lý gói `dnf`, phù hợp với việc cài đặt Apache và AWS CLI trong workshop.

---

![Đặt tên và chọn Amazon Machine Image cho EC2 Instance](/images/5/5.3.1/001.png?featherlight=false&width=70pc)
![Đặt tên và chọn Amazon Machine Image cho EC2 Instance](/images/5/5.3.1/002.png?featherlight=false&width=70pc)


<p align="center">
  <b>Hình 5.3.1: Đặt tên và chọn Amazon Machine Image cho EC2 Instance</b>
</p>


#### Chọn Instance Type

Tại phần **Instance type**, tôi chọn:

```text
t3.micro
```

Cấu hình chính:

```text
vCPU:   2
Memory: 1 GiB
```

Như đã phân tích ở phần chi phí, Web Server của workshop chỉ chạy Apache và một số file HTML đơn giản nên không cần sử dụng Instance có tài nguyên lớn.

`t3.micro` đáp ứng đủ nhu cầu của bài lab đồng thời giúp hạn chế chi phí.

![Lựa chọn Amazon Linux 2023 và t3.micro](/images/5/5.3.1/003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.3.2: Lựa chọn Amazon Linux 2023 và Instance Type t3.micro</b>
</p>

---

#### Cấu hình Key Pair

Tại phần **Key pair**, tôi tạo hoặc lựa chọn Key Pair:

```text
backup-lab-key
```

Key Pair được sử dụng để kết nối SSH tới EC2.

File Private Key phải được lưu ở vị trí an toàn và **không được upload lên GitHub.**.

---

#### Cấu hình Network

Tại phần **Network settings**, tôi giữ EC2 trong VPC mặc định của tài khoản và bật:

```text
Auto-assign public IP = Enable
```

Public IPv4 được sử dụng để truy cập Web Server từ trình duyệt Internet.

##### Security Group

Tôi tạo Security Group:

```text
backup-lab-web-sg
```

với hai Inbound Rule:

| Type | Port | Source | Mục đích |
| --- | --- | --- | --- |
| SSH | 22 | My IP | Quản trị EC2 |
| HTTP | 80 | 0.0.0.0/0 | Cho phép truy cập website |

HTTP được phép từ Internet vì website của workshop cần được kiểm tra trực tiếp từ trình duyệt.

Ngược lại, SSH không mở:

```text
0.0.0.0/0
```

mà chỉ giới hạn theo địa chỉ IP quản trị nhằm giảm phạm vi truy cập không cần thiết.

Đây cũng là một biện pháp bảo mật cơ bản được áp dụng trong workshop.

![Cấu hình Security Group cho Web Server](/images/5/5.3.1/004.png?featherlight=false&width=70pc)
![Cấu hình Security Group cho Web Server](/images/5/5.3.1/005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.3.3: Cấu hình Security Group cho Web Server</b>
</p>

---

### Cấu hình Root Volume

Tại phần **Configure storage**, tôi sử dụng:

```text
Root Volume:
8 GiB
gp3
```

Dung lượng 8 GiB đủ để cài Amazon Linux, Apache và các thành phần cần thiết cho workshop.

Dữ liệu cần bảo vệ sẽ không chỉ lưu trên Root Volume. Một EBS Data Volume riêng sẽ được tạo ở phần **5.3.3** để mô phỏng rõ hơn quá trình Backup và Restore dữ liệu ứng dụng.

---

### Khởi tạo EC2

Sau khi kiểm tra toàn bộ cấu hình, tôi chọn:

```text
Launch instance
```

Sau khi EC2 được tạo, tôi chờ đến khi:

```text
Instance state:
Running

Status checks:
3/3 checks passed
```

![EC2 Web Server được khởi tạo thành công](/images/5/5.3.1/007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.3.4: EC2 Web Server được khởi tạo thành công</b>
</p>

### Kết quả mong đợi

```text
EC2 Instance = Running

Status checks = 3/3 passed

Public IPv4 = Available

Security Group = HTTP + SSH configured
```

EC2 hiện đã sẵn sàng để cài đặt Web Server.