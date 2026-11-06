---
title : "Blog Posts"
date :  05-11-2026
weight : 2
chapter : false
pre : " <b> 3.2 </b> "
---

# 📌 Lập kế hoạch và đo lường khả năng đáp ứng EC2 với Amazon EC2 Capacity Manager

Chào mọi người, mình vừa đọc bài **“Plan and measure EC2 flexibility with Amazon EC2 Capacity Manager”** của **Yagya Vir Singh và Brian Phillippi**, đăng trên AWS Compute Blog ngày **04/11/2026**.

Bài viết đề cập đến một câu hỏi khá thú vị: Khi hệ thống cần mở rộng, làm sao biết nên chọn loại EC2 instance và Availability Zone nào? Sau khi triển khai, làm sao kiểm tra những lựa chọn đó có giúp hệ thống nhận đủ tài nguyên hay không?

Mình tổng hợp lại những nội dung đã tìm hiểu để cùng trao đổi với mọi người. Các số liệu bên dưới thuộc tình huống minh họa AnyCompany trong bài gốc của AWS, không phải kết quả thử nghiệm của mình. [1]

## 🔎 Trước tiên, Amazon EC2 là gì?

**Amazon EC2**, viết đầy đủ là **Amazon Elastic Compute Cloud**, là dịch vụ cung cấp tài nguyên tính toán theo nhu cầu trên AWS. Có thể hiểu đơn giản là mình khởi tạo các máy chủ ảo, gọi là instance, để chạy ứng dụng và tăng hoặc giảm tài nguyên khi cần.

Mỗi instance type có cấu hình tài nguyên khác nhau, như CPU, bộ nhớ, khả năng mạng và lưu trữ, để phù hợp với từng nhu cầu sử dụng. [2]

## 🔎 Amazon EC2 Capacity Manager là gì?

**Amazon EC2 Capacity Manager** là công cụ giúp theo dõi, phân tích và quản lý capacity của EC2 từ một giao diện tập trung, bao gồm On-Demand Instances, Spot Instances và Capacity Reservations.

Công cụ hỗ trợ tổng hợp dữ liệu qua nhiều Region và, khi tích hợp AWS Organizations, qua các tài khoản thành viên trong tổ chức. Nhờ đó, đội vận hành có thể quan sát việc sử dụng tài nguyên và tìm cơ hội tối ưu mà không phải tự tổng hợp từ nhiều nơi. [3]

Trong bài này, AWS tập trung vào hai khả năng:

- **Real-time supportability checks:** đánh giá khả năng đáp ứng yêu cầu của từng cặp instance type và Availability Zone trước khi mở rộng, không cần khởi tạo hoặc đặt trước instance.
- **Capacity fulfillment metrics:** đo lượng tài nguyên thực tế đã được cấp so với lượng yêu cầu, đồng thời cho biết tài nguyên đến từ instance type và Availability Zone nào.

Hai khả năng này thuộc **AWS Unified Operations**. Điểm cần phân biệt là fulfillment metrics đo kết quả khởi tạo thực tế, không phải đo độ chính xác của supportability checks. [1][4]

📷 ![Hình 1 – Tổng quan hai khả năng, nguồn AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-1-1.png)

## 💡 “Flexibility” trong bài được hiểu như thế nào?

Thay vì giới hạn workload vào một instance type trong một Availability Zone (AZ), mình cấu hình cho phép sử dụng nhiều lựa chọn phù hợp. EC2 sẽ có thêm phương án đáp ứng khi một lựa chọn không đủ capacity.

Trong bài, một **“combination”** là một cặp **instance type × AZ**, chẳng hạn `r5.16xlarge` tại `use1-az5`.

AWS minh họa cách đánh giá và cải thiện sự linh hoạt này qua bốn bước dưới đây. [1]

## 🛠️ Chuẩn bị trước khi thực hiện

- Bật Capacity Manager cho một tài khoản hoặc ở cấp tổ chức thông qua management account hay delegated administrator.
- Xác nhận tài khoản được bao phủ bởi AWS Unified Operations để sử dụng hai khả năng được đề cập.
- Nếu muốn theo dõi theo workload, kích hoạt tag tương ứng làm monitored tag key. Một số tag như `aws:eks:cluster-name` và `aws:autoscaling:groupName` có sẵn, không cần kích hoạt. Bài gốc sử dụng `aws:eks:cluster-name`.

Có thể truy vấn dữ liệu từ các commercial AWS Regions thông qua Region đã bật Capacity Manager. [1]

## 1️⃣ Kiểm tra workload có nhận đủ tài nguyên hay không

Trong Capacity Manager, mở tab **Fulfillment**, chọn khoảng thời gian và nhóm chỉ số **On-Demand** hoặc **Spot**. Sau đó lọc theo Region, tài khoản hoặc tag.

Bốn chỉ số cần chú ý:

- **`CapacityTotalRequestedVcpu`:** tổng số vCPU được yêu cầu. Các lần thử lại trong vòng 5 phút với cùng cấu hình chỉ được tính một lần.
- **`CapacityTotalFulfilledVcpu`:** tổng số vCPU thực tế được cấp.
- **`CapacityRequestFulfillmentRate`:** tỷ lệ đáp ứng yêu cầu, tính theo lượng vCPU được cấp chia cho lượng yêu cầu.
- **`AvgInstAzCountPerRequestedVcpu`:** số tổ hợp instance type × AZ trung bình mà yêu cầu được cấu hình để sử dụng. Giá trị lớn hơn 1 thể hiện yêu cầu có sự linh hoạt.

Mình có thể sắp xếp theo **Requested** để xem workload có nhu cầu lớn nhất, hoặc theo **request fulfillment rate** để tìm workload cần cải thiện.

Trong ví dụ của AWS:

- **search-index** yêu cầu **51.712 vCPU**, nhận được **45.632 vCPU**, tương ứng tỷ lệ đáp ứng **88,2%**; các yêu cầu sử dụng trung bình **21 tổ hợp**.
- **batch-etl** chỉ sử dụng được **một tổ hợp** và có tỷ lệ đáp ứng **72,8%**.

Đây là điểm khởi đầu để xem workload nào đang thiếu capacity và cần phân tích sâu hơn. Nếu dùng CLI, bài gốc truy vấn qua `get-capacity-manager-metric-data`. [1]

📷 ![Hình 2 – Fulfillment theo workload, nguồn AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-2.png)

## 2️⃣ Xem sự linh hoạt đang phát huy hiệu quả ở đâu

Tại một dòng trong tab **Fulfillment**, chọn **View breakdown** để xem chi tiết theo instance type, instance family và AZ.

Điểm mình thấy cần đọc kỹ là tỷ lệ đáp ứng của toàn bộ yêu cầu và tỷ lệ của từng tổ hợp được tính ở hai cấp khác nhau. Khi một yêu cầu có nhiều lựa chọn, tổ hợp này có thể bù cho tổ hợp khác đang thiếu capacity.

Với **search-index**:

- Ba tổ hợp tại `use1-az5` đáp ứng khoảng **68,6–82,8%**.
- `r5.16xlarge` tại `use1-az4` và `use1-az6` đáp ứng **98,5–100%**.
- `r6a.16xlarge` tại `use1-az4` đáp ứng **100%**.

Như vậy, các AZ khác đã bù được một phần thiếu hụt tại `use1-az5`.

Ở phần breakdown của bài gốc, **28 tổ hợp** có tỷ lệ đáp ứng trung bình riêng lẻ là **46,6%**, trong khi tỷ lệ ở cấp yêu cầu đạt **88,2%**. Đây là ví dụ cho thấy lợi ích của việc có nhiều phương án cấp tài nguyên.

Tuy nhiên, `use1-az5` vẫn cung cấp tới **26.880 vCPU**, hơn một nửa tổng lượng được cấp. Vì vậy, đây là nơi cần xem xét bổ sung lựa chọn hoặc đặt trước capacity, thay vì chỉ nhìn tỷ lệ thấp rồi loại bỏ.

Mỗi dòng còn có liên kết **Check supportability** để kiểm tra tổ hợp tương ứng. Khi truy vấn chỉ số `InstAzFulfillmentRate` qua API, cần có chiều `availability-zone-id` và một trong hai chiều `instance-type` hoặc `instance-family`. [1]

📷 ![Hình 3 – So sánh tỷ lệ ở cấp yêu cầu và từng tổ hợp, nguồn AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-3.png)

📷 ![Hình 4 – Breakdown và liên kết Check supportability, nguồn AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-4.png)

## 3️⃣ Kiểm tra khả năng đáp ứng trước khi mở rộng

Trước một đợt scale up hoặc migration lớn, vào tab **Supportability** và nhập:

- Loại tài nguyên: On-Demand hoặc Spot.
- Lượng capacity cần sử dụng.
- Các AZ dự kiến.
- Danh sách instance type hoặc yêu cầu về thuộc tính tài nguyên.

Kết quả gồm điểm cho từng tổ hợp và điểm tổng hợp:

- **High:** tổ hợp có capacity vượt mức yêu cầu với phần dư đáng kể.
- **Medium:** có thể đáp ứng nhưng phần dư ít.
- **Low:** capacity thấp hơn mức yêu cầu.

Trong ví dụ, AnyCompany kiểm tra nhu cầu **100 instance** trên ba AZ: `use1-az4`, `use1-az5` và `use1-az6`, với năm loại instance: `r5.16xlarge`, `r6a.16xlarge`, `r7a.16xlarge`, `r6i.16xlarge` và `r7i.16xlarge`.

Kết quả tổng hợp là **High**, nhưng **15 tổ hợp** có kết quả khác nhau:

- **4 tổ hợp** đạt High.
- **5 tổ hợp** đạt Medium.
- **6 tổ hợp** đạt Low.

Cụ thể, `r5.16xlarge` đạt High ở cả ba AZ; `r7a.16xlarge` đạt High tại `use1-az6`. `r6a.16xlarge` đạt Medium ở cả ba AZ, còn `r7i.16xlarge` đạt Medium tại `use1-az4` và `use1-az6`. Tất cả tổ hợp của `r6i.16xlarge` đều đạt Low.

Một lần kiểm tra hỗ trợ tối đa **10 AZ**, có thể thuộc nhiều Region, và tối đa **100 tổ hợp**. Ngoài việc liệt kê instance type, có thể chọn theo thuộc tính; ví dụ trong bài là **64 vCPU** và bộ nhớ tối thiểu **524.288 MiB**.

> Điều cần nhớ: điểm số chỉ mang tính hướng dẫn tại thời điểm kiểm tra, không phải cam kết giữ sẵn tài nguyên. Nếu cần bảo đảm capacity, AWS hướng tới Capacity Reservations hoặc future-dated Capacity Reservations.

AWS cũng khuyến nghị kiểm tra ở giai đoạn lập kế hoạch, tránh gọi trước mọi lần khởi tạo vì có thể tạo thêm phụ thuộc liên Region và tiêu tốn quota theo giờ. [1]

📷 ![Hình 5 – Kiểm tra nhu cầu 100 instance, nguồn AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-5.png)

## 4️⃣ Đưa kết quả vào cấu hình thực tế

Sau khi xác định các lựa chọn phù hợp, cập nhật cấu hình **Auto Scaling group**, **EC2 Fleet** hoặc **Karpenter** để những lần khởi tạo tiếp theo sử dụng được sự linh hoạt đó.

Với Auto Scaling group trong bài, nhóm thực hiện:

- Giữ `r5.16xlarge`, `r6a.16xlarge` và `r7a.16xlarge`.
- Bổ sung `r7i.16xlarge` vì đạt Medium tại hai AZ.
- Không bổ sung `r6i.16xlarge` vì các tổ hợp đều đạt Low.

Ví dụ sử dụng **mixed instances policy** với chiến lược On-Demand **“prioritized”**. Với workload được mô tả bằng yêu cầu vCPU và bộ nhớ, có thể sử dụng **attribute-based instance type selection** để mở rộng tập instance phù hợp.

Sau thay đổi, theo dõi `AvgInstAzCountPerRequestedVcpu` để kiểm tra số lựa chọn trong các yêu cầu đã tăng hay chưa.

Nếu có nhu cầu cố định ở một AZ cụ thể, có thể kết hợp đặt trước phần tài nguyên nền và giữ phần còn lại linh hoạt. Bài gốc minh họa việc đặt trước **10 instance r5.16xlarge tại use1-az5**.

Chi tiết đáng chú ý: `r5.16xlarge` tại AZ này từng chỉ đáp ứng **82,8%**, dù kiểm tra hiện tại cho điểm **High**. Hai kết quả phản ánh dữ liệu lịch sử và trạng thái tại thời điểm kiểm tra nên không mâu thuẫn.

**Capacity Reservations vẫn bị tính phí khi chưa sử dụng**, vì vậy cần đặt số lượng phù hợp. Với reservation trong ví dụ không có thời hạn cam kết, việc hủy không phát sinh phí hủy; các instance đang chạy tiếp tục hoạt động theo giá On-Demand. [1]

📷 ![Hình 6 – Từ đánh giá đến cấu hình fleet, nguồn AWS](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2026/11/04/ComputeBlog-2823-6.png)

## 📝 Điều mình rút ra sau khi đọc

Việc cho phép sử dụng nhiều instance type và AZ sẽ hữu ích hơn khi mình có số liệu để kiểm tra hiệu quả của từng lựa chọn.

Supportability checks hỗ trợ quyết định trước khi mở rộng; fulfillment metrics giúp nhìn lại tài nguyên thực tế đã được cấp. Từ đó, mình biết nên bổ sung lựa chọn ở đâu và phần nào cần đặt trước.

AWS đề xuất kiểm tra fulfillment rate hằng tuần, xem breakdown của các workload chưa đạt mục tiêu, đồng thời đưa supportability check vào quy trình quản lý thay đổi trước khi bổ sung hoặc mở rộng workload. [1]

**Mọi người đã từng gặp tình huống Auto Scaling cần tăng instance nhưng không nhận đủ capacity chưa? Khi đó mọi người thường ưu tiên thêm instance type, mở rộng sang AZ khác hay kết hợp Capacity Reservations?** Mình mong được nghe thêm kinh nghiệm thực tế và góp ý để hiểu rõ hơn 😊

## 📚 Nguồn tham khảo

**[1] Yagya Vir Singh và Brian Phillippi — “Plan and measure EC2 flexibility with Amazon EC2 Capacity Manager”, AWS Compute Blog, 04/11/2026.** Nguồn cho toàn bộ quy trình bốn bước, số liệu AnyCompany và sáu hình minh họa:

[https://aws.amazon.com/blogs/compute/plan-and-measure-ec2-flexibility-with-amazon-ec2-capacity-manager/](https://aws.amazon.com/blogs/compute/plan-and-measure-ec2-flexibility-with-amazon-ec2-capacity-manager/)

**[2] AWS Documentation — “What is Amazon EC2?”:**

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)

**[3] AWS Documentation — “EC2 Capacity Manager”:**

[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/capacity-manager.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/capacity-manager.html)

**[4] AWS What's New — “New features for Amazon EC2 Capacity Manager”, 04/11/2026:**

[https://aws.amazon.com/about-aws/whats-new/2026/11/ec2-capacity-manager-supportability-and-fulfillment/](https://aws.amazon.com/about-aws/whats-new/2026/11/ec2-capacity-manager-supportability-and-fulfillment/)


**Link bài đăng:**
https://www.facebook.com/groups/awsstudygroupfcj/permalink/2327679437997015/#