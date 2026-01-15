# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 7 - ELB and ASG
> 
> Date: January 2026

---

## Purpose
- Provide **scalability**, **high availability**, and **fault tolerance** for applications on AWS
- **Elastic Load Balancers (ELB)** distribute incoming traffic across multiple targets
- **Auto Scaling Groups (ASG)** automatically adjust the number of EC2 instances based on load

## Key Concepts
- **Scalability**
  - **Vertical scaling**: increase instance size (limited by hardware)
  - **Horizontal scaling (elasticity)**: increase number of instances
- **High Availability**
  - Deploy across **multiple Availability Zones**
  - Designed to survive AZ failure
- **Elastic Load Balancer (ELB)**
  - Managed load balancer with a **single DNS endpoint**
  - Performs health checks and SSL termination
- **Load Balancer Types**
  - Classic Load Balancer (deprecated)
  - Application Load Balancer (ALB)
  - Network Load Balancer (NLB)
  - Gateway Load Balancer (GWLB)
- **Auto Scaling Group (ASG)**
  - Maintains min, desired, and max number of EC2 instances
  - Integrates with ELB and CloudWatch
## How It Works
- Users send traffic to an **ELB DNS name**
- ELB distributes requests to healthy targets across AZs
- **Health checks** detect unhealthy instances and stop routing traffic
- **Security groups** restrict EC2 access to only allow traffic from the ELB
- **ALB** routes traffic at Layer 7 using:
  - Paths (`/users`)
  - Hostnames (`api.example.com`)
  - Headers or query strings
- **NLB** routes traffic at Layer 4 using IP and port
- **GWLB** forwards encapsulated traffic to security appliances using GENEVE
- **ASG**:
  - Scales out/in based on CloudWatch metrics
  - Automatically registers/deregisters instances with ELB
  - Replaces unhealthy instances

## Code / Config

## Common Pitfalls

## Key exam notes
- **ELB is managed** (AWS handles maintenance and scaling)
- **ALB**
  - Layer 7 (HTTP/HTTPS)
  - Supports path-, host-, and header-based routing
  - Client IP forwarded via headers
  - Cross-zone load balancing **enabled by default**
- **NLB**
  - Layer 4 (TCP/UDP/TLS)
  - Ultra-low latency, high throughput
  - One static IP per AZ
  - Cross-zone LB **disabled by default** (extra cost)
- **GWLB**
  - Layer 3/4
  - Used for security inspection
  - Uses **GENEVE (UDP 6081)**
- **Sticky Sessions**
  - Use cookies
  - Can cause uneven load distribution
- **Connection Draining / Deregistration Delay**
  - Allows in-flight requests to complete
- **ASG**
  - Free service (pay only for EC2)
  - Uses launch templates
  - Supports instance refresh with minimum healthy percentage
- Common scaling metrics:
  - CPU utilisation
  - Request count per target
  - Network In / Out

| Load Balancer | Layer | Protocols | Best For |
|--------------|-------|-----------|----------|
| ALB | L7 | HTTP, HTTPS | Web apps, APIs, microservices |
| NLB | L4 | TCP, UDP, TLS | High performance, low latency |
| GWLB | L3/L4 | IP (GENEVE) | Security & traffic inspection |

| Layer | Name | Can Inspect | AWS Example |
|-----|-----|------------|------------|
| 7 | Application | URLs, headers, cookies | ALB |
| 4 | Transport | IP + port | NLB |
| 3 | Network | IP addresses | GWLB |
| 2 | Data Link | MAC addresses | (Hidden by AWS) |

---

## Detailed Notes

### Video notes
- Scalability: can hangle higher load by adapting
  - Vertical: increasing the size of an instance. Good for non-distributed system (e.g. database). There is a hardware limit to vertical scaling
  - Horizontal (elasticity): increasing the number of instances. Good for distributed system
- High availability:
  - Running in more than 2 centres (e.g. 2 AZ's)
  - Goal is to survive centre loss

- Load balancing
  - A server that will forward traffic to multiple services (instances) downstream
  - In AWS this is via **Elastic Load Balancer (ELB)**
    - Users just have single endpoint to ELB - DNS
    - Spreads load
    - Handle failures of downstream instances via health checks
    - Provide SSL termination for websites
    - High available across zones
  - ELB is a **managed load balancer**
    - AWS guarantees it is working, taking care of upgrades and maintenance
  - Types of Load Balancer
    1) Classic Load Balancer (deprecated)
    2) Apploication Load Balancer (ALB)
    3) Network Load Balancer (NLB)
    4) Gateway Load Balancer (GWLB)
  - Security
    - Users can access load balancer from anywhere via security group. 
    - Set security group of EC2 instance to security group of ELB, so only allow traffic from the ELB


- ALB
  - Application layer, allowing balancing to multiple HTTP applications. 
  - Can route to mutliple machinces, or mulpiple applications on the same machine
  - Can route to :
    - Different routes on URL (e.g. /users, /transactions)
    - Hostname (e.g. www.domain1.com, www.domain2.com)
    - Query strings / headers
  - Target groups (i.e. where traffic can be sent) include:
    - EC2 instance
    - ECS tasks (containers)
    - Lambda functions (serverless function)
    - Private IP addresses (on prem services)
  - Get a fixed hostname with the ELB, to sent to users
  - Client IP is not seen directly by EC2, but added in headers from ELB to EC2 instance


- Hands-on creating ALB
  - Created "group" which grouped multiple instances as targets
  - Then link this group to the ELB via a **listener**
  - Set Security Group to only allow EC2 instance to accept request from ELB IP
  - Add conditions to the ALB listener to route the traffic (e.g. setup /error path condition)


- Network Load Balancer (NLB) - layer 4
  - Forward **TCP & UDP traffic** to instances
  - Handle millions of requests per second, ultra-low latency
  - **One static IP per AZ**, and supports assigning elastic IP
  - Target groups can be:
    - EC2 instances
    - Private IP addresses
    - Application Load Balancer
  - Again good practice to set EC2 instances to accept traffic from NLB (rather than all traffic)
  - Health checks by NLB, support TCP, HTTP and HTTPS

- Hands on creating NLB



- GWLB
  - Good examples are security (e.g. firewall, intrusion detection)
  - Route traffic to GWLB, to target group of 3rd party security apps, back to GWLB, and then to application
    - It therefore analyses network layer (IP packets, level 3)
  - It combines the funvtion of **Transparant network gateway** (single entry for traffic) and **load balancer**
  - Uses the **GENEVE protocol on port 6081**
    - (Generic Network Virtualization Encapsulation)
    - **GENEVE** is a **network tunnelling protocol** used to encapsulate network traffic so it can be forwarded to network appliances **without changing the original packet**.
    - Allows traffic to be **intercepted, inspected, or filtered**
  - Target groups can be
    - EC2 instances
    - Private IP addresses


- ELB Sticky Sessions
  - Implement **stickiness** so that a client is always redirected to the same load balancer
    - Use case for a users session data
  - Works for CLB, ALB & NLB
  - A **cookie** is used for stickiness and has a expiration date that is controllable
    - Application-based cookies 
      - Custom cookie - generated by target and can include custom attributes
      - Application cookie - generated by ELB
    - Duration based cookie
      - Generated by ELB
  - Warning - may bring imbalance to the backend EC2 instances


- Cross-Zone Load Balancer
  - A CZLB means each load balancer instance will distribute evenly across all registered instances in all AZ's
  - Each ELB is in a different AZ in this example
  - (Without CZLB, the requests are disributed in the AZ of the ELB they are sent to)
  - CZLB is default on ALB
    - No charge for inter-AZ data
  - CZLB is disabled as defaul on NLB & GWLB
    - There is charge for inter-AZ data


- SSL/TLS basics
  - SSL referes to **Secure Socket Layer** used to encrypt connections when traffic is in transit
  - TSL refers to **Transport Layer Security** - it is a newer version
  - Certificates are issued by a Certificate Authority 
    - They must be reset when expired
  - A load balancer uses a certificate for connection to user over HTTPS. Data to EC2 instance is over HTTP
    - X.509 cert
    - Can be managed through AWS Certificate Manager
    - HTTPS listener needs a default certificate, and any optional ones added
      - Clients can use SNI (Server Name Indicator) to specify hostname they reach
        - Solves problem of loading multiple SSL certs onto one web server
        - Only works on ALB and NLB


- Connection Draining or Deregistration delay
  - Allows time to complete "in flight requests" while the instance is de-registering or unhealthy
  - Stops sending new requests to the EC2 instance which is de-registering
  - Can set the time, so based on use case (e.g. low value for quick requests)


- Auto Scaling Group (ASG)
  - Allows scale out or scale in as the load changes (horizontal scaling)
  - Can ensure that a certain min, desired and max number of instances available
  - Automatically register a new instance to ELB
  - Re-create an instance if one is terminated (e.g. if unhealthy) when connected to an ELB
  - ASG are free, just pay for EC2 instances
  - Attributes
    - Launch template (info on how to launch: AMI, data, EBS, security groups etc)
    - Size
    - Scaling properties
  - Can use a CloudWatch alarm to monitor metrics (e.g. CPU) to support scaling requirements


- Scaling Policies
  - Dynamic scaling
    - Target tracking scaling (eg CPU to 40%), simple to setup
    - Simple / Step scaling
      - Steps at set points eg. up at 70% CPU, down at 30%
  - Scheduled scaling
    - Based on date-time usage patterns
  - Predictive scaling
    - forecast load and schedule of scaling ahead of load
- Good metrics to scale on:
  - CPU usage average over instances
  - Request Count Per Target
  - Average Network In/Out load
  - Custom metrics
- Scaling cooldowns
  - After a creation/delete, the ASG will not create/delete any instances to allow metrics to stabilise 


- Instance Refresh
  - Want to update all EC2 instances, when there is a new launch template (e.g. new AMI)
  - Can set a min healthy percentage, so old ones are turned off and new instances are created. Can specify a warm up time for new instances to again make sure they are operational. 







### Personal notes
