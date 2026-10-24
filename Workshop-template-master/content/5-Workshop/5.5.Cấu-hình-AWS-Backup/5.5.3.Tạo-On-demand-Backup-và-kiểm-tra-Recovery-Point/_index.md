---
title: "Create an On-demand Backup and Check Recovery Points"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---

## Create an On-demand Backup and Check Recovery Points

After completing the Backup Plan, the system can run automatic scheduled Backups.

However, waiting until:

```text
00:30
```

to perform testing would make the workshop take more time.

Therefore, I use **On-demand Backup** to create Recovery Points immediately.

On-demand Backup does not replace the Backup Plan.

The two mechanisms coexist:

```text
Backup Plan
→ Scheduled backups

On-demand Backup
→ Immediate backups when needed
```

On-demand Backup is used in the lab to check the system before proceeding to the incident simulation scenario.

---

### Check the Data Before Backup

Before running the Backup, I check the current data state again.

On EBS:

```bash
cat /data/version.txt
cat /data/customer.txt
cat /data/website-data/app-data.txt
```

Results:

```text
AWS Backup Recovery Lab - Version 1
Customer data before backup
Important website data
```

On Amazon S3:

```text
application-data/
├── s3-version.txt
└── ec2-version.txt
```

The main file used for testing is:

```text
s3-version.txt
```

with the following content:

```text
AWS Backup Recovery Lab - S3 Version 1
```

This is the **Before Incident** state that I want to save in the Recovery Point.

![Data State Before Performing the Backup](/images/5/5.5.3/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Data State Before Performing the Backup</b>
</p>

---

### Create an On-demand Backup for EC2

I navigate to:

```text
AWS Backup
→ Protected resources / Create on-demand backup
```

![Create an On-demand Backup for EC2](/images/5/5.5.3/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an On-demand Backup for EC2</b>
</p>

In the **Resource type** section, I select:

```text
EC2
```

Resource:

```text
backup-lab-web-server
```

Backup window:

```text
Create backup now
```

Backup Vault:

```text
BackupLab-Vault
```

Retention:

```text
7 days
```

IAM Role:

```text
AWSBackupDefaultServiceRole
```

After reviewing the configuration, I select:

```text
Create on-demand backup
```

![Create an On-demand Backup for EC2](/images/5/5.5.3/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an On-demand Backup for EC2</b>
</p>

![Create an On-demand Backup for EC2](/images/5/5.5.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an On-demand Backup for EC2</b>
</p>

---

### Monitor the EC2 Backup Job

After creating the On-demand Backup, I navigate to:

```text
AWS Backup
→ Jobs
→ Backup jobs
```

The Backup Job may progress through the following statuses:

```text
CREATED
   ↓
RUNNING
   ↓
COMPLETED
```

I only continue the lab when:

```text
Status = COMPLETED
```

The `COMPLETED` status indicates that AWS Backup has successfully created the EC2 Recovery Point.

![EC2 Backup Job Completed with the COMPLETED Status](/images/5/5.5.3/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 Backup Job Completed with the COMPLETED Status</b>
</p>

---

### Create an On-demand Backup for Amazon S3

After the EC2 Backup Job completes, I continue by creating a separate On-demand Backup for Amazon S3.

Resource type:

```text
S3
```

Resource:

```text
aws-backup-lab-s3-162918102026
```

Configuration:

```text
Create backup now

Backup Vault:
BackupLab-Vault

Retention:
7 days

IAM Role:
AWSBackupDefaultServiceRole
```

After reviewing the configuration, I select:

```text
Create on-demand backup
```

![Create an On-demand Backup for s3](/images/5/5.5.3/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an On-demand Backup for s3</b>
</p>

![Create an On-demand Backup for S3](/images/5/5.5.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an On-demand Backup for S3</b>
</p>

---

### Monitor the S3 Backup Job

The S3 Backup Job may take longer than the EC2 Backup Job, especially during the first Backup.

While the Backup is running, I do not:

```text
Delete objects

Modify the data used for testing

Create an additional duplicate Backup Job
```

I continue monitoring at:

```text
AWS Backup
→ Jobs
→ Backup jobs
```

until:

```text
S3
Status = COMPLETED
```

Only after the S3 Backup Job completes do I proceed to check the Recovery Points.

![Create an On-demand Backup for S3](/images/5/5.5.3/0008.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an On-demand Backup for S3</b>
</p>

---

### Check Protected Resources

After the first Backup Jobs complete, I navigate to:

```text
AWS Backup
→ Protected resources
```

The system should display the protected resources:

```text
Amazon EC2

Amazon S3
```

This indicates that AWS Backup has created Recovery Points corresponding to the workshop resources.

![EC2 and S3 Appear in the Protected Resources List](/images/5/5.5.3/0009.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 and S3 Appear in the Protected Resources List</b>
</p>

---

### Check Recovery Points in the Backup Vault

I continue by navigating to:

```text
AWS Backup
→ Backup vaults
→ BackupLab-Vault
```

Before performing the Backup:

```text
Recovery Points = 0
```

After a successful Backup, `BackupLab-Vault` should contain Recovery Points for:

```text
Amazon EC2

Amazon S3
```

A Recovery Point can be understood as:

> **The backed-up state of a resource at a specific point in time.**

AWS Backup stores and manages these Recovery Points in the Backup Vault.

The Recovery Points created in this step will be the data source used when performing Restore in a later section.

![EC2 and S3 Recovery Points Stored in BackupLab-Vault](/images/5/5.5.3/0010.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 and S3 Recovery Points Stored in BackupLab-Vault</b>
</p>