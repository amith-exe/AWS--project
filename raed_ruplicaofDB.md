# AWS RDS Read Replicas Notes

## What is a Read Replica?

A **Read Replica** is a copy of your primary database that is used only for reading data.

Think of it as:

```text
Primary Database
      ↓
Copies Data
      ↓
Read Replica
```

The replica continuously receives updates from the primary database.

---

# Why Do We Need Read Replicas?

Most applications perform far more reads than writes.

Example: Discord

```text
1000 Users Reading Messages
50 Users Sending Messages
```

Reads are much more common than writes.

Without replicas:

```text
All Reads
      ↓
Primary Database
      ↓
High Load
```

The primary database becomes a bottleneck.

---

# Solution

Use Read Replicas.

```text
Users
  ↓
SELECT Queries
  ↓
Read Replica

INSERT/UPDATE/DELETE
  ↓
Primary Database
```

This distributes the workload.

---

# How Read Replication Works

AWS uses:

```text
Asynchronous Replication
```

Meaning:

```text
Primary Database
      ↓
Copies Changes
      ↓
Replica Database
```

Replication happens continuously.

---

# Read Replica Architecture

```text
Application
      ↓
      ├── Reads (SELECT)
      ↓
      │
      ├── Read Replica
      │
      └── Writes
             ↓
      Primary Database
```

---

# What Queries Go Where?

## Read Operations

Handled by replicas.

Examples:

```sql
SELECT * FROM users;

SELECT * FROM bookings;

SELECT * FROM venues;
```

---

## Write Operations

Handled by primary database only.

Examples:

```sql
INSERT INTO users;

UPDATE bookings;

DELETE FROM venues;

UPSERT data;
```

---

# Important Limitation

Read Replicas are:

```text
Read Only
```

You cannot run:

```sql
INSERT
UPDATE
DELETE
```

on a replica.

Only:

```sql
SELECT
```

queries.

---

# Replication Lag

Replication is not instant.

Because replication is asynchronous:

```text
Primary
      ↓
Few Seconds Delay
      ↓
Replica
```

This delay is called:

```text
Replication Lag
```

---

# Example

User creates booking:

```text
12:00:00
Booking Created
```

Replica may receive update at:

```text
12:00:02
```

or

```text
12:00:05
```

For a few seconds:

```text
Primary = Updated
Replica = Old Data
```

---

# When Is Lag Acceptable?

Good:

```text
Chat History
Product Catalog
Blog Posts
Venue Listings
Analytics
Reports
```

---

# When Is Lag NOT Acceptable?

Bad:

```text
Bank Transactions
Payments
Inventory Systems
Stock Trading
Real-Time Counters
```

These require strong consistency.

---

# Benefits of Read Replicas

## 1. Better Read Performance

Instead of:

```text
1000 Reads
      ↓
Primary DB
```

Use:

```text
500 Reads → Replica 1
500 Reads → Replica 2
```

Much less load.

---

## 2. Scalability

Traffic grows:

```text
Primary
      ↓
1 Replica
      ↓
3 Replicas
      ↓
10 Replicas
```

Scale reads without changing the primary.

---

## 3. Reduce Load on Primary

Primary can focus on:

```text
INSERT
UPDATE
DELETE
```

while replicas handle:

```text
SELECT
```

---

## 4. Geographic Distribution

Place replicas closer to users.

Example:

```text
Primary → Mumbai

Replica → Frankfurt

Replica → Singapore
```

European users read from Frankfurt.

Asian users read from Singapore.

Result:

```text
Lower Latency
Faster Responses
```

---

## 5. High Availability

If primary fails:

```text
Primary
   ↓
Failure
```

Replica may be promoted to become:

```text
New Primary
```

This helps disaster recovery.

---

# Read Replica vs Backup

Many beginners confuse these.

---

## Read Replica

Purpose:

```text
Performance
Scaling
High Availability
```

Updates continuously.

Can serve live traffic.

---

## Backup

Purpose:

```text
Data Recovery
Disaster Recovery
```

Used only when restoring data.

Cannot serve application traffic.

---

# Example

```text
Primary Database
       ↓
Read Replica
       ↓
Handles Queries

Backup
       ↓
Stored in S3
       ↓
Used for Recovery
```

Different purposes.

---

# Read Replica vs Multi-AZ

## Read Replica

Goal:

```text
Increase Read Performance
```

Can serve application queries.

---

## Multi-AZ

Goal:

```text
High Availability
```

Standby database remains hidden.

Cannot serve application traffic.

---

# PatientPing Example

Primary:

```text
patientping-db
```

Replica:

```text
patientping-replica
```

Both contain:

```text
Same Data
```

But:

```text
Writes → patientping-db

Reads → patientping-replica
```

---

# Production Example

BookMyVenue:

```text
Users Search Venues
Users Browse Images
Users Read Reviews
```

Thousands of read requests.

Architecture:

```text
NestJS API
      ↓
      ├── Writes
      ↓
      │
      ├── Primary PostgreSQL
      │
      └── Reads
             ↓
      Read Replicas
```

This is how large applications scale databases.

---

# Creating a Read Replica

## Step-by-Step (AWS Console)

```text
RDS
 ↓
Databases
 ↓
Select Primary DB
 ↓
Actions
 ↓
Create Read Replica
```

---

## Detailed Configuration

### 1. Basic Settings

```text
DB Instance Identifier:
patientping-replica

Source DB:
patientping-db (auto-selected)
```

---

### 2. Instance Configuration

```text
DB Instance Class:
Same or smaller than primary (for cost saving)

Storage:
Automatically inherited from primary

Storage Type:
gp2 / gp3 / io1 (same as primary)
```

---

### 3. Connectivity

```text
VPC:
Same as primary

Subnet Group:
Same as primary

Availability Zone:
Choose same or different AZ

Public Access:
No (recommended for production)

Security Group:
Same as primary OR custom SG allowing app access
```

---

### 4. Database Options

```text
Port:
Same as primary (e.g., 5432 for PostgreSQL)

Parameter Group:
Same as primary

Option Group:
Same as primary
```

---

### 5. Replication Settings

```text
Replication Type:
Asynchronous (default)

Multi-AZ:
Optional (usually disabled for replicas)

Auto Minor Version Upgrade:
Enabled (recommended)
```

---

### 6. Monitoring & Maintenance

```text
Enable Monitoring:
Optional (CloudWatch)

Backup:
Optional (replicas can have backups too)

Maintenance Window:
Choose preferred time
```

---

### 7. Create Replica

Click:

```text
Create Read Replica
```

---

## What Happens After Creation?

```text
Status:
Creating
      ↓
Backing up primary
      ↓
Initializing replica
      ↓
Available
```

Time:

```text
5–15 minutes (depends on DB size)
```

---

## After Creation

You will get:

```text
Replica Endpoint:
patientping-replica.xxxxxx.rds.amazonaws.com
```

Use this endpoint for:

```text
SELECT queries only
```

---

## Important Notes

### 1. Application Changes Required

Your app must route queries:

```text
Writes → Primary Endpoint
Reads → Replica Endpoint
```

---

### 2. Multiple Replicas

You can create:

```text
1 → 5 → 10 replicas
```

Each replica gets its own endpoint.

---

### 3. Cross-Region Replicas

You can create replicas in another region:

```text
Primary → Mumbai
Replica → Frankfurt
```

Useful for:

```text
Disaster Recovery
Global Applications
```

---

### 4. Promotion to Primary

If needed:

```text
Select Replica
 ↓
Actions
 ↓
Promote
```

This converts replica into a standalone primary DB.

---

# Key Takeaways

### Primary Database

```text
Handles Reads + Writes
```

---

### Read Replica

```text
Handles Reads Only
```

---

### Replication

```text
Asynchronous
```

---

### Replication Lag

```text
Few Seconds Delay
```

---

### Benefits

```text
Better Performance
Scalability
Reduced Load
Geographic Distribution
High Availability
```

---

# Big Picture

```text
Application
      ↓
      ├── INSERT/UPDATE/DELETE
      ↓
      │
      ├── Primary Database
      │
      └── SELECT Queries
             ↓
      Read Replicas
```

**Rule of Thumb:**
Use the **Primary DB for writes**, use **Read Replicas for reads**, and use **Backups for recovery**.
