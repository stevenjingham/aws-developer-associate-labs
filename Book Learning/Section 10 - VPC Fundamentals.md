# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 10 - VPC Fundamentals
> 
> Date: January 2026

---

## Purpose
- Provide **name resolution** by translating human-friendly domain names into IP addresses
- Enable **high availability, failover, and traffic control** at the DNS level
- Allow applications to scale globally by directing users to the **best endpoint**
- Separate **domain registration** from **DNS traffic management**

## Key Concepts
- **DNS (Domain Name System)**: Backbone of the internet that resolves hostnames to IPs
- **Hierarchy**
  - Root (`.`)
  - Top-Level Domain (TLD): `.com`, `.org`
  - Second-Level Domain (SLD): `example.com`
  - Subdomain: `www.example.com`
  - FQDN: `api.www.example.com`
- **Route 53**
  - Fully managed, highly available, scalable DNS
  - **Authoritative DNS** (you control records)
  - Also acts as a **Domain Registrar**
- **Hosted Zones**
  - Public Hosted Zone: DNS records for internet-facing domains
  - Private Hosted Zone: DNS records only resolvable inside a VPC
- **DNS Records**
  - A / AAAA – hostname → IPv4 / IPv6
  - CNAME – hostname → hostname (not root domain)
  - Alias – Route 53 only, hostname → AWS resource
  - NS – Name Servers for a hosted zone
- **TTL (Time To Live)**
  - Controls how long DNS responses are cached


## How It Works
1. User enters a URL (e.g. `www.example.com`)
2. Browser checks **local DNS cache**
3. If not cached:
  - Queries **Root DNS** → points to TLD (.com)
  - Queries **TLD DNS** → points to authoritative NS
  - Queries **Authoritative DNS (Route 53)** → returns record
4. Browser connects to returned IP or AWS resource
5. Routing policy determines **which DNS response** is returned (not traffic routing)

## Code / Config

## Common Pitfalls

## Key exam notes
- Route 53 is **authoritative DNS** and a **domain registrar**
- Alias records:
  - Route 53 only
  - Work at root domain
  - Free queries
- Health checks enable **Automated DNS Failover**
- Routing policies decide **DNS response**, not packet routing
- IP-based routing requires **known client CIDR ranges**
- Private Hosted Zones only work **inside a VPC**
- Multi-Value routing returns up to **8 healthy records**
---

## Detailed Notes

### Video notes
- Virtual Private Cloud
  - A private network to deploy resources. 
  - It is a **regional** resource
  - Inside a VPC is a **Subnet**
    - Subnets allow a **partition of a network** within the VPC
    - They are an **Availability Zone** resource
  - A public subnet is accessible from the internet
  - A private subnet is not accessible from the internet
  - A **route table** is used to define the access between private and public subnets
  - **Internet Gateway (IGW)** allows our VPC to connect to internet. A public subnet has a route to the internet gateway
  - To allow a private subnet to access the internet while remaining private we can use:
    - **NAT Gateway** - AWS managed
    - **NAT instance** - self-managed
    - A NAT instance is managed in the public subnet, and links to the IGW


- Network ACL (NACL)
  - A firewall which controls traffic to and from **subnet**
  - Can have ALLOW or DENY rules on IP addresses
  - Are attached at a subnet level
- A Security Group
  - Controls traffic to and from an ENI (Elastic Network Interface) or EC2 **instance**
  - Can have ALLOW only, on IP addresses or other SG's
- **VPC Flow Logs**
  - Captures information about IP traffic in instances to monitor and troubleshoot connection issues
  - Logs VPC, Subnet and ENI flows#


- VPC Peering
  - **Connects 2-VPC's** privately using AWS Networks
  - Makes them behave as if they were in the same network
    - Must ensure they dont not have overlapping CIDR (IP address range)
    - It is **not transitive** - cannot link A-C even if A-B and B-C are linked
- VPC Endpoint Gateway
  - VPC Endpoints allow to connect to AWS Services using a private network instead of a public www network
  - Gives enhanced security and lower latency when connecting to AWS services
  - S3 and DynamoDB have a VPC Gateway endpoint. Other services have Interface Endpoint. 
- Site to Site VPN
  - Connects on-prem VPC to AWS
  - Connection is auto encrypted
  - Goes over public internet
- Direction Connection(DX)
  - Establish a physical connection between on-prem and AWS
  - Connection is private, secure and fast. 
  - Takes a month plus to establish


- Typical 2-tier solution architecture
  - Public subnet: multi-AZ ELB
  - Private subnet: EC2 instances in auto-scaling group. In different AZ's
  - Data subnet (also private): Amazon RDS and ElastiCache
- A LAMP Stack on an EC2 means:
  - **Linux** OS
  - **Apache** web server
  - **MySQL** database on DBS
  - **PHP** application logic
  - (Can add ElastiCache and EBS drive)

### Personal notes
