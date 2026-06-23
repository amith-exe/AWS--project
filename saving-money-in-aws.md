# AWS Compute Scaling & Cost Optimization Notes

---

# 1. Auto Scaling Groups (ASG)

## Problem

Managing hundreds of servers manually is difficult.

## What is ASG?

An **Auto Scaling Group (ASG)** automatically launches, removes, and replaces EC2 instances.

You define:

* Desired instances
* Minimum instances
* Maximum instances
* Scaling rules

AWS handles the rest.

---

## How It Works

```text
Traffic Increases
       ↓
Launch More Servers
       ↓
Traffic Decreases
       ↓
Terminate Extra Servers
```

Example:

```text
CPU > 70%
      ↓
Add 2 servers

CPU < 20%
      ↓
Remove servers
```

This is called **Horizontal Scaling**.

---

## Self-Healing

```text
Desired = 1
Min = 1
Max = 1
```

If the server crashes:

```text
Server Fails
      ↓
ASG Detects Failure
      ↓
Terminate Instance
      ↓
Launch New Instance
```

ASG provides **high availability**.

---

## Real Example

```text
Normal Traffic → 2 servers
High Traffic   → 10 servers
Low Traffic    → 2 servers
```

---

# 2. Reserved Instances (RI)

## What is RI?

A pricing model where you commit to using compute resources for:

```text
1 year or 3 years
```

In return, AWS gives discounts.

---

## Key Idea

```text
Long-term commitment
        ↓
Lower Cost
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
* Backend APIs
* Always-running applications

---

## Advantages

* Significant savings
* Predictable billing
* Guaranteed capacity

---

## Limitations

* Less flexible
* Long-term commitment
* Hard to change configurations

---

# 3. Savings Plans

Savings Plans provide discounts with more flexibility than Reserved Instances.

---

## How It Works

You commit to spending:

```text
$X per hour
for 1-3 years
```

AWS automatically applies discounts.

---

## Key Difference

Reserved Instances:

```text
Commit to specific instance types
```

Savings Plans:

```text
Commit to spending amount
```

---

## Flexibility

You can often change:

* Instance sizes
* Instance families
* Availability Zones
* Regions

while still getting discounts.

---

## Discounts

Typical savings:

```text
26% - 72%
```

---

## Best For

Dynamic environments:

* Auto Scaling applications
* Growing systems
* Changing infrastructure

---

# 4. Spot Instances

## What are Spot Instances?

Spot Instances let you use AWS's **unused EC2 capacity** at massive discounts.

Typical savings:

```text
Up to ~90% cheaper
than On-Demand pricing
```

---

## The Catch

AWS can reclaim the server at any time.

You get approximately:

```text
2-minute warning
```

before the instance is terminated.

---

## Key Idea

```text
Very Cheap
     +
Can disappear anytime
```

---

## Best For

Fault-tolerant and stateless workloads:

* Batch processing
* Background jobs
* Data analytics
* AI/ML training
* Video rendering
* Offline computations
* CI/CD build servers

---

## Not Suitable For

Always-on systems:

* Databases
* Production APIs
* Authentication servers
* Critical backend services

---

## Example

Suppose an image processing job takes 10 hours.

```text
Spot Instance
      ↓
Server terminates after 4 hours
      ↓
Launch another Spot Instance
      ↓
Continue processing
```

Because the workload is stateless, losing one server is acceptable.

---

# Reserved Instances vs Savings Plans vs Spot Instances

| Feature               | Reserved Instances | Savings Plans     | Spot Instances          |
| --------------------- | ------------------ | ----------------- | ----------------------- |
| Commitment            | 1-3 years          | 1-3 years         | None                    |
| Discounts             | High               | High              | Very High (~90%)        |
| Flexibility           | Lower              | Higher            | Very High               |
| Reliability           | High               | High              | Low                     |
| Can AWS terminate it? | No                 | No                | Yes                     |
| Best For              | Stable workloads   | Dynamic workloads | Interruptible workloads |

---

# Real Production Strategy

```text
Databases
      ↓
Reserved Instances

Application Servers (ASG)
      ↓
Savings Plans

Background Jobs / AI Training
      ↓
Spot Instances

Temporary Testing Systems
      ↓
On-Demand Pricing
```

---

# Big Picture

```text
Auto Scaling Groups
      ↓
Scale Up During High Traffic
      ↓
Scale Down During Low Traffic
      ↓
Replace Failed Instances
      ↓
Optimize Costs Using:

Reserved Instances
Savings Plans
Spot Instances
```

---

# One-Line Definitions

### Auto Scaling Group (ASG)

Automatically adjusts the number of servers based on demand and replaces failed instances.

### Reserved Instance (RI)

Commit to long-term compute usage for significant discounts.

### Savings Plan

Commit to a spending amount and receive flexible discounts across compute resources.

### Spot Instance

Use AWS's unused compute capacity at massive discounts, but AWS can terminate the instance at any time with about a 2-minute warning.
