---
title: "Mô phỏng sự cố trên Amazon S3"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.7.3. </b> "
---

## Mô phỏng sự cố trên Amazon S3

Sau khi mô phỏng sự cố đối với EC2 và EBS, tôi tiếp tục tạo một tình huống mất dữ liệu trên Amazon S3.

Object được sử dụng trong bài kiểm thử là:

```text
application-data/s3-version.txt
```

Trước khi xảy ra sự cố, object này chứa nội dung:

```text
AWS Backup Recovery Lab - S3 Version 1
```

Trạng thái này đã được AWS Backup lưu trong Recovery Point được tạo ở phần trước.

---

### Xóa object hiện tại

Tôi truy cập:

```text
Amazon S3
→ Bucket
→ application-data/
```
![Mô phỏng sự cố trên S3](/FCAJ_Worklog/images/5/5.7.3/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Mô phỏng sự cố trên S3</b>
</p>

Sau đó chọn object:

```text
s3-version.txt
```

và chọn:

```text
Delete
```

Tại bước xác nhận, tôi thực hiện thao tác xóa object thông thường.

Sau khi hoàn thành, `s3-version.txt` không còn xuất hiện trong danh sách object hiện tại của thư mục:

```text
application-data/
```
![Mô phỏng sự cố trên S3](/FCAJ_Worklog/images/5/5.7.3/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Mô phỏng sự cố trên S3</b>
</p>

![Mô phỏng sự cố trên S3](/FCAJ_Worklog/images/5/5.7.3/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Mô phỏng sự cố trên S3</b>
</p>

Trạng thái lúc này:

```text
Before Incident:
s3-version.txt = tồn tại

After Incident:
s3-version.txt = không còn hiển thị
```

---

### Tại sao chỉ xóa object thông thường?

S3 Bucket của workshop đã được bật:

```text
Versioning = Enabled
```

Vì vậy, khi thực hiện thao tác `DELETE` thông thường mà không chỉ định Version ID, Amazon S3 không xóa vĩnh viễn phiên bản trước đó của object.

Thay vào đó, S3 tạo một:

```text
Delete Marker
```

và đặt Delete Marker trở thành phiên bản hiện tại của object.

Có thể hình dung trạng thái trước khi xóa như sau:

```text
Trước khi Delete

s3-version.txt
└── Version 1
    Current version
```

Sau khi thực hiện Delete:

```text
Sau khi Delete

s3-version.txt
├── Delete Marker
│   Current version
│
└── Version 1
    Previous version
```

Do Delete Marker trở thành phiên bản hiện tại nên trong danh sách object thông thường:

```text
s3-version.txt
```

sẽ có vẻ như đã bị xóa.

Một yêu cầu đọc object thông thường cũng sẽ hoạt động như thể object không còn tồn tại.

![Mô phỏng sự cố trên S3](/FCAJ_Worklog/images/5/5.7.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Mô phỏng sự cố trên S3</b>
</p>
---

### Không xóa vĩnh viễn các Version

Trong bước mô phỏng sự cố, tôi **không bật Show versions rồi xóa toàn bộ các Version ID** của:

```text
s3-version.txt
```

Lý do là việc xóa trực tiếp một Version ID cụ thể có thể xóa vĩnh viễn phiên bản đó.

Mục tiêu của workshop chỉ là mô phỏng tình huống:

```text
Người dùng xóa nhầm object
```

chứ không phải phá hủy toàn bộ lịch sử Versioning của S3.

Do đó, tôi chỉ thực hiện:

```text
Simple Delete
       ↓
Delete Marker
       ↓
Object không còn hiển thị
```

Cách này giúp tạo ra tình huống mất dữ liệu có kiểm soát nhưng vẫn giữ nguyên các thành phần cần thiết cho quá trình kiểm thử Restore.

---

### Kiểm tra trạng thái sau khi xóa

Sau khi thực hiện thao tác Delete, tôi quay lại:

```text
Amazon S3
→ Bucket
→ application-data/
```

và xác nhận:

```text
s3-version.txt
```

không còn xuất hiện trong danh sách object thông thường.

Trạng thái hiện tại có thể biểu diễn:

```text
application-data/
└── s3-version.txt
    → Không còn hiển thị
```
![Mô phỏng sự cố trên S3](/FCAJ_Worklog/images/5/5.7.3/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.7.4: s3-version.txt không còn xuất hiện sau khi mô phỏng thao tác xóa</b>
</p>

---

### Kiểm tra Delete Marker bằng Show versions

Do S3 Bucket đã bật Versioning, tôi có thể bật:

```text
Show versions
```

để kiểm tra lịch sử Version của object.

Khi đó, có thể quan sát:

```text
s3-version.txt
├── Delete Marker
│   → Current version
│
└── Version 1
    → Previous version
```

Điều này cho thấy object không bị xóa vĩnh viễn khỏi toàn bộ lịch sử Versioning.
![Mô phỏng sự cố trên S3](/FCAJ_Worklog/images/5/5.7.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.7.5: Delete Marker được tạo sau khi xóa object trong S3 Bucket đã bật Versioning</b>
</p>

S3 Versioning cung cấp thêm một lớp bảo vệ trước thao tác xóa nhầm.

Tuy nhiên, mục tiêu chính của workshop vẫn là sử dụng:

```text
AWS Backup Recovery Point
```

để thực hiện quá trình khôi phục dữ liệu theo kiến trúc đã xây dựng.

Có thể phân biệt hai cơ chế như sau:

```text
S3 Versioning
→ Lưu các phiên bản của object trong S3

AWS Backup
→ Tạo và quản lý Recovery Point
→ Phục vụ quá trình Backup & Restore
```

Trong workshop, hai cơ chế này được sử dụng kết hợp với nhau.

Sau bước này, trạng thái của dữ liệu S3 đã khác với trạng thái được lưu trong Recovery Point:

```text
Recovery Point:
s3-version.txt = tồn tại

Current S3:
s3-version.txt = không còn hiển thị
```

Đây là trạng thái cần thiết để kiểm tra khả năng khôi phục Amazon S3 ở phần Restore sau này.