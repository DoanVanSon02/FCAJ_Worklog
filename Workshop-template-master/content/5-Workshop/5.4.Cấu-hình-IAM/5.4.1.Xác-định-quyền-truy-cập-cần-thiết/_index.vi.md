---
title: "Xác định quyền truy cập cần thiết"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

### Xác định quyền truy cập cần thiết

Trước khi tạo IAM Policy, tôi xác định chính xác các thao tác mà EC2 cần thực hiện đối với Amazon S3.

Trong workshop, EC2 chỉ cần làm việc với dữ liệu trong:

```text
application-data/
```

của S3 Bucket đã tạo ở phần trước.

Các quyền cần thiết gồm:

| Permission | Mục đích |
| --- | --- |
| `s3:GetBucketLocation` | Xác định Region của S3 Bucket |
| `s3:ListBucket` | Liệt kê các object cần thiết trong Bucket |
| `s3:GetObject` | Đọc hoặc tải object từ S3 |
| `s3:PutObject` | Upload object từ EC2 lên S3 |

Tôi **không cấp**:

```text
s3:DeleteObject
```

vì EC2 không cần quyền xóa object để đáp ứng chức năng của bài lab.

Việc xác định quyền trước khi tạo Policy giúp tránh tình trạng cấp quyền vượt quá nhu cầu thực tế.

Ví dụ, một cách đơn giản hơn là gắn:

```text
AmazonS3FullAccess
```

cho EC2.

Tuy nhiên, Managed Policy này cung cấp nhiều quyền hơn mức cần thiết, bao gồm khả năng thao tác với nhiều tài nguyên và các hành động mà ứng dụng của workshop không sử dụng.

Trong kiến trúc hiện tại:

```text
EC2 chỉ cần
     ↓
một S3 Bucket cụ thể
     ↓
một prefix cụ thể
     ↓
một nhóm hành động cụ thể
```

Vì vậy tôi lựa chọn xây dựng **Custom IAM Policy** thay vì sử dụng `AmazonS3FullAccess`.

Đây là cách nguyên tắc **Least Privilege** được áp dụng vào hệ thống ngay từ giai đoạn thiết kế quyền truy cập.