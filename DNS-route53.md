# AWS Route 53 Private Hosted Zone & A Records Notes

## What is Route 53?

**Amazon Route 53** is AWS's DNS (Domain Name System) service.

It converts:

```text
Domain Name
      ↓
IP Address
```

Example:

```text
www.patientping.internal
        ↓
10.0.10.50
```

Instead of remembering IP addresses, applications use domain names.

---

# What is a Hosted Zone?

A **Hosted Zone** is a container that stores DNS records for a domain.

Example:

```text
patientping.internal
        ↓
DNS Records
```

Records tell Route 53 where a domain should point.

---

# Public vs Private Hosted Zone

## Public Hosted Zone

Accessible from anywhere on the Internet.

Examples:

```text
google.com
amazon.com
bookmyvenue.com
```

---

## Private Hosted Zone

Accessible **only inside a VPC**.

Example:

```text
patientping.internal
```

Only AWS resources inside the associated VPC can resolve these names.

Best for:

* EC2
* RDS
* Internal APIs
* Microservices

---

# Why Use a Private Hosted Zone?

Instead of:

```text
10.0.0.120
```

Applications can use:

```text
database.patientping.internal
```

Benefits:

* Easier to remember
* Easier to manage
* Internal only
* More secure

---

# What is an A Record?

An **A (Address) Record** maps a domain name to an **IPv4 address**.

Example:

```text
www.patientping.internal
           ↓
10.0.10.50
```

When someone requests:

```text
www.patientping.internal
```

DNS returns:

```text
10.0.10.50
```

---

# Components of an A Record

### Record Name

The hostname.

Example:

```text
www
api
db
@
```

`@` represents the root domain.

Example:

```text
@
```

means:

```text
patientping.internal
```

---

### Value

The destination IPv4 address.

Example:

```text
10.0.10.50
```

---

### TTL (Time To Live)

How long DNS resolvers cache the record.

Example:

```text
TTL = 15 seconds
```

After 15 seconds, DNS checks Route 53 again for updates.

---

# Public IP vs Private IP

## Public Hosted Zone

Must point to:

```text
Public IP
```

Example:

```text
54.201.xx.xx
```

Accessible from the Internet.

---

## Private Hosted Zone

Can point to:

```text
Private IP
```

Example:

```text
10.0.10.50
```

Accessible only within the VPC.

---

# Multiple A Records

One domain can have multiple A records.

Example:

```text
api.example.com
        ↓
10.0.1.10

api.example.com
        ↓
10.0.2.10
```

Used for:

* Load Balancing
* High Availability
* Multi-Region Deployments

---

# Architecture

```text
EC2
   ↓
Requests DNS
   ↓
Route 53 Private Hosted Zone
   ↓
A Record
   ↓
Private IP
```

---

# Part 1: Create Private Hosted Zone

Navigate:

```text
AWS Console
 ↓
Route 53
 ↓
Hosted Zones
```

Click:

```text
Create Hosted Zone
```

Configure:

```text
Domain Name:
patientping.internal

Type:
Private Hosted Zone

Associated VPC:
patientping
```

Click:

```text
Create Hosted Zone
```

AWS automatically creates:

```text
NS Record
SOA Record
```

---

# Part 2: Enable DNS Hostnames

Before creating records, enable DNS hostnames.

Navigate:

```text
AWS Console
 ↓
VPC
 ↓
Your VPCs
```

Select:

```text
patientping
```

Open:

```text
Actions
 ↓
Edit VPC Settings
```

Enable:

```text
✔ Enable DNS Hostnames
```

Save.

> **Note:** It may take a few minutes for this setting to become active.

---

# Part 3: Create an A Record

Navigate:

```text
AWS Console
 ↓
Route 53
 ↓
Hosted Zones
 ↓
patientping.internal
```

Click:

```text
Create Record
```

Configure:

```text
Record Type:
A

Record Name:
www

Value:
10.0.10.50

TTL:
15 Seconds
```

Click:

```text
Create Records
```

---

# Verify

Inside the hosted zone, you should now see:

```text
www.patientping.internal
        ↓
10.0.10.50
```

---

# Real-World Example

Instead of applications using:

```text
10.0.1.25
```

they connect to:

```text
db.patientping.internal
```

If the database IP changes:

```text
10.0.1.25
      ↓
10.0.2.18
```

Only the DNS record needs updating—the application continues using:

```text
db.patientping.internal
```

No code changes required.

---

# Key Takeaways

### Route 53

```text
AWS DNS Service
```

### Hosted Zone

```text
Container for DNS Records
```

### Private Hosted Zone

```text
Private DNS inside a VPC
```

### A Record

```text
Maps a Domain Name
to an IPv4 Address
```

### Record Name

```text
Hostname
```

### Value

```text
Destination IP Address
```

### TTL

```text
How long DNS is cached
```

### Best Practice

```text
Public Hosted Zone
→ Public IP

Private Hosted Zone
→ Private IP
```

---

# Big Picture

```text
Application
      ↓
www.patientping.internal
      ↓
Route 53 Private Hosted Zone
      ↓
A Record
      ↓
10.0.10.50
      ↓
EC2 / Internal Service
```
<img width="707" height="440" alt="image" src="https://github.com/user-attachments/assets/531ad6e7-6388-4777-9e1a-58ef2896aad0" />
# AWS Route 53 DNS TTL, Caching & DNS Verification Notes

## What is TTL?

**TTL (Time To Live)** tells DNS resolvers **how long they can cache a DNS record before checking Route 53 again**.

Example:

```text
TTL = 60 Seconds
```

The DNS resolver keeps the record for 60 seconds before requesting an updated value.

---

# Why Does DNS Use Caching?

Without caching:

```text
Every User Request
        ↓
Route 53
```

This would generate millions of DNS requests.

With caching:

```text
First Request
      ↓
Route 53
      ↓
DNS Cache

Next Requests
      ↓
Use Cached Record
```

Benefits:

* Faster website loading
* Fewer DNS queries
* Lower AWS cost

---

# How DNS Caching Works

Suppose:

```text
www.patientping.internal
        ↓
10.0.10.50

TTL = 60 Seconds
```

### First Request

```text
Client
   ↓
Route 53
   ↓
10.0.10.50
```

The resolver stores the result for 60 seconds.

---

### Second Request (Within TTL)

```text
Client
   ↓
DNS Cache
   ↓
10.0.10.50
```

Route 53 is **not contacted**.

---

### After TTL Expires

```text
Client
      ↓
Route 53
      ↓
Checks for latest IP
```

The cache refreshes with the newest record.

---

# Updating a DNS Record

Suppose the server changes:

```text
Old

www.patientping.internal
        ↓
10.0.10.50
```

Updated to:

```text
www.patientping.internal
        ↓
10.0.10.99
```

Immediately after updating:

```text
Route 53
      ↓
10.0.10.99
```

However, clients that already cached:

```text
10.0.10.50
```

continue using it until the TTL expires.

---

# Eventual Consistency

DNS changes are **eventually consistent**.

This means:

```text
Update Record
      ↓
Route 53 Updates Immediately
      ↓
Clients Continue Using Cache
      ↓
TTL Expires
      ↓
Clients Receive New Record
```

Eventually, everyone sees the new IP.

---

# TTL Trade-Off

## Low TTL

Example:

```text
TTL = 15 Seconds
```

Advantages:

* DNS updates propagate quickly
* Better for failover and testing

Disadvantages:

* More DNS queries
* Slightly higher latency
* Higher Route 53 cost

---

## High TTL

Example:

```text
TTL = 3600 Seconds (1 Hour)
```

Advantages:

* Fewer DNS queries
* Faster responses from cache
* Lower cost

Disadvantages:

* DNS changes take longer to reach users

---

# Choosing a TTL

### Development / Testing

```text
15–60 Seconds
```

Allows quick testing of DNS changes.

---

### Production

```text
300–3600 Seconds
```

Reduces DNS traffic and improves performance.

---

# Example Timeline

Suppose:

```text
TTL = 60 Seconds
```

Timeline:

```text
10:00
Client asks Route 53
Gets:
10.0.10.50

10:00–10:59
Uses Cached Value

10:01
Administrator changes DNS
to:
10.0.10.99

10:01–10:59
Client still uses:
10.0.10.50

11:00
TTL Expires

Client asks Route 53 again

Receives:
10.0.10.99
```

---

# Why "It's Always DNS"

A common production issue:

```text
Wrong IP Added
      ↓
Clients Cache Wrong IP
      ↓
DNS Fixed
      ↓
Clients Still Use Cached Record
      ↓
Must Wait For TTL To Expire
```

Even after fixing the DNS record, users may continue reaching the wrong server until cached entries expire.

---

# Verifying DNS

A **Private Hosted Zone** is only accessible from resources **inside the associated VPC**.

That means:

From your laptop:

```bash
dig www.patientping.internal +short
```

Result:

```text
No meaningful result
```

because your laptop is **outside the VPC**.

---

From an EC2 instance inside the VPC:

```text
EC2
   ↓
Route 53 Private Hosted Zone
   ↓
www.patientping.internal
   ↓
10.0.10.50
```

The domain resolves successfully.

---

# Why?

Private Hosted Zones are associated with a VPC.

Only resources inside that VPC can query those DNS records.

```text
Internet
    ↓
Cannot Resolve
patientping.internal

Inside VPC
     ↓
Can Resolve
patientping.internal
```

---

# Verifying DNS from EC2

## Step 1

SSH into EC2:

```bash
ssh patientping
```

---

## Step 2

Install the DNS utility (if not already installed):

```bash
sudo dnf install bind-utils
```

This installs the `dig` command.

---

## Step 3

Run a DNS lookup:

```bash
dig www.patientping.internal +short
```

Expected output:

```text
10.0.10.50
```

This confirms:

* Route 53 is working
* Private Hosted Zone is associated with the VPC
* DNS resolution inside the VPC works correctly

---

## Step 4

Exit the server:

```bash
exit
```

---

# What is dig?

`dig` (**Domain Information Groper**) is a command-line tool used to query DNS servers.

It helps verify:

* Domain resolution
* DNS records
* Troubleshoot DNS issues

Example:

```bash
dig google.com
```

or

```bash
dig www.patientping.internal +short
```

The `+short` option returns only the resolved IP address.

---

# Additional Notes

* DNS caching can occur at multiple layers: browser cache, OS cache, ISP resolver, and Route 53 resolver.
* Clearing local DNS cache (e.g., restarting network service or flushing DNS) can help test updates faster.
* Route 53 supports both **Public Hosted Zones** (internet-facing) and **Private Hosted Zones** (VPC-only).
* DNS propagation delays are usually due to caching, not Route 53 delays.

---

# Key Takeaways

### TTL

```text
How long a DNS record is cached.
```

### DNS Cache

```text
Stores DNS responses temporarily to reduce queries.
```

### Eventual Consistency

```text
DNS changes become visible gradually as caches expire.
```

### Private Hosted Zone

```text
Only accessible inside the associated VPC.
```

### dig

```text
Command-line tool used to query DNS records.
```

### +short

```text
Displays only the resolved IP address.
```

---

# Big Picture

```text
Outside AWS
(Laptop)
      ↓
dig www.patientping.internal
      ↓
No Result

──────────────────────────────────

Inside AWS VPC

EC2
 ↓
dig www.patientping.internal
 ↓
Route 53 Private Hosted Zone
 ↓
A Record
 ↓
10.0.10.50
```

**Rule of Thumb**

```text
Low TTL  → Faster DNS updates, more queries

High TTL → Slower DNS updates, fewer queries

Public Hosted Zone → Accessible from the Internet

Private Hosted Zone → Accessible only within the VPC
```

