# Day 67 — TerraWeek Capstone: Multi-Environment Infrastructure with Workspaces and Modules

## Project Overview
This repository contains a production-grade Terraform configuration designed to deploy a multi-environment AWS infrastructure using a single codebase. By combining **Custom Child Modules** and **Terraform Workspaces**, we isolate and manage three distinct environment tiers—`dev`, `staging`, and `prod`—safely scaling resource footprints without duplicate code patterns.

### My 7-Day TerraWeek Journey

| Day | Concepts Learned |
| :--- | :--- |
| **Day 61** | IaC, HCL, init/plan/apply/destroy, state basics |
| **Day 62** | Providers, resources, dependencies, lifecycle |
| **Day 63** | Variables, outputs, data sources, locals, functions |
| **Day 64** | Remote backend, locking, import, drift |
| **Day 65** | Custom modules, registry modules, versioning |
| **Day 66** | EKS with modules, real-world provisioning |
| **Day 67** | Workspaces, multi-env, capstone project |

---

## Task 1: Learn & Initialize Terraform Workspaces

### Initialization & Workspace Execution Commands
```bash
# Initialize root directory repository context
mkdir -p terraweek-capstone && cd terraweek-capstone
terraform init

# Check active default context
terraform workspace show

# Build distinct lifecycle target workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# List all local structural workspaces
terraform workspace list

# Switch baseline pointers to the development lane
terraform workspace select dev
```

### Theoretical Answers

#### 1. What does `terraform.workspace` return inside a config?
It returns a **string** containing the exact name of the currently active Terraform workspace (e.g., `default`, `dev`, `staging`, or `prod`). This value can be evaluated dynamically inside configurations to safely map different resource sizes, tags, or names based on the active runtime environment.

#### 2. Where does each workspace store its state file?
* **Local Backend:** Stored inside a root subfolder tracking path: `.terraform.tfstate.d/<workspace-name>/terraform.tfstate`.
* **Remote Backend (e.g., S3):** Stored under a workspace prefix key pattern inside your S3 bucket: `env:/<workspace-name>/<backend-key-path>`. (Note: The `default` workspace remains located directly at the root of the specified key path).

#### 3. How is this different from using separate directories per environment?
Workspaces enable you to manage completely separate environmental lifecycles using **one single codebase**, dynamically shifting resource layouts via parameters. Conversely, separate directory patterns require physically isolating configurations into distinct folder structures (e.g., a `dev/` folder and a `prod/` folder). While separate folders create a completely strict safety boundary that reduces accidental operational blunders, workspaces reduce folder tracking clutter and enforce consistency across environments.

---

### 📸 Task 1 Screenshots Placeholder

<img width="1152" height="246" alt="image" src="https://github.com/user-attachments/assets/c2624607-7702-458b-9d64-05d61aba2073" />


## Task 2: Project Structure Setup

### Repository File Architecture
```text
terraweek-capstone/
  ├── .gitignore                # Protects states, backups, and local variable secrets
  ├── providers.tf              # Engine source versioning rules and target AWS cloud configurations
  ├── locals.tf                 # Formulates workspace-driven naming parameters and generic tags
  ├── variables.tf              # Defines root level configurations catching environment schemas
  ├── main.tf                   # Root driver orchestration - links and invokes custom child modules
  ├── outputs.tf                # Declares clean tracking output blocks returned back on screen
  ├── dev.tfvars                # Unique execution variable inputs assigned for the Dev tier
  ├── staging.tfvars            # Unique execution variable inputs assigned for the Staging tier
  ├── prod.tfvars               # Unique execution variable inputs assigned for the Production tier
  └── modules/                  # Isolated structural child layers
      ├── vpc/
      │   ├── main.tf
      │   ├── variables.tf
      │   └── outputs.tf
      ├── security-group/
      │   ├── main.tf
      │   ├── variables.tf
      │   └── outputs.tf
      └── ec2-instance/
          ├── main.tf
          ├── variables.tf
          └── outputs.tf
```

### Root `.gitignore` Configuration
```text
.terraform/
*.tfstate
*.tfstate.backup
*.tfvars
.terraform.lock.hcl
```

### Theoretical Answer: Why is this structure considered best practice?
* **Separation of Concerns (SoC):** Breaking monolith configurations out into discrete descriptive files (`providers`, `variables`, `outputs`, `locals`) makes codebases significantly easier to read, scale, and maintain over time.
* **DRY Architecture (Don't Repeat Yourself):** Abstracting common foundational objects like networks, compute layers, and firewalls into structural templates within `modules/` lets you call them repeatedly across multiple environments via input properties instead of duplicating identical code blocks.
* **Data Security & Git Integrity:** Keeping active configuration caches, operational locks, and parameter values hidden away behind the `.gitignore` wrapper keeps confidential operational credentials, sensitive workspace histories, and backend secrets from leaking onto public Git platforms.

---

### 📸 Task 2 Screenshots Placeholder

<img width="574" height="1224" alt="image" src="https://github.com/user-attachments/assets/4be4366f-0acb-456e-8bbe-d1f09d38fb5a" />


---

## Task 3: Build the Custom Modules

### 1. VPC Module Configuration (`modules/vpc/`)

#### `modules/vpc/variables.tf`
```hcl
variable "cidr" { type = string }
variable "public_subnet_cidr" { type = string }
variable "environment" { type = string }
variable "project_name" { type = string }
```

#### `modules/vpc/main.tf`
```hcl
resource "aws_vpc" "this" {
  cidr_block           = var.cidr
  enable_dns_hostnames = true
  tags = {
    Name        = "${var.project_name}-${var.environment}-vpc"
    Environment = var.environment
    Project     = var.project_name
  }
}

resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id
  tags = {
    Name        = "${var.project_name}-${var.environment}-igw"
    Environment = var.environment
    Project     = var.project_name
  }
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.this.id
  cidr_block              = var.public_subnet_cidr
  map_public_ip_on_launch = true
  tags = {
    Name        = "${var.project_name}-${var.environment}-public-subnet"
    Environment = var.environment
    Project     = var.project_name
  }
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.this.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.this.id
  }
  tags = {
    Name        = "${var.project_name}-${var.environment}-public-rt"
    Environment = var.environment
    Project     = var.project_name
  }
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}
```

#### `modules/vpc/outputs.tf`
```hcl
output "vpc_id" { value = aws_vpc.this.id }
output "subnet_id" { value = aws_subnet.public.id }
```

---

### 2. Security Group Module Configuration (`modules/security-group/`)

#### `modules/security-group/variables.tf`
```hcl
variable "vpc_id" { type = string }
variable "ingress_ports" { type = list(number) }
variable "environment" { type = string }
variable "project_name" { type = string }
```

#### `modules/security-group/main.tf`
```hcl
resource "aws_security_group" "this" {
  name        = "${var.project_name}-${var.environment}-sg"
  description = "Dynamic Ingress SG managed by Terraform for environment: ${var.environment}"
  vpc_id      = var.vpc_id

  dynamic "ingress" {
    for_each = var.ingress_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name        = "${var.project_name}-${var.environment}-sg"
    Environment = var.environment
    Project     = var.project_name
  }
}
```

#### `modules/security-group/outputs.tf`
```hcl
output "sg_id" { value = aws_security_group.this.id }
```

---

### 3. EC2 Instance Module Configuration (`modules/ec2-instance/`)

#### `modules/ec2-instance/variables.tf`
```hcl
variable "ami_id" { type = string }
variable "instance_type" { type = string }
variable "subnet_id" { type = string }
variable "security_group_ids" { type = list(string) }
variable "environment" { type = string }
variable "project_name" { type = string }
```

#### `modules/ec2-instance/main.tf`
```hcl
resource "aws_instance" "this" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  tags = {
    Name        = "${var.project_name}-${var.environment}-server"
    Environment = var.environment
    Project     = var.project_name
  }
}
```

#### `modules/ec2-instance/outputs.tf`
```hcl
output "instance_id" { value = aws_instance.this.id }
output "public_ip" { value = aws_instance.this.public_ip }
```

---

### 📸 Task 3 Screenshots Placeholder

<img width="1188" height="206" alt="image" src="https://github.com/user-attachments/assets/d93e7bbe-1119-4838-8117-af960ec25a02" />


## Task 4: Root Module & Workspace Configurations

### Root File Setup

#### `providers.tf`
```hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

#### `locals.tf`
```hcl
locals {
  environment = terraform.workspace
  name_prefix = "${var.project_name}-${local.environment}"
  common_tags = {
    Project     = var.project_name
    Environment = local.environment
    ManagedBy   = "Terraform"
    Workspace   = terraform.workspace
  }
}
```

#### `variables.tf`
```hcl
variable "project_name" {
  type    = string
  default = "terraweek"
}

variable "vpc_cidr" { type = string }
variable "subnet_cidr" { type = string }
variable "instance_type" { type = string }

variable "ingress_ports" {
  type    = list(number)
  default = []
}
```

#### `main.tf`
```hcl
module "vpc" {
  source             = "./modules/vpc"
  cidr               = var.vpc_cidr
  public_subnet_cidr = var.subnet_cidr
  environment        = local.environment
  project_name       = var.project_name
}

module "security_group" {
  source        = "./modules/security-group"
  vpc_id        = module.vpc.vpc_id
  ingress_ports = var.ingress_ports
  environment   = local.environment
  project_name  = var.project_name
}

module "ec2_instance" {
  source             = "./modules/ec2-instance"
  ami_id             = "ami-0c7217cdde317cfec" # Standard Ubuntu 22.04 LTS AMI in us-east-1
  instance_type      = var.instance_type
  subnet_id          = module.vpc.subnet_id
  security_group_ids = [module.security_group.sg_id]
  environment        = local.environment
  project_name       = var.project_name
}
```

#### `outputs.tf`
```hcl
output "environment_details" {
  value = {
    workspace     = local.environment
    vpc_id        = module.vpc.vpc_id
    subnet_id     = module.vpc.subnet_id
    ec2_public_ip = module.ec2_instance.public_ip
  }
  description = "Key resource deployment addresses per active environment"
}
```

### Variable Mappings Profiles (`.tfvars`)
*(Note: Adjusted to map `t3.micro` sizes to comply with real-world target cloud sandbox policies while keeping port setups distinctly mapped per environment.)*

#### `dev.tfvars`
```hcl
vpc_cidr      = "10.0.0.0/16"
subnet_cidr   = "10.0.1.0/24"
instance_type = "t3.micro"
ingress_ports = [22, 80]
```

#### `staging.tfvars`
```hcl
vpc_cidr      = "10.1.0.0/16"
subnet_cidr   = "10.1.1.0/24"
instance_type = "t3.micro"
ingress_ports = [22, 80, 443]
```

#### `prod.tfvars`
```hcl
vpc_cidr      = "10.2.0.0/16"
subnet_cidr   = "10.2.1.0/24"
instance_type = "t3.micro"
ingress_ports = [80, 443]
```

---

## Task 5: Deploy and Verify Environments

### Multi-Environment Apply Cycle Commands
```bash
# Deploy Dev
terraform workspace select dev
terraform apply -var-file="dev.tfvars" -auto-approve

# Deploy Staging
terraform workspace select staging
terraform apply -var-file="staging.tfvars" -auto-approve

# Deploy Prod
terraform workspace select prod
terraform apply -var-file="prod.tfvars" -auto-approve
```

### Verification Questions

**Are all three environments completely isolated from each other?**

Yes. They are fully isolated logically and physically. Each tier tracks through an independent, sandboxed state file slice within the workspace parameters. Inside AWS, they live inside completely distinct VPC network rings with non-overlapping subnets (`10.0.x.x`, `10.1.x.x`, and `10.2.x.x`), preventing cross-network communication.

---

### 📸 Task 5 Verification Dashboard Placeholders

<img width="1542" height="838" alt="image" src="https://github.com/user-attachments/assets/7e255eba-04a7-4221-8d84-7eb1180b2b3a" />

<img width="2870" height="502" alt="image" src="https://github.com/user-attachments/assets/8a69f4d3-cadf-4d45-b14f-cd2cd0bcaaf2" />

<img width="2864" height="216" alt="image" src="https://github.com/user-attachments/assets/57a3e380-987b-46ed-8b05-dbb24161d5e1" />


---

## Task 6: Cloud Infrastructure Best Practices Guide

* **File Layout Isolation:** Separate configuration blocks logically across `main.tf`, `variables.tf`, `outputs.tf`, and `locals.tf` to maximize code readability and tracking.
* **State Management:** Secure production tracking architectures by leveraging remote state storage engines (like AWS S3) combined with active locking mechanics (via DynamoDB) to avoid data corruption.
* **Parameterize Configurations:** Avoid hardcoded variables. Drive structural variations dynamically via environment `.tfvars` files.
* **Encapsulated Modules:** Build isolated child modules focused on single infrastructure jobs, and explicitly define input parameters and outputs.
* **Workspace Management:** Use workspaces to maintain simple infrastructure lifecycle stages. For critical, highly varied architectures, consider transitioning to a separate directory configuration layout to avoid accidental overrides.
* **Secure Code Practices:** Ensure your root `.gitignore` parameters prevent active state files and local variable profiles from entering version tracking.
* **Pre-Commit Code Audits:** Run `terraform fmt -recursive` and `terraform validate` routinely before push hooks to enforce compliance and catch errors early.
* **Consistent Naming & Tagging:** Establish a uniform resource prefix strategy (e.g., `<project>-<environment>-<resource>`) and enrich instances with robust contextual ownership tracking tags.
* **Environment Lifecycle Cleanup:** Automatically destroy developer sandboxes and non-production testing footprints when idle to minimize cloud spend.

---

## Task 7: Infrastructure Destruction & Workspace Erasure

### Reverse-Order Teardown Chain Commands
```bash
# Destroy Production
terraform workspace select prod
terraform destroy -var-file="prod.tfvars" -auto-approve

# Destroy Staging
terraform workspace select staging
terraform destroy -var-file="staging.tfvars" -auto-approve

# Destroy Development
terraform workspace select dev
terraform destroy -var-file="dev.tfvars" -auto-approve

# Clear the supplementary environment tracks completely
terraform workspace select default
terraform workspace delete dev
terraform workspace delete staging
terraform workspace delete prod
```

### Verification Challenge Check

**Is your AWS account completely clean?**

Yes. Running these destructive workflows completely removes all custom instances, routing structures, firewalls, and gateway attachments from the AWS dashboard, leaving zero lingering operational footprints or orphan costs.

---

### 📸 Task 7 Final Clean Screenshot Placeholder
> **[INSERT SCREENSHOT HERE]**
> *Screenshot Description: Capture your terminal workspace screen showing the final workspace deletions, alongside a clean AWS console showing instances successfully terminated.*
