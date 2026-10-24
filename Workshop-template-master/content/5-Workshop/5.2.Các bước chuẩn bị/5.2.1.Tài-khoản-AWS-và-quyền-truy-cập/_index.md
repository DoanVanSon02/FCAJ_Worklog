---
title: "AWS account and access permissions"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.2.1. </b> "
---

### AWS account and access permissions

The workshop requires an **AWS Account** that can create and manage resources in the following services:

* **Amazon EC2 and Amazon EBS**
* **Amazon S3**
* **AWS Identity and Access Management (IAM)**
* **AWS Backup**
* **Amazon CloudWatch**

The account used for the workshop must have sufficient permissions to create and manage the resources required during deployment, including:

* EC2 Instance
* EBS Volume
* S3 Bucket
* IAM Role and IAM Policy
* Backup Vault
* Backup Plan
* CloudWatch Alarm

When building the system, I **do not use an Access Key stored directly on EC2**. Instead, access permissions between services are granted through **IAM Roles**.

This approach helps limit the storage of credentials directly on the server and better aligns with security principles when working with AWS services.

For the EC2 Web Server, access to Amazon S3 will be provided through the IAM Role:

```text
BackupLab-EC2-S3-Role
```