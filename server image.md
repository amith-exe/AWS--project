# AWS AMI (Amazon Machine Image) Notes

## What is an AMI?

**AMI (Amazon Machine Image)** is a **template or blueprint** used to create EC2 instances.

Think of it as:

```text
AMI = Operating System + Installed Software + Configuration + Filesystem
```

When you launch an EC2 instance, AWS copies the AMI and boots a new virtual machine from it.

---

# Real-World Analogy

Imagine you have configured your laptop exactly the way you like:

* Windows installed
* VS Code installed
* Node.js installed
* Git configured
* Browser bookmarks added
* Development tools installed

Instead of repeating all these steps on another laptop, you create a **full disk image**.

An AMI works similarly.

```text
Configured Server
        ↓
Create AMI
        ↓
Launch New Server
        ↓
Everything is already installed
```

---

# Relationship Between AMI and EC2

```text
AMI
+
Instance Type
+
Storage
↓
EC2 Instance
```

Example:

```text
AMI:
Amazon Linux

Instance Type:
t3.micro
```

Result:

```text
Amazon Linux Server
2 vCPUs
1 GB RAM
```

---

# AMI vs Instance Type

## AMI

Defines:

* Operating System
* Installed Software
* Configuration
* Filesystem

Examples:

* Amazon Linux
* Ubuntu
* Windows Server

---

## Instance Type

Defines:

* CPU
* RAM
* Network Performance
* GPU
* Hardware Resources

Examples:

* t3.micro
* t3.small
* m5.large
* g4dn.xlarge

---

# Types of AMIs

## AWS-Provided AMIs

Created and maintained by AWS.

Examples:

* Amazon Linux 2023
* Windows Server 2022
* Deep Learning AMI

---

## Marketplace AMIs

Created by vendors.

Examples:

* WordPress
* Jenkins
* MongoDB
* Elasticsearch

---

## Custom AMIs

Created by you.

Examples:

```text
Ubuntu
Node.js
Docker
Nginx
NestJS Application
PM2
Monitoring Tools
```

Save everything as:

```text
bookmyvenue-backend-v1
```

Launch new servers instantly.

---

# Why Create Your Own AMI?

## 1. Simple Backup

Suppose your server looks like:

```text
Amazon Linux
Node.js
Nginx
NestJS Backend
Database Configuration
```

Server crashes:

```text
Delete Server
Launch from AMI
Everything is restored
```

### Benefits

* Quick recovery
* Known good state
* Disaster recovery

---

# Example

Before AMI:

```text
Create EC2
Install Node.js
Install Nginx
Configure PM2
Deploy App
```

Time:

30-60 minutes.

After AMI:

```text
Launch EC2 from AMI
```

Time:

2-3 minutes.

---

# 2. Cold Storage and Cost Savings

Suppose you have a demo server.

You only need it once every month.

Running:

```text
24 hours × 30 days
```

means paying for:

* CPU
* RAM
* Storage
* Public IP

Most of the time:

```text
Server = Doing Nothing
```

---

## Better Approach

```text
Running Server
      ↓
Create AMI
      ↓
Terminate EC2
      ↓
Pay only for snapshot storage
      ↓
Need server again?
      ↓
Launch new EC2 from AMI
```

---

# Why is this cheaper?

Running EC2:

You pay for:

* Compute
* Memory
* Storage
* Networking

AMI:

You mostly pay only for:

* EBS snapshot storage

Storage is much cheaper than keeping a server running.

---

# Example

Demo Project:

```text
NestJS Backend
PostgreSQL
Nginx
```

Not needed for 2 months.

Instead of:

```text
Running 24/7
```

You can:

```text
Create AMI
Terminate Instance
```

Later:

```text
Launch new instance from AMI
```

Everything returns.

---

# 3. Transfer to Another Region

AMIs are region-specific.

Example:

```text
Mumbai Region
(ap-south-1)
```

You created:

```text
bookmyvenue-backend-v1
```

Now users are in Europe.

You want servers closer to them.

---

## Solution

```text
Copy AMI
Mumbai
      ↓
Frankfurt
      ↓
Launch Instance
```

Same configuration.

Different region.

---

# Why is this useful?

## Disaster Recovery

If one region fails:

```text
Mumbai Region
      ↓
Unavailable
      ↓
Launch from copied AMI
Frankfurt Region
```

Application continues.

---

## Lower Latency

```text
Indian Users
↓
Mumbai Servers

European Users
↓
Frankfurt Servers
```

Better performance.

---

# 4. Cookie Cutter for New Servers

A cookie cutter creates identical cookies.

AMIs do the same thing.

---

# Without AMI

Every server:

```text
Launch EC2
Install Node.js
Install Docker
Install Git
Install PM2
Clone Project
Configure Nginx
Configure Environment Variables
Deploy Application
```

Again and again.

---

# With AMI

Create one server:

```text
Amazon Linux
Node.js
Docker
PM2
Nginx
Application
Monitoring Agents
```

Create:

```text
bookmyvenue-backend-v1
```

Launch:

```text
EC2 #1
EC2 #2
EC2 #3
EC2 #4
```

Every server is identical.

---

# Why Companies Use This

## Auto Scaling

Traffic increases:

```text
100 users
↓
10,000 users
```

AWS launches:

```text
New Server
New Server
New Server
```

Each server comes from:

```text
Backend AMI
```

Everything is preconfigured.

---

# Production Example

Company Setup:

```text
Ubuntu
Node.js
NestJS
Docker
Nginx
PM2
CloudWatch Agent
Monitoring Tools
```

Create:

```text
company-backend-v1
```

Launch:

```text
10 identical servers
```

No manual setup.

---

# AMI Workflow

## Step 1

Create server:

```text
Amazon Linux
Node.js
NestJS
Docker
Nginx
```

---

## Step 2

Create AMI:

```text
bookmyvenue-backend-v1
```

---

## Step 3

AWS creates:

```text
AMI Metadata
+
EBS Snapshot
```

---

## Step 4

Launch EC2:

```text
bookmyvenue-backend-v1
      ↓
New EC2 Instance
      ↓
Everything already installed
```

---

# Big Picture

```text
Base AMI
      ↓
Configure Server
      ↓
Install Applications
      ↓
Create Custom AMI
      ↓
Backup
      ↓
Cost Savings
      ↓
Cross-Region Deployment
      ↓
Identical Server Creation
      ↓
Auto Scaling
```

---

# For Your Future NestJS Project

Eventually you may have:

```text
Ubuntu
Node.js
Docker
PM2
Nginx
NestJS API
Monitoring Agents
```

Create:

```text
bookmyvenue-api-v1
```

Whenever needed:

```text
Launch EC2 from AMI
```

Your backend server is ready in minutes instead of configuring everything from scratch.

---

# One-Line Definition

**AMI (Amazon Machine Image) is a reusable blueprint containing an operating system, installed software, configurations, and files that AWS uses to create identical EC2 instances quickly and consistently.**
