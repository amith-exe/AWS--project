# EC2
<img width="743" height="422" alt="image" src="https://github.com/user-attachments/assets/ca7c3870-6ac8-4f90-a4ff-a74e7b656574" />
# SSH Key Generation & Importing to AWS EC2

*A Complete Guide for Windows and Linux*

---

# What is an SSH Key?

SSH (Secure Shell) is a protocol used to securely connect to remote machines over a network.

Instead of logging in with passwords, SSH uses **public-key cryptography**.

When you generate an SSH key pair, two files are created:

```
Private Key  ← Keep Secret
Public Key   ← Safe to Share
```

Example:

```
patientping-key       → Private Key
patientping-key.pub   → Public Key
```

---

# How SSH Authentication Works

```text
Your Computer                     AWS EC2 Instance
-------------                    -----------------
Private Key  ------------------>  Verifies using Public Key
                Authentication Successful
```

1. You keep the **private key** on your computer.
2. The **public key** is stored on the server (EC2).
3. During login, the server verifies that you own the matching private key.
4. No password needs to be transmitted over the internet.

---

# Why Use SSH Keys Instead of Passwords?

### More Secure

* Resistant to brute-force attacks
* Uses strong cryptography

### Convenient

* No need to type passwords repeatedly
* Easy automation and scripting

### Widely Supported

Used by:

* AWS EC2
* GitHub
* GitLab
* Linux Servers
* Docker Hosts
* Kubernetes Nodes

---

# Step 1: Generate an SSH Key Pair

## Windows (PowerShell)

```powershell
ssh-keygen -t ed25519 -C "patientping-key" -f $HOME\.ssh\patientping-key
```

---

## Linux/macOS

```bash
ssh-keygen -t ed25519 -C "patientping-key" -f ~/.ssh/patientping-key
```

---

# Command Breakdown

```bash
ssh-keygen
```

SSH utility used to:

* Generate keys
* Manage keys
* Convert key formats

---

## `-t ed25519`

Specifies the key algorithm.

```bash
-t ed25519
```

### Why Ed25519?

* Faster than RSA
* Smaller key size
* Strong security
* Modern recommended standard

AWS, GitHub, and modern Linux systems all support Ed25519.

---

## `-C`

Adds a comment inside the public key.

```bash
-C "patientping-key"
```

Examples:

```bash
-C "github-amith"
-C "production-server"
-C "aws-key"
```

The comment helps identify what the key is used for.

---

## `-f`

Specifies where to save the key files.

Windows:

```powershell
-f $HOME\.ssh\patientping-key
```

Linux:

```bash
-f ~/.ssh/patientping-key
```

---

# Understanding `$HOME` and `~`

## Windows

```powershell
$HOME
```

Example:

```text
C:\Users\amith
```

Therefore:

```powershell
$HOME\.ssh\patientping-key
```

becomes:

```text
C:\Users\amith\.ssh\patientping-key
```

---

## Linux

```bash
~
```

or

```bash
$HOME
```

Example:

```text
/home/amith
```

Therefore:

```bash
~/.ssh/patientping-key
```

becomes:

```text
/home/amith/.ssh/patientping-key
```

---

# Files Created

After running:

```bash
ssh-keygen -t ed25519 -C "patientping-key" -f ~/.ssh/patientping-key
```

Two files are created:

### Private Key

```text
patientping-key
```

Contains:

```text
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

Keep this secret.

Never:

* Upload to GitHub
* Email it
* Share it

---

### Public Key

```text
patientping-key.pub
```

Contains something like:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA.... patientping-key
```

This file is safe to share.

---

# Viewing the Public Key

## Windows

```powershell
Get-Content $HOME\.ssh\patientping-key.pub
```

or

```powershell
cat $HOME\.ssh\patientping-key.pub
```

---

## Linux

```bash
cat ~/.ssh/patientping-key.pub
```

---

# Step 2: Import Public Key into AWS EC2

## Windows

```powershell
aws ec2 import-key-pair `
  --key-name "patientping-key" `
  --public-key-material fileb://$HOME/.ssh/patientping-key.pub
```

---

## Linux

```bash
aws ec2 import-key-pair \
  --key-name "patientping-key" \
  --public-key-material fileb://~/.ssh/patientping-key.pub
```

---

# Command Breakdown

## `aws ec2 import-key-pair`

Tells AWS:

> "Upload my locally generated public key and create an EC2 Key Pair."

---

## `--key-name`

```bash
--key-name "patientping-key"
```

This is simply the name shown inside AWS.

Example:

```text
patientping-key
production-key
github-server-key
```

You select this key when launching EC2 instances.

---

## `--public-key-material`

```bash
--public-key-material
```

Specifies that public key data is being provided.

---

## `fileb://`

```bash
fileb://~/.ssh/patientping-key.pub
```

Means:

> Read the file locally and send its contents as raw binary data.

This avoids encoding issues, especially on Windows.

---

# What Happens Internally?

## Step 1

Generate keys:

```text
Private Key
Public Key
```

↓

## Step 2

Upload Public Key:

```text
AWS EC2 Key Pair
```

↓

## Step 3

Launch EC2:

```text
EC2 Instance
```

↓

## Step 4

Connect:

```text
Private Key → Authentication → EC2
```

---

# Connecting to an EC2 Instance

## Linux/macOS

```bash
ssh -i ~/.ssh/patientping-key ubuntu@13.51.24.110
```

---

## Windows PowerShell

```powershell
ssh -i $HOME\.ssh\patientping-key ubuntu@13.51.24.110
```

---

# Command Breakdown

## `ssh`

Starts an SSH session.

---

## `-i`

Specifies which private key to use.

```bash
-i ~/.ssh/patientping-key
```

---

## `ubuntu`

Username on the remote server.

Different AMIs use different users:

| Distribution | Username |
| ------------ | -------- |
| Ubuntu       | ubuntu   |
| Amazon Linux | ec2-user |
| Debian       | admin    |
| CentOS       | centos   |

---

## `13.51.24.110`

Public IP address of the EC2 instance.

---

# Real-World Example: Production and Staging Servers

## Production Key

```bash
ssh-keygen -t ed25519 -C "production-server" -f ~/.ssh/prod_key
```

Import:

```bash
aws ec2 import-key-pair \
  --key-name "prod-key" \
  --public-key-material fileb://~/.ssh/prod_key.pub
```

---

## Staging Key

```bash
ssh-keygen -t ed25519 -C "staging-server" -f ~/.ssh/staging_key
```

Import:

```bash
aws ec2 import-key-pair \
  --key-name "staging-key" \
  --public-key-material fileb://~/.ssh/staging_key.pub
```

This keeps environments isolated and improves security.

---

# Using SSH Keys with GitHub

Generate:

```bash
ssh-keygen -t ed25519 -C "github-amith" -f ~/.ssh/github_ed25519
```

View public key:

```bash
cat ~/.ssh/github_ed25519.pub
```

Copy and paste the output into:

```
GitHub
↓
Settings
↓
SSH and GPG Keys
↓
New SSH Key
```

After that:

```bash
git clone git@github.com:username/repository.git
git push
git pull
```

No passwords required.

---

# Best Practices

✅ Use `ed25519`

✅ Keep private keys secret

✅ Create separate keys for different environments

✅ Backup important private keys securely

❌ Never upload private keys to GitHub

❌ Never share private keys with anyone

❌ Never store private keys inside your project repository

---

# Quick Revision

```text
ssh-keygen
    ↓
Private Key + Public Key
    ↓
Import Public Key to AWS
    ↓
Launch EC2
    ↓
Connect using Private Key
    ↓
Authentication Successful
```
# Launching an EC2 Instance (webserver-app)

## Objective

Create an Amazon Linux EC2 instance named **webserver-app** inside the **web-public-a** subnet.

---

## Prerequisites

Make sure the following resources already exist:

* VPC: `web-vpc`
* Subnet: `web-public-a`
* Key Pair: `web-key`

---

# Step-by-Step Instructions

## Step 1: Open EC2 Dashboard

1. Log in to the AWS Console.
2. Search for **EC2** in the top search bar.
3. Open the **EC2 Dashboard**.
4. From the left sidebar, click:

```text
Instances
```

5. Click:

```text
Launch instances
```

---

## Step 2: Configure Basic Details

### Name

Set the Name tag:

```text
webserver-app
```

> **Note:** The name is case-sensitive.

---

### Amazon Machine Image (AMI)

Select:

```text
Amazon Linux
```

This is the operating system that will be installed on the server.

---

### Instance Type

Select:

```text
t3.micro
```

Specifications:

* 2 vCPUs
* 1 GB RAM
* Free Tier eligible

---

### Key Pair

Select:

```text
web-key
```

This key pair will be used later to securely connect to the EC2 instance using SSH.

---

## Step 3: Configure Networking

Click:

```text
Edit
```

Then configure:

### VPC

```text
web-vpc
```

### Subnet

```text
web-public-a
```

### Auto-assign Public IP

```text
Disable
```

This means the instance will receive only a private IP address and cannot be directly accessed from the internet.

---

## Step 4: Configure Security Group

Under **Firewall (Security Groups)**:

Select:

```text
Create security group
```

### Security Group Name

```text
web-empty
```

### Description

```text
Placeholder security group for web application server
```

### Remove Default Rules

AWS automatically creates an SSH rule:

```text
Type: SSH
Port: 22
Source: Anywhere (0.0.0.0/0)
```

Click **Remove**.

Final configuration:

```text
Inbound Rules: None
```

The instance will initially be completely locked down.

---

## Step 5: Storage

Leave the default configuration.

Typically:

```text
8 GB gp3 EBS Volume
```

---

## Step 6: Advanced Details

Leave all settings as default.

No changes are required.

---

## Step 7: Launch the Instance

Click:

```text
Launch instance
```

You should see a message similar to:

```text
Success:
Successfully initiated launch of instance
(i-123123123)
```

---

## Step 8: Verify

Go back to:

```text
EC2
→ Instances
```

Confirm that the new instance appears:

```text
webserver-app
```

---

# Final Configuration Summary

| Setting               | Value         |
| --------------------- | ------------- |
| Name                  | webserver-app |
| AMI                   | Amazon Linux  |
| Instance Type         | t3.micro      |
| Key Pair              | web-key       |
| VPC                   | web-vpc       |
| Subnet                | web-public-a  |
| Auto-assign Public IP | Disabled      |
| Security Group        | web-empty     |
| Inbound Rules         | None          |
| Storage               | Default       |
| Advanced Details      | Default       |

---

# Architecture

```text
AWS VPC (web-vpc)
│
├── Subnet: web-public-a
│
└── EC2 Instance
      ├── Name: webserver-app
      ├── AMI: Amazon Linux
      ├── Type: t3.micro
      ├── Private IP Only
      └── Security Group: web-empty
```

---

# Quick Revision

```text
EC2
 ↓
Launch Instance
 ↓
Name → webserver-app
 ↓
AMI → Amazon Linux
 ↓
Type → t3.micro
 ↓
Key Pair → web-key
 ↓
VPC → web-vpc
 ↓
Subnet → web-public-a
 ↓
Public IP → Disabled
 ↓
Security Group → web-empty
 ↓
Remove SSH Rule
 ↓
Launch Instance
```

# AMIs and Instance Types

When launching an EC2 instance, two important decisions are:

1. **Which AMI to use?**
2. **Which Instance Type to use?**

---

# 1. Amazon Machine Image (AMI)

An **Amazon Machine Image (AMI)** is a pre-configured template used to create an EC2 instance.

It contains:

* Operating System (Linux, Windows, etc.)
* Installed software packages
* System configurations
* Boot instructions

Think of an AMI as:

```text
AMI = Blueprint or Template of a Computer
```

When you launch an EC2 instance, AWS creates a virtual machine using this template.

---

## Example

We selected:

```text
Amazon Linux
```

AWS creates an EC2 instance with:

* Amazon Linux operating system
* Default system configurations
* Required boot files

---

## Common AMIs

| AMI                      | Use Case                         |
| ------------------------ | -------------------------------- |
| Amazon Linux             | General-purpose Linux server     |
| Ubuntu Server            | Web applications and development |
| Windows Server           | Windows applications and .NET    |
| Red Hat Enterprise Linux | Enterprise workloads             |
| Debian                   | Lightweight Linux environments   |

---

## Custom AMIs

AWS also allows you to create your own AMIs.

### Why create a custom AMI?

Suppose you always install:

* Nginx
* Node.js
* Docker
* Application code
* Configuration files

Instead of repeating these steps every time, you can:

```text
Configure Server
        ↓
Create AMI
        ↓
Launch New Instances Instantly
```

Benefits:

* Faster deployments
* Consistent configurations
* Easy server replication
* Useful for Auto Scaling
* Easy migration between AWS regions

---

## AMI vs Docker Image

| Feature                   | AMI   | Docker Image |
| ------------------------- | ----- | ------------ |
| Contains Operating System | ✅ Yes | ❌ No         |
| Full Virtual Machine      | ✅ Yes | ❌ No         |
| Includes Applications     | ✅ Yes | ✅ Yes        |
| Used for EC2              | ✅ Yes | ❌ No         |
| Lightweight               | ❌ No  | ✅ Yes        |

Think of it like this:

```text
AMI     = Entire Computer
Docker  = Application running inside a Computer
```

---

# 2. Instance Type

An **Instance Type** defines the hardware resources allocated to your EC2 instance.

It determines:

* CPU
* RAM
* Storage performance
* Network performance
* GPU availability
* Processor architecture

Think of it as:

```text
AMI           = Software
Instance Type = Hardware
```

---

# Example: t3.micro

We selected:

```text
t3.micro
```

Specifications:

* 2 vCPUs
* 1 GB RAM
* Burstable CPU performance
* Free Tier eligible

Approximate cost:

```text
$0.0104 per hour
≈ $0.25 per day
≈ $7.60 per month
```

Suitable for:

* Learning AWS
* Small websites
* Development environments
* Testing applications
* Personal projects

---

# Instance Family Naming

Example:

```text
t3.micro
```

Breakdown:

```text
t      → Instance family
3      → Generation
micro  → Instance size
```

---

## Instance Families

### T Family (Burstable)

Examples:

```text
t2.micro
t3.micro
t3.small
```

Use cases:

* Development
* Small web applications
* Testing environments

---

### M Family (General Purpose)

Examples:

```text
m5.large
m6i.large
```

Use cases:

* Application servers
* Medium-sized databases
* Production web applications

---

### C Family (Compute Optimized)

Examples:

```text
c6i.large
c7g.xlarge
```

Use cases:

* Scientific computing
* High-performance applications
* Batch processing

---

### R Family (Memory Optimized)

Examples:

```text
r6i.large
r7g.xlarge
```

Use cases:

* Large databases
* In-memory caching
* Analytics systems

---

### P and G Families (GPU Instances)

Examples:

```text
g5.xlarge
p5.48xlarge
```

Use cases:

* Machine Learning
* Deep Learning
* AI Training
* Video rendering

---

# Scaling Example

Small server:

```text
t3.micro
2 vCPUs
1 GB RAM
≈ $7.60/month
```

Enterprise server:

```text
u7in-32tb.224xlarge
224 vCPUs
32 TB RAM
≈ $263,520/month
```

AWS provides instance sizes ranging from tiny learning servers to massive enterprise systems.

---

# Quick Revision

```text
Launch EC2 Instance
        ↓
Choose AMI
        ↓
Operating System + Software Template
        ↓
Choose Instance Type
        ↓
CPU + RAM + Hardware Resources
        ↓
Launch Server
```

**Remember:**

```text
AMI           = Software Blueprint
Instance Type = Hardware Configuration
EC2 Instance  = Running Virtual Machine
```
# Public and Private IP Addresses in AWS

## Overview

In IPv4 networking, IP addresses are divided into two categories:

1. **Public IP Addresses**
2. **Private IP Addresses**

The rules for these address ranges are defined in standards such as **RFC 1918** and **RFC 6598**.

---

# Public IP Addresses

A **Public IP Address** is globally unique and can be reached over the internet.

Examples:

```text
8.8.8.8
54.221.123.45
13.51.24.110
```

Characteristics:

* Accessible from anywhere on the internet
* Must be globally unique
* Used for internet-facing services
* Assigned by AWS or Internet Service Providers (ISPs)

### AWS Examples

Resources that often need public IPs:

* Web Servers
* Load Balancers
* Bastion Hosts
* Public APIs

---

# Private IP Addresses

A **Private IP Address** is used only inside private networks.

These addresses cannot be directly accessed from the public internet.

Characteristics:

* Used for internal communication
* Not routable on the internet
* Can be reused by different organizations
* More secure because they are not directly exposed

---

# Private IPv4 Address Ranges (RFC 1918)

| CIDR Block       | Address Range                   |
| ---------------- | ------------------------------- |
| `10.0.0.0/8`     | `10.0.0.0 – 10.255.255.255`     |
| `172.16.0.0/12`  | `172.16.0.0 – 172.31.255.255`   |
| `192.168.0.0/16` | `192.168.0.0 – 192.168.255.255` |

---

# Our VPC CIDR

We created the VPC with:

```text
10.0.0.0/22
```

This falls within:

```text
10.0.0.0/8
```

Therefore, our VPC uses **private IP addresses**.

---

# Communication Inside AWS

Most communication inside a VPC uses **private IP addresses**.

Example:

```text
Web Server      → 10.0.0.10
Database Server → 10.0.1.20
Redis Server    → 10.0.2.30
```

Communication:

```text
10.0.0.10  →  10.0.1.20
```

Traffic never leaves the VPC and does not require the internet.

---

# Communication with the Internet

When an EC2 instance needs to communicate with systems outside the VPC, it uses a **public IP address**.

Example:

```text
EC2 Instance
Private IP: 10.0.0.10
Public IP : 54.221.123.45
```

Communication:

```text
Internet
     ↓
54.221.123.45
     ↓
EC2 Instance
10.0.0.10
```

AWS automatically maps the public IP to the instance's private IP.

---

# Why Use Private IPs Inside a VPC?

### Security

Private IPs are not directly reachable from the internet.

### Performance

Traffic remains inside AWS's private network.

### Cost Efficiency

Internal communication does not require public internet routing.

### Scalability

Millions of private IP addresses can be used within VPCs.

---

# Typical AWS Architecture

```text
Internet
    │
Public IP
54.221.123.45
    │
Web Server
10.0.0.10
    │
Database Server
10.0.1.20
    │
Cache Server
10.0.2.30
```

* External users communicate using the **Public IP**.
* Internal AWS resources communicate using **Private IPs**.

---

# Key Points for Exams/Interviews

* Public IPs are globally reachable over the internet.
* Private IPs are used only inside private networks.
* EC2 instances always have a private IP address.
* EC2 instances receive public IP addresses only when internet access is required.
* Our VPC CIDR `10.0.0.0/22` is a **private CIDR block** because it belongs to the `10.0.0.0/8` private range.
* Communication between AWS resources inside the same VPC usually happens using **private IP addresses**.
<img width="733" height="620" alt="image" src="https://github.com/user-attachments/assets/6923e5ef-bd34-48a2-84d0-f618cf899e99" />


