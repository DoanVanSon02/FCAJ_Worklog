---
title: "Proposal"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

## 2.1. Summary

My project is **“Building a Backup & Recovery System for a Web Server on AWS”**.

The objective of the project is to build a Web Server on AWS, back up data, monitor the Backup process, and test recovery capabilities when data is modified or deleted.

The system uses the following main services:

- **Amazon EC2** to deploy the Web Server.
- **Amazon EBS** to store server data.
- **Amazon S3** to store object data.
- **AWS IAM** to manage access permissions between services.
- **AWS Backup** to centrally manage Backup and Restore.
- **Amazon CloudWatch** to monitor Backup activities.

The main workflow of the project:

```text
Build the system
        ↓
Back up data
        ↓
Monitoring
        ↓
Simulate incidents
        ↓
Restore
        ↓
Verify results
```

---

## 2.2. Problem Statement

During Web Server operation, data may be modified or lost due to accidental actions, application errors, or system failures.

Without an appropriate Backup, data recovery may take a long time or may not be possible.

In addition, a system's data may exist across different types of storage, such as:

```text
Amazon EBS
Amazon S3
```

Therefore, the project requires a solution that enables centralized Backup management and can Restore data when an incident occurs.

In addition to creating Backups, the system also needs a Monitoring mechanism to track the status of Backup Jobs.

---

## 2.3. Proposed Solution

I propose building an Apache Web Server on **Amazon EC2**.

EC2 uses **Amazon EBS** to store server data and **Amazon S3** to store some object data.

AWS IAM is used to manage access permissions. EC2 accesses S3 through an **IAM Role** instead of storing an Access Key directly on the server, while applying the **Least Privilege** principle.

**AWS Backup** is used to perform Backups for EC2/EBS and S3. Recovery Points are managed in a Backup Vault and are used to Restore data when an incident occurs.

**Amazon CloudWatch** is used to track AWS Backup metrics and create Alarms for monitoring.

After the Backup is completed, I will deliberately simulate several incidents, such as:

- Modifying file contents.
- Deleting files on EBS.
- Corrupting application data.
- Modifying Website content.
- Deleting objects on S3.

Then, the Recovery Point created before the incident will be used to restore the system.

Overall architecture:

```text
EC2 + EBS ──────┐
                │
                ▼
            AWS Backup
                │
S3 ─────────────┘
                │
                ▼
          Backup Vault
                │
                ▼
        Recovery Points
                │
                ▼
             Restore


AWS Backup
    │
    ▼
CloudWatch
    │
    ▼
  Alarm
```

---

## 2.4. Expected Results

After completing the project, I expect to achieve the following results:

- The Web Server runs on Amazon EC2.
- Data is stored on EBS and Amazon S3.
- EC2 accesses S3 using an IAM Role without hard-coding an Access Key.
- AWS Backup successfully creates Recovery Points for EC2/EBS and S3.
- CloudWatch can monitor Backup activities.
- Scenarios involving modified or deleted data can be simulated.
- EC2/EBS can be Restored from a Recovery Point.
- Objects on S3 can be recovered.
- Data after Restore returns to its state before the incident.

The final results must demonstrate the following workflow:

```text
BACKUP
   ↓
RECOVERY POINT
   ↓
INCIDENT
   ↓
RESTORE
   ↓
VALIDATION
   ↓
PASS
```

Detailed implementation, configuration of each service, illustrative images, and test results will be presented in **Section 5 – Workshop**.