---
title: "Resource naming and tagging conventions"
date: 2026-01-01
weight: 6
chapter: false
pre: " <b> 5.2.6. </b> "
---

### Resource naming and tagging conventions

To make the resources in the workshop easy to identify, manage, and clean up after completion, I use a consistent convention for **resource names** and **Tags**.

Naming and tagging resources from the outset helps reduce confusion between resources, especially when the same AWS Account contains multiple EC2 Instances, EBS Volumes, IAM Roles, Backup Plans, or other resources.

---

#### Resource naming conventions

The main resources in the workshop are named as follows:

| Resource | Name |
| --- | --- |
| EC2 Instance | `backup-lab-web-server` |
| Security Group | `backup-lab-web-sg` |
| EBS Data Volume | `backup-lab-data-volume` |
| IAM Policy | `BackupLab-EC2-S3-Policy` |
| IAM Role | `BackupLab-EC2-S3-Role` |
| Backup Vault | `BackupLab-Vault` |
| Backup Plan | `BackupLab-Daily-Plan` |
| Backup Rule | `Daily-Backup-Rule` |
| Resource Assignment | `BackupLab-Resources` |
| CloudWatch Alarm | `BackupLab-BackupCompleted-Test-Alarm` |

These names are used consistently throughout the workshop to make it easy to identify the function of each resource.

For example:

```text
backup-lab-web-server
```

is used for the EC2 Instance running the Web Server.

Meanwhile:

```text
BackupLab-Vault
```

is used as the Backup Vault to manage the Recovery Points created by AWS Backup.

---

#### Amazon S3 Bucket naming convention

For Amazon S3, bucket names must be unique, so I use the following structure:

```text
aws-backup-lab-s3-<unique-suffix>
```

In which:

```text
<unique-suffix>
```

is an additional value that helps prevent the bucket name from duplicating the names of existing buckets.

For example:

```text
aws-backup-lab-s3-abc123
```

The actual bucket name may vary depending on name availability at the time the resource is created.

---

#### Tagging conventions

In addition to naming resources, I use **Tags** to identify resources belonging to the same workshop.

Resources that support Tags will use:

```text
Project = AWS-Backup-Recovery-Lab
Environment = Lab
```

In which:

* **Project** is used to identify resources belonging to the Backup & Recovery project.
* **Environment** is used to identify this as a Lab environment.

For example, an EC2 Instance can be assigned the following Tags:

```text
Name        = backup-lab-web-server
Project     = AWS-Backup-Recovery-Lab
Environment = Lab
```

The `Name` Tag helps directly identify the EC2 Instance's name in the AWS Management Console.

The `Project` and `Environment` Tags help distinguish workshop resources from other resources in the same AWS Account.

---

#### Benefits of using naming and tagging conventions

Using consistent resource names and Tags makes the workshop easier to manage.

When I need to inspect a resource, I can quickly identify which component of the system it belongs to.

For example:

```text
backup-lab-web-server
```

indicates that this is the EC2 Instance for the Web Server.

```text
backup-lab-data-volume
```

indicates that this is the EBS Data Volume used to store data.

```text
BackupLab-Vault
```

indicates that this is the Backup Vault used in the Backup & Recovery system.

In addition, Tags help identify resources belonging to the same project when inspection or cleanup is needed after completing the workshop.

As a result, I can easily identify resources with:

```text
Project = AWS-Backup-Recovery-Lab
```

and inspect them before performing the **Resource cleanup** step at the end of the workshop.