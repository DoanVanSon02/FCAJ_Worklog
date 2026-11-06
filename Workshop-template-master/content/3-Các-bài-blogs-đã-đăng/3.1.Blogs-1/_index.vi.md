---
title : "Blog Posts"
date :  05-11-2026
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---
# 🚀 Amazon S3 Tables hỗ trợ chuyển đổi Apache Hive sang Apache Iceberg

Chào mọi người 👋

Mình vừa đọc bài viết **“Amazon S3 Tables now offer managed migration of Apache Hive tables to Apache Iceberg”** của tác giả **Channy Yun** trên AWS News Blog, đăng ngày **04/11/2026**.

Mình xin chia sẻ lại nội dung bằng tiếng Việt, kèm một số giải thích để những bạn đang tìm hiểu về dữ liệu trên AWS có thể cùng theo dõi. Bài này dựa trên tài liệu AWS, chưa phải trải nghiệm triển khai thực tế của mình, nên rất mong nhận được góp ý và chia sẻ từ mọi người.

## 📚 Trước tiên, cùng làm rõ ba khái niệm

### 1. Amazon S3 Tables là gì?

Tên chính thức là **Amazon S3 Tables**, có chữ “s”. Đây là khả năng lưu trữ và quản lý các bảng Apache Iceberg trên Amazon S3 theo hình thức được AWS quản lý hoàn toàn.

S3 Tables tự động thực hiện những công việc như gộp các tệp nhỏ thành tệp lớn hơn (*compaction*), quản lý snapshot và xóa các tệp không còn được tham chiếu. Dữ liệu có thể được sử dụng bởi các công cụ tương thích như Amazon Athena, Apache Spark và Trino.

Có thể hiểu đơn giản: **S3 Tables giúp lưu trữ dữ liệu dạng bảng và giảm công việc bảo trì bảng cho người dùng.** [1]

### 2. Apache Hive là gì?

**Apache Hive** là một hệ thống kho dữ liệu phân tán mã nguồn mở, cho phép đọc, ghi và phân tích dữ liệu quy mô lớn bằng SQL trên các hệ thống lưu trữ phân tán.

Một thành phần quan trọng là **Hive Metastore**, nơi quản lý metadata của các bảng và phân vùng. Metadata là thông tin mô tả dữ liệu, giúp các công cụ biết cách làm việc với bảng. [2]

Trong phạm vi cập nhật của AWS lần này, “bảng Hive” là các bảng được đăng ký trong **AWS Glue Data Catalog**, với dữ liệu lưu dưới định dạng **Parquet trên Amazon S3**. [4]

### 3. Apache Iceberg là gì?

**Apache Iceberg** là một **định dạng bảng mở — open table format** — dành cho các tập dữ liệu phân tích lớn.

Những khả năng đáng chú ý gồm:

- **Schema evolution:** hỗ trợ thay đổi cấu trúc bảng, chẳng hạn thêm, đổi tên hoặc sắp xếp lại cột.
- **Hidden partitioning:** xử lý thông tin phân vùng, giúp người dùng bớt phải đưa logic phân vùng vào truy vấn.
- **Time travel:** truy vấn dữ liệu theo snapshot, tức một phiên bản trạng thái của bảng.
- Cho phép nhiều công cụ như **Spark, Trino, Flink và Hive** cùng làm việc với bảng. [3]

## 📢 AWS vừa công bố điều gì?

Amazon S3 Tables hiện hỗ trợ **managed migration** — quá trình chuyển đổi do dịch vụ quản lý — từ bảng Apache Hive sang Apache Iceberg.

Tính năng áp dụng cho bảng Parquet trên Amazon S3 đã được đăng ký trong AWS Glue Data Catalog. Người dùng có thể hiện đại hóa data lake mà không phải tự xây dựng pipeline hoặc cấp phát tài nguyên tính toán phục vụ migration, trong khi hệ thống đang vận hành vẫn tiếp tục hoạt động trong phần lớn quá trình.

Theo bài viết, Iceberg hỗ trợ quản lý bảng ở quy mô petabyte, cùng các tính năng như schema evolution, hidden partitioning và time travel. Khi sử dụng S3 Tables, việc lưu trữ bảng còn đi kèm khả năng bảo trì, tối ưu hóa tự động, replication và S3 Intelligent-Tiering để hỗ trợ hiệu năng và chi phí khi dữ liệu tăng lên. [4]

## ⚙️ Điểm mới của managed migration

Trước đây, chuyển đổi bảng Hive sang Iceberg có thể mất nhiều tuần để xây dựng và quản lý hạ tầng. Việc chuyển đổi kiểu dữ liệu và chuẩn bị khả năng rollback — quay lại trạng thái trước đó khi cần — cũng làm tăng độ phức tạp.

Với cập nhật này, quá trình migration được cung cấp dưới dạng thao tác do dịch vụ quản lý và có thể điều khiển qua API.

S3 Tables xử lý các công việc gồm:

- Xác thực dữ liệu.
- Kiểm tra tính tương thích.
- Chuyển đổi kiểu dữ liệu.
- Nén lại tệp.
- Gộp các tệp nhỏ bằng compaction.

Trước khi sao chép, dịch vụ xác định các object nguồn thuộc phạm vi migration và tạo **báo cáo tương thích** để người dùng kiểm tra.

Trong quá trình này, có thể đồng bộ nhiều lần mà không phải dừng ghi vào bảng nguồn. Tuy nhiên, theo AWS, vẫn cần tạm dừng ghi trong vài phút ở bước **cutover** — thời điểm chuyển hệ thống sang sử dụng bảng đích.

> Vì vậy, mình hiểu đây là cách giảm thời gian gián đoạn, chứ không nên mặc định rằng toàn bộ quá trình hoàn toàn không có downtime. [4]

## 💰 Chi phí được đề cập như thế nào?

Theo bài viết, đối với managed migration:

- Người dùng trả phí cho dữ liệu được sao chép.
- Không có phí riêng cho bước chuyển đổi Hive sang Iceberg.
- Các object nguồn trong S3 Intelligent-Tiering vẫn giữ nguyên access tier hiện tại.

Điểm cần phân biệt là **“không thu phí chuyển đổi Hive sang Iceberg” không có nghĩa toàn bộ quá trình migration đều miễn phí.** [4]

## 🖥️ Thực hiện migration trên AWS Console

Quy trình được bài viết minh họa như sau:

### Bước 1: Chọn nơi lưu bảng đích

Truy cập [Amazon S3 console](https://console.aws.amazon.com/s3), chọn **table bucket đích**, sau đó chọn **Create table from migration**.

![Chọn Create table from migration trong Amazon S3 console](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2026/09/20/2026-s3-tables-hive-to-iceberg-1.jpg)

*Nguồn ảnh: AWS News Blog [4].*

### Bước 2: Tạo namespace và đặt tên bảng

Tạo namespace và tên cho bảng Iceberg đích.

Bảng đích được tạo với các thiết lập mặc định do AWS khuyến nghị, hướng đến hiệu năng truy vấn và hiệu quả lưu trữ.

![Tạo namespace và đặt tên bảng Iceberg đích](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2026/10/30/2026-s3-tables-hive-to-iceberg-2.jpg)

*Nguồn ảnh: AWS News Blog [4].*

### Bước 3: Bắt đầu chuyển đổi

S3 Tables kiểm tra bảng nguồn, tạo báo cáo tương thích và bắt đầu sao chép dữ liệu.

Trong quá trình migration, bảng đích ở trạng thái **read-only**, tức chỉ đọc. Khi hoàn tất migration, bảng đích được chuyển sang **read-write**, tức có thể đọc và ghi.

![Theo dõi quá trình migration bảng Hive sang Iceberg](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2026/10/12/2026-s3-tables-hive-to-iceberg-3.jpg)

*Nguồn ảnh: AWS News Blog [4].*

### Bước 4: Truy vấn dữ liệu

Có thể làm việc với dữ liệu thông qua tích hợp trực tiếp với **Amazon Athena** và **Amazon SageMaker Unified Studio** trên AWS console, hoặc sử dụng công cụ tương thích với **Apache Iceberg REST API**.

Tham khảo: [Tài liệu migration của S3 Tables](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-migrate.html).

## 💻 Code mẫu sử dụng AWS CLI

Ngoài Console, AWS cũng hỗ trợ thao tác thông qua **AWS CLI**, **AWS SDKs** hoặc **Agent Toolkit for AWS**.

Dưới đây là đoạn code mẫu trong bài viết gốc. Đoạn lệnh tạo tệp cấu hình JSON rồi gọi API thông qua lệnh `start-table-migration`.

> Các giá trị như account ID, IAM role ARN, bucket, KMS key, database, table và schema đều là ví dụ; cần thay bằng cấu hình phù hợp với môi trường thực tế trước khi chạy.

```bash
cat > /tmp/start-migration.json <<'EOF'
{
  "roleARN": "arn:aws:iam::123456789012:role/S3TablesMigrationRole",
  "destinationTable": {
    "tableBucketARN": "arn:aws:s3tables:us-east-1:123456789012:bucket/my-destination-bucket",
    "namespace": "mynamespace",
    "name": "my_destination_table",
    "encryptionConfiguration": {
      "sseAlgorithm": "aws:kms",
      "kmsKeyArn": "arn:aws:kms:us-east-1:123456789012:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    },
    "storageClassConfiguration": { "storageClass": "INTELLIGENT_TIERING" },
    "tags": { "team": "s3tables", "purpose": "migration" },
    "properties": {
      "writeFormat": "parquet",
      "icebergFormatVersion": 2,
      "compressionCodec": "zstd",
      "dictionaryEncoding": true
    },
    "schema": {
      "type": "struct",
      "schemaId": 0,
      "identifierFieldIds": [1],
      "fields": [
        { "id": 1, "name": "id",      "type": "long",   "required": true, "doc": "primary key" },
        { "id": 2, "name": "payload", "type": "string", "required": false }
      ]
    },
    "icebergSortOrder": {
      "orderId": 1,
      "fields": [ { "sourceId": 1, "transform": "identity", "direction": "asc", "nullOrder": "nulls-first" } ]
    }
  },
  "sourceType": "glueTable",
  "source": {
    "glueTable": {
      "catalogId": "123456789012",
      "databaseName": "my_glue_database",
      "tableName": "my_source_table"
    }
  },
  "dataMigration": {
    "maintenanceSettings": {
      "icebergCompaction": { "targetFileSizeMB": 512, "strategy": "binpack" },
      "icebergSnapshotManagement": { "minSnapshotsToKeep": 3, "maxSnapshotAgeHours": 168 }
    },
    "migrationPolicies": {
      "unmatchedSourceFieldsPolicy": "fail",
      "incompatibleValuesPolicy": "fail"
    }
  },
  "reviewMigrationPlan": false,
  "migrationOutput": {
    "bucket": "arn:aws:s3:::my-report-bucket",
    "prefixes": {
      "tableCompatibilityReport": "reports/table-compatibility/",
      "objectCompatibilityReport": "reports/object-compatibility/",
      "objectMigrationReport": "reports/object-migration/"
    }
  }
}
EOF

aws s3tables start-table-migration \
  --cli-input-json file:///tmp/start-migration.json
```

*Nguồn code: AWS News Blog [4].*

Để dễ đọc cấu hình, mình chia các thành phần chính như sau:

| Thành phần | Ý nghĩa |
| --- | --- |
| `roleARN` | IAM role dùng cho migration. |
| `destinationTable` | Thông tin bảng đích, gồm table bucket, namespace, tên bảng, mã hóa, storage class, schema và thứ tự sắp xếp. |
| `source` | Bảng nguồn trong AWS Glue Data Catalog. |
| `dataMigration` | Thiết lập bảo trì và chính sách xử lý các trường hợp dữ liệu không tương thích. |
| `migrationOutput` | Vị trí lưu các báo cáo tương thích và kết quả migration. |

Trong ví dụ, AWS sử dụng **Parquet**, **Iceberg format version 2**, nén **Zstandard**, kích thước tệp mục tiêu **512 MB** và các thiết lập quản lý snapshot được khai báo trong JSON. Đây là cấu hình minh họa, cần được đối chiếu với bảng nguồn và nhu cầu sử dụng. [4]

## 🔄 Đồng bộ thay đổi trước khi hoàn tất

Sau lần sao chép ban đầu, bảng nguồn có thể tiếp tục phát sinh thay đổi.

Để bảng đích bắt kịp các thay đổi đó, bài viết hướng dẫn sử dụng lệnh `sync-table-migration`. Có thể thực hiện nhiều lần trước khi hoàn tất migration; mỗi lần sync chỉ sao chép phần đã thay đổi.

Luồng thao tác được mô tả là:

**Start → Sync → Complete**

Trong đó, bước **Sync** có thể lặp lại nhiều lần trước khi chuyển sang **Complete**. [4]

## 🤖 Hỗ trợ từ coding agent và Agent Toolkit for AWS

Nếu sử dụng coding agent như [Kiro](https://kiro.dev/), bài viết giới thiệu skill `migrating-hive-tables-to-s3-tables` trong **Agent Toolkit for AWS**.

Skill này cung cấp hướng dẫn từng bước cùng các cơ chế kiểm soát an toàn cho agent, hỗ trợ:

- Phân tích IAM policy hiện có và đề xuất policy tương đương cho phía đích.
- Kiểm tra kế hoạch migration trước khi bắt đầu.
- Thực hiện các API theo quy trình **Start → Sync → Complete**.
- Theo dõi tiến độ, hướng dẫn kiểm tra dữ liệu và cutover.

Tham khảo: [README của skill trên GitHub](https://github.com/aws/agent-toolkit-for-aws/tree/main/skills/specialized-skills/analytics-skills/migrating-hive-tables-to-s3-tables).

## 🌏 Các Region được hỗ trợ khi ra mắt

Theo thông báo ngày **04/11/2026**, managed migration được cung cấp tại:

| Khu vực | Region |
| --- | --- |
| US East | N. Virginia, Ohio |
| US West | Oregon |
| Asia Pacific | Mumbai, Singapore, Sydney, Tokyo |
| Europe | Frankfurt, Ireland, Stockholm |

Ở thời điểm ra mắt, tính năng hỗ trợ bảng Hive đăng ký trong AWS Glue, lưu dữ liệu dưới dạng Parquet, với nguồn và đích nằm trong **cùng tài khoản AWS và cùng Region**. [4]

## 🛠️ Công cụ hỗ trợ tìm hiểu và xử lý sự cố

Bài viết cũng giới thiệu **AWS MCP Server** và các **plugin** để kết hợp với công cụ AI, phục vụ những việc như gọi API, tra cứu tài liệu, tìm thông tin Region được hỗ trợ hoặc xử lý sự cố.

- [AWS MCP Server](https://docs.aws.amazon.com/agent-toolkit/latest/userguide/getting-started-aws-mcp-server.html)
- [Plugins](https://docs.aws.amazon.com/agent-toolkit/latest/userguide/plugins.html)

Người dùng có thể thử tính năng trên [S3 console](https://console.aws.amazon.com/s3) và gửi phản hồi qua [AWS re:Post cho Amazon S3](https://repost.aws/tags/TADSTjraA0Q4-a1dxk6eUYaw/amazon-simple-storage-service) hoặc các kênh AWS Support đang sử dụng.

## 💬 Góc nhìn của mình

Sau khi đọc, mình thấy điểm đáng chú ý là AWS đang giúp giảm khối lượng công việc phải tự xây dựng khi chuyển các bảng Hive hiện có sang Iceberg.

Với người đang học như mình, đây cũng là một ví dụ để hiểu rõ hơn mối liên hệ giữa lưu trữ dữ liệu, metadata, định dạng bảng và công cụ truy vấn. Một thao tác migration còn liên quan đến tính tương thích, đồng bộ dữ liệu và thời điểm chuyển hệ thống sang bảng mới.

Nếu làm lab, mình muốn thử trên một bảng nhỏ, đối chiếu cấu trúc và kết quả truy vấn trước/sau, rồi quan sát cách đồng bộ khi bảng nguồn phát sinh thêm dữ liệu.

**Anh chị và các bạn đã từng chuyển từ Hive sang Iceberg chưa? Trong thực tế, phần nào thường khó nhất: schema, kiểu dữ liệu, phân vùng, quyền truy cập hay bước cutover?**

Rất mong được nghe thêm kinh nghiệm và góp ý của mọi người 😊

## 📎 Nguồn tham khảo

1. **AWS — Amazon S3 Tables:**  
   https://aws.amazon.com/s3/features/tables/

2. **Apache Software Foundation — Apache Hive:**  
   https://hive.apache.org/

3. **Apache Software Foundation — Apache Iceberg:**  
   https://iceberg.apache.org/

4. **Channy Yun — AWS News Blog, ngày 04/11/2026:**  
   *Amazon S3 Tables now offer managed migration of Apache Hive tables to Apache Iceberg*  
   https://aws.amazon.com/blogs/aws/amazon-s3-tables-now-offer-managed-migration-of-apache-hive-tables-to-apache-iceberg/

5. **AWS Documentation — Migrating tables to S3 Tables:**  
   https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-migrate.html


**Link bài đăng:**
https://www.facebook.com/groups/awsstudygroupfcj/permalink/2327147558050203/#