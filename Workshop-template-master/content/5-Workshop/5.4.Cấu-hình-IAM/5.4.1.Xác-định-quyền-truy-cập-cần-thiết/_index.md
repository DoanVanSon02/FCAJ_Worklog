---
title: "Identifying the Required Access Permissions"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.4.1. </b> "
---

### Identifying the Required Access Permissions

Before creating an IAM Policy, I identify the exact operations that EC2 needs to perform on Amazon S3.

In the workshop, EC2 only needs to work with data under:

```text
application-data/
```

in the S3 Bucket created in the previous section.

The required permissions include:

| Permission | Purpose |
| --- | --- |
| `s3:GetBucketLocation` | Identify the S3 Bucket's Region |
| `s3:ListBucket` | List the necessary objects in the Bucket |
| `s3:GetObject` | Read or download objects from S3 |
| `s3:PutObject` | Upload objects from EC2 to S3 |

I **do not grant**:

```text
s3:DeleteObject
```

because EC2 does not need permission to delete objects to fulfill the lab's functions.

Identifying permissions before creating the Policy helps avoid granting permissions beyond actual requirements.

For example, a simpler approach would be to attach:

```text
AmazonS3FullAccess
```

to EC2.

However, this Managed Policy provides more permissions than necessary, including the ability to interact with multiple resources and perform actions that the workshop's application does not use.

In the current architecture:

```text
EC2 only needs
     ↓
one specific S3 Bucket
     ↓
one specific prefix
     ↓
one specific set of actions
```

Therefore, I choose to create a **Custom IAM Policy** instead of using `AmazonS3FullAccess`.

This is how the principle of **Least Privilege** is applied to the system from the access permissions design stage.