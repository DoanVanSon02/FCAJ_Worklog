---
title: "Security principles"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5.2.5. </b> "
---

### Applied security principles

From the preparation stage, I define several basic security principles for the system to reduce risks during the deployment and use of AWS services.

The main principles applied in the workshop include:

* Do not hard-code Access Keys on EC2.
* Use IAM Roles to grant permissions between services.
* Apply the Least Privilege principle.
* Limit the use of full administrative permissions for services.
* Keep Amazon S3 private.
* Open only the necessary network ports on EC2.

---

#### Do not hard-code Access Keys

In the workshop, EC2 accesses Amazon S3 through an **IAM Role** instead of storing credentials directly on the server.

Specifically, EC2 uses:

```text
BackupLab-EC2-S3-Role
```

to receive the necessary permissions to access Amazon S3.

I do not store information such as:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

directly on EC2.

This approach helps reduce the risk of exposing credentials while using the server and better aligns with AWS's permissions mechanism.

---

#### Apply the Least Privilege principle

**Least Privilege** is the principle of granting users or services only the permissions truly necessary to complete their tasks.

In the workshop, EC2 is not granted full administrative permissions for Amazon S3.

Instead, I use a custom IAM Policy:

```text
BackupLab-EC2-S3-Policy
```

This Policy grants only the necessary permissions for the S3 Bucket used in the lab.

Some of the permissions used include:

```text
s3:ListBucket
s3:GetObject
s3:GetObjectVersion
s3:PutObject
```

These permissions allow EC2 to:

* List objects in the bucket.
* Read objects from Amazon S3.
* Read object versions.
* Upload objects from EC2 to Amazon S3.

The workshop does not grant broad permissions such as:

```text
AdministratorAccess
```

or:

```text
AmazonS3FullAccess
```

to EC2 just to complete the lab.

Limiting permissions helps reduce the scope of impact if a resource or application experiences a security incident.

---

#### Use IAM Roles between services

The services in the workshop use **IAM Roles** to perform the necessary operations.

For EC2:

```text
BackupLab-EC2-S3-Role
```

is used to access Amazon S3.

For AWS Backup:

```text
AWSBackupDefaultServiceRole
```

is used to perform Backup and Restore operations on protected resources.

Separating IAM Roles helps ensure that each service uses only the permissions required for its function.

---

#### Limit public resources

The Amazon S3 Bucket in the workshop is kept **private**.

The setting:

```text
Block Public Access = Enabled
```

is enabled to reduce the possibility of the bucket or objects accidentally being exposed publicly on the Internet.

In this architecture, S3 is not used as a public website.

EC2 accesses data in S3 through an IAM Role, so there is no need to allow public access to the bucket.

---

#### Restrict access ports on EC2

The EC2 Security Group opens only the ports necessary for the workshop.

The ports used include:

```text
HTTP 80
SSH 22
```

In which:

* **HTTP 80** is used to allow users to access the Web Server.
* **SSH 22** is used only when access to and administration of EC2 are required.

For SSH, access sources should be restricted instead of allowing access from the entire Internet when unnecessary.

Opening only the necessary ports helps reduce the Web Server's attack surface.

---

#### Summary of security principles

The security principles applied in the workshop can be summarized as follows:

| Principle | Application in the workshop |
| --- | --- |
| **Do not hard-code Access Keys** | EC2 uses an IAM Role to access S3 |
| **Least Privilege** | Grant only the necessary S3 permissions |
| **Separate permissions between services** | EC2 and AWS Backup use separate IAM Roles |
| **Limit public resources** | Enable Block Public Access on S3 |
| **Restrict network ports** | EC2 opens only HTTP 80 and SSH 22 when necessary |

Applying these principles helps the system meet basic security requirements while limiting overly broad permissions or the exposure of credentials during the workshop deployment.