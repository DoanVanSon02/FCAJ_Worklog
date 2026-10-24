---
title: "Chuẩn bị AWS Backup và tạo Backup Vault"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

## Chuẩn bị AWS Backup và tạo Backup Vault

Trước khi cấu hình Backup Plan, tôi kiểm tra các điều kiện cần thiết để AWS Backup có thể bảo vệ các tài nguyên của workshop.

Hai nhóm tài nguyên chính được sử dụng gồm:

```text
Amazon EC2
Amazon S3
```

Sau đó, tôi tạo một Backup Vault riêng để lưu trữ và quản lý các Recovery Point của bài lab.

---

### Kiểm tra các Resource Type sử dụng với AWS Backup

Tôi truy cập:

```text
AWS Backup
→ Settings
→ Service opt-in
```

và kiểm tra các Resource Type được sử dụng trong workshop:

```text
Amazon EC2 = Enabled
Amazon S3  = Enabled
```

AWS Backup cho phép quản lý những loại tài nguyên được sử dụng trong từng Region.

Trong workshop này, tôi sử dụng EC2 và S3 để thể hiện hai nhóm dữ liệu cần được bảo vệ:

```text
Amazon EC2
├── Root EBS
└── Data EBS

Amazon S3
└── application-data/
```

Việc kiểm tra Service opt-in giúp bảo đảm AWS Backup có thể quản lý các loại tài nguyên được sử dụng trong bài lab.

![Kiểm tra EC2 và S3](/images/5/5.5.1/0001.png?featherlight=false&width=70pc)
<p align="center">
  <b>Hình 5.5.1: Kiểm tra Amazon EC2 và Amazon S3 trong AWS Backup Service opt-in</b>
</p>

---

### Kiểm tra điều kiện của S3 trước khi Backup

Trước khi tiếp tục, tôi kiểm tra lại S3 Bucket tại:

```text
Amazon S3
→ Bucket
→ Properties
```

và xác nhận:

```text
Bucket Versioning = Enabled
```

Versioning là cấu hình quan trọng đối với quá trình bảo vệ dữ liệu S3 bằng AWS Backup.

Bucket đồng thời vẫn duy trì các cấu hình bảo mật đã thiết lập trước đó:

```text
Block Public Access = Enabled

ACLs = Disabled
```

Việc bật Versioning không làm thay đổi chính sách bảo mật của S3 Bucket.

![Kiểm tra Versioning](/images/5/5.5.1/0006.png?featherlight=false&width=70pc)
<p align="center">
  <b>Kiểm tra Versioning = enabled</b>
</p>
---

### Tạo Backup Vault

Sau khi kiểm tra các điều kiện cần thiết, tôi truy cập:

```text
AWS Backup
→ Vaults
→ Create new vault
```
![Tạo Backup Vault](/images/5/5.5.1/0002.png?featherlight=false&width=70pc)
<p align="center">
  <b>Tạo Backup Vault</b>
</p>

Tại phần Vault name, tôi sử dụng:

```text
BackupLab-Vault
```
Vault type:

```text
Backup vault
```
Encryption key:

```text
Default
```
![Tạo Backup Vault](/images/5/5.5.1/0003.png?featherlight=false&width=70pc)
<p align="center">
  <b>Tạo Backup Vault</b>
</p>

Các Tag được sử dụng:

```text
Project     = AWS-Backup-Recovery-Lab
Environment = Lab
```

Sau khi kiểm tra cấu hình, tôi chọn:

```text
Create backup vault
```
![Tạo Backup Vault](/images/5/5.5.1/0004.png?featherlight=false&width=70pc)
<p align="center">
  <b>Tạo Backup Vault</b>
</p>
---

### Tại sao cần Backup Vault?

Backup Vault được sử dụng để quản lý các Recovery Point được tạo bởi AWS Backup.

Luồng hoạt động có thể mô tả như sau:

```text
EC2 / S3
   ↓
AWS Backup
   ↓
BackupLab-Vault
   ↓
Recovery Points
```

**Recovery Point** đại diện cho trạng thái đã được sao lưu của một tài nguyên tại một thời điểm cụ thể.

Khi xảy ra sự cố, Recovery Point tương ứng sẽ được lựa chọn để thực hiện quá trình Restore.

Do đó, Backup Vault đóng vai trò là nơi AWS Backup quản lý các bản backup phục vụ cho quá trình khôi phục sau này.

---

### Tại sao sử dụng Vault riêng?

AWS Backup có thể có các Backup Vault mặc định.

Tuy nhiên, trong workshop tôi tạo riêng:

```text
BackupLab-Vault
```

để quản lý các Recovery Point của bài lab.

Việc sử dụng Vault riêng giúp:

```text
Tách Recovery Point của bài lab
        ↓
Dễ quản lý
        ↓
Dễ kiểm tra
        ↓
Dễ Cleanup
```

Cách này cũng giúp tránh nhầm lẫn giữa các Recovery Point của workshop với những bản backup khác có thể tồn tại trong cùng AWS Account.

---

### Tại sao không bật Vault Lock?

AWS Backup hỗ trợ cơ chế **Vault Lock** để tăng mức độ bảo vệ cho các Recovery Point.

Tuy nhiên, trong workshop này tôi **không bật Vault Lock**.

Lý do là môi trường hiện tại chỉ phục vụ mục đích học tập và kiểm thử theo quy trình:

```text
Tạo Recovery Point
      ↓
Restore
      ↓
Cleanup
```

Nếu sử dụng cơ chế khóa nghiêm ngặt trong một bài lab ngắn, việc xóa Recovery Point và dọn dẹp tài nguyên sau khi hoàn thành có thể trở nên phức tạp hơn.

Trong phạm vi workshop, việc bật Vault Lock không đóng góp trực tiếp vào mục tiêu chính là kiểm thử quá trình Backup và Restore.

Tương tự, tôi không sử dụng:

```text
Logically Air-Gapped Vault
```

vì bài lab chỉ thực hiện Backup & Recovery cơ bản trong cùng AWS Account và Region.

![Tạo Backup Vault](/images/5/5.5.1/0005.png?featherlight=false&width=70pc)
<p align="center">
  <b>Tạo Backup Vault</b>
</p>

<p align="center">
  <b>Hình 5.5.2: Tạo BackupLab-Vault để lưu trữ Recovery Point</b>
</p>

---

### Kết quả cần đạt

Sau khi hoàn thành phần này, Backup Vault cần có trạng thái:

```text
Backup Vault = BackupLab-Vault

Encryption = AWS Backup default KMS key

Vault Lock = Disabled

Recovery Points = 0
```

Tại thời điểm này:

```text
Recovery Points = 0
```

vì chưa có Backup Job nào được thực hiện.

`BackupLab-Vault` hiện đã sẵn sàng để được sử dụng trong Backup Plan ở phần tiếp theo.