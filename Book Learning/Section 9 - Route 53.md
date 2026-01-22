# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 9 - Route 53
> 
> Date: January 26

---

## Purpose


## Key Concepts

## How It Works


## Code / Config

## Common Pitfalls

## Key exam notes

---

## Detailed Notes

### Video notes
- DNS (Domain Name System) transfers a human friendly hostname to an IP address
  - It is the backbone of the internet
  - There is a heirarchy of the hostname
    - Top Level e.g. .com
    - Second Level Domain e.g. google.com
    - Subdomain - ww.google.com
    - FQDN Fully Qualified Domain Name - api.www.google.com
    - Protocol e.g. http://
    - URL = protocol + FQDN
- Route 53 is a **domain registrar** for AWS
- A Name Server (NS) resolves DNS queries
- Process to get to websiteL
  - Browser request to URL (e.g. www.example.com)
  - Lookup in Local DNS Server (Cached). If not present:
    - look up in Root DNS server (says go to .com NS)
    - Look up in TLD DNS .com server (says go to example.com NS)
    - Look up in SLD DNS example.com server (gives IP address of hostname)
    - Browser goes to IP address of site


- Route 53 is a high available, scalable and fully mananged DNS
  - It is **authoritive** meaning the customer can update DNS records
- It is also a Domain Registrar
- It contains **records** with info on:
  - Domain name
  - Record Type - supports **A / AAAA / CNAME / NS** / and others
  - IP Value
  - Routing policy (e.g. respond to queries)
  - TTL
- Record Types
  - A - maps hostname to IPv4
  - AAAA - maps to IPv6
  - CNAME - maps hostname to hostname 
    - Target hostname must be A, AAAA
  - NS
    - Name Servers for the Hosted Zone. Controls how traffic is routed for a domain
- Hosted Zones
  - A container for records that define how to route traffic to a domain and sub-domain
  - **Public Hosted Zone** - contain records that specify how to route public domain names over internet
  - **Private Hosted Zone** - contains records to specify how to route on VPC (private domain names). Only queried within VPC. 


- Time To Live (TTL)
  - Tells client (i.e. local) how long to cache the result of the DNS request
  - Means requests to Route 53 are lower, as results are cached
    - High TLL = less traffic to R53, but longer to change record
    - Low TLL = more traffic, but less time to change record


- CNAME vs Alias
  - AWS resources (e.g. ELB) expose an AWS hostname. Uusually AWS services do not have a fixed IP address, but do have a hostname
  - CNAME allows pointing to another hostname, but **not** for the root domain (e.g. not mydomain.com, www.mydomain.com is OK)
    - e.g. can do app.mydomain.com --> blabla.newdomain.com
    - CNAME is DNS wide, not unique to R53
  - Alias points a hostname to an AWS resource
    - Works for root and non-root domain
    - Always of type A/AAAA
    - Is a Route 53 concept. 
    - Good for ELB, CloudFront, API Gateway, etc 


- Routing Policies
  - Defines how Route 53 responds to DNS query
    - N.B - does not route traffic, just gives response to DNS query
  - Route 53 supports the routing policies:
    - Simple 
    - Weighted
    - Failover
    - Latency Based
    - Geolocation
    - Multi-value answer
    - Geoproximity


- **Simple Routing Policy**
  - Routes traffic to single resource
    - Can specify multiples values in one record, random one will be chosen by client. 
  - Can't be associated with health check


- **Weighted policy routing**
  - Controls the percentage of requests that go to each resource
    - e.g. 70% to one, 30% to second
  - Set a weight (doesnt not need to sum to 100)
  - Good for load balancing across regions, testing new app versions
    - Set weight to 0 to stop sending traffic to resource (unless all at 0)
  - Can be associated with health check
  - DNS records must have same name and type


- **Latency routing policy**
  - Redirect to resource that has the least latency (closest to us)
    - Good when user latency is priority
    - Latency defined on traffic between users and AWS regions
  - Can be associated with health check


- Health checks
  - Check the health of a public resource
  - If the health check shows a resource is down, then can make use of **Automated DNS Failover**
    - Can define the healthy threshold for a health checker (e.g. 3 failures in 1min)
  - Can check
    - Endpoint (global health checkers check endpoint, if 18% say healthy it is healthy). Check for 200/300 response etc. 
      - Need to allow Route 53 healthchecks in firewall settings
    - Other health checks (calculated health checks)
    - CloudWatch alarms
  - For a **private resource** we can use a CloudWatch alarm that is linked to the health check (as standard health check is outside VPC)


- **Failure routing policy**
  - Route 53 has a **primary and secondary** record
    - If primary has healthcheck down, will be auto-routed to secondary
    - Only have single primary and secondary


- **Geolocation routing policy**
  - Based on where the user is located (different to latency!)
    - Specified by continent, country or US state
  - Should create a **default** record in case no match in location
  - Use case is website localisation, restrict content distriubtion
  - Can be associated with health checks


- **Geoproximity routing policy**
  - Routes traffic to resources based on geographic location of users and resources
  - Ability to shift more traffic to resources based on a defined **bias**
    - Change size of geographic region can change the bias
      - Expand bias = more traffic to resource
      - Shrink bias = less traffic
      - Can be value -99-99
  - Resources can be AWS resources, or non-AWS resources where a long/lat is needed
  - Must use Route 53 Traffic Flow for this feature
    - Traffic flow allows us to create Traffic Policy
    - Its a UI tool to allow complex routing (can define all the policies we've seen, not just geoproximity)
      - Although very useful for geoproximity as gives graphical view
    - 


- **IP-based routing policy**
  - Routing is based on users IP address
    - Use case to optimise performance, reduce network costs. 
  - In R53, provide list of CIDR's for your clients and corresponding endpoint to connect to.
  - Only useful if know IP of clients to allow routing to be applicable. Can set a default record for unknown IP.


- **Multi-value routing policy**
  - Route traffic to multiple resources, so R53 returns multiple values
  - Can be associated with health checks
  - Can return upto 8x records, healthy resources only returned.
  - Not suitable substitute for ELB, as client chooses which to use. 
  - Good use case is to replace a **simple routing policy**, as in simple the resource may be unhealthy.


- Domain registrar vs DNS service
  - Buy / register a domain name on a **Domain registrar** (e.g. GoDaddy, Amazon Registrar)
  - **DNS service** allows management of DNS records
    - Usually can use DNS Service provided by registrar, but dont have to
    - e.g. can buy domain on GoDaddy and use Route 53 to manage


### Personal notes
