# AWS IAM
<img width="740" height="485" alt="image" src="https://github.com/user-attachments/assets/7eaaf83b-5adf-45cb-a345-b67c8d0b6039" />
<img width="726" height="617" alt="image" src="https://github.com/user-attachments/assets/58b310ef-95a4-46fb-bf6d-878b63ad9877" />
<img width="752" height="417" alt="image" src="https://github.com/user-attachments/assets/e597acc3-0dd3-40ec-af83-76efc68bf86b" />
<img width="742" height="287" alt="image" src="https://github.com/user-attachments/assets/ace2b69b-2145-4c39-ad22-9081b54083a3" />
# AWS IAM Users & Inline Policies Notes

## What is IAM?

**IAM (Identity and Access Management)** controls:

```text
Who can access AWS?
What can they do?
Which resources can they access?
```

Think of IAM as:

```text
AWS Security & Permissions System
```

---

# IAM User

An IAM User represents a person or application that needs AWS access.

Example:

```text
Developer
Admin
Intern
CI/CD Pipeline
```

Each user gets their own identity.

---

# Why Create Separate Users?

Bad Practice:

```text
Everyone uses Root Account
```

Problems:

* No accountability
* Full access to everything
* High security risk

---

Good Practice:

```text
Root Account
      ↓
Create IAM Users
      ↓
Give Only Required Permissions
```

This follows the:

```text
Principle of Least Privilege
```

Users only get the permissions they need.

---

# Assignment 1: Create IAM User

Create:

```text
User Name:
patientping-admin-vinny
```

No console access.

No permissions.

---

## Steps

```text
AWS Console
 ↓
IAM
 ↓
Users
 ↓
Create User
```

Configure:

```text
User Name:
patientping-admin-vinny

Console Access:
Disabled
```

Then:

```text
Next
 ↓
No Permissions
 ↓
Next
 ↓
Create User
```

Verify user appears in:

```text
IAM → Users
```

---

# IAM Policies

Policies define:

```text
What actions are allowed or denied?
```

Policies are written in JSON format.

---

# IAM Policy JSON Structure (Detailed Explanation)

A typical IAM policy JSON has the following structure:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ec2:Describe*"],
      "Resource": "*"
    }
  ]
}
```

---

## 1. Version

```json
"Version": "2012-10-17"
```

* Defines the policy language version
* Always use `"2012-10-17"` (latest and recommended)

---

## 2. Statement

```json
"Statement": [ ... ]
```

* Contains one or more permission rules
* Each rule is an object inside the array

---

## 3. Effect

```json
"Effect": "Allow"
```

Determines whether access is:

```text
Allow → Grants permission
Deny  → Explicitly blocks permission
```

---

## 4. Action

```json
"Action": ["ec2:DescribeInstances"]
```

Defines what operations are allowed.

Examples:

```text
ec2:DescribeInstances → View EC2 instances
s3:PutObject → Upload file to S3
rds:CreateDBInstance → Create database
```

You can also use wildcards:

```json
"Action": ["ec2:Describe*"]
```

This means:

```text
All EC2 read-only (describe) actions
```

---

## 5. Resource

```json
"Resource": "*"
```

Defines which resources the policy applies to.

Examples:

```text
"*" → All resources
Specific ARN → Only one resource
```

Example of specific resource:

```json
"Resource": "arn:aws:s3:::my-bucket/*"
```

---

## 6. Optional: Condition (Advanced)

```json
"Condition": {
  "IpAddress": {
    "aws:SourceIp": "192.168.1.0/24"
  }
}
```

* Adds extra rules
* Example: Allow access only from specific IP

---

# What is an Inline Policy?

An Inline Policy is attached directly to one IAM user.

Example:

```text
Vinny User
      ↓
Inline Policy
```

Only that user receives the permissions.

---

# Problem with Inline Policies

Suppose:

```text
10 Developers
```

Need same permissions.

You would need:

```text
10 Separate Policies
```

Hard to manage and update.

---

# Better Solution

Use:

```text
Managed Policies
```

which can be reused across multiple users.

---

# Assignment 2: Create EC2 Read-Only Policy

Goal:

```text
Vinny can view EC2 instances
But cannot modify anything
```

---

## Policy JSON

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ec2:Describe*"],
      "Resource": "*"
    }
  ]
}
```

---

## What This Allows

```text
View EC2 Instances
View Security Groups
View VPCs
View AMIs
View Launch Templates
```

---

## What This Does NOT Allow

```text
Launch Instances
Delete Instances
Modify Security Groups
Create Resources
```

Read-only access.

---

# More IAM Policy Examples

## Example 1: S3 Read-Only Access

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": "*"
    }
  ]
}
```

---

## Example 2: Allow EC2 Start/Stop Only

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:StartInstances",
        "ec2:StopInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

---

## Example 3: Deny Deleting S3 Buckets

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": ["s3:DeleteBucket"],
      "Resource": "*"
    }
  ]
}
```

---

# Steps to Create Inline Policy

Navigate:

```text
IAM
 ↓
Users
 ↓
patientping-admin-vinny
 ↓
Permissions
 ↓
Add Permissions
 ↓
Create Inline Policy
```

---

## Policy Editor

Choose:

```text
JSON
```

Delete existing content.

Paste:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ec2:Describe*"],
      "Resource": "*"
    }
  ]
}
```

---

## Create Policy

Click:

```text
Next
```

Policy Name:

```text
patientping-ec2-readonly
```

Then:

```text
Create Policy
```

---

# Verify

Navigate:

```text
IAM
 ↓
Users
 ↓
patientping-admin-vinny
 ↓
Permissions
```

You should see:

```text
patientping-ec2-readonly
```

attached.

---

# Real-World Example

```text
Developer
     ↓
Can View EC2

Cannot Delete EC2

Cannot Create EC2

Cannot Change Networking
```

Safe access for dashboards and monitoring.

---

# Key Takeaways

### IAM User

```text
Identity for a person or application.
```

### Policy

```text
Defines permissions using JSON.
```

### Effect

```text
Allow or Deny
```

### Action

```text
What operations are allowed?
```

### Resource

```text
Which AWS resource is affected?
```

### Inline Policy

```text
Policy attached directly to one user.
```

### Least Privilege

```text
Give only the permissions required.
```

---

# Big Picture

```text
Root Account
      ↓
IAM User (Vinny)
      ↓
Inline Policy
      ↓
ec2:Describe*
      ↓
Can View EC2
Cannot Modify EC2
```

