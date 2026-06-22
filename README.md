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

