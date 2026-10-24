---
title: "Check Recovery Points Before Creating an Incident"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.7.1. </b> "
---

### Check Recovery Points Before Creating an Incident

Before modifying or deleting any data, I check the Backup status again to ensure that the system has valid Recovery Points.

This check is very important because the Recovery Points will be used as the data source for performing Restore in the next section.

I navigate to:

```text
AWS Backup
→ Backup vaults
→ BackupLab-Vault
```

Here, I confirm that Recovery Points for the two resources:

```text
Amazon EC2

Amazon S3
```

already exist in the Backup Vault.

---

#### Check the Backup Job Status

In addition to checking the Recovery Points, I check the corresponding Backup Jobs again.

The required status is:

```text
COMPLETED
```

This demonstrates that the Backup process completed successfully before I begin simulating the incident.

The required sequence is:

```text
Backup succeeds
       ↓
Recovery Point exists
       ↓
Only then begin creating the incident
```

If data is modified or deleted **before a valid Recovery Point exists**, I will not have a suitable Backup to use for the Restore process.

![EC2 and S3 Recovery Points Exist Before the Incident Simulation](/images/5/5.7.1/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 and S3 Recovery Points Exist Before the Incident Simulation</b>
</p>

---

#### Do Not Create New Backups After the Incident

After I begin modifying the data, I do not perform another On-demand Backup.

The reason is that the current Recovery Point contains the data state:

```text
Version 1
```

Whereas after simulating the incident, the data will become:

```text
Version 2

Deleted

Corrupted
```

If I create a new Backup after this point, the new Recovery Point may contain the modified or corrupted data state itself.

In that case, the Restore test would no longer accurately reflect the original objective.

The sequence must remain:

```text
Version 1
   ↓
BACKUP
   ↓
Recovery Point
   ↓
INCIDENT
   ↓
RESTORE
```

Therefore, from the moment I begin simulating the incident, I only use the Recovery Points created **before the incident** and do not create any new Backups until the Restore and Validation processes are complete.