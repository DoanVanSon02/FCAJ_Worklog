---
title: "Region selection"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.2.2. </b> "
---

## Region selection

The main resources in the workshop are deployed in:

```text
US East (N. Virginia)
Region: us-east-1
```

I selected **US East (N. Virginia) – `us-east-1`** and use this Region throughout the workshop.

Using a single Region consistently keeps resources deployed in one location, makes them easier to manage, and reduces cases where resources cannot be found due to accidentally switching to the wrong Region during configuration.

Before starting each section of the workshop, check the Region selected in the upper-right corner of the **AWS Management Console** and ensure that the current Region is:

```text
US East (N. Virginia)
us-east-1
```

![Selecting the US East (N. Virginia) Region](/FCAJ_Worklog/images/5/5.2.2.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.2.1: Selecting the US East (N. Virginia) Region – us-east-1</b>
</p>

**AWS Identity and Access Management (IAM)**, however, is a **Global** service, so when accessing the IAM interface, the displayed Region may be:

```text
Global
```

This does not affect the deployment of other resources in the workshop in the `us-east-1` Region.

Defining the Region from the outset ensures that resources such as **Amazon EC2, Amazon EBS, Amazon S3, AWS Backup, and Amazon CloudWatch** are deployed consistently within the same Region, making it easier to manage, test, and clean up resources after completing the workshop.