---
title: "Restore Amazon S3 Data"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.8.2. </b> "
---

After the Amazon EC2 Restore process is completed, I continue restoring the data deleted from Amazon S3.

During the incident simulation, the object:

```text
application-data/s3-version.txt
```

was deleted from the current object list.

Before the incident occurred, this object contained:

```text
AWS Backup Recovery Lab - S3 Version 1
```

Because S3 Versioning was enabled, a normal Delete operation created a Delete Marker, causing the object to no longer appear in the current list.

In this step, I use the **AWS Backup Recovery Point** created before the incident to restore the object.

---

### Select the S3 Recovery Point

I navigate to:

```text
AWS Backup
→ Protected resources
```

Then I select the workshop's Amazon S3 Bucket.

In the Recovery Point list, I select the Backup created **before the incident occurred**.

Then I select:

```text
Restore
```

![Restore S3](/FCAJ_Worklog/images/5/5.8.2/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Restore S3</b>
</p>

It is important that the selected Recovery Point contains the following state:

```text
application-data/s3-version.txt
→ AWS Backup Recovery Lab - S3 Version 1
```

---

### Use Item-level Restore

AWS Backup allows restoring:

```text
The entire S3 Bucket
```

or:

```text
One or more specific objects
```

In the workshop, only the object:

```text
application-data/s3-version.txt
```

was deleted.

Therefore, I use:

```text
Item-level restore
```

instead of restoring the entire Bucket.

---

### Why Use Item-level Restore?

If the entire Bucket is restored, AWS Backup must process more data than necessary.

Meanwhile, the incident only affects:

```text
application-data/s3-version.txt
```

Therefore:

```text
Incident affecting one object
        ↓
Restore one object
```

is more appropriate for the scope of the test.

This approach also makes the Restore process easier to verify because only the lost data is restored.

---

### Select the Object to Restore

Under Restore settings, I select:

```text
Item level restore
```

Then I enter the S3 URI of the object to restore:

```text
s3:s3://aws-backup-lab-s3-162918102026/application-data/s3-version.txt
```

Entering the correct S3 URI helps AWS Backup identify the exact object that needs to be restored.

![Configure Item-level Restore for s3-version.txt](/FCAJ_Worklog/images/5/5.8.2/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Configure Item-level Restore for s3-version.txt</b>
</p>

---

### Select the Version

AWS Backup can restore the object version saved in the Recovery Point.

In the workshop, I only need to restore the object's state before the incident, so I use:

```text
Latest version
```

in the Recovery Point.

The goal is to return the object to the following state:

```text
AWS Backup Recovery Lab - S3 Version 1
```

---

### Restore Destination

Under Destination, I select:

```text
Restore to source bucket
```

I **do not create a new S3 Bucket**.

The Restore flow is as follows:

```text
S3 Source Bucket
       │
       │ Backup
       ▼
   AWS Backup
       │
       ▼
Recovery Point
       │
       │ Restore
       ▼
S3 Source Bucket
```

In the workshop, restoring to the Source Bucket itself is the most appropriate option because the goal is to recover the object deleted from the current application.

---

### Why Can the Object Be Restored When a Delete Marker Already Exists?

AWS Backup performs S3 Restore using a mechanism that does not destroy existing data.

In the workshop scenario:

```text
Version 1
   ↓
Delete
   ↓
Delete Marker
   ↓
Object no longer appears
   ↓
AWS Backup Restore
   ↓
Object reappears
```

After a normal Delete operation is performed, the Delete Marker becomes the Current Version.

The object therefore no longer appears in the standard list.

However, the state before the incident has been saved in the AWS Backup Recovery Point.

When performing Restore, AWS Backup can restore the object from the backed-up state.

This also explains why, in section 5.7, I only perform:

```text
Simple Delete
```

instead of permanently deleting all Version IDs of the object.

---

### Configure Encryption

Under:

```text
Restored object encryption
```

I select:

```text
Use original encryption keys (default)
```

This configuration allows AWS Backup to reuse the encryption method or Encryption Key of the source object when possible.

The workshop does not require changing the encryption method, so I keep the default configuration.

![Restore S3](/FCAJ_Worklog/images/5/5.8.2/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Restore S3</b>
</p>

---

### Configure ACL

The original S3 Bucket was configured with:

```text
ACLs disabled

Bucket owner enforced
```

Additionally, I did not enable Backup ACL during the Backup process.

Therefore, during Restore, I keep:

```text
Restore ACLs = Disabled
```

This ensures that the Restore configuration matches the original S3 Bucket settings.

---

### Restore Role

Under:

```text
Restore role
```

I select:

```text
AWSBackupDefaultServiceRole
```

This is the Service Role used by AWS Backup to perform Restore operations.

This Role needs the corresponding permissions for AWS Backup to restore Amazon S3 data.

The related Managed Policies include:

```text
AWSBackupServiceRolePolicyForRestores

AWSBackupServiceRolePolicyForS3Restore
```

![Restore S3](/FCAJ_Worklog/images/5/5.8.2/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Restore S3</b>
</p>

---

### Monitor the S3 Restore Job

After starting Restore, I navigate to:

```text
AWS Backup
→ Jobs
→ Restore jobs
```

Here, I monitor the Amazon S3 Restore Job.

The information to check:

```text
Resource type:
S3
```

and the final status:

```text
Status:
COMPLETED
```

I only proceed to verify the data after:

```text
Status = COMPLETED
```

![Amazon S3 Restore Job completed with the status COMPLETED](/FCAJ_Worklog/images/5/5.8.2/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Amazon S3 Restore Job completed with the status COMPLETED</b>
</p>

The `COMPLETED` status indicates that AWS Backup has finished restoring the object from the Recovery Point.

However, this is not the only evidence that the data has been restored correctly.

In section **5.8.3 – Validate Data After Recovery**, I will directly check:

```text
application-data/s3-version.txt
```

to confirm that the object has reappeared and contains the correct content:

```text
AWS Backup Recovery Lab - S3 Version 1
```