# AWS Multi-Tier Web Application Infrastructure as Code (CloudFormation)

## Project Overview

This project provides a comprehensive set of CloudFormation templates for deploying a secure, highly available three-tier web application architecture across multiple AWS availability zones.
The infrastructure-as-code approach enables rapid, consistent deployment of production-ready infrastructure while reducing costs through auto-scaling, improving reliability with multi-AZ architecture, and enhancing security through defense-in-depth principles.

This repository contains CloudFormation templates for deploying a multi-tier web application infrastructure on AWS.

## AWS Architecture Overview

### Foundation

- **VPC (Virtual Private Cloud)**: A private network for your AWS resources
- **Subnets**: Divides your network into public and private sections across multiple zones for reliability
- **Internet Gateway**: Allows resources to connect to the internet

![VPC Architecture](public/images/vpc.png)

### Web Layer

- **Load Balancer**: Distributes traffic to multiple servers
- **Auto Scaling Group**: Automatically adds or removes servers based on demand
- **S3 Bucket**: Stores static website files like HTML, CSS, and images

### Application Layer

- **EC2 Instances**: Virtual servers that run your application code
- **Security Groups**: Act as firewalls controlling traffic between components

### Data Layer

- **RDS Database**: Managed MySQL database for storing application data

### Security

- **IAM Roles**: Controls who can access what resources

## Business Value

- **Scalability**: Handles varying traffic levels automatically
- **High Availability**: Runs across multiple zones to prevent downtime
- **Cost Efficiency**: Only pays for resources when needed
- **Security**: Isolates sensitive components in private networks

## Template Documentation

### Network Infrastructure

- **vpc.yaml**

  - Creates a VPC with CIDR 172.16.0.0/16
  - Deploys 6 subnets across 2 availability zones:
    - Public subnets for web/load balancer tier
    - App private subnets for application tier
    - Data private subnets for database tier
  - Configures Internet Gateway and route tables for internet access
  - Sets up a Bastion host in public subnet for secure SSH access
  - Deploys application instances in private subnets with appropriate security groups
  - Associates route tables with both public subnets

### Compute Resources

- **ec2.yaml**

  - Provisions EC2 instances with Amazon Linux AMI
  - Configures security groups for appropriate access
  - Sets up SSH key access configuration

- **asg.yaml**
  - Creates an Auto Scaling Group with:
    - Launch template for EC2 instances
    - Scaling policies based on CPU utilization
    - CloudWatch alarms for scaling triggers
    - Target tracking for maintaining instance count

### Storage Resources

- **s3-bucket.yaml**

  - Creates a basic S3 bucket with appropriate properties
  - Configures bucket access controls and lifecycle policies

- **s3-static.yaml**
  - Sets up S3 bucket configured for static website hosting
  - Enables website endpoint configuration
  - Works with the index.html file in this repository

### Database Resources

- **rds.yaml**
  - Provisions a MySQL RDS database instance
  - Configures multi-AZ deployment for high availability
  - Sets up security groups for controlled database access
  - Manages database parameters and settings

### Identity and Access Management

- **iam.yaml**
  - Defines IAM users, groups, roles, and policies
  - Establishes service roles for AWS resources
  - Sets up instance profiles for EC2 instances
  - Configures proper permission boundaries

## Usage

Each template can be deployed independently or combined for a complete architecture:

```bash
# Deploy VPC infrastructure
aws cloudformation create-stack --stack-name vpc-stack --template-body file://vpc.yaml

# Deploy database
aws cloudformation create-stack --stack-name database-stack --template-body file://rds.yaml

# Deploy web hosting (choose one)
aws cloudformation create-stack --stack-name static-site --template-body file://s3-static.yaml
# OR
aws cloudformation create-stack --stack-name dynamic-site --template-body file://asg.yaml
```

The templates support both static websites hosted on S3 and dynamic applications with database backends.

## Notes

### Subnet Naming Convention

The template follows a consistent naming convention for subnets:

#### AZ 1 (First Availability Zone)

- **PublicSubnet1A**: Public subnet in first AZ
- **AppPrivateSubnet1A**: Application tier private subnet in first AZ
- **DataPrivateSubnet1A**: Database tier private subnet in first AZ

#### AZ 2 (Second Availability Zone)

- **PublicSubnet2B**: Public subnet in second AZ
- **AppPrivateSubnet2B**: Application tier private subnet in second AZ
- **DataPrivateSubnet2B**: Database tier private subnet in second AZ

This naming scheme clearly indicates both the subnet's purpose and its availability zone location.

## Accessing Instances

The architecture follows a secure cascading access pattern:

1. Copy the bastion.pem key to the Bastion host:

   ```
   scp -i bastion.pem bastion.pem ec2-user@BASTION_PUBLIC_IP:~/
   ```

2. SSH from Bastion to App1 (management server) using its private IP:

   ```
   ssh -i bastion.pem ec2-user@APP1_PRIVATE_IP
   ```

3. From App1, ping App2 (application server) to verify connectivity:
   ```
   ping APP2_PRIVATE_IP
   ```

This secure design isolates application servers behind multiple security layers.
