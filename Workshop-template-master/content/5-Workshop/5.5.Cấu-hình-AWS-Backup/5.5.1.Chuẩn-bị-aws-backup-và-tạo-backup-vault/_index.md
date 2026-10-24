---
title: "Prepare AWS Backup and Create a Backup Vault"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

## Prepare AWS Backup and Create a Backup Vault

Before configuring the Backup Plan, I check the prerequisites to ensure AWS Backup can protect the workshop resources.

The two main resource groups used are:

```text
Amazon EC2
Amazon S3
```

Then, I create a dedicated Backup Vault to store and manage the lab's Recovery Points.

---

### Check the Resource Types Used with AWS Backup

I navigate to:

```text
AWS Backup
→ Settings
→ Service opt-in
```

and check the Resource Types used in the workshop:

```text
Amazon EC2 = Enabled
Amazon S3  = Enabled
```

AWS Backup allows you to manage the resource types used in each Region.

In this workshop, I use EC2 and S3 to represent two groups of data that need protection:

```text
Amazon EC2
├── Root EBS
└── Data EBS

Amazon S3
└── application-data/
```

Checking Service opt-in helps ensure AWS Backup can manage the resource types used in the lab.

![Check EC2 and S3](/FCAJ_Worklog/images/5/5.5.1/0001.png?featherlight=false&width=70pc)
<p align="center">
  <b>Figure 5.5.1: Check Amazon EC2 and Amazon S3 in AWS Backup Service opt-in</b>
</p>

---

### Check the S3 Prerequisites Before Backup

Before continuing, I check the S3 Bucket again at:

```text
Amazon S3
→ Bucket
→ Properties
```

and confirm:

```text
Bucket Versioning = Enabled
```

Versioning is an important configuration for protecting S3 data with AWS Backup.

The Bucket also retains the previously configured security settings:

```text
Block Public Access = Enabled

ACLs = Disabled
```

Enabling Versioning does not change the S3 Bucket's security policy.

![Check Versioning](/FCAJ_Worklog/images/5/5.5.1/0006.png?featherlight=false&width=70pc)
<p align="center">
  <b>Check Versioning = enabled</b>
</p>

---

### Create a Backup Vault

After checking the prerequisites, I navigate to:

```text
AWS Backup
→ Vaults
→ Create new vault
```

![Create a Backup Vault](/FCAJ_Worklog/images/5/5.5.1/0002.png?featherlight=false&width=70pc)
<p align="center">
  <b>Create a Backup Vault</b>
</p>

For Vault name, I use:

```text
BackupLab-Vault
```

Vault type:

```text
Backup vault
```

Encryption key:

```text
Default
```

![Create a Backup Vault](/FCAJ_Worklog/images/5/5.5.1/0003.png?featherlight=false&width=70pc)
<p align="center">
  <b>Create a Backup Vault</b>
</p>

The Tags used are:

```text
Project     = AWS-Backup-Recovery-Lab
Environment = Lab
```

After reviewing the configuration, I select:

```text
Create backup vault
```

![Create a Backup Vault](/FCAJ_Worklog/images/5/5.5.1/0004.png?featherlight=false&width=70pc)
<p align="center">
  <b>Create a Backup Vault</b>
</p>

---

### Why Is a Backup Vault Needed?

A Backup Vault is used to manage the Recovery Points created by AWS Backup.

The workflow can be described as follows:

```text
EC2 / S3
   ↓
AWS Backup
   ↓
BackupLab-Vault
   ↓
Recovery Points
```

A **Recovery Point** represents the backed-up state of a resource at a specific point in time.

When an incident occurs, the corresponding Recovery Point will be selected to perform the Restore process.

Therefore, the Backup Vault serves as the place where AWS Backup manages backups for later recovery.

---

### Why Use a Dedicated Vault?

AWS Backup may have default Backup Vaults.

However, in this workshop, I create a dedicated vault:

```text
BackupLab-Vault
```

to manage the lab's Recovery Points.

Using a dedicated Vault helps:

```text
Separate the lab's Recovery Points
        ↓
Easier management
        ↓
Easier verification
        ↓
Easier Cleanup
```

This approach also helps avoid confusion between the workshop's Recovery Points and other backups that may exist in the same AWS Account.

---

### Why Not Enable Vault Lock?

AWS Backup supports **Vault Lock** to increase protection for Recovery Points.

However, in this workshop, I **do not enable Vault Lock**.

The reason is that the current environment is used only for learning and testing through the following workflow:

```text
Create a Recovery Point
      ↓
Restore
      ↓
Cleanup
```

Using a strict locking mechanism in a short lab may make deleting Recovery Points and cleaning up resources after completion more complicated.

Within the scope of this workshop, enabling Vault Lock does not directly contribute to the main objective of testing the Backup and Restore process.

Similarly, I do not use:

```text
Logically Air-Gapped Vault
```

because the lab only performs basic Backup & Recovery within the same AWS Account and Region.

![Create a Backup Vault](/FCAJ_Worklog/images/5/5.5.1/0005.png?featherlight=false&width=70pc)
<p align="center">
  <b>Create a Backup Vault</b>
</p>

<p align="center">
  <b>Figure 5.5.2: Create BackupLab-Vault to Store Recovery Points</b>
</p>

---

### Expected Results

After completing this section, the Backup Vault should have the following status:

```text
Backup Vault = BackupLab-Vault

Encryption = AWS Backup default KMS key

Vault Lock = Disabled

Recovery Points = 0
```

At this point:

```text
Recovery Points = 0
```

because no Backup Jobs have been performed yet.

`BackupLab-Vault` is now ready to be used in the Backup Plan in the next section.