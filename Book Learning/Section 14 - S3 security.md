# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 14 - S3 Security
> 
> Date: February 2026

---

## Purpose
Understand how Amazon S3 secures data, controls access, and customises object access.


## Key Concepts
### S3 Encryption at Rest
S3 supports multiple encryption methods to protect objects at rest.

Encryption options:
- **SSE-S3** – AWS manages and owns the keys (AES-256, default)
- **SSE-KMS** – Keys managed by AWS KMS, with audit and access control
- **SSE-C** – Customer provides and manages encryption keys
- **Client-side encryption** – Data encrypted before upload

Different methods offer trade-offs between simplicity, control, and cost.

### Encryption in Transit
Data sent to and from S3 can be encrypted using SSL/TLS.

- HTTP is unencrypted
- HTTPS encrypts data in transit
- Bucket policies can enforce HTTPS-only access

### Cross-Origin Resource Sharing (CORS)
CORS controls whether a web browser can access S3 resources from a different origin.

- Origin = protocol + domain + port
- Required when a web app loads resources from another bucket or domain
- Configured at the bucket level using CORS rules

### MFA Delete
MFA Delete adds extra protection for critical S3 operations.

- Requires a one-time MFA code
- Protects against permanent object deletion and versioning suspension
- Requires versioning and can only be enabled by the bucket owner (root)

### S3 Access Logs
S3 access logging records requests made to a bucket for audit and analysis.

- Logs include allowed and denied requests
- Logs must be delivered to a **separate bucket** in the same region
- Used for security reviews, compliance, and troubleshooting

### Pre-signed URLs
Pre-signed URLs provide temporary access to S3 objects.

- Can grant time-limited GET or PUT access
- Generated via Console, CLI, or SDK
- Permissions are inherited from the creator
- URLs automatically expire

### S3 Access Points
Access Points simplify bucket access management at scale.

- Each access point has its own policy and DNS name
- Policies can restrict access to specific prefixes
- IAM policies can reference access points directly
- VPC access points restrict access to private VPC traffic only

### S3 Object Lambda
S3 Object Lambda allows objects to be modified dynamically when accessed.

- Uses Lambda to transform data on-the-fly
- Original object remains unchanged
- Useful for filtering, redaction, or format conversion
- Access is managed through Object Lambda access points


## How It Works
S3 combines encryption, access controls, and request-level features to securely store and deliver objects while supporting fine-grained access and dynamic transformations.


## Code / Config

## Common Pitfalls

## Key exam notes
- Default encryption is SSE-S3
- SSE-KMS integrates with CloudTrail and IAM
- MFA Delete requires versioning
- Pre-signed URLs inherit creator permissions
- Object Lambda modifies responses, not stored objects
---

## Detailed Notes

### Video notes

- S3 Objects can be encrypted
  - 4x main methods
    1) Server-side Encryption with S3 managed keys (SSS-S3)
        - Key is handled managed and owned by AWS
        - Encryption is AES-256
        - Is set default for new buckets and objects
        - Must send header "x-amz-server-side-encryption": "AES256"
    2) SSE with KMS keys stored in AWS KMS (SSE-KMS)
       - Key Management Service (KMS) handles and manages the keys
       - User has control and audit of the keys. Audit via CloudTrail
       - Must send header "x-amz-server-side-encryption": "aws:kms"
       - Has limitation as need to use KMS; and need to do API calls to KMS. May go over Service Quota, although can set a bucket key that would reduce API calls for that bucket
       - N.B. there is also now **Double SSE-KMS - DSSE-KMS**
    3) SSE with customer-provided keys (SSS-C)
       - Keys managed outside of AWS, as keys are outside
       - Must pass the key into AWS. S3 does **not** store the key provided.
       - HTTPS must be used
       - Encryption key must be provided with every request made, to upload or read
    4) Client side encryption
       - Client encrypts before uploading
       - Client fully manages key, and encryption process
  - Encryption in transit (SSL/TLS)
    - HTTP Endpoint is not encrypted
    - HTTPS is; using encryption in transit
    - A bucket policy can be defined, to enforce HTTPS/in flight encrption


- Default is SSE-S3, automatically applied
  - Can set a bucket policy to force a different type of encryption to the one desired


- **Cross-origin resource sharing (CORS)**
  - Origin = scheme (protocol https) + host (domain) + port
    - e.g. https://www.example.com (implied port is 443 for HTTP, 80 for HTTP)
    - Same origin would be https://www.example.com/app1; https://www.example.com/app2
    - Cross origin would be  https://www.example.com/app1; https://www.other.com/app2
  - If a response from one request points to another origin, then CORS must be enabled within 2nd host to reply
  - e.g. web-browser, request to bucket 1, within response we also need image from bucket 2. Send request to bucket 2 with the origin from bucket 1. With CORS it will allow bucket 2 to respond


- MFA Delete
  - Forces users to generate a code on a device during important operations on S3
  - e.g. permanently delete an object; or suspend versioning
  - Must have **versioning enabled** and bucket owner (root account) can enable/disable MFA Delete


- S3 Access Logs
  - May want to see any request made to S3, for audit purposes. Includes denied requests
  - Target bucket will be logged by a logging bucket - must be **in same region** 
    - Cant use same bucket, as will be logging loop as writes get written!
  - Can then analyse the logs with an analysis tool


- Pre-signed URLs
  - Allows temporary access to download or upload a file
    - Generate a URL with S3 console, AWS CLI or SDK
      - Use-case: give a person a URL to allow GET/PUT on an S3 bucket. Permission inherited from user making URL.
      - URL will expire (S3 Console - 1min to 720mins; CLI 1s to 608400s == 168hrs)


- S3 Access Points
  - Access Points have a policy attached that allow access to certain parts of the bucket 
    - e.g. finance users can access /finance/ directory
  - Means that the policy is moved from S3 definition, to policy defined per access point (e.g. access point for finance, engineering etc)
  - IAM polocies can be aligned to the access points
  - Overall this make security of bucket simpler.
  - Each Access Point has its own DNS name (Internet origin or VPC origin) and an access point policy
  - A VPC Origin Access Point
    - Defines that an access point can only be accessed from inside a VPC
    - Do this by creating a VPC Endpoint to access the Access Point. VPC Endpoint must allow access to the access point and S3 bucket



- S3 Object Lambda
  - Use AWS Lambda Function to change an object before it is called by application
    - e.g. use case; one user gets full file, one user get redacted file; all while using **same** S3 bucket resource
    - other use case could be to convert data (eg. xml to json)
  - There will be:
    1) S3 Object Lambda Access Point - that a user will have access to
    2) Lambda function - that does a function on the object
    3) Supporting S3 Access Point - allowing lambda function to access S3 object
    - Note: other user may be allowed to go directly to S3 object rather than via access point
  - 


### Personal notes
