---
title: "Creating an EBS Data Volume"
date: 2026-01-01
weight: 3
chapter: false
pre: " <b> 5.3.3. </b> "
---

### Creating an EBS Data Volume

Although EC2 already has a Root Volume, I create an additional, separate EBS Volume to store application data.

The goal is to separate:

```text
Operating system / Website
        ↓
   Root Volume

Application data
        ↓
   Data Volume
```

Separating the Data Volume makes Backup & Recovery testing clearer.

---

#### Checking the EC2 Availability Zone

Before creating EBS, I check EC2 at:

```text
EC2
→ Instances
→ backup-lab-web-server
```

Then record the EC2 instance's **Availability Zone**.

For example:

```text
us-east-1c
```

EBS can only be attached directly to an EC2 instance located in the **same Availability Zone**.

---

#### Creating an EBS Volume

I navigate to:

```text
EC2
→ Elastic Block Store
→ Volumes
→ Create volume
```

Volume configuration:

```text
Volume type:
gp3

Size:
1 GiB

Availability Zone:
Same Availability Zone as EC2
```

For example:

```text
EC2:
us-east-1c

EBS:
us-east-1c
```

The Tag used:

```text
Name        = backup-lab-data-volume
```

#### Why choose 1 GiB gp3?

The data used in the workshop consists of only a few small text files.

Therefore, 1 GiB is sufficient to test the process


Creating a larger volume does not benefit the lab but increases storage costs.

![Creating EBS](/FCAJ_Worklog/images/5/5.3.3/001.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.3.7: Creating a 1 GiB gp3 EBS Data Volume</b>
</p>

After successful creation, the EBS Volume should have the following information:

```text
Name = backup-lab-data-volume

Size = 1 GiB

Type = gp3

State = Available

Availability Zone = same AZ as EC2
```

---

#### Attaching EBS to EC2

In the EBS Volume list, I select:

```text
backup-lab-data-volume
```

Then select:

```text
Actions
→ Attach volume
```

In the Instance selection section, I select:

```text
backup-lab-web-server
```

and attach the volume.

Once completed, the EBS state changes from:

```text
Available
```

to:

```text
In-use
```

This indicates that the Data Volume has been successfully attached to EC2.

---

#### Checking EBS on Linux

After attaching EBS, I return to the EC2 Terminal and run:

```bash
lsblk
```

During deployment, the Data Volume may appear under a name such as:

```text
xvdb
```

or, on some Instance types:

```text
nvme1n1
```

Therefore, the actual output of the `lsblk` command must be used to identify the correct device before formatting.

For example:

```text
NAME    SIZE MOUNTPOINT
nvme0n1      8G /
nvme1n1      1G
```

In the example above:

```text
/dev/nvme0n1
```

is the Root Volume, and:

```text
/dev/nvme1n1
```

is the Data Volume that has just been attached to EC2.

---

![Checking EBS on Linux](/FCAJ_Worklog/images/5/5.3.3/003.png?featherlight=false&width=70pc)

#### Creating a Filesystem

For the new Data Volume that does not yet contain any data, I format it with the `ext4` filesystem.

For example, if the device is:

```text
/dev/nvme1n1
```

I run:

```bash
sudo mkfs -t ext4 /dev/nvme1n1
```

> **Note:** `mkfs` must only be used when the volume is new and does not yet have a filesystem. Do not run this command on a volume that already contains data to be recovered, because formatting will erase the existing data.

---

#### Mounting the Data Volume

I create the mount point:

```bash
sudo mkdir -p /data
```

Then mount the EBS Volume to the `/data` directory:

```bash
sudo mount /dev/nvme1n1 /data
```

Check:

```bash
df -h /data
```

The result should show that the EBS Volume is mounted at:

```text
/data
```

---

![Mounting EBS](/FCAJ_Worklog/images/5/5.3.3/004.png?featherlight=false&width=70pc)

#### Configuring Automatic Mounting After a Reboot

If only the `mount` command is used, the EBS Volume may not be automatically mounted again after EC2 reboots.

Therefore, I obtain the Volume's UUID using the command:

```bash
sudo blkid /dev/nvme1n1
```

Example output:

```text
UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

Then open the file:

```bash
sudo nano /etc/fstab
```

and add the line:

```text
UUID=<VOLUME-UUID> /data ext4 defaults,nofail 0 2
```

After saving the file, I check the configuration using:

```bash
sudo mount -a
```

If no errors appear, continue checking:

```bash
df -h /data
```

The Data Volume should still be displayed at:

```text
/data
```

---

#### Creating Test Data

After the EBS Data Volume has been successfully mounted, I create the initial data before performing a Backup.

Create the `version.txt` file:

```bash
echo "AWS Backup Recovery Lab - Version 1" | sudo tee /data/version.txt
```

Create the customer data file:

```bash
echo "Customer data before backup" | sudo tee /data/customer.txt
```

Create a directory to store website data:

```bash
sudo mkdir -p /data/website-data
```

Create application data:

```bash
echo "Important website data" | sudo tee /data/website-data/app-data.txt
```

---

#### Checking the Data

I check the newly created files using the commands:

```bash
cat /data/version.txt
cat /data/customer.txt
cat /data/website-data/app-data.txt
```

Expected result:

```text
AWS Backup Recovery Lab - Version 1
Customer data before backup
Important website data
```

These files will be used in the **Failure Simulation** section.

Later:

```text
version.txt
```

will be modified,

```text
customer.txt
```

will be deleted,

and:

```text
app-data.txt
```

will be replaced with incorrect data.

After Restore, these three files will be checked again to confirm that the Recovery Point has correctly restored the data to its state before the failure.

---

#### Checking the EBS Status

I use:

```bash
lsblk
df -h /data
```

to check the Data Volume and mount point.

![Checking the Data Volume and mount](/FCAJ_Worklog/images/5/5.3.3/007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.3.8: EBS Data Volume attached and mounted at /data</b>
</p>

Then check the data again:

```bash
cat /data/version.txt
cat /data/customer.txt
cat /data/website-data/app-data.txt
```

![Checking the data](/FCAJ_Worklog/images/5/5.3.3/006.png?featherlight=false&width=70pc)


Result:

```text
AWS Backup Recovery Lab - Version 1
Customer data before backup
Important website data
```

<!-- Insert an image of the test data on EBS here -->

<p align="center">
  <b>Figure 5.3.9: Test data created on EBS before Backup</b>
</p>

---

#### Results

After completing this section, the system should reach the following state:

```text
EBS = In-use

Data Volume = 1 GiB gp3

Mount point = /data

version.txt = Version 1

customer.txt = exists

app-data.txt = normal data
```