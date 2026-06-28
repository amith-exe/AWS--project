# AWS Monitoring Notes (CloudWatch, CloudWatch Agent, CloudTrail)

---

# Why Monitoring is Important

Once an application is deployed, we need to know:

* Is the server healthy?
* Is the application running?
* Is CPU or memory high?
* Did someone change AWS resources?
* Should we notify the team when something goes wrong?

AWS provides four major services:

```text
CloudWatch Metrics  → Monitor resource usage
CloudWatch Agent    → Monitor OS & application logs
CloudWatch Alarms   → Send alerts automatically
CloudTrail          → Audit AWS API activity
```

---

# 1. CloudWatch Metrics (External Monitoring)

## What is External Monitoring?

AWS monitors EC2 **from outside the operating system**.

It collects metrics without installing anything.

Examples:

```text
CPU Usage
Network In/Out
Disk Read/Write
Status Checks
```

These are available automatically.

---

## Why Use CloudWatch Metrics?

It helps answer questions like:

```text
Is the server overloaded?
Is CPU too high?
Is the server idle?
Is network traffic increasing?
```

---

## Architecture

```text
EC2 Instance
      ↓
AWS Hypervisor
      ↓
CloudWatch Metrics
      ↓
Dashboard
```

No software installation required.

---

## Creating a Dashboard

Navigate:

```text
AWS Console
 ↓
CloudWatch
 ↓
Dashboards
 ↓
Create Dashboard
```

Name:

```text
patientping-dashboard
```

Add Widget:

```text
Line Graph
 ↓
EC2
 ↓
Per-Instance Metrics
 ↓
CPUUtilization
 ↓
Create Widget
```

Set Time Range:

```text
15 Minutes
```

Save Dashboard.

---

## Testing CPU Usage

SSH:

```bash
ssh patientping
```

Run:

```bash
yes > /dev/null
```

This continuously prints "y" and discards the output, forcing the CPU to work.

Leave running:

```text
5 Minutes
```

CloudWatch should show a CPU spike.

Stop:

```text
Ctrl + C
```

---

# 2. CloudWatch Agent (Internal Monitoring)

## Why CloudWatch Agent?

CloudWatch Metrics only monitors from outside.

It cannot see:

```text
Memory Usage
Disk Usage
Swap Usage
Application Logs
```

CloudWatch Agent runs **inside the operating system**.

---

## What Can It Monitor?

Examples:

```text
RAM Usage
Disk Space
Swap Usage
Application Logs
System Logs
```

---

## Architecture

```text
EC2
     ↓
CloudWatch Agent
     ↓
CloudWatch Logs
CloudWatch Metrics
```

---

## Why IAM Permission?

The agent sends data to AWS.

Therefore it needs permission.

Policy allows:

```text
Create Log Groups
Create Log Streams
Upload Logs
```

---

# CloudWatch Agent Workflow

```text
Install Agent
       ↓
Create Config File
       ↓
Start Agent
       ↓
Collect Logs & Metrics
       ↓
Send To CloudWatch
```

---

# CloudWatch Agent Configuration

Configuration file:

```text
/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/patientping-monitoring.json
```

Defines:

```text
Which logs?
Which metrics?
How often?
```

---

## Metrics Collected

Memory:

```text
mem_used_percent
```

Swap:

```text
swap_used_percent
```

Collection Interval:

```text
60 Seconds
```

---

## Logs Collected

Example:

```text
/var/log/patientping.log
```

CloudWatch uploads this log automatically.

---

# Setting Up CloudWatch Agent

## Step 1

Create IAM Role:

```text
IAM
 ↓
Roles
 ↓
Create Role
```

Trusted Entity:

```text
AWS Service
EC2
```

Role Name:

```text
patientping-monitoring-role
```

---

## Step 2

Create CloudWatch Inline Policy.

Navigate:

```text
IAM
 ↓
Roles
 ↓
patientping-monitoring-role
 ↓
Permissions
 ↓
Create Inline Policy
```

Paste provided JSON.

Policy Name:

```text
patientping-cloudwatch-logs-access
```

---

## Step 3

Create another inline policy.

Name:

```text
patientping-ssm-access
```

Paste SSM JSON.

This allows the application to continue reading SSM parameters.

---

## Step 4

Attach Role to EC2.

Navigate:

```text
EC2
 ↓
Instances
 ↓
patientping-web-v2
 ↓
Actions
 ↓
Security
 ↓
Modify IAM Role
```

Select:

```text
patientping-monitoring-role
```

Update.

---

## Step 5

SSH:

```bash
ssh patientping
```

Install Agent:

```bash
sudo dnf upgrade

sudo dnf install amazon-cloudwatch-agent
```

---

## Step 6

Create configuration file.

```bash
sudo nano /opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.d/patientping-monitoring.json
```

Paste the provided JSON.

Save.

---

## Step 7

Start Agent.

```bash
sudo systemctl start amazon-cloudwatch-agent

sudo systemctl enable amazon-cloudwatch-agent
```

---

## Step 8

Run application.

```bash
cd ~/patientping-web

PYTHONUNBUFFERED=1 uv run patientping.py 2>&1 | sudo tee -a /var/log/patientping.log
```

---

## Step 9

Verify Logs.

Navigate:

```text
CloudWatch
 ↓
Logs
 ↓
Log Groups
 ↓
patientping-monitoring
```

Open log stream.

You should see:

```text
PatientPing listening...
```

---

# 3. CloudWatch Alarms

## Why Alarms?

Monitoring is useful only if someone notices problems.

Instead of checking dashboards manually:

```text
Server Problem
      ↓
CloudWatch Detects
      ↓
Alarm
      ↓
Email Notification
```

---

## Common Alarm Examples

```text
CPU > 80%

Disk > 90%

Memory > 95%

HTTP 500 Errors

Application Down
```

---

## SNS (Simple Notification Service)

CloudWatch uses SNS to send notifications.

Example:

```text
CloudWatch Alarm
       ↓
SNS Topic
       ↓
Email
SMS
Lambda
```

---

# Creating Alarm

Navigate:

```text
CloudWatch
 ↓
Alarms
 ↓
Create Alarm
```

---

Select Metric:

```text
EC2
 ↓
Per-Instance Metrics
 ↓
CPUUtilization
```

---

Configure:

```text
Statistic:
Average

Period:
1 Minute

Threshold:
Greater Than

20%
```

---

Notification:

```text
Create SNS Topic

patientping-alerts

Enter Email
```

---

Alarm Name:

```text
patientping-cpu-alarm
```

Description:

```text
Alert when CPU exceeds 20%
```

Create Alarm.

---

## Confirm Subscription

AWS sends confirmation email.

Click:

```text
Confirm Subscription
```

Without confirmation, emails won't arrive.

---

## Test Alarm

SSH:

```bash
ssh patientping
```

Run:

```bash
yes > /dev/null
```

Wait:

```text
Few Minutes
```

Expected:

```text
Alarm Email Received
```

Stop:

```text
Ctrl + C
```

---

# 4. CloudTrail

## What is CloudTrail?

CloudTrail records **every AWS API call**.

Unlike CloudWatch, CloudTrail monitors **AWS account activity**, not server metrics.

---

## Why CloudTrail?

Questions it answers:

```text
Who deleted an EC2?

Who modified Security Groups?

Who created an IAM User?

Who changed the VPC?
```

---

## Information Recorded

Every event contains:

```text
User

Action

Time

Resource

Success/Failure
```

Example:

```text
User:
amith

Action:
RunInstances

Time:
10:32 AM

Resource:
EC2
```

---

## Architecture

```text
AWS User
      ↓
AWS API Call
      ↓
CloudTrail
      ↓
Audit Logs
```

---

## Why Is It Important?

Useful for:

```text
Security Auditing

Compliance

Troubleshooting

Incident Investigation
```

---

## CloudTrail vs CloudWatch

| CloudWatch             | CloudTrail                    |
| ---------------------- | ----------------------------- |
| Monitors system health | Monitors AWS account activity |
| CPU, Memory, Logs      | API Calls                     |
| Server Monitoring      | Audit Logging                 |
| Application Focus      | Security Focus                |

---

# Complete Monitoring Architecture

```text
                    EC2
                     │
        ┌────────────┼─────────────┐
        │            │             │
        ▼            ▼             ▼
CloudWatch     CloudWatch Agent   CloudTrail
 Metrics          Logs & RAM      AWS API Logs
        │            │             │
        └──────┬─────┘             │
               ▼                   ▼
        CloudWatch Dashboard   Audit History
               │
               ▼
        CloudWatch Alarm
               │
               ▼
         SNS Notification
               │
               ▼
             Email
```

---

# Key Takeaways

### CloudWatch Metrics

```text
External Monitoring

No Installation Required
```

---

### CloudWatch Agent

```text
Internal Monitoring

Collects RAM, Swap, Disk & Logs
```

---

### CloudWatch Alarm

```text
Automatically Notifies
When Conditions Are Met
```

---

### SNS

```text
Notification Service

Email
SMS
Lambda
```

---

### CloudTrail

```text
Records Every AWS API Action

Used For Auditing & Security
```

---

# Production Flow

```text
User Requests
       ↓
Application Running
       ↓
CloudWatch Metrics
       ↓
CloudWatch Agent
       ↓
CloudWatch Dashboard
       ↓
CloudWatch Alarm
       ↓
SNS Email Alert

Meanwhile...

AWS API Calls
       ↓
CloudTrail
       ↓
Audit Logs
```

**Rule of Thumb**

```text
CloudWatch Metrics → Server Health

CloudWatch Agent → OS Metrics & Logs

CloudWatch Alarm → Automatic Alerts

CloudTrail → AWS Account Audit Logs
```
