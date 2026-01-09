# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 4 - IAM & AWS CLI
> 
> Date: January 2026

---

## Purpose
IAM (Identity and Access Management) controls who can access AWS and what actions they can perform on which resources

Used to enforce security and least privilege
## Key Concepts
- Global service (not region-specific)
- Users = individual identities (root user exists by default → do not use)
- Groups = collections of users (permissions assigned at group level)
- Policies = JSON documents defining permissions
- Least Privilege Principle = grant only required permissions
- Roles = permissions for AWS services, not people
- MFA = additional security layer for users (especially root)

## How It Works

## Code / Config

## Common Pitfalls

## Key exam notes


## Detailed Notes

### Video notes

- IAM = Identity and Access Management
  - It is a global service
- Users are people in an organisation, and they can be grouped
  - The root user is created by default and shouldn't be used
  - Groups cannot be put into other groups
- Users or groups can be given permissions
  - Assigned in JSON policies
  - Says which services the users can utilise
  - Want to give the "least privilege principle"


- Hands-on practice creating an admin-user-1; and a group


- IAM Policies inheritance
  - All users in a group will **inherit** the policy of the group
    - A user can inherit from more than one group
  - Can also have an "inline" policy for specific users
- Policy structure within JSON
  - Version
  - Id
  - Statement
    - **SID** (identifier)
    - **Effect** - where the statement allows or denies access
    - **Principal** - account, users or roles which this policy is applied to
    - **Action** - list of actions the policy allows or denies
    - **Resource** - list of resources the action applies to
    - **Conditions** - when the policy is in effect. 


- Hands-on practice creating policies


- IAM Password policy
  - Can set min length, and specify character types. Can set expiration dates and prevent pwd re-use
- Multi-factor authentication
  - Users have access to account, so want to protect **root user and IAM user**
  - Benefit as requires a physical device so less likely to hack. Options on AWS are:
    - Virtual (e.g. phone)
      - Google Authenticator 
      - Authy
    - Universal 2nd Factor key 
      - (e.g. YubiKey)
    - Hardware Key fob
    

- Hands-on practice creating password policy


- Can access AWS via:
  - AWS Management Console
  - AWS CLI - using access keys
  - AWS SDK (Software Development Kit) - also via access keys
- Access keys are generated through the AWS console
- AWS CLI allows access to public APIs in AWS services. It is open source. 
- AWS SDK is a set of libraries to access/manage AWS services. It can be embedded in application (e.g. Java, iOS, etc)



- Hands-on practice creating access keys & using CLI


- AWS Cloudshell is an alternative to the Command Line via Windows
  - It is a terminal within AWS
  - Can upload/download files etc. Files are stored if created via Cloudshell


- **IAM Roles** for Services
  - Some AWS Service will need permissions to perform actions on your behalf
  - This is done via IAM roles
    - e.g. an EC2 instance may need to do some actions on AWS, so we need to apply an IAM Role to the EC2 instance
  - Common roles are EC2 instance, Lamdba roles, CloudFormation
  - Hands on creating a role for EC2 instance
  - Good visual of policy within the role
    ~~~
    {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "sts:AssumeRole"
            ],
            "Principal": {
                "Service": [
                    "ec2.amazonaws.com"
                ]
            }
        }
    ]
    }
    ~~~

- IAM Security tools
  - Credentials report - shows users and status of credentials
  - Access Advisor - shows service permissions granted to user, and when last accessed (good to align to least privilege best practice)


- Shared responsibility model
  - Defines for what **AWS** is responsible for, and what the **user** is responsible for
  - AWS is responsible for:
    - Infracstucutre
    - Configuration and vulnerability access
    - Compliance validation
  - User is responsible for:
    - Users, groups, role and policies
    - Enabling MFA, rotation of keys 
    - Setting appropriate permissions
    - Auditing IAM for users

### Personal notes

- IAM Best Practice
  - Do not use root account except for AWS account setup
  - One physical user = one AWS user
  - Use groups, permsission and strong passwords
  - Audit permissions regularly