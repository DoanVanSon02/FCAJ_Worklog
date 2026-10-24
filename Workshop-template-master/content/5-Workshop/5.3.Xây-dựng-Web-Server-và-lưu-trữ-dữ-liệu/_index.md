---
title: "Building a Web Server and Storing Data"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---


### Overview

After completing the preparation steps, I begin building the system that needs to be protected in the workshop.

In this section, a **Web Server** is deployed on **Amazon EC2** and uses **Apache HTTP Server** to serve Web content. EC2 acts as the system's main server and will be used as a resource to be protected by AWS Backup in the following sections.

In addition to the EC2 Root Volume, I create a separate **Amazon EBS Data Volume** to store application data. The Data Volume is mounted at:

```text
/data
```

Separating application data from the Root Volume makes Backup and Restore testing clearer, as the state of the data files can be checked directly before and after recovery.

In addition to block data on Amazon EBS, the workshop also uses **Amazon S3** to store object data.

Data on S3 is organized under the prefix:

```text
application-data/
```

The S3 Bucket is kept private, with **Block Public Access** and **Versioning** enabled to support the Backup & Recovery process in the following sections.


The desired state after completing section 5.3 includes:

```text
EC2 Web Server = Running

Apache = Active

Website = Accessible via HTTP

EBS Data Volume = Mounted at /data

Test data on EBS = Created

S3 Bucket = Created

Block Public Access = Enabled

S3 Versioning = Enabled

Test data on S3 = Created
```

At this point, **EC2 has not yet been granted permission to access Amazon S3**.

I do not use an Access Key directly on EC2. Access permissions between EC2 and S3 will be configured through an **IAM Role** in the section:

```text
5.4. IAM Configuration
```

### Contents

1. [Launch Amazon EC2](5.3.1.khởi-tạo-amazon-ec2/)
2. [Install Apache Web Server](5.3.2.cài-đặt-apache-web-server/)
3. [Create an EBS Data Volume](5.3.3.tạo-ebs-data-volume/)
4. [Create an Amazon S3 Bucket](5.3.4.tạo-amazon-s3-bucket/)