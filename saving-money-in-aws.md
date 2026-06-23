# AWS Reserved Instances (RI) and Savings Plans

## Complete Study Notes

---

# Table of Contents

1. Why AWS Cost Optimization Matters
2. What are Reserved Instances (RI)?
3. How Reserved Instances Work
4. Types of Reserved Instances
5. Advantages and Limitations of RI
6. What are Savings Plans?
7. How Savings Plans Work
8. Types of Savings Plans
9. Reserved Instances vs Savings Plans
10. Real-World Examples
11. Cost Comparisons
12. Production Best Practices
13. Which One Should You Use?

---

# Why AWS Cost Optimization Matters

Cloud computing uses a **pay-as-you-go** pricing model.

For EC2:

You pay for:

* CPU
* Memory
* Storage
* Networking
* Running time

---

# Example

Suppose:

```text id="wkkxkq"
t3.micro
≈ $7.60/month
```

One server:

```text id="yts4iy"
$7.60/month
```

100 servers:

```text id="6nk6t3"
$760/month
```

1000 servers:

```text id="c5hr6h"
$7,600/month
```

Large companies often spend:

```text id="yyf7hf"
Thousands
Millions
of dollars every month
```

Even a small discount can save huge amounts of money.

---

# On-Demand Pricing

This is the default pricing model.

Think:

```text id="yd8skm"
Pay only when using
```

Example:

```text id="3f6bfx"
Run server
↓
Pay per hour
```

Stop server:

```text id="wp0ixk"
No compute charges
```

---

# Advantages

* Maximum flexibility
* No commitment
* Easy to scale

---

# Disadvantages

Most expensive option.

---

# Example

```text id="2fx0vx"
t3.micro
$7.60/month
```

If you know:

```text id="cb4gu7"
Server will run for years
```

paying full price is often wasteful.

---

# What are Reserved Instances (RI)?

Reserved Instances are AWS discounts for committing to use compute resources for:

```text id="u53i5t"
1 year
or
3 years
```

Think:

```text id="jlwm50"
Buying in bulk
```

---

# Real-Life Analogy

Imagine Netflix.

Monthly:

```text id="jlwm51"
₹500/month
```

Yearly plan:

```text id="jlwm52"
₹5000/year
```

You commit longer.

Company gives discount.

Reserved Instances work similarly.

---

# How Reserved Instances Work

AWS says:

```text id="jlwm53"
Promise you'll use this server
for 1-3 years.
```

You say:

```text id="jlwm54"
Okay.
```

AWS replies:

```text id="jlwm55"
Here's a discount.
```

---

# Example

On-Demand:

```text id="jlwm56"
t3.micro
$7.60/month
```

Reserved:

```text id="jlwm57"
≈ $5/month
```

Savings every month.

---

# What Are You Actually Reserving?

You are reserving:

```text id="jlwm58"
Compute capacity
```

You're NOT reserving:

* Particular data
* Particular server machine
* Particular EBS volume

You are committing:

```text id="jlwm59"
I will use this much compute
for a long time.
```

---

# Characteristics of Reserved Instances

Traditionally tied to:

* Region
* Instance Type
* Sometimes Availability Zone

---

# Example

Reservation:

```text id="jlwm60"
Region:
Mumbai

Instance:
t3.micro

Term:
1 year
```

You receive discounts only for:

```text id="jlwm61"
Mumbai
t3.micro
```

---

# Problem with Reserved Instances

Suppose after 6 months:

You need:

```text id="jlwm62"
t3.large
```

instead of:

```text id="jlwm63"
t3.micro
```

Your reservation may no longer perfectly match.

Reserved Instances can become restrictive.

---

# Types of Reserved Instances

## Standard Reserved Instances

Highest discounts.

Lower flexibility.

Good for:

```text id="jlwm64"
Predictable workloads
```

Examples:

* Databases
* Internal applications
* Long-running APIs

---

## Convertible Reserved Instances

Lower discounts.

More flexibility.

Can change:

* Instance family
* Operating system
* Tenancy

Good for:

```text id="jlwm65"
Growing applications
```

---

# Advantages of Reserved Instances

## Large Discounts

Can save:

```text id="jlwm66"
Up to ~72%
```

---

## Capacity Reservation

AWS can reserve capacity.

Useful during:

* Black Friday
* Product launches
* Large events

---

## Predictable Costs

Easy budgeting.

---

# Limitations of Reserved Instances

## Less Flexible

Bound by:

* Region
* Instance types
* Terms

---

## Long Commitment

Usually:

```text id="jlwm67"
1 year
or
3 years
```

---

## Harder to Adapt

Infrastructure changes frequently.

Reserved Instances can become inconvenient.

---

# What are Savings Plans?

AWS introduced Savings Plans to solve RI limitations.

Think:

```text id="jlwm68"
Reserved Instances
but more flexible
```

---

# How Savings Plans Work

AWS asks:

```text id="jlwm69"
Can you commit to spending
a certain amount per hour
for 1-3 years?
```

Example:

```text id="jlwm70"
$1/hour
for
3 years
```

In exchange:

AWS gives discounts.

---

# Important Difference

Reserved Instances:

```text id="jlwm71"
Commit to instance type.
```

Savings Plans:

```text id="jlwm72"
Commit to spending level.
```

Much more flexible.

---

# Example

Commit:

```text id="jlwm73"
$1/hour
```

You can use:

```text id="jlwm74"
t3.micro
t3.small
t3.medium
t3.large
```

AWS automatically applies discounts.

---

# Types of Savings Plans

## Compute Savings Plans

Most flexible.

Can change:

* Regions
* Availability Zones
* Instance Families
* Operating Systems

---

# Example

Today:

```text id="jlwm75"
Mumbai
t3.micro
```

Tomorrow:

```text id="jlwm76"
Frankfurt
m5.large
```

Discount still applies.

---

## EC2 Instance Savings Plans

More restrictive.

Can change:

* Availability Zone
* Instance sizes

Must stay:

* Same instance family
* Same region

---

# Example

Allowed:

```text id="jlwm77"
t3.micro
↓
t3.small
↓
t3.medium
```

Not allowed:

```text id="jlwm78"
t3.micro
↓
m5.large
```

---

# Real Example

Suppose:

BookMyVenue backend.

Today:

```text id="jlwm79"
t3.micro
```

Six months later:

```text id="jlwm80"
Need more RAM
```

Upgrade:

```text id="jlwm81"
t3.medium
```

Savings Plan still applies.

Reserved Instance may not.

---

# Discounts

Savings Plans typically save:

```text id="jlwm82"
26% to 72%
```

depending on:

* Service
* Term
* Payment option

---

# Payment Options

## No Upfront

Pay gradually.

Lower discount.

---

## Partial Upfront

Pay some now.

Better discount.

---

## All Upfront

Pay entire amount.

Highest discount.

---

# Real-World Example

Company:

50 EC2 instances.

Cost:

```text id="jlwm83"
$10,000/month
```

Savings Plan:

40% discount.

New cost:

```text id="jlwm84"
$6,000/month
```

Savings:

```text id="jlwm85"
$4,000/month
$48,000/year
```

Simply by changing pricing.

No architecture changes.

---

# Why Turning Off Servers Isn't Always Best

Many people think:

```text id="jlwm86"
Turn off servers at night.
```

Example:

Save:

```text id="jlwm87"
$500/month
```

But:

Savings Plan:

```text id="jlwm88"
Save:
$1500/month
```

while servers remain:

```text id="jlwm89"
Running 24/7
```

Sometimes discounts beat shutdown strategies.

---

# Reserved Instances vs Savings Plans

| Feature                | Reserved Instance | Savings Plan         |
| ---------------------- | ----------------- | -------------------- |
| Commitment             | Specific compute  | Spending amount      |
| Flexibility            | Lower             | Higher               |
| Change AZ              | Limited           | Yes                  |
| Change Instance Size   | Limited           | Yes                  |
| Change Instance Family | Difficult         | Compute Plan allows  |
| Discounts              | High              | High                 |
| Best For               | Stable systems    | Dynamic environments |

---

# Real Production Examples

## Good Reserved Instance Candidate

Database server:

```text id="jlwm90"
PostgreSQL
Runs for years
Rarely changes
```

Reserved Instance works well.

---

## Good Savings Plan Candidate

Backend API:

```text id="jlwm91"
NestJS
Traffic grows
Servers resize frequently
Autoscaling
```

Savings Plan works better.

---

# What Large Companies Usually Do

Combination strategy:

```text id="jlwm92"
Databases
↓
Reserved Instances

Application Servers
↓
Savings Plans

Experimental Systems
↓
On-Demand
```

---

# For Your Projects

Currently:

```text id="jlwm93"
Learning AWS
One EC2 instance
Few hours per week
```

Use:

```text id="jlwm94"
On-Demand
```

Do NOT buy:

* Reserved Instances
* Savings Plans

---

# Future Scenario

Suppose:

BookMyVenue grows.

You run:

```text id="jlwm95"
API Servers
Background Workers
Databases
Monitoring Systems
```

Always running.

Then:

```text id="jlwm96"
Savings Plans
```

can reduce costs significantly.

---

# Big Picture

```text id="jlwm97"
On-Demand
↓
Maximum flexibility
Highest cost

Reserved Instances
↓
Less flexibility
Large discounts

Savings Plans
↓
High flexibility
Large discounts
```

---

# One-Line Definitions

### Reserved Instance (RI)

A pricing model where you commit to using specific EC2 compute resources for 1-3 years in exchange for significant discounts.

### Savings Plan

A pricing model where you commit to spending a certain amount on AWS compute for 1-3 years and receive significant discounts with much greater flexibility than Reserved Instances.
