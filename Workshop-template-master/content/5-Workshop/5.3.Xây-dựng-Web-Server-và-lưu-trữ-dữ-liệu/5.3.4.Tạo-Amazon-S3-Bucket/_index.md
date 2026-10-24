---
title: "Creating an Amazon S3 Bucket"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.3.4. </b> "
---

## Creating an Amazon S3 Bucket

In addition to data on EC2/EBS, the workshop uses **Amazon S3** to store object data.

S3 serves as the second data source protected by AWS Backup. This allows the lab to test Backup & Recovery on both:

```text
Compute + Block Storage
EC2 / EBS

and

Object Storage
Amazon S3
```

I navigate to:

```text
AWS Management Console
→ Amazon S3
→ Create bucket
```

---

### Configuring the Bucket

In the **Bucket type** section, I select:

```text
General purpose
```

The Bucket name follows the convention:

```text
aws-backup-lab-s3-<unique-suffix>
```

Since S3 Bucket names must be unique across AWS, the actual suffix may vary depending on which names are available at the time of creation.

![Creating an S3 Bucket](/FCAJ_Worklog/images/5/5.3.4/001.png?featherlight=false&width=70pc)

The Region used:

```text
US East (N. Virginia)
us-east-1
```

---

### Configuring Object Ownership

In the **Object Ownership** section, I keep the configuration:

```text
ACLs disabled
Bucket owner enforced
```

ACLs are not required in the workshop architecture because access permissions are managed through IAM.

Disabling ACLs makes the permissions model simpler and easier to control.

---

### Configuring Block Public Access

In the **Block Public Access settings for this bucket** section, I keep:

```text
Block all public access
= Enabled
```

The S3 Bucket in the workshop is accessed only by EC2 and AWS Backup through appropriate IAM permissions.

Internet users do not need direct access to the S3 Bucket.

Therefore, the bucket is not configured for public access.

This is also an important security measure in the workshop architecture.

![](/FCAJ_Worklog/images/5/5.3.4/002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.3.10: S3 Bucket configured to block public access</b>
</p>

---

### Enabling Versioning

![Versioning](/FCAJ_Worklog/images/5/5.3.4/003.png?featherlight=false&width=70pc)


Select:

```text
Enable
```

Result:

```text
Bucket Versioning:
Enabled
```

Versioning helps retain multiple versions of the same object and is an important configuration for protecting S3 data in the lab.

<p align="center">
  <b>Figure 5.3.11: Enabling Versioning for the S3 Bucket</b>
</p>

---

### Creating Test Data on S3

In the Bucket, I create the prefix:

```text
application-data/
```

Then create the file:

```text
s3-version.txt
```

with the content:

```text
AWS Backup Recovery Lab - S3 Version 1
```

The file is uploaded to:

```text
application-data/s3-version.txt
```

The structure after uploading:

```text
S3 Bucket
│
└── application-data/
    │
    └── s3-version.txt
```

---

### Why place data under `application-data/`?

Instead of granting permissions on all objects in the bucket, in the following IAM section, I will restrict EC2 permissions to the prefix:

```text
application-data/*
```

Restricting permissions by prefix clearly illustrates the principle of **Least Privilege**.

EC2 is granted permissions only on the data area required for the workshop rather than the entire S3 Bucket.

---

### Checking the Object

After uploading, I open:

```text
application-data/
```

and confirm that the file:

```text
s3-version.txt
```

exists.

The file content:

```text
AWS Backup Recovery Lab - S3 Version 1
```

![Checking the object](/FCAJ_Worklog/images/5/5.3.4/004.png?featherlight=false&width=70pc)


<p align="center">
  <b>Figure 5.3.12: Test data stored on Amazon S3</b>
</p>

---

### Results

After completing this section, the S3 Bucket should have the following state:

```text
Bucket = Created

Region = us-east-1

Block Public Access = Enabled

ACL = Disabled

Versioning = Enabled

application-data/
└── s3-version.txt
```

Amazon S3 now contains the initial test data.