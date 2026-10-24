---
title: "Create a Backup Plan and Assign Resources"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

## Create a Backup Plan and Assign Resources

After creating the Backup Vault, I continue by creating a **Backup Plan** to define the backup schedule, Recovery Point retention period, and resources that need protection.

The Backup Vault answers the question:

```text
Where are backups managed?
```

While the Backup Plan answers:

```text
What schedule do backups follow,
and how long are they retained?
```

---

### Create a Backup Plan

I navigate to:

```text
AWS Backup
→ Backup plans
→ Create backup plan
```

![Create a Backup Plan](/FCAJ_Worklog/images/5/5.5.2/0001.png?featherlight=false&width=70pc)
<p align="center">
  <b>Create a Backup Plan</b>
</p>

Then select:

```text
Build a new plan
```

Backup Plan name:

```text
BackupLab-Daily-Plan
```

Backup Rule name:

```text
Daily-Backup-Rule
```

I configure the Backup Rule as follows:

| Setting | Value Used |
| --- | --- |
| Backup Vault | `BackupLab-Vault` |
| Backup frequency | Daily |
| Start time | 00:30 |
| Time zone | Asia/Saigon (UTC+07) |
| Start within | 8 hours |
| Complete within | 7 days |
| Continuous Backup / PITR | Disabled |
| Cold storage | Disabled |
| Total retention | 7 days |
| Backup indexes | Disabled |
| Scan mode | None |
| Destination copy | Not configured |

**Start within** defines the time window during which the Backup Job is allowed to start.

**Complete within** defines the maximum time allowed for the Backup Job to complete after it has started.

![Create a Backup Plan](/FCAJ_Worklog/images/5/5.5.2/0002.png?featherlight=false&width=70pc)
<p align="center">
  <b>Create a Backup Plan</b>
</p>

![Create a Backup Plan](/FCAJ_Worklog/images/5/5.5.2/0003.png?featherlight=false&width=70pc)
<p align="center">
  <b>Create a Backup Plan</b>
</p>

![Create a Backup Plan](/FCAJ_Worklog/images/5/5.5.2/0004.png?featherlight=false&width=70pc)
<p align="center">
  <b>Create a Backup Plan</b>
</p>

![Create a Backup Plan](/FCAJ_Worklog/images/5/5.5.2/0005.png?featherlight=false&width=70pc)
<p align="center">
  <b>Create a Backup Plan</b>
</p>

---

### Why Use Daily Backup?

The workshop system has only one Web Server and a small amount of data.

I use:

```text
Frequency = Daily
```

instead of Hourly because the workshop's objective is to demonstrate a basic scheduled Backup policy.

In a real-world system, the Backup frequency should be selected based on:

```text
RPO
+
Rate of data change
+
Storage costs
```

However, within the scope of this workshop, Daily Backup is sufficient to demonstrate automated backups.

---

### Why Retain Recovery Points for 7 Days?

I configure:

```text
Retention = 7 days
```

because the lab only needs to retain Recovery Points for a short period to perform the following workflow:

```text
Backup
   ↓
Simulate incidents
   ↓
Restore
   ↓
Validation
```

Retaining Backups for a longer period does not add value to the test but may increase storage costs.

---

### Why Not Use Cold Storage?

Cold Storage is more suitable for Backups that need to be retained for a long time.

The workshop only uses:

```text
Retention = 7 days
```

so I keep:

```text
Cold storage = Disabled
```

Moving short-term Backups to a long-term storage tier does not align with the lab's objectives and data lifecycle.

---

### Point-in-time recovery

I keep:

```text
Point-in-time recovery
= Disabled
```

The workshop's objective is to perform **scheduled recovery** from a specific Recovery Point.

---

### Configure S3 Backup

In the S3 Advanced settings, I do not use Backup ACL.

The S3 Bucket was previously configured with:

```text
ACLs disabled
Bucket owner enforced
```

Therefore, including ACLs in the Backup is unnecessary.

I use:

```text
Back up ACLs = Disabled
Back up object tags = Disabled
```

Object Tags are also not used in the test scenario, so they do not need to be protected within the scope of the workshop.

---

### Recovery Point Tag

I add the Tag:

```text
Project = AWS-Backup-Recovery-Lab
```

to the Recovery Point.

This Tag helps identify Backups belonging to the workshop and makes resource management more convenient.

After completing the configuration, I select:

```text
Create plan
```

---

### Assign Resources to the Backup Plan

The Backup Plan currently only defines:

```text
When are backups performed?

How long are backups retained?

Which Vault stores the backups?
```

but does not yet define:

```text
Which resources are backed up?
```

Therefore, I continue by selecting:

```text
Assign resources
```

Resource Assignment name:

```text
BackupLab-Resources
```

---

### Configure the IAM Role for AWS Backup

In the IAM Role section, I select:

```text
Create default role
```

AWS Backup will create or use:

```text
AWSBackupDefaultServiceRole
```

This Role allows AWS Backup to perform the necessary operations when backing up and restoring resources.

![Configure the IAM Role for AWS Backup](/FCAJ_Worklog/images/5/5.5.2/0006.png?featherlight=false&width=70pc)
<p align="center">
  <b>Configure the IAM Role for AWS Backup</b>
</p>

This Role is entirely different from:

```text
BackupLab-EC2-S3-Role
```

which was created in Section 5.4.

---

### Distinguish Between the Two IAM Roles

The IAM Role for EC2:

```text
BackupLab-EC2-S3-Role
        ↓
Used by EC2
        ↓
Access S3
```

While the IAM Role for AWS Backup:

```text
AWSBackupDefaultServiceRole
        ↓
Used by AWS Backup
        ↓
Backup / Restore
```

I do not use `BackupLab-EC2-S3-Role` as the Service Role for AWS Backup because the two Roles serve different purposes.

---

### Check S3 Backup Permissions

`AWSBackupDefaultServiceRole` uses the policies required for AWS Backup to perform Backup and Restore operations.

The general policies include:

```text
AWSBackupServiceRolePolicyForBackup

AWSBackupServiceRolePolicyForRestores
```

For Amazon S3, AWS Backup also uses policies related to S3 Backup and Restore:

```text
AWSBackupServiceRolePolicyForS3Backup

AWSBackupServiceRolePolicyForS3Restore
```

Because the workshop includes both Backup and Restore of S3 data, I check the Role to ensure the necessary permissions have been configured before continuing.

![AWSBackupDefaultServiceRole Used by AWS Backup](/FCAJ_Worklog/images/5/5.5.2/0009.png?featherlight=false&width=70pc)

<p align="center">
  <b>AWSBackupDefaultServiceRole Used by AWS Backup</b>
</p>

---

### Select Resources to Back Up

Under **Resource selection**, I select:

```text
Include specific resource types
```

instead of:

```text
Include all resource types
```

The reason is that the workshop only needs to protect the specific resources included in its design.

I select:

```text
Amazon EC2
→ backup-lab-web-server

Amazon S3
→ aws-backup-lab-s3-162918102026
```

![Assign EC2 and S3 to BackupLab-Daily-Plan](/FCAJ_Worklog/images/5/5.5.2/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Assign EC2 and S3 to BackupLab-Daily-Plan</b>
</p>

---

### Why Not Select EBS Separately?

The EC2 instance in the workshop has:

```text
Root EBS
+
Data EBS /data
```

However, I **do not assign the EBS Volume as an independent resource** in the Backup Plan.

In the lab:

```text
EC2
├── Root EBS
└── Data EBS
        ↓
   Back up EC2
```

already meets the objective of protecting the entire Web Server.

If I also select Data EBS as a separate resource:

```text
EC2 Backup
+
Separate EBS Backup
```

this would create an additional independent Backup workflow that is unnecessary within the scope of the workshop.

---

### Exclude Resources and Refine Selection

I leave the following field empty:

```text
Exclude specific resource IDs
```

because only the specific resources already selected are included in the Backup Plan.

I also do not use:

```text
Refine selection using tags
```

Tag-based selection is more suitable when a system has many resources and needs to automatically protect resources based on Tags.

The workshop has only one EC2 instance and one S3 Bucket, so selecting them directly makes the configuration easier to verify and manage.

After completing the configuration, I select:

```text
Assign resources
```

![Assign EC2 and S3 to BackupLab-Daily-Plan](/FCAJ_Worklog/images/5/5.5.2/0008.png?featherlight=false&width=70pc)

<p align="center">
  <b>Assign EC2 and S3 to BackupLab-Daily-Plan</b>
</p>

After this step, `BackupLab-Daily-Plan` has the complete Backup Rule and all the resources that need protection.