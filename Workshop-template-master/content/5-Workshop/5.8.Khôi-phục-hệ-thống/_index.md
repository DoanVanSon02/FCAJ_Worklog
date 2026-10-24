---
title: "System Recovery and Validation"
date: 2026-01-01
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

### Overview

After completing the incident simulation in section 5.7, the current state of the system differs from the state saved in the Recovery Point.

The comparison is as follows:

```text
RECOVERY POINT
────────────────────────────
Website       = Operating normally
version.txt   = Version 1
customer.txt  = Exists
app-data.txt  = Normal data
s3-version    = Exists


CURRENT SYSTEM AFTER INCIDENT
────────────────────────────
Website       = INCIDENT TEST
version.txt   = Version 2
customer.txt  = Deleted
app-data.txt  = Corrupted
s3-version    = Deleted
```

In this section, I use the Recovery Point created **before the incident occurred** to restore EC2/EBS and Amazon S3 data.

The ultimate goal is to verify whether the system can return to its original state.

The workshop recovery flow:

```text
Recovery Point
      ↓
AWS Backup
      ↓
Restore Job
   ┌───────┴────────┐
   ↓                ↓
EC2 + EBS          S3
   ↓                ↓
Validation       Validation
   └───────┬────────┘
           ↓
      Recovery Pass
```

For Amazon EC2, AWS Backup does not directly overwrite the EC2 instance affected by the incident.

Instead, the Recovery Point is used to create a new EC2 Instance along with the corresponding EBS Volumes.

Therefore, after the Restore process, the following may coexist:

```text
ORIGINAL EC2
→ State after the Incident

RESTORED EC2
→ State from the Recovery Point
```

This approach allows me to easily compare the affected system directly with the restored system.

For Amazon S3, the object:

```text
application-data/s3-version.txt
```

was deleted in section 5.7.

During Restore, I use the Amazon S3 Recovery Point to perform:

```text
Item-level Restore
```

instead of restoring the entire Bucket.

The S3 recovery flow:

```text
S3 Source Bucket
       │
       │ Backup
       ▼
   AWS Backup
       │
       ▼
Recovery Point
       │
       │ Restore
       ▼
S3 Source Bucket
```

After the Restore Jobs are completed, I not only check the status:

```text
COMPLETED
```

in AWS Backup but also directly verify the data.

The data to be confirmed includes:

```text
version.txt
→ Version 1

customer.txt
→ Reappears

app-data.txt
→ Important website data

Website
→ AWS Backup & Recovery Lab

s3-version.txt
→ Reappears
```

This helps clearly distinguish between:

```text
Restore Job completed
```

and:

```text
Data actually restored correctly
```

The final results are checked across three states:

```text
Before Incident
       ↓
After Incident
       ↓
After Restore
```

The goal is to demonstrate that the data after Restore returns to the exact state saved in the Recovery Point before the incident.

The end-to-end workshop flow now becomes:

```text
BUILD
  ↓
BACKUP
  ↓
MONITOR
  ↓
INCIDENT
  ↓
RESTORE
  ↓
VALIDATE
```

If all data and services return to the expected state, the Backup & Recovery process is considered:

```text
PASS
```

#### Contents

1. [Restore Amazon EC2 and EBS](5.8.1.khôi-phục-amazon-ec2-và-ebs/)
2. [Restore Amazon S3 Data](5.8.2.Khôi-phục-dữ-liệu-amazon-s3/)
3. [Validate Data After Recovery](5.8.3.kiểm-tra-dữ-liệu-sau-khi-khôi-phục/)