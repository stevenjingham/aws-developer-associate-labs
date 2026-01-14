# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 6 - EC2 Instance Storage
> 
> Date: January 2026

---

## Purpose
- **EC2 (Elastic Compute Cloud)** provides scalable virtual servers in the cloud
- Solves the need for **on-demand compute infrastructure** without managing physical hardware
- Enables full control over OS, compute, storage, and networking (IaaS)

---

## Key Concepts
- **EC2 Instance** – a virtual machine
- **AMI** – template defining OS and initial software
- **Instance Type** – CPU, memory, networking capacity
- **EBS** – block storage for EC2
- **Security Groups** – instance-level firewalls
- **Key Pairs** – used for SSH access
- **User Data** – bootstrap scripts run at launch
- **IAM Roles** – grant permissions to EC2 securely
- **Availability Zones (AZs)** – physical data centres within a region

## How It Works
- You launch an EC2 instance by choosing:
    - An **AMI** (Linux, Windows, macOS)
    - An **instance type**
    - **Storage** (EBS volumes)
    - **Security groups** (firewall rules)
    - A **key pair** for access
- The instance runs in a **specific AZ** within a region
- Optional **User Data** scripts execute automatically on first boot
- Instances can:
    - Be accessed via **SSH** or browser-based **EC2 Instance Connect**
    - Scale manually or via **Auto Scaling Groups**
    - Assume **IAM roles** to access AWS services

## Code / Config

## Common Pitfalls


## Key Exam Notes


---

## Detailed Notes

### Video notes

- EBS (Elastic Block Storage) is a network drive that attached to EC2 instances. 
  - Allows to **persist data** even after termination
  - Can only be mounted to **one instance** at a time (although some EBS do have "multi-attach" feature now)
    - Can attach 2x ECB to one instance
  - Bound to a **specific availability zone**
    - To move volume to another AZ, need to take a snapshot
  - As it is a network drive:
    - there may be **latency** in comms
    - they can be **detached and re-attached** to another instance quickly
  - Have a provisioned capacity (size in Gb, and I/OPS in/outpersecond)
    - Can increase over time
  - There is a **"delete on termination"** option to delete the ECB when the EC2 instance is deleted
    - Root **is deleted** as default
    - Non-root are **not deleted** as default. 


- Hands-on creating volumes via AWS UI
  - Showed that cannot attach ECB to EC2 in different AZ


- EBS Snapshot is a **backup of an EBS volume** at a point of time. 
  - Recommended to detach volume to do snapshot (but not required)
  - Can copy snapshot over AZ or Region
  - **EBS Snapshot Archive** allows cheaper storage, but longer restore
  - **Recycle Bin** - Can setup rules for retaining deleted snapshots to prevent accidental deletion
    - Specify retention (1 day to 1 year)
  - **Fast Snapshot Restore (FSR)**
    - Forces full initialisation of snapshot with **no latency, but costly**


- Hands-on with snapshots


- Amazon Machine Image (AMI) is a **customisation of an EC2 instance**
  - Can define own s/w, configuration, OS, monitoring
  - Faster boot / configuration time as s/w is pre-packaged
  - AMI's are built for a **specific region** so cannot use in a different region without copying it across
  - Can launch an EC2 from :
    - Public AMI (from AWS)
    - Define own AMI
    - AMI marketplace
  - In hands-on we will:
    1) Create an EC2 instance and customise
    2) Stop instance (for data integrity)
    3) Build AMI from instance
    4) Launch instance from the AMI

    

- EBS Volume Types
  1) gp2/gp3 = general purpose SSD
     - cost effective, virtual desktops, dev and test env
     - 1Gb-16Tb
  2) io1/io2 = highest performance SSD for mission-critical, low latency or high throughput
     - Provisioned IOPS
     - Good for database workloads
     - io1 4Gb-16Tb
     - io2 4Gb-64Tb, sub-millisecond latency
  3) st1 = low cost HDD for frequent access, throughput intensive
     - cannot be boot volume
     - 125Gb-16Tb
     - big data, data warehouse
  4) sc1 = lowest cost HDD for less frequent access
     - cannot be boot volume
     - 125Gb-16Tb
     - for infrequent access; lowest cost
  - EBS Volumes are defined by Size | Throughput | IOPS
  - Only gp2/3, io1,io2 can be used as boot volumes



- **EC2 Instance Store** is used for **high performance hardware disk**
    - Better I/O performance
    - Lose storage if stopped (terminology is ephemeral)
    - Good for buffer, cache, temporary content
    - Risk of data loss if hardware fails
        - Responsibility to back-up if needed


- **Multi-attach** allows same EBS volume to attach to multiple instances in same AZ
  - Only for **io1/io2** family
  - Each instance has read/write access
  - Use case for high application availability in clustered Linux apps (e.g. Teradata)
  - Up to 16 EC2 instances


- **Amazon EFS** (Elastic File System)
  - Managed NFS (network file system) that can be **mounted on many EC2**, in multi-AZ
  - Highly available, scalable, expensive and pay-per-use
  - Use case
    - Data sharing, web serving, Wordpress
  - Uses **security group to control access to EFS**
  - Compatible with **Linux** based systems
  - No capacity planning needed
  - Classes
    - EFS Scale
      - Grows automatically
    - Performance mode 
      1) General purpose
      2) Max IO
    - Throughput mode
      1) Bursting - scales with amount of storage for workloads with basic performance
      2) Provisioned - set, regardless of use
      3) Elastic - auto-scales, good for unpredicatable IO needs. Only pay for usage
  - Storage classes
    - Storage tiers
      1) Standard
      2) Infrequent access - lower price
      3) Archive - rarely accessed data (e.g. few times per year), cheapest
    - Can implement **lifecycle policies** to move files between tiers


- Hands-on creating EFS


- EBS vs EFS
  - EBS is one instance, are locked to AZ (other than copy to snapshot)
  - EFS is a network file system to connect to many EC2 instances across many AZ's

### Personal notes
