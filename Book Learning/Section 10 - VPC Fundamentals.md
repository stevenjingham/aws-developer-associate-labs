# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 10 - VPC Fundamentals
> 
> Date: January 2026

---

## Purpose
- Provide a **logically isolated private network** in AWS to deploy resources securely
- Control **networking, routing, and access** between resources and the internet
- Enable **secure architectures** (public-facing apps with private backends)
- Form the foundation for most AWS services (EC2, RDS, ALB, Lambda, etc.)

## Key Concepts
- **VPC (Virtual Private Cloud)**
  - A private network in AWS
  - **Regional** resource
- **Subnets**
  - Sub-divisions of a VPC
  - **AZ-specific**
  - Public subnet → has route to Internet Gateway
  - Private subnet → no direct internet access
- **Route Tables**
  - Define where traffic is allowed to go
  - Control access between subnets, IGW, NAT, VPC peers
- **Internet Gateway (IGW)**
  - Allows resources in public subnets to access the internet
- **NAT**
  - Allows **outbound-only internet access** from private subnets
  - NAT Gateway (managed, recommended)
  - NAT Instance (self-managed, legacy)
- **Security Layers**
  - Security Groups → instance/ENI level, allow-only, stateful
  - NACLs → subnet level, allow + deny, stateless
- **Connectivity**
  - VPC Peering
  - VPC Endpoints
  - Site-to-Site VPN
  - Direct Connect (DX)

## How It Works
- A VPC is created with a **CIDR block** (IP range)
- Subnets divide the VPC across **Availability Zones**
- Traffic flow is controlled by:
  - Route tables (where traffic goes)
  - Security Groups (instance firewall)
  - NACLs (subnet firewall)
- Internet access:
  - Public subnet → route to IGW
  - Private subnet → route to NAT → IGW
- Private AWS access:
  - VPC Endpoints allow access to AWS services **without using the public internet**
- On-prem connectivity:
  - VPN → encrypted, over public internet
  - Direct Connect → physical, private, high throughput

## Code / Config

## Common Pitfalls

## Key exam notes
- VPC = **regional**, Subnet = **AZ**
- Public subnet = route to **Internet Gateway**
- Private subnet + internet access = **NAT Gateway**
- Security Groups:
  - Allow-only
  - Stateful
  - Instance/ENI level
- NACLs:
  - Allow + Deny
  - Stateless
  - Subnet level
- VPC Peering:
  - No overlapping CIDRs
  - Not transitive
- VPC Endpoints:
  - Gateway Endpoint → S3, DynamoDB
  - Interface Endpoint → most other services
- Direct Connect:
  - Private, fast, expensive, slow to provision
- Typical 2-tier architecture:
  - Public subnet → ALB
  - Private subnet → EC2 ASG
  - Data subnet → RDS / ElastiCache
- LAMP stack:
  - Linux, Apache, MySQL, PHP
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
