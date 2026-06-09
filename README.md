# Challenge Lab: Automating Infrastructure Deployment

## Table of Contents
- [Overview](#overview)
- [Learning Objectives](#learning-objectives)
- [Lab Scenario](#lab-scenario)
- [Prerequisites & Setup](#prerequisites--setup)
- [Lab Architecture](#lab-architecture)
- [Challenge Tasks](#challenge-tasks)
  - [Challenge 1: Static Website with CloudFormation](#challenge-1-static-website-with-cloudformation)
  - [Challenge 2: Version Control with CodeCommit](#challenge-2-version-control-with-codecommit)
  - [Challenge 3: CI/CD Pipeline & Dynamic Website](#challenge-3-cicd-pipeline--dynamic-website)
  - [Challenge 4: Multi-Region Deployment](#challenge-4-multi-region-deployment)
- [Key Concepts](#key-concepts)
- [Troubleshooting Guide](#troubleshooting-guide)
- [Additional Resources](#additional-resources)

---

## Overview

This lab guides you through automating AWS infrastructure deployment using industry best practices. You'll progress from manually creating resources to fully automating deployments across multiple AWS regions using Infrastructure as Code (IaC) and Continuous Integration/Continuous Delivery (CI/CD) principles.

**Duration:** ~90 minutes  
**Difficulty Level:** Intermediate to Advanced  
**AWS Services:** CloudFormation, CodeCommit, CodePipeline, EC2, VPC, S3, IAM

---

## Learning Objectives

Upon completing this lab, you will be able to:

1. ✅ Deploy a virtual private cloud (VPC) networking layer using CloudFormation templates
2. ✅ Deploy an application layer using CloudFormation templates
3. ✅ Use Git with CodeCommit to version control and deploy CloudFormation templates
4. ✅ Implement a CI/CD pipeline using CodePipeline for automated stack updates
5. ✅ Duplicate network and application resources across multiple AWS regions
6. ✅ Configure EC2 instances with proper security groups, IAM roles, and user data scripts
7. ✅ Implement outputs and exports for cross-stack resource references

---

## Lab Scenario

**The Café Context:**

The café staff has been manually managing AWS resources through the AWS Management Console. While this worked initially, they now face challenges:
- Difficulty replicating deployments to new AWS regions for international expansion
- Inconsistent configurations between development and production environments
- Manual processes that are time-consuming and error-prone

**Your Role (Sofía):**

You'll take on the role of Sofía, an infrastructure engineer tasked with:
1. Creating CloudFormation templates for infrastructure automation
2. Implementing version control using AWS CodeCommit
3. Setting up CI/CD pipelines with CodePipeline
4. Deploying the café application across multiple regions

---

## Prerequisites & Setup

### Required AWS Services Access
- AWS CloudFormation
- AWS CodeCommit
- AWS CodePipeline
- Amazon EC2
- Amazon VPC
- Amazon S3
- AWS Systems Manager (Parameter Store)
- AWS IAM

### Pre-Created Resources
The lab environment includes:
- CloudFormation pipelines (CafeNetworkPipeline, CafeAppPipeline)
- CodeCommit repository (CFTemplatesRepo)
- IAM roles and policies
- EC2 key pairs for specific regions

### Development Environment
- VS Code IDE (code-server) on EC2 instance
- Git client pre-installed
- AWS CLI pre-configured with appropriate credentials
- Template files for reference

---

## Lab Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS Account                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           CodeCommit Repository                      │   │
│  │        (CFTemplatesRepo with Git)                    │   │
│  │  ┌─ cafe-network.yaml                               │   │
│  │  ├─ cafe-app.yaml                                   │   │
│  │  ├─ template1.yaml (reference)                      │   │
│  │  └─ template2.yaml (reference)                      │   │
│  └──────────────────────────────────────────────────────┘   │
│           ▲                      ▲                           │
│           │ (Git Push)           │ (Git Push)               │
│           │                      │                          │
│  ┌────────┴──────────┐  ┌────────┴─────────────┐           │
│  │  CafeNetworkPip   │  │   CafeAppPipeline    │           │
│  │    (CodePipeline) │  │   (CodePipeline)     │           │
│  └────────┬──────────┘  └────────┬─────────────┘           │
│           │                      │                          │
│           ▼                      ▼                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │          AWS CloudFormation                          │   │
│  │  ┌─ update-cafe-network Stack                       │   │
│  │  └─ update-cafe-app Stack                           │   │
│  └────────┬─────────────────────────┬──────────────────┘   │
│           │                         │                      │
│           ▼                         ▼                      │
│  ┌──────────────────────┐  ┌──────────────────────────┐   │
│  │  Network Layer       │  │  Application Layer       │   │
│  │  ├─ VPC             │  │  ├─ EC2 Instance         │   │
│  │  ├─ Public Subnet   │  │  ├─ Security Group       │   │
│  │  ├─ Route Table     │  │  └─ IAM Role             │   │
│  │  └─ IGW             │  │                          │   │
│  └──────────────────────┘  └──────────────────────────┘   │
│                                                              │
│  Multi-Region Deployment:                                  │
│  us-east-1 (N. Virginia) ← REPLICATED TO → us-west-2      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Challenge Tasks

### Challenge 1: Static Website with CloudFormation

**Objective:** Create a CloudFormation template that provisions and configures an S3 bucket to host a static website.

#### Task 1.1: Connect to the IDE
1. Access the AWS Details section to obtain:
   - LabIDEURL
   - LabIDEPassword
2. Open VS Code IDE in a new browser tab
3. Authenticate with the provided password

#### Task 1.2: Create S3 CloudFormation Template

Create a new file `S3.yaml` with the following structure:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "cafe S3 template"

Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
```

**Key Points:**
- YAML indentation is critical (2 spaces per level)
- Resources section has no indentation
- Resource names indented by 2 spaces
- Properties indented by 4 spaces

#### Task 1.3: Deploy the Stack

```bash
# Check default region
aws configure get region

# Create the stack
aws cloudformation create-stack --stack-name CreateBucket --template-body file://S3.yaml
```

**Verification:**
- Check CloudFormation console for CreateBucket stack
- Verify S3 bucket created with auto-generated name
- S3 bucket name format: `createbucket-s3bucket-<random-string>`

#### Task 1.4: Update Template for Website Hosting

Enhance the template with:

1. **Deletion Policy:** Retain bucket on stack deletion
2. **Static Website Hosting:** Configure index.html as index document
3. **Outputs:** Export website URL

```yaml
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      WebsiteConfiguration:
        IndexDocument: index.html

Outputs:
  WebsiteURL:
    Description: URL of the S3 website
    Value: !GetAtt S3Bucket.WebsiteURL
```

#### Task 1.5: Upload Website Content

```bash
# Download website files
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/15-lab-mod11-challenge-CFn/s3/static-website.zip
unzip static-website.zip -d static
cd static

# Configure bucket ownership
aws s3api put-bucket-ownership-controls --bucket <BUCKET-NAME> \
  --ownership-controls Rules=[{ObjectOwnership=BucketOwnerPreferred}]

# Set public access settings
aws s3api put-public-access-block --bucket <BUCKET-NAME> \
  --public-access-block-configuration \
  "BlockPublicAcls=false,RestrictPublicBuckets=false,IgnorePublicAcls=false,BlockPublicPolicy=false"

# Upload website files
aws s3 cp --recursive . s3://<BUCKET-NAME>/ --acl public-read
```

#### Task 1.6: Update Stack

```bash
# Validate template
aws cloudformation validate-template --template-body file://S3.yaml

# Update the stack
aws cloudformation update-stack --stack-name CreateBucket --template-body file://S3.yaml
```

**Expected Results:**
- Stack status: UPDATE_COMPLETE
- CloudFormation Outputs tab displays website URL
- Accessing URL shows café website

**Answers to Questions:**
- Q1: Yes, bucket created with auto-generated name (createbucket-s3bucket-*)
- Q2: Same region as AWS CLI default (usually us-east-1)
- Q3: Minimum 3 lines (AWSTemplateFormatVersion, Description, Resources section)

---

### Challenge 2: Version Control with CodeCommit

**Objective:** Store CloudFormation templates in a version-controlled repository for team collaboration.

#### Task 2.1: Access CodeCommit Repository

1. Navigate to CodeCommit console
2. Locate repository: `CFTemplatesRepo`
3. Clone the repository URL (HTTPS GRC)

#### Task 2.2: Clone Repository

```bash
# Clone the repository
git clone <CLONE-URL>

# Navigate to cloned directory
cd CFTemplatesRepo

# Check status
git status
```

**Expected Output:**
```
On branch main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean
```

#### Task 2.3: Git Workflow Basics

```bash
# Check repository status
git status

# Stage changes
git add templates/cafe-network.yaml

# Commit changes
git commit -m 'initial commit of network template' templates/cafe-network.yaml

# Check updated status (should show "ahead by 1 commit")
git status

# Push to remote
git push
```

**Key Git Commands:**
- `git status` - View tracked/untracked files and branch status
- `git add <file>` - Stage file for commit
- `git commit -m "<message>"` - Commit staged changes
- `git push` - Push commits to remote repository
- `git log` - View commit history

---

### Challenge 3: CI/CD Pipeline & Dynamic Website

**Objective:** Use CodePipeline to automatically deploy infrastructure updates when templates are pushed to CodeCommit.

#### Task 3.1: Analyze Pre-Configured Pipelines

**CafeNetworkPipeline:**
- Source: CodeCommit (CFTemplatesRepo)
- Source file: `templates/cafe-network.yaml`
- Deploy: CloudFormation (stack name: `update-cafe-network`)
- Action: RunChangeSet + ExecuteChangeSet

**CafeAppPipeline:**
- Source: CodeCommit (CFTemplatesRepo)
- Source file: `templates/cafe-app.yaml`
- Deploy: CloudFormation (stack name: `update-cafe-app`)

#### Task 3.2: Create Network Layer Template

Create `templates/cafe-network.yaml`:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Network layer for the cafe"

Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: Cafe VPC

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs ""]
      Tags:
        - Key: Name
          Value: Cafe Public Subnet

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: Cafe IGW

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: Cafe Public RT

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  SubnetRouteTableAssociation:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet
      RouteTableId: !Ref PublicRouteTable

Outputs:
  PublicSubnet:
    Description: The subnet ID to use for public web servers
    Value: !Ref PublicSubnet
    Export:
      Name: !Sub '${AWS::StackName}-SubnetID'
  
  VpcId:
    Description: The VPC ID
    Value: !Ref VPC
    Export:
      Name: !Sub '${AWS::StackName}-VpcID'
```

#### Task 3.3: Commit and Deploy Network Stack

```bash
# Add, commit, and push template
git add templates/cafe-network.yaml
git commit -m 'initial commit of network template' templates/cafe-network.yaml
git push

# Monitor pipeline execution
# Visit CodePipeline console and observe CafeNetworkPipeline
# Verify Source stage → Deploy stage transitions
# Check CloudFormation console for update-cafe-network stack
```

**Verification:**
- Pipeline Source stage: Succeeded
- Pipeline Deploy stage: Succeeded
- CloudFormation stack: CREATE_COMPLETE
- VPC console shows: Cafe VPC, Cafe Public Subnet
- Stack Outputs tab shows exported values

#### Task 3.4: Create Application Layer Template

Create `templates/cafe-app.yaml`:

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Application layer for the cafe"

Parameters:
  LatestAmiId:
    Type: AWS::EC2::Image::Id
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2

  CafeNetworkParameter:
    Type: String
    Default: update-cafe-network
    Description: The CloudFormation stack name for the network layer

  InstanceTypeParameter:
    Type: String
    Default: t2.small
    AllowedValues:
      - t2.micro
      - t2.small
      - t3.micro
      - t3.small
    Description: EC2 instance type for the web server

Mappings:
  RegionMap:
    us-east-1:
      keypair: vockey
    us-west-2:
      keypair: cafe-oregon

Resources:
  CafeSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security group for cafe web server
      VpcId: !ImportValue
        Fn::Sub: '${CafeNetworkParameter}-VpcID'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
      Tags:
        - Key: Name
          Value: Cafe Security Group

  CafeInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref LatestAmiId
      InstanceType: !Ref InstanceTypeParameter
      KeyName: !FindInMap [RegionMap, !Ref "AWS::Region", keypair]
      IamInstanceProfile: CafeRole
      NetworkInterfaces:
        - DeviceIndex: '0'
          AssociatePublicIpAddress: 'true'
          SubnetId: !ImportValue
            Fn::Sub: '${CafeNetworkParameter}-SubnetID'
          GroupSet:
            - !Ref CafeSG
      Tags:
        - Key: Name
          Value: Cafe Web Server
      UserData:
        Fn::Base64:
          !Sub |
            #!/bin/bash
            yum -y update
            yum install -y httpd mariadb-server wget
            amazon-linux-extras install -y lamp-mariadb10.2-php7.2 php7.2
            systemctl enable httpd
            systemctl start httpd
            systemctl enable mariadb
            systemctl start mariadb
            wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-200-ACACAD-3-113230/15-lab-mod11-challenge-CFn/s3/cafe-app.sh
            chmod +x cafe-app.sh
            ./cafe-app.sh

Outputs:
  WebServerPublicIP:
    Description: Public IP address of the web server
    Value: !GetAtt CafeInstance.PublicIp
```

#### Task 3.5: Validate and Deploy Application

```bash
# Validate template
aws cloudformation validate-template --template-body file://templates/cafe-app.yaml

# Add, commit, and push
git add templates/cafe-app.yaml
git commit -m 'initial commit of app template' templates/cafe-app.yaml
git push

# Monitor CafeAppPipeline in CodePipeline console
# Verify update-cafe-app stack creation
```

**Verification:**
- CafeAppPipeline: Source Succeeded → Deploy Succeeded
- CloudFormation stack: CREATE_COMPLETE
- EC2 instance created with proper configuration
- Access website: `http://<public-ip-address>/cafe`
- Website displays café branding with server information (Region, AZ)

**Question Answers:**
- Q4: LatestAmiId shows resolved AMI ID (e.g., ami-xxxxxxxxx)
- Q5: Stack ServiceRole ARN in Stack Info
- Q6: Commits show hash, author, timestamp, and commit messages

---

### Challenge 4: Multi-Region Deployment

**Objective:** Replicate infrastructure across multiple AWS regions to demonstrate scalability and disaster recovery capabilities.

#### Task 4.1: Deploy Network Stack to us-west-2

```bash
# Create network stack in Oregon region
aws cloudformation create-stack \
  --stack-name update-cafe-network \
  --template-body file://templates/cafe-network.yaml \
  --region us-west-2
```

**Verification:**
- Switch region to us-west-2 in CloudFormation console
- Verify stack: CREATE_COMPLETE
- VPC console shows network resources in us-west-2

#### Task 4.2: Create EC2 Key Pair for us-west-2

```bash
# In EC2 console, switch to us-west-2
# Create key pair named "cafe-oregon"
# Note: Template RegionMap references this key pair
```

#### Task 4.3: Deploy Application to us-west-2

1. Upload cafe-app.yaml to S3:
```bash
aws s3 cp templates/cafe-app.yaml s3://<repobucket-name>/
```

2. Copy S3 object URL

3. In CloudFormation console (us-west-2):
   - Create stack
   - Use S3 URL from clipboard
   - Stack name: Choose a name
   - Instance Type: t3.micro
   - Accept defaults for other parameters

**Result:**
- Application stack created in us-west-2
- EC2 instance uses cafe-oregon key pair (from RegionMap)
- Instance type: t3.micro (parameter override)
- Website accessible at: `http://<public-ip>/cafe`
- Server info shows: us-west-2 region

#### Task 4.4: Compare Multi-Region Deployment

Access café websites in both regions:
- **us-east-1:** `http://<east-ip>/cafe` - Shows N. Virginia region
- **us-west-2:** `http://<west-ip>/cafe` - Shows Oregon region

**Key Observations:**
- Same CloudFormation template deployed to different regions
- Template automatically adapts using parameters and mappings
- Different instance types can be specified per deployment
- Key pairs region-specific (vockey vs cafe-oregon)
- Both running identical café application

---

## Key Concepts

### CloudFormation Templates

**Template Structure:**
```yaml
AWSTemplateFormatVersion: "2010-09-09"  # Version (required)
Description: String                      # Template description
Metadata: Object                         # Additional metadata
Parameters: Object                       # Input parameters
Mappings: Object                         # Fixed key-value pairs
Conditions: Object                       # Conditional logic
Resources: Object                        # AWS resources (required)
Outputs: Object                          # Return values
```

**Key Features:**

| Feature | Purpose | Example |
|---------|---------|---------|
| **Parameters** | User-provided inputs | Instance type, key pair name |
| **Mappings** | Fixed lookup tables | Region-specific values |
| **Resources** | AWS infrastructure | EC2, VPC, S3, etc. |
| **Outputs** | Export values | VPC ID, subnet ID, public IP |
| **Exports** | Cross-stack references | Share values between stacks |
| **Conditions** | Conditional resource creation | Deploy based on parameters |
| **Metadata** | Additional template info | CloudFormation init configs |

### Infrastructure as Code (IaC)

**Benefits:**
- **Versioning** - Track all infrastructure changes in Git
- **Replication** - Deploy identical environments quickly
- **Consistency** - Eliminate manual configuration drift
- **Automation** - Reduce manual errors and deployment time
- **Disaster Recovery** - Rapidly recreate infrastructure
- **Testing** - Create/destroy test environments on demand

### CI/CD Pipeline

**Components:**
1. **Source Stage** - Monitors CodeCommit repository
2. **Deploy Stage** - Invokes CloudFormation
   - Creates ChangeSets (preview changes)
   - Executes ChangeSets (applies changes)

**Workflow:**
```
Git Push → CodeCommit → CodePipeline → CloudFormation → AWS Resources
```

### CloudFormation Intrinsic Functions

```yaml
# Reference parameters and resources
!Ref ResourceName
!Ref ParameterName

# Get attribute values
!GetAtt Resource.Attribute

# Substitute string values
!Sub '${VariableName}-suffix'

# Import exported values
!ImportValue ExportName

# Select from list
!Select [Index, List]

# Get AZs in region
!GetAZs ""

# Map lookup
!FindInMap [MapName, TopKey, SecondKey]

# Join list values
!Join [Delimiter, [List, Of, Values]]

# If conditional
!If [ConditionName, ValueIfTrue, ValueIfFalse]
```

### YAML Best Practices

1. **Indentation** - Use 2 spaces (never tabs)
2. **Colons** - Must be followed by space
3. **Lists** - Use dash with space prefix
4. **Quotes** - Use for strings with special characters
5. **Comments** - Start with # and space

### Git Workflow

```bash
# Stage changes
git add <file>

# Commit with descriptive message
git commit -m "<descriptive message>"

# Push to remote
git push

# Check status
git status

# View log
git log

# Pull latest
git pull
```

---

## Troubleshooting Guide

### CloudFormation Stack Creation/Update Failures

**Problem:** Stack shows ROLLBACK_COMPLETE status

**Solution:**
1. Navigate to Events tab
2. Look for first UPDATE_FAILED entry
3. Read Status Reason for error details
4. Correct template syntax
5. Rerun stack update or delete and recreate

**Common Issues:**
- **YAML Syntax Error** - Check indentation and colons
- **Invalid Property** - Verify property name in documentation
- **Missing Required Property** - Check AWS resource documentation
- **Invalid Reference** - Ensure parameter/resource names are correct

### Pipeline Deployment Failures

**Problem:** CafeNetworkPipeline shows Failed status

**Solutions:**
1. Check Source stage - Verify template file is in correct location
2. Check Deploy stage - Review CloudFormation errors
3. Fix template and push again: `git commit && git push`
4. Or manually retry: Pipeline → Release change button

**File Location Requirements:**
- Network template: `templates/cafe-network.yaml`
- App template: `templates/cafe-app.yaml`
- CodeCommit repository: `CFTemplatesRepo`

### EC2 Instance Issues

**Problem:** Website not loading after EC2 instance starts

**Solutions:**
1. **Wait for full startup** - UserData script takes 2-3 minutes
2. **Check security group** - Verify port 80 is open to 0.0.0.0/0
3. **Check public IP** - Confirm instance has public IP address
4. **Check website** - Verify index.html and cafe-app.sh downloaded successfully
5. **SSH to instance** - Debug application issues:
   ```bash
   ssh -i <key-pair> ec2-user@<public-ip>
   sudo tail -f /var/log/cloud-init-output.log
   ```

### Git/CodeCommit Issues

**Problem:** Cannot push to CodeCommit

**Solutions:**
1. **Check Git configuration:**
   ```bash
   git config --list
   ```
2. **Verify remote URL:**
   ```bash
   git remote -v
   ```
3. **Check credentials** - Ensure AWS CLI credentials are valid
4. **Check branch** - Ensure on correct branch (usually main)

**Problem:** Files appear untracked after commit

**Solution:**
```bash
# Verify files are staged
git status

# Add specific files
git add templates/cafe-network.yaml

# Try commit again
git commit -m "message"
```

### S3 Website Hosting Issues

**Problem:** Website returns 403 Forbidden

**Solutions:**
1. Check bucket ownership controls - Must be BucketOwnerPreferred
2. Check public access settings - Must allow public read
3. Check object permissions - Objects must have public-read ACL
4. Verify index.html exists in bucket root

### Cross-Stack References Issues

**Problem:** ImportValue fails to find export

**Solutions:**
1. Verify export name matches exactly (case-sensitive)
2. Confirm network stack created first (exports must exist)
3. Verify both stacks in same region
4. Check stack names match parameter defaults
5. Review Outputs tab of source stack

---

## Additional Resources

### AWS Documentation
- [AWS CloudFormation User Guide](https://docs.aws.amazon.com/cloudformation/)
- [AWS CloudFormation Resource Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [AWS CodeCommit User Guide](https://docs.aws.amazon.com/codecommit/)
- [AWS CodePipeline User Guide](https://docs.aws.amazon.com/codepipeline/)
- [AWS EC2 User Guide](https://docs.aws.amazon.com/ec2/)

### Reference Templates

**Minimal S3 Static Website:**
```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Static website S3 bucket"
Resources:
  S3Bucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
    Properties:
      WebsiteConfiguration:
        IndexDocument: index.html
Outputs:
  WebsiteURL:
    Value: !GetAtt S3Bucket.WebsiteURL
```

**Minimal VPC with Public Subnet:**
```yaml
Resources:
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
  
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
```

### Git Quick Reference

```bash
# Initial setup
git clone <url>
cd <repo>

# Daily workflow
git status              # Check changes
git add <file>         # Stage changes
git commit -m "msg"    # Commit locally
git push               # Push to remote
git pull               # Get latest

# Troubleshooting
git log                # View history
git diff               # See changes
git reset <file>       # Unstage file
git checkout <file>    # Discard changes
```

### YAML Syntax Reference

```yaml
# Comments start with hash
# Simple value
key: value

# Indented child
parent:
  child: value
  another: value

# List
items:
  - item1
  - item2
  - item3

# Multiline string
description: |
  This is a multiline
  string value

# Reference
reference: !Ref SomeResource

# Get attribute
attribute: !GetAtt Resource.Attribute

# Substitution
substituted: !Sub '${AWS::StackName}-suffix'
```

---

## Lab Completion Checklist

✅ **Challenge 1: Static Website**
- [ ] Created S3.yaml template
- [ ] Deployed CreateBucket stack
- [ ] Configured static website hosting
- [ ] Uploaded website files
- [ ] Updated stack successfully
- [ ] Website accessible via S3 URL

✅ **Challenge 2: Version Control**
- [ ] Cloned CFTemplatesRepo
- [ ] Understood Git workflow
- [ ] Created cafe-network.yaml
- [ ] Committed and pushed to CodeCommit

✅ **Challenge 3: CI/CD Pipeline**
- [ ] Created network layer template with VPC, subnet, IGW
- [ ] Added outputs with exports
- [ ] Analyzed CafeNetworkPipeline
- [ ] Verified network stack deployment
- [ ] Created application layer template
- [ ] Analyzed CafeAppPipeline
- [ ] Verified application stack deployment
- [ ] Accessed café website in us-east-1

✅ **Challenge 4: Multi-Region**
- [ ] Created network stack in us-west-2
- [ ] Created EC2 key pair for us-west-2
- [ ] Created application stack in us-west-2
- [ ] Verified dual-region deployment
- [ ] Accessed café website in us-west-2

---

## Summary & Key Takeaways

**What You Learned:**
1. CloudFormation enables infrastructure automation
2. Templates can be versioned in Git repositories
3. CI/CD pipelines automate stack deployments
4. Same templates can deploy to multiple regions
5. Parameters and mappings enable flexible deployments
6. Outputs and exports enable cross-stack communication

**Benefits Realized:**
- Eliminated manual resource creation
- Reduced deployment time significantly
- Enabled rapid regional expansion
- Ensured consistent configurations
- Created automated disaster recovery capability
- Established DevOps best practices

**Next Steps:**
- Implement additional CloudFormation features (conditions, custom resources)
- Explore AWS CodeBuild for custom deployment logic
- Implement stack policies and change sets
- Create custom Lambda-backed resources
- Implement infrastructure monitoring and alerting

---

**Lab Documentation Created:** June 9, 2026  
**Repository:** [uriel25x/Challenge-Café-lab-Automating-Infrastructure-Deployment-Codes](https://github.com/uriel25x/Challenge-Caf-lab-Automating-Infrastructure-Deployment-Codes)
