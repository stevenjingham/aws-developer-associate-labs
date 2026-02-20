# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 17 - Elastic Beanstalk
> 
> Date: February 2026

---

## Purpose
Provide a **fully managed application deployment platform** so developers can deploy and scale applications without managing infrastructure complexity.

- Handles provisioning (EC2, ELB, ASG, etc.)
- Manages deployments and scaling
- Developer-focused abstraction layer over AWS services
- Free service (pay for underlying resources)

**Typical Use Case:**
- Web applications with standard architecture
- Quickly deploying environments (dev, test, prod)
- Avoiding manual infrastructure configuration


## Key Concepts
### Core Components

- **Application**
  - Logical container for environments, versions, configs

- **Application Version**
  - Specific iteration of application code

- **Environment**
  - Collection of AWS resources running one application version
  - One app per environment
  - Typically: dev / test / prod

---

### Environment Tiers

1️⃣ **Web Server Environment**
- Uses ELB + Auto Scaling
- Handles HTTP requests

2️⃣ **Worker Environment**
- Uses SQS queue
- Processes background jobs

---

### Deployment Modes

- **Single Instance**
  - One EC2 instance
  - Good for dev/test

- **High Availability**
  - Load balancer + ASG
  - Recommended for production

### Deployment Strategies

1️⃣ **All at Once**
- Fastest
- Causes downtime
- No additional cost

2️⃣ **Rolling**
- Updates instances in batches
- Runs below capacity
- No additional cost
- Slower deployment

3️⃣ **Rolling with Additional Batches**
- Launches new instances before terminating old
- Maintains full capacity
- Small additional cost
- Good for production

4️⃣ **Immutable**
- Creates temporary ASG with new version
- Swaps once healthy
- Zero downtime
- Double capacity cost
- Fast rollback
- Good for production

5️⃣ **Blue/Green**
- Create separate environment (v2)
- Validate independently
- Swap CNAME/URL when ready
- Manual process

6️⃣ **Traffic Splitting**
- Canary deployment
- Send small % traffic to new version
- Automated rollback if unhealthy
- Once healthy, shifts traffic fully

---

### Environment Swapping

- Can swap CNAMEs between environments
- Enables Blue/Green deployments
- No DNS TTL wait (internal swap)

---

### Lifecycle Policy

- Max 1000 application versions
- Automatically delete old versions based on:
  - Time
  - Storage space
- Versions in use are not deleted
- Option to retain source bundle in S3

---

### .ebextensions

Configuration files included in deployment package.

Requirements:
- Located in `.ebextensions/` directory
- YAML or JSON
- `.config` extension

Allows:
- Custom option settings
- Add AWS resources (RDS, ElastiCache, etc.)
- Advanced configuration beyond UI

⚠️ Resources created via `.ebextensions` are deleted if environment is deleted.

---

### Cloning

- Clone environment configuration
- Useful for test → prod
- Copies:
  - Instance type
  - Scaling config
  - Env variables
- Does NOT copy:
  - Database data

---

### Migration Scenarios (Exam Favorite)

#### Changing Load Balancer Type

- Cannot change LB type after environment creation
- Must:
  1. Create new environment
  2. Deploy app
  3. Perform CNAME swap

---

#### RDS with Beanstalk

⚠️ RDS inside Beanstalk is tied to environment lifecycle.

Not recommended for production.

To migrate RDS out:

1. Snapshot RDS
2. Prevent RDS deletion in console
3. Create new environment without RDS
4. Attach existing RDS
5. CNAME swap
6. Terminate old environment
7. Manually clean failed CloudFormation stack

---



## How It Works
1. Upload application (ZIP or CLI)
2. Beanstalk creates:
  - EC2 instances
  - Auto Scaling Group
  - Load Balancer
  - Security Groups
  - CloudFormation Stack (behind the scenes)
3. Application version deployed
4. Auto Scaling & monitoring handled automatically

> Beanstalk is powered by CloudFormation stacks.

## Code / Config

## Common Pitfalls

## Key exam notes
- Beanstalk = PaaS abstraction over EC2, ASG, ELB
- Immutable deployment = safest zero-downtime strategy
- Blue/Green = separate environments + CNAME swap
- Traffic Splitting = Canary deployment
- RDS in Beanstalk is not production best practice
- Uses CloudFormation stacks internally
- Worker tier uses SQS
- Single instance mode is not highly available
---

## Detailed Notes

### Video notes

- Elastic Beanstalk supports deployment of AWS Services / Applications
  - On AWS developers may struggle with infrastructure management, code deployments, configuring all services, scaling
  - Often same architecture is used in many use cases 
  - Beanstalk is a managed service that is a developer view of deploying apps
    - It handles all the config, so application code can be devs focus
    - Dev has full control of configuration
  - It is free, but pay for underlying instances
- Components are:
  - Application - collection of componenets (env, versions, configs)
  - Application version - iteration of app code
  - Environment - collection of AWS resources running app version (only one app per env). Can be dev, test, prd
    - Tiers: 
      - Web Server Environment - uses ELB etc
      - or, Worker Environment Tier - uses SQS Queue 
- Can have different deployment modes:
  - Single instance - good for dev
  - High availability with LB - good for prod. 


- Beanstalk Deployment Options
  1. **All at once** - fast, but instances are not available to serve traffic for some time.
     - Con as some downtime, but is very quick. No additinal cost.
  2. **Rolling** - update a few instances at a time, and move to update others once healthy
     - App runs below capacity. Will be running both versions simultaneously for some time. No additional cost. Can be long deployment. 
  3. **Rolling with additional batches** - like rolling, but spins new instances to move batch (so old application still available)
     - Runs at capacity. Will be running both versions simultaneously for some time. Small additional cost. Can be long deployment. Good for prod
  4. **Immutable** - spins new instances with new temporary ASG, deploys and swaps all instances once new instances healthy
     - Zero downtime, high cost as double capacity. Longest deployment. Quick rollback in case of failure. Good for prod. 
  5. **Blue Green** - create new env, and switch over when ready
     - Zero downtime, new environment running v2 that can be validated indepedently and then can saw URLs to new env. Is manual over Beanstalk
  6. **Traffic splitting** - for canary testing, send small % to new deployment
     - New app on temporary ASG with same capacity. Health monitored, and roll back can be very quick. Automated. Once healthy, instances moved to original ASG. 


- Can swap environment (which effectively swaps the URLs); which allows a blue-green for example


- Elastic Beanstalk CLI
  - can be used specifically for Beanstalk & speeds up working with Beanstalk from CLI
  - supports deployment, can use zip to upload application to EC2 instances (via S3 bucket)


- Beanstalk **lifecycle policy**
  - Can store max 1000 applications, but if get more than that we will not be able to deploy more
  - Lifecycle policy 
    - allows to **phase out old versions** based on either
      1) Time 
      2) Space (when too many versions)
    - versions in use will not be deleted
    - can opt **not to delete source bundle** in S3 to prevent data loss (and later allows re-deployment)


- Beanstalk extensions
  - All parameters usually set in UI, can be configured through files (in zip file containing code)
  - Requirements of file:
    - Must be in .ebextensions/ directory in root of code
    - YAML or JSON
    - .config file appendix
  - Allows update of detail settings using option_settings
  - Ability to add resources such as RDS, ElastiCache
  - Resources managed by .ebextensions are deleted if the environment is deleted


- Beanstalk and CloudFormation
  - Beanstalk relies on CloudFormation - CloudFormation works behind the scenes of Beanstalk
  - Through the extensions, we can handle a lot more than via the Beanstalk UI!
  - Cloudformation creates **stacks** that handles the Beanstalk resources


- Cloning
  - Can clone an environment with exact same config
    - e.g. test to prod. 
    - Allows settings to be updated after cloning. 
  - Clones type and config of resources (but not data e.g. in a database)
  - Clones env variables


- Beanstalk Migration
  - e.g. Load Balancer
    - After creating env, we cannot change type of ELB (can change config)
    - Can create new env with same config execpt LB (cannot clone, as will have same LB type)
    - Deploy app on new env
    - Perform CNAME change swap to Route53 update to new LB
  - e.g. RDS with Elastic Beanstalk
    - RDS is great for dev and test
    - But RDS is not good for prd via beanstalk as the DB is tied to environment lifecycle
    - If this is the state in prd:
      - Snapshot of RDS DB
      - In RDS console, set RDS DB from deletion
      - New env, without RDS
      - Point new env to existing RDS
      - CNAME swap to new env
      - Terminate old env (RDS not deleted as we set in console)
      - Will need to delete CF stack manually as it will be in failed state after not deleting RDS
      - RDS now lives outside the new env. 


### Personal notes
