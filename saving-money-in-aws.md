# AWS Compute Scaling & Cost Optimization Notes

## Auto Scaling Groups (ASG), Reserved Instances & Savings Plans

---

# 1. Auto Scaling Groups (ASG)

## The Problem

One server is easy.

But what if you need:

```text
100 servers
500 servers
1000 servers
```

Manually creating and managing them becomes difficult.

---

## What is an Auto Scaling Group?

An **Auto Scaling Group (ASG)** automatically launches and manages EC2 instances for you.

You define:

1. Desired number of instances
2. Minimum and maximum limits
3. Scaling rules

AWS handles everything else.

---

## How It Works

```text
Auto Scaling Group
      ↓
Launch EC2 Instances
      ↓
Monitor Load
      ↓
Scale Up or Down Automatically
```

---

## Automatic Scaling

Example:

```text
CPU Usage > 70%
       ↓
Launch 2 more servers
```

Later:

```text
CPU Usage < 20%
       ↓
Terminate extra servers
```

This is called:

```text
Horizontal Scaling
```

---

## Self-Healing Servers

ASG can automatically replace failed instances.

Example:

```text
Desired Capacity = 1
Minimum = 1
Maximum = 1
```

If the server crashes:

```text
Server Crashes
      ↓
ASG Detects Failure
      ↓
Terminate Bad Instance
      ↓
Launch New Instance
```

This ensures high availability.

---

## Real Production Example

Application traffic:

```text
Normal → 2 servers
High traffic → 10 servers
Low traffic → 2 servers
```

Everything scales automatically based on demand.

---

# 2. Reserved Instances (RI)

## Why Cost Optimization Matters

Cloud servers cost money.

Example:

```text
t3.micro
≈ $7.60/month
```

Scaling increases cost:

```text
100 servers → $760/month
1000 servers → $7,600/month
```

Even small discounts can save a lot.

---

## What is a Reserved Instance?

A Reserved Instance is a pricing model where you commit to using compute resources for:

```text
1 Year
or
3 Years
```

In return, AWS gives a discount.

---

## Key Idea

```text
Long-term commitment = Lower cost
```

---

## Characteristics

* Tied to region and instance type
* Predictable pricing
* Capacity reservation

---

## Best For

Stable workloads:

* Databases
* Backend services
* Always-running applications

---

## Advantages

* Significant cost savings
* Predictable billing
* Guaranteed capacity

---

## Limitations

* Less flexible
* Long-term commitment
* Hard to change configuration

---

# 3. Savings Plans

Savings Plans provide discounts with more flexibility than Reserved Instances.

---

## How Savings Plans Work

You commit to spending:

```text
$X per hour
for 1–3 years
```

AWS automatically applies discounts to your usage.

---

## Key Difference

Reserved Instances:

```text
Commit to specific instance types
```

Savings Plans:

```text
Commit to a spending amount
```

---

## Flexibility

You can change:

* Instance sizes
* Instance families
* Regions (in many cases)

while still getting discounts.

---

## Discounts

Typical savings:

```text
26% – 72%
```

---

## Best For

Dynamic environments:

* Auto scaling systems
* Growing applications
* Changing infrastructure

---

# Reserved Instances vs Savings Plans

| Feature                | Reserved Instances | Savings Plans     |
| ---------------------- | ------------------ | ----------------- |
| Commitment             | Specific compute   | Spending amount   |
| Flexibility            | Lower              | Higher            |
| Change Instance Size   | Limited            | Yes               |
| Change Instance Family | Limited            | Yes               |
| Discounts              | High               | High              |
| Best For               | Stable workloads   | Dynamic workloads |

---

# Real Production Strategy

Many companies combine pricing models:

```text
Databases
      ↓
Reserved Instances

Application Servers (Auto Scaling)
      ↓
Savings Plans

Temporary / Testing Systems
      ↓
On-Demand Pricing
```

---

# Big Picture

```text
Use Auto Scaling Groups
      ↓
Scale Up During High Traffic
      ↓
Scale Down During Low Traffic
      ↓
Automatically Replace Failed Servers
      ↓
Reduce Costs Using
Reserved Instances or Savings Plans
```

---

# One-Line Definitions

### Auto Scaling Group (ASG)

Automatically adjusts the number of servers based on demand and replaces failed instances.

### Reserved Instance (RI)

A pricing model where you commit to long-term usage for lower cost.

### Savings Plan

A flexible pricing model where you commit to spending and receive discounts across different compute resources.
