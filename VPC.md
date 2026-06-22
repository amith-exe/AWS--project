# AWS NOTES

### REGIONS IN AWS
<img width="746" height="638" alt="image" src="https://github.com/user-attachments/assets/4f131336-69c5-4045-9900-af2a3053503d" />
## VPC 
With Amazon Virtual Private Cloud (Amazon VPC), you can launch AWS resources in a logically isolated virtual network that you've defined. This virtual network closely resembles a traditional network that you'd operate in your own data center, with the benefits of using the scalable infrastructure of AWS.

The following diagram shows an example VPC. The VPC has one subnet in each of the Availability Zones in the Region, EC2 instances in each subnet, and an internet gateway to allow communication between the resources in your VPC and the internet.
<img width="562" height="342" alt="image" src="https://github.com/user-attachments/assets/ac4acd1f-90d0-40fe-8748-ffa9988ef887" />
This box (the VPC) is really just a group of IP addresses within a region we choose.

Sometimes we want multiple VPCs, for example to separate production resources from development servers. Or we might want some servers in a primary location and others in a backup region.
<img width="742" height="581" alt="image" src="https://github.com/user-attachments/assets/8d8abd3b-5165-4970-82e9-a8552a18f2cc" />
## Creating a VPC
<img width="737" height="617" alt="image" src="https://github.com/user-attachments/assets/8c6c7861-2552-4b12-a226-9cf506ce4ef1" />
# Classless Inter-Domain Routing (CIDR)

## What is CIDR?

CIDR (Classless Inter-Domain Routing) is an IP addressing method that allocates IP addresses efficiently and improves internet routing. It uses variable-length subnet masks (VLSM) to create networks of different sizes.

---

## IP Address Structure

An IP address has two parts:

* **Network Address** – Identifies the network.
* **Host Address** – Identifies a device within that network.

---

## Classful Addressing (Before CIDR)

IPv4 addresses are 32 bits long and were divided into fixed classes:

| Class | Network Bits | Example       | Network Part | Host Part |
| ----- | ------------ | ------------- | ------------ | --------- |
| A     | 8 bits       | 44.0.0.1      | 44           | 0.0.0.1   |
| B     | 16 bits      | 128.16.0.2    | 128.16       | 0.2       |
| C     | 24 bits      | 192.168.1.100 | 192.168.1    | 100       |

### Limitations of Classful Addressing

* Fixed network sizes caused **IP address wastage**.
* Example: A company with 300 devices couldn't use Class C (254 hosts), so it had to use Class B (65,534 hosts), wasting thousands of addresses.
* Networks couldn't be easily combined because subnet masks were fixed.

---

## CIDR (Classless Addressing)

CIDR uses **Variable Length Subnet Masking (VLSM)**, allowing flexible allocation of network and host bits.

**Notation:** `IP_Address/Prefix_Length`

Example:

* `192.0.2.0/24`

  * Network bits: 24
  * Network address: `192.0.2`
  * Host bits: 8

---

## Benefits of CIDR

* Reduces IP address wastage.
* Supports networks of different sizes.
* Reduces routing table entries.
* Improves routing efficiency and packet delivery.
* Enables subnetting and supernetting.
* Used extensively in cloud networks and VPCs.

---

## CIDR Blocks

A CIDR block is a group of IP addresses sharing the same network prefix.

Examples:

* `/24` → 256 addresses
* `/16` → 65,536 addresses
* `/8` → 16,777,216 addresses

Larger prefixes (e.g., `/24`) create smaller networks, while smaller prefixes (e.g., `/16`) create larger networks.

---

## Supernetting with CIDR

CIDR allows combining multiple networks into a larger block.

Example:

* `192.168.0.0/23`
* Subnet Mask: `255.255.254.0`

This combines:

* `192.168.0.0/24`
* `192.168.1.0/24`

into one larger network.

---

## CIDR in IPv6

IPv6 uses **128-bit addresses** and also supports CIDR notation.

Example:

* `2001:db8::/32`

  * First 32 bits represent the network prefix.

CIDR in IPv6 allows efficient aggregation and routing of the huge IPv6 address space.

---

## AWS and CIDR

AWS uses CIDR extensively in **Amazon VPC**:

* Create isolated virtual networks (VPCs).
* Allocate IPv4 and IPv6 CIDR blocks.
* Aggregate CIDRs across route tables, security groups, and firewalls.
* Support Bring Your Own IP (BYOIP) and IPv6 address management through **Amazon VPC IP Address Manager (IPAM)**.
<img width="1063" height="597" alt="image" src="https://github.com/user-attachments/assets/fdc6376d-338c-447e-958b-39f3c2d69560" />
<img width="747" height="627" alt="image" src="https://github.com/user-attachments/assets/15c10a1e-9efb-4e68-99a0-5a0b96166c01" />

## Subneting
<img width="733" height="618" alt="image" src="https://github.com/user-attachments/assets/af1781ac-dac7-4b39-850b-0e6400e5155c" />
# Internet Gateway (IGW)

## What is an Internet Gateway?

An **Internet Gateway (IGW)** is a **horizontally scaled, redundant, and highly available** AWS VPC component that enables communication between a **VPC and the internet**.

It supports both:

* **IPv4**
* **IPv6**

An Internet Gateway itself does **not create bandwidth limitations or availability risks**.

---

## Main Functions of an Internet Gateway

* Allows resources in a **public subnet** to access the internet.
* Allows internet users to connect to resources in a VPC using their **public IP addresses**.
* Provides a target in the **route table** for internet-bound traffic.
* Performs **one-to-one NAT** for IPv4 traffic.

---

## Public and Private Subnets

### Public Subnet

A subnet is **public** if its route table contains:

```text
0.0.0.0/0 → Internet Gateway (IPv4)
::/0       → Internet Gateway (IPv6)
```

Instances in a public subnet can access the internet if they have:

* Public IPv4 address
* Elastic IP address
* IPv6 address

---

### Private Subnet

A subnet is **private** if its route table **does not contain a route to an Internet Gateway**.

Instances in private subnets:

* Cannot access the internet directly
* Cannot receive incoming internet traffic
* Remain isolated inside the VPC

---

## IPv4 and NAT

For IPv4 internet communication:

Requirements:

1. Internet Gateway attached to the VPC
2. Route to the Internet Gateway
3. Public IPv4 address or Elastic IP

### How NAT Works

When traffic leaves the VPC:

```text
Private IP → Public IP
```

When replies return:

```text
Public IP → Private IP
```

The Internet Gateway performs this **one-to-one Network Address Translation (NAT)** automatically.

---

## IPv6 Internet Access

Requirements:

1. VPC has an IPv6 CIDR block
2. Subnet has an IPv6 CIDR block
3. Instance has an IPv6 address
4. Route table contains:

```text
::/0 → Internet Gateway
```

IPv6 addresses are **globally unique and public by default**, so no NAT is required.

---

## Default vs Nondefault VPC

| Component                 | Default VPC | Nondefault VPC |
| ------------------------- | ----------- | -------------- |
| Internet Gateway attached | Yes         | No             |
| IPv4 route (0.0.0.0/0)    | Yes         | No             |
| IPv6 route (::/0)         | No          | No             |
| Auto-assigned Public IPv4 | Yes         | No             |
| Auto-assigned IPv6        | No          | No             |

---

## Steps to Enable Internet Access

1. Create or use a VPC.
2. Attach an **Internet Gateway** to the VPC.
3. Create a **public subnet**.
4. Add a route in the route table:

```text
0.0.0.0/0 → Internet Gateway
```

(Optional for IPv6)

```text
::/0 → Internet Gateway
```

5. Launch an EC2 instance with a:

   * Public IPv4 address
   * Elastic IP
   * IPv6 address

---

## Architecture Example

```text
Internet
    │
    ▼
Internet Gateway (IGW)
    │
    ▼
Public Route Table
(0.0.0.0/0 → IGW)
    │
    ▼
Public Subnet
    │
    ▼
EC2 Instance (Public IP)
```

Private subnets do not have the route:

```text
0.0.0.0/0 → IGW
```

Therefore, instances inside them cannot communicate directly with the internet.

---

## Key Points for Exams/Interviews

* IGW connects a VPC to the internet.
* Public subnet = Route to IGW.
* Private subnet = No route to IGW.
* IPv4 requires Public IP + NAT.
* IPv6 does not require NAT.
* Internet Gateway is highly available, redundant, and horizontally scaled.
* There is no charge for the Internet Gateway itself; only data transfer charges apply.

# AWS Route Tables - Short Notes

## What is a Route Table?

A **Route Table** is a set of rules (**routes**) that determines where network traffic from your VPC is directed.

A route consists of:

* **Destination** → Where the traffic should go.
* **Target** → Through which gateway or connection the traffic should travel.

---

## Key Concepts

### 1. Main Route Table

* Automatically created with every VPC.
* Controls routing for all subnets that are **not explicitly associated** with another route table.

---

### 2. Custom Route Table

* A route table created manually by the user.
* Used to define custom routing rules for specific subnets.

---

### 3. Destination

The range of IP addresses to which traffic should be sent (**Destination CIDR**).

Example:

```text
172.16.0.0/12
```

This could represent an external corporate network.

---

### 4. Target

The gateway or connection through which traffic is sent.

Examples:

* Internet Gateway (IGW)
* NAT Gateway
* Virtual Private Gateway (VGW)
* Transit Gateway (TGW)
* Network Interface (ENI)

---

### 5. Local Route

A default route automatically created inside every VPC that enables communication between resources within the same VPC.

Examples:

```text
10.0.0.0/16 → local
2001:db8::/56 → local
```

* IPv4 VPCs have an IPv4 local route.
* Dual-stack VPCs have both IPv4 and IPv6 local routes.

---

### 6. Route Table Association

The link between a route table and:

* A subnet
* An Internet Gateway
* A Virtual Private Gateway

Associations determine which routes are applied to traffic.

---

### 7. Subnet Route Table

A route table associated with a subnet.

Example:

```text
0.0.0.0/0 → Internet Gateway
```

This makes the subnet a **public subnet**.

---

### 8. Route Propagation

When a **Virtual Private Gateway (VGW)** is attached and route propagation is enabled:

* AWS automatically adds VPN routes to the route table.
* No need to manually create or delete VPN routes.

Used mainly with:

* Site-to-Site VPN connections

---

### 9. Gateway Route Table

A route table associated with:

* Internet Gateway (IGW)
* Virtual Private Gateway (VGW)

Used for advanced traffic routing scenarios.

---

### 10. Edge Association

Associates a route table with an:

* Internet Gateway
* Virtual Private Gateway

Used to route incoming VPC traffic to a network appliance such as:

* Firewall
* Security appliance
* Packet inspection device

Example:

```text
Internet → IGW → Firewall ENI → VPC
```

---

### 11. Transit Gateway Route Table

A route table associated with a **Transit Gateway (TGW)**.

Used to route traffic between:

* Multiple VPCs
* VPNs
* On-premises networks

Acts as a central routing hub.

---

### 12. Local Gateway Route Table

A route table associated with an **AWS Outposts Local Gateway**.

Used to route traffic between:

* AWS Outposts
* On-premises networks

---

## Quick Summary

| Concept                     | Purpose                                    |
| --------------------------- | ------------------------------------------ |
| Main Route Table            | Default route table for a VPC              |
| Custom Route Table          | User-created route table                   |
| Destination                 | IP range where traffic should go           |
| Target                      | Gateway or connection used to send traffic |
| Local Route                 | Communication within the VPC               |
| Route Table Association     | Links route table to subnet/gateway        |
| Subnet Route Table          | Routes traffic for a subnet                |
| Route Propagation           | Automatically adds VPN routes              |
| Gateway Route Table         | Associated with IGW or VGW                 |
| Edge Association            | Sends traffic through appliances/firewalls |
| Transit Gateway Route Table | Routes between multiple networks           |
| Local Gateway Route Table   | Routes traffic for AWS Outposts            |
#### Example VPC with route tables

The following diagram shows a VPC with five subnets, a main route table, and three custom route tables. All four route tables have local routes. Custom route table 1 has a route to an internet gateway, and it is associated with the public subnet in Availability Zone A. Custom route table 2 has a route to a peered VPC, and it is associated with the private subnet in Availability Zone B. Custom route table 3 has a route to a virtual private gateway, and it is associated with the VPN-only subnets in both Availability Zones.
<img width="557" height="507" alt="image" src="https://github.com/user-attachments/assets/88aa1724-96fe-4558-9b3b-ca119850e6cd" />


<img width="718" height="510" alt="image" src="https://github.com/user-attachments/assets/dc8695c5-400f-4992-954f-b47860ca99c9" />




