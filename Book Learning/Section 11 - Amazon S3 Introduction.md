# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 11 - Amazon S3 Introduction
> 
> Date: January 2026

---

## Purpose
- S3 provides **highly durable, scalable object storage** for almost any use case
- Common uses:
    - Backups & disaster recovery
    - Data archiving
    - Hosting static websites
    - Media hosting
    - Data lakes & analytics
    - Application assets (images, files, logs)

## Key Concepts
- **Buckets**
    - Containers for objects
    - Must be **globally unique**
    - Created at a **regional** level
    - Naming rules:
        - Lowercase only
        - 3–63 characters
        - No underscores, no IP-style names
- **Objects**
    - Files stored in S3
    - Identified by a **key** (full path)
        - Example: `s3://my-bucket/folder1/folder2/file.txt`
    - Max object size: **5 TB**
        - Multipart upload for files >5 GB
- **No directories**
    - S3 uses a **flat namespace**
    - Folders are just prefixes in object keys
- **Durability & Availability**
    - Durability is extremely high and consistent across classes
    - Availability varies by storage class
## How It Works
- Objects are uploaded to buckets using a unique key
- Data is automatically replicated across multiple devices within a region
- Access is controlled using:
    - IAM policies
    - Bucket policies
    - ACLs
- Optional features:
    - Versioning to protect against accidental deletes
    - Replication to copy objects across buckets/regions
    - Storage classes to optimize cost vs access frequency
- S3 can directly serve content (e.g. static websites)

## Code / Config

## Common Pitfalls
- Assuming S3 has real directories (it does not)
- Forgetting buckets must be **globally unique**
- Forgetting to allow public read for static website hosting
- Assuming replication copies existing objects automatically
- Forgetting minimum storage durations (Glacier classes)
- Assuming delete replication includes permanent deletes (it doesn’t)
- Confusing durability with availability
## Key exam notes
S3 = **object storage**, not block or file storage
- Buckets:
    - Global namespace
    - Regional resource
- Objects:
    - Max size 5 TB
    - Multipart upload >5 GB
- Versioning:
    - Protects against accidental deletes
    - Delete creates delete marker
- Replication:
    - Requires versioning
    - No chaining
    - New objects only
- Storage Classes:
    - **Standard** → frequent access, 99.99% availability
    - **Standard-IA** → infrequent access, backups
    - **One Zone-IA** → cheaper, single AZ
    - **Glacier Instant Retrieval** → ms access, archive
    - **Glacier Flexible Retrieval** → minutes to hours
    - **Glacier Deep Archive** → lowest cost, 12–48h retrieval
    - **Intelligent-Tiering** → automatic tiering with small fee
- Static website hosting:
    - Requires public access
    - Bucket name appears in URL
---

## Detailed Notes

### Video notes
- Amazon S3 is a data and storage solution
  - e.g. backup, storage, arhcieve, host media/apps , data lakes, static website
- Files are stored in **buckets** - which must be globally unique
  - Buckets are defined at a rgional level
  - Naming: lowercase, no underscore, 3-63char, no IP, few other rules
- Objects are the files stored
  - Objects have a **key** which gives the path
    - Prefix and object name
    - e.g. s3://my-bucket/folder1/folder2/file.txt
  - Max object size is 5Tb. 
    - Can upload in 5Gb increments in multi-part upload
- There is no directory in S3, just keys


- S3 Security
  - Can be user based, IAM policies (if user IAM allows, or resource policy allows AND no exlicit DENY); 
  - or resource based:
    - Bucket policies - bucket wide rules from the S3 console
      - JSON based: resouce; effect; actions; principal (i.e. user)
      - Can use to grant public access, etc
    - Object Access Control List - finer grain
    - Bucket Access Control List
  - or can encypt objects in S3 using keys


- Can be used to host static websites, where URL will contain the bucket name
  - Need to allow public read on the bucket for www access!
  - e.g. www.mybucket.websitename.amazonaws.com


- Versioning
  - Can be done in S3, by enabling at the bucket level
    - If the file in the bucket is over-written, AWS auto versions it
  - Best practice as avoid accidental deletes, and allows to backout change
  - If delete a file with versioning, it creates a "delete version", which allows to rollback to get original file if required


- Replication
  - Can be Cross-Region Replication(CRR) or Same-Region Replication (SRR)
  - Allows asynchronous replication
  - Buckets can be in different AWS accounts
  - Can give compliance, lower latency etc
  - Only replicated **new replications** - need to do batch replication to do initial replication
    - No chaining, eg. cannot do replication from 1-2 and 2-3 and expect 1-3 to be the same. 
  - Done by created a replication rule in a bucket, and provide the target bucket name to replicate in.
  - Delete markers are not replicated as default, but can set it to be replicated in the rule set. 
    - However, permanent deletes of a file are not replicated. 


-  Storage Classes
  - Defined by **durability and availability**
    - Durability - high, same for all classes. 
    - Availability - measures how ready a service is. e.g. 99.99% means out for 53mins a year
  - Classes of S3
    - Standard: general purpose, 99.99%, used for freq access data, low latency, high throughpout
    - Standard Infrequent access: 99.9%, useful for backups/disaster
    - One Zone Infrequent access: 99.5%, data lost if AZ destroyed, used for 2nd data backup
    - Glacier Instant Retrieval:  millisecond retrieval once per quarter, min storage 90 days
    - Glacier Flexible Retrieval: expedited (1-5mins), standard (3-5hrs), bulk(5-12hrs), min 90 day storage
    - Glacier Deep Archive: for long term storage; standard 12hr, bulk 48hrs. 180 day min storage
    - Intelligent-tiering: small monthly auto-tiering fee. Frequent access tier, infrequent, archive instant, archive acess, deep archive




### Personal notes
