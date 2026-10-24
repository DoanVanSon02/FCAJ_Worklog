---
title: "Clean Up Resources"
date: 2026-01-01
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

After completing Backup, Monitoring, Restore, and Validation, I delete the resources that are no longer in use to avoid incurring additional costs.

Cleanup must be performed in the correct order because some resources depend on one another.

---

### Delete the Resource Assignment and Backup Plan

First, I navigate to:

```text
AWS Backup
→ Backup plans
→ BackupLab-Daily-Plan
```

Then I delete the Resource Assignment:

```text
BackupLab-Resources
```

Next, I delete the Backup Plan:

```text
BackupLab-Daily-Plan
```

Deleting the Backup Plan stops future automatic Backups but does not remove previously created Recovery Points.

The Cleanup flow:

```text
BackupLab-Resources
        ↓
Delete Resource Assignment
        ↓
BackupLab-Daily-Plan
        ↓
Delete Backup Plan
```

![Delete resources](/FCAJ_Worklog/images/5/5.9/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

![Delete resources](/FCAJ_Worklog/images/5/5.9/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

---

### Delete Recovery Points and the Backup Vault

Next, I navigate to:

```text
AWS Backup
→ Backup vaults
→ BackupLab-Vault
```

I delete all Recovery Points for:

```text
Amazon EC2

Amazon S3
```

After confirming:

```text
Recovery Points = 0
```

![Delete resources](/FCAJ_Worklog/images/5/5.9/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

I proceed to delete:

```text
BackupLab-Vault
```

![Delete resources](/FCAJ_Worklog/images/5/5.9/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

The required order:

```text
Recovery Points
        ↓
Delete
        ↓
Recovery Points = 0
        ↓
Delete BackupLab-Vault
```

Deleting Recovery Points first prevents the Backup Vault from being unable to be deleted because it still contains Recovery Points.

---

### Terminate Amazon EC2

After completing AWS Backup Cleanup, I move on to Amazon EC2.

I navigate to:

```text
Amazon EC2
→ Instances
```

and terminate both Instances:

```text
backup-lab-web-server-restored

backup-lab-web-server
```

Where:

```text
backup-lab-web-server
→ Original EC2

backup-lab-web-server-restored
→ EC2 created through the Restore process
```

I use:

```text
Terminate
```

instead of only:

```text
Stop
```

because the workshop is complete and the Instances are no longer needed.

If EC2 is only stopped, some related resources, such as EBS, may continue to incur costs.

![Delete resources](/FCAJ_Worklog/images/5/5.9/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

---

### Delete Remaining EBS Volumes

After the EC2 Instances are terminated, I navigate to:

```text
EC2
→ Elastic Block Store
→ Volumes
```

and check for EBS Volumes that remain in the following state:

```text
Available
```

The `Available` state indicates that the Volume is not currently attached to any EC2 Instance.

If I confirm that a Volume belongs to the workshop, for example:

```text
backup-lab-data-volume
```

I delete that Volume.

The goal is to ensure that no EBS Volumes from the lab remain and continue to incur storage costs.

![Delete resources](/FCAJ_Worklog/images/5/5.9/0006.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

---

### Delete the Amazon S3 Bucket

Next, I clean up the Amazon S3 Bucket.

Because the Bucket has:

```text
Versioning = Enabled
```

deleting only the current objects is not enough to make the Bucket completely empty.

I access the S3 Bucket and enable:

```text
Show versions
```

Then I delete all:

```text
Current versions

Previous versions

Delete markers
```

![Delete resources](/FCAJ_Worklog/images/5/5.9/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

This can be visualized as:

```text
S3 Bucket
   ↓
Show versions
   ↓
Current versions
   +
Previous versions
   +
Delete markers
   ↓
Delete
   ↓
Bucket Empty
```

Only after confirming that the Bucket is completely empty do I delete the S3 Bucket.

![Delete resources](/FCAJ_Worklog/images/5/5.9/0008.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

---

### Delete CloudWatch Alarms

Next, I navigate to:

```text
Amazon CloudWatch
→ Alarms
```

and delete the Alarms created specifically for the workshop:

```text
BackupLab-BackupCompleted-Test-Alarm

BackupLab-S3-BackupCompleted-Test-Alarm
```

After deletion, CloudWatch no longer contains the Alarms used to monitor the lab's Backup Jobs.

![Delete resources](/FCAJ_Worklog/images/5/5.9/0009.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

---

### Delete the Custom IAM Role and Policy

Then I clean up the IAM Resources that I created myself during the workshop.

The IAM Role to delete:

```text
BackupLab-EC2-S3-Role
```

![Delete resources](/FCAJ_Worklog/images/5/5.9/0010.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

The Custom IAM Policy to delete:

```text
BackupLab-EC2-S3-Policy
```

![Delete resources](/FCAJ_Worklog/images/5/5.9/0011.png?featherlight=false&width=70pc)

<p align="center">
  <b>Delete resources</b>
</p>

The order that can be followed:

```text
BackupLab-EC2-S3-Role
        ↓
Delete Role
        ↓
BackupLab-EC2-S3-Policy
        ↓
Delete Policy
```

I only delete IAM Resources created specifically for the workshop.

---

### Do Not Delete AWS Service Roles If Unsure

I do not delete AWS Service Roles or Service-Linked Roles unless I am sure they are used only for the workshop.

For example:

```text
AWSBackupDefaultServiceRole

AWSServiceRoleForBackup
```

These Roles do not incur costs simply by existing and may continue to be used by AWS for other labs or workloads.

Therefore, during Cleanup, I focus on the Custom IAM Resources that I intentionally created.

---

### Check the Status After Cleanup

After completion, I check the services again to ensure that no lab resources remain active.

| Service | Expected status |
| --- | --- |
| EC2 | No lab Instances remain |
| EBS | No lab Volumes remain |
| S3 | The lab Bucket has been deleted |
| AWS Backup | No Backup Plan remains |
| AWS Backup | No Recovery Points remain |
| AWS Backup | The Backup Vault has been deleted |
| CloudWatch | The lab Alarms have been deleted |
| IAM | The Custom Role and Policy have been deleted |

---

### Overall Cleanup Order

The entire Cleanup process is performed in the following order:

```text
Resource Assignment
        ↓
Backup Plan
        ↓
Recovery Points
        ↓
Backup Vault
        ↓
EC2 Instances
        ↓
EBS Volumes
        ↓
S3 Versions / Delete Markers
        ↓
S3 Bucket
        ↓
CloudWatch Alarms
        ↓
Custom IAM Role / Policy

This order helps reduce errors caused by dependencies between resources and makes Cleanup easier to manage.
```

---