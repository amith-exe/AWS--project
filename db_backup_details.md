# AWS RDS Storage, IOPS & Backups Notes

## 1. RDS Storage Types

When creating an RDS database, AWS asks how the database files should be stored.

Storage affects:

* Performance
* Latency
* Cost
* Number of database operations per second

---

# General Purpose SSD (gp3)

Recommended default for most applications.

Features:

```text
Baseline Performance: 3000 IOPS
Scalable up to: 16000 IOPS
Low Cost
Good Performance
```

Best for:

* Web applications
* NestJS backends
* PostgreSQL databases
* Small and medium workloads

---

# Provisioned IOPS SSD (io1 / io2)

Designed for high-performance databases.

Features:

```text
Guaranteed IOPS
Low Latency
Predictable Performance
Higher Cost
```

Best for:

* Banking systems
* ERP systems
* High transaction databases
* Enterprise applications

---

# gp2 vs gp3

## gp2

Older generation.

```text
Performance depends on storage size.
More storage = More IOPS.
```

Example:

```text
20 GB → Lower IOPS
200 GB → Higher IOPS
```

---

## gp3

Newer generation.

```text
Storage and IOPS are independent.
```

Example:

```text
20 GB
3000 IOPS
```

without needing extra storage.

---

# Recommendation

```text
Learning → gp2
Production → gp3
```

Use gp3 until performance becomes a bottleneck.

---

# 2. What is IOPS?

IOPS =

```text
Input / Output Operations Per Second
```

Measures how many storage operations can be performed every second.

Think of it as:

```text
Database Speed Limit
```

---

# Example

Database needs:

```text
Read User Data
Write Booking
Update Payment
Store Logs
```

Each operation uses I/O.

---

### Higher IOPS

```text
More Reads
More Writes
Faster Database
```

---

### Lower IOPS

```text
Slower Queries
Longer Wait Times
```

---

# Highway Analogy

```text
IOPS = Number of Cars
Road = Storage
```

Small road:

```text
1000 Cars/Second
```

Large road:

```text
16000 Cars/Second
```

More traffic can flow.

---

# For Your Project

BookMyVenue:

```text
PostgreSQL
Few Users
Few Bookings
```

Default:

```text
3000 IOPS
```

is more than enough.

You won't need Provisioned IOPS for a long time.

---

# 3. RDS Backups

Databases fail.

People make mistakes.

Applications contain bugs.

Backups protect your data.

---

# AWS Automatic Backups

RDS automatically creates:

```text
Daily Snapshots
+
Continuous Transaction Log Backups
```

---

# Daily Snapshot

```text
Complete Database Copy
```

Example:

```text
Monday 2:00 AM
Full Database Backup
```

---

# Transaction Logs

Every database change is recorded.

Example:

```text
Insert Booking
Update User
Delete Venue
```

All stored in transaction logs.

---

# Why Both Are Needed

```text
Snapshot
     +
Transaction Logs
     =
Point-in-Time Recovery
```

---

# Retention Period

How long backups are stored.

Typical values:

```text
1 Day
7 Days
30 Days
35 Days
```

Example:

```text
Retention = 7 Days
```

AWS keeps:

```text
Last 7 Days of Backups
```

---

# Backup Storage

Backups are stored separately in:

```text
Amazon S3
```

Not on the database server itself.

---

# Cost

Backup storage costs money.

Example:

```text
20 GB Database
7 Days Retention
```

Can result in much more than:

```text
20 GB Backup Storage
```

because multiple backups are stored.

---

# 4. Restore Options

AWS provides multiple recovery methods.

---

# Point-In-Time Recovery (PITR)

Restore database to an exact moment.

Example:

```text
12:00 PM → Everything OK
12:05 PM → Bad Script Deletes Data
```

Restore to:

```text
12:04:59 PM
```

Data recovered.

---

# Snapshot Restore

Restore from a specific backup snapshot.

Example:

```text
Restore Monday Backup
```

Fast and simple.

---

# Manual Snapshots

You can create snapshots yourself.

Example:

```text
Before Migration
Before Major Update
Before Prisma Migration
```

Create:

```text
Manual Snapshot
```

If something breaks:

```text
Restore Snapshot
```

---

# Important: Restores Create New Databases

AWS never overwrites the original database.

Instead:

```text
Original Database
       ↓
Restore
       ↓
New Database Instance
```

This is safer.

---

# Why This Is Useful

## Testing

```text
Production DB
      ↓
Restore
      ↓
Test Environment
```

---

## Cloning

```text
Restore Backup
      ↓
Create Exact Copy
```

---

## Migration

Instead of:

```text
Export Data
Import Data
```

you can:

```text
Restore Snapshot
```

Much faster and more reliable.

---

# Production Best Practices

### Storage

```text
Development → gp2 or gp3
Production → gp3
High Performance → io2
```

---

### Backups

```text
Development → 1 Day
Production → 7–35 Days
```

---

### Before Major Changes

Always create:

```text
Manual Snapshot
```

before:

* Prisma Migrations
* PostgreSQL Upgrades
* Large Data Imports
* Schema Changes

---

# Big Picture

```text
Application
      ↓
RDS PostgreSQL
      ↓
gp3 Storage
      ↓
3000+ IOPS
      ↓
Automatic Backups
      ↓
Snapshots + Transaction Logs
      ↓
Point-In-Time Recovery
      ↓
Fast Restore & Cloning
```

---

# Key Takeaways

### IOPS

```text
Database Storage Speed
```

---

### gp3

```text
Recommended Default
```

---

### io2

```text
High Performance Storage
```

---

### Backups

```text
Daily Snapshots
+
Transaction Logs
```

---

### Point-In-Time Recovery

```text
Restore to Exact Time
```

---

### Snapshot Restore

```text
Restore from Known Good State
```

---

### Best Rule

```text
One Backup = No Backup

Multiple Backups = Safety
```
