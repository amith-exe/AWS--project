# AWS Systems Manager (SSM) Parameter Store Notes

# The Problem SSM Solves

Every application needs configuration.

Examples:

```text
Database URL
API Keys
JWT Secret
Domain Name
Debug Mode
SMTP Credentials
Redis URL
```

Many beginners store them directly in:

```env
.env
```

Example:

```env
DATABASE_URL=postgresql://...
JWT_SECRET=mysecret
```

This works locally but becomes difficult in production.

Problems:

```text
Secrets stored on servers
Hard to update
Hard to manage multiple environments
Security risks
No centralized management
```

---

# What is SSM Parameter Store?

SSM Parameter Store is AWS's centralized configuration storage service.

Think of it as:

```text
AWS Cloud Key-Value Vault
```

Applications can fetch configuration whenever they need it.

---

# Architecture

Without SSM:

```text
EC2
 ↓
.env File
 ↓
Application
```

With SSM:

```text
EC2
 ↓
IAM Role
 ↓
SSM Parameter Store
 ↓
Application
```

Configuration is stored centrally.

---

# What Can Be Stored?

Examples:

```text
DATABASE_URL
JWT_SECRET
REDIS_URL
API_KEYS
APP_NAME
DEBUG_MODE
```

---

# Parameter Types

## String

Regular text.

Example:

```text
/APP_NAME

PatientPing
```

---

## StringList

Multiple values.

Example:

```text
server1,server2,server3
```

---

## SecureString

Encrypted values.

Example:

```text
JWT_SECRET
Database Password
API Keys
```

Uses:

```text
AWS KMS Encryption
```

---

# Parameter Paths

Parameters can be organized into folders.

Example:

```text
/global/APP_NAME

/prod/DATABASE_URL

/dev/DATABASE_URL

/secrets/JWT_SECRET
```

Think of it like directories.

---

# Why Use Paths?

Without organization:

```text
DATABASE_URL
JWT_SECRET
APP_NAME
REDIS_URL
```

Messy.

With paths:

```text
/prod/*
/dev/*
/staging/*
```

Easy to manage.

---

# Parameter Versioning

Whenever a value changes:

```text
DATABASE_URL v1
DATABASE_URL v2
DATABASE_URL v3
```

AWS keeps versions.

Useful for:

```text
Auditing
Rollback
Debugging
```

---

# SSM Stores Everything as Strings

Important concept:

```text
Everything = String
```

Examples:

```text
5
↓
"5"

true
↓
"true"

3.14
↓
"3.14"
```

Applications must convert values.

---

# Example

Stored:

```text
PORT=5432
```

Retrieved:

```python
"5432"
```

Convert:

```python
port = int("5432")
```

Result:

```python
5432
```

---

# Why Not Hardcode Secrets?

Bad:

```python
JWT_SECRET="abc123"
```

Problems:

```text
Visible in code
Visible in GitHub
Hard to change
```

---

Good:

```python
JWT_SECRET = SSM.get("/JWT_SECRET")
```

Now secrets stay in AWS.

---

# IAM and SSM

SSM parameters are protected by IAM.

By default:

```text
Nobody Can Read Them
```

You must grant permission.

---

# Example IAM Policy

```json
{
  "Effect": "Allow",
  "Action": [
    "ssm:GetParameter",
    "ssm:GetParameters"
  ],
  "Resource": [
    "arn:aws:ssm:*:*:parameter/DATABASE_URL",
    "arn:aws:ssm:*:*:parameter/CMO_NAME"
  ]
}
```

---

# Meaning

Allow:

```text
Read DATABASE_URL
Read CMO_NAME
```

Deny:

```text
Everything Else
```

Principle of Least Privilege.

---

# Why Attach Policy to Role?

Instead of:

```text
EC2
 ↓
AWS Keys
```

Use:

```text
EC2
 ↓
IAM Role
 ↓
SSM Policy
 ↓
SSM Access
```

More secure.

---

# PatientPing Architecture

```text
SSM Parameter Store
      ↓
DATABASE_URL
CMO_NAME
      ↓
IAM Policy
      ↓
IAM Role
      ↓
EC2 Instance
      ↓
PatientPing App
```

---

# How App Loads Parameters

Startup:

```text
App Starts
     ↓
Request SSM Parameters
     ↓
IAM Checks Permissions
     ↓
SSM Returns Values
     ↓
Application Uses Config
```

---

# Fallback Strategy

Many applications do:

```text
Try SSM
    ↓
If Failed
    ↓
Use .env
```

Useful for local development.

---

# Benefits of SSM

### Centralized

```text
One Place For Config
```

### Secure

```text
IAM Controlled
Encryption Available
```

### No Secrets In Code

```text
GitHub Safe
```

### Easy Updates

```text
Change Value Once
All Servers Use It
```

### Auditable

```text
Track Changes
```

---

# Real Production Example

BookMyVenue:

```text
/prod/DATABASE_URL
/prod/JWT_SECRET
/prod/GOOGLE_MAPS_API_KEY

/dev/DATABASE_URL
/dev/JWT_SECRET
```

NestJS fetches them at startup.

---

# Step-by-Step Guide

## Part 1: Create Parameters

Navigate:

```text
AWS Console
 ↓
Systems Manager
 ↓
Parameter Store
 ↓
Create Parameter
```

---

### DATABASE_URL

Name:

```text
/DATABASE_URL
```

Tier:

```text
Standard
```

Type:

```text
String
```

Value:

```text
postgresql://postgres:PASSWORD@YOUR_RDS_ENDPOINT:5432/patientping
```

Click:

```text
Create Parameter
```

---

### CMO_NAME

Name:

```text
/CMO_NAME
```

Type:

```text
String
```

Value:

```text
Dr. Strangelove
```

Click:

```text
Create Parameter
```

---

## Part 2: Create SSM Access Policy

Navigate:

```text
IAM
 ↓
Roles
 ↓
patientping-ec2-readonly-role
 ↓
Permissions
 ↓
Create Inline Policy
```

Choose JSON:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ssm:GetParameter",
        "ssm:GetParameters"
      ],
      "Resource": [
        "arn:aws:ssm:*:*:parameter/DATABASE_URL",
        "arn:aws:ssm:*:*:parameter/CMO_NAME"
      ]
    }
  ]
}
```

Policy Name:

```text
patientping-ssm-access
```

Create Policy.

---

## Part 3: Verify Role

Role should now contain:

```text
patientping-ec2-readonly
patientping-ssm-access
```

---

## Part 4: SSH Into EC2

```bash
ssh patientping
```

---

## Part 5: Start Application

```bash
cd ~/patientping-web

uv run patientping.py
```

Expected:

```text
Loaded DATABASE_URL and CMO_NAME from SSM
```

---

## Part 6: Test Web Application

Open:

```text
http://YOUR_PUBLIC_IP:8080
```

Expected:

```text
PatientPing Chief Medical Officer:
Dr. Strangelove
```

---

## Part 7: Remove .env

Since config now comes from SSM:

```bash
rm .env
```

Application should still work.

---

# Useful CLI Commands

View parameter:

```bash
aws ssm get-parameter --name /DATABASE_URL
```

```bash
aws ssm get-parameter --name /CMO_NAME
```

Check current role:

```bash
aws sts get-caller-identity
```

List role policies:

```bash
aws iam list-role-policies \
--role-name patientping-ec2-readonly-role
```

---

# Key Takeaways

### SSM Parameter Store

```text
AWS Central Configuration Store
```

### Stores

```text
Configuration
Secrets
Application Settings
```

### Access Control

```text
IAM Policies
IAM Roles
```

### Everything Is Stored As

```text
String
```

### Production Best Practice

```text
Local Development → .env

Production → SSM Parameter Store
```
