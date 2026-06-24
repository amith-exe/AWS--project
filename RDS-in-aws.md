# AWS RDS PostgreSQL Notes

## What is RDS?

**Amazon RDS (Relational Database Service)** is AWS's managed database service.

Instead of installing PostgreSQL yourself on an EC2 server, AWS manages:

* Installation
* Updates
* Backups
* Monitoring
* Storage
* Recovery

You only manage the database itself.

---

# Why Use RDS?

Without RDS:

```text
EC2
 ↓
Install PostgreSQL
 ↓
Configure Backups
 ↓
Manage Updates
 ↓
Handle Recovery
```

With RDS:

```text
Create RDS
 ↓
AWS Handles Everything
```

---

# Why Put RDS in Private Subnets?

Databases should NOT be exposed to the internet.

Bad:

```text
Internet
    ↓
Database
```

Good:

```text
Internet
    ↓
Application Server
    ↓
Database (Private Subnet)
```

Only your backend can access the database.

---

# Architecture

```text
Public Subnet
     ↓
NestJS / EC2
     ↓
Private Subnet
     ↓
PostgreSQL RDS
```

This is the standard production setup.

---

# Step 1: Create DB Subnet Group

A DB Subnet Group tells AWS:

```text
These are the subnets
where the database is allowed to live.
```

Create:

```text
Name:
patientping-private-subnet-group
```

Select:

```text
patientping-private-a
patientping-private-b
```

Why 2 subnets?

```text
High Availability
Future Failover Support
```

---

# Step 2: Create PostgreSQL Database

Navigate:

```text
AWS Console
 ↓
Aurora & RDS
 ↓
Databases
 ↓
Create Database
```

Select:

```text
Full Configuration
```

---

# Database Settings

### Engine

```text
PostgreSQL
```

### Version

```text
PostgreSQL 17
```

### Template

```text
Free Tier
```

or

```text
Dev/Test
```

---

### Availability Options (Important)

AWS provides multiple availability configurations:

#### 1. Single-AZ (Basic Setup)

```text
Single-AZ
```

* Runs database in one Availability Zone
* Cheapest option
* No automatic failover
* Good for development or small apps

---

#### 2. Multi-AZ (High Availability)

```text
Multi-AZ
```

* Primary DB + standby replica in another AZ
* Automatic failover if primary fails
* Higher cost
* Recommended for production

---

#### 3. Multi-AZ DB Cluster (New Option)

```text
Multi-AZ DB Cluster
```

* Multiple readable instances
* Faster failover
* Better performance
* Supports read scaling
* More expensive but highly reliable

---

#### 4. Read Replicas

```text
Read Replicas
```

* Used for scaling read traffic
* Can be in same or different region
* Not used for failover (unless promoted manually)

---

### Recommendation

```text
Dev/Test → Single-AZ
Production → Multi-AZ or Multi-AZ Cluster
```

---

# Database Configuration

### Database Identifier

```text
patientping-db
```

This is the database server name.

---

### Username

```text
postgres
```

Default PostgreSQL admin user.

---

### Password

Create a strong password.

Example:

```text
PatientPing2025
```

Use:

```text
Letters + Numbers
```

(Alphanumeric)

Save this password safely.

You'll need it later for NestJS.

---

# Instance Configuration

### Instance Type

```text
db.t3.micro
```

Small and inexpensive.

---

### Storage

```text
20 GiB
```

Default is fine.

---

### Storage Autoscaling

```text
Disable
```

Keeps costs predictable.

---

# Connectivity Configuration

### Connect to EC2

```text
Don't Connect
```

We'll configure this later.

---

### VPC

```text
patientping
```

Must be same VPC as your backend.

---

### DB Subnet Group

```text
patientping-private-subnet-group
```

---

### Public Access

```text
NO
```

Very important.

This means:

```text
Internet
    ✘
Cannot access database
```

---

### Security Group

Create:

```text
patientping-rds-sg
```

Controls who can access PostgreSQL.

---

### Availability Zone

```text
us-east-1a
```

(For Single-AZ setup only. Multi-AZ will automatically use multiple zones.)

---

# Monitoring

Select:

```text
Database Insights Standard
```

Disable:

```text
Performance Insights
```

to reduce complexity and cost.

---

# Additional Configuration

### Database Name

```text
patientping
```

This creates:

```sql
CREATE DATABASE patientping;
```

automatically.

---

### Backups

```text
Enabled
Retention: 1 Day
```

Allows recovery if something goes wrong.

---

### Encryption

```text
Enable if available
```

Protects stored data.

---

### Maintenance

```text
Default Settings
```

---

# Wait for Availability

RDS takes:

```text
5-10 Minutes
```

to provision.

Status changes:

```text
Creating
    ↓
Available
```

---

# Important Information to Save

After creation, note:

### Endpoint

Example:

```text
patientping-db.abc123.us-east-1.rds.amazonaws.com
```

This is your database host.

---

### Database Name

```text
patientping
```

---

### Username

```text
postgres
```

---

### Password

```text
Your chosen password
```

---

# Future NestJS Connection

Example:

```env
DATABASE_URL=postgresql://postgres:PASSWORD@patientping-db.xxxxx.us-east-1.rds.amazonaws.com:5432/patientping
```

Prisma will use this later.

---

# Security Best Practice

```text
Public Access = No
```

Only allow:

```text
NestJS Server
        ↓
Security Group Rules
        ↓
RDS Database
```

Never expose PostgreSQL directly to the internet.

---

# Big Picture

```text
Internet
    ↓
EC2 / NestJS Backend
(Public Subnet)
    ↓
Security Group
    ↓
PostgreSQL RDS
(Private Subnet)
```

This is how most production applications are deployed on AWS.

---

# One-Line Definition

**Amazon RDS is a managed database service that runs PostgreSQL, MySQL, and other databases while AWS handles backups, maintenance, storage, and recovery.**
