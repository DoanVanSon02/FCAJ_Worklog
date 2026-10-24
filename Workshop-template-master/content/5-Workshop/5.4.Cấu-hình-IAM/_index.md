---
title: "IAM Configuration"
date: 2026-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Overview

After completing the setup of EC2, EBS, and Amazon S3, the system has all the resources required to run the Web Server and store data. However, at this point, EC2 has not yet been granted permission to access Amazon S3.

In this workshop, I **do not configure an Access Key and Secret Access Key directly on EC2**. Instead, access permissions are provided through an **IAM Role** attached directly to the EC2 Instance.

This approach allows applications running on EC2 to receive **temporary credentials** from AWS instead of storing long-term credentials on the server.

The permissions flow used is as follows:

```text
BackupLab-EC2-S3-Policy
          │
          ▼
BackupLab-EC2-S3-Role
          │
          ▼
      Amazon EC2
          │
          ▼
      Amazon S3
   application-data/*
```

In this flow:

* **BackupLab-EC2-S3-Policy** defines the actions EC2 is allowed to perform on Amazon S3.
* **BackupLab-EC2-S3-Role** is the IAM Role attached to EC2.
* **Amazon EC2** uses the IAM Role to obtain temporary credentials.
* Access is restricted to data under the `application-data/` prefix in the S3 Bucket.

The goal of this section is to allow EC2 to perform the necessary operations:

```text
EC2 can:

✓ List the necessary data on S3
✓ Read objects
✓ Upload objects
```

At the same time, EC2 is not granted unnecessary permissions:

```text
EC2 cannot:

✗ Delete objects
```

The workshop also does not use:

```text
✗ Hard-coded Access Keys
✗ AmazonS3FullAccess
✗ AdministratorAccess
```

This configuration applies the principle of **Least Privilege**, which means granting EC2 only the permissions necessary to perform the lab's functions.

In section 5.4, I will identify the permissions EC2 needs, create a Custom IAM Policy, create an IAM Role, attach the Role to EC2, and perform tests to confirm that the permissions work as designed.

After completing this section, the desired access flow is:

```text
AWS IAM
   │
   ▼
BackupLab-EC2-S3-Policy
   │
   ▼
BackupLab-EC2-S3-Role
   │
   ▼
Amazon EC2
   │
   ├── ListBucket
   ├── GetObject
   └── PutObject
   │
   ▼
Amazon S3
application-data/
```

The `DeleteObject` permission is not granted. Therefore, when EC2 attempts to delete an object on S3, the expected result is:

```text
AccessDenied
```

This provides practical evidence of applying the principle of **Least Privilege** in the system.

#### Contents

1. [Identify the Required Access Permissions](5.4.1.xác-định-quyền-truy-cập-cần-thiết/)
2. [Create and Attach an IAM Role to EC2](5.4.2.tạo-và-gắn-iam-role-cho-ec2/)
3. [Test Access to Amazon S3](5.4.3.kiểm-tra-quyền-truy-cập-amazon-s3/)
4. [Check the Results](5.4.4.kiểm-tra-kết-quả/)