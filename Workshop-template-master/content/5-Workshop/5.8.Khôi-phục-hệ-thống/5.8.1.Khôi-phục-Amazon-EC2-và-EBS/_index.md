---
title: "Restoring Amazon EC2 and EBS"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.8.1. </b> "
---

## Restoring Amazon EC2 and EBS

After simulating the incident, I use the Amazon EC2 Recovery Point created **prior to the incident** to perform the Restore process.

The goal of this step is to recreate an EC2 Instance along with its Root EBS and Data EBS from the state saved in the Recovery Point.

---

### Selecting a Recovery Point

I navigate to:

```text
AWS Backup
→ Protected resources
```

Then select the Amazon EC2 resource corresponding to:

```text
backup-lab-web-server
```

From the Recovery Point list, I select the Backup created **before the incident simulation**.

Then select:

```text
Restore
```

It is important not to select a Recovery Point created after the data has been altered.

The Recovery Point to use must contain the state:

```text
Website
→ Operating normally

version.txt
→ Version 1

customer.txt
→ Exists

app-data.txt
→ Important website data
```
![Restoring EC2](/images/5/5.8.1/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>[Restoring EC2]</b>
</p>

---

### How AWS Backup Restores EC2

AWS Backup does not overwrite the malfunctioning EC2 directly.

When Restoring an EC2 Recovery Point, AWS Backup creates a new set of resources including:

```text
New EC2 Instance
+
Root EBS Volume
+
Data EBS Volume
```

Therefore, the original EC2 remains in its post-incident state.

This restoration method is suitable for the workshop because I can retain the malfunctioning EC2 to compare it directly with the Restored EC2.

---

### Restore Configuration

On the Restore screen, I verify the information:

```text
Instance type

VPC

Subnet

Security Group

Instance profile

Restore role
```

I keep the network configuration the same as the original Web Server so the Restored EC2 can operate in the same environment.

Configuration used:

```text
Instance type:
t3.micro

VPC:
VPC being used for the workshop

Subnet:
Corresponding subnet

Security Group:
backup-lab-web-sg
```

AWS Backup can use the configuration information of the protected EC2 to provide default values for:

```text
Instance Type
VPC
Subnet
Security Group
```

I double-check this information before starting the Restore to ensure the new EC2 is created in the correct workshop environment.

![Amazon EC2 Restore Configuration from Recovery Point](/images/5/5.8.1/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Amazon EC2 Restore Configuration from Recovery Point</b>
</p>

---

### Instance Profile of the Restored EC2

The initial EC2 is using the IAM Role:

```text
BackupLab-EC2-S3-Role
```

This Role is used to:

```text
EC2
 ↓
BackupLab-EC2-S3-Role
 ↓
Amazon S3
```

During the Restore process, you can either reuse the Instance Profile of the original EC2 or create the Restored EC2 without an attached Instance Profile.

Select:

```text
Restore with original IAM role
```

---

### Restore Role

In the section:

```text
Restore role
```

I select:

```text
Default
```

This is the IAM Role used by AWS Backup to perform Restore operations.

![Amazon EC2 Restore Configuration from Recovery Point](/images/5/5.8.1/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Amazon EC2 Restore Configuration from Recovery Point</b>
</p>

---

### Starting the Restore

After verifying all configurations, I select:

```text
Restore backup
```

Then navigate to:

```text
AWS Backup
→ Jobs
→ Restore jobs
```

to monitor the Restore process.

The Restore Job may sequentially transition through the states:

```text
PENDING
   ↓
RUNNING
   ↓
COMPLETED
```

I only proceed to check the EC2 after:

```text
Status = COMPLETED
```

This indicates that AWS Backup has completed the process of recreating resources from the Recovery Point.

![EC2 Restore Job completed with COMPLETED status](/images/5/5.8.1/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 Restore Job completed with COMPLETED status</b>
</p>

---

### Identifying the Restored EC2

After the Restore Job is completed, I navigate to:

```text
Amazon EC2
→ Instances
```

AWS Backup has created a new EC2 Instance.

To avoid confusion with the malfunctioning EC2, I name the new Instance:

```text
backup-lab-web-server-restored
```

At this point, the system has two Instances:

```text
backup-lab-web-server
→ Original EC2
→ Post-Incident state


backup-lab-web-server-restored
→ New EC2
→ Created from the Recovery Point
```

I check the new EC2 and verify:

```text
Instance state:
Running

Status checks:
3/3 checks passed
```

![Original EC2 and Restored EC2 coexist](/images/5/5.8.1/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Original EC2 and Restored EC2 coexist</b>
</p>

---