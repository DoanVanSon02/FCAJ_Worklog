---
title: "Validate Data After Recovery"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.8.3. </b> "
---

## Validate Data After Recovery

A Restore Job displaying:

```text
COMPLETED
```

is not the only evidence needed to conclude that the system has been recovered correctly.

I need to directly verify the data after Restore.

This helps distinguish between:

```text
Restore Job completed
```

and:

```text
Data actually restored correctly
```

Therefore, I perform Validation for each component of the system.

---

### Check `version.txt`

On the restored EC2:

```text
backup-lab-web-server-restored
```

I run:

```bash
cat /data/version.txt
```

Before the incident, the data was:

```text
AWS Backup Recovery Lab - Version 1
```

After the incident simulation:

```text
AWS Backup Recovery Lab - Version 2 - DATA CHANGED
```

After Restore, the expected result is:

```text
AWS Backup Recovery Lab - Version 1
```

The test flow:

```text
BEFORE BACKUP
Version 1
     ↓
INCIDENT
Version 2
     ↓
RESTORE
Version 1 ✓
```

This result demonstrates that data changed after the Backup was taken is not present in the Recovery Point used for Restore.

---

### Check `customer.txt`

Next, I check the directory:

```bash
ls -l /data
```

and confirm that the file:

```text
customer.txt
```

has reappeared.

Then I check its contents:

```bash
cat /data/customer.txt
```

Expected result:

```text
Customer data before backup
```

The test flow:

```text
BEFORE INCIDENT
customer.txt exists
        ↓
INCIDENT
customer.txt deleted
        ↓
RESTORE
customer.txt reappears ✓
```

This demonstrates that a file deleted after the Backup was taken can be recovered from the Recovery Point.

---

### Check `app-data.txt`

I continue checking:

```bash
cat /data/website-data/app-data.txt
```

Expected result:

```text
Important website data
```

instead of the content changed during the incident simulation:

```text
CORRUPTED APPLICATION DATA
```

The test flow:

```text
BEFORE
Important website data
        ↓
INCIDENT
CORRUPTED APPLICATION DATA
        ↓
RESTORE
Important website data ✓
```

Through these three files, I have tested three different types of incidents:

```text
Data modified       → Restore ✓

File deleted        → Restore ✓

Incorrect content   → Restore ✓
```

![Data on EBS restored to its state before the incident](/FCAJ_Worklog/images/5/5.8.3/0001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Data on EBS restored to its state before the incident</b>
</p>

The Terminal screenshot should show the following results:

```text
AWS Backup Recovery Lab - Version 1

Customer data before backup

Important website data
```

This is one of the most important pieces of evidence in the entire workshop because it directly demonstrates that the data on EBS has been restored to its state before the incident.

---

### Check Apache

After confirming the data on EBS, I check the Apache Web Server status:

```bash
sudo systemctl status httpd
```

Expected result:

```text
Active: active (running)
```

If Apache is not running, I can start it again using:

```bash
sudo systemctl start httpd
```

Then check again:

```bash
sudo systemctl status httpd
```

The goal is to ensure that the Web Server on the restored EC2 can operate normally.

![Data on EBS restored to its state before the incident](/FCAJ_Worklog/images/5/5.8.3/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Data on EBS restored to its state before the incident</b>
</p>

---

### Check the Website

I obtain the:

```text
Public IPv4
```

of the EC2:

```text
backup-lab-web-server-restored
```

Then access:

```text
http://<RESTORED-PUBLIC-IP>
```

The restored Website should display the original content again:

```text
AWS Backup & Recovery Lab

Web Server is running successfully.
```

Meanwhile, the original EC2 still retains the state after the incident:

```text
INCIDENT TEST
```

Therefore, I can open two browser tabs for a direct comparison:

```text
ORIGINAL EC2
────────────────
INCIDENT TEST


RESTORED EC2
────────────────
AWS Backup & Recovery Lab
Web Server is running successfully.
```

![Compare the website on the affected EC2 with the restored EC2](/FCAJ_Worklog/images/5/5.8.3/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Compare the website on the affected EC2 with the restored EC2</b>
</p>

![Compare the website on the affected EC2 with the restored EC2](/FCAJ_Worklog/images/5/5.8.3/0004.png?featherlight=false&width=70pc)

<p align="center">
  <b>Compare the website on the affected EC2 with the restored EC2</b>
</p>

This image helps visually demonstrate that the restored EC2 has returned to the Website state before the incident occurred.

---

### Check Amazon S3

After checking EC2/EBS, I continue verifying the Amazon S3 data.

I navigate to:

```text
Amazon S3
→ Bucket
→ application-data/
```

and confirm that the object:

```text
s3-version.txt
```

has reappeared.

Then I open or Download the object to check its contents.

Expected result:

```text
AWS Backup Recovery Lab - S3 Version 1
```

The test flow:

```text
BEFORE INCIDENT
s3-version.txt exists
        ↓
INCIDENT
s3-version.txt deleted
        ↓
AWS BACKUP RESTORE
        ↓
s3-version.txt reappears ✓
```

![s3-version.txt reappears after Restore](/FCAJ_Worklog/images/5/5.8.3/0005.png?featherlight=false&width=70pc)

<p align="center">
  <b>s3-version.txt reappears after Restore</b>
</p>

![Contents of s3-version.txt restored to Version 1](/FCAJ_Worklog/images/5/5.8.3/0006.png?featherlight=false&width=70pc)

<p align="center">
  <b>Contents of s3-version.txt restored to Version 1</b>
</p>

---

### Check Restore Jobs

Finally, I return to:

```text
AWS Backup
→ Jobs
→ Restore jobs
```

and confirm:

```text
EC2 Restore Job
Status = COMPLETED

S3 Restore Job
Status = COMPLETED
```

![EC2 and S3 Restore Jobs completed successfully](/FCAJ_Worklog/images/5/5.8.3/0007.png?featherlight=false&width=70pc)

<p align="center">
  <b>EC2 and S3 Restore Jobs completed successfully</b>
</p>

This image provides evidence from AWS Backup, while the Terminal, Website, and Amazon S3 images provide evidence from the restored data itself.

After completing the validation process, the required results are:

```text
✓ version.txt returns to Version 1

✓ customer.txt reappears

✓ app-data.txt returns to its original content

✓ Apache operates normally

✓ The restored Website displays the content from before the incident

✓ s3-version.txt reappears

✓ Contents of s3-version.txt = S3 Version 1

✓ EC2 Restore Job = COMPLETED

✓ S3 Restore Job = COMPLETED
```