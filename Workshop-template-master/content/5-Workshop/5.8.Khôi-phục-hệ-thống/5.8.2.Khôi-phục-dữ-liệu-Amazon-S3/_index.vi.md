---
title: "Khôi phục dữ liệu Amazon S3"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.8.2. </b> "
---


Sau khi quá trình Restore Amazon EC2 hoàn thành, tôi tiếp tục khôi phục dữ liệu đã bị xóa trên Amazon S3.

Trong phần mô phỏng sự cố, object:

```text
application-data/s3-version.txt
```

đã bị xóa khỏi danh sách object hiện tại.

Trước khi xảy ra sự cố, object này chứa:

```text
AWS Backup Recovery Lab - S3 Version 1
```

Do S3 Versioning được bật, thao tác Delete thông thường đã tạo một Delete Marker khiến object không còn xuất hiện trong danh sách hiện tại.

Trong bước này, tôi sử dụng **AWS Backup Recovery Point** được tạo trước sự cố để khôi phục object.

---

### Lựa chọn S3 Recovery Point

Tôi truy cập:

```text
AWS Backup
→ Protected resources
```

Sau đó chọn Amazon S3 Bucket của workshop.

Trong danh sách Recovery Point, tôi lựa chọn bản Backup được tạo **trước khi xảy ra sự cố**.

Sau đó chọn:

```text
Restore
```
![Khôi phục S3](/images/5/5.8.2/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Khôi phục S3</b>
</p>


Điều quan trọng là Recovery Point được lựa chọn phải chứa trạng thái:

```text
application-data/s3-version.txt
→ AWS Backup Recovery Lab - S3 Version 1
```

---

### Sử dụng Item-level Restore

AWS Backup cho phép khôi phục:

```text
Toàn bộ S3 Bucket
```

hoặc:

```text
Một hoặc nhiều object cụ thể
```

Trong workshop, chỉ có object:

```text
application-data/s3-version.txt
```

bị xóa.

Do đó, tôi sử dụng:

```text
Item-level restore
```

thay vì Restore toàn bộ Bucket.

---

### Tại sao sử dụng Item-level Restore?

Nếu Restore toàn bộ Bucket, AWS Backup phải xử lý nhiều dữ liệu hơn mức cần thiết.

Trong khi sự cố chỉ ảnh hưởng tới:

```text
application-data/s3-version.txt
```

Do đó:

```text
Sự cố ở một object
        ↓
Restore một object
```

phù hợp hơn với phạm vi của bài kiểm thử.

Cách này cũng giúp quá trình Restore dễ kiểm tra hơn vì chỉ khôi phục đúng dữ liệu đã bị mất.

---

### Chọn object cần Restore

Tại phần Restore settings, tôi chọn:

```text
Item level restore
```

Sau đó nhập S3 URI của object cần khôi phục:

```text
s3:s3://aws-backup-lab-s3-162918102026/application-data/s3-version.txt
```


Việc nhập chính xác S3 URI giúp AWS Backup xác định đúng object cần được khôi phục.

![Cấu hình Item-level Restore cho s3-version.txt](/images/5/5.8.2/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Cấu hình Item-level Restore cho s3-version.txt</b>
</p>

---

### Lựa chọn Version

AWS Backup có thể khôi phục phiên bản của object được lưu trong Recovery Point.

Trong workshop, tôi chỉ cần khôi phục trạng thái object trước sự cố nên sử dụng:

```text
Latest version
```

trong Recovery Point.

Mục tiêu là đưa object quay trở lại trạng thái:

```text
AWS Backup Recovery Lab - S3 Version 1
```
---
### Restore Destination

Tại phần Destination, tôi chọn:

```text
Restore to source bucket
```

Tôi **không tạo một S3 Bucket mới**.

Luồng Restore được thực hiện như sau:

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

Trong workshop, việc Restore về chính Source Bucket phù hợp nhất vì mục tiêu là phục hồi object đã bị xóa khỏi ứng dụng hiện tại.

---

### Vì sao object có thể được Restore khi đã có Delete Marker?

AWS Backup thực hiện S3 Restore theo cơ chế không phá hủy dữ liệu hiện có.

Trong tình huống của workshop:

```text
Version 1
   ↓
Delete
   ↓
Delete Marker
   ↓
Object không còn hiển thị
   ↓
AWS Backup Restore
   ↓
Object xuất hiện trở lại
```

Sau khi thao tác Delete thông thường được thực hiện, Delete Marker trở thành Current Version.

Object do đó không còn xuất hiện trong danh sách thông thường.

Tuy nhiên, trạng thái trước sự cố đã được lưu trong AWS Backup Recovery Point.

Khi thực hiện Restore, AWS Backup có thể khôi phục lại object từ trạng thái đã được Backup.

Điều này cũng giải thích vì sao trong phần 5.7 tôi chỉ thực hiện:

```text
Simple Delete
```

thay vì xóa vĩnh viễn toàn bộ Version ID của object.

---

### Cấu hình Encryption

Tại mục:

```text
Restored object encryption
```

tôi chọn:

```text
Use original encryption keys (default)
```

Cấu hình này cho phép AWS Backup sử dụng lại phương thức hoặc Encryption Key của object nguồn khi có thể.

Trong workshop không có yêu cầu thay đổi phương thức mã hóa nên tôi giữ cấu hình mặc định.

![Khôi phục S3](/images/5/5.8.2/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Khôi phục S3</b>
</p>
---

### Cấu hình ACL

S3 Bucket ban đầu đã được cấu hình:

```text
ACLs disabled

Bucket owner enforced
```

Ngoài ra, trong quá trình Backup tôi cũng không bật Backup ACL.

Do đó trong quá trình Restore, tôi giữ:

```text
Restore ACLs = Disabled
```

Việc này giúp cấu hình Restore phù hợp với thiết lập ban đầu của S3 Bucket.

---

### Restore Role

Tại mục:

```text
Restore role
```

tôi lựa chọn:

```text
AWSBackupDefaultServiceRole
```

Đây là Service Role được AWS Backup sử dụng để thực hiện các thao tác Restore.

Role này cần các quyền tương ứng để AWS Backup có thể khôi phục dữ liệu Amazon S3.

Các Managed Policy liên quan gồm:

```text
AWSBackupServiceRolePolicyForRestores

AWSBackupServiceRolePolicyForS3Restore
```
![Khôi phục S3](/images/5/5.8.2/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Khôi phục S3</b>
</p>
---

### Theo dõi S3 Restore Job

Sau khi bắt đầu Restore, tôi truy cập:

```text
AWS Backup
→ Jobs
→ Restore jobs
```

Tại đây, tôi theo dõi Restore Job của Amazon S3.

Thông tin cần kiểm tra:

```text
Resource type:
S3
```

và trạng thái cuối cùng:

```text
Status:
COMPLETED
```

Tôi chỉ tiếp tục kiểm tra dữ liệu sau khi:

```text
Status = COMPLETED
```

![Amazon S3 Restore Job hoàn thành với trạng thái COMPLETED](/images/5/5.8.2/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Amazon S3 Restore Job hoàn thành với trạng thái COMPLETED</b>
</p>

Trạng thái `COMPLETED` cho biết AWS Backup đã hoàn thành quá trình Restore object từ Recovery Point.

Tuy nhiên, đây chưa phải là bằng chứng duy nhất cho thấy dữ liệu đã được khôi phục chính xác.

Trong phần **5.8.3 – Kiểm tra dữ liệu sau khi khôi phục**, tôi sẽ kiểm tra trực tiếp:

```text
application-data/s3-version.txt
```

để xác nhận object đã xuất hiện trở lại và có đúng nội dung:

```text
AWS Backup Recovery Lab - S3 Version 1
```