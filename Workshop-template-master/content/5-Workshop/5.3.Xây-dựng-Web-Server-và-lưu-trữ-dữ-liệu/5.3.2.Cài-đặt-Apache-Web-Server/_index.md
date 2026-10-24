---
title: "Installing Apache Web Server"
date: 2026-01-01
weight: 2
chapter: false
pre: " <b> 5.3.2. </b> "
---

### Installing Apache Web Server

Once EC2 is running reliably, I proceed to install **Apache HTTP Server** to turn EC2 into an actual Web Server.

Using a Web Server gives the workshop a concrete system whose state can be checked before and after Backup/Restore, rather than simply backing up an empty EC2 instance.

---

#### Connecting to EC2

In the AWS Management Console, I navigate to:

```text
EC2
→ Instances
→ backup-lab-web-server
→ Connect
```

Then select:

```text
In SSH Client
Copy the address under "4. Connect to your instance using its Public DNS"
```

I then open a Terminal, connect to EC2 via SSH, and perform the operations.

![Connecting to EC2 via SSH](/FCAJ_Worklog/images/5/5.3.2/001.png?featherlight=false&width=70pc)

---

#### Updating the System

Before installing Apache, I update the system packages using the command:

```bash
sudo yum update -y
```

Once the update is complete, EC2 is ready for Web Server installation.

---

#### Installing Apache HTTP Server

I install Apache using the command:

```bash
sudo yum install httpd -y
```

Once the installation is complete, start the Apache service:

```bash
sudo systemctl start httpd
```

To make Apache start automatically when EC2 is rebooted, I run:

```bash
sudo systemctl enable httpd
```

---

#### Checking the Apache Status

To check whether Apache is running, I use:

```bash
sudo systemctl status httpd
```

Expected result:

```text
Active: active (running)
```

The `active (running)` status indicates that Apache HTTP Server has started successfully and is running on EC2.

![Apache HTTP Server running on EC2](/FCAJ_Worklog/images/5/5.3.2/002.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.3.5: Apache HTTP Server running on EC2</b>
</p>

---

#### Creating Website Content

Once Apache is running, I create a test Web page at:

```text
/var/www/html/index.html
```

Open the file using the command:

```bash
sudo nano /var/www/html/index.html
```

Then add the following content:

```html
<!DOCTYPE html>
<html>
<head>
    <title>AWS Backup & Recovery Lab</title>
</head>

<body>
    <h1>AWS Backup & Recovery Lab</h1>

    <h2>Web Server is running successfully.</h2>

    <p>Server: Amazon EC2</p>
    <p>Operating System: Amazon Linux 2023</p>
    <p>Web Server: Apache HTTP Server</p>

    <hr>

    <p>This server will be protected using AWS Backup.</p>
</body>
</html>
```

After entering the content, I save the `index.html` file.

This website serves as the initial content of the Web Server and will help verify the system's state during Backup and Restore in the following sections.

---

#### Checking the Website from a Browser

After creating the Web page, I obtain the EC2 instance's **Public IPv4 Address** at:

```text
EC2
→ Instances
→ backup-lab-web-server
```

Then access the following address using a browser:

```text
http://<PUBLIC-IP>
```

Here:

```text
<PUBLIC-IP>
```

is replaced with the EC2 instance's actual Public IPv4 address.

For example:

```text
http://54.x.x.x
```

If Apache and the Security Group are configured correctly, the browser will display:

```text
AWS Backup & Recovery Lab

Web Server is running successfully.

Server: Amazon EC2
Operating System: Amazon Linux 2023
Web Server: Apache HTTP Server

This server will be protected using AWS Backup.
```

![Website running successfully on Amazon EC2](/FCAJ_Worklog/images/5/5.3.2/003.png?featherlight=false&width=70pc)

<p align="center">
  <b>Figure 5.3.6: Website running successfully on Amazon EC2</b>
</p>

The image above shows the initial state of the Web Server **before a failure occurs**.

In the later Restore section, the system's state after recovery will be checked and compared with this initial state.

---

#### Results

After completing this section, the system should reach the following states:

```text
Apache = active (running)

HTTP port 80 = accessible

Website = displayed successfully
```

EC2 is now functioning as a Web Server and is ready for the next step of configuring an **EBS Data Volume** to store application data.