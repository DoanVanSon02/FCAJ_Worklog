---
title : "Blog Posts"
date :  05-11-2026
weight : 1
chapter : false
pre : " <b> 3.1 </b> "
---
# 🚀 Amazon S3 Tables Supports Migration from Apache Hive to Apache Iceberg: Concepts, Workflow, and Sample Code

Hi everyone 👋

I recently read the article **“Amazon S3 Tables now offer managed migration of Apache Hive tables to Apache Iceberg”** by **Channy Yun** on the AWS News Blog, published on **November 4, 2026**.

I would like to share the content in Vietnamese, along with some explanations so that those learning about data on AWS can follow along. This post is based on AWS documentation rather than my own hands-on implementation experience, so I would really appreciate everyone's feedback and insights.

## 📚 First, let's clarify three concepts

### 1. What is Amazon S3 Tables?

The official name is **Amazon S3 Tables**, with an “s”. It provides the ability to store and manage Apache Iceberg tables on Amazon S3, fully managed by AWS.

S3 Tables automatically performs tasks such as combining small files into larger ones (*compaction*), managing snapshots, and deleting unreferenced files. The data can be used by compatible tools such as Amazon Athena, Apache Spark, and Trino.

Simply put: **S3 Tables helps store tabular data and reduces table maintenance work for users.** [1]

### 2. What is Apache Hive?

**Apache Hive** is an open-source distributed data warehouse system that enables users to read, write, and analyze large-scale data using SQL on distributed storage systems.

An important component is **Hive Metastore**, which manages metadata for tables and partitions. Metadata is information that describes the data, helping tools understand how to work with tables. [2]

In the context of this AWS update, “Hive tables” are tables registered in the **AWS Glue Data Catalog**, with data stored in **Parquet format on Amazon S3**. [4]

### 3. What is Apache Iceberg?

**Apache Iceberg** is an **open table format** for large analytical datasets.

Notable capabilities include:

- **Schema evolution:** supports changes to a table's structure, such as adding, renaming, or reordering columns.
- **Hidden partitioning:** handles partition information, reducing the need for users to include partitioning logic in queries.
- **Time travel:** queries data using a snapshot, which represents a version of the table's state.
- Allows multiple tools such as **Spark, Trino, Flink, and Hive** to work with the same table. [3]

## 📢 What has AWS just announced?

Amazon S3 Tables now supports **managed migration** — a migration process managed by the service — from Apache Hive tables to Apache Iceberg.

This feature applies to Parquet tables on Amazon S3 that are registered in the AWS Glue Data Catalog. Users can modernize their data lakes without building their own pipelines or provisioning compute resources for migration, while their production systems continue operating throughout most of the process.

According to the article, Iceberg supports managing petabyte-scale tables, along with features such as schema evolution, hidden partitioning, and time travel. With S3 Tables, table storage also comes with automatic maintenance and optimization, replication, and S3 Intelligent-Tiering to support performance and cost efficiency as data grows. [4]

## ⚙️ What is new in managed migration?

Previously, migrating Hive tables to Iceberg could require weeks of work to build and manage infrastructure. Converting data types and preparing rollback capabilities — returning to a previous state when needed — also added complexity.

With this update, migration is available as a service-managed operation that can be controlled through APIs.

S3 Tables handles tasks including:

- Data validation.
- Compatibility checks.
- Data type conversion.
- File recompression.
- Combining small files through compaction.

Before copying data, the service identifies the source objects within the migration scope and generates a **compatibility report** for users to review.

During this process, synchronization can be performed multiple times without stopping writes to the source table. However, according to AWS, writes still need to be paused for a few minutes during **cutover** — the point at which the system switches to using the destination table.

> Therefore, my understanding is that this reduces downtime, rather than guaranteeing that the entire process has no downtime at all. [4]

## 💰 What does the article say about costs?

According to the article, for managed migration:

- Users pay for the data copied.
- There is no separate charge for the Hive-to-Iceberg conversion.
- Source objects in S3 Intelligent-Tiering remain in their current access tier.

The distinction to keep in mind is that **“no charge for Hive-to-Iceberg conversion” does not mean the entire migration process is free.** [4]

## 🖥️ Performing migration in the AWS Console

The article illustrates the following workflow:

### Step 1: Choose where to store the destination table

Open the [Amazon S3 console](https://console.aws.amazon.com/s3), select the **destination table bucket**, and then choose **Create table from migration**.

![Select Create table from migration in the Amazon S3 console](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2026/09/20/2026-s3-tables-hive-to-iceberg-1.jpg)

*Image source: AWS News Blog [4].*

### Step 2: Create a namespace and name the table

Create a namespace and a name for the destination Iceberg table.

The destination table is created with AWS-recommended default settings designed for query performance and storage efficiency.

![Create a namespace and name the destination Iceberg table](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2026/10/30/2026-s3-tables-hive-to-iceberg-2.jpg)

*Image source: AWS News Blog [4].*

### Step 3: Start the migration

S3 Tables validates the source table, generates a compatibility report, and begins copying data.

During migration, the destination table is **read-only**, meaning it can only be read. Once migration is complete, the destination table becomes **read-write**, meaning it supports both reading and writing.

![Monitor the migration of a Hive table to Iceberg](https://d2908q01vomqb2.cloudfront.net/da4b9237bacccdf19c0760cab7aec4a8359010b0/2026/10/12/2026-s3-tables-hive-to-iceberg-3.jpg)

*Image source: AWS News Blog [4].*

### Step 4: Query the data

You can work with the data through direct integrations with **Amazon Athena** and **Amazon SageMaker Unified Studio** in the AWS console, or use tools compatible with the **Apache Iceberg REST API**.

Reference: [S3 Tables migration documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-migrate.html).

## 💻 Sample code using the AWS CLI

In addition to the Console, AWS also supports operations through the **AWS CLI**, **AWS SDKs**, or **Agent Toolkit for AWS**.

Below is the sample code from the original article. The commands create a JSON configuration file and then call the API through the `start-table-migration` command.

> Values such as the account ID, IAM role ARN, bucket, KMS key, database, table, and schema are examples; replace them with settings appropriate for your actual environment before running the commands.

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

*Code source: AWS News Blog [4].*

To make the configuration easier to read, I have broken down its main components as follows:

| Component | Meaning |
| --- | --- |
| `roleARN` | The IAM role used for migration. |
| `destinationTable` | Destination table details, including the table bucket, namespace, table name, encryption, storage class, schema, and sort order. |
| `source` | The source table in the AWS Glue Data Catalog. |
| `dataMigration` | Maintenance settings and policies for handling incompatible data. |
| `migrationOutput` | The location for storing compatibility reports and migration results. |

In this example, AWS uses **Parquet**, **Iceberg format version 2**, **Zstandard** compression, a target file size of **512 MB**, and the snapshot management settings specified in the JSON. This is an illustrative configuration that needs to be checked against the source table and usage requirements. [4]

## 🔄 Synchronizing changes before completion

After the initial copy, the source table may continue to change.

To bring the destination table up to date with those changes, the article instructs users to run `sync-table-migration`. This can be done multiple times before completing the migration; each sync only copies what has changed.

The workflow is described as:

**Start → Sync → Complete**

The **Sync** step can be repeated multiple times before proceeding to **Complete**. [4]

## 🤖 Support from coding agents and Agent Toolkit for AWS

For those using coding agents such as [Kiro](https://kiro.dev/), the article introduces the `migrating-hive-tables-to-s3-tables` skill in **Agent Toolkit for AWS**.

This skill provides step-by-step guidance with safety guardrails for agents, supporting the following tasks:

- Analyzing existing IAM policies and recommending equivalent policies for the destination.
- Validating the migration plan before starting.
- Executing APIs through the **Start → Sync → Complete** workflow.
- Monitoring progress and guiding data validation and cutover.

Reference: [The skill's README on GitHub](https://github.com/aws/agent-toolkit-for-aws/tree/main/skills/specialized-skills/analytics-skills/migrating-hive-tables-to-s3-tables).

## 🌏 Supported Regions at launch

According to the announcement on **November 4, 2026**, managed migration is available in:

| Area | Regions |
| --- | --- |
| US East | N. Virginia, Ohio |
| US West | Oregon |
| Asia Pacific | Mumbai, Singapore, Sydney, Tokyo |
| Europe | Frankfurt, Ireland, Stockholm |

At launch, the feature supports Hive tables registered in AWS Glue, with data stored in Parquet format, where the source and destination are in the **same AWS account and the same Region**. [4]

## 🛠️ Tools for learning and troubleshooting

The article also introduces **AWS MCP Server** and **plugins** that can be used with AI tools for tasks such as calling APIs, searching documentation, finding supported Region information, or troubleshooting.

- [AWS MCP Server](https://docs.aws.amazon.com/agent-toolkit/latest/userguide/getting-started-aws-mcp-server.html)
- [Plugins](https://docs.aws.amazon.com/agent-toolkit/latest/userguide/plugins.html)

Users can try the feature in the [S3 console](https://console.aws.amazon.com/s3) and provide feedback through [AWS re:Post for Amazon S3](https://repost.aws/tags/TADSTjraA0Q4-a1dxk6eUYaw/amazon-simple-storage-service) or their usual AWS Support channels.

## 💬 My perspective

After reading the article, what stands out to me is that AWS is helping reduce the amount of work users need to build themselves when migrating existing Hive tables to Iceberg.

For someone still learning like me, this is also an example that helps clarify the relationship between data storage, metadata, table formats, and query tools. A migration also involves compatibility, data synchronization, and the point at which the system switches to the new table.

If I were to do a lab, I would like to try it with a small table, compare the structure and query results before and after migration, and then observe how synchronization works when new data is added to the source table.

**Have any of you migrated from Hive to Iceberg? In practice, which part is usually the most challenging: schema, data types, partitioning, access permissions, or cutover?**

I would love to hear everyone's experiences and feedback 😊

## 📎 References

1. **AWS — Amazon S3 Tables:**  
   https://aws.amazon.com/s3/features/tables/

2. **Apache Software Foundation — Apache Hive:**  
   https://hive.apache.org/

3. **Apache Software Foundation — Apache Iceberg:**  
   https://iceberg.apache.org/

4. **Channy Yun — AWS News Blog, November 4, 2026:**  
   *Amazon S3 Tables now offer managed migration of Apache Hive tables to Apache Iceberg*  
   https://aws.amazon.com/blogs/aws/amazon-s3-tables-now-offer-managed-migration-of-apache-hive-tables-to-apache-iceberg/

5. **AWS Documentation — Migrating tables to S3 Tables:**  
   https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-migrate.html