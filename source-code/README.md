# Café Lab CloudFormation Templates

This folder contains all the CloudFormation template files used in the Challenge Lab: Automating Infrastructure Deployment.

## Files Overview

### 1. **S3.yaml** - Initial S3 Template
- **Purpose:** Basic S3 bucket creation
- **Resources:** Single S3 bucket
- **Use Case:** First challenge task to understand CloudFormation basics
- **Status:** Starting point, minimal configuration

### 2. **S3-final.yaml** - Enhanced S3 Template
- **Purpose:** S3 bucket configured for static website hosting
- **Resources:** S3 bucket with static website configuration
- **Features:**
  - DeletionPolicy: Retain (preserves bucket on stack deletion)
  - Static website hosting enabled
  - index.html as index document
  - CloudFormation output with website URL
- **Use Case:** Final version after updating the stack
- **Status:** Production-ready for static websites

### 3. **cafe-network.yaml** - Network Layer Template
- **Purpose:** Create VPC and networking infrastructure
- **Resources:**
  - VPC (10.0.0.0/16 CIDR)
  - Internet Gateway (IGW)
  - Public Subnet (10.0.0.0/24 CIDR)
  - Public Route Table
  - Route to Internet Gateway
  - Subnet-to-Route Table association
- **Outputs:**
  - PublicSubnet ID (exported for app stack)
  - VPC ID (exported for app stack)
- **Stack Name:** update-cafe-network
- **Region:** us-east-1 (primary), us-west-2 (secondary)
- **Status:** Foundation for application deployment

### 4. **cafe-app.yaml** - Application Layer Template
- **Purpose:** Deploy EC2 web server with café application
- **Parameters:**
  - LatestAmiId: Automatic lookup of latest Amazon Linux 2 AMI
  - CafeNetworkParameter: Reference to network stack name (default: update-cafe-network)
  - InstanceTypeParameter: Flexible instance type selection (default: t2.small)
- **Resources:**
  - EC2 Instance (CafeInstance)
    - Uses SSM Parameter for latest AMI
    - Public IP address assigned
    - Deployed in public subnet from network stack
  - Security Group (CafeSG)
    - Allows HTTP (port 80) from anywhere
    - Allows SSH (port 22) from anywhere
- **Mappings:**
  - RegionMap: Region-specific key pairs
    - us-east-1: vockey
    - us-west-2: cafe-oregon
- **UserData:**
  - Updates system packages
  - Installs Apache HTTP Server
  - Installs MariaDB database
  - Installs PHP and LAMP stack
  - Downloads and executes cafe-app.sh script
- **Outputs:**
  - WebServerPublicIP: Public IP of web server
- **Stack Name:** update-cafe-app
- **Status:** Application deployment ready

## Deployment Workflow

```
┌─────────────────────────────────────────────────┐
│ 1. S3.yaml → CreateBucket Stack                │
│    (Static website files)                       │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│ 2. cafe-network.yaml → update-cafe-network      │
│    Stack (via CodePipeline)                     │
│    Creates: VPC, Subnet, IGW, Routes           │
│    Exports: SubnetID, VpcID                     │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│ 3. cafe-app.yaml → update-cafe-app Stack       │
│    (via CodePipeline)                           │
│    Creates: EC2 Instance, Security Group       │
│    Uses: Exported values from network stack    │
│    Runs: Café application on EC2               │
└─────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────┐
│ 4. Multi-Region Replication                    │
│    Deploy same templates to us-west-2          │
│    Uses different key pair (cafe-oregon)       │
│    Optional instance type override (t3.micro)  │
└─────────────────────────────────────────────────┘
```

## Key CloudFormation Concepts Used

### Intrinsic Functions
- `!Ref` - Reference parameters and resources
- `!GetAtt` - Get resource attributes
- `!Sub` - String substitution
- `!ImportValue` - Import exported stack values
- `!Select` - Select from lists
- `!GetAZs` - Get availability zones
- `!FindInMap` - Lookup from mappings

### Cross-Stack References
- **Exports:** Network stack exports SubnetID and VpcID
- **Imports:** App stack imports values for subnet and VPC placement

### Parameters
- Allow flexible deployment without template modification
- Instance type selection enables different resource sizes
- CafeNetworkParameter enables referencing different network stacks

### Mappings
- Region-specific key pairs (vockey, cafe-oregon)
- Enables same template deployment across regions

### Dependencies
- Explicit DependsOn declarations ensure proper resource creation order
- CloudFormation automatically detects dependencies from references

## Deployment Commands

### Deploy via CloudFormation CLI

```bash
# Network Stack
aws cloudformation create-stack \
  --stack-name update-cafe-network \
  --template-body file://cafe-network.yaml \
  --region us-east-1

# Application Stack
aws cloudformation create-stack \
  --stack-name update-cafe-app \
  --template-body file://cafe-app.yaml \
  --region us-east-1

# Deploy to secondary region (us-west-2)
aws cloudformation create-stack \
  --stack-name update-cafe-network \
  --template-body file://cafe-network.yaml \
  --region us-west-2
```

### Deploy via CodePipeline (Automated)

```bash
# Push templates to CodeCommit
git add templates/cafe-network.yaml
git commit -m 'Deploy network stack'
git push

# Pipeline automatically triggers deployment
```

## Template Validation

```bash
# Validate template syntax
aws cloudformation validate-template --template-body file://cafe-network.yaml
aws cloudformation validate-template --template-body file://cafe-app.yaml
```

## Stack Updates

```bash
# Update existing stack
aws cloudformation update-stack \
  --stack-name update-cafe-network \
  --template-body file://cafe-network.yaml

# Update stack via Git push (CodePipeline)
git add cafe-app.yaml
git commit -m 'Update application stack'
git push
```

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| Template validation error | Check YAML indentation (2 spaces) and colons |
| ImportValue not found | Ensure network stack created first and exports exist |
| EC2 instance not starting | Wait 2-3 minutes for UserData script to complete |
| Website returns 403 | Check S3 bucket public access settings |
| Stack rollback | Check Events tab for first failure reason |

### Debugging Commands

```bash
# View stack events
aws cloudformation describe-stack-events \
  --stack-name update-cafe-network

# View stack resources
aws cloudformation list-stack-resources \
  --stack-name update-cafe-app

# View stack outputs
aws cloudformation describe-stacks \
  --stack-name update-cafe-network \
  --query 'Stacks[0].Outputs'
```

## Best Practices Demonstrated

✅ **Infrastructure as Code** - All infrastructure defined in templates
✅ **Modularity** - Separate network and application stacks
✅ **Reusability** - Same templates deployed across regions
✅ **Version Control** - Templates stored in CodeCommit
✅ **CI/CD Automation** - CodePipeline for automated deployment
✅ **Cross-Stack References** - Exports/Imports for stack communication
✅ **Parameter Flexibility** - Allow deployment customization
✅ **Region Flexibility** - Mappings enable multi-region deployment
✅ **Documentation** - Clear resource naming and tagging
✅ **Security** - Security groups restrict access appropriately

## Related Documentation

- [Main Lab Documentation](../README.md)
- [AWS CloudFormation User Guide](https://docs.aws.amazon.com/cloudformation/)
- [AWS Resource Types Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)

## File Statistics

| File | Lines | Resources | Purpose |
|------|-------|-----------|---------|
| S3.yaml | 6 | 1 (S3 Bucket) | Basic template example |
| S3-final.yaml | 13 | 1 (S3 Bucket) | Enhanced with website hosting |
| cafe-network.yaml | 87 | 6 (VPC, IGW, Subnet, Routes) | Network infrastructure |
| cafe-app.yaml | 89 | 2 (EC2, Security Group) | Application deployment |

## License & Attribution

These templates are part of the AWS Challenge Lab curriculum for infrastructure automation training.

**Created:** June 9, 2026  
**Repository:** uriel25x/Challenge-Café-lab-Automating-Infrastructure-Deployment-Codes
