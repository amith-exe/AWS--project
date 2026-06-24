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
# Group
<img width="763" height="628" alt="image" src="https://github.com/user-attachments/assets/c2d9081b-917b-4ca6-8abc-afe0252ba9da" />
# AWS IAM Managed Policies & Groups - Step-by-Step Guide

## Goal

Move from:

```text
IAM User
    ↓
Inline Policy
```

To:

```text
IAM User
    ↓
IAM Group
    ↓
Customer Managed Policy
```

This is the AWS best practice.

---

# Why Not Inline Policies?

Problem:

```text
Vinny
  ↓
Inline Policy
```

If 10 developers need the same permission:

```text
10 Users
    ↓
10 Separate Inline Policies
```

Hard to manage.

---

# Better Architecture

```text
Customer Managed Policy
          ↓
IAM Group
          ↓
Multiple Users
```

Example:

```text
patientping-ec2-readonly
            ↓
patientping-ec2-readers
            ↓
Vinny
Sarah
John
Alex
```

One policy controls everyone.

---

# Part 1: Create Customer Managed Policy

Navigate:

```text
AWS Console
 ↓
IAM
 ↓
Policies
 ↓
Create Policy
```

---

## Policy Editor

Select:

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

Description (Optional):

```text
Read-only access to EC2 resources
```

Click:

```text
Create Policy
```

---

## Verify

Navigate:

```text
IAM
 ↓
Policies
```

Search:

```text
patientping-ec2-readonly
```

Confirm it exists.

---

# Part 2: Create IAM Group

Navigate:

```text
IAM
 ↓
User Groups
 ↓
Create Group
```

---

## Group Configuration

Group Name:

```text
patientping-ec2-readers
```

---

## Attach Policy

Search:

```text
patientping-ec2-readonly
```

Select it.

Click:

```text
Create User Group
```

---

## Verify

Navigate:

```text
IAM
 ↓
User Groups
```

Confirm:

```text
patientping-ec2-readers
```

exists.

---

# Part 3: Add Vinny to Group

Navigate:

```text
IAM
 ↓
User Groups
 ↓
patientping-ec2-readers
```

Open:

```text
Users Tab
```

Click:

```text
Add Users
```

---

## Select User

Choose:

```text
patientping-admin-vinny
```

Click:

```text
Add Users
```

---

## Verify

Inside:

```text
IAM
 ↓
User Groups
 ↓
patientping-ec2-readers
 ↓
Users
```

You should see:

```text
patientping-admin-vinny
```

listed.

---

# Part 4: Remove Inline Policy

Now Vinny receives permissions from the group.

The old inline policy is no longer needed.

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

---

## Remove Policy

Under:

```text
Permissions Policies
```

Locate:

```text
patientping-ec2-readonly
(Inline Policy)
```

Select it.

Click:

```text
Remove
```

Confirm removal.

---

## Verify Final Setup

Navigate:

```text
IAM
 ↓
Users
 ↓
patientping-admin-vinny
```

You should see:

```text
No Inline Policies
```

---

Navigate:

```text
IAM
 ↓
User Groups
 ↓
patientping-ec2-readers
```

You should see:

```text
Attached Policy:
patientping-ec2-readonly

User:
patientping-admin-vinny
```

---

# Final Architecture

```text
Customer Managed Policy
(patientping-ec2-readonly)
              ↓
IAM Group
(patientping-ec2-readers)
              ↓
IAM User
(patientping-admin-vinny)
```

---

# Benefits

### Easier Management

```text
Update Policy Once
 ↓
All Users Updated
```

---

### Reusable

```text
Add New Developer
 ↓
Add To Group
 ↓
Done
```

---

### Follows AWS Best Practices

```text
Users
 ↓
Groups
 ↓
Policies
```

instead of:

```text
Users
 ↓
Inline Policies
```

---

# Quick Verification Checklist

✅ Customer Managed Policy Created

```text
patientping-ec2-readonly
```

✅ User Group Created

```text
patientping-ec2-readers
```

✅ Policy Attached To Group

✅ Vinny Added To Group

✅ Inline Policy Removed

✅ Vinny Still Has EC2 Read-Only Access Through Group
# ROLES
<img width="725" height="457" alt="image" src="https://github.com/user-attachments/assets/5d3c816d-9ef3-4472-b3a9-4fee7610840a" />
<img width="741" height="560" alt="image" src="https://github.com/user-attachments/assets/767ef09f-11bb-4737-916a-12682da7fabe" />
<img width="736" height="532" alt="image" src="https://github.com/user-attachments/assets/a7ac57ae-57d2-4ec6-b3d0-41424b9c5fa5" />
# AWS IAM Roles for EC2 Notes

## What is an IAM Role?

An IAM Role is a set of permissions that AWS services can temporarily assume.

Unlike IAM Users:

```text
IAM User
    ↓
Used by Humans

IAM Role
    ↓
Used by AWS Services
```

Examples:

```text
EC2
Lambda
RDS
ECS
```

---

# Why Use Roles Instead of Access Keys?

Bad Practice:

```text
Store AWS Access Keys
inside EC2 Server
```

Problems:

* Keys can leak
* Difficult to rotate
* Security risk

---

Good Practice:

```text
EC2
 ↓
IAM Role
 ↓
Temporary Credentials
 ↓
AWS API Access
```

AWS automatically manages credentials.

---

# PatientPing Example

Goal:

```text
EC2 Instance
      ↓
Can View EC2 Resources
```

using:

```text
patientping-ec2-readonly-role
```

---

# Architecture

```text
EC2 Instance
      ↓
IAM Role
      ↓
Managed Policy
(patientping-ec2-readonly)
      ↓
ec2:Describe*
```

---

# Benefits

### More Secure

```text
No Access Keys Stored
```

### Easy Permission Management

```text
Change Policy
      ↓
EC2 Gets New Permissions
```

### AWS Best Practice

```text
EC2 → IAM Role → Policy
```

---

# Step 1: Create IAM Role

Navigate:

```text
AWS Console
 ↓
IAM
 ↓
Roles
 ↓
Create Role
```

---

## Trusted Entity

Select:

```text
Trusted Entity Type:
AWS Service

Use Case:
EC2
```

Click:

```text
Next
```

---

## Attach Policy

Search:

```text
patientping-ec2-readonly
```

Select the policy.

Click:

```text
Next
```

---

## Role Details

Role Name:

```text
patientping-ec2-readonly-role
```

Click:

```text
Create Role
```

---

## Verify

Navigate:

```text
IAM
 ↓
Roles
```

Confirm:

```text
patientping-ec2-readonly-role
```

exists.

---

# Step 2: Attach Role to EC2

Navigate:

```text
AWS Console
 ↓
EC2
 ↓
Instances
```

Select:

```text
patientping-web-v2
```

---

Open:

```text
Actions
 ↓
Security
 ↓
Modify IAM Role
```

---

Select:

```text
patientping-ec2-readonly-role
```

Click:

```text
Update IAM Role
```

---

# Verify Role Attachment

Navigate:

```text
EC2
 ↓
Instances
 ↓
patientping-web-v2
```

Look for:

```text
IAM Role
```

Expected:

```text
patientping-ec2-readonly-role
```

---

# Real-World Example

```text
EC2 Instance
      ↓
Needs S3 Access
      ↓
Attach S3 Role

EC2 Instance
      ↓
Needs DynamoDB Access
      ↓
Attach DynamoDB Role
```

No access keys needed.

---

# Key Takeaways

### IAM User

```text
Used by People
```

### IAM Group

```text
Manage Multiple Users
```

### Managed Policy

```text
Reusable Permissions
```

### IAM Role

```text
Permissions for AWS Services
```

### EC2 Role

```text
Allows EC2 to access AWS resources
without storing credentials.
```

---

# Big Picture

```text
Managed Policy
(patientping-ec2-readonly)
            ↓
IAM Role
(patientping-ec2-readonly-role)
            ↓
EC2 Instance
(patientping-web-v2)
            ↓
Can View EC2 Resources
```
# Deny Policies

<img width="743" height="633" alt="image" src="https://github.com/user-attachments/assets/4d49ae2f-df0e-4f1d-8222-d04ef3a7210d" />
# AWS IAM Deny Policies Notes

## What is a Deny Policy?

A Deny Policy explicitly blocks access to AWS resources.

Example:

```text
Allow Policy
     +
Deny Policy
     ↓
Deny Wins
```

In AWS IAM:

```text
Explicit Deny
      >
Allow
```

Deny always takes priority.

---

# Why Use Deny Policies?

Normally, follow:

```text
Principle of Least Privilege
```

Meaning:

```text
Only grant permissions that are needed.
```

Do NOT do:

```text
Allow Everything
     ↓
Deny Dangerous Actions
```

Instead:

```text
Start With No Access
      ↓
Add Permissions As Needed
```

---

# When Are Deny Policies Useful?

## 1. Compromised EC2 Instance

Example:

```text
EC2 Hacked
      ↓
Attach Deny-All Policy
      ↓
AWS Access Immediately Blocked
```

---

## 2. Rogue Application

Example:

```text
Application Bug
      ↓
Creating Thousands of Resources
      ↓
Attach Deny-All
      ↓
Stops Damage
```

---

## 3. Security Emergency

Example:

```text
Data Breach
      ↓
Lock Down Resource
      ↓
Investigate
```

---

# Deny-All Policy

Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyEverything",
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*"
    }
  ]
}
```

---

## Meaning

### Effect

```json
"Effect": "Deny"
```

Block access.

---

### Action

```json
"Action": "*"
```

All AWS actions.

Examples:

```text
EC2
S3
RDS
IAM
CloudWatch
Everything
```

---

### Resource

```json
"Resource": "*"
```

All AWS resources.

---

# Result

```text
Role
 ↓
Allow Policy
 ↓
Can Access AWS

Role
 ↓
Allow Policy
 +
Deny-All Policy
 ↓
Cannot Access AWS
```

Deny wins.

---

# Step 1: Verify EC2 Has Access

SSH into EC2:

```bash
ssh patientping
```

Run:

```bash
aws ec2 describe-instances --no-cli-pager
```

Expected:

```text
Large JSON Output
```

This proves:

```text
EC2 Role Works
```

---

# Step 2: Create Deny-All Policy

Navigate:

```text
AWS Console
 ↓
IAM
 ↓
Policies
 ↓
Create Policy
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
      "Sid": "DenyEverything",
      "Effect": "Deny",
      "Action": "*",
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
patientping-deny-all
```

Click:

```text
Create Policy
```

---

# Step 3: Attach Policy to Role

Navigate:

```text
IAM
 ↓
Roles
 ↓
patientping-ec2-readonly-role
```

Open:

```text
Permissions
 ↓
Add Permissions
 ↓
Attach Policies
```

Search:

```text
patientping-deny-all
```

Select it.

Click:

```text
Add Permissions
```

---

# Step 4: Test Again

Return to EC2.

Run:

```bash
aws ec2 describe-instances --no-cli-pager
```

Expected:

```text
AccessDenied
```

or

```text
UnauthorizedOperation
```

---

# Why Did It Stop Working?

Before:

```text
EC2
 ↓
Role
 ↓
Allow EC2 Describe
 ↓
Success
```

After:

```text
EC2
 ↓
Role
 ↓
Allow EC2 Describe

AND

Deny Everything
 ↓
Access Denied
```

Because:

```text
Explicit Deny
Always Wins
```

---

# Real Production Use

If a server is compromised:

```text
EC2 Compromised
      ↓
Attach Deny-All Policy
      ↓
AWS Access Blocked
      ↓
Investigate Incident
```

This acts like an emergency kill switch.

---

# Key Takeaways

### Allow Policy

```text
Grants Permission
```

### Deny Policy

```text
Blocks Permission
```

### Explicit Deny

```text
Always Wins
```

### Deny-All Policy

```text
Blocks All AWS Actions
```

### Common Use

```text
Security Incidents
Compromised Servers
Emergency Lockdowns
```

---

# Big Picture

```text
EC2 Instance
      ↓
IAM Role
      ↓
Allow Policy
      ↓
Can Access AWS

EC2 Instance
      ↓
IAM Role
      ↓
Allow Policy
      +
Deny-All Policy
      ↓
Cannot Access AWS
```

**Rule of Thumb:**

```text
Least Privilege First

Deny Policies Only For
Emergency Boundaries
And Security Controls
```


