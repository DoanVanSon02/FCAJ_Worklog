---
title: "Simulate an Incident on Amazon S3"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.7.3. </b> "
---

## Simulate an Incident on Amazon S3

After simulating incidents on EC2 and EBS, I continue by creating a data loss scenario on Amazon S3.

The object used in the test is:

```text
application-data/s3-version.txt
```

Before the incident, this object contains:

```text
AWS Backup Recovery Lab - S3 Version 1
```

This state was saved by AWS Backup in the Recovery Point created in the previous section.

---

### Delete the Current Object

I navigate to:

```text
Amazon S3
→ Bucket
→ application-data/
```

![Simulate an Incident on S3](/images/5/5.7.3/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Simulate an Incident on S3</b>
</p>

Then select the object:

```text
s3-version.txt
```

and select:

```text
Delete
```

At the confirmation step, I perform a normal object deletion.

After completion, `s3-version.txt` no longer appears in the current object listing of the folder:

```text
application-data/
```

![Simulate an Incident on S3](/images/5/5.7.3/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Simulate an Incident on S3</b>
</p>

![Simulate an Incident on S3](/images/5/5.7.3/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Simulate an Incident on S3</b>
</p>

The state at this point:

```text
Before Incident:
s3-version.txt = exists

After Incident:
s3-version.txt = no longer visible
```

---

### Why Only Perform a Normal Object Deletion?

The workshop's S3 Bucket has:

```text
Versioning = Enabled
```

Therefore, when performing a normal `DELETE` operation without specifying a Version ID, Amazon S3 does not permanently delete the previous version of the object.

Instead, S3 creates a:

```text
Delete Marker
```

and makes the Delete Marker the current version of the object.

The state before deletion can be illustrated as follows:

```text
Before Delete

s3-version.txt
└── Version 1
    Current version
```

After performing Delete:

```text
After Delete

s3-version.txt
├── Delete Marker
│   Current version
│
└── Version 1
    Previous version
```

Since the Delete Marker becomes the current version, in the normal object listing:

```text
s3-version.txt
```

will appear to have been deleted.

A normal request to read the object will also behave as if the object no longer exists.

![Simulate an Incident on S3](/images/5/5.7.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Simulate an Incident on S3</b>
</p>

---

### Do Not Permanently Delete the Versions

During the incident simulation, I **do not enable Show versions and then delete all Version IDs** of:

```text
s3-version.txt
```

The reason is that directly deleting a specific Version ID can permanently delete that version.

The workshop's objective is only to simulate the scenario:

```text
A user accidentally deletes an object
```

rather than destroy the entire S3 Versioning history.

Therefore, I only perform:

```text
Simple Delete
       ↓
Delete Marker
       ↓
Object is no longer visible
```

This approach creates a controlled data loss scenario while retaining the components necessary for testing Restore.

---

### Check the State After Deletion

After performing the Delete operation, I return to:

```text
Amazon S3
→ Bucket
→ application-data/
```

and confirm that:

```text
s3-version.txt
```

no longer appears in the normal object listing.

The current state can be represented as:

```text
application-data/
└── s3-version.txt
    → No longer visible
```

![Simulate an Incident on S3](/images/5/5.7.3/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.7.4: s3-version.txt No Longer Appears After Simulating the Delete Operation</b>
</p>

---

### Check the Delete Marker Using Show versions

Since Versioning is enabled on the S3 Bucket, I can enable:

```text
Show versions
```

to check the object's version history.

The following can then be observed:

```text
s3-version.txt
├── Delete Marker
│   → Current version
│
└── Version 1
    → Previous version
```

This shows that the object has not been permanently deleted from the entire Versioning history.

![Simulate an Incident on S3](/images/5/5.7.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.7.5: A Delete Marker Is Created After Deleting an Object in an S3 Bucket with Versioning Enabled</b>
</p>

S3 Versioning provides an additional layer of protection against accidental deletion.

However, the main objective of the workshop is still to use:

```text
AWS Backup Recovery Point
```

to perform data recovery according to the established architecture.

The two mechanisms can be distinguished as follows:

```text
S3 Versioning
→ Stores object versions in S3

AWS Backup
→ Creates and manages Recovery Points
→ Supports the Backup & Restore process
```

In the workshop, these two mechanisms are used together.

After this step, the S3 data state differs from the state saved in the Recovery Point:

```text
Recovery Point:
s3-version.txt = exists

Current S3:
s3-version.txt = no longer visible
```

This is the state needed to test Amazon S3 recovery capabilities in the later Restore section.