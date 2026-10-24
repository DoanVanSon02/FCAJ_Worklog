---
title: "Testing Access to Amazon S3"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.4.3. </b> "
---

## Testing Access to Amazon S3

After the IAM Role is attached to EC2, I perform tests directly on the server.

The goal of this testing step is not only to confirm that **allowed** permissions work, but also to confirm that permissions **not granted** are actually denied.

Testing process:

```text
Verify the IAM Role
        ↓
List objects
        ↓
Download an object
        ↓
Upload an object
        ↓
Attempt to delete an object
        ↓
AccessDenied
```

---

### Verifying That EC2 Has Received the IAM Role

First, I check AWS CLI:

```bash
aws --version
```

![Checking the AWS CLI version](/FCAJ_Worklog/images/5/5.4.3/0001.png?featherlight=false&width=70pc)

Amazon Linux 2023 usually already includes AWS CLI. If it is not installed, AWS CLI must be installed before proceeding
([See the instructions](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)).

Then run:

```bash
aws sts get-caller-identity
```

The result has the following format:

```json
{
    "UserId": "...",
    "Account": "...",
    "Arn": "arn:aws:sts::<ACCOUNT-ID>:assumed-role/BackupLab-EC2-S3-Role/..."
}
```

The important part to check is:

```text
assumed-role/BackupLab-EC2-S3-Role
```

This proves that EC2 is using the newly created IAM Role.

![Checking the AWS CLI version](/FCAJ_Worklog/images/5/5.4.3/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.4.5: EC2 using BackupLab-EC2-S3-Role</b>
</p>

---

### Testing Permission to List Data on S3

I run:

```bash
aws s3 ls s3://aws-backup-lab-s3-162918102026/application-data/
```

The result should include:

```text
s3-version.txt
```

This proves that the flow:

```text
EC2
 ↓
IAM Role
 ↓
s3:ListBucket
 ↓
Amazon S3
```

is working correctly.

---

### Testing Permission to Read Data from S3

Next, I download:

```text
s3-version.txt
```

from S3 to EC2:

```bash
aws s3 cp s3://aws-backup-lab-s3-162918102026/application-data/s3-version.txt /tmp/s3-version.txt
```

Then check the content:

```bash
cat /tmp/s3-version.txt
```

Result:

```text
AWS Backup Recovery Lab - S3 Version 1
```

This proves that the permission:

```text
s3:GetObject
```

is working correctly.

---

### Testing Permission to Upload from EC2 to S3

To test permission to write objects, I use the file created on EBS:

```text
/data/version.txt
```

and upload it to S3:

```bash
aws s3 cp /data/version.txt s3://aws-backup-lab-s3-162918102026/application-data/ec2-version.txt
```

Then check again:

```bash
aws s3 ls s3://aws-backup-lab-s3-162918102026/application-data/
```

The result should include:

```text
ec2-version.txt
s3-version.txt
```

This proves that the permission:

```text
s3:PutObject
```

is working correctly.

An additional check can be performed in the AWS Management Console at:

```text
Amazon S3
→ Bucket
→ application-data/
```

to confirm that:

```text
ec2-version.txt
```

has been created.

![Verifying that EC2 lists, reads, and uploads data to S3 through an IAM Role](/FCAJ_Worklog/images/5/5.4.3/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.4.6: EC2 reading and uploading data to Amazon S3 through an IAM Role</b>
</p>

---

### Testing the Principle of Least Privilege

Simply proving that allowed permissions work is not enough to confirm that the Policy is actually restricted.

Therefore, I perform an additional test with a permission that is **not granted**.

In the Policy, EC2 is not granted:

```text
s3:DeleteObject
```

I attempt to delete the object that was just uploaded:

```bash
aws s3 rm s3://aws-backup-lab-s3-162918102026/application-data/ec2-version.txt
```

Expected result:

```text
AccessDenied
```

or a message indicating that the Role is not allowed to perform:

```text
s3:DeleteObject
```

In this case, `AccessDenied` **is not a workshop configuration error**, but the desired result.

The test results can be summarized as follows:

```text
GetBucketLocation → Allowed  ✓

ListBucket        → Allowed  ✓

GetObject         → Allowed  ✓

PutObject         → Allowed  ✓

DeleteObject      → Denied   ✓
```

This proves that the IAM Policy does not grant permissions beyond the functions that EC2 actually needs.

![Testing DeleteObject](/FCAJ_Worklog/images/5/5.4.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.4.7: Testing Least Privilege – EC2 does not have permission to delete objects on S3</b>
</p>

---

> **Note – Do Not Use Access Keys on EC2:** In this workshop, I do not run `aws configure` to enter and store an `AWS Access Key ID` or `AWS Secret Access Key` on EC2. Instead, EC2 uses `BackupLab-EC2-S3-Role` to obtain temporary credentials. This approach avoids storing long-term credentials on the server, does not require manual Access Key management, and allows centralized permission control through the IAM Policy.

---