# Day 61 – Introduction to Terraform & Infrastructure as Code (IaC)

## 📌 Objective

The objective of this lab was to understand the fundamentals of **Infrastructure as Code (IaC)** using **Terraform**, configure AWS credentials, provision cloud resources, understand Terraform state management, and learn the complete Terraform workflow from creating to destroying infrastructure.

---

# Task 1 – Understanding Infrastructure as Code (IaC)

## What is Infrastructure as Code (IaC)?

Infrastructure as Code (IaC) is the practice of managing and provisioning infrastructure using configuration files instead of manually creating resources through a cloud provider's web console.

With IaC, infrastructure becomes version-controlled just like application code. Developers and DevOps engineers can automate deployments, reproduce environments consistently, and reduce manual errors.

---

## Why does IaC matter in DevOps?

Infrastructure as Code is one of the core principles of DevOps because it enables automation and consistency.

Benefits include:

- Eliminates manual infrastructure provisioning
- Infrastructure can be version controlled using Git
- Makes deployments repeatable and reliable
- Reduces human error
- Speeds up provisioning of cloud resources
- Makes disaster recovery easier
- Supports CI/CD automation

---

## Problems IaC Solves Compared to Manual AWS Console

Creating resources manually in AWS Console introduces several challenges:

| Manual AWS Console | Infrastructure as Code |
|-------------------|-----------------------|
| Time consuming | Automated |
| Human errors | Consistent deployments |
| Difficult to reproduce | Fully repeatable |
| No version history | Git version control |
| Hard to rollback | Easy rollback using Git |
| Manual documentation | Self-documenting code |

---

## Terraform vs Other Infrastructure Tools

| Tool | Purpose |
|------|---------|
| Terraform | Infrastructure Provisioning |
| AWS CloudFormation | AWS-only Infrastructure Provisioning |
| Ansible | Configuration Management |
| Pulumi | Infrastructure using Programming Languages |

### Terraform vs AWS CloudFormation

Terraform is cloud-agnostic and supports multiple cloud providers such as AWS, Azure, GCP, Oracle Cloud, and VMware.

CloudFormation only works with AWS services.

---

### Terraform vs Ansible

Terraform creates infrastructure like:

- EC2 Instances
- VPCs
- S3 Buckets
- Load Balancers

Ansible configures infrastructure after it has been created.

Example:

Terraform creates an EC2 instance.

Ansible installs Docker and Nginx on that EC2 instance.

---

### Terraform vs Pulumi

Terraform uses HCL (HashiCorp Configuration Language), which is simple and declarative.

Pulumi uses programming languages such as:

- Python
- JavaScript
- TypeScript
- Go
- C#

---

## What is Declarative?

Terraform follows a **Declarative** approach.

You only define the desired end state.

Example:

"I need one EC2 instance."

Terraform automatically determines how to create it.

---

## What is Cloud Agnostic?

Cloud agnostic means the same Terraform workflow works across different cloud providers by only changing the provider configuration.

Supported providers include:

- AWS
- Azure
- Google Cloud
- DigitalOcean
- Oracle Cloud
- VMware

---

# Task 2 – Installing Terraform & AWS CLI

## Install Terraform

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform

terraform -version
```

---

## Install AWS CLI

```bash
brew install awscli # for macOS
```

---

## Configure AWS

```bash
aws configure
```

Provide:

- AWS Access Key
- AWS Secret Key
- Default Region
- Output Format

---

## Verify AWS Access

```bash
aws sts get-caller-identity
```

This command verifies that AWS credentials are configured correctly.

---

## 📸 Screenshot 1

**Terraform Version**

<img width="1120" height="144" alt="image" src="https://github.com/user-attachments/assets/c5ee7e9b-abf7-456b-943d-b735df4fa1fe" />


---

## 📸 Screenshot 2

**AWS CLI Verification**

<img width="1198" height="252" alt="image" src="https://github.com/user-attachments/assets/387297cf-2225-4f2e-8e33-015f0b91de25" />


---

# Task 3 – Create Your First Terraform Configuration

## Create Project Directory

```bash
mkdir terraform-basics

cd terraform-basics
```

---

## Create main.tf

```bash
# Terraform block - specifies required providers
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

# Provider configuration - AWS region
provider "aws" {
  region = "eu-west-1" # Change to your region if different
}

# Resource: S3 Bucket
resource "aws_s3_bucket" "my_bucket" {
  bucket = "terraweek-sopatel-2026" # MUST be globally unique!

  tags = {
    Name        = "TerraWeek-Day1"
    Environment = "Dev"
  }
}


```

---

## Terraform Configuration

The configuration includes:

- AWS Provider
- S3 Bucket Resource

---

## Initialize Terraform

```bash
terraform init
```

### What happened?

Terraform downloaded the required AWS provider plugin and initialized the working directory.

---

## Preview Infrastructure

```bash
terraform plan
```

Terraform compares the desired infrastructure with the current state and displays what will change.

---

## Apply Configuration

```bash
terraform apply
```

Terraform created the S3 Bucket inside AWS.

---

## What is inside `.terraform` directory?

The directory stores:

- Provider plugins
- Provider binaries
- Module cache
- Backend information

Terraform uses these files internally.

---

## 📸 Screenshot 3

**terraform init**

<img width="1180" height="648" alt="image" src="https://github.com/user-attachments/assets/6ed0032c-bb7f-493c-87c7-398890798cf8" />

---

## 📸 Screenshot 4

**terraform plan**

<img width="1186" height="1414" alt="image" src="https://github.com/user-attachments/assets/590f0b19-7dd1-460d-b3dc-2b90a79e3a32" />

---

## 📸 Screenshot 5

**terraform apply**

<img width="1168" height="1364" alt="image" src="https://github.com/user-attachments/assets/7625d47c-ed2d-4d71-a18c-a6697c14a0be" />


---

## 📸 Screenshot 6

**S3 Bucket in AWS Console**

<img width="1906" height="842" alt="image" src="https://github.com/user-attachments/assets/5adf4ef7-bfd8-4a7f-9f8e-9a689355430b" />

---

# Task 4 – Add an EC2 Instance

Added an EC2 resource inside the same Terraform configuration.

```bash
# Resource: EC2 Instance
resource "aws_instance" "my_instance" {
  ami           = "ami-0d0463f1527996442" # Amazon Linux 2 for eu-west-1
  instance_type = "t3.micro"

  tags = {
    Name = "TerraWeek-Day1"
  }
}
```

Commands executed:

```bash
terraform fmt

terraform validate

terraform plan

terraform apply
```

Terraform detected that the bucket already existed in the Terraform State File, so only the EC2 instance was created.

---

## How did Terraform know the bucket already existed?

Terraform stores every managed resource inside **terraform.tfstate**.

During every execution Terraform compares:

- Current configuration
- Current state
- Actual cloud infrastructure

Since the bucket already existed in the state file, Terraform only created the EC2 instance.

---


## 📸 Screenshot 7

**terraform plan (EC2)**

<img width="990" height="530" alt="image" src="https://github.com/user-attachments/assets/5d49a052-a263-4746-ba90-708ff040f2e3" />

---

## 📸 Screenshot 8

**EC2 Instance Running**

<img width="2822" height="640" alt="image" src="https://github.com/user-attachments/assets/460b747b-2007-4089-b893-65f0fc98858f" />

---

# Task 5 – Understanding Terraform State

Useful commands:

```bash
terraform show

terraform state list

terraform state show aws_s3_bucket.my_bucket

terraform state show aws_instance.my_instance
```

---

## What does the State File store?

The Terraform state file stores:

- Resource IDs
- Resource ARNs
- Resource attributes
- Public IP addresses
- Bucket names
- Dependencies
- Provider information
- Metadata

Terraform uses this file as the source of truth.

---

## Why should you NEVER edit terraform.tfstate manually?

Manual changes can:

- Corrupt the state
- Cause resource drift
- Break future Terraform operations
- Lead to accidental deletion of infrastructure

---

## Why should terraform.tfstate NOT be committed to Git?

The state file may contain:

- Sensitive information
- Resource IDs
- Infrastructure metadata
- Secrets (depending on providers)

For production environments, Terraform Remote State should be used.

---

## 📸 Screenshot 9

**terraform state list**

<img width="1128" height="118" alt="image" src="https://github.com/user-attachments/assets/91142921-1327-4da6-9891-793047e1ff3f" />

---

## 📸 Screenshot 10

**terraform show**

<img width="1202" height="1392" alt="image" src="https://github.com/user-attachments/assets/b7b97e34-70b5-4630-bd0c-210bb22f93e1" />

---

# Task 6 – Modify Infrastructure

Changed the EC2 Name tag.

Terraform Plan showed:

```
Plan:
0 to add
1 to change
0 to destroy
```

The **~** symbol indicated an in-place update.

Applied the changes:

```bash
terraform apply
```

Verified the updated tag in AWS Console.

---

## 📸 Screenshot 11

**Updated EC2 Tag**

<img width="2882" height="442" alt="image" src="https://github.com/user-attachments/assets/26eae34a-223a-4841-a51f-644b6e5a072b" />

---

# Destroy Infrastructure

Executed:

```bash
terraform destroy
```

Terraform removed:

- EC2 Instance
- S3 Bucket

Verified everything was deleted from AWS Console.


---

# Terraform Commands Summary

| Command | Purpose |
|----------|---------|
| `terraform init` | Initialize Terraform working directory |
| `terraform fmt` | Format Terraform files |
| `terraform validate` | Validate configuration |
| `terraform plan` | Preview infrastructure changes |
| `terraform apply` | Create or update infrastructure |
| `terraform show` | Display Terraform state |
| `terraform state list` | List managed resources |
| `terraform state show` | Display resource details |
| `terraform destroy` | Delete all managed infrastructure |

---

# Key Learnings

- Learned the fundamentals of Infrastructure as Code.
- Understood why Terraform is widely used in DevOps.
- Installed and configured Terraform with AWS.
- Created an S3 bucket using Terraform.
- Added an EC2 instance using the same configuration.
- Learned the Terraform lifecycle: Init → Plan → Apply → Destroy.
- Understood how Terraform State tracks infrastructure.
- Learned why the state file is critical and should never be edited manually.
- Successfully provisioned and destroyed AWS resources using Terraform.

---
