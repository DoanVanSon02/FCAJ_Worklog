---
title: "Bản đề xuất"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## 2.1. Tóm tắt

Đề tài của tôi là **“Xây dựng hệ thống Backup & Recovery cho Web Server trên AWS”**.

Mục tiêu của project là xây dựng một Web Server trên AWS, thực hiện sao lưu dữ liệu, giám sát quá trình Backup và kiểm thử khả năng khôi phục khi dữ liệu bị thay đổi hoặc xóa.

Hệ thống sử dụng các dịch vụ chính gồm:

- **Amazon EC2** để triển khai Web Server.
- **Amazon EBS** để lưu dữ liệu của máy chủ.
- **Amazon S3** để lưu dữ liệu dạng object.
- **AWS IAM** để quản lý quyền truy cập giữa các dịch vụ.
- **AWS Backup** để quản lý Backup và Restore tập trung.
- **Amazon CloudWatch** để theo dõi hoạt động Backup.

Quy trình chính của project:

```text
Xây dựng hệ thống
        ↓
Backup dữ liệu
        ↓
Monitoring
        ↓
Mô phỏng sự cố
        ↓
Restore
        ↓
Kiểm tra kết quả
```

---

## 2.2. Vấn đề cần giải quyết

Trong quá trình vận hành Web Server, dữ liệu có thể bị thay đổi hoặc mất do thao tác nhầm, lỗi ứng dụng hoặc sự cố hệ thống.

Nếu không có bản Backup phù hợp, việc khôi phục dữ liệu có thể mất nhiều thời gian hoặc không thể thực hiện.

Ngoài ra, dữ liệu của một hệ thống có thể tồn tại trên nhiều loại lưu trữ khác nhau như:

```text
Amazon EBS
Amazon S3
```

Do đó, project cần một giải pháp cho phép quản lý Backup tập trung và có khả năng Restore dữ liệu khi xảy ra sự cố.

Bên cạnh việc tạo Backup, hệ thống cũng cần có cơ chế Monitoring để theo dõi trạng thái của các Backup Job.

---

## 2.3. Giải pháp đề xuất

Tôi đề xuất xây dựng một Web Server Apache trên **Amazon EC2**.

EC2 sử dụng **Amazon EBS** để lưu dữ liệu của máy chủ và **Amazon S3** để lưu một số dữ liệu dạng object.

AWS IAM được sử dụng để quản lý quyền truy cập. EC2 truy cập S3 thông qua **IAM Role** thay vì lưu Access Key trực tiếp trên máy chủ, đồng thời áp dụng nguyên tắc **Least Privilege**.

**AWS Backup** được sử dụng để thực hiện Backup cho EC2/EBS và S3. Các Recovery Point được quản lý trong Backup Vault và được sử dụng để Restore khi xảy ra sự cố.

**Amazon CloudWatch** được sử dụng để theo dõi các metric của AWS Backup và tạo Alarm phục vụ việc giám sát.

Sau khi Backup hoàn thành, tôi sẽ chủ động mô phỏng một số sự cố như:

- Thay đổi nội dung file.
- Xóa file trên EBS.
- Làm sai dữ liệu ứng dụng.
- Thay đổi nội dung Website.
- Xóa object trên S3.

Sau đó, Recovery Point được tạo trước sự cố sẽ được sử dụng để khôi phục hệ thống.

Kiến trúc tổng quát:

```text
EC2 + EBS ──────┐
                │
                ▼
            AWS Backup
                │
S3 ─────────────┘
                │
                ▼
          Backup Vault
                │
                ▼
        Recovery Points
                │
                ▼
             Restore


AWS Backup
    │
    ▼
CloudWatch
    │
    ▼
  Alarm
```

---

## 2.4. Kết quả dự kiến

Sau khi hoàn thành project, tôi dự kiến đạt được các kết quả sau:

- Web Server hoạt động trên Amazon EC2.
- Dữ liệu được lưu trên EBS và Amazon S3.
- EC2 truy cập S3 bằng IAM Role và không hard-code Access Key.
- AWS Backup tạo được Recovery Point cho EC2/EBS và S3.
- CloudWatch có thể theo dõi hoạt động Backup.
- Có thể mô phỏng tình huống dữ liệu bị thay đổi hoặc xóa.
- EC2/EBS có thể được Restore từ Recovery Point.
- Object trên S3 có thể được khôi phục.
- Dữ liệu sau Restore trở lại trạng thái trước sự cố.

Kết quả cuối cùng cần chứng minh được quy trình:

```text
BACKUP
   ↓
RECOVERY POINT
   ↓
INCIDENT
   ↓
RESTORE
   ↓
VALIDATION
   ↓
PASS
```

Phần triển khai chi tiết, cấu hình từng dịch vụ, hình ảnh minh họa và kết quả kiểm thử sẽ được trình bày tại **Mục 5 – Workshop**.