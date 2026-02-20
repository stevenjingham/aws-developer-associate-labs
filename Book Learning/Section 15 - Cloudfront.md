# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 15 - Cloudfront
> 
> Date: February 2026

---

## Purpose
CloudFront is a **Content Delivery Network (CDN)** that improves performance by caching content at global **edge locations (Points of Presence)**.

It helps to:

- Improve read performance and user experience
- Reduce load on backend servers
- Provide built-in DDoS protection
- Securely distribute content worldwide

Unlike S3 Cross-Region Replication:
- CloudFront caches content globally
- S3 CRR replicates data between buckets (not caching, not edge-based)

## Key Concepts
### Edge Locations
- Users connect to nearest edge location
- Content is cached at the edge after first request

---

### Origins

CloudFront fetches content from an origin:

- S3 bucket (secured using Origin Access Control)
- VPC Origin (private ALB/EC2 in private subnet)
- Custom Origin (HTTP endpoint)

---

### Cache Key

Each cached object is identified by a **Cache Key**.

Default:
```
Hostname + URL path
```

Can include (via Cache Policy):
- Headers
- Cookies
- Query strings

Goal:
> Maximise Cache Hit Ratio

TTL:
- Configurable (0 seconds to 1 year)

---

### Cache Behaviour

Defines routing + caching rules per path pattern.

Examples:
```
/*        → EC2
/images/* → S3
/api/*    → ALB
```

- Default behaviour applies to `/*`
- More specific paths override default

---

### Origin Request Policy

Controls what CloudFront forwards to the origin:

- Headers
- Cookies
- Query strings

Forward too much:
- Lower cache efficiency
- More origin calls
- Higher cost

Forward too little:
- Authentication failures
- Incorrect API responses

---

### Cache Invalidation

Used when backend content changes before TTL expires.

Options:
```
*           → invalidate all
/images/*   → invalidate specific path
```

Uses:
```
CreateInvalidation API
```

---

### VPC Origin vs Public Origin

VPC Origin:
- Connects to private ALB/EC2
- No public exposure required
- More secure

Public Origin:
- Uses public IP
- Must allow CloudFront IP ranges in security group
- Less secure

---

### Geo-Restriction

Restrict access by country:

- Allow list
- Block list

Based on Geo-IP location.

---

### Signed URLs / Signed Cookies

Used to distribute private or paid content.

Signed URL:
- Grants access to single file

Signed Cookie:
- Grants access to multiple files

Can restrict by:
- Expiration time
- IP address
- Path
- Trusted signer

---

### Trusted Signers

Two options:

1. Trusted Key Group (Recommended)
  - API-based key rotation
  - Better security

2. AWS Account Key Pair
  - Managed by root account
  - Not recommended

Public / Private Key Model:
- Public key → stored in CloudFront
- Private key → used by application to sign URLs

---

### Origin Groups (Failover)

Provides high availability.

- Primary origin
- Secondary origin

If primary fails → automatically routes to secondary.

---

### Multiple Origins

Route based on content type:

```
/*      → S3
/api/*  → ALB
```

---

### Field-Level Encryption

Extra encryption layer beyond HTTPS.

- Encrypt sensitive fields at edge
- Decrypt at backend
- Uses public key encryption
- Up to 10 fields supported

---

### Pricing & Price Classes

Data transfer cost varies by region.

Price Classes:
- PC-All → all edge locations
- PC-200 → excludes most expensive regions
- PC-100 → only cheapest regions

---

### Real-Time Logs

CloudFront logs can stream to:

- Kinesis Data Streams
- Lambda
- Data Firehose

Used for monitoring and performance analysis.


## How It Works
1. User makes request.
2. DNS routes to nearest CloudFront edge location.
3. Edge checks cache:
  - Cache hit → return immediately.
  - Cache miss → forward request to origin.
4. Origin responds.
5. Edge caches response (based on TTL + cache policy).
6. Future requests served from edge.

## Code / Config

## Common Pitfalls

## Key exam notes
- CloudFront = CDN (performance + caching).
- Cache lives at edge locations.
- Default cache key = hostname + path.
- Cache Policy affects cache key.
- Origin Request Policy affects what is sent to origin.
- Supports S3, ALB, EC2, private VPC origins.
- Signed URL = single file.
- Signed Cookie = multiple files.
- Geo-restriction works at country level.
- Origin groups provide failover.
- Field-level encryption adds security beyond HTTPS.
- Real-time logs integrate with Kinesis.
---

## Detailed Notes

### Video notes
- Cloudfront is a **Content Delivery Network**, improving read performance as the content is **cached at the edge**
  - Meaning user experience is improved
  - Lots of Points of Presence globally
  - Gives DDos protection
- User connects to local point of presence; which connects to server. Result is cached in edge location
- Origins can be
  - S3 bucket (secured using Origin Access Control)
  - VPC Origin (for private subnets)
  - Customer Origin (HTTP) - e.g. S3 website
- Cloudfront vs S3 Cross-Region Replication
  - S3CRR is not global; files are updated real time (not cached)



- The cache lives at each Cloudfront edge location
  - Each object in the cache is identified using the unique **Cache Key**
    - Default = hostname + resource portion of URL
    - If have a app that serves content based on user/location etc, can add other elements using a **Cloudfront cache policy**
      - e.g. headers, cookie, query strings (could be noone, whitelist, all-expect, all)
        - Need header from the server
    - Can also control the TLL (0s to 1yr)
  - Want to maximise the Cache hit ratio
  - Can invalidate part of cache using **CreateInvalidation** API


- Cache Invalidations
  - (use in case of back-end update, if TTL is too long)
  - Can invalidate all files (e.g. *), or special path (e.g. /images/*)
  - Can invalidate part of cache using **CreateInvalidation** API


- Cache Behaviour
  - May want to configure different settings for different path URLs
    - e.g may want to route /* to EC2, or /images to S3 bucket
  - e.g. from Cloudfront, may have a route from cloudfront /login to EC2 instance; which returns a signedCookie. This cookie can then be used by Ckouldfront on subsequent requests to allow access to /* endpoints
  - There is a default cache behaviour that applies to /* unless another behaviour overrules. 


- An Origin Request Policy controls what CloudFront forwards to the origin.
  - CloudFront does NOT automatically send everything.
  - You choose whether to forward:
    - Headers
    - Cookies
    - Query strings
  - If you forward too much:
    - Cache efficiency drops
    - More origin calls
    - Higher costSlower performance
  - If you forward too little:
    - Authentication may fail
    - API responses may be wrong


- Cloudfront usign ALB/EC2 as an origin
  - Using VPC Origin allows to deliver conent from apps hosted on VPC private subnet 
    - So no need to expose them to internet
  - Cloudfront at the edge location connects to VPC Origin with VPC
    - The Origin then connects to a private subnet containing ALB, NLB, EC2 etc
  - Can do via public IP connecting Edge location to ALB/EC2 instance
    - But must allow IP address within ALB/EC2 security group settings
    - Less secure than the VPC method


- Geo-restriction
  - Can limit access by country based on Geo-IP address
  - Can set either ALLOW or BLOCK list


- Signed URLs / Signed Cookies
  - This use case is to distribute paid shared content to premium users over the world
  - Can use signed url/cookie with:
    - URL expiratino
    - IP ranges to access data from (ie. client ID)
    - Trusted signers (which AWS account can create signed urls)
  - Signed URL is for single file; signle cookie can be used for many files
  - Cloudfront Signed URL:
    - allows access to path, no matter the origin
    - account wide key-pair, only root can mange
    - can filter by IP, path, date, exp
    - caching features
    - (S3 signed url, issues reqyest as per person making pre-signed url, limited life, uses IAM key of IAM principal)


- Cloudfront Signed URL process
  - Two types of signers
    1. Trusted key group (recommended)
       - Can leaverage APIs to create and rotate keys
    2. AWS Account that contains key pair
       - Need to manage keys from root account
       - But not recmmendined as need to use root account
  - A public and private key is created
    - Public key is used by Cloudfront to verify URLS
    - Private key is used by applications (eg. within EC2 instance) to sign URLs
      - Keep this private
      

- Advanced options
  - Pricing: cost of data out from each edge varies by location
  - Price classes: can reduce price by reducing locations
    - PC-All - all regions
    - PC-200 - most regions, but exludes most expensive regions
    - PC-100 - gets only 100 cheapest
  - Multiple Origin
    - ablity to route different kinds of irgins based on content type
      - e.g. /* to S3; /api/* to ALB
  - Origin groups
    - To increase high-availability; if one fails, use secondary origin
    - e.g. connect to primary EC2 instance. If get error, use Origin B
    - Can use with S3 buckets to get high availability multi-region availability
  - Field Level Encryption
    - Can protect sensitive information through application stack
      - Add additional layer alongside HTTPS 
    - Sensitive info encrypted at edge, close to user
    - Can assign 10x field that can be encrypted
    - Encrypt using public key at edge. End app can decrypt the field


- Real time logs can be sent to **Kinesis Data Stream**
  - Can monitor and take action based on content delivery performance
  - Kinesis connect to Lambda/DataFirehose to improve performance


### Personal notes
