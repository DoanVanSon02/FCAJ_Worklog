---
title: "Lựa chọn các dịch vụ AWS"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.2.4. </b> "
---

### Lựa chọn các dịch vụ AWS

Trước khi bắt đầu triển khai hệ thống, tôi xác định các dịch vụ AWS phù hợp với mục tiêu của workshop.

Hệ thống **Backup & Recovery cho Web Server trên AWS** sử dụng các dịch vụ chính gồm:

* Amazon EC2
* Amazon EBS
* Amazon S3
* AWS Identity and Access Management (IAM)
* AWS Backup
* Amazon CloudWatch

Mỗi dịch vụ đảm nhiệm một vai trò riêng trong quá trình xây dựng, lưu trữ dữ liệu, sao lưu, khôi phục và giám sát hệ thống.

---

#### Amazon EC2

**Amazon Elastic Compute Cloud (Amazon EC2)** được sử dụng để triển khai Web Server chạy Apache.

Tôi lựa chọn Amazon EC2 vì workshop cần một máy chủ thực tế có thể:

* Cài đặt Web Server.
* Tạo dữ liệu thử nghiệm.
* Gắn thêm EBS Volume.
* Truy cập Amazon S3.
* Mô phỏng tình huống dữ liệu bị thay đổi hoặc xảy ra sự cố.
* Thực hiện Backup và Restore bằng AWS Backup.

EC2 phù hợp với bài lab vì cho phép quan sát rõ trạng thái của Web Server và dữ liệu **trước và sau khi thực hiện khôi phục**.

Trong kiến trúc của workshop, EC2 đóng vai trò là thành phần xử lý chính và chạy ứng dụng Web.

---

#### Amazon EBS

**Amazon Elastic Block Store (Amazon EBS)** được sử dụng làm Data Volume gắn với EC2.

Thay vì chỉ lưu toàn bộ dữ liệu trên Root Volume của EC2, tôi sử dụng thêm một EBS Volume riêng và mount tại:

```text
/data
```

EBS Volume này được sử dụng để lưu dữ liệu cần bảo vệ trong quá trình thực hiện workshop.

Việc tách riêng Data Volume giúp mô phỏng rõ hơn dữ liệu ứng dụng cần được sao lưu. Khi thực hiện Restore EC2, có thể kiểm tra trực tiếp xem EBS Volume và các file dữ liệu có được khôi phục về trạng thái trước khi xảy ra sự cố hay không.

---

#### Amazon S3

**Amazon Simple Storage Service (Amazon S3)** được lựa chọn để lưu trữ dữ liệu dạng object của ứng dụng.

Lý do lựa chọn Amazon S3 gồm:

* Không cần tự triển khai máy chủ lưu trữ.
* Là dịch vụ lưu trữ được AWS quản lý.
* Có khả năng mở rộng theo lượng dữ liệu.
* Hỗ trợ **Versioning**.
* Được **AWS Backup hỗ trợ**.
* Phù hợp với việc lưu trữ file và object của ứng dụng.

Trong workshop, tính năng **Versioning** được bật cho S3 Bucket để hỗ trợ quản lý các phiên bản của object và đáp ứng yêu cầu khi sử dụng AWS Backup cho Amazon S3.

S3 Bucket được giữ ở trạng thái **private** và bật:

```text
Block Public Access = Enabled
```

Người dùng Internet không truy cập trực tiếp vào S3 Bucket.

EC2 sẽ truy cập dữ liệu trong S3 thông qua IAM Role thay vì sử dụng Access Key được lưu trực tiếp trên máy chủ.

---

#### AWS Identity and Access Management (IAM)

**AWS Identity and Access Management (IAM)** được sử dụng để quản lý quyền truy cập giữa các thành phần trong hệ thống.

Workshop sử dụng hai IAM Role với hai mục đích khác nhau.

IAM Role:

```text
BackupLab-EC2-S3-Role
```

được gắn vào EC2 để Web Server có thể truy cập Amazon S3 theo các quyền được cho phép.

Đối với quyền EC2 truy cập S3, tôi sử dụng custom IAM Policy:

```text
BackupLab-EC2-S3-Policy
```

Policy này chỉ cung cấp những quyền Amazon S3 cần thiết cho bài lab thay vì cấp toàn bộ quyền quản trị S3.

IAM Role thứ hai là:

```text
AWSBackupDefaultServiceRole
```

Role này được AWS Backup sử dụng để thực hiện các hoạt động **Backup** và **Restore** đối với các tài nguyên được bảo vệ.

Việc sử dụng IAM Role giúp hạn chế việc lưu Access Key trực tiếp trên EC2 và hỗ trợ áp dụng nguyên tắc **Least Privilege** trong hệ thống.

---

#### AWS Backup

**AWS Backup** là thành phần trung tâm của workshop và được sử dụng để quản lý quá trình sao lưu và khôi phục dữ liệu.

Tôi lựa chọn AWS Backup thay vì quản lý các bản snapshot thủ công vì dịch vụ này cho phép quản lý tập trung quá trình Backup & Recovery.

Quy trình chính có thể được mô tả như sau:

```text
Backup Plan
     ↓
Backup Job
     ↓
Backup Vault
     ↓
Recovery Point
     ↓
Restore
```

Trong đó:

* **Backup Plan** xác định cách thức và lịch thực hiện sao lưu.
* **Backup Job** đại diện cho một quá trình sao lưu được thực hiện.
* **Backup Vault** được sử dụng để quản lý các bản sao lưu.
* **Recovery Point** đại diện cho bản sao lưu của tài nguyên tại một thời điểm nhất định.
* **Restore** sử dụng Recovery Point để khôi phục tài nguyên khi xảy ra sự cố.

AWS Backup có thể bảo vệ cả **EC2/EBS và Amazon S3**, phù hợp với mục tiêu xây dựng một quy trình Backup & Recovery thống nhất cho các loại dữ liệu khác nhau trong workshop.

---

#### Amazon CloudWatch

**Amazon CloudWatch** được sử dụng để giám sát hoạt động của AWS Backup.

Trong workshop, CloudWatch được sử dụng để:

* Theo dõi các metric liên quan đến AWS Backup.
* Quan sát trạng thái hoạt động của Backup Job.
* Cấu hình CloudWatch Alarm phục vụ quá trình giám sát.
* Hỗ trợ phát hiện khi hoạt động Backup có vấn đề.

Việc bổ sung Amazon CloudWatch giúp hệ thống không chỉ thực hiện **Backup và Restore** mà còn có khả năng theo dõi trạng thái hoạt động của quá trình sao lưu.

CloudWatch vì vậy đóng vai trò là thành phần **Monitoring** trong kiến trúc của workshop.

---

Tóm lại, các dịch vụ được lựa chọn trong workshop có vai trò như sau:

| Dịch vụ AWS | Vai trò trong hệ thống |
| --- | --- |
| **Amazon EC2** | Triển khai Web Server |
| **Amazon EBS** | Lưu trữ dữ liệu dạng block của EC2 |
| **Amazon S3** | Lưu trữ dữ liệu dạng object |
| **AWS IAM** | Quản lý quyền truy cập giữa các dịch vụ |
| **AWS Backup** | Quản lý Backup và Restore tập trung |
| **Amazon CloudWatch** | Giám sát hoạt động Backup |

Các dịch vụ trên kết hợp với nhau để tạo thành hệ thống **Backup & Recovery cho Web Server trên AWS**, trong đó EC2, EBS và S3 là các tài nguyên cần bảo vệ; IAM kiểm soát quyền truy cập; AWS Backup thực hiện sao lưu và khôi phục; còn CloudWatch đảm nhiệm chức năng giám sát.