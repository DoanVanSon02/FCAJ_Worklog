---
title: "Create CloudWatch Alarms for EC2 and S3"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.6.2. </b> "
---

## Create CloudWatch Alarms for EC2 and S3

After confirming that Amazon CloudWatch has received metrics from AWS Backup, I create **CloudWatch Alarms** to test the automated monitoring mechanism.

In the workshop, I create separate Alarms for:

```text
Amazon EC2
Amazon S3
```

based on the metric:

```text
NumberOfBackupJobsCompleted
```

The purpose of the Alarms in the workshop is:

> To confirm that CloudWatch can evaluate AWS Backup Metrics and automatically change the Alarm state when the configured condition is met.

---

### Create an Alarm for Amazon EC2

![Create an Alarm for Amazon EC2](/images/5/5.6.2/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon EC2</b>
</p>

![Create an Alarm for Amazon EC2](/images/5/5.6.2/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon EC2</b>
</p>

I select the metric with the following details:

```text
Namespace:
AWS/Backup

Metric:
NumberOfBackupJobsCompleted

BackupVaultName:
BackupLab-Vault

ResourceType:
EC2
```

![Create an Alarm for Amazon EC2](/images/5/5.6.2/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon EC2</b>
</p>

![Create an Alarm for Amazon EC2](/images/5/5.6.2/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon EC2</b>
</p>

Then select:

```text
Select metric
```

---

### Configure the Metric

In the Metric configuration section, I use:

```text
Statistic:
Sum

Period:
5 minutes
```

I use `Sum` because the metric represents the number of Backup Jobs occurring within the monitored time interval.

The Period:

```text
5 minutes
```

is used to match the interval at which AWS Backup updates metrics in Amazon CloudWatch.

---

### Configure the Alarm Condition

In the condition section, I use:

```text
Threshold type:
Static
```

Condition:

```text
Whenever:
NumberOfBackupJobsCompleted
is Greater/Equal
```

Threshold:

```text
1
```

This means:

```text
NumberOfBackupJobsCompleted >= 1
```

The Alarm will be triggered when CloudWatch records at least one completed Backup Job during the evaluation period.

Datapoints are configured as:

```text
1 out of 1
```

This means that only one datapoint needs to meet the condition for the Alarm to transition to the state:

```text
ALARM
```

---

### Configure Missing Data

For Missing Data, I select:

```text
Treat missing data as not breaching
```

The reason is that the absence of a new datapoint does not mean that a Backup Job has encountered an error.

This can be understood as:

```text
No Backup Job running
          ≠
Backup Job failure
```

Therefore, when there is no new data, the Alarm should not automatically treat this as a condition breach.

![Create an Alarm for Amazon EC2](/images/5/5.6.2/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon EC2</b>
</p>

![Create an Alarm for Amazon EC2](/images/5/5.6.2/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon EC2</b>
</p>

---

### Notification

CloudWatch Alarms can be integrated with Amazon SNS to send notifications such as:

```text
Email
SMS
or other alerting workflows
```

However, the scope of the workshop focuses on demonstrating the mechanism:

```text
Metric
   ↓
Alarm Evaluation
   ↓
Alarm State
```

so I do not configure SNS Notifications.

In a real-world system, CloudWatch Alarms can be integrated with Amazon SNS or other services to send alerts to the operations team when a Backup Job encounters a problem.

Since we are not configuring notifications, remove the notification configuration, then scroll down and click **Next**.

![Create an Alarm for Amazon EC2](/images/5/5.6.2/0008.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon EC2</b>
</p>

---

### Name the EC2 Alarm

The EC2 Alarm is named:

```text
BackupLab-BackupCompleted-Test-Alarm
```

Description:

```text
Test alarm when an EC2 AWS Backup job
completes successfully.
```

Then I select:

```text
Create alarm
```

![Configure a CloudWatch Alarm to Monitor Amazon EC2 Backup Jobs](/images/5/5.6.2/0009.png?featherlight=false&width=70pc)

<p align="center">
  <b>Configure a CloudWatch Alarm to Monitor Amazon EC2 Backup Jobs</b>
</p>

![Create an Alarm for Amazon EC2](/images/5/5.6.2/0010.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon EC2</b>
</p>

---

### Create an Alarm for Amazon S3

![Create an Alarm for Amazon S3](/images/5/5.6.2/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon S3</b>
</p>

![Create an Alarm for Amazon S3](/images/5/5.6.2/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon S3</b>
</p>

To monitor both Resource Types in the workshop, I continue by creating a similar Alarm for Amazon S3.

The selected metric:

```text
Namespace:
AWS/Backup

Metric:
NumberOfBackupJobsCompleted

BackupVaultName:
BackupLab-Vault

ResourceType:
S3
```

![Create an Alarm for Amazon S3](/images/5/5.6.2/0011.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon S3</b>
</p>

Configuration:

```text
Statistic:
Sum

Period:
5 minutes

Threshold:
>= 1

Datapoints:
1 out of 1
```

![Create an Alarm for Amazon S3](/images/5/5.6.2/0012.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon S3</b>
</p>

![Create an Alarm for Amazon S3](/images/5/5.6.2/0013.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon S3</b>
</p>

The Alarm is named:

```text
BackupLab-S3-BackupCompleted-Test-Alarm
```

Description:

```text
Test alarm when an S3 AWS Backup job
completes successfully.
```

![Create an Alarm for Amazon S3](/images/5/5.6.2/0014.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon S3</b>
</p>

![Create an Alarm for Amazon S3](/images/5/5.6.2/0015.png?featherlight=false&width=70pc)

<p align="center">
  <b>Create an Alarm for Amazon S3</b>
</p>

Creating separate Alarms for EC2 and S3 allows the two Resource Types to be monitored independently.

This can be described as follows:

```text
EC2 Backup
     ↓
EC2 Metric
     ↓
EC2 Alarm


S3 Backup
     ↓
S3 Metric
     ↓
S3 Alarm
```

If only a single metric aggregated by Backup Vault were used, it would be less clear which Resource Type generated the datapoint.

---

### CloudWatch Alarm States

After creation, the Alarm may initially be in the state:

```text
INSUFFICIENT_DATA
```

This state appears when CloudWatch does not yet have enough datapoints to evaluate the condition.

When a matching datapoint appears, the Alarm may transition to:

```text
ALARM
```

if:

```text
NumberOfBackupJobsCompleted >= 1
```

If there are no datapoints breaching the condition during the evaluation period, the Alarm may be in the state:

```text
OK
```

The three main states can be understood as follows:

```text
INSUFFICIENT_DATA
→ Not enough data to evaluate

OK
→ The Alarm condition is not breached

ALARM
→ The Alarm condition has been met
```

![CloudWatch Alarm State After Evaluating the AWS Backup Metric](/images/5/5.6.2/0016.png?featherlight=false&width=70pc)

<p align="center">
  <b>CloudWatch Alarm State After Evaluating the AWS Backup Metric</b>
</p>

---

### Why Does the Lab Alarm Use `Completed` Instead of `Failed`?

In the workshop, the metric:

```text
NumberOfBackupJobsCompleted
```

with the condition:

```text
NumberOfBackupJobsCompleted >= 1
```

is used to **test the CloudWatch Alarm mechanism**.

It helps demonstrate the flow:

```text
AWS Backup
      ↓
Metric is generated
      ↓
CloudWatch receives the Metric
      ↓
CloudWatch evaluates the Threshold
      ↓
Alarm changes state
```

However, in a real-world environment, an alert for:

```text
Backup completed
```

is usually less important than an alert for:

```text
Backup failed
```

A more suitable configuration for a real-world environment could use:

```text
Metric:
NumberOfBackupJobsFailed

Condition:
>= 1
```

The alerting flow would then be:

```text
Backup Job FAILED
       ↓
NumberOfBackupJobsFailed
       ↓
CloudWatch Alarm
       ↓
SNS / Email / Incident
       ↓
Administrator
```

Since the scope of the workshop does not require deliberately causing a properly functioning Backup Job to fail, I use `NumberOfBackupJobsCompleted` to test the functionality of CloudWatch Alarms.

`NumberOfBackupJobsFailed` is noted as a more suitable implementation option when the system is applied in a real-world environment.