---
title: "Workshop"
date: 2026-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# BUILDING A BACKUP & RECOVERY SYSTEM FOR A WEB SERVER ON AWS

#### Overview

**AWS Backup** is a centralized backup management service from Amazon Web Services that helps automate and manage data backups across multiple AWS services. The service allows you to create backup plans, store Recovery Points in Backup Vaults, and restore resources when an incident occurs.

In this lab, we will build a Web Server on **Amazon EC2**, using **Amazon EBS** to store data and **Amazon S3** to store object data. The system will then be integrated with **AWS Backup** to perform data backup and recovery.

In addition, **AWS Identity and Access Management (IAM)** is used to control access permissions between services, while **Amazon CloudWatch** is used to track the status of Backup Jobs and support monitoring of backup activities.

After completing the Backup configuration, an incident scenario will be simulated by modifying or deleting data on EC2, EBS, and S3. The previously created Recovery Points will be used to restore the system and verify its data recovery capabilities.


#### Contents

1. [Introduction](<5.1.giới-thiệu/>)
2. [Preparation steps](<5.2.các-bước-chuẩn-bị/>)
3. [Building a Web Server and storing data](5.3.xây-dựng-web-server-và-lưu-trữ-dữ-liệu/)
4. [Configuring IAM](5.4.cấu-hình-iam/)
5. [Configuring AWS Backup](5.5.cấu-hình-aws-backup/)
6. [Monitoring Backup with CloudWatch](5.6.giám-sát-backup-bằng-cloudwatch/)
7. [Simulating an incident](5.7.mô-phỏng-sự-cố/)
8. [Restoring the system](5.8.khôi-phục-hệ-thống/)
9. [Cleaning up resources](5.9.dọn-dẹp-tài-nguyên/)