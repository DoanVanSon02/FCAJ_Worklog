---
title: "Creating and Attaching an IAM Role to EC2"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.4.2. </b> "
---

## Creating and Attaching an IAM Role to EC2

This section includes three consecutive steps:

```text
Create a Custom IAM Policy
          ↓
Create an IAM Role for EC2
          ↓
Attach the IAM Role to EC2
```

---

### Creating a Custom IAM Policy

I navigate to:

```text
AWS Management Console
→ IAM
→ Policies
→ Create policy
→ Select a service → S3
```

![Creating a policy](/images/5/5.4.2/0001.png?featherlight=false&width=70pc)

Then select:

```text
JSON
```

to create the Policy myself.

The Policy is configured as follows:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowBucketInformation",
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation"
      ],
      "Resource": "arn:aws:s3:::aws-backup-lab-s3-162918102026"
    },
    {
      "Sid": "AllowListApplicationData",
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::aws-backup-lab-s3-162918102026",
      "Condition": {
        "StringLike": {
          "s3:prefix": [
            "application-data",
            "application-data/*"
          ]
        }
      }
    },
    {
      "Sid": "AllowApplicationDataObjects",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::aws-backup-lab-s3-162918102026/application-data/*"
    }
  ]
}
```

![Creating a policy](/images/5/5.4.2/0013.png?featherlight=false&width=70pc)

### Explanation of the Permissions in the Policy

The Policy is divided into three different permission groups.

#### Permission to Retrieve Bucket Information

```text
s3:GetBucketLocation
```

allows AWS CLI to identify the Bucket's Region.

This permission is applied at the Bucket level:

```text
arn:aws:s3:::arn:aws:s3:::aws-backup-lab-s3-162918102026
```

#### Permission to List Objects

```text
s3:ListBucket
```

allows EC2 to list the contents of the Bucket.

However, this permission is not granted without restrictions.

I use the condition:

```json
"Condition": {
  "StringLike": {
    "s3:prefix": [
      "application-data",
      "application-data/*"
    ]
  }
}
```

to restrict the scope to:

```text
application-data/
```

This helps avoid granting permission to list unnecessary data outside the area used by the application.

#### Permissions to Perform Operations on Objects

The two permissions:

```text
s3:GetObject
s3:PutObject
```

are applied only to:

```text
arn:aws:s3:::arn:aws:s3:::aws-backup-lab-s3-162918102026/application-data/*
```

Therefore, EC2 does not automatically have permission to read or write every object in the Bucket.

---

### Naming the IAM Policy

I use the name:

```text
BackupLab-EC2-S3-Policy
```

Description:

```text
Allow the Backup Lab EC2 instance to access
application-data in the lab S3 bucket.
```

After reviewing the Policy, I select:

```text
Create policy
```

<p align="center">
  <b>Figure 5.4.1: Creating a Custom IAM Policy that allows EC2 to access Amazon S3</b>
</p>

![Creating a policy](/images/5/5.4.2/0003.png?featherlight=false&width=70pc)

Check again after creation:

![Creating a policy](/images/5/5.4.2/0004.png?featherlight=false&width=70pc)

---

### Creating an IAM Role for EC2

After the Custom Policy is created, I proceed to create an IAM Role so that EC2 can use this Policy.

I navigate to:

```text
IAM
→ Roles
→ Create role
```

Under:

```text
Trusted entity type
```

select:

```text
AWS service
```

Service:

```text
EC2
```

Then select:

```text
Next
```

![Creating an IAM Role for EC2](/images/5/5.4.2/0005.png?featherlight=false&width=70pc)

---

### Why Is the Trusted Entity EC2?

An IAM Role needs to specify which service is allowed to use the Role.

In this case:

```text
Amazon EC2
     │
     │ Assume Role
     ▼
BackupLab-EC2-S3-Role
```

Therefore, the Trusted Entity must be Amazon EC2.

The corresponding Trust Policy has the following structure:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

I do not need to enter this Trust Policy manually when creating the Role using:

```text
AWS service → EC2
```

because AWS automatically creates the corresponding Trust Policy.

---

### Attaching the Policy to the Role

At the **Add permissions** step, I search for:

```text
BackupLab-EC2-S3-Policy
```

and select this Policy.

I do not use:

```text
AmazonS3FullAccess
```

or:

```text
AdministratorAccess
```

because these two Policies provide a broader scope of permissions than the lab requires.

![Attaching the Policy to the Role](/images/5/5.4.2/0006.png?featherlight=false&width=70pc)

The Role name used:

```text
BackupLab-EC2-S3-Role
```

Description:

```text
IAM role for the Backup Lab EC2 instance
to access application data in Amazon S3.
```

Then select:

```text
Create role
```

![Attaching the policy to the Role](/images/5/5.4.2/0007.png?featherlight=false&width=70pc)

![Attaching the policy to the Role](/images/5/5.4.2/0008.png?featherlight=false&width=70pc)

Check after creation is complete:

![Attaching the policy to the Role](/images/5/5.4.2/0009.png?featherlight=false&width=70pc)

---

### Attaching the IAM Role to EC2

The IAM Role has been created, but EC2 does not automatically use this Role.

I navigate to:

```text
EC2
→ Instances
```

select:

```text
backup-lab-web-server
```

then:

```text
Actions
→ Security
→ Modify IAM role
```

![Attaching the policy to the Role](/images/5/5.4.2/0010.png?featherlight=false&width=70pc)

Under:

```text
IAM role
```

select:

```text
BackupLab-EC2-S3-Role
```

then select:

```text
Update IAM role
```

![Attaching the policy to the Role](/images/5/5.4.2/0011.png?featherlight=false&width=70pc)

After this step, the permissions flow becomes:

```text
BackupLab-EC2-S3-Policy
          ↓
BackupLab-EC2-S3-Role
          ↓
backup-lab-web-server
          ↓
      Amazon S3
```

After completing this step, EC2 has the IAM Role attached and is ready for Amazon S3 access testing in the next section.