---
title: "Công cụ sử dụng"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.2.3. </b> "
---

### Công cụ sử dụng

Workshop chủ yếu được thực hiện thông qua **AWS Management Console** để có thể quan sát trực tiếp các tài nguyên và thuận tiện cho việc thực hiện từng bước triển khai.

Các công cụ được sử dụng trong workshop gồm:

| Công cụ | Mục đích |
| --- | --- |
| **AWS Management Console** | Tạo và quản lý các dịch vụ AWS |
| **EC2 Instance Connect** | Truy cập terminal của EC2 |
| **Linux Shell** | Cài đặt Apache, tạo và kiểm tra dữ liệu |
| **AWS CLI trên EC2** | Kiểm tra khả năng EC2 truy cập Amazon S3 |
| **Trình duyệt Web** | Kiểm tra Web Server thông qua giao thức HTTP |
| **draw.io** | Xây dựng sơ đồ kiến trúc hệ thống |

#### AWS Management Console

**AWS Management Console** là công cụ chính được sử dụng xuyên suốt workshop.

Thông qua giao diện quản lý của AWS, tôi thực hiện các thao tác như:

* Khởi tạo và quản lý EC2 Instance.
* Tạo và cấu hình EBS Volume.
* Tạo và quản lý S3 Bucket.
* Tạo IAM Role và IAM Policy.
* Cấu hình Backup Vault và Backup Plan.
* Theo dõi Backup Job.
* Theo dõi metric và cấu hình Alarm trên CloudWatch.

Việc sử dụng AWS Management Console giúp quan sát trực tiếp trạng thái của từng tài nguyên và dễ dàng kiểm tra kết quả sau mỗi bước cấu hình.

#### EC2 Instance Connect

**EC2 Instance Connect** được sử dụng để truy cập terminal của EC2 Instance trực tiếp từ AWS Management Console.

Thông qua terminal của EC2, tôi có thể thực hiện các thao tác cần thiết trên Web Server mà không cần lưu Private Key trực tiếp trên máy tính trong quá trình thực hiện bài lab.

#### Linux Shell

Sau khi kết nối tới EC2, **Linux Shell** được sử dụng để thực hiện các thao tác trên máy chủ như:

* Cài đặt Apache HTTP Server.
* Khởi động và kiểm tra trạng thái Web Server.
* Tạo thư mục và file dữ liệu.
* Mount EBS Data Volume.
* Kiểm tra dữ liệu trước và sau khi thực hiện Restore.

#### AWS CLI trên EC2

**AWS CLI** được sử dụng trên EC2 để kiểm tra khả năng truy cập Amazon S3 thông qua IAM Role được gắn vào EC2.

AWS CLI hỗ trợ kiểm tra các thao tác như:

* Liệt kê object trong S3 Bucket.
* Tải file từ EC2 lên Amazon S3.
* Đọc dữ liệu từ Amazon S3.
* Xác minh quyền truy cập S3 của EC2.

Việc sử dụng AWS CLI trong trường hợp này giúp kiểm tra trực tiếp rằng EC2 có thể truy cập S3 thông qua **IAM Role** mà không cần lưu Access Key và Secret Access Key trên máy chủ.

#### Trình duyệt Web

**Trình duyệt Web** được sử dụng để kiểm tra khả năng truy cập Web Server sau khi Apache được cài đặt trên EC2.

Thông qua Public IPv4 Address hoặc Public DNS của EC2, có thể kiểm tra xem Web Server có hoạt động và trả về nội dung HTTP như mong đợi hay không.

#### draw.io

**draw.io** được sử dụng để xây dựng sơ đồ kiến trúc tổng quan của hệ thống.

Sơ đồ giúp thể hiện mối quan hệ giữa các thành phần chính như:

* Amazon EC2.
* Amazon EBS.
* Amazon S3.
* AWS IAM.
* AWS Backup.
* Backup Vault.
* Amazon CloudWatch.


Việc sử dụng các công cụ trên giúp tôi có thể trực tiếp quan sát quá trình tạo, cấu hình, sao lưu, giám sát và khôi phục tài nguyên trong toàn bộ workshop.