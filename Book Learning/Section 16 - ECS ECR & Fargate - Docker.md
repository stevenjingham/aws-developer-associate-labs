# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 16 - ECS ECR & Fargate - Docker
> 
> Date: February 2026

---

## Purpose
Provide a managed way to **build, store, and run containerized applications** on AWS.

- **Docker** → Package application + dependencies into containers.
- **Amazon ECR** → Store container images.
- **Amazon ECS** → Orchestrate and run containers.
- **AWS Fargate** → Serverless compute engine for containers.
- **Amazon EKS** → Managed Kubernetes alternative to ECS.

**Primary use cases:**
- Microservices architecture
- Lift-and-shift migrations
- Event-driven workloads
- Batch / scheduled container jobs


## Key Concepts
### Containers
- Built from a **Dockerfile**
- Image stored in a repository (ECR)
- Run as containers on compute infrastructure
- Share host resources (CPU, memory, networking)

---

### Amazon ECR
- Private or Public registry
- Integrated with ECS/EKS
- Backed by S3
- Supports image vulnerability scanning
- IAM controls push/pull access

---

### Amazon ECS

AWS-native container orchestration service.

#### Launch Types

**1️⃣ EC2 Launch Type**
- You provision & manage EC2 instances
- ECS Agent runs on each instance
- More flexibility, more operational overhead

**2️⃣ Fargate Launch Type**
- Serverless
- No EC2 provisioning
- Define CPU & memory only
- AWS manages scaling & infrastructure

---

### IAM in ECS (High Exam Importance)

- **EC2 Instance Profile**
  - Used by ECS Agent
  - Pulls images from ECR
  - Calls ECS APIs

- **ECS Task Role**
  - Attached to Task Definition
  - Used by the running container
  - Grants application-level permissions (e.g., S3, DynamoDB)

> Exam trap: Instance role ≠ Task role.

---

### Task Definitions

JSON blueprint defining how containers run:

- Image
- CPU / memory
- Port mappings
- IAM task role
- Environment variables
- Logging configuration
- Networking mode
- Volumes

- Up to 10 containers per task

## How It Works
1. Build Docker image
2. Push image to ECR
3. Create Task Definition
4. Create ECS Service or Run Task
5. Attach Load Balancer (optional)
6. Configure Auto Scaling (optional)

---

### Load Balancing

#### EC2 Launch Type
- Supports **dynamic host port mapping**
- Only container port required in task definition
- Must allow ALB Security Group access to *all ephemeral ports*
- Supports ALB, NLB (CLB not recommended)

#### Fargate
- Each task has its own ENI & private IP
- No host port required
- ALB connects directly to task IP
- Security groups attached to ENI

---

### Storage & Volumes

- **EFS**
  - Works with EC2 & Fargate
  - Shared persistent storage
- Cannot mount S3 directly
- **Bind Mount**
  - Share data between containers in same task
  - EC2 → tied to EC2 lifecycle
  - Fargate → ephemeral storage (20–200 GB)

---

### ECS Auto Scaling

Uses **Application Auto Scaling**

Scale based on:
- Average CPU
- Average Memory
- ALB Request Count per Target

Scaling strategies:
- Target Tracking
- Step Scaling
- Scheduled Scaling

> Exam trap: ECS Service scaling ≠ EC2 Auto Scaling.

- EC2 ASG scales instances
- ECS Service scales tasks
- Capacity Providers help coordinate scaling

---

### Rolling Updates

Control deployment behavior using:

- **Minimum healthy percent (0–100%)**
- **Maximum percent (100–200%)**

Determines:
- How many tasks must stay running
- How many new tasks can be created

Supports zero-downtime deployments.

---

### Event Integrations

- **EventBridge**
  - Trigger ECS tasks on events (e.g., S3 upload)
  - Scheduled tasks
- **SQS**
  - ECS service polls queue
- Can emit events when task completes

---

### Task Placement (EC2 Only)

Placement process:
1. Check CPU, memory, port requirements
2. Apply placement constraints
3. Apply placement strategy
4. Select best instance

#### Placement Strategies
- **Binpack** → Minimize number of instances
- **Random**
- **Spread** → High availability (e.g., across AZs)

#### Placement Constraints
- **distinctInstance**
- **memberOf** (Cluster Query Language expression)

---

### Amazon EKS

- Managed Kubernetes service
- Cloud-agnostic container orchestration
- Supports EC2 and Fargate

Node types:
- Managed node groups
- Self-managed nodes
- Fargate (no nodes)

Supports storage via:
- StorageClass
- CSI drivers (EBS, EFS)

---


## Code / Config

## Common Pitfalls

## Key exam notes
- Fargate = serverless containers (no EC2 management)
- EC2 launch type = more control, more ops overhead
- Use Task Role for app permissions
- Use Capacity Providers to link ECS scaling with ASG
- Spread strategy = HA
- Binpack = cost optimization
- EFS = shared persistent storage
- EventBridge can trigger ECS tasks
- ALB recommended for most ECS workloads

EKS = Kubernetes, cloud-agnostic alternative to ECS
---

## Detailed Notes

### Video notes
- Docker is a software dev platform to deploy apps
  - Apps are packages in containers that can run on any OS and always runs the same
    - No compatability issues, less work, easier to maintain and delpoy
    - Use case: MS arch, shift apps from prem to AWS
- Server (e.g. EC2 instance) host multiple docker containers running images
- Images are stored in **Docker Repositories**
  - Public hub
  - Private hub - Amazon **ECR (Amazon Elastic Container Registary)**
- Resources are shared with the host. Many containers on one server. 
  - Data can be shared across containers
- Need to create a Dockerfile, build an image, push to repo, and run the image in container
- Docker container management on AWS
  - **Amazon Elastic Container Service (ECS)** is AWS own container platform
  - **Amazon Elastic Kubernetes Service (EKS)** is AWS managed Kubernetes
  - **AWS Fargate** is AWS serverless container platform and works with ECS & EKS
  - **Amazon ECR** stores the containers


- Amazon ECS
  - EC2 Launch Type
    - ECS Cluster hosts many EC2 instances (ECS tasks)
    - Must provision the EC2 instances and maintain infra
    - Each EC2 instance must run the ECS Agent to register in the ECS cluster
    - AWS takes can of starting/stopping containers
    - IAM Roles for ECS
        - EC2 Instance Profile is used by ECS agent
        - ECS Agent makes API calls to the ECS service and pulls docker image from ECR
        - ECS Task Role allow each task to have a specific role. Each role aligns different ECS services to run
  - Fargate launch types
      - Do not need to provision infra. Just define the task defintions
      - AWS runs ECS tasks based on CPU/RAM needed. Fargate scales auto. Very easy to manage
      - Its serverless
  - ECS - Load Balancer
    - ALB can be used and good for most use cases into the ECS Cluster
    - NLB recommended for high throughput/performance
    - CLB is supported but not recommended. Cannot use for Fargate
  - Data Volume (EFS Elastic File System)
    - Can mount an EFS onto ECS tasks
    - Works for EC2 and Fargate Launch Types
    - Allows sharing the same data between each EC2 containers/Fargate containers
    - (NB cannot mount an S3)


- ECS Service Auto-scaling
  - Can use **Application Auto Scaling** tp auto scale number of ECS tasks
  - Can scale on:
    - Average CPU Utilisation
    - Average Memory Utilisation
    - ALB Request Per Target
  - Can also setup scaling for Target Tracking (Cloudwatch metric), Step Scaling (Cloudwatch alarm), or Scheduled Scaling
  - Note ECS Service Auto Scaling (task level) is **not the same** as EC2 Auto Scaling (EC2 instance level)
    - Fargate Auto Scaling is much easier to scale as it is serverless
  - It is able to Auto Scaling EC2 Instance
    - This accommodates ECS Service Scaling
    - Can use ASG (based on CPU usage), or ECS Cluster Capacity Provider to scale for ECS tasks (scales when cannot run another task)


- ECS - Rolling Updates
  - When updating service to v1 to v2, can control how many tasks can be started and stopped and in which order
  - Can set a min healthy perc (0-100%) and maximum perc (100-200%)
    - Will terminate tasks above minimum if allowed
    - Will create tasks up to maximum percentage; using newer version
    - e.g. if min is 50% and max 100% and we start with 4x tasks on v1:
      - Step 1 - 4x v1
      - Step 2 - 2x v1 (50% cap)
      - Step 3 - 2x v1, 2x v2 (100%)
      - Step 4 - 2x v2 (50%)
      - Step 5 - 4x v4
  - Depending on values, could create v2 without deleting v1. Then after creation of v2, we can delete v1


- ECS tasks invoked by **event bridge**
  - May have rule on event brindge to run a ECS task (e.g. get object, once it is uploaded to S3)
  - Or another use-case may be to run a task on an **Event Bridge Schedule**
- ECS SQS Queue
  - Messages sent to SQS queue, and ECS Service polls for tasks
- Can also use events when task finished; and send event to EventBridge to take action after task finished (e.g. send email to user)


- **Task definitions** are metadata in JSON form to tell ECS how to run a Docker container.  
  - JSON hosts crucial info such as:
    - Image name; port binding container and host; memory CPU needs, env vars; networking info; IAM role; loggging configu
      - host port exposes E2 instance to website. container port exposes image to host.
    - Can define up to 10x containers in Task Defintion
- ECS Load Balancing (EC2 launch type)
  - Get **dynamic host port mapping** if you only define the container port in task definition
    - e.g. allows random host port; meaning can use ALB to connect to an ECS Cluster. ECS Cluster can support mapping to ECS Task
  - Must allow **any port** from the ALB Security Group as we do not know mapping
- ECS Load Balancing (Fargate)
  - Each task has unique private IP
  - Do not need to define the host port as it is Fargate, Just define the container port.
  - ALB will connect to each of the task IP addresses
  - ECS ENI SG needs to allow port 80 from ALB. ALB SG needs to allow port from web.
- IAM Roles within ECS - have one IAM role per Task Definition
  - Each service running the task, inherits the IAM Role
  - So define the IAM roles in task definition
- ECS Environement Variables
  - Can be:
    - Hardcoded
    - SSM Parameter Store (eg. API Keys). Fetch values from here - define in definition. Use ARN value.
    - Secrets manager (eg. DB pwd). Fetch values from here - define in definition. Use ARN value.
    - Bulk load from S3 bucket. Fetch values as defined in task defintino
- Data volumes can be **bind mount**
  - Can share data between **multiple containers in the same task defintion**. Works for EC2 and Fargate
    - share data, or sidecar app use case is logging etc. 
  - Bind mount allows for example multiple app containers to write to shared storage, and sidecar container to read
  - If doing EC2 task, use EC2 instance storage - so data is tied to EC2 lifecycle
  - If doing Fargate task, use ephemeral storage - so data is tied to container(s) using them. (20-200Gb)


- ECS Task Placement
  - When a task of type EC2 is launched, ECS must determine where to place based on CPU, memory, available port on EC2 instance
    - e.g. ECS service determines which EC2 instance to go on
    - They are a best effort
    - Also applies when scaling down, deciding which to terminate
  - Can define a **task placement strategy** and **task placement constraints**
- Placement process is:
  1. Identify requirements (CPU, memory, port as per task definition)
  2. Identify instances that satisfy task placement constraints
  3. Identify instances that satisfy task placement strategy
  4. Places in best instance
- Placement strategies
  1. **Binpack** - places task based on least available CPU or memory. Minimises number of instances
  2. **Random** - places tasks randomly onto EC2 instances
  3. **Spread** - places task evenly based on specified value (e.g. spread over AZ regions). Maximises high availability
  - Can mix stragies together
- Placement constraints:
  1. distinctInstance - place each task on different container instance
  2. memberOf - places task on instances that satisfy expression (e.g. ecs.instance-type = t2.*). Uses Cluster Query Language to define expression. 



- Amazon ECR (Amazon Container Registry)
  - Can store and manage images on AWS. Can be private or public
  - Fully integrated on ECS, and backed by S3
  - Set IAM role to EC2 instance, allowing it to pull images from ECR
  - Supports image vulnerability scanning, versioning etc.


- AWS Co-pilot
  - CLI took to build, release and operate prod-ready containerized applications
  - Helps focus on building apps vs infra (it sets up infra for ECS, VPC, ELB, ECR etc )


- Amazon EKS
  - **Amazon Elastic Kubernetes Service (EKS)** is AWS managed Kubernetes
  - Launch a managed kubernetes cluster on AWS
    - It is open source system for auto deploy, scale and management of containers
    - (Alternative to ECS)
    - EKS supports EC2 and Fargate
  - Kubernetes is **cloud agnostic**
  - Node types are:
    - Managed node groups - creates and managed nodes (EC2 instances) for you. Nodes are part of ASG managed by EKS
    - Self-managed nodes - user creation, and need to register to EKS cluster 
    - Fargate - no nodes
  - Can host data volumes by specifying a **StorageClass** on EKS cluster. Leverages a **Container Storage Interface (CSI)** compliant driver
    - Supprts EBS, EFS

### Personal notes
