# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 21 Serverless Lambda
> 
> Date: April 2026

---

## Purpose
**AWS Lambda** is a **serverless, event-driven compute service** that allows you to run code without provisioning or managing servers.

You upload your function code, configure how it should run, and AWS handles provisioning compute infrastructure, execution environments, scaling, and infrastructure maintenance.

### Serverless does NOT mean "there are no servers"

There are still servers running your code. The important distinction is:

> **You don't provision or manage the servers. AWS does.**

Lambda is part of the broader AWS serverless ecosystem, alongside services such as DynamoDB, S3, API Gateway, SNS, SQS, EventBridge, Step Functions, Aurora Serverless and Fargate.

### When would you choose Lambda?

Lambda is particularly suitable when:

- Work is event-driven
- Workloads are intermittent
- You want automatic scaling
- You don't want to manage servers
- Functions are relatively short-lived
- You want to pay based on usage rather than keeping servers running

Example:

```text
S3 upload
   ↓
Lambda
   ↓
Generate image thumbnail
   ↓
Store thumbnail in S3
```


## Key Concepts

### 1. Lambda Functions

A Lambda function contains the code Lambda executes.

Important configuration includes:

- Runtime
- Memory
- Timeout
- Environment variables
- IAM execution role
- VPC configuration
- Layers
- Concurrency settings
- Triggers/event sources

Lambda supports runtimes such as Python, Node.js, Java, .NET, Go and Ruby. Lambda can also run compatible container images.

### 2. Event-Driven Architecture

Lambda is heavily based around events.

```text
API Gateway → Lambda
S3          → Lambda
SNS         → Lambda
EventBridge → Lambda
SQS         → Lambda
Kinesis     → Lambda
DynamoDB    → Lambda
```

There are three important invocation patterns:

#### Synchronous

The caller waits for Lambda to finish.

Examples:

- API Gateway
- ALB
- AWS SDK/CLI
- Direct invocation

#### Asynchronous

The event is accepted and Lambda processes it later.

Examples:

- S3
- SNS
- EventBridge

The caller does not wait for the Lambda result.

#### Event Source Mapping

Lambda polls the source.

Examples:

- SQS
- Kinesis Data Streams
- DynamoDB Streams

```text
SQS / Kinesis / DynamoDB Stream
             ↓
      Event Source Mapping
             ↓
           Lambda
```

### 3. Automatic Scaling

Lambda creates execution environments as demand increases.

**Concurrency** means approximately:

> The number of function invocations being processed simultaneously.

### 4. Lambda Execution Role

The Lambda execution role is an IAM role assumed by the function at runtime.

It controls what the Lambda function can access, such as:

- SQS
- Kinesis
- DynamoDB
- CloudWatch Logs
- Secrets Manager
- X-Ray

### 5. Resource-Based Policies

Resource-based policies control who or what can invoke/access the Lambda function.

Easy exam distinction:

| Permission | Question |
|---|---|
| Execution role | What can Lambda access? |
| Resource policy | Who can invoke/access Lambda? |

---

## How It Works

# AWS Lambda

## Purpose


### When would you choose Lambda?

Lambda is particularly suitable when:

- Work is event-driven
- Workloads are intermittent
- You want automatic scaling
- You don't want to manage servers
- Functions are relatively short-lived
- You want to pay based on usage rather than keeping servers running

Example:

```text
S3 upload
   ↓
Lambda
   ↓
Generate image thumbnail
   ↓
Store thumbnail in S3
```

---

## Key Concepts



## How It Works

### Synchronous Invocations

The caller waits for the result.

```text
Client
  ↓
API Gateway
  ↓
Lambda
  ↓
Response
```

The client/application generally handles errors and decides whether to retry.

### Lambda Integration with ALB

An Application Load Balancer can use Lambda as a target.

```text
Client
  ↓
ALB
  ↓
Lambda Target Group
  ↓
Lambda
```

ALB converts the HTTP request into the Lambda event format and converts the Lambda response back into an HTTP response.

The event can contain:

- HTTP method
- Path
- Query parameters
- Headers
- Request body

ALB also supports multi-value headers/query parameters.

### Asynchronous Invocations

The caller does not wait for Lambda to finish.

```text
S3 / SNS / EventBridge
        ↓
      Event
        ↓
      Lambda
```

AWS retries failed asynchronous invocations. Therefore, the function should be **idempotent**.

> Processing the same event multiple times should produce the same intended result as processing it once.

### Dead Letter Queues

A DLQ can capture events that Lambda could not successfully process after retries.

```text
Source
  ↓
Lambda
  ↓
Failure / retries exhausted
  ↓
DLQ
```

### Lambda and EventBridge

EventBridge can trigger Lambda using:

1. **Scheduled rules** using rate or cron expressions.
2. **Event patterns** reacting to events from AWS services/applications.

Example:

```text
EventBridge
    ↓
Every hour
    ↓
Lambda
```

### Lambda and S3

S3 can generate events for actions such as object creation and deletion.

```text
User uploads image
       ↓
S3
       ↓
Lambda
       ↓
Generate thumbnail
       ↓
S3
```

S3 can send notifications to Lambda, SNS or SQS.

Do not assume exactly-once processing. Lambda should tolerate duplicate events.

### Event Source Mapping

An event source mapping connects Lambda to sources that Lambda polls.

```text
SQS
 ↓
Event Source Mapping
 ↓
Lambda
```

#### SQS

Lambda polls SQS using long polling and processes messages in batches.

If processing fails, the message becomes available again after the **visibility timeout**.

A common AWS recommendation is for the SQS visibility timeout to be at least **6× the Lambda timeout**, with additional buffer where appropriate.

For FIFO queues, messages with the same `MessageGroupId` are processed in order. Different message groups can be processed concurrently.

#### Kinesis / DynamoDB Streams

Lambda reads stream records using iterators.

For streams:

- Records remain in the stream according to stream retention.
- Lambda can process multiple shards concurrently.
- Failed batches can be retried.
- Ordering and idempotency must be considered.

### Lambda Event and Context

A handler receives:

```text
handler(event, context)
```

**Event** = information about what happened / the invocation request.

**Context** = information about the Lambda execution environment.

Examples include:

- `aws_request_id`
- Function name
- Function version
- Remaining execution time

### Lambda Destinations

Destinations route invocation results to another service.

For asynchronous invocations, destinations can be configured for success or failure.

Possible destinations include:

- SQS
- SNS
- Lambda
- EventBridge

```text
Lambda
  │
  ├── Success → EventBridge
  │
  └── Failure → SQS
```

A DLQ primarily preserves failed events, whereas destinations can route success or failure results and provide more flexible workflows.

### Environment Variables

Environment variables separate configuration from code.

Example:

```text
ENVIRONMENT=production
DATABASE_HOST=my-db.example.com
LOG_LEVEL=INFO
```

For secrets such as passwords/API keys, Secrets Manager or Systems Manager Parameter Store is generally more appropriate.

### Monitoring and X-Ray

Lambda integrates with CloudWatch Logs and CloudWatch Metrics.

Useful Lambda metrics include:

- Invocations
- Errors
- Duration
- Throttles
- Concurrent executions

AWS X-Ray provides distributed tracing, helping identify latency across services such as API Gateway → Lambda → DynamoDB.

### Lambda@Edge and CloudFront Functions

Both can run logic close to users at CloudFront edge locations.

#### CloudFront Functions

Best for lightweight operations such as:

- Header manipulation
- URL rewrites
- Redirects
- Cache-key manipulation

#### Lambda@Edge

More capable and suitable for more complex edge logic.

| | CloudFront Functions | Lambda@Edge |
|---|---|---|
| Complexity | Lightweight | More complex |
| Typical use | Headers, redirects, cache keys | Advanced processing |
| Runtime | JavaScript | Node.js / Python |
| Best for | Simple edge logic | Complex edge logic |

### Lambda in a VPC

By default, Lambda is not connected to your VPC.

For private resources:

```text
Lambda
   ↓
Private subnet
   ↓
RDS
```

Configure:

- VPC
- Subnets
- Security groups

Lambda uses Elastic Network Interfaces (ENIs) to connect to the VPC.

**Important exam trap:** putting Lambda in a public subnet does not give it a public IP or automatic internet access.

For internet access:

```text
Lambda
 ↓
Private subnet
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

For private access to supported AWS services, use VPC endpoints where appropriate.

### Lambda Performance

#### Memory

Lambda memory can be configured from approximately **128 MB to 10,240 MB**.

Increasing memory also increases CPU.

Therefore:

> If an application is CPU-bound, increasing memory can improve performance.

#### Timeout

- Default: 3 seconds
- Maximum: 900 seconds / 15 minutes

### Execution Context

Lambda execution environments can be reused.

Expensive initialisation can therefore be placed outside the handler when safe:

```python
db = create_connection()

def handler(event, context):
    # Reuse db
    ...
```

However, reuse is never guaranteed. Do not rely on in-memory state for correctness.

### `/tmp` Storage

`/tmp` is local ephemeral storage.

Useful for:

- Temporary files
- Downloading S3 objects
- File processing
- Temporary output

It is not durable storage and is not shared between Lambda execution environments.

### Lambda Layers

Layers package reusable code/dependencies separately from the function.

Useful for:

- Shared libraries
- Custom runtimes
- Large dependencies
- Shared code

### Lambda + EFS

Lambda can mount Amazon EFS when configured appropriately in a VPC.

```text
Lambda A ──┐
Lambda B ──┼── EFS
Lambda C ──┘
```

EFS provides persistent, shared filesystem storage.

| Storage | Persistent? | Shared? | Typical use |
|---|---:|---:|---|
| `/tmp` | No | No | Temporary local files |
| Layer | Package/version based | Reusable | Dependencies/code |
| S3 | Yes | Yes | Objects/files |
| EFS | Yes | Yes | Shared filesystem |

### Lambda Concurrency and Throttling

Concurrency is the number of Lambda executions running simultaneously.

Your notes identify a default account-level concurrency quota of approximately **1,000**, subject to account/region quota and increase requests.

#### Reserved Concurrency

Reserved concurrency limits/reserves capacity for a specific function.

It can protect other functions from being starved during a traffic spike.

#### Throttling

When Lambda cannot accept an invocation because concurrency is exhausted, the invocation is throttled.

For synchronous invocation, this can result in an HTTP **429** response.

### Cold Starts

A cold start occurs when Lambda must create and initialise a new execution environment.

```text
Create environment
        ↓
Load runtime
        ↓
Load dependencies
        ↓
Run initialisation
        ↓
Execute handler
```

### Provisioned Concurrency

Provisioned Concurrency keeps a specified number of execution environments initialized and ready.

It reduces cold-start latency.

**Reserved concurrency** = limits/reserves concurrency.

**Provisioned concurrency** = pre-initializes execution environments.

### Lambda Dependencies

Dependencies can be packaged with the Lambda function or supplied through Layers.

Examples:

- Database clients
- Third-party libraries
- X-Ray SDK
- Utility libraries

### Lambda + CloudFormation

Lambda can be deployed using CloudFormation.

#### Inline

Use `Code.ZipFile` for small functions directly in the template.

#### S3

Store the deployment artifact in S3 and reference it from CloudFormation.

```text
Code
 ↓
S3
 ↓
CloudFormation
 ↓
Lambda
```

### Lambda Container Images

Lambda can run container images stored in Amazon ECR.

Container images can be up to **10 GB** and must be compatible with Lambda's execution model / Runtime API.

The Lambda Runtime Interface Emulator can be used for local testing.

### Lambda Versions and Aliases

#### `$LATEST`

`$LATEST` is mutable.

#### Published versions

Published versions are immutable.

```text
Lambda
 ├── $LATEST
 ├── Version 1
 ├── Version 2
 └── Version 3
```

#### Aliases

An alias is a mutable pointer to a version.

```text
prod → Version 3
dev  → Version 4
```

Aliases can support deployment/traffic shifting but cannot point to another alias.

### Lambda + CodeDeploy

CodeDeploy can automate traffic shifting between Lambda versions through aliases.

Strategies include:

- All-at-once
- Canary
- Linear

Pre-traffic and post-traffic hooks can be used for validation.

### Lambda Function URLs

A Lambda Function URL provides a dedicated HTTPS endpoint directly to Lambda.

```text
Client
  ↓
Lambda Function URL
  ↓
Lambda
```

It does not require API Gateway.

Authentication options include:

- `NONE`
- `AWS_IAM`

Function URLs also support CORS configuration.

---

## Code / Config

### Basic Lambda handler

```python
def handler(event, context):
    print(event)

    return {
        "statusCode": 200,
        "body": "Hello from Lambda"
    }
```

The exact response format depends on the invoking service.

### Environment variables

```text
ENVIRONMENT=production
DATABASE_HOST=my-db.example.com
LOG_LEVEL=INFO
```

### Timeout

```text
Timeout = 30 seconds
```

Remember:

```text
Default = 3 seconds
Maximum = 900 seconds
```

### Memory

```text
Memory = 1024 MB
```

More memory also provides more CPU.

### VPC configuration

Configure:

```text
VPC
 ├── Subnet A
 ├── Subnet B
 └── Security Group
```

### Reserved concurrency

Conceptually:

```text
ReservedConcurrency = 100
```

This limits maximum concurrent executions for the function to 100.

---

## Common Pitfalls

### 1. "Serverless means no servers"

❌ Incorrect.

AWS still uses servers. You simply don't provision or manage them.

### 2. Lambda is always synchronous

❌ Incorrect.

Lambda supports:

- Synchronous invocation
- Asynchronous invocation
- Event source mappings

### 3. SQS directly invokes Lambda

Be precise:

> SQS uses a **Lambda event source mapping**, and Lambda polls SQS.

### 4. Lambda in a public subnet has internet access

❌ Incorrect.

A public subnet does not automatically give Lambda internet connectivity or a public IP.

For outbound internet access, a typical architecture uses:

```text
Private subnet
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

### 5. Execution role and resource policy are the same

❌ Incorrect.

```text
Execution Role
→ What Lambda can access

Resource Policy
→ Who can invoke/access Lambda
```

### 6. `/tmp` is permanent storage

❌ Incorrect.

`/tmp` is ephemeral. Use S3/EFS/etc. for durable storage.

### 7. Warm environments are guaranteed

❌ Incorrect.

AWS can terminate an execution environment at any time.

### 8. Provisioned and reserved concurrency are the same

❌ Incorrect.

| Feature | Purpose |
|---|---|
| Reserved concurrency | Reserve/limit concurrency |
| Provisioned concurrency | Pre-initialize execution environments |

### 9. Lambda automatically makes processing idempotent

❌ Incorrect.

Retries and duplicate events can occur. Implement idempotency where required.

### 10. Lambda can run forever

❌ Incorrect.

Maximum execution time is **900 seconds / 15 minutes**.

For longer-running workloads consider services such as Step Functions, ECS/Fargate, EC2 or Batch depending on the use case.

### 11. `$LATEST` is immutable

❌ Incorrect.

`$LATEST` is mutable. Published numbered versions are immutable.

### 12. Alias = version

❌ Not exactly.

A version is immutable code/configuration. An alias is a mutable pointer to a version.

---

## Key exam notes

### ⭐ Invocation models

```text
Synchronous
API Gateway
ALB
SDK/CLI
     ↓
Lambda
     ↓
Response immediately
```

```text
Asynchronous
S3
SNS
EventBridge
     ↓
Lambda
     ↓
Retries
     ↓
DLQ / Destination
```

```text
Event Source Mapping
SQS
Kinesis
DynamoDB Streams
     ↓
Lambda polls source
```

### IAM

```text
Execution Role
= permissions Lambda has
```

```text
Resource-Based Policy
= permissions for services/accounts to invoke Lambda
```

### Storage

```text
/tmp
= ephemeral local storage
```

```text
Layers
= reusable dependencies/code
```

```text
S3
= durable object storage
```

```text
EFS
= durable shared filesystem
```

### Networking

```text
Lambda outside VPC by default
```

```text
Lambda in public subnet
≠
Internet access
```

For internet:

```text
Lambda
 ↓
Private subnet
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

For private AWS service access:

```text
Lambda
 ↓
VPC Endpoint
 ↓
AWS service
```

### Performance

```text
More RAM
   ↓
More CPU
   ↓
Potentially faster execution
```

Maximum execution:

```text
15 minutes / 900 seconds
```

### Concurrency

```text
Concurrency
= simultaneous executions
```

```text
Reserved concurrency
= limit/reserve capacity
```

```text
Provisioned concurrency
= pre-warm environments
```

```text
Concurrency exhausted
       ↓
Throttle
       ↓
Synchronous → 429
```

### Versions

```text
$LATEST
= mutable
```

```text
Published version
= immutable
```

```text
Alias
= mutable pointer to version
```

### Deployment

CodeDeploy can perform:

```text
All-at-once
Canary
Linear
```

using Lambda aliases.

### Limits worth memorising

| Lambda limit/configuration | Value |
|---|---:|
| Memory | 128 MB – 10,240 MB |
| Maximum execution time | 900 seconds / 15 minutes |
| Environment variables | 4 KB |
| `/tmp` storage | Up to 10 GB |
| Container image | Up to 10 GB |
| Default concurrency | 1,000/account/region (quota; can request increase) |

For deployment packages, remember the distinction between compressed upload/package limits and uncompressed extracted size; these limits vary by deployment method.

---

## Detailed Notes

### The Lambda mental model

For the exam, think:

> **Event → Invocation model → Execution environment → Function → Result**

Example:

```text
S3
 │
 │ event
 ▼
Lambda
 │
 │ creates/reuses execution environment
 ▼
Function
 │
 │ executes
 ▼
Result
```

The key question in Lambda exam scenarios is often:

> **Who invokes Lambda, and how?**

---

### The three invocation patterns

#### Pattern 1 — Direct synchronous

```text
Client
  │
  │ Request
  ▼
API Gateway
  │
  ▼
Lambda
  │
  │ Response
  ▼
API Gateway
  │
  ▼
Client
```

Use when the caller needs the result immediately.

#### Pattern 2 — Asynchronous

```text
S3
 │
 ▼
Lambda
 │
 ├── Success
 │
 └── Failure
       ↓
   DLQ/Destination
```

Use when the caller does not need to wait.

Key concern:

**Retries → idempotency.**

#### Pattern 3 — Polling/event source mapping

```text
SQS
 │
 │ Lambda polls
 ▼
Event Source Mapping
 │
 ▼
Lambda
```

Use this for:

- SQS
- Kinesis
- DynamoDB Streams

Key concerns:

- Batch size
- Polling
- Visibility timeout for SQS
- Ordering
- Stream shards
- Retries
- Duplicate processing
- Scaling

---

### Lambda scaling mental model

1 request:

```text
Request
  ↓
Lambda environment #1
```

10 simultaneous requests:

```text
Request ──► Environment #1
Request ──► Environment #2
Request ──► Environment #3
...
Request ──► Environment #10
```

Lambda can scale execution environments as demand increases, subject to concurrency quotas.

---

### Cold vs warm invocation

#### Cold

```text
Request
 ↓
Create environment
 ↓
Initialise runtime
 ↓
Load dependencies
 ↓
Initialisation code
 ↓
Handler
```

#### Warm

```text
Request
 ↓
Existing environment
 ↓
Handler
```

Expensive initialisation should generally happen outside the handler where it is safe to reuse.

However, the application must **never depend on the environment being reused**.

---

### A useful exam decision tree

#### 1. Does the caller need the response immediately?

**Yes → Synchronous**

Think:

- API Gateway
- ALB
- SDK

**No → Asynchronous**

Think:

- S3
- SNS
- EventBridge

#### 2. Is Lambda reading from a queue/stream?

If:

- SQS
- Kinesis
- DynamoDB Streams

think:

> **Event Source Mapping**

#### 3. Does Lambda need private resources?

If:

```text
Lambda → RDS/private resource
```

think:

> **VPC configuration**

If it needs internet from the VPC:

> **NAT Gateway**

If it needs private access to AWS services:

> **VPC Endpoint**

#### 4. Is the problem cold-start latency?

Think:

> **Provisioned Concurrency**

Not reserved concurrency.

#### 5. Is the problem one function consuming all concurrency?

Think:

> **Reserved Concurrency**

#### 6. Does the question mention deployment versions?

Think:

```text
Published version = immutable
Alias = mutable pointer
```

For gradual rollout:

> **CodeDeploy + Lambda alias**

#### 7. Does the question mention reusable dependencies?

Think:

> **Lambda Layers**

#### 8. Does Lambda need shared persistent filesystem storage?

Think:

> **EFS**

#### 9. Does Lambda need temporary local files?

Think:

> **`/tmp`**

#### 10. Does the question ask for a simple HTTP endpoint without API Gateway?

Think:

> **Lambda Function URL**

---

### Biggest Lambda exam traps

```text
Synchronous
→ caller waits
→ API Gateway / ALB / SDK
→ caller handles errors

Asynchronous
→ caller doesn't wait
→ S3 / SNS / EventBridge
→ automatic retries
→ idempotency important
→ DLQ / Destination

Event Source Mapping
→ Lambda polls
→ SQS / Kinesis / DynamoDB Streams
```

```text
Execution Role
→ what Lambda can access

Resource Policy
→ who can invoke Lambda
```

```text
Reserved Concurrency
→ limit/reserve capacity

Provisioned Concurrency
→ pre-warm environments
```

```text
$LATEST
→ mutable

Published version
→ immutable

Alias
→ mutable pointer to version
```

```text
/tmp
→ ephemeral

S3
→ durable object storage

EFS
→ durable shared filesystem

Layer
→ reusable dependencies/code
```

```text
Lambda + VPC
→ no automatic internet access

Public subnet
→ does NOT automatically give Lambda a public IP

Internet from VPC
→ NAT Gateway

Private AWS service access
→ VPC Endpoint
```

```text
More Lambda RAM
→ more CPU

Maximum execution
→ 15 minutes

Concurrency
→ simultaneous executions
```

---

## Detailed Notes

### Video notes

- Lambda is a popular service, allowing serverless design
  - Serverless allows engineers to not manages servers, can just deploy code/functions
  - Was FaaS - function as a service - but examples to anything managed (e.g. DB, messaging, storage)
  - Serverless does not mean no servers, just do not provision them
- AWS supports serverless:
  - Lambda
  - DynamoDB
  - Cognito
  - API Gateway
  - S3
  - SNS / SQS
  - Kinesis Firehose
  - Aurora Serverless
  - Step functions
  - Fargate


- Lambda Overview
  - Virtual servers are limited by RAM/CPU; continuously running; scaling
  - While Lambda runs functions, limited by time (short executions), run on demand and scaling is automated
    - Pricing is easy, based on request and compute time. Do not need EC2 running idle.
      - 1,000,000 free, then cheap per call or duraction
    - Integrated with lots of AWS suite (e.g. Gateway, S3, DynamoDB, etc)
    - Easy monitoring
    - Increasing RAM will also improve CPU and network
      - But pay more for increase in RAM
    - Supports lots of languages
    - Can run Lambda Container Image via Lambda Runtime API
  - Different triggers can run the lambda, e.g. lots of different event sources
  - Can debug via Lambda logs. 
  - Can set role execution policy (e.g. runtime before failing etc. )
  - Can set IAM roles to lambda. 


- Synchronous Invocations
  - Triggered by User CLI, SDK, API Gateway, ALB
  - Results return directly; error handling done by client side (e.g. retry)
  - Await results

- Lambda Integration with ALB
  - Synchronous allows client to call ALB, to invoke a Target Group containing the Lambda
  - ALB converts HTTP request to JSON; sends this into Lambda with request params etc. 
  - ALB converts the response from JSON to HTTP response. 
  - ALB can have multi-value headers; allowing multiple query params into the Lambda, via an array in JSON. 


- Asynchronous Invocations
  - e.g. from services such as S3, SNS, Cloudwatch events
  - Events are placed in event queue
    - If fail, have 3 tries in total (1min after, 2min after)
  - Need to ensure that the processing in idempotent
  - Can setup a Dead Letter Queue
    - Need to allow permissions on lambda to send to SNS queue. 


- Lambda and Cloudwatch events
  - Either
    1) CRON (event pattern) or Rate (timebased) EventBridge rule to trigger every hour for example
    2) CodePipeline change to react to change and trigger lambda


- Lambda and S3 events notifications
  - e.g. when object created, removed etc
  - Use case can get thumbnail of image of every image uploaded
  - S3 can send events to SNS or SQS, or directly to Lambda for async functions
    - Can setup DLQ on lambda function
  - Note - 2 quick events on unversioned file may just make one event. Events usually sent in seconds but can take minute.
  - Need to allow S3 permissions to trigger lambda



- Event source mapping
  - e.g. from a Kinesis Data Stream, SQS, or DynamoDB stream
  - Records from all of these need to be polled from source; and record is returned
  - Lambda function can then run synchronously. Event source mapping is internal to the Lambda
    - For streams, iterator from each shard to read (dont remove from stream). Can do multiple in parrallel for high load.  
      - Errors; entire batch reprocessed until timeout
      - Scaling - one per shard, can use parallelization 
    - For queue, iterator via long polling
      - Set visibility timeout to 6x lambda time
      - Supports FIFO or non-ordered. For FIFO, messages with same groupId processed in order. 
      - Can scale based on queue size for base queue. For FIFO, can scale to number of active message groups
      - Errors returned to queue, can make DLQ.
  - Lambda functions will need role permissions to read SQS queues and streams etc. 



- Lambda event and context objects
  - Lambda will receive the Event Object, but also gives context object
    - Event object is in JSON and contains information from the invoking service; lambda runtime converts the event to an object
    - Context object provides methods and properties that provide information about the function and runtime env. Passed to your function by lambda at runtime
      - e.g. aws_request_id, function_name


- Lambda Destinations
  - Can configure a send result to a **destination**
    - Async - can be for success or failed events (e.g. SQS, SNS, Lambda, EventBridge). Can use instead of DLQ.
    - Streams - Event source mapping for discarded event batches to SQS or SNS
  - Lambda needs permission to write to SQS, SNS etc. 


- Lambda Permissions - IAM Roles and Resource Policies
  - Grants Lambda functions permissions to access AWS services/resources
    - e.g. upload to Cloudwatch, Read Kinesis, Read DynamoDB stream, Read SQS, deploy lambda on VPC, upload data to X-Ray
    - When using an **event source mapping** Lambda will execution role to read event data 
  - Resource Based Policies
    - Give other accounts and AWS services to use Lambda
      - eg. give S3 access to Lambda


- Lambda Environment Variables
  - Key Value pairs in string form to allow function behaviour without updating code
  - Can also store secrets to be utilised. 


- Monitoring and X-Ray tracing
  - AWS Lambda Execution logs are stored in CloudWatch - assuming function has execution role in IAM to write to cloudwatch
  - Cloudwatch Metrics are displayed also
  - Can enable X-Ray deamon with policy, and have env variables to communicate with daemon
    - AWS_XRAY_DAEMON_ADDRESS, AWS_XRAY_CONTEXT_MISSING, _X_AMZN_TRACE_ID


- Lambda@Edge and Cloudfront functions
  - Can do edge functions to be close to users, and reduce latency. Runs at CloudFront distructions, and can customise CDN
  - CloudFront fuctions are light weight JavaScript, can run in sub-ms. Can do millions of RPS. e.g. cacheKey, or Header manipulation
  - Lambda@Edge can handle more complex, in NodeJS or Python. Can do 1000s of RPS. Can be 5-10s of logic. 


- Lambda in VPC
  - By default, Lambda is outside VPC, so cannot access resources in VPC.
  - Can define a VPC ID, Subnet and Security Groups in Lambda, which will setup a **Elastic Network Interface** to private VPC
    - AWSLambdaVPCAccessExecutionRole needed by VPC
  - Lambda function in VPC does not have internet access. 
  - Deploying a lambda function in a public subnet does not give it access to internet, or a public IP
    - Can deploy in private subnet, and give access to NAT Gateway
  - Can add VPC endpoints in VPC, to allow lambda in VPC to access items on AWS Cloud


- Lambda function performance
  - RAM 
    - from 128Mb to 10Gb. More RAM, more vCPU
    - At 1792Mb == 1 full vCPU
    - Above this, need multi-threading to benefit
    - **If application is CPU bound**, increase RAM
      - But it will cost more, so want to optimise the RAM value. 
    - Default timeout is 3s. Max is 900s
  - Execution context
    - is a temporary runtime environment that initialises external dependencies
    - e.g. db connections, http clients
    - maintained for some time 
    - next function invocation can re-use the context, to same initialisation time
    - includes /tmp directory
  - /tmp space
    - allows temporary files to be written
    - has 10Gb size maximum
    - can download and work on large files
    - Need to use KMS Data Keys to encrypt. 
  - Should do time heavy outside the lambada if possible (e.g. connect to DB, as then it does not need to do this on every invocation) 


- Lambda Layers
  - Allow customer runtimes (e.g. C++ or Rust) or all externalise dependencies
    - e.g. can reference different functions, or heavy files
    - This would also allow other lambda functions from getting the same sub-layer files


- Lambda file system mounting
  - Functions can access EFS if running in a VPC
  - Configure lambda to mount the EFS to local directory during initialisation
  - Must leverage EFS access point (can be limited number of these, one function = one connection)
  - Also if functions all connect at same time, may get connection burst limit to EFS


- Ephemeral storage (/tmp) is fastest as connected direclty, but limited to function only. 
- Lambda layers are also fast, and they offer durable storage
- S3 and EFS can be connected, to allow elastic durable storage


- Lambda Concurrency and Throttling
  - Could have up to 1000 concurrent executions 
    - For more than 1000 limit, can over a support limit
  - Can set a limit on which the lambda will be limited
    - Each invocation over the concurrency limit will trigger a "throttle"
      - For synchronous - will give 429 response
      - For async - will retry automatically then go to DLQ
    - With no reserve limit, if we have many consumers and get lots of users invocing the lambda, then we may mean that some consumers are throttled unfairly
  - Can have a cold start
    - New instance means that the code has to be run. First request has long latency, as need dependencies to load
    - Other option is to do **provision concurrency**
      - Concurrency is allocated before the function is needed
      - Means that it will be ready to run when called


- Lambda Dependencies
  - Need to add if function depends on libraries. eg. X-Ray SDK, Database Clients etc
    - AWS SDK is standard with lambda


- Lambda and Cloudformation
  - Can use Cloudformation to upload lambdas
  - Can do 
    - Inline - use within cloudformation setup; use Code.ZipFile property. Cannot include dependencies
    - Via S3 - store Lambda zip in S3 and give that in the CLouldformation code
  - Can use BucketPolicy and Execution Roles to allow different account to retieve Lambda from another account


- Lambda Container Images 
  - Can use containers of up to 10Gb from ECR
  - Base Image must implement the Lambda Runtime API
  - Allows packing complex dependencies, in a container to run. Can create own image. 
  - Can test the containers locally using the Lambda Runtime Interface Emulator
- Lambda Versions and Aliases
  - Usually uses $LATEST function - this is mutable
  - When publish a fuciton we create a version. These are immutable. Each version of lambda can be accessed
  - Aliases are "pointers" to a version
    - We can create a "dev" "prod" aliases and point to different versions. 
    - They are mutable
    - Supports canary deployment; which allows stable configuations. 
  - Aliases cannot reference other aliases


- Lambda and CodeDeploy
  - Codedeploy helps automated traffic shift for Lambda aliases - eg. increase % on v2 from v1
  - Can have
    - Linear - e.g. 10% every 10mins
    - Canary - try x% then 100%
    - All at once
  - Can create pre and post traffic hooks to check the health of the Lambda function


- Lambda function URL
  - Dedicated HTTP(S) endpoint for Lambda fuction
  - Gives a unique URL to invoke via public internet
  - (Do not go through gateway etc.)
  - Can be applied to any aliases or to $LATEST
  - Can throttle using Reserved Concurrecny settings
  - For security
    - Can use Resource-based Policy; or CORS
    - Can have AuthTypeNONE for all unathorised access
    - Can have AuthTyep AWS_IAM - for authenticaion of user


- Lalmba Limts
  - Execution
    - Memory 123Mb - 10Gb
    - Execution time - 900s
    - Env variables - 4kb
    - Disk capacity - 512Mb to 10Gb
    - Concurrency executions - 1000 (can be increase with request to AWS)
  - Deployment 
    - Deployment size compressed 50Mb
    - Uncompressed - 25oMb
    - Can use /tmp to load other files at startup
    - 
### Personal notes
