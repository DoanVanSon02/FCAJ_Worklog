---
title: "Estimated Costs"
date: 2026-01-01
weight: 7
chapter: false
pre: " <b> 5.2.7. </b> "
---

## Deployment Cost Estimate

Although the workshop is built on a small scale, AWS resources may still incur costs while they are running. Therefore, before deployment, I estimate the costs based on the actual configuration used in this lab.

The values below are calculated for the Region:

```text
US East (N. Virginia)
us-east-1
```

Actual costs may vary depending on usage duration, data volume, the number of Backup/Restore operations, and AWS pricing at the time of deployment.

---

### Amazon EC2

The Web Server in the workshop uses the following configuration:

```text
AMI: Amazon Linux 2023
Instance type: t3.micro
vCPU: 2
Memory: 1 GiB
Pricing model: On-Demand
```

#### Why choose `t3.micro`?

The purpose of EC2 in the workshop is simply to run a basic Apache Web Server, create test data, and execute some AWS CLI commands. The system does not receive high traffic and does not run applications that require substantial CPU or memory resources.

`t3.micro` provides **2 vCPUs and 1 GiB of RAM**, which is sufficient for the lab requirements while costing significantly less than larger instances such as `t3.small` or `t3.medium`.

The T3 family is also suitable for workloads with low CPU utilization most of the time but occasional needs for short bursts of higher performance.

Reference: [Amazon EC2 T3 Instances](https://aws.amazon.com/ec2/instance-types/t3/)

I use **On-Demand** instead of a Reserved Instance because the workshop only runs for a short period and does not require a commitment to use EC2 for 1 or 3 years.

The On-Demand price of a Linux `t3.micro` instance in `us-east-1` is approximately:

```text
0.0104 USD/hour
```

If I forget and leave EC2 running continuously for approximately **30 days (720 hours)**:

```text
0.0104 × 720
≈ 7.49 USD/month
```

#### Public IPv4

EC2 in the workshop uses a Public IPv4 address so that I can access the Web Server from a browser and connect to the instance.

AWS currently charges approximately:

```text
0.005 USD / Public IPv4 / hour
```

If maintained for 30 days:

```text
0.005 × 720
= 3.60 USD/month
```

This also shows why terminating EC2 after completing the workshop is necessary, rather than leaving the resource running.

---

### Amazon EBS

EC2 uses two EBS Volumes:

```text
Root Volume:
8 GiB gp3

Data Volume:
1 GiB gp3
Mount point: /data
```

Total provisioned capacity:

```text
8 GiB + 1 GiB
= 9 GiB
```

#### Why choose `gp3`?

`gp3` is a General Purpose SSD type suitable for typical workloads.

With the default configuration, gp3 provides:

```text
3,000 IOPS
125 MiB/s throughput
```

This level of performance far exceeds the requirements of a small Web Server in the workshop.

In `us-east-1`, gp3 costs approximately:

```text
0.08 USD/GiB-month
```

while gp2 costs approximately:

```text
0.10 USD/GiB-month
```

Since the lab does not require high IOPS, I keep the default gp3 performance configuration instead of purchasing additional IOPS or throughput.

Reference: [AWS Documentation – EBS Volume Types](https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-plan-storage-compare-volume-types.html)

Cost for 9 GiB:

```text
9 × 0.08
= 0.72 USD/month
```

If EBS is maintained for only about one day:

```text
0.72 / 30
≈ 0.024 USD/day
```

#### Why is the Data Volume only 1 GiB?

The workshop's test data consists of only a few text files, such as:

```text
version.txt
customer.txt
app-data.txt
```

Therefore, there is no need to create a larger volume.

Using a small volume also helps limit costs.

---

### Amazon S3

The S3 Bucket uses:

```text
Storage class: S3 Standard
Versioning: Enabled
Block Public Access: Enabled
```

#### Why use S3 Standard?

Objects in the workshop are created, read, deleted, and restored multiple times within a short period.

Therefore, I use **S3 Standard** instead of storage classes intended for infrequently accessed data or long-term storage.

S3 Standard does not require me to manage a separate storage server and is suitable for files that are accessed frequently.

The S3 Standard price in this Region is approximately:

```text
0.023 USD/GB-month
```

for the first storage tier.

In the workshop, the objects are only a few very small text files. Suppose the total data volume even reaches **1 MB**:

```text
1 MB ≈ 0.000977 GB
```

The approximate monthly storage cost is:

```text
0.000977 × 0.023
≈ 0.000022 USD/month
```

Therefore, the lab's S3 Storage cost is almost negligible.

S3 also charges for requests such as PUT, GET, and LIST. However, the workshop makes only a very small number of requests, so this cost is also very low.

For example, the S3 Standard PUT price is approximately:

```text
0.005 USD / 1,000 requests
```

#### Impact of Versioning

Versioning is enabled because it is a necessary configuration for the S3 Backup scenario and helps retain object versions.

However, older versions still occupy storage space and continue to incur charges.

Therefore, in a real-world system, if the data volume is large and changes frequently, a **Lifecycle Policy** should also be used to manage older versions.

---

### AWS IAM

The workshop uses:

```text
BackupLab-EC2-S3-Role
BackupLab-EC2-S3-Policy
AWSBackupDefaultServiceRole
```

AWS IAM does not charge separately for creating IAM Roles or IAM Policies. AWS only charges for the services accessed by these identities.

Reference: [AWS IAM Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started.html)

Therefore:

```text
Direct IAM cost:
0 USD
```

Using an IAM Role in the workshop mainly serves to meet security requirements and avoid storing an Access Key directly on EC2.

---

### AWS Backup

AWS Backup does not charge a fixed fee simply for creating a Backup Plan or Backup Vault.

Costs mainly depend on:

* The actual amount of backup data.
* The Recovery Point retention period.
* The type of resource being protected.
* Restore operations.

Reference: [AWS Backup Pricing](https://aws.amazon.com/backup/pricing/)

In the workshop, I use:

```text
Backup Vault:
BackupLab-Vault

Retention:
7 days

Protected resources:
EC2 + EBS
S3
```

#### Why retain Recovery Points for only 7 days?

The goal of the workshop is to test Backup & Recovery over a short period, not to build a long-term backup storage system.

A retention period of 7 days is sufficient to carry out the following process:

```text
Backup
   ↓
Create a Recovery Point
   ↓
Simulate a failure
   ↓
Restore
   ↓
Verify the results
```

while limiting the retention of unnecessary Recovery Points and the additional costs they incur.

#### EC2/EBS Backup Cost Example

AWS lists a rate of approximately:

```text
0.05 USD/GB-month
```

for EBS backup storage in the example for US East (N. Virginia).

Reference: [AWS Backup Pricing](https://aws.amazon.com/backup/pricing/)

To make this easier to understand, assume that the workshop's entire **9 GiB of EBS storage** becomes billable backup data and that the Recovery Point is retained for the full 7 days:

```text
9 × 0.05 × 7 / 30
≈ 0.105 USD
```

This is a simplified calculation for estimation purposes.

In practice, EBS backups are snapshot-based, and subsequent backups are incremental, so the actual billable storage may differ from the total provisioned capacity.

#### S3 Backup

AWS Backup for S3 also charges based on backup storage and generates additional GET/LIST requests as well as EventBridge events.

In particular, AWS specifies that S3 objects smaller than **128 KB** are billed as backup objects with a size of 128 KB.

Reference: [AWS Backup Pricing](https://aws.amazon.com/backup/pricing/)

The lab contains only a few very small text files, so the S3 Backup cost is very low.

However, in a real-world system with millions of objects, the number of objects and requests can generate significant costs.

---

### Amazon CloudWatch

The workshop uses CloudWatch to monitor AWS Backup metrics and create Alarms.

For example:

```text
BackupLab-BackupCompleted-Test-Alarm
```

A standard-resolution metric alarm costs approximately:

```text
0.10 USD / alarm metric / month
```

and is billed based on how long the Alarm exists.

Reference: [Amazon CloudWatch Pricing](https://aws.amazon.com/cloudwatch/pricing/)

If the workshop retains only one Alarm for one day:

```text
0.10 / 30
≈ 0.0033 USD
```

If two Alarms are used, such as one Alarm monitoring EC2 and another monitoring S3, and they are retained for the entire month, the theoretical cost is approximately:

```text
2 × 0.10
= 0.20 USD/month
```

In the lab, the number of metrics and Alarms is very small, so the CloudWatch cost is negligible.

---

### Additional Costs During Restore

When restoring EC2, AWS Backup creates **a new EC2 instance along with the corresponding EBS Volumes**.

This means that, for a period of time, the following may exist simultaneously:

```text
Original EC2
   +
Restored EC2
```

For example, if the restored EC2 uses `t3.micro` and exists for approximately **2 hours** while I verify the data:

```text
EC2:
0.0104 × 2
= 0.0208 USD

Public IPv4:
0.005 × 2
= 0.0100 USD
```

Adding a very small amount for the restored instance's EBS storage, the total additional cost is approximately:

```text
≈ 0.03 USD
```

This explains why the restored EC2 also needs to be terminated after the **Validation** process is complete.

---

### Summary of Estimated Workshop Costs

With the following assumptions:

```text
1 EC2 t3.micro running for 24 hours
1 Public IPv4 in use for 24 hours
9 GiB gp3
1 EC2/EBS Recovery Point retained for 7 days
S3 containing only a few text files
1 CloudWatch Alarm
Restored EC2 running for approximately 2 hours
```

the estimate based on listed prices is:

| Component | Estimated Cost |
| --- | ---: |
| EC2 `t3.micro` for 24 hours | ~0.25 USD |
| Public IPv4 for 24 hours | ~0.12 USD |
| 9 GiB EBS gp3 for 1 day | ~0.024 USD |
| S3 with less than 1 MB of data | <0.001 USD |
| EC2/EBS Backup retained for approximately 7 days | ~0.105 USD* |
| S3 Backup | Almost negligible for a few small objects |
| IAM | 0 USD |
| 1 CloudWatch Alarm for 1 day | ~0.003 USD |
| Restored EC2 existing for approximately 2 hours | ~0.03 USD |
| **Estimated Total** | **Approximately 0.5–0.6 USD** |

\* The AWS Backup figure here is a **conservative estimate**, assuming that the entire 9 GiB becomes billable backup storage. The actual cost depends on the amount of data blocks stored in the backup.

The figure above **does not include Free Tier benefits, credits, or account-specific offers**, as each AWS account may have different conditions.

---

### Cost Observations

The estimate above shows that the test data itself on S3 and EBS costs very little.

The more noteworthy factors are:

* EC2 running time.
* Public IPv4 costs.
* Retaining Recovery Points for a long period.
* Additional resources created during Restore.

Therefore, after completing the workshop, I need to check and delete resources that are no longer in use.

These resources will be handled in the section:

```text
5.9. Resource Cleanup
```

Thorough cleanup helps prevent EC2 instances, Public IPv4 addresses, EBS Volumes, Recovery Points, or CloudWatch Alarms from continuing to exist and incurring unexpected costs.