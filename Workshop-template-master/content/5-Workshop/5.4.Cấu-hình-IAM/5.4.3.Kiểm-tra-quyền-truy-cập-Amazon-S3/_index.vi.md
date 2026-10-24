---
title: "Kiểm tra quyền truy cập Amazon S3"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

## Kiểm tra quyền truy cập Amazon S3

Sau khi IAM Role được gắn vào EC2, tôi thực hiện kiểm tra trực tiếp trên máy chủ.

Mục tiêu của bước kiểm tra không chỉ là xác nhận những quyền **được phép** hoạt động, mà còn xác nhận các quyền **không được cấp** thực sự bị từ chối.

Quy trình kiểm tra:

```text
Xác nhận IAM Role
        ↓
List object
        ↓
Download object
        ↓
Upload object
        ↓
Thử Delete object
        ↓
AccessDenied
```

---

### Xác nhận EC2 đã nhận IAM Role

Trước tiên, tôi kiểm tra AWS CLI:

```bash
aws --version
```
![Kiểm tra version AWS CLI](/FCAJ_Worklog/images/5/5.4.3/0001.png?featherlight=false&width=70pc)

Amazon Linux 2023 thường đã có AWS CLI. Nếu chưa có, cần cài đặt AWS CLI trước khi tiếp tục
([Xem hướng dẫn](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)).

Sau đó chạy:

```bash
aws sts get-caller-identity
```

Kết quả có dạng:

```json
{
    "UserId": "...",
    "Account": "...",
    "Arn": "arn:aws:sts::<ACCOUNT-ID>:assumed-role/BackupLab-EC2-S3-Role/..."
}
```

Phần quan trọng cần kiểm tra là:

```text
assumed-role/BackupLab-EC2-S3-Role
```

Điều này chứng minh EC2 đang sử dụng IAM Role vừa tạo.

![Kiểm tra version AWS CLI](/FCAJ_Worklog/images/5/5.4.3/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.4.5: EC2 sử dụng BackupLab-EC2-S3-Role</b>
</p>

---

### Kiểm tra quyền liệt kê dữ liệu trên S3

Tôi thực hiện:

```bash
aws s3 ls s3://aws-backup-lab-s3-162918102026/application-data/
```

Kết quả cần xuất hiện:

```text
s3-version.txt
```

Điều này chứng minh luồng:

```text
EC2
 ↓
IAM Role
 ↓
s3:ListBucket
 ↓
Amazon S3
```

đang hoạt động đúng.

---

### Kiểm tra quyền đọc dữ liệu từ S3

Tiếp theo, tôi tải:

```text
s3-version.txt
```

từ S3 về EC2:

```bash
aws s3 cp s3://aws-backup-lab-s3-162918102026/application-data/s3-version.txt /tmp/s3-version.txt
```

Sau đó kiểm tra nội dung:

```bash
cat /tmp/s3-version.txt
```

Kết quả:

```text
AWS Backup Recovery Lab - S3 Version 1
```

Điều này chứng minh quyền:

```text
s3:GetObject
```

đang hoạt động đúng.

---

### Kiểm tra quyền upload từ EC2 lên S3

Để kiểm tra quyền ghi object, tôi sử dụng file đã tạo trên EBS:

```text
/data/version.txt
```

và upload lên S3:

```bash
aws s3 cp /data/version.txt s3://aws-backup-lab-s3-162918102026/application-data/ec2-version.txt
```

Sau đó kiểm tra lại:

```bash
aws s3 ls s3://aws-backup-lab-s3-162918102026/application-data/
```

Kết quả cần có:

```text
ec2-version.txt
s3-version.txt
```

Điều này chứng minh quyền:

```text
s3:PutObject
```

hoạt động đúng.

Có thể kiểm tra thêm trong AWS Management Console tại:

```text
Amazon S3
→ Bucket
→ application-data/
```

để xác nhận:

```text
ec2-version.txt
```

đã được tạo.

![Kiểm tra EC2 liệt kê, đọc và upload dữ liệu lên S3 thông qua IAM Role](/FCAJ_Worklog/images/5/5.4.3/0003.png?featherlight=false&width=70pc)
<p align="center">
  <b>Hình 5.4.6: EC2 đọc và upload dữ liệu lên Amazon S3 thông qua IAM Role</b>
</p>

---

### Kiểm thử nguyên tắc Least Privilege

Chỉ chứng minh những quyền được cho phép hoạt động là chưa đủ để xác nhận Policy thực sự được giới hạn.

Do đó, tôi thực hiện thêm một bài kiểm thử với quyền **không được cấp**.

Trong Policy, EC2 không được cấp:

```text
s3:DeleteObject
```

Tôi thử xóa object vừa được upload:

```bash
aws s3 rm s3://aws-backup-lab-s3-162918102026/application-data/ec2-version.txt
```

Kết quả mong đợi:

```text
AccessDenied
```

hoặc một thông báo cho biết Role không được phép thực hiện:

```text
s3:DeleteObject
```

Trong trường hợp này, `AccessDenied` **không phải lỗi cấu hình của workshop**, mà là kết quả mong muốn.

Kết quả kiểm thử có thể tổng hợp như sau:

```text
GetBucketLocation → Allowed  ✓

ListBucket        → Allowed  ✓

GetObject         → Allowed  ✓

PutObject         → Allowed  ✓

DeleteObject      → Denied   ✓
```

Điều này chứng minh IAM Policy không cấp quyền vượt quá chức năng mà EC2 thực sự cần.

![Kiểm tra DeleteObject](/FCAJ_Worklog/images/5/5.4.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.4.7: Kiểm thử Least Privilege – EC2 không có quyền xóa object trên S3</b>
</p>

---

> **Chú ý – Không sử dụng Access Key trên EC2:** Trong workshop này, tôi không chạy `aws configure` để nhập và lưu `AWS Access Key ID` hoặc `AWS Secret Access Key` trên EC2. Thay vào đó, EC2 sử dụng `BackupLab-EC2-S3-Role` để nhận temporary credentials. Cách này giúp tránh lưu thông tin xác thực dài hạn trên máy chủ, không yêu cầu quản lý Access Key thủ công và cho phép kiểm soát quyền tập trung thông qua IAM Policy.

---

