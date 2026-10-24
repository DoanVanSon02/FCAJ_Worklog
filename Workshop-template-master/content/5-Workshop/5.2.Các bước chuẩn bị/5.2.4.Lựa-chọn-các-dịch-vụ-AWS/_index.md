---
title: "AWS service selection"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.2.4. </b> "
---

### AWS service selection

Before starting the system deployment, I identify the AWS services that suit the objectives of the workshop.

The **Backup & Recovery system for a Web Server on AWS** uses the following main services:

* Amazon EC2
* Amazon EBS
* Amazon S3
* AWS Identity and Access Management (IAM)
* AWS Backup
* Amazon CloudWatch

Each service has a specific role in building the system, storing data, backing up, restoring, and monitoring the system.

---

#### Amazon EC2

**Amazon Elastic Compute Cloud (Amazon EC2)** is used to deploy a Web Server running Apache.

I chose Amazon EC2 because the workshop requires an actual server that can:

* Run an installed Web Server.
* Create test data.
* Attach additional EBS Volumes.
* Access Amazon S3.
* Simulate situations where data is modified or an incident occurs.
* Perform Backup and Restore using AWS Backup.

EC2 is suitable for the lab because it allows clear observation of the Web Server and data status **before and after recovery**.

In the workshop architecture, EC2 serves as the main processing component and runs the Web application.

---

#### Amazon EBS

**Amazon Elastic Block Store (Amazon EBS)** is used as a Data Volume attached to EC2.

Instead of storing all data only on the EC2 Root Volume, I use an additional, separate EBS Volume and mount it at:

```text
/data
```

This EBS Volume is used to store the data that needs to be protected during the workshop.

Separating the Data Volume helps more clearly simulate application data that needs to be backed up. When performing an EC2 Restore, it is possible to directly verify whether the EBS Volume and data files have been restored to their state before the incident.

---

#### Amazon S3

**Amazon Simple Storage Service (Amazon S3)** is selected to store the application's object data.

The reasons for choosing Amazon S3 include:

* No need to deploy a storage server yourself.
* It is a storage service managed by AWS.
* It can scale with the amount of data.
* It supports **Versioning**.
* It is **supported by AWS Backup**.
* It is suitable for storing application files and objects.

In the workshop, **Versioning** is enabled for the S3 Bucket to support the management of object versions and meet the requirements for using AWS Backup for Amazon S3.

The S3 Bucket is kept **private**, with the following setting enabled:

```text
Block Public Access = Enabled
```

Internet users do not access the S3 Bucket directly.

EC2 will access data in S3 through an IAM Role instead of using an Access Key stored directly on the server.

---

#### AWS Identity and Access Management (IAM)

**AWS Identity and Access Management (IAM)** is used to manage access permissions between components in the system.

The workshop uses two IAM Roles for two different purposes.

The IAM Role:

```text
BackupLab-EC2-S3-Role
```

is attached to EC2 so that the Web Server can access Amazon S3 according to the granted permissions.

For EC2's access permissions to S3, I use a custom IAM Policy:

```text
BackupLab-EC2-S3-Policy
```

This Policy provides only the Amazon S3 permissions necessary for the lab instead of granting full S3 administrative permissions.

The second IAM Role is:

```text
AWSBackupDefaultServiceRole
```

This Role is used by AWS Backup to perform **Backup** and **Restore** operations on protected resources.

Using IAM Roles helps limit the storage of Access Keys directly on EC2 and supports applying the **Least Privilege** principle in the system.

---

#### AWS Backup

**AWS Backup** is the central component of the workshop and is used to manage the data backup and recovery process.

I chose AWS Backup instead of managing snapshots manually because this service allows centralized management of the Backup & Recovery process.

The main workflow can be described as follows:

```text
Backup Plan
     ↓
Backup Job
     ↓
Backup Vault
     ↓
Recovery Point
     ↓
Restore
```

In this workflow:

* **Backup Plan** defines the backup method and schedule.
* **Backup Job** represents a backup process being performed.
* **Backup Vault** is used to manage backups.
* **Recovery Point** represents a backup of a resource at a specific point in time.
* **Restore** uses a Recovery Point to restore a resource when an incident occurs.

AWS Backup can protect both **EC2/EBS and Amazon S3**, aligning with the objective of building a unified Backup & Recovery process for different types of data in the workshop.

---

#### Amazon CloudWatch

**Amazon CloudWatch** is used to monitor AWS Backup activities.

In the workshop, CloudWatch is used to:

* Monitor metrics related to AWS Backup.
* Observe the operational status of Backup Jobs.
* Configure CloudWatch Alarms to support monitoring.
* Help detect problems with Backup activities.

Adding Amazon CloudWatch allows the system not only to perform **Backup and Restore** but also to track the operational status of the backup process.

CloudWatch therefore serves as the **Monitoring** component in the workshop architecture.

---

In summary, the services selected for the workshop have the following roles:

| AWS service | Role in the system |
| --- | --- |
| **Amazon EC2** | Deploy the Web Server |
| **Amazon EBS** | Store EC2 block data |
| **Amazon S3** | Store object data |
| **AWS IAM** | Manage access permissions between services |
| **AWS Backup** | Centrally manage Backup and Restore |
| **Amazon CloudWatch** | Monitor Backup activities |

The services above work together to form the **Backup & Recovery system for a Web Server on AWS**, in which EC2, EBS, and S3 are the resources to be protected; IAM controls access permissions; AWS Backup performs backup and recovery; and CloudWatch handles monitoring.