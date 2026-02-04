# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 13 -  Advanced S3 
> 
> Date: February 2026

---

## Purpose
Understand how Amazon S3 manages object storage, events, performance, and cost optimisation.


## Key Concepts


## How It Works
- Objects are stored in S3 buckets and accessed via object keys.  
- S3 automatically handles durability, replication, and scaling, while lifecycle rules and performance features optimise cost and access patterns.  
- Event notifications allow S3 to act as a source of events in event-driven architectures.



## Code / Config

## Common Pitfalls

## Key exam notes
### Storage Class Transitions & Lifecycle Rules
S3 allows objects to be automatically moved between different storage classes based on access patterns and age.  
Lifecycle rules help reduce cost by transitioning infrequently accessed objects to cheaper storage tiers or deleting them when they are no longer needed.

Lifecycle rules:
- Can transition objects (e.g. Standard → Glacier)
- Can expire (delete) objects automatically
- Can be applied using object prefixes (folders) or object tags
- Are evaluated asynchronously by S3

S3 Analytics provides reports that recommend lifecycle transitions based on usage patterns, but **does not move objects automatically**.

### Event-Driven Processing with S3
S3 can emit events when actions occur on objects (e.g. upload, delete, restore).  
These events allow S3 to integrate with serverless and distributed systems.

Event notifications can trigger:
- Lambda functions
- SNS topics
- SQS queues
- Amazon EventBridge

EventBridge provides enhanced capabilities such as advanced JSON filtering, multiple targets, and event replay.

### S3 Performance & Scalability
S3 is designed to scale automatically and support high request rates with low latency.

Key performance characteristics:
- Baseline latency of ~100–200ms
- Per-prefix request limits:
    - ~3,500 PUT/COPY/POST/DELETE requests per second
    - ~5,500 GET requests per second
- Unlimited prefixes per bucket

Proper object key design (using multiple prefixes) helps avoid performance bottlenecks.

### Large Object Transfers & Optimisation
S3 provides multiple mechanisms to improve upload and download performance for large objects.

- **Multipart Upload**
    - Recommended for objects >100MB
    - Required for objects >5GB
    - Uploads parts in parallel to improve speed and resiliency

- **S3 Transfer Acceleration**
    - Uses AWS edge locations to speed up uploads
    - Data travels over public internet to the edge, then AWS private network to S3

- **Byte-Range Fetches**
    - Retrieve specific byte ranges of an object
    - Enable parallel downloads
    - Useful for large files or partial reads (e.g. file headers)

### Metadata & Object Tagging
S3 objects can store additional information using metadata and tags.

- **Object Metadata**
    - Key-value pairs defined at upload time
    - Keys must use the `x-amz-meta-` prefix
    - Cannot be modified after upload

- **Object Tags**
    - Key-value pairs attached to objects
    - Used for access control, cost allocation, and lifecycle rules
    - Can be modified after upload

S3 does not support searching objects by metadata or tags directly; external indexing (e.g. DynamoDB) is required.

---

## Detailed Notes

### Video notes
- S3; moving classes
  - Can transtion obhects between S3 storage classes
  - Can be done manually, or can be automated using **lifecycle rules**
    - e.g. move to glacier after not using long time
  - Or can have **expiration actions**, so files are deleted if conditions met
  - Rules can be created for certain prefix (directory), or tags 
  - S3 Analytics, can give a report giving recommendation for storage to allow lifecycles rules to be defined


- S3 Event Notifications
  - events are when an object created, removed, restored etc. Event notification actions something to a target. 
  - Target destinations can be Lambda function, SNS topic or SQS queue
  - e.g. use case - can generate thumbnails of images uploaded. when jpg file uploaded send it to different downstream services
    - Can filter based on object name e.g. .jpg
  - IAM Permissions is required to allow the S3 to reach the target
    - e.g. for Lambda neds a Lambda resource policy that allows S3 to connect
    - Amazon EventBridge enhances the notification as it gets all the events from S3, but can have:
      - Advanced filtering using JSON rules
      - Multiple destinations
      - More capabilities (e.g. archive, replay events)



- S3 Performance
  - Baseline perofmrnace, scales automatically for high requests with 100-200ms latency 
    - Can achieve 3500x PUT/COPY/POST/DELETE or 5500x GET requests per second, per prefix in a bucket
    - No limits to number of prefix's in bucket (e.g. prefix is folder)
  - Multi-part upload is recommended for files >100Mb and required for >5GB
    - Helps parallelise uploads for speeding up
  - S3 Transfer Acceleration
    - increase transfer speed by transferring to an AWS edge location, then to the S3 bucket in the target region
    - e.g. use public internet to edge location, then private aws infra to S3 location
  - S3 Byte-range fetches
    - Parallelise GETS by requesting specifc byte ranges, to speed up download and give resilence in case of failures
    - Can be done in parallel to speed up
    - Could also just request certain bytes e.g. header data of a file 


- Metadata and tags
  - User-defined object metadata
    - can set a key-value meta-data. Key must start with x-amz-meta- (e.g. x-amz-meta-photoLocation)
  - Tags are key-value pairs for objects in S3
    - Useful for fine-grainder permissions (e.g. access to objects with specific tags), or analytics
  - Can search or filter by metadata or tags
    - Would need to do this by putting data into DB table to allow to search

### Personal notes
