# Introduction

> Course: Ultimate AWS Certified Developer Associate 2026 DVA-C02 - Stephane Maarek
> 
> Section: 18 - CloudFormation
> 
> Date: February 2026

---

## Purpose

Provide a **declarative Infrastructure as Code (IaC)** service to define, provision, and manage AWS infrastructure.

- Automatically creates resources in the correct order
- Manages dependencies between resources
- Enables version-controlled infrastructure
- Uses **Stacks** to group resources
- Powered by templates (YAML/JSON)

**Primary Benefits:**
- Infrastructure as Code (no manual provisioning)
- Reusable templates
- Predictable deployments
- Cost visibility (via tags, stack-level estimation)
- Easy recreation/deletion of environments

## Key Concepts

### Template Structure

Sections:

- `AWSTemplateFormatVersion`
- `Description`
- **Resources** (only mandatory section)
- `Parameters`
- `Mappings`
- `Conditions`
- `Outputs`
- Intrinsic Functions (helpers)

> Templates are uploaded (often to S3) and used to create a **Stack**.  
> Stack = collection of AWS resources managed together.

---

### Resources (Mandatory Section)

- Defines AWS resources
- 700+ supported resource types
- Format: `AWS::Service::ResourceType`
- CloudFormation controls creation, updates, deletion order
- Resources can reference each other

Docs specify whether updates cause:
- In-place modification
- Replacement (delete + recreate)

If not supported → use **Custom Resources**

---

### Parameters

Dynamic inputs provided at stack creation.

Use when:
- Value not known beforehand
- Value changes between environments

Features:
- Types (String, List, AWS-specific types, etc.)
- `AllowedValues`
- `Default`
- `NoEcho` (hide sensitive values)

- !Ref ParameterName

#### Pseudo Parameters (built-in):

- AWS::AccountId
- AWS::Region
- AWS::StackName
- AWS::StackId
- AWS::NoValue

### Mappings

- Static key-value variables inside template.

- Used for:
  - Region differences
  - AMI mappings
  - Environment variations
- Access with:
  - !FindInMap [MapName, TopLevelKey, SecondLevelKey]
- Use Mappings for known static values.
- Use Parameters for user-specific input.

### Outputs
- Expose stack values for reuse.
- Example:
  - Outputs:
      VPCId:
        Value: !Ref MyVPC
        Export:
          Name: MyVPCExport
  - Import in another stack:
    - !ImportValue MyVPCExport
⚠️ Cannot delete exporting stack until imports are removed.

### Conditions
- Control resource creation based on logic.
- Example:
  - Conditions:
      IsProd: !Equals [!Ref EnvType, prod]
  - Apply to resource:
    - Condition: IsProd
  - Supported functions:
    - Fn::Equals
    - Fn::And
    - Fn::Or
    - Fn::Not
    - Fn::If 
  - Intrinsic Functions
    - !Ref → Returns resource ID or parameter value
    - !GetAtt → Gets resource attribute
    - !FindInMap → Reads mapping
    - !ImportValue → Cross-stack import
    - !Base64 → Encode UserData
    - Condition functions (And, Or, If, Equals)

## How It Works
- Write template
- Upload template
- Create Stack
- CloudFormation provisions resources
- On update:
  - Upload new template version
  - Review change set
  - Apply changes
- Deleting a stack deletes all associated resources (unless protected).

## Code / Config

## Common Pitfalls

## Key exam notes
- CloudFormation = Declarative IaC
- Stack = deployment unit
- Resources = only mandatory section
- Parameters = dynamic inputs
- Mappings = static values
- Outputs + ImportValue = cross-stack linking
- DeletionPolicy protects data
- Capabilities required for IAM resources
- StackSets for multi-account deployment
- Rollbacks automatic by default
---

## Detailed Notes

### Video notes
- AWS CloudFormation is a declarative way to define AWS Infrastructure 
  - CloudFormation will create resoures in the right order and link them together
  - Can use **Infrastructure Composer** to visualise components on AWS
  - The benefits are:
    - Uses Infrastructure as Code (no manual resources; code controlled via codes/MR)
    - Cost (can see tags vs. resources for stack costs, can estimate costs with template, in dev could automate deletion/creation at certain times)
    - Productivity (can re-create and delete as required, declarative and dont need to do orchestation)
    - Can re-use templates
  - Templates must be uploaded in S3 and then Cloudformation references it to create a Stack
    - A **stack** is a collection of AWS resources
  - To update a template we can't edit previous versions. Have to upload a new version
    - When updating AWS will show which resources are changing (being added new, modifying or deleting)
  - Stacks are identified by name. When deleting a stack, all accosiated resources are deleted. 
  - To create templates:
    - Manual via code in Infrastructure Composer
    - Automated via YAML files, and using CLI or CD tools
  - The template contains
    - AWSTemplateFormatVersion 
      - identifies capabilities of template
    - Description
      - comments on template
    - Resources
      - **only mandatory section**. Defines AWS resources
    - Parameters
      - dynamic inputs for template. AWS UI will ask for input when creating stack
    - Mappings
      - static variables
    - Outputs
      - references to what is created in template
    - Conditionals
      - list of conditions when creating resources
    - (Helpers) References
    - (Helpers) Functions


- Resources
  - This is **only mandatory section**. 
  - Defines AWS resources
    - They can references each other
    - There are 700+ resources
    - CloudFormation defines the creation, updates and deletes of resources. (and in which order)
    - Resource type identifiers are in the form **service-provider::name::data-type**
    - Lots of documentation to create the YAML
      - Docs say if a change to each params means that the existing resource with be deleted or modified
  - For any resources **not yet supported** by CloudFormation, then need to use **CloudFormation Custom Resources**


- Parameters 
  - dynamic inputs for template. AWS UI will ask for input when creating stack
  - when to use a paramters
    - some inputs are not known before, if likely to change then it should be parameter
  - can be lots of types (string, value, list, AWS-specific paramter etc)
  - can set **AllowedValues** to define what is allowed; and can set **default values**
    - user will only be able to select those values given (via dropdown etc)
  - can set a **NoEcho**; which means value is not displayed anywhere -- eg for password
  - to reference a parameter in the YML use:
    - **Value: !Ref ParameterName** or Fn::Ref ParameterName
    - Can reference anywhere in template. And can reference other references
    - (NB can refence a resource in same way using !Ref ResourceName)
  - **Pseudo Parameters** are enabled by default. They are for example:
    - AWS::AccountId, AWS::Region, AWS::StackId, AWS::StackName, AWS::NoValue


- Mappings
    - static/fixed variables
    - handy to differentiate between different environments (dev vs prod), regions, AMI types
    - values are hardcoded in template
    - uses for example
      - RegionMap (mapping name)
        - us-east-1 (top level key1)
          - HMV: 123 (second level key: value1)
        - us-east-2 (key2)
          - HMV: 789 (key: value2)
    - To access a mapping we use:
      - **!FindInMap [Map name, TopLevelKey, SecondLevelKey]** or Fn::FindInMap
- Use mapping when know the value to be used and can be defined in maps. It is safer control
  - However, only use when the values are known and use parameters when the value is user specific


- Outputs
  - references to what is created in template. Allows other stacks to import any values exported
  - This allows stacks to link (e.g. network stack to application stack)
  - Usage:
    - Outputs:
      - StackGroupResource:
        - Description:
        - Value: !Ref Resource
        - Export: 
          - Name: exportName (must be unique across stacks in region)
  - Use in other stack with:
    - !ImportValue: stackName
- Note - cant delete stack until all stacks/references using the exports are deleted


- Conditions 
  - used to control the creation of resources/outputs based on a condition
    - e.g. may only want to define certain resources in certain env / regions / paramater values
  - Condition can reference other conditions, resources or mappings
  - e.g. 
    - Conditions:
      - CreateConditionName: !Equals [!Ref EnvType, prod] --> true for prd
  - Can be Fn::And, Equals, If, Not, Or
  - Use a condition with resource use:
    - Condition: CreateConditionName


- Intrinsic Functions
  - Fn::Ref
    - Gets referenced resource/parameter. Usually returns ID of resource. 
    - e.g. !Ref ResourceName
  - Fn::GetAtt
    - Gets an attribute of a resource (e.g. availabilityZone)
    - See docs to see what attributes are on each resource
    - e.g. !GetAtt EC2InstanceName.AvailabilityZone
  - Fn::FindInMap
    - Return value from a map
    - !FindInMap [Map name, TopLevelKey, SecondLevelKey]
  - Fn::ImportValue
    - Import a value exported from another stack
      - !ImportValue: resourceName
  - Fn::Base64
    - Converts string to Base64 to pass UserData 
    - e.g. !Base64 "string"
  - (and the condition functions as per above)


- CloudFormation Rollbacks
  - If stack creation fails there are 2x options:
    1) Default - everything rolls back. Log available
    2) Disable rollback - troubleshoot
  - If stack update fails:
    - Default - stack rolls back to previous working state. Stack is deleted
    - Rollback but preserve successful resources
  - If rollback fails:
    - Fix resource manually using **ContinueUpdateRollback API** from console (or can do via CLI)


- Service Roles
  - CloudFormation can use IAM role to allow create/update/delete stack resources
    - Can say CloudFormation can define S3 etc. 
  - Gives ability of users to update stack even if they do not have permissions (User has CloudFormation permission via iam:passRole)
  - This is good for achieving least privilege principle
  - If no IAM role set via CloudFront will use users role. 


- Capabilities
  - **CAPABILITY_NAMES_IAM** and **CAPABILITY_IAM**
    - used to enable CF template to create or update IAM resrouces (e.g. IAM User, Role,...)
  - **CAPABILITY_AUTO_EXPAND**
    - when template has macros/nests to perform dynamic transforms. template may change in deployment
  - **InsufficientCapabilitiesException**
    - exception thrown by CF is capabilities have not been acknowledged when deploying template (user input via UI)
      - is a security measure


- Deletion policy
  - Allows to control what happens when CF template is deleted or resource is removed from template
  - Default is **DeletionPolicy: Delete**
    - However, note; will not delete an S3 bucket which is not empty!
    - (Need to manually delete files, or do CustomResource to delete files)
  - **DeletionPolicy: Retain**
    - specifies resources to retain
    - works with any resources
  - **DeletionPolicy: Snapshot**
    - creates final snapshot before deletion
    - some resources allowed - e.g. EBS Volume, ElasticCache, RDS


- Stack Policy
  - By default, during a stack update, all updates are allowed on all resources
  - Stack policy is a JSON document that defines the update actions allowed on specific 
    - e.g can DENY any updates on PRD DB
  - It protects resources from unintentional updates
  - When implementing a stack policy, all resources are protected by default. Need an explicit ALLOW on any resources allowed


- Termination Protection
  - Prevent accidental deletes of CloudFormation **Stacks** - uses **TerminationProtection**
  - Enabled via the console - deactivated by default, but can activate.
  - When user aims to delete stack, it gives an error


- Custom Resources
  - Are used to define resources;
    1) **not yet supported by Cloudformation**
    2) custom provision logic for resources outside CF (e.g, on-prem or 3rd party)
    3) custom scripts to run via Lambda functions (e.g. to empty s£ bucket)
  - Defined in template using **AWS::CloudFormation::CustomResource** or **Custom::MyCustomName**
    - Backed by a Lambda function or SNS topic (pass in arn in service token field)


- StackSets
  - Allows updates of multiple stacks across accounts/regions in single operation
  - StackSet updates all stacks in it
  - Can be applied to all accounts in AWS Organisation

### Personal notes
