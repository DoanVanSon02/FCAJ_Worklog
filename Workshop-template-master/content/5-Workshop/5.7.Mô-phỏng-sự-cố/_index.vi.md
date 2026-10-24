---
title: "Mô phỏng sự cố"
date: 2026-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

### Tổng quan

Sau khi hoàn thành quá trình Backup và xác nhận Recovery Point của Amazon EC2/EBS và Amazon S3 đã được tạo thành công, tôi tiến hành mô phỏng một số tình huống mất hoặc thay đổi dữ liệu.

Mục tiêu của bước này không phải làm hỏng hệ thống một cách ngẫu nhiên mà là tạo ra **một sự cố có kiểm soát** để kiểm tra khả năng khôi phục của AWS Backup.

Trạng thái hệ thống trước khi xảy ra sự cố:

```text
Website
→ Hoạt động bình thường

/data/version.txt
→ AWS Backup Recovery Lab - Version 1

/data/customer.txt
→ Customer data before backup

/data/website-data/app-data.txt
→ Important website data

S3:
application-data/s3-version.txt
→ AWS Backup Recovery Lab - S3 Version 1
```

Các dữ liệu trên đã được lưu trong Recovery Point ở phần 5.5.

Trong phần này, tôi mô phỏng ba dạng sự cố chính:

```text
Thay đổi dữ liệu
       ↓
Xóa dữ liệu
       ↓
Làm sai nội dung dữ liệu
```

Ngoài ra, nội dung Website trên EC2 cũng được thay đổi để dễ dàng phân biệt trạng thái hệ thống trước và sau sự cố.

Luồng kiểm thử được thực hiện theo trình tự:

```text
Dữ liệu Version 1
        ↓
      Backup
        ↓
  Recovery Point
        ↓
 Mô phỏng sự cố
        ↓
Version 2 / Deleted / Corrupted
        ↓
      Restore
        ↓
Quay về trạng thái Version 1
```

Trước khi tạo sự cố, tôi kiểm tra lại Backup Vault để bảo đảm Recovery Point của EC2 và S3 đã tồn tại và các Backup Job có trạng thái:

```text
COMPLETED
```

Sau khi xác nhận Backup thành công, tôi bắt đầu thay đổi dữ liệu trên EBS.

Các tình huống được mô phỏng gồm:

```text
version.txt
→ Version 1 thành Version 2

customer.txt
→ Bị xóa

app-data.txt
→ Nội dung bị thay đổi

Website
→ Chuyển sang trạng thái INCIDENT TEST
```

Đối với Amazon S3, object:

```text
application-data/s3-version.txt
```

được xóa bằng thao tác Delete thông thường.

Do S3 Bucket đã bật Versioning, thao tác này tạo **Delete Marker** và khiến object không còn hiển thị trong danh sách hiện tại, trong khi phiên bản trước vẫn được giữ lại.

Sau khi hoàn thành mô phỏng, hệ thống sẽ có sự khác biệt rõ ràng giữa trạng thái được lưu trong Recovery Point và trạng thái hiện tại:

```text
RECOVERY POINT
────────────────────────────
Website       = Normal
version.txt   = Version 1
customer.txt  = Exists
app-data.txt  = Normal
s3-version    = Exists


CURRENT SYSTEM
────────────────────────────
Website       = INCIDENT TEST
version.txt   = Version 2
customer.txt  = Deleted
app-data.txt  = Corrupted
s3-version    = Deleted
```

Đây chính là điều kiện cần thiết để kiểm tra khả năng Restore trong phần tiếp theo.

Trong quá trình mô phỏng sự cố, tôi vẫn giữ nguyên các tài nguyên cần thiết cho quá trình khôi phục và không thực hiện các thao tác có thể làm mất khả năng Restore, như:

```text
✗ Terminate EC2 gốc

✗ Format EBS Volume

✗ Xóa toàn bộ S3 object versions

✗ Xóa Backup Vault

✗ Xóa Recovery Point

✗ Thay đổi IAM Role của AWS Backup

✗ Tạo thêm Backup sau khi xảy ra sự cố
```

Sau khi hoàn thành phần 5.7, hệ thống đã sẵn sàng để sử dụng Recovery Point được tạo **trước sự cố** nhằm khôi phục dữ liệu ở phần 5.8.

#### Nội dung

1. [Kiểm tra Recovery Point trước khi tạo sự cố](5.7.1.kiểm-tra-recovery-point-trước-khi-tạo-sự-cố/)
2. [Mô phỏng sự cố trên EC2 và EBS](5.7.2.mô-phỏng-sự-cố-trên-ec2-và-ebs/)
3. [Mô phỏng sự cố trên Amazon S3](5.7.3.mô-phỏng-sự-cố-trên-amazon-s3/)