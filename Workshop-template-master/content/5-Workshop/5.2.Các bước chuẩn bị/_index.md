---
title: "Preparation steps"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Overview

Before deploying the **Backup & Recovery system on AWS**, it is necessary to fully prepare the working environment and identify the AWS services to be used, the deployment Region, access permissions, and resource management conventions.

Establishing consistent preparations from the outset makes the deployment process easier to follow, reduces confusion between resources, and facilitates testing, recovery, and resource cleanup after completing the workshop.

The workshop preparation covers the following main topics:

* **AWS account and access permissions:**  
  Check the AWS account and ensure that the person carrying out the workshop has sufficient permissions to create and manage EC2, EBS, S3, IAM, AWS Backup, and CloudWatch resources.

* **Region selection:**  
  Identify the Region to be used throughout the workshop so that resources are deployed in one location and are easy to manage.

* **Tools used:**  
  Prepare the necessary tools, such as AWS Management Console, EC2 Instance Connect, Linux Shell, AWS CLI, a Web browser, and draw.io.

* **AWS service selection:**  
  Identify the roles of Amazon EC2, Amazon EBS, Amazon S3, AWS IAM, AWS Backup, and Amazon CloudWatch in the system architecture.

* **Security principles:**  
  Define principles such as not hard-coding Access Keys, applying Least Privilege, using IAM Roles, and limiting unnecessary public resources.

* **Resource naming and tagging conventions:**  
  Establish consistent naming conventions for EC2 Instances, EBS Volumes, IAM Roles, IAM Policies, Backup Vaults, Backup Plans, and other resources to facilitate management.

* **Estimated costs:**  
  Identify resources that may incur costs and limit the deployment scale to what is necessary for the lab.

After completing the preparation steps above, the AWS environment will be ready to deploy the **Web Server, data storage system, and Backup & Recovery components** in the following sections.


#### Contents

1. [AWS account and access permissions](5.2.1.tài-khoản-aws-và-quyền-truy-cập/)
2. [Region selection](5.2.2.lựa-chọn-region/)
3. [Tools used](5.2.3.công-cụ-sử-dụng/)
4. [AWS service selection](5.2.4.lựa-chọn-các-dịch-vụ-aws/)
5. [Applied security principles](5.2.5.nguyên-tắc-bảo-mật/)
6. [Resource naming and tagging conventions](5.2.6.quy-ước-đặt-tên-và-tag-tài-nguyên/)
7. [Estimated costs](5.2.7.chi-phí-dự-kiến/)