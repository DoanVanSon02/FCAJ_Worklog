---
title: "Tools used"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.2.3. </b> "
---

### Tools used

The workshop is primarily carried out through the **AWS Management Console** to allow direct observation of resources and facilitate each deployment step.

The tools used in the workshop include:

| Tool | Purpose |
| --- | --- |
| **AWS Management Console** | Create and manage AWS services |
| **EC2 Instance Connect** | Access the EC2 terminal |
| **Linux Shell** | Install Apache, create and verify data |
| **AWS CLI on EC2** | Verify EC2's ability to access Amazon S3 |
| **Web browser** | Test the Web Server through the HTTP protocol |
| **draw.io** | Create the system architecture diagram |

#### AWS Management Console

**AWS Management Console** is the main tool used throughout the workshop.

Through the AWS management interface, I perform tasks such as:

* Launching and managing EC2 Instances.
* Creating and configuring EBS Volumes.
* Creating and managing S3 Buckets.
* Creating IAM Roles and IAM Policies.
* Configuring Backup Vaults and Backup Plans.
* Monitoring Backup Jobs.
* Monitoring metrics and configuring Alarms in CloudWatch.

Using the AWS Management Console allows direct observation of each resource's status and makes it easy to verify the results after each configuration step.

#### EC2 Instance Connect

**EC2 Instance Connect** is used to access the EC2 Instance terminal directly from the AWS Management Console.

Through the EC2 terminal, I can perform the necessary tasks on the Web Server without storing a Private Key directly on my computer while carrying out the lab.

#### Linux Shell

After connecting to EC2, **Linux Shell** is used to perform tasks on the server, such as:

* Installing Apache HTTP Server.
* Starting and checking the status of the Web Server.
* Creating directories and data files.
* Mounting the EBS Data Volume.
* Verifying data before and after performing a Restore.

#### AWS CLI on EC2

**AWS CLI** is used on EC2 to verify access to Amazon S3 through the IAM Role attached to EC2.

AWS CLI supports checking operations such as:

* Listing objects in an S3 Bucket.
* Uploading files from EC2 to Amazon S3.
* Reading data from Amazon S3.
* Verifying EC2's access permissions to S3.

Using AWS CLI in this case helps directly verify that EC2 can access S3 through an **IAM Role** without storing an Access Key and Secret Access Key on the server.

#### Web browser

A **Web browser** is used to verify access to the Web Server after Apache is installed on EC2.

Through the EC2 Public IPv4 Address or Public DNS, it is possible to check whether the Web Server is running and returning HTTP content as expected.

#### draw.io

**draw.io** is used to create the overall system architecture diagram.

The diagram helps illustrate the relationships between the main components, such as:

* Amazon EC2.
* Amazon EBS.
* Amazon S3.
* AWS IAM.
* AWS Backup.
* Backup Vault.
* Amazon CloudWatch.


Using the tools above allows me to directly observe the process of creating, configuring, backing up, monitoring, and restoring resources throughout the workshop.