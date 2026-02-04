# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: AWS CLI SDK IAM Roles & Policies
> 
> Date: February 2026

---

## Purpose
- Allow AWS resources and applications to **authenticate, authorise, and interact with AWS services securely**
- Provide **identity-aware access** without hardcoding credentials
- Protect AWS APIs from abuse using authentication, rate limiting, and quotas

## Key Concepts
- **EC2 Instance Metadata Service (IMDS)**
  - Allows an EC2 instance to discover information about itself
  - No IAM permissions required just to read metadata
- **IMDSv2**
  - Uses session-based tokens
  - More secure than IMDSv1
- **AWS CLI Profiles**
  - Support multiple AWS accounts or roles from one machine
- **MFA with CLI**
  - Temporary credentials via AWS STS
- **AWS SDK**
  - Programmatic access to AWS services
- **Rate Limits vs Service Quotas**
  - Rate limits = API call frequency
  - Service quotas = resource limits
- **Credential Provider Chain**
  - Ordered lookup for credentials
- **AWS Signature Version 4**
  - Cryptographic signing of AWS API requests

## How It Works
- **EC2 Metadata**
  - EC2 exposes metadata at:
    - `http://169.254.169.254/latest/meta-data`
  - Can retrieve:
    - Instance ID, hostname, IP
    - IAM role name (not the policy)
    - User Data (bootstrap scripts)
  - IMDSv2 requires:
    1. Request a session token
    2. Pass token in request headers
- **CLI & SDK Authentication**
  - CLI/SDK resolves credentials using the provider chain
  - Uses IAM roles automatically when running on EC2/ECS/Lambda
- **MFA via STS**
  - STS issues temporary credentials
  - Credentials expire automatically
- **API Protection**
  - Requests are signed using **SigV4**
  - Rate limits protect backend services
  - Exponential backoff handles throttling safely

## Code / Config

## Common Pitfalls

## Key exam notes
- IMDS runs at 169.254.169.254
- IMDSv2 requires a session token
- Metadata can reveal IAM role name but not policies
- Prefer IAM roles over access keys
- MFA + STS = temporary credentials
- Rate limits → use exponential backoff
- Service quotas → request increases
- Credential Provider Chain order matters
- AWS APIs require SigV4 signing
- SDKs and CLI sign requests automatically
---

## Detailed Notes

### Video notes
- EC2 Instance Metadata 
  - allows EC2 instances to learn about themsevels without using an IAM Role for that purpose
  - Go to http://169.254.169.254/latest/meta-data from EC2 instance
  - e.g. can get instance name, IP address and can get IAM role name (but cannot retrieve the IAM policy)
  - Can get userdata as well, which about the bootstrap data
  - IMDSv2, need to get a session data first, and then use this in the header for API call
    - Did not need token for v1; but newer version is more secure


- A "profile" can be configures (aws configure) to create a profile to have a different account to use via CLI
  - Need to give name, access key, secret key and region
  - use --profile profilename when using AWS commands


- MFA within CLI, can be done by creating a temporary session
  - To do so, must run the **STS GetSessionToken API call**
  - e.g. aws sts get-session-token --serial-number arn-of-the-mfa-device --token-code code-from-token --duration-seconds 3600
  - (Need to have assigned a MFA device within AWS first within IAM Roles UI)
    - This gives an arm number to use in the 
  - API gives credentials back, inc. an access-key a session-token can be used which can be used in CLI


- AWS SDK allows AWS to be used via code directly
  - Java NET Python etc
  - Use lots in code and for Lambda functions


- AWS Limits
  - Rate limits - number of times can call in a row
    - For IntermittentErrors when have too many calls, should use **Exponential Backoff**
      - This is increasing the time between calls until we get a valid response e.g. 1s, 2s,4s,8s, 16s
      - This is implemented in AWS SDK API already
      - Only use this for 5xx errors (not 4xx errors as these are request invalid calls)
    - For ConsistentErrors we should request an API throttling limit increase to allow handle more calls
  - Service Quota / Service Limit (i.e. resourses we can run)
    - can request increase (e..g number of CPU's allowed) by raising a ticket or Service Quotas API


- AWS CLI Credientials Provider Chain
  - The heirarchy is:
    1) CLI options / Java system properties
    2) Environment variables
    3) CLI credential file
    4) CLI configuration file / default configuration file
    5) Container credntials (for ECS tasks)
    6) Instance profile credentials (for EC2 instance profiles)
  - Never store credentials in code
  - Use IAM roles where possible (e.g. EC2 instance roles, ECS Roles, Lambda roles); or outside of AWS use env variables


- AWS Signature v4 Signing
  - When ising AWS API, the request is signed with AWS credentials (access key and secret key) to allow identity
  - Using SDK or CLI, the HTTP requests are signed for you
  - Via another request, we should sign using SigV4
    - Need to send Authroisation headers; or query string
      - Passing in signature to these

### Personal notes
