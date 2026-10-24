---
title: "Monitor AWS Backup Metrics"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.6.1. </b> "
---

## Monitor AWS Backup Metrics

After the Amazon EC2 and Amazon S3 Backup Jobs complete, I use **Amazon CloudWatch** to monitor the metrics sent by AWS Backup.

I navigate to:

```text
AWS Management Console
→ CloudWatch
→ Metrics
→ All metrics / Classic metrics
```

Then select the namespace:

```text
AWS/Backup
```

Here, CloudWatch displays the metrics sent by AWS Backup.

![AWS Backup Metrics for EC2 and S3 in Amazon CloudWatch](/FCAJ_Worklog/images/5/5.6.1/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>AWS Backup Metrics for EC2 and S3 in Amazon CloudWatch</b>
</p>

---

### Select the Backup Vault and Resource Type

All Backup Jobs in the previous section use the Backup Vault:

```text
BackupLab-Vault
```

Therefore, I focus on metrics with the dimension:

```text
BackupVaultName = BackupLab-Vault
```

Then distinguish the resources using:

```text
ResourceType = EC2

ResourceType = S3
```

This classification allows me to monitor the Backup status of each resource type separately within the same Backup Vault.

The monitoring flow can be described as follows:

```text
BackupLab-Vault
       │
       ├── ResourceType = EC2
       │
       └── ResourceType = S3
```

---

### Metrics Used

Some important metrics for the workshop include:

| Metric | Meaning |
| --- | --- |
| `NumberOfBackupJobsCreated` | Number of Backup Jobs created by AWS Backup |
| `NumberOfBackupJobsRunning` | Number of Backup Jobs currently running |
| `NumberOfBackupJobsCompleted` | Number of completed Backup Jobs |
| `NumberOfBackupJobsFailed` | Number of Backup Jobs with the Failed status |
| `NumberOfRecoveryPointsCompleted` | Number of successfully created Recovery Points |

After the On-demand Backups for EC2 and S3 complete, I can monitor the corresponding metrics.

For Amazon EC2:

```text
EC2
├── NumberOfBackupJobsCreated
├── NumberOfBackupJobsRunning
├── NumberOfBackupJobsCompleted
└── NumberOfRecoveryPointsCompleted
```

For Amazon S3:

```text
S3
├── NumberOfBackupJobsCreated
├── NumberOfBackupJobsRunning
├── NumberOfBackupJobsCompleted
└── NumberOfRecoveryPointsCompleted
```

The appearance of these metrics in CloudWatch indicates that AWS Backup activities have been recorded by the Monitoring system.

![AWS Backup Metrics for EC2 and S3 in Amazon CloudWatch](/FCAJ_Worklog/images/5/5.6.1/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>AWS Backup Metrics for EC2 and S3 in Amazon CloudWatch</b>
</p>

![AWS Backup Metrics for EC2 and S3 in Amazon CloudWatch](/FCAJ_Worklog/images/5/5.6.1/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>AWS Backup Metrics for EC2 and S3 in Amazon CloudWatch</b>
</p>

The images should show:

```text
Namespace:
AWS/Backup

BackupVaultName:
BackupLab-Vault

ResourceType:
EC2 / S3
```

and metrics such as:

```text
NumberOfBackupJobsCompleted
NumberOfRecoveryPointsCompleted
```

---

### Meaning of `NumberOfBackupJobsCompleted`

The metric:

```text
NumberOfBackupJobsCompleted
```

indicates the number of Backup Jobs that AWS Backup has successfully completed.

In Section 5.5, both EC2 and S3 have:

```text
Backup Job
Status = COMPLETED
```

Therefore, CloudWatch can record the corresponding metric.

Verification flow:

```text
On-demand Backup
       ↓
Backup Job
       ↓
COMPLETED
       ↓
NumberOfBackupJobsCompleted
       ↓
Amazon CloudWatch
```

This metric is used in the workshop to confirm that **CloudWatch actually receives data from AWS Backup**.

---

### The `NumberOfBackupJobsFailed` Metric

In a real-world environment, one of the more important metrics is:

```text
NumberOfBackupJobsFailed
```

This metric indicates the number of Backup Jobs with the status:

```text
FAILED
```

The alerting flow can be implemented as follows:

```text
Backup Job
     ↓
FAILED
     ↓
NumberOfBackupJobsFailed >= 1
     ↓
CloudWatch Alarm
     ↓
Alert the administrator
```

This metric is important in real-world systems because it helps detect when a Backup Job does not complete successfully.

In the current workshop, all Backup Jobs have completed with the status:

```text
COMPLETED
```

so the following condition has not occurred:

```text
NumberOfBackupJobsFailed > 0
```

Instead, in the next section, I use:

```text
NumberOfBackupJobsCompleted
```

to test the mechanism:

```text
AWS Backup Metric
       ↓
Amazon CloudWatch
       ↓
CloudWatch Alarm
```

This approach allows me to test the Monitoring functionality without changing the Backup configurations that are working properly.