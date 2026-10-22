# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 22 - Serverless Dynamo DB
> 
> Date: October 2026

---

## Purpose
Amazon DynamoDB is a fully managed, serverless NoSQL database designed for low-latency access and horizontal scaling. AWS manages the underlying infrastructure, availability and scaling capabilities.

Typical use cases include user profiles, shopping carts, gaming data, session state, event-driven applications and high-throughput key-value/document workloads.
Relational databases vs. NoSQL
Traditional relational databases, such as database engines available through Amazon RDS, use tables, rows, columns, relationships and SQL. They can scale vertically by increasing instance capacity. Read replicas can scale some read workloads, but horizontal write scaling often requires additional design.

NoSQL databases use non-relational data models. DynamoDB is a key-value and document database designed for distributed, horizontal scaling. It does not support arbitrary relational joins in the same way as a relational database, so data models are often designed around the application's access patterns.

NoSQL does not mean there is no schema or that every NoSQL database works identically.

## Key Concepts
Tables, items and attributes
Table
├── Item (similar to a row)
│    ├── Attribute
│    └── Attribute
└── Item
├── Attribute
└── Attribute
- Table: Collection of items.
- Item: Collection of attributes, broadly similar to a row.
- Attribute: Named value, broadly similar to a column value.
  Items in one table can have different attributes, allowing a flexible schema. An absent attribute is not necessarily the same as an explicitly stored NULL.
  Data types
- Scalar: String (S), Number (N), Binary (B), Boolean (BOOL), Null (NULL).
- Document: List (L) and Map (M).
- Sets: String set (SS), Number set (NS) and Binary set (BS). Sets contain unique values of the relevant supported type.
  Primary keys
  Every table has a primary key. Key design influences uniqueness, query patterns and distribution.

Partition key only: The partition key uniquely identifies each item.

Partition key + sort key: The combination uniquely identifies an item. Either attribute may appear in multiple items. Items sharing a partition key can be queried together and organized by sort key.

Example:
UserId (partition key)	GameId (sort key)	Score
user-101	game-1	120
user-101	game-2	250
user-102	game-1	180


Choose partition keys with good distribution. A very popular key can become a hot key.
Read consistency
- Eventually consistent read (default for many read operations): A recent write may not be visible immediately to every read. It consumes less read capacity than a strongly consistent read of the same item.
- Strongly consistent read: Reflects all successful writes completed before the read. It costs twice the read capacity of an eventually consistent read for the same item size.
  Strong reads are supported for table and LSI reads where the operation allows them. GSI reads are eventually consistent only.
  Read and write capacity modes
  Provisioned capacity
- Specify read capacity units (RCUs) and write capacity units (WCUs).
- Plan and monitor demand; Auto Scaling can adjust capacity within configured bounds.
- You pay for provisioned capacity even if you do not use it all.
- Exceeding available capacity can cause throttling.
  On-demand capacity
- Capacity adapts to workload without specifying RCUs/WCUs in advance.
- Useful for unpredictable or variable traffic.
- Charged based on read request units (RRUs) and write request units (WRUs).
- May cost more for steady, predictable traffic.
- Not literally unlimited: service quotas and workload/traffic constraints still apply.
  Capacity-mode switching is subject to AWS restrictions; do not assume it can be switched repeatedly without limits.
  RCU and WCU basics
  For provisioned capacity:
- 1 RCU supports one strongly consistent read per second, or two eventually consistent reads per second, for an item up to 4 KB.
- 1 WCU supports one standard write per second for an item up to 1 KB.
  Capacity rounds up to size blocks: reads use 4 KB blocks and writes use 1 KB blocks. For example, a 6 KB item read consumes the capacity for two 4 KB blocks. Transactional operations consume more capacity than standard operations.
  Partitions, hot keys and throttling
  DynamoDB distributes data across internal partitions. A hashing process based on the partition key determines item placement. Traffic needs to be distributed across keys and partitions.

Possible causes of throttling:
- Hot key: one partition-key value receives disproportionate traffic.
- Hot partition: traffic is unevenly concentrated.
- Large items, which consume more capacity per request.
- Traffic spikes or insufficient provisioned capacity.
  Mitigations:
- Retry with exponential backoff.
- Choose well-distributed, high-cardinality partition keys.
- Consider write sharding for very hot logical keys.
- Review capacity settings and item sizes.
  Basic operations
  Writes
- PutItem: Creates an item or fully replaces the existing item with the same primary key.
- UpdateItem: Updates selected attributes; can create an item if it does not exist unless conditions prevent it.
- Conditional writes: Proceed only when a condition expression is true.
  Reads
- GetItem: Gets one item using its complete primary key.
- Query: Gets items for a partition-key value, optionally constrained by sort-key conditions.
- Scan: Evaluates items across a table or index; potentially expensive.
  Deletes
- DeleteItem: Deletes an item by key.
- DeleteTable: Deletes a table and its data.
  Query vs. Scan
  Prefer Query when you know the partition-key value and the access pattern fits. Use sort-key conditions to narrow results.

Scan evaluates items across a table or index. A FilterExpression is applied after items are evaluated, so it does not necessarily reduce read capacity consumed. Avoid frequent large scans when a key-based query or index can meet the requirement.
Batch operations
- BatchWriteItem: Up to 25 put/delete operations per request. It does not support UpdateItem or per-item conditional writes.
- BatchGetItem: Up to 100 items per request, subject to response-size limits.
- A batch may return unprocessed items. Retry those items, preferably with exponential backoff.
- Batch operations are not all-or-nothing transactions.
  Conditional writes and optimistic locking
  A condition expression allows a write only when a condition is true. This can prevent accidental overwrites and implement optimistic locking.

Example: an item has Version = 7. A client updates it only if the version is still 7, and the update changes the version to 8. If another client changed it first, the condition fails and the stale update is rejected.

Useful condition functions include:
- attribute_exists(path)
- attribute_not_exists(path)
- attribute_type(path, type)
- contains(path, operand)
- begins_with(path, prefix) for supported attribute types
- size(path)
  A common create-only condition is attribute_not_exists(partition_key), preventing replacement of an existing item with the same key.
  Secondary indexes
  Indexes support alternate query patterns.

Local Secondary Index (LSI)
- Same partition key as the base table, different sort key.
- Must be defined when the table is created.
- Up to five LSIs per table.
- Uses the base table's provisioned capacity rather than separately provisioned index capacity.
- Supports strongly consistent reads where applicable.
  Global Secondary Index (GSI)
- Has its own partition key and optionally its own sort key.
- Enables query patterns using different key attributes.
- Can be added after table creation.
- Has separate capacity settings when using provisioned mode.
- Supports eventually consistent reads only.
  Throttling: A GSI write-capacity bottleneck can affect base-table writes because writes must also maintain the index. Monitor index capacity and throttling. LSIs have no separate provisioned RCU/WCU, but their use still contributes to table-level capacity pressure.
  PartiQL
  PartiQL is a SQL-compatible query language for supported DynamoDB operations. It offers statements such as SELECT, INSERT, UPDATE and DELETE, plus batch execution in supported interfaces.

PartiQL does not turn DynamoDB into a relational database or remove the need to design for DynamoDB's keys and access patterns.
DAX vs. ElastiCache
DynamoDB Accelerator (DAX) is a managed in-memory cache designed specifically for DynamoDB. It can provide microsecond-level latency for cached reads and help reduce repeated read load, including hot-key workloads. Its item cache TTL is configurable; five minutes is a commonly taught default, but check the current default for your configuration.

ElastiCache is a general-purpose in-memory cache (for example, Redis OSS or Memcached). It is useful for application-managed cache structures, computed results, session data and flexible caching patterns.

Use DAX when the access pattern maps to DynamoDB operations; consider ElastiCache for more general application caching.
DynamoDB Streams
Streams record item-level modifications in the order they occur for each item. A stream view can contain:
- Keys only
- New image
- Old image
- Both old and new images
  Use cases include triggering Lambda, reacting to changes in near real time, and propagating data to other systems.

Records are retained for up to 24 hours. Streams are not a permanent audit log and do not retroactively create records for changes before the stream was enabled.
Time to Live (TTL)
TTL expires items based on a configured timestamp attribute.
- The attribute must be a Number containing a Unix epoch timestamp in seconds.
- Deletion is asynchronous and may occur days after expiry.
- TTL deletion does not consume write capacity in the same way as a client-issued delete.
- If an item must be considered expired at an exact time, filter it in application logic rather than relying on immediate deletion.
  CLI pagination and expressions
- --projection-expression: Attributes to return.
- --filter-expression: Filters evaluated items before they are returned.
- --page-size: Page size for underlying calls in supported AWS CLI commands; can affect call count.
- --max-items: Maximum number of items returned by the CLI command.
- --starting-token: Token to continue CLI pagination.
  Do not confuse CLI pagination with service-level Limit and LastEvaluatedKey. Filtering does not mean DynamoDB avoids evaluating the items.
  Transactions
  DynamoDB transactions provide ACID guarantees for coordinated operations across multiple items/tables within supported limits.
- TransactGetItems
- TransactWriteItems
  Transactions incur additional processing and capacity costs. A common exam rule is that transactional reads and writes consume approximately twice the capacity of equivalent standard operations for the same item size.

Use transactions when multiple changes must succeed or fail together. A conditional write is often sufficient for a single-item condition.
Session state
Possible storage choices:
- DynamoDB: Durable, managed, scalable key-value state.
- ElastiCache: In-memory, low-latency state.
- EFS: Shared network filesystem.
- EBS: Block storage attached to EC2; not a general shared serverless session store.
- S3: Object storage, usually not ideal for tiny, frequently updated session values.
  Choose based on latency, durability, sharing, data size and operational needs.
  Write sharding
  Write sharding spreads traffic for one hot logical key across multiple partition-key values.
  CustomerA#0
  CustomerA#1
  CustomerA#2
  CustomerA#3
  The application must know how to choose shards on writes and query/aggregate across them on reads. Random suffixes can distribute writes but make reads harder; deterministic suffixes may be easier to locate if the access pattern supports them.
  Write patterns
- Concurrent writes: Multiple clients write the same item; updates may overwrite one another if unprotected.
- Conditional writes: Write only if a condition is true.
- Atomic updates: For example, atomically increment a counter using an update expression.
- Batch writes: Multiple puts/deletes in one request, but not an all-or-nothing transaction.
  DynamoDB with S3
  Store large objects in S3 and metadata/object keys in DynamoDB.
  Large file → S3
  DynamoDB item → S3 key + searchable metadata
  This keeps DynamoDB items smaller while allowing the application to find the corresponding object.
  Cleanup, copying and backup
- Cleanup: Scan and delete individual items when needed, or delete the whole table if all its data is disposable. Whole-table deletion has operational consequences.
- Copy/backup: Consider AWS Backup, DynamoDB export to S3, AWS Glue, or application code. The right choice depends on whether you need a backup, export, migration, transformation or replication.
  Security and availability
- IAM policies control access to DynamoDB actions and resources.
- Data is encrypted at rest; encryption options can be configured.
- Backup and restore capabilities are available.
- Global tables support multi-Region replication.
- DynamoDB Local can support local development/testing.
  IAM, encryption, backups and multi-Region replication are separate features; one does not automatically configure the others.
## How It Works
Capacity calculation examples
For an item up to 4 KB:
- 1 RCU = one strongly consistent read per second.
- 1 RCU = two eventually consistent reads per second.
  A 6 KB item requires two 4 KB read blocks. For standard writes, 1 WCU supports one write per second up to 1 KB; a 1.5 KB item requires two 1 KB write units. Transactional operations consume additional capacity.

## Code / Config

## Common Pitfalls
NoSQL means no schema: DynamoDB has a flexible item schema, but tables still require a primary-key design and supported attribute types.
Poor partition key: Low-cardinality or very popular keys can create hot partitions and throttling.
Scan instead of Query: Query is usually better when the partition-key value is known.
FilterExpression saves read capacity: A filter is applied after evaluation and may not reduce capacity consumed.
Strong and eventual consistency are the same: Eventual reads can briefly return older data; strong reads cost more. GSIs only support eventual reads.
On-demand is unlimited: Quotas and traffic constraints still apply.
Ignoring size rounding: Capacity is calculated in blocks (4 KB reads, 1 KB writes).
Batch operations are transactions: They can partially succeed and return unprocessed items.
BatchWriteItem supports updates/conditions: It supports puts and deletes, not UpdateItem or per-item conditional writes.
LSI and GSI are interchangeable: Their key schemas, creation timing, capacity and consistency differ.
TTL deletes exactly at expiry: Deletion is asynchronous; enforce exact expiry in application logic.
Streams are permanent history: Retention is up to 24 hours.
DAX and ElastiCache are the same: DAX is DynamoDB-focused; ElastiCache is general-purpose.
Conditional writes retry automatically: A failed condition requires application handling.
Batch writes are atomic: They are not all-or-nothing.
S3 and DynamoDB are interchangeable: A common pattern is large files in S3, searchable metadata in DynamoDB.
| Feature | LSI | GSI |
|---|---|---|
| Partition key | Same as table | Can be different |
| Sort key | Different from table's sort key | Optional; can be different |
| Add after table creation? | No | Yes |
| Separate provisioned capacity? | No | Yes |
| Strongly consistent reads? | Supported | No |
## Key exam notes
Primary keys
Partition key only
→ uniquely identifies each item
Partition key + sort key
→ combination must be unique
→ same partition-key items can be queried together
Capacity
1 RCU
→ 1 strongly consistent read/sec up to 4 KB
→ 2 eventually consistent reads/sec up to 4 KB
1 WCU
→ 1 standard write/sec up to 1 KB
Remember size rounding and additional capacity for transactional operations.
Capacity modes
Provisioned
→ specify RCU/WCU
→ plan and monitor
→ pay for provisioned capacity
On-demand
→ adapts to workload
→ pay per request unit
→ useful for unpredictable traffic
Operations
GetItem → one item by complete primary key
Query   → partition key + optional sort-key condition
Scan    → evaluates across table/index; can be expensive
PutItem → create or replace item
UpdateItem → update attributes; can create if absent unless prevented
Indexes
LSI
→ same partition key, different sort key
→ create with table
→ maximum 5
GSI
→ alternate key schema
→ can be added later
→ separate capacity in provisioned mode
→ eventually consistent reads only
Reliability and performance
Hot key / hot partition → throttling risk
Throttling → exponential backoff, review capacity, distribute keys
Optimistic locking → version attribute + conditional update
Special features
DAX → DynamoDB-focused in-memory cache
Streams → item-level changes, up to 24-hour retention
TTL → Unix epoch seconds; asynchronous deletion
Transactions → ACID operations across multiple items/tables
S3 + DynamoDB → large object in S3; key/metadata in DynamoDB
---

## Detailed Notes

### Video notes


### Personal notes

- Traditional architecture has a database layer. e.g. RDS. 
  - This is limited to vertical scaling, but horizontal scaling is limited to reading capability/read replicas
- No SQL databases are non-relational
  - They are distributed. 
  - Limited support to query joins, aggregation etc.
  - All data is needed in one row. 
  - NoSQL can be scaled horizontally.
  - Eg. MongoDB, or DynamoDB
- DynamoDB
  - Fully managed, high availability. 
  - Millions of requests per second, trillions of rows
  - Fast and consistent
  - Integrated with IAM
  - Low cost, and can auto scale
  - Basics
    - Is made of tables
    - Each table has a **Primary Key**
    - Each table has items == rows
    - Each item has attributes; these can be null so new added over time
    - Data types:
      - Scalar (eg String)
      - Document - list or maps
      - Sets
  - Primary Key choice can be
    - **Partition key** - unique for each item. Must be "diverse" e.g. userId
    - **Partition key + Sort Key** - combination must be unique, eg. userId and game id
      - Data is grouped by partition key



- Read and Write Capacity Modes
  - Control tables capacity
  - Reads can be Strongly Consistent Read or Eventually Consistent Read (default)
    - Writes will be done to one server, and needs to be replicated across other servers
    - Stale data could be read from ECR database
    - Consistent read ensures that after a write, the read is always correct. However this costs more RCU and has higher latency
  - RCU = Read Capacity Units
    - 1RCU = 1x SCR per second; or 2x ECR per second; for up to 4kB size
  - WCU = Write Capacity Units
    - 1WCU = 1 write per second for item upto 1Kb. (larger items need more)
  - Can do:
    1) Provisioned - specify number of r/w per second
       - Need to plan before.
       - Pay for provisioned, even if dont use
       - Need to determine RCU and WCU needed. 
       - Burst capacity can be used for temporary exceeded. If exceed that then get ProvisionedThroughPutExceededException
    2) On-Demand - auto scales with workload
       - No capacity planning needed, good for unpredictable workloads or traffic 
       - Pay for what use, but more expensive (~2.5x)
       - Unlimited RCU/WCU
       - Charged on Read Request Units RRU, and Write Request Units WRU
  - Can switch between modes every 24hrs. 
- Partitions Internal
  - Data is stored in partitions. 
  - Partition keys go through hashing algo to know which partition the item is in. 
  - WCU and RCU are spread evenly over partitions
    - Throttling - if we exceed the RCU or WCU get exception as above.
    - This may occur due to:
      - Hot keys: one key is read too many times (e.g. popular item)
      - Hot partition
      - Very large items
    - Solutions can be:
      - **Exponential backoff** 
      - Distribute partition keys


- Basic Operations
  - Writing Data
    - PutItem (creates new item, or fully replaces)
    - UpdateItem
    - ConditionalWrite (only if condition is met)
  - Reading Data
    - GetItem (based on primary key)
      - Can be eventually or strongly consistent read
    - Query (can use KeyConditionExpression, FilterExpression)
      - Can be multiple items
    - Scan (reads the entire table)
      - Not very efficient, but used for exporting all table
  - Deleting 
    - DeleteItem
    - DeleteTable
  - Batch Operations
    - Lowers the number of calls, saving latency
    - Part of batch can fail; can then retry the batch
    - BatchWriteItem
      - Does 25 writes, but can only do Put or Delete
    - BatchGetItem
      - Can recieve upto 100 items
  - PartiQL
    - SQL compatible query language to support API calls to DynamoDB

- Conditional Writes
  - Condition expressions could be:
    - Attribute_exists
    - non_exists
    - attribute_type
    - contains
    - begins_with
    - size
  - Very useful to see if attribute_not_exists(partition_key), so we do not overwrite an item

- Index's - eg. supports more queries, based on table data
  - There are 2:
    1) **Local Secondary Index** (LSI) - alternative sort key for table
      - Consists of one scalar attribute
      - Up to 5x LSI per table
      - Must be defined **at creation time**
    2) **Global Seconardy Index** (GSI) - alternative primary key
      - Speeds up queries on non-key attributes
      - Consists of scalar attributes
      - Must provision RCU and WCU for index
      - Can be added or modified after table creation
  - Indexes and throttling
    - GSI throttles will throttle the main table (so be careful!)
    - LSI uses WCU and RCU of main table, so no additional considerations


- DynamoDB PartiQL
  - SQL syntax to manipulate table
  - Supports INSERT, UPDATE, SELECT, DELETE and batch processes


- Optimistic Locking
  - Feature called **Conditional Writes**
    - strategy to ensure an item has not changed before updating/deleting it
  - Each item has an attribute that acts as version number
    - Clients send request to update only is version is equal to xx value
    - If it is not, then client will get error and wont update table. 


- DynamoDB Accelerator (DAX
  - Fully managed, highly available in memory cache for DynamoDB
  - Supports microsecond latency and solves the "Hot Key" problem
  - 5min TTL defualt
  - It is secure etc. 
- Difference vs. ElastiCache
  - DAX is for individual objects in cache
  - ElastiCache would be better for any result of processing of data in table


- DynamoDB Streams
  - Ordered list of item-level modifications (eg inserts) in table
  - Can be sent to KDS, Lambda
  - Retentiion for up to 24hrs
  - Use case - react to changes in real time. 
  - Can define what is in the stream: eg. KEYs, old image, new image
  - Records are **not** retrospective once created


- Dynamo TTL
  - Automatically delete items after an expiry timestamp
  - Does not use any WCU's
  - TTL attribute must be a Number type with "Unix Epoch timestamp" value
  - Expired items deleted few days after expiration (will need to filter if want 100% accuracy)
  - Deletion process scans and deletes the items


- DynamoDB CLI
  - --projection-expression - attributes to retrienve
  - --filter-expression - filer itesm before returning
  - --page-size - increase number of items, but is more calls behind the scenes. Default 1000 items
  - --max-items - max items
  - --starting-token - paging key

- Transactions
  - Provides ACID, by ensuring all operations work across muliptles items or tables
  - Can be ReadMoves or WriteModes. Additional over standard modes
  - Will use 2x WCU and RCU as the operation has prepare and commit 
  - Operations are
    - TransactGetItems
    - TransactWriteItems


- Session State
  - Can store the Sesssion State in cache
  - ElastiCache does this in memory, but DynamoDB is serverless and can scale
  - EFS can store in a network drive
  - EBS could be used for local caching
  - S3 is not meant for low size


- Write Sharding
  - Strategy to allow bettter distributiuon of items across partitions (e.g. if partition key is A or B)
    - Can add random suffix and calculated suffix to make the partition greater


- Write Types
  - Concurrect Writes - could overwrite
  - Conitional writes - look at the version value key
  - Atomic writes - both increase value etc
  - Batch writes - multiple items updated

- DynanoDB with S3
  - Can use metadata of S3 bucket to store the S3 item in table
  - Can then use small data in DB, to find S3 large item

- Operations 
  - Cleanup: Scan and delete, or Drop Table (cheaper)
  - Copy - use AWS Backup, AWS Glue, Or Write own code to read and write


- Security
  - Controlled by IAM
  - Encrypted
  - Backup and Restore available
  - Can be global multiregion
  - Can have local DB for testing