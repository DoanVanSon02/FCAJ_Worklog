---
title: "Lựa chọn Region"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.2.2. </b> "
---

## Lựa chọn Region

Các tài nguyên chính trong workshop được triển khai tại:

```text
US East (N. Virginia)
Region: us-east-1
```

Tôi lựa chọn **US East (N. Virginia) – `us-east-1`** và duy trì Region này xuyên suốt quá trình thực hiện workshop.

Việc sử dụng thống nhất một Region giúp các tài nguyên được triển khai tập trung, dễ quản lý và hạn chế trường hợp không tìm thấy tài nguyên do chuyển nhầm Region trong quá trình cấu hình.

Trước khi thực hiện mỗi phần của workshop, cần kiểm tra Region đang được lựa chọn ở góc trên bên phải của **AWS Management Console** và bảo đảm Region hiện tại là:

```text
US East (N. Virginia)
us-east-1
```

![Lựa chọn Region US East (N. Virginia)](/FCAJ_Worklog/images/5/5.2.2.png?featherlight=false&width=70pc)

<p align="center">
  <b>Hình 5.2.1: Lựa chọn Region US East (N. Virginia) – us-east-1</b>
</p>

Riêng **AWS Identity and Access Management (IAM)** là dịch vụ **Global**, do đó khi truy cập giao diện IAM, khu vực hiển thị có thể là:

```text
Global
```

Điều này không ảnh hưởng đến việc các tài nguyên khác trong workshop được triển khai tại Region `us-east-1`.

Việc xác định Region ngay từ đầu giúp quá trình triển khai các tài nguyên như **Amazon EC2, Amazon EBS, Amazon S3, AWS Backup và Amazon CloudWatch** được thực hiện thống nhất trong cùng một khu vực, thuận tiện hơn cho việc quản lý, kiểm thử và dọn dẹp tài nguyên sau khi hoàn thành workshop.