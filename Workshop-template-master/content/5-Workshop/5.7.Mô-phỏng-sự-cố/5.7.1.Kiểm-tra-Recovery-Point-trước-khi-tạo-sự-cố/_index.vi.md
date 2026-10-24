---
title: "Kiểm tra Recovery Point trước khi tạo sự cố"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.7.1. </b> "
---

### Kiểm tra Recovery Point trước khi tạo sự cố

Trước khi thay đổi hoặc xóa bất kỳ dữ liệu nào, tôi kiểm tra lại trạng thái Backup để bảo đảm hệ thống đã có Recovery Point hợp lệ.

Việc kiểm tra này rất quan trọng vì Recovery Point sẽ được sử dụng làm nguồn dữ liệu để thực hiện Restore ở phần tiếp theo.

Tôi truy cập:

```text
AWS Backup
→ Backup vaults
→ BackupLab-Vault
```

Tại đây, tôi xác nhận Recovery Point của hai tài nguyên:

```text
Amazon EC2

Amazon S3
```

đã tồn tại trong Backup Vault.

---

#### Kiểm tra trạng thái Backup Job

Ngoài việc kiểm tra Recovery Point, tôi kiểm tra lại các Backup Job tương ứng.

Trạng thái cần đạt là:

```text
COMPLETED
```

Điều này chứng minh quá trình Backup đã hoàn thành thành công trước khi tôi bắt đầu mô phỏng sự cố.

Trình tự bắt buộc là:

```text
Backup thành công
       ↓
Recovery Point tồn tại
       ↓
Mới bắt đầu tạo sự cố
```

Nếu dữ liệu bị thay đổi hoặc xóa **trước khi có một Recovery Point hợp lệ**, tôi sẽ không có bản Backup phù hợp để sử dụng cho quá trình Restore.

![Recovery Point của EC2 và S3 tồn tại trước khi mô phỏng sự cố](/FCAJ_Worklog/images/5/5.7.1/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Recovery Point của EC2 và S3 tồn tại trước khi mô phỏng sự cố</b>
</p>

---

#### Không tạo Backup mới sau khi xảy ra sự cố

Sau khi bắt đầu thay đổi dữ liệu, tôi không thực hiện thêm một On-demand Backup mới.

Lý do là Recovery Point hiện tại đang chứa trạng thái dữ liệu:

```text
Version 1
```

Trong khi sau khi mô phỏng sự cố, dữ liệu sẽ trở thành:

```text
Version 2

Deleted

Corrupted
```

Nếu tôi tạo một Backup mới sau thời điểm này, Recovery Point mới có thể chứa chính trạng thái dữ liệu đã bị thay đổi hoặc bị lỗi.

Khi đó, việc kiểm thử Restore sẽ không còn thể hiện đúng mục tiêu ban đầu.

Trình tự cần được giữ nguyên:

```text
Version 1
   ↓
BACKUP
   ↓
Recovery Point
   ↓
INCIDENT
   ↓
RESTORE
```

Vì vậy, từ thời điểm bắt đầu mô phỏng sự cố, tôi chỉ sử dụng Recovery Point đã được tạo **trước sự cố** và không tạo thêm Backup mới cho đến khi hoàn thành quá trình Restore và Validation.