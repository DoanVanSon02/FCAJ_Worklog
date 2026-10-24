---
title : "Introduction"
date :  "`r Sys.Date()`"
weight : 1
chapter : false
pre : " <b> 5.1 </b> "
---
### Introduction to AWS Backup and Recovery

**AWS Backup** is a centralized backup service from Amazon Web Services that allows you to manage and automate data backups for multiple AWS resources from a unified interface. Through AWS Backup, administrators can create backup plans, define retention periods, manage backups, and perform recovery when an incident occurs.

In AWS Backup, backups are managed as **Recovery Points** and stored in **Backup Vaults**. A Recovery Point represents the state of a resource at a specific point in time and can be used to restore the resource when data is modified or deleted, or when the system encounters a failure.

In this workshop, AWS Backup is used to protect two main groups of resources:

* **Amazon EC2 and the EBS Volumes attached to EC2**: protect the Web Server and the data stored on the server.
* **Amazon S3**: protect the object data used by the application.

In addition to AWS Backup, the workshop also uses **AWS Identity and Access Management (IAM)** to manage access permissions between services and **Amazon CloudWatch** to monitor metrics related to the backup process.

---

### Workshop overview

In this workshop, a simple Web Server will be deployed on **Amazon EC2** and will use **Apache HTTP Server** to serve Web content to users.

An **Amazon EBS Data Volume** is attached to EC2 to store the data that needs to be protected. In addition, **Amazon S3** is used to store the application's object data.

To allow EC2 to access S3 without storing an Access Key directly on the server, the workshop uses:

```text
BackupLab-EC2-S3-Role
```

combined with:

```text
BackupLab-EC2-S3-Policy
```

This IAM Role is attached to EC2 and provides only the necessary S3 permissions.

AWS Backup uses a separate IAM Role:

```text
AWSBackupDefaultServiceRole
```

This Role allows AWS Backup to perform the necessary operations during resource **Backup and Restore**.

After the system is built, **AWS Backup** will be configured to back up EC2/EBS and S3. The backups are managed as **Recovery Points** in a **Backup Vault**.

**Amazon CloudWatch** is used to monitor AWS Backup metrics, helping to observe the status of Backup Jobs and configure Alarms for monitoring.

After the backup process is completed, the workshop will simulate an incident by modifying or deleting data. The Recovery Points created before the incident will then be used to restore the system.

---

### Architecture overview

![Architecture overview of the Backup & Recovery system on AWS](/images/5/5.1.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.1: Architecture overview of the Backup & Recovery system on AWS</b>
</p>

In the architecture above, users access the Web Server on **Amazon EC2** through the HTTP protocol. EC2 uses **Amazon EBS** to store data and uses an IAM Role to access **Amazon S3**.

**AWS Backup** backs up EC2 along with the protected EBS Volumes and the data on Amazon S3. The Recovery Points are stored in a **Backup Vault**.

When an incident occurs, AWS Backup uses a Recovery Point to restore EC2/EBS or S3 data. **Amazon CloudWatch** receives metrics from AWS Backup to support monitoring of the backup process.