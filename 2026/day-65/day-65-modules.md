# Day 65: Terraform Modules - Build Reusable Infrastructure

## Overview
Today I learned about Terraform modules - the way to package, reuse, and share infrastructure code. Just like functions in programming, modules allow us to write once and call many times.

---

## Task 1: Module Structure

### Directory Structure
```
terraform-modules/
├── main.tf              # Root module - calls child modules
├── variables.tf         # Root variables
├── outputs.tf           # Root outputs
├── providers.tf         # Provider config
└── modules/
    ├── ec2-instance/
    │   ├── main.tf       # EC2 resource definition
    │   ├── variables.tf  # Module inputs
    │   └── outputs.tf    # Module outputs
    └── security-group/
        ├── main.tf       # Security group resource definition
        ├── variables.tf  # Module inputs
        └── outputs.tf    # Module outputs
```

### Theory: Root Module vs Child Module

**Root Module**: The main configuration in the root directory where you run Terraform commands (`terraform init/plan/apply`). It's the entry point that orchestrates the entire infrastructure and calls child modules.

**Child Module**: A reusable module stored in a subdirectory (like `modules/ec2-instance/`) that is called by the root module. It encapsulates specific functionality and can be reused multiple times with different inputs.

---

## Task 2: Custom EC2 Module

### `modules/ec2-instance/variables.tf`
```hcl
variable "ami_id" {
  description = "AMI ID for the EC2 instance"
  type        = string
}

variable "instance_type" {
  description = "Instance type for EC2"
  type        = string
  default     = "t2.micro"
}

variable "subnet_id" {
  description = "Subnet ID for EC2 instance"
  type        = string
}

variable "security_group_ids" {
  description = "List of security group IDs"
  type        = list(string)
}

variable "instance_name" {
  description = "Name of the instance"
  type        = string
}

variable "tags" {
  description = "Additional tags for the instance"
  type        = map(string)
  default     = {}
}
```

### `modules/ec2-instance/main.tf`
```hcl
resource "aws_instance" "main" {
  ami                    = var.ami_id
  instance_type          = var.instance_type
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.security_group_ids

  tags = merge(
    {
      Name = var.instance_name
    },
    var.tags
  )
}
```

### `modules/ec2-instance/outputs.tf`
```hcl
output "instance_id" {
  description = "ID of the EC2 instance"
  value       = aws_instance.main.id
}

output "public_ip" {
  description = "Public IP of the EC2 instance"
  value       = aws_instance.main.public_ip
}

output "private_ip" {
  description = "Private IP of the EC2 instance"
  value       = aws_instance.main.private_ip
}
```

---

## Task 3: Custom Security Group Module

### `modules/security-group/variables.tf`
```hcl
variable "vpc_id" {
  description = "VPC ID for the security group"
  type        = string
}

variable "sg_name" {
  description = "Name of the security group"
  type        = string
}

variable "ingress_ports" {
  description = "List of ports to allow ingress"
  type        = list(number)
  default     = [22, 80]
}

variable "tags" {
  description = "Tags for the security group"
  type        = map(string)
  default     = {}
}
```

### `modules/security-group/main.tf`
```hcl
resource "aws_security_group" "main" {
  name        = var.sg_name
  description = "Security group created by module"
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

  tags = var.tags
}
```

### `modules/security-group/outputs.tf`
```hcl
output "sg_id" {
  description = "Security group ID"
  value       = aws_security_group.main.id
}
```

### Dynamic Blocks Explanation
This is my first time using a dynamic block. It loops over the `ingress_ports` list and generates separate ingress rules for each port. Instead of writing multiple ingress blocks manually, dynamic blocks allow us to generate repeated nested blocks from a list or map.

---

## Task 4: Root Module Wiring

### `providers.tf`
```hcl
provider "aws" {
  region = "ap-south-1"
}
```

### `variables.tf` (Root)
```hcl
variable "aws_region" {
  description = "AWS region"
  type        = string
  default     = "ap-south-1"
}

variable "environment" {
  description = "Environment name"
  type        = string
  default     = "dev"
}
```

### `main.tf` (Root)
```hcl
# Data source for AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}

# Local values for common tags
locals {
  common_tags = {
    Environment = var.environment
    Project     = "terraweek"
    ManagedBy   = "terraform"
  }
}

# --- USE PUBLIC REGISTRY VPC MODULE ---
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "terraweek-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-south-1a", "ap-south-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]

  enable_nat_gateway   = false
  enable_dns_hostnames = true

  tags = local.common_tags
}

# --- CALL SECURITY GROUP MODULE ---
module "web_sg" {
  source        = "./modules/security-group"
  vpc_id        = module.vpc.vpc_id
  sg_name       = "terraweek-web-sg"
  ingress_ports = [22, 80, 443]
  tags          = local.common_tags
}

# --- CALL EC2 MODULE - WEB SERVER ---
module "web_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t3.micro"
  subnet_id          = module.vpc.public_subnets[0]
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "terraweek-web"
  tags               = local.common_tags
}

# --- CALL EC2 MODULE - API SERVER ---
module "api_server" {
  source             = "./modules/ec2-instance"
  ami_id             = data.aws_ami.amazon_linux.id
  instance_type      = "t3.micro"
  subnet_id          = module.vpc.public_subnets[0]
  security_group_ids = [module.web_sg.sg_id]
  instance_name      = "terraweek-api"
  tags               = local.common_tags
}
```

### `outputs.tf` (Root)
```hcl
output "web_server_ip" {
  description = "Public IP of web server"
  value       = module.web_server.public_ip
}

output "api_server_ip" {
  description = "Public IP of API server"
  value       = module.api_server.public_ip
}

output "web_server_private_ip" {
  description = "Private IP of web server"
  value       = module.web_server.private_ip
}

output "api_server_private_ip" {
  description = "Private IP of API server"
  value       = module.api_server.private_ip
}

output "security_group_id" {
  description = "Security group ID"
  value       = module.web_sg.sg_id
}

output "vpc_id" {
  description = "VPC ID"
  value       = module.vpc.vpc_id
}

output "public_subnets" {
  description = "Public subnet IDs"
  value       = module.vpc.public_subnets
}

output "private_subnets" {
  description = "Private subnet IDs"
  value       = module.vpc.private_subnets
}
```

### Screenshots

**Screenshot 1: Directory Structure (Task 1)**

<img width="612" height="814" alt="image" src="https://github.com/user-attachments/assets/6095562e-cd55-4efc-bce6-1d0224133b60" />


**Screenshot 2: EC2 Instances in AWS Console (Task 4)**

<img width="2872" height="400" alt="image" src="https://github.com/user-attachments/assets/b7bbdf07-8b89-4435-99b3-3a39c4636198" />


**Screenshot 3: Terraform Apply Output (Task 4)**

<img width="1274" height="958" alt="image" src="https://github.com/user-attachments/assets/23c8205b-4d24-4d9b-b6c9-b8a4e65dc2ae" />


**Screenshot 4: Terraform State List (Task 5)**

<img width="1130" height="682" alt="image" src="https://github.com/user-attachments/assets/f52faca8-7d5e-40a3-8bf3-5278f7f6ab1c" />


---

## Task 5: Public Registry Module

### Registry Module Used
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"
  # ... configuration
}
```

### Where Terraform Downloads Modules
Terraform downloads registry modules to the `.terraform/modules/` directory. When you run `terraform init`, it downloads all required modules to this cache.

```bash
ls -la .terraform/modules/
```

### Module Prefixes in State
From `terraform state list`:

- `module.vpc.` - Registry VPC module
- `module.web_server.` - Custom EC2 module
- `module.api_server.` - Custom EC2 module (same module called twice)
- `module.web_sg.` - Custom security group module

### Comparison: Hand-written VPC vs Registry VPC Module

| Aspect | Hand-written VPC | Registry VPC Module |
|---|---|---|
| Resources Created | ~5-6 resources | ~15-20 resources |
| Code Lines | ~50-60 lines | ~15-20 lines |
| Maintenance | Manual | Community maintained |
| Features | Basic | Advanced (NAT, route tables, etc.) |
| Flexibility | Customizable | Highly configurable via inputs |

**Resources created by hand-written VPC:**
- VPC
- Subnet
- Internet Gateway
- Route Table
- Route Table Association

**Resources created by registry VPC module:**
- VPC
- Multiple public subnets (2)
- Multiple private subnets (2)
- Internet Gateway
- NAT Gateway (if enabled)
- Route Tables (public and private)
- Route Table Associations
- And more...

---

## Task 6: Module Versioning and Best Practices

### Version Pinning Examples

**Exact Version:**
```hcl
version = "5.1.0"
```

**Constraint (`~> 5.0`) - Any 5.x version:**
```hcl
version = "~> 5.0"
```

**Version Range:**
```hcl
version = ">= 5.0, < 6.0"
```

### Upgrading Modules
```bash
terraform init -upgrade
```

### Five Module Best Practices

**1. Always Pin Versions for Registry Modules**
Always specify exact or constrained versions for public modules to prevent unexpected changes when module authors release new versions. For example: `version = "~> 5.0"` or `version = "5.1.0"`. This ensures your infrastructure doesn't break due to incompatible updates.

**2. Keep Modules Focused - One Concern Per Module**
Each module should do one thing well. For example, the EC2 module only handles EC2 instances, and the Security Group module only handles security groups. This makes modules easier to understand, test, and maintain. If a module does too much, it becomes harder to reuse and debug.

**3. Use Variables for Everything, Hardcode Nothing**
All configurable values should be variables. No hardcoded AMI IDs, instance types, or names. This makes modules truly reusable across different environments (dev, staging, production) and different regions. Hardcoded values break reusability.

**4. Always Define Outputs for Important Attributes**
Expose resource attributes that callers might need. For example: `instance_id`, `public_ip`, `sg_id`. This allows the root module to reference and use these values. Without outputs, calling modules cannot access or use the resources created by the module.

**5. Add Documentation to Every Custom Module**
Include a `README.md` file in every custom module with:
- Purpose of the module
- All inputs (variables) with descriptions
- All outputs
- Usage examples
- Dependencies

This makes modules shareable and maintainable by team members. Documentation saves time and reduces errors when others (or future you) use the module.

---

## Commands Used
```bash
# Initialize Terraform and download modules
terraform init

# Plan changes
terraform plan

# Apply changes
terraform apply -auto-approve

# List resources in state
terraform state list

# Upgrade modules
terraform init -upgrade

# Destroy everything
terraform destroy -auto-approve
```

---

## Key Learnings
- **Modules are reusable**: Write once, use multiple times
- **Local vs Registry modules**: Local for custom code, Registry for community modules
- **Dynamic blocks**: Generate repeated blocks from lists
- **Module outputs**: Expose resource attributes to callers
- **Version pinning**: Critical for production stability
- **Module state**: Resources are prefixed with `module.<name>.` in state
- **Best practices**: Pin versions, keep focused, use variables, define outputs, add documentation
