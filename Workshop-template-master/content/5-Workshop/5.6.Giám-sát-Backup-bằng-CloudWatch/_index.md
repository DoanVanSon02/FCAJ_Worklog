---
title: "Monitor AWS Backup with Amazon CloudWatch"
date: 2026-01-01
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

### Overview

After the Amazon EC2 and Amazon S3 Backup Jobs complete successfully, I continue by adding a monitoring mechanism to the system using **Amazon CloudWatch**.

AWS Backup is responsible for performing Backup and Restore operations, while Amazon CloudWatch is used to track metrics related to the status of Backup Jobs and Recovery Points.

At this point, the system not only supports:

```text
Backup
   ↓
Recovery Point
   ↓
Restore
```

but also includes a monitoring mechanism:

```text
AWS Backup
     ↓
Backup Metrics
     ↓
Amazon CloudWatch
     ↓
CloudWatch Alarm
```

Adding CloudWatch helps monitor Backup activities instead of only performing backups without a mechanism to observe their status.

In a real-world environment, if a Backup Job does not complete as expected and no monitoring system is in place, the administrator may not detect the issue in time.

AWS Backup provides metrics through the namespace:

```text
AWS/Backup
```

These metrics can be distinguished by dimensions such as:

```text
BackupVaultName
ResourceType
```

In the workshop, I use:

```text
BackupVaultName = BackupLab-Vault
```

and monitor two resource types separately:

```text
ResourceType = EC2

ResourceType = S3
```

Some important metrics used include:

```text
NumberOfBackupJobsCreated
NumberOfBackupJobsRunning
NumberOfBackupJobsCompleted
NumberOfBackupJobsFailed
NumberOfRecoveryPointsCompleted
```

In the lab, the EC2 and S3 Backup Jobs have completed successfully, so I use:

```text
NumberOfBackupJobsCompleted
```

to test the following flow:

```text
AWS Backup
     ↓
Backup Job
     ↓
Metric
     ↓
Amazon CloudWatch
     ↓
CloudWatch Alarm
```

Separate CloudWatch Alarms are created for:

```text
Amazon EC2

Amazon S3
```

with the test condition:

```text
NumberOfBackupJobsCompleted >= 1
```

The purpose of the Alarms in the workshop is to confirm that CloudWatch can receive metrics from AWS Backup, evaluate the configured condition, and change the Alarm state.

The system's overall monitoring flow after configuration is:

```text
                   AWS Backup
                       │
                       │ AWS/Backup Metrics
                       ▼
                 Amazon CloudWatch
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
      EC2 Backup Metric    S3 Backup Metric
             │                   │
             ▼                   ▼
         EC2 Alarm           S3 Alarm
```

In a real-world environment, the metric:

```text
NumberOfBackupJobsFailed
```

would be more suitable for alerting when a Backup Job fails.

However, within the scope of the workshop, I do not deliberately cause a properly functioning Backup Job to fail. Instead, I use the `NumberOfBackupJobsCompleted` metric to demonstrate that the **Metric → Alarm** mechanism works correctly.

After completing this section, the system will include all the following components:

```text
Backup
   ↓
Recovery Point

Monitoring
   ↓
CloudWatch Alarm

Restore
   ↓
To be tested in the next section
```

#### Contents

1. [Monitor AWS Backup Metrics](5.6.1.theo-dõi-aws-backup-metrics/)
2. [Create CloudWatch Alarms for EC2 and S3](5.6.2.tạo-cloudwatch-alarm-cho-ec2-và-s3/)