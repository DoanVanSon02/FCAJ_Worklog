---
title: "Launching Amazon EC2"
date: 2026-01-01
weight: 1
chapter: false
pre: " <b> 5.3.1. </b> "
---

### Launching Amazon EC2

Amazon EC2 is used as the system's main Web Server. This instance will run Apache, host the test website, and be used as a resource to be protected by AWS Backup in the following sections.

I navigate to:

```text
AWS Management Console
→ EC2
→ Instances
→ Launch instances
```

---

#### Naming the Instance

In the **Name and tags** section, I use:

```text
Name = backup-lab-web-server
```

---

#### Selecting an Amazon Machine Image

In the **Application and OS Images** section, I select:

```text
Amazon Linux 2023
Architecture: 64-bit (x86)
```

Amazon Linux 2023 is selected because it is a Linux operating system provided by AWS and optimized to run on EC2.

This operating system supports the `dnf` package manager, which is suitable for installing Apache and AWS CLI in the workshop.

---

![Naming and selecting an Amazon Machine Image for the EC2 Instance](/images/5/5.3.1/001.png?featherlight=false&width=70pc)
![Naming and selecting an Amazon Machine Image for the EC2 Instance](/images/5/5.3.1/002.png?featherlight=false&width=70pc)


<p align="center">
  <b>Figure 5.3.1: Naming and selecting an Amazon Machine Image for the EC2 Instance</b>
</p>


#### Selecting an Instance Type

In the **Instance type** section, I select:

```text
t3.micro
```

Main specifications:

```text
vCPU:   2
Memory: 1 GiB
```

As discussed in the cost section, the workshop's Web Server only runs Apache and serves a few simple HTML files, so an Instance with substantial resources is not required.

`t3.micro` meets the lab's requirements while helping limit costs.

![Selecting Amazon Linux 2023 and t3.micro](/images/5/5.3.1/003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.3.2: Selecting Amazon Linux 2023 and the t3.micro Instance Type</b>
</p>

---

#### Configuring the Key Pair

In the **Key pair** section, I create or select the Key Pair:

```text
backup-lab-key
```

The Key Pair is used to connect to EC2 via SSH.

The Private Key file must be stored in a secure location and **must not be uploaded to GitHub.**

---

#### Configuring the Network

In the **Network settings** section, I keep EC2 in the account's default VPC and enable:

```text
Auto-assign public IP = Enable
```

The Public IPv4 address is used to access the Web Server from a browser over the Internet.

##### Security Group

I create the Security Group:

```text
backup-lab-web-sg
```

with two Inbound Rules:

| Type | Port | Source | Purpose |
| --- | --- | --- | --- |
| SSH | 22 | My IP | Administer EC2 |
| HTTP | 80 | 0.0.0.0/0 | Allow access to the website |

HTTP access is allowed from the Internet because the workshop's website needs to be checked directly from a browser.

In contrast, SSH is not open to:

```text
0.0.0.0/0
```

but is restricted to the administrator's IP address to reduce unnecessary access.

This is also a basic security measure applied in the workshop.

![Configuring the Security Group for the Web Server](/images/5/5.3.1/004.png?featherlight=false&width=70pc)
![Configuring the Security Group for the Web Server](/images/5/5.3.1/005.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.3.3: Configuring the Security Group for the Web Server</b>
</p>

---

### Configuring the Root Volume

In the **Configure storage** section, I use:

```text
Root Volume:
8 GiB
gp3
```

A capacity of 8 GiB is sufficient to install Amazon Linux, Apache, and the components required for the workshop.

The data to be protected will not be stored only on the Root Volume. A separate EBS Data Volume will be created in section **5.3.3** to more clearly simulate the process of backing up and restoring application data.

---

### Launching EC2

After reviewing the entire configuration, I select:

```text
Launch instance
```

After EC2 is created, I wait until:

```text
Instance state:
Running

Status checks:
3/3 checks passed
```

![EC2 Web Server successfully launched](/images/5/5.3.1/007.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.3.4: EC2 Web Server successfully launched</b>
</p>

### Expected Results

```text
EC2 Instance = Running

Status checks = 3/3 passed

Public IPv4 = Available

Security Group = HTTP + SSH configured
```

EC2 is now ready for Web Server installation.