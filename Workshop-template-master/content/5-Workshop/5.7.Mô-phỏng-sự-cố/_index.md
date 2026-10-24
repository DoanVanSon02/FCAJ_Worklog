---
title: "Simulate Incidents"
date: 2026-01-01
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

### Overview

After completing the Backup process and confirming that Recovery Points for Amazon EC2/EBS and Amazon S3 have been successfully created, I simulate several scenarios involving data loss or modification.

The objective of this step is not to damage the system randomly, but to create **a controlled incident** to test the recovery capabilities of AWS Backup.

The system state before the incident:

```text
Website
→ Operating normally

/data/version.txt
→ AWS Backup Recovery Lab - Version 1

/data/customer.txt
→ Customer data before backup

/data/website-data/app-data.txt
→ Important website data

S3:
application-data/s3-version.txt
→ AWS Backup Recovery Lab - S3 Version 1
```

The data above was saved in the Recovery Points in Section 5.5.

In this section, I simulate three main types of incidents:

```text
Modify data
       ↓
Delete data
       ↓
Corrupt data contents
```

In addition, the Website content on EC2 is modified to make it easy to distinguish the system states before and after the incident.

The testing flow follows this sequence:

```text
Version 1 data
        ↓
      Backup
        ↓
  Recovery Point
        ↓
 Simulate incidents
        ↓
Version 2 / Deleted / Corrupted
        ↓
      Restore
        ↓
Return to the Version 1 state
```

Before creating the incident, I check the Backup Vault again to ensure that the EC2 and S3 Recovery Points exist and that the Backup Jobs have the status:

```text
COMPLETED
```

After confirming that the Backup was successful, I begin modifying the data on EBS.

The simulated scenarios include:

```text
version.txt
→ Version 1 changed to Version 2

customer.txt
→ Deleted

app-data.txt
→ Contents modified

Website
→ Changed to the INCIDENT TEST state
```

For Amazon S3, the object:

```text
application-data/s3-version.txt
```

is deleted using a normal Delete operation.

Since Versioning is enabled on the S3 Bucket, this operation creates a **Delete Marker** and causes the object to disappear from the current listing, while the previous version is retained.

After completing the simulation, there will be a clear difference between the state saved in the Recovery Points and the current system state:

```text
RECOVERY POINT
────────────────────────────
Website       = Normal
version.txt   = Version 1
customer.txt  = Exists
app-data.txt  = Normal
s3-version    = Exists


CURRENT SYSTEM
────────────────────────────
Website       = INCIDENT TEST
version.txt   = Version 2
customer.txt  = Deleted
app-data.txt  = Corrupted
s3-version    = Deleted
```

This provides the necessary conditions to test Restore capabilities in the next section.

During the incident simulation, I retain the resources required for recovery and do not perform operations that could compromise Restore capabilities, such as:

```text
✗ Terminate the original EC2 instance

✗ Format the EBS Volume

✗ Delete all S3 object versions

✗ Delete the Backup Vault

✗ Delete Recovery Points

✗ Change the AWS Backup IAM Role

✗ Create additional Backups after the incident
```

After completing Section 5.7, the system is ready to use the Recovery Points created **before the incident** to restore data in Section 5.8.

#### Contents

1. [Check Recovery Points Before Creating an Incident](5.7.1.kiểm-tra-recovery-point-trước-khi-tạo-sự-cố/)
2. [Simulate Incidents on EC2 and EBS](5.7.2.mô-phỏng-sự-cố-trên-ec2-và-ebs/)
3. [Simulate an Incident on Amazon S3](5.7.3.mô-phỏng-sự-cố-trên-amazon-s3/)