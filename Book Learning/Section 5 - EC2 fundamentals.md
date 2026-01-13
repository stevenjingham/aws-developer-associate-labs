# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 5 - EC2 fundamentals
> 
> Date: January 2026

---

## Purpose
- **EC2 (Elastic Compute Cloud)** provides **Infrastructure as a Service (IaaS)** on AWS
- Used to run **virtual servers** in the cloud with full control over OS, compute, storage, and networking
- Core building block for hosting applications on AWS
## Key Concepts
- **EC2 Instance** = virtual machine
- **EBS** = block storage attached to EC2
- **ELB** = distributes traffic across instances
- **Auto Scaling Group (ASG)** = automatically scales EC2 instances
- **Instance Types** define CPU, memory, networking capacity
- **Security Groups** act as instance-level firewalls
- **EC2 User Data** runs bootstrap scripts at first launch
- **IAM Roles** provide permissions to EC2 without access keys
## How It Works
- You launch an EC2 instance by selecting:
  - **AMI** (OS image: Linux, Windows, macOS)
  - **Instance type** (CPU, RAM, network performance)
  - **Storage** (EBS volumes)
  - **Security groups** (firewall rules)
  - **Key pair** (for SSH access)
- Optional **User Data script** runs automatically on first boot (e.g. install software)
- Instances can be:
  - Accessed via **SSH (port 22)** using key pairs
  - Scaled manually or automatically via **ASG**
- EC2 instances can assume **IAM roles** to call AWS APIs securely

## Code / Config

## Common Pitfalls
- Forgetting to open port 22 / 80 / 443 in security groups
- Using 0.0.0.0/0 for SSH in production 
- Confusing security groups (stateful) with NACLs 
- Hard-coding AWS credentials instead of using IAM roles 
- Losing the private key (.pem) file 
- Assuming spot instances are reliable for long-running workloads

## Key exam notes
- **EC2 is region-based**, but instances run in a specific **Availability Zone (AZ)**
- Prefer **IAM roles** over access keys for EC2
- **User Data** runs only on **first launch** (unless the instance is recreated)

### Security Groups
- Only **allow** rules (no explicit deny)
- **Stateful** (return traffic is automatically allowed)
- Locked to a **region**
- Can reference **other security groups**

### Instance Families
- `t` / `m` → General purpose
- `c` → Compute optimised
- `r` → Memory optimised
- `i` / `d` → Storage optimised

### EC2 Purchase Options
- **On-Demand** → short workloads, pay per second
- **Reserved** → long workloads, lowest cost, fixed attributes
- **Savings Plans** → commitment to usage
- **Spot** → cheapest, can be interrupted
- **Dedicated Host** → compliance / BYOL
- **Capacity Reservation** → guaranteed capacity, no discount


---

## Detailed Notes

### Video notes
- EC2 is **Elastic Compute Cloud** - it is a way to do IaaS on AWS. It allows:
  - Renting virtual machines (EC2)
  - Storing data (EBS)
  - Distributing load (ELB)
  - Scaling service using auto-scaling group (ASG)
- When setting up the EC2 instances can define the
  - OS (Linux, Windows or MacOS)
  - CPU, RAM, Storage Space
  - Network card (speed, IP address)
  - Firewall rules for security groups
  - Bootstrap script for EC2 User Data (i.e. running commands when machine starts e.g. install software)


- Hands-on creating an EC2 instance with a simple bootstrap script to output Hello World web page


- EC2 instance type basics
  - There are different types 
    - e.g. m5.2xlarge
      - m: instance class 
      - 5: generation
      - 2xlarge: size
    - There is:
      - **General purpose** - t - good for diverse workloads (e.g. web servers). Balances compute, memory, networking
      - **Compute optimised** - c - good for batch processing, media, machine learning
      - **Memory optimised** - r - fast performance for large data sets in memory (e.g. databases, real time processing of unstructured data)
      - **Storage optimised** - i or d - for high read and write for non-sql databases 


- Security groups in AWS control traffic into or out of EC2 instances
  - They only contain allow rules. They can reference by IP or security group
  - It acts as a firewall to the EC2 instance
  - They regulate access to ports, authorised IP ranges, control of inbound and outbound network
  - SG's are locked to a region
  - Good practice to maintain one separate SG for SSH access
  - Can reference other security groups in the rules (e.g. can allow inbound traffic from SG1 and SG2, but not SG3)
  - Classic ports to know
    - 21 - FTP
    - 22 = SSH 
    - 80 = HTTP 
    - 443 = HTTPS
    - 3389 = Remote Desktop Protocol for remote windows instance


- EC2 Instance Connect allows to connect to EC2 via SSH
  - SSH allows control of EC2 instance using CLI via Port 22 via a local machine
  - SSH handles encryption etc. 
  - It uses key-pair authentication


- Hands-on using pem file to setup SSH connection to EC2 instance
```ssh -i "pemfilename.pem" ec2-user@{ipAddressOfEc2Instance```
  - Can also use "Connect" UI within AWS on browser to use "E2 Instance Connect"
- Hands-on to use IAM role as security on EC2, allowing to run ```aws iam list-users```


- EC2 instance purchase options:
  1) On-demand - short workload, predictable pricing, price per second
  2) Reserved (1 & 3 years) - long workloads, lower price, reserve specific instance attributes (e.g. region, type, OS)
  3) Saving plan (1 & 3 years) - commitment to usage (e.g. £10/hour), long workload, locked to instance family and region 
  4) Spot instances - short workload, cheap, can lose instances so less reliable. Define a max price and will lose instance if goes over. Good for batch jobs that can stop
  5) Dedicated host - book entire physical server, fully dedicated to use. Good for Bring Your Own Licence, or strong compliance need
  6) Capacity reservation - can access capacity at any time. No pricing discount. Cost even if not using the instance. Good for short term work in specific AZ. 

### Personal notes
- Hands on setting a budget within AWS, to enable auto-emails etc. (Not really linked to EC2 section, but was good practice and could be used in real apps)
- Timeouts when aiming to access an EC2 instance, check the security group rules as this is the likely cause. 
- Putty allows Windows <10 to use SSH, to connect to EC2 instances
- Dont enter access key etc into EC2 instance, as other users could then access this