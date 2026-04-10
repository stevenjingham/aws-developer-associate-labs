# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 20 - Monitoring and Audit
> 
> Date: March 2026

---

## Purpose

Provide **monitoring, logging, tracing, and auditing** capabilities to:
- Debug applications
- Monitor performance (latency, errors)
- Detect outages
- Analyse trends and optimise costs
- Audit user and system activity

Core services:
- **CloudWatch** → Metrics, logs, alarms
- **X-Ray** → Distributed tracing
- **CloudTrail** → API auditing
- **EventBridge** → Event-driven automation

## Key Concepts

### Monitoring Services

- **CloudWatch**
  - Metrics, Logs, Alarms, Events
- **X-Ray**
  - Distributed tracing across microservices
- **CloudTrail**
  - Audit API calls and changes
- **EventBridge**
  - Event routing and scheduling

---

### CloudWatch Metrics

- Metrics exist per AWS service (e.g. CPUUtilization)
- Organized into **namespaces**
- Use **dimensions** (e.g. instanceId, env)
  - Up to 30 per metric
- Have timestamps
- Can build dashboards

Monitoring levels:
- Basic: 5 minutes
- Detailed: 1 minute (extra cost)

---

### Custom Metrics

- Created via `PutMetricData`
- Can define:
  - Dimensions
  - Resolution (1s–60s)
- Supports historical/future timestamps

---

### CloudWatch Logs

- **Log Group** → application
- **Log Stream** → instance/container

Features:
- Configurable retention (1 day → 10 years)
- Encrypted (default or KMS)
- Can export to:
  - S3 (batch, not real-time)
  - Kinesis / Firehose / Lambda (real-time via subscriptions)

Sources:
- EC2 (via agent)
- Lambda
- ECS
- API Gateway
- VPC Flow Logs
- CloudTrail

---

### Logs Insights

- Query and analyse logs
- Works across multiple log groups
- Historical queries only

---

### EC2 Logging

- Requires **CloudWatch Agent**

Agents:
1. Old → Logs only
2. Unified → Logs + system metrics (CPU, RAM, disk, etc.)

Requires IAM permissions.

---

### Metric Filters

- Extract patterns from logs (e.g. "ERROR")
- Convert into metrics
- Can trigger alarms

---

### CloudWatch Alarms

- Trigger actions based on metrics

States:
- OK
- Alarm
- Insufficient Data

Targets:
- EC2 actions (stop/reboot/recover)
- Auto Scaling
- SNS notifications

---

### Composite Alarms

- Combine multiple alarms (AND/OR)
- Reduce noise

---

### Synthetics Canary

- Simulates user behaviour
- Detects issues before users

Features:
- Runs scripts (Node.js / Python)
- Uses headless browser
- Captures latency + screenshots
- Can trigger alarms

---

### EventBridge

- Event routing service

Flow:
Source → EventBridge → Target

Supports:
- Scheduled events
- Event pattern matching

Targets:
- Lambda
- SNS
- SQS
- More

Features:
- Cross-account event aggregation
- Schema registry
- Event replay
- Resource-based policies

---

### AWS X-Ray

- Distributed tracing for microservices

Capabilities:
- Trace end-to-end requests
- Identify latency and failures
- Visual service map

---

### X-Ray Concepts

- **Trace** → full request path
- **Segment** → service
- **Subsegment** → internal step
- **Annotations** → indexed metadata
- **Metadata** → non-indexed data

---

### X-Ray Sampling

- Reduces cost by limiting traces

Default:
- 1 request/sec (reservoir)
- 5% of additional traffic

---

### X-Ray Integration

Steps:
1. Add SDK to app
2. Enable tracing
3. Run X-Ray daemon (or AWS-managed integration)

Works with:
- EC2
- Lambda
- ECS
- Beanstalk

---

### ECS X-Ray Modes

- Daemon per EC2 instance
- Sidecar container per app
- Fargate → sidecar only

---

### CloudTrail

- Audits API calls across AWS

Tracks:
- Who did what, when

Event types:
- Management events (default)
- Data events (optional)
- Insights events (anomaly detection)

Storage:
- 90 days in CloudTrail
- Long-term in S3


## How It Works
1. Applications/services emit:
  - Metrics → CloudWatch
  - Logs → CloudWatch Logs
  - Traces → X-Ray
  - API activity → CloudTrail

2. CloudWatch:
  - Stores metrics/logs
  - Triggers alarms

3. EventBridge:
  - Routes events to targets

4. X-Ray:
  - Builds service maps

5. CloudTrail:
  - Records audit logs


## Code / Config



## Common Pitfalls

- EC2 logs not sent without agent
- Confusing CloudWatch vs CloudTrail vs X-Ray
- Using short polling instead of long polling (logs/events context)
- Incorrect visibility timeout assumptions (ties into monitoring)
- Forgetting IAM roles for X-Ray or CloudWatch agents
- Not enabling detailed monitoring when needed
- Assuming CloudWatch logs export is real-time (S3 export is batch)

## Key exam notes
- CloudWatch = Metrics + Logs + Alarms
- CloudTrail = Audit API calls
- X-Ray = Distributed tracing
- EventBridge = Event routing
- Logs → CloudWatch Logs
- Metrics → CloudWatch Metrics
- Traces → X-Ray
- API history → CloudTrail
- Synthetics Canary = simulate user behaviour
- Metric Filters = logs → metrics
- DLQ + alarms often used together
- X-Ray requires:
  - SDK
  - IAM role
  - Daemon/integration
- CloudTrail:
  - Enabled by default
  - 90-day history
  - Can integrate with EventBridge
- EventBridge:
  - Can trigger Lambda/SNS
  - Supports cross-account aggregation
---

## Detailed Notes

### Video notes
- Monitoring is important to allow debugging, review application latency, see outages. 
  - Can review trends - to support application development and estimate costs
- CloudWatch allows
  - Metrics
  - Logs
  - Events - e.g. notifications when events happen
  - Alarms - react in real time to metrics/events
- AWS X-Ray
  - Troubleshooting performance and errors
  - Tracing of microservices (collates calls, see call trace)
- AWS CloudTrail
  - Internal monitoring of API calls
  - Audit changes 


- CloudWatch Metrics
  - Allows metrics for every AWS service
  - e.g. CPUUtilisation, NetworkIn
  - Metrics belong to **namespaces**
  - A **dimension** is a attribute of a metric (instanceId, env etc)
    - Up to 30x dimensions per metric
  - Metrics have timestamps
  - Can create CloudWatch dashboards of metrics
- Can have for example default EC2 monitoring every 5mins; but detailed can be 1 every minute. Additional cost
  - This would allow quicker scaling for example. 



- CLoudWatch Custom Metrics
  - can set own metrics, using **PutMetricData** API call
  - Ability to use dimensions on metrics
  - Can set metric resolution - either standard 60s, or high res 1/5/10/30s
  - Can get a metric from two weeks in past, or 2 hours in future. e.g. set date to swtich on 2 weeks in past. 


- CloudWatch Logs
  - Destination to store app logs
  - **Log Group** - name, representing app
  - **Log Stream** - instances within app, log files, containers
  - Can define log expiration policy (e.g. never, 1day-10year)
  - CloudWatch can send logs to S3, Kinesis, AWS Lambda, OpenSearch
  - Logs are encrypted as default, can setup KMS based with own keys
  - Sources of logs are
    - SDK, 
    - Eastic beanstalk
    - ECS
    - Lambda
    - VPC Flows
    - API Gateway
    - CloudTrail
    - R53
  - **CloudWatch Logs Insights** allows to query logs. 
    - Search and analyse logs
    - Can export data or create dashboard to re-run again
    - Can query multiple Log Groups 
    - Can only query historical data
  - Exporting logs can be 
    - S3 - export can take 12hrs, batch so not reat time
    - Subscriptions - allows real time into **destinations** Kinesis Data Strea, Firehose or Lambda
      - filters allow whick logs are delivered to destinations
      - can aggreage data from different accounts/regions using these subscription filters etc


- ClouldWatch Logs for EC2
- By default no logs from EC2 will go to Cloudwatch
  - Need to run a CloudWatch **agent** to push the logs
  - Must have the correct IAM permssions
    - There are 2x types of agent
      1) CloudWatch Log Agent - old version, only logs
      2) CloudWatch Unified Agent - additional details on system (RAM etc), centralised config using SSM Parameter store
         - Metrics could be CPU, Disk, RAM, Netstat, Processes, Swap space



- Metric filters 
  - Can filter logs (e.g. to find ERROR, or IP)
  - Can use these filters to create a **metric** of the data
  - Can define 3x dimensions per filter
  - e.g. can set CW alarm after 3x ERRORs filtered


- CloudWatch Alarms
  - Are used to **trigger notifications** for any metric
  - Various options of sampling percentage, max, min, etc
  - Alarm states are
    - OK
    - Insufficient Data
    - Alarm
  - Period - is the length of time to review 
- Alarm targets can be
  - Amazon EC2 to stop/reboot/recover
  - EC2 autoscaling
  - SNS notification
- Composite Alarms
  - Can monitor the states of other alarms; to effectively montior lots of metrics
  - Can be AND or OR for combinations
  - Helps reduce noise
- EC2 instance recovery
  - Status checks can be monitored, if the alarm is breached then can recover the instance. 
- Can test alarm state using CLI to set alarm status


- CloudWatch Synthetics Canary
  - Is a configurable script to reproduce what customers do programmatically to find issue before customers are impacted
  - Checks availability and latency of endpoints, and store load time data and screenshots
  - Integrates with alarms to allow functions/response etc
  - Scripts can be nodeJS or Python; access to headless Chrome browser
  - Can run once, or on a regular schedule
- Blueprints can be used eg.
  - Heartbeat - load URL
  - API Canary - test basic REST API
  - Broken Link Checker
  - Visual Monitor - check screenshot vs. baseline. 
  - Canary recorded


- EventBridge
  - Allows schedules (eg hourly) or event pattern (eg. if user signs in) to run lambda functions
  - Eventbridge sits in middle of Source (EC2, S3 event) --> EnvetBridge --> Destinations (e.g. lamda, SNS)
    - Eventbridge will output a JSON object
  - Can also send to partner SaaS buses or custom bus (e.g. private apps)
  - Can replay archived events
  - Schema Registry
    - Allows to generate code using the structure
  - Resource Based Policy allows permissino for specfic bus
    - Can define what can create send a put event into bus


- EventBridge Aggregation
  - Allows central account to push an event into a Central Account (if resource policy of central account allows input from other accounts)
  - Allows central control


- AWS X-Ray
  - Supports debug in production, especially for distributed MS
  - Allows visual analysis of trace of MS calls, and which are passing/failing
  - ALlows understanding of interaction of MS's and request timing/status/etc
  - Can use for Lambda, Beanstalk, EC2 and more
  - Traces the end-to-end request; annotation is added
  - To enable, :
    1) Import XRay SDK into code; 
    2) SDK captures calls to other services and trace. 
    3) Install X-Ray Daemon (e.g. on EC2) or enalbe X-Ray AWS integratino
  - X-Ray combines all data from all inputs
  - Need to ensure that Lambda has IAM execution role (AWSXRayWriteOnlyAccess)


- X-Ray Instrumentation
  - Can add this to measure performance, diagnose errors and write trace info
  - Small config changes for adding to cde
  - Definitions
    - **Segment** - each app will send
    - **Subsegment** - more detailed than segment
    - **Trace** - segments collected for end-to-end
    - **Sampling** - reduce amount of requests to XRay, reducing costs
    - **Annotations** - K-V pairs used to index traces and use filters
    - **Metadata** - K-V not indexed, used for searching
  - Sampling
    - Can control amount of data to record
    - Can control rules without changing code, supported via XRay Daemon
    - Default sampling - record first request each second; then 5% of additional requests
      - One RPS is **reservoir**, to ensure one trace is recorded
      - 5% is the **rate** above reservoir
    - Can have own sampling rules. Can differ them by application or endpoint etc. 



- X Ray APIs
  - These are used by the X Ray Daemon; need to have IAM policy to allow these
  - Write:
    - PutTraceSegments 
    - PutTelemetryRecords - metrics
    - GetSamplingRules - gets latest rules
    - GetSamplingTargets 
    - GetSamplingStatisticsSummaries
  - Read
    - GetServiceGraph - main graph
    - BatchGetTraces - gets trace by ID
    - GetTraceSummaries
    - GetTraceGraph - gets graph for one trace ID


- X Ray with Beanstalk
  - Can set via console, or add with configuration file
  - Daemon must have the IAM permissions; and the code must be running the SDK


- X Ray and ECS Integration Options
  - ECS Cluster: runs EC2 with single Daemon container per instance (shared by apps)
  - ECS Cluster - Sidecar: runs Daemon per app container (so may have multiple daemons per EC2 instance)
  - Fargate - Sidecar: as we do not control the EC2 instances
- Will run on a port 2000, with UDP protocol. Must set the env name for the daemon address.


- AWS Distro for OpenTelemetry
  - OT is a single set to contribute traces/metrics etc. Similar to XRay but opensource
  - Auto-instrumentation agents collect traces without changing code. 
  - Supports EC2, Fargate, etc.


- CloudTrail
  - Provides governance, compliance and audit of AWS account
  - Gets history of events/calls within AWS Account
  - Can put logs into CloudWatchLogs or S3
  - Enabled by default. Sees
    - ManagementEvent (eg. creating security )
    - Data events (e.g. read write events on S3 object. Not on as default as many occurances)
    - CloudTrail insights events - defects unusual activity. Paid for service. 
  - Allows investigation on who changed what and when :D
  - Stored in CloudTrail for 90 days, then can store in S3


- Can combine EventBridge to CloudTrail to see certain triggers from cloudtrail-->eventbridge-->SNS


- Cloudtrail
  - Audit API calls made by user/services
- CloudWatch
  - Metrics, Logs and alarms from services
- X-Ray
  - Automated trace analysis, visualisation

### Personal notes
