# AWS Security Groups & EC2 ↔ RDS Communication Notes

## Goal

Allow the application server (EC2) to connect to the PostgreSQL database (RDS).

Architecture:

```text
Internet
    ↓
EC2 (Public Subnet)
    ↓
RDS PostgreSQL (Private Subnet)
```

Even though the database is in a private subnet, EC2 can still access it because both resources are inside the same VPC.

---

# VPC Routing

AWS automatically creates a route like:

```text
10.0.0.0/16 → local
```

This means:

```text
EC2 (10.0.0.x)
      ↓
Can reach
      ↓
RDS (10.0.1.x)
```

without using the Internet Gateway.

Traffic stays entirely inside AWS.

---

# What is a Security Group?

A Security Group (SG) is a **stateful virtual firewall** attached to AWS resources.

Examples:

* EC2
* RDS
* Load Balancers

Security Groups control:

```text
Who can enter?
Who can leave?
```

---

# Inbound Rules

Inbound rules control:

```text
Incoming traffic
```

Think:

```text
Can someone enter my house?
```

Example:

```text
SSH
Port: 22
Source: My Laptop IP
```

Meaning:

```text
Laptop
   ↓
EC2
```

allowed.

---

# Outbound Rules

Outbound rules control:

```text
Outgoing traffic
```

Think:

```text
Can I leave my house?
```

Example:

```text
All Traffic
Destination: 0.0.0.0/0
```

Meaning:

```text
EC2
 ↓
Internet
```

allowed.

---

# Understanding Traffic Flow

When your application connects to PostgreSQL:

```text
EC2
 ↓
RDS
```

two things happen:

### Step 1

EC2 sends request:

```text
EC2
 ↓
Port 5432
 ↓
RDS
```

Requires:

```text
EC2 Outbound Rule
```

---

### Step 2

RDS receives request.

Requires:

```text
RDS Inbound Rule
```

---

# Why Security Groups Are Stateful

Normal firewall:

```text
Request Allowed
Response Blocked
```

possible.

Security Groups:

```text
Request Allowed
      ↓
Response Automatically Allowed
```

No extra rule needed.

This is called:

```text
Stateful Firewall
```

---

# EC2 Security Group

Typical outbound rule:

```text
Type: All Traffic
Destination: 0.0.0.0/0
```

Meaning:

```text
EC2 can connect anywhere.
```

Usually this already exists.

---

# RDS Security Group

RDS should NOT be public.

Bad:

```text
Source: 0.0.0.0/0
Port: 5432
```

Anyone on the Internet could try connecting.

---

Good:

```text
Type: PostgreSQL
Port: 5432
Source:
patientping-public SG
```

Meaning:

```text
Only EC2 instances
using patientping-public SG
can connect.
```

---

# Security Group Referencing

Instead of allowing:

```text
10.0.0.0/16
```

AWS allows:

```text
Source:
patientping-public SG
```

This means:

```text
Any EC2
with this SG attached
can access RDS.
```

This is more secure and easier to manage.

---

# Step-by-Step Guide: Connect to EC2 and Access RDS Database

## Step 1: Connect to EC2 via SSH

Make sure you have your `.pem` key file.

```bash
chmod 400 your-key.pem
```

Connect to EC2:

```bash
ssh -i your-key.pem ec2-user@EC2_PUBLIC_IP
```

Example:

```bash
ssh -i patientping.pem ec2-user@13.233.xxx.xxx
```

---

## Step 2: Verify PostgreSQL Client is Installed

```bash
which psql
```

Expected output:

```bash
/usr/bin/psql
```

If not installed:

```bash
sudo yum install postgresql -y
```

---

## Step 3: Get RDS Endpoint

Go to AWS Console → RDS → Databases → Copy endpoint.

Example:

```text
patientping-db.xxxxx.ap-south-1.rds.amazonaws.com
```

---

## Step 4: Connect to PostgreSQL from EC2

```bash
psql -h RDS_ENDPOINT -U postgres -d patientping
```

Example:

```bash
psql -h patientping-db.xxxxx.ap-south-1.rds.amazonaws.com \
-U postgres \
-d patientping
```

Enter your database password when prompted.

---

## Step 5: Verify Connection

If successful, you will see:

```sql
patientping=>
```

You can run:

```sql
\dt
```

to list tables.

---

## Step 6: Exit Database

```sql
\q
```

---

# Connection Verification Summary

Successful connection means:

```text
DNS Works
VPC Routing Works
Security Groups Work
Database Works
Authentication Works
```

---

# Troubleshooting

### Connection Timeout

Usually:

```text
Security Group Problem
```

Fix:

* Ensure RDS inbound rule allows port 5432 from EC2 SG

---

### Authentication Failed

Usually:

```text
Wrong Username
Wrong Password
```

---

### Host Not Found

Usually:

```text
Wrong Endpoint
```

---

### Cannot SSH into EC2

Check:

```text
Security Group allows port 22 from your IP
Correct key (.pem) used
Correct username (ec2-user / ubuntu)
```

---

# Production Architecture

```text
Internet
     ↓
Load Balancer
     ↓
EC2 / NestJS
(Public or Private)
     ↓
Security Group
     ↓
PostgreSQL RDS
(Private Subnet)
```

Database remains hidden from the internet.

---

# Key Takeaways

### Inbound Rule

Controls:

```text
Who can connect TO me?
```

### Outbound Rule

Controls:

```text
Who can I connect TO?
```

### Security Group

```text
Stateful Firewall
```

### Best Practice

```text
EC2 → Public

RDS → Private

Allow PostgreSQL (5432)
only from EC2 Security Group
```

Never expose PostgreSQL directly to the internet.
