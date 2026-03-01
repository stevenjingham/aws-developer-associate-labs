# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: Integration and Messaging - SQS, SNS, Kenisis
> 
> Date: February 2026 

---

## Purpose
AWS provides middleware services to enable communication between distributed applications (AWS or external).

Two communication patterns:

- **Synchronous (App → App)**
    - Tight coupling
    - Sensitive to traffic spikes
    - Less resilient

- **Asynchronous (App → Queue/Topic → App)**
    - Decoupled architecture
    - More resilient to spikes
    - Improves scalability and reliability

Main services:
- **Amazon SQS** – Queue-based messaging
- **Amazon SNS** – Pub/Sub notifications
- **Kinesis Data Streams** – Real-time streaming
- **Kinesis Data Firehose** – Streaming delivery to destinations


## Key Concepts
### Amazon SQS (Standard)

Fully managed message queue service.

#### Core Characteristics

- Unlimited throughput
- Max message size: 1024 KB
- Message retention: up to 14 days
- At-least-once delivery (duplicates possible)
- Best-effort ordering
- Low latency

#### Producer Flow

- Use `SendMessage`
- Message remains until deleted by consumer

#### Consumer Flow

- Poll using `ReceiveMessage`
- Up to 10 messages per request
- Must call `DeleteMessage`
- Duplicate processing possible

#### Scaling

- Auto Scaling Group can scale using queue length

#### Security

- HTTPS in-flight encryption
- KMS at-rest encryption
- Client-side encryption
- IAM policies or SQS access policies

---


#### Queue Access Policy

Resource-based policy for:
- Cross-account access
- Allowing S3 event notifications
- Controlling send/receive permissions

---

#### Visibility Timeout

- Default: 30 seconds
- Message becomes invisible after polling
- Reappears if not deleted

Use `ChangeMessageVisibility` to extend time.

Design considerations:
- Too short → duplicates
- Too long → slow failure recovery

---

#### Dead Letter Queue (DLQ)

- Moves messages after max receive count exceeded
- Used for debugging failed messages
- FIFO must use FIFO DLQ
- Standard must use Standard DLQ
- Messages can be redriven to source queue

---

#### Delay Queue

- Default delay: 0 seconds
- Max delay: 15 minutes
- Can configure per queue or per message (`DelaySeconds`)

---

#### Long Polling

- Wait 1–20 seconds for messages
- Reduces empty responses
- Fewer API calls
- Lower cost
- Preferred over short polling

---

#### Extended Client

- Java library
- Stores large payloads in S3
- Sends metadata to SQS

---

#### Important APIs

- `CreateQueue`
- `DeleteQueue`
- `PurgeQueue`
- `SendMessage`
- `ReceiveMessage`
- `DeleteMessage`
- `ChangeMessageVisibility`
- Batch APIs for cost reduction

---

### Amazon SQS FIFO

First-In-First-Out queue.

#### Characteristics

- Strict ordering
- Limited throughput
- Exactly-once processing (via deduplication)

#### Requirements

- Must provide `MessageGroupId`
- Deduplication:
    - Explicit Deduplication ID
    - Content-based (SHA-256 hash)

#### Message Groups

- Ordering guaranteed within group
- Different groups processed in parallel

---

### Amazon SNS

Publish/Subscribe messaging service.

#### Core Model

- Producer publishes to one SNS topic
- All subscribers receive all messages

#### Limits

- 100,000 topics
- 12.5 million subscriptions per topic

#### Supported Subscribers

- Email / Email-JSON
- SMS
- HTTP / HTTPS
- SQS
- Lambda
- Firehose

#### Characteristics

- Push-based delivery
- No message persistence
- IAM or SNS access policies
- KMS encryption supported
- Can be FIFO (only connects to SQS FIFO)

---

### SNS + SQS Fan-Out Pattern

- Publish once to SNS
- Fan out to multiple SQS queues
- Fully decoupled
- No data loss
- Easy to add new consumers
- Supports cross-region delivery

#### Filter Policies

- Configure filtering between SNS topic and SQS subscription
- Example: Only send cancelled orders to specific queue

---

### Kinesis Data Streams

Real-time streaming data ingestion service.

#### Use Cases

- Logs
- Metrics
- Clickstreams
- IoT data

#### Characteristics

- Data retention: up to 365 days
- Supports replay
- Data ordered by Partition Key and timestamp
- KMS encryption supported

---

#### Capacity Modes

##### Provisioned

- Manually configure shards
- Each shard:
    - 1 MB/s write
    - 2 MB/s read
- Pay per shard
- Manual scaling

##### On-Demand

- Auto scaling
- No shard management
- Pay per stream hour

---

#### Consumer Model

- Use `PutRecord`
- Must describe stream
- Retrieve shard iterator
- Read from shards

---

### Kinesis Data Firehose

Fully managed delivery service.

#### Purpose

- Buffer streaming data
- Batch writes to:
    - S3
    - Redshift
    - Splunk
    - HTTP endpoints

#### Characteristics

- Serverless
- Auto scaling
- Near real-time
- Buffer based on size/time
- Supports Lambda transformation
- Supports CSV, JSON, Parquet, Avro, Raw, Binary

---

### Kinesis Streams vs Firehose

| Kinesis Data Streams | Kinesis Data Firehose |
|----------------------|-----------------------|
| Real-time streaming | Near real-time delivery |
| Producer + consumer model | Fully managed delivery |
| Manual or on-demand scaling | Auto scaling |
| Data retention up to 365 days | No data storage |
| Replay capability | No replay |

---

### Apache Flink

- Used for stream processing on AWS
- Works with Kinesis Data Streams
- Does NOT read from Firehose

---

---
## How It Works


## Code / Config

## Common Pitfalls

## Key exam notes
| Scenario | Likely Service |
|----------|----------------|
| Decouple applications | SQS |
| Pub/Sub notifications | SNS |
| Fan-out to multiple consumers | SNS + SQS |
| Ordered processing | SQS FIFO |
| Replay streaming data | Kinesis Data Streams |
| Deliver streaming data to S3 | Firehose |
| Failed message debugging | DLQ |
| Reduce empty polling | Long Polling |
---

## Detailed Notes

### Video notes
- AWS has middleware to allow communication between multiple applications (on AWS or external)
- There are 2x patterns of communication
  - Synchronous - app to app (can be problematic if sudden spikes of unexpected traffic)
  - Asynchronous - app to queue to app (decouples; so less problematic from spikes)



- **SQS (Simple Queue System): Standard Queue**
  - One or more **Producer** send messages into a queue
  - One or more **Consumer** will poll for messages, and delete once actioned
- SQS is a fully managed service
  - Supports decoupling fo application
  - Unlimited throughput and number of messages in queue
  - Short lived - message retention for max 14 days; otherwise lost
  - Low latency for publish and receive
  - Limitation of 1024kb of data
  - Could have duplicate messages (i.e. at least once delivery; may get duplicates)
  - Best effort ordering (can be out of order)
- Producing messages
  - produced by producer using SDK (SendMessage API)
  - message persisted in SQS until **consumer deletes**
- Consuming messages
  - could be running EC2; on prem; or Lambda
  - polls for messages; and can receive up to 10x per request
  - once processed, the consumer will delete message using DeleteMessageAPI
    - If multiple consumers, some may receive request as a repeat if it was not processed & deleted by first consumer quickly enough
- SQS with Auto-Scaling Group can be done using the SQS queue length to support the scale of the ASG. 
- Encryption can be:
  - In flight using HTTPS API
  - At rest unsing KMS keys
  - Client side 
- Can have IAM access controls, or SQS Access Policies. 


- SQS Queue Access Policy
  - Added to SQS Queue to manage access policy
  - Use cases could be 
    - **Cross Account Access** (e.g. allow access for a different account); 
    - **Publish S3 event notification** to SQS Queue
  - Defines what services or accounts can send / receive messages into an SQS 


- SQS Message Visibility Timeout
  - After a message is polled, it becomes **invisible to other consumers**
  - After a timeout time (default 30s), it becomes visible again to other consumers, unless deleted by the original consumer
  - Potentially it could allow messages to be processed twice
    - However, if a consumer is processing the message, it can call **Change Message Visibility** API to get more time
  - So, the timeout time should be considered when being set. 
    - If too low, may get duplicate reads
    - If too high, in case of consumer crash it will take long time to process message


- Dead Letter Queue
  - If message is read many times, but not deleted from a queue - there may be an error with message
  - Can set a threshold for **number of reads** before a message is set to Dead Letter Queue
    - Having the message in DLQ removes from main queue, allowing for other processing/debugging to occur on it
  - DLQ of FIFO queue must be FIFO. DLQ of standard much be standard.
  - Retention time in DQL should be set to allow time to process message
- Once reviewed in DLQ, we can **redrive to source SQS**
  - e.g. if we fix consumers to allow to process, or fixed ticket


- SQS Delay Queue
  - Default delay in messages is 0s; but can set up to 15min delay
  - Can set a default on the queue; or when sending message
    - Can override default using **DelaySeconds** param


- Other SQS topics:
  - Long Polling
    - Option for consumer to **wait** when polling in case the queue does not have enough messages
    - Can be 1-20s
    - Means less API calls by consumer, and lower latency in messages. Preffereable to short polling
    - Can be default of queue, or set when consumer makes call
  - Extended Client
    - Java Library to allow large messages
    - Producers sends large message to S3; small metadata message sent to SQS queue.
  - Other APIs to know:
    - CreateQueue, DeleteQueue
    - PurgeQueue
    - SendMessage (DelaySeconds), RecieveMessages, DeleteMessage
    - MaxNumberOfMessages (1-10)
    - RecieveMessagesWaitTimeSeconds - for long polling
    - ChangeMessageVisibility - change message timeout
  - Batch APUs to reduce calls and costs:
    - SendMessage, DeleteMessage, ChangeMessageVisibility



- SQS FIFO Queue
  - First In First Out
  - Means messages are in correct order
  - Limits the throughput of messages
  - Can do exactly-one send capability (by reomving duplicates using Deduplication ID)
  - Must send a **Message Group ID** to allow groups of messages to be ordered
  - Deduplication
    - If queue see's same message, the message will be rejected.
    - Can do by:
      - De-duplication ID, or
      - Content based (SHA-256 hash of message body)
  - MessageGroup
    - Allows messages in group to be sorted
    - ALlows consumers to get specific messageGroups. Uses 1x queue that is ordered for multiple consumers
      - Messages in different message groups are not ordered



- Amazon SNS (Simple Notification Service)
  - Publish and Subscribe
  - Event producer **sends message to one SNS Topic**
  - Receivers subscribe to topics to get notifications
    - Each receiver will get **all messages** for the topic
    - Can have 12.5M subscriptions per topic
    - 100,000 topics limit
    - Receivers could be Emails, EmailJSON, SMS, HTTP, HTTPS Endpoints, SQS, Lambda, FireHose
  - SNS could receive data from lots of AWS providers
- Encryption can be:
  - In flight using HTTPS API
  - At rest unsing KMS keys
  - Client side
- Data is not persisted
- Can have IAM access controls, or SNS Access Policies.
- SNS can be FIFO too. Again it limits throughput. Only SQS FIFO can connect. 



- SNS and SQS **Fan Out Pattern**
  - Allows push once to SNS; and fan out to lots of SQS Queues
  - Fully decoupled, no data loss
  - Allows easy addition of SQS subscribers to a service creating messages
  - Cross-region delivery
- Could allow good use for S3 events (where only one message is created)
- Can also send to Firehose etc. 
- Fan out pattern can be good for filtering policy (e.g. some SQS queue might just want cancelled orders)
  - Can set a filter policy between SNS topic and SQS receiver


- Kinesis Data Streams
  - Services to collect and store stremaing data **in real time**
    - e.g. metrics and logs; or click streams
  - Real time data sent by producers to Kinesis Data Stream to be used by consumers
    - Consumers could be Apps, Lambda, Data Firehose, ApacheFlink
- Can be on stream for 365 days, allows replay by consumers, cannot be deleted
- Data ordering by datetime and Partition ID
- At rest KMS encryption and in flight HTTPS encryptions
- There are 2x **capacity modes**
  1) Provisioned mode
     - Choose number of shards to stream data; more shards allows data stream
     - Each shard is 1MB/s and 2MBs out
     - Scale manually to increase shards
     - Pay per shard 
  2) On-demand
     - No need to provision capacity
     - Scales automatically 
     - Pay per stream per hour
- "put-record" in SDK
- Consumers must describe the stream (e.g. give name) to allow to get data
  - Need to give shard; get shard iterator to keep getting the data



- Amazon Data Firehose
  - Producers send data (e.g. apps, kinesis) into firehose
  - The firehose has a buffer to store stream, then does **batch writes** to destinations
    - e.g. S3 , redshift, splunk etc; or HTTP endpoints
  - Can also send stream to Lambda first to do a transform of data
- It is fully managed service, auto-scaling, serverless
- Near real time, with buffering capability based on size and time
- Supports CSV, JSON Parquet Avro, Raw, Binary


- Kinesis || Firehose
  - Steaming data || Load streaming data to S3/etc
  - Producer and consumer || Fully managed
  - Real time || Near real time
  - Provisioned/On-demand || Auto scaling 
  - Data storage 365 days || No adata store
  - Relay capability || No relay


- Apache Flink allows processing of data streams on AWS
  - Does not read from firehose

### Personal notes
















