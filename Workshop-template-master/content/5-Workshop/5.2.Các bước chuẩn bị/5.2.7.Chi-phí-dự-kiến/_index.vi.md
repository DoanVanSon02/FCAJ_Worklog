---
title: "Chi phí dự kiến"
date: 2026-01-01
weight: 7
chapter: false
pre: " <b> 5.2.7. </b> "
---

## Ước tính chi phí triển khai

Mặc dù workshop được xây dựng với quy mô nhỏ, các tài nguyên AWS vẫn có thể phát sinh chi phí trong thời gian hoạt động. Vì vậy, trước khi triển khai, tôi ước tính chi phí dựa trên chính cấu hình được sử dụng trong bài lab.

Các giá trị dưới đây được tính tại Region:

```text
US East (N. Virginia)
us-east-1
```

Chi phí thực tế có thể thay đổi tùy theo thời gian sử dụng, lượng dữ liệu, số lần thực hiện Backup/Restore và chính sách giá của AWS tại thời điểm triển khai.

---

### Amazon EC2

Web Server trong workshop sử dụng cấu hình:

```text
AMI: Amazon Linux 2023
Instance type: t3.micro
vCPU: 2
Memory: 1 GiB
Pricing model: On-Demand
```

#### Tại sao lựa chọn `t3.micro`?

Mục tiêu của EC2 trong workshop chỉ là chạy một Web Server Apache đơn giản, tạo dữ liệu thử nghiệm và thực hiện một số lệnh AWS CLI. Hệ thống không có lượng truy cập lớn và không chạy các ứng dụng yêu cầu nhiều CPU hoặc bộ nhớ.

`t3.micro` cung cấp **2 vCPU và 1 GiB RAM**, đủ cho yêu cầu của bài lab nhưng có chi phí thấp hơn đáng kể so với các instance lớn như `t3.small` hoặc `t3.medium`.

Dòng T3 cũng phù hợp với các workload có mức sử dụng CPU thấp trong phần lớn thời gian nhưng đôi lúc cần tăng hiệu năng trong thời gian ngắn.

Nguồn tham khảo: [Amazon EC2 T3 Instances](https://aws.amazon.com/ec2/instance-types/t3/)

Tôi sử dụng **On-Demand** thay vì Reserved Instance vì workshop chỉ chạy trong thời gian ngắn, không cần cam kết sử dụng EC2 trong 1 hoặc 3 năm.

Giá On-Demand của `t3.micro` Linux tại `us-east-1` khoảng:

```text
0,0104 USD/giờ
```

Nếu quên để EC2 chạy liên tục trong khoảng **30 ngày (720 giờ)**:

```text
0,0104 × 720
≈ 7,49 USD/tháng
```

#### Public IPv4

EC2 trong workshop sử dụng Public IPv4 để tôi có thể truy cập Web Server từ trình duyệt và kết nối đến instance.

AWS hiện tính khoảng:

```text
0,005 USD / Public IPv4 / giờ
```

Nếu duy trì trong 30 ngày:

```text
0,005 × 720
= 3,60 USD/tháng
```

Điều này cũng cho thấy việc terminate EC2 sau khi hoàn thành workshop là cần thiết, thay vì chỉ để tài nguyên tiếp tục hoạt động.

---

### Amazon EBS

EC2 sử dụng hai EBS Volume:

```text
Root Volume:
8 GiB gp3

Data Volume:
1 GiB gp3
Mount point: /data
```

Tổng dung lượng provisioned:

```text
8 GiB + 1 GiB
= 9 GiB
```

#### Tại sao lựa chọn `gp3`?

`gp3` là loại General Purpose SSD phù hợp với các workload thông thường.

Với cấu hình mặc định, gp3 cung cấp:

```text
3.000 IOPS
125 MiB/s throughput
```

Mức hiệu năng này vượt xa yêu cầu của một Web Server nhỏ trong workshop.

Tại `us-east-1`, gp3 có mức giá khoảng:

```text
0,08 USD/GiB-tháng
```

trong khi gp2 có giá khoảng:

```text
0,10 USD/GiB-tháng
```

Vì bài lab không yêu cầu IOPS cao, tôi giữ nguyên cấu hình hiệu năng mặc định của gp3 thay vì mua thêm IOPS hoặc throughput.

Nguồn tham khảo: [AWS Documentation – EBS Volume Types](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-storage-compare-volume-types.html)

Chi phí cho 9 GiB:

```text
9 × 0,08
= 0,72 USD/tháng
```

Nếu chỉ duy trì EBS trong khoảng một ngày:

```text
0,72 / 30
≈ 0,024 USD/ngày
```

#### Tại sao Data Volume chỉ có 1 GiB?

Dữ liệu thử nghiệm của workshop chỉ gồm một số file text như:

```text
version.txt
customer.txt
app-data.txt
```
Vì vậy không cần tạo volume lớn hơn.

Việc sử dụng volume nhỏ cũng giúp hạn chế chi phí.

---

### Amazon S3

S3 Bucket sử dụng:

```text
Storage class: S3 Standard
Versioning: Enabled
Block Public Access: Enabled
```

#### Tại sao sử dụng S3 Standard?

Các object trong workshop được tạo, đọc, xóa và restore nhiều lần trong thời gian ngắn.

Vì vậy tôi sử dụng **S3 Standard** thay vì các storage class dành cho dữ liệu ít truy cập hoặc lưu trữ dài hạn.

S3 Standard không yêu cầu tôi phải quản lý máy chủ lưu trữ riêng và phù hợp với các file đang được truy cập thường xuyên.

Giá S3 Standard tại khu vực này vào khoảng:

```text
0,023 USD/GB-tháng
```

cho mức dung lượng đầu tiên.

Trong workshop, các object chỉ là một số file text rất nhỏ. Giả sử tổng dữ liệu thậm chí lên tới **1 MB**:

```text
1 MB ≈ 0,000977 GB
```

Chi phí lưu trữ một tháng xấp xỉ:

```text
0,000977 × 0,023
≈ 0,000022 USD/tháng
```

Do đó, chi phí S3 Storage của bài lab gần như không đáng kể.

S3 còn tính phí theo các request như PUT, GET và LIST. Tuy nhiên workshop chỉ thực hiện một số lượng rất nhỏ request nên phần chi phí này cũng rất thấp.

Ví dụ giá PUT của S3 Standard khoảng:

```text
0,005 USD / 1.000 request
```

#### Ảnh hưởng của Versioning

Versioning được bật vì đây là cấu hình cần thiết cho bài toán Backup của S3 và giúp duy trì các phiên bản object.

Tuy nhiên, các version cũ vẫn chiếm dung lượng và tiếp tục được tính phí.

Vì vậy trong hệ thống thực tế, nếu dữ liệu lớn và thay đổi thường xuyên thì cần kết hợp **Lifecycle Policy** để quản lý các version cũ.

---

### AWS IAM

Workshop sử dụng:

```text
BackupLab-EC2-S3-Role
BackupLab-EC2-S3-Policy
AWSBackupDefaultServiceRole
```

AWS IAM không có phí riêng cho việc tạo IAM Role hoặc IAM Policy. AWS chỉ tính phí cho các dịch vụ mà các identity này truy cập.

Nguồn tham khảo: [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started.html)

Do đó:

```text
Chi phí IAM trực tiếp:
0 USD
```

Việc sử dụng IAM Role trong workshop chủ yếu nhằm đáp ứng yêu cầu bảo mật và tránh lưu Access Key trực tiếp trên EC2.

---

### AWS Backup

AWS Backup không tính một khoản phí cố định chỉ để tạo Backup Plan hoặc Backup Vault.

Chi phí chủ yếu phụ thuộc vào:

* Lượng dữ liệu backup thực tế.
* Thời gian lưu Recovery Point.
* Loại tài nguyên được bảo vệ.
* Các thao tác Restore.

Nguồn tham khảo: [AWS Backup Pricing](https://aws.amazon.com/backup/pricing/)

Trong workshop tôi sử dụng:

```text
Backup Vault:
BackupLab-Vault

Retention:
7 ngày

Protected resources:
EC2 + EBS
S3
```

#### Tại sao chỉ giữ Recovery Point 7 ngày?

Mục tiêu của workshop là kiểm thử Backup & Recovery trong thời gian ngắn, không phải xây dựng hệ thống lưu trữ backup dài hạn.

Retention 7 ngày đủ để thực hiện quy trình:

```text
Backup
   ↓
Tạo Recovery Point
   ↓
Mô phỏng sự cố
   ↓
Restore
   ↓
Kiểm tra kết quả
```

đồng thời hạn chế việc giữ các Recovery Point không cần thiết và phát sinh thêm chi phí.

#### Ví dụ chi phí Backup EC2/EBS

AWS đưa ra mức khoảng:

```text
0,05 USD/GB-tháng
```

cho EBS backup storage trong ví dụ tại US East (N. Virginia).

Nguồn tham khảo: [AWS Backup Pricing](https://aws.amazon.com/backup/pricing/)

Để dễ hình dung, giả sử toàn bộ **9 GiB EBS** của workshop trở thành dữ liệu backup cần tính phí và Recovery Point được giữ đủ 7 ngày:

```text
9 × 0,05 × 7 / 30
≈ 0,105 USD
```

Đây là cách tính đơn giản để ước lượng.

Trong thực tế, EBS backup dựa trên snapshot và các backup tiếp theo có tính chất incremental, vì vậy dung lượng thực tế được tính phí có thể khác với toàn bộ dung lượng provisioned.

#### Backup S3

AWS Backup cho S3 cũng tính theo dung lượng backup và phát sinh thêm các request GET/LIST cũng như EventBridge events.

Đặc biệt, AWS quy định object S3 nhỏ hơn **128 KB** được tính như một object backup có kích thước 128 KB.

Nguồn tham khảo: [AWS Backup Pricing](https://aws.amazon.com/backup/pricing/)

Trong bài lab chỉ có một số file text rất nhỏ, vì vậy chi phí Backup S3 là rất thấp.

Tuy nhiên, ở hệ thống thực tế có hàng triệu object thì số lượng object và request có thể tạo ra chi phí đáng kể.

---

### Amazon CloudWatch

Workshop sử dụng CloudWatch để theo dõi AWS Backup metrics và tạo Alarm.

Ví dụ:

```text
BackupLab-BackupCompleted-Test-Alarm
```

Một standard-resolution metric alarm có giá khoảng:

```text
0,10 USD / alarm metric / tháng
```

và được tính theo thời gian Alarm tồn tại.

Nguồn tham khảo: [Amazon CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/)

Nếu workshop chỉ giữ một Alarm trong một ngày:

```text
0,10 / 30
≈ 0,0033 USD
```

Nếu sử dụng hai Alarm, chẳng hạn một Alarm theo dõi EC2 và một Alarm theo dõi S3, và để chúng tồn tại cả tháng thì chi phí lý thuyết vào khoảng:

```text
2 × 0,10
= 0,20 USD/tháng
```

Trong bài lab, lượng metric và Alarm rất nhỏ nên chi phí CloudWatch không đáng kể.

---

### Chi phí phát sinh khi Restore

Khi Restore EC2, AWS Backup tạo ra **một EC2 mới cùng các EBS Volume tương ứng**.

Điều này có nghĩa là trong một khoảng thời gian có thể tồn tại đồng thời:

```text
EC2 cũ
   +
EC2 restored
```

Ví dụ nếu EC2 restored sử dụng `t3.micro` và tồn tại khoảng **2 giờ** để tôi kiểm tra dữ liệu:

```text
EC2:
0,0104 × 2
= 0,0208 USD

Public IPv4:
0,005 × 2
= 0,0100 USD
```

Cộng thêm một lượng rất nhỏ cho EBS của instance restored, tổng chi phí phát sinh khoảng:

```text
≈ 0,03 USD
```

Điều này giải thích tại sao sau khi hoàn thành quá trình **Validation**, EC2 restored cũng cần được terminate.

---

### Tổng hợp chi phí ước tính của workshop

Với giả định:

```text
1 EC2 t3.micro chạy 24 giờ
1 Public IPv4 chạy 24 giờ
9 GiB gp3
1 Recovery Point EC2/EBS giữ 7 ngày
S3 chỉ chứa vài file text
1 CloudWatch Alarm
EC2 restored chạy khoảng 2 giờ
```

ước tính theo giá niêm yết:

| Thành phần | Chi phí ước tính |
| --- | ---: |
| EC2 `t3.micro` trong 24 giờ | ~0,25 USD |
| Public IPv4 trong 24 giờ | ~0,12 USD |
| 9 GiB EBS gp3 trong 1 ngày | ~0,024 USD |
| S3 với dữ liệu dưới 1 MB | <0,001 USD |
| EC2/EBS Backup giữ khoảng 7 ngày | ~0,105 USD* |
| S3 Backup | Gần như không đáng kể với vài object nhỏ |
| IAM | 0 USD |
| 1 CloudWatch Alarm trong 1 ngày | ~0,003 USD |
| EC2 restored tồn tại khoảng 2 giờ | ~0,03 USD |
| **Tổng ước tính** | **Khoảng 0,5–0,6 USD** |

\* Con số AWS Backup ở đây là **ước tính bảo thủ** bằng cách giả sử toàn bộ 9 GiB trở thành dung lượng backup tính phí. Chi phí thực tế phụ thuộc lượng block dữ liệu được lưu trong backup.

Con số trên **chưa áp dụng Free Tier, credit hoặc các ưu đãi của tài khoản**, vì mỗi tài khoản AWS có thể có điều kiện khác nhau.

---

### Nhận xét về chi phí

Từ ước tính trên có thể thấy bản thân dữ liệu thử nghiệm trên S3 và EBS có chi phí rất nhỏ.

Phần đáng chú ý hơn là:

* Thời gian chạy EC2.
* Chi phí Public IPv4.
* Việc giữ Recovery Point trong thời gian dài.
* Các tài nguyên được tạo thêm trong quá trình Restore.

Vì vậy, sau khi hoàn thành workshop, tôi cần kiểm tra và xóa các tài nguyên không còn sử dụng.

Các tài nguyên này sẽ được xử lý tại phần:

```text
5.9. Dọn dẹp tài nguyên
```

Việc dọn dẹp đầy đủ giúp tránh trường hợp EC2, Public IPv4, EBS Volume, Recovery Point hoặc CloudWatch Alarm tiếp tục tồn tại và phát sinh chi phí ngoài dự kiến.