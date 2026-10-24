---
title: "Tạo EBS Data Volume"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

### Tạo EBS Data Volume

Mặc dù EC2 đã có Root Volume, tôi tạo thêm một EBS Volume riêng để lưu dữ liệu ứng dụng.

Mục tiêu là tách:

```text
Hệ điều hành / Website
        ↓
   Root Volume

Dữ liệu ứng dụng
        ↓
   Data Volume
```

Việc tách Data Volume giúp quá trình kiểm thử Backup & Recovery rõ ràng hơn.

---

#### Kiểm tra Availability Zone của EC2

Trước khi tạo EBS, tôi kiểm tra EC2 tại:

```text
EC2
→ Instances
→ backup-lab-web-server
```

Sau đó ghi lại **Availability Zone** của EC2.

Ví dụ:

```text
us-east-1c
```

EBS chỉ có thể được attach trực tiếp vào EC2 nằm trong **cùng Availability Zone**.

---

#### Tạo EBS Volume

Tôi truy cập:

```text
EC2
→ Elastic Block Store
→ Volumes
→ Create volume
```

Cấu hình Volume:

```text
Volume type:
gp3

Size:
1 GiB

Availability Zone:
Cùng Availability Zone với EC2
```

Ví dụ:

```text
EC2:
us-east-1c

EBS:
us-east-1c
```
Tag được sử dụng:

```text
Name        = backup-lab-data-volume
```

#### Tại sao chọn 1 GiB gp3?

Dữ liệu được sử dụng trong workshop chỉ gồm một số file text nhỏ.

Do đó, 1 GiB đã đủ để kiểm thử quá trình


Việc tạo volume lớn hơn không mang lại lợi ích cho bài lab nhưng lại làm tăng chi phí lưu trữ.

![Tạo EBS](/FCAJ_Worklog/images/5/5.3.3/001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.3.7: Tạo EBS Data Volume 1 GiB gp3</b>
</p>

Sau khi tạo thành công, EBS Volume cần có các thông tin:

```text
Name = backup-lab-data-volume

Size = 1 GiB

Type = gp3

State = Available

Availability Zone = cùng AZ với EC2
```

---

#### Gắn EBS vào EC2

Tại danh sách EBS Volume, tôi chọn:

```text
backup-lab-data-volume
```

Sau đó chọn:

```text
Actions
→ Attach volume
```

Tại phần lựa chọn Instance, tôi chọn:

```text
backup-lab-web-server
```

và thực hiện attach.

Sau khi hoàn thành, trạng thái của EBS chuyển từ:

```text
Available
```

sang:

```text
In-use
```

Điều này cho biết Data Volume đã được gắn thành công vào EC2.

---

#### Kiểm tra EBS trên Linux

Sau khi attach EBS, tôi quay lại Terminal của EC2 và chạy:

```bash
lsblk
```

Ở lần triển khai, Data Volume có thể xuất hiện dưới tên như:

```text
xvdb
```

hoặc trên một số loại Instance:

```text
nvme1n1
```

Do đó, cần sử dụng kết quả thực tế của lệnh `lsblk` để xác định đúng thiết bị trước khi format.

Ví dụ:

```text
NAME    SIZE MOUNTPOINT
nvme0n1      8G /
nvme1n1      1G
```

Trong ví dụ trên:

```text
/dev/nvme0n1
```

là Root Volume và:

```text
/dev/nvme1n1
```

là Data Volume vừa được gắn vào EC2.

---

![Kiểm tra EBS trong linux](/FCAJ_Worklog/images/5/5.3.3/003.png?featherlight=false&width=70pc)

#### Tạo filesystem

Với Data Volume mới chưa có dữ liệu, tôi format bằng filesystem `ext4`.

Ví dụ thiết bị là:

```text
/dev/nvme1n1
```

tôi thực hiện:

```bash
sudo mkfs -t ext4 /dev/nvme1n1
```

> **Lưu ý:** `mkfs` chỉ được sử dụng khi volume mới chưa có filesystem. Không được chạy lệnh này trên một volume đã chứa dữ liệu cần khôi phục vì quá trình format sẽ làm mất dữ liệu hiện có.

---

#### Mount Data Volume

Tôi tạo mount point:

```bash
sudo mkdir -p /data
```

Sau đó mount EBS Volume vào thư mục `/data`:

```bash
sudo mount /dev/nvme1n1 /data
```

Kiểm tra:

```bash
df -h /data
```

Kết quả cần cho thấy EBS Volume được mount tại:

```text
/data
```

---
![Mount EBS](/FCAJ_Worklog/images/5/5.3.3/004.png?featherlight=false&width=70pc)

#### Cấu hình tự động mount sau khi reboot

Nếu chỉ sử dụng lệnh `mount`, EBS Volume có thể không được tự động mount lại sau khi EC2 reboot.

Do đó, tôi lấy UUID của Volume bằng lệnh:

```bash
sudo blkid /dev/nvme1n1
```

Ví dụ kết quả:

```text
UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

Sau đó mở file:

```bash
sudo nano /etc/fstab
```

và thêm dòng:

```text
UUID=<UUID-CUA-VOLUME> /data ext4 defaults,nofail 0 2
```

Sau khi lưu file, tôi kiểm tra cấu hình bằng:

```bash
sudo mount -a
```

Nếu không xuất hiện lỗi, tiếp tục kiểm tra:

```bash
df -h /data
```

Data Volume cần tiếp tục được hiển thị tại:

```text
/data
```

---

#### Tạo dữ liệu thử nghiệm

Sau khi EBS Data Volume đã được mount thành công, tôi tạo dữ liệu ban đầu trước khi thực hiện Backup.

Tạo file `version.txt`:

```bash
echo "AWS Backup Recovery Lab - Version 1" | sudo tee /data/version.txt
```

Tạo file dữ liệu khách hàng:

```bash
echo "Customer data before backup" | sudo tee /data/customer.txt
```

Tạo thư mục lưu dữ liệu website:

```bash
sudo mkdir -p /data/website-data
```

Tạo dữ liệu ứng dụng:

```bash
echo "Important website data" | sudo tee /data/website-data/app-data.txt
```

---

#### Kiểm tra dữ liệu

Tôi kiểm tra các file vừa tạo bằng các lệnh:

```bash
cat /data/version.txt
cat /data/customer.txt
cat /data/website-data/app-data.txt
```

Kết quả mong đợi:

```text
AWS Backup Recovery Lab - Version 1
Customer data before backup
Important website data
```

Các file này sẽ được sử dụng ở phần **Mô phỏng sự cố**.

Sau này:

```text
version.txt
```

sẽ bị thay đổi,

```text
customer.txt
```

sẽ bị xóa,

và:

```text
app-data.txt
```

sẽ được thay bằng dữ liệu không đúng.

Sau khi Restore, ba file này sẽ được kiểm tra lại để xác nhận Recovery Point đã khôi phục đúng trạng thái dữ liệu trước khi xảy ra sự cố.

---

#### Kiểm tra trạng thái EBS

Tôi sử dụng:

```bash
lsblk
df -h /data
```

để kiểm tra Data Volume và mount point.

![Kiểm tra Data Volume và mount](/FCAJ_Worklog/images/5/5.3.3/007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.3.8: EBS Data Volume được gắn và mount tại /data</b>
</p>

Sau đó kiểm tra lại dữ liệu:

```bash
cat /data/version.txt
cat /data/customer.txt
cat /data/website-data/app-data.txt
```
![Kiểm tra dữ liệu](/FCAJ_Worklog/images/5/5.3.3/006.png?featherlight=false&width=70pc)


Kết quả:

```text
AWS Backup Recovery Lab - Version 1
Customer data before backup
Important website data
```

<!-- Chèn hình dữ liệu thử nghiệm trên EBS tại đây -->

<p align="center">
  <b>Hình 5.3.9: Dữ liệu thử nghiệm được tạo trên EBS trước khi Backup</b>
</p>

---

#### Kết quả

Sau khi hoàn thành phần này, hệ thống cần đạt được trạng thái:

```text
EBS = In-use

Data Volume = 1 GiB gp3

Mount point = /data

version.txt = Version 1

customer.txt = tồn tại

app-data.txt = dữ liệu bình thường
```