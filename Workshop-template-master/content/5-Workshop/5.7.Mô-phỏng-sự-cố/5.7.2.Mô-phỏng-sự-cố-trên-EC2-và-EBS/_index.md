---
title: "Simulate Incidents on EC2 and EBS"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.7.2. </b> "
---

### Simulate Incidents on EC2 and EBS

After confirming that the Recovery Point exists, I begin simulating incidents on EC2 and EBS.

I reconnect to the EC2 instance:

```text
backup-lab-web-server
```

through:

```text
In SSH client
```

The data on the EBS Data Volume is currently stored at:

```text
/data
```

Before making changes, I check the initial state again:

```bash
cat /data/version.txt
cat /data/customer.txt
cat /data/website-data/app-data.txt
```

Results:

```text
AWS Backup Recovery Lab - Version 1

Customer data before backup

Important website data
```

This is the data state saved in the Recovery Point.

---

#### Modify `version.txt`

First, I simulate a scenario where data is overwritten or modified after the Backup.

The file used:

```text
/data/version.txt
```

Before the incident, the file contains:

```text
AWS Backup Recovery Lab - Version 1
```

I change the content to Version 2:

```bash
echo "AWS Backup Recovery Lab - Version 2 - DATA CHANGED" | sudo tee /data/version.txt
```

Then check again:

```bash
cat /data/version.txt
```

Result:

```text
AWS Backup Recovery Lab - Version 2 - DATA CHANGED
```

The data state at this point:

```text
Before Backup:
Version 1

After Incident:
Version 2
```

After performing Restore in the next section, I expect the data to return to:

```text
After Restore:
Version 1
```

---

#### Delete `customer.txt`

Next, I simulate a scenario where an important data file is deleted after the Backup.

The original file:

```text
/data/customer.txt
```

I run:

```bash
sudo rm /data/customer.txt
```

Then check the directory:

```bash
ls -l /data
```

At this point:

```text
customer.txt
```

no longer appears in the listing.

I can check directly:

```bash
cat /data/customer.txt
```

The system will report that the file does not exist.

This scenario simulates an important data file being accidentally deleted by a user or application after the Backup has been created.

State:

```text
Before Incident:
customer.txt = exists

After Incident:
customer.txt = deleted
```

After the Restore process, the goal is for the file:

```text
/data/customer.txt
```

to reappear with the content:

```text
Customer data before backup
```

---

#### Corrupt the Contents of `app-data.txt`

I continue by simulating a scenario where the file still exists but its contents are no longer correct.

The file:

```text
/data/website-data/app-data.txt
```

originally contains:

```text
Important website data
```

I overwrite the content with:

```bash
echo "CORRUPTED APPLICATION DATA" | sudo tee /data/website-data/app-data.txt
```

Then check:

```bash
cat /data/website-data/app-data.txt
```

Result:

```text
CORRUPTED APPLICATION DATA
```

This scenario is different from deleting a file.

The file still exists, but the data inside has been modified.

After the three tests, the data state on EBS is:

```text
version.txt
→ data modified

customer.txt
→ deleted

app-data.txt
→ incorrect contents
```

Creating three different types of incidents allows the subsequent Restore process to test more scenarios instead of just a single type of error.

![Simulate Incidents on EBS](/FCAJ_Worklog/images/5/5.7.2/0002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Simulate Incidents on EBS</b>
</p>

---

#### Modify the Website Content

In addition to the data on EBS, I modify the Web page content to make it easy to observe the difference between the current EC2 instance and the EC2 instance restored later.

Website file:

```text
/var/www/html/index.html
```

I edit it using:

```bash
sudo nano /var/www/html/index.html
```

Then replace the content with:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Incident Test</title>
</head>
<body>
    <h1>INCIDENT TEST</h1>

    <h2>Website data was changed after backup.</h2>

    <p>This page represents the system state after an incident.</p>
</body>
</html>
```

After saving, I revisit:

```text
http://<PUBLIC-IP>
```

The Website now displays:

```text
INCIDENT TEST

Website data was changed after backup.
```

Modifying the Website allows me to easily distinguish between two states:

```text
Current EC2 instance
→ INCIDENT TEST

EC2 instance restored later
→ AWS Backup & Recovery Lab
```

![Simulate Incidents on EBS](/FCAJ_Worklog/images/5/5.7.2/0003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Simulate Incidents on EBS</b>
</p>

If the Restore process succeeds, the restored EC2 instance must display the Website content from before the incident.

---