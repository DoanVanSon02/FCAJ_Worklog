---
title: "Configure AWS Backup"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

#### Overview

After completing the Web Server, EBS, Amazon S3, and IAM setup, the system has all the resources that need protection.

In this section, I use **AWS Backup** to build a centralized backup mechanism for the main workshop resources, including:

```text
Amazon EC2
├── Root EBS
└── Data EBS

Amazon S3
└── application-data/
```

Instead of manually creating individual snapshots or backups, AWS Backup allows centralized management of the entire backup and recovery process.

The main workflow used in the workshop:

```text
Backup Vault
     ↓
Backup Plan
     ↓
Resource Assignment
     ↓
Backup Job
     ↓
Recovery Point
     ↓
Restore
```

In this workflow:

* **Backup Vault** is used to manage Recovery Points.
* **Backup Plan** defines the Backup schedule and data retention period.
* **Resource Assignment** identifies the resources that need protection.
* **Backup Job** represents the process through which AWS Backup performs a backup.
* **Recovery Point** represents the backed-up state of a resource at a specific point in time.
* **Restore** uses a Recovery Point to recover resources after an incident occurs.

In this workshop, I create a dedicated Backup Vault:

```text
BackupLab-Vault
```

to manage the lab's Recovery Points.

Then, I create a Backup Plan:

```text
BackupLab-Daily-Plan
```

with the Backup Rule:

```text
Daily-Backup-Rule
```

The Backup Plan is configured to perform scheduled backups and retain Recovery Points for:

```text
7 days
```

The resources included in the Backup Plan are:

```text
Amazon EC2
→ backup-lab-web-server

Amazon S3
→ aws-backup-lab-s3-<unique-suffix>
```

AWS Backup uses the Service Role:

```text
AWSBackupDefaultServiceRole
```

to perform Backup and Restore operations on the protected resources.

This Role is different from:

```text
BackupLab-EC2-S3-Role
```

which was used in the previous IAM section to allow EC2 to access Amazon S3.

After configuring the Backup Plan, I use **On-demand Backup** to create Recovery Points immediately instead of waiting for the scheduled Backup time.

The two mechanisms are used in parallel:

```text
Backup Plan
→ Automatic scheduled backups

On-demand Backup
→ Immediate backups when testing is needed
```

Before performing the Backup, the data on EBS and S3 is kept in its initial state:

```text
Version 1
```

After the Backup Job completes successfully, the Recovery Points for EC2 and Amazon S3 will be stored in:

```text
BackupLab-Vault
```

The testing process in the workshop follows this sequence:

```text
Version 1
    ↓
BACKUP
    ↓
Recovery Point Version 1
    ↓
INCIDENT
    ↓
Version 2 / Deleted / Corrupted
    ↓
RESTORE
    ↓
Version 1
```

Following this sequence ensures that the Recovery Point is created before the incident occurs and can be used to demonstrate that the Backup & Recovery process works correctly.

#### Contents

1. [Prepare AWS Backup and Create a Backup Vault](5.5.1.chuẩn-bị-aws-backup-và-tạo-backup-vault/)
2. [Create a Backup Plan and Assign Resources](5.5.2.tạo-backup-plan-và-gán-tài-nguyên/)
3. [Create an On-demand Backup and Check Recovery Points](5.5.3.tạo-on-demand-backup-và-kiểm-tra-recovery-point/)