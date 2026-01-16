# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: RDS + Aurora + ElastiCache
> 
> Date: January 2026

---

## Purpose
Provide fully managed relational databases and in-memory data stores on AWS, removing the operational burden of running databases yourself while improving availability, scalability, performance, and security.

This includes:
- **RDS & Aurora** for transactional SQL workloads
- **ElastiCache & MemoryDB** for ultra-fast in-memory caching and state management

## Key Concepts
### Relational Databases (RDS & Aurora)
- **RDS**: Managed SQL databases (Postgres, MySQL, MariaDB, Oracle, IBM DB2)
- **Aurora**: AWS-optimized MySQL/Postgres-compatible database with higher performance
- **Read Replicas**: Scale reads (ASYNC, eventually consistent)
- **Multi-AZ**: High availability & disaster recovery (SYNC, standby only)
- **Storage**: Backed by EBS (RDS), auto-scaling storage
- **Security**: KMS encryption, TLS, IAM auth, Security Groups
- **No SSH access** (fully managed)

### RDS Proxy
- Connection pooling layer for RDS/Aurora
- Lives **inside a VPC only**
- Reduces DB load and failover time
- Common with Lambda-heavy workloads

### In-Memory Datastores (ElastiCache & MemoryDB)
- **Redis / Memcached**: Key-value, in-memory databases
- **Purpose**: Reduce DB load, ultra-low latency reads
- **Stateless apps**: App instances don’t store session/state locally
- **Caching patterns**: Cache-aside, write-through
- **TTL & eviction** strategies required
## How It Works
### RDS
- AWS provisions DB instance, manages OS, backups, patching
- Application connects via a single endpoint
- Optional:
    - **Read Replicas** for scaling reads
    - **Multi-AZ** for automatic failover
- Storage can auto-scale up to a defined maximum

### Aurora
- Uses distributed storage:
    - 6 copies across 3 AZs
    - Writes require 4/6 copies
    - Reads require 3/6 copies
- One **writer endpoint** (master)
- One **reader endpoint** (load balances replicas)
- Faster replication and failover than standard RDS

### RDS Proxy
- App connects to proxy instead of DB
- Proxy pools and reuses DB connections
- IAM authentication supported
- Must be accessed from within the VPC

### ElastiCache / MemoryDB
- Application checks cache **before** hitting the DB
- Cache holds frequently accessed data in memory
- Redis supports replication, backups, persistence
- Memcached is simpler, faster, but non-persistent
- MemoryDB adds durability + Redis compatibility


## Code / Config

## Common Pitfalls

## Key exam notes
- **RDS does not allow SSH** access
- **Read Replicas** = scaling, **Multi-AZ** = availability
- **Aurora** is cloud-native, faster, auto-scaling storage
- **RDS Proxy is VPC-only**
- **Redis** = HA, persistence, backups
- **Memcached** = no replication, no persistence
- Cache improves performance & enables **stateless applications**
- Lambda + RDS → **use RDS Proxy**
- Encryption must be enabled at creation (or via snapshot restore)
---

## Detailed Notes

### Video notes
- RBS is a Relational Database Service (SQL)
  - Postgres, MySQL, MariaDB, Oracle, IMB DB2
  - Use RBS as it is a managed service, so better than using own DB on EC2
    - Automated provisioning, OS patching
    - Backups and restore to Point in Time
    - Read replicas for read performance
    - Multi-AZ setup
    - Scaling vertical and horizontal
      - RDS Auto-scaling detects and increases storage 
      - Need to set Maximum Storage Threshold
      - Useful for apps with unpredictable workloads
    - Storage backed by EBS
  - However, **cannot** SSH connect into RDS instance


- Read replicas allow scale by creating replicas of a DB, which only allow reads
  - Can create up to 15x, over same AZ, region or cross region
  - Read replicas is ASYNC, so reads are eventually consistent
  - Replicas can be promoted to their own DB
  - Example use case is to allow a reporting/monitoring application to access the DB data
    - Means production application is not impacted
  - Network cost of Read Replicas:
    - In same region - no cost
    - In different region - cost
    - (N.B. usually cost if data transfers over AZ for AWS)
- RDS Multi AZ
  - Is used for disaster recovery, in case of loss of Master DB
  - It is SYNC replication. One DNS name given to the application to connect to DB, to allow automatical failover to standby DB
  - Standby DB is not used for read
  - (NB can setup read replicas as Multi-AZ for Disaster Recovery)
- To go from single AZ to multi-AZ
  - Zero downtime or need to stop DB
  - Modify option on AWS UI
  - Snapshot created of master, Standby loaded from snapshot, Sync estabilished to get any new writes after snapshot


- Amazon Aurora
  - Is a proprietary tech from AWS
  - Postgres and MySQL are supported by Aurora DB. ie. drivers will work as if AUrora was a MySQL db
  - It is cloud optimised. Claims 5x improvement over MySQL
  - Storage auto grows from 10Gb to 256Tb
  - Can have 15 read replicas and replication is fast
- It has High Availability and Read Scaling
  - 6 copies of data across 3AZ
    - 4 copies for writes
    - 3 copies for reads
    - Self-healing with peer-peer replication
    - Storage is striped across 100s of volumes, which is auto-expanding
    - Supports for cross region replication
  - One Aurora instance is master, taking writes
    - Can have 15 replicas to serve reads
- Aurora DB cluser
  - Application is given a Writer Endpoint pointing to the master DB
    - Master writes data, and is replaced in case of failure
  - Application also given a Reader Endpoint
    - Which is a Connection Load Balancer to all the read replicas


- RDS & Aurora Security
  - At-rest encryption
    - Using AWS KMS
    - If master is not encrypted, replicas are not
    - To encrypt an existing DB, make snapshot and restore as encrypted
  - In-flight encryption
    - TLS-ready by default, using AWS TLS root certificates
- Can use IAM roles for authentication
- Can use Security Groups to control network access
- No SSH anavilable
- Audit logs can be enabled, and sent to CloudWatch for longer retentino


- Amazon RDS Proxy, used for connecting to DB
  - Allows apps to pool and share DB connections
  - Good use if there is lots of applications connecting to same DB, as controlled via Proxy
  - Serverless, autoscaling and highly-available (multui-AZ)
  - Reduces failover time
  - Increase security, by enforcing IAM Authentication
  - RDS proxy is not available publicly - must be accessed from VPC (Virtual Private Cloud)
  - Very useful in Lambda functions, as they can multiply lots and the proxy pools the connects from functions into the DB


- ElastiCache
    - Used to get managed Redis or Memcached
    - Caches are in memory databases, with really high performance, low latency
    - Reduced load off databases with read intensive workloads
    - Helps make application stateless 
    - AWS takes care of OS mantainance, setup, monitoring
    - Need application to query cache, before DB read. This reduces reads to DB.
      - Need to write to cache
      - Need a cache invalidation strategy to ensure the cache stays up to date.
    - Redis vs Memcached
      - Redis
        - Multi-AZ
        - Read Replicas to scale and high availability
        - Backup and restore
      - Memcached
        - Sharding of data
        - No high availability/replication
        - Non-persistent 
        - Multi-thread for good performance


- Caching Implementation Considerations
  - Queries
    - Is it safe to cache data? Data may become out of date
    - Is it effective? If data changes quickly it may not be effective
    - Is structured well? eg. key value caching
  - Caching design patterns
    - Lazy Loading / Cache-Aside
      - Pro - Only requested data is cached, after a DB read
      - Pro - Node failures are not fatal, just need to re-populate cache
      - Con - Cache miss results in many calls - read cache, read DB, write cache
      - Con - state data as cache may not updated
    - Write Through 
      - Cache is added when database is updated. Write to DB, means write to cache
      - Pro - Data is never stale. Reads are quick
      - Pro - Write penalty, vs. a read penalty. Good for users as reads are expected to be quicker
      - Con - missing data until it is in cache. Mitigation is to implement lazy loading too. 
      - Con - cache churn, as cache has all data, but may not be read. 
  - Cache Evictions and Time-To-Live (TTL)
    - Cache evection can occur in 3-ways
      1) Delete item in cache explicitly
      2) Least recently used data when memory is full
      3) TTL set in cache (good when not using write-through)



- Amazon MemoryDB for Redis
  - Redis compatible, durable, in-memory database service
  - Ultra-fast performance with over 160M rps
  - Multi-AZ, scales 10Gb to 100sTb

  

### Personal notes
